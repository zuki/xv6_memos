# shadowの導入

```
$ cd xv6-fudan
$ wget https://github.com/shadow-maint/shadow/releases/download/v4.9/shadow-4.9.tar.xz
$ tar Jxv shadow-4.9.tar.xz
$ cd shadow-4.9
$ sed -e 's:#ENCRYPT_METHOD DES:ENCRYPT_METHOD SHA512:' -i etc/login.defs       # LFS 11.0 による
$ sed -e "224s/rounds/min_rounds/" -i libmisc/salt.c
$ copy ../obj/user/musl-gcc.specs .
$ vi musl-gcc.specs
$ ./configure --host=aarch64-linux-gnu --sysconfdir=/etc --with-group-name-max-length=32 \
--enable-static --disable-nls --disable-shared CFLAGS="-specs /home/vagrant/xv6-fudan/shadow-4.9/musl-gcc.specs \
  -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096"

shadow will be compiled with the following features:

	auditing support:		no
	CrackLib support:		no
	PAM support:			no
	SELinux support:		no
	BtrFS support:			maybe
	ACL support:			no
	Extended Attributes support:	no
	tcb support (incomplete):	no
	shadow group support:		yes
	S/Key support:			no
	SHA passwords encryption:	yes
	bcrypt passwords encryption:	no
	yescrypt passwords encryption:	no
	nscd support:			yes
	sssd support:			yes
	subordinate IDs support:	yes
	use file caps:			no
	install su:			yes

$ make
$ find src -executable -type f
src/free_subid_range
src/groupadd
src/su
src/pwunconv
src/chgpasswd
src/groupdel
src/sulogin
src/lastlog
src/grpconv
src/check_subid_range
src/chage
src/pwconv
src/nologin
src/passwd
src/vipw
src/grpunconv
src/pwck
src/login
src/expiry
src/get_subid_owners
src/chsh
src/groupmems
src/logoutd
src/new_subid_range
src/faillog
src/chfn
src/newuidmap
src/id
src/newgrp
src/newgidmap
src/userdel
src/groups
src/groupmod
src/list_subid_ranges
src/useradd
src/usermod
src/chpasswd
src/gpasswd
src/newusers
src/grpck

$ file src/login							// dynamic linkされてる
login: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, not stripped
```

## 手作業でリンク

- staticリンクされたコマンドができた

```
$ aarch64-linux-gnu-ld ../../libc/lib/Scrt1.o ../../libc/lib/crti.o /usr/lib/gcc-cross/aarch64-linux-gnu/9/crtbeginS.o vipw.o ../libmisc/.libs/libmisc.a ../lib/.libs/libshadow.a ../../libc/lib/libc.a /usr/lib/gcc-cross/aarch64-linux-gnu/9/libgcc.a /usr/lib/gcc-cross/aarch64-linux-gnu/9/crtendS.o ../../libc/lib/crtn.o -o vipw
$ file vipw
vipw: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, not stripped
```

## Makefileを調査

### 1. とりあえず`chage`コマンドのリンク部分を読む

```
chage$(EXEEXT): $(chage_OBJECTS) $(chage_DEPENDENCIES) $(EXTRA_chage_DEPENDENCIES)
	@rm -f chage$(EXEEXT)
	$(AM_V_CCLD)$(LINK) $(chage_OBJECTS) $(chage_LDADD) $(LIBS)
```

### 2. 変数を展開

