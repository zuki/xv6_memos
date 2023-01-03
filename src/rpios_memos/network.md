# xv6-netとCircleのnet部分を調査

## TCP/IPを実現しているコード

1はxv6-net, 2はCircle

- 基幹
	1. net.c
	2. net.cpp, netconfig.cpp
- 物理層
	1. e1000.c
	2. netdevlayer.cpp, lan7800.cpp, usbcdcethernet.cpp
- リンク層
	1. ethernet.c, arp.c
	2. linklayer.cpp, arphandler.cpp,
- インターネット層
	1. ip.c, icmp.c
	2. networklayer.cpp, icmphandler.cpp
- トランスポート層
	1. udp.c, tcp.c
	2. transportlayer.cpp, udpconnection.cpp, tcpconnection.cpp
- アプリケーション層
	1. echo.c, ifconfig.c, ifdown.c, ifget.c, ifup.c, udpechoserver.c, tcpechoserver.c
	2. dhcpclient.cpp, dnsclientcpp, httpclient.cp, httpdaemon.cpp, ntpclient.cpp, ntpdaemon.cpp, tftpdaemon.cpp
- ライブラリ
	1. socket.c
	2. netsocket.cpp, socket.cpp, netconnection.cpp

## 構造体

```
## 基幹

struct netdev_ops {
    int (*open)(struct netdev *dev);
    int (*stop)(struct netdev *dev);
    int (*xmit)(struct netdev *dev, uint16_t type, const uint8_t *packet, size_t size, const void *dst);
};

struct netdev {
    struct netdev *next;
    struct netif *ifs;
    int index;
    char name[IFNAMSIZ];
    uint16_t type;
    uint16_t mtu;
    uint16_t flags;
    uint16_t hlen;
    uint16_t alen;
    uint8_t addr[16];
    uint8_t peer[16];
    uint8_t broadcast[16];
    struct netdev_ops *ops;
    void *priv;
};

## 物理層

struct tx_desc
{
  uint64_t addr;
  uint16_t length;
  uint8_t cso;
  uint8_t cmd;
  uint8_t status;
  uint8_t css;
  uint16_t special;
};

struct rx_desc
{
  uint64_t addr;       /* Address of the descriptor's data buffer */
  uint16_t length;     /* Length of data DMAed into data buffer */
  uint16_t csum;       /* Packet checksum */
  uint8_t status;      /* Descriptor status */
  uint8_t errors;      /* Descriptor Errors */
  uint16_t special;
};

## リンク層

struct netproto {
    struct netproto *next;
    uint16_t type;			// NETPROTO_TYPE_IP, NETPROTO_TYPE_ARP, NETPROTO_TYPE_IPV6
    void (*handler)(uint8_t *packet, size_t plen, struct netdev *dev);
};

## ネットワーク層

struct netif {
    struct netif *next;
    uint8_t family;			// NETIF_FAMILY_IPV4, NETIF_FAMILY_IPV4
    struct netdev *dev;
    /* Depends on implementation of protocols. */
};

struct ip_route {
    uint8_t used;
    ip_addr_t network;
    ip_addr_t netmask;
    ip_addr_t nexthop;
    struct netif *netif;
};

struct ip_protocol {
    struct ip_protocol *next;
    uint8_t type;
    void (*handler)(uint8_t *payload, size_t len, ip_addr_t *src, ip_addr_t *dst, struct netif *netif);
};
```

## 送受信信処理 (xv6-net)

### TCP送信

- syscall.c#sys_sendto(int sockfd, const void *buf, size_t len, int flags,
             const struct sockaddr *dest_addr, socklen_t addrlen);
  - socket.c#socketwrite(struct socket *s, char *addr, int n);
    - tcp.c#tcp_api_send(int soc, uint8_t *buf, size_t len);
      - tcp.c#tcp_tx(struct tcp_cb *cb, uint32_t seq, uint32_t ack,
               uint8_t flg, uint8_t *buf, size_t len);
        - ip.c#ip_tx(struct netif *netif, uint8_t protocol, const uint8_t *buf,
                size_t len, const ip_addr_t *dst);
        - ip.c#ip_tx_core (struct netif *netif, uint8_t protocol, const uint8_t *buf,
                      size_t len, const ip_addr_t *src, const ip_addr_t *dst,
                      const ip_addr_t *nexthop, uint16_t id, uint16_t offset);
        - ip.c#ip_tx_netdev(struct netif *netif, uint8_t *packet, size_t plen, const ip_addr_t *dst);
        	- e1000.c#e1000_tx(struct netdev *dev, uint16_t type, const uint8_t *packet,
        	                   size_t len, const void *dst);
        	  - ethernet.c#ethernet_tx_helper(struct netdev *dev, uint16_t type, const uint8_t *payload,
        	                                  size_t plen, const void *dst, ssize_t (*e1000_tx_cb)(struct netdev*,
        	                                  uint8_t*, size_t));
        	    - e1000.c#e1000_tx_cb(struct netdev *netdev, uint8_t *data, size_t len);

