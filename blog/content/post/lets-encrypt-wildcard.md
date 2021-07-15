---
title: "サイトをワイルドカード証明書を使って暗号化する"
date: 2021-07-15
description: "サイトをワイルドカード証明書を使って暗号化することで, ドメインが増えても HTTPS 対応ができるようにしました."
author: "Forell"
tags:
- DNS
- Security
categories:
- infra
---

お久しぶりです, 今年も部長の Forell ([@F0rell](https://twitter.com/F0rell)) です.<br/>
~~名前が違う？まあそこは察してください.~~

Let's Encrypt がワイルドカード証明書を発行するようになってからずいぶん経ちますが, LinuxClub が運用するサイトでは今までワイルドカード証明書を使わないでサイトを HTTPS 対応してきました.<br/>
そのようにしなかった理由として以下のことがあげられます.

1. 運用するドメインが少なかったこと
2. 上記に加えて増やす予定も今までなかったこと
3. ワイルドカード証明書の発行までが面倒なこと

特に, 1 と 3 の理由が大きく, 自動化含めてメリットがあんまりありませんでした. <br/>
しかし, 課外活動 HP[^1] の再作成にあたり, ステージング環境などを整えたいことと個人的に興味があったという理由からワイルドカード証明書の発行をすることといたしました.
一部サイトはサーバが分離されている都合上まだ証明書の更新はされていませんが, じきに更新したいと考えております.

ワイルドカード証明書の発行にあたり, 実際に行ったことや苦労した点をまとめたいと思います.

## やったこと

(ワイルドカード証明書の仕組みについては省略して説明いたします)

LinuxClub では ConoHa 上で VPS を 3 台借りており, そのうち 2 台は DNS を運用するためのソフトウェアである NSD を導入しております. <br />
詳しい構成や zone ファイル等は都合上明かすことはできませんが, linux.it.teu.ac.jp の名前解決をマスター, スレーブ合わせてこの 2 台で行っています.[^2]

まず, 以下のように zone ファイルを書き換えて更新するスクリプトを書きました. 最後の `sleep 600` はキャッシュによって DNS-01 チャレンジが失敗する可能性があるため 10 分ほどしてから DNS-01 チャレンジをするために挿入した感じです.

```sh
#!/bin/sh

FILENAME="/etc/nsd/linux.it.teu.ac.jp.zone"
BACKUPFOLDER="/etc/nsd/backups/"
OldSerial=`grep "Serial" $FILENAME | awk '{ print $1 }'`
OldDate=`echo $OldSerial | cut -c 1-8`
OldPostfix=`echo $OldSerial | cut -c 9-10`
NewDate=`date +"%Y%m%d"`
NewPostfix="01"

# backup old zone file
cp -rp $FILENAME ${BACKUPFOLDER}linux.it.teu.ac.jp_${OldSerial}

if [ "$NewDate" -eq "$OldDate" ]
then
    NewPostfix="0`expr $OldPostfix + 1`"
fi

if [ "$NewPostfix" -ge "10" ]
then
    NewPostfix=`echo $NewPostfix | cut -c 2-3`
fi

NewSerial="$NewDate$NewPostfix"
OldToken=`grep "_acme-challenge" $FILENAME | awk '{ print $5 }'`

sed -i -e 's/'$OldSerial'/'$NewSerial'/' $FILENAME

sed -i -e 's/'$OldToken'/'$CERTBOT_VALIDATION'/' $FILENAME

nsd-control reload

sleep 600
```

次に, 実際にワイルドカード証明書を発行できるかどうか試すために dry-run モードで certbot を実行します. (v オプションは詳細を見るために念のためつけました) <br/>
`certbot certonly --manual --server https://acme-v02.api.letsencrypt.org/directory -d '*.linux.it.teu.ac.jp --manual-auth-hook <上記スクリプトファイル>' --dry-run -v`

これで, エラーが発生しなければ `--dry-run` オプションを外して実際に実行して, ワイルドカード証明書を発行します. たまに, キャッシュ残って失敗することがありますが, その場合は sleep の時間を増やすなりしてキャッシュが消えるまで待つことをおすすめします.

証明書を手に入れたあとは, nginx などの web サーバでその証明書を指定し, `systemctl restart nginx` することで更新が完了いたします. <br/>
実際に, ワイルドカード証明書が反映されている例が以下になります.

![ワイルドカード証明書の反映](/post_media/lets-encrypt-wildcard/image.png)

## 苦労した点

今回の作業において, 苦労した部分は DNS-01 チャレンジが失敗してしまう部分にありました. というのも, キャッシュによって認証に必要なトークンが zone ファイルに書かれているのにも関わり実際に返されるトークンが違い, 何度も認証に失敗してしまいました. (dry-run モードで試しておいてよかったです) <br/>
とりあえず, sleep を使うという力技で解決させましたが, ここらへんはどうにかしたいところです.

また, 最初に書いたようにいまのところは一部サイトしかワイルドカード証明書は使われてない (それぞれの VPS に web サーバがある) ので, LinuxClub が保有する全てのサイトでワイルドカード証明書が使われるようにしたいですね.

[^1]: https://github.com/lc-tut/club-portal
[^2]: 補足: 実際には VPS 外, つまり LinuxClub が管理していないサーバにスレーブサーバが 1 つありますが, ここでは割愛させていただきます.
