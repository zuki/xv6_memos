# 実機で動かない

```
>?L?+?W??k׋?֕?+??"????m?깢?ɵ??????????????2????R??
```

## irq_init()の追加行をコメントアウト

```
>?L?+?W??k׋?֕?+??"????m?깢?ɵ??????????????2????R??[0]clock_init: clock init o  k
[0]rand_init: rand_init ok
[0]init_vfssw: init_vfssw ok
mountinit ok
[0]install_rootfs: install_rootfs ok
pagecache_init ok
[0]timer_init: timerfreq = 0x124f800
[3]timer_init: timerfreq = 0x124f800
[2]timer_init: timerfreq = 0x124f800
[0]main: cpu 0 init finished
[3]main: cpu 3 init finished
[2]main: cpu 2 init finished
[1]timer_init: timerfreq = 0x124f800
[0]emmc_card_init: poweron
[1]main: cpu 1 init finished
[0]emmc_issue_command_int: rrror occured whilst waiting for command complete int
[0]emmc_card_reset: found valid version 4.xx SD card
[0]dev_init: partition[0]: LBA=0x800, #SECS=0x20000
[0]dev_init: partition[1]: LBA=0x20800, #SECS=0xf0000
[0]dev_init: partition[2]: LBA=0x110800, #SECS=0xef800
[0]iinit: sb: size 100000 nblocks 99835 ninodes 1024 nlog 126 logstart 2 inodes1
[0]initlog: init log ok
							// ここでストール
```

## printcheck

```
[0]dw2_hc: dw2_hc created
[0]dw2_hc_enable_rport: enabled rootport
[0]dw2_rport_init: 1
[0]dw2_rport_init: 2
[0]dw2_hc_get_desc: call
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
```

## clean_and_invalidate_data_cache_range(uint64_t addr, uint64_t length)にわたすaddrをvaに変更した

```diff
$ git diff kern/usb/dw2xferstagedata.c
diff --git a/kern/usb/dw2xferstagedata.c b/kern/usb/dw2xferstagedata.c
index ce33c55..5708b4d 100644
--- a/kern/usb/dw2xferstagedata.c
+++ b/kern/usb/dw2xferstagedata.c
@@ -316,9 +316,9 @@ boolean dw2_xfer_stagedata_is_ststage(dw2_xfer_stagedata_t *self)
     return self->ststage;
 }

-uint32_t dw2_xfer_stagedata_get_dmaaddr(dw2_xfer_stagedata_t *self)
+uint64_t dw2_xfer_stagedata_get_dmaaddr(dw2_xfer_stagedata_t *self)
 {
-    return (uint32_t)(uintptr_t)self->buffp;
+    return (uint64_t)(uintptr_t)self->buffp;
 }

 uint32_t dw2_xfer_stagedata_get_bpt(dw2_xfer_stagedata_t *self)
```

```
[1]dw2_hc: dw2_hc created
[1]dw2_hc_enable_rport: enabled rootport
[1]dw2_rport_init: 1
[1]dw2_rport_init: 2
[1]dw2_hc_get_desc: call
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[1]clean_and_invalidate_data_cache_range: 2											// ここは通過した
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00100008
[1]dw2_hc_start_channel: 10-10
[e]n/2sb/_sthrd.c:a16e :ssertion failed.

[e]n/2sb/_sthrd.c:a16e :ssertion failed.
```

## dw2_hc_channel_intr_hdl()でassert()を修正

```diff
 void dw2_hc_channel_intr_hdl(dw2_hc_t *self, unsigned channel)
 {
+    info("called");
     dw2_xfer_stagedata_t *stdata = self->stdata[channel];
     dw2_fsched_t *fsched = dw2_xfer_stagedata_get_fsched(stdata);
-    assert(fsched != 0);									// これは不要（self->fused == falseの場合は0）
+    info("stdata=0x%p, fsched=0x%p", stdata, fsched);
     usb_req_t *urb = stdata->urb;
+    info("urb=0x%p, rpenabled=0x%x", urb, self->rpenabled);
     assert(urb != 0);

     if (!self->rpenabled) {
```

