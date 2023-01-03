# eggman氏による「ベアメタルでRaspberry Pi 3のUSBコントロール転送」

[Qiita記事](https://qiita.com/eggman/items/5b9e4f5802f0ec22921f)

## USB02 実行結果

- ubuntuで実行
- コンパイラは`aarch64-linux-gnu-gcc`に変更
- QEMUのマシン指定は`raspi3b`に変更
- ubuntuのqemuは`trace backend = log`でコンパイルされている
- `make trun`でトレース出力された

```
$ make trun
make kernel.elf
make[1]: Entering directory '/home/vagrant/raspi_eggman/qemu-raspi3/usb02'
aarch64-linux-gnu-gcc -mcpu=cortex-a53 -fpic -ffreestanding -std=gnu99 -O2 -Wall -Wextra -c -o kernel.o kernel.c
aarch64-linux-gnu-gcc -Wl,--build-id=none -T linker.ld -o kernel.elf -ffreestanding -O2 -nostdlib boot.o kernel.o
aarch64-linux-gnu-objdump -D kernel.elf > kernel.list
make[1]: Leaving directory '/home/vagrant/raspi_eggman/qemu-raspi3/usb02'
qemu-system-aarch64 -M raspi3b -m 1024 -serial null -serial mon:stdio -nographic -device usb-kbd -kernel kernel.elf -trace events=events
usb_port_claim bus 0, port 1
usb_hub_reset dev 0
usb_port_attach bus 0, port 1, devspeed full, portspeed full+high
usb_dwc2_attach port 0x7f2788096a30
usb_dwc2_attach_speed full-speed device attached
usb_dwc2_bus_start start SOFs
usb_dwc2_raise_global_irq 0x01000000
usb_port_claim bus 0, port 1.1
usb_port_attach bus 0, port 1.1, devspeed full+high, portspeed full
usb_hub_attach dev 0, port 1
usb_dwc2_reset_enter === RESET enter ===
usb_dwc2_detach port 0x7f2788096a30
usb_dwc2_bus_stop stop SOFs
usb_dwc2_bus_stop stop SOFs
usb_dwc2_reset_hold === RESET hold ===
usb_dwc2_reset_exit === RESET exit ===
usb_dwc2_attach port 0x7f2788096a30
usb_dwc2_attach_speed full-speed device attached
usb_dwc2_bus_start start SOFs
usb_dwc2_raise_global_irq 0x01000000
usb_hub_reset dev 0

> qemu exit: Ctrl-A x / qemu monitor: Ctrl-A c      // kernel_main()で出力
> usb02
> usb_buffer0:00083100
> usb_buffer1:00083000

USB_CORE_GUID    usb_dwc2_glbreg_read  0x003c GUID      val 0x00000000
00000000
USB_CORE_GSNPSID usb_dwc2_glbreg_read  0x0040 GSNPSID   val 0x4f54294a
4F54294A
usb_dwc2_hreg0_read   0x0440 HPRT0     val 0x00021003
usb_dwc2_hreg0_write  0x0440 HPRT0     val 0x00021003 old 0x00021003 result 0x00021001
usb_dwc2_hreg0_action disable PRTINT
usb_dwc2_lower_global_irq 0x01000000
usb_dwc2_hreg0_read   0x0440 HPRT0     val 0x00021001
usb_dwc2_hreg0_write  0x0440 HPRT0     val 0x00021101 old 0x00021001 result 0x00021101
usb_dwc2_hreg0_action disable PRTINT
usb_dwc2_hreg0_read   0x0440 HPRT0     val 0x00021101
usb_dwc2_hreg0_write  0x0440 HPRT0     val 0x00021001 old 0x00021101 result 0x0002100d
usb_dwc2_hreg0_action call usb_port_reset
usb_dwc2_detach port 0x7f2788096a30
usb_dwc2_bus_stop stop SOFs
usb_dwc2_raise_global_irq 0x01000000
usb_dwc2_attach port 0x7f2788096a30
usb_dwc2_attach_speed full-speed device attached
usb_dwc2_bus_start start SOFs
usb_hub_reset dev 0
usb_dwc2_hreg0_action enable PRTINT
usb_dwc2_glbreg_read  0x0008 GAHBCFG   val 0x00000000
usb_dwc2_glbreg_write 0x0008 GAHBCFG   val 0x00000001 old 0x00000000 result 0x00000001
usb_dwc2_glbreg_write 0x0018 GINTMSK   val 0x11000000 old 0x00000000 result 0x11000000
usb_dwc2_update_irq level=1
usb_dwc2_hreg0_write  0x0440 HPRT0     val 0x0000100a old 0x0002100d result 0x00021005
usb_dwc2_hreg0_action disable PRTINT
usb_dwc2_lower_global_irq 0x01000000
usb_dwc2_hreg0_read   0x0418 HAINTMSK  val 0x00000000
usb_dwc2_hreg0_write  0x0418 HAINTMSK  val 0x00000003 old 0x00000000 result 0x00000003
usb_dwc2_hreg1_read   0x050c HCINTMSK40 val 0x00000000
usb_dwc2_hreg1_write  0x050c HCINTMSK0 val 0x00000001 old 0x00000000 result 0x00000001
usb_dwc2_hreg1_read   0x052c HCINTMSK41 val 0x00000000
usb_dwc2_hreg1_write  0x052c HCINTMSK1 val 0x00000001 old 0x00000000 result 0x00000001
usb_dwc2_hreg1_read   0x0500 HCCHAR  40 val 0x00000000
usb_dwc2_hreg1_write  0x0500 HCCHAR  0 val 0x00000040 old 0x00000000 result 0x00000040
usb_dwc2_hreg1_read   0x0520 HCCHAR  41 val 0x00000000
usb_dwc2_hreg1_write  0x0520 HCCHAR  1 val 0x00008040 old 0x00000000 result 0x00008040
usb_dwc2_glbreg_read  0x0014 GINTSTS   val 0x14000021
usb_dwc2_glbreg_write 0x0014 GINTSTS   val 0x14000021 old 0x14000021 result 0xffffffff04000021
usb_dwc2_update_irq level=0
14000021 usb interrupt
Connector ID Status Change
Periodic TxFIFO Empty
Non-periodic TxFIFO Empty
usb_dwc2_hreg1_read   0x0500 HCCHAR  40 val 0x00000040
usb_dwc2_hreg1_write  0x0500 HCCHAR  0 val 0x00000040 old 0x00000040 result 0x00000040
usb_dwc2_hreg1_read   0x0520 HCCHAR  41 val 0x00008040
usb_dwc2_hreg1_write  0x0520 HCCHAR  1 val 0x00008040 old 0x00008040 result 0x00008040
usb_dwc2_hreg1_write  0x0514 HCDMA   0 val 0x00083100 old 0x00000000 result 0x00083100
usb_dwc2_hreg1_read   0x0514 HCDMA   40 val 0x00083100
usb_dwc2_hreg1_write  0x0514 HCDMA   0 val 0xc0083100 old 0x00083100 result 0xc0083100
usb_dwc2_hreg1_write  0x0510 HCTSIZ  0 val 0x60080008 old 0x00000000 result 0x60080008
usb_dwc2_hreg1_read   0x0500 HCCHAR  40 val 0x00000040
usb_dwc2_hreg1_write  0x0500 HCCHAR  0 val 0x80000040 old 0x00000040 result 0x80000040
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x564c00832800 pkt 0x7f2788096a78 ep 0
usb_dwc2_handle_packet ch 0 dev 0x564c00832800 pkt 0x7f2788096a78 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073204992 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7f2788096a78, state undef -> setup
usb_hub_control dev 0, req 0xa006, value 256, index 0, langth 64
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7f2788096a78, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_raise_host_irq 0x0001
usb_dwc2_raise_global_irq 0x02000000
usb_dwc2_hreg1_write  0x0534 HCDMA   1 val 0x00083000 old 0x00000000 result 0x00083000
usb_dwc2_hreg1_read   0x0534 HCDMA   41 val 0x00083000
usb_dwc2_hreg1_write  0x0534 HCDMA   1 val 0xc0083000 old 0x00083000 result 0xc0083000
usb_dwc2_hreg1_write  0x0530 HCTSIZ  1 val 0x40080040 old 0x00000000 result 0x40080040
usb_dwc2_hreg1_read   0x0520 HCCHAR  41 val 0x00008040
usb_dwc2_hreg1_write  0x0520 HCCHAR  1 val 0x80008040 old 0x00008040 result 0x80008040
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x564c00832800 pkt 0x7f2788096b28 ep 0
usb_dwc2_handle_packet ch 1 dev 0x564c00832800 pkt 0x7f2788096b28 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7f2788096b28, state undef -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7f2788096b28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 10
usb_dwc2_memory_write addr -1073205248 len 10
usb_dwc2_packet_done status USB_RET_SUCCESS actual 10 len 54 pcnt 0
usb_dwc2_raise_host_irq 0x0002
GET_DESCRIPTOR
0000000Ausb_dwc2_work_bh
usb_dwc2_raise_global_irq 0x00000008

00000029
00000008
0000000A
00000000
00000001
00000000
00000000
00000000
000000FF
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
QEMU: Terminated
```

## USB03 実行結果

### `make run`

```
qemu-system-aarch64 -M raspi3b -m 1024 -serial null -serial mon:stdio -nographic -device usb-kbd -kernel kernel.elf
qemu exit: Ctrl-A x / qemu monitor: Ctrl-A c
usb03
usb_buffer0: 84200
usb_buffer1: 84100
usb_buffer3: 84000
USB_CORE_GUID    0
USB_CORE_GSNPSID 4f54294a
 usb interrupt 14000029
 Connector ID STATUS Change
 Periodic TxFIFO Empty
 Non-periodic TxFIFO Empty
 Host and Device Start of Frame

GET DEVICE_DESCRIPTOR
Len Typ Maj Min Cla Scl Prt MPS Vendor  Product Relese  id1 id2 id3 NConf
12  01  10  01  09  00  00  08  09  04  aa  55  01  01  01  02  03  01

SET ADDRESS 2

GET CONFIG_DESCRIPTOR
Len Typ Tlen   NInf ConfV idx Attr MP |-> interface_desc
09  02  19 00  01   01    00  e0   00  09  04  00 00 01 09 00 00 00

SET CONFIG 1    // 1 = Config_Desc.ConfigValue

GET HUB_DESCRIPTOR (Table 11.23.2.1)
Len Typ Npt Char   Pw
0a  29  08  0a 00  01  00 00 00 ff 00 00 00 00 00 00 00 00

Hub device 0 has 8 ports

PORT STATUS 0 01 00 00 00
PORT STATUS 1 01 01 01 00
PORT STATUS 1 01 01 00 00
SET PORT 1 POWER ON
PORT STATUS 1 01 01 00 00
CLEAR PORT FEATURE
SET PORT 1 RESET
CLEAR PORT FEATURE
GET DEVICE_DESCRIPTOR
12 01 00 02 00 00 00 08 27 06 01 00 00 00 01 04 0b 01
GET CONFIG_DESCRIPTOR
09 02 22 00 01 01 08 a0 32 09 04 00 00 01 03 01 01 00
QEMU: Terminated
```

### `make trun`

```
qemu-system-aarch64 -M raspi3b -m 1024 -serial null -serial mon:stdio -nographic -device usb-kbd -kernel kernel.elf -trace events=events
usb_port_claim bus 0, port 1
usb_hub_reset dev 0
usb_port_attach bus 0, port 1, devspeed full, portspeed full+high
usb_dwc2_attach port 0x7fd6558669e0
usb_dwc2_attach_speed full-speed device attached
usb_dwc2_bus_start start SOFs
usb_dwc2_raise_global_irq 0x01000000
usb_port_claim bus 0, port 1.1
usb_port_attach bus 0, port 1.1, devspeed full+high, portspeed full
usb_hub_attach dev 0, port 1
usb_dwc2_reset_enter === RESET enter ===
usb_dwc2_detach port 0x7fd6558669e0
usb_dwc2_bus_stop stop SOFs
usb_dwc2_bus_stop stop SOFs
usb_dwc2_reset_hold === RESET hold ===
usb_dwc2_reset_exit === RESET exit ===
usb_dwc2_attach port 0x7fd6558669e0
usb_dwc2_attach_speed full-speed device attached
usb_dwc2_bus_start start SOFs
usb_dwc2_raise_global_irq 0x01000000
usb_hub_reset dev 0
qemu exit: Ctrl-A x / qemu monitor: Ctrl-A c
usb03
usb_buffer0: 84200
usb_buffer1: 84100
usb_bufusb_dwc2_raise_global_irq 0x00000008
fer3: 84000
usb_dwc2_glbreg_read  0x003c GUID      val 0x00000000
USB_CORE_GUID    0
usb_dwc2_glbreg_read  0x0040 GSNPSID   val 0x4f54294a
USB_CORE_GSNPSID 4f54294a
usb_dwc2_lower_global_irq 0x01000000
usb_dwc2_detach port 0x7fd6558669e0
usb_dwc2_bus_stop stop SOFs
usb_dwc2_raise_global_irq 0x01000000
usb_dwc2_attach port 0x7fd6558669e0
usb_dwc2_attach_speed full-speed device attached
usb_dwc2_bus_start start SOFs
usb_hub_reset dev 0
usb_dwc2_glbreg_read  0x0008 GAHBCFG   val 0x00000000
usb_dwc2_glbreg_write 0x0008 GAHBCFG   val 0x00000001 old 0x00000000 result 0x00000001
usb_dwc2_glbreg_write 0x0018 GINTMSK   val 0x11000000 old 0x00000000 result 0x11000000
usb_dwc2_update_irq level=1
usb_dwc2_lower_global_irq 0x01000000
usb_dwc2_glbreg_read  0x0014 GINTSTS   val 0x14000029
usb_dwc2_glbreg_write 0x0014 GINTSTS   val 0x14000029 old 0x14000029 result 0xffffffff04000021
usb_dwc2_update_irq level=0
 usb interrupt 14000029
 Connector ID STATUS Change
 Periodic TxFIFO Empty
 Non-periodic TxFIFO Empty
 Host and Device Start of Frame
GET DEVICE_DESCRIPTOR
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state undef -> setup
usb_hub_control dev 0, req 0x8006, value 256, index 0, langth 64
usb_desc_device dev 0 query device, len 64, ret 18
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_raise_host_irq 0x0001
usb_dwc2_raise_global_irq 0x02000000
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state undef -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 18
usb_dwc2_memory_write addr -1073200896 len 18
usb_dwc2_packet_done status USB_RET_SUCCESS actual 18 len 46 pcnt 0
usb_dwc2_raise_host_irq 0x0002
usb_dwc2_work_bh
usb_dwc2_raise_global_irq 0x00000008
12 01 10 01 09 00 00 08 09 04 aa 55 01 01 01 02 03 01
SET ADDRESS 2
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_hub_control dev 0, req 0x5, value 2, index 0, langth 0
usb_set_addr dev 2
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 0
usb_dwc2_memory_write addr -1073200896 len 0
usb_dwc2_packet_done status USB_RET_SUCCESS actual 0 len 64 pcnt 1
usb_dwc2_work_bh
GET CONFIG_DESCRIPTOR
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_hub_control dev 2, req 0x8006, value 512, index 0, langth 64
usb_desc_config dev 2 query config 0, len 64, ret 25
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0,09 02 19 00 01 01 00 e0 00 09 04 00 00 01 09 00 00 00
SET CONFIG 1
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_hub_control dev 2, req 0x9, value 1, index 0, langth 0
usb_set_config dev 2, config 1, ret 0
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 0
usb_dwc2_memory_write addr -1073200896 len 0
usb_dwc2_packet_done status USB_RET_SUCCESS actual 0 len 64 pcnt 1
usb_dwc2_work_bh
GET HUB_DESCRIPTOR
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_hub_control dev 2, req 0xa006, value 10496, index 0, langth 64
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 10
usb_dwc2_memory_write addr -1073200896 len 10
usb_dwc2_packet_done status USB_RET_SUCCESS actual 10 len 54 pcnt 0
usb_dwc2_work_bh
0a 29 08 0a 00 01 00 00 00 ff 00 00 00 00 00 00 00 00
Hub device 0 has 8 ports
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_hub_control dev 2, req 0x8000, value 0, index 0, langth 4
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 2
usb_dwc2_memory_write addr -1073200896 len 2
usb_dwc2_packet_done status USB_RET_SUCCESS actual 2 len 62 pcnt 0
PORT STATUS 0 usb_dwc2_work_bh
01 00 00 00
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_hub_control dev 2, req 0xa300, value 0, index 1, langth 4
usb_hub_get_port_status dev 2, port 1, status 0x101, changed 0x1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0,PORT STATUS usb_dwc2_work_bh
1 01 01 01 00
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_hub_control dev 2, req 0x2301, value 16, index 1, langth 0
usb_hub_clear_port_feature dev 2, port 1, feature change-connection
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 0
usb_dwc2_memory_write addr -1073200896 len 0
usb_dwc2_packet_done status USB_RET_SUCCESS actual 0 len 64 pcnt 1
usb_dwc2_work_bh
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_hub_control dev 2, req 0xa300, value 0, index 1, langth 4
usb_hub_get_port_status dev 2, port 1, status 0x101, changed 0x0
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0,PORT STATUusb_dwc2_work_bh
S 1 01 01 00 00
SET PORT 1 POWER ON
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_hub_control dev 2, req 0x2303, value 8, index 1, langth 0
usb_hub_set_port_feature dev 2, port 1, feature power
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 0
usb_dwc2_memory_write addr -1073200896 len 0
usb_dwc2_packet_done status USB_RET_SUCCESS actual 0 len 64 pcnt 1
usb_dwc2_work_bh
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_hub_control dev 2, req 0xa300, value 0, index 1, langth 4
usb_hub_get_port_status dev 2, port 1, status 0x101, changed 0x0
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0,usb_dwc2_memory_write addr -1073200896 len 4
usb_dwc2_packet_done status USB_RET_SUCCESS actual 4 len 60 pcnt 0
PORTusb_dwc2_work_bh
 STATUS 1 01 01 00 00
CLEAR PORT FEATURE
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_hub_control dev 2, req 0x2301, value 16, index 1, langth 0
usb_hub_clear_port_feature dev 2, port 1, feature change-connection
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 0
usb_dwc2_memory_write addr -1073200896 len 0
usb_dwc2_packet_done status USB_RET_SUCCESS actual 0 len 64 pcnt 1
usb_dwc2_work_bh
SET PORT 1 RESET
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_hub_control dev 2, req 0x2303, value 4, index 1, langth 0
uusb_dwc2_memory_write addr -1073200896 len 0
usb_dwc2_packet_done status USB_RET_SUCCESS actual 0 len 64 pcnt 1
usb_dwc2_work_bh
CLEAR PORT FEATURE
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_hub_control dev 2, req 0x2301, value 20, index 1, langth 0
usb_hub_clear_pousb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560eb800 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 2
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560eb800 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_hub_control dev 2, req 0x2301, value 17, index 1, langth 0
usb_hub_clear_poGET DEVICE_DESCRIPTOR
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560e8c00 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560e8c00 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_desc_device dev 0 query device, len 64, ret 18
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560e8c00 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560e8c00 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 18
usb_dwc2_memory_write addr -1073200896 len 18
usb_dwc2_packet_done status USB_RET_SUCCESS actual 18 len 46 pcnt 0
usb_dwc2_work_bh
12 01 00 02 00 00 00 08 27 06 01 00 00 00 01 04 0b 01
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560e8c00 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560e8c00 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 0
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560e8c00 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560e8c00 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_set_addr dev 3
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866ad8, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 0
usb_dwc2_memory_write addr -1073200896 len 0
usb_dwc2_packet_done status USB_RET_SUCCESS actual 0 len 64 pcnt 1
usb_dwc2_work_bh
GET CONFIG_DESCRIPTOR
usb_dwc2_find_device 3
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 0 dev 0x7fd6560e8c00 pkt 0x7fd655866a28 ep 0
usb_dwc2_handle_packet ch 0 dev 0x7fd6560e8c00 pkt 0x7fd655866a28 ep 0 type Ctrl dir Out mps 64 len 8 pcnt 1
usb_dwc2_memory_read addr -1073200640 len 8
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866a28, state complete -> setup
usb_desc_config dev 3 query config 0, len 64, ret 34
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866a28, state setup -> complete
usb_dwc2_packet_status status USB_RET_SUCCESS len 8
usb_dwc2_packet_done status USB_RET_SUCCESS actual 8 len 0 pcnt 0
usb_dwc2_find_device 3
usb_dwc2_device_found device found on port 0
usb_dwc2_enable_chan ch 1 dev 0x7fd6560e8c00 pkt 0x7fd655866ad8 ep 0
usb_dwc2_handle_packet ch 1 dev 0x7fd6560e8c00 pkt 0x7fd655866ad8 ep 0 type Ctrl dir In mps 64 len 64 pcnt 1
usb_packet_state_change bus 0, port 1.1, ep 0, packet 0x7fd655866ad8, state complete -> setup
usb_packet_state09 02 22 00 01 01 08 a0 32 09 04 00 00 01 03 01 01 00
QEMU: Terminated
```
