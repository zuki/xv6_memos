# ブロックサイズを4096Bに変更

- MBRはbootパーティション(bnoは0始まり)


```
dd if=/dev/zero of=obj/boot.img seek=131071 bs=512 count=1 // 64MB確保
1+0 records in
1+0 records out
512 bytes copied, 8.0777e-05 s, 6.3 MB/s
# -F 32 specify FAT32
# -s 1 specify one sector per cluster so that we can create a smaller one
mkfs.vfat -F 32 -s 1 obj/boot.img
mkfs.fat 4.1 (2017-01-24)

./obj/mkfs obj/fs.img obj/user/bin/mkfs obj/user/bin/init obj/user/bin/cat obj/user/bin/ls obj/user/bin/sh
nmeta 43 (boot, super, log blocks 30, inode blocks 7, bitmap blocks 4) blocks 99957 total 100000

balloc: first 101 blocks have been allocated
balloc: write bitmap block at sector 39

dd if=/dev/zero of=obj/sd.img seek=2097151 bs=512 count=1   // 1024MB確保
1+0 records in
1+0 records out
512 bytes copied, 8.7965e-05 s, 5.8 MB/s
Checking that no-one is using this disk right now ... OK

Disk obj/sd.img: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Created a new DOS disklabel with disk identifier 0x52d667df.
obj/sd.img1: Created a new partition 1 of type 'W95 FAT32 (LBA)' and of size 64 MiB.
obj/sd.img2: Created a new partition 2 of type 'Linux' and of size 959 MiB.
obj/sd.img3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x52d667df

Device      Boot  Start     End Sectors  Size Id Type
obj/sd.img1        2048  133119  131072   64M  c W95 FAT32 (LBA)    // start: 0x800
obj/sd.img2      133120 2097151 1964032  959M 83 Linux              // start: 0x20800
```

## `MBR`

```
P1: 0020 2100 0c49 0108 0008 0000 0000 0200: LBA=0x800,   SECTORS=0x20000 =  64MB
P2: 0049 0208 8351 0110 0008 0200 00f8 1d00: LBA=0x20800, SECTORS=0x1df800 = 959MB
```
# `make qemu`

```
$ make qemu
qemu-system-aarch64 -M raspi3 -nographic -serial null -serial mon:stdio -drive file=obj/sd.img,if=sd,format=raw -kernel obj/kernel8.img
[CPU0] irq_init: success.
- mbox write: 0x7ef08
- mbox_read: 0x7ef08
- clock rate 50000000
- SD base clock rate from mailbox: 50000000.
- sdInit: reset the card
- Divisor selected = 104, pow 2 shift count = 6
- EMMC: Set clock, status 0x1ff0000 CONTROL1: 0xe6807
- Send IX_GO_IDLE_STATE command
- sdSendCommandA response: 0
- EMMC: Sending ACMD41 SEND_OP_COND status 1ff0000
- Divisor selected = 2, pow 2 shift count = 0
- EMMC: Set clock, status 0x1ff0000 CONTROL1: 0xe0207
- EMMC: SD Card Type 2 SC 1024Mb UHS-I 0 mfr 170 'XY:QEMU!' r0.1 2/2006, #deadbeef RCA 4567

sd_init: Partition 1: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
- Status: 0
- CHS address of first absolute sector: header=5d, sector=0, cylinder=0
- Partition type: 0
- CHS address of last absolute sector: header=5d, sector=0, cylinder=0
- LBA of first absolute sector: 0x0
- Number of sectors: 0

sd_init: Partition 2: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
- Status: 0
- CHS address of first absolute sector: header=5d, sector=0, cylinder=0
- Partition type: 0
- CHS address of last absolute sector: header=5d, sector=0, cylinder=0
- LBA of first absolute sector: 0x0
- Number of sectors: 0
sd_init: Boot signature: 0 0
kern/sd.c:571: assertion failed.
 MBR is not valid
kern/console.c:250: kernel panic at cpu 0.
```

# `sd.img`

```
[ boot block | sb block | log | inode blocks | free bit map | data blocks ]
[          1 |        1 |  30 |            7 |            4 |        99957]
```

## Super block (0x4101000, fs.img: 0x1000)

```
04101000: a086 0100 7586 0100 b801 0000 1e00 0000  ....u...........
04101010: 0200 0000 2000 0000 2700 0000 0000 0000  .... ...'.......
04101020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04101030: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

## Log block (0x4102000, fs.img: 0x2000, len: 0x1E000)

```
04102000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04102010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04102020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04102030: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

