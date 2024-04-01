---
title: "CSS の Important user agent declarations ってなに？"
emoji: "💥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["フロントエンド", "CSS"]
published: false
publication_name: "cybozu_frontend"
---

# CSS における Cascade Sorting Order

CSS は Cascading Style Sheets という名前の通り、カスケードと呼ばれる仕組みでスタイルの適用順序を決定します。

ID やクラスを使って詳細度をコントロールしたり、`!important` を使ってスタイルを強制したり、最近では `@layer` や `@scope` などもカスケードに関与します。

そしてこれらには優先順位が定められています。CSS Cascading and Inheritance Level 4 の中であれば Cascade Sorting Order のセクションで仕様を確認できます。

https://drafts.csswg.org/css-cascade/#cascade-sort

## Origin and Importance

このときの適用順序でまず最初に登場するのが **Origin and Importance** です。これは何よりも優先して判定されます。

ここでの Origin = オリジンは、その CSS 定義がどこから来たものかを指しています。次の順序で判定され、上にあるほうが優先されます。

1. Transition declarations
1. Important user agent declarations
1. Important user declarations
1. Important author declarations
1. Animation declarations
1. Normal author declarations
1. Normal user declarations
1. Normal user agent declarations

個々にどういったものか見てみましょう

### Transition declarations

何よりも `transition` 定義によるトランジションは優先されます。たとえ変化後のプロパティに `!important` の強い指定が入っていても、トランジション動作が保証されます。

次のような例がわかりやすいです。

```css
.content {
  color: red;
  transition: color 3s;
}

.content:hover {
  color: blue !important;
}
```

ホバー時のスタイルに `!important` が付与されていますが、これによってトランジションが無視されることはなく、ホバー時には 3 秒かけてカラーが変化します。ただこれは `!important` を無視するといった意味ではないので注意が必要です。仮に、通常時にのみ `!important` が付与されている場合にはトランジションは機能しません。

### Important user agent declarations

ブラウザが持つスタイルシートの中で、`!important` が付与されているものです。（本記事のメインテーマです。）

### Important user declarations

ユーザーが独自にブラウザに設定する "ユーザースタイルシート" と呼ばれる機能を用いているケースで、そのうち`!important` が付与されているものです。

ただ、ユーザースタイルシートは Chrome などでは遥か昔に機能としては削除されているようです。
https://src.chromium.org/viewvc/chrome?revision=234007&view=revision

### Important author declarations

開発者が用意した `*.css` ファイルや`<style>` タグなどで提供され、我々が普段意識するものは "作成者スタイルシート" と呼ばれます。そのうち`!important` が付与されているものがここに該当します。

### Animation declarations

`@keyframes` を用いたアニメーション定義で、`!important`が付与されていない全てのスタイルより優先されます。次の例を見てみます。

```html
<main class="animateClass" id="animateID">this is animate</main>
```

```css
.animateClass {
  animation: 3s animate;
}

@keyframes animate {
  from {
    color: red;
  }
  to {
    color: green;
  }
}

#animateID {
  color: blue;
}
```

ID を使ったスタイル適用を含むためクラス定義のスタイルより優先されそうですが、`@keyframes` が優先されるため、カラーが 3 秒かけて赤 → 緑と変化した後に青に変わるような動作となります。

### Normal author declarations

作成者スタイルシートのうち、`!important` が付与されていないものです。

通常開発時は慣習として `!important` を気軽に使うべきではないと言われることも多いかと思いますが、その形でスタイルを定義した場合にはほぼ全てがここに該当します。

### Normal user declarations

ユーザースタイルシートのうち、`!important` が付与されていないものです。

### Normal user agent declarations

ブラウザが持つスタイルシートのうち、`!important` が付与されていないものです。最も順位が低く、簡単に上書きできるようになっています。

## 最強のスタイル / Important user agent declarations

さて本題です。

ここまでは大前提となる知識の話をしていましたが、しれっと **Important user agent declarations** とやらが登場していました。

これは`transition` を除くと一番上です。`!important` が付与された作成者スタイルシートより上ということは、**上書き不可能**と言っても差し支えないでしょう。あまりにも強い。

これはつまり "完全に上書きできないブラウザのスタイル" が存在することを意味します。

私は今のところ通常の Web 開発をしていて `!important` でも上書きができないブラウザ標準のスタイルに悩まされたことは無く、一体どんなものか気になってしまいました。

というわけで、これを探ってみよう、というのが本記事の主題です。（ここまでが長い）

## ブラウザのスタイルシートを覗いてみる

何はともあれブラウザが持つスタイルシートを覗いて、何に `!important` が付与されているか見てみましょう。

次のページを参考にさせていただきました。
https://coliss.com/articles/build-websites/operation/css/user-agent-stylesheets.html
