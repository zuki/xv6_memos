# xv6

```
1]dw2_hc: dw2_hc created
[1]dw2_hc_enable_rport: enabled rootport
[1]dw2_hc_get_desc: call dw2_hc_control_message
[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0100, index=0, len=8
    data: 00 00 00 00 00 00 00 00

[1]dw2_hc_control_message: setup_data: 0xffff000038bfdd50
[1]dw2_hc_submit_block_request: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, timeout=0
[1]dw2_hc_submit_block_request: Control xfer: IN request
[1]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[1]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[1]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feba0
[1]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[1]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[1]dw2_hc_start_channel: 6: clean & invalidate
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[1]dw2_hc_start_channel: 10-9: char=0x00100008
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80100008
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[0]dw2_hc_channel_intr_hdl: state: no_split
[3]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=1, stage=0, timeout=0
[3]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=1, stage=0, timeout=0
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[3]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fecb0, bpt: 8, ppt: 1, packets: 1

[3]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feba0
[3]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[3]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[3]dw2_hc_start_channel: 6: clean & invalidate
[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fecb0, length=0x8
[3]dw2_hc_start_channel: 10-9: char=0x00108008
[3]dw2_hc_start_channel: 10-10: fsched=0x0
[3]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[3]dw2_hc_start_channel: char=0x80108008
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fecb0, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fecb0, length=0x8
[0]dw2_hc_channel_intr_hdl: state: no_split
[2]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=1, timeout=0
[2]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=1, timeout=0
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[2]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec04, bpt: 0, ppt: 1, packets: 1

[2]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feba0
[2]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[2]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[2]dw2_hc_start_channel: 6: clean & invalidate
[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0
[2]dw2_hc_start_channel: 10-9: char=0x00100008
[2]dw2_hc_start_channel: 10-10: fsched=0x0
[2]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[2]dw2_hc_start_channel: char=0x80100008
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec04, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0
[0]dw2_hc_channel_intr_hdl: state: no_split
[1]dw2_hc_control_message: result len=8

== dev_desc(8): 0xffff00003b3fecb0 (0xffff00003b3fecb0) - 0xffff00003b3fecb8: size=0x8 ==

ffff00003b3fecb0: 1201 0002 0900 0240                     .......@

[1]dw2_hc_get_desc: call dw2_hc_control_message
[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0100, index=0, len=18
    data: 12 01 00 02 09 00 02 40 00 00 00 00 00 00 00 00 00 00

[1]dw2_hc_control_message: setup_data: 0xffff000038bfdd50
[1]dw2_hc_submit_block_request: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, timeout=0
[1]dw2_hc_submit_block_request: Control xfer: IN request
[1]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[1]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[1]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feba0
[1]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[1]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[1]dw2_hc_start_channel: 6: clean & invalidate
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[1]dw2_hc_start_channel: 10-9: char=0x00100040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80100040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[0]dw2_hc_channel_intr_hdl: state: no_split
[3]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=1, stage=0, timeout=0
[3]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=1, stage=0, timeout=0
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[3]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fecb0, bpt: 18, ppt: 1, packets: 1

[3]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feba0
[3]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[3]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[3]dw2_hc_start_channel: 6: clean & invalidate
[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fecb0, length=0x12
[3]dw2_hc_start_channel: 10-9: char=0x00108040
[3]dw2_hc_start_channel: 10-10: fsched=0x0
[3]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[3]dw2_hc_start_channel: char=0x80108040
[3]dw2_hc_channel_intr_hdl: called
[3]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[3]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[3]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fecb0, bpt: 18, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fecb0, length=0x12
[3]dw2_hc_channel_intr_hdl: state: no_split
[1]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=1, timeout=0
[1]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=1, timeout=0
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[1]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec04, bpt: 0, ppt: 1, packets: 1

[1]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feba0
[1]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[1]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[1]dw2_hc_start_channel: 6: clean & invalidate
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0
[1]dw2_hc_start_channel: 10-9: char=0x00100040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80100040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec04, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0
[0]dw2_hc_channel_intr_hdl: state: no_split
[2]dw2_hc_control_message: result len=18

== dev_desc: 0xffff00003b3fecb0 (0xffff00003b3fecb0) - 0xffff00003b3fecc2: size=0x12 ==

ffff00003b3fecb0: 1201 0002 0900 0240 2404 1425 b30b 0000 .......@$..%....
ffff00003b3fecc0: 0000                                    ..

[2]usb_dev_init: 3: addr=1
[2]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x0, req=5, value=0x0001, index=0, len=0
[2]dw2_hc_control_message: setup_data: 0xffff000038bfdd60
[2]dw2_hc_submit_block_request: host=0xffff00003b3fede0, urb=0xffff000038bfdd18, timeout=0
[2]dw2_hc_submit_block_request: Control xfer: OUT request no data
[2]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd18, in=0, stage=0, timeout=0
[2]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd18, in=0, stage=0, timeout=0
[2]dw2_hc_xfer_stage_async: 1: ch=0
[2]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[2]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd60, bpt: 8, ppt: 1, packets: 1

[2]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feba0
[2]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[2]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[2]dw2_hc_start_channel: 6: clean & invalidate
[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd60, length=0x8
[2]dw2_hc_start_channel: 10-9: char=0x00100040
[2]dw2_hc_start_channel: 10-10: fsched=0x0
[2]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[2]dw2_hc_start_channel: char=0x80100040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd18, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd60, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd60, length=0x8
[0]dw2_hc_channel_intr_hdl: state: no_split
[3]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd18, in=1, stage=1, timeout=0
[3]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd18, in=1, stage=1, timeout=0
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feba0
[3]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec04, bpt: 0, ppt: 1, packets: 1

[3]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feba0
[3]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[3]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feba0
[3]dw2_hc_start_channel: 6: clean & invalidate
[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0
[3]dw2_hc_start_channel: 10-9: char=0x00108040
[3]dw2_hc_start_channel: 10-10: fsched=0x0
[3]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[3]dw2_hc_start_channel: char=0x80108040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feba0, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd18, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feba0
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec04, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec04, length=0x0
[0]dw2_hc_channel_intr_hdl: state: no_split
[3]dw2_hc_control_message: result len=0
[3]usb_dev_init: cfg_desc[1]=0xffff00003b3fec90
[3]usb_dev_init: 5.2: cindex=0
[3]dw2_hc_get_desc: call dw2_hc_control_message
[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0200, index=0, len=9
    data: 00 00 00 00 00 00 00 00 00

[3]dw2_hc_control_message: setup_data: 0xffff000038bfdd50
[3]dw2_hc_submit_block_request: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, timeout=0
[3]dw2_hc_submit_block_request: Control xfer: IN request
[3]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=0, timeout=0
[3]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=0, timeout=0
[3]dw2_hc_xfer_stage_async: 1: ch=0
[3]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb80
[3]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feb80
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[3]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feb80
[3]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feb80
[3]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feb80
[3]dw2_hc_start_channel: 6: clean & invalidate
[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[3]dw2_hc_start_channel: 10-9: char=0x00500040
[3]dw2_hc_start_channel: 10-10: fsched=0x0
[3]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[3]dw2_hc_start_channel: char=0x80500040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb80, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feb80
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[0]dw2_hc_channel_intr_hdl: state: no_split
[1]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=1, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=1, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb80
[1]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feb80
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec90, bpt: 9, ppt: 1, packets: 1

[1]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feb80
[1]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feb80
[1]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feb80
[1]dw2_hc_start_channel: 6: clean & invalidate
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec90, length=0x9
[1]dw2_hc_start_channel: 10-9: char=0x00508040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80508040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb80, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feb80
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec90, bpt: 9, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec90, length=0x9
[0]dw2_hc_channel_intr_hdl: state: no_split
[1]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=1, timeout=0
[1]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=1, timeout=0
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb80
[1]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feb80
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3febe4, bpt: 0, ppt: 1, packets: 1

[1]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feb80
[1]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feb80
[1]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feb80
[1]dw2_hc_start_channel: 6: clean & invalidate
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3febe4, length=0x0
[1]dw2_hc_start_channel: 10-9: char=0x00500040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80500040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb80, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feb80
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3febe4, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3febe4, length=0x0
[0]dw2_hc_channel_intr_hdl: state: no_split
[1]dw2_hc_control_message: result len=9

== config_desc(8): 0xffff00003b3fec90 (0xffff00003b3fec90) - 0xffff00003b3fec99: size=0x9 ==

ffff00003b3fec90: 0902 2900 0101 00e0 01                  ..)......

[1]usb_dev_init: cfg_desc[2]=0xffff00003b3fec70
[1]dw2_hc_get_desc: call dw2_hc_control_message
[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0200, index=0, len=41
    data: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 50 67 0c 00 00 00 ff ff 02 00 00 00 00 00 00 00 09 02 29 00 01 01 00 e01

[1]dw2_hc_control_message: setup_data: 0xffff000038bfdd50
[1]dw2_hc_submit_block_request: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, timeout=0
[1]dw2_hc_submit_block_request: Control xfer: IN request
[1]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb60
[1]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feb60
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[1]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feb60
[1]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feb60
[1]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feb60
[1]dw2_hc_start_channel: 6: clean & invalidate
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[1]dw2_hc_start_channel: 10-9: char=0x00500040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80500040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb60, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feb60
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8
[0]dw2_hc_channel_intr_hdl: state: no_split
[1]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=1, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=1, stage=0, timeout=0
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb60
[1]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feb60
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec70, bpt: 41, ppt: 1, packets: 1

[1]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feb60
[1]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feb60
[1]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feb60
[1]dw2_hc_start_channel: 6: clean & invalidate
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec70, length=0x29
[1]dw2_hc_start_channel: 10-9: char=0x00508040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80508040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb60, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feb60
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec70, bpt: 41, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec70, length=0x29
[0]dw2_hc_channel_intr_hdl: state: no_split
[1]dw2_hc_xfer_stage: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=1, timeout=0
[1]dw2_hc_xfer_stage_async: host=0xffff00003b3fede0, urb=0xffff000038bfdd08, in=0, stage=1, timeout=0
[1]dw2_hc_xfer_stage_async: 1: ch=0
[1]dw2_hc_xfer_stage_async: 2: stdata=0xffff00003b3feb60
[1]dw2_xfer_stagedata: no use fsched

stagedata: stagedata: 0xffff00003b3feb60
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3febc4, bpt: 0, ppt: 1, packets: 1

[1]dw2_hc_xfer_stage_async: enable channel interrupt: stdata[0]=0xffff00003b3feb60
[1]dw2_hc_start_trans: host=0xffff00003b3fede0, stdata=0xffff00003b3feb60
[1]dw2_hc_start_channel: host=0xffff00003b3fede0, stdata=0xffff00003b3feb60
[1]dw2_hc_start_channel: 6: clean & invalidate
[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3febc4, length=0x0
[1]dw2_hc_start_channel: 10-9: char=0x00500040
[1]dw2_hc_start_channel: 10-10: fsched=0x0
[1]dw2_hc_start_channel: ch=0x0, intrmask=0x78f
[1]dw2_hc_start_channel: char=0x80500040
[0]dw2_hc_channel_intr_hdl: called
[0]dw2_hc_channel_intr_hdl: stdata=0xffff00003b3feb60, fsched=0x0
[0]dw2_hc_channel_intr_hdl: urb=0xffff000038bfdd08, rpenabled=0x1
[0]dw2_hc_channel_intr_hdl: substate: wait_for_xfer_complete

stagedata: stagedata: 0xffff00003b3feb60
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3febc4, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3febc4, length=0x0
[0]dw2_hc_channel_intr_hdl: state: no_split
[2]dw2_hc_control_message: result len=41

== config_desc: 0xffff00003b3fec70 (0xffff00003b3fec70) - 0xffff00003b3fec99: size=0x29 ==

ffff00003b3fec70: 0902 2900 0101 00e0 0109 0400 0001 0900 ..).............
ffff00003b3fec80: 5067 0c00 0000 ffff 0200 0000 0000 0000 Pg..............
ffff00003b3fec90: 0902 2900 0101 00e0 01                  ..)......

[2]cfg_parser: skip(0xffff00003b3fec7b), end_pos(0xffff00003b3fec99)
[2]cfg_parser: skip(0xffff00003b3fec84), end_pos(0xffff00003b3fec99)
[2]cfg_parser: skip(0xffff00003b3fec90), end_pos(0xffff00003b3fec99)
[2]cfg_parser: skip(0xffff00003b3fec90), end_pos(0xffff00003b3fec99)
loopkern/console.c:348: kernel panic at cpu 2.
```

## ディスクリプタを格納するメモリが64バイト境界にないのが原因か?

- とりあえずディスクリプタ用には`kmalloc()`ではなく`kalloc()`を使うように変更した
- `Device ven424-2514, dev9-0-2 found`まで動くようになった
- 別のエラー発生