## Inode block (0x3120000, fs.img: 0x20000, len: 0x7000)

```
inum = 0
04120000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04120010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04120020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04120030: 0000 0000 0000 0000 0000 0000 0000 0000  ................

inum = 1: '/'
04120040: 0100 0000 0000 0100 0010 0000 2b00 0000  ............+...
04120050: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04120060: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04120070: 0000 0000 0000 0000 0000 0000 0000 0000  ................

inum = 2: 'mkfs'
04120080: 0200 0000 0000 0100 b8ba 0000 2c00 0000  ............,...
04120090: 2d00 0000 2e00 0000 2f00 0000 3000 0000  -......./...0...
041200a0: 3100 0000 3200 0000 3300 0000 3400 0000  1...2...3...4...
041200b0: 3500 0000 3600 0000 3700 0000 0000 0000  5...6...7.......

inum = 3: 'init'
041200c0: 0200 0000 0000 0100 2863 0000 3800 0000  ........(c..8...
041200d0: 3900 0000 3a00 0000 3b00 0000 3c00 0000  9...:...;...<...
041200e0: 3d00 0000 3e00 0000 0000 0000 0000 0000  =...>...........
041200f0: 0000 0000 0000 0000 0000 0000 0000 0000  ................

inum = 4: 'cat'
04120100: 0200 0000 0000 0100 589e 0000 3f00 0000  ........X...?...
04120110: 4000 0000 4100 0000 4200 0000 4300 0000  @...A...B...C...
04120120: 4400 0000 4500 0000 4600 0000 4700 0000  D...E...F...G...
04120130: 4800 0000 0000 0000 0000 0000 0000 0000  H...............

inum = 5: 'ls'
04120140: 0200 0000 0000 0100 08b4 0000 4900 0000  ............I...
04120150: 4a00 0000 4b00 0000 4c00 0000 4d00 0000  J...K...L...M...
04120160: 4e00 0000 4f00 0000 5000 0000 5100 0000  N...O...P...Q...
04120170: 5200 0000 5300 0000 5400 0000 0000 0000  R...S...T.......

inum = 6: 'sh'
04120180: 0200 0000 0000 0100 80e7 0000 5500 0000  ............U...
04120190: 5600 0000 5700 0000 5800 0000 5900 0000  V...W...X...Y...
041201a0: 5a00 0000 5b00 0000 5c00 0000 5d00 0000  Z...[...\...]...
041201b0: 5e00 0000 5f00 0000 6000 0000 6100 0000  ^..._...`...a...

inum = 7: 'console'
041201c0: 0300 0100 0100 0100 0000 0000 0000 0000  ................
041201d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
041201e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
041201f0: 0000 0000 0000 0000 0000 0000 0000 0000  ................

inum = 8: 'ls.txt'
04120200: 0200 0000 0000 0100 f100 0000 6500 0000  ............e...
04120210: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04120220: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04120230: 0000 0000 0000 0000 0000 0000 0000 0000  ................

```

## Free bit map block (0x4127000, fs.img: 0x27000, len: 0x4000)

```
04127000: ffff ffff ffff ffff ffff ffff 1f00 0000  ................
04127010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

## Data block (0x412B00, fs.img: 0x2B000)

