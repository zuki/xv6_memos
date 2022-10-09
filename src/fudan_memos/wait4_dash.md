# dashでエラーになる件

- ここでは、myls, ls, ls -l の3コマンドが実行できているが、1つも実行できずエラーになることもあり
- エラーは同じ。x29がスタックポインタになっている
- 一度エラーになると、rm objしないと同じエラーが続く

```
iinit: ok
init log[1]: ok
proc[1] > sys_execve
proc[1] > sys_gettid
proc[1] > sys_openat
proc[1] > sys_dup
proc[1] >> sys_dup
proc[1] > sys_ioctl
proc[1] > sys_writev
init: starting dash
proc[1] > sys_rt_sigprocmask
proc[1] >> sys_rt_sigprocmask
proc[1] > sys_clone
proc[6] > sys_gettid
proc[6] > sys_rt_sigprocmask
proc[1] >> sys_rt_sigprocmask
proc[1] >> sys_rt_sigprocmask
proc[1] > sys_wait4
proc[6] > sys_rt_sigprocmask
proc[6] > sys_execve
proc[6] > sys_gettid
proc[6] > sys_getpid
proc[6] > sys_rt_sigprocmask
proc[6] > sys_rt_sigaction
proc[6] > sys_geteuid
proc[6] > sys_getppid
proc[6] > sys_ioctl
proc[6] > sys_writev
__expand_head: *pn=96, n=4096
proc[6] > sys_brk
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
proc[6] > sys_writev
- brk1=0x436000
proc[6] > sys_brk
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
proc[6] > sys_writev
- brk2=0x437000, ret=0x436000
proc[6] > sys_getcwd
proc[6] > sys_ioctl
proc[6] >> sys_ioctl
proc[6] > sys_rt_sigaction
proc[6] >> sys_rt_sigaction
proc[6] >> sys_rt_sigaction
...
proc[6] > sys_openat
proc[6] > sys_fcntl
proc[6] > sys_close
proc[6] > sys_fcntl
proc[6] > sys_ioctl
proc[6] > sys_getpgid
proc[6] > sys_rt_sigaction
proc[6] >> sys_rt_sigaction
proc[6] >> sys_rt_sigaction
...
proc[6] > sys_setpgid
proc[6] > sys_ioctl
proc[6] > sys_write
# proc[6] > sys_read
/bin/myls
proc[6] > sys_rt_sigprocmask
proc[6] > sys_clone
proc[6] > sys_rt_sigprocmask
proc[6] > sys_setpgid
proc[7] > sys_getpid
proc[7] > sys_setpgid
proc[7] > sys_ioctl
proc[6] > sys_writev
musl call wait4: pid=-1, status=0xfffffffffb1c, dest=0
proc[7] > sys_rt_sigprocmask
proc[6] > sys_wait4
proc[7] > sys_execve
proc[7] > sys_gettid
proc[7] > sys_openat
proc[7] > sys_fstat
proc[7] > sys_read
proc[7] > sys_fstatat
proc[7] > sys_read
proc[7] >> sys_read
proc[7] >> sys_read
...
proc[7] > sys_fstatat
proc[7] > sys_read
proc[7] >> sys_read
proc[7] >> sys_read
...
proc[7] > sys_fstatat
proc[7] > sys_read
proc[7] >> sys_read
proc[7] >> sys_read
...
proc[7] > sys_fstatat
proc[7] > sys_read
proc[7] >> sys_read
proc[7] >> sys_read
...
proc[7] > sys_fstatat
proc[7] > sys_read
proc[7] >> sys_read
proc[7] >> sys_read
...
proc[7] > sys_fstatat
proc[7] > sys_read
proc[7] >> sys_read
proc[7] >> sys_read
...
proc[7] > sys_fstatat
proc[7] > sys_read
proc[7] >> sys_read
proc[7] >> sys_read
...
proc[7] > sys_fstatat
proc[7] > sys_read
proc[7] >> sys_read
proc[7] >> sys_read
...
proc[7] > sys_ioctl
proc[7] > sys_writev
drwxrwxr-x   1       4096 .             
drwxrwxr-x   1       4096 ..            
drwxrwxr-x   2       8448 bin           
drwxrwxr-x   3        384 dev           
drwxrwxr-x   8        320 etc           
drwxrwxrwx   9        192 lib           
drwxrwxr-x  10        192 home          
drwxrwxr-x  12        192 usr           
proc[7] > sys_close
proc[7] > sys_exit
proc[6] > sys_writev
- return=7												// これはexitした子プロセスのpid
proc[6] >> sys_writev
musl call wait4: pid=-1, status=0xfffffffffb1c, dest=0	// もう一度wait4をcallしているのは何故?
proc[6] > sys_wait4
proc[6] > sys_writev
- return=-10											// -ECHILD
proc[6] > sys_ioctl
proc[6] > sys_write
# proc[6] > sys_read
/bin/ls
proc[6] > sys_rt_sigprocmask
proc[6] > sys_clone
proc[6] > sys_rt_sigprocmask
proc[6] > sys_setpgid
proc[8] >> sys_setpgid
proc[8] > sys_ioctl
proc[6] > sys_writev
musl call wait4: pid=-1, status=0xfffffffffb1c, dest=0
proc[6] > sys_wait4
proc[8] > sys_rt_sigprocmask
proc[8] > sys_execve
proc[8] > sys_gettid
proc[8] > sys_ioctl
proc[8] >> sys_ioctl
proc[8] >> sys_ioctl
proc[8] > sys_writev
__expand_head: *pn=128, n=4096
proc[8] > sys_brk
sys_brk: n=0x0, sz=0x445000
- invalid n 0 return 0x445000
proc[8] > sys_writev
- brk1=0x445000
proc[8] > sys_brk
sys_brk: n=0x446000, sz=0x445000
- ok return 0x446000
proc[8] > sys_writev
- brk2=0x446000, ret=0x445000
proc[8] >> sys_writev
__expand_head: *pn=18464, n=20480
proc[8] > sys_brk
sys_brk: n=0x44b000, sz=0x446000
- ok return 0x44b000
proc[8] > sys_writev
- brk2=0x44b000, ret=0x446000
proc[8] > sys_openat
proc[8] > sys_fcntl
proc[8] > sys_getdents64
proc[8] >> sys_getdents64
proc[8] > sys_close
proc[8] > sys_writev
bin  dev  etc  home  lib  usr
proc[8] > sys_close
proc[8] >> sys_close
proc[8] > sys_exit
proc[6] > sys_writev
- return=8													// return child->pid
proc[6] >> sys_writev
musl call wait4: pid=-1, status=0xfffffffffb1c, dest=0
proc[6] > sys_wait4
proc[6] > sys_writev
- return=-10												// return -ECHLD
proc[6] > sys_ioctl
proc[6] > sys_write
# proc[6] > sys_read
/bin/ls -l
proc[6] > sys_rt_sigprocmask
proc[6] > sys_clone
proc[6] > sys_rt_sigprocmask
proc[6] > sys_setpgid
proc[6] > sys_writev
musl call wait4: pid=-1, status=0xfffffffffb1c, dest=0
proc[6] > sys_wait4
unexpected interrupt 0 at cpu 0

======= PROC[9]: /bin/dash TRAP FRAME DUMP =========
sp:    0x0000fffffffffb20  tpidr: 0x00000000004342f0
spsr:  0x0000000000000000  esr:   0x0000000002000000
elr:   0x0000fffffffffb20  x0:    0x0000000000000000
x1:    0x0000000000000000  x2:    0x0000000000000000
x3:    0x0000000000000008  x4:    0x0000000000435068
x5:    0x0000000000434bc8  x6:    0x2f2f2f2f2f2f2f2f
x7:    0x0000000000432000  x8:    0x00000000000000dc
x9:    0x0000000000000000  x10:   0x0000000000000000
x11:   0x0000000000000000  x12:   0x696177206c6c6163
x13:   0x3d646970203a3474  x14:   0x0000000000000001
x15:   0x00000000004295e0  x16:   0x000000000041eec0
x17:   0x0000000000000022  x18:   0x0000fffffffffa90
x19:   0x0000000000432000  x20:   0x0000000000434b98
x21:   0x0000000000000000  x22:   0x0000000000432000
x23:   0x0000000000000002  x24:   0x0000000000434b70
x25:   0x0000fffffffffd68  x26:   0x0000000000000000
x27:   0x0000000000000001  x28:   0x0000000000000000
x29:   0x0000fffffffffb20  x30:   0x0000fffffffffb20
===================== DUMP END =====================

==== stack proc[9] =========
fffffffffb20: 0000fffffffffc90 00000000004044d4 000000000042a095 0000000000434b98
fffffffffb40: 0000000000000000 0000000000405534 0000fffffffffc90 0000000000436100
fffffffffb60: 0000000000434b70 0000000000434bc8 000000000042a095 ffffffff00432000
fffffffffb80: 0000fffffffffd68 0000000000000000 0000fffffffffd68 0000000000000000
fffffffffba0: 0000fffffffffb10 0000000000413f08 0000000000432000 0000fffffffffb10
fffffffffbc0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffbe0: 0000000000000000 0000000000000000 0000000000000000 000000000042a095
fffffffffc00: 000000000040d514 0000000000000000 0000fffffffffc30 000000000040973c
fffffffffc20: 0000000000433000 000000000040976c 0000fffffffffc90 00000000004043a8
fffffffffc40: 0000000000000000 0000000000434b98 0000000000000020 00114b8f3798ec48
fffffffffc60: 0000fffffffffc90 0000000000404660 000000000042a095 0000000000434b98
fffffffffc80: 0000000000000000 00114b8f3798ec48 0000fffffffffd90 0000000000402f64
fffffffffca0: 0000000000414758 0000000000432000 0000000000000002 0000000000000000
fffffffffcc0: 0000fffffffffdd0 0000000000432000 0000000000000000 0000000000000000
fffffffffce0: 0000000000000000 0000000000000000 0000000000432000 0000000000433000
fffffffffd00: 0000fffffffffd78 ffffffff00000000 0000000000000000 0000000000434ba8
fffffffffd20: 0000000000434950 0000000000000000 0000000000000000 0000000000434bc8
fffffffffd40: 0000000000434bc0 0000000000000000 0000000000434b70 0000000000434b98
fffffffffd60: 0000000000434bb0 0000000000000000 0000fffffffffd68 0000000000000000
fffffffffd80: 00000000ffffffff 00114b8f3798ec48 0000fffffffffdf0 0000000000400454
fffffffffda0: 0000000000432000 0000fffffffffe60 0000000000000000 000000000040d7f0
fffffffffdc0: 0000000000429658 00000000004003bc 0000000000434b18 0000000000434b90
fffffffffde0: 0000000000000188 00114b8f3798ec48 0000000000000000 0000000000419b58
fffffffffe00: 0000000000000001 0000fffffffffeb8 0000000000400150 0000fffffffffec8
fffffffffe20: 0000000000402000 0000000000000000 0000000100000000 0000fffffffffeb8
fffffffffe40: 0000000400000000 0000000000434b18 0000000000434b20 00000000000001f8
fffffffffe60: 0000000000434b18 0000000000434b20 00000000000001f8 00114b8f3798ec48
fffffffffe80: 0000000000000000 0000000000400988 0000000000402178 0000000000403000
fffffffffea0: 0000000000000000 0000000000000000 0000000000000001 0000ffffffffffe9
fffffffffec0: 0000000000000000 0000000000000000 0000000000000010 0000000000000887
fffffffffee0: 0000000000000006 0000000000001000 0000000000000011 0000000000000064
ffffffffff00: 0000000000000003 0000000000400040 0000000000000004 0000000000000038
ffffffffff20: 0000000000000005 0000000000000004 0000000000000007 0000000000000000
ffffffffff40: 0000000000000008 0000000000000000 0000000000000009 00000000004004a0
ffffffffff60: 000000000000000b 0000000000000000 000000000000000c 0000000000000000
ffffffffff80: 000000000000000d 0000000000000000 000000000000000e 0000000000000000
ffffffffffa0: 0000000000000017 0000000000000000 0000000000000033 0000000000001207
ffffffffffc0: 000000000000000f 0000ffffffffffe1 0000000000000000 0000000000000000
ffffffffffe0: 3436686372616100 622f006873616400 00687361642f6e69 0000000000000000

==== stack proc[9] =========

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0xfffffffffb20
 SPSR_EL1 0x0 FAR_EL1 0xfffffffffb00
	irq_error: irq of type 8 unimplemented.
```

