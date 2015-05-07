# To build, install the Android NDK on GNU/Linux or Mac OS X:
#    http://developer.android.com/sdk/ndk/index.html

### these modify the calling shell
# point pkg-config to the .pc files generated from these builds
export PKG_CONFIG_PATH=$(LOCAL)/lib/pkgconfig
# workaround for cross-compiling bug in autoconf
export ac_cv_func_malloc_0_nonnull=yes

CWD = $(shell pwd)
PROJECT_ROOT = $(CWD)
EXTERNAL_ROOT = $(PROJECT_ROOT)/external

# Android now has 64-bit and 32-bit versions of the NDK for GNU/Linux.  We
# assume that the build platform uses the appropriate version, otherwise the
# user building this will have to manually set NDK_PROCESSOR or NDK_TOOLCHAIN.
CPU := $(shell uname -m)
ifeq ($(CPU),x86_64)
  NDK_PROCESSOR=x86_64
else
  NDK_PROCESSOR=x86
endif

# Android NDK setup
NDK_BASE ?=  /usr/local/android-ndk
NDK_PLATFORM_LEVEL ?= 21
NDK_TOOLCHAIN_VERSION ?= 4.8
APP_ABI ?= armeabi

# PIEFLAGS for SDK 16/Android L must be set to -fPIE -pie, but can override for earlier targets 
PIEFLAGS ?= -fPIE -pie

NDK_ABI := $(APP_ABI)
ifneq ($(filter arm%, $(APP_ABI)),)
  NDK_ABI := arm
endif
ifneq ($(filter arm64%, $(APP_ABI)),)
 NDK_ABI := arm64
endif
NDK_SYSROOT=$(NDK_BASE)/platforms/android-$(NDK_PLATFORM_LEVEL)/arch-$(NDK_ABI)
NDK_UNAME := $(shell uname -s | tr '[A-Z]' '[a-z]')
ifneq ($(filter mips%, $(NDK_ABI)),)
 HOST := $(NDK_ABI)el-linux-android
endif
ifneq ($(filter arm64, $(NDK_ABI)),)
 HOST := aarch64-linux-android
endif
ifneq ($(filter arm, $(NDK_ABI)),)
 HOST := arm-linux-androideabi
endif
ifneq ($(filter x86, $(NDK_ABI)),)
 HOST := i686-linux-android
endif
ifneq ($(filter x86_64, $(NDK_ABI)),)
 HOST := x86_64-linux-android
endif

NDK_TOOLCHAIN := $(HOST)-$(NDK_TOOLCHAIN_VERSION)
ifneq ($(filter x86%, $(NDK_ABI)),)
 NDK_TOOLCHAIN := $(NDK_ABI)-$(NDK_TOOLCHAIN_VERSION)
endif
NDK_TOOLCHAIN_BASE=$(NDK_BASE)/toolchains/$(NDK_TOOLCHAIN)/prebuilt/$(NDK_UNAME)-$(NDK_PROCESSOR)

# include Android's build flags
include $(NDK_BASE)/toolchains/$(NDK_TOOLCHAIN)/setup.mk

# install root for built files
DESTDIR = $(CWD)
# TODO try adding the Android-style /data/app.name here
prefix = 
LOCAL := $(DESTDIR)$(prefix)

PATH := ${PATH}:$(NDK_TOOLCHAIN_BASE)/bin:$(LOCAL)/bin

CC := $(NDK_TOOLCHAIN_BASE)/bin/$(HOST)-gcc --sysroot=$(NDK_SYSROOT)
CXX := $(NDK_TOOLCHAIN_BASE)/bin/$(HOST)-g++ --sysroot=$(NDK_SYSROOT)
CPP := $(NDK_TOOLCHAIN_BASE)/bin/$(HOST)-cpp --sysroot=$(NDK_SYSROOT)
LD := $(NDK_TOOLCHAIN_BASE)/bin/$(HOST)-ld
AR := $(NDK_TOOLCHAIN_BASE)/bin/$(HOST)-ar
RANLIB := $(NDK_TOOLCHAIN_BASE)/bin/$(HOST)-ranlib
STRIP := $(NDK_TOOLCHAIN_BASE)/bin/$(HOST)-strip \
	--strip-unneeded -R .note -R .comment --strip-debug

CFLAGS = -I$(LOCAL)/include $(TARGET_CFLAGS)
LDFLAGS = -L$(LOCAL)/lib -Wl,--rpath,$(LOCAL)/lib $(TARGET_LDFLAGS)
#ALL_LDFLAGS = -Wl,--entry=main,-rpath=$(ANDROID_NDK_ROOT)/build/platforms/android-$(NDK_PLATFORM_VER)/arch-arm/usr/lib,-dynamic-linker=/system/bin/linker -L$(NDK_SYSROOT)/usr/lib  -nostdlib -lc -ldl
#LIBS = -lc -ldl


