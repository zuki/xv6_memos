# メモ一覧

## 作業メモ

| ファイル名                                       | 説明                                                           | 更新更新日 |
| :----------------------------------------------- | :------------------------------------------------------------- | :--------- |
| [実機での実行方法](raspi.md)                     | 実機で実行するためのSDカードの処理                             |
| [change_branch.md](change_branch.md)             | dynamicブランチに切り替える場合の手順                          |
| [MEMO.md](MEMO.md)                               | 演習に沿って作業した際の作業メモ                               | 2021.08.22 |
| [memo2.md](memo2.md)                             | 実行速度を上げるための作業をした際の作業メモ                   | 2021.09.23 |
| [coreutils.md](coreutils.md)                     | CoreUtils導入時の作業メモ                                      | 2021.09.17 |
| [oldmalloc.md](oldmalloc.md)                     | Muslのmallocをoldmallocにした際の作業メモ                      | 2021.09.25 |
| [dash.md](dash.md)                               | Dash導入（未完）時の作業メモ                                   | 2021.09.30 |
| [mmaptest.md](mmaptest.md)                       | mmaptest2をAll OKにするまでの作業メモ                          | 2021.10.25 |
| [musl.md](musl.md)                               | muslを/usr/localに導入（libc.soも作成）。How to use muslの翻訳 | 2021.11.09 |
| [vfs.md](vfs.md)                                 | VFS導入時の作業メモ（vfs以外にも多くの機能を導入している）     | 2021.11.22 |
| [shadow.md](shadow.md)                           | shadow 4.9の導入を試みた際のメモ（失敗）                       | 2021.11.21 |
| [bash.md](bash.md)                               | bash 5.1.8の導入を試みた際のメモ（失敗）                       | 2021.11.25 |
| [sd_img.md](sd_img.md)                           | sd.img内のinode, data等のアドレス計算の方法                    | 2021.11.25 |
| [branch_sitemusl.md](branch_sitemusl.md)         | /usr/local/muslを使ってシステム再構築                          | 2021.11.26 |
| [binutils.md](binutils.md)                       | static linkしたbinutils 2.36をxv6に導入した際のメモ            | 2021.12.10 |
| [dynamic_link.md](dynamic_link.md)               | 動的リンクしたアプリケーションを動かすための改造メモ           | 2021.12.12 |
| [dash/readelf.md](dash/readelf.md)               | dashのデータ解析                                               |
| [dash/strace_dynamic.md](dash/strace_dynamic.md) | raspi linuxで実行したdash (dynamic link) のstraceログ          |
| [dash/strace_static.md](dash/strace_dynamic.md)  | raspi linuxで実行したdash (static link) のstraceログ           |
| [wait4_noreturn.md](wait4_noreturn.md)           | mmap版でwait4()のreturnでエラーが発生する件を調査したメモ      |
| [dash_vfork.md](dash_vfork.md)                   | mmap版でvforkでスタックが消える?問題を調査したメモ             |
| [mmap_dynamic.md](mmap_dynamic.md)               | mmap版でdynamic linkアプリを動かす                             | 2022.03.28 |

## 翻訳

| ファイル名                                                                 | 説明                                                                                                                                                        |
| :------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [arm/qa7.md](arm/qa7/qa7.md)                                               | ARM Quad A7 coreの翻訳                                                                                                                                      |
| [arm/bcm2835_peripheral/README.md](arm/bcm2837_arm_peripherals/index.html) | BCM2835 ARM Periferalsの翻訳                                                                                                                                |
| [arm/programmer_guid](arm/programmer_guide/book/index.html)                | Programmer’s Guide for ARMv8-Aの翻訳                                                                                                                        |
| [arm/raspi_firmware_wiki/README.md](arm/raspi_firmware_wiki/index.html) | Raspberry PiファームウェアWiki |
| [aarch64_kernel_pt.md](aarch64_kernel_pt.md)                               | [Wenbo Shen: AArch64 Kernel Page Tables](https://wenboshen.org/posts/2018-09-09-page-table.html) の翻訳                                                     |
| [mair_attr.md](mair_attr.md)                                               | AArch64の各種資料の翻訳                                                                                                                                     |
| [system_regs.md](arm/system_regs/book/index.html)                          | AArch64システムレジスタへのリンク集                                                                                                                         |
| [reg_about_exception.md](reg_about_exception.md)                           | AArch64例外関係のレジスタの説明                                                                                                                             |
| [sdcard_spec.md](sdcard_spec.md)                                           | SDCard規格の極一部翻訳                                                                                                                                      |
| [mit_lab_mmap.md](mit_lab_mmap.md)                                         | [MIT演習: MMAP](https://pdos.csail.mit.edu/6.828/2019/labs/mmap.html)の翻訳                                                                                 |
| [ext2_layout.md](ext2_layout.md)                                           | [The Second Extended File System / Dave Poirier](http://www.nongnu.org/ext2-doc/ext2.pdf) の翻訳                                                            |
| [vfs_xv6.md](vfs_xv6.md)                                                   | [A Virtual Filesystem Layer implementation in the XV6 Operating System / Caio Araújo Neponoceno de Lima](https://maups.github.io/papers/tcc_004.pdf) の翻訳 |
| [sd_functions.md](sd_functions.md)                                         | `kern/sd.c`の関数一覧                                                                                                                                       |
| [gcc-specs.md](gcc-specs.md)                                               | GCC specファイル文法の翻訳                                                                                                                                  |

# 講義資料の翻訳

- [ラボ0](docs_ja/lab0.md): 開発環境
- [ラボ1](docs_ja/lab1.md): ブート
- [ラボ2](docs_ja/lab2.md): メモリ管理
- [ラボ3](docs_ja/lab3.md): 割り込みと例外
- [ラボ4](docs_ja/lab4.md): マルチコアとロック
- [ラボ5](docs_ja/lab5.md): プロセス管理
- [ラボ6](docs_ja/lab6.md): ドライバとlibc
- [ラボ7](docs_ja/lab7.md): ファイルシステムとshell