```
[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0100, index=0, len=8
  data: 0xffff000038bdf000 (0x8 bytes)
    00 e0 bd 38 00 00 ff ff


stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bdf000, bpt: 8, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdf000, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bdf000, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdf000, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

== dev_desc(8): 0xffff000038bdf000 (0xffff000038bdf000) - 0xffff000038bdf008: size=0x8 ==

ffff000038bdf000: 1201 0002 0900 0240                     .......@

[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0100, index=0, len=18
  data: 0xffff000038bdf000 (0x12 bytes)
    12 01 00 02 09 00 02 40 55 55 15 55 55 55 55 55 55 55


stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bdf000, bpt: 18, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdf000, length=0x12

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bdf000, bpt: 18, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdf000, length=0x12

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

== dev_desc: 0xffff000038bdf000 (0xffff000038bdf000) - 0xffff000038bdf012: size=0x12 ==

ffff000038bdf000: 1201 0002 0900 0240 2404 1425 b30b 0000 .......@$..%....
ffff000038bdf010: 0001                                    ..

[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x0, req=5, value=0x0001, index=0, len=0

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd60, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd60, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd60, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd60, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0
[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0200, index=0, len=9
  data: 0xffff000038bde000 (0x9 bytes)
    00 d0 bd 38 00 00 ff ff 55


stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bde000, bpt: 9, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bde000, length=0x9

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bde000, bpt: 9, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bde000, length=0x9

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

== config_desc(8): 0xffff000038bde000 (0xffff000038bde000) - 0xffff000038bde009: size=0x9 ==

ffff000038bde000: 0902 2900 0101 00e0 01                  ..)......

[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0200, index=0, len=41
  data: 0xffff000038bde000 (0x29 bytes)
    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00


stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd50, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd50, length=0x8

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bde000, bpt: 41, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bde000, length=0x29

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bde000, bpt: 41, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bde000, length=0x29

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

stagedata: stagedata: 0xffff00003b3febd0
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fec34, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fec34, length=0x0

== config_desc: 0xffff000038bde000 (0xffff000038bde000) - 0xffff000038bde029: size=0x29 ==

ffff000038bde000: 0902 2900 0101 00e0 0109 0400 0001 0900 ..).............
ffff000038bde010: 0100 0705 8103 0100 0c09 0400 0101 0900 ................
ffff000038bde020: 0200 0705 8103 0100 0c                  .........

[2]usb_dev_init: Device ven424-2514, dev9-0-2 found
[2]usb_func_get_if_name: func name=int9-0-1
[2]usb_dev_init: Interface int9-0-1 found
[2]usb_dev_init: Function is not supported
[2]usb_func_get_if_name: func name=int9-0-2
[2]usb_dev_init: Interface int9-0-2 found
[2]usb_devfactory_get_device: Using device/interface int9-0-2

// SET_CONFIGURATION
[2]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x0, req=9, value=0x0001, index=0, len=0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdda0, bpt: 8, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdda0, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdda0, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdda0, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
// SET INTERFACE
[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x1, req=11, value=0x0001, index=0, len=0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd60, bpt: 8, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd60, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd60, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd60, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
// CLASS GET_DESCRIPTOR
[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0xa0, req=6, value=0x2900, index=0, len=9
  data: 0xffff000038bdd000 (0x9 bytes)
    00 c0 bd 38 00 00 ff ff 55


stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd40, bpt: 8, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd40, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd40, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd40, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bdd000, bpt: 9, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdd000, length=0x9

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bdd000, bpt: 9, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdd000, length=0x9

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
// SET_FEATURE
[2]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0008, index=1, len=0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
[2]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0008, index=2, len=0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0008, index=3, len=0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0008, index=4, len=0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0xa3, req=0, value=0x0000, index=1, len=4
  data: 0xffff00003b3feb50 (0x4 bytes)
    69 6e 74 39


stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3feb50, bpt: 4, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3feb50, length=0x4

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3feb50, bpt: 4, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3feb50, length=0x4

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
[1]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0004, index=1, len=0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fece0, type=0xa3, req=0, value=0x0000, index=1, len=4
  data: 0xffff00003b3feb50 (0x4 bytes)
    01 01 01 00


stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdd20, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdd20, length=0x8

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3feb50, bpt: 4, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3feb50, length=0x4

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3feb50, bpt: 4, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3feb50, length=0x4

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0

stagedata: stagedata: 0xffff00003b3fe900
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe964, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe964, length=0x0
[2]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fe910, type=0x80, req=6, value=0x0100, index=0, len=8
  data: 0xffff000038bdc000 (0x8 bytes)
    00 b0 bd 38 00 00 ff ff


stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdc70, bpt: 8, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdc70, length=0x8

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdc70, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdc70, length=0x8

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bdc000, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdc000, length=0x8

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bdc000, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdc000, length=0x8

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe864, bpt: 0, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe864, length=0x0

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe864, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe864, length=0x0

== dev_desc(8): 0xffff000038bdc000 (0xffff000038bdc000) - 0xffff000038bdc008: size=0x8 ==

ffff000038bdc000: 1201 0002 0900 0240                     .......@

[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fe910, type=0x80, req=6, value=0x0100, index=0, len=18
  data: 0xffff000038bdc000 (0x12 bytes)
    12 01 00 02 09 00 02 40 54 55 55 55 55 55 5d 45 55 55


stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdc70, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdc70, length=0x8

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdc70, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdc70, length=0x8

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bdc000, bpt: 18, ppt: 1, packets: 1

[2]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdc000, length=0x12

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: true, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bdc000, bpt: 18, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bdc000, length=0x12

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe864, bpt: 0, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe864, length=0x0

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe864, bpt: 0, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe864, length=0x0

== dev_desc: 0xffff000038bdc000 (0xffff000038bdc000) - 0xffff000038bdc012: size=0x12 ==

ffff000038bdc000: 1201 0002 0900 0240 2404 1425 b30b 0000 .......@$..%....
ffff000038bdc010: 0001                                    ..

[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fe910, type=0x0, req=5, value=0x0002, index=0, len=0

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdc80, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdc80, length=0x8

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdc80, bpt: 8, ppt: 1, packets: 1

[0]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdc80, length=0x8

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff00003b3fe864, bpt: 0, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe864, length=0x0

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: true, stage: true, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff00003b3fe864, bpt: 0, ppt: 1, packets: 1

[1]clean_and_invalidate_data_cache_range: 1: addr=0xffff00003b3fe864, length=0x0
[3]dw2_hc_control_message: host=0xffff00003b3fede0, ep=0xffff00003b3fe910, type=0x80, req=6, value=0x0200, index=0, len=9
  data: 0xffff000038bdb000 (0x9 bytes)
    00 a0 bd 38 00 00 ff ff 53


stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 0
    : buffp: 0xffff000038bfdc70, bpt: 8, ppt: 1, packets: 1

[3]clean_and_invalidate_data_cache_range: 1: addr=0xffff000038bfdc70, length=0x8

stagedata: stagedata: 0xffff00003b3fe800

stagedata: stagedata: 0xffff00003b3fe800
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : channel: 0, in: false, stage: false, fsused: false, split: false
    : state: 0, substate: 1, trstatus: 0
    : state: 0, substate: 1, trstatus: 0
    : buffp: 0xffff000038bfdc70, bpt: 8, ppt: 1, packets: 1

[  c: bufapd invalfda0e_dataccac e_t: g,:pp:: 1dr=0cfets00003
bfdc71, length=0x8
[0]tlganatnd snaaedatae 0xfffc0ch0_b3ne800
:   dr channel:003 bn:c70u ,esgah=0x8
se, fsused: false, split: false
    : state: 0, substate: 0, trstatus: 35
    : buffp: 0x0, bpt: 8, ppt: 0, packets: 0

[2]dw2_xfer_stagedata_get_pid: bad next: 402653182      // 0x17fffffe
kern/usb/dw2xferstagedata.c:322: assertion failed.
kern/console.c:348: kernel panic at cpu 2.
```

## 以下の2点を修正

- endpoint.c:usb_ep_skip_pid()のstastus=trueの際のassert文をコメントアウト（これは要修正）
- mboxでMACアドレスを取得する関数のバグを修正

```
== dev_desc(8): 0xffff000038bdf000 (0xffff000038bdf000) - 0xffff000038bdf008: size=0x8 ==

ffff000038bdf000: 1201 0002 0900 0240                     .......@


== dev_desc: 0xffff000038bdf000 (0xffff000038bdf000) - 0xffff000038bdf012: size=0x12 ==

ffff000038bdf000: 1201 0002 0900 0240 2404 1425 b30b 0000 .......@$..%....
ffff000038bdf010: 0001                                    ..


== config_desc(8): 0xffff000038bde000 (0xffff000038bde000) - 0xffff000038bde009: size=0x9 ==

ffff000038bde000: 0902 2900 0101 00e0 01                  ..)......


== config_desc: 0xffff000038bde000 (0xffff000038bde000) - 0xffff000038bde029: size=0x29 ==

ffff000038bde000: 0902 2900 0101 00e0 0109 0400 0001 0900 ..).............
ffff000038bde010: 0100 0705 8103 0100 0c09 0400 0101 0900 ................
ffff000038bde020: 0200 0705 8103 0100 0c                  .........

[2]usb_dev_init: Device ven424-2514, dev9-0-2 found
[2]usb_func_get_if_name: func name=int9-0-1
[2]usb_dev_init: Interface int9-0-1 found
[2]usb_dev_init: Function is not supported
[2]usb_func_get_if_name: func name=int9-0-2
[2]usb_dev_init: Interface int9-0-2 found
[2]usb_devfactory_get_device: Using device/interface int9-0-2

== dev_desc(8): 0xffff000038bdc000 (0xffff000038bdc000) - 0xffff000038bdc008: size=0x8 ==

ffff000038bdc000: 1201 0002 0900 0240                     .......@


== dev_desc: 0xffff000038bdc000 (0xffff000038bdc000) - 0xffff000038bdc012: size=0x12 ==

ffff000038bdc000: 1201 0002 0900 0240 2404 1425 b30b 0000 .......@$..%....
ffff000038bdc010: 0001                                    ..


== config_desc(8): 0xffff000038bdb000 (0xffff000038bdb000) - 0xffff000038bdb009: size=0x9 ==

ffff000038bdb000: 0902 2900 0101 00e0 01                  ..)......


== config_desc: 0xffff000038bdb000 (0xffff000038bdb000) - 0xffff000038bdb029: size=0x29 ==

ffff000038bdb000: 0902 2900 0101 00e0 0109 0400 0001 0900 ..).............
ffff000038bdb010: 0100 0705 8103 0100 0c09 0400 0101 0900 ................
ffff000038bdb020: 0200 0705 8103 0100 0c                  .........

[3]usb_dev_init: Device ven424-2514, dev9-0-2 found
[3]usb_func_get_if_name: func name=int9-0-1
[3]usb_dev_init: Interface int9-0-1 found
[3]usb_dev_init: Function is not supported
[3]usb_func_get_if_name: func name=int9-0-2
[3]usb_dev_init: Interface int9-0-2 found
[3]usb_devfactory_get_device: Using device/interface int9-0-2

== dev_desc(8): 0xffff000038bd9000 (0xffff000038bd9000) - 0xffff000038bd9008: size=0x8 ==

ffff000038bd9000: 1201 1002 ff00 ff40                     .......@


== dev_desc: 0xffff000038bd9000 (0xffff000038bd9000) - 0xffff000038bd9012: size=0x12 ==

ffff000038bd9000: 1201 1002 ff00 ff40 2404 0078 0003 0000 .......@$..x....
ffff000038bd9010: 0001                                    ..


== config_desc(8): 0xffff000038bd8000 (0xffff000038bd8000) - 0xffff000038bd8009: size=0x9 ==

ffff000038bd8000: 0902 2700 0101 00e0 01                  ..'......


== config_desc: 0xffff000038bd8000 (0xffff000038bd8000) - 0xffff000038bd8027: size=0x27 ==

ffff000038bd8000: 0902 2700 0101 00e0 0109 0400 0003 ff00 ..'.............
ffff000038bd8010: ff00 0705 8102 0002 0007 0502 0200 0200 ................
ffff000038bd8020: 0705 8303 1000 04                       .......

[3]usb_dev_init: Device ven424-7800 found
[3]usb_devfactory_get_device: Using device/interface ven424-7800
[1]lan7800_init_macaddr: MAC address is ?'??H
[3]usb_stdhub_enumerate_ports: Port 1: Device configured
[1]usb_stdhub_enumerate_ports: Port 1: Device configured
[0]dw2_hc_channel_intr_hdl: Transaction failed 1 (status 0x223)
[2]dw2_hc_submit_block_request: failed out command to get data
[2]usb_stdhub_enumerate_ports: Cannot get hub status
[2]usb_stdhub_config: Port enumeration failed
[2]usb_dev_config: Cannot configure device 0
[2]dw2_rport_init: cannot configure device
[2]_usb_device: Device ven424-2514, dev9-0-2 removed
[2]dw2_hc_rescan_dev: Cannot initialize root port
[2]usbhc_init: dw2hc initialized
```

