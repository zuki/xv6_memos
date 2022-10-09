# musl インストール

```
$ git clone -o upstream git://git.musl-libc.org/musl
$ cd musl
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ ./configure --host=aarch64-linux-gnu
$ make
$ sudo make install
$ sudo make install
./tools/install.sh -D -m 644 lib/Scrt1.o /usr/local/musl/lib/Scrt1.o
./tools/install.sh -D -m 644 lib/crti.o /usr/local/musl/lib/crti.o
./tools/install.sh -D -m 644 lib/crtn.o /usr/local/musl/lib/crtn.o
./tools/install.sh -D -m 644 lib/crt1.o /usr/local/musl/lib/crt1.o
./tools/install.sh -D -m 644 lib/rcrt1.o /usr/local/musl/lib/rcrt1.o
./tools/install.sh -D -m 644 lib/libc.a /usr/local/musl/lib/libc.a
./tools/install.sh -D -m 755 lib/libc.so /usr/local/musl/lib/libc.so
./tools/install.sh -D -m 644 lib/libm.a /usr/local/musl/lib/libm.a
./tools/install.sh -D -m 644 lib/librt.a /usr/local/musl/lib/librt.a
./tools/install.sh -D -m 644 lib/libpthread.a /usr/local/musl/lib/libpthread.a
./tools/install.sh -D -m 644 lib/libcrypt.a /usr/local/musl/lib/libcrypt.a
./tools/install.sh -D -m 644 lib/libutil.a /usr/local/musl/lib/libutil.a
./tools/install.sh -D -m 644 lib/libxnet.a /usr/local/musl/lib/libxnet.a
./tools/install.sh -D -m 644 lib/libresolv.a /usr/local/musl/lib/libresolv.a
./tools/install.sh -D -m 644 lib/libdl.a /usr/local/musl/lib/libdl.a
./tools/install.sh -D -m 644 lib/musl-gcc.specs /usr/local/musl/lib/musl-gcc.specs
./tools/install.sh -D -l /usr/local/musl/lib/libc.so /lib/ld-musl-aarch64.so.1 || true
./tools/install.sh -D -m 644 include/aio.h /usr/local/musl/include/aio.h
./tools/install.sh -D -m 644 include/alloca.h /usr/local/musl/include/alloca.h
./tools/install.sh -D -m 644 include/ar.h /usr/local/musl/include/ar.h
./tools/install.sh -D -m 644 include/arpa/ftp.h /usr/local/musl/include/arpa/ftp.h
./tools/install.sh -D -m 644 include/arpa/inet.h /usr/local/musl/include/arpa/inet.h
./tools/install.sh -D -m 644 include/arpa/nameser.h /usr/local/musl/include/arpa/nameser.h
./tools/install.sh -D -m 644 include/arpa/nameser_compat.h /usr/local/musl/include/arpa/nameser_compat.h
./tools/install.sh -D -m 644 include/arpa/telnet.h /usr/local/musl/include/arpa/telnet.h
./tools/install.sh -D -m 644 include/arpa/tftp.h /usr/local/musl/include/arpa/tftp.h
./tools/install.sh -D -m 644 include/assert.h /usr/local/musl/include/assert.h
./tools/install.sh -D -m 644 obj/include/bits/alltypes.h /usr/local/musl/include/bits/alltypes.h
./tools/install.sh -D -m 644 arch/generic/bits/dirent.h /usr/local/musl/include/bits/dirent.h
./tools/install.sh -D -m 644 arch/generic/bits/errno.h /usr/local/musl/include/bits/errno.h
./tools/install.sh -D -m 644 arch/aarch64/bits/fcntl.h /usr/local/musl/include/bits/fcntl.h
./tools/install.sh -D -m 644 arch/aarch64/bits/fenv.h /usr/local/musl/include/bits/fenv.h
./tools/install.sh -D -m 644 arch/aarch64/bits/float.h /usr/local/musl/include/bits/float.h
./tools/install.sh -D -m 644 arch/aarch64/bits/hwcap.h /usr/local/musl/include/bits/hwcap.h
./tools/install.sh -D -m 644 arch/generic/bits/io.h /usr/local/musl/include/bits/io.h
./tools/install.sh -D -m 644 arch/generic/bits/ioctl.h /usr/local/musl/include/bits/ioctl.h
./tools/install.sh -D -m 644 arch/generic/bits/ioctl_fix.h /usr/local/musl/include/bits/ioctl_fix.h
./tools/install.sh -D -m 644 arch/generic/bits/ipc.h /usr/local/musl/include/bits/ipc.h
./tools/install.sh -D -m 644 arch/generic/bits/ipcstat.h /usr/local/musl/include/bits/ipcstat.h
./tools/install.sh -D -m 644 arch/generic/bits/kd.h /usr/local/musl/include/bits/kd.h
./tools/install.sh -D -m 644 arch/generic/bits/limits.h /usr/local/musl/include/bits/limits.h
./tools/install.sh -D -m 644 arch/generic/bits/link.h /usr/local/musl/include/bits/link.h
./tools/install.sh -D -m 644 arch/aarch64/bits/mman.h /usr/local/musl/include/bits/mman.h
./tools/install.sh -D -m 644 arch/generic/bits/msg.h /usr/local/musl/include/bits/msg.h
./tools/install.sh -D -m 644 arch/generic/bits/poll.h /usr/local/musl/include/bits/poll.h
./tools/install.sh -D -m 644 arch/aarch64/bits/posix.h /usr/local/musl/include/bits/posix.h
./tools/install.sh -D -m 644 arch/generic/bits/ptrace.h /usr/local/musl/include/bits/ptrace.h
./tools/install.sh -D -m 644 arch/aarch64/bits/reg.h /usr/local/musl/include/bits/reg.h
./tools/install.sh -D -m 644 arch/generic/bits/resource.h /usr/local/musl/include/bits/resource.h
./tools/install.sh -D -m 644 arch/generic/bits/sem.h /usr/local/musl/include/bits/sem.h
./tools/install.sh -D -m 644 arch/aarch64/bits/setjmp.h /usr/local/musl/include/bits/setjmp.h
./tools/install.sh -D -m 644 arch/generic/bits/shm.h /usr/local/musl/include/bits/shm.h
./tools/install.sh -D -m 644 arch/aarch64/bits/signal.h /usr/local/musl/include/bits/signal.h
./tools/install.sh -D -m 644 arch/generic/bits/socket.h /usr/local/musl/include/bits/socket.h
./tools/install.sh -D -m 644 arch/generic/bits/soundcard.h /usr/local/musl/include/bits/soundcard.h
./tools/install.sh -D -m 644 arch/aarch64/bits/stat.h /usr/local/musl/include/bits/stat.h
./tools/install.sh -D -m 644 arch/generic/bits/statfs.h /usr/local/musl/include/bits/statfs.h
./tools/install.sh -D -m 644 arch/aarch64/bits/stdint.h /usr/local/musl/include/bits/stdint.h
./tools/install.sh -D -m 644 obj/include/bits/syscall.h /usr/local/musl/include/bits/syscall.h
./tools/install.sh -D -m 644 arch/generic/bits/termios.h /usr/local/musl/include/bits/termios.h
./tools/install.sh -D -m 644 arch/aarch64/bits/user.h /usr/local/musl/include/bits/user.h
./tools/install.sh -D -m 644 arch/generic/bits/vt.h /usr/local/musl/include/bits/vt.h
./tools/install.sh -D -m 644 include/byteswap.h /usr/local/musl/include/byteswap.h
./tools/install.sh -D -m 644 include/complex.h /usr/local/musl/include/complex.h
./tools/install.sh -D -m 644 include/cpio.h /usr/local/musl/include/cpio.h
./tools/install.sh -D -m 644 include/crypt.h /usr/local/musl/include/crypt.h
./tools/install.sh -D -m 644 include/ctype.h /usr/local/musl/include/ctype.h
./tools/install.sh -D -m 644 include/dirent.h /usr/local/musl/include/dirent.h
./tools/install.sh -D -m 644 include/dlfcn.h /usr/local/musl/include/dlfcn.h
./tools/install.sh -D -m 644 include/elf.h /usr/local/musl/include/elf.h
./tools/install.sh -D -m 644 include/endian.h /usr/local/musl/include/endian.h
./tools/install.sh -D -m 644 include/err.h /usr/local/musl/include/err.h
./tools/install.sh -D -m 644 include/errno.h /usr/local/musl/include/errno.h
./tools/install.sh -D -m 644 include/fcntl.h /usr/local/musl/include/fcntl.h
./tools/install.sh -D -m 644 include/features.h /usr/local/musl/include/features.h
./tools/install.sh -D -m 644 include/fenv.h /usr/local/musl/include/fenv.h
./tools/install.sh -D -m 644 include/float.h /usr/local/musl/include/float.h
./tools/install.sh -D -m 644 include/fmtmsg.h /usr/local/musl/include/fmtmsg.h
./tools/install.sh -D -m 644 include/fnmatch.h /usr/local/musl/include/fnmatch.h
./tools/install.sh -D -m 644 include/ftw.h /usr/local/musl/include/ftw.h
./tools/install.sh -D -m 644 include/getopt.h /usr/local/musl/include/getopt.h
./tools/install.sh -D -m 644 include/glob.h /usr/local/musl/include/glob.h
./tools/install.sh -D -m 644 include/grp.h /usr/local/musl/include/grp.h
./tools/install.sh -D -m 644 include/iconv.h /usr/local/musl/include/iconv.h
./tools/install.sh -D -m 644 include/ifaddrs.h /usr/local/musl/include/ifaddrs.h
./tools/install.sh -D -m 644 include/inttypes.h /usr/local/musl/include/inttypes.h
./tools/install.sh -D -m 644 include/iso646.h /usr/local/musl/include/iso646.h
./tools/install.sh -D -m 644 include/langinfo.h /usr/local/musl/include/langinfo.h
./tools/install.sh -D -m 644 include/lastlog.h /usr/local/musl/include/lastlog.h
./tools/install.sh -D -m 644 include/libgen.h /usr/local/musl/include/libgen.h
./tools/install.sh -D -m 644 include/libintl.h /usr/local/musl/include/libintl.h
./tools/install.sh -D -m 644 include/limits.h /usr/local/musl/include/limits.h
./tools/install.sh -D -m 644 include/link.h /usr/local/musl/include/link.h
./tools/install.sh -D -m 644 include/locale.h /usr/local/musl/include/locale.h
./tools/install.sh -D -m 644 include/malloc.h /usr/local/musl/include/malloc.h
./tools/install.sh -D -m 644 include/math.h /usr/local/musl/include/math.h
./tools/install.sh -D -m 644 include/memory.h /usr/local/musl/include/memory.h
./tools/install.sh -D -m 644 include/mntent.h /usr/local/musl/include/mntent.h
./tools/install.sh -D -m 644 include/monetary.h /usr/local/musl/include/monetary.h
./tools/install.sh -D -m 644 include/mqueue.h /usr/local/musl/include/mqueue.h
./tools/install.sh -D -m 644 include/net/ethernet.h /usr/local/musl/include/net/ethernet.h
./tools/install.sh -D -m 644 include/net/if.h /usr/local/musl/include/net/if.h
./tools/install.sh -D -m 644 include/net/if_arp.h /usr/local/musl/include/net/if_arp.h
./tools/install.sh -D -m 644 include/net/route.h /usr/local/musl/include/net/route.h
./tools/install.sh -D -m 644 include/netdb.h /usr/local/musl/include/netdb.h
./tools/install.sh -D -m 644 include/netinet/ether.h /usr/local/musl/include/netinet/ether.h
./tools/install.sh -D -m 644 include/netinet/icmp6.h /usr/local/musl/include/netinet/icmp6.h
./tools/install.sh -D -m 644 include/netinet/if_ether.h /usr/local/musl/include/netinet/if_ether.h
./tools/install.sh -D -m 644 include/netinet/igmp.h /usr/local/musl/include/netinet/igmp.h
./tools/install.sh -D -m 644 include/netinet/in.h /usr/local/musl/include/netinet/in.h
./tools/install.sh -D -m 644 include/netinet/in_systm.h /usr/local/musl/include/netinet/in_systm.h
./tools/install.sh -D -m 644 include/netinet/ip.h /usr/local/musl/include/netinet/ip.h
./tools/install.sh -D -m 644 include/netinet/ip6.h /usr/local/musl/include/netinet/ip6.h
./tools/install.sh -D -m 644 include/netinet/ip_icmp.h /usr/local/musl/include/netinet/ip_icmp.h
./tools/install.sh -D -m 644 include/netinet/tcp.h /usr/local/musl/include/netinet/tcp.h
./tools/install.sh -D -m 644 include/netinet/udp.h /usr/local/musl/include/netinet/udp.h
./tools/install.sh -D -m 644 include/netpacket/packet.h /usr/local/musl/include/netpacket/packet.h
./tools/install.sh -D -m 644 include/nl_types.h /usr/local/musl/include/nl_types.h
./tools/install.sh -D -m 644 include/paths.h /usr/local/musl/include/paths.h
./tools/install.sh -D -m 644 include/poll.h /usr/local/musl/include/poll.h
./tools/install.sh -D -m 644 include/pthread.h /usr/local/musl/include/pthread.h
./tools/install.sh -D -m 644 include/pty.h /usr/local/musl/include/pty.h
./tools/install.sh -D -m 644 include/pwd.h /usr/local/musl/include/pwd.h
./tools/install.sh -D -m 644 include/regex.h /usr/local/musl/include/regex.h
./tools/install.sh -D -m 644 include/resolv.h /usr/local/musl/include/resolv.h
./tools/install.sh -D -m 644 include/sched.h /usr/local/musl/include/sched.h
./tools/install.sh -D -m 644 include/scsi/scsi.h /usr/local/musl/include/scsi/scsi.h
./tools/install.sh -D -m 644 include/scsi/scsi_ioctl.h /usr/local/musl/include/scsi/scsi_ioctl.h
./tools/install.sh -D -m 644 include/scsi/sg.h /usr/local/musl/include/scsi/sg.h
./tools/install.sh -D -m 644 include/search.h /usr/local/musl/include/search.h
./tools/install.sh -D -m 644 include/semaphore.h /usr/local/musl/include/semaphore.h
./tools/install.sh -D -m 644 include/setjmp.h /usr/local/musl/include/setjmp.h
./tools/install.sh -D -m 644 include/shadow.h /usr/local/musl/include/shadow.h
./tools/install.sh -D -m 644 include/signal.h /usr/local/musl/include/signal.h
./tools/install.sh -D -m 644 include/spawn.h /usr/local/musl/include/spawn.h
./tools/install.sh -D -m 644 include/stdalign.h /usr/local/musl/include/stdalign.h
./tools/install.sh -D -m 644 include/stdarg.h /usr/local/musl/include/stdarg.h
./tools/install.sh -D -m 644 include/stdbool.h /usr/local/musl/include/stdbool.h
./tools/install.sh -D -m 644 include/stdc-predef.h /usr/local/musl/include/stdc-predef.h
./tools/install.sh -D -m 644 include/stddef.h /usr/local/musl/include/stddef.h
./tools/install.sh -D -m 644 include/stdint.h /usr/local/musl/include/stdint.h
./tools/install.sh -D -m 644 include/stdio.h /usr/local/musl/include/stdio.h
./tools/install.sh -D -m 644 include/stdio_ext.h /usr/local/musl/include/stdio_ext.h
./tools/install.sh -D -m 644 include/stdlib.h /usr/local/musl/include/stdlib.h
./tools/install.sh -D -m 644 include/stdnoreturn.h /usr/local/musl/include/stdnoreturn.h
./tools/install.sh -D -m 644 include/string.h /usr/local/musl/include/string.h
./tools/install.sh -D -m 644 include/strings.h /usr/local/musl/include/strings.h
./tools/install.sh -D -m 644 include/stropts.h /usr/local/musl/include/stropts.h
./tools/install.sh -D -m 644 include/sys/acct.h /usr/local/musl/include/sys/acct.h
./tools/install.sh -D -m 644 include/sys/auxv.h /usr/local/musl/include/sys/auxv.h
./tools/install.sh -D -m 644 include/sys/cachectl.h /usr/local/musl/include/sys/cachectl.h
./tools/install.sh -D -m 644 include/sys/dir.h /usr/local/musl/include/sys/dir.h
./tools/install.sh -D -m 644 include/sys/epoll.h /usr/local/musl/include/sys/epoll.h
./tools/install.sh -D -m 644 include/sys/errno.h /usr/local/musl/include/sys/errno.h
./tools/install.sh -D -m 644 include/sys/eventfd.h /usr/local/musl/include/sys/eventfd.h
./tools/install.sh -D -m 644 include/sys/fanotify.h /usr/local/musl/include/sys/fanotify.h
./tools/install.sh -D -m 644 include/sys/fcntl.h /usr/local/musl/include/sys/fcntl.h
./tools/install.sh -D -m 644 include/sys/file.h /usr/local/musl/include/sys/file.h
./tools/install.sh -D -m 644 include/sys/fsuid.h /usr/local/musl/include/sys/fsuid.h
./tools/install.sh -D -m 644 include/sys/inotify.h /usr/local/musl/include/sys/inotify.h
./tools/install.sh -D -m 644 include/sys/io.h /usr/local/musl/include/sys/io.h
./tools/install.sh -D -m 644 include/sys/ioctl.h /usr/local/musl/include/sys/ioctl.h
./tools/install.sh -D -m 644 include/sys/ipc.h /usr/local/musl/include/sys/ipc.h
./tools/install.sh -D -m 644 include/sys/kd.h /usr/local/musl/include/sys/kd.h
./tools/install.sh -D -m 644 include/sys/klog.h /usr/local/musl/include/sys/klog.h
./tools/install.sh -D -m 644 include/sys/membarrier.h /usr/local/musl/include/sys/membarrier.h
./tools/install.sh -D -m 644 include/sys/mman.h /usr/local/musl/include/sys/mman.h
./tools/install.sh -D -m 644 include/sys/mount.h /usr/local/musl/include/sys/mount.h
./tools/install.sh -D -m 644 include/sys/msg.h /usr/local/musl/include/sys/msg.h
./tools/install.sh -D -m 644 include/sys/mtio.h /usr/local/musl/include/sys/mtio.h
./tools/install.sh -D -m 644 include/sys/param.h /usr/local/musl/include/sys/param.h
./tools/install.sh -D -m 644 include/sys/personality.h /usr/local/musl/include/sys/personality.h
./tools/install.sh -D -m 644 include/sys/poll.h /usr/local/musl/include/sys/poll.h
./tools/install.sh -D -m 644 include/sys/prctl.h /usr/local/musl/include/sys/prctl.h
./tools/install.sh -D -m 644 include/sys/procfs.h /usr/local/musl/include/sys/procfs.h
./tools/install.sh -D -m 644 include/sys/ptrace.h /usr/local/musl/include/sys/ptrace.h
./tools/install.sh -D -m 644 include/sys/quota.h /usr/local/musl/include/sys/quota.h
./tools/install.sh -D -m 644 include/sys/random.h /usr/local/musl/include/sys/random.h
./tools/install.sh -D -m 644 include/sys/reboot.h /usr/local/musl/include/sys/reboot.h
./tools/install.sh -D -m 644 include/sys/reg.h /usr/local/musl/include/sys/reg.h
./tools/install.sh -D -m 644 include/sys/resource.h /usr/local/musl/include/sys/resource.h
./tools/install.sh -D -m 644 include/sys/select.h /usr/local/musl/include/sys/select.h
./tools/install.sh -D -m 644 include/sys/sem.h /usr/local/musl/include/sys/sem.h
./tools/install.sh -D -m 644 include/sys/sendfile.h /usr/local/musl/include/sys/sendfile.h
./tools/install.sh -D -m 644 include/sys/shm.h /usr/local/musl/include/sys/shm.h
./tools/install.sh -D -m 644 include/sys/signal.h /usr/local/musl/include/sys/signal.h
./tools/install.sh -D -m 644 include/sys/signalfd.h /usr/local/musl/include/sys/signalfd.h
./tools/install.sh -D -m 644 include/sys/socket.h /usr/local/musl/include/sys/socket.h
./tools/install.sh -D -m 644 include/sys/soundcard.h /usr/local/musl/include/sys/soundcard.h
./tools/install.sh -D -m 644 include/sys/stat.h /usr/local/musl/include/sys/stat.h
./tools/install.sh -D -m 644 include/sys/statfs.h /usr/local/musl/include/sys/statfs.h
./tools/install.sh -D -m 644 include/sys/statvfs.h /usr/local/musl/include/sys/statvfs.h
./tools/install.sh -D -m 644 include/sys/stropts.h /usr/local/musl/include/sys/stropts.h
./tools/install.sh -D -m 644 include/sys/swap.h /usr/local/musl/include/sys/swap.h
./tools/install.sh -D -m 644 include/sys/syscall.h /usr/local/musl/include/sys/syscall.h
./tools/install.sh -D -m 644 include/sys/sysinfo.h /usr/local/musl/include/sys/sysinfo.h
./tools/install.sh -D -m 644 include/sys/syslog.h /usr/local/musl/include/sys/syslog.h
./tools/install.sh -D -m 644 include/sys/sysmacros.h /usr/local/musl/include/sys/sysmacros.h
./tools/install.sh -D -m 644 include/sys/termios.h /usr/local/musl/include/sys/termios.h
./tools/install.sh -D -m 644 include/sys/time.h /usr/local/musl/include/sys/time.h
./tools/install.sh -D -m 644 include/sys/timeb.h /usr/local/musl/include/sys/timeb.h
./tools/install.sh -D -m 644 include/sys/timerfd.h /usr/local/musl/include/sys/timerfd.h
./tools/install.sh -D -m 644 include/sys/times.h /usr/local/musl/include/sys/times.h
./tools/install.sh -D -m 644 include/sys/timex.h /usr/local/musl/include/sys/timex.h
./tools/install.sh -D -m 644 include/sys/ttydefaults.h /usr/local/musl/include/sys/ttydefaults.h
./tools/install.sh -D -m 644 include/sys/types.h /usr/local/musl/include/sys/types.h
./tools/install.sh -D -m 644 include/sys/ucontext.h /usr/local/musl/include/sys/ucontext.h
./tools/install.sh -D -m 644 include/sys/uio.h /usr/local/musl/include/sys/uio.h
./tools/install.sh -D -m 644 include/sys/un.h /usr/local/musl/include/sys/un.h
./tools/install.sh -D -m 644 include/sys/user.h /usr/local/musl/include/sys/user.h
./tools/install.sh -D -m 644 include/sys/utsname.h /usr/local/musl/include/sys/utsname.h
./tools/install.sh -D -m 644 include/sys/vfs.h /usr/local/musl/include/sys/vfs.h
./tools/install.sh -D -m 644 include/sys/vt.h /usr/local/musl/include/sys/vt.h
./tools/install.sh -D -m 644 include/sys/wait.h /usr/local/musl/include/sys/wait.h
./tools/install.sh -D -m 644 include/sys/xattr.h /usr/local/musl/include/sys/xattr.h
./tools/install.sh -D -m 644 include/syscall.h /usr/local/musl/include/syscall.h
./tools/install.sh -D -m 644 include/sysexits.h /usr/local/musl/include/sysexits.h
./tools/install.sh -D -m 644 include/syslog.h /usr/local/musl/include/syslog.h
./tools/install.sh -D -m 644 include/tar.h /usr/local/musl/include/tar.h
./tools/install.sh -D -m 644 include/termios.h /usr/local/musl/include/termios.h
./tools/install.sh -D -m 644 include/tgmath.h /usr/local/musl/include/tgmath.h
./tools/install.sh -D -m 644 include/threads.h /usr/local/musl/include/threads.h
./tools/install.sh -D -m 644 include/time.h /usr/local/musl/include/time.h
./tools/install.sh -D -m 644 include/uchar.h /usr/local/musl/include/uchar.h
./tools/install.sh -D -m 644 include/ucontext.h /usr/local/musl/include/ucontext.h
./tools/install.sh -D -m 644 include/ulimit.h /usr/local/musl/include/ulimit.h
./tools/install.sh -D -m 644 include/unistd.h /usr/local/musl/include/unistd.h
./tools/install.sh -D -m 644 include/utime.h /usr/local/musl/include/utime.h
./tools/install.sh -D -m 644 include/utmp.h /usr/local/musl/include/utmp.h
./tools/install.sh -D -m 644 include/utmpx.h /usr/local/musl/include/utmpx.h
./tools/install.sh -D -m 644 include/values.h /usr/local/musl/include/values.h
./tools/install.sh -D -m 644 include/wait.h /usr/local/musl/include/wait.h
./tools/install.sh -D -m 644 include/wchar.h /usr/local/musl/include/wchar.h
./tools/install.sh -D -m 644 include/wctype.h /usr/local/musl/include/wctype.h
./tools/install.sh -D -m 644 include/wordexp.h /usr/local/musl/include/wordexp.h
./tools/install.sh -D obj/musl-gcc /usr/local/musl/bin/musl-gcc
```

