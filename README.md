# LinuxClub BLOG 

## 記事の書き方

GitHubアカウントをLinuxClubのOrganizationに追加する. (Adminへ依頼)

### 1. GitHubからリポジトリをCloneする.

```shell
$ git clone https://github.com/lc-tut/lc-tut.github.io
$ cd lc-tut.github.io/
```

### 2. 記事を書く

ブランチを切ってください.

`blog/content/post/`の中に記事を作成してください.

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
$ hugo serve
```

記事をローカルでビルド:

```shell
$ cd blog
$ hugo -d ../docs
```

#### 3-2. Dockerでビルド

```shell
$ cd lc-tut.github.io/
$ docker run --rm -it -v $PWD:/src -u hugo jguyomard/hugo-builder hugo
```

#### 4. GitHubへPush

執筆中...

```
$ git checkout source
$ git add -A
$ git commit -m 'YOUR MESSAGE'
$ git push origin HEAD
$ git subtree push --prefix docs/ origin master
```

## 構築環境

```shell
$ go version
go version go1.11.10 linux/amd64

$ hugo version
Hugo Static Site Generator v0.56.0-DEV/extended linux/amd64 BuildDate: unknown
```

## 参考資料

[GitHub PagesのUser Pagesでドキュメントルートを変更するにはmasterを殺す - Qiita](https://qiita.com/kwappa/items/03ffdeb89039a7249619)