## region_mapでdebug出力をすると動く

- cprintf("map_region[%d]: pa=0x%llx\n", thisproc()->pid, pa);
- このdebug出力を外すとエラー
- paの書き込みのタイミングか？
- region_mapを呼び出す前にpaにアクセスしてもエラーにならない(map_anon_page())

## 解決?

- map_anon_page()を以下のように変更したらエラーが出なくなった
- タイミングでエラーになることもあるので解決ではない模様
- `break swtch if $x30==0x0000fffffffffb20`で監視するとエラーにならない（その後、エラー発生）

### 変更前

```
if (map_region(p->pgdir, addr, PGSIZE, V2P(page), perm) < 0) {
```

### 変更後

```
uint64_t pa = V2P(page);
if (map_region(p->pgdir, addr, PGSIZE, pa, perm) < 0) {
```

```
(gdb) si
0x000000000041bd94 in ?? ()			// <vfork>のsys_clone()呼び出し後の<__syscall_ret>のcall
(gdb) 
0x0000000000419f18 in ?? ()			// <__syscall_ret>の先頭
(gdb) 
alltraps () at kern/trapasm.S:14
14	    stp     x29, x30, [sp, #-16]!
(gdb) c
```

## dash.asm

```
000000000041bd80 <vfork>:
  41bd80:       d2801b88        mov     x8, #0xdc                       // #220
  41bd84:       d2800220        mov     x0, #0x11                       // #17
  41bd88:       d2800001        mov     x1, #0x0                        // #0
  41bd8c:       f81f0ffe        str     x30, [sp, #-16]!
  41bd90:       d4000001        svc     #0x0
  41bd94:       97fff861        bl      419f18 <__syscall_ret>
  41bd98:       f84107fe        ldr     x30, [sp], #16
  41bd9c:       d65f03c0        ret
```

## dashにデバッグprint

```
# ls
vforkexec start
makejob(0x434b48, 1) returns %1
call vfork
forkparent: In parent shell:  child = 7
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 7
tryexec2: /bin/ls
bin  dev  etc  home  lib  usr
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# ls -l
vforkexec start
makejob(0x434b70, 1) returns %1
call vfork
forkparent: In parent shell:  child = 8
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 8
tryexec2: /bin/ls
total 11
drwxrwxr-x 1 root root 8448 Feb 25  2022 bin
...
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# myls
vforkexec start
makejob(0x434b48, 1) returns %1
call vfork
forkparent: In parent shell:  child = 9
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 9
tryexec2: /bin/myls
??????drwxrwxr-x   1       4096 .  
...           
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# readelf -h /bin/cat
vforkexec start
makejob(0x434ba0, 1) returns %1
call vfork
forkparent: In parent shell:  child = 10
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 10
tryexec2: /usr/local/bin/readelf
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  ...
  Section header string table index: 16
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# hellos
vforkexec start
makejob(0x434b48, 1) returns %1
call vfork
forkparent: In parent shell:  child = 11
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
unexpected interrupt 0 at cpu 0

======= PROC[11]: /bin/dash TRAP FRAME DUMP =========
sp:    0x0000fffffffffb20  tpidr: 0x00000000004342f0
spsr:  0x0000000000000000  esr:   0x0000000002000000
elr:   0x0000000000000001  x0:    0x0000000000000000
x1:    0x0000000000000000  x2:    0x0000000000000002
x3:    0x0000000000423f5c  x4:    0x0000000000429872
x5:    0x0000000000433d96  x6:    0x6f6676206c6c6163
x7:    0x6b726f6676206c6c  x8:    0x00000000000000dc
x9:    0x0000000000000001  x10:   0x0000000000000031
x11:   0x0000000000000000  x12:   0x2f20682d20666c65
x13:   0x007461632f6e6962  x14:   0x0000000000000000
x15:   0x00000000004298e0  x16:   0x000000000041f2b8
x17:   0x0000000000000022  x18:   0x0000fffffffffa50
x19:   0x0000000000432000  x20:   0x000000000041c8a0
x21:   0x0000000000000000  x22:   0x0000000000432000
x23:   0x0000000000000001  x24:   0x0000000000434b48
x25:   0x0000fffffffffd68  x26:   0x0000000000000000
x27:   0x0000000000000001  x28:   0x0000000000000000
x29:   0x0000fffffffffb20  x30:   0x0000000000000001
===================== DUMP END =====================

==== stack proc[11] =========
fffffffffb00: 0000000000432000 000000000041be98 0000000000000001 000000000040c90c
fffffffffb20: 0000fffffffffc90 00000000004044d4 000000000042a395 0000000000434b70
fffffffffb40: 0000000000000000 0000fffffffffbf8 0000fffffffffc90 0000000000436140
fffffffffb60: 0000000000434b48 0000000000434b88 000000000042a395 0000000500432000
fffffffffb80: 0000000000000001 0000000000434b48 0000fffffffffd68 0000000000000000
fffffffffba0: 0000000000000001 0000000000000000 0000000000432000 0000000000434b98
fffffffffbc0: 0000000000433040 0000000000000000 000000000000000c 0000fffffffffc08
fffffffffbe0: 000000000041c4b0 0000000000434b98 0000000000000002 0000000000000000
fffffffffc00: 0000000000434b68 0000000000000000 0000000000000011 00000001000081ed
fffffffffc20: 0000000000000000 0000fffffffffb50 0000000000000000 0000000000004168
fffffffffc40: 0000000000001000 0000000000000021 00000000621827e8 00114b8f3798ec48
fffffffffc60: 0000fffffffffc90 0000000000404660 000000000042a395 0000000000434b70
fffffffffc80: 0000000000000000 00114b8f3798ec48 0000fffffffffd90 0000000000402f64
fffffffffca0: 0000000000414858 0000000000432000 0000000000000002 0000000000000000
fffffffffcc0: 0000fffffffffdd0 0000000000432000 0000000000000000 0000000000000000
fffffffffce0: 0000000000000000 0000000000000000 0000000000432000 0000000000433000
fffffffffd00: 0000fffffffffd78 ffffffff00000000 0000000000000000 0000000000434b68
fffffffffd20: 0000000000434950 0000000000000000 0000000000000000 0000000000434b88
fffffffffd40: 0000000000434b80 0000000000000000 0000000000434b48 0000000000434b70
fffffffffd60: 0000000000434b70 0000000000000000 0000fffffffffd68 0000000000000000
fffffffffd80: 0000000000000005 00114b8f3798ec48 0000fffffffffdf0 0000000000400454
fffffffffda0: 0000000000432000 0000fffffffffe60 0000000000000000 000000000040d8f0
fffffffffdc0: 0000000000429958 00000000004003bc 0000000000434b18 0000000000434b68
fffffffffde0: 00000000000001b0 00114b8f3798ec48 0000000000000000 0000000000419c58
fffffffffe00: 0000000000000001 0000fffffffffeb8 0000000000400150 0000fffffffffec8
fffffffffe20: 0000000000402000 0000000000000000 0000000100000000 0000fffffffffeb8
fffffffffe40: 0000000400000000 0000000000434b18 0000000000434b20 00000000000001f8
fffffffffe60: 0000000000434b18 0000000000434b20 00000000000001f8 00114b8f3798ec48
fffffffffe80: 0000000000000000 0000000000400988 0000000000402178 0000000000403000
fffffffffea0: 0000000000000000 0000000000000000 0000000000000001 0000ffffffffffe9
fffffffffec0: 0000000000000000 0000000000000000 0000000000000010 0000000000000887
fffffffffee0: 0000000000000006 0000000000001000 0000000000000011 0000000000000064
ffffffffff00: 0000000000000003 0000000000400040 0000000000000004 0000000000000038
ffffffffff20: 0000000000000005 0000000000000004 0000000000000007 0000000000000000
ffffffffff40: 0000000000000008 0000000000000000 0000000000000009 00000000004004a0
ffffffffff60: 000000000000000b 0000000000000000 000000000000000c 0000000000000000
ffffffffff80: 000000000000000d 0000000000000000 000000000000000e 0000000000000000
ffffffffffa0: 0000000000000017 0000000000000000 0000000000000033 0000000000001207
ffffffffffc0: 000000000000000f 0000ffffffffffe1 0000000000000000 0000000000000000
ffffffffffe0: 3436686372616100 622f006873616400 00687361642f6e69 0000000000000000

==== stack proc[11] =========

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x1
 SPSR_EL1 0x0 FAR_EL1 0x1
	irq_error: irq of type 8 unimplemented.
```