#------------------------------------------------------------------------------#
# download sources and verify


TARBALLDIRS = popt-1.16
SOURCEDIRS = libgpg-error libgcrypt cryptsetup lvm2 util-linux

all: cryptsetup-install


#------------------------------------------------------------------------------#
# libgpg-error

libgpg-error/config.sub: config.sub
	cp config.sub libgpg-error/config.sub

libgpg-error/config.guess: config.guess
	cp config.guess libgpg-error/config.guess

libgpg-error/Makefile: libgpg-error/config.sub libgpg-error/config.guess
	cd libgpg-error && \
		./autogen.sh && automake --add-missing && \
		./configure CC="$(CC)" CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			--disable-languages --disable-doc --enable-static --host=$(HOST) --prefix=$(prefix)

libgpg-error/src/.libs/libgpg-error.a: libgpg-error/Makefile
	$(MAKE) -C libgpg-error

libgpg-error-build: libgpg-error/src/.libs/libgpg-error.a

$(LOCAL)/lib/libgpg-error.a: libgpg-error/src/.libs/libgpg-error.a
	$(MAKE) -C libgpg-error DESTDIR=$(DESTDIR) prefix= install

libgpg-error-install: $(LOCAL)/lib/libgpg-error.a

#------------------------------------------------------------------------------#
# 

libgcrypt/config.sub: config.sub
	cp config.sub libgcrypt/config.sub

libgcrypt/config.guess: config.guess
	cp config.guess libgcrypt/config.guess

libgcrypt/Makefile: libgcrypt/config.sub libgcrypt/config.guess
	cd libgcrypt && \
		autoreconf -i && \
		CC="$(CC)" CFLAGS="$(CFLAGS)" LDFLAGS="-lgpg-error $(LDFLAGS)" \
			./configure --enable-maintainer-mode --enable-static --host=$(HOST) --with-gpg-error-prefix=$(LOCAL) --prefix=$(prefix)

libgcrypt/src/.libs/libgcrypt.a: $(LOCAL)/lib/libgpg-error.a libgcrypt/Makefile
	$(MAKE) -C libgcrypt

libgcrypt-build: libgcrypt/src/.libs/libgcrypt.a

$(LOCAL)/lib/libgcrypt.a: libgcrypt/src/.libs/libgcrypt.a
	$(MAKE) -C libgcrypt DESTDIR=$(DESTDIR) prefix= install

libgcrypt-install: $(LOCAL)/lib/libgcrypt.a

#------------------------------------------------------------------------------#
# 

# TODObuild libuuid from e2fsprogs

#------------------------------------------------------------------------------#
# LVM2

lvm2/Makefile:
	cp config.sub lvm2/autoconf/config.sub
	cp config.guess lvm2/autoconf/config.guess
	-cd lvm2 && patch -N -p1 --reject-file=- < ../lvm2-android.diff
	cd lvm2 && \
		autoreconf && \
		CC="$(CC)" CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" ./configure --host=$(HOST) --prefix=$(prefix) --enable-static_link --enable-lvm1_fallback --enable-fsadm --with-clvmd=cman --with-cluster=internal --with-pool=internal --with-user= --with-group= --with-dmdir=device-mapper.0 --with-device-uid=0 --with-device-gid=6 --with-device-mode=0660 --with-clvmd=none --with-pool=none --with-cluster=none --with-snapshots=none --with-mirrors=none --enable-pkgconfig

lvm2/libdm/ioctl/libdevmapper.a: lvm2/Makefile
	$(MAKE) CC="$(CC)" CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" -C lvm2

lvm2-build: lvm2/libdm/ioctl/libdevmapper.a

$(LOCAL)/lib/libdevmapper.a: lvm2/libdm/ioctl/libdevmapper.a
	$(MAKE) -C lvm2 DESTDIR=$(DESTDIR) prefix= install

lvm2-install: $(LOCAL)/lib/libdevmapper.a


#------------------------------------------------------------------------------#
# popt

popt-1.16.tar.gz.asc: popt-1.16.tar.gz
	touch $@
# TODO verify

popt-1.16.tar.gz:
	wget -c http://rpm5.org/files/popt/popt-1.16.tar.gz
	sha1sum -c popt-1.16.tar.gz.sha1

