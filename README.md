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

画像などのメディアファイルは`blog/static/post_media/`に配置してください.

### 3. 記事をビルドする

ビルドには次の2つの方法があります.

- ローカルにHugoをインストールしてビルド
- DockerのHugoイメージでビルド

#### 3-1. ローカルでビルド

以下の手順でローカルにHugoをインストールする.

[Install Hugo | Hugo](https://gohugo.io/getting-started/installing/)

記事のプレビューを確認:

```shell
$ cd blog
$ pwd
/path/to/lc-tut.github.io/blog
$ hugo serve
```

記事をローカルでビルド:

```shell
$ cd blog
$ pwd
/path/to/lc-tut.github.io/blog
$ hugo -d ../docs
```

#### 3-2. Dockerでビルド

執筆中

```shell
$ cd lc-tut.github.io/
$ docker run --rm -it -v $PWD:/src -u hugo jguyomard/hugo-builder hugo
```

#### 4. GitHubへPush

`source`ブランチへビルドしたファイル郡と記事をPushする.

```shell
$ git checkout source
$ git add -A
$ git commit -m 'YOUR MESSAGE'
$ git push origin HEAD
```

`master`ブランチへ公開するファイルをPushする.

```shell
$ git subtree push --prefix docs/ origin master
```

`git subtree`は古いgitには含まれないので追加を行う.

## 構築環境

```shell
$ go version
go version go1.11.10 linux/amd64

$ hugo version
Hugo Static Site Generator v0.56.0-DEV/extended linux/amd64 BuildDate: unknown
```

## 参考資料

[GitHub PagesのUser Pagesでドキュメントルートを変更するにはmasterを殺す - Qiita](https://qiita.com/kwappa/items/03ffdeb89039a7249619)