```
'/'
0412b000: 0100 2e00 0000 0000 0000 0000 0000 0000  ................
0412b010: 0100 2e2e 0000 0000 0000 0000 0000 0000  ................
0412b020: 0200 6d6b 6673 0000 0000 0000 0000 0000  ..mkfs..........
0412b030: 0300 696e 6974 0000 0000 0000 0000 0000  ..init..........
0412b040: 0400 6361 7400 0000 0000 0000 0000 0000  ..cat...........
0412b050: 0500 6c73 0000 0000 0000 0000 0000 0000  ..ls............
0412b060: 0600 7368 0000 0000 0000 0000 0000 0000  ..sh............

'mkfs'
0412c000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
0412c010: 0200 b700 0100 0000 bc05 4000 0000 0000  ..........@.....
0412c020: 4000 0000 0000 0000 78b6 0000 0000 0000  @.......x.......

'init'
04138000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
04138010: 0200 b700 0100 0000 5802 4000 0000 0000  ........X.@.....
04138020: 4000 0000 0000 0000 e85e 0000 0000 0000  @........^......

'cat'
0413f000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
0413f010: 0200 b700 0100 0000 7002 4000 0000 0000  ........p.@.....
0413f020: 4000 0000 0000 0000 189a 0000 0000 0000  @...............

'ls'
04149000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
04149010: 0200 b700 0100 0000 a801 4000 0000 0000  ..........@.....
04149020: 4000 0000 0000 0000 c8af 0000 0000 0000  @...............

'sh'
04155000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
04155010: 0200 b700 0100 0000 fc02 4000 0000 0000  ..........@.....
04155020: 4000 0000 0000 0000 40e3 0000 0000 0000  @.......@.......

'ls.txt'
04165000: 2e20 2020 2020 2020 2020 2020 2020 2034  .              4
04165010: 3030 3020 3120 3430 3936 0a2e 2e20 2020  000 1 4096...
04165020: 2020 2020 2020 2020 2020 3430 3030 2031            4000 1
04165030: 2034 3039 360a 6d6b 6673 2020 2020 2020   4096.mkfs
04165040: 2020 2020 2038 3030 3020 3220 3437 3830       8000 2 4780
04165050: 300a 696e 6974 2020 2020 2020 2020 2020  0.init
04165060: 2038 3030 3020 3320 3235 3338 340a 6361   8000 3 25384.ca
04165070: 7420 2020 2020 2020 2020 2020 2038 3030  t            800
04165080: 3020 3420 3430 3533 360a 6c73 2020 2020  0 4 40536.ls
04165090: 2020 2020 2020 2020 2038 3030 3020 3520           8000 5
041650a0: 3436 3038 380a 7368 2020 2020 2020 2020  46088.sh
041650b0: 2020 2020 2038 3030 3020 3620 3539 3236       8000 6 5926
041650c0: 340a 636f 6e73 6f6c 6520 2020 2020 2020  4.console
041650d0: 2030 2037 2030 0a6c 732e 7478 7420 2020   0 7 0.ls.txt
041650e0: 2020 2020 2020 3830 3030 2038 2032 3135        8000 8 215
041650f0: 0a00 0000 0000 0000 0000 0000 0000 0000  ................

```

# カード読み取りでエラー

## 1. MBRの読み取りは正常に行われるが、super blockの読み取りでエラー発生。

```
main: [CPU0] Init success.
sdSendCommandA: bno=0x20808                 // SBのbno
sdSendCommandA: index: 15, arg: 0x4101000   // argはSBのセクタ番号で正しい
- EMMC: Sending command READ_MULTI code 12220032 arg 4101000
- inter1: 0x0       // EMM_INTERRUPTクリア後
- inter2: 0x0       // *EMMC_ARG1 = arg;の直後
- cmd: 0x12220032
- inter3: 0x18001   // *EMMC_CMDTM = cmd->code (0x12220032)の直後
- inter4: 0x18001
EMMC: Wait for interrupt 0x1 timeout:
 - count: 1000000, status: 0x1ff0000, inter: 0x18001, resp0: 0x900
- result: 0x2
* EMMC send command error, resp=2
kern/console.c:250: kernel panic at cpu 0.
```

## 2. 割り込みassertを削除

```
main: [CPU0] Init success.
sdSendCommandA: bno=0x20808
sdSendCommandA: index: 15, arg: 4101000
- EMMC: Sending command READ_MULTI code 12220036 arg 4101000
- inter1: 0x0
- inter2: 0x0
- cmd: 0x12220036
- inter3: 0x21
- inter4: 0x21
sb: size 100000, nblocks 99957, ninodes 440, nlog 30, logstart 2, inodestart 32, bmap start 39
iinit: ok
sdSendCommandA: bno=0x20810
sdSendCommandA: index: 15, arg: 4102000
- EMMC: Sending command READ_MULTI code 12220036 arg 4102000
- inter1: 0x0
- inter2: 0x0
- cmd: 0x12220036
- inter3: 0x21
- inter4: 0x21
sdSendCommandA: bno=0x20810
sdSendCommandA: index: 20, arg: 4102000
- EMMC: Sending command WRITE_MULTI code 19220026 arg 4102000
- inter1: 0x0
- inter2: 0x0
- cmd: 0x19220026
- inter3: 0x11
- inter4: 0x11          // ここで止まる
```

## `INT_READ_RDY`と`INT_DATA_DONE`を適切に待つとエラーがなくなった

### 処理時間計測

1. シェルコマンドが出るまで、27秒
2. lsコマンドが結果を表示するまで、12秒
3. `ls > ls.txt`は、2分45秒
4. `ls ls.txt`は、2秒
5. `cat ls.txt`は、12秒
6. `ls`は、2秒

### 考察