### 実行結果

```
>???+?W??k׋?֕?+??"????m?깢?ɵ??????????????2????R??[0]clock_init: clock init ok
[0]rand_init: rand_init ok
[0]init_vfssw: init_vfssw ok
mountinit ok
[0]install_rootfs: install_rootfs ok
pagecache_init ok
[0]timer_init: timerfreq = 0x124f800
[2]timer_init: timerfreq = 0x124f800
[1]timer_init: timerfreq = 0x124f800
[2]main: cpu 2 init finished
[1]main: cpu 1 init finished
[3]timer_init: timerfreq = 0x124f800
[0]main: cpu 0 init finished
[2]emmc_card_init: poweron
[3]main: cpu 3 init finished
[2]emmc_issue_command_int: rrror occured whilst waiting for command complete interrupt
[2]emmc_card_reset: found valid version 4.xx SD card
[2]dev_init: partition[0]: LBA=0x800, #SECS=0x20000
[2]dev_init: partition[1]: LBA=0x20800, #SECS=0xf0000
[2]dev_init: partition[2]: LBA=0x110800, #SECS=0xef800
[2]iinit: sb: size 100000 nblocks 99835 ninodes 1024 nlog 126 logstart 2 inodestart 128 bmapstart 161
[2]initlog: init log ok
[2]dw2_hc: dw2_hc created
[2]dw2_hc_enable_rport: enabled rootport
[2]dw2_rport_init: 1
[2]dw2_rport_init: 2
[2]dw2_hc_get_desc: call
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[2]dw2_xfer_stagedata: no use fsched
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feba0
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00100008
[2]dw2_hc_start_channel: 10-10: fsched=0x0
[2]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[2]dw2_hc_start_channel: char=0x80100008
[2]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[1]dw2_xfer_stagedata: no use fsched
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feba0
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fecb0, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00108008
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80108008
[1]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fecb0, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[1]dw2_xfer_stagedata: no use fsched
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feba0
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00100008
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80100008
[1]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]usb_dev_init: 1
[3]dw2_hc_get_desc: call
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[3]dw2_xfer_stagedata: no use fsched
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feba0
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00100040
[3]dw2_hc_start_channel: 10-10: fsched=0x0
[3]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[3]dw2_hc_start_channel: char=0x80100040
[3]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[1]dw2_xfer_stagedata: no use fsched
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feba0
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fecb0, length=0x0000000000000012
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00108040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80108040
[1]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fecb0, length=0x0000000000000012
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[1]dw2_xfer_stagedata: no use fsched
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feba0
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00100040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80100040
[1]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]usb_dev_init: 2

== usbdevice: 0xffff00003b3fecb0 (0xffff00003b3fecb0) - 0xffff00003b3fecc2: size=0x12 ==

ffff00003b3fecb0: 1201 0002 0900 0240 2404 1425 b30b 0000 .......@$..%....
ffff00003b3fecc0: 0000                                    ..
[3]usb_dev_init: 3: addr=1
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[3]dw2_xfer_stagedata: no use fsched
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feba0
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd60, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00100040
[3]dw2_hc_start_channel: 10-10: fsched=0x0
[3]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[3]dw2_hc_start_channel: char=0x80100040
[3]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd18, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd60, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[3]dw2_xfer_stagedata: no use fsched
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feba0
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00108040
[3]dw2_hc_start_channel: 10-10: fsched=0x0
[3]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[3]dw2_hc_start_channel: char=0x80108040
[3]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd18, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]usb_dev_init: 4
[3]usb_dev_init: 5.2: cindex=0
[3]dw2_hc_get_desc: call
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb80
[3]dw2_xfer_stagedata: no use fsched
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feb80
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500040
[3]dw2_hc_start_channel: 10-10: fsched=0x0
[3]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[3]dw2_hc_start_channel: char=0x80500040
[3]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb80, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb80
[1]dw2_xfer_stagedata: no use fsched
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feb80
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec90, length=0x0000000000000009
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80508040
[1]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb80, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec90, length=0x0000000000000009
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb80
[2]dw2_xfer_stagedata: no use fsched
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feb80
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3febe4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500040
[2]dw2_hc_start_channel: 10-10: fsched=0x0
[2]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[2]dw2_hc_start_channel: char=0x80500040
[2]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb80, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3febe4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]usb_dev_init: 5.3
[1]usb_dev_init: 5.4
[1]dw2_hc_get_desc: call
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb60
[1]dw2_xfer_stagedata: no use fsched
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feb60
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80500040
[1]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb60, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb60
[1]dw2_xfer_stagedata: no use fsched
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feb60
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec70, length=0x0000000000000029
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80508040
[1]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb60, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec70, length=0x0000000000000029
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb60
[2]dw2_xfer_stagedata: no use fsched
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: self->stdata[0]=0xffff00003b3feb60
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3febc4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500040
[2]dw2_hc_start_channel: 10-10: fsched=0x0
[2]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[2]dw2_hc_start_channel: char=0x80500040
[2]dw2_hc_start_channel: 10-13
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb60, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3febc4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]usb_dev_init: 6

== fromdevice: 0xffff00003b3fec70 (0xffff00003b3fec70) - 0xffff00003b3fec99: size=0x29 ==

ffff00003b3fec70: 0902 2900 0101 00e0 0109 0400 0001 0900 ..).............
ffff00003b3fec80: 5067 0c00 0000 ffff 0200 0000 0000 0000 Pg.
ffff00003b3fec90: 0902 2900 0101 00e0 0112                ..
																// ここでストール
```

