---
title: "Next.jsでServer Componentsがちょっとだけテストできるようになってた"
emoji: "🚚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

先日 Vercel の[@leeerob](https://twitter.com/leeerob)氏が次のようなツイートをしていました。

https://twitter.com/leeerob/status/1698758756924825723

> I'm working on updated testing docs for the @nextjs App Router.
> `next@canary` supports Jest for server & client components, metadata, `server-only`, and more

なんと `next@canary` で、Server Components の Jest でのテスト実行がサポートされているとのことです。

## 現時点(2023/09/08)での Server Components テストの課題

Next.js App Router で開発するときの大きな課題の一つがテストで、Server Components だと testing-library などを利用したテストにはかなり制限があります。

### `server-only` パッケージの import があるとテストできない

Server Components を用いた開発をしていると、Server 向けのコードが Client 側に紛れ込んでしまうリスクがあります。たとえば、秘匿情報である環境変数を参照する Server 向けのコードが存在していたときに、それを Client Component で import してしまうと、意図せず漏れ出てしまいます。

これを防ぐため、バックエンドでのみ利用されるコードには `server-only` パッケージを import しておくことで、仮にこれを Client Component から import していた場合は、ビルドの時点でエラーとして弾くことができる仕組みが存在します。

https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns#keeping-server-only-code-out-of-the-client-environment

一方、Next.js でのコンポーネントを Jest でテストする際は、`next/jest` を用いると、SWC を利用した Rust Compiler を利用できます。
https://nextjs.org/docs/pages/building-your-application/optimizing/testing#setting-up-jest-with-the-rust-compiler

このとき、すべてのコンポーネントが Client Component として処理されてしまうため、先述の `server-only` のチェックに該当してしまいビルドエラーが発生し、テストが実行できませんでした。

### `generateMetadata`

App Router の Page または Layout で metadata を作成する際、`generateMetadata` などの関数が利用できますが、こちらも `server-only` と同様で、Client Component で `generateMetadata` を利用している場合は Rust Compiler 側のチェックでエラーとなってしまいます。

これによって、Page コンポーネント自体で Server Components 固有の機能を利用していなくても、`generateMetadata` が定義されているだけでビルドエラーになってしまい、テストの実行ができません。

###

## 該当の PR

## 実際に試してみる

## 現状サポートされているもの

## Async Server Components は？
