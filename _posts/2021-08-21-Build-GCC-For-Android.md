---
layout: post
title: 编译arm64-v8a的GCC不成功的记录
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

在尝试编译Clang过程中，也尝试了GCC的交叉编译，不过没有成功。不是编译GCC报错，是编译出的GCC没办法编译出的能在Android平台上运行的程序，提示需要链接Scrt1.o/crti.o，但是这些是glibc库提供的可执行启动运行时，正常是需要链接ndk里提供的crtandroid.o，我个人猜测是bionic没有配置，这个问题并没有解决.

<!--more-->

## 要求
交叉编译的GCC代码版本必须>=8，原因是之前的版本make脚本交叉编译时有问题（本来GCC5的时候就有人在社区提了patch，不知道为什么直到几年后的8.0大版本才合并进去）。  
[https://gcc.gnu.org/bugzilla/show_bug.cgi?id=82371](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=82371)


## 独立工具链
测试使用的16e的ndk，因为17以后版本就不带GCC的工具链了，心想GCC源码还是用GCC编译比较好。
```
build/tools/make-standalone-toolchain.sh --toolchain=aarch64-linux-android-4.9 --platform=android-21 --install-dir=/home/yu/Desktop/android-21-arm64v8a
```


## 编译过程
编译GCC一定需要GMP、MPFR、MPC，因此这三个库需要提前编译
```
# GMP
export NDK=/home/yu/Desktop/android-ndk-r16b
export TARGET=aarch64-linux-android
export TOOLCHAIN=/home/yu/Desktop/android-21-arm64v8a
export SYSROOT=$TOOLCHAIN/sysroot
export API=21
export AR=$TOOLCHAIN/bin/$TARGET-ar
export CC=$TOOLCHAIN/bin/$TARGET-gcc
export AS=$TOOLCHAIN/bin/$TARGET-as
export CXX=$TOOLCHAIN/bin/$TARGET-g++
export LD=$TOOLCHAIN/bin/$TARGET-ld
export RANLIB=$TOOLCHAIN/bin/$TARGET-ranlib
export STRIP=$TOOLCHAIN/bin/$TARGET-strip
export CFLAGS=-D__ANDROID_API__=$API

./configure --host=$TARGET --disable-shared --enable-static --prefix=/home/yu/Desktop/gcc_request
make -j4


# MPFR
export NDK=/home/yu/Desktop/android-ndk-r16b
export TARGET=aarch64-linux-android
export TOOLCHAIN=/home/yu/Desktop/android-21-arm64v8a
export SYSROOT=$TOOLCHAIN/sysroot
export API=21
export AR=$TOOLCHAIN/bin/$TARGET-ar
export CC=$TOOLCHAIN/bin/$TARGET-gcc
export AS=$TOOLCHAIN/bin/$TARGET-as
export CXX=$TOOLCHAIN/bin/$TARGET-g++
export LD=$TOOLCHAIN/bin/$TARGET-ld
export RANLIB=$TOOLCHAIN/bin/$TARGET-ranlib
export STRIP=$TOOLCHAIN/bin/$TARGET-strip
export CFLAGS=-D__ANDROID_API__=$API

./configure --host=$TARGET --disable-shared --enable-static --with-gmp=/home/yu/Desktop/gcc_request --prefix=/home/yu/Desktop/gcc_request
make -j4


# MPC
export NDK=/home/yu/Desktop/android-ndk-r16b
export TARGET=aarch64-linux-android
export TOOLCHAIN=/home/yu/Desktop/android-21-arm64v8a
export SYSROOT=$TOOLCHAIN/sysroot
export API=21
export AR=$TOOLCHAIN/bin/$TARGET-ar
export CC=$TOOLCHAIN/bin/$TARGET-gcc
export AS=$TOOLCHAIN/bin/$TARGET-as
export CXX=$TOOLCHAIN/bin/$TARGET-g++
export LD=$TOOLCHAIN/bin/$TARGET-ld
export RANLIB=$TOOLCHAIN/bin/$TARGET-ranlib
export STRIP=$TOOLCHAIN/bin/$TARGET-strip
export CFLAGS=-D__ANDROID_API__=$API

./configure --host=$TARGET --disable-shared --enable-static --with-gmp=/home/yu/Desktop/gcc_request --with-mpfr=/home/yu/Desktop/gcc_request --prefix=/home/yu/Desktop/gcc_request
make -j4


# GCC
export PATH=/home/yu/Desktop/android-21-arm64v8a/bin:$PATH
export CFLAGS="-D__ANDROID_API__=21 -fPIE -pie -I/home/yu/Desktop/android-21-arm64v8a/sysroot/usr/include"
export CXXFLAGS="-D__ANDROID_API__=21 -fPIE -pie -I/home/yu/Desktop/android-21-arm64v8a/sysroot/usr/include"
export LDFLAGS="-fPIE -pie -L/home/yu/Desktop/android-21-arm64v8a/sysroot/usr/lib"

./configure --host=aarch64-linux-android \
--enable-static \
--disable-shared \
--enable-languages=c,c++ \
--enable-initfini-array \
--enable-target-optspace \
--disable-threads \
--disable-lto \
--disable-libquadmath \
--disable-multilib \
--disable-libgomp \
--disable-libmudflap \
--disable-libatomic \
--disable-tls \
--disable-nls \
--disable-sjlj-exceptions \
--disable-libstdc++-v3 \
--disable-libsanitizer \
--disable-plugins \
--disable-libgcc \
--disable-libssp \
--disable-docs \
--disable-libitm \
--with-gnu-ld \
--with-gnu-as \
--with-mpc=/home/yu/Desktop/gcc_request \
--with-mpfr=/home/yu/Desktop/gcc_request \
--with-gmp=/home/yu/Desktop/gcc_request \
--prefix=/home/yu/Desktop/gcc_output

make -j4
```

# 未解决的问题
编译出的GCC在Android上不可以用，我认为问题出在GCC编译的配置上，问题的表现如下：  
```
ld: cannot find Scrt1.o: No such file or directory
ld: cannot find crti.o: No such file or directory
ld: cannot find crtbeginS.o: No such file or directory
ld: cannot find -lgcc
ld: cannot find -lgcc
ld: cannot find crtendS.o: No such file or directory
ld: cannot find crtn.o: No such file or directory
```

我对比了人家可用的GCC编译的链接过程，7.2.0是可用的，8.5.0为我们自己编译的。其中人家链接了crtend_android.o，而我们链接了不存在的crtendS.o、crtn.o，这个问题还没有解决：  
```
7.2.0/collect2 --eh-frame-hdr -dynamic-linker /system/bin/linker64 -X -EL -maarch64linux -z noexecstack -z relro -z now -pie -o /data/user/0/coding.yu.ccompiler.new/files/gcc/default /data/user/0/coding.yu.ccompiler.new/files/gcc/bin/../lib/gcc/aarch64-linux-android/7.2.0/../../../../aarch64-linux-android/lib/crtbegin_dynamic.o -L/data/user/0/coding.yu.ccompiler.new/files/gcc/bin/../lib/gcc/aarch64-linux-android/7.2.0 -L/data/user/0/coding.yu.ccompiler.new/files/gcc/bin/../lib/gcc -L/data/user/0/coding.yu.ccompiler.new/files/gcc/bin/../lib/gcc/aarch64-linux-android/7.2.0/../../../../aarch64-linux-android/lib /data/user/0/coding.yu.ccompiler.new/files/gcc/tmpdir/ccGloDhn.o -lz -ldl -lm -llog -lgcc -lc -ldl -lgcc /data/user/0/coding.yu.ccompiler.new/files/gcc/bin/../lib/gcc/aarch64-linux-android/7.2.0/../../../../aarch64-linux-android/lib/crtend_android.o
8.5.0/collect2 --eh-frame-hdr -dynamic-linker /system/bin/linker -X -EL -maarch64linux -pie -o test Scrt1.o crti.o crtbeginS.o -L/data/data/com.example.prootclang/files/SYSROOT/usr/bin/../lib/gcc/aarch64-linux-android/8.5.0 -L/data/data/com.example.prootclang/files/SYSROOT/usr/bin/../lib/gcc -L/data/data/com.example.prootclang/files/SYSROOT/usr/bin/../lib/gcc/aarch64-linux-android/8.5.0/../../.. ./ccoXgD4a.o -lz -ldl -lm -llog -lgcc -lc -lgcc crtendS.o crtn.o
```