## dw2_hc_control_message()にdataとして渡すデータで64バイトアラインでないものをすべてkalloc()に変更

- usbの初期化が成功。login可能となる

```
?+?W??k׋?֕?+??"????m?깢?ɵ??????????????2????R??[0]clock_init: clock init ok
[0]rand_init: rand_init ok
[0]init_vfssw: init_vfssw ok
mountinit ok
[0]install_rootfs: install_rootfs ok
pagecache_init ok
[0]timer_init: timerfreq = 0x124f800
[1]timer_init: timerfreq = 0x124f800
[2]timer_init: timerfreq = 0x124f800
[0]main: cpu 0 init finished
[2]main: cpu 2 init finished
[1]main: cpu 1 init finished
[0]emmc_card_init: poweron
[3]timer_init: timerfreq = 0x124f800
[3]main: cpu 3 init finished
[0]emmc_issue_command_int: rrror occured whilst waiting for command complete interrupt
[0]emmc_card_reset: found valid version 4.xx SD card
[0]dev_init: partition[0]: LBA=0x800, #SECS=0x20000
[0]dev_init: partition[1]: LBA=0x20800, #SECS=0xf0000
[0]dev_init: partition[2]: LBA=0x110800, #SECS=0xef800
[0]iinit: sb: size 100000 nblocks 99835 ninodes 1024 nlog 126 logstart 2 inodestart 128 bmapstart 161
[0]initlog: init log ok

== dev_desc: 0xffff000038bdf000 (0xffff000038bdf000) - 0xffff000038bdf012: size=0x12 ==

ffff000038bdf000: 1201 0002 0900 0240 2404 1425 b30b 0000 .......@$..%....
ffff000038bdf010: 0001                                    ..


== config_desc: 0xffff000038bde000 (0xffff000038bde000) - 0xffff000038bde029: size=0x29 ==

ffff000038bde000: 0902 2900 0101 00e0 0109 0400 0001 0900 ..).............
ffff000038bde010: 0100 0705 8103 0100 0c09 0400 0101 0900 ................
ffff000038bde020: 0200 0705 8103 0100 0c                  .........

[1]usb_dev_init: Device ven424-2514, dev9-0-2 found
[1]usb_func_get_if_name: func name=int9-0-1
[1]usb_dev_init: Interface int9-0-1 found
[1]usb_dev_init: Function is not supported
[1]usb_func_get_if_name: func name=int9-0-2
[1]usb_dev_init: Interface int9-0-2 found
[1]usb_devfactory_get_device: Using device/interface int9-0-2

== dev_desc: 0xffff000038bdb000 (0xffff000038bdb000) - 0xffff000038bdb012: size=0x12 ==

ffff000038bdb000: 1201 0002 0900 0240 2404 1425 b30b 0000 .......@$..%....
ffff000038bdb010: 0001                                    ..


== config_desc: 0xffff000038bda000 (0xffff000038bda000) - 0xffff000038bda029: size=0x29 ==

ffff000038bda000: 0902 2900 0101 00e0 0109 0400 0001 0900 ..).............
ffff000038bda010: 0100 0705 8103 0100 0c09 0400 0101 0900 ................
ffff000038bda020: 0200 0705 8103 0100 0c                  .........

[1]usb_dev_init: Device ven424-2514, dev9-0-2 found
[1]usb_func_get_if_name: func name=int9-0-1
[1]usb_dev_init: Interface int9-0-1 found
[1]usb_dev_init: Function is not supported
[1]usb_func_get_if_name: func name=int9-0-2
[1]usb_dev_init: Interface int9-0-2 found
[1]usb_devfactory_get_device: Using device/interface int9-0-2

== dev_desc: 0xffff000038bd4000 (0xffff000038bd4000) - 0xffff000038bd4012: size=0x12 ==

ffff000038bd4000: 1201 1002 ff00 ff40 2404 0078 0003 0000 .......@$..x....
ffff000038bd4010: 0001                                    ..


== config_desc: 0xffff000038bd3000 (0xffff000038bd3000) - 0xffff000038bd3027: size=0x27 ==

ffff000038bd3000: 0902 2700 0101 00e0 0109 0400 0003 ff00 ..'.............
ffff000038bd3010: ff00 0705 8102 0002 0007 0502 0200 0200 ................
ffff000038bd3020: 0705 8303 1000 04                       .......

[2]usb_dev_init: Device ven424-7800 found
[2]usb_devfactory_get_device: Using device/interface ven424-7800
[3]lan7800_init_macaddr: MAC address is b8:27:eb:ab:e8:48
[3]usb_stdhub_enumerate_ports: Port 1: Device configured
[2]usb_stdhub_enumerate_ports: Port 1: Device configured
[3]dw2_rport_init: Device configured
[3]usbhc_init: dw2hc initialized

[2]mount: source: /dev/sdc3, target: /mnt, type: ext2, flags: 0x0

[0]fileopen: cant namei /etc/issue
Welcome to xv6 2022-06-26 (musl) mini tty

mini login: root
Password:
[1]fileopen: cant namei /etc/profile
[1]fileopen: cant namei /.profile
# ls
bin  dev  etc  home  lib  mnt  test.txt  usr
# ls -l
total 5
drwxrwxr-x 1 root root 1664 Nov 29 17:10 bin
drwxrwxr-x 1 root root  384 Nov 29 17:10 dev
drwxrwxr-x 1 root root  320 Nov 29 17:10 etc
drwxrwxr-x 1 root root  192 Nov 29 17:10 home
drwxrwxrwx 1 root root  256 Nov 29 17:10 lib
drwxrwxr-x 3 root root 4096 Nov 29 17:10 mnt
-rwxr-xr-x 1 root root   94 Nov 29 17:10 test.txt
drwxrwxr-x 1 root root  256 Nov 29 17:10 usr
# ls /bin
bigtest  echo       init   mkfs       mount   pipetest  sigtest3   umount
cat      getty      login  mmaptest   mysh    sigtest   su         utest
date     hello-dyn  ls     mmaptest2  passwd  sigtest2  timertest  vi
# ls /usr/local/bin
file
# ls /usr/bin
'['          cp          fmt         mkdir      printf      size      truncate
 addr2line   csplit      fold        mkfifo     ptx         sleep     tsort
 ar          cut         gawk        mknod      pwd         sort      tty
 as          dash        getlimits   mktemp     ranlib      split     uname
 awk         date        ginstall    mv         readelf     stat      unexpand
 b2sum       dcgen       gprof       nice       readlink    stdbuf    uniq
 base32      dd          groups      nl         realpath    strings   unlink
 base64      df          head        nm         rm          strip     uptime
 basename    dir         hostid      nohup      rmdir       stty      users
 basenc      dircolors   id          nproc      runcon      sum       vdir
 c++filt     dirname     join        numfmt     sed         sync      wc
 cat         du          kill        objcopy    seq         tac       who
 chcon       echo        ld          objdump    sh          tail      whoami
 chgrp       elfedit     ld.bfd      od         sha224sum   tee       yes
 chmod       env         link        paste      sha256sum   test
 chown       expand      ln          pathchk    sha384sum   timeout
 chroot      expr        logname     pinky      sha512sum   touch
 cksum       factor      ls          pr         shred       tr
 comm        false       md5sum      printenv   shuf        true
# date
Tue Nov 29 17:13:49 JST 2022
# whoami
root
#
```

# simpleusb

```
logger: Circle 44.5 started on Raspberry Pi 3 Model B+ (AArch64)
00:00:00.66 timer: SpeedFactor is 1.51
00:00:01.47 hc: ep=0x60b440, type=0x80, req=6, value=0x0100, index=0, len=8
00:00:01.47 data: Dumping 0x8 bytes starting at 0x60B4C0
00:00:01.48 data: B4C0: 55 55 55 55 55 55 55 55-55 55 55 55 55 5D 55 55
00:00:01.48 stdata: channel: 0, in: 0, stage: 0
00:00:01.49 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.49 stdata:     buffp: 0x60b540, bpt: 8, ppt: 1, packets: 1

00:00:01.50 stdata: channel: 0, in: 1, stage: 0
00:00:01.50 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.51 stdata:     buffp: 0x60b4c0, bpt: 8, ppt: 1, packets: 1

00:00:01.51 stdata: channel: 0, in: 0, stage: 1
00:00:01.52 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.52 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:01.53 hc: ep=0x60b440, type=0x80, req=6, value=0x0100, index=0, len=18
00:00:01.54 data: Dumping 0x12 bytes starting at 0x60B4C0
00:00:01.54 data: B4C0: 12 01 00 02 09 00 02 40-55 55 55 55 55 5D 55 55
00:00:01.55 data: B4D0: 55 55 55 55 55 55 55 55-55 55 55 55 75 75 55 55
00:00:01.55 stdata: channel: 0, in: 0, stage: 0
00:00:01.56 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.56 stdata:     buffp: 0x60b540, bpt: 8, ppt: 1, packets: 1

00:00:01.57 stdata: channel: 0, in: 1, stage: 0
00:00:01.57 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.58 stdata:     buffp: 0x60b4c0, bpt: 18, ppt: 1, packets: 1

00:00:01.58 stdata: channel: 0, in: 0, stage: 1
00:00:01.59 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.59 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:01.60 usbdev: Dumping 0x12 bytes starting at 0x60B4C0
00:00:01.61 usbdev: B4C0: 12 01 00 02 09 00 02 40-24 04 14 25 B3 0B 00 00
00:00:01.61 usbdev: B4D0: 00 01 25 B8 55 55 55 55-55 55 55 55 75 75 55 55
00:00:01.62 hc: ep=0x60b440, type=0x0, req=5, value=0x0001, index=0, len=0
00:00:01.63 stdata: channel: 0, in: 0, stage: 0
00:00:01.63 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.63 stdata:     buffp: 0x60b540, bpt: 8, ppt: 1, packets: 1

00:00:01.64 stdata: channel: 0, in: 1, stage: 1
00:00:01.65 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.65 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:01.71 hc: ep=0x60b440, type=0x80, req=6, value=0x0200, index=0, len=9
00:00:01.71 data: Dumping 0x9 bytes starting at 0x60B540
00:00:01.72 data: B540: 00 05 01 00 00 00 00 00-34 34 30 2C 20 74 79 70
00:00:01.72 stdata: channel: 0, in: 0, stage: 0
00:00:01.73 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.73 stdata:     buffp: 0x60ba80, bpt: 8, ppt: 1, packets: 1

00:00:01.74 stdata: channel: 0, in: 1, stage: 0
00:00:01.74 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.75 stdata:     buffp: 0x60b540, bpt: 9, ppt: 1, packets: 1

00:00:01.75 stdata: channel: 0, in: 0, stage: 1
00:00:01.76 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.76 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:01.77 hc: ep=0x60b440, type=0x80, req=6, value=0x0200, index=0, len=41
00:00:01.78 data: Dumping 0x29 bytes starting at 0x60B540
00:00:01.78 data: B540: 09 02 29 00 01 01 00 E0-01 12 78 E0 20 74 79 70
00:00:01.79 data: B550: 65 3D 30 78 30 2C 20 72-65 71 3D 35 2C 20 76 61
00:00:01.79 data: B560: 6C 75 65 3D 30 78 30 30-30 31 2C 20 69 6E 64 65
00:00:01.80 stdata: channel: 0, in: 0, stage: 0
00:00:01.81 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.81 stdata:     buffp: 0x60ba80, bpt: 8, ppt: 1, packets: 1

00:00:01.82 stdata: channel: 0, in: 1, stage: 0
00:00:01.82 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.83 stdata:     buffp: 0x60b540, bpt: 41, ppt: 1, packets: 1

00:00:01.83 stdata: channel: 0, in: 0, stage: 1
00:00:01.84 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.84 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:01.85 usbdev: Dumping 0x29 bytes starting at 0x60B540
00:00:01.85 usbdev: B540: 09 02 29 00 01 01 00 E0-01 09 04 00 00 01 09 00
00:00:01.86 usbdev: B550: 01 00 07 05 81 03 01 00-0C 09 04 00 01 01 09 00
00:00:01.87 usbdev: B560: 02 00 07 05 81 03 01 00-0C B4 37 00 69 6E 64 65
00:00:01.87 usbdev0-1: Device ven424-2514, dev9-0-2 found
00:00:01.88 usbdev0-1: Interface int9-0-1 found
00:00:01.88 usbdev0-1: Function is not supported
00:00:01.89 usbdev0-1: Interface int9-0-2 found
00:00:01.89 usbdev0-1: Using device/interface int9-0-2
```

## 以後はLAN7800

