+++
author = "koyama"
title = "初心者向けハンズオンを行いました"
date = "2018-02-03"
description = "初心者を対象にWeb/サーバー構築/Python/C++の講習会を行いました。"
tags = [
    "ビギナー",
    "ハンズオン",
    "講習会"
]
categories = [
    "Activity",
]
+++

次期会計らしいKoyamaです。

11月から毎週火曜日に初心者/中級者を対象にそれぞれが得意とする分野のテーマでハンズオンを行いました。テーマはWeb(HTML/CSS), インフラ(WordPress), プログラミング(Python, C++)です。この記事では各回を担当したメンバーがハンズオンの概要について解説していきます。

## Web(HTML, CSS)

Webを担当したhomirunです。
今回はHTML5とCSS3について説明しました。
以下のスライドを使用しました。
<iframe src="//www.slideshare.net/slideshow/embed_code/key/tWJ0eEOnFaPPpV" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/chinokf/htmlcss-85884198" title="HTML_CSS" target="_blank">HTML_CSS</a> </strong> from <strong><a href="https://www.slideshare.net/chinokf" target="_blank">homirun </a></strong> </div>


## サーバ構築(WordPress)

Koyamaとpanakumaが担当しました。Windows上に作成したVMを使ってWebサーバ(nginx)とDBサーバ(MySQL)を構築してWordPressを設置しました。VMの環境構築に時間がかかった為、2回に分けて行いました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">サーバー構築のLT中！<a href="https://twitter.com/hashtag/LC%E5%8B%89%E5%BC%B7%E4%BC%9A?src=hash&amp;ref_src=twsrc%5Etfw">#LC勉強会</a> <a href="https://twitter.com/hashtag/LinuxClub?src=hash&amp;ref_src=twsrc%5Etfw">#LinuxClub</a> <a href="https://t.co/fuxVjW1d3A">pic.twitter.com/fuxVjW1d3A</a></p>&mdash; 東京工科大学 LinuxClub (@lc_tut) <a href="https://twitter.com/lc_tut/status/932917213232635905?ref_src=twsrc%5Etfw">2017年11月21日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

資料はGithubに公開しました。

[tomoyk/ServerHandson](https://github.com/tomoyk/ServerHandson)

## プログラミング(Pyhon)

### 自己紹介
Pythonの初心者入門 および スクレイピングハンズオンを担当したパナくまです。
Pythonのコーディング経験は6ヶ月弱ぐらいですが、書きやすい言語なので簡単なことをサクッと書くくらいは出来るようにスライドを作りました。
### 担当したテーマ
- 初心者入門
    - 基本構文の使い方
- スクレイピングハンズオン
    - beautifulsoup4を使った簡単なスクレイピング

### テーマの内容
#### 初心者入門
まずはプログラミング言語の要である変数やfor文 if文 ユーザー定義関数などを簡単に説明しました。
#### スクレイピングハンズオン
beautifulsoupライブラリを用いて[妹さえいればいい。の新着情報](http://imotosae.com/news/)のトピックスを拾うというプログラムを作りました。
### サンプルソース
#### スクレイピングハンズオン

```python:imosae-soup.py

from urllib import request
from bs4 import BeautifulSoup as BS

def main():
    url = "http://imotosae.com/news/"
    req = request.Request(url)
    res = request.urlopen(req)
    html = res.read()
    soup = BS(html, "lxml")
    topics = soup.find_all('h1', 'c-thumb-index__title')
    for i in range(len(topics)):
        print(topics[i].string, "\n")

if __name__ == '__main__':
    main()

```
### 資料
#### 初心者入門

<script async class="speakerdeck-embed" data-id="4055177b0da94f1e9e7ce194cdd2e595" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

#### スクレイピングハンズオン

<script async class="speakerdeck-embed" data-id="b2fbd13d5ac94fb4bf38cf95cffebad4" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>


## プログラミング(C++でポインタ入門)
### 自己紹介
C＋＋で主にポインタの講座を担当したLapuaです。ポインタやC＋＋への理解は浅いですが、4,5月頃に自分が勉強した際にどこがわかりにくかったかを思い出してスライドを作りました。

### このテーマの狙い
メンバーの中に「C言語を勉強したことあるけどポインタで躓いた」という声を何人かから聞いたのがテーマ決めの経緯でした。C/C++未経験者も多くいらっしゃいましたが、参照渡しと値渡しの違いを理解してもらえれば他の言語でも役立つだろうと考えました。当初はC言語を扱う予定でしたが、入門者にとって標準出力(printfやcout)がC＋＋の方がわかりやすいだろうと考えC＋＋を採用した。とにかく混乱しないように難易度を低めにすることを心掛けました。

### テーマ内容
#### 1. std::cout
C++の前提知識を必要としない内容ということなのでHello,wolrdから解説をしました。その後慣れてもらうためにFizzBuzzをアクティブラーニング形式で実施した。

#### 2. &とポインタ変数
時間と難易度を考慮してメモリについての説明は簡易的に済ませ、コードを書いて実行してみてから理解することを狙いました。

![](https://i.imgur.com/B4HVwbx.png)
![](https://i.imgur.com/AQB7G1C.png)

コードを実行結果を見比べて説明。変数の前に&をつけるとアドレスを取り出す。宣言時に*をつけるとポインタ変数というアドレスを格納する変数を宣言することができる。このように説明したがうまく理解してもらえただろうか...

#### 3. ポインタを使った少し実践的な内容
値渡しと参照渡しの有名なサンプルコードのひとつはこれでしょう。

![](https://i.imgur.com/EZWbIha.png)

変数を入れ替える関数を作るが、値渡しであるため実行しても入れ替えられていないというオチ。手を動かしてほしかったためswap関数の処理内容は考えてもらいました。
ここで値渡しと参照渡しの違いを説明し、ネタばらしをしたのですがすんなりと理解というわけにはいきませんでした・・・。繰り返し説明しどこがわからないか質問を受けたりもしましたが難しかったです。
説明を終えたところでポインタを使えば直接参照できるというヒントを与えてアクティブラーニング形式でコードを修正。ここでも説明に苦戦しましたが、なんとか理解してもらえたかな？ということころで終了。

### ハンズオンを終えて
やはり前提知識なしでいきなりポインタまで教えるのは難しかったです。しかし多少内容を省略しても混乱しないように心掛けたのは良かったかなと思います。今後も何か自分にできることを見つけ、サークル活動をしていきたいと思います。

