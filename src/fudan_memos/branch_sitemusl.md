# 2021.11.26 muslを/usr/local/musl（newmalloc)に変更して再コンパイル

## coreutils

```
$ make clean
$ ./configure --host=aarch64-linux-gnu CFLAGS="-specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096"
$ make
$ file src/ls
src/ls: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, not stripped
```

# dash

```
$ make clean
$ ./configure --host=aarch64-linux-gnu --enable-static CFLAGS="-specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096 -Isrc"
$ make
$ file src/dash
src/dash: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, not stripped
```

# xv6

```
$ vi users/Makefile
$ rm -rf obj
$ make
$ make qemu

Welcome to xv6 2021-09-15 (oldmalloc)  tty

 login: root
Password:
# ls
sys_execve: faled to fetch envs
-dash: 1: ls: Bad address

# ls /
sys_execve: path=/bin/ls, uargv=0x434448, uenvp=0x434460
fetchint: addr=434448, *addr=0x434410
- argv[0] = 'ls'
fetchint: addr=434450, *addr=0x434428
- argv[1] = '/'
fetchint: addr=434458, *addr=0x0
argv[2] = '(null)'
fetchint: addr=434460, *addr=0x60000240
fetchstr: addr=0x60000240, size=0x439000
sys_execve: failed to fetch envs
-dash: 1: ls: Bad address

(gdb) p *proc
  vmas = {{
      addr = 1610612736, (0x6000_0000)
--Type <RET> for more, q to quit, c to continue without paging--
      size = 4096,
      prot = 3,
      flags = 34,
      fd = 0,
      offset = 0,
      ref_count = 0,
      stored_size = 0,
      file = 0xffff00003b3dde00
    }, {
 ```

## envpはproc->vmapにマッピングされている

- fetchint(), fetchstr()でvmasを調べるよう変更

```
# ls /
sys_execve: path=/bin/ls, uargv=0x434448, uenvp=0x434460
- argv[0] = 'ls'
- argv[1] = '/'
argv[2] = '(null)'
- envp[0] = 'PWD=/.'                                            // envpは正常に読めてる
argv[1] = '/'
data abort: insruction 0x422364, fault addr 0xfffffffffffffffd, dfs=4
kern/console.c:284: kernel panic at cpu 0.

$ aarch64-linux-gnu-objdump -d coreutils-8.32/src/ls | grep 422364
<enframe>:
  422344:	54000100 	b.eq	422364 <enframe+0xf4>  // b.none
  422354:	5400008d 	b.le	422364 <enframe+0xf4>
  422364:	385fd001 	ldurb	w1, [x0, #-3]                     // <= x0 = 0

# cat /etc/passwd
sys_execve: path=/bin/cat, uargv=0x434458, uenvp=0x434470
- argv[0] = 'cat'
- argv[1] = '/etc/passwd'
argv[2] = '(null)'
- envp[0] = 'PWD=/.'
envp[1] = '(null)'
data abort: insruction 0x409e44, fault addr 0xfffffffffffffffd, dfs=4
kern/console.c:284: kernel panic at cpu 0.

$ aarch64-linux-gnu-objdump -d coreutils-8.32/src/cat | grep 409e44
  409e24:	54000100 	b.eq	409e44 <enframe+0xf4>  // b.none
  409e34:	5400008d 	b.le	409e44 <enframe+0xf4>
  409e44:	385fd001 	ldurb	w1, [x0, #-3]
```

## ls, catで同じエラー

- mallocに関係しているようだ
- muslをoldmmalocに戻す（muslはnewmallocでコンパイルしていた）

## muslを再コンパイル

```
$ make clean
$ CROSS_COMPILE=aarch64-linux-gnu- ./configure --target=aarch64 --with-malloc=oldmalloc
$ make
$ sudo make install
```

## coreutilsを再コンパイル

```
$ make clean
$ ./configure --host=aarch64-linux-gnu CFLAGS="-specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096"
$ make
```

## dashを再コンパイル

```
$ make clean
$ ./configure --host=aarch64-linux-gnu --enable-static CFLAGS="-specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096 -Isrc"
$ make
```

## xv6を再コンパイル

```
$ rm -rf obj
$ make
$ make qemu
```

## 実行

```
# ls /
sys_execve: path=/bin/ls, uargv=0x4336d0, uenvp=0x4336e8
fetchint: addr=4336d0, *addr=0x433698
- argv[0] = 'ls'
fetchint: addr=4336d8, *addr=0x4336b0
- argv[1] = '/'
fetchint: addr=4336e0, *addr=0x0
argv[2] = '(null)'
fetchint: addr=4336e8, *addr=0x436080       // oldmallocではenvpもvmapにマッピングされていない
- envp[0] = 'PWD=/.'
fetchint: addr=4336f0, *addr=0x0
envp[1] = '(null)'
bin  dev  etc  home                         // 正常
# ls -al
total 13
?????????? ? ?    ?       ?            ?            // 先頭行の修正を戻したがこれは直らなかった
drwxrwxr-x 1 root root 4096 Nov 26  2021 ..
drwxrwxr-x 1 root root 2080 Nov 26  2021 bin
drwxrwxr-x 1 root root   96 Nov 26  2021 dev
drwxrwxr-x 1 root root   64 Nov 26  2021 etc
drwxrwxr-x 1 root root   48 Nov 26  2021 home

# ls -al
total 13

drwxrwxr-x 1 root root 4096 Nov 26  2021 .          // 修正を戻した
drwxrwxr-x 1 root root 4096 Nov 26  2021 ..
drwxrwxr-x 1 root root 2080 Nov 26  2021 bin
drwxrwxr-x 1 root root   96 Nov 26  2021 dev
drwxrwxr-x 1 root root   64 Nov 26  2021 etc
drwxrwxr-x 1 root root   48 Nov 26  2021 home
# cat /etc/passwd
root:I8.2J./J//FF.:0:0:root:/:/bin/dash
vagrant:G8.E223I/.//.:1000:1000:,,,:/home/vagrant:/bin/dash
```
