---
title: "ソフトウェアTNC を Windows 11 に導入する"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["通信", "衛星通信", "ソフトウェアTNC", "CubeSat"]
published: true
---

## はじめに
アマチュア無線帯で通信する多くの人工衛星はAX.25（アマチュア無線に特化した通信プロトコル）を使用します。そこではアマチュア無線で送受信するFM電波の音声データとデジタルデータの変換をTNC（terminal node controller）で行います。そして、最近ではPCのソフトウェア上で機能するソフトウェアTNCがよく使用されています。また、完全に筆者調べにはなりますが、日本の大学衛星の地上局ではよく **SoundModem** と **AGW OnlineKISS** を組み合わせて使用するケースが多いように感じます。

私も例に漏れず、その二つを併せて使おうと思ったのですが、使うことができるようになるまでに若干のトラブルが存在したので、備忘録がてらの記事を執筆しておきます。

## SoundModem のダウンロード
以下のリンクの公式ホームページから直接ダウンロードすることができます。

https://uz7.ho.ua/packetradio.htm

`soundmodem114.zip`というリンクを踏むことでダウンロードできます。ダウンロード後は解凍し、ドキュメントなどの分かりやすい位置に配置してあげてください。

筆者が試したところ、Google Chrome ではリンクを踏んでも一向にダウンロードできなかったため、同様の症状に遭遇した場合は Microsoft Edge などの別のブラウザを使用してみてください。

## AGW OnlineKISS のダウンロード
### 必要なランタイムのダウンロード
AGW OnlineKISS を使用する上で、Visual Basic 6.0 のランタイムが必要となる。Microsoft 公式からの配布は確認できなかったため、筆者は Vector よりインストーラを入手した。

https://www.vector.co.jp/soft/win95/util/se342080.html

インストーラを使用して、ダウンロードすることで環境がぐちゃぐちゃになることが怖かったので、インストーラを解凍して手動でインストールすることとした。以下の手順を踏むことでインストールが可能である。

1. `VB6RTSP6Maximum.exe` を 7-Zip などで解凍する
2. `Setup.exe` を同様に解凍する
3. 内部に存在する以下のファイルを、`C:\Windows\SysWOW64` へコピーする
    ```
     ・ MSCOMM32.OCX
     ・ COMDLG32.OCX
     ・ RICHTX32.OCX
     ・ MSWINSCK.OCX
    ```
4. コマンドプロンプトを管理者権限で開き、以下のコマンドを実行する
    ```
    cd C:\Windows\SysWOW64
    regsvr32 MSCOMM32.OCX
    regsvr32 COMDLG32.OCX
    regsvr32 RICHTX32.OCX
    regsvr32 MSWINSCK.OCX
    ```

### AGW OnlineKISS のインストール
以下のリンクの公式ホームページから直接ダウンロードすることができます。

https://www.satblog.info/software/

`AGW OnlineKISS plus`というリンクを踏むことでダウンロードできます。（`AGW`とかで検索すると簡単に見つけることが出来ます。）ダウンロード後は解凍し、ドキュメントなどの分かりやすい位置に配置してあげてください。

### ini ファイルの設定
`agw_online_kiss` フォルダの中に、`agw_onlinekiss.ini` というファイルが存在します。これを適切に書き換えてあげる必要があります。また、同時に`agw_online_kiss_plus.exe` と同じ階層に `logfiles` というフォルダを作成してください。

```diff
- FILE_PATH = D:\Amateurfunk
+ FILE_PATH = ./logfiles

- CALLSIGN = DK3WN
+ CALLSIGN = <自身のコールサイン>

- QTH_LAT = 49.73145
- QTH_LONG = 8.95564
- QTH_HEIGHT = 0.28
+ QTH_LAT = <自身の地上局の緯度>
+ QTH_LONG = <自身の地上局の経度>
+ QTH_HEIGHT = <自身の地上局の高さ（km）>

- TLEFILE = D:\Amateurfunk\Orbitron\TLE\amateur.txt
+ TLEFILE = ./TLE.txt
```

緯度・経度は Google Maps 上で右クリックすることで調べることができます。

### TLE ファイルの設定
AGW OnlineKiss において、TLEが必要になる場面がいまいちわからなかったのでひとまず筆者はISSのTLEを追加しました。

まず、`agw_online_kiss_plus.exe` と同じ階層に、`TLE.txt` を作成し、以下の内容をコピーしてください。もし、AGW OnlineKiss の機能をフルに活かしたい場合は自身で対象の衛星の最新のTLEを調べて、コピペしてください。この際、`(` などの記号が `TLE.txt` に含まれているとエラーが発生してしまうため、含めないようにしましょう。

```
ISS
1 25544U 98067A   25268.88924235  .00024001  00000-0  42185-3 0  9994
2 25544  51.6319 167.9312 0003311   8.6325 351.4720 15.50358116530788
```

おそらく、対象となる衛星のTLEを複数登録することで衛星ごとにログファイルを分類することができるので、AGW OnlineKISS　で衛星追跡を行わない人でもインデックスの登録は済ませておくと良いでしょう。

## さいごに
以上の手順を進めると、無事に AGW OnlineKISS を開くことができるようになったかと思います。肝心のソフトウェアTNCの使い方に関しては筆者も今から学ぶところですので、後々記事にしたいなあと考えております...。