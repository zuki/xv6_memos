# raspi linuxで実行

```
execve("/usr/bin/dash", ["dash"], 0xfffffd8957a0 /* 24 vars */) = 0
brk(NULL)                               = 0xaaaad0e1f000
faccessat(AT_FDCWD, "/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=26913, ...}) = 0
mmap(NULL, 26913, PROT_READ, MAP_PRIVATE, 3, 0) = 0xffffb8aad000
close(3)                                = 0
openat(AT_FDCWD, "/lib/aarch64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0\267\0\1\0\0\0\350A\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1449992, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xffffb8aab000
mmap(NULL, 1518672, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0xffffb8912000
mprotect(0xffffb8a6c000, 65536, PROT_NONE) = 0
mmap(0xffffb8a7c000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x15a000) = 0xffffb8a7c000
mmap(0xffffb8a82000, 11344, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xffffb8a82000
close(3)                                = 0
mprotect(0xffffb8a7c000, 12288, PROT_READ) = 0
mprotect(0xaaaab872b000, 8192, PROT_READ) = 0
mprotect(0xffffb8ab6000, 4096, PROT_READ) = 0
munmap(0xffffb8aad000, 26913)           = 0
getuid()                                = 1002
getgid()                                = 1002
getpid()                                = 3686
rt_sigaction(SIGCHLD, {sa_handler=0xaaaab8711f48, sa_mask=~[RTMIN RT_1], sa_flags=0}, NULL, 8) = 0
geteuid()                               = 1002
brk(NULL)                               = 0xaaaad0e1f000
brk(0xaaaad0e40000)                     = 0xaaaad0e40000
getppid()                               = 3683
newfstatat(AT_FDCWD, "/home/dspace/work/c", {st_mode=S_IFDIR|0775, st_size=4096, ...}, 0) = 0
newfstatat(AT_FDCWD, ".", {st_mode=S_IFDIR|0775, st_size=4096, ...}, 0) = 0
ioctl(0, TCGETS, {B9600 opost isig icanon echo ...}) = 0
ioctl(1, TCGETS, {B9600 opost isig icanon echo ...}) = 0
geteuid()                               = 1002
getegid()                               = 1002
rt_sigaction(SIGINT, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGINT, {sa_handler=0xaaaab8711f48, sa_mask=~[RTMIN RT_1], sa_flags=0}, NULL, 8) = 0
rt_sigaction(SIGQUIT, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGQUIT, {sa_handler=SIG_IGN, sa_mask=~[RTMIN RT_1], sa_flags=0}, NULL, 8) = 0
rt_sigaction(SIGTERM, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTERM, {sa_handler=SIG_IGN, sa_mask=~[RTMIN RT_1], sa_flags=0}, NULL, 8) = 0
openat(AT_FDCWD, "/dev/tty", O_RDWR)    = 3
fcntl(3, F_DUPFD, 10)                   = 10
close(3)                                = 0
fcntl(10, F_SETFD, FD_CLOEXEC)          = 0
ioctl(10, TIOCGPGRP, [3683])            = 0
getpgid(0)                              = 3683
rt_sigaction(SIGTSTP, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTSTP, {sa_handler=SIG_IGN, sa_mask=~[RTMIN RT_1], sa_flags=0}, NULL, 8) = 0
rt_sigaction(SIGTTOU, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTTOU, {sa_handler=SIG_IGN, sa_mask=~[RTMIN RT_1], sa_flags=0}, NULL, 8) = 0
rt_sigaction(SIGTTIN, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTTIN, {sa_handler=SIG_DFL, sa_mask=~[RTMIN RT_1], sa_flags=0}, NULL, 8) = 0
setpgid(0, 3686)                        = 0
ioctl(10, TIOCSPGRP, [3686])            = 0
wait4(-1, 0xffffce79205c, WNOHANG|WSTOPPED, NULL) = -1 ECHILD (No child processes)
write(2, "$ ", 2)                       = 2
read(0, "exit\n", 8192)                 = 5
ioctl(10, TIOCSPGRP, [3683])            = 0
setpgid(0, 3683)                        = 0
close(10)                               = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```
