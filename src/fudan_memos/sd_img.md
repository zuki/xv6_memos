# sd.imgのファイル構成

## param変更後のアドレス

```
Device      Boot  Start     End Sectors  Size Id Type
obj/sd.img1        2048  133119  131072   64M  c W95 FAT32 (LBA)
obj/sd.img2      133120  952319  819200  400M 83 Linux
obj/sd.img3      952320 2097151 1144832  559M 83 Linux

sd.img2の先頭: 133120 * 512 = 68_157_440 = 0x04100000

nmeta 309 (boot, super, log blocks 270 inode blocks 33, bitmap blocks 4) blocks 99691 total 100000

v6の1 block = 4096 = 0x1000

boot:   0x04100000    1 block
04100000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04100010: 0000 0000 0000 0000 0000 0000 0000 0000  ................

super:  0x04101000    1 block
04101000: a086 0100 6b85 0100 0004 0000 0e01 0000  ....k...........
04101010: 0200 0000 1001 0000 3101 0000 0000 0000  ........1.......

log:    0x04102000    270 block = 0x10e000
04102000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04102010: 0000 0000 0000 0000 0000 0000 0000 0000  ................

inode:  0x04210000    33 block = 0x21000
04210000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04210010: 0000 0000 0000 0000 0000 0000 0000 0000  ................

bitmap: 0x04231000    4 block = 0x4000
04231000: ffff ffff ffff ffff ffff ffff ffff ffff  ................
04231010: ffff ffff ffff ffff ffff ffff ffff ffff  ................

data:   0x4235000
04235000: 0100 2e00 0000 0000 0000 0000 0000 0000  ................
04235010: 0100 2e2e 0000 0000 0000 0000 0000 0000  ................
04235020: 0200 6269 6e00 0000 0000 0000 0000 0000  ..bin...........
04235030: 0300 6465 7600 0000 0000 0000 0000 0000  ..dev...........
04235040: 0800 6574 6300 0000 0000 0000 0000 0000  ..etc...........
04235050: 0900 686f 6d65 0000 0000 0000 0000 0000  ..home..........
```

### inodeアドレスの計算

    INODE: inum * 0x80 (128 byte) + 0x04210000

```
inode[0]:
04210000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
04210010: 0000 0000 0000 0000 0000 0000 0000 0000  ................

inode[1]: 1 * 0x80 + 0x4210000 = 0x4210080
          type majo mino nlin size----- mode-----
          uid------ gid------ atime--------------
          ------------------- mtime--------------
          ------------------- ctime--------------
          ------------------- addres 0 ----------
          addrs 1 ----------- addres 2 ----------
04210080: 0100 0000 0000 0100 0010 0000 fd41 0000  .............A..
04210090: 0000 0000 0000 0000 75d9 9e61 0000 0000  ........u..a....
042100a0: 85ec 731e 0000 0000 75d9 9e61 0000 0000  ..s.....u..a....
042100b0: 85ec 731e 0000 0000 75d9 9e61 0000 0000  ..s.....u..a....
042100c0: 85ec 731e 0000 0000 3501 0000 0000 0000  ..s.....5.......

inode[2]: 2 * 0x80 + 0x4210000 = 0x4210100
04210100: 0100 0000 0000 0100 2008 0000 fd41 0000  ........ ....A..
04210110: 0000 0000 0000 0000 75d9 9e61 0000 0000  ........u..a....
04210120: 8351 741e 0000 0000 75d9 9e61 0000 0000  .Qt.....u..a....
04210130: 8351 741e 0000 0000 75d9 9e61 0000 0000  .Qt.....u..a....
04210140: 8351 741e 0000 0000 3601 0000 0000 0000  .Qt.....6.......
```

### dataアドレスの計算

    DATA: addrs[n] * 0x1000 (4096 byte) + 0x4100000

```
inode[1]のaddrs[0] = 0x0135: 0x135 * 0x1000 + 0x0410000 = 0x04235000

04235000: 0100 2e00 0000 0000 0000 0000 0000 0000  ................
04235010: 0100 2e2e 0000 0000 0000 0000 0000 0000  ................
04235020: 0200 6269 6e00 0000 0000 0000 0000 0000  ..bin...........
04235030: 0300 6465 7600 0000 0000 0000 0000 0000  ..dev...........
04235040: 0800 6574 6300 0000 0000 0000 0000 0000  ..etc...........
04235050: 0900 686f 6d65 0000 0000 0000 0000 0000  ..home..........
```

