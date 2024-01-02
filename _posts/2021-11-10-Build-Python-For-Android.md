---
layout: post
title: 编译armeabi-v7a/arm64-v8a架构的Python
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

原来应用里带的还是好几年前的Python包，而且实现方式是仅支持target<=28，准备自己编译一个，支持target30.

<!--more-->

## 环境准备
* Ubuntu
* 安装和交叉编译相同版本的Python


## 编译libressl
```
#!/bin/bash

ARCH_BIT=64
COMPILE_ROOT=`pwd`
API=21
ANDROID_NDK_ROOT=/home/xxxxxx/Desktop/android-ndk-r21e
ANDROID_GCC_ROOT=${ANDROID_NDK_ROOT}/toolchains/llvm/prebuilt/linux-x86_64
ANDROID_GCC_PATH=${ANDROID_GCC_ROOT}/bin

if [ $ARCH_BIT -eq 32 ];then
    BUILD_PATH=${COMPILE_ROOT}/build32
    OUT_PATH=${COMPILE_ROOT}/out32
    CROSS_COMPILER=arm-linux-androideabi-
    CROSS_COMPILER_CLANG=armv7a-linux-androideabi${API}-
else
    BUILD_PATH=${COMPILE_ROOT}/build64
    OUT_PATH=${COMPILE_ROOT}/out64
    CROSS_COMPILER=aarch64-linux-android-
    CROSS_COMPILER_CLANG=aarch64-linux-android${API}-
fi

mkdir -p ${BUILD_PATH}
mkdir -p ${OUT_PATH}
export PATH=${ANDROID_NDK_ROOT}:${ANDROID_GCC_PATH}:$PATH

export CC="${CROSS_COMPILER_CLANG}clang" 
export CPP="${CROSS_COMPILER_CLANG}clang -E"
export CXX="${CROSS_COMPILER_CLANG}clang++"

export AS="${CROSS_COMPILER}as"
export LD="${CROSS_COMPILER}ld"
export GDB="${CROSS_COMPILER}gdb"
export STRIP="${CROSS_COMPILER}strip"
export RANLIB="${CROSS_COMPILER}ranlib"
export OBJCOPY="${CROSS_COMPILER}objcopy"
export OBJDUMP="${CROSS_COMPILER}objdump"
export AR="${CROSS_COMPILER}ar"
export NM="${CROSS_COMPILER}nm"
export READELF="${CROSS_COMPILER}readelf"

export CXXFLAGS="-D__ANDROID_API__=${API}"


cd ${BUILD_PATH}

if [ $ARCH_BIT -eq 32 ];then
    ../configure --host=arm-linux-androideabi --prefix=${OUT_PATH}
else
    ../configure --host=aarch64-linux-android --prefix=${OUT_PATH}
fi

make
make install
```

## 编译Python

### 修改Module/Setup文件
```
SSL=$(OPENSSL_PATH)
_ssl _ssl.c \
	-DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
	-L$(SSL)/lib -lssl -lcrypto

ZLIB=$(ZLIB_PATH)
zlib zlibmodule.c -I$(ZLIB)/include -L$(ZLIB)/lib -lz

_sha1 sha1module.c
_sha256 sha256module.c
_sha512 sha512module.c
_sha3 _sha3/sha3module.c
```

如果需要其他的内置模块，去掉模块前的注释即可

### 编译脚本
注：clang -E在编译Python是必须的，否则./configure测试会报编译不过。
```
#!/bin/bash

ARCH_BIT=64
COMPILE_ROOT=`pwd`
API=21
ANDROID_NDK_ROOT=/home/yuruxuan/Desktop/android-ndk-r21e
ANDROID_GCC_ROOT=${ANDROID_NDK_ROOT}/toolchains/llvm/prebuilt/linux-x86_64
ANDROID_GCC_PATH=${ANDROID_GCC_ROOT}/bin

if [ $ARCH_BIT -eq 32 ];then
	BUILD_PATH=${COMPILE_ROOT}/build32
	OUT_PATH=${COMPILE_ROOT}/out32
	CROSS_COMPILER=arm-linux-androideabi-
	CROSS_COMPILER_CLANG=armv7a-linux-androideabi${API}-
	export OPENSSL_PATH=/home/yuruxuan/Desktop/PythonProjects/libressl-3.4.1/out32
else
	BUILD_PATH=${COMPILE_ROOT}/build64
	OUT_PATH=${COMPILE_ROOT}/out64
	CROSS_COMPILER=aarch64-linux-android-
	CROSS_COMPILER_CLANG=aarch64-linux-android${API}-
	export OPENSSL_PATH=/home/yuruxuan/Desktop/PythonProjects/libressl-3.4.1/out64
fi

export ZLIB_PATH=/home/yuruxuan/Desktop/android-ndk-r21e/sysroot/usr

mkdir -p ${BUILD_PATH}
mkdir -p ${OUT_PATH}
export PATH=${ANDROID_NDK_ROOT}:${ANDROID_GCC_PATH}:$PATH


export CC="${CROSS_COMPILER_CLANG}clang" 
export CPP="${CROSS_COMPILER_CLANG}clang -E"
export CXX="${CROSS_COMPILER_CLANG}clang++"

export AS="${CROSS_COMPILER}as"
export LD="${CROSS_COMPILER}ld"
export GDB="${CROSS_COMPILER}gdb"
export STRIP="${CROSS_COMPILER}strip"
export RANLIB="${CROSS_COMPILER}ranlib"
export OBJCOPY="${CROSS_COMPILER}objcopy"
export OBJDUMP="${CROSS_COMPILER}objdump"
export AR="${CROSS_COMPILER}ar"
export NM="${CROSS_COMPILER}nm"
export READELF="${CROSS_COMPILER}readelf"
export CONFIG_SITE="config.site"

cd ${BUILD_PATH}

echo -e "ac_cv_file__dev_ptmx=yes\nac_cv_file__dev_ptc=no" > config.site

if [ $ARCH_BIT -eq 32 ];then
	../configure --host=arm-linux-androideabi \
	--build=x86_64-linux-gnu \
	--disable-ipv6 \
	--with-openssl=${OPENSSL_PATH} \
	--with-ensurepip=install \
	--prefix=${OUT_PATH} \
	LDFLAGS="-Wl,--allow-shlib-undefined -D__ANDROID_API__=${API} -fPIC" \
	CFLAGS="-D__ANDROID_API__=${API}" \
	CPPFLAGS="-D__ANDROID_API__=${API}" \
	CXXFLAGS="-D__ANDROID_API__=${API}"
else
	../configure --host=aarch64-linux-android \
	--build=x86_64-linux-gnu \
	--disable-ipv6 \
	--with-openssl=${OPENSSL_PATH} \
	--with-ensurepip=install \
	--prefix=${OUT_PATH} \
	LDFLAGS="-Wl,--allow-shlib-undefined -D__ANDROID_API__=${API} -fPIC" \
	CFLAGS="-D__ANDROID_API__=${API}" \
	CPPFLAGS="-D__ANDROID_API__=${API}" \
	CXXFLAGS="-D__ANDROID_API__=${API}"
fi

make -j4
make install
```

### pip管理工具
make install后，pip被装在了~/.local/pip路径下，每种架构编译后，需要把pip目录合并到编译出的python目录

## 注意事项
不要把ndk里的libz、libm放到python的lib下面，否则会出现不可预测的问题，我也不知道为什么（比如timedetla数值overflow）