1. ファイル読み込みだけのコマンドはファイルサイズに比例する。大体1ブロック1秒。
2. bufキャッシュは効いている
3. ファイル書き込みがあると以前より改善されたが、相変わらず遅い

```
main: [CPU0] Init success.
sb: size 100000, nblocks 99957, ninodes 440, nlog 30, logstart 2, inodestart 32, bmap start 39
iinit: ok
initlog: ok
init log success
init: starting sh
$ ls > ls.txt
$ ls ls.txt
ls.txt         8000 8 241
$ cat ls.txt
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46088
sh             8000 6 59264
console        0 7 0
ls.txt         8000 8 215
$ ls
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46088
sh             8000 6 59264
console        0 7 0
ls.txt         8000 8 241
$
```

## `INT_READ_RDY`を待たない（ブロックを連続読み込みをすると途中で止まる）

```
sdSendCommandA: bno=0x20958
sdSendCommandA: index: 15, arg: 0x412b000
- EMMC: Sending command READ_MULTI code 12220036 arg 412b000
r, 0: inter=0x0, 1: inter=0x20, 2: inter=0x20, 3: inter=0x20, 4: inter=0x20, 5: inter=0x20, 6: inter=0x20, 7: inter=0x20, intr=0x22
```

## readでi loopごとに`INT_READ_RDY`を待つ

```
sdSendCommandA: bno=0x20958
sdSendCommandA: index: 15, arg: 0x412b000
- EMMC: Sending command READ_MULTI code 12220036 arg 412b000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0EMMC: Wait for interrupt 0x20 timeout:
 - count: -1, status: 0x1ff0000, inter: 0x2, resp0: 0x900
, intr=0x0
EMMC: Wait for interrupt 0x2 timeout:
 - count: -1, status: 0x1ff0000, inter: 0x0, resp0: 0x900
```

## 上出来、i=7だけ`INT_READ_RDY`を待たない