## `break swtch if (uint64_t)$x30 < 0x5`でgdb実行するとエラーは発生しない

- x30が小さな値（実行できたコマンド数か？）に設定されてエラーとなる
- 表記のbreakポイントを設定して、gdb配下で実行するとエラーは発生しない
- エラー時のトラップフレームはsys_clone呼び出し後の状態を示しているようだ
- エラーはdash:vforkexec()のvfork()実行後に子プロセスの実行で発生している模様
- fork()とvfork()で処理を分ける必要があるのか。
	- vfrok()はSYS_cloneを呼び出しているだけ
	- fork()はSYS_cloneの呼び出し前後で多くの処理をしている

### vfork()

```c
# musl/src/process/vfork.c

pid_t vfork(void)
{
        return syscall(SYS_clone, SIGCHLD, 0);
}

000000000041bf18 <vfork>:
  41bf18:       d2801b88        mov     x8, #0xdc                       // #220
  41bf1c:       d2800220        mov     x0, #0x11                       // #17
  41bf20:       d2800001        mov     x1, #0x0                        // #0
  41bf24:       f81f0ffe        str     x30, [sp, #-16]!						// このspがおかしくなるとx30の値もおかしくなる
  41bf28:       d4000001        svc     #0x0
  41bf2c:       97fff861        bl      41a0b0 <__syscall_ret>
  41bf30:       f84107fe        ldr     x30, [sp], #16
  41bf34:       d65f03c0        ret
```

### sys_clone()にbreakを設定して、実行した際の、呼び出し時のp->tf

- エラーが発生せず

```
(gdb) p/x *p->tf
$12 = {
  q0 = 0x0,
  tpidr_el0 = 0x4342f0,
  esr_el1 = 0x56000000,
  sp_el0 = 0xfffffffffb10,
  spsr_el1 = 0x60000000,
  elr_el1 = 0x41bf2c,
  x0 = 0x11,
  x1 = 0x0,
  x2 = 0x2,
  x3 = 0x423ffc,
  x4 = 0x4299a2,
  x5 = 0x433d96,
  x6 = 0x6f6676206c6c6163,
  x7 = 0x6b726f6676206c6c,
  x8 = 0xdc,
  x9 = 0x1,
  x10 = 0x31,
  x11 = 0x0,
  x12 = 0x429598,
  x13 = 0x429578,
  x14 = 0x0,
  x15 = 0x429a20,
  x16 = 0x41f358,
  x17 = 0x22,
  x18 = 0xfffffffffa50,
  x19 = 0x432000,
  x20 = 0x41c938,
  x21 = 0x0,
  x22 = 0x432000,
  x23 = 0x3,
  x24 = 0x434ba0,
  x25 = 0xfffffffffd68,
  x26 = 0x0,
  x27 = 0x1,
  x28 = 0x0,
  x29 = 0xfffffffffb20,
  x30 = 0x40c954
}
(gdb) x/-8gx p->tf->sp_el0
0xfffffffffad0:	0x0000000000433220	0x000000000041c9d8
0xfffffffffae0:	0x0000000000000000	0x0a0000000041c970
0xfffffffffaf0:	0x0000000000432000	0x000000000041c938
0xfffffffffb00:	0x0000000000000000	0x0000000000432000
(gdb) x/8gx p->tf->sp_el0
0xfffffffffb10:	0x000000000040c954	0x00114b8f3798ec48
0xfffffffffb20:	0x0000fffffffffc90	0x00000000004044d4
0xfffffffffb30:	0x000000000042a4d5	0x0000000000434bc8
0xfffffffffb40:	0x0000000000000000	0x0000fffffffffbf8
```

### 続けてコマンドを実行

```
(gdb) p/x *p->tf
$13 = {
  q0 = 0x0,
  tpidr_el0 = 0x4342f0,
  esr_el1 = 0x56000000,
  sp_el0 = 0xfffffffffb10,
  spsr_el1 = 0x60000000,
  elr_el1 = 0x41bf2c,
  x0 = 0x11,
  x1 = 0x0,
  x2 = 0x2,
  x3 = 0x423ffc,
  x4 = 0x4299a2,
  x5 = 0x433d96,
  x6 = 0x6f6676206c6c6163,
  x7 = 0x6b726f6676206c6c,
  x8 = 0xdc,
  x9 = 0x1,
  x10 = 0x31,
  x11 = 0xfffffffff9f0,
  x12 = 0x64203a676e696e72,
  x13 = 0x203a34203a687361,
  x14 = 0x0,
  x15 = 0x429270,
  x16 = 0x41f358,
  x17 = 0x434528,
  x18 = 0x4330a8,
  x19 = 0x432000,
  x20 = 0x41c938,
  x21 = 0x0,
  x22 = 0x432000,
  x23 = 0x1,
  x24 = 0x434b48,
  x25 = 0xfffffffffd68,
  x26 = 0x0,
  x27 = 0x1,
  x28 = 0x0,
  x29 = 0xfffffffffb20,
  x30 = 0x40c954
}
(gdb) x/-8gx p->tf->sp_el0
0xfffffffffad0:	0x0000000000433220	0x000000000041c9d8
0xfffffffffae0:	0x0000000000000000	0x0a0000000041c970
0xfffffffffaf0:	0x0000000000432000	0x000000000041c938
0xfffffffffb00:	0x0000000000000000	0x0000000000432000
(gdb) x/8gx p->tf->sp_el0
0xfffffffffb10:	0x000000000040c954	0x00114b8f3798ec48
0xfffffffffb20:	0x0000fffffffffc90	0x00000000004044d4
0xfffffffffb30:	0x000000000042a4d5	0x0000000000434b70
0xfffffffffb40:	0x0000000000000000	0x0000fffffffffbf8
(gdb) c
Continuing.

Thread 1 hit Breakpoint 1, sys_clone () at kern/sysproc.c:85
85	    struct proc *p = thisproc();
(gdb) n
87	    int flags = (int)p->tf->x0;
(gdb) p/x *p->tf
$14 = {
  q0 = 0x0,
  tpidr_el0 = 0x4342f0,
  esr_el1 = 0x56000000,
  sp_el0 = 0xfffffffffb10,
  spsr_el1 = 0x60000000,
  elr_el1 = 0x41bf2c,
  x0 = 0x11,
  x1 = 0x0,
  x2 = 0x2,
  x3 = 0x423ffc,
  x4 = 0x4299a2,
  x5 = 0x433d96,
  x6 = 0x6f6676206c6c6163,
  x7 = 0x6b726f6676206c6c,
  x8 = 0xdc,
  x9 = 0x1,
  x10 = 0x31,
  x11 = 0x0,
  x12 = 0x429598,
  x13 = 0x429578,
  x14 = 0x0,
  x15 = 0x429a20,
  x16 = 0x41f358,
  x17 = 0x22,
  x18 = 0xfffffffffa50,
  x19 = 0x432000,
  x20 = 0x41c938,
  x21 = 0x0,
  x22 = 0x432000,
  x23 = 0x1,
  x24 = 0x434b48,
  x25 = 0xfffffffffd68,
  x26 = 0x0,
  x27 = 0x1,
  x28 = 0x0,
  x29 = 0xfffffffffb20,
  x30 = 0x40c954
}
(gdb) x/-8gx p->tf->sp_el0
0xfffffffffad0:	0x0000000000433220	0x000000000041c9d8
0xfffffffffae0:	0x0000000000000000	0x0a0000000041c970
0xfffffffffaf0:	0x0000000000432000	0x000000000041c938
0xfffffffffb00:	0x0000000000000000	0x0000000000432000
(gdb) x/8gx p->tf->sp_el0
0xfffffffffb10:	0x000000000040c954	0x00114b8f3798ec48
0xfffffffffb20:	0x0000fffffffffc90	0x00000000004044d4
0xfffffffffb30:	0x000000000042a4d5	0x0000000000434b70
0xfffffffffb40:	0x0000000000000000	0x0000fffffffffbf8
```

