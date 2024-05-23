---
title: "Next.js で React Compiler を試しつつ出力コードを見てみる"
emoji: "🛵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

## React Compiler

React 19 Beta から React Compiler が導入され利用可能となりました。

https://react.dev/learn/react-compiler

※単体での検証としては次の記事が参考になります。
https://zenn.dev/kazukix/articles/react-compiler

## Next.js での利用

React Compiler のドキュメント内には、各種バンドラやフレームワークで利用する方法も[記載](https://react.dev/learn/react-compiler#usage-with-nextjs)されています。

というわけで、Next.js で実際に試してみよう、というのがこの記事の主旨です。

## 事前準備 / セットアップ

基本的にドキュメントに従って進めます。注意点としては、執筆時点での Next.js の Stable バージョン 14.2 ではまだ React 19 が利用できないため、canary バージョンの利用が必要です。

適当なディレクトリを作成し、その中で `create-next-app` を実行します。

実験用のためオプションは適当に選択しますが、せっかくなので Turbopack だけは有効化してみましょう。

```sh
$ npx create-next-app@canary .
```

React Compiler を実行するための Babel プラグインも追加します。

```sh
npm install babel-plugin-react-compiler
```

簡単な入力・表示を行う Client Component を作成します。

```tsx:page.tsx
"use client";

import { ChangeEventHandler, useState } from "react";
import { NameView } from "./NameView";
import { AgeView } from "./AgeView";
import style from "./page.module.css";

export default function App() {
  const [name, setName] = useState("John Doe");
  const [age, setAge] = useState(20);

  const handleChangeName: ChangeEventHandler<HTMLInputElement> = (e) => {
    setName(e.target.value);
  };
  const handleChangeAge: ChangeEventHandler<HTMLInputElement> = (e) => {
    setAge(Number(e.target.value));
  };

  return (
    <main className={style.main}>
      <label>
        name <input type="text" value={name} onChange={handleChangeName} />
      </label>
      <label>
        age <input type="number" value={age} onChange={handleChangeAge} />
      </label>

      <hr />

      <NameView name={name} />
      <AgeView age={age} />
    </main>
  );
}
```

```tsx:NameView.tsx
type Props = {
  name: string;
};

export const NameView = ({ name }: Props) => {
  return <section>name: {name}</section>;
};
```

```tsx:AgeView.tsx
type Props = {
  age: number;
};

export const AgeView = ({ age }: Props) => {
  return <section>age: {age}</section>;
};
```

name と age の入力フォームが存在し、それらを Props として受け取るコンポーネントが存在しています。page.tsx でのイベントハンドラや、表示用の `<NameView>`・`<AgeView>` のメモ化は一切行っていない状態です。

## React Compiler 無しでの実行結果（従来の挙動）

まず従来の挙動を見てみましょう。

`npm run dev` で起動し、name や age を入力してみます。
その際、React DevTools を利用し、再レンダリング時にハイライトをしてみましょう。

結果は次の通りです。（見やすさのため、少々 CSS を調整しています）

![](/images/next-r-compiler/img1.gif)

入力のたびに、 `<NameView>`・`<AgeView>` の両方が再レンダリングされていることがわかります。

## React Compiler を利用した場合

では、React Compiler を有効化してみましょう。

`next.config.js` に設定を追加します。

```js:next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    reactCompiler: true,
  },
};

export default nextConfig;
```

この状態で `next run dev` をしてみると、起動時に reactCompiler が有効な旨のログが表示されます。

![](/images/next-r-compiler/img2.png)

再度入力を試してみると、結果は次のとおりです。

![](/images/next-r-compiler/img3.gif)

name を入力した場合には `<NameView>` のみが再レンダリングされ、age を入力した場合には `<AgeView>` のみが再レンダリングされることがわかります。

また、React DevTools でコンポーネントツリーを確認してみると、React Compiler によるメモ化が機能していることも表示されています。

![](/images/next-r-compiler/img4.png)

正常に動いていますね！

## ビルドされた生成物を比較してみる

せっかくなので、Client で利用されるビルド生成物の内容も比較してみます。

以下は、React Compiler の無効・有効状態で生成される JS の内容で Diff を取り、`handleChangeName` や `handleChangeAge` といったイベントハンドラの付近のコードを抜粋したものです。

![](/images/next-r-compiler/img5.png)

React Compiler が有効化された後は、 `Symbol.for("react.memo_cache_sentinel")` を用いたロジックが組み込まれていることがわかります。

どうやら、メモ化に関係する値は、キャッシュの初期値としては一通り `Symbol.for("react.memo_cache_sentinel")` が設定されているようです。

他に依存がないものについては、値が `Symbol.for("react.memo_cache_sentinel")` のときのみキャッシュの値がリフレッシュされることで、常にキャッシュの値が利用されるような仕組みになっているようです。

依存がある値については、依存値と直接比較を行うようです。初期値は `Symbol.for("react.memo_cache_sentinel")` なので、初回は必ずキャッシュがリフレッシュされることになります。

試しに、次のような関数を追加してみます。

```tsx
const handleFireLog = () => {
  console.log(name);
  console.log(age);
};
```

この状態での生成物を見てみると、次のようなコードが追加されています。

![](/images/next-r-compiler/img6.png)

`$[3]` や `$[4]` と name や age を比較していますね。これらは初期値は `Symbol.for("react.memo_cache_sentinel")` なので、初回あるいは name や age が変更時にのみ関数が再生成されることになります。

## 所感

実験したのは生成したばかりのプロジェクトということもありますが、Babel プラグインの追加と next.config.js の設定変更のみで機能するため、非常に導入は簡単でした。現段階では Production への導入は推奨されていないようですが、安定してきたら積極的に検証していきたいところです。

一方で、どういったコードが出力されるのかが魔法のように感じる部分もあるので、もう少し Compiler 自体の中身を追いかけてみたいところです。
