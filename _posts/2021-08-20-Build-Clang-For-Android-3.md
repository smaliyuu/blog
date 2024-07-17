---
layout: post
title: 编译armeabi-v7a架构的Clang遇到的问题
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

本来我以为已经没什么问题了，但是在尝试编译armv7版本的clang的时候还是遇到了问题，简单总结下。

<!--more-->


## 编译armeabi-v7a架构的clang，libcxx，libcxxabi

```
# 交叉编译Android平台需要指定-ldl
export LDFLAGS="-ldl"

# 配置
cmake -S llvm -B build_arm -G Ninja \
-DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi" \
-DCMAKE_BUILD_TYPE=MinSizeRel \
-DCMAKE_C_FLAGS="-Os -D_LIBCPP_HAS_NO_OFF_T_FUNCTIONS" \
-DCMAKE_CXX_FLAGS="-Os -D_LIBCPP_HAS_NO_OFF_T_FUNCTIONS" \
-DCMAKE_SYSTEM_NAME=Android \
-DCMAKE_SYSTEM_VERSION=21 \
-DCMAKE_ANDROID_ARCH_ABI=armeabi-v7a \
-DCMAKE_ANDROID_NDK=/home/yu/Desktop/android-ndk-r21e \
-DLLVM_DEFAULT_TARGET_TRIPLE=armv7a-unknown-linux-androideabi

# 编译
cmake --build build_arm

# 安装
cmake --install build_arm/ --prefix clang_install_arm/

```

大部分都和arm64-v8a一致，其中加入了一个宏定义-D_LIBCPP_HAS_NO_OFF_T_FUNCTIONS，如果不加的话会报fseek/ftello找不到的错误：  

```
error: use of undeclared identifier 'fseeko' if (fseeko(__file_, __width > 0 ? __width * __off : 0, __whence))
```

fseeko/ftello函数在Android api>=24才可以用，我们是api=21肯定没有，我看了下include/c++/v1/fstream的源码，通过编译时宏定义，选择使用fseek函数来代替fseeko。详情请见NDK/sysroot/usr/include/stdio.h 和 llvm_project/build_arm/include/c++/v1/fstream.  
  
问题是解决了，但是有一个疑惑，为什么我在编译arm64的时候没遇到这个问题呢，难道cmake帮我配置了？那armv7怎么没配置，很奇怪。
