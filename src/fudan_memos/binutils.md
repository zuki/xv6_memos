# binutilsのインストール

```
$ wget http://ftp.gnu.org/gnu/binutils/binutils-2.36.tar.xz
# tar Jxf binutils-2.37.tar.xz
$ cd binutils-2.37
$ ./configure -host=aarch64-linux-gnu --disable-nls --prefix=$(pwd)/bin CFLAGS="-specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096"
$ make configure-host
$ make LDFLAGS="-all-static"
...
make[4]: Entering directory '/home/vagrant/lfspkgs/binutils-2.37/ld'
/bin/bash ./libtool  --tag=CC   --mode=compile aarch64-linux-gnu-gcc -DHAVE_CONFIG_H -I.  -I. -I. -I../bfd -I./../bfd -I./../include -I./../zlib  -specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096     -DLOCALEDIR="\"/home/vagrant/lfspkgs/binutils-2.37/bin/share/locale\""  -W -Wall -Wstrict-prototypes -Wmissing-prototypes -Wshadow -Wstack-usage=262144 -DELF_LIST_OPTIONS=true -DELF_SHLIB_LIST_OPTIONS=true -DELF_PLT_UNWIND_LIST_OPTIONS=false -specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096     -MT libdep_plugin.lo -MD -MP -MF .deps/libdep_plugin.Tpo -c -o libdep_plugin.lo libdep_plugin.c
libtool: compile:  aarch64-linux-gnu-gcc -DHAVE_CONFIG_H -I. -I. -I. -I../bfd -I./../bfd -I./../include -I./../zlib -specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -fno-plt -fno-pic -z max-page-size=4096 -DLOCALEDIR=\"/home/vagrant/lfspkgs/binutils-2.37/bin/share/locale\" -W -Wall -Wstrict-prototypes -Wmissing-prototypes -Wshadow -Wstack-usage=262144 -DELF_LIST_OPTIONS=true -DELF_SHLIB_LIST_OPTIONS=true -DELF_PLT_UNWIND_LIST_OPTIONS=false -specs /usr/local/musl/lib/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -fno-plt -fno-pic -z max-page-size=4096 -MT libdep_plugin.lo -MD -MP -MF .deps/libdep_plugin.Tpo -c libdep_plugin.c -fpie -fpie -o libdep_plugin.o
aarch64-linux-gnu-gcc: fatal error: /usr/local/musl/lib/musl-gcc.specs: attempt to rename spec ‘cpp_options’ to already defined spec ‘old_cpp_options’
compilation terminated.
make[4]: *** [Makefile:1582: libdep_plugin.lo] Error 1
make[4]: Leaving directory '/home/vagrant/lfspkgs/binutils-2.37/ld'
make[3]: *** [Makefile:1814: install-recursive] Error 1
make[3]: Leaving directory '/home/vagrant/lfspkgs/binutils-2.37/ld'
make[2]: *** [Makefile:1956: install] Error 2
make[2]: Leaving directory '/home/vagrant/lfspkgs/binutils-2.37/ld'
make[1]: *** [Makefile:7331: install-ld] Error 2
make[1]: Leaving directory '/home/vagrant/lfspkgs/binutils-2.37'
make: *** [Makefile:2339: install] Error 2
```

## やり直し

```
$ rm -rf binutils-2.37
# tar Jxf binutils-2.37.tar.xz
$ cd binutils-2.37
$ CC=/usr/local/musl/bin/musl-gcc ./configure -host=aarch64-linux-gnu --disable-nls --prefix=$(pwd)/bin
$ make configure-host
$ make LDFLAGS="-all-static"
$ make install
$ ls bin
aarch64-linux-gnu  bin  include  lib  share
$ ls bin/bin
addr2line  as       elfedit  ld      nm       objdump  readelf  strings
ar         c++filt  gprof    ld.bfd  objcopy  ranlib   size     strip
$ ls bin/lib
bfd-plugins  libbfd.la  libctf.la       libctf-nobfd.la  libopcodes.la
libbfd.a     libctf.a   libctf-nobfd.a  libopcodes.a
$ ls bin/include/
ansidecl.h  bfdlink.h  ctf.h          dis-asm.h     symcat.h
bfd.h       ctf-api.h  diagnostics.h  plugin-api.h
$ ls bin/share
info  man
$ ls bin/share/info
as.info  bfd.info  binutils.info  dir  gprof.info  ld.info
$ ls bin/share/man
man1
$ ls bin/share/man/man1
addr2line.1  c++filt.1  gprof.1  objcopy.1  readelf.1  strip.1
ar.1         dlltool.1  ld.1     objdump.1  size.1     windmc.1
as.1         elfedit.1  nm.1     ranlib.1   strings.1  windres.1
$ ls bin/aarch64-linux-gnu/
bin  lib
$ ls bin/aarch64-linux-gnu/bin
ar  as  ld  ld.bfd  nm  objcopy  objdump  ranlib  readelf  strip
$ ls bin/aarch64-linux-gnu/lib
ldscripts
$ file bin/bin/readelf
bin/bin/readelf: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, with debug_info, not stripped
$ aarch64-linux-gnu-readelf -h bin/bin/readelf
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x400b60
  Start of program headers:          64 (bytes into file)
  Start of section headers:          4620872 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         4
  Size of section headers:           64 (bytes)
  Number of section headers:         24
  Section header string table index: 23
```

# xv6へ取り込み

```
$ mkdir -p XV6/user/local/bin
$ cp -r bin/bin/* XV6/user/local/bin/
$ vi XV6/user/src/mkfs/main.c
$ vi xv6/mksd.mk
```

# 実行結果

```
# ls
bin  dev  etc  home  lib  usr
# ls /usr/local/bin
as  gprof  nm  objcopy	objdump  readelf  size	strings

# /usr/local/bin/readelf -h /bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x401cec
  Start of program headers:          64 (bytes into file)
  Start of section headers:          316944 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         4
  Size of section headers:           64 (bytes)
  Number of section headers:         17
  Section header string table index: 16
```