```
$ make qemu
qemu-system-aarch64 -M raspi3 -nographic -serial null -serial mon:stdio -drive file=obj/sd.img,if=sd,format=raw -kernel obj/kernel8.img
[CPU0] irq_init: success.
- mbox write: 0x7ef08
- mbox_read: 0x7ef08
- clock rate 50000000
- SD base clock rate from mailbox: 50000000.
- sdInit: reset the card
- Divisor selected = 104, pow 2 shift count = 6
- EMMC: Set clock, status 0x1ff0000 CONTROL1: 0xe6807
- Send IX_GO_IDLE_STATE command
- EMMC: Sending command GO_IDLE_STATE code 0 arg 0
sdSendCommandA: index: 6, arg: 0x1aa
- EMMC: Sending command SEND_IF_COND code 8020000 arg 1aa
- sdSendCommandA response: 0
- EMMC: Sending ACMD41 SEND_OP_COND status 1ff0000
sdSendCommandA: index: 36, arg: 0x51ff8000
- EMMC: Sending command APP_CMD code 37000000 arg 0
- EMMC: Sending command SD_SENDOPCOND code 29020000 arg 51ff8000
- EMMC: Sending command ALL_SEND_CID code 2010000 arg 0
- EMMC: Sending command SEND_REL_ADDR code 3020000 arg 0
- EMMC: Sending command SEND_CSD code 9010000 arg 45670000
- Divisor selected = 2, pow 2 shift count = 0
- EMMC: Set clock, status 0x1ff0000 CONTROL1: 0xe0207
- EMMC: Sending command CARD_SELECT code 7030000 arg 45670000
- EMMC: Sending command APP_CMD code 37020000 arg 45670000
- EMMC: Sending command SEND_SCR code 33220010 arg 0
sdSendCommandA: index: 32, arg: 0x45670002
- EMMC: Sending command APP_CMD code 37020000 arg 45670000
- EMMC: Sending command SET_BUS_WIDTH code 6020000 arg 45670002
sdSendCommandA: index: 13, arg: 0x200
- EMMC: Sending command SET_BLOCKLEN code 10020000 arg 200
- EMMC: SD Card Type 2 SC 1024Mb UHS-I 0 mfr 170 'XY:QEMU!' r0.1 2/2006, #deadbeef RCA 4567
sdSendCommandA: bno=0x0
sdSendCommandA: index: 15, arg: 0x0
- EMMC: Sending command READ_MULTI code 12220036 arg 0

sd_init: Partition 1: 00 20 21 00 0c 49 01 08 00 08 00 00 00 00 02 00
- Status: 0
- CHS address of first absolute sector: header=5d, sector=32, cylinder=33
- Partition type: 12
- CHS address of last absolute sector: header=5d, sector=73, cylinder=1
- LBA of first absolute sector: 0x800
- Number of sectors: 131072

sd_init: Partition 2: 00 49 02 08 83 8a 08 82 00 08 02 00 00 f8 1d 00
- Status: 0
- CHS address of first absolute sector: header=5d, sector=73, cylinder=2
- Partition type: 131
- CHS address of last absolute sector: header=5d, sector=138, cylinder=8
- LBA of first absolute sector: 0x20800
- Number of sectors: 1964032
sd_init: Boot signature: 55 aa
sd_init: success
user_init: proc 1 (initcode) success.
user_idle_init: proc 2 success.
user_idle_init: proc 3 success.
user_idle_init: proc 4 success.
user_idle_init: proc 5 success.
main: [CPU0] Init success.
sdSendCommandA: bno=0x20808
sdSendCommandA: index: 15, arg: 0x4101000
- EMMC: Sending command READ_MULTI code 12220036 arg 4101000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sb: size 100000, nblocks 99957, ninodes 440, nlog 30, logstart 2, inodestart 32, bmap start 39
iinit: ok
sdSendCommandA: bno=0x20810
sdSendCommandA: index: 15, arg: 0x4102000
- EMMC: Sending command READ_MULTI code 12220036 arg 4102000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20810
sdSendCommandA: index: 20, arg: 0x4102000
- EMMC: Sending command WRITE_MULTI code 19220026 arg 4102000
w01234567
initlog: ok
init log success
sdSendCommandA: bno=0x20900
sdSendCommandA: index: 15, arg: 0x4120000
- EMMC: Sending command READ_MULTI code 12220036 arg 4120000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20958
sdSendCommandA: index: 15, arg: 0x412b000
- EMMC: Sending command READ_MULTI code 12220036 arg 412b000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x209c0
sdSendCommandA: index: 15, arg: 0x4138000
- EMMC: Sending command READ_MULTI code 12220036 arg 4138000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x209c8
sdSendCommandA: index: 15, arg: 0x4139000
- EMMC: Sending command READ_MULTI code 12220036 arg 4139000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x209d0
sdSendCommandA: index: 15, arg: 0x413a000
- EMMC: Sending command READ_MULTI code 12220036 arg 413a000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x209d8
sdSendCommandA: index: 15, arg: 0x413b000
- EMMC: Sending command READ_MULTI code 12220036 arg 413b000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
mknodat: path 'console', major:minor 1:1
sdSendCommandA: bno=0x20818
sdSendCommandA: index: 15, arg: 0x4103000
- EMMC: Sending command READ_MULTI code 12220036 arg 4103000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20818
sdSendCommandA: index: 20, arg: 0x4103000
- EMMC: Sending command WRITE_MULTI code 19220026 arg 4103000
w01234567
sdSendCommandA: bno=0x20820
sdSendCommandA: index: 15, arg: 0x4104000
- EMMC: Sending command READ_MULTI code 12220036 arg 4104000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20820
sdSendCommandA: index: 20, arg: 0x4104000
- EMMC: Sending command WRITE_MULTI code 19220026 arg 4104000
w01234567
sdSendCommandA: bno=0x20810
sdSendCommandA: index: 20, arg: 0x4102000
- EMMC: Sending command WRITE_MULTI code 19220026 arg 4102000
w01234567
sdSendCommandA: bno=0x20900
sdSendCommandA: index: 20, arg: 0x4120000
- EMMC: Sending command WRITE_MULTI code 19220026 arg 4120000
w01234567
sdSendCommandA: bno=0x20958
sdSendCommandA: index: 20, arg: 0x412b000
- EMMC: Sending command WRITE_MULTI code 19220026 arg 412b000
w01234567
sdSendCommandA: bno=0x20810
sdSendCommandA: index: 20, arg: 0x4102000
- EMMC: Sending command WRITE_MULTI code 19220026 arg 4102000
w01234567
init: starting sh
sdSendCommandA: bno=0x20aa8
sdSendCommandA: index: 15, arg: 0x4155000
- EMMC: Sending command READ_MULTI code 12220036 arg 4155000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20ab0
sdSendCommandA: index: 15, arg: 0x4156000
- EMMC: Sending command READ_MULTI code 12220036 arg 4156000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20ab8
sdSendCommandA: index: 15, arg: 0x4157000
- EMMC: Sending command READ_MULTI code 12220036 arg 4157000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20ac0
sdSendCommandA: index: 15, arg: 0x4158000
- EMMC: Sending command READ_MULTI code 12220036 arg 4158000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20ac8
sdSendCommandA: index: 15, arg: 0x4159000
- EMMC: Sending command READ_MULTI code 12220036 arg 4159000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20ad0
sdSendCommandA: index: 15, arg: 0x415a000
- EMMC: Sending command READ_MULTI code 12220036 arg 415a000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20ad8
sdSendCommandA: index: 15, arg: 0x415b000
- EMMC: Sending command READ_MULTI code 12220036 arg 415b000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20ae0
sdSendCommandA: index: 15, arg: 0x415c000
- EMMC: Sending command READ_MULTI code 12220036 arg 415c000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20ae8
sdSendCommandA: index: 15, arg: 0x415d000
- EMMC: Sending command READ_MULTI code 12220036 arg 415d000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20af0
sdSendCommandA: index: 15, arg: 0x415e000
- EMMC: Sending command READ_MULTI code 12220036 arg 415e000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20af8
sdSendCommandA: index: 15, arg: 0x415f000
- EMMC: Sending command READ_MULTI code 12220036 arg 415f000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
$ ls
sdSendCommandA: bno=0x20a48
sdSendCommandA: index: 15, arg: 0x4149000
- EMMC: Sending command READ_MULTI code 12220036 arg 4149000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20a50
sdSendCommandA: index: 15, arg: 0x414a000
- EMMC: Sending command READ_MULTI code 12220036 arg 414a000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20a58
sdSendCommandA: index: 15, arg: 0x414b000
- EMMC: Sending command READ_MULTI code 12220036 arg 414b000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20a60
sdSendCommandA: index: 15, arg: 0x414c000
- EMMC: Sending command READ_MULTI code 12220036 arg 414c000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20a68
sdSendCommandA: index: 15, arg: 0x414d000
- EMMC: Sending command READ_MULTI code 12220036 arg 414d000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20a70
sdSendCommandA: index: 15, arg: 0x414e000
- EMMC: Sending command READ_MULTI code 12220036 arg 414e000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20a78
sdSendCommandA: index: 15, arg: 0x414f000
- EMMC: Sending command READ_MULTI code 12220036 arg 414f000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20a80
sdSendCommandA: index: 15, arg: 0x4150000
- EMMC: Sending command READ_MULTI code 12220036 arg 4150000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
sdSendCommandA: bno=0x20a88
sdSendCommandA: index: 15, arg: 0x4151000
- EMMC: Sending command READ_MULTI code 12220036 arg 4151000
r, 0: inter=0x0, 1: inter=0x0, 2: inter=0x0, 3: inter=0x0, 4: inter=0x0, 5: inter=0x0, 6: inter=0x0, 7: inter=0x0, intr=0x2
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46088
sh             8000 6 59264
console        0 7 0
```