### fork()

```c
# musl/src/process/fork.c

pid_t fork(void)
{
	sigset_t set;
	__fork_handler(-1);
	__block_app_sigs(&set);
	int need_locks = libc.need_locks > 0;
	if (need_locks) {
		__ldso_atfork(-1);
		__inhibit_ptc();
		for (int i=0; i<sizeof atfork_locks/sizeof *atfork_locks; i++)
			if (*atfork_locks[i]) LOCK(*atfork_locks[i]);
		__malloc_atfork(-1);
		__tl_lock();
	}
	pthread_t self=__pthread_self(), next=self->next;
	pid_t ret = _Fork();
	int errno_save = errno;
	if (need_locks) {
		if (!ret) {
			for (pthread_t td=next; td!=self; td=td->next)
				td->tid = -1;
			if (__vmlock_lockptr) {
				__vmlock_lockptr[0] = 0;
				__vmlock_lockptr[1] = 0;
			}
		}
		__tl_unlock();
		__malloc_atfork(!ret);
		for (int i=0; i<sizeof atfork_locks/sizeof *atfork_locks; i++)
			if (*atfork_locks[i])
				if (ret) UNLOCK(*atfork_locks[i]);
				else **atfork_locks[i] = 0;
		__release_ptc();
		__ldso_atfork(!ret);
	}
	__restore_sigs(&set);
	__fork_handler(!ret);
	if (ret<0) errno = errno_save;
	return ret;
}

# mu

pid_t _Fork(void)
{
	pid_t ret;
	sigset_t set;
	__block_all_sigs(&set);
	__aio_atfork(-1);
	LOCK(__abort_lock);
	ret = __syscall(SYS_clone, SIGCHLD, 0);
	if (!ret) {
		pthread_t self = __pthread_self();
		self->tid = __syscall(SYS_gettid);
		self->robust_list.off = 0;
		self->robust_list.pending = 0;
		self->next = self->prev = self;
		__thread_list_lock = 0;
		libc.threads_minus_1 = 0;
		if (libc.need_locks) libc.need_locks = -1;
	}
	UNLOCK(__abort_lock);
	__aio_atfork(!ret);
	__restore_sigs(&set);
	return __syscall_ret(ret);
}
```

### 実行時ログ

