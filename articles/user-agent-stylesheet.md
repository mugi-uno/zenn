---
title: "!importantで上書きできないブラウザのスタイルとは"
emoji: "🖍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["フロントエンド", "CSS"]
published: false
publication_name: "cybozu_frontend"
---

## 究極魔法 `!important`

CSS において話題に出すといろいろな意味で盛り上がるキーワードが`!important`です。

CSS でのスタイル宣言時に`!important`を付与すると、細かな詳細度の差異などを無視して強制的にスタイルを適用できます。濫用するとあっという間に無秩序になるため、一般的には慎重な利用が推奨されることが多いです。

さて、ではこの`!important`ですが、何もかもを上書きできるのでしょうか？

実はそうではありません。今回は、CSS 仕様をいろいろと調べているうちに、`!important` で上書きできないスタイルの存在を知ったため、その情報を記事にしています。

## CSS における Cascade Sorting Order

CSS は Cascading Style Sheets という名前の通り、カスケードと呼ばれる仕組みでスタイルの適用順序を決定します。

ID やクラスを使って詳細度をコントロールしたり、先に挙げた`!important` を使ってスタイルを強制したり、最近では `@layer` や `@scope` などもカスケードに関与します。

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

@[codepen](https://codepen.io/mugi-uno/pen/abxVgEP)

ホバー時のスタイルに `!important` が付与されていますが、これによってトランジションが無視されることはなく、ホバー時には 3 秒かけてカラーが変化します。ただこれは `!important` を無視するといった意味ではないので注意が必要です。仮に、通常時にのみ `!important` が付与されている場合にはトランジションは機能しません。

### Important user agent declarations

ユーザエージェントスタイルシートとも呼ばれるブラウザが持つスタイルシートの中で、`!important` が付与されているものです。

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
  animation: 3s animate infinite;
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

ID を使ったスタイル適用を含むためクラス定義のスタイルより優先されそうですが、`@keyframes` が優先されるため、文字カラーが 3 秒かけて赤 → 緑と変化するアニメーションが繰り返されます。

@[codepen](https://codepen.io/mugi-uno/pen/zYXPVWM)

ただ順序的には各種 `!important` が付与されたスタイルよりも下位になるため、仮に `!important` が付与されたスタイルが存在する場合はアニメーションは実行されません。仮に次のスタイルを追加すると、文字カラーはグレーのまま変化しません。

```css
main {
  color: gray !important;
}
```

@[codepen](https://codepen.io/mugi-uno/pen/wvZPLxJ)

### Normal author declarations

作成者スタイルシートのうち、`!important` が付与されていないものです。

通常開発時は慣習として `!important` を気軽に使うべきではないと言われることも多いかと思いますが、その形でスタイルを定義した場合にはほぼ全てがここに該当します。

### Normal user declarations

ユーザースタイルシートのうち、`!important` が付与されていないものです。

### Normal user agent declarations

ブラウザが持つスタイルシートのうち、`!important` が付与されていないものです。最も順位が低く、簡単に上書きできるようになっています。

## 最強のスタイル / Important user agent declarations

ここまでは大前提となる知識の話をしていましたが、しれっと **Important user agent declarations** とやらが登場していました。

`transition` を除くと一番上です。`!important` が付与された作成者スタイルシートより上ということは、**上書き不可能**と言っても差し支えないでしょう。あまりにも強いですね。

これはつまり "完全に上書きできないブラウザのスタイル" が存在することを意味します。

私は今のところ通常の Web 開発をしていて `!important` でも上書きができないブラウザ標準のスタイルに悩まされることはありませんでした。一体どんなものが該当するのでしょうか

## ブラウザのスタイルシートを覗いてみる

ブラウザが持つスタイルシートを覗いて、何に `!important` が付与されているか見てみましょう。

次の記事を参考にさせていただきました。

https://zenn.dev/qnighy/articles/674e1dc8c93944

### Google Chrome での例

Google Chrome のユーザエージェントスタイルシートは次のものが該当します。

https://chromium.googlesource.com/chromium/src/+/refs/heads/main/third_party/blink/renderer/core/html/resources/html.css

数だけで言えば `!important` の宣言数は結構多く、grep してみると 60 件弱ヒットします。

一部を抜粋すると、たとえば次のようにプレースホルダのスタイルなどに `!important` が付与されています。

```css
input::-webkit-input-placeholder {
  text-overflow: inherit;
  line-height: initial !important;
  white-space: pre;
  word-wrap: normal;
  overflow: hidden;
  -webkit-user-modify: read-only !important;
}
```

`line-height` を見てみると、`initial` に対して `!important` が付与されています。これを上書きしようとしてみます。

```html
<input type="text" placeholder="Place Holder" />
```

```css
input::-webkit-input-placeholder {
  color: red;
  line-height: 1000px !important;
}
```

すると次のような結果となります。

![](/images/ua-style/ua1.png)

実際にプレースホルダの見た目を確認してみると、上書きした `color` が有効であり赤色で表示されていますが、一方で `line-height` は変化していません。`!important` を付与しているにも関わらず上書きできていないことがわかります。

ただ少しややこしいのが、DevTools の Elements > Style で確認してみると、`line-height` は問題なく上書きできるように見えています。ただ、Elements > Computed で確認してみると、値は `normal` となっており、ユーザエージェントスタイルシート上の `initial !important` が適用されていることがわかります。

![](/images/ua-style/ua2.png)

特殊なケースではあるので、これ自体は DevTools 上の表示上の問題かもしれません。

### Safari で確認してみる

Safari におけるユーザエージェントスタイルシートは次のものが該当します。

https://github.com/WebKit/WebKit/blob/main/Source/WebCore/css/html.css

Chrome と似た形で、プレースホルダに対しての `line-height` への `!important` が付与されています。

```css
input::placeholder {
  white-space: pre;
  word-wrap: normal;
  overflow: hidden;
  line-height: initial !important;
}
```

同様に上書きを試みます。

```css
input::placeholder {
  color: red;
  line-height: 1000px;
}
```

結果としては次の通りで、同じ用に `line-height` は上書きできていません。

![](/images/ua-style/ua3.png)

### 付与されている基準は？

他の `!important` が付与されているものを見てみると、`-webkit-user-modify` や、`input[type="file" i]` などブラウザが追加で提供するフォームコントロールの見た目といったものが該当するようです。ブラウザとして見た目や挙動を確実に保証したいものに関して付与されているものと思われます。（なにか明確な基準がわかる方がいたら教えてもらえたら嬉しいです）

## まとめ

というわけで、Origin and Importance と `!important` で上書きできないブラウザのスタイルについての紹介でした。

DevTools などではパッと見で有効に見えるのは罠かもしれませんね。今後開発していて踏む可能性もゼロではないので、心の片隅にでも覚えておきたいところですね。