# muslを利用してコンパイル

```
$ cat hello.c
#include <stdio.h>

int main() {
	printf("hello world!\n");
	return 0;
}
$ /usr/local/musl/bin/musl-gcc hello.c
$ aarch64-linux-gnu-readelf -a a.out
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x540
  Start of program headers:          64 (bytes into file)
  Start of section headers:          6576 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         7
  Size of section headers:           64 (bytes)
  Number of section headers:         24
  Section header string table index: 23

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         00000000000001c8  000001c8
       000000000000001a  0000000000000000   A       0     0     1
  [ 2] .hash             HASH             00000000000001e8  000001e8
       000000000000003c  0000000000000004   A       4     0     8
  [ 3] .gnu.hash         GNU_HASH         0000000000000228  00000228
       0000000000000028  0000000000000000   A       4     0     8
  [ 4] .dynsym           DYNSYM           0000000000000250  00000250
       00000000000000f0  0000000000000018   A       5     3     8
  [ 5] .dynstr           STRTAB           0000000000000340  00000340
       0000000000000071  0000000000000000   A       0     0     1
  [ 6] .rela.dyn         RELA             00000000000003b8  000003b8
       00000000000000d8  0000000000000018   A       4     0     8
  [ 7] .rela.plt         RELA             0000000000000490  00000490
       0000000000000048  0000000000000018  AI       4    17     8
  [ 8] .init             PROGBITS         00000000000004d8  000004d8
       0000000000000010  0000000000000000  AX       0     0     4
  [ 9] .plt              PROGBITS         00000000000004f0  000004f0
       0000000000000050  0000000000000010  AX       0     0     16
  [10] .text             PROGBITS         0000000000000540  00000540
       0000000000000124  0000000000000000  AX       0     0     8
  [11] .fini             PROGBITS         0000000000000664  00000664
       0000000000000010  0000000000000000  AX       0     0     4
  [12] .rodata           PROGBITS         0000000000000678  00000678
       000000000000000d  0000000000000000   A       0     0     8
  [13] .eh_frame         PROGBITS         0000000000000688  00000688
       000000000000009c  0000000000000000   A       0     0     8
  [14] .init_array       INIT_ARRAY       0000000000010db8  00000db8
       0000000000000008  0000000000000008  WA       0     0     8
  [15] .fini_array       FINI_ARRAY       0000000000010dc0  00000dc0
       0000000000000008  0000000000000008  WA       0     0     8
  [16] .dynamic          DYNAMIC          0000000000010dc8  00000dc8
       00000000000001d0  0000000000000010  WA       5     0     8
  [17] .got              PROGBITS         0000000000010f98  00000f98
       0000000000000068  0000000000000008  WA       0     0     8
  [18] .data             PROGBITS         0000000000011000  00001000
       0000000000000008  0000000000000000  WA       0     0     8
  [19] .bss              NOBITS           0000000000011008  00001008
       0000000000000008  0000000000000000  WA       0     0     1
  [20] .comment          PROGBITS         0000000000000000  00001008
       000000000000002a  0000000000000001  MS       0     0     1
  [21] .symtab           SYMTAB           0000000000000000  00001038
       00000000000006f0  0000000000000018          22    55     8
  [22] .strtab           STRTAB           0000000000000000  00001728
       00000000000001d5  0000000000000000           0     0     1
  [23] .shstrtab         STRTAB           0000000000000000  000018fd
       00000000000000af  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific)

There are no section groups in this file.

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x0000000000000188 0x0000000000000188  R      0x8
  INTERP         0x00000000000001c8 0x00000000000001c8 0x00000000000001c8
                 0x000000000000001a 0x000000000000001a  R      0x1
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000724 0x0000000000000724  R E    0x10000
  LOAD           0x0000000000000db8 0x0000000000010db8 0x0000000000010db8
                 0x0000000000000250 0x0000000000000258  RW     0x10000
  DYNAMIC        0x0000000000000dc8 0x0000000000010dc8 0x0000000000010dc8
                 0x00000000000001d0 0x00000000000001d0  RW     0x8
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000000db8 0x0000000000010db8 0x0000000000010db8
                 0x0000000000000248 0x0000000000000248  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .hash .gnu.hash .dynsym .dynstr .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame
   03     .init_array .fini_array .dynamic .got .data .bss
   04     .dynamic
   05
   06     .init_array .fini_array .dynamic .got

Dynamic section at offset 0xdc8 contains 25 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libc.so]
 0x000000000000000c (INIT)               0x4d8
 0x000000000000000d (FINI)               0x664
 0x0000000000000019 (INIT_ARRAY)         0x10db8
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x10dc0
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x0000000000000004 (HASH)               0x1e8
 0x000000006ffffef5 (GNU_HASH)           0x228
 0x0000000000000005 (STRTAB)             0x340
 0x0000000000000006 (SYMTAB)             0x250
 0x000000000000000a (STRSZ)              113 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x10f98
 0x0000000000000002 (PLTRELSZ)           72 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x490
 0x0000000000000007 (RELA)               0x3b8
 0x0000000000000008 (RELASZ)             216 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000000000001e (FLAGS)              BIND_NOW
 0x000000006ffffffb (FLAGS_1)            Flags: NOW PIE
 0x000000006ffffff9 (RELACOUNT)          6
 0x0000000000000000 (NULL)               0x0

Relocation section '.rela.dyn' at offset 0x3b8 contains 9 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000010db8  000000000403 R_AARCH64_RELATIV                    640
000000010dc0  000000000403 R_AARCH64_RELATIV                    5f8
000000010fd8  000000000403 R_AARCH64_RELATIV                    4d8
000000010ff0  000000000403 R_AARCH64_RELATIV                    644
000000010ff8  000000000403 R_AARCH64_RELATIV                    664
000000011000  000000000403 R_AARCH64_RELATIV                    11000
000000010fd0  000400000401 R_AARCH64_GLOB_DA 0000000000000000 __cxa_finalize + 0
000000010fe0  000500000401 R_AARCH64_GLOB_DA 0000000000000000 _ITM_registerTMCloneTa + 0
000000010fe8  000600000401 R_AARCH64_GLOB_DA 0000000000000000 _ITM_deregisterTMClone + 0

Relocation section '.rela.plt' at offset 0x490 contains 3 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000010fb0  000300000402 R_AARCH64_JUMP_SL 0000000000000000 puts + 0
000000010fb8  000400000402 R_AARCH64_JUMP_SL 0000000000000000 __cxa_finalize + 0
000000010fc0  000700000402 R_AARCH64_JUMP_SL 0000000000000000 __libc_start_main + 0

The decoding of unwind sections for machine type AArch64 is not currently supported.

Symbol table '.dynsym' contains 10 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000000004d8     0 SECTION LOCAL  DEFAULT    8
     2: 0000000000011000     0 SECTION LOCAL  DEFAULT   18
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts
     4: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
     6: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
     7: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main
     8: 00000000000004d8     4 FUNC    GLOBAL DEFAULT    8 _init
     9: 0000000000000664     4 FUNC    GLOBAL DEFAULT   11 _fini

Symbol table '.symtab' contains 74 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000000001c8     0 SECTION LOCAL  DEFAULT    1
     2: 00000000000001e8     0 SECTION LOCAL  DEFAULT    2
     3: 0000000000000228     0 SECTION LOCAL  DEFAULT    3
     4: 0000000000000250     0 SECTION LOCAL  DEFAULT    4
     5: 0000000000000340     0 SECTION LOCAL  DEFAULT    5
     6: 00000000000003b8     0 SECTION LOCAL  DEFAULT    6
     7: 0000000000000490     0 SECTION LOCAL  DEFAULT    7
     8: 00000000000004d8     0 SECTION LOCAL  DEFAULT    8
     9: 00000000000004f0     0 SECTION LOCAL  DEFAULT    9
    10: 0000000000000540     0 SECTION LOCAL  DEFAULT   10
    11: 0000000000000664     0 SECTION LOCAL  DEFAULT   11
    12: 0000000000000678     0 SECTION LOCAL  DEFAULT   12
    13: 0000000000000688     0 SECTION LOCAL  DEFAULT   13
    14: 0000000000010db8     0 SECTION LOCAL  DEFAULT   14
    15: 0000000000010dc0     0 SECTION LOCAL  DEFAULT   15
    16: 0000000000010dc8     0 SECTION LOCAL  DEFAULT   16
    17: 0000000000010f98     0 SECTION LOCAL  DEFAULT   17
    18: 0000000000011000     0 SECTION LOCAL  DEFAULT   18
    19: 0000000000011008     0 SECTION LOCAL  DEFAULT   19
    20: 0000000000000000     0 SECTION LOCAL  DEFAULT   20
    21: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS Scrt1.c
    22: 0000000000000540     0 NOTYPE  LOCAL  DEFAULT   10 $x
    23: 000000000000055c     0 NOTYPE  LOCAL  DEFAULT   10 $x
    24: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /usr/local/musl/lib/crti.
    25: 00000000000004d8     0 NOTYPE  LOCAL  DEFAULT    8 $x
    26: 0000000000000664     0 NOTYPE  LOCAL  DEFAULT   11 $x
    27: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /usr/local/musl/lib/crtn.
    28: 00000000000004e0     0 NOTYPE  LOCAL  DEFAULT    8 $x
    29: 000000000000066c     0 NOTYPE  LOCAL  DEFAULT   11 $x
    30: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    31: 0000000000000588     0 NOTYPE  LOCAL  DEFAULT   10 $x
    32: 0000000000000588     0 FUNC    LOCAL  DEFAULT   10 deregister_tm_clones
    33: 00000000000005b8     0 FUNC    LOCAL  DEFAULT   10 register_tm_clones
    34: 0000000000011000     0 NOTYPE  LOCAL  DEFAULT   18 $d
    35: 00000000000005f8     0 FUNC    LOCAL  DEFAULT   10 __do_global_dtors_aux
    36: 0000000000011008     1 OBJECT  LOCAL  DEFAULT   19 completed.9126
    37: 0000000000010dc0     0 NOTYPE  LOCAL  DEFAULT   15 $d
    38: 0000000000010dc0     0 OBJECT  LOCAL  DEFAULT   15 __do_global_dtors_aux_fin
    39: 0000000000000640     0 FUNC    LOCAL  DEFAULT   10 frame_dummy
    40: 0000000000010db8     0 NOTYPE  LOCAL  DEFAULT   14 $d
    41: 0000000000010db8     0 OBJECT  LOCAL  DEFAULT   14 __frame_dummy_init_array_
    42: 000000000000069c     0 NOTYPE  LOCAL  DEFAULT   13 $d
    43: 0000000000011008     0 NOTYPE  LOCAL  DEFAULT   19 $d
    44: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hello.c
    45: 0000000000000678     0 NOTYPE  LOCAL  DEFAULT   12 $d
    46: 0000000000000644     0 NOTYPE  LOCAL  DEFAULT   10 $x
    47: 0000000000000700     0 NOTYPE  LOCAL  DEFAULT   13 $d
    48: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    49: 0000000000000720     0 NOTYPE  LOCAL  DEFAULT   13 $d
    50: 0000000000000720     0 OBJECT  LOCAL  DEFAULT   13 __FRAME_END__
    51: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS
    52: 0000000000010dc8     0 OBJECT  LOCAL  DEFAULT  ABS _DYNAMIC
    53: 0000000000010fc8     0 OBJECT  LOCAL  DEFAULT  ABS _GLOBAL_OFFSET_TABLE_
    54: 00000000000004f0     0 NOTYPE  LOCAL  DEFAULT    9 $x
    55: 0000000000011010     0 NOTYPE  GLOBAL DEFAULT   19 _bss_end__
    56: 0000000000011008     0 OBJECT  GLOBAL HIDDEN    18 __TMC_END__
    57: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts
    58: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize
    59: 0000000000011008     0 NOTYPE  GLOBAL DEFAULT   19 __bss_start__
    60: 0000000000011000     0 OBJECT  GLOBAL HIDDEN    18 __dso_handle
    61: 00000000000004d8     4 FUNC    GLOBAL DEFAULT    8 _init
    62: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
    63: 0000000000011010     0 NOTYPE  GLOBAL DEFAULT   19 __bss_end__
    64: 0000000000000540     0 FUNC    GLOBAL DEFAULT   10 _start
    65: 000000000000055c    40 FUNC    GLOBAL DEFAULT   10 _start_c
    66: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
    67: 0000000000011008     0 NOTYPE  GLOBAL DEFAULT   19 __bss_start
    68: 0000000000000644    32 FUNC    GLOBAL DEFAULT   10 main
    69: 0000000000011010     0 NOTYPE  GLOBAL DEFAULT   19 __end__
    70: 0000000000000664     4 FUNC    GLOBAL DEFAULT   11 _fini
    71: 0000000000011008     0 NOTYPE  GLOBAL DEFAULT   18 _edata
    72: 0000000000011010     0 NOTYPE  GLOBAL DEFAULT   19 _end
    73: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main

Histogram for bucket list length (total of 3 buckets):
 Length  Number     % of total  Coverage
      0  0          (  0.0%)
      1  2          ( 66.7%)     28.6%
      2  0          (  0.0%)     28.6%
      3  0          (  0.0%)     28.6%
      4  0          (  0.0%)     28.6%
      5  1          ( 33.3%)    100.0%

Histogram for `.gnu.hash' bucket list length (total of 2 buckets):
 Length  Number     % of total  Coverage
      0  1          ( 50.0%)
      1  0          (  0.0%)      0.0%
      2  1          ( 50.0%)    100.0%