```
main: [CPU0] Init success.
iinit: ok
init log[1]: ok
init: starting dash
sys_clone: flags=0x11, stacksiz=0x0, ptid=0, tls=0x8, ctid=-545, x5=0x40478c, x6=0x7473203a74696e69
sys_wait4[1]: pid: -1, wstatus: 0x0, options: 0x0, rusage: 0x0
# ls
vforkexec start
makejob(0x434b48, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=7
vfork: pid=vfork: pid=0
forkparent: In parent shell:  child = 7
forkchild: shell 7, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/ls
bin  dev  etc  home  lib  usr
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# ls -l
vforkexec start
makejob(0x434b70, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=8
forkparent: In parent shell:  child = 8
vfork: pid=8vfork: pid=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 8, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/ls
total 11
drwxrwxr-x 1 root root 8448 Feb 25  2022 bin
drwxrwxr-x 1 root root  384 Feb 25  2022 dev
drwxrwxr-x 1 root root  320 Feb 25  2022 etc
drwxrwxr-x 1 root root  192 Feb 25  2022 home
drwxrwxrwx 1 root root  192 Feb 25  2022 lib
drwxrwxr-x 1 root root  192 Feb 25  2022 usr
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# readelf -l /bin/cat
vforkexec start
makejob(0x434ba0, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=9
vfork: pid=9vfork: pid=0
forkparent: In parent shell:  child = 9
forkchild: shell 9, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /usr/local/bin/readelf

Elf file type is EXEC (Executable file)
Entry point 0x400eac
There are 4 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x0000000000018584 0x0000000000018584  R E    0x1000
  LOAD           0x00000000000189b0 0x00000000004199b0 0x00000000004199b0
                 0x0000000000000958 0x0000000000001a88  RW     0x1000
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x00000000000189b0 0x00000000004199b0 0x00000000004199b0
                 0x0000000000000650 0x0000000000000650  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00     .init .text .fini .rodata .eh_frame 
   01     .init_array .fini_array .data.rel.ro .got .got.plt .data .bss 
   02     
   03     .init_array .fini_array .data.rel.ro .got .got.plt 
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# echo abc > test
# cat test
vforkexec start
makejob(0x434b70, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=10
forkparent: In parent shell:  child = 10
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
vfork: pid=0
forkchild: shell 10, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/cat
abc
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# whoami
vforkexec start
makejob(0x434b48, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=11
vfork: pid=11vfork: pid=0
forkparent: In parent shell:  child = 11
forkchild: shell 11, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/whoami
root
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# myls
vforkexec start
makejob(0x434b48, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=12
vfork: pid=12vfork: pid=0
forkparent: In parent shell:  child = 12
forkchild: shell 12, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/myls
??????drwxrwxr-x   1       4096 .             
drwxrwxr-x   1       4096 ..            
drwxrwxr-x   2       8448 bin           
drwxrwxr-x   3        384 dev           
drwxrwxr-x   8        320 etc           
drwxrwxrwx   9        192 lib           
drwxrwxr-x  10        192 home          
drwxrwxr-x  12        192 usr           
-rw-rw-r-- 159          4 test          
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# hellos
vforkexec start
makejob(0x434b48, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=13
vfork: pid=13vfork: pid=0
forkparent: In parent shell:  child = 13
forkchild: shell 13, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/hellos
hello static world!
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# ls bin
vforkexec start
makejob(0x434b70, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=14
forkparent: In parent shell:  child = 14
vfork: pid=14vfork: pid=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 14, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/ls
'['	    df		   hostid      mywc	  seq	      timeout
 b2sum	    dir		   id	       newinit	  sh	      touch
 base32     dircolors	   init        nice	  sha1sum     tr
 base64     dirname	   join        nl	  sha224sum   true
 basename   du		   kill        nohup	  sha256sum   truncate
 basenc     echo	   link        nproc	  sha384sum   tsort
 big	    env		   ln	       numfmt	  sha512sum   ttbasic
 cat	    envtest	   login       od	  shred       tty
 chcon	    expand	   logname     passwd	  shuf	      umount
 chgrp	    expr	   ls	       paste	  sleep       uname
 chmod	    factor	   md5sum      pathchk	  sort	      unexpand
 chown	    false	   mkdir       pinky	  split       uniq
 chroot     fmt		   mkfifo      pr	  stat	      unlink
 cksum	    fold	   mkfs        printenv   stdbuf      uptime
 comm	    getdentstest   mknod       printf	  stty	      users
 cp	    getlimits	   mktemp      ptx	  su	      vdir
 csplit     getty	   mmaptest    pwd	  sum	      wc
 cut	    ginstall	   mmaptest2   readlink   sync	      whoami
 dash	    groups	   mount       realpath   tac	      yes
 date	    head	   mv	       rm	  tail
 dcgen	    hello	   mycat       rmdir	  tee
 dd	    hellos	   myls        runcon	  test
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# cp test test2
vforkexec start
makejob(0x434b98, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=15
vfork: pid=15vfork: pid=0
forkparent: In parent shell:  child = 15
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 15, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/cp
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# diff test test2
warning: dash: 11: diff: not found
# cat test test2 > test3
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# cat test3
vforkexec start
makejob(0x434b70, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=17
forkparent: In parent shell:  child = 17
vfork: pid=17vfork: pid=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 17, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/cat
vforkexec start
makejob(0x434be8, 1) returns %1
call vfork
vfork: pid=16
vfork: pid=vfork: pid=0
forkparent: In parent shell:  child = 16
forkchild: shell 16, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/cat
abc
abc
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# wc -l test2
vforkexec start
makejob(0x434b98, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=18
vfork: pid=18vfork: pid=0
forkparent: In parent shell:  child = 18
forkchild: shell 18, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/wc
1 test2
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# wc test3
vforkexec start
makejob(0x434b70, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=19
vfork: pid=0
forkparent: In parent shell:  child = 19
forkchild: shell 19, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/wc
 16  42 298 test3
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# ls /bin
vforkexec start
makejob(0x434b70, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=20
vfork: pid=20vfork: pid=0
forkparent: In parent shell:  child = 20
forkchild: shell 20, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/ls
'['	    df		   hostid      mywc	  seq	      timeout
 b2sum	    dir		   id	       newinit	  sh	      touch
 base32     dircolors	   init        nice	  sha1sum     tr
 base64     dirname	   join        nl	  sha224sum   true
 basename   du		   kill        nohup	  sha256sum   truncate
 basenc     echo	   link        nproc	  sha384sum   tsort
 big	    env		   ln	       numfmt	  sha512sum   ttbasic
 cat	    envtest	   login       od	  shred       tty
 chcon	    expand	   logname     passwd	  shuf	      umount
 chgrp	    expr	   ls	       paste	  sleep       uname
 chmod	    factor	   md5sum      pathchk	  sort	      unexpand
 chown	    false	   mkdir       pinky	  split       uniq
 chroot     fmt		   mkfifo      pr	  stat	      unlink
 cksum	    fold	   mkfs        printenv   stdbuf      uptime
 comm	    getdentstest   mknod       printf	  stty	      users
 cp	    getlimits	   mktemp      ptx	  su	      vdir
 csplit     getty	   mmaptest    pwd	  sum	      wc
 cut	    ginstall	   mmaptest2   readlink   sync	      whoami
 dash	    groups	   mount       realpath   tac	      yes
 date	    head	   mv	       rm	  tail
 dcgen	    hello	   mycat       rmdir	  tee
 dd	    hellos	   myls        runcon	  test
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# tail -1 test3
vforkexec start
makejob(0x434b98, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=21
forkparent: In parent shell:  child = 21
vfork: pid=21vfork: pid=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 21, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/tail
abc
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# mv test test4
vforkexec start
makejob(0x434b98, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=22
vfork: pid=0
forkparent: In parent shell:  child = 22
forkchild: shell 22, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/mv
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# ls  
vforkexec start
makejob(0x434b48, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=0
vfork: pid=23
forkchild: shell 23, oldlvl=0, lvforked=1
forkparent: In parent shell:  child = 23
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/ls
bin  dev  etc  home  lib  test2  test3	test4  usr
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# ls -l /bin
vforkexec start
makejob(0x434b98, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=24
vfork: pid=24vfork: pid=0
forkparent: In parent shell:  child = 24
forkchild: shell 24, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /bin/ls
total 19254
-rwxr-xr-x 1 root root 135880 Feb 25  2022 '['
-rwxr-xr-x 1 root root 164784 Feb 25  2022  b2sum
-rwxr-xr-x 1 root root 145672 Feb 25  2022  base32
-rwxr-xr-x 1 root root 145664 Feb 25  2022  base64
-rwxr-xr-x 1 root root 130368 Feb 25  2022  basename
-rwxr-xr-x 1 root root 156104 Feb 25  2022  basenc
-rwxr-xr-x 1 root root  41872 Feb 25  2022  big
-rwxr-xr-x 1 root root 135696 Feb 25  2022  cat
-rwxr-xr-x 1 root root 174840 Feb 25  2022  chcon
-rwxr-xr-x 1 root root 191864 Feb 25  2022  chgrp
-rwxr-xr-x 1 root root 169520 Feb 25  2022  chmod
-rwxr-xr-x 1 root root 196144 Feb 25  2022  chown
-rwxr-xr-x 1 root root 180224 Feb 25  2022  chroot
-rwxr-xr-x 1 root root 135848 Feb 25  2022  cksum
-rwxr-xr-x 1 root root 141560 Feb 25  2022  comm
-rwxr-xr-x 1 root root 245728 Feb 25  2022  cp
-rwxr-xr-x 1 root root 258192 Feb 25  2022  csplit
-rwxr-xr-x 1 root root 146960 Feb 25  2022  cut
-rwxr-xr-x 1 root root 258320 Feb 25  2022  dash
-rwxr-xr-x 1 root root 214848 Feb 25  2022  date
-rwxr-xr-x 1 root root   1335 Feb 25  2022  dcgen
-rwxr-xr-x 1 root root 185264 Feb 25  2022  dd
-rwxr-xr-x 1 root root 229400 Feb 25  2022  df
-rwxr-xr-x 1 root root 332120 Feb 25  2022  dir
-rwxr-xr-x 1 root root 179344 Feb 25  2022  dircolors
-rwxr-xr-x 1 root root 129960 Feb 25  2022  dirname
-rwxr-xr-x 1 root root 382712 Feb 25  2022  du
-rwxr-xr-x 1 root root 120368 Feb 25  2022  echo
-rwxr-xr-x 1 root root 151560 Feb 25  2022  env
-rwxr-xr-x 1 root root  39992 Feb 25  2022  envtest
-rwxr-xr-x 1 root root 137200 Feb 25  2022  expand
-rwxr-xr-x 1 root root 259824 Feb 25  2022  expr
-rwxr-xr-x 1 root root 306232 Feb 25  2022  factor
-rwxr-xr-x 1 root root 115672 Feb 25  2022  false
-rwxr-xr-x 1 root root 146696 Feb 25  2022  fmt
-rwxr-xr-x 1 root root 141528 Feb 25  2022  fold
-rwxr-xr-x 1 root root  41112 Feb 25  2022  getdentstest
-rwxr-xr-x 1 root root 158192 Feb 25  2022  getlimits
-rwxr-xr-x 1 root root  66160 Feb 25  2022  getty
-rwxr-xr-x 1 root root 271160 Feb 25  2022  ginstall
-rwxr-xr-x 1 root root 142176 Feb 25  2022  groups
-rwxr-xr-x 1 root root 145176 Feb 25  2022  head
-rwxr-xr-x 1 root root   8112 Feb 25  2022  hello
-rwxr-xr-x 1 root root  16744 Feb 25  2022  hellos
-rwxr-xr-x 1 root root 129792 Feb 25  2022  hostid
-rwxr-xr-x 1 root root 152448 Feb 25  2022  id
-rwxr-xr-x 1 root root  25456 Feb 25  2022  init
-rwxr-xr-x 1 root root 157056 Feb 25  2022  join
-rwxr-xr-x 1 root root 136552 Feb 25  2022  kill
-rwxr-xr-x 1 root root 129784 Feb 25  2022  link
-rwxr-xr-x 1 root root 187880 Feb 25  2022  ln
-rwxr-xr-x 1 root root  80048 Feb 25  2022  login
-rwxr-xr-x 1 root root 130000 Feb 25  2022  logname
-rwxr-xr-x 1 root root 332120 Feb 25  2022  ls
-rwxr-xr-x 1 root root 150240 Feb 25  2022  md5sum
-rwxr-xr-x 1 root root 148648 Feb 25  2022  mkdir
-rwxr-xr-x 1 root root 130512 Feb 25  2022  mkfifo
-rwxr-xr-x 1 root root  56960 Feb 25  2022  mkfs
-rwxr-xr-x 1 root root 140256 Feb 25  2022  mknod
-rwxr-xr-x 1 root root 147184 Feb 25  2022  mktemp
-rwxr-xr-x 1 root root  58224 Feb 25  2022  mmaptest
-rwxr-xr-x 1 root root  85720 Feb 25  2022  mmaptest2
-rwxr-xr-x 1 root root  16936 Feb 25  2022  mount
-rwxr-xr-x 1 root root 256176 Feb 25  2022  mv
-rwxr-xr-x 1 root root  40552 Feb 25  2022  mycat
-rwxr-xr-x 1 root root  46520 Feb 25  2022  myls
-rwxr-xr-x 1 root root  40872 Feb 25  2022  mywc
-rwxr-xr-x 1 root root  25752 Feb 25  2022  newinit
-rwxr-xr-x 1 root root 135592 Feb 25  2022  nice
-rwxr-xr-x 1 root root 244928 Feb 25  2022  nl
-rwxr-xr-x 1 root root 136016 Feb 25  2022  nohup
-rwxr-xr-x 1 root root 136400 Feb 25  2022  nproc
-rwxr-xr-x 1 root root 182616 Feb 25  2022  numfmt
-rwxr-xr-x 1 root root 188368 Feb 25  2022  od
-rwxr-xr-x 1 root root  70720 Feb 25  2022  passwd
-rwxr-xr-x 1 root root 135840 Feb 25  2022  paste
-rwxr-xr-x 1 root root 130368 Feb 25  2022  pathchk
-rwxr-xr-x 1 root root 181792 Feb 25  2022  pinky
-rwxr-xr-x 1 root root 205456 Feb 25  2022  pr
-rwxr-xr-x 1 root root 129512 Feb 25  2022  printenv
-rwxr-xr-x 1 root root 298384 Feb 25  2022  printf
-rwxr-xr-x 1 root root 412544 Feb 25  2022  ptx
-rwxr-xr-x 1 root root 140560 Feb 25  2022  pwd
-rwxr-xr-x 1 root root 150744 Feb 25  2022  readlink
-rwxr-xr-x 1 root root 151080 Feb 25  2022  realpath
-rwxr-xr-x 1 root root 176536 Feb 25  2022  rm
-rwxr-xr-x 1 root root 131232 Feb 25  2022  rmdir
-rwxr-xr-x 1 root root 129512 Feb 25  2022  runcon
-rwxr-xr-x 1 root root 163864 Feb 25  2022  seq
-rwxr-xr-x 1 root root  59280 Feb 25  2022  sh
-rwxr-xr-x 1 root root 150248 Feb 25  2022  sha1sum
-rwxr-xr-x 1 root root 154600 Feb 25  2022  sha224sum
-rwxr-xr-x 1 root root 154600 Feb 25  2022  sha256sum
-rwxr-xr-x 1 root root 158744 Feb 25  2022  sha384sum
-rwxr-xr-x 1 root root 158744 Feb 25  2022  sha512sum
-rwxr-xr-x 1 root root 171616 Feb 25  2022  shred
-rwxr-xr-x 1 root root 167088 Feb 25  2022  shuf
-rwxr-xr-x 1 root root 154880 Feb 25  2022  sleep
-rwxr-xr-x 1 root root 283000 Feb 25  2022  sort
-rwxr-xr-x 1 root root 174536 Feb 25  2022  split
-rwxr-xr-x 1 root root 266352 Feb 25  2022  stat
-rwxr-xr-x 1 root root 142616 Feb 25  2022  stdbuf
-rwxr-xr-x 1 root root 166528 Feb 25  2022  stty
-rwxr-xr-x 1 root root  72056 Feb 25  2022  su
-rwxr-xr-x 1 root root 151120 Feb 25  2022  sum
-rwxr-xr-x 1 root root 130248 Feb 25  2022  sync
-rwxr-xr-x 1 root root 239136 Feb 25  2022  tac
-rwxr-xr-x 1 root root 198176 Feb 25  2022  tail
-rwxr-xr-x 1 root root 137056 Feb 25  2022  tee
-rwxr-xr-x 1 root root 131360 Feb 25  2022  test
-rwxr-xr-x 1 root root 176688 Feb 25  2022  timeout
-rwxr-xr-x 1 root root 206520 Feb 25  2022  touch
-rwxr-xr-x 1 root root 151120 Feb 25  2022  tr
-rwxr-xr-x 1 root root 115672 Feb 25  2022  true
-rwxr-xr-x 1 root root 140344 Feb 25  2022  truncate
-rwxr-xr-x 1 root root 141400 Feb 25  2022  tsort
-rwxr-xr-x 1 root root  65912 Feb 25  2022  ttbasic
-rwxr-xr-x 1 root root 130608 Feb 25  2022  tty
-rwxr-xr-x 1 root root  16936 Feb 25  2022  umount
-rwxr-xr-x 1 root root 130128 Feb 25  2022  uname
-rwxr-xr-x 1 root root 137144 Feb 25  2022  unexpand
-rwxr-xr-x 1 root root 147776 Feb 25  2022  uniq
-rwxr-xr-x 1 root root 129792 Feb 25  2022  unlink
-rwxr-xr-x 1 root root 167056 Feb 25  2022  uptime
-rwxr-xr-x 1 root root 131832 Feb 25  2022  users
-rwxr-xr-x 1 root root 332120 Feb 25  2022  vdir
-rwxr-xr-x 1 root root 157072 Feb 25  2022  wc
-rwxr-xr-x 1 root root 136600 Feb 25  2022  whoami
-rwxr-xr-x 1 root root 130016 Feb 25  2022  yes
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# objdump -d /bin/hello
vforkexec start
makejob(0x434ba0, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=25
vfork: pid=25vfork: pid=0
forkparent: In parent shell:  child = 25
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 25, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /usr/local/bin/objdump

/bin/hello:     file format elf64-littleaarch64


Disassembly of section .init:

00000000000004d8 <_init>:
 4d8:	a9bf7bfd 	stp	x29, x30, [sp, #-16]!
 4dc:	910003fd 	mov	x29, sp
 4e0:	a8c17bfd 	ldp	x29, x30, [sp], #16
 4e4:	d65f03c0 	ret

Disassembly of section .plt:

00000000000004f0 <.plt>:
 4f0:	a9bf7bf0 	stp	x16, x30, [sp, #-16]!
 4f4:	b0000010 	adrp	x16, 1000 <__FRAME_END__+0x8d8>
 4f8:	f947d611 	ldr	x17, [x16, #4008]
 4fc:	913ea210 	add	x16, x16, #0xfa8
 500:	d61f0220 	br	x17
 504:	d503201f 	nop
 508:	d503201f 	nop
 50c:	d503201f 	nop

0000000000000510 <puts@plt>:
 510:	b0000010 	adrp	x16, 1000 <__FRAME_END__+0x8d8>
 514:	f947da11 	ldr	x17, [x16, #4016]
 518:	913ec210 	add	x16, x16, #0xfb0
 51c:	d61f0220 	br	x17

0000000000000520 <__cxa_finalize@plt>:
 520:	b0000010 	adrp	x16, 1000 <__FRAME_END__+0x8d8>
 524:	f947de11 	ldr	x17, [x16, #4024]
 528:	913ee210 	add	x16, x16, #0xfb8
 52c:	d61f0220 	br	x17

0000000000000530 <__libc_start_main@plt>:
 530:	b0000010 	adrp	x16, 1000 <__FRAME_END__+0x8d8>
 534:	f947e211 	ldr	x17, [x16, #4032]
 538:	913f0210 	add	x16, x16, #0xfc0
 53c:	d61f0220 	br	x17

Disassembly of section .text:

0000000000000540 <main>:
 540:	a9bf7bfd 	stp	x29, x30, [sp, #-16]!
 544:	90000000 	adrp	x0, 0 <_init-0x4d8>
 548:	9119e000 	add	x0, x0, #0x678
 54c:	910003fd 	mov	x29, sp
 550:	97fffff0 	bl	510 <puts@plt>
 554:	52800000 	mov	w0, #0x0                   	// #0
 558:	a8c17bfd 	ldp	x29, x30, [sp], #16
 55c:	d65f03c0 	ret

0000000000000560 <_start>:
 560:	d280001d 	mov	x29, #0x0                   	// #0
 564:	d280001e 	mov	x30, #0x0                   	// #0
 568:	910003e0 	mov	x0, sp
 56c:	b0000001 	adrp	x1, 1000 <__FRAME_END__+0x8d8>
 570:	91372021 	add	x1, x1, #0xdc8
 574:	927cec1f 	and	sp, x0, #0xfffffffffffffff0
 578:	14000001 	b	57c <_start_c>

000000000000057c <_start_c>:
 57c:	aa0003e2 	mov	x2, x0
 580:	b0000004 	adrp	x4, 1000 <__FRAME_END__+0x8d8>
 584:	b0000003 	adrp	x3, 1000 <__FRAME_END__+0x8d8>
 588:	b0000000 	adrp	x0, 1000 <__FRAME_END__+0x8d8>
 58c:	f947fc84 	ldr	x4, [x4, #4088]
 590:	d2800005 	mov	x5, #0x0                   	// #0
 594:	f947ec63 	ldr	x3, [x3, #4056]
 598:	f947f800 	ldr	x0, [x0, #4080]
 59c:	f8408441 	ldr	x1, [x2], #8
 5a0:	17ffffe4 	b	530 <__libc_start_main@plt>
 5a4:	d503201f 	nop

00000000000005a8 <deregister_tm_clones>:
 5a8:	d0000000 	adrp	x0, 2000 <__dso_handle>
 5ac:	91002000 	add	x0, x0, #0x8
 5b0:	d0000001 	adrp	x1, 2000 <__dso_handle>
 5b4:	91002021 	add	x1, x1, #0x8
 5b8:	eb00003f 	cmp	x1, x0
 5bc:	540000c0 	b.eq	5d4 <deregister_tm_clones+0x2c>  // b.none
 5c0:	b0000001 	adrp	x1, 1000 <__FRAME_END__+0x8d8>
 5c4:	f947f421 	ldr	x1, [x1, #4072]
 5c8:	b4000061 	cbz	x1, 5d4 <deregister_tm_clones+0x2c>
 5cc:	aa0103f0 	mov	x16, x1
 5d0:	d61f0200 	br	x16
 5d4:	d65f03c0 	ret

00000000000005d8 <register_tm_clones>:
 5d8:	d0000000 	adrp	x0, 2000 <__dso_handle>
 5dc:	91002000 	add	x0, x0, #0x8
 5e0:	d0000001 	adrp	x1, 2000 <__dso_handle>
 5e4:	91002021 	add	x1, x1, #0x8
 5e8:	cb000021 	sub	x1, x1, x0
 5ec:	d37ffc22 	lsr	x2, x1, #63
 5f0:	8b810c41 	add	x1, x2, x1, asr #3
 5f4:	eb8107ff 	cmp	xzr, x1, asr #1
 5f8:	9341fc21 	asr	x1, x1, #1
 5fc:	540000c0 	b.eq	614 <register_tm_clones+0x3c>  // b.none
 600:	b0000002 	adrp	x2, 1000 <__FRAME_END__+0x8d8>
 604:	f947f042 	ldr	x2, [x2, #4064]
 608:	b4000062 	cbz	x2, 614 <register_tm_clones+0x3c>
 60c:	aa0203f0 	mov	x16, x2
 610:	d61f0200 	br	x16
 614:	d65f03c0 	ret

0000000000000618 <__do_global_dtors_aux>:
 618:	a9be7bfd 	stp	x29, x30, [sp, #-32]!
 61c:	910003fd 	mov	x29, sp
 620:	f9000bf3 	str	x19, [sp, #16]
 624:	d0000013 	adrp	x19, 2000 <__dso_handle>
 628:	39402260 	ldrb	w0, [x19, #8]
 62c:	35000140 	cbnz	w0, 654 <__do_global_dtors_aux+0x3c>
 630:	b0000000 	adrp	x0, 1000 <__FRAME_END__+0x8d8>
 634:	f947e800 	ldr	x0, [x0, #4048]
 638:	b4000080 	cbz	x0, 648 <__do_global_dtors_aux+0x30>
 63c:	d0000000 	adrp	x0, 2000 <__dso_handle>
 640:	f9400000 	ldr	x0, [x0]
 644:	97ffffb7 	bl	520 <__cxa_finalize@plt>
 648:	97ffffd8 	bl	5a8 <deregister_tm_clones>
 64c:	52800020 	mov	w0, #0x1                   	// #1
 650:	39002260 	strb	w0, [x19, #8]
 654:	f9400bf3 	ldr	x19, [sp, #16]
 658:	a8c27bfd 	ldp	x29, x30, [sp], #32
 65c:	d65f03c0 	ret

0000000000000660 <frame_dummy>:
 660:	17ffffde 	b	5d8 <register_tm_clones>

Disassembly of section .fini:

0000000000000664 <_fini>:
 664:	a9bf7bfd 	stp	x29, x30, [sp, #-16]!
 668:	910003fd 	mov	x29, sp
 66c:	a8c17bfd 	ldp	x29, x30, [sp], #16
 670:	d65f03c0 	ret
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# readelf -lW /bin/hello
vforkexec start
makejob(0x434ba0, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=26
vfork: pid=0
forkparent: In parent shell:  child = 26
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
forkchild: shell 26, oldlvl=0, lvforked=0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /usr/local/bin/readelf

Elf file type is DYN (Position-Independent Executable file)
Entry point 0x560
There are 7 program headers, starting at offset 64

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  PHDR           0x000040 0x0000000000000040 0x0000000000000040 0x000188 0x000188 R   0x8
  INTERP         0x0001c8 0x00000000000001c8 0x00000000000001c8 0x00001a 0x00001a R   0x1
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x00072c 0x00072c R E 0x1000
  LOAD           0x000db8 0x0000000000001db8 0x0000000000001db8 0x000250 0x000258 RW  0x1000
  DYNAMIC        0x000dc8 0x0000000000001dc8 0x0000000000001dc8 0x0001d0 0x0001d0 RW  0x8
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
  GNU_RELRO      0x000db8 0x0000000000001db8 0x0000000000001db8 0x000248 0x000248 R   0x1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp 
   02     .interp .hash .gnu.hash .dynsym .dynstr .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame 
   03     .init_array .fini_array .dynamic .got .data .bss 
   04     .dynamic 
   05     
   06     .init_array .fini_array .dynamic .got 
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# readelf -SW /bin/hello
vforkexec start
makejob(0x434ba0, 1) returns %1
call vfork
sys_clone: flags=0x11, stacksiz=0x0, ptid=2, tls=0x423ffc, ctid=4364706, x5=0x433d96, x6=0x6f6676206c6c6163
vfork: pid=27
forkparent: In parent shell:  child = 27
vfork: pid=27vfork: pid=0
forkchild: shell 27, oldlvl=0, lvforked=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0

forkreset end
setsignal start
setsignal end
freejo start
freejob end and forkchild return
tryexec2: /usr/local/bin/readelf
There are 24 section headers, starting at offset 0x19b0:

Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        00000000000001c8 0001c8 00001a 00   A  0   0  1
  [ 2] .hash             HASH            00000000000001e8 0001e8 00003c 04   A  4   0  8
  [ 3] .gnu.hash         GNU_HASH        0000000000000228 000228 000028 00   A  4   0  8
  [ 4] .dynsym           DYNSYM          0000000000000250 000250 0000f0 18   A  5   3  8
  [ 5] .dynstr           STRTAB          0000000000000340 000340 000071 00   A  0   0  1
  [ 6] .rela.dyn         RELA            00000000000003b8 0003b8 0000d8 18   A  4   0  8
  [ 7] .rela.plt         RELA            0000000000000490 000490 000048 18  AI  4  17  8
  [ 8] .init             PROGBITS        00000000000004d8 0004d8 000010 00  AX  0   0  4
  [ 9] .plt              PROGBITS        00000000000004f0 0004f0 000050 10  AX  0   0 16
  [10] .text             PROGBITS        0000000000000540 000540 000124 00  AX  0   0  8
  [11] .fini             PROGBITS        0000000000000664 000664 000010 00  AX  0   0  4
  [12] .rodata           PROGBITS        0000000000000678 000678 000015 01 AMS  0   0  8
  [13] .eh_frame         PROGBITS        0000000000000690 000690 00009c 00   A  0   0  8
  [14] .init_array       INIT_ARRAY      0000000000001db8 000db8 000008 08  WA  0   0  8
  [15] .fini_array       FINI_ARRAY      0000000000001dc0 000dc0 000008 08  WA  0   0  8
  [16] .dynamic          DYNAMIC         0000000000001dc8 000dc8 0001d0 10  WA  5   0  8
  [17] .got              PROGBITS        0000000000001f98 000f98 000068 08  WA  0   0  8
  [18] .data             PROGBITS        0000000000002000 001000 000008 00  WA  0   0  8
  [19] .bss              NOBITS          0000000000002008 001008 000008 00  WA  0   0  1
  [20] .comment          PROGBITS        0000000000000000 001008 00002a 01  MS  0   0  1
  [21] .symtab           SYMTAB          0000000000000000 001038 0006f0 18     22  55  8
  [22] .strtab           STRTAB          0000000000000000 001728 0001d4 00      0   0  1
  [23] .shstrtab         STRTAB          0000000000000000 0018fc 0000af 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), p (processor specific)
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x3, rusage: 0x0
# 
```

