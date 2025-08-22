---
title: "RC-S300をRaspberryPiを使用する"
emoji: "💳"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["RCS300", "FeliCa", "PaSoRi", "SONY", "raspberrypi"]
published: true
---

# はじめに
とある興味からラズパイとFeliCaで遊びたいと考え、勢いで非接触型ICカードリーダー/ライターであるPaSoRi **RC-S300**を購入しました。「最新版買っとけば問題ないでしょ！」と適当にポチったはいいものの、到着を待っている間に
- 超有名ライブラリであるnfcpyはRC-S300に非対応
- RC-S300はスペック上ではLinuxに非対応

という衝撃の事実を知ることとなりました。大学生にとって、3,000円弱とはそう簡単に諦められる金額ではなかったため、色々と作業を続け、なんとか動かすことにしました。「RC-S300をラズパイで使いたいのに...」と困っている人がほかにも存在すると信じ、記事を執筆しております。

<br>

![RC-S300](/images/rc-s300-raspi/rcs300.jpg =400x)
*RC-S300（SONY公式ページより引用）*

# 環境
- Raspberry Pi 3 Model B
  - OSは Raspberry Pi OS Lite 64-bit（バージョン：Bookworm）
  - 2025年8月現在、最新のOS
- PaSoRi RC-S300
  - 詳細は[公式ホームページ](https://www.sony.co.jp/Products/felica/consumer/products/RC-S300.html)をご確認ください

# 準備
ラズパイの導入や初期設定の部分については省略します。もし、分からない...という方がいらっしゃれば、「ラズパイ 初期設定」とググってみてください。

次にラズパイにPC/SCの環境を導入する。PC/SC（Personal Computer / Smart Card）とは異なるメーカ間でも非接触ICカードリーダ/ライタを同一コンピュータで利用するための標準規格のことです。
詳細については詳しくまとめられた記事があったため、参考にしてください。
https://www.orangetags.jp/%E7%94%A8%E8%AA%9E%E9%9B%86/pcsc%E3%81%A8%E3%81%AF

さて、以下のコードをラズパイのターミナルより実行してください。
``` bash
sudo apt install libusb-dev libpcsclite-dev opensc pcscd pcsc-tools
```

何をやっているかを簡単に説明すると、全体としてはPC/SCを利用する上で必要なパッケージを導入しています。それぞれ
- libusb-dev
  - USBデバイスとの通信用ライブラリ
- libpcsclite-dev
  - PC/SC標準APIの実装ライブラリ
- opensc
  - スマートカードを扱うためのツール
- pcscd
  - スマートカードを様々なソフトウェアに繋げるためのデーモン（常駐プログラム）
- pcsc-tools
  - PC/SCの動作確認やデバッグのためのツール

といった感じです。

# 動作確認
ターミナルで
``` bash
pcsc_scan
```
と入力し、PC/SC device scannerを起動しましょう。まず、
``` bash
PC/SC device scanner
V 1.6.2 (c) 2001-2022, Ludovic Rousseau <ludovic.rousseau@free.fr>
Using reader plug'n play mechanism
Scanning present readers...
0: SONY FeliCa RC-S300/P (0303109) 00 00
 
[時刻]
 Reader 0: SONY FeliCa RC-S300/P (0303109) 00 00
  Event number: 2
  Card state: Card removed, 
```
と表示されれば成功です。RC-S300も認識され、カードを検知していない状態であることがわかります。これに対し、任意のカードを載せると、
``` bash
[時刻]
 Reader 0: SONY FeliCa RC-S300/P (0303109) 00 00
  Event number: 3
  Card state: Card inserted, 
  ATR: 3B 8F 80 01 80 4F 0C A0 00 00 03 06 11 00 3B 00 00 00 00 42

ATR: 3B 8F 80 01 80 4F 0C A0 00 00 03 06 11 00 3B 00 00 00 00 42
+ TS = 3B --> Direct Convention
+ T0 = 8F, Y(1): 1000, K: 15 (historical bytes)
  TD(1) = 80 --> Y(i+1) = 1000, Protocol T = 0 
-----
  TD(2) = 01 --> Y(i+1) = 0000, Protocol T = 1 
-----
+ Historical bytes: 80 4F 0C A0 00 00 03 06 11 00 3B 00 00 00 00
  Category indicator byte: 80 (compact TLV data object)
    Tag: 4, len: F (initial access data)
      Initial access data: 0C A0 00 00 03 06 11 00 3B 00 00 00 00
+ TCK = 42 (correct checksum)

Possibly identified card (using /usr/share/pcsc/smartcard_list.txt):
3B 8F 80 01 80 4F 0C A0 00 00 03 06 11 00 3B 00 00 00 00 42
3B 8F 80 01 80 4F 0C A0 00 00 03 06 .. 00 3B 00 00 00 00 ..
	FeliCa (as per PCSC std part3)
3B 8F 80 01 80 4F 0C A0 00 00 03 06 11 00 3B 00 00 00 00 42
	RFID - FeliCa (generic) (as per PCSC std part3)
	Suica public transit card (Japan IC system)
	(also: Hayakaken, ICOCA, Kitaca, manaca, nimoca, PASMO, PiTaPa, SUGOCA, TOICA)
	https://en.wikipedia.org/wiki/Suica
	Octopus, MTR network from Hong Kong, 2014
```
と表示され、無事カードを読み取れたことがわかると同時に、ATR（Answer To Reset）と呼ばれるカードの状態やプロトコルを含んだ情報が得られました。また、カードを取り除くと、
``` bash
[時刻]
 Reader 0: SONY FeliCa RC-S300/P (0303109) 00 00
  Event number: 4
  Card state: Card removed, 
```
と表示され、無事カードの除去も認識できていると確認できました。

# 利用例
無事、PC/SC環境を導入できたので、次はPythonのスクリプトを用いたFeliCaの読み取りを行ってみたいと思います。PythonスクリプトでICカードリーダにアクセスするためのライブラリであるpyscardをインストールしてあげましょう。
```bash
pip install pyscard
```
とすることで簡単にダウンロードできます。pyscardの詳細については[公式のユーザマニュアル](https://pyscard.sourceforge.io/user-guide.html)を参照してください。

サンプルコードとして、ICカードのIDm（FeliCaカードが製造される際にICチップに記録される、書き換え不可能な固有ID）を読み取るPythonコードを作成しました。こちらをSSHなどでラズパイに送信し、実行することでICカードのIDmを得ることができます。
``` Python
from smartcard.System import readers
import time

r = readers() # 使用可能なICカードリーダを取得
print(r) # 使用可能なICカードリーダを表示

while True:
    try:
        connection = r[0].createConnection() # 最初に見つかったICカードリーダに接続するためのオブジェクトを作成
        connection.connect() # 読み取り可能なカードと接続 （カードがセットされていない場合は例外処理）

        send = [0xFF, 0xCA, 0x00, 0x00, 0x00] # IDmを取得するためのAPDU（スマートカードとデータのやりとりを行うための標準規格）コマンド
        recv, sw1, sw2 = connection.transmit(send) # APDUコマンドを送信
        
        print(" ".join(f"{b:02X}" for b in recv)) # APDUコマンドに対する返値を表示
        break
    except:
        time.sleep(1) # カードがセットされていない場合は何もしない

>>> ['SONY FeliCa RC-S300/P (0303109) 00 00']
>>> XX XX XX XX XX XX XX XX （IDmとステータスコードが表示される）
```

基本的な流れについてはコード上にコメントとして記載した通りです。

APDUコマンドについての詳細は以下の記事がわかりやすかったので、参考にしてみてください。
https://ramble.impl.co.jp/2114/

# さいごに
本記事ではRC-S300とRaspberry Piを用いて、ICカードの読み取りを行う方法についてまとめました。「RC-S300はラズパイに非対応」と聞いてから諦めることなく、色々試行錯誤したことが実を結んで、個人的にもホッとしています。ラズパイも記事のライティングも何もかも初心者ですので、誤り等を発見した際はこそっと教えていただけますと幸いです...。

これからは別記事にはなりますが、冒頭軽く触れた「とある興味」にあたる部分について試行錯誤を続けていくことができればと思います。完成次第、リンクも添付しますので、引き続きご確認いただけますと嬉しいです。