No version information found in this file.
$ aarch64-linux-gnu-nm a.out
0000000000011010 B __bss_end__
0000000000011010 B _bss_end__
0000000000011008 B __bss_start
0000000000011008 B __bss_start__
0000000000011008 b completed.9126
                 w __cxa_finalize
0000000000000588 t deregister_tm_clones
00000000000005f8 t __do_global_dtors_aux
0000000000010dc0 d __do_global_dtors_aux_fini_array_entry
0000000000011000 D __dso_handle
0000000000010dc8 a _DYNAMIC
0000000000011008 D _edata
0000000000011010 B __end__
0000000000011010 B _end
0000000000000664 T _fini
0000000000000640 t frame_dummy
0000000000010db8 d __frame_dummy_init_array_entry
0000000000000720 r __FRAME_END__
0000000000010fc8 a _GLOBAL_OFFSET_TABLE_
00000000000004d8 T _init
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
                 U __libc_start_main
0000000000000644 T main
                 U puts
00000000000005b8 t register_tm_clones
0000000000000540 T _start
000000000000055c T _start_c
0000000000011008 D __TMC_END__
$ ls -l /lib/ld-musl-aarch64.so.1
lrwxrwxrwx 1 root root 27 Nov  9 09:19 /lib/ld-musl-aarch64.so.1 -> /usr/local/musl/lib/libc.so
$ ls -l /usr/local/musl/lib/libc.so
-rwxr-xr-x 1 root root 829296 Nov  9 09:19 /usr/local/musl/lib/libc.so
$ aarch64-linux-gnu-nm /usr/local/musl/lib/libc.so
00000000000ac4eb b a.3636
000000000003b958 T a64l
00000000000ab970 b abbrevs
00000000000ab978 b abbrevs_end
0000000000025974 T abort
00000000000ace50 b __abort_lock
...

