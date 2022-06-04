---
title: Linux Socket CAN——工具集嵌入式平台移植
index_img: /img/post/can-tool/tool.jpeg
date: 2022-06-02 12:14:54
categories:
- Linux Driver
tags:
- 网络设备
comment: 'valine'
excerpt: CAN总线设备如果被抽象为网络设备，那也支持像ip这类网络工具，Socket CAN的工具集是Canutils，这些工具还没在arm64的平台上移植过，后面主要介绍上述工具在arm64平台的移植过程。
---

<!--more-->

# 前言
       CAN总线设备如果被抽象为网络设备，那也支持像ip这类网络工具，Socket CAN的工具集是Canutils，这些工具还没在arm64的平台上移植过，后面主要介绍上述工具在arm64平台的移植过程。
       
# 1. 网络工具ip移植
       在对Linux Socket CAN驱动测试时需要应用软件工具ipiproute中的ip， 下面简要介绍iproute嵌入式移植。

1)下载源码：<https://src.fedoraproject.org/repo/pkgs/iproute/>，当时选择的版本为2.6.39。

2）修改Makefile
```makefile
CC = aarch64-linux-gnu-gcc                           修改为交叉编译器
...
SUBDIRS=lib ip                                              修改编译目标，只保留这两个
```

3）编译
```html
make
```

4）将/ip/ip拷贝到嵌入式系统的文件系统中。

# 2. canutils移植
       Canutils是基于GNU GPLv2许可的开源代码，包括canconfig、canecho、cansend、candump、cansequence五个工具，用于检测和监控Socket CAN接口。本平台采用arm64处理器，故交叉编译工具采用aarch64-linux-gnu

1）下载源码：<http://www.pengutronix.de/software/socket-can/download/canutils> ，下载最新版本canutils 4.0.6；

2）因为编译canutils需要libsocketcan库支持，下载libsocketcan：<http://www.pengutronix.de/software/libsocketcan/download/>，下载最新版本libsocketcan 0.0.11，因为以前版本不支持交叉编译工具aarch64-linux-gnu,故下载最新版本。

3）解压libsocketcan-0.0.11.tar.bz2。执行configure命令。（其中--host是指定交叉工具链，--prefix是指定库的生成位置） 
**配置**
```html
./configure \
--host=aarch64-linux-gnu \
--prefix=~/workspace/can/install/libsocketcan 
```
**编译**
```html
make
```
**安装**
```html
make install
```
       libsocketcan编译完成。

4）解压canutils-4.0.6.tar.bz2，进入解压目录，因canutils不支持交叉编译工具aarch64-linux-gnu，故需要修改文件。

       修改configure 文件2604行：ac_ct_CC=$ac_cv_prog_ac_ct_CC为ac_ct_CC="aarch64-linux-gnu-gcc"

       修改/config/autocof/config.sub文件：
```html       
241行：添加  | aarch64 | aarch64_be 
318行：添加| aarch64-* | aarch64_be-* 
```

       执行configure命令。（其中--host是指定交叉工具链，--prefix是指定库的生成位置，libsocketcan_LIBS是指定canconfig需要链接的库，LDFLAGS是指定外部库的路径，CPPFLAGS是指定外部头文件的路径）
```html
./configure \
--host=aarch64-linux-gnu \
--prefix=~/workspace/can/install/canutils  libsocketcan_LIBS=-lsocketcan LDFLAGS=-L~/workspace/can/install/libsocketcan/lib  libsocketcan_CFLAGS=-I~/workspace/can/install/libsocketcan/include CFLAGS=-I~/workspace/can/install/libsocketcan/include 
```
**编译**
```html
make
```

**安装**
```html
make install
```
       /workspace/can/install/canutils下生成四个目录，分别拷贝到开发板文件系统的相应目录。
