# ピン使用レファレンス

ARMとVideoCoreはピン空間を共有しているため、同じピンに同時にアクセス
することは避けなければなりません。[Firmware DT blob](https://github.com/raspberrypi/firmware/blob/master/extra/dt-blob.dts)に
GPUファームウェアで使用されるピンが記述されていますがあまり実用的では
ありません。そこで以下にレファレンス表を示します。

| ピン | RPi 3B+ |
|--:|:---|
| 40 | AUD |
| 41 | AUD |
| 44 | CAM |
| 45 | CAM |
| 46 | SMPS |
| 47 | SMPS |