```

# muslの使い方

[How to Use musl](https://www.musl-libc.org/how.html)の翻訳

確立されたアプリケーションバイナリのエコシステムにおいて、Cライブラリは最も交換が困難な
コンポーネントの一つです。musl は、システム全体の libc としても、堅牢なスタティックリンク
アプリケーションを作るためのツールとしても、シンプルで効率的に使用できるように設計されて
います。

musl の開発とユーザーコミュニティは、musl を使用するための 3 つの主要な方法をサポート
しています。

## 使用法 1

musl-gcc ラッパーを使えば、スタティックリンクされたプログラムや最小限のダイナミック
リンクされたプログラムを簡単にデプロイすることができます。

```bash
 ./configure && make install
$ cat > hello.c << EOF
#include <stdio.h>
int main(int argc, char **argv)
{ printf("hello %d\n", argc); }
EOF
$ musl-gcc -static -Os hello.c
$ ./a.out
hello 1
$ size ./a.out
   text    data     bss     dec     hex filename
  13302     140    1152   14594    3902 a.out
$
```

musl は、高速で非侵襲的なビルドプロセスとホスト上の既存の GCC ツールチェインを
再利用するためのラッパースクリプトにより、この使用方法をシンプルにします。

最新のソースリリースをダウンロードするか、gitリポジトリをクローンすることで
始められます。

## 使用法 2

musl ベースの Linux ディストリビューションやミニディストリビューションのシステム
ワイドな libc として使用されます。これらはほとんどの場合、Linux From Scratchの原則に
基づいたソースベースのビルドシステムに、新しいパッケージ管理ツールを追加した形に
なっています。

            Core: Muslおよびコンパイラツールチェイン、通常はホストシステム上に構築されます。

        Base: 基本的なパッケージで、クロスコンパイラや musl-gccラッパーを使ってホストシステム上でビルド
        される場合もあれば、chrootターゲットや仮想システム内でビルドされる場合もあります。

    Extras: ターゲットとなるベースシステムが動作可能になった時点で、そのシステム上でビルド可能な追加プログラムのセレクション。

muslベースのディストリビューションに関する情報は、musl community wikiを参照してください。

## 使用法 3

独自のシステムを構築するためのベースとして。プロジェクトの可能性は以下の通りです。

- マルチメディア機器
- ルーター／ファイアウォール
- VoIPデバイス
- レスキューディスク
- 携帯電話
- キオスク
- ライトデスクトップシステム
-
musl-gccラッパースクリプト、開発環境を備えた完全なmuslベースのホストシステム、または、
muslクロスコンパイラを使用して、muslベースの新しいシステムをブートストラップすることが
できます。