## 各ファイルのinode番号

```
# ls -ail /
total 13

1 drwxrwxr-x 1 root root 4096 Nov 25  2021 .
1 drwxrwxr-x 1 root root 4096 Nov 25  2021 ..
2 drwxrwxr-x 1 root root 2080 Nov 25  2021 bin
3 drwxrwxr-x 1 root root   96 Nov 25  2021 dev
8 drwxrwxr-x 1 root root   64 Nov 25  2021 etc
9 drwxrwxr-x 1 root root   48 Nov 25  2021 home

# ls -il /dev
total 2

4 brw-rw-rw- 1 root root 0, 0 Nov 25  2021 sdc1
5 brw-rw-rw- 1 root root 0, 1 Nov 25  2021 sdc2
6 brw-rw-rw- 1 root root 0, 2 Nov 25  2021 sdc3
7 crw-rw-rw- 1 root root 1, 0 Nov 25  2021 tty
# ls -il /etc
total 2

140 -rw-r--r-- 1 root root 1062 Nov 25  2021 group
139 -rw-r--r-- 1 root root  100 Nov 25  2021 passwd
# ls -il /home
total 1

10 drwxrwxr-x 1 vagrant vagrant 32 Nov 25  2021 vagrant
#

# ls -li /bin
total 18260

122 -rwxr-xr-x 1 root root 121424 Nov 25  2021 '['
 88 -rwxr-xr-x 1 root root 145928 Nov 25  2021  b2sum
 34 -rwxr-xr-x 1 root root 127008 Nov 25  2021  base32
 71 -rwxr-xr-x 1 root root 127000 Nov 25  2021  base64
 94 -rwxr-xr-x 1 root root 111712 Nov 25  2021  basename
 82 -rwxr-xr-x 1 root root 137440 Nov 25  2021  basenc
138 -rwxr-xr-x 1 root root 978336 Nov 25  2021  bash
 20 -rwxr-xr-x 1 root root  41856 Nov 25  2021  big
 75 -rwxr-xr-x 1 root root 117032 Nov 25  2021  cat
 86 -rwxr-xr-x 1 root root 156320 Nov 25  2021  chcon
133 -rwxr-xr-x 1 root root 177440 Nov 25  2021  chgrp
127 -rwxr-xr-x 1 root root 151000 Nov 25  2021  chmod
110 -rwxr-xr-x 1 root root 161568 Nov 25  2021  chroot
 85 -rwxr-xr-x 1 root root 117192 Nov 25  2021  cksum
 67 -rwxr-xr-x 1 root root 122896 Nov 25  2021  comm
105 -rwxr-xr-x 1 root root 227168 Nov 25  2021  cp
 52 -rwxr-xr-x 1 root root 239920 Nov 25  2021  csplit
 90 -rwxr-xr-x 1 root root 128400 Nov 25  2021  cut
137 -rwxr-xr-x 1 root root 253104 Nov 25  2021  dash
102 -rwxr-xr-x 1 root root 196376 Nov 25  2021  date
 60 -rwxr-xr-x 1 root root   1335 Nov 25  2021  dcgen
 31 -rwxr-xr-x 1 root root 167096 Nov 25  2021  dd
 89 -rwxr-xr-x 1 root root 210984 Nov 25  2021  df
 59 -rwxr-xr-x 1 root root 318088 Nov 25  2021  dir
 38 -rwxr-xr-x 1 root root 160480 Nov 25  2021  dircolors
 92 -rwxr-xr-x 1 root root 111304 Nov 25  2021  dirname
131 -rwxr-xr-x 1 root root 364424 Nov 25  2021  du
 99 -rwxr-xr-x 1 root root 101704 Nov 25  2021  echo
118 -rwxr-xr-x 1 root root 133216 Nov 25  2021  env
 27 -rwxr-xr-x 1 root root  39976 Nov 25  2021  envtest
107 -rwxr-xr-x 1 root root 118544 Nov 25  2021  expand
113 -rwxr-xr-x 1 root root 241168 Nov 25  2021  expr
 35 -rwxr-xr-x 1 root root 287576 Nov 25  2021  factor
 49 -rwxr-xr-x 1 root root  97016 Nov 25  2021  false
 64 -rwxr-xr-x 1 root root 128040 Nov 25  2021  fmt
 54 -rwxr-xr-x 1 root root 122864 Nov 25  2021  fold
 28 -rwxr-xr-x 1 root root  41096 Nov 25  2021  getdentstest
124 -rwxr-xr-x 1 root root 130824 Nov 25  2021  getlimits
 12 -rwxr-xr-x 1 root root  66144 Nov 25  2021  getty
125 -rwxr-xr-x 1 root root 252592 Nov 25  2021  ginstall
104 -rwxr-xr-x 1 root root 123512 Nov 25  2021  groups
 32 -rwxr-xr-x 1 root root 126512 Nov 25  2021  head
 29 -rwxr-xr-x 1 root root 111136 Nov 25  2021  hostid
 95 -rwxr-xr-x 1 root root 133784 Nov 25  2021  id
 16 -rwxr-xr-x 1 root root  25224 Nov 25  2021  init
 79 -rwxr-xr-x 1 root root 142496 Nov 25  2021  join
103 -rwxr-xr-x 1 root root 122304 Nov 25  2021  kill
134 -rwxr-xr-x 1 root root 111128 Nov 25  2021  link
 83 -rwxr-xr-x 1 root root 169488 Nov 25  2021  ln
 18 -rwxr-xr-x 1 root root  80032 Nov 25  2021  login
 41 -rwxr-xr-x 1 root root 111336 Nov 25  2021  logname
 77 -rwxr-xr-x 1 root root 318088 Nov 25  2021  ls
117 -rwxr-xr-x 1 root root 131384 Nov 25  2021  md5sum
 69 -rwxr-xr-x 1 root root 129984 Nov 25  2021  mkdir
 36 -rwxr-xr-x 1 root root 111856 Nov 25  2021  mkfifo
 14 -rwxr-xr-x 1 root root  52648 Nov 25  2021  mkfs
 30 -rwxr-xr-x 1 root root 121592 Nov 25  2021  mknod
 68 -rwxr-xr-x 1 root root 128520 Nov 25  2021  mktemp
 22 -rwxr-xr-x 1 root root  58120 Nov 25  2021  mmaptest
 13 -rwxr-xr-x 1 root root  84920 Nov 25  2021  mmaptest2
 21 -rwxr-xr-x 1 root root  16920 Nov 25  2021  mount
 84 -rwxr-xr-x 1 root root 237608 Nov 25  2021  mv
 15 -rwxr-xr-x 1 root root  40536 Nov 25  2021  mycat
 24 -rwxr-xr-x 1 root root  46504 Nov 25  2021  myls
 25 -rwxr-xr-x 1 root root  40856 Nov 25  2021  mywc
 40 -rwxr-xr-x 1 root root 116928 Nov 25  2021  nice
128 -rwxr-xr-x 1 root root 230368 Nov 25  2021  nl
 44 -rwxr-xr-x 1 root root 117352 Nov 25  2021  nohup
 45 -rwxr-xr-x 1 root root 117744 Nov 25  2021  nproc
136 -rwxr-xr-x 1 root root 163976 Nov 25  2021  numfmt
132 -rwxr-xr-x 1 root root 165904 Nov 25  2021  od
 17 -rwxr-xr-x 1 root root  70704 Nov 25  2021  passwd
 39 -rwxr-xr-x 1 root root 117184 Nov 25  2021  paste
 37 -rwxr-xr-x 1 root root 111704 Nov 25  2021  pathchk
 97 -rwxr-xr-x 1 root root 167440 Nov 25  2021  pinky
 93 -rwxr-xr-x 1 root root 191280 Nov 25  2021  pr
 43 -rwxr-xr-x 1 root root 110848 Nov 25  2021  printenv
126 -rwxr-xr-x 1 root root 276384 Nov 25  2021  printf
 42 -rwxr-xr-x 1 root root 394016 Nov 25  2021  ptx
 78 -rwxr-xr-x 1 root root 121896 Nov 25  2021  pwd
115 -rwxr-xr-x 1 root root 132088 Nov 25  2021  readlink
116 -rwxr-xr-x 1 root root 136520 Nov 25  2021  realpath
 91 -rwxr-xr-x 1 root root 162064 Nov 25  2021  rm
130 -rwxr-xr-x 1 root root 112576 Nov 25  2021  rmdir
 70 -rwxr-xr-x 1 root root 110856 Nov 25  2021  runcon
123 -rwxr-xr-x 1 root root 142152 Nov 25  2021  seq
 23 -rwxr-xr-x 1 root root  59264 Nov 25  2021  sh
 51 -rwxr-xr-x 1 root root 131392 Nov 25  2021  sha1sum
114 -rwxr-xr-x 1 root root 135744 Nov 25  2021  sha224sum
121 -rwxr-xr-x 1 root root 135744 Nov 25  2021  sha256sum
 50 -rwxr-xr-x 1 root root 139888 Nov 25  2021  sha384sum
 61 -rwxr-xr-x 1 root root 139888 Nov 25  2021  sha512sum
 96 -rwxr-xr-x 1 root root 157512 Nov 25  2021  shred
 87 -rwxr-xr-x 1 root root 148424 Nov 25  2021  shuf
 73 -rwxr-xr-x 1 root root 131976 Nov 25  2021  sleep
 81 -rwxr-xr-x 1 root root 260160 Nov 25  2021  sort
 48 -rwxr-xr-x 1 root root 156192 Nov 25  2021  split
 47 -rwxr-xr-x 1 root root 248032 Nov 25  2021  stat
 56 -rwxr-xr-x 1 root root 128136 Nov 25  2021  stdbuf
 66 -rwxr-xr-x 1 root root 147992 Nov 25  2021  stty
 11 -rwxr-xr-x 1 root root  72040 Nov 25  2021  su
 55 -rwxr-xr-x 1 root root 137016 Nov 25  2021  sum
 46 -rwxr-xr-x 1 root root 111592 Nov 25  2021  sync
100 -rwxr-xr-x 1 root root 220480 Nov 25  2021  tac
135 -rwxr-xr-x 1 root root 180216 Nov 25  2021  tail
111 -rwxr-xr-x 1 root root 118392 Nov 25  2021  tee
 57 -rwxr-xr-x 1 root root 116912 Nov 25  2021  test
 74 -rwxr-xr-x 1 root root 158800 Nov 25  2021  timeout
 72 -rwxr-xr-x 1 root root 188248 Nov 25  2021  touch
 98 -rwxr-xr-x 1 root root 136872 Nov 25  2021  tr
 58 -rwxr-xr-x 1 root root  97008 Nov 25  2021  true
 63 -rwxr-xr-x 1 root root 121688 Nov 25  2021  truncate
101 -rwxr-xr-x 1 root root 122744 Nov 25  2021  tsort
 26 -rwxr-xr-x 1 root root  65896 Nov 25  2021  ttbasic
129 -rwxr-xr-x 1 root root 111952 Nov 25  2021  tty
 19 -rwxr-xr-x 1 root root  16920 Nov 25  2021  umount
 62 -rwxr-xr-x 1 root root 111464 Nov 25  2021  uname
106 -rwxr-xr-x 1 root root 118488 Nov 25  2021  unexpand
120 -rwxr-xr-x 1 root root 129112 Nov 25  2021  uniq
 53 -rwxr-xr-x 1 root root 111128 Nov 25  2021  unlink
119 -rwxr-xr-x 1 root root 148624 Nov 25  2021  uptime
 33 -rwxr-xr-x 1 root root 117088 Nov 25  2021  users
108 -rwxr-xr-x 1 root root 318088 Nov 25  2021  vdir
 65 -rwxr-xr-x 1 root root 138240 Nov 25  2021  wc
 76 -rwxr-xr-x 1 root root 162736 Nov 25  2021  who
109 -rwxr-xr-x 1 root root 117936 Nov 25  2021  whoami
112 -rwxr-xr-x 1 root root 111360 Nov 25  2021  yes
```
