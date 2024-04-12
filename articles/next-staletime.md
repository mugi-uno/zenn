---
title: "Next.js 14.2 で追加された StaleTimes を素振り"
emoji: "🚚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

## Next.js v14.2.0

2024/04/12 に Next.js v14.2.0 がリリースされました。

https://nextjs.org/blog/next-14-2

Turbopack のパフォーマンス・サポート状況の強化などいくつか機能追加や改善が含まれますが、その中のひとつに experimental なオプションとしての`staleTimes` の追加があります。

これを触って理解しておこう、というのがこの記事の目的です。

## Router Cache と StaleTimes

`staleTimes` オプションは一言で説明すると「従来カスタマイズできなかった Router Cache の生存期間の設定」を実現します。

Router Cache は、Next.js のキャッシュ機構のひとつであり、prefetch した内容や既に訪れた内容を Client-side でインメモリにキャッシュし、再訪問時の高速な表示を実現する機能です。

Router Cache そのものについて深く知りたい場合は、[@akfm_sato](https://twitter.com/akfm_sato) さんのアウトプットを参考にすることを推奨します。

https://zenn.dev/akfm/articles/next-app-router-client-cache

Next.js が提供する他のキャッシュ機構、たとえば Data Cache や Full Route Cache に関しては、`fetch()` のオプションや Route Segment Config を用いて Opt-out することができます。しかし、Router Cache については Opt-out する方法が存在せず、適宜 `router.refresh()` を実行したり、`revalidatePath()` や `revalidateTag()` を呼び出す必要がありました。

今回ここに `staleTimes` オプションが追加されたことで、Router Cache についても開発者のニーズに応じた調整が可能となりました。

## 試してみる

WIP

---
