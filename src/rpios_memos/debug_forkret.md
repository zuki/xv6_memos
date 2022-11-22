# forkret()前後のレジスタの内容

## usbあり

```
pgdir=0xffff000038bfb000, va=0xffff000038bfc000		//　icode

(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0x3b3fe320	0xffff0000	0x00000000	0x00000000	// この4行が壊れている
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe	// これ以下は正しい
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
```

### watch *0xffff000038bfc000

```
(gdb) c
Continuing.
[Switching to Thread 1.3]

Thread 3 hit Hardware watchpoint 1: *0xffff000038bfc000

Old value = <unreadable>
New value = 952086528	(0x38bfb000)
kfree (va=0xffff000038bfc000) at kern/mm.c:156
156	    kmem.freelist = r;
(gdb) c
Continuing.

Thread 3 hit Hardware watchpoint 1: *0xffff000038bfc000

Old value = 952086528	(0x38bfb000)
New value = 952086696	(0x38bfb0a8)
0xffff0000000ac4dc in memmove (dst=0xffff000038bfc000, src=0xffff000000086840 <icode>, n=47) at inc/string.h:26
26	        while (n-- > 0) *d++ = *s++;
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0x38bfb0a8	0xffff0000	0x00000000	0x00000000
0xffff000038bfc010:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff000038bfc020:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) p/x d
$1 = 0xffff000038bfc001
(gdb) p/x *d
$2 = 0xb0
(gdb) p/x sd
No symbol "sd" in current context.
(gdb) p/x *s
$3 = 0x1b
(gdb) s

Thread 3 hit Hardware watchpoint 1: *0xffff000038bfc000

Old value = 952086696	(0x38bfb0a8)
New value = 952048552	(0x38bf1ba8)
0xffff0000000ac4dc in memmove (dst=0xffff000038bfc000, src=0xffff000000086840 <icode>, n=46) at inc/string.h:26
26	        while (n-- > 0) *d++ = *s++;
(gdb) 

Thread 3 hit Hardware watchpoint 1: *0xffff000038bfc000

Old value = 952048552	(0x38bf1ba8)
New value = 947919784	(0x3880aba8)
0xffff0000000ac4dc in memmove (dst=0xffff000038bfc000, src=0xffff000000086840 <icode>, n=45) at inc/string.h:26
26	        while (n-- > 0) *d++ = *s++;
(gdb) 

Thread 3 hit Hardware watchpoint 1: *0xffff000038bfc000

Old value = 947919784	(0x3880aba8)
New value = -763356248	(0xd2801ba8)
0xffff0000000ac4dc in memmove (dst=0xffff000038bfc000, src=0xffff000000086840 <icode>, n=44) at inc/string.h:26
26	        while (n-- > 0) *d++ = *s++;
(gdb) 
28	    return dst;
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0xd2801ba8	0x100000e0	0xd2800001	0xd2800002
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) c
Continuing.
[Switching to Thread 1.4]

Thread 4 hit Breakpoint 2, forkret () at kern/proc.c:235
235	        info("usbhc inited");
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0x3b3fe320	0xffff0000	0x00000000	0x00000000	// 0xffff00003b3fe320
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) x/-16wx 0xffff000038bfc000
0xffff000038bfbfc0:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff000038bfbfd0:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff000038bfbfe0:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff000038bfbff0:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) x/16wx 0xffff000000086840
0xffff000000086840 <icode>:	0xd2801ba8	0x100000e0	0xd2800001	0xd2800002
0xffff000000086850 <icode+16>:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000000086860 <init>:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000000086870 <cpuid>:	0xd10043ff	0xd53800a0	0xf90007e0	0xf94007e0
(gdb) disas 0x3b3fe320
No function contains specified address.
(gdb) q
```

## void dw2_hc()の中

(struct dw2_hc)->hublistはstruct list_headへのポインタだがallocせずに使用していた。
list_init(self->hublist)の後で次のように16バイトが初期化された。

```
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
```

(struct dw2_hc)->hublistをポインタではなく変数とし、list_init(&self->hublist)としたら
ここでは次のようにクリアされなくなった。

```
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0xd2801ba8	0x100000e0	0xd2800001	0xd2800002
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
```

ただし、dw2hc_init()後は次のように先頭8バイトが何らかのポインタに置き換わっている。

```
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0x3b3fe310	0xffff0000	0xd2800001	0xd2800002
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
```