```
00:00:01.90 hc: ep=0x60b440, type=0xa0, req=6, value=0x2900, index=0, len=9
00:00:01.90 data: Dumping 0x9 bytes starting at 0x616240
00:00:01.91 data: 6240: 50 AE 0A 00 00 00 00 00-00 00 00 00 00 00 00 00
00:00:01.91 stdata: channel: 0, in: 0, stage: 0
00:00:01.92 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.92 stdata:     buffp: 0x60bb00, bpt: 8, ppt: 1, packets: 1

00:00:01.93 stdata: channel: 0, in: 1, stage: 0
00:00:01.93 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.94 stdata:     buffp: 0x616240, bpt: 9, ppt: 1, packets: 1

00:00:01.94 stdata: channel: 0, in: 0, stage: 1
00:00:01.95 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.95 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:01.96 hc: ep=0x60b440, type=0x0, req=9, value=0x0001, index=0, len=0
00:00:01.97 stdata: channel: 0, in: 0, stage: 0
00:00:01.97 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.98 stdata:     buffp: 0x60cd00, bpt: 8, ppt: 1, packets: 1

00:00:01.98 stdata: channel: 0, in: 1, stage: 1
00:00:01.99 stdata:     state: 0, substate: 0, trstatus: 0
00:00:01.99 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:02.05 hc: ep=0x60b440, type=0x1, req=11, value=0x0001, index=0, len=0
00:00:02.05 stdata: channel: 0, in: 0, stage: 0
00:00:02.06 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.06 stdata:     buffp: 0x60ba00, bpt: 8, ppt: 1, packets: 1

00:00:02.07 stdata: channel: 0, in: 1, stage: 1
00:00:02.07 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.08 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:02.08 hc: ep=0x60b440, type=0x23, req=3, value=0x0008, index=1, len=0
00:00:02.09 stdata: channel: 0, in: 0, stage: 0
00:00:02.10 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.10 stdata:     buffp: 0x60ba00, bpt: 8, ppt: 1, packets: 1

00:00:02.11 stdata: channel: 0, in: 1, stage: 1
00:00:02.11 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.12 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:02.12 hc: ep=0x60b440, type=0x23, req=3, value=0x0008, index=2, len=0
00:00:02.13 stdata: channel: 0, in: 0, stage: 0
00:00:02.13 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.14 stdata:     buffp: 0x60ba00, bpt: 8, ppt: 1, packets: 1

00:00:02.15 stdata: channel: 0, in: 1, stage: 1
00:00:02.15 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.15 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:02.16 hc: ep=0x60b440, type=0x23, req=3, value=0x0008, index=3, len=0
00:00:02.17 stdata: channel: 0, in: 0, stage: 0
00:00:02.17 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.18 stdata:     buffp: 0x60ba00, bpt: 8, ppt: 1, packets: 1

00:00:02.18 stdata: channel: 0, in: 1, stage: 1
00:00:02.19 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.19 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:02.20 hc: ep=0x60b440, type=0x23, req=3, value=0x0008, index=4, len=0
00:00:02.21 stdata: channel: 0, in: 0, stage: 0
00:00:02.21 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.21 stdata:     buffp: 0x60ba00, bpt: 8, ppt: 1, packets: 1

00:00:02.22 stdata: channel: 0, in: 1, stage: 1
00:00:02.23 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.23 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:02.75 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:02.75 data: Dumping 0x4 bytes starting at 0x60BA00
00:00:02.76 data: BA00: 23 03 08 00 04 00 00 00-34 34 30 2C 20 74 79 70
00:00:02.76 stdata: channel: 0, in: 0, stage: 0
00:00:02.77 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.77 stdata:     buffp: 0x618900, bpt: 8, ppt: 1, packets: 1

00:00:02.78 stdata: channel: 0, in: 1, stage: 0
00:00:02.78 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.79 stdata:     buffp: 0x60ba00, bpt: 4, ppt: 1, packets: 1

00:00:02.79 stdata: channel: 0, in: 0, stage: 1
00:00:02.80 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.80 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:02.81 hc: ep=0x60b440, type=0x23, req=3, value=0x0004, index=1, len=0
00:00:02.82 stdata: channel: 0, in: 0, stage: 0
00:00:02.82 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.83 stdata:     buffp: 0x618900, bpt: 8, ppt: 1, packets: 1

00:00:02.83 stdata: channel: 0, in: 1, stage: 1
00:00:02.84 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.84 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:02.95 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:02.95 data: Dumping 0x4 bytes starting at 0x60BA00
00:00:02.96 data: BA00: 01 01 01 00 04 00 00 00-34 34 30 2C 20 74 79 70
00:00:02.96 stdata: channel: 0, in: 0, stage: 0
00:00:02.97 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.97 stdata:     buffp: 0x618900, bpt: 8, ppt: 1, packets: 1

00:00:02.98 stdata: channel: 0, in: 1, stage: 0
00:00:02.98 stdata:     state: 0, substate: 0, trstatus: 0
00:00:02.99 stdata:     buffp: 0x60ba00, bpt: 4, ppt: 1, packets: 1

00:00:03.00 stdata: channel: 0, in: 0, stage: 1
00:00:03.00 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.01 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.01 hc: ep=0x618900, type=0x80, req=6, value=0x0100, index=0, len=8
00:00:03.02 data: Dumping 0x8 bytes starting at 0x618B80
00:00:03.02 data: 8B80: 20 20 20 20 62 75 66 66-70 3A 20 30 78 36 30 37
00:00:03.03 stdata: channel: 0, in: 0, stage: 0
00:00:03.03 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.04 stdata:     buffp: 0x618a80, bpt: 8, ppt: 1, packets: 1

00:00:03.05 stdata: channel: 0, in: 1, stage: 0
00:00:03.05 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.05 stdata:     buffp: 0x618b80, bpt: 8, ppt: 1, packets: 1

00:00:03.06 stdata: channel: 0, in: 0, stage: 1
00:00:03.07 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.07 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.08 hc: ep=0x618900, type=0x80, req=6, value=0x0100, index=0, len=18
00:00:03.08 data: Dumping 0x12 bytes starting at 0x618B80
00:00:03.09 data: 8B80: 12 01 00 02 09 00 02 40-70 3A 20 30 78 36 30 37
00:00:03.09 data: 8B90: 37 63 30 2C 20 62 70 74-3A 20 30 2C 20 70 70 74
00:00:03.10 stdata: channel: 0, in: 0, stage: 0
00:00:03.11 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.11 stdata:     buffp: 0x618a80, bpt: 8, ppt: 1, packets: 1

00:00:03.12 stdata: channel: 0, in: 1, stage: 0
00:00:03.12 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.13 stdata:     buffp: 0x618b80, bpt: 18, ppt: 1, packets: 1

00:00:03.13 stdata: channel: 0, in: 0, stage: 1
00:00:03.14 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.14 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.15 usbdev: Dumping 0x12 bytes starting at 0x618B80
00:00:03.15 usbdev: 8B80: 12 01 00 02 09 00 02 40-24 04 14 25 B3 0B 00 00
00:00:03.16 usbdev: 8B90: 00 01 25 B8 20 62 70 74-3A 20 30 2C 20 70 70 74
00:00:03.17 hc: ep=0x618900, type=0x0, req=5, value=0x0002, index=0, len=0
00:00:03.17 stdata: channel: 0, in: 0, stage: 0
00:00:03.18 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.18 stdata:     buffp: 0x618a80, bpt: 8, ppt: 1, packets: 1

00:00:03.19 stdata: channel: 0, in: 1, stage: 1
00:00:03.19 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.20 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.25 hc: ep=0x618900, type=0x80, req=6, value=0x0200, index=0, len=9
00:00:03.26 data: Dumping 0x9 bytes starting at 0x618A80
00:00:03.26 data: 8A80: 00 05 02 00 00 00 00 00-39 30 30 2C 20 74 79 70
00:00:03.27 stdata: channel: 0, in: 0, stage: 0
00:00:03.28 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.28 stdata:     buffp: 0x618980, bpt: 8, ppt: 1, packets: 1

00:00:03.29 stdata: channel: 0, in: 1, stage: 0
00:00:03.29 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.30 stdata:     buffp: 0x618a80, bpt: 9, ppt: 1, packets: 1

00:00:03.30 stdata: channel: 0, in: 0, stage: 1
00:00:03.31 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.31 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.32 hc: ep=0x618900, type=0x80, req=6, value=0x0200, index=0, len=41
00:00:03.32 data: Dumping 0x29 bytes starting at 0x618A80
00:00:03.33 data: 8A80: 09 02 29 00 01 01 00 E0-01 12 78 E0 20 74 79 70
00:00:03.34 data: 8A90: 65 3D 30 78 30 2C 20 72-65 71 3D 35 2C 20 76 61
00:00:03.34 data: 8AA0: 6C 75 65 3D 30 78 30 30-30 32 2C 20 69 6E 64 65
00:00:03.35 stdata: channel: 0, in: 0, stage: 0
00:00:03.35 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.36 stdata:     buffp: 0x618980, bpt: 8, ppt: 1, packets: 1

00:00:03.36 stdata: channel: 0, in: 1, stage: 0
00:00:03.37 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.37 stdata:     buffp: 0x618a80, bpt: 41, ppt: 1, packets: 1

00:00:03.38 stdata: channel: 0, in: 0, stage: 1
00:00:03.38 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.39 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.40 usbdev: Dumping 0x29 bytes starting at 0x618A80
00:00:03.40 usbdev: 8A80: 09 02 29 00 01 01 00 E0-01 09 04 00 00 01 09 00
00:00:03.41 usbdev: 8A90: 01 00 07 05 81 03 01 00-0C 09 04 00 01 01 09 00
00:00:03.41 usbdev: 8AA0: 02 00 07 05 81 03 01 00-0C B4 37 00 69 6E 64 65
00:00:03.42 usbdev0-1: Device ven424-2514, dev9-0-2 found
00:00:03.43 usbdev0-1: Interface int9-0-1 found
00:00:03.43 usbdev0-1: Function is not supported
00:00:03.43 usbdev0-1: Interface int9-0-2 found
00:00:03.44 usbdev0-1: Using device/interface int9-0-2
00:00:03.44 hc: ep=0x618900, type=0xa0, req=6, value=0x2900, index=0, len=9
00:00:03.45 data: Dumping 0x9 bytes starting at 0x619600
00:00:03.46 data: 9600: 50 AE 0A 00 00 00 00 00-00 00 00 00 00 00 00 00
00:00:03.46 stdata: channel: 0, in: 0, stage: 0
00:00:03.47 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.47 stdata:     buffp: 0x618c00, bpt: 8, ppt: 1, packets: 1

00:00:03.48 stdata: channel: 0, in: 1, stage: 0
00:00:03.48 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.49 stdata:     buffp: 0x619600, bpt: 9, ppt: 1, packets: 1

00:00:03.49 stdata: channel: 0, in: 0, stage: 1
00:00:03.50 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.50 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.51 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=2, len=4
00:00:03.51 data: Dumping 0x4 bytes starting at 0x619580
00:00:03.52 data: 9580: 40 BE 0A 00 00 00 00 00-00 00 00 00 00 00 00 00
00:00:03.53 stdata: channel: 0, in: 0, stage: 0
00:00:03.53 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.54 stdata:     buffp: 0x618c00, bpt: 8, ppt: 1, packets: 1

00:00:03.54 stdata: channel: 0, in: 1, stage: 0
00:00:03.55 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.55 stdata:     buffp: 0x619580, bpt: 4, ppt: 1, packets: 1

00:00:03.56 stdata: channel: 0, in: 0, stage: 1
00:00:03.56 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.57 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.57 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=3, len=4
00:00:03.58 data: Dumping 0x4 bytes starting at 0x618C00
00:00:03.58 data: 8C00: A3 00 00 00 02 00 04 00-20 42 45 20 30 41 20 30
00:00:03.59 stdata: channel: 0, in: 0, stage: 0
00:00:03.60 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.60 stdata:     buffp: 0x619980, bpt: 8, ppt: 1, packets: 1

00:00:03.61 stdata: channel: 0, in: 1, stage: 0
00:00:03.61 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.62 stdata:     buffp: 0x618c00, bpt: 4, ppt: 1, packets: 1

00:00:03.62 stdata: channel: 0, in: 0, stage: 1
00:00:03.63 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.63 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.64 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=4, len=4
00:00:03.64 data: Dumping 0x4 bytes starting at 0x619980
00:00:03.65 data: 9980: A3 00 00 00 03 00 04 00-20 30 30 20 30 30 20 30
00:00:03.66 stdata: channel: 0, in: 0, stage: 0
00:00:03.66 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.67 stdata:     buffp: 0x618b00, bpt: 8, ppt: 1, packets: 1

00:00:03.67 stdata: channel: 0, in: 1, stage: 0
00:00:03.68 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.68 stdata:     buffp: 0x619980, bpt: 4, ppt: 1, packets: 1

00:00:03.69 stdata: channel: 0, in: 0, stage: 1
00:00:03.69 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.70 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.70 hc: ep=0x618900, type=0x0, req=9, value=0x0001, index=0, len=0
00:00:03.71 stdata: channel: 0, in: 0, stage: 0
00:00:03.71 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.72 stdata:     buffp: 0x618b00, bpt: 8, ppt: 1, packets: 1

00:00:03.73 stdata: channel: 0, in: 1, stage: 1
00:00:03.73 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.73 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.79 hc: ep=0x618900, type=0x1, req=11, value=0x0001, index=0, len=0
00:00:03.80 stdata: channel: 0, in: 0, stage: 0
00:00:03.80 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.81 stdata:     buffp: 0x619680, bpt: 8, ppt: 1, packets: 1

00:00:03.81 stdata: channel: 0, in: 1, stage: 1
00:00:03.82 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.82 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.83 hc: ep=0x618900, type=0x23, req=3, value=0x0008, index=1, len=0
00:00:03.83 stdata: channel: 0, in: 0, stage: 0
00:00:03.84 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.84 stdata:     buffp: 0x619680, bpt: 8, ppt: 1, packets: 1

00:00:03.85 stdata: channel: 0, in: 1, stage: 1
00:00:03.85 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.86 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.87 hc: ep=0x618900, type=0x23, req=3, value=0x0008, index=2, len=0
00:00:03.87 stdata: channel: 0, in: 0, stage: 0
00:00:03.88 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.88 stdata:     buffp: 0x619680, bpt: 8, ppt: 1, packets: 1

00:00:03.89 stdata: channel: 0, in: 1, stage: 1
00:00:03.89 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.90 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:03.90 hc: ep=0x618900, type=0x23, req=3, value=0x0008, index=3, len=0
00:00:03.91 stdata: channel: 0, in: 0, stage: 0
00:00:03.91 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.92 stdata:     buffp: 0x619680, bpt: 8, ppt: 1, packets: 1

00:00:03.93 stdata: channel: 0, in: 1, stage: 1
00:00:03.93 stdata:     state: 0, substate: 0, trstatus: 0
00:00:03.94 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:04.45 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:04.46 data: Dumping 0x4 bytes starting at 0x619680
00:00:04.46 data: 9680: 23 03 08 00 03 00 00 00-39 30 30 2C 20 74 79 70
00:00:04.47 stdata: channel: 0, in: 0, stage: 0
00:00:04.47 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.48 stdata:     buffp: 0x619e40, bpt: 8, ppt: 1, packets: 1

00:00:04.48 stdata: channel: 0, in: 1, stage: 0
00:00:04.49 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.49 stdata:     buffp: 0x619680, bpt: 4, ppt: 1, packets: 1

00:00:04.50 stdata: channel: 0, in: 0, stage: 1
00:00:04.50 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.51 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:04.51 hc: ep=0x618900, type=0x23, req=3, value=0x0004, index=1, len=0
00:00:04.52 stdata: channel: 0, in: 0, stage: 0
00:00:04.53 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.53 stdata:     buffp: 0x619e40, bpt: 8, ppt: 1, packets: 1

00:00:04.54 stdata: channel: 0, in: 1, stage: 1
00:00:04.54 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.55 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:04.65 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:04.66 data: Dumping 0x4 bytes starting at 0x619680
00:00:04.66 data: 9680: 01 01 01 00 03 00 00 00-39 30 30 2C 20 74 79 70
00:00:04.67 stdata: channel: 0, in: 0, stage: 0
00:00:04.67 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.68 stdata:     buffp: 0x619e40, bpt: 8, ppt: 1, packets: 1

00:00:04.68 stdata: channel: 0, in: 1, stage: 0
00:00:04.69 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.69 stdata:     buffp: 0x619680, bpt: 4, ppt: 1, packets: 1

00:00:04.70 stdata: channel: 0, in: 0, stage: 1
00:00:04.70 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.71 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:04.72 hc: ep=0x619e40, type=0x80, req=6, value=0x0100, index=0, len=8
00:00:04.72 data: Dumping 0x8 bytes starting at 0x619800
00:00:04.73 data: 9800: 20 20 20 20 62 75 66 66-70 3A 20 30 78 36 30 37
00:00:04.73 stdata: channel: 0, in: 0, stage: 0
00:00:04.74 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.74 stdata:     buffp: 0x619f40, bpt: 8, ppt: 1, packets: 1

00:00:04.75 stdata: channel: 0, in: 1, stage: 0
00:00:04.75 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.76 stdata:     buffp: 0x619800, bpt: 8, ppt: 1, packets: 1

00:00:04.77 stdata: channel: 0, in: 0, stage: 1
00:00:04.77 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.77 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:04.78 hc: ep=0x619e40, type=0x80, req=6, value=0x0100, index=0, len=18
00:00:04.79 data: Dumping 0x12 bytes starting at 0x619800
00:00:04.79 data: 9800: 12 01 10 02 FF 00 FF 40-70 3A 20 30 78 36 30 37
00:00:04.80 data: 9810: 37 63 30 2C 20 62 70 74-3A 20 30 2C 20 70 70 74
00:00:04.81 stdata: channel: 0, in: 0, stage: 0
00:00:04.81 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.82 stdata:     buffp: 0x619f40, bpt: 8, ppt: 1, packets: 1

00:00:04.82 stdata: channel: 0, in: 1, stage: 0
00:00:04.83 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.83 stdata:     buffp: 0x619800, bpt: 18, ppt: 1, packets: 1

00:00:04.84 stdata: channel: 0, in: 0, stage: 1
00:00:04.84 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.85 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:04.85 usbdev: Dumping 0x12 bytes starting at 0x619800
00:00:04.86 usbdev: 9800: 12 01 10 02 FF 00 FF 40-24 04 00 78 00 03 00 00
00:00:04.86 usbdev: 9810: 00 01 07 A8 20 62 70 74-3A 20 30 2C 20 70 70 74
00:00:04.87 hc: ep=0x619e40, type=0x0, req=5, value=0x0003, index=0, len=0
00:00:04.88 stdata: channel: 0, in: 0, stage: 0
00:00:04.88 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.89 stdata:     buffp: 0x619f40, bpt: 8, ppt: 1, packets: 1

00:00:04.89 stdata: channel: 0, in: 1, stage: 1
00:00:04.90 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.90 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:04.96 hc: ep=0x619e40, type=0x80, req=6, value=0x0200, index=0, len=9
00:00:04.96 data: Dumping 0x9 bytes starting at 0x619F40
00:00:04.97 data: 9F40: 00 05 03 00 00 00 00 00-65 34 30 2C 20 74 79 70
00:00:04.98 stdata: channel: 0, in: 0, stage: 0
00:00:04.98 stdata:     state: 0, substate: 0, trstatus: 0
00:00:04.98 stdata:     buffp: 0x619900, bpt: 8, ppt: 1, packets: 1

00:00:04.99 stdata: channel: 0, in: 1, stage: 0
00:00:05.00 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.00 stdata:     buffp: 0x619f40, bpt: 9, ppt: 1, packets: 1

00:00:05.01 stdata: channel: 0, in: 0, stage: 1
00:00:05.01 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.02 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.02 hc: ep=0x619e40, type=0x80, req=6, value=0x0200, index=0, len=39
00:00:05.03 data: Dumping 0x27 bytes starting at 0x619F40
00:00:05.03 data: 9F40: 09 02 27 00 01 01 00 E0-01 FD B8 E0 20 74 79 70
00:00:05.04 data: 9F50: 65 3D 30 78 30 2C 20 72-65 71 3D 35 2C 20 76 61
00:00:05.05 data: 9F60: 6C 75 65 3D 30 78 30 30-30 33 2C 20 69 6E 64 65
00:00:05.05 stdata: channel: 0, in: 0, stage: 0
00:00:05.06 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.06 stdata:     buffp: 0x619900, bpt: 8, ppt: 1, packets: 1

00:00:05.07 stdata: channel: 0, in: 1, stage: 0
00:00:05.07 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.08 stdata:     buffp: 0x619f40, bpt: 39, ppt: 1, packets: 1

00:00:05.08 stdata: channel: 0, in: 0, stage: 1
00:00:05.09 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.09 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.10 usbdev: Dumping 0x27 bytes starting at 0x619F40
00:00:05.11 usbdev: 9F40: 09 02 27 00 01 01 00 E0-01 09 04 00 00 03 FF 00
00:00:05.11 usbdev: 9F50: FF 00 07 05 81 02 00 02-00 07 05 02 02 00 02 00
00:00:05.12 usbdev: 9F60: 07 05 83 03 10 00 04 E9-30 33 2C 20 69 6E 64 65
00:00:05.13 usbdev0-1: Device ven424-7800 found
00:00:05.13 usbdev0-1: Using device/interface ven424-7800
00:00:05.13 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=2, len=4
00:00:05.14 data: Dumping 0x4 bytes starting at 0x61A480
00:00:05.15 data: A480: 40 BE 0A 00 00 00 00 00-00 00 00 00 00 00 00 00
00:00:05.15 stdata: channel: 0, in: 0, stage: 0
00:00:05.16 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.16 stdata:     buffp: 0x61a500, bpt: 8, ppt: 1, packets: 1

00:00:05.17 stdata: channel: 0, in: 1, stage: 0
00:00:05.17 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.18 stdata:     buffp: 0x61a480, bpt: 4, ppt: 1, packets: 1

00:00:05.18 stdata: channel: 0, in: 0, stage: 1
00:00:05.19 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.19 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.20 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=3, len=4
00:00:05.21 data: Dumping 0x4 bytes starting at 0x61A500
00:00:05.21 data: A500: A3 00 00 00 02 00 04 00-20 42 45 20 30 41 20 30
00:00:05.22 stdata: channel: 0, in: 0, stage: 0
00:00:05.22 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.23 stdata:     buffp: 0x619fc0, bpt: 8, ppt: 1, packets: 1

00:00:05.23 stdata: channel: 0, in: 1, stage: 0
00:00:05.24 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.24 stdata:     buffp: 0x61a500, bpt: 4, ppt: 1, packets: 1

00:00:05.25 stdata: channel: 0, in: 0, stage: 1
00:00:05.25 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.26 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.26 hc: ep=0x619e40, type=0x0, req=9, value=0x0001, index=0, len=0
00:00:05.27 stdata: channel: 0, in: 0, stage: 0
00:00:05.28 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.28 stdata:     buffp: 0x619fc0, bpt: 8, ppt: 1, packets: 1

00:00:05.29 stdata: channel: 0, in: 1, stage: 1
00:00:05.29 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.30 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.35 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=0, len=4
00:00:05.36 data: Dumping 0x4 bytes starting at 0x29D808
00:00:05.36 data: D808: 08 16 08 00 00 00 00 00-40 D8 29 00 00 00 00 00
00:00:05.37 stdata: channel: 0, in: 0, stage: 0
00:00:05.37 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.38 stdata:     buffp: 0x61a580, bpt: 8, ppt: 1, packets: 1

00:00:05.38 stdata: channel: 0, in: 1, stage: 0
00:00:05.39 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.39 stdata:     buffp: 0x29d808, bpt: 4, ppt: 1, packets: 1

00:00:05.40 stdata: channel: 0, in: 0, stage: 1
00:00:05.40 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.41 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.42 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=16, len=4
00:00:05.42 data: Dumping 0x4 bytes starting at 0x29D80C
00:00:05.43 data: D80C: 00 00 00 00 40 D8 29 00-00 00 00 00 48 A8 08 00
00:00:05.43 stdata: channel: 0, in: 0, stage: 0
00:00:05.44 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.44 stdata:     buffp: 0x61a580, bpt: 8, ppt: 1, packets: 1

00:00:05.45 stdata: channel: 0, in: 1, stage: 0
00:00:05.45 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.46 stdata:     buffp: 0x29d80c, bpt: 4, ppt: 1, packets: 1

00:00:05.47 stdata: channel: 0, in: 0, stage: 1
00:00:05.47 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.47 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.48 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=16, len=4
00:00:05.49 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:05.49 data: D7CC: 02 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:05.50 stdata: channel: 0, in: 0, stage: 0
00:00:05.50 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.51 stdata:     buffp: 0x61a580, bpt: 8, ppt: 1, packets: 1

00:00:05.52 stdata: channel: 0, in: 0, stage: 0
00:00:05.52 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.52 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:05.53 stdata: channel: 0, in: 1, stage: 1
00:00:05.53 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.54 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.55 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=16, len=4
00:00:05.55 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:05.56 data: D7CC: 02 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:05.56 stdata: channel: 0, in: 0, stage: 0
00:00:05.57 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.57 stdata:     buffp: 0x61a580, bpt: 8, ppt: 1, packets: 1

00:00:05.58 stdata: channel: 0, in: 1, stage: 0
00:00:05.58 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.59 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:05.60 stdata: channel: 0, in: 0, stage: 1
00:00:05.60 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.61 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.61 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=284, len=4
00:00:05.62 data: Dumping 0x4 bytes starting at 0x29D75C
00:00:05.62 data: D75C: B8 27 EB AB D0 D7 29 00-00 00 00 00 78 40 09 00
00:00:05.63 stdata: channel: 0, in: 0, stage: 0
00:00:05.63 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.64 stdata:     buffp: 0x61a580, bpt: 8, ppt: 1, packets: 1

00:00:05.65 stdata: channel: 0, in: 0, stage: 0
00:00:05.65 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.65 stdata:     buffp: 0x29d75c, bpt: 4, ppt: 1, packets: 1

00:00:05.66 stdata: channel: 0, in: 1, stage: 1
00:00:05.67 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.67 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.68 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=280, len=4
00:00:05.68 data: Dumping 0x4 bytes starting at 0x29D75C
00:00:05.69 data: D75C: E8 48 00 00 D0 D7 29 00-00 00 00 00 78 40 09 00
00:00:05.70 stdata: channel: 0, in: 0, stage: 0
00:00:05.70 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.70 stdata:     buffp: 0x61a580, bpt: 8, ppt: 1, packets: 1

00:00:05.71 stdata: channel: 0, in: 0, stage: 0
00:00:05.72 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.72 stdata:     buffp: 0x29d75c, bpt: 4, ppt: 1, packets: 1

00:00:05.73 stdata: channel: 0, in: 1, stage: 1
00:00:05.73 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.74 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.74 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=1028, len=4
00:00:05.75 data: Dumping 0x4 bytes starting at 0x29D75C
00:00:05.75 data: D75C: B8 27 EB AB D0 D7 29 00-00 00 00 00 78 40 09 00
00:00:05.76 stdata: channel: 0, in: 0, stage: 0
00:00:05.76 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.77 stdata:     buffp: 0x61a880, bpt: 8, ppt: 1, packets: 1

00:00:05.78 stdata: channel: 0, in: 0, stage: 0
00:00:05.78 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.79 stdata:     buffp: 0x29d75c, bpt: 4, ppt: 1, packets: 1

00:00:05.79 stdata: channel: 0, in: 1, stage: 1
00:00:05.80 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.80 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.81 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=1024, len=4
00:00:05.81 data: Dumping 0x4 bytes starting at 0x29D75C
00:00:05.82 data: D75C: E8 48 00 80 D0 D7 29 00-00 00 00 00 78 40 09 00
00:00:05.83 stdata: channel: 0, in: 0, stage: 0
00:00:05.83 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.84 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:05.84 stdata: channel: 0, in: 0, stage: 0
00:00:05.85 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.85 stdata:     buffp: 0x29d75c, bpt: 4, ppt: 1, packets: 1

00:00:05.86 stdata: channel: 0, in: 1, stage: 1
00:00:05.86 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.87 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.87 lan7800: MAC address is B8:27:EB:AB:E8:48
00:00:05.88 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=144, len=4
00:00:05.88 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:05.89 data: D7CC: 18 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:05.90 stdata: channel: 0, in: 0, stage: 0
00:00:05.90 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.91 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:05.91 stdata: channel: 0, in: 0, stage: 0
00:00:05.92 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.92 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:05.93 stdata: channel: 0, in: 1, stage: 1
00:00:05.93 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.94 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:05.94 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=148, len=4
00:00:05.95 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:05.95 data: D7CC: 00 08 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:05.96 stdata: channel: 0, in: 0, stage: 0
00:00:05.97 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.97 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:05.98 stdata: channel: 0, in: 0, stage: 0
00:00:05.98 stdata:     state: 0, substate: 0, trstatus: 0
00:00:05.99 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:05.99 stdata: channel: 0, in: 1, stage: 1
00:00:06.00 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.00 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.01 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=16, len=4
00:00:06.02 data: Dumping 0x4 bytes starting at 0x29D80C
00:00:06.02 data: D80C: 02 00 00 00 40 D8 29 00-00 00 00 00 48 A8 08 00
00:00:06.03 stdata: channel: 0, in: 0, stage: 0
00:00:06.03 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.04 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.04 stdata: channel: 0, in: 1, stage: 0
00:00:06.05 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.05 stdata:     buffp: 0x29d80c, bpt: 4, ppt: 1, packets: 1

00:00:06.06 stdata: channel: 0, in: 0, stage: 1
00:00:06.06 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.07 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.07 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=16, len=4
00:00:06.08 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.09 data: D7CC: 00 00 30 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.09 stdata: channel: 0, in: 0, stage: 0
00:00:06.10 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.10 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.11 stdata: channel: 0, in: 0, stage: 0
00:00:06.11 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.12 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.12 stdata: channel: 0, in: 1, stage: 1
00:00:06.13 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.13 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.14 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=128, len=4
00:00:06.15 data: Dumping 0x4 bytes starting at 0x29D80C
00:00:06.15 data: D80C: 00 00 30 00 40 D8 29 00-00 00 00 00 48 A8 08 00
00:00:06.16 stdata: channel: 0, in: 0, stage: 0
00:00:06.16 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.17 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.17 stdata: channel: 0, in: 1, stage: 0
00:00:06.18 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.18 stdata:     buffp: 0x29d80c, bpt: 4, ppt: 1, packets: 1

00:00:06.19 stdata: channel: 0, in: 0, stage: 1
00:00:06.19 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.20 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.20 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=128, len=4
00:00:06.21 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.22 data: D7CC: 26 81 76 60 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.22 stdata: channel: 0, in: 0, stage: 0
00:00:06.23 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.23 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.24 stdata: channel: 0, in: 0, stage: 0
00:00:06.24 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.25 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.25 stdata: channel: 0, in: 1, stage: 1
00:00:06.26 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.26 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.27 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=200, len=4
00:00:06.28 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.28 data: D7CC: 17 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.29 stdata: channel: 0, in: 0, stage: 0
00:00:06.29 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.30 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.30 stdata: channel: 0, in: 0, stage: 0
00:00:06.31 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.31 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.32 stdata: channel: 0, in: 1, stage: 1
00:00:06.32 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.33 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.33 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=204, len=4
00:00:06.34 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.35 data: D7CC: 17 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.35 stdata: channel: 0, in: 0, stage: 0
00:00:06.36 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.36 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.37 stdata: channel: 0, in: 0, stage: 0
00:00:06.37 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.38 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.38 stdata: channel: 0, in: 1, stage: 1
00:00:06.39 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.39 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.40 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=152, len=4
00:00:06.41 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.41 data: D7CC: 00 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.42 stdata: channel: 0, in: 0, stage: 0
00:00:06.42 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.43 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.43 stdata: channel: 0, in: 0, stage: 0
00:00:06.44 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.44 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.45 stdata: channel: 0, in: 1, stage: 1
00:00:06.45 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.46 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.47 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=12, len=4
00:00:06.47 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.48 data: D7CC: FF FF FF FF 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.48 stdata: channel: 0, in: 0, stage: 0
00:00:06.49 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.49 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.50 stdata: channel: 0, in: 0, stage: 0
00:00:06.50 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.51 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.51 stdata: channel: 0, in: 1, stage: 1
00:00:06.52 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.52 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.53 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=268, len=4
00:00:06.54 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.54 data: D7CC: 00 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.55 stdata: channel: 0, in: 0, stage: 0
00:00:06.55 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.56 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.56 stdata: channel: 0, in: 0, stage: 0
00:00:06.57 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.57 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.58 stdata: channel: 0, in: 1, stage: 1
00:00:06.58 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.59 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.60 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=208, len=4
00:00:06.60 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.61 data: D7CC: 00 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.61 stdata: channel: 0, in: 0, stage: 0
00:00:06.62 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.62 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.63 stdata: channel: 0, in: 0, stage: 0
00:00:06.63 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.64 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.65 stdata: channel: 0, in: 1, stage: 1
00:00:06.65 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.65 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.66 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=176, len=4
00:00:06.67 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.67 data: D7CC: 00 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.68 stdata: channel: 0, in: 0, stage: 0
00:00:06.68 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.69 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.69 stdata: channel: 0, in: 1, stage: 0
00:00:06.70 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.70 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.71 stdata: channel: 0, in: 0, stage: 1
00:00:06.71 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.72 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.73 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=176, len=4
00:00:06.73 data: Dumping 0x4 bytes starting at 0x29D78C
00:00:06.74 data: D78C: 02 04 00 00 D0 D7 29 00-00 00 00 00 CC 41 09 00
00:00:06.74 stdata: channel: 0, in: 0, stage: 0
00:00:06.75 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.75 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.76 stdata: channel: 0, in: 0, stage: 0
00:00:06.76 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.77 stdata:     buffp: 0x29d78c, bpt: 4, ppt: 1, packets: 1

00:00:06.78 stdata: channel: 0, in: 1, stage: 1
00:00:06.78 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.79 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.79 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=20, len=4
00:00:06.80 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.80 data: D7CC: 02 04 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.81 stdata: channel: 0, in: 0, stage: 0
00:00:06.81 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.82 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.83 stdata: channel: 0, in: 1, stage: 0
00:00:06.83 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.83 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.84 stdata: channel: 0, in: 0, stage: 1
00:00:06.85 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.85 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.86 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=20, len=4
00:00:06.86 data: Dumping 0x4 bytes starting at 0x29D78C
00:00:06.87 data: D78C: D0 41 00 00 D0 D7 29 00-00 00 00 00 E8 41 09 00
00:00:06.87 stdata: channel: 0, in: 0, stage: 0
00:00:06.88 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.88 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.89 stdata: channel: 0, in: 0, stage: 0
00:00:06.89 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.90 stdata:     buffp: 0x29d78c, bpt: 4, ppt: 1, packets: 1

00:00:06.91 stdata: channel: 0, in: 1, stage: 1
00:00:06.91 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.92 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.92 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=20, len=4
00:00:06.93 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:06.93 data: D7CC: D0 41 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:06.94 stdata: channel: 0, in: 0, stage: 0
00:00:06.94 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.95 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:06.96 stdata: channel: 0, in: 1, stage: 0
00:00:06.96 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.97 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:06.97 stdata: channel: 0, in: 0, stage: 1
00:00:06.98 stdata:     state: 0, substate: 0, trstatus: 0
00:00:06.98 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:06.99 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=20, len=4
00:00:06.99 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:07.00 data: D7CC: 50 41 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:07.01 stdata: channel: 0, in: 0, stage: 0
00:00:07.01 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.01 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.02 stdata: channel: 0, in: 1, stage: 0
00:00:07.03 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.03 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:07.04 stdata: channel: 0, in: 0, stage: 1
00:00:07.04 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.05 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.05 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=256, len=4
00:00:07.06 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:07.06 data: D7CC: C0 41 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:07.07 stdata: channel: 0, in: 0, stage: 0
00:00:07.07 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.08 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.09 stdata: channel: 0, in: 1, stage: 0
00:00:07.09 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.10 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:07.10 stdata: channel: 0, in: 0, stage: 1
00:00:07.11 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.11 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.12 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=256, len=4
00:00:07.12 data: Dumping 0x4 bytes starting at 0x29D78C
00:00:07.13 data: D78C: 00 38 00 00 D0 D7 29 00-00 00 00 00 28 42 09 00
00:00:07.14 stdata: channel: 0, in: 0, stage: 0
00:00:07.14 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.15 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.15 stdata: channel: 0, in: 0, stage: 0
00:00:07.16 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.16 stdata:     buffp: 0x29d78c, bpt: 4, ppt: 1, packets: 1

00:00:07.17 stdata: channel: 0, in: 1, stage: 1
00:00:07.17 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.18 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.18 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=264, len=4
00:00:07.19 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:07.19 data: D7CC: 00 38 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:07.20 stdata: channel: 0, in: 0, stage: 0
00:00:07.21 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.21 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.22 stdata: channel: 0, in: 1, stage: 0
00:00:07.22 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.23 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:07.23 stdata: channel: 0, in: 0, stage: 1
00:00:07.24 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.24 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.25 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=264, len=4
00:00:07.26 data: Dumping 0x4 bytes starting at 0x29D78C
00:00:07.26 data: D78C: 01 00 00 00 D0 D7 29 00-00 00 00 00 44 42 09 00
00:00:07.27 stdata: channel: 0, in: 0, stage: 0
00:00:07.27 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.28 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.28 stdata: channel: 0, in: 0, stage: 0
00:00:07.29 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.29 stdata:     buffp: 0x29d78c, bpt: 4, ppt: 1, packets: 1

00:00:07.30 stdata: channel: 0, in: 1, stage: 1
00:00:07.30 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.31 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.31 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=196, len=4
00:00:07.32 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:07.33 data: D7CC: 01 00 00 00 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:07.33 stdata: channel: 0, in: 0, stage: 0
00:00:07.34 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.34 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.35 stdata: channel: 0, in: 1, stage: 0
00:00:07.35 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.36 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:07.36 stdata: channel: 0, in: 0, stage: 1
00:00:07.37 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.37 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.38 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=196, len=4
00:00:07.39 data: Dumping 0x4 bytes starting at 0x29D78C
00:00:07.39 data: D78C: 00 00 00 80 D0 D7 29 00-00 00 00 00 60 42 09 00
00:00:07.40 stdata: channel: 0, in: 0, stage: 0
00:00:07.40 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.41 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.41 stdata: channel: 0, in: 0, stage: 0
00:00:07.42 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.42 stdata:     buffp: 0x29d78c, bpt: 4, ppt: 1, packets: 1

00:00:07.43 stdata: channel: 0, in: 1, stage: 1
00:00:07.43 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.44 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.44 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=260, len=4
00:00:07.45 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:07.46 data: D7CC: 00 00 00 80 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:07.46 stdata: channel: 0, in: 0, stage: 0
00:00:07.47 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.47 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.48 stdata: channel: 0, in: 1, stage: 0
00:00:07.48 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.49 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:07.49 stdata: channel: 0, in: 0, stage: 1
00:00:07.50 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.50 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.51 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=260, len=4
00:00:07.52 data: Dumping 0x4 bytes starting at 0x29D78C
00:00:07.52 data: D78C: 21 00 EE 05 D0 D7 29 00-00 00 00 00 80 42 09 00
00:00:07.53 stdata: channel: 0, in: 0, stage: 0
00:00:07.53 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.54 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.54 stdata: channel: 0, in: 0, stage: 0
00:00:07.55 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.55 stdata:     buffp: 0x29d78c, bpt: 4, ppt: 1, packets: 1

00:00:07.56 stdata: channel: 0, in: 1, stage: 1
00:00:07.56 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.57 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.57 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=192, len=4
00:00:07.58 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:07.59 data: D7CC: 21 00 EE 05 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:07.59 stdata: channel: 0, in: 0, stage: 0
00:00:07.60 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.60 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.61 stdata: channel: 0, in: 1, stage: 0
00:00:07.61 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.62 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:07.62 stdata: channel: 0, in: 0, stage: 1
00:00:07.63 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.63 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.64 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=192, len=4
00:00:07.65 data: Dumping 0x4 bytes starting at 0x29D78C
00:00:07.65 data: D78C: 00 00 00 80 D0 D7 29 00-00 00 00 00 9C 42 09 00
00:00:07.66 stdata: channel: 0, in: 0, stage: 0
00:00:07.66 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.67 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.67 stdata: channel: 0, in: 0, stage: 0
00:00:07.68 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.68 stdata:     buffp: 0x29d78c, bpt: 4, ppt: 1, packets: 1

00:00:07.69 stdata: channel: 0, in: 1, stage: 1
00:00:07.69 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.70 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.71 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:07.71 data: Dumping 0x4 bytes starting at 0x29D76C
00:00:07.72 data: D76C: 00 00 00 00 A0 D7 29 00-00 00 00 00 3C 3C 09 00
00:00:07.72 stdata: channel: 0, in: 0, stage: 0
00:00:07.73 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.73 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.74 stdata: channel: 0, in: 1, stage: 0
00:00:07.74 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.75 stdata:     buffp: 0x29d76c, bpt: 4, ppt: 1, packets: 1

00:00:07.75 stdata: channel: 0, in: 0, stage: 1
00:00:07.76 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.76 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.77 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=292, len=4
00:00:07.78 data: Dumping 0x4 bytes starting at 0x29D76C
00:00:07.78 data: D76C: 00 00 00 00 A0 D7 29 00-00 00 00 00 3C 3C 09 00
00:00:07.79 stdata: channel: 0, in: 0, stage: 0
00:00:07.79 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.80 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.80 stdata: channel: 0, in: 0, stage: 0
00:00:07.81 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.81 stdata:     buffp: 0x29d76c, bpt: 4, ppt: 1, packets: 1

00:00:07.82 stdata: channel: 0, in: 1, stage: 1
00:00:07.82 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.83 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.84 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=288, len=4
00:00:07.84 data: Dumping 0x4 bytes starting at 0x29D76C
00:00:07.85 data: D76C: C3 0F 00 00 A0 D7 29 00-00 00 00 00 3C 3C 09 00
00:00:07.85 stdata: channel: 0, in: 0, stage: 0
00:00:07.86 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.86 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.87 stdata: channel: 0, in: 0, stage: 0
00:00:07.87 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.88 stdata:     buffp: 0x29d76c, bpt: 4, ppt: 1, packets: 1

00:00:07.89 stdata: channel: 0, in: 1, stage: 1
00:00:07.89 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.89 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.90 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:07.91 data: Dumping 0x4 bytes starting at 0x29D79C
00:00:07.91 data: D79C: 00 00 00 00 D0 D7 29 00-00 00 00 00 AC 42 09 00
00:00:07.92 stdata: channel: 0, in: 0, stage: 0
00:00:07.92 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.93 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:07.94 stdata: channel: 0, in: 1, stage: 0
00:00:07.94 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.94 stdata:     buffp: 0x29d79c, bpt: 4, ppt: 1, packets: 1

00:00:07.95 stdata: channel: 0, in: 0, stage: 1
00:00:07.95 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.96 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:07.97 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:07.97 data: Dumping 0x4 bytes starting at 0x29D75C
00:00:07.98 data: D75C: 00 00 00 00 A0 D7 29 00-00 00 00 00 60 3C 09 00
00:00:07.98 stdata: channel: 0, in: 0, stage: 0
00:00:07.99 stdata:     state: 0, substate: 0, trstatus: 0
00:00:07.99 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:08.00 stdata: channel: 0, in: 1, stage: 0
00:00:08.00 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.01 stdata:     buffp: 0x29d75c, bpt: 4, ppt: 1, packets: 1

00:00:08.02 stdata: channel: 0, in: 0, stage: 1
00:00:08.02 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.03 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.03 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=288, len=4
00:00:08.04 data: Dumping 0x4 bytes starting at 0x29D75C
00:00:08.04 data: D75C: 41 0F 00 00 A0 D7 29 00-00 00 00 00 60 3C 09 00
00:00:08.05 stdata: channel: 0, in: 0, stage: 0
00:00:08.05 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.06 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:08.07 stdata: channel: 0, in: 0, stage: 0
00:00:08.07 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.08 stdata:     buffp: 0x29d75c, bpt: 4, ppt: 1, packets: 1

00:00:08.08 stdata: channel: 0, in: 1, stage: 1
00:00:08.09 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.09 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.10 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:08.10 data: Dumping 0x4 bytes starting at 0x29D75C
00:00:08.11 data: D75C: 41 0F 00 00 A0 D7 29 00-00 00 00 00 60 3C 09 00
00:00:08.12 stdata: channel: 0, in: 0, stage: 0
00:00:08.12 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.12 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:08.13 stdata: channel: 0, in: 1, stage: 0
00:00:08.14 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.14 stdata:     buffp: 0x29d75c, bpt: 4, ppt: 1, packets: 1

00:00:08.15 stdata: channel: 0, in: 0, stage: 1
00:00:08.15 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.16 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.16 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=292, len=4
00:00:08.17 data: Dumping 0x4 bytes starting at 0x29D79C
00:00:08.17 data: D79C: C2 0F 00 00 D0 D7 29 00-00 00 00 00 AC 42 09 00
00:00:08.18 stdata: channel: 0, in: 0, stage: 0
00:00:08.18 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.19 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:08.20 stdata: channel: 0, in: 1, stage: 0
00:00:08.20 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.21 stdata:     buffp: 0x29d79c, bpt: 4, ppt: 1, packets: 1

00:00:08.21 stdata: channel: 0, in: 0, stage: 1
00:00:08.22 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.22 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.23 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:08.23 data: Dumping 0x4 bytes starting at 0x29D79C
00:00:08.24 data: D79C: 21 80 00 00 D0 D7 29 00-00 00 00 00 AC 42 09 00
00:00:08.25 stdata: channel: 0, in: 0, stage: 0
00:00:08.25 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.26 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:08.26 stdata: channel: 0, in: 1, stage: 0
00:00:08.27 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.27 stdata:     buffp: 0x29d79c, bpt: 4, ppt: 1, packets: 1

00:00:08.28 stdata: channel: 0, in: 0, stage: 1
00:00:08.28 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.29 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.29 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=292, len=4
00:00:08.30 data: Dumping 0x4 bytes starting at 0x29D79C
00:00:08.30 data: D79C: 61 80 00 00 D0 D7 29 00-00 00 00 00 AC 42 09 00
00:00:08.31 stdata: channel: 0, in: 0, stage: 0
00:00:08.32 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.32 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:08.33 stdata: channel: 0, in: 0, stage: 0
00:00:08.33 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.34 stdata:     buffp: 0x29d79c, bpt: 4, ppt: 1, packets: 1

00:00:08.34 stdata: channel: 0, in: 1, stage: 1
00:00:08.35 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.35 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.36 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=288, len=4
00:00:08.37 data: Dumping 0x4 bytes starting at 0x29D79C
00:00:08.37 data: D79C: 43 0F 00 00 D0 D7 29 00-00 00 00 00 AC 42 09 00
00:00:08.38 stdata: channel: 0, in: 0, stage: 0
00:00:08.38 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.39 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:08.39 stdata: channel: 0, in: 0, stage: 0
00:00:08.40 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.40 stdata:     buffp: 0x29d79c, bpt: 4, ppt: 1, packets: 1

00:00:08.41 stdata: channel: 0, in: 1, stage: 1
00:00:08.41 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.42 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.42 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:08.43 data: Dumping 0x4 bytes starting at 0x29D7CC
00:00:08.44 data: D7CC: 00 00 21 80 10 D8 29 00-00 00 00 00 34 16 08 00
00:00:08.44 stdata: channel: 0, in: 0, stage: 0
00:00:08.45 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.45 stdata:     buffp: 0x61a700, bpt: 8, ppt: 1, packets: 1

00:00:08.46 stdata: channel: 0, in: 1, stage: 0
00:00:08.46 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.47 stdata:     buffp: 0x29d7cc, bpt: 4, ppt: 1, packets: 1

00:00:08.47 stdata: channel: 0, in: 0, stage: 1
00:00:08.48 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.48 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.49 usbhub: Port 1: Device ven424-7800 configured
00:00:08.49 hc: ep=0x618900, type=0xa0, req=0, value=0x0000, index=0, len=4
00:00:08.50 data: Dumping 0x4 bytes starting at 0x61A680
00:00:08.51 data: A680: 50 6F 72 74 20 31 3A 20-44 65 76 69 63 65 20 76
00:00:08.51 stdata: channel: 0, in: 0, stage: 0
00:00:08.52 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.52 stdata:     buffp: 0x61a800, bpt: 8, ppt: 1, packets: 1

00:00:08.53 stdata: channel: 0, in: 1, stage: 0
00:00:08.53 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.54 stdata:     buffp: 0x61a680, bpt: 4, ppt: 1, packets: 1

00:00:08.54 stdata: channel: 0, in: 0, stage: 1
00:00:08.55 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.55 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.56 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:08.57 data: Dumping 0x4 bytes starting at 0x619680
00:00:08.57 data: 9680: 03 05 11 00 03 00 00 00-39 30 30 2C 20 74 79 70
00:00:08.58 stdata: channel: 0, in: 0, stage: 0
00:00:08.58 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.59 stdata:     buffp: 0x61a680, bpt: 8, ppt: 1, packets: 1

00:00:08.59 stdata: channel: 0, in: 1, stage: 0
00:00:08.60 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.60 stdata:     buffp: 0x619680, bpt: 4, ppt: 1, packets: 1

00:00:08.61 stdata: channel: 0, in: 0, stage: 1
00:00:08.61 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.62 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.62 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=2, len=4
00:00:08.63 data: Dumping 0x4 bytes starting at 0x61A480
00:00:08.64 data: A480: 00 01 00 00 00 00 00 00-00 00 00 00 00 00 00 00
00:00:08.64 stdata: channel: 0, in: 0, stage: 0
00:00:08.65 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.65 stdata:     buffp: 0x61a680, bpt: 8, ppt: 1, packets: 1

00:00:08.66 stdata: channel: 0, in: 1, stage: 0
00:00:08.66 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.67 stdata:     buffp: 0x61a480, bpt: 4, ppt: 1, packets: 1

00:00:08.67 stdata: channel: 0, in: 0, stage: 1
00:00:08.68 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.68 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.69 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=3, len=4
00:00:08.70 data: Dumping 0x4 bytes starting at 0x61A500
00:00:08.70 data: A500: 00 01 00 00 02 00 04 00-20 42 45 20 30 41 20 30
00:00:08.71 stdata: channel: 0, in: 0, stage: 0
00:00:08.71 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.72 stdata:     buffp: 0x61a680, bpt: 8, ppt: 1, packets: 1

00:00:08.72 stdata: channel: 0, in: 1, stage: 0
00:00:08.73 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.73 stdata:     buffp: 0x61a500, bpt: 4, ppt: 1, packets: 1

00:00:08.74 stdata: channel: 0, in: 0, stage: 1
00:00:08.74 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.75 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.75 usbhub: Port 1: Device ven424-2514, dev9-0-2 configured
00:00:08.76 hc: ep=0x60b440, type=0xa0, req=0, value=0x0000, index=0, len=4
00:00:08.77 data: Dumping 0x4 bytes starting at 0x61A800
00:00:08.77 data: A800: 50 6F 72 74 20 31 3A 20-44 65 76 69 63 65 20 76
00:00:08.78 stdata: channel: 0, in: 0, stage: 0
00:00:08.78 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.79 stdata:     buffp: 0x61a880, bpt: 8, ppt: 1, packets: 1

00:00:08.79 stdata: channel: 0, in: 1, stage: 0
00:00:08.80 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.80 stdata:     buffp: 0x61a800, bpt: 4, ppt: 1, packets: 1

00:00:08.81 stdata: channel: 0, in: 0, stage: 1
00:00:08.81 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.82 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.82 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:08.83 data: Dumping 0x4 bytes starting at 0x60BA00
00:00:08.84 data: BA00: 03 05 11 00 04 00 00 00-34 34 30 2C 20 74 79 70
00:00:08.84 stdata: channel: 0, in: 0, stage: 0
00:00:08.85 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.85 stdata:     buffp: 0x61a800, bpt: 8, ppt: 1, packets: 1

00:00:08.86 stdata: channel: 0, in: 1, stage: 0
00:00:08.86 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.87 stdata:     buffp: 0x60ba00, bpt: 4, ppt: 1, packets: 1

00:00:08.87 stdata: channel: 0, in: 0, stage: 1
00:00:08.88 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.88 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.89 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=2, len=4
00:00:08.90 data: Dumping 0x4 bytes starting at 0x619580
00:00:08.90 data: 9580: 00 01 00 00 00 00 00 00-00 00 00 00 00 00 00 00
00:00:08.91 stdata: channel: 0, in: 0, stage: 0
00:00:08.91 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.92 stdata:     buffp: 0x61a800, bpt: 8, ppt: 1, packets: 1

00:00:08.92 stdata: channel: 0, in: 1, stage: 0
00:00:08.93 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.93 stdata:     buffp: 0x619580, bpt: 4, ppt: 1, packets: 1

00:00:08.94 stdata: channel: 0, in: 0, stage: 1
00:00:08.94 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.95 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:08.95 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=3, len=4
00:00:08.96 data: Dumping 0x4 bytes starting at 0x618C00
00:00:08.97 data: 8C00: 00 01 00 00 02 00 04 00-20 42 45 20 30 41 20 30
00:00:08.97 stdata: channel: 0, in: 0, stage: 0
00:00:08.98 stdata:     state: 0, substate: 0, trstatus: 0
00:00:08.98 stdata:     buffp: 0x61a800, bpt: 8, ppt: 1, packets: 1

00:00:08.99 stdata: channel: 0, in: 1, stage: 0
00:00:08.99 stdata:     state: 0, substate: 0, trstatus: 0
00:00:09.00 stdata:     buffp: 0x618c00, bpt: 4, ppt: 1, packets: 1

00:00:09.00 stdata: channel: 0, in: 0, stage: 1
00:00:09.01 stdata:     state: 0, substate: 0, trstatus: 0
00:00:09.01 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:09.02 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=4, len=4
00:00:09.03 data: Dumping 0x4 bytes starting at 0x619980
00:00:09.03 data: 9980: 00 01 00 00 03 00 04 00-20 30 30 20 30 30 20 30
00:00:09.04 stdata: channel: 0, in: 0, stage: 0
00:00:09.04 stdata:     state: 0, substate: 0, trstatus: 0
00:00:09.05 stdata:     buffp: 0x61a800, bpt: 8, ppt: 1, packets: 1

00:00:09.05 stdata: channel: 0, in: 1, stage: 0
00:00:09.06 stdata:     state: 0, substate: 0, trstatus: 0
00:00:09.06 stdata:     buffp: 0x619980, bpt: 4, ppt: 1, packets: 1

00:00:09.07 stdata: channel: 0, in: 0, stage: 1
00:00:09.07 stdata:     state: 0, substate: 0, trstatus: 0
00:00:09.08 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:09.08 dwroot: Device configured
00:00:09.09 kernel: Compile time: Nov 28 2022 17:23:20
00:00:09.09 hc: ep=0x60b440, type=0x80, req=6, value=0x0100, index=0, len=18
00:00:09.10 data: Dumping 0x12 bytes starting at 0x29DA40
00:00:09.10 data: DA40: 0E 00 00 00 00 00 00 00-0D 00 00 00 00 00 00 00
00:00:09.11 data: DA50: F0 D5 0A 00 00 00 00 00-F0 09 08 00 00 00 00 00
00:00:09.12 stdata: channel: 0, in: 0, stage: 0
00:00:09.12 stdata:     state: 0, substate: 0, trstatus: 0
00:00:09.13 stdata:     buffp: 0x61a800, bpt: 8, ppt: 1, packets: 1

00:00:09.13 stdata: channel: 0, in: 1, stage: 0
00:00:09.14 stdata:     state: 0, substate: 0, trstatus: 0
00:00:09.14 stdata:     buffp: 0x29da40, bpt: 18, ppt: 1, packets: 1

00:00:09.15 stdata: channel: 0, in: 0, stage: 1
00:00:09.15 stdata:     state: 0, substate: 0, trstatus: 0
00:00:09.16 stdata:     buffp: 0x6077c0, bpt: 0, ppt: 1, packets: 1

00:00:09.16 kernel: USB hub: Vendor 0x0424, Product 0x2514
00:00:09.17 kernel: Dumping device descriptor
00:00:09.17 kernel: Dumping 0x12 bytes starting at 0x29DA40
00:00:09.18 kernel: DA40: 12 01 00 02 09 00 02 40-24 04 14 25 B3 0B 00 00
00:00:09.19 kernel: DA50: 00 01 25 B8 00 00 00 00-F0 09 08 00 00 00 00 00
```

