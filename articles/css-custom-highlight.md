---
title: "CSS Custom Highlight API でのテキストハイライト"
emoji: "🖍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CSS", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

先日、[Safari Technology Preview 175](https://webkit.org/blog/14398/release-notes-for-safari-technology-preview-175/)の情報を眺めていたところ、Web API について次の変更の記述がありました。

> Added support for priority to CSS Highlight API

CSS Highlight API に `priority` とやらのサポートが追加されたとのことです。

自分はそもそも CSS Highlight API が何かを知らなかったため、少し調べてみました。

## CSS Custom Highlight API とは

mdn がとても参考になります。
https://developer.mozilla.org/en-US/docs/Web/API/CSS_Custom_Highlight_API

CSS Custom Highlight API とは、Web ページにおいて、テキストの範囲を装飾するための API のことです。

もともと存在していた [`Range()`](https://developer.mozilla.org/ja/docs/Web/API/Range)と組み合わせることで、JavaScript 上からテキスト範囲をコントロールしつつ、その範囲に対して DOM 構造を維持したままスタイルリングすることができます。

## 従来の DOM を利用したテキストハイライトの実装

テキストのハイライトが必要となる代表例としては、キーワード検索でヒットした部分をスタイリングするケースなどでしょう。

![](/images/css-highlight/search.png)

このような機能を実装する際、従来ではハイライトを行いたい部分を `<span>` などで囲み、その要素に対してスタイリングを行うことで実現可能でした。

```html
<span class="highlight">Lor</span>
em ipsum do
<span class="highlight">lor</span>
sit amet, consec ...
```

```css
.highlight {
  background-color: #ff0;
}
```

しかしこのようにハイライト箇所をタグで囲う方法は、色々と考慮すべき点がつきまといます。

### 考慮ポイント : HTML タグのネスト

ハイライト対象になりうるテキストが、すでに HTML 構造上で何らかのタグで囲まれている可能性があります。

たとえば、次のように先頭のワードのみスタイリングされているケースを考えてみましょう。

```html
<span class="first-word">Lorem</span> ipsum dolor sit amet, consectetur ...
```

このとき、次のイメージのように、すでにスタイリングされている部分とそうでない部分を跨ぐように "em ip" の箇所についてハイライトを行おうとしてみます。

![](/images/css-highlight/pre-1.png)

この場合、ハイライト用の `<span>` タグは、該当のテキストを分割した上で複数追加する必要があります。

```html
<span class="first-word">Lor<span class="highlight">em</span></span
><span class="highlight"> ip</span>sum dolor sit amet, consectetur ...
```

これをコード上から制御しようと思うと明らかに大変そうですね。

### 考慮ポイント : CSS 詳細度

ハイライト部分をタグで囲ってスタイリングした場合、詳細度によってはハイライトが適用されない可能性があります。

少し極端な例ではありますが、次のように `<span>` に対して何らかのスタイルが適用されており、そのスタイルの詳細度が高いと、ハイライト用のスタイルが適用されません。

```html
<div class="text">
  Lorem <span class="highlight">ipsum</span> dolor sit amet, consectetur ...
</div>
```

```css
.text > span {
  background-color: white;
}

.highlight {
  background-color: #ff0;
}
```

![](/images/css-highlight/pre-2.png)
_ハイライト用スタイルが適用されていない例_

正しく適用するためには、常にハイライト用のスタイルは詳細度で他のスタイルを上回るよう注意が必要となります。

## CSS Custom Highlight API でハイライトしてみる

では、CSS Custom Highlight API で実際にハイライトを適用してみましょう。

次の HTML で、先頭のほうの "em ip" の部分をハイライトしてみます。

```html
<div class="text">
  <span class="first-word">Lorem</span> ipsum dolor sit amet, consectetur ...
</div>
```

```css
.first-word {
  font-weight: bold;
}

.text > span {
  /* 上書きされるのを確認するため、!importantを意図的に付与 */
  background-color: white !important;
}
```

初期状態では次のような見た目です。

![](/images/css-highlight/api-1.png)

CSS Custom Highlight API でのハイライト時は、ハイライト範囲を [`Range`](https://developer.mozilla.org/en-US/docs/Web/API/Range) で定義します。

`Range` は [`Window.getSelection()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/getSelection) 経由で選択範囲から取得することもできますし、[`Document.createRange()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/createRange) や `Range()` コンストラクタ経由で生成もできます。

今回は `Range()` コンストラクタで任意の範囲を指定します。指定には DOM Node を用います。

```js
const text = document.querySelector(".text");

// spanタグ内のテキストノード
const node1 = text.childNodes[1].childNodes[0];
// spanタグ以降のテキストノード
const node2 = text.childNodes[2];

const range = new Range();

// 開始位置: spanタグ内のテキスト3文字目の後ろ
range.setStart(node1, 3);
// 終了位置: spanタグ以降のテキスト3文字目の後ろ
range.setEnd(node2, 3);
```

`Range` を生成したら、`Highlight` オブジェクトを生成し、`HighlightRegistry` 経由でハイライトを登録します。

```js
const highlight = new Highlight(range);

CSS.highlights.set("my-highlight", highlight);
```

最後に、`::highlight()` 疑似要素を用いてハイライト用のスタイルを定義します。このとき、`HighlightRegistry` 登録時に指定した名称を使って紐付けることができます。

```css
.text::highlight(my-highlight) {
  background-color: red;
}
```

この状態で表示してみると、次のようにハイライトが適用されます。

![](/images/css-highlight/api-2.png)

DevTools などで HTML を確認してみると、特に DOM 構造上の変化がないことを確認できます。また、`background-color: white !important;` のスタイルが付与されているにも関わらず、ハイライト用スタイルが優先されて表示されていることもわかります。

![](/images/css-highlight/api-3.png)

DOM 構造に手を加えることなく、かつ既存の CSS の詳細度を気にすることなく、ハイライトを適用できました。

## CSS Custom Highlight API の優先順位

当然のように `!important` のスタイルよりも優先してハイライトのスタイルが適用されていますが、一体どういうことでしょうか？

https://drafts.csswg.org/css-highlight-api-1/#c-and-h には次の記述があります。

> The cascading and inheritance of custom highlight pseudo-elements is handled identically to that of the built-in highlight pseudo-elements, as defined in CSS Pseudo-Elements 4 § 3.5 Cascading and Per-Element Highlight Styles.

> DeepL 翻訳: カスタム・ハイライト擬似要素のカスケードと継承は、CSS 擬似要素 4 セクション 3.5 カスケードと要素単位のハイライト・スタイル で定義されているように、組み込みのハイライト擬似要素と同じように扱われます。

どうやら、組み込みのハイライトスタイルと同様のルールで適用されるようです。

さらに、 https://drafts.csswg.org/css-pseudo-4/#highlight-text には次の説明があります。

> A highlight pseudo-element suppresses the normal drawing of any associated text, and the text decorations (other than shadows) that had been applied to that text. Instead the topmost active highlight overlay redraws that text (and those decorations) over all the highlight overlay backgrounds using that highlight’s own color.

> DeepL 翻訳: ハイライト擬似要素は、関連付けられたテキストと、そのテキストに適用されていたテキスト装飾（影以外）の通常の描画を抑制します。その代わりに、一番上のアクティブなハイライト・オーバーレイが、そのテキスト（とそれらの装飾）を、そのハイライト独自の色を使って、すべてのハイライト・オーバーレイの背景の上に再描画します。

つまり、`::highlight` 擬似要素上でのスタイリングについては、元々持っていたテキスト関連のスタイリングは抑制され、ハイライト独自のスタイリングが優先されて適用されるようです。

### `::highlight` 擬似要素のスタイリングの制約

優先してスタイリングされるとなると、場合によっては悪用できそうな感じもありますが、実際には次のような制約があり、指定できる CSS プロパティは限られています。

https://drafts.csswg.org/css-highlight-api-1/#applicable-properties

> Custom highlight pseudo-elements, like the built-in highlight pseudo-elements, can only be styled with a limited set of properties. See CSS Pseudo-Elements 4 § 3.2 Styling Highlights for the full list.

> DeepL 翻訳: カスタム・ハイライト擬似要素は、組み込みのハイライト擬似要素と同様に、限られたプロパティのセットでしかスタイルを設定できません。全リストは CSS 擬似要素 4 §3.2 ハイライトのスタイリング を参照してください。

利用可能なのは次のプロパティのみのようです。

- `color`
- `background-color`
- `text-decoration`
- `text-shadow`
- `stroke-color` `fill-color` `stroke-width`

試しに `font-size` `font-weight` などを指定しても無視されます。これは注意が必要そうですね。

## サポート状況

2023 年 8 月現在での、メジャーブラウザのサポート状況は次のとおりです。

- Chrome >= 105
- Edge >= 105
- Firefox … Nightly で有効
- Safari … 動作しない?(自分の環境では確認できませんでした)

ただ、仕様的に CSS Custom Highlight API は現在 Editor's draft の状態なので、将来的に全ブラウザで安定的に使えるようになったときに備えて予習しておく程度に留めておくほうが現状は無難かもしれません。

## まとめ

というわけで、CSS Custom Highlight API を試してみた、という記事でした。自分も過去にハイライト実装で苦労した覚えがあるので、安定的に利用可能になったら便利に使えそうだなと思いました。

なお、検索機能を実装したデモページを次の URL で公開しています。興味があればどうぞ。

https://mugi-uno.github.io/css-highlight-sandbox/

https://github.com/mugi-uno/css-highlight-sandbox
