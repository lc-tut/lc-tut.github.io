# LC-BLOG DOCUMENT

## 更新方法

	# Remoteから取得
	git clone https://github.com/lc-tut/lc-tut.github.io
	cd lc-tut.github.io/
	
	# Dockerコンテナの取得
	docker pull jekyll/jekyll
	
	# 記事を書く
	cd _posts/
	vim 2017-01-01-yourTitle.md
	cd ..
	
	# Dockerコンテナでビルド
	docker run --rm --volume=$(pwd):/srv/jekyll -it jekyll/jekyll:pages jekyll build
	
	# Dockerコンテナでサーバ起動
	docker run --rm --volume=$(pwd):/srv/jekyll -it -p 4000:4000 jekyll/jekyll:pages jekyll serve --watch --incremental --force_polling --verbose
	
	# ブラウザからlocalhost:4000へアクセス
	w3m http://localhost:4000/

	# ブログ記事を書いたらslackのblogチャンネルで教えてください.

## このブログについて(環境)

**Jekyll + Github Pages**で構築してあります。

サイト開発は手元のローカルで行いました。構築は以下の手順を参考にしました。

[MacにHomeBrew,rbenv,bundlerをインストールする - Qiita](http://qiita.com/shinkuFencer/items/3679cfd966f6a61ccd1b)

- jekyll : `jekyll 3.4.3`
- ruby : `ruby 2.0.0p648 (2015-12-16 revision 53162) [universal.x86_64-darwin16]`
- bundle : `Bundler version 1.15.1`
- gem : `2.0.14.1`

### ローカル開発環境

`$ github-pages versions`

    +------------------------------+---------+
    | Gem                          | Version |
    +------------------------------+---------+
    | jekyll                       | 3.4.3   |
    | jekyll-sass-converter        | 1.5.0   |
    | kramdown                     | 1.13.2  |
    | liquid                       | 3.0.6   |
    | rouge                        | 1.11.1  |
    | github-pages-health-check    | 1.3.4   |
    | jekyll-redirect-from         | 0.12.1  |
    | jekyll-sitemap               | 1.0.0   |
    | jekyll-feed                  | 0.9.2   |
    | jekyll-gist                  | 1.4.0   |
    | jekyll-paginate              | 1.1.0   |
    | jekyll-coffeescript          | 1.0.1   |
    | jekyll-seo-tag               | 2.2.3   |
    | jekyll-github-metadata       | 2.3.1   |
    | jekyll-avatar                | 0.4.2   |
    | jemoji                       | 0.8.0   |
    | jekyll-mentions              | 1.2.0   |
    | jekyll-relative-links        | 0.4.0   |
    | jekyll-optional-front-matter | 0.1.2   |
    | jekyll-readme-index          | 0.1.0   |
    | jekyll-default-layout        | 0.1.4   |
    | jekyll-titles-from-headings  | 0.1.5   |
    | listen                       | 3.0.6   |
    | activesupport                | 4.2.8   |
    | minima                       | 2.1.1   |
    | jekyll-swiss                 | 0.4.0   |
    | jekyll-theme-primer          | 0.2.1   |
    | jekyll-theme-architect       | 0.0.4   |
    | jekyll-theme-cayman          | 0.0.4   |
    | jekyll-theme-dinky           | 0.0.4   |
    | jekyll-theme-hacker          | 0.0.4   |
    | jekyll-theme-leap-day        | 0.0.4   |
    | jekyll-theme-merlot          | 0.0.4   |
    | jekyll-theme-midnight        | 0.0.4   |
    | jekyll-theme-minimal         | 0.0.4   |
    | jekyll-theme-modernist       | 0.0.4   |
    | jekyll-theme-slate           | 0.0.4   |
    | jekyll-theme-tactile         | 0.0.4   |
    | jekyll-theme-time-machine    | 0.0.4   |
    +------------------------------+---------+

### 作成した時のGithubでのプラグインサポート状況

- Dependency  Version
- jekyll  3.4.5
- activesupport  4.2.8
- github-pages-health-check  1.3.4
- github-pages  142
- html-pipeline  2.6.0
- jekyll-avatar  0.4.2
- jekyll-coffeescript  1.0.1
- jekyll-default-layout  0.1.4
- jekyll-feed  0.9.2
- jekyll-gist  1.4.0
- jekyll-github-metadata  2.4.0
- jekyll-mentions  1.2.0
- jekyll-optional-front-matter  0.1.2
- jekyll-paginate  1.1.0
- jekyll-readme-index  0.1.0
- jekyll-redirect-from  0.12.1
- jekyll-relative-links  0.4.1
- jekyll-sass-converter  1.5.0
- jekyll-seo-tag  2.2.3
- jekyll-sitemap  1.0.0
- jekyll-swiss  0.4.0
- jekyll-theme-architect  0.0.4
- jekyll-theme-cayman  0.0.4
- jekyll-theme-dinky  0.0.4
- jekyll-theme-hacker  0.0.4
- jekyll-theme-leap-day  0.0.4
- jekyll-theme-merlot  0.0.4
- jekyll-theme-midnight  0.0.4
- jekyll-theme-minimal  0.0.4
- jekyll-theme-modernist  0.0.4
- jekyll-theme-primer  0.2.1
- jekyll-theme-slate  0.0.4
- jekyll-theme-tactile  0.0.4
- jekyll-theme-time-machine  0.0.4
- jekyll-titles-from-headings  0.2.0
- jemoji  0.8.0
- kramdown  1.13.2
- liquid  3.0.6
- listen  3.0.6
- minima  2.1.1
- nokogiri  1.7.2
- rouge  1.11.1
- ruby  2.4.0
- safe_yaml  1.0.4
- sass  3.4.25

## テーマについて

OSSテーマ「 blog 」を採用しました。作者が中国語圏の方みたいでコメントが読めません。

[WakelessDragon/blog: 博客](https://github.com/WakelessDragon/blog)

### テーマのカスタマイズ

テーマを一部修正しました。修正した箇所は以下です。

- head.html
	- favicon
- style.css
	- BlogのDescriptionをモバイルUIで非表示
	- フォント設定を日本語に最適化
- post.html
	- SNSをTwitter, はてブ, Pocketに変更
-  script.js
	- consoleに出る謎のメッセージを削除

### ビルド時のエラー

    $ jekyll s
    Configuration file: /Users/MYNAME/TEST_BLOG/blog/_config.yml
    Configuration file: /Users/MYNAME/TEST_BLOG/blog/_config.yml
    Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `gems: [jekyll-paginate]` in your configuration file.
    Source: /Users/MYNAME/TEST_BLOG/blog
    Destination: /Users/MYNAME/TEST_BLOG/blog/_site

ビルドすると`jekyll-paginate`が不足していると怒られるので`_config.yml`の末尾に`jekyll-paginate`に関する記述を以下の形式で追加した。

    gems: 
      - jekyll-paginate

### 各ファイルの役割

- ./_config.yml メインのコンフィグ
- ./_includes/footer.html 共通フッター
- ./_includes/head.html 共通ヘッダー(headタグ)
- ./_includes/header.html 共通ヘッダー
- ./static/css/style.css メインのCSSファイル
- ./static/css/* 個別のCSSファイル
- ./static/js/* 個別のJSファイル
- ./static/img/* 固定ページ(ポスト以外)画像やサイトの素材
- ./pages/* 固定ページ, 各ページ(Archive, Category, Tag)
- ./_posts/* ブログ記事（ポストデータ）
- ./_layouts/* 各ページやデフォルトのレイアウトテンプレート
- ./images/* 個別ブログ記事の画像

## 参考URI

- [Dependency versions | GitHub Pages](https://pages.github.com/versions/) Githubで動くプラグイン
- [Adding a Jekyll theme to your GitHub Pages site - User Documentation](https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site/) サイトの建て方
- [GitHub Pages の Jekyll で Web サイトを無料公開する方法 - Qiita](http://qiita.com/takuya0301/items/374b2ab5be407b138ef9) サイトの建て方

