---
layout: post
title: 编译arm64-v8a架构的Clang(上)
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

目前软件里面使用的人家的GCC编译模块，觉得这样不太好。自己想了两个方案，一个是换成Clang，另一个是换成自己编译的GCC，不过这两种都需要自己编译，今天先研究Clang。

<!--more-->


## 准备环境

* Ubuntu 20.04 LTS 6G内存 50G硬盘空间
* git
* cmake
* gcc 10 & g++ 10
* Android NDK r21e
  
我首先尝试编译最新的12.0.1版本，发现系统自带的gcc(9)版本编译不过，查论坛才知道需要使用10版本。


## 下载代码

```
git clone -b llvmorg-12.0.1 --depth 1 https://github.com/llvm/llvm-project.git
```
-b llvmorg-12.0.1: 指定tag版本  
--depth 1: 无提交历史  


## 编译

### 生成配置

```
cmake -S llvm -B build -G Ninja \
-DLLVM_ENABLE_PROJECTS="clang" \
-DCMAKE_BUILD_TYPE=MinSizeRel \
-DCMAKE_SYSTEM_NAME=Android \
-DCMAKE_SYSTEM_VERSION=21 \
-DCMAKE_ANDROID_ARCH_ABI=arm64-v8a \
-DCMAKE_ANDROID_NDK=/home/yu/Desktop/android-ndk-r21e \
-DCMAKE_ANDROID_STL_TYPE=c++_static \
-DLLVM_DEFAULT_TARGET_TRIPLE=aarch64-unknown-linux-android
```

* -S 源码路径
* -B 构建路径
* -DLLVM_ENABLE_PROJECTS="clang" 编译的子项目，这里只添加了clang
* -DCMAKE_BUILD_TYPE=MinSizeRel 编译版本为最小生成二进制
* -DCMAKE_ANDROID_NDK=/home/... NDK绝对路径
* -DCMAKE_ANDROID_STL_TYPE=c++_static 使用c++库的版本和形式
* -DLLVM_DEFAULT_TARGET_TRIPLE=aarch64-unknown-linux-android 编译的target
  
其中有两项需要说一下：编译MinSizeRel和Release生成的二进制大小相差不多，好像没有什么作用；最开始 -DLLVM_DEFAULT_TARGET_TRIPLE 没有加上，默认生成的是x86_64开头的target，但是也能正常工作。


### 开始编译

```
cmake --build build
```
我编译了差不多半个小时，编译完整个项目30个G。


## 使用

build/bin/clang-12就是我们想要的文件，clang单文件版本，120MB。  

这个单文件版本依然需要工具链的其他部分配合使用，比如as,ld，或者编译时候顺便把llvm的linker编译出来 -DLLVM_ENABLE_PROJECTS="clang;lld"。


## 另外想说的

gcc 7.0才30MB，这个clang单文件就120MB，太大了。后来我编译了6.0版本的clang，编译完85MB，还是很大，但是也只能这样了。  

2021-08-17更新：一方面我没有注意是否去掉可执行文件的debug_info，另一方面也没有使用-Os优化编译大小


## 参考资料
LLVM/Clang项目主页：[https://github.com/llvm/llvm-project](https://github.com/llvm/llvm-project)  
cmake使用教程（九）-关于安卓的交叉编译：[https://juejin.cn/post/6844903565803126798](https://juejin.cn/post/6844903565803126798)  
How To Cross-Compile Clang/LLVM：[https://llvm.org/docs/HowToCrossCompileLLVM.html](https://llvm.org/docs/HowToCrossCompileLLVM.html)  
Cross-compilation using Clang：[https://clang.llvm.org/docs/CrossCompilation.html](https://clang.llvm.org/docs/CrossCompilation.html)  
