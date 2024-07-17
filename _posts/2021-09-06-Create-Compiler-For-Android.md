---
layout: post
title: 构建Android编译器和交叉编译补充
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

这段时间为了去除原来GCC模块，实验了很多思路，最后选定使用jniLibs放可执行文件这个方案。不过为了以后不再踩坑，在此记录下。

<!--more-->


## Clang + LLD > jniLibs

这是选定的方案，目前用着没有问题。编译的时候使用前面文章里的命令编译Clang、CXX、LLD就可以了。这个方案的缺点是Clang和LLD可执行文件太大，Clang 120MB + LLD 80MB，strip之后放入apk打包，占用大小50MB。GCC不太好迁移到这个方案，GCC命令依赖cc1、cc1plus，可能需要指定依赖的文件路径或者分步编译。


### 编译C语言命令
```
/data/app/~~PhH2YAfgALdzaQTWUZMfEg==/com.test.package-oEFkOFLYdXsEAG9M9rExQw==/lib/arm64/libclang.so \
/data/user/0/com.test.package/files/default.c \
--sysroot /data/user/0/com.test.package/files/SYSROOT \
-w -pie -lz -ldl -lm \
-I /data/user/0/com.test.package/files/SYSROOT/usr/include \
-I /data/user/0/com.test.package/files/SYSROOT/usr/lib/clang/12.0.1/include \
-I /data/user/0/com.test.package/files/SYSROOT/usr/include/aarch64-linux-android \
-o /data/user/0/com.test.package/files/default
```

### 编译C++命令
```
/data/app/~~vAHklWJbnoGUKWkeijdZxQ==/com.test.packagepp-bQ9QosVfvyGquj0WeBS4gA==/lib/arm64/libclang.so \
/data/user/0/com.test.packagepp/files/default.cpp \
--sysroot /data/user/0/com.test.packagepp/files/SYSROOT \
-w -pie -lz -ldl -lm -lc++ -lc++abi \
-isystem /data/user/0/com.test.packagepp/files/SYSROOT/usr/include \
-isystem /data/user/0/com.test.packagepp/files/SYSROOT/usr/include/aarch64-linux-android \
-isystem /data/user/0/com.test.packagepp/files/SYSROOT/usr/include/c++/v1 \
-isystem /data/user/0/com.test.packagepp/files/SYSROOT/usr/lib/clang/12.0.1/include \
-o /data/user/0/com.test.packagepp/files/default
```

### 注意事项

1. PATH环境变量里需要配置自己的带ld的目录，由于我们希望使用lld，创建软连接到lib/arm64/liblld.so文件。由于每次安装同包名apk后so的路径都会改变，需要每次安装后都重新创建软连接，因此我们偷懒，每次运行都重新创建。  
2. C++的include路径是有顺序的，如果使用-I添加search_path会添加在sysroot默认路径之后，而且sysroot默认添加后，-I再添加同样的路径将会被忽略，这就导致include顺序错误，-isystem会先于sysroot默认添加配置。


## Clang + LD(GNU) > jniLibs

这个方案成功了一半。先说用人家的ld，这个是可以的，但是ld --verbose会显示人家的路径，这正是我要去掉的。自己交叉编译binutils过了，但是提取出来的ld一直提示我的crtbegin_dynamic.o架构不对，但实际是对的，这个不知道为什么，大胆推测需要GCC和binutils一起编译，没有再继续尝试。


## Clang + LD(GNU) > PRoot

这个方案在64位架构(Android 11 Target=30)上运行没有问题，但是在32位(Android 10 Target=30)上运行失败，连最基本的clang -v都运行失败，因此决定切换到jniLibs方案，不过编译出来的default可执行文件还是PRoot运行。


## 附：Ubuntu新装系统编译环境和注意事项

1. Ubuntu 20.04 使用的ndk r16e构建的独立工具链
2. 需要apt-get install gcc-10 g++-10 python git wget cmake ninja-build  
3. binutils编译两个架构的时候，尽量用两套不同的代码，同样的代码需要清除config.cache
