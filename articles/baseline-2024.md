---
title: "Baseline 2024 Newly Available な機能をみてみよう"
emoji: "🎂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frontend"]
published: true
publication_name: "cybozu_frontend"
---

:::message
この記事は、[CYBOZU SUMMER BLOG FES '24](https://cybozu.github.io/summer-blog-fes-2024/) (Frontend Stage) DAY 20 の記事です。
:::

# Baseline とは

Baseline というものをご存知でしょうか。

Google I/O 2023 にで発表された、API のブラウザ間での互換性を明示するための取り組みです。

https://web.dev/baseline/overview

日々ブラウザで利用可能な API は増えていますが、ブラウザ間によっても実装状況に差異があります。新しい機能の存在を知ったとしても「で、これは結局もう使っていいの？」という判断は別途必要となります。

Baseline では、各機能について Newly Available と Widely available という基準を設けています。

- Newly Available: コアブラウザ（Chrome, Edge, Firefox, Safari）の最新安定バージョンで利用可能になったもの
- Widely Available: Newly Available になってから 30 ヶ月が経過したもの

たとえば、提供するサービスの動作仕様として "コアブラウザの安定最新版" として提示していた場合であれば、Baseline を参考にすることで "Newly Available の機能であれば利用して問題ない" といった判断ができます。

## 最近の Newly Available をみてみよう

Newly Available は前述の通りコアブラウザの最新安定バージョンで利用可能になったものです。つまり、最近 Newly Available になった機能とは、近い将来に当たり前に利用されるようになる可能性のある機能とも言えます。

ということで、この記事では、2024 年に入ってから新たに Newly Available となった機能をいくつかピックアップしておさらいしてみたいと思います。

# Pickup Newly Available 2024/1〜2024/8

**前提**

- 2024 年 1 月から 2024 年 8 月 の間に Newly Available となった機能です
- すべての機能ではありません。筆者の独断と偏見でピックアップしています

## `Object.groupBy()` / `Map.groupBy()`

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/groupBy

JavaScript で、反復可能な要素に対して `groupBy` メソッドが利用可能になりました。

```ts
Object.groupBy(
  [
    { type: "bird", name: "Peacock" },
    { type: "fish", name: "Tuna" },
    { type: "animal", name: "Dog" },
    { type: "animal", name: "Horse" },
    { type: "fish", name: "Sardine" },
    { type: "animal", name: "Lion" },
  ],
  ({ type }) => type
);

/*
Result:
{
    "bird": [
        { "type": "bird", "name": "Peacock" }
    ],
    "fish": [
        { "type": "fish", "name": "Tuna" },
        { "type": "fish", "name": "Sardine" }
    ],
    "animal": [
        { "type": "animal", "name": "Dog" },
        { "type": "animal", "name": "Horse" },
        { "type": "animal", "name": "Lion" }
    ]
}
*/
```

同様の処理をしたい場合、従来では `Array.prototype.reduce()` で対応したり、Utility 系のライブラリに含まれる同様のメソッドに頼るケースもありましたが、現在ではネイティブで簡単にグルーピング可能となりました。

## `CustomStateSet` および `:state()` 擬似クラス

https://developer.mozilla.org/en-US/docs/Web/API/CustomStateSet
https://developer.mozilla.org/en-US/docs/Web/CSS/:state

カスタム要素において任意の状態を扱うための API である `CustomStateSet` と、該当の状態に応じてマッチさせるための CSS 擬似クラスである `:state()` が利用可能になりました。

`CustomStateSet` は、[`ElementInternals`](https://developer.mozilla.org/ja/docs/Web/API/ElementInternals)オブジェクトの `states` プロパティ経由で操作できます。実際に利用する際には、カスタム要素の中で `attachInternals()` メソッドを呼び出すことで、カスタム要素に紐付いた `ElementInternals` オブジェクトを得られます。

`CustomStateSet` に対して任意の状態を定義した場合には、同じ名前で `:state()` 擬似クラスを使うことで CSS 上でもマッチします。

たとえば次のコード例の場合、カスタム要素である `MyElement` をクリックすると、カスタム状態として `"clicked"` を内部に保持します。CSS からは `:state(clicked)` でマッチさせることができます。

```ts:JavaScript
class MyElement extends HTMLElement {
  #internals;

  constructor() {
    super();
    this.#internals = this.attachInternals();
    this.addEventListener("click", this.#onClick.bind(this));
  }

  #onClick() {
    this.#internals.states.add("clicked");
  }
}

customElements.define("my-element", MyElement);
```

```css:CSS
my-element:state(clicked) {
  color: red;
}
```

```html:HTML
<my-element>MyElement</my-element>
```

以下の実行例にて、クリックすると文字の色が変わることを確認できます。

@[codepen](https://codepen.io/mugi-uno/pen/MWMQvov?default-tab=js,result)

## `CSS.registerProperty()` および CSS `@property`

https://developer.mozilla.org/ja/docs/Web/API/CSS/registerProperty_static
https://developer.mozilla.org/ja/docs/Web/CSS/@property

CSS において、型やデフォルト値などを保持したカスタムプロパティを定義するための JavaScript API である `CSS.registerProperty()` と、同等の仕組みを CSS から直接利用するための `@property` が利用可能になりました。

CSS カスタムプロパティ自体は [Widely available](https://developer.mozilla.org/ja/docs/Web/CSS/--*) であり、すでに幅広いブラウザで利用可能ですが、加えて `CSS.registerProperty()` や `@property` を用いることで、より強力に扱うことができます。

以下は `@property` の例です。`--my-color` という名前でカラーのみを受け付けるカスタムプロパティを定義しています。デフォルトカラーが宣言されており、`inherits: false` により、親要素で定義した値は子孫に継承されていないことがわかります。

@[codepen](https://codepen.io/mugi-uno/pen/jOjZmjJ?default-tab=css,result)

## Popover API

https://developer.mozilla.org/ja/docs/Web/API/Popover_API

要素をポップオーバーとして扱うための `popover` および `popovertarget` 属性が利用可能となりました。

ポップオーバー表示したい要素には `popover` 属性および ID を付与し、トリガーする要素には `popovertarget` 属性および対象の ID を指定します。
`popovertarget` を指定するトリガー要素は `<input type="button">` や `<button>` が利用できます。（なお、`HTMLInputElement#popoverTargetElement` を用いることで、ID や`popovertarget`を介さない紐づけも可能です。）

従来では ポップオーバーの実現には JavaScript が必須でしたが、Popover API の登場により、HTML のみで基本的な動作が実現可能となり、Progressive Enhancement にも寄与します。また、ポップオーバーを雑に実装してしまうとアクセシビリティ面でも問題が生じることがありますが、フォーカス制御など、デフォルトでアクセシビリティ面での配慮が含まれるのも特徴となります。

以下は簡単な Popover API の例です。HTML のみですが、ボタンをクリックするだけでポップオーバーがトグルすることがわかります。

@[codepen](https://codepen.io/mugi-uno/pen/YzoeQXr?default-tab=html,result)

## Declarative Shadow DOM および `Document.parseHTMLUnsafe()` / `setHTMLUnsafe()`

https://web.dev/articles/declarative-shadow-dom?hl=en
https://developer.mozilla.org/ja/docs/Web/API/Document/parseHTMLUnsafe_static
https://developer.mozilla.org/ja/docs/Web/API/Element/setHTMLUnsafe

Web Components を構成する要素の一つである Shadow DOM ですが、従来では JavaScript を利用した命令的な Shadow Root の構築が必要であり、SSR が必要なケースでは Shadow DOM と組み合わせた利用が難しい状態でした。

新たに Newly Available となった Declarative Shadow DOM によって、その名の通り Shadow DOM を宣言的に構築することが可能となり、HTML Parser によって解析され、HTML のみで Shadow Root を構築可能となっています。また、JavaScript からも `Document.parseHTMLUnsafe()` / `setHTMLUnsafe()` を用いることで、動的に Declarative Shadow DOM を扱うことも可能です。

詳細については、サイボウズの [saku](https://zenn.dev/s_a_k_u) が記事を書いてくれているので、そちらが大変参考になります。
https://zenn.dev/cybozu_frontend/articles/web-standardized-component-in-server-and-client

## Set の集合演算

https://tc39.es/proposal-set-methods/

JavaScript での Set オブジェクトにおいて、集合演算を行うための次のメソッドが利用可能になりました。

- `Set.prototype.intersection()`
- `Set.prototype.union()`
- `Set.prototype.difference()`
- `Set.prototype.symmetricDifference()`
- `Set.prototype.isSubsetOf()`
- `Set.prototype.isSupersetOf()`
- `Set.prototype.isDisjointFrom()`

```ts
new Set([1, 2, 3, 4, 5]).intersection(new Set([3, 0, 1, 6]));
// → Result: Set(2) { 1, 3 }

new Set([1, 2, 3, 4, 5]).union(new Set([3, 0, 1, 6]));
// → Result: Set(7) { 0, 1, 2, 3, 4, 5, 6 }

new Set([1, 2, 3, 4, 5]).difference(new Set([3, 0, 1, 6]));
// → Result: Set(3) { 2, 4, 5 }

new Set([1, 2, 3, 4, 5]).symmetricDifference(new Set([3, 0, 1, 6]));
// → Result: Set(5) { 0, 2, 4, 5, 6 }

new Set([1, 3, 5]).isSubsetOf(new Set([1, 2, 3, 4, 5]));
// → Result: true
new Set([1, 3, 5, 6]).isSubsetOf(new Set([1, 2, 3, 4, 5]));
// → Result: false

new Set([1, 2, 3, 4, 5]).isSupersetOf(new Set([1, 3, 5]));
// → Result: true
new Set([1, 2, 3, 4, 5]).isSupersetOf(new Set([1, 3, 5, 6]));
// → Result: false

new Set([1, 2, 3, 4, 5]).isDisjointFrom(new Set([1, 2, 3, 4, 5]));
// → Result: false
new Set([1, 2, 3, 4, 5]).isDisjointFrom(new Set([1, 3, 5, 6]));
// → Result: false
new Set([1, 2, 3, 4, 5]).isDisjointFrom(new Set([0, 6]));
// → Result: true
```

愚直に手動での実装を考えてみると、意外と手間になる類のものです。
必要になるシチュエーションが来たときに、引き出しとして持っておくとかなりコードが簡潔に済みそうですね。

## `Element # checkVisibility()`

https://developer.mozilla.org/ja/docs/Web/API/Element/checkVisibility

要素が可視状態かを検査するための `checkVisibility()` メソッドが利用可能になりました。

要素がユーザー視点で「見えている」かは、さまざまな要因によって決定されるため判断が難しいものです。`checkVisibility()` メソッドを用いることで、それらの要因を一通り考慮したうえで可視状態の判別ができます。

デフォルトでは CSS の `display` プロパティや `content-visibility` に応じたチェックが行われますが、オプションによって `visibility` プロパティによる不可視状態や `opacity` プロパティによる不透明度によるチェックも行えます。

```html
<div class="content1">content1</div>
<div class="content2">content2</div>
<div class="content3">content3</div>
```

```css
.content2 {
  display: none;
}
.content3 {
  opacity: 0;
}
```

```ts
document.querySelector(".content1").checkVisibility();
// → true
document.querySelector(".content2").checkVisibility();
// → false (display: none のため)
document.querySelector(".content3").checkVisibility();
// → true (opacity: 0 だがオプション指定が無いため)
document.querySelector(".content3").checkVisibility({ opacityProperty: true });
// → true (opacity: 0 でオプション指定があるため)
```

# まとめ

2024 年 1 月から 8 月で新たに Newly Available になった機能をいくつかピックアップして見ていきました。知っていた機能はあったでしょうか？

現状では「普段からめちゃ使っているぜ！」というものは少ないかもしれませんが、すべてのコアブラウザの最新安定版ではすでに利用可能であり、時間が経てば使っていることが当たり前になっている可能性の高い機能たちなので、最低限概要だけでも抑えておくといつか役に立つかも知れませんね。

なお、Baseline の状況については、一部は Web Platform Dashboard で確認することができます。

https://web.dev/blog/web-platform-dashboard?hl=en
https://webstatus.dev/

たまに眺めてみると「あ、これもう使っていいのか」という発見があって楽しいので、ぜひチェックしてみてください。