# 実機でSDカードを作成

1. Ubuntu DisksでSDカードを初期化（No partitioning (Empty)）
2. Restore Disk imageでsd.imgをレストア
3. sfdiskで確認。

```
$ sudo sfdisk -l /dev/sdc
Disk /dev/sdc: 14.5 GiB, 15552479232 bytes, 30375936 sectors
Disk model: Storage Device
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x526d32b3

Device     Boot  Start     End Sectors  Size Id Type
/dev/sdc1         2048  133119  131072   64M  c W95 FAT32 (LBA)
/dev/sdc2       133120 2097151 1964032  959M 83 Linux
```

### 処理時間計測

1. シェルコマンドが出るまで、1分28秒
2. lsコマンドが結果を表示するまで、40秒
3. `ls > ls.txt2`は、9分
4. `ls ls2.txt`は、5秒
5. `cat ls2.txt`は、38秒
6. `ls`は、6秒
7. `cat ls2.txt`は、7秒

# writeが遅い問題

- lsが表示行1行毎に`filewrite`を2回実行している。
- lsの実装がdinode一つごとに出力している。

```
execve: '/init' start
mknodat: path 'console', major:minor 1:1
dirlink: name=console, inum=7
 - wi: ip=1, off=0x70, n=16
sys_writev call filewrite
- fw: 'console', n=17
 - wi: ip=7, off=0x0, n=17
init: starting shsys_writev call filewrite
- fw: 'console', n=1
 - wi: ip=7, off=0x11, n=1

execve: 'sh' start
sys_writev call filewrite
- fw: 'console', n=0
sys_writev call filewrite
- fw: 'console', n=2
 - wi: ip=7, off=0x12, n=2
$ ls > ls.txt
dirlink: name=ls.txt, inum=8
 - wi: ip=1, off=0x80, n=16
execve: 'ls' start
sys_writev call filewrite
- fw: 'ls.txt', n=26            // .              4000 1 4096
 - wi: ip=8, off=0x0, n=26
sys_writev call filewrite
- fw: 'ls.txt', n=1             // 改行
 - wi: ip=8, off=0x1a, n=1
sys_writev call filewrite
- fw: 'ls.txt', n=26            // ..             4000 1 4096
 - wi: ip=8, off=0x1b, n=26
sys_writev call filewrite
- fw: 'ls.txt', n=1             // 改行
 - wi: ip=8, off=0x35, n=1
sys_writev call filewrite
- fw: 'ls.txt', n=27            // mkfs           8000 2 47800
 - wi: ip=8, off=0x36, n=27
sys_writev call filewrite
- fw: 'ls.txt', n=1             // 改行
 - wi: ip=8, off=0x51, n=1
sys_writev call filewrite
- fw: 'ls.txt', n=27
 - wi: ip=8, off=0x52, n=27
sys_writev call filewrite
- fw: 'ls.txt', n=1
 - wi: ip=8, off=0x6d, n=1
sys_writev call filewrite
- fw: 'ls.txt', n=27
 - wi: ip=8, off=0x6e, n=27
sys_writev call filewrite
- fw: 'ls.txt', n=1
 - wi: ip=8, off=0x89, n=1
sys_writev call filewrite
- fw: 'ls.txt', n=27
 - wi: ip=8, off=0x8a, n=27
sys_writev call filewrite
- fw: 'ls.txt', n=1
 - wi: ip=8, off=0xa5, n=1
sys_writev call filewrite
- fw: 'ls.txt', n=27
 - wi: ip=8, off=0xa6, n=27
sys_writev call filewrite
- fw: 'ls.txt', n=1
 - wi: ip=8, off=0xc1, n=1
sys_writev call filewrite
- fw: 'ls.txt', n=20
 - wi: ip=8, off=0xc2, n=20
sys_writev call filewrite
- fw: 'ls.txt', n=1
 - wi: ip=8, off=0xd6, n=1
sys_writev call filewrite
- fw: 'ls.txt', n=25
 - wi: ip=8, off=0xd7, n=25
sys_writev call filewrite
- fw: 'ls.txt', n=1
 - wi: ip=8, off=0xf0, n=1
sys_writev call filewrite
- fw: 'console', n=0
sys_writev call filewrite
- fw: 'console', n=2
 - wi: ip=7, off=0x20, n=2
$ cat ls.txt
execve: 'cat' start
sys_write call filewrite
- fw: 'console', n=241
 - wi: ip=7, off=0x2d, n=241
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46088
sh             8000 6 59264
console        0 7 0
ls.txt         8000 8 215
sys_writev call filewrite
- fw: 'console', n=0
sys_writev call filewrite
- fw: 'console', n=2
 - wi: ip=7, off=0x11e, n=2
$
```

