# カレントブランチからdynamicブランチに切り替え

```
$ git stash
$ git checkout dynamic
$ rm -rf obj
$ cd coreutils-8.32
$ make clean
$ make
$ cd ..
$ make
$ make qemu
```

# dynamicブランチからカレントブランチに切り替え

```
$ git checkout current_branch
$ git stash pop
$ rm -rf obj
$ cd coreutils-8.32
$ CC=/usr/local/musl/bin/musl-gcc ./configure --host=aarch64-linux-gnu CFLAGS="-std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096"
$ make
$ cd ../dash
CC=/usr/local/musl/bin/musl-gcc ./configure --host=aarch64-linux-gnu --enable-static CFLAGS="-std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096 -Isrc"
$ make
$ cd ..
$ make
$ make qemu
```

## カレントブランチに戻った時にcoreutilsとdashをconfigureし直さないと次のエラーが発生

- coreutils

```
$ make    // xv6のmake
...
coreutils-8.32/src/ls.c:38:10: fatal error: config.h: No such file or directory
   38 | #include <config.h>
      |          ^~~~~~~~~~
compilation terminated.
make[1]: *** [<builtin>: coreutils-8.32/src/ls.o] Error 1
make[1]: Leaving directory '/home/vagrant/xv6-fudan'
make: *** [Makefile:67: all] Error 2
vagrant@ubuntu-bionic:~/xv6-fudan$ cd coreutils-8.32/
vagrant@ubuntu-bionic:~/xv6-fudan/coreutils-8.32$ make clean
/bin/bash ./config.status --recheck
/bin/bash: ./config.status: No such file or directory
make: *** [Makefile:7059: config.status] Error 127
```

- dash

```
$ make    // xv6のmake
...
make[1]: *** No rule to make target 'dash/src/dash', needed by 'obj/fs.img'.  Stop.
make[1]: Leaving directory '/home/vagrant/xv6-fudan'
make: *** [Makefile:67: all] Error 2
vagrant@ubuntu-bionic:~/xv6-fudan$ cd dash
vagrant@ubuntu-bionic:~/xv6-fudan/dash$ ls
aclocal.m4      compile      configure     DASHCONFIGURED  Makefile.am  musl-gcc.specs
autom4te.cache  config.h     configure.ac  depcomp         Makefile.in  src
ChangeLog       config.h.in  COPYING       install-sh      missing      stamp-h1
vagrant@ubuntu-bionic:~/xv6-fudan/dash$ make clean
make: *** No rule to make target 'clean'.  Stop.
vagrant@ubuntu-bionic:~/xv6-fudan/dash$ make
make: *** No targets specified and no makefile found.  Stop.
```