## standardhub.c -> dev_name_service_add_dev()

```
(gdb) 
54	    info->next = self->list;
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0xd2801ba8	0x100000e0	0xd2800001	0xd2800002
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) p/x info->next
$8 = 0x0
(gdb) p/x self->list
$9 = 0x100000e0d2801ba8
(gdb) n
55	    self->list = info;
(gdb) p/x info
$10 = 0xffff00003b3fe9b0		// 上書きされた値
(gdb) p/x self->list		
$11 = 0x100000e0d2801ba8		// 0xffff000038bfc000の先頭8バイトの値
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0xd2801ba8	0x100000e0	0xd2800001	0xd2800002
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) p/x info->next
$12 = 0x100000e0d2801ba8
(gdb) n
56	}
(gdb) p/x info->next
$13 = 0x100000e0d2801ba8
(gdb) x/16wx 0xffff000038bfc000
0xffff000038bfc000:	0x3b3fe9b0	0xffff0000	0xd2800001	0xd2800002
0xffff000038bfc010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff000038bfc020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff000038bfc030:	0x00000000	0x00000000	0x00000000	0x00000000
```

```
void dev_name_service_add_dev(dev_ns_t *self, const char *name, void *dev, boolean blkdev)
{
    dev_info_t *info = (dev_info_t *) kmalloc(sizeof(dev_info_t));
    info->name = (char *) kmalloc(strlen(name)+1);
    safestrcpy(info->name, name, strlen(name)+1);
    info->dev = dev;
    info->blkdev = blkdev;
    info->next = self->list;
    self->list = info;
}
```

dev_name_serviceのオブジェクトが作成されていなかった。self=0だった

```
qemu-system-aarch64 -M raspi3b -nographic -serial null -serial mon:stdio -drive file=obj/sd.img,if=sd,format=raw -netdev user,id=net0,hostfwd=tcp::8080-:80 -device usb-net,netdev=net0  -kernel obj/kernel8.img -S -gdb tcp::1234
[2]free_range: 0xffff00000034b000 ~ 0xffff00003b400000, 241845 pages
[2]console_init: devsw[1].termios: 0xffff00003b3ff000
[2]clock_init: clock init ok
[2]rand_init: rand_init ok
[2]init_vfssw: init_vfssw ok
mountinit ok 
[2]install_rootfs: install_rootfs ok
pagecache_init ok
[2]proc_initx: pgdir=0xffff000038bfb000, va=0xffff000038bfc000
[2]timer_init: timerfreq = 0x3b9aca0
[1]timer_init: timerfreq = 0x3b9aca0
[3]timer_init: timerfreq = 0x3b9aca0
[2]main: cpu 2 init finished
[3]main: cpu 3 init finished
[1]main: cpu 1 init finished
[0]timer_init: timerfreq = 0x3b9aca0
[0]main: cpu 0 init finished
[2]emmc_card_init: poweron
[2]emmc_issue_command_int: rrror occured whilst waiting for command complete interrupt
[1]emmc_card_reset: found valid version 2.00 SD card
[1]dev_init: partition[0]: LBA=0x800, #SECS=0x20000
[1]dev_init: partition[1]: LBA=0x20800, #SECS=0xf0000
[1]dev_init: partition[2]: LBA=0x110800, #SECS=0xef800
[1]iinit: sb: size 100000 nblocks 99835 ninodes 1024 nlog 126 logstart 2 inodestart 128 bmapstart 161
[0]initlog: init log ok
[0]usb_dev_init: Device ven409-55aa, dev9-0-0 found
[0]usb_dev_init: Product: QEMU QEMU USB Hub
[0]usb_func_get_if_name: func name=int9-0-0
[0]usb_dev_init: Interface int9-0-0 found
[0]usb_devfactory_get_device: Using device/interface int9-0-0
[2]usb_stdhub_enumerate_ports: start
[3]usb_dev_init: Device ven525-a4a2, dev2-0-0 found
[1]usb_dev_init: Product: QEMU RNDIS/QEMU USB Network Device
[1]usb_func_get_if_name: func name=int2-6-0
[1]usb_dev_init: Interface int2-6-0 found
[1]usb_devfactory_get_device: Using device/interface int2-6-0
[1]usb_func_get_if_name: func name=inta-0-0
[1]usb_dev_init: Interface inta-0-0 found
[1]usb_dev_init: Function is not supported
[1]usb_func_get_if_name: func name=inta-0-0
[1]usb_dev_init: Interface inta-0-0 found
[1]usb_dev_init: Function is not supported
[2]cdcether_configure: MAC address is 40:54:0:12:34:57
[2]usb_dev_config: 0 is ok
[2]usb_stdhub_enumerate_ports: Port 1: Device configured
[3]usb_dev_config: 0 is ok
[3]dw2_rport_init: Device configured
[3]usbhc_init: dw2hc initialized

[3]forkret: usbhc inited
[1]mount: source: /dev/sdc3, target: /mnt, type: ext2, flags: 0x0

[1]fileopen: cant namei /etc/issue
Welcome to xv6 2022-06-26 (musl) mini tty

mini login: root
Password:
[2]fileopen: cant namei /etc/profile
[3]fileopen: cant namei /.profile
# ls
bin  dev  etc  home  lib  mnt  test.txt  usr
# ls -l
total 5
drwxrwxr-x 1 root root 1664 Nov 22  2022 bin
drwxrwxr-x 1 root root  384 Nov 22  2022 dev
drwxrwxr-x 1 root root  320 Nov 22  2022 etc
drwxrwxr-x 1 root root  192 Nov 22  2022 home
drwxrwxrwx 1 root root  256 Nov 22  2022 lib
drwxrwxr-x 3 root root 4096 Nov 21  2022 mnt
-rwxr-xr-x 1 root root   94 Nov 22  2022 test.txt
drwxrwxr-x 1 root root  256 Nov 22  2022 usr
```


