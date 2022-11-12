# Linuxの`drivers/net/usb/Kconfig`

```
comment "USBネットワークアダプタのサポートにはホスト側のUSBサポートが必要である"
    depends on !USB && NET

menuconfig USB_NET_DRIVERS
    tristate "USB Network Adapters"
    default USB if USB
    depends on USB && NET

if USB_NET_DRIVERS

config USB_LAN78XX
    tristate "Microchip LAN78XX Based USB Ethernet Adapters"
    select MII
    select PHYLIB
    select MICROCHIP_PHY
    select FIXED_PHY
    select CRC32
    help
      このオプションはMicrochip LAN78XXベースのUSB 2 & USB 3
      10/100/1000 Ethernet アダプタをサポートする。
      LAN7800 : USB 3 to 10/100/1000 Ethernet adapter
      LAN7850 : USB 2 to 10/100/1000 Ethernet adapter
      LAN7801 : USB 3 to 10/100/1000 Ethernet adapter (MAC only)

      LAN7801には適切なPHYドライバが必要.

      このドライバをモジュールとしてコンパイルするにはここでMを
      選択する: モジュール名はlan78xxとなる。

config USB_USBNET
    tristate "Multi-purpose USB Networking Framework"
    select MII
    help
      このドライバは効率的な転送のための深いキューをサポートする
      共通のネットワークドライバコアの周りに構築される"minidrivers"
      （これは小さなパケットと高速により、より良いパフォーマンスを
      もたらす）を備えることでUSB上で複数のネットワークリンクを
      サポートする。

      USBホストは "usbnet"を実行し、リンクの他方の端は以下のいずれか
      ある。

      - 別のUSBホスト: USBの"ネットワーク"、または"データ転送"ケーブルを
      使用する場合。これらは、"Laplink"パラレルケーブルやいくつかの
      マザーボードのようにラップトップをPCにネットワークするために
      よく使用される。これらは、多くのサプライヤーから提供されている
      専用チップに依存している。

      - インテリジェントなUSBガジェット: おそらくinuxシステムを組み
      込んでいる。これには、Linuxが動いているPDA (iPaq、Yopy、
      Zaurusなど) や、標準のCDC-Ethernet仕様を使って相互運用する
      デバイス (多くのケーブルモデムを含む) が含まれる。

      - ネットワークアダプタハードウェア（10/100イーサネット用のもの
      など）: このドライバフレームワークを使用する。

      リンクは、リンクが2ノードリンクの場合は "usb0"、ほとんどの
      CDC-Ethernetデバイスの場合は"eth0"という名前で表示される。
      これらの2ノードリンクはルーティングではなくイーサネットブリッジ
      (CONFIG_BRIDGE)を使うとより簡単に管理することができる。

      詳しくは<http://www.linux-usb.org/usbnet/>を参照されたい。

      このドライバをモジュールとしてコンパイルするには、ここでMを
      選択する: モジュール名は"usbnet"となる。

config USB_NET_CDCETHER
    tristate "CDC Ethernet support (smart devices such as cable modems)"
    depends on USB_USBNET
    default y
    help
      このオプションは、デバイスのファームウェアに簡単に実装できる
      CDC（Communication Device Class）イーサネットコントロール
      モデル仕様に準拠したデバイスをサポートする。CDCの仕様は
      <http://www.usb.org/>から入手可能である。

      CDC EthernetはUSB接続をサポートするDOCSISケーブルモデム用の
      実装オプションであり、非Microsoft USBホストに使用される。
      Linux-USB CDC Ethernet Gadgetドライバはオープン実装である。
      このドライバは、少なくとも以下のデバイスで動作するはずである。

      * Dell Wireless 5530 HSPA
      * Ericsson PipeRider (all variants)
      * Ericsson Mobile Broadband Module (all variants)
      * Motorola (DM100 and SB4100)
      * Broadcom Cable Modem (reference design)
      * Toshiba (PCX1100U and F3507g/F3607gw)
      * ...

      このドライバは "ethX"という名前のインターフェイスを作成する。
      ここでXは他にどのようなネットワークデバイスを使用しているかに
      依存する。ただし、IEEE 802の"local assignment"ビットがアドレスに
      設定されている場合は"usbX"という名前が使用される。

config USB_NET_CDC_SUBSET_ENABLE
    tristate
    depends on USB_NET_CDC_SUBSET

config USB_NET_CDC_SUBSET
    tristate "Simple USB Network Links (CDC Ethernet subset)"
    depends on USB_USBNET
    default y
    help
      このドライバーモジュールはデバイス固有の情報がなくても動作可能な
      USBネットワークデバイスをサポートする。このようなドライバを使用
      する場合は選択する。

      多くのUSBホスト-ホストケーブルはこのモードで動作するが、Win32
      システムと通信できなかったり、より一般的には特定のイベント
      （他方の側でホストを再接続するなど）をうまく処理できない場合が
      あることに注意が必要である。また、一般にこれらのデバイスは
      恒久的な割り当てイーサネットアドレスを持たない。
```
