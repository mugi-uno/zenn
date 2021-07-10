---
title: "Next.jsのビルドにesbuild(esbuild-loader)を使う"
emoji: "🏎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "webpack", "esbuild"]
published: false
---

esbuild が爆速なのは周知の事実ですが、Next.js のビルドに利用できないかと思い立ち調べたところ、esbuild-loader を使うのが手っ取り早いようです。

https://github.com/privatenumber/esbuild-loader

esbuild-loader 自体のリポジトリにサンプルがあります。

https://github.com/privatenumber/esbuild-loader-examples/tree/master/examples/next

やっていることは至極シンプルです。

- esbuild-loader を依存に追加
- next.config.js で webpack 設定を上書き

next.config.js で上書きしている設定は、ざっくり次の３点です。

- `*.js` を対象とする Loader を esbuild-loader に差し替え
- プラグインに ESBuildPlugin を差し込み
- minimizer で利用している TerserPlugin を ESBuildMinifyPlugin に差し替え

### 試してみる

次のサンプルアプリを拝借し、esbuild-loader の設定を組み込んでみます。

https://github.com/reck1ess/next-realworld-example-app

それぞれで自分の環境で `time npm run build` を実行して計測してみました。

**通常のビルド**

- 13.06s
- 13.26s
- 13.98s
- 12.60s
- 12.84s

avg. 13.148s

**esbuild-loader を利用した場合**

- 11.41s
- 11.29s
- 11.48s
- 11.49s
- 11.78s

avg. 11.49s

おおよそ平均で 13%ほど速くなりました。リポジトリの規模によってはもっと恩恵が大きくなるかもしれません。

esbuild 自体による制約(ES5 へのビルドができないとか)はありますが、開発時のみ利用するなどで効率を上げるといった使い方はできそうです。

覚えておくと使えるシチュエーションがあるかもしれません。