## `ls`の出力をまとめて出力するよう修正

- `ls > ls.txt`が34秒に
- `ls > ls2.txt`は21秒  （lsはキャッシュから）
- 実機ではだんまりに（fsを更新したら再現せず）

```
execve: '/init' start
mknodat: path 'console', major:minor 1:1
dirlink: name=console, inum=7
 - wi: ip=1, off=0x70, n=16
sys_writev call filewrite
- fw: 'console', n=17
 - wi: ip=7, off=0x0, n=17
init: starting shsys_writev call filewrite
- fw: 'console', n=1
 - wi: ip=7, off=0x11, n=1

execve: 'sh' start
sys_writev call filewrite
- fw: 'console', n=0
sys_writev call filewrite
- fw: 'console', n=2
 - wi: ip=7, off=0x12, n=2
$ ls > ls.txt
dirlink: name=ls.txt, inum=8
 - wi: ip=1, off=0x80, n=16
execve: 'ls' start
sys_writev call filewrite
- fw: 'ls.txt', n=0
sys_writev call filewrite
- fw: 'ls.txt', n=239           // まとめて出力
 - wi: ip=8, off=0x0, n=239
sys_writev call filewrite
- fw: 'console', n=0
sys_writev call filewrite
- fw: 'console', n=2
 - wi: ip=7, off=0x20, n=2
$ cat ls.txt
execve: 'cat' start
sys_write call filewrite
- fw: 'console', n=239
 - wi: ip=7, off=0x2d, n=239
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46440
sh             8000 6 59264
console        0 7 0
ls.txt         8000 8 0
sys_writev call filewrite
- fw: 'console', n=0
sys_writev call filewrite
- fw: 'console', n=2
 - wi: ip=7, off=0x11c, n=2
$
```


