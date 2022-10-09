# USB MSDとネットワークブートによるデバッグ

## はじめに

bootcode.binには様々なブートモードのデバッグに利用できるデバッグ出力が
あります。中でもUSBマスストレージとネットワークブートの2つのブート
モードには多くのデバッグ機能があります。

[bootcode.bin](https://github.com/raspberrypi/firmware/files/1466884/bootcode.bin.gz)を
クリックすると最新のデバッグ機能を含む第2ステージの最新ビルドをダウンロード
できます。

## UARTシリアル接続で接続する

これを行うにはAdafruit USBシリアルドングルのようなシンプルな3線式シリアル
ドングルの使用を勧めますが、2台目のRaspberry Piで手軽に行うことできます。
GPIOコネクタの6, 8, 10ピンを使用するだけです。

ホストコンピューターでputty（Windows PCの場合）やscreen（Linuxベースの
コンピュータやMACの場合）を実行します。

### Windows 10

- puttyをインストールします
- "device manager"を検索して開きます
- Device Managerで"Ports (COM and LPT0"セクションを見つけて開くと
  COM5などのシリアルデバイスがリストアップされます
- puttyを開き"Session"をクリックします
- "Serial line"をデバイスマネージャのものと一致するように変更します
  （私の場合はCOM5でした）
- 速度を115200に変更します
- 必要に応じて"Serial"ツリーノードにあるその他の設定を変更します。特に、
  ハードウェアフローコントロールを無効にする必要があるでしょう
- そして、'open'をクリックします

### Linux (Debian)

- シリアルドングルをLinuxコンピュータに接続します
- `ls /dev/tty*`とタイプすると`/dev/ttyUSB0`のようなものが見つかるはずです
- `screen /dev/ttyUSB0 115200`とタイプするとシリアルに接続します

### MAC

-  MACにシリアルドングルを接続します。
- `ls /dev/tty*`とタイプすると`/dev/tty.usbserial-FTHIH75J`のようなものが
  見つかるはずです。
- `screen /dev/tty.usbserial-FTHIH75J 115200`とタイプするとシリアルに
  接続します

## 共通 - USBシリアルをテストする

標準的なRaspbianビルドを使用してデバッグするRaspberry Piデバイスを起動
します。ログインしてターミナルを開き (Ctrl-Alt-T)、`sudo raspi-config`と
タイプしてraspi-configアプリケーションを開きます。

`Interfacing options`と`serial`とたどり、シリアルログインシェルを有効に
します。そして、`sudo reboot`とタイプします。

Piが再起動すると8番ピンから115200ボーでシリアルが出力されるはずです
（6番はグランド）。正しく設定されていることを確認してから次に進みましょう。

## デバッグの有効化

bootcode.binを使用するには2通りの方法があり、デバッグの方法にも2通りの
方法があります。2つの方法の特徴は次のとおりです。

- ソース（SDカード、USB MSDまたはイーサネット経由）からbootcode.binを
  読み込み、それから、同じソースからstart.elfとkernel.imgを読み込みます
- SDカードからbootcode.binを読み込み（特に、config.txtとstart.elfがSD
  カードにふくまれていない状態で）、次に、USB MSDやイーサネットから
  起動して他のファイル（start.elf, config.txt, kernel.imgなど）を探し
  ます

ここで(2)の方法を使う理由は、標準ブートモードに問題がある時にそれ以上
進むためにデバッグが必要があるからです (すなわち、USB MSDやネットワーク
から読み込めないのは何かをデバッグする)。

上記の(1)の方法でデバッグ出力を有効にするには、config.txtに
`uart_2ndstage=1`を追加する必要があります。

上記の(2)の方法でデバッグ出力を有効にするには、bootcode.binファイルの
他に`UART`という名前のファイルをファイルシステムに追加する必要があります。

以上を行うとブートフェーズでブート出力が表示されるはずです。この出力は
次で始まるでしょう。

```
Raspberry Pi Bootcode
```
