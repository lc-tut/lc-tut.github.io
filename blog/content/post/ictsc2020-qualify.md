+++
author = "koyama"
title = "ICTSC2020 予選を11位で通過しました"
date = "2020-11-09"
description = "ICTSC2020 予選のWriteUpと感想です．"
tags = [
    "ICTSC",
    "contest",
    "infra"
]
categories = [
    "contest",
]
+++

ICTSC(ICTトラブルシューティングコンテスト) 2020にチーム「インターネット」で参加しました．結果は40チーム中11位で予選を通過しました．

[「ICTトラブルシューティングコンテスト2020」予選結果発表 – ICTSC | ICT トラブルシューティングコンテスト](https://icttoracon.net/archives/8648)

チーム名の「インターネット」はkoyamaが適当につけました．インターネットは人権ですね．教科書にも書いてあるはずです．

チームは以下のLinuxClubのメンバー4人です．

- [koyama](https://twitter.com/tmyk_kym)
- [panakuma](https://twitter.com/pana_pana_kuma)
- [Mikaner](https://twitter.com/MikanerExMachin)
- [cl0wn](https://twitter.com/cl0wn65536)


# Write-up

## ルーティング

### Networkが作成できない？ [unknown]

prefixがかぶっていることに気づいたタイミングで終了

公式の解説：[networkが作成できない？  |  ICTSC Tech Blog](https://blog.icttoracon.net/2020/11/02/network%e3%81%8c%e4%bd%9c%e6%88%90%e3%81%a7%e3%81%8d%e3%81%aa%e3%81%84%ef%bc%9f/)

### 経路を受け取れない [panakuma]
対象のルータにログインをし、journalctlでBGPのプロセスが出しているエラーを追っていると何らかの原因でプロセスが死んでいることが判明し、`/var/log`の中身を読んだりしているとメモリ不足が原因だと判明しました。
解決策としては、ルータに割り当てるメモリの量を増やすという単純な物でした。

### 繋がらなくなっちゃった！ [panakuma]
以下の事象が原因でトラブルが発生していた
- OSPFのルート交換をするインターフェースの指定が間違っていた
- OSPFのエリアの設定が間違っていた
- ルータIDが間違っていた

VyOSで動作していたため、とりあえず`show configuration`を叩いてみたり、OSPFのステータスを確認しながらトラブルシューティングをしていきました。

また、この問題ではなぜこのような事象が発生したかを問われていたので、問題文を参考にルータの設定をいじくっていたら間違えてしまったのだろうという仮説を元に回答をしました。

### またビルド失敗しちゃった～… [cl0wn]

マルチステージビルドをしているDockerfileがあります。一つ目のイメージで実行ファイルを作って、もう一つのイメージでそれを動かすということをしています。

エラーを見ると
```
standard_init_linux.go:211: exec user process caused "no such file or directory"
```

どうやら、一つ目のイメージにはあったファイルが2つ目のイメージにはなくてダイナミックリンクに失敗しているっぽいです。

したがって、static linkにしてやれば解決します。

```Dockerfile
RUN CGO_ENABLED=0 go build -o /app ./main.go
```

公式の解説：[またビルド失敗しちゃった～…  |  ICTSC Tech Blog](https://blog.icttoracon.net/2020/11/02/%e3%81%be%e3%81%9f%e3%83%93%e3%83%ab%e3%83%89%e5%a4%b1%e6%95%97%e3%81%97%e3%81%a1%e3%82%83%e3%81%a3%e3%81%9f%ef%bd%9e/)

## ウェブ

### WEBページが見れない [cl0wn]

この問題は「apacheサーバーのdocumentルートを/home/user/html/に変更したところ、そのディレクトリ以下のファイルhome.htmlがブラウザから閲覧できない」という問題を解消しなさいという問題でした。

まず、`/home`、`/home/user`、`/home/user/html`ディレクトリのothersに実行権限、`/home/user/html`以下のhome.htmlのothersに閲覧権限が付与されているかを確認しました。すると`/home/user`のothersに実行権限が付与されていないことが分かりました。したがって、`chmod o+x /home/user`しました。

この状態で`/home.html`にアクセスするとまだ表示されません。エラーログには`client denied by server configuration: /home/user/html/home.html`が吐かれていました。上で権限の設定はしたので、なんでだろうなと思って調べてたら、[この記事](https://qiita.com/nwsoyogi/items/c8eb1fedef3c00c5fbac)を見つけました。試しに`/etc/httpd/conf/httpd.conf`に以下を追記してみます。

```
<Directory "/home/user/html">
    Require all granted
</Directory>
```

エラー内容が変わりました。どうやらうまく動いたようです。

どんなエラーが出ていたかというと`AH00132: file permissions deny server access`でした。問題文に「SELinuxを無効にしてはいけない」と書いてあったのでSELinuxのせいかなと思いました。試しに`setenforce Permissive`にすると、`/home.html`が正常に表示されるようになりました。`ls -Z /home/user/html/home.html`を使ってSELinuxのコンテキストを確認してみるとファイルタイプが`user_home_t`になっていました。apacheがファイルを読むためにはファイルタイプが`httpd_sys_content_t`に設定されていなければならないそうです。したがって、ファイルタイプを変更すれば問題は解決しそうです。chconによるファイルタイプの変更はリブート時にもとに戻ってしまうそうなので、`semanage`を使ってファイルタイプを変更します。`setenforce Enforcing`でSELinuxの設定をもとに戻して、`sudo yum install policycoreutils-python`でsemanageをインストール、`sudo semanage fcontext -a -t httpd_sys_content_t /home/user/html/home.html`を実行しました。すると正常に`/home.html`が閲覧できるようになりました。

公式の解説：[WEBページが見れない  |  ICTSC Tech Blog](https://blog.icttoracon.net/2020/11/02/web%e3%83%9a%e3%83%bc%e3%82%b8%e3%81%8c%e8%a6%8b%e3%82%8c%e3%81%aa%e3%81%84/)

## コンテナ

### どこからもアクセスできなくなっちゃった [koyama]

LoadBalancerの先にあるKubernetesクラスタを対象に，クライアントからのアクセス制御を適切に行う問題でした．アクセス制御にはKubernetesのNetworkPolicyを使う制約が設定されていました．

リクエストのフローを整理すると以下になります．

```
host01
 | src: 192.168.0.10
 | dst: 192.168.0.1:30080
 |
 v
lb
 | src: 172.16.0.254
 | dst: 172.16.0.11:30080 or 172.16.0.12:30080
 |
 v
worker01 or worker02
```

この段階で，異なるネットワークセグメントを超える必要があることが分かりました．lbにはHAProxyがインストールされていて，IP Masqueradeが設定されているということだったのでL4 Direct Server Returnができそうだなと思いました．回答例ではKeepalived + LVSの組み合わせでやっていたのでつらぽよでした．

HAProxyの設定ファイルと格闘していたら時間切れになったので悲しかったです．HAProxyに不慣れなせいで，設定ファイルの記法で怒られまくりました．default, global, front, backendを上手く書く方法を学びたいなと思います．HAProxyはしばらく触りたくないです．

公式の解説: [どこからもアクセスできなくなっちゃった | ICTSC Tech Blog](https://blog.icttoracon.net/2020/11/02/%e3%81%a9%e3%81%93%e3%81%8b%e3%82%89%e3%82%82%e3%82%a2%e3%82%af%e3%82%bb%e3%82%b9%e3%81%a7%e3%81%8d%e3%81%aa%e3%81%8f%e3%81%aa%e3%81%a3%e3%81%a1%e3%82%83%e3%81%a3%e3%81%9f/)

### hostnameでつながらない！！ [Mikaner]
docker-compose で立てたDatabaseサーバーのhostnameをdatabaseにしたところ，databaseで参照してもipが取れないという問題でした。

hostnameを変更してもDockerのDNSに反映されないみたいなので，Dockerのコンテナ名を変更しました。

変更前
(あとで追記)

変更後
(あとで追記)

### ダイエットしようぜ！ [koyama, cl0wn]

GoでSHA256を計算するコードとDockerfileが渡されました．与えられたDockerfileにより作成したイメージはサイズが796MBでした．問題では，このイメージサイズを出来る限り削減することが求められました．

koyamaは以下の方針でイメージサイズの削減に取り組みました．

- Dockerのマルチステージビルドの利用
- 軽量コンテナイメージ [scratch](https://hub.docker.com/_/scratch) の利用 
- [Goのビルドオプション `-w` と `-s` の付与 ](http://tdoc.info/blog/2016/03/01/go_diet.html)

実際のDockerfileは以下です．

```
FROM golang:alpine AS build-env
COPY hash.go /work/
WORKDIR /work
RUN go build -ldflags="-w -s" -o myhash hash.go

FROM scratch
COPY --from=build-env /work/myhash /myhash
ENTRYPOINT ["/myhash"]
```

イメージサイズが**1.49MB**になりました．

```
$ docker images | grep ictsc-dit
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
ictsc-dit                latest              f74546496cca        8 minutes ago       1.49MB
```

upxによるバイナリサイズの削減ができそうだったので，cl0wnにお願いしました．

goupxというツールを見つけたのでそれを利用しました。

Dockerfileを以下のように変更しました。

```
FROM golang:alpine AS build-env
RUN apk add --no-cache git upx binutils
RUN mkdir /gobin
ENV GOBIN /gobin
ENV PATH $PATH:/gobin
COPY hash.go /work/
WORKDIR /work
RUN go build -ldflags="-w -s" -o myhash hash.go
RUN strip myhash
RUN go get github.com/pwaller/goupx
RUN goupx myhash

FROM scratch
COPY --from=build-env /work/myhash /myhash
ENTRYPOINT ["/myhash"]
```
[参考](https://qiita.com/circus/items/450254c59d194cbf22d7)

これによりイメージのファイルサイズが600kBまで落ちました。goのバイナリにupxをそのまま使うと実行できなくなるという話を聞いたのでチキってgoupxを使いましたが、オプションが使えないというデメリットがありました。upxの`-9`を試してみたかったなと思いました。


公式の解説: [ダイエットしようぜ！ | ICTSC Tech Blog](https://blog.icttoracon.net/2020/11/02/%e3%83%80%e3%82%a4%e3%82%a8%e3%83%83%e3%83%88%e3%81%97%e3%82%88%e3%81%86%e3%81%9c%ef%bc%81/)

## データベース

### 備品は何処へ [Mikaner]
データベースから必要な情報を取ってくるSQLを書き，それを実行してCSVファイルに書き込み，SQLファイルとCSVファイルの両方を提出すれば良い問題でした。

Mikanerは以下の手順で問題の回答に当たりました。
1. CSVファイルの書き出し方法を検索
    => [ここ](https://weblabo.oscasierra.net/mysql-select-into-outfile/)が見つかったので参考にしました。
2. データベース内の構造を確認(下の図参照)
3. SQLを結合し，データが取得できるか確認, 指定されたデータの全データ出力
4. WHERE句とORDER BY句を追加し出力
5. 不備が無いかを確認し提出

以下はコンテスト中に作成した図と提出したSQLのメモです。

<img src="/post_media/ictsc2020-qualify/mysql.png">


#### SQLの結合
SELECT句にて必要な情報を並べ，あとは結合するだけの簡単なお仕事でした。
AS句を使えばもっと楽に書けたのですが，当時のMikanerさんはAS句の使い方をうろ覚えだったので，確実性を求めてメモにコピペをして済ませていました。
楽だから使えばいいのにね。
```sql
USE List;
SELECT Equipment_list.ID, Equipment_list.Name, Order_company_list.Name, Manufacturing_company_list.Name, Equipment_list.Price
FROM Equipment_list
INNER JOIN Order_company_list
ON Equipment_list.Order_company = Order_company_list.ID
INNER JOIN Manufacturing_company_list
ON Equipment_list.Manufacturing_campany = Manufacturing_company_list.ID -- ここ地味にc"a"mpanyになっててうざかった
INNER JOIN Location_list
ON User_place = Location_list.ID
INTO OUTFILE '/tmp/submit.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
ESCAPED BY '"'
LINES TERMINATED BY '\r\n';
```
このあたりでKoyamaさんとPanakumaさんにSQLを見てもらったところ，INNER JOINはあまり使わないという意見を頂いたので，本番環境を意識してLEFT JOINに変更しました。

ここで変更したファイルで意気揚々と出力させようとしたところ，submit.csvがすでにあるから書けないという問題が発生しました。
/tmp内にアクセスして削除を試みましたが，権限がないため削除ができませんでした。sudo権限ください。
面倒くさかったので出力名を'test1.csv'に変更して出力しました。
このあとは実行した分だけ番号が増えていきます。
```sql
USE List;
SELECT Equipment_list.ID, Equipment_list.Name, Order_company_list.Name, Manufacturing_company_list.Name, Equipment_list.Price
FROM Equipment_list
LEFT JOIN Order_company_list -- 変更
ON Equipment_list.Order_company = Order_company_list.ID
LEFT JOIN Manufacturing_company_list -- 変更
ON Equipment_list.Manufacturing_campany = Manufacturing_company_list.ID
LEFT JOIN Location_list -- 変更
ON User_place = Location_list.ID
INTO OUTFILE '/tmp/test1.csv' -- 変更
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
ESCAPED BY '"'
LINES TERMINATED BY '\r\n';
```
#### 出力結果
あとで追記。
#### WHERE句とORDER BY句を追記
あとは並び替えるだけと思って問題文よく読んだら，HQという場所にあるE社のPCだけを出力するっぽいことがわかったので，WHERE句とORDER BY句を追記しました。
よく読まなかったら危なかった。
```sql
USE List;
SELECT Equipment_list.ID, Equipment_list.Name, Order_company_list.Name, Manufacturing_company_list.Name, Equipment_list.Price 
FROM Equipment_list
LEFT JOIN Order_company_list
ON Equipment_list.Order_company = Order_company_list.ID
LEFT JOIN Manufacturing_company_list
ON Equipment_list.Manufacturing_campany = Manufacturing_company_list.ID
LEFT JOIN Location_list
ON Use_place = Location_list.ID
WHERE Manufacturing_company_list.Name='Manufacturing_company_E' AND Location_list.Building_name='HQ' -- 追記
ORDER BY Equipment_list.Price DESC -- 追記
INTO OUTFILE '/tmp/test2.csv' -- 変更
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
ESCAPED BY '"'
LINES TERMINATED BY '\r\n';
```
#### 不備の確認
出力先ファイルをsubmit.csvに変更し，出力したCSVファイルの名前もsubmit.csvに変更し，提出しました。提出直前にLocation_listとEquipment_listをつないでいるON句でUse_placeのテーブル指定を忘れていたことに気付きましたが，修正面倒だしFROMの方だから問題ないやろってお気持ちで放置しました。

公式の解説:[備品は何処へ | ICTSC Tech Blog](https://blog.icttoracon.net/2020/11/02/%e5%82%99%e5%93%81%e3%81%af%e4%bd%95%e5%87%a6%e3%81%b8/)

## プログラミング

### jsonが壊れた！！ [koyama]

誤りのあるGoのソースコードが渡されるので，それを修正して正常な動作が確認できればOKな問題でした．

構造体の記述に問題があり，ダブルクオートの欠落とJSONスキーマ指定に不備がありました．

変更前

```
type User struct {
        ID int32 `json="id"`
        Name string `json="name"`
        PasswordHash string `json="password_hash"`
        Count int32 `json="access_count`
}
```

変更後

```
type User struct {
        ID int32 `json:"id"`
        Name string `json:"name"`
        PasswordHash string `json:"password_hash"`
        Count int32 `json:"access_count"`
}
```

公式の解説: [jsonが壊れた！！ | ICTSC Tech Blog](https://blog.icttoracon.net/2020/11/02/json%e3%81%8c%e5%a3%8a%e3%82%8c%e3%81%9f%ef%bc%81%ef%bc%81/)

# 感想

## Mikaner

## cl0wn

- エラーで調べれば対処が書かれていることが多く、筋道立っていてありがたかったです。
- upxもうちょっとやりたいです
- ネットワーク系の問題を遠ざけていましたが、Networkが作成できない？とかやればよかったかなと思ってます
- ネットワーク系の問題を本選までに解けるようになりたいなーと思う今日この頃です

## panakuma
低レイヤーで問題が少なかったので、あまり歯ごたえがなかったなぁという感じです。問題のクオリティが例年より低いなと感じました。

## koyama

簡単な問題で確実に得点できたのが良かったです．cl0wnとMikanerの2人が活躍してくれたおかげだと思います．ダイエット問は自身があったものの，Twitterで300KB程度のチームを見つけて悲しくなりました．Kubernetes問をやりながらLBでアクセス制御しちゃダメなのが納得できない気持ちでした．本戦に向けて精進したいと思います．