## qemu

```
[3]dw2_hc: dw2_hc created
[3]dw2_hc_enable_rport: enabled rootport
[3]dw2_rport_init: 1
[3]dw2_rport_init: 2
[3]dw2_hc_get_desc: call
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00100008
[3]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fecb0, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00108008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fecb0, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec04, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00100008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec04, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]usb_dev_init: 1
[3]dw2_hc_get_desc: call
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00100008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fecb0, length=0x0000000000000012
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00108008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fecb0, length=0x0000000000000012
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec04, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00100008
[0]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec04, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[2]usb_dev_init: 2

== PRINT (usbdevice) 0xffff00003b3fec00 (0xffff00003b3fecb0) - 0xffff00003b3fec12 ==

ffff00003b3fec00: 1201 1001 0900 0008 0904 aa55 0101 0102 ...........U....
[2]usb_dev_init: 3: addr=1
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd60, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00100008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd60, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec04, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00108008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec04, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]usb_dev_init: 4
[2]usb_dev_init: 5.2: cindex=0
[2]dw2_hc_get_desc: call
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec90, length=0x0000000000000009
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec90, length=0x0000000000000009
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3febe4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3febe4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]usb_dev_init: 5.3
[3]usb_dev_init: 5.4
[3]dw2_hc_get_desc: call
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd50, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec80, length=0x0000000000000019
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00508008
[2]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fec80, length=0x0000000000000019
[2]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3febd4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3febd4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]usb_dev_init: 6

== PRINT (fromdevice) 0xffff00003b3fec00 (0xffff00003b3fec80) - 0xffff00003b3fec19 ==

ffff00003b3fec00: 0902 1900 0101 00e0 0009 0400 0001 0900 ................
[3]usb_dev_init: 7
[3]usb_dev_init: Device ven409-55aa, dev9-0-0 found
[3]dw2_hc_get_desc: call
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd10, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd10, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fea50, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fea50, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9a4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00500008
[0]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9a4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd00, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd00, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fea50, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fea50, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9a4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9a4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd00, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd00, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fea50, length=0x000000000000000a
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fea50, length=0x000000000000000a
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9a4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9a4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_get_desc: call
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd10, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00500008
[0]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd10, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe920, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe920, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe874, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe874, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd00, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd00, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3feb10, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00508008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3feb10, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe894, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe894, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd00, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd00, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe910, length=0x000000000000001a
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00508008
[0]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe910, length=0x000000000000001a
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe864, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe864, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]usb_dev_init: Product: QEMU QEMU USB Hub
[3]usb_dev_init: 8
[3]usb_func_get_if_name: func name=int9-0-0
[3]usb_dev_init: Interface int9-0-0 found
[3]usb_devfactory_get_device: Using device/interface int9-0-0
[3]usb_dev_init: 9
[3]dw2_rport_init: 3
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdda0, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdda0, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00508008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_get_desc: call
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd40, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd40, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9e0, length=0x0000000000000009
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9e0, length=0x0000000000000009
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00500008
[0]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe950, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00508008
[0]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe950, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 10-13
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe950, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe950, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe724, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_get_desc: call
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc70, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00100008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc70, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe6a0, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00108008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe6a0, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00100008
[2]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[0]usb_dev_init: 1
[0]dw2_hc_get_desc: call
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc70, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00100040
[0]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc70, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe6a0, length=0x0000000000000012
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00108040
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe6a0, length=0x0000000000000012
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00100040
[2]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[1]usb_dev_init: 2

== PRINT (usbdevice) 0xffff00003b3fe600 (0xffff00003b3fe6a0) - 0xffff00003b3fe612 ==

ffff00003b3fe600: 1201 0002 0200 0040 2505 a2a4 0000 0102 .......@%.......
[1]usb_dev_init: 3: addr=2
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc80, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00100040
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc80, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00108040
[1]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[1]usb_dev_init: 4
[1]usb_dev_init: 5.2: cindex=1
[1]dw2_hc_get_desc: call
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc70, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00900040
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc70, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 10-13
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9c0, length=0x0000000000000009
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00908040
[1]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9c0, length=0x0000000000000009
[2]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00900040
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]usb_dev_init: 5.3
[2]usb_dev_init: 5.4
[2]dw2_hc_get_desc: call
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc70, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00900040
[2]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc70, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fead0, length=0x0000000000000050
[2]clean_and_invalidate_data_cache_range: 1-1: addr=0x000000003b3feb10, length=0x0000000000000010
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00908040
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fead0, length=0x0000000000000050
[0]clean_and_invalidate_data_cache_range: 1-1: addr=0x000000003b3feb10, length=0x0000000000000010
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00900040
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe5f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]usb_dev_init: 6

== PRINT (fromdevice) 0xffff00003b3fea00 (0xffff00003b3fead0) - 0xffff00003b3fea50 ==

ffff00003b3fea00: 0902 5000 0201 07c0 3209 0400 0001 0206 ..P.....2.......
ffff00003b3fea10: 0005 0524 0010 0105 2406 0001 0d24 0f03 ...$....$....$..
ffff00003b3fea20: 0000 0000 ea05 0000 0007 0581 0310 0020 ...............
ffff00003b3fea30: 0904 0100 000a 0000 0009 0401 0102 0a00 ................
ffff00003b3fea40: 0004 0705 8202 4000 0007 0502 0240 0000 ......@......@..
[2]usb_dev_init: 7
[2]usb_dev_init: Device ven525-a4a2, dev2-0-0 found
[2]dw2_hc_get_desc: call
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc30, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00900040
[2]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc30, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4b0, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00908040
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4b0, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe404, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00900040
[3]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe404, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00900040
[0]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9c0, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00908040
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9c0, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe424, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00900040
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe424, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00900040
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9c0, length=0x000000000000000a
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00908040
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe9c0, length=0x000000000000000a
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe424, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00900040
[2]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe424, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_get_desc: call
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc30, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00900040
[0]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc30, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe480, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00908040
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe480, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3d4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00900040
[1]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3d4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00900040
[0]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe480, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00908040
[0]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe480, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3d4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00900040
[0]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3d4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00900040
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdc20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe450, length=0x000000000000003c
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00908040
[3]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe450, length=0x000000000000003c
[1]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3a4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00900040
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3a4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]usb_dev_init: Product: QEMU RNDIS/QEMU USB Network Device
[1]usb_dev_init: 8
[1]usb_func_get_if_name: func name=int2-6-0
[1]usb_dev_init: Interface int2-6-0 found
[1]usb_devfactory_get_device: Using device/interface int2-6-0
[1]usb_func_get_if_name: func name=inta-0-0
[1]usb_dev_init: Interface inta-0-0 found
[1]usb_dev_init: Function is not supported
[1]usb_func_get_if_name: func name=inta-0-0
[1]usb_dev_init: Interface inta-0-0 found
[1]usb_dev_init: Function is not supported
[1]usb_dev_init: 9
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3a0, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3a0, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00500008
[0]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe560, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00508008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe560, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe540, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00508008
[2]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe540, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe520, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe520, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe500, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe500, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4e0, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00508008
[0]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4e0, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4c0, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00508008
[2]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4c0, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdcc0, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00900040
[3]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdcc0, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00908040
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2f4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_get_desc: call
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdbc0, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00900040
[2]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdbc0, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe270, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00908040
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe270, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1c4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00900040
[0]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1c4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdbb0, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00900040
[2]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdbb0, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4a0, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00908040
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4a0, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1e4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00900040
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1e4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdbb0, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00900040
[1]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdbb0, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe260, length=0x000000000000001a
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00908040
[0]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe260, length=0x000000000000001a
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00900040
[3]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[1]cdcether_configure: MAC address is 40:54:0:12:34:57
[1]usb_dev_config: 0 is ok
[1]usb_stdhub_enumerate_ports: Port 1: Device configured
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2b0, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe2b0, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00500008
[0]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe950, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe950, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3a0, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe3a0, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe560, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00508008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe560, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_control_message: 1
[3]dw2_hc_control_message: 2
[3]dw2_hc_control_message: 3
[3]dw2_hc_submit_block_request: 1
[3]dw2_hc_submit_block_request: 2-1.1
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00500008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe540, length=0x0000000000000004
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00508008
[2]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe540, length=0x0000000000000004
[1]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_control_message: 1
[0]dw2_hc_control_message: 2
[0]dw2_hc_control_message: 3
[0]dw2_hc_submit_block_request: 1
[0]dw2_hc_submit_block_request: 2-1.1
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00500008
[0]dw2_hc_start_channel: 10-13
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe520, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00508008
[0]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe520, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_xfer_stage: 1
[0]dw2_hc_xfer_stage: 2
[0]dw2_hc_xfer_stage: 3
[0]dw2_hc_xfer_stage_async: 1: ch=0
[0]dw2_hc_xfer_stage_async: 2
[0]dw2_hc_xfer_stage_async: 3
[0]dw2_hc_xfer_stage_async: 4
[0]dw2_hc_xfer_stage_async: 6
[0]dw2_hc_start_trans: 1
[0]dw2_hc_start_trans: 2-2
[0]dw2_hc_start_channel: 1
[0]dw2_hc_start_channel: 2
[0]dw2_hc_start_channel: 3
[0]dw2_hc_start_channel: 4
[0]dw2_hc_start_channel: 5
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[0]dw2_hc_start_channel: 6
[0]dw2_hc_start_channel: 7
[0]dw2_hc_start_channel: 9
[0]dw2_hc_start_channel: 10-9: char=0x00500008
[0]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_control_message: 1
[1]dw2_hc_control_message: 2
[1]dw2_hc_control_message: 3
[1]dw2_hc_submit_block_request: 1
[1]dw2_hc_submit_block_request: 2-1.1
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe500, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe500, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4e0, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4e0, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[0]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_control_message: 1
[2]dw2_hc_control_message: 2
[2]dw2_hc_control_message: 3
[2]dw2_hc_submit_block_request: 1
[2]dw2_hc_submit_block_request: 2-1.1
[2]dw2_hc_xfer_stage: 1
[2]dw2_hc_xfer_stage: 2
[2]dw2_hc_xfer_stage: 3
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2
[2]dw2_hc_xfer_stage_async: 3
[2]dw2_hc_xfer_stage_async: 4
[2]dw2_hc_xfer_stage_async: 6
[2]dw2_hc_start_trans: 1
[2]dw2_hc_start_trans: 2-2
[2]dw2_hc_start_channel: 1
[2]dw2_hc_start_channel: 2
[2]dw2_hc_start_channel: 3
[2]dw2_hc_start_channel: 4
[2]dw2_hc_start_channel: 5
[2]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[2]clean_and_invalidate_data_cache_range: 2
[2]dw2_hc_start_channel: 6
[2]dw2_hc_start_channel: 7
[2]dw2_hc_start_channel: 9
[2]dw2_hc_start_channel: 10-9: char=0x00500008
[2]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x0000000038bfdd20, length=0x0000000000000008
[0]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_xfer_stage: 1
[3]dw2_hc_xfer_stage: 2
[3]dw2_hc_xfer_stage: 3
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2
[3]dw2_hc_xfer_stage_async: 3
[3]dw2_hc_xfer_stage_async: 4
[3]dw2_hc_xfer_stage_async: 6
[3]dw2_hc_start_trans: 1
[3]dw2_hc_start_trans: 2-2
[3]dw2_hc_start_channel: 1
[3]dw2_hc_start_channel: 2
[3]dw2_hc_start_channel: 3
[3]dw2_hc_start_channel: 4
[3]dw2_hc_start_channel: 5
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4c0, length=0x0000000000000004
[3]clean_and_invalidate_data_cache_range: 2
[3]dw2_hc_start_channel: 6
[3]dw2_hc_start_channel: 7
[3]dw2_hc_start_channel: 9
[3]dw2_hc_start_channel: 10-9: char=0x00508008
[3]dw2_hc_start_channel: 10-13
[0]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe4c0, length=0x0000000000000004
[0]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_xfer_stage: 1
[1]dw2_hc_xfer_stage: 2
[1]dw2_hc_xfer_stage: 3
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2
[1]dw2_hc_xfer_stage_async: 3
[1]dw2_hc_xfer_stage_async: 4
[1]dw2_hc_xfer_stage_async: 6
[1]dw2_hc_start_trans: 1
[1]dw2_hc_start_trans: 2-2
[1]dw2_hc_start_channel: 1
[1]dw2_hc_start_channel: 2
[1]dw2_hc_start_channel: 3
[1]dw2_hc_start_channel: 4
[1]dw2_hc_start_channel: 5
[1]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[1]clean_and_invalidate_data_cache_range: 2
[1]dw2_hc_start_channel: 6
[1]dw2_hc_start_channel: 7
[1]dw2_hc_start_channel: 9
[1]dw2_hc_start_channel: 10-9: char=0x00500008
[1]dw2_hc_start_channel: 10-13
[3]clean_and_invalidate_data_cache_range: 1: addr=0x000000003b3fe1b4, length=0x0000000000000000
[3]clean_and_invalidate_data_cache_range: 2
[2]usb_dev_config: 0 is ok
[2]dw2_rport_init: 4
[2]dw2_rport_init: Device configured
[2]usbhc_init: dw2hc initialized

[3]mount: source: /dev/sdc3, target: /mnt, type: ext2, flags: 0x0

[0]fileopen: cant namei /etc/issue
Welcome to xv6 2022-06-26 (musl) mini tty

mini login:
```

## usb機能なし

```
??L>???+?W??k׋?֕?+??"????m?깢?ɵ??????????????2????R??[0]rand_init: rand_init   k
pagecache_init ok
[0]main: cpu 0 init finished
[2]main: cpu 2 init finished
[0]emmc_card_init: poweron
[1]main: cpu 1 init finished
[3]main: cpu 3 init finished
[0]emmc_issue_command_int: rrror occured whilst waiting for command complete int
[0]emmc_card_reset: found valid version 4.xx SD card
[0]dev_init: LBA of 1st block 0x20800, 0xf0000 blocks totally
[0]iinit: sb: size 800000 nblocks 799419 ninodes 1024 nlog 126 logstart 2 inode5
[1]execve: bad
init: exec sh failed

[2]fileopen: cant namei /etc/issue
Welcome to xv6 2022-06-26 (musl) mini tty

mini login:
```

[0]dw2_hc_control_kern/message: 9
kern/usb/dw2hcd.c: 16: assertion failed.
