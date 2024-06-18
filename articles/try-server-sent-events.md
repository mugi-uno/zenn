---
title: "Server-Sent Events を複数パターンで実装してみる"
emoji: "⚡️⚡️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["フロントエンド", "sse"]
published: false
publication_name: "cybozu_frontend"
---

# Server-Sent Events (SSE)

目新しい技術というわけではありませんが、最近 Server-Sent Events (SSE) について言及する記事をよく見かけます。

何番煎じかはわかりませんが、個人的に興味があることと、正直触ってみたことがなかったので、コードを書きつつ調べてみました。

※本記事で登場するサンプルコードは次のリポジトリで公開しています。
https://github.com/mugi-uno/sse-sandbox

## SSE とは

SSE 自体を解説する記事は無数に存在するため詳細な説明は割愛しますが、簡単に言うと、サーバーからクライアントへ一方向の Push 通信を行うための仕組みです。

MDN にもページが存在するため、参考になります。
https://developer.mozilla.org/ja/docs/Web/API/Server-sent_events/Using_server-sent_events

独自プロトコルを必要とせず、HTTP/1.1 でも動作するのも特徴です。

## SSE の歴史

wikipedia に SSE に関するページが存在し、次のような記述があります。
https://ja.wikipedia.org/wiki/サーバー送信イベント

> SSE メカニズムは、2004 年に始まった「WHATWG Web Applications 1.0」提案の一部として Ian Hickson により初めて規程された

というわけで、SSE 自体は特別モダンな技術というわけではありません。

2014 年時点で Rails を用いた SSE の実装記事があったりもします。

https://www.school.ctc-g.co.jp/columns/masuidrive/masuidrive15.html

## 対話型 AI 向け IF での需要

SSE が急に注目されるようになった理由は、ChatGPT を中心とした対話型 AI での UI の影響が大きいと思われます。回答が断片的に少しずつ返ってくるアレです。

![ChatGPT での少しずつ返ってくるレスポンス例](/images/try-sse/chatgpt-sse.gif)

実際 ChatGPT では SSE が活用されているようで、API でも `stream` オプションが存在しています。

https://platform.openai.com/docs/api-reference/chat/create

![ChatGPT API での stream オプション](/images/try-sse/chatgpt-api.png)

参考

