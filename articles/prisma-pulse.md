---
title: "Prisma Pulse でデータ更新をリアルタイムで検知する"
emoji: "🔊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Prisma"]
published: true
publication_name: "cybozu_frontend"
---

## Prisma Pulse

先日、ORM である Prisma から、新しいサービスである Prima Pulse が発表されました。

https://www.prisma.io/blog/introducing-pulse-jtu4UPC8ujy4

対応するデータベースの変更を検知し、リアルタイムで変更を通知するためのサービスとのことです。

申し込んでいた Early Access への招待が届いたので、実際に試してみました。

> Early Access 中なので、パブリックな記事を書いていいのか不安でしたが、
> Prisma 側に確認したところ「問題ないよ！」との回答を頂きました。ありがとうございます。

### 現状の制約

Early Access 段階では次の条件を満たす必要があるようです。

- Postgres > 12
- DB が Public アクセスできること
- DB で Logical Replication が有効であること
- 管理者権限でアクセス可能なこと

## DB の準備と Prisma Cloud Project の設定

とりあえずアクセス可能なデータベースが必要です。Supabase に対応しているようなので、そちらで試してみましょう。

適当にプロジェクトを作成後、SQL Editor から パブリケーションを作成します。

![](/images/prisma-pulse/publication.png)

実際には次に Pulse を有効にするテーブルの選択が必要ですが、まだテーブルが存在しないので後ほど対応します。Prisma から接続する際に必要になるため、接続用 URI を取得しておきます。

![](/images/prisma-pulse/uri.png)

Early Access の対象であれば、Prisma Cloud Projects から Pulse の設定が可能です。(URI を入力するだけなので説明は割愛します。)

うまくいけば、次のように Pulse is enabled となります。

![](/images/prisma-pulse/enabled.png)

アプリケーションからの接続で利用するため、別途 API KEY も発行してコピーしておきます。

## アプリケーションの準備

Prisma のドキュメントに沿って簡易的なプロジェクトを作っておきます。
https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-postgresql

```sh
npm init -y
npm install prisma typescript ts-node @types/node --save-dev
npx tsc --init
npx prisma init
```

追加で Pulse 関係で必要となるパッケージも入れておきます。

```sh
npm install @prisma/extension-pulse
npm install dotenv
```

Pulse 設定を適用した Prisma Client を作成します。

```ts:prsima.ts
import "dotenv/config";
import { PrismaClient } from "@prisma/client";
import { withPulse } from "@prisma/extension-pulse";

const apiKey: string =
  process.env.PULSE_API_KEY !== undefined ? process.env.PULSE_API_KEY : "";

export const prisma = new PrismaClient().$extends(
  withPulse({ apiKey: apiKey })
);
```

実験用でデータは何でもよいので、Prisma schema のドキュメントにある[Example](https://www.prisma.io/docs/concepts/components/prisma-schema#example)のまま User / Post の model を定義し、`npx prisma migrate dev` をしておきます。

Supabase 側でテーブルが増えているのを確認できれば OK です。

![](/images/prisma-pulse/table.png)

## Pulse から変更を受け取る

では実際に Pulse を Pulse してみましょう。

Early Access 向けのドキュメント記載のコードそのままですが、`user-subscription.ts` というファイルを作成し、Pulse からの通知を受け取ります。

```ts:user-subscription.ts
import { prisma } from "./prisma";

async function main() {
  const subscription = await prisma.user.subscribe({});

  if (subscription instanceof Error) {
    throw subscription;
  }

  for await (const event of subscription) {
    console.log("just received an event:", event);
  }
}

main();
```

`subscribe()` の戻り値は AsyncIterator になっており、for await...of で処理できます。

では実行してみましょう。

```sh
px tsx user-subscription.ts
```

![](/images/prisma-pulse/run.png)

ここで Hello from ... で出力される ID は、トラブル発生時のサポートなどで利用するらしいです。

この状態で、Supabase 側の GUI から User テーブルにレコードを INSERT してみます。

![](/images/prisma-pulse/insert.png)

すると..!!

![](/images/prisma-pulse/run.png)

何も起きませんでした。Replication の設定をしてないからですね。

Supabase の Database → Replication から User テーブルを Source として有効化しておきましょう。

![](/images/prisma-pulse/rep.png)

再度レコードを挿入してみると...

![](/images/prisma-pulse/yeah.png)

更新を検知できましたね！

## 所感

導入自体は想像より遥かに簡単でした。ほぼなんの苦労もなく `subscribe()` までたどり着けてしまいました。

ただ、実際に採用することを考えると、色々と懸念事項がありそうな印象です。特に、`subscribe()` で受け取れなかった場合のケアが必要かどうかは重要になりそうで、現状ではそういったケースではリトライなども無く全て破棄されてしまうようです。

「会員登録完了時にメールを送る」といったケースで利用した場合、タイミングによっては一生メールが届かないといった自体に繋がるため、リカバリーのためのバッチ処理なども別途必要になってきそうです。

ただ、Pulse を利用した場合には、一定間隔での cron での処理などと違い、リアルタイムでの処理が可能なため、ユーザー体験の向上に繋がるかもしれません。また、ポーリングが不要になることで DB 負荷軽減にも繋がります。サービスとして重要視すべき点によっては一役買うこともありそうです。
