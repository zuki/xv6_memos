# ブランチの説明

## new_mmap

- ユーザメモリをmmapで実装
- dashは内部コマンドを含め動く
- myls, mmaptestなどxv6由来のプログラムは動くがcoreutilsは動かない
- 2/18現在、sys_wait4()完了後に未定義割り込みが発生してdashに戻らず

## mmap_alloc

- new_mmapでkernelメモリからuserメモリへのコピーにcopyout()を使用。
- fault_addr=0x0でFLT_TRANSLATIONが発生してinitが立ち上がらず。
- 2/16断念

## dynamic

- mmapを使わないdyanamic link対応版
- dynamic linkしたhello worldが動く
- coreutilsをdynamic linkしたプログラムは動かない

## oldmalloc

- muslをmalloc/oldmallocでコンパイラしたstatic版
- 現在のところ、すべてのブランチでmalloc/oldmallocを使っている
