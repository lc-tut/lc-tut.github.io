+++
author = "koyama"
title = "スポーツ大会エントリサイトを支えた技術 2018"
date = "2018-06-20"
description = "LinuxClubでは2018年度スポーツ大会エントリーサイトの運営を行いました。この記事では使用した技術やシステム運用の詳細を紹介します。"
tags = [
    "web",
    "application",
    "Python",
    "Server",
    "Security",
    "Nginx",
    "TLS"
]
categories = [
    "Infra",
]
+++

## はじめに

LinuxClubではスポーツ大会のエントリーサイトを運営しています。今年も体育会からの要請があり運営を行いました。この記事では運用に際し改善、改良を行った点を紹介します。

## DNS

従来は `www2.linux.it.teu.ac.jp/sports20xx/` の下に設置していました。今年度のサイトはサブドメイン  `sports.linux.it.teu.ac.jp` に設置しました。

## Nginx

担当: panakuma

ページ読み込みの高速化によるユーザビリティとセキュリティの向上のためHTTP/2とTLSv1.3に対応させました。

### TLSv1.3対応

TLSv1.3はまだドラフト段階の規格なので5月現在で最新であったdraft22/23に対応しました。

使用したプログラム
- Nginx 1.13.10
- OpenSSL 1.1.1-pre3

それぞれソースを公式サイトからダウンロードし、OpenSSLを組み込んでNginxをmakeしました。

### レスポンスヘッダ/Cookie属性の変更

担当: koyama

セキュリティを高めるためにレスポンスヘッダへ以下を付与しました。

- strict-transport-security: max-age=15768000; includeSubdomains
- x-frame-options: SAMEORIGIN
- x-xss-protection: 1; mode=block
- x-content-type-options: nosniff
- cache-control: private, no-store
- pragma: no-cache

<img src="/post_media/sportsfes2018/curl.png" width="500">

それぞれのヘッダの役割は以下です。

- `strict-transport-security` はhttpsへ強制リダイレクトを実現しています。
- `x-frame-options` はクリックジャッキング対策で付与しています。
- `xss-protection` はブラウザのXSSフィルターを強制的にONに設定します。
- `x-content-type-options` はContent-Typeの誤認識によるXSSを回避するために付与しています。
- `cache-control` はプロキシサーバなどで機密情報が保存されないために付与しています。
- `program` は古いブラウザでのキャッシュ保持を防ぎます。

ブラウザのCookieに `secure` 属性と `httponly` 属性を付与しました。 `secure` 属性はクッキーをhttpsでやり取りするために付与します。 `httponly`　属性はJavaScriptからのCookieへのアクセスを禁止します。

セッションのタイムアウトを実装するためにCookieの `expires` 属性へセッション期限を設定しています。

これらはフレームワークの機能を使っています。そのため、セッションタイムの実装などを直接書かずに実現しています。

## ソースコードの修正(サーバサイド)

担当: koyama

既存のPythonコードを修正することで、チーム数制限を実装しました。従来はエントリー可能なチーム数の上限を設定していなかった為、上限なくエントリーできてしまいました。今年度のシステムはエントリー済みチーム数を取得することによって、エントリー上限に達するとエントリができなくなる仕組みを作りました。

<img src="/post_media/sportsfes2018/basketball.png" width="500">

## ソースコードの修正(フロント)

担当: homirun

### デザインの変更
例年ページのテーマカラーが変わっていたため今年も気分で変えてみました。
また競技のルールを表示するページをアコーディオンボタンで開けるようにしました。
```javascript
let check = 0
$('.expander').click(function() {
	if (check === 0){
		$(this).closest('article').children('aside').show(300);
		$(this).children('i').removeClass('fa-angle-down')
		$(this).children('i').addClass('fa-angle-up')
		check = 1
	}else{
		$(this).closest('article').children('aside').hide(300);
		$(this).children('i').removeClass('fa-angle-up')
		$(this).children('i').addClass('fa-angle-down')
		check = 0
	}
});
```
checkで現在のメニューの開閉状態を保持しています。
checkが0のときはarticleの子要素を表示させ、1のときは非表示にさせてます。