## usbなし

```
pgidr=0xffff00003bbfd000, va=0xffff00003bbfe000		// icode
pgidr=0xffff00003bbf7000, va=0xffff00003bbf8000		// ispin
pgidr=0xffff00003bbf3000, va=0xffff00003bbf4000		// ispin
pgidr=0xffff00003bbeb000, va=0xffff00003bbec000		// ispin
pgidr=0xffff00003bbe5000, va=0xffff00003bbe6000		// ispin


220	        cprintf("fret\n");
(gdb) p/x 0xffff00003bbe6000
$1 = 0xffff00003bbe6000
(gdb) disas 0xffff00003bbe6000
No function contains specified address.
(gdb) x/16wx 0xffff00003bbfe000
0xffff00003bbfe000:	0xd2801ba8	0x100000e0	0xd2800001	0xd2800002
0xffff00003bbfe010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff00003bbfe020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff00003bbfe030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) x/16wx 0xffff00003bbe6000
0xffff00003bbe6000:	0xd2800f88	0xd4000001	0x17fffffe	0x6e69622f
0xffff00003bbe6010:	0x696e692f	0x00000074	0xd503201f	0x00000000
0xffff00003bbe6020:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff00003bbe6030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) x/16wx 0xffff00003bbe6000
0xffff00003bbe6000:	0xd2800f88	0xd4000001	0x17fffffe	0x6e69622f
0xffff00003bbe6010:	0x696e692f	0x00000074	0xd503201f	0x00000000
0xffff00003bbe6020:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff00003bbe6030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) x/16wx 0xffff00003bbf8000
0xffff00003bbf8000:	0xd2800f88	0xd4000001	0x17fffffe	0x6e69622f
0xffff00003bbf8010:	0x696e692f	0x00000074	0xd503201f	0x00000000
0xffff00003bbf8020:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff00003bbf8030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) x/16wx 0xffff00003bbf4000
0xffff00003bbf4000:	0xd2800f88	0xd4000001	0x17fffffe	0x6e69622f
0xffff00003bbf4010:	0x696e692f	0x00000074	0xd503201f	0x00000000
0xffff00003bbf4020:	0x00000000	0x00000000	0x00000000	0x00000000
0xffff00003bbf4030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) i r
x0             0xffff0000001146e0  -281474975578400
x1             0xffff000000126248  -281474975505848
x2             0xffff000000126268  -281474975505816
x3             0x0                 0
x4             0x20802             133122
x5             0x2                 2
x6             0x80                128
x7             0x181               385
x8             0x0                 0
x9             0xffff00000007ff50  -281474976186544
x10            0x0                 0
x11            0x0                 0
x12            0x0                 0
x13            0x0                 0
x14            0x0                 0
x15            0xffff00000008a3a8  -281474976144472
x16            0x0                 0
x17            0x0                 0
x18            0x0                 0
x19            0xffff0000000e26b8  -281474975783240
x20            0x0                 0
x21            0x0                 0
x22            0x0                 0
x23            0x0                 0
x24            0x0                 0
x25            0x0                 0
x26            0x0                 0
x27            0x0                 0
x28            0x0                 0
x29            0xffff00003bbffeb0  -281473974272336
x30            0xffff00000008a40c  -281474976144372
sp             0xffff00003bbffeb0  0xffff00003bbffeb0
pc             0xffff00000008a40c  0xffff00000008a40c <forkret+100>
cpsr           0x200001c5          536871365
(gdb) 
(gdb) x/16gx 0xffff00003bbffeb0
0xffff00003bbffeb0:	0x0000000000000000	0xffff00000008dde0
0xffff00003bbffec0:	0x0000000000000000	0x0000000000000000
0xffff00003bbffed0:	0x0000000000000000	0x0000000000000000
0xffff00003bbffee0:	0x0000000000000000	0x0000000000000000
0xffff00003bbffef0:	0x0000000000000000	0x0000000000000000
0xffff00003bbfff00:	0x0000000000000000	0x0000000000000000
0xffff00003bbfff10:	0x0000000000000000	0x0000000000000000
0xffff00003bbfff20:	0x0000000000000000	0x0000000000000000
(gdb) x/-16gx 0xffff00003bbffeb0
0xffff00003bbffe30:	0xffff00003bbffe60	0xffff000000088850
0xffff00003bbffe40:	0xffff00003bbffe70	0xffff000000126038
0xffff00003bbffe50:	0xffff000000126028	0x000000003bbffe20
0xffff00003bbffe60:	0xffff00003bbffe70	0xffff0000000885a4
0xffff00003bbffe70:	0xffff00003bbffeb0	0xffff00000008a40c
0xffff00003bbffe80:	0xffff00003bbffeb0	0x000000010008a404
0xffff00003bbffe90:	0x000c32bb000c3500	0x0000007e00000400
0xffff00003bbffea0:	0x0000008000000002	0x0000040000000181
(gdb) 
0xffff00003bbffe30:	0xffff00003bbffe60
(gdb) x/-16gx 0xffff00003bbffe20
0xffff00003bbffda0:	0xffff00003bbffdc0	0xffff0000001137c8
0xffff00003bbffdb0:	0xffff00003bbffdd0	0xffff00000008a740
0xffff00003bbffdc0:	0xffff0000001136a0	0xffff0000001146c8
0xffff00003bbffdd0:	0xffff00003bbffdf0	0xffff000000126268
0xffff00003bbffde0:	0xffff000000126008	0xffff000000126248
0xffff00003bbffdf0:	0xffff00003bbffe10	0xffff00000008cf48
0xffff00003bbffe00:	0xffff000000126248	0xffff0000001146e0
0xffff00003bbffe10:	0xffff00003bbffe30	0xffff000000088824
(gdb) x/-16gx 0xffff00003bbffe30
0xffff00003bbffdb0:	0xffff00003bbffdd0	0xffff00000008a740
0xffff00003bbffdc0:	0xffff0000001136a0	0xffff0000001146c8
0xffff00003bbffdd0:	0xffff00003bbffdf0	0xffff000000126268
0xffff00003bbffde0:	0xffff000000126008	0xffff000000126248
0xffff00003bbffdf0:	0xffff00003bbffe10	0xffff00000008cf48
0xffff00003bbffe00:	0xffff000000126248	0xffff0000001146e0
0xffff00003bbffe10:	0xffff00003bbffe30	0xffff000000088824
0xffff00003bbffe20:	0xffff00003bbffe40	0xffff000000126028
(gdb) x/-16gx 0xffff00003bbffeb0
0xffff00003bbffe30:	0xffff00003bbffe60	0xffff000000088850
0xffff00003bbffe40:	0xffff00003bbffe70	0xffff000000126038
0xffff00003bbffe50:	0xffff000000126028	0x000000003bbffe20
0xffff00003bbffe60:	0xffff00003bbffe70	0xffff0000000885a4
0xffff00003bbffe70:	0xffff00003bbffeb0	0xffff00000008a40c
0xffff00003bbffe80:	0xffff00003bbffeb0	0x000000010008a404
0xffff00003bbffe90:	0x000c32bb000c3500	0x0000007e00000400
0xffff00003bbffea0:	0x0000008000000002	0x0000040000000181
(gdb) x/-16gx 0xffff00003bbffe30
0xffff00003bbffdb0:	0xffff00003bbffdd0	0xffff00000008a740
0xffff00003bbffdc0:	0xffff0000001136a0	0xffff0000001146c8
0xffff00003bbffdd0:	0xffff00003bbffdf0	0xffff000000126268
0xffff00003bbffde0:	0xffff000000126008	0xffff000000126248
0xffff00003bbffdf0:	0xffff00003bbffe10	0xffff00000008cf48
0xffff00003bbffe00:	0xffff000000126248	0xffff0000001146e0
0xffff00003bbffe10:	0xffff00003bbffe30	0xffff000000088824
0xffff00003bbffe20:	0xffff00003bbffe40	0xffff000000126028
(gdb) 
0xffff00003bbffdb0:	0xffff00003bbffdd0
(gdb) n
trapret () at kern/trapasm.S:41
41	    ldp     x9, x10, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:42
42	    ldp     x11, x12, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:44
44	    msr     spsr_el1, x9
(gdb) 
45	    msr     elr_el1, x10
(gdb) 
46	    msr     sp_el0, x11
(gdb) 
47	    msr     tpidr_el0, x12
(gdb) 
49	    ldp     x0, x1, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:50
50	    ldp     x2, x3, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:51
51	    ldp     x4, x5, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:52
52	    ldp     x6, x7, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:53
53	    ldp     x8, x9, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:54
54	    ldp     x10, x11, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:55
55	    ldp     x12, x13, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:56
56	    ldp     x14, x15, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:57
57	    ldp     x16, x17, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:58
58	    ldp     x18, x19, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:59
59	    ldp     x20, x21, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:60
60	    ldp     x22, x23, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:61
61	    ldp     x24, x25, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:62
62	    ldp     x26, x27, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:63
63	    ldp     x28, x29, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:64
64	    ldp     x30, xzr, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:66
66	    ldr     q0, [sp], #16
(gdb) 
trapret () at kern/trapasm.S:68
68	    ic      iallu
(gdb) 
69	    dsb     sy
(gdb) 
70	    isb
(gdb) 
71	    eret
(gdb) i r
x0             0x0                 0
x1             0x0                 0
x2             0x0                 0
x3             0x0                 0
x4             0x0                 0
x5             0x0                 0
x6             0x0                 0
x7             0x0                 0
x8             0x0                 0
x9             0x0                 0
x10            0x0                 0
x11            0x0                 0
x12            0x0                 0
x13            0x0                 0
x14            0x0                 0
x15            0x0                 0
x16            0x0                 0
x17            0x0                 0
x18            0x0                 0
x19            0x0                 0
x20            0x0                 0
x21            0x0                 0
x22            0x0                 0
x23            0x0                 0
x24            0x0                 0
x25            0x0                 0
x26            0x0                 0
x27            0x0                 0
x28            0x0                 0
x29            0x0                 0
x30            0x0                 0
sp             0xffff00003bc00000  0xffff00003bc00000
pc             0xffff00000008de48  0xffff00000008de48 <trapret+104>
cpsr           0x200001c5          536871365
...
(gdb) disas
Dump of assembler code for function trapret:
   0xffff00000008dde0 <+0>:	ldp	x9, x10, [sp], #16
   0xffff00000008dde4 <+4>:	ldp	x11, x12, [sp], #16
   0xffff00000008dde8 <+8>:	msr	spsr_el1, x9
   0xffff00000008ddec <+12>:	msr	elr_el1, x10
   0xffff00000008ddf0 <+16>:	msr	sp_el0, x11
   0xffff00000008ddf4 <+20>:	msr	tpidr_el0, x12
   0xffff00000008ddf8 <+24>:	ldp	x0, x1, [sp], #16
   0xffff00000008ddfc <+28>:	ldp	x2, x3, [sp], #16
   0xffff00000008de00 <+32>:	ldp	x4, x5, [sp], #16
   0xffff00000008de04 <+36>:	ldp	x6, x7, [sp], #16
   0xffff00000008de08 <+40>:	ldp	x8, x9, [sp], #16
   0xffff00000008de0c <+44>:	ldp	x10, x11, [sp], #16
   0xffff00000008de10 <+48>:	ldp	x12, x13, [sp], #16
   0xffff00000008de14 <+52>:	ldp	x14, x15, [sp], #16
   0xffff00000008de18 <+56>:	ldp	x16, x17, [sp], #16
   0xffff00000008de1c <+60>:	ldp	x18, x19, [sp], #16
   0xffff00000008de20 <+64>:	ldp	x20, x21, [sp], #16
   0xffff00000008de24 <+68>:	ldp	x22, x23, [sp], #16
   0xffff00000008de28 <+72>:	ldp	x24, x25, [sp], #16
   0xffff00000008de2c <+76>:	ldp	x26, x27, [sp], #16
   0xffff00000008de30 <+80>:	ldp	x28, x29, [sp], #16
   0xffff00000008de34 <+84>:	ldp	x30, xzr, [sp], #16
   0xffff00000008de38 <+88>:	ldr	q0, [sp], #16
   0xffff00000008de3c <+92>:	ic	iallu
   0xffff00000008de40 <+96>:	dsb	sy
   0xffff00000008de44 <+100>:	isb
=> 0xffff00000008de48 <+104>:	eret
End of assembler dump.
(gdb) x/16gx 0xffff00003bc00000		// sp
0xffff00003bc00000:	0xffff00003bbff000	0x0000000000000000
0xffff00003bc00010:	0x0000000000000000	0x0000000000000000
0xffff00003bc00020:	0x0000000000000000	0x0000000000000000
0xffff00003bc00030:	0x0000000000000000	0x0000000000000000
0xffff00003bc00040:	0x0000000000000000	0x0000000000000000
0xffff00003bc00050:	0x0000000000000000	0x0000000000000000
0xffff00003bc00060:	0x0000000000000000	0x0000000000000000
0xffff00003bc00070:	0x0000000000000000	0x0000000000000000
(gdb) x/-16gx 0xffff00003bc00000	//
0xffff00003bbfff80:	0x0000000000000000	0x0000000000000000
0xffff00003bbfff90:	0x0000000000000000	0x0000000000000000
0xffff00003bbfffa0:	0x0000000000000000	0x0000000000000000
0xffff00003bbfffb0:	0x0000000000000000	0x0000000000000000
0xffff00003bbfffc0:	0x0000000000000000	0x0000000000000000
0xffff00003bbfffd0:	0x0000000000000000	0x0000000000000000
0xffff00003bbfffe0:	0x0000000000000000	0x0000000000000000
0xffff00003bbffff0:	0x0000000000000000	0x0000000000000000
(gdb) x/16gx 0xffff00003bbff000		// [sp]
0xffff00003bbff000:	0xffff00003bbfe000	0x0000000000000000
0xffff00003bbff010:	0x0000000000000000	0x0000000000000000
0xffff00003bbff020:	0x0000000000000000	0x0000000000000000
0xffff00003bbff030:	0x0000000000000000	0x0000000000000000
0xffff00003bbff040:	0x0000000000000000	0x0000000000000000
0xffff00003bbff050:	0x0000000000000000	0x0000000000000000
0xffff00003bbff060:	0x0000000000000000	0x0000000000000000
0xffff00003bbff070:	0x0000000000000000	0x0000000000000000
(gdb) x/16gx 0xffff00003bbfe000		// [[sp]] => icode
0xffff00003bbfe000:	0x100000e0d2801ba8	0xd2800002d2800001
0xffff00003bbfe010:	0xd2800f88d4000001	0x17fffffed4000001
0xffff00003bbfe020:	0x696e692f6e69622f	0xd503201f00000074
0xffff00003bbfe030:	0x0000000000000000	0x0000000000000000
0xffff00003bbfe040:	0x0000000000000000	0x0000000000000000
0xffff00003bbfe050:	0x0000000000000000	0x0000000000000000
0xffff00003bbfe060:	0x0000000000000000	0x0000000000000000
0xffff00003bbfe070:	0x0000000000000000	0x0000000000000000
(gdb) x/16wx 0xffff00003bbfe000
0xffff00003bbfe000:	0xd2801ba8	0x100000e0	0xd2800001	0xd2800002
0xffff00003bbfe010:	0xd4000001	0xd2800f88	0xd4000001	0x17fffffe
0xffff00003bbfe020:	0x6e69622f	0x696e692f	0x00000074	0xd503201f
0xffff00003bbfe030:	0x00000000	0x00000000	0x00000000	0x00000000
(gdb) c
Continuing.
[Inferior 1 (process 1) exited normally]
(gdb) q
```