# /usr/local/musl/bin/musl-gcc -S musl/src/process/vfork.c

```
$ cat vfork.s
	.arch armv8-a
	.file	"vfork.c"
	.text
	.align	2
	.global	vfork
	.type	vfork, %function
vfork:
.LFB0:
	.cfi_startproc
	stp	x29, x30, [sp, -16]!
	.cfi_def_cfa_offset 16
	.cfi_offset 29, -16
	.cfi_offset 30, -8
	mov	x29, sp
	mov	w2, 0
	mov	w1, 17
	mov	x0, 220
	bl	syscall
	ldp	x29, x30, [sp], 16
	.cfi_restore 30
	.cfi_restore 29
	.cfi_def_cfa_offset 0
	ret
	.cfi_endproc
.LFE0:
	.size	vfork, .-vfork
	.ident	"GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0"
	.section	.note.GNU-stack,"",@progbits
```

## in /usr/local/musl/lib/libc.a

```
0000000000000000 <syscall>:
   0:   f81a0ffe        str     x30, [sp, #-96]!
   4:   aa0003e8        mov     x8, x0
   8:   910183e0        add     x0, sp, #0x60
   c:   a90103e0        stp     x0, x0, [sp, #16]
  10:   9100c3e0        add     x0, sp, #0x30
  14:   f90013e0        str     x0, [sp, #32]
  18:   aa0103e0        mov     x0, x1
  1c:   b9002fff        str     wzr, [sp, #44]
  20:   a9030be1        stp     x1, x2, [sp, #48]
  24:   aa0203e1        mov     x1, x2
  28:   aa0303e2        mov     x2, x3
  2c:   a90413e3        stp     x3, x4, [sp, #64]
  30:   910183e4        add     x4, sp, #0x60
  34:   a9051be5        stp     x5, x6, [sp, #80]
  38:   128002e5        mov     w5, #0xffffffe8                 // #-24
  3c:   b9002be5        str     w5, [sp, #40]
  40:   37f802e5        tbnz    w5, #31, 9c <syscall+0x9c>
  44:   91002083        add     x3, x4, #0x8
  48:   f9000be3        str     x3, [sp, #16]
  4c:   b9402be5        ldr     w5, [sp, #40]
  50:   f9400083        ldr     x3, [x4]
  54:   f9400be4        ldr     x4, [sp, #16]
  58:   37f80365        tbnz    w5, #31, c4 <syscall+0xc4>
  5c:   91003c85        add     x5, x4, #0xf
  60:   927df0a5        and     x5, x5, #0xfffffffffffffff8
  64:   f9000be5        str     x5, [sp, #16]
  68:   b9402be6        ldr     w6, [sp, #40]
  6c:   f9400084        ldr     x4, [x4]
  70:   f9400be5        ldr     x5, [sp, #16]
  74:   36f800a6        tbz     w6, #31, 88 <syscall+0x88>
  78:   31001cdf        cmn     w6, #0x7
  7c:   5400006a        b.ge    88 <syscall+0x88>  // b.tcont
  80:   f9400fe5        ldr     x5, [sp, #24]
  84:   8b26c0a5        add     x5, x5, w6, sxtw
  88:   f94000a5        ldr     x5, [x5]
  8c:   d4000001        svc     #0x0
  90:   94000000        bl      0 <__syscall_ret>
  94:   f84607fe        ldr     x30, [sp], #96
  98:   d65f03c0        ret
  9c:   110020a3        add     w3, w5, #0x8
  a0:   b9002be3        str     w3, [sp, #40]
  a4:   7100007f        cmp     w3, #0x0
  a8:   5400008d        b.le    b8 <syscall+0xb8>
  ac:   91003c83        add     x3, x4, #0xf
  b0:   927df063        and     x3, x3, #0xfffffffffffffff8
  b4:   17ffffe5        b       48 <syscall+0x48>
  b8:   f9400fe4        ldr     x4, [sp, #24]
  bc:   8b25c084        add     x4, x4, w5, sxtw
  c0:   17ffffe3        b       4c <syscall+0x4c>
  c4:   110020a6        add     w6, w5, #0x8
  c8:   b9002be6        str     w6, [sp, #40]
  cc:   710000df        cmp     w6, #0x0
  d0:   54fffc6c        b.gt    5c <syscall+0x5c>
  d4:   f9400fe4        ldr     x4, [sp, #24]
  d8:   8b25c084        add     x4, x4, w5, sxtw
  dc:   17ffffe3        b       68 <syscall+0x68>

0000000000000000 <__syscall_ret>:
   0:   a9bf7bf3        stp     x19, x30, [sp, #-16]!
   4:   b140041f        cmn     x0, #0x1, lsl #12
   8:   aa0003f3        mov     x19, x0
   c:   54000068        b.hi    18 <__syscall_ret+0x18>  // b.pmore
  10:   a8c17bf3        ldp     x19, x30, [sp], #16
  14:   d65f03c0        ret
  18:   94000000        bl      0 <___errno_location>
  1c:   aa0003e1        mov     x1, x0
  20:   4b1303f3        neg     w19, w19
  24:   92800000        mov     x0, #0xffffffffffffffff         // #-1
  28:   b9000033        str     w19, [x1]
  2c:   17fffff9        b       10 <__syscall_ret+0x10>
```