- [How ChatGPT Uses Server-Sent Events to Stream Real-Time Conversation - DEV Community](https://dev.to/rohitdhas/how-chatgpt-uses-server-sent-events-to-stream-real-time-conversation-3976)
- [Real-time Web with Server Sent Events | by Maryann Gitonga | Medium](https://medium.com/@maryanngitonga/real-time-web-with-server-sent-events-84a335ac1856)

# SSE 実装 / サーバー側

というわけで、実際にサーバー側・クライアント側の両方を作ってみます。
まずはサーバー側です。

実装についても MDN の記述が大変参考になるので、まずはそちらを見てみましょう。

https://developer.mozilla.org/ja/docs/Web/API/Server-sent_events/Using_server-sent_events#サーバからのイベントの送信

> イベントを送信するサーバー側のスクリプトは、 MIME タイプ  text/event-stream  で応答する必要があります

ということで、指定の MIME タイプでレスポンスを返せば良さそうです。

また、Server-Sent Events で送信するデータの内容は、HTML Standard で仕様として定められています。

https://html.spec.whatwg.org/multipage/server-sent-events.html#event-stream-interpretation

ざっくり次のようなものです。

- 空行で区切る
- `:` 始まりはコメント
- 先頭に `data:` のような形で Prefix を持つ
- `event:` を使うことで、イベントの種類を分けたりもできる

これを満たすように実装してみましょう。

## デプロイ環境に応じた SSE 対応状況

せっかくなのでどこかにデプロイして動かしたいところですが、まずはデプロイ先が SSE をサポートしてるかを把握したほうがよさそうです。

いくつかの Edge 環境について調べてみました。

### AWS / Lambda

Lambda は、Streaming に対応しているようです。
https://aws.amazon.com/jp/blogs/compute/introducing-aws-lambda-response-streaming/

次の記事も参考になります
https://zenn.dev/microcms/articles/aws-serverless-http-response-streaming

### Cloudflare Workers

Worker AI のサンプルなどで `text/event-stream` が登場するため、少なくとも動くようです。
https://blog.cloudflare.com/ru-ru/workers-ai-streaming-ja-jp

お世話になる機会も多い DevelopersIO でも、Cloudflare Workers から SSE を返す実装記事があり、参考になりました。
https://dev.classmethod.jp/articles/cloudflare-workers-langchain-stream/

### Deno Deploy

公式ブログにコード例を含むサンプルが掲載されています。
https://deno.com/blog/deploy-streams#server-sent-events

## Cloudflare Workers での実装

Cloudflare Workers での、簡単な SSE の実装は次のような形となります。

```typescript
const TEXT =
  "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.";

export default {
  async fetch(_request, _env, _ctx): Promise<Response> {
    const { readable, writable } = new TransformStream();
    const writer = writable.getWriter();
    const encoder = new TextEncoder();
    let chunks = TEXT.split(" ");

    const intervalId = setInterval(() => {
      const chunk = chunks.shift();
      writer.write(encoder.encode(`data: ${chunk}\n\n`));
      if (chunks.length === 0) {
        clearInterval(intervalId);
      }
    }, 100);

    return new Response(readable, {
      headers: { "Content-Type": "text/event-stream" },
    });
  },
} satisfies ExportedHandler<Env>;
```

一定長のテキストを 100 ms ごとに単語ごとに送信しています。特徴は次のとおりです。

- `'Content-Type': 'text/event-stream'` で MIME タイプを指定
- `ReadableStream` (`TransformStream`) を使ってデータをストリーム送信

## Deno Deploy での実装例

Deno Deploy で同様の実装を行ってみた例が次のコードです。

```typescript
import { FreshContext } from "$fresh/server.ts";
import { ServerSentEventStreamTarget } from "https://deno.land/std@0.190.0/http/server_sent_event.ts";

const TEXT =
  "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.";

export const handler = (_req: Request, _ctx: FreshContext): Response => {
  const sse = new ServerSentEventStreamTarget({});

  const chunks = TEXT.split(" ");

  const intervalId = setInterval(() => {
    const chunk = chunks.shift()!;

    sse.dispatchMessage(chunk);

    if (chunks.length === 0) {
      clearInterval(intervalId);
      sse.close();
    }
  }, 100);

  return sse.asResponse();
};
```

大きく違うのは、`ServerSentEventStreamTarget` の存在です。SSE 向けに用意されており、`text/event-stream` の MIME タイプの設定は自動で行われ、`dispatchMessage` などの API を利用することで、`data:` などの Prefix を独自で付与する必要もありません。便利ですね。

## Hono & Cloudflare Workers

先の Cloudflare Workers の例では独自でレスポンスを構築しましたが、Deno Deploy における `ServerSentEventStreamTarget` のように、Hono を用いることでより簡単に SSE を Cloudflare Workers で実現できます。

参考: https://azukiazusa.dev/blog/hono-streaming-response/

Hono には Streaming Helper と呼ばれる API 群が用意されており、その中の `streamSSE` を使って実装してみると次のような形となります。
https://hono.dev/helpers/streaming#streamsse

```typescript
import { Hono } from "hono";
import { streamSSE } from "hono/streaming";

const TEXT =
  "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.";

const app = new Hono();

app.get("/sse", (c) => {
  return streamSSE(c, async (stream) => {
    let chunks = TEXT.split(" ");

    while (chunks.length > 0) {
      const chunk = chunks.shift()!;
      await stream.sleep(100);
      await stream.writeSSE({ data: chunk });
    }
  });
});
```

Deno Deploy と同様、MIME タイプや Prefix などを意識する必要がなく、かなりシンプルに実現できていますね。

# SSE 実装 / クライアント側

続いて、ここまでで実装したサーバー側のエンドポイントに対して、受信するクライアント側コードを実装してみましょう。

## EventSource

クライアント側での手っ取り早い実装方法としては `EventSource` を使う方法があります。

https://developer.mozilla.org/ja/docs/Web/API/EventSource

`EventSource` は SSE の受信に特化したインタフェースです。大半のランタイムで利用可能で、`data`・`event` などに応じたデータのパースも自動で行ってくれます。

先に紹介したサーバー側の実装（テキストを単語ごとに SSE 送信）に対して、React で `EventSource` を利用して受信して表示するコンポーネント例が次の通りです。

```tsx
import { useRef, useState } from "react";

type Props = {
  url: string;
};

export const View = ({ url }: Props) => {
  const [text, setText] = useState("");
  const eventSourceRef = useRef<EventSource | null>(null);

  const handleClick = () => {
    if (eventSourceRef.current) return;

    const eventSource = new EventSource(url);

    eventSourceRef.current = eventSource;

    eventSource.onmessage = (event) => {
      setText((prevText) =>
        prevText ? prevText + " " + event.data : prevText + event.data
      );
    };
  };

  return (
    <>
      <button onClick={handleClick}>Run</button>
      <pre>{text}</pre>
    </>
  );
};
```

ボタンをクリックしたタイミングで、指定 URL に対して `EventSource` を作成し、受け取ったメッセージを表示しています。データのパースは自動で行われ、`onmessage` や `addEventListener` でデータの受信時に処理を実行できます。

実際に、Cloudflare Workers / Deno Deploy / Cloudflare Workers + Hono で作成した SSE エンドポイントに対して、このコンポーネントで受信してみると、次のような動きになります。

![SSEの動作例](/images/try-sse/sse-sample.gif)

受け取ったものから表示されており、AI の回答のような雰囲気が出てますね。

## `EventSource` の課題

`EventSource` でかなり簡単にクライアント側を実装できましたが、`EventSource` にはいくつかの課題が存在しており、場合によっては制約となります。

制約については次の記事も大変参考になりました。
https://zenn.dev/teramotodaiki/scraps/f016ed832d6f0d

### 自動的に再接続する

`EventSource` は、接続が終了すると、自動的に再接続するという仕様になっています。

再接続間隔などに関しての仕様は HTML Standard 上に記述があります。

https://html.spec.whatwg.org/multipage/server-sent-events.html#sse-processing-model

> 2. Wait a delay equal to the reconnection time of the event source.
> 3. Optionally, wait some more. In particular, if the previous attempt failed, then user agents might introduce an exponential backoff delay to avoid overloading a potentially already overloaded server. Alternatively, if the operating system has reported that there is no network connectivity, user agents might wait for the operating system to announce that the network connection has returned before retrying.

簡単に整理すると、接続が終了した後、まず `EventSource` には `retry` オプションが存在しており、その値に応じて待機します。加えて、ユーザーエージェント（≒ ブラウザ）に応じてサーバに負荷をかけないような形で指数関数的な関数を利用して待つ可能性がある、とのことです。

実際、先程のサンプルでもしばらく待っていると再度接続して無限にレスポンスを取得してしまいます。

![EventSourceが再接続する図](/images/try-sse/sse-retry.gif)

チャットの AI などの場合、発言ごとにレスポンスが完結する形が多いと思われるため、明示的に `EventSource` を閉じるようなイベントを用いるなど、何らかの工夫が必要となってきます。

### GET 限定

`EventSource` は GET リクエストにしか対応しておらず、POST などの他メソッドを使いたい場合には利用できません。

わかりやすい例としては、ChatGPT の API のエンドポイントは POST を要求するため、`EventSource` は使えなかったりします。

https://platform.openai.com/docs/api-reference/chat/create

![ChatGPTのAPIがPOSTを要求している図](/images/try-sse/chatgpt-post.png)

## `fetch()`

というわけで、`EventSource` が使えない場合の代替手段ですが、`fetch()` を使って SSE を受信する方法があります。

`EventSource` での実装相当のものを再現すると次のような形となります。

```tsx
"use client";

import { useState } from "react";

type Props = {
  url: string;
};

export const View = ({ url }: Props) => {
  const [text, setText] = useState("");

  const handleClick = async () => {
    const res = await fetch(url);
    const reader = res.body?.getReader()!;
    const decoder = new TextDecoder();

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      if (!value) continue;

      const lines = decoder.decode(value);
      const [type, raw] = lines.trim().split(": ");

      if (type === "data" && raw) {
        setText((prevText) =>
          prevText ? prevText + " " + raw : prevText + raw
        );
      }
    }
  };

  return (
    <>
      <button onClick={handleClick}>Run</button>
      <pre>{text}</pre>
    </>
  );
};
```

大きな差異となるのは、受け取ったデータを独自でパースする必要がある点です。今回は単純なテキストなので良いですが、本格的に活用し始めると複数のイベントに応じてシリアライズされた JSON などが含まれているケースが想定されるため、少々手間な作業になるかもしれません。

実際には次のような `fetch()` を Wrap したライブラリなどに頼るのも一つの手かもしれません。

e.g. Azure/fetch-event-source
https://github.com/Azure/fetch-event-source

# まとめ

というわけで、簡単ではありますが SSE の サーバーとクライアントを実際に作ってみたという実験記事でした。

バックエンド側については、Hono など、SSE をサポートしている機能を利用できると非常に簡単になる印象でした。
また、クライアント側においては、`EventSource` が使えるかどうかが一つ大きなポイントになりそうですね。