## 実機で

- ファイルシステムを再作成したら本件は再現しなかった。
- `ls > ls.txt`は1分21秒（ただし、直前に`ls`コマンドを実行しているので、実質2分くらいか）

```
$ ls > ls3.txt                      // ここでストール

切断、再実行

$ ls
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46088
sh             8000 6 59264
console        0 7 0
ls.txt         8000 8 241
ls2.txt        8000 9 267
ls3.txt        8000 10 293          // ファイルはできている
$ cat ls3.txt
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46088
sh             8000 6 59264
console        0 7 0
ls.txt         8000 8 241
ls2.txt        8000 9 267
ls3.txt        8000 10 267$         // 最後の改行がない
```

- SDカードの動作周波数が遅い？

```
[CPU0] irq_init: success.
- mbox write: 0x7ef08
- mbox_read: 0x7ef08
- clock rate 200000000
- SD base clock rate from mailbox: 200000000.
- sdInit: reset the card
- Divisor selected = 104, pow 2 shift count = 6
- EMMC: Set clock, status 0x1ff0000 CONTROL1: 0xe6807
- Send IX_GO_IDLE_STATE command
- sdSendCommandA response: 0
- EMMC: Sending ACMD41 SEND_OP_COND status 1ff0000
- EMMC: Retrying ACMD SEND_OP_COND status 1ff0000
- EMMC: Pi does not support switch voltage, fixed at 3.3volt
- Divisor selected = 2, pow 2 shift count = 0
- EMMC: Set clock, status 0x1ff0000 CONTROL1: 0xe0207
- EMMC: SD Card Type 2 HC 14832Mb UHS-I 0 mfr 2 'TM:SA16G' r3.1 8/2019, #299f694

sd_init: Partition 1: 00 20 21 00 0c 49 01 08 00 08 00 00 00 00 02 00
- Status: 0
- CHS address of first absolute sector: header=5d, sector=32, cylinder=33
- Partition type: 12
- CHS address of last absolute sector: header=5d, sector=73, cylinder=1
- LBA of first absolute sector: 0x800
- Number of sectors: 131072

sd_init: Partition 2: 00 49 02 08 83 8a 08 82 00 08 02 00 00 f8 1d 00
- Status: 0
- CHS address of first absolute sector: header=5d, sector=73, cylinder=2
- Partition type: 131
- CHS address of last absolute sector: header=5d, sector=138, cylinder=8
- LBA of first absolute sector: 0x20800
- Number of sectors: 1964032
sd_init: Boot signature: 55 aa
sd_init: success
```

## `bwrite`のデバッグ

- `sdrw()`を呼び出しているのは、`bio.c#bwrite()`
- `bwrite()`を呼び出しているのは、`log.c#write_head(), write_log()`

### QEMU

```
$ ls > ls.txt
write_log: bno=0x3          // ここから `ls`コマンド関係
- bw: bno=0x3
write_log: bno=0x4
- bw: bno=0x4
write_head: bno=0x2
- bw: bno=0x2
- bw: bno=0x20
- bw: bno=0x2b
write_head: bno=0x2
- bw: bno=0x2
write_log: bno=0x3          // ここからが > による書き込み関係
- bw: bno=0x3
write_log: bno=0x4
- bw: bno=0x4
write_log: bno=0x5
- bw: bno=0x5
write_head: bno=0x2
- bw: bno=0x2
- bw: bno=0x27
- bw: bno=0x65
- bw: bno=0x20
write_head: bno=0x2
- bw: bno=0x2
$
```

### 実機

```
$ ls > ls.txt
write_log: bno=0x3
- bw: bno=0x3
write_log: bno=0x4
- bw: bno=0x4
write_head: bno=0x2
- bw: bno=0x2
- bw: bno=0x65
- bw: bno=0x20
write_head: bno=0x2
- bw: bno=0x2
```

# 実行速度が遅い問題が解決

- プロセスを切り替えるスライスタイム(timerのリセットタイム）が1秒に設定されていた
- 1/60秒に変更したら普通の処理速度となった
- BSIZEやlsコマンドの細工はそのまま
  - BSIZEは最大ファイル容量が増えるのでそのまま
  - lsコマンドは特に問題がないのでそのまま

# GNUのcoreutilsを導入

[導入メモ](coreutils.md)

# aarch64のデータモデル

![aarch64 data model](screens/aarch64_data_model.png)
