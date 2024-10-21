---
title: 'Next.jsでブログを作ってGitHub Pagesにデプロイした話'
coverImage: null
dates:
  postDate: '2023-10-03T20:17:02.878364'
  updateDate: '2024-03-08T17:33:28.518380'
tags: ['技術系技防録', 'Next.js', 'TypeScript']
---

# (どうでもいい) 背景

JavaScriptのフロントエンドフレームワークをこれまでまったく触ってこなかったので，触ってみたくなった．
食ったラーメンの記録とか技術系の備忘録をランダムに記録していく用にMarkdownで記事を書くブログを作ることにした．
GitHub Pagesでホームページとかも作ってみたかったのでそれのついでにブログもデプロイすることにした．

# 使った技術

Next.jsのテンプレートの[blog-starter](https://github.com/vercel/next.js/tree/canary/examples/blog-starter)をベースに使って開発を始めた．
そのため使った技術はこのあたり．

- TypeScript

  TSはじめて触ったけどやっぱ型付けはあったほうがいいって再確認．
- Next.js

  TSX使うとTSの中でHTMLが書けるのが新鮮でめっちゃ面白かった．
- Tailwind CSS

  クラス名でスタイルを色々指定できるのはめっちゃ便利．
  だけどデフォルトで箇条書きの点とか消されてるのはきもいし，通常のCSSの書き方をしたファイルを使うところで躓いた

Markdownのパース & 変換には[remark](https://github.com/remarkjs/awesome-remark)と[rehype](https://github.com/rehypejs/awesome-rehype)を使っている．
プラグインでGFMに対応とかコード部分のシンタックスハイライトとか色々できるので便利．

# ブログの設計・デザイン方針

備忘録とかを適当にあげるだけのブログなので特に特別な機能はつけずに，デザインもなるべくシンプルを目指した．
記事部分のスタイルはGitHubのMarkdownの表示をベースにしている．
というか[GitHubで使われてるCSS](https://github.com/sindresorhus/github-markdown-css)をほぼまんま使ってる．
あと，GitHub Pagesは静的なサイトしかデプロイできないので，難しいことはあまりできない．

# Markdownパーサ

先述の通りMarkdownパーサには[remark](https://github.com/remarkjs/awesome-remark)を使って[rehype](https://github.com/rehypejs/awesome-rehype)でHTMLに変換している．
GitHub Flavored Markdown (GFM) に対応するためにremarkのプラグインである[remark-gfm](https://github.com/remarkjs/remark-gfm)を，
シンタックスハイライトを使うために[rehype-highlight](https://github.com/rehypejs/rehype-highlight)を，
目次のために[rehype-slug](https://github.com/rehypejs/rehype-slug)と[rehype-toc](https://github.com/JS-DevTools/rehype-toc)を噛ませている．

rehype-highlightは内部的に[highlight.js](https://highlightjs.org/)を使っているのでhighlight.jsのGitHubテーマを使っている．

## 目次の生成

目次の生成にはrehype-tocを使っているが，目次が生成される位置が記事と同じ`div`タグ内はデザイン上嫌だったので次のようにしてあげることで，目次のASTを`tocNode`でゲットできる．

```js
let tocNode = null
const result = await unified()
  :
  :
  .use(rehypeToc, {
    nav: false, // no wrapping with nav
    customizeTOC: (toc) => {
      tocNode = toc
      return false
    },
  })
  :
  :
  .process(markdown)
```

ASTをHTMLに変換するには[hast-util-to-html](https://github.com/syntax-tree/hast-util-to-html)を使えばHTMLの文字列で得られる．

# GitHub Pagesへのデプロイ

GitHub Pagesへのデプロイの準備の手順はこんな感じ

1. GitHubのrepoのSettings -> PagesでSourceをGitHub Actionsにする．
2. たぶんサジェストでNext.js用のGitHub Actionsのテンプレートが表示される (されなかったらNext.jsで検索する) のでそれを選ぶとyamlを編集できる．
3. 特にこのyamlは編集する必要はないので右上のCommit changes...を押す．

これでこのrepoのmainブランチにプッシュされるたびに自動でActionsが走りサイトが静的ビルドされGitHub Pagesにデプロイされる．
(GitHub Actionsのだんだん実行されていく感じ見てて楽しい)
ビルドされたサイトはブランチとは別の場所に保存される．

# 感想

TypeScriptのプログラムの中に直接HTML書けるのが画期的でめっちゃ面白かった．
次はサーバとのやりとりとかするようなサイトも作りたい．

# 今後追加したい機能とか

- ~~前後の記事へのリンク~~ 2023-10-12に追加！
- ~~記事一覧ページのページネーション機能~~ 2023-10-05に追加！
- 記事のタグ機能
  - タグをクリックするとそのタグを含んだ投稿の一覧が見れる & ページネーションもされる
- Next.js 13への対応

  blog-starter自体がNext.js 13より前の書き方で書かれているので，13の書き方に変更したい．