そのほか`Font Awesome`を用いボタンのアイコンなどわかりやすいものに変更しました。

　
### UI崩れ
モバイル端末でトップページを開いた際に画像がはみ出してしまう事があったのでCSSの該当箇所に`max-width: 100%`を追記し修正しました。

## 脆弱性診断を実施

担当: koyama

オープンソースの脆弱性診断ツールである `OWASP ZAP` と `BurpSuite(Community Edition)`を使用しました。

まず、 `OWASP ZAP` のスキャン機能を使って既知な脆弱性を探しました。指摘された事項としては、レスポンスヘッダに `Progma` ヘッダが未付与であることだけでした。

次に、 `BurpSuite` （ローカルプロキシ）を使って手動で診断します。ブラウザを起動してプロキシの設定を `localhost:8080` に設定します。BurpSuiteのProxyタブ内からIntereptをOffにしておきます。診断は以下の手順で行います。

1.ブラウザを起動してWebサイトにアクセスします。

2.Proxyタブ内から`HTTP Proxy`を選ぶとリクエスト一覧が表示されます。

<img src="/post_media/sportsfes2018/burp-httpHistory.png" width="500">

3.リクエスト一覧から診断したいリクエストを見つけて、`右クリック -> Send to Repeater` を実行します。

4.Repeaterタブを開いてリクエストを書き換えて `Go` をクリックします。右側にレスポンス結果が表示されるので、この結果から脆弱性の有無を判断します。

<img src="/post_media/sportsfes2018/burp-repeater.png" width="500">

以下は診断過程の一部です。クロスサイトスクリプティングの脆弱性がないか確認をしています。

<img src="/post_media/sportsfes2018/xss-check.png" width="500">

## 運用

担当: koyama

問い合わせ先はGoogle Formを使用しました。また、Google Formが送信されるとSlackへ通知されるようプラグインを設定しました。

<img src="/post_media/sportsfes2018/google-form.png" width="500">

サイト監視用スクリプトを作成しました。スクリプトは実行されるとcurlコマンドを実行してステータスコードを確認します。ステータスコードが400未満であれば正常であると判断します。それ以外の場合は異常であると判断してSlackに通知を送信します。

```shellscript
#!/bin/bash

URL="https://sports.linux.it.teu.ac.jp/"
DEBUG=1
DATE=$(date "+%Y/%m/%d %H:%M:%S")

# Send Req
RESPONSE_CODE=$(curl -LI $URL -o /dev/null -w '%{http_code}\n' -s)

# Handle ReqCode
if [ $RESPONSE_CODE -lt 500 ] ; then # RESPONSE_CODE < 500

  # SUCCESS
  test $DEBUG -gt 0 && echo "[DEBUG] Successful access"

else
  
  # FAIL
  test $DEBUG -gt 0 && echo "[DEBUG] Fail to access"

  ### Post to Slack
  TOKEN=''
  USER='SiteChecker'
  CHANNEL='sportsfes'
  MESSAGE="[Error:$RESPONSE_CODE] Fail to access $URL at $DATE"

  curl -s -XPOST -d "token=$TOKEN" \
                 -d "channel=#$CHANNEL" \
                 -d "text=$MESSAGE" \
                 -d "username=$USER" \
                 "https://slack.com/api/chat.postMessage" # >& /dev/null &

fi
```

<img src="/post_media/sportsfes2018/site-checker.png" width="500">

Nginxのログ解析用のスクリプトをPythonで作成しました。

<script src="https://gist.github.com/tomoyk/b7502d2f2a19e21291d8ad60eb2892b6.js"></script>

<img src="/post_media/sportsfes2018/chart.png" width="500">