# vfork()

```c
#include <unistd.h>

int main() {
	pid_t pid = vfork();
	return 0;
}
```

```aarch64
$ cat vfork.s
	.arch armv8-a
	.file	"vfork.c"
	.text
	.align	2
	.global	main
	.type	main, %function
main:
.LFB0:
	.cfi_startproc
	stp	x29, x30, [sp, -32]!
	.cfi_def_cfa_offset 32
	.cfi_offset 29, -32
	.cfi_offset 30, -24
	mov	x29, sp
	bl	vfork
	str	w0, [sp, 28]
	mov	w0, 0
	ldp	x29, x30, [sp], 32
	.cfi_restore 30
	.cfi_restore 29
	.cfi_def_cfa_offset 0
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.ident	"GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0"
	.section	.note.GNU-stack,"",@progbits
```

```
$ /usr/local/musl/bin/musl-gcc -v -static -o vfork vfork.c
Using built-in specs.
Reading specs from /usr/local/musl/lib/musl-gcc.specs
rename spec cpp_options to old_cpp_options
COLLECT_GCC=aarch64-linux-gnu-gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc-cross/aarch64-linux-gnu/9/lto-wrapper
Target: aarch64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 9.3.0-17ubuntu1~20.04' --with-bugurl=file:///usr/share/doc/gcc-9/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++,gm2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-9 --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --without-target-system-zlib --enable-libpth-m2 --enable-multiarch --enable-fix-cortex-a53-843419 --disable-werror --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=aarch64-linux-gnu --program-prefix=aarch64-linux-gnu- --includedir=/usr/aarch64-linux-gnu/include
Thread model: posix
gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04) 
COLLECT_GCC_OPTIONS='-v' '-static' '-o' 'vfork' '-specs=/usr/local/musl/lib/musl-gcc.specs' '-mlittle-endian' '-mabi=lp64'
 /usr/lib/gcc-cross/aarch64-linux-gnu/9/cc1 -quiet -v -imultiarch aarch64-linux-gnu vfork.c -nostdinc -isystem /usr/local/musl/include -isystem /usr/lib/gcc-cross/aarch64-linux-gnu/9/include -quiet -dumpbase vfork.c -mlittle-endian -mabi=lp64 -auxbase vfork -version -fasynchronous-unwind-tables -fstack-protector-strong -Wformat -Wformat-security -fstack-clash-protection -o /tmp/ccUtgOD1.s
GNU C17 (Ubuntu 9.3.0-17ubuntu1~20.04) version 9.3.0 (aarch64-linux-gnu)
	compiled by GNU C version 9.3.0, GMP version 6.2.0, MPFR version 4.0.2, MPC version 1.1.0, isl version isl-0.22.1-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
#include "..." search starts here:
#include <...> search starts here:
 /usr/local/musl/include
 /usr/lib/gcc-cross/aarch64-linux-gnu/9/include
End of search list.
GNU C17 (Ubuntu 9.3.0-17ubuntu1~20.04) version 9.3.0 (aarch64-linux-gnu)
	compiled by GNU C version 9.3.0, GMP version 6.2.0, MPFR version 4.0.2, MPC version 1.1.0, isl version isl-0.22.1-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
Compiler executable checksum: 81dc89815922acfa6e9752071147b98e
COLLECT_GCC_OPTIONS='-v' '-static' '-o' 'vfork' '-specs=/usr/local/musl/lib/musl-gcc.specs' '-mlittle-endian' '-mabi=lp64'
 /usr/lib/gcc-cross/aarch64-linux-gnu/9/../../../../aarch64-linux-gnu/bin/as -v -EL -mabi=lp64 -o /tmp/cclCxOK2.o /tmp/ccUtgOD1.s
GNU assembler version 2.34 (aarch64-linux-gnu) using BFD version (GNU Binutils for Ubuntu) 2.34
COMPILER_PATH=/usr/lib/gcc-cross/aarch64-linux-gnu/9/:/usr/lib/gcc-cross/aarch64-linux-gnu/9/:/usr/lib/gcc-cross/aarch64-linux-gnu/:/usr/lib/gcc-cross/aarch64-linux-gnu/9/:/usr/lib/gcc-cross/aarch64-linux-gnu/:/usr/lib/gcc-cross/aarch64-linux-gnu/9/../../../../aarch64-linux-gnu/bin/
LIBRARY_PATH=/usr/lib/gcc-cross/aarch64-linux-gnu/9/:/usr/lib/gcc-cross/aarch64-linux-gnu/9/../../../../aarch64-linux-gnu/lib/../lib/:/lib/../lib/:/usr/lib/aarch64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc-cross/aarch64-linux-gnu/9/../../../../aarch64-linux-gnu/lib/:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-v' '-static' '-o' 'vfork' '-specs=/usr/local/musl/lib/musl-gcc.specs' '-mlittle-endian' '-mabi=lp64'
 /usr/lib/gcc-cross/aarch64-linux-gnu/9/collect2 -plugin /usr/lib/gcc-cross/aarch64-linux-gnu/9/liblto_plugin.so -plugin-opt=/usr/lib/gcc-cross/aarch64-linux-gnu/9/lto-wrapper -plugin-opt=-fresolution=/tmp/ccX8gmX1.res -plugin-opt=-pass-through=/usr/lib/gcc-cross/aarch64-linux-gnu/9/libgcc.a -plugin-opt=-pass-through=/usr/lib/gcc-cross/aarch64-linux-gnu/9/libgcc_eh.a -plugin-opt=-pass-through=-lc -dynamic-linker /lib/ld-musl-aarch64.so.1 -nostdlib -static -z relro -o vfork /usr/local/musl/lib/Scrt1.o /usr/local/musl/lib/crti.o /usr/lib/gcc-cross/aarch64-linux-gnu/9/crtbeginS.o -L/usr/local/musl/lib -L /usr/lib/gcc-cross/aarch64-linux-gnu/9/. /tmp/cclCxOK2.o --start-group /usr/lib/gcc-cross/aarch64-linux-gnu/9/libgcc.a /usr/lib/gcc-cross/aarch64-linux-gnu/9/libgcc_eh.a -lc --end-group /usr/lib/gcc-cross/aarch64-linux-gnu/9/crtendS.o /usr/local/musl/lib/crtn.o
COLLECT_GCC_OPTIONS='-v' '-static' '-o' 'vfork' '-specs=/usr/local/musl/lib/musl-gcc.specs' '-mlittle-endian' '-mabi=lp64'
```

