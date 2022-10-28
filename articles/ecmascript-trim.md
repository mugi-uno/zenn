---
title: "String.prototype.trim() は何をトリムするのか？"
emoji: "✂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript"]
published: false
publication_name: "cybozu_frontend"
---

> [@okunokentaro](https://zenn.dev/okunokentaro) さんが似た内容で先にスクラップを投稿されており、本記事の執筆時期と内容が重なってしまいました。こちらでは ECMAScript に加え、Java での調査結果なども含まれています。併せてご参考ください！
> https://zenn.dev/okunokentaro/scraps/256c7d9a56ac69
>
> （本記事の公開はご本人にも確認を取っております）

# `String.prototype.trim()`

JavaScript でコードを書いていて、とある文字列の端から空白を削除したくなったらどうしますか？

多くの人は `String.prototype.trim()` を使うかと思います。

では、ここで削除される "空白" は何を指すか知っているでしょうか？

恥ずかしながら、私は正確には把握しておらず、「半角・全角スペースとか改行、タブあたりをいい感じに消してくれる良いヤツ」ぐらいの認識でした。

そんななか、業務においてこの "空白" の仕様を正確に把握しなければいけないシチュエーションが発生し調べたので、記事として残しておきます。

# MDN 上の記述

まずは、MDN で `String.prototype.trim()` のページを確認してみます。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String/trim

> `trim()` メソッドは、文字列の両端の空白を削除します。このコンテクストでの空白には、空白文字（スペースやタブ、ノーブレークスペースなど）とすべての改行文字（LF や CR など）を含みます。

- 空白文字（スペースやタブ、ノーブレークスペースなど）
- すべての改行文字（LF や CR など）

が削除対象とのことですが、ノーブレークスペース "など" と記述があるように、一部の仕様については省略されているようです。

もう少し正確な仕様を知りたいので、ECMAScript の spec を追ってみましょう。

# ECMA-262 上の記述

`String.prototype.trim()` の仕様を確認します。

https://tc39.es/ecma262/#sec-string.prototype.trim

文字列のトリム部分については `TrimString` を見ると、次のような説明があります。
https://tc39.es/ecma262/#sec-trimstring

> The definition of white space is the union of WhiteSpace and LineTerminator. When determining whether a Unicode code point is in Unicode general category “Space_Separator” (“Zs”), code unit sequences are interpreted as UTF-16 encoded code point sequences as specified in 6.1.4.

空白 (white space) の定義は `WhiteSpace` と `LineTerminator` を見ると確認できそうです。また、Unicode 内の `Space_Separator` に含まれるかもチェックされるようです。（`Space_Separator` については後述）

## `WhiteSpace`

`WhiteSpace` の定義を確認すると、該当する文字コードが記載されています。
https://tc39.es/ecma262/#prod-WhiteSpace

| Code Point | Name                                                |
| :--------- | :-------------------------------------------------- |
| `U+0009`   | CHARACTER TABULATION <TAB> : 水平タブ               |
| `U+000B`   | LINE TABULATION <VT> : 垂直タブ                     |
| `U+000C`   | FORM FEED (FF) <FF> : 改ページ                      |
| `U+FEFF`   | ZERO WIDTH NO-BREAK SPACE <ZWNBSP> : ゼロ幅スペース |

具体的が文字が出てきましたね。少なくともこの４つの文字は削除対象であることがわかりました。

しかし、これらに加えて次のような記述も存在します。

- `any code point in general category “Space_Separator”`

`Space_Separator` とは、一体なんでしょうか？

### Unicode / General Category

Unicode では、文字・数字・句読点・記号などの文字の特性に応じて General Category (カテゴリ) を割り当てて分類しています。`Space_Separator` もこの General Category のひとつで、空白に関する文字群が割り当てられています。
なお、General Category の分類には 2 文字の別名も存在し、`Space_Separator` の場合は `"Zs"` が該当します。

https://unicode.org/reports/tr44/#General_Category_Values

General Category についての詳細は、Unicode 仕様の "4.5 General Category" に記載があり、
その中でも `Zs = Separator, space` の記述を確認できます。
https://www.unicode.org/versions/Unicode15.0.0/ch04.pdf

### Zs = Space_Separator に含まれる文字

では、`Space_Separator` に含まれる文字は何でしょうか。

unicode.org では、Unicode の文字情報の検索用に Unicode Utilities というページが公開されており、そこから `Character Property Index` を辿ることで General Category ごとの文字を確認することができます。

https://util.unicode.org/UnicodeJsps/properties.jsp?a=General_Category#General_Category

最終的には次のページに到達し、`Space_Separator` に含まれる文字を確認できます。
https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=[:General_Category=Space_Separator:]

というわけで、得られた空白文字の一覧がこちらです。

| Code Point | Name                                                                                                                   |
| :--------- | :--------------------------------------------------------------------------------------------------------------------- |
| `U+0020`   | SPACE : 半角スペース                                                                                                   |
| `U+00A0`   | NO-BREAK SPACE : `&nbsp`                                                                                               |
| `U+1680`   | OGHAM SPACE MARK : [オガム文字](https://ja.wikipedia.org/wiki/%E3%82%AA%E3%82%AC%E3%83%A0%E6%96%87%E5%AD%97)のスペース |
| `U+2000`   | EN QUAD : `1en` の幅のスペース                                                                                         |
| `U+2001`   | EM QUAD : `1em` の幅のスペース                                                                                         |
| `U+2002`   | EN SPACE : `1en` の幅のスペース                                                                                        |
| `U+2003`   | EM SPACE : `1em` の幅のスペース                                                                                        |
| `U+2004`   | THREE-PER-EM SPACE : `1em` の 1/3 のスペース                                                                           |
| `U+2005`   | FOUR-PER-EM SPACE : `1em` の 1/4 のスペース                                                                            |
| `U+2006`   | SIX-PER-EM SPACE : `1em` の 1/6 のスペース                                                                             |
| `U+2007`   | FIGURE SPACE : 数字と同じ幅のスペース                                                                                  |
| `U+2008`   | PUNCTUATION SPACE : ピリオドと同じ幅のスペース                                                                         |
| `U+2009`   | THIN SPACE : `1em` の 1/6 のスペース                                                                                   |
| `U+200A`   | HAIR SPACE : `THIN SPACE` よりさらに狭いスペース                                                                       |
| `U+202F`   | NARROW NO-BREAK SPACE : `THIN SPACE` と同幅のノーブレークスペース                                                      |
| `U+205F`   | MEDIUM MATHEMATICAL SPACE : 数学用スペース                                                                             |
| `U+3000`   | IDEOGRAPHIC SPACE : 全角スペース                                                                                       |

とても多いですね。

これらすべてが対象となりトリムされるようです。

## `LineTerminator`

`WhiteSpace` のほうでだいぶ寄り道をしてしまいましたが、ECMA-262 での空白の定義には `LineTerminator` も含まれます。
https://tc39.es/ecma262/#sec-line-terminators

こちらは該当する文字が列挙されています。

| Code Point | Name                      |
| :--------- | :------------------------ |
| `U+000A`   | LINE FEED (LF) <LF>       |
| `U+000D`   | CARRIAGE RETURN (CR) <CR> |
| `U+2028`   | LINE SEPARATOR <LS>       |
| `U+2029`   | PARAGRAPH SEPARATOR <PS>  |

いわゆる改行系の皆さんですね。

# まとめ

というわけで、`String.prototype.trim()` で消されるのは次の文字であることがわかりました。

| Code Point | Name                      |
| :--------- | :------------------------ |
| `U+0009`   | CHARACTER TABULATION      |
| `U+000B`   | LINE TABULATION           |
| `U+000C`   | FORM FEED (FF)            |
| `U+FEFF`   | ZERO WIDTH NO-BREAK SPACE |
| `U+0020`   | SPACE                     |
| `U+00A0`   | NO-BREAK SPACE            |
| `U+1680`   | OGHAM SPACE MARK          |
| `U+2000`   | EN QUAD                   |
| `U+2001`   | EM QUAD                   |
| `U+2002`   | EN SPACE                  |
| `U+2003`   | EM SPACE                  |
| `U+2004`   | THREE-PER-EM SPACE        |
| `U+2005`   | FOUR-PER-EM SPACE         |
| `U+2006`   | SIX-PER-EM SPACE          |
| `U+2007`   | FIGURE SPACE              |
| `U+2008`   | PUNCTUATION SPACE         |
| `U+2009`   | THIN SPACE                |
| `U+200A`   | HAIR SPACE                |
| `U+202F`   | NARROW NO-BREAK SPACE     |
| `U+205F`   | MEDIUM MATHEMATICAL SPACE |
| `U+3000`   | IDEOGRAPHIC SPACE         |
| `U+000A`   | LINE FEED (LF)            |
| `U+000D`   | CARRIAGE RETURN (CR)      |
| `U+2028`   | LINE SEPARATOR            |
| `U+2029`   | PARAGRAPH SEPARATOR       |

ここで出会って一生使うことのないかもしれない文字コードもあるかもしれませんが、勉強になりました。

# 余談 / なぜ調べる必要があったか？

通常では `String.prototype.trim()` を使っていても、それが原因で特別問題になることは少ないかもしれません。

現在サイボウズ社内では kintone というサービスの[フロントエンド刷新が進行中](https://blog.cybozu.io/entry/2021/07/20/170000)です。
その中で、10 年前に独自実装されたオリジナルの文字列トリム処理が存在しており、
「これってホントに `String.prototype.trim()` に置き換えてもいいのか……？？」と迷いが生まれたため、詳細に調べることになりました。

結果的には、既存のオリジナルの文字列トリム処理は、ほぼ `String.prototype.trim()` と同等の処理であることがわかり、安心して置き換えることができました。
（10 年前の段階で、ちゃんと作ってあってすごいな〜という感想でした）
