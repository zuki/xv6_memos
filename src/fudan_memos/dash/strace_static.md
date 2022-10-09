# maps of dash

```
$ cat maps
00400000-0042a000 r-xp 00000000 b3:02 636547                             /home/dspace/work/c/dash
00439000-0043c000 rw-p 00029000 b3:02 636547                             /home/dspace/work/c/dash
0043c000-0043d000 rw-p 00000000 00:00 0
2464d000-2464e000 rw-p 00000000 00:00 0                                  [heap]
ffffaed72000-ffffaed73000 r--p 00000000 00:00 0                          [vvar]
ffffaed73000-ffffaed74000 r-xp 00000000 00:00 0                          [vdso]
ffffd925b000-ffffd927c000 rw-p 00000000 00:00 0                          [stack]
```

# strace of static linked dash

```
execve("./dash", ["./dash"], 0xffffc1877b00 /* 24 vars */) = 0
set_tid_address(0x43cbf4)               = 2227
getpid()                                = 2227
rt_sigprocmask(SIG_UNBLOCK, [RT_1 RT_2], NULL, 8) = 0
rt_sigaction(SIGCHLD, {sa_handler=0x40f1f8, sa_mask=~[RTMIN RT_1 RT_2], sa_flags=SA_RESTORER, sa_restorer=0x41b8ec}, NULL, 8) = 0
geteuid()                               = 1002
brk(NULL)                               = 0x2464d000
brk(0x2464e000)                         = 0x2464e000
getppid()                               = 2224
newfstatat(AT_FDCWD, "/home/dspace/work/c", {st_mode=S_IFDIR|0775, st_size=4096, ...}, 0) = 0
newfstatat(AT_FDCWD, ".", {st_mode=S_IFDIR|0775, st_size=4096, ...}, 0) = 0
ioctl(0, TIOCGWINSZ, {ws_row=25, ws_col=80, ws_xpixel=880, ws_ypixel=550}) = 0
ioctl(1, TIOCGWINSZ, {ws_row=25, ws_col=80, ws_xpixel=880, ws_ypixel=550}) = 0
rt_sigaction(SIGINT, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGINT, {sa_handler=0x40f1f8, sa_mask=~[RTMIN RT_1 RT_2], sa_flags=SA_RESTORER, sa_restorer=0x41b8ec}, NULL, 8) = 0
rt_sigaction(SIGQUIT, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGQUIT, {sa_handler=SIG_IGN, sa_mask=~[RTMIN RT_1 RT_2], sa_flags=SA_RESTORER, sa_restorer=0x41b8ec}, NULL, 8) = 0
rt_sigaction(SIGTERM, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTERM, {sa_handler=SIG_IGN, sa_mask=~[RTMIN RT_1 RT_2], sa_flags=SA_RESTORER, sa_restorer=0x41b8ec}, NULL, 8) = 0
openat(AT_FDCWD, "/dev/tty", O_RDWR|O_LARGEFILE) = 3
fcntl(3, F_DUPFD, 10)                   = 10
close(3)                                = 0
fcntl(10, F_SETFD, FD_CLOEXEC)          = 0
ioctl(10, TIOCGPGRP, [2224])            = 0
getpgid(0)                              = 2224
rt_sigaction(SIGTSTP, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTSTP, {sa_handler=SIG_IGN, sa_mask=~[RTMIN RT_1 RT_2], sa_flags=SA_RESTORER, sa_restorer=0x41b8ec}, NULL, 8) = 0
rt_sigaction(SIGTTOU, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTTOU, {sa_handler=SIG_IGN, sa_mask=~[RTMIN RT_1 RT_2], sa_flags=SA_RESTORER, sa_restorer=0x41b8ec}, NULL, 8) = 0
rt_sigaction(SIGTTIN, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTTIN, {sa_handler=SIG_DFL, sa_mask=~[RTMIN RT_1 RT_2], sa_flags=SA_RESTORER, sa_restorer=0x41b8ec}, NULL, 8) = 0
setpgid(0, 2227)                        = 0
ioctl(10, TIOCSPGRP, [2227])            = 0
write(2, "$ ", 2)                       = 2
read(0, "ps\n", 1024)                   = 3
newfstatat(AT_FDCWD, "/usr/local/sbin/ps", 0xffffd927af60, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/local/bin/ps", 0xffffd927af60, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/sbin/ps", 0xffffd927af60, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/bin/ps", {st_mode=S_IFREG|0755, st_size=137456, ...}, 0) = 0
rt_sigprocmask(SIG_SETMASK, ~[RTMIN RT_1 RT_2], NULL, 8) = 0
clone(child_stack=NULL, flags=SIGCHLD)  = 2228
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
setpgid(2228, 2228)                     = 0
wait4(-1, [{WIFEXITED(s) && WEXITSTATUS(s) == 0}], WSTOPPED, NULL) = 2228
--- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=2228, si_uid=1002, si_status=0, si_utime=1, si_stime=2} ---
rt_sigreturn({mask=[]})                 = 2228
wait4(-1, 0xffffd927af9c, WNOHANG|WSTOPPED, NULL) = -1 ECHILD (No child processes)
ioctl(10, TIOCSPGRP, [2227])            = 0
write(2, "$ ", 2)                       = 2
read(0, "exit\n", 1024)                 = 5
ioctl(10, TIOCSPGRP, [2224])            = 0
setpgid(0, 2224)                        = 0
close(10)                               = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```