### TCP受信

#### 初期化と登録

1. E1000がデータを受信し、割り込みハンドラが呼び出される
2. e1000intrl(): e1000_rx()をcall
3. e1000_rx(): ethernetヘッダーを処理してnetdev_receive()経由でipハンドラ(ip_rx）をcall
4. ip_rx(): ipヘッダーを処理して、tcpハンドラ(tcp_rx)をcall
5. tcp_rx(): tcpヘッダーを処理してtcpイベント処理を行う

- tcp.c
  - tcp_init()
    - ip_add_protocol(IP_PROTOCOL_TCP, tcp_rx);
- ip.c
  - ip_init()
    - netproto_register(NETPROTO_TYPE_IP, ip_rx);
  - ip_rx()
    - protocol->handler(payload, plen, src, dst, iface): call tcp_rx
  - ip_add_protocol(type, (*handler)(payload, len, src, dst, netif));
- net.c
  - netdev_receive()
    - protocol->handler(packet, plen, dev): call ip_rx
  - netproto_register(type, (*handler)(packet, plen, dev))
- e1000intr();
  - e1000_rx(struct e1000 *dev);
    - データを受信してdesc(addr, length)にセット
    - ethernet_rx_helper(netdev, addr, length, netdev_receive): -> 上位層にデータを渡す

#### 処理

- syscall.c#sys_recvfrom(int sockfd, void *buf, size_t len, int flags,
                         struct sockaddr *src_addr, socklen_t *addrlen);
  - socket.c#socketread(struct socket *s, char *addr, int n);
    - tcp.c#tcp_api_recv(int soc, uint8_t *buf, size_t size): データの到着を待つ
      - e1000intr()から順にデータが設定されていく

## Circelの送受員処理

- socket
  - Receive()
    - transportLayer#Receive

- transportlayer
  - Process()
    - networklayer#Receive(): tcp/udpパケットを取得
      - netconnection#PacketReceive()
  - Receive()
    - netconnection#Receive(): データを取得

- netconnection
  - PacketReceive(): tcp/udpデータ処理
    - RxQueue.Enqueue: 受信データをエンキュー
  - Receive()
    - RxQueue.Dequeue: トランスポート層へ（データ）

- networaklayer
  - Process()
    - linklayer#Receive()
      - RxQueue.Enqueue(): IPパケットから(TCP／UDP／ICMP)データとパラメタをエンキュー
  - Receive()
    - RxQueue.Dequeue(): -> トランスポート層へ(TCP／UDP／ICMPデータとパラメタ)

- linklayer
  - Process()
    - netdevlayer#Recieve()
      - IPRxQueue.Enqueu(): EthernetパケットからIPパケットを取り出してエンキュー
  - Receive()
    - IPRxQueue.Dequeu(): -> ネットワーク層へ(IPパケット)

- netdevlayer
  - Process()
    - device#ReceiveFrame()
      - RxQueue.Enqueu(): Ethernetパケットをそのままエンキュー
  - Receive()
    - RxQueuee.Dequeue(): -> リンク層へ（Ethernetパケット）

- netdevice
  - lan7800#ReceiveFrame(void *pBuffer, unsigned *pResultLength)
    - pBufferにEthernetパケットをセット（ここでUSB Bulk inによるデータの取得を行う）

### Circle受信処理の呼び出し

```
CNetTask::Run() {
  while(1) {
    netsubsystem#Process();
    scheduler#Yield();
  }
}

CNetSubSystem::Process() {
  netdevlayer.Process();      // ethernetパケット取得
  linklayer.Process();        // ipパケットを取り出す
  networklayer.Process();     // tcp/udpパケットを取り出す
  transportlayer.Process();   // データを取り出す
}
