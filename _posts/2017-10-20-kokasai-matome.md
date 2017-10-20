---
title: 紅華祭 2017を終えて
tags: kokasai 2017 festival
categories: poem
---

LinuxClubの1年のKoyamaです。

今後、このブログを通じてサークルの活動の様子を積極的に発信していきたいと思います。この記事では2017年度 紅華祭を振り返ってみたいと思います。まず、部長から全体を通したコメントです。

## 全体を通じて

前日に完成している展示物がなくて焦ったものですがみなさんいいものを作ってきてくれて驚きました。普段コンソールに向かっているので見栄えのいいものと言われても困ったとは思いますが良い作品を展示できてよかったです。

## 個人制作

1,2年生を中心に個人作品を制作しました。`作品名 [作者]`という形式で見出しをつけています。

### 1. 空飛ぶ絨毯 [Koyama]

Webカメラで取得した映像とあらかじめ用意した動画をクロマキー合成するプログラムを作成し、展示しました。使用した言語は`Python`でライブラリは`OpenCV`を使用しました。以下の記事を参考にしながらコードを書きました。

- [Python OpenCV3で画像の画素値を二値化して出力 \| from umentu import stupid](https://www.blog.umentu.work/python-opencv3%E3%81%A7%E7%94%BB%E5%83%8F%E3%81%AE%E7%94%BB%E7%B4%A0%E5%80%A4%E3%82%92%E4%BA%8C%E5%80%A4%E5%8C%96%E3%81%97%E3%81%A6%E5%87%BA%E5%8A%9B/)
- [RPi + Python + OpenCV その３「クロマキー」 \| OpenCV \| kosakalab](http://make.kosakalab.com/rpi/opencv_3/)
- [Getting Started with Videos — OpenCV-Python Tutorials 1 documentation](http://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_gui/py_video_display/py_video_display.html)

ソースコードは以下です。

<script src="https://gist.github.com/tomoyk/698c6294637b16b140da404854e37389.js"></script>

途中でOpenCVのエラーで終了してしまうのでシェル芸で回避しました。パラメータを変えることで背景の動画を変更できる機能も実装してうまく連携させました。

	while true
	do
	  for foo in {1..3}
	  do
	    python3 origin.py $foo
	  done
	done

実際に動かすと以下のようになります。

<img src="https://pbs.twimg.com/media/DLlmyHjUIAAnZ7c.jpg:large" width="500">

<img src="/images/kokasai2017/koyama_screenshot.jpg" width="300">

### 2. 移植版ドアセンサー [Koyama]

サークルの部室にドアが開くと音が鳴る謎のシステム(Raspi + スピーカ)を作りました。その移植版をArduino UNOとProcessingを使って作成しました。センサーは秋月電子で販売されているドア用センサーを使いました。

[ケース入りリードスイッチ（磁石付セット）ＭＣ－１４ＡＧ: センサ一般 秋月電子通商 電子部品 ネット通販](http://akizukidenshi.com/catalog/g/gP-04025/)

Arduinoには次の回路を実装しました。

<img src="https://raw.githubusercontent.com/tomoyk/kokasai17/master/proc_testVisualizer/images/architecture.jpg" width="500">

次にArduinoにArduinoへ以下のプログラムを書き込みます。このプログラムではdigital 7番ピンに接続されたセンサーの値を取得してシリアルに書き出す処理を行っています。`delay(100)`によって遅延処理を行っています。

	#define SENSOR_PIN 7

	void setup(){
	  Serial.begin(9600);
	}

	void loop(){
	  int val = digitalRead(SENSOR_PIN);
	  Serial.write(val);
	  delay(100);
	}

Arduinoにプログラムが書き込めているかシリアルモニタを使って確認します。

次にProcessingを起動します。そして、以下のプログラムを入力します。

<script src="https://gist.github.com/tomoyk/f67947c7f7913bde2ec0db272487e5e8.js"></script>

プログラムの概要について説明します。

- プログラム先頭の`import processing.sound.*;`ではあらかじめインストールしたサウンドを再生するライブラリを読み込んでいます。
- その後にある`boolean`や`PImage`や`Soundfile`ではセンサーの状態、画像ファイル、音声ファイルを保持する変数を宣言しています。
- `setup()`ではシリアル、画像、音声の初期化を行っています。
- `setBack()`では背景を設定しています。
- `draw()`ではセンサーの値に応じてドアが開いているか、閉じているかを判断しています。また、その状況に応じて画像や音声を切り替えています。

次に画像ファイルと音声ファイルを用意します。

画像はいらすとやから入手しました。

[いろいろな状態のドアのイラスト \| かわいいフリー素材集 いらすとや](http://www.irasutoya.com/2016/07/blog-post_832.html)

ダウンロードした画像ファイルのドアが揃うようにGIMPで調整しました。そして、ファイル名を`door_close.png`と`door_open.png`にリネームします。

音声は音声合成システム**Open JTalk**を使用して作成しました。今回は一時的な使用だった為、Dockerを使いました。

[yamamotofebc/open_jtalk - Docker Hub](https://hub.docker.com/r/yamamotofebc/open_jtalk/)

まず、イメージを入手します。

`docker pull yamamotofebc/open_jtalk`

次にリファレンスに従って生成したい文字列を設定してコマンドを実行します。単純な読み上げにしか対応していないので`こんにちは`を`こんにちわ`として設定しています。

	echo "こんにちわ" | docker run -i --rm yamamotofebc/open_jtalk > door_hello.wav
	echo "ばいばいきん" | docker run -i --rm yamamotofebc/open_jtalk > door_close.wav

これらの準備が終わったらProcessingのIDEに戻って実行ボタンを押下します。LinuxではIDEをroot権限で起動しないとシリアルデバイスにアクセスできないので注意が必要です。

うまく動作すればドアセンサーが離れたときに画面上にドアが開いている画像が表示され、「こんにちは」という音声が流れます。

<img src="https://raw.githubusercontent.com/tomoyk/kokasai17/master/proc_testVisualizer/images/demoOpen.jpg" width="300">
<img src="https://raw.githubusercontent.com/tomoyk/kokasai17/master/proc_testVisualizer/images/demoClose.jpg" width="300">

### 3. 物体認識 [部長]
DarkNetを使った物体認識ソフトウェアを作りました。
基本的な構造としては`Docker`を使い、`フロントエンド`、`バックエンド`、`静的ファイルの配信サーバー`という形で構築し、AWS上で動作させていました。

※`DarkNet`とはC言語で書かれた機械学習ライブラリです。学習済みデータが公開されているので画像をぶん投げるだけで解析してくれる優れものです。

#### フロントエンド
使用ライブラリ
```
- React
- Recompose
- Material UI
```

フロントエンドではES6に`React`, `Recompose`を使いました。
来場者にアクセスさせて写真をアップロード、解析した写真を表示する機能しか必要なかったのですが無駄にモダンな構成になっています。特筆することはないです。

#### バックエンド
使用ライブラリ
```
- PIL (画像処理)
- falcon (APIサーバー)
```

バックエンドはpythonで書きました。falconでルーティングしてDarkNetに投げるなどの役割を担っています。

PILはiphoneから投げられた画像に対して処理をしています。画像は主にEXIFと呼ばれる情報の格納部分があり、そこにOrientationという画像の向きを保持する情報があります。iphoneで撮影された画像はその部分が常にright-topという横向きの状態で保持されるらしく、PILを使って画像を縦向きに回転して保存する処理をしてます。

PILは
```python
img = Image.open(filename)
exif = img._getexif()
orientation = exif.get(0x112, 1)
img.save(original_filename)

```
とかするとexifをスッととってこれて楽でよかったです。


#### 静的ファイルの配信サーバー
nginxでDarkNetで書き出された画像を配信してるだけです。特筆事項なし。

#### 反省など
色々ばたついてて作り始めるのが遅かったせいで大したものが作れなかったのが心残りです。それとDarkNetをDockerの上に載せていたせいでレスポンスがだいぶ悪くなってしまいました。workerも4つしか動かしておらずお世辞にも十分だったとは言えない感じです。来年はGTX TITANとか買って望みたいです。嘘です。

### 4. OpenCVのチュートリアル [homirun]

Webカメラで撮影された動画をリアルタイムでカスケード分類器を用い顔認識させてみました。
顔を検出してその上に某氏の画像をオーバーレイしました。
PyConの日に作りました。

使用ライブラリ
- OpenCV
- numpy
- pyplot

<img src="/images/kokasai2017/homirun_screenshot.png" width="500">

<script src="https://gist.github.com/homirun/035f39415f9d976f9da86ffe2632519f.js"></script>

### 5. 声でツイートできるプログラム

　音声認識で録音した音声をテキストに起こして、ツイッターにポストするプログラムです。言語はPythonで、音声認識はGoogle Cloud APIを使いました。

ソースコードは以下です。

#### 全体の実行スクリプト

<script src="https://gist.github.com/kumapana/1f492a0cd948896eb090b28ac14c32a1.js"></script>

#### Google APIを叩くソース

<script src="https://gist.github.com/kumapana/1f492a0cd948896eb090b28ac14c32a1.js"></script>

#### ツイッターに投げるソース

<script src="https://gist.github.com/kumapana/be3eb1e828384f427d00349dfe387117.js"></script>

#### ハマった(?)点

Google Cloud APIから吐かれるjsonファイルの取り扱いで1日ほどハマってました。(最終的にK氏に解決していただいた)

当初の予定では、京都大学 河原研究室が開発したJuliusという音声認識システムを使う予定でしたが、認識精度が低かったため、Google APIに投げることにしました。

### 6. CPU use rate [lapua]

CPU使用率をリアルタイムで表示するGUIアプリ。16コアまで対応。使用ライブラリはQt5.0.0
初めてのクラス設計に苦労しました。

<img src="/images/kokasai2017/lapua_screenshot.png" width="500">

### 7. 数当てゲーム [ayu]

コンソールから入力された数字に対して当ってるだの当ってないだの評価して返す簡単なゲーム．Javaで書きました．GUI化させていきたい．

<img src="/images/kokasai2017/ayu_screenshot.png" width="500">

### 8. 目覚まし [uwdd]

今回初めて~~まともな~~プログラミング作品を作りました。朝起きれなくて遅刻が増えてきたので今回は目覚まし時計を作りました。まだいくつか不具合があったりデザインが簡素すぎたりするので暇があれば直していきたい。

<img src="/images/kokasai2017/uwdd_screenshot.png" width="500">

## 毎年恒例?のLC OB会

LinuxClubの歴史は長いらしく、古くはLinux研究所という名前だったそうです。

学祭で社会人OBが集まるということが恒例になっているようです。

## 2日目から始まったライブ中継

2日目から先輩のカメラでライブ中継を行いました。

<iframe width="560" height="315" src="https://www.youtube.com/embed/KyDryF34rFQ?rel=0" frameborder="0" allowfullscreen></iframe>

## 大学からの取材

大学の広報から取材を受けました。何かしらの映像で公開されるかと思われます。おそらく、紹介動画かと思われます。

[学園祭（紅華祭・かまた祭）[2017年] \| 学生生活 \| 東京工科大学](http://www.teu.ac.jp/student/006532.html)

## 来年に向けて

展示品の数が少なかったので来年はもっと数を増やすべきだと思いました。来場された方が見て分かるよう説明を加えるなど「何を、どのように実現しているか」を明確にする必要があると感じました。

また、来場された方から「今年は冊子を作ってないんですか？」と聞かれることがあったので来年は部誌のような冊子を作成したいと思います。

**来場して頂きありがとうございました。ぜひ来年もお越しください!!**