```
000000000040024c <main>:
  40024c:	a9be7bfd 	stp	x29, x30, [sp, #-32]!
  400250:	910003fd 	mov	x29, sp
  400254:	940000b0 	bl	400514 <vfork>
  400258:	b9001fe0 	str	w0, [sp, #28]
  40025c:	52800000 	mov	w0, #0x0                   	// #0
  400260:	a8c27bfd 	ldp	x29, x30, [sp], #32
  400264:	d65f03c0 	ret

0000000000400514 <vfork>:
  400514:       d2801b88        mov     x8, #0xdc                       // #220
  400518:       d2800220        mov     x0, #0x11                       // #17
  40051c:       d2800001        mov     x1, #0x0                        // #0
  400520:       f81f0ffe        str     x30, [sp, #-16]!
  400524:       d4000001        svc     #0x0
  400528:       94000104        bl      400938 <__syscall_ret>
  40052c:       f84107fe        ldr     x30, [sp], #16
  400530:       d65f03c0        ret
  400534:       d503201f        nop
  400538:       d503201f        nop
  40053c:       d503201f        nop

0000000000400938 <__syscall_ret>:
  400938:       a9bf7bf3        stp     x19, x30, [sp, #-16]!
  40093c:       b140041f        cmn     x0, #0x1, lsl #12
  400940:       aa0003f3        mov     x19, x0
  400944:       54000068        b.hi    400950 <__syscall_ret+0x18>  // b.pmore
  400948:       a8c17bf3        ldp     x19, x30, [sp], #16
  40094c:       d65f03c0        ret
  400950:       9400006e        bl      400b08 <__errno_location>
  400954:       aa0003e1        mov     x1, x0
  400958:       4b1303f3        neg     w19, w19
  40095c:       92800000        mov     x0, #0xffffffffffffffff         // #-1
  400960:       b9000033        str     w19, [x1]
  400964:       17fffff9        b       400948 <__syscall_ret+0x10>
  400968:       d503201f        nop
  40096c:       d503201f        nop
```