```
rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0100, index=0, len=8
rpi_usb_init.md:[1]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0100, index=0, len=18
rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x0, req=5, value=0x0001, index=0, len=0
rpi_usb_init.md:[1]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0200, index=0, len=9
rpi_usb_init.md:[1]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x80, req=6, value=0x0200, index=0, len=41

rpi_usb_init.md:[2]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x0, req=9, value=0x0001, index=0, len=0
rpi_usb_init.md:[1]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x1, req=11, value=0x0001, index=0, len=0

rpi_usb_init.md:[1]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0xa0, req=6, value=0x2900, index=0, len=9

rpi_usb_init.md:[2]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0008, index=1, len=0
rpi_usb_init.md:[2]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0008, index=2, len=0
rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0008, index=3, len=0
rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0008, index=4, len=0

rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0xa3, req=0, value=0x0000, index=1, len=4
rpi_usb_init.md:[1]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0x23, req=3, value=0x0004, index=1, len=0
rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fece0, type=0xa3, req=0, value=0x0000, index=1, len=4
rpi_usb_init.md:[2]dw2_hc_control_message: ep=0xffff00003b3fe910, type=0x80, req=6, value=0x0100, index=0, len=8
rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fe910, type=0x80, req=6, value=0x0100, index=0, len=18
rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fe910, type=0x0, req=5, value=0x0002, index=0, len=0
rpi_usb_init.md:[3]dw2_hc_control_message: ep=0xffff00003b3fe910, type=0x80, req=6, value=0x0200, index=0, len=9
```

