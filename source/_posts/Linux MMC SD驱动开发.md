---
title: Linux MMC/SD驱动开发
index_img: /img/post/mmc sd/mmc sd.jpeg
date: 2018-12-26 16:15:52
categories:
- Linux Driver
tags:
- 存储设备
comment: 'valine'
excerpt: Linux下MMC/SD驱动主要分三层：card层、core层、host层
---

<!--more-->

# 1. Linux MMC/SD驱动模型分析
        Linux下MMC/SD驱动主要分三层：card层、core层、host层。Linux下MMC/SD驱动框架见图1：

![ 图1](1.png)

**card层：**
要把操作的数据以块设备的处理方式写到记忆体上或从记忆体上读取。

**core层：**
则是将数据以何种格式，何种方式在 MMC/SD主机控制器与MMC/SD卡的记忆体(即块设备)之间进行传递，这种格式、方式被称之为规范或协议。 

**host层：**
就是要实现的具体MMC/SD相应控制器的驱动程序，包括MMC/SD控制器的初始化、寄存器读写及SD卡的命令、数据的传输接口。

       当前新版内核已将card层和core层合并。

# 2. Linux MMC/SD驱动框架的重要数据结构
## 2.1 struct mmc_host 结构体

![](2.1.png)

struct mmc_host用于与core层的命令请求，数据 传输等信息。

## 2.2 struct mmc_host_ops 结构体

![](2.2.png)

       其中request 主要是SD卡命令数据传输操作入口，set_ios是配置SD控制器寄存器操作入口，是驱动中必须要实现的两个基本操作。

# 3. SD卡初始化流程
       SD卡初始化流程是在core层实现的，集成了SD卡的所有命令，对于驱动开发来说，只需要实现request操作，并对命令和数据中断用mmc_request_done()函数进行上报处理即可。SD卡初始化流程见图2：

![ 图2](2.png)

# 4. Linux MMC/SD驱动热插拔功能
       对于热插拔功能，MMC/SD框架提供了mmc_detect_change()函数对卡插拔时间进行上报，此功能一般MMC/SD控制器会提插拔中断功能，驱动软件实现时只需在中断函数中上报即可。
