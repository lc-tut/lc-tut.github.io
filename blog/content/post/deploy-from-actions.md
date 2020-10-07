---
title: "記事を GitHub Actions を使ってデプロイするようにしました"
date: "2020-10-08"
description: "今まで手動でビルドしデプロイしていたこのブログを GitHub Actions を使って自動でやってくれるようにしました."
author: "Forell"
tags:
- GitHub Actions
- 自動化
- CI/CD
categories:
- infra
---

この記事は, GitHub Actions のテストも兼ねた投稿です.

何気に去年から部長やってます Forell ([@F0rell](https://twitter.com/F0rell)) です.
このブログが長らく更新されてなかったのと, 自分が何もやってないのもあったので動かしてみました.

さて, URL からわかる通り, このブログは GitHub Pages を用いて公開されています (該当リポジトリ: [https://github.com/lc-tut/lc-tut.github.io](https://github.com/lc-tut/lc-tut.github.io)).
サイトを構成するために [Hugo](https://gohugo.io/) を用いており, リポジトリでは source ブランチを Hugo を用いてビルドして master に向けて該当ディレクトリ (`docs/`) を `git subtree` を用いて push することによってページを公開できるようにしております.

しかし, 手動でビルド, master に push するのははっきり言って面倒くさいです (初記事なのに何言ってるんだと思いますが, 大目に見てほしいです).
加えて, やり方が書いてあるといえども, `git subtree` という普段 (私が) 使わないコマンドも用いるため, 公開までのハードルが高いです.

そのため, GitHub Actions を使って自動でビルド + デプロイをしたいと考えました.

# GitHub Actions とは
かなり雑に言うと GitHub が提供している CI/CD ツールです.

詳しいことは[ここ](https://github.co.jp/features/actions)とか[ここ](https://knowledge.sakura.ad.jp/23478/)見たらいいかと思います.
語れるほどやっているわけではないので URL 貼っつけになって申し訳ないです.

# 使った Action など
今回使った Action は以下の 3 つです:

- [actions/checkout@v2](https://github.com/actions/checkout)
- [peaceiris/actions-hugo@v2](https://github.com/peaceiris/actions-hugo)
- [peaceiris/actions-gh-pages@v3](https://github.com/peaceiris/actions-gh-pages)

また, 今回使用したワークフローは以下のようになります:

```yaml
name: deploy gh-pages

on:
  push:
    branches:
      - source

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - name: checkout
        uses: actions/checkout@v2
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "0.76.0"

      - name: Build
        run: cd blog && hugo

      - name: deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./blog/public
```

ほぼ `peaceiris/actions-hugo@v2` にのっている例と同じです.