```
EXEEXT =
chage_OBJECTS = chage.$(OBJEXT)
OBJEXT = o

chage_DEPENDENCIES = $(am__DEPENDENCIES_2) $(am__DEPENDENCIES_3) \
	$(am__DEPENDENCIES_1) $(am__DEPENDENCIES_1) \
	$(am__DEPENDENCIES_1)
am__DEPENDENCIES_1 =
am__DEPENDENCIES_2 = $(am__DEPENDENCIES_1) \
	$(top_builddir)/libmisc/libmisc.la \
	$(top_builddir)/lib/libshadow.la $(am__DEPENDENCIES_1)
am__DEPENDENCIES_3 =

EXTRA_chage_DEPENDENCIES =

AM_V_CCLD = $(am__v_CCLD_$(V))
V =
am__v_CCLD_ = $(am__v_CCLD_$(AM_DEFAULT_VERBOSITY))
AM_DEFAULT_VERBOSITY = 0
am__v_CCLD_0 = @echo "  CCLD    " $@;

LINK = $(LIBTOOL) $(AM_V_lt) --tag=CC $(AM_LIBTOOLFLAGS) \
	$(LIBTOOLFLAGS) --mode=link $(CCLD) $(AM_CFLAGS) $(CFLAGS) \
	$(AM_LDFLAGS) $(LDFLAGS) -o $@
LIBTOOL = /bin/bash ../libtool
AM_V_lt = $(am__v_lt_$(V))
am__v_lt_ = $(am__v_lt_$(AM_DEFAULT_VERBOSITY))
am__v_lt_0 = --silent

AM_LIBTOOLFLAGS =
LIBTOOLFLAGS =
CCLD = $(CC)
AM_CFLAGS =
AM_LDFLAGS =
LDFLAGS =

chage_LDADD = $(LDADD) $(LIBPAM_SUID) $(LIBAUDIT) $(LIBSELINUX) $(LIBECONF)
LDADD = ../libmisc/libmisc.la ../lib/libshadow.la
INTLLIBS =
LIBTCB =
LIBPAM_SUID =
LIBAUDIT =
LIBSELINUX =
LIBECONF =
LIBS =
```

### 3. 展開結果

```
chage: chage.o ../libmisc/libmisc.la ../lib/libshadow.la
	@rm -f change
	@echo "  CCLD    " $@;/bin/bash ../libtool --silent --mode=link aarch64-linux-gnu-gcc -specs /home/vagrant/xv6-fudan/obj/user/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096 -Ilib -I libmisc -I/home/vagrant/xv6-fudan/libc/obj/include/ -I/home/vagrant/xv6-fudan/libc/arch/aarch64/ -I/home/vagrant/xv6-fudan/libc/arch/generic/ -I/home/vagrant/xv6-fudan/libc/include -L/home/vagrant/xv6-fudan/libc/lib -static -Wl,--fatal-warnings -o $@ change.o ../libmisc/libmisc.la ../lib/libshadow.la
```

## libtoolを調査

- linkモードで`-all-static`を指定すれば良さそう
- `src/Makefile`で空白だった`AM_LDFLAGS`に設定
- `chage`コマンドをリンクしてstaticリンクされていることを確認した

```
$ vi musl-gcc.specs             // インクルードファイルを追加し、ファイルはすべてフルパスにする
$ cd shadow-4.9/src
$ vi Makefile					// AM_LDFLAGS = -all-staticを追加
$ V=1 make chage				// chageだけリンクしてみる。V=1とするとverboseモードになって出力が増える
/bin/bash ../libtool  --tag=CC   --mode=link aarch64-linux-gnu-gcc  -specs /home/vagrant/xv6-fudan/shadow-4.9/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096 -all-static -L/home/vagrant/xv6-fudan/libc/lib -static -Wl,--fatal-warnings -o chage chage.o  ../libmisc/libmisc.la ../lib/libshadow.la
libtool: link: aarch64-linux-gnu-gcc -specs /home/vagrant/xv6-fudan/shadow-4.9/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -fno-plt -fno-pic -fpie -z max-page-size=4096 -static -Wl,--fatal-warnings -o chage chage.o  -L/home/vagrant/xv6-fudan/libc/lib ../libmisc/.libs/libmisc.a ../lib/.libs/libshadow.a
$ file src/chage
src/chage: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, not stripped
```

## 全コマンドを再作成

```
$ make clean-binPROGRAMS		// コンパイル済みのコマンドを削除
 rm -f login su					// /binにインストールされるコマンド
$ make clean-sbinPROGRAMS		// /sbinにインストールされるコマンド
 rm -f nologin
$ make clean-ubinPROGRAMS		// /usr/binにインストールされるコマンド
 rm -f faillog lastlog chage chfn chsh expiry gpasswd newgrp passwd newgidmap newuidmap
$ make clean-usbinPROGRAMS		// /usr/sbinにインストールされるコマンド
 rm -f chgpasswd chpasswd groupadd groupdel groupmems groupmod grpck \
       grpconv grpunconv logoutd newusers pwck pwconv pwunconv useradd userdel usermod vipw
$ make clean-noinstPROGRAMS		// インストールされないコマンド
 rm -f groups id sulogin list_subid_ranges get_subid_owners new_subid_range free_subid_range check_subid_range
$ cd ..
$ make   						// リンクだけ実行される
$ file src/vipw
src/vipw: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, not stripped
$ file src/login
src/login: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, not stripped
```
