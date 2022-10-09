# 仮想メモリ

/*
 * A virtual address 'la' has a four-part structure as follows:
 * +-----9-----+-----9-----+-----9-----+-----9-----+---------12---------+
 * |  Level 0  |  Level 1  |  Level 2  |  Level 3  | Offset within Page |
 * |   Index   |   Index   |   Index   |   Index   |                    |
 * +-----------+-----------+-----------+-----------+--------------------+
 *  \PTX(0, va)/\PTX(1, va)/\PTX(2, va)/\PTX(3, va)/
 */

 # va=0x0xfffffffe0000

```
111111111 111111111 111111111 111100000 000000000000

 - レベル0: PTX(0, va) = 0x1ff
 - レベル1: PTX(1, va) = 0x1ff
 - レベル2: PTX(2, va) = 0x1ff
 - レベル3: PTX(3, va) = 0x1e0
 - オフセット: 0x000
