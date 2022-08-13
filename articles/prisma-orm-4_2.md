---
title: "Prisma OpenTelemetry tracing で Prisma のボトルネックを追う"
emoji: "⏱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Prisma"]
published: false
---

## Prisma OpenTelemetry tracing

2022/8/9 に Prisma の 4.2.0 がリリースされました。
https://github.com/prisma/prisma/releases/tag/4.2.0

その中で、 `OpenTelemetry tracing` というプレビュー版機能が追加されています。

Prisma Client 経由で実行された処理についてログを取得し、
パフォーマンス低下時のボトルネック調査に役立てることができるようです。

トレーシングの概要は、別途解説したブログが公開されています。
https://www.prisma.io/blog/tracing-launch-announcement-pmk4rlpc0ll

トレーシング/メトリクスのための OSS である[OpenTelemetry](https://opentelemetry.io/) に準拠しているため、トレース結果は OpenTelemetry に互換のあるサービス/システムを用いて解析することができ、対応しているものとして [Jaeger](https://www.jaegertracing.io/)・[Honeycomb](https://www.honeycomb.io/trace/)・[Datadog](https://www.datadoghq.com/ja/) などが例として挙げられています。（[New Relic](https://newrelic.com/) とかもいけそう）

## トレーシングを試してみる

トレーシングの具体的な使い方の例は、新たに追加された公式ドキュメントが参考になります
https://www.prisma.io/docs/concepts/components/prisma-client/opentelemetry-tracing

この内容に従ってトレーシングを試してみます。

### 注意事項: パフォーマンスへの影響

ドキュメントの最初のほうに、トレーシングそのものがパフォーマンスに影響を与えてしまう可能性がある旨が[注意事項として記載](https://www.prisma.io/docs/concepts/components/prisma-client/opentelemetry-tracing#considerations-and-prerequisites)されています。

> If your application sends a large number of spans to a collector, this can have a significant performance impact

パフォーマンスが低下しているところにトレーシングを入れた結果で息の根を止めてしまった、、ということにならないよう注意は必要ですね。

### 依存関係の追加

必要な依存関係を追加します。Prisma は 4.2.0 以上のバージョンが入れば ok です。

```
npm install typescript ts-node @types/node -D
npm install prisma@latest -D
npm install @prisma/client@latest
npm install @prisma/instrumentation@latest
```

TS 周りのセットアップをします

```json:tsconfig.json
{
  "compilerOptions": {
    "sourceMap": true,
    "outDir": "dist",
    "strict": true,
    "lib": ["esnext"],
    "esModuleInterop": true,
    "downlevelIteration": true
  }
}
```

OpenTelemetry 周りで必要なパッケージも追加します。

```
npm install @opentelemetry/semantic-conventions @opentelemetry/exporter-trace-otlp-http @opentelemetry/instrumentation @opentelemetry/sdk-trace-base @opentelemetry/sdk-trace-node @opentelemetry/resources
npm install @opentelemetry/exporter-jaeger
```

インストール後、実験用に DB をセットアップします

```
npx prisma init --datasource-provider=sqlite
```

### スキーマファイルで Feature Flag を有効化

```diff js:schema.prisma
 generator client {
    provider = "prisma-client-js"
+  previewFeatures = ["tracing"]
 }
```

### ダミーデータを突っ込む

seed を使ってダミーのデータを投入します

```ts:prisma/seed.ts
import { Prisma, PrismaClient } from "@prisma/client";

const client = new PrismaClient();

const args = [...new Array(10000).keys()].map(
  (key): Prisma.UserCreateInput => ({
    name: `name_${key}`,
    email: `email${key}@example.com`,
  })
);

const insert = async () => {
  for (const arg of args) {
    await client.user.create({ data: arg });
  }
};

insert();
```

```
npx ts-node prisma/seed.ts
```

### 実行用ファイルを準備

OpenTelemetry セットアップ用のコードを作成します

```ts: setup.ts
import { SemanticResourceAttributes } from "@opentelemetry/semantic-conventions";
import { registerInstrumentations } from "@opentelemetry/instrumentation";
import { SimpleSpanProcessor } from "@opentelemetry/sdk-trace-base";
import { NodeTracerProvider } from "@opentelemetry/sdk-trace-node";
import { PrismaInstrumentation } from "@prisma/instrumentation";
import { JaegerExporter } from "@opentelemetry/exporter-jaeger";
import { Resource } from "@opentelemetry/resources";

// Configure the trace provider
const provider = new NodeTracerProvider({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: "example application",
  }),
});

const exporter = new JaegerExporter({ host: "localhost", port: 6832 });

provider.addSpanProcessor(new SimpleSpanProcessor(exporter));

// Register your auto-instrumentors
registerInstrumentations({
  tracerProvider: provider,
  instrumentations: [new PrismaInstrumentation()],
});

// Register the provider globally
provider.register();
```

そして、実行用のファイルを用意します。
先に作成した OpenTelemetry 用のセットアップをロードした上でダミークエリを発行します。

```ts:main.ts
import "./setup"; // setup OpenTracing

import { PrismaClient } from "@prisma/client";

const client = new PrismaClient();

const query = async () => {
  // run dummy queries
  await client.user.findMany({ where: { email: { contains: "1" } } });
  await client.user.findMany({ where: { email: { contains: "10" } } });
  await client.user.findMany({ where: { email: { contains: "100" } } });
};

query();
```

### Jaeger の起動

実行ファイルが整ったので、ローカルでトレース結果を可視化するため、OSS の Jaeger を利用します。

https://www.jaegertracing.io/

起動自体は Docker で一発です。

```
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.6
```

### トレースしてみる

やっと準備ができました。では実行してみましょう

```
npx ts-node ./main.ts
```

`http://localhost:16686/search` に接続し、 Jaerge のダッシュボードを確認してみます。

うまくトレースできていれば、左の `Service` から `example application` に切り替えられます。

![](/images/prisma-orm-4_2/jaeger-top.png)

上のグラフ内から、左上にある一際大きい丸をクリックして詳細にいくと、処理全体の時間に加え、細分化した際に個々に要した時間も確認できます。
さらに内容を見ていくことで、発行されたクエリまで追うことができます。

![](/images/prisma-orm-4_2/jaeger-detail.png)

---

今回は Prisma 自体の簡易的なセットアップから行ったため、いくつかプラスで必要な作業もありましたが、実際に Prisma を導入済みのケースであれば、OpenTelemetry 自体のセットアップのみで済むため、手軽に試してみることができそうです。

うまく利用できればパフォーマンス改善時に強い味方になりそうです。