```
00:00:01.47 hc: ep=0x60b440, type=0x80, req=6, value=0x0100, index=0, len=8
00:00:01.53 hc: ep=0x60b440, type=0x80, req=6, value=0x0100, index=0, len=18
00:00:01.62 hc: ep=0x60b440, type=0x0, req=5, value=0x0001, index=0, len=0
00:00:01.71 hc: ep=0x60b440, type=0x80, req=6, value=0x0200, index=0, len=9
00:00:01.77 hc: ep=0x60b440, type=0x80, req=6, value=0x0200, index=0, len=41

00:00:01.90 hc: ep=0x60b440, type=0xa0, req=6, value=0x2900, index=0, len=9
00:00:01.96 hc: ep=0x60b440, type=0x0, req=9, value=0x0001, index=0, len=0
00:00:02.05 hc: ep=0x60b440, type=0x1, req=11, value=0x0001, index=0, len=0

00:00:02.08 hc: ep=0x60b440, type=0x23, req=3, value=0x0008, index=1, len=0
00:00:02.12 hc: ep=0x60b440, type=0x23, req=3, value=0x0008, index=2, len=0
00:00:02.16 hc: ep=0x60b440, type=0x23, req=3, value=0x0008, index=3, len=0
00:00:02.20 hc: ep=0x60b440, type=0x23, req=3, value=0x0008, index=4, len=0

00:00:02.75 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:02.81 hc: ep=0x60b440, type=0x23, req=3, value=0x0004, index=1, len=0
00:00:02.95 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:03.01 hc: ep=0x618900, type=0x80, req=6, value=0x0100, index=0, len=8
00:00:03.08 hc: ep=0x618900, type=0x80, req=6, value=0x0100, index=0, len=18
00:00:03.17 hc: ep=0x618900, type=0x0, req=5, value=0x0002, index=0, len=0
00:00:03.25 hc: ep=0x618900, type=0x80, req=6, value=0x0200, index=0, len=9

00:00:03.32 hc: ep=0x618900, type=0x80, req=6, value=0x0200, index=0, len=41
00:00:03.44 hc: ep=0x618900, type=0xa0, req=6, value=0x2900, index=0, len=9
00:00:03.51 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=2, len=4
00:00:03.57 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=3, len=4
00:00:03.64 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=4, len=4
00:00:03.70 hc: ep=0x618900, type=0x0, req=9, value=0x0001, index=0, len=0
00:00:03.79 hc: ep=0x618900, type=0x1, req=11, value=0x0001, index=0, len=0
00:00:03.83 hc: ep=0x618900, type=0x23, req=3, value=0x0008, index=1, len=0
00:00:03.87 hc: ep=0x618900, type=0x23, req=3, value=0x0008, index=2, len=0
00:00:03.90 hc: ep=0x618900, type=0x23, req=3, value=0x0008, index=3, len=0
00:00:04.45 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:04.51 hc: ep=0x618900, type=0x23, req=3, value=0x0004, index=1, len=0
00:00:04.65 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:04.72 hc: ep=0x619e40, type=0x80, req=6, value=0x0100, index=0, len=8
00:00:04.78 hc: ep=0x619e40, type=0x80, req=6, value=0x0100, index=0, len=18
00:00:04.87 hc: ep=0x619e40, type=0x0, req=5, value=0x0003, index=0, len=0
00:00:04.96 hc: ep=0x619e40, type=0x80, req=6, value=0x0200, index=0, len=9
00:00:05.02 hc: ep=0x619e40, type=0x80, req=6, value=0x0200, index=0, len=39
00:00:05.13 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=2, len=4
00:00:05.20 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=3, len=4
00:00:05.26 hc: ep=0x619e40, type=0x0, req=9, value=0x0001, index=0, len=0
00:00:05.35 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=0, len=4
00:00:05.42 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=16, len=4
00:00:05.48 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=16, len=4
00:00:05.55 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=16, len=4
00:00:05.61 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=284, len=4
00:00:05.68 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=280, len=4
00:00:05.74 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=1028, len=4
00:00:05.81 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=1024, len=4
00:00:05.88 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=144, len=4
00:00:05.94 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=148, len=4
00:00:06.01 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=16, len=4
00:00:06.07 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=16, len=4
00:00:06.14 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=128, len=4
00:00:06.20 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=128, len=4
00:00:06.27 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=200, len=4
00:00:06.33 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=204, len=4
00:00:06.40 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=152, len=4
00:00:06.47 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=12, len=4
00:00:06.53 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=268, len=4
00:00:06.60 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=208, len=4
00:00:06.66 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=176, len=4
00:00:06.73 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=176, len=4
00:00:06.79 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=20, len=4
00:00:06.86 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=20, len=4
00:00:06.92 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=20, len=4
00:00:06.99 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=20, len=4
00:00:07.05 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=256, len=4
00:00:07.12 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=256, len=4
00:00:07.18 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=264, len=4
00:00:07.25 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=264, len=4
00:00:07.31 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=196, len=4
00:00:07.38 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=196, len=4
00:00:07.44 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=260, len=4
00:00:07.51 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=260, len=4
00:00:07.57 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=192, len=4
00:00:07.64 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=192, len=4
00:00:07.71 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:07.77 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=292, len=4
00:00:07.84 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=288, len=4
00:00:07.90 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:07.97 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:08.03 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=288, len=4
00:00:08.10 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:08.16 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=292, len=4
00:00:08.23 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:08.29 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=292, len=4
00:00:08.36 hc: ep=0x619e40, type=0x40, req=160, value=0x0000, index=288, len=4
00:00:08.42 hc: ep=0x619e40, type=0xc0, req=161, value=0x0000, index=288, len=4
00:00:08.49 hc: ep=0x618900, type=0xa0, req=0, value=0x0000, index=0, len=4
00:00:08.56 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:08.62 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=2, len=4
00:00:08.69 hc: ep=0x618900, type=0xa3, req=0, value=0x0000, index=3, len=4
00:00:08.76 hc: ep=0x60b440, type=0xa0, req=0, value=0x0000, index=0, len=4
00:00:08.82 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=1, len=4
00:00:08.89 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=2, len=4
00:00:08.95 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=3, len=4
00:00:09.02 hc: ep=0x60b440, type=0xa3, req=0, value=0x0000, index=4, len=4
00:00:09.09 hc: ep=0x60b440, type=0x80, req=6, value=0x0100, index=0, len=18
```
