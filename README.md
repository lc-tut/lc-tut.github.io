# LinuxClub BLOG

## 記事の書き方

GitHubアカウントをLinuxClubのOrganizationに追加する. (Adminへ依頼)

### 0. ブランチの使い方

- `master`: 公開するデータを配置する.
- `source`
  - hugoのプロジェクトを配置する.
  - 記事を書く時はここからブランチを切る.

### 1. GitHubからリポジトリをCloneする.

```shell
$ git clone https://github.com/lc-tut/lc-tut.github.io
$ cd lc-tut.github.io/
```

デフォルトブランチは`source`なので注意してください.

### 2. 記事を書く

ブランチを切ってください.

```shell
$ git branch koya/add-post-hoge
$ git checkout koya/add-post-hoge
```

`blog/content/post/`の中に記事を作成してください.

```shell
$ cd blog/content/post
$ cp fuga.md hoge.md  # ファイルをコピーする
$ vim blog/content/post/hoge.md
```

以下のテンプレートを使っても構いません.

```
+++
author = "koya"
title = "LinuxClub Blogをはじめました."
date = "2017-06-17"
description = "LinuxClubのブログで情報を発信していきます."
tags = [
    "world",
    "hello",
    "linuxclub"
]
categories = [
    "uncategorized",
]
+++

# ここにタイトル

ここに本文
```

細かなフォーマットは以下に書かれています.

[Front Matter | Hugo](https://gohugo.io/content-management/front-matter/)

画像などのメディアファイルは`blog/static/post_media/`に配置してください.

### 3. 記事をビルドする

記事はGitHub Actionsで自動ビルドされ，公開されます．
sourceブランチへ変更が加えられると自動で実行されます．
PullRequestを作成してsourceブランチに変更をMergeするとよいです．

以下の手順でローカルにHugoをインストールする.

[Install Hugo | Hugo](https://gohugo.io/getting-started/installing/)

記事のプレビューを確認:

```shell
$ cd blog
$ pwd
/path/to/lc-tut.github.io/blog
$ hugo serve
```

#### 4. GitHubへPush

`source`ブランチへファイル郡と記事をPushする.

```shell
$ git add -p
$ git commit -m 'YOUR MESSAGE'
$ git push origin HEAD
```

## 構築環境

```shell
$ go version
go version go1.11.10 linux/amd64

$ hugo version
Hugo Static Site Generator v0.56.0-DEV/extended linux/amd64 BuildDate: unknown
```