popt-1.16: popt-1.16.tar.gz.asc
	tar xzf popt-1.16.tar.gz
	patch -p0 < popt-1.16-android.diff

popt-1.16/config.sub: popt-1.16 config.sub
	cp config.sub popt-1.16/config.sub

popt-1.16/config.guess: popt-1.16 config.guess
	cp config.guess popt-1.16/config.guess

popt-1.16/.libs/libpopt.a: popt-1.16 popt-1.16/config.sub popt-1.16/config.guess
	cd popt-1.16 && \
		autoreconf -i && \
		CC="$(CC)" CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure --prefix=$(prefix) --host=$(HOST) --enable-static
	$(MAKE) -C popt-1.16

popt-build: popt-1.16/.libs/libpopt.a

$(LOCAL)/lib/libpopt.a: popt-1.16/.libs/libpopt.a
	$(MAKE) -C popt-1.16 DESTDIR=$(DESTDIR) prefix= install

popt-install: $(LOCAL)/lib/libpopt.a


#------------------------------------------------------------------------------#
# util-linux

util-linux/config/config.sub: config.sub
	cp config.sub util-linux/config/config.sub

util-linux/config/config.guess: config.guess
	cp config.guess util-linux/config/config.guess

util-linux/config/config.rpath:
	touch $@

util-linux/configure: util-linux/configure.ac util-linux/config/config.guess util-linux/config/config.sub util-linux/config/config.rpath
	-cd util-linux && patch -p1 -N --reject-file=- < ../util-linux-android.diff
	cd util-linux && ./autogen.sh

util-linux/Makefile: util-linux/configure
	cd util-linux && \
		CC="$(CC)" CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure --host=$(HOST) --prefix=$(prefix) --enable-static --without-ncurses --disable-all-programs --enable-libuuid --enable-pkgconfig --without-python

util-linux/shlibs/uuid/src/.libs/libuuid.a: util-linux/Makefile
	$(MAKE) -C util-linux

util-linux-build: util-linux/shlibs/uuid/src/.libs/libuuid.a

$(LOCAL)/lib/libuuid.a: util-linux/shlibs/uuid/src/.libs/libuuid.a
	$(MAKE) -C util-linux DESTDIR=$(DESTDIR) prefix= install

util-linux-install: $(LOCAL)/lib/libuuid.a


#------------------------------------------------------------------------------#
# cryptsetup

cryptsetup/config.sub: config.sub
	cp config.sub cryptsetup/config.sub

cryptsetup/config.guess: config.guess
	cp config.guess cryptsetup/config.guess

cryptsetup/Makefile: cryptsetup/config.sub cryptsetup/config.guess
	-patch -N -p0 --reject-file=- < cryptsetup-android.diff
	rm -f lib/*.la
	cd cryptsetup && \
	  autoreconf -i && \
		CC="$(CC)" CFLAGS="$(CFLAGS) $(PIEFLAGS)" LDFLAGS="$(LDFLAGS) $(PIEFLAGS)" \
			./configure --host=$(HOST) --prefix=$(prefix) --enable-static --disable-selinux --enable-static-cryptsetup

cryptsetup/src/cryptsetup.static: $(LOCAL)/lib/libgpg-error.a $(LOCAL)/lib/libgcrypt.a $(LOCAL)/lib/libdevmapper.a $(LOCAL)/lib/libpopt.a $(LOCAL)/lib/libuuid.a cryptsetup/Makefile
	$(MAKE) -C cryptsetup

cryptsetup-build: cryptsetup/src/cryptsetup.static

$(LOCAL)/sbin/cryptsetup: cryptsetup/src/cryptsetup.static
	$(MAKE) -C cryptsetup DESTDIR=$(DESTDIR) prefix= install

cryptsetup-install: $(LOCAL)/sbin/cryptsetup

#------------------------------------------------------------------------------#
# clean

%-clean:
	cd $* && git reset --hard && git clean -dfx

tarballs_clean:
	rm -f -- rm *.sig *.bz2 *.gz *.tgz *.asc

install_clean:
	rm -rf -- bin include lib sbin share usr etc

sourcedirs_clean: libgpg-error-clean libgcrypt-clean lvm2-clean util-linux-clean
	rm -rf -- $(TARBALLDIRS)

clean: sourcedirs_clean install_clean

distclean: clean tarballs_clean

.PHONY = libgpg-error-build libgcrypt-build lvm2-build popt-build	\
	cryptosetup-build util-linux-build libgpg-error-install		\
	libgcrypt-install lvm2-install popt-install cryptosetup-install	\
	util-linux-ng-install clean tarballs_clean sourcedirs_clean
