---
title: 嵌入式软件开发（三）——嵌入式linux系统
index_img: /img/post/嵌入式linux/嵌入式linux.jpg
date: 2022-02-15 20:38:46
categories:
- 嵌入式软件开发
tags:
- 嵌入式
- 嵌入式linux
comment: 'valine'
excerpt: 上一节聊到裸机程序开发，虽然通过一些操作系统的思想可以有条件的实现受限多任务，但裸机程序仍然无法高效的实现多任务处理。
---
## 前言：
<div class="markdown-body">
&emsp;&emsp;上一节聊到裸机程序开发，虽然通过一些操作系统的思想可以有条件的实现受限多任务，但裸机程序仍然无法高效的实现多任务处理。操作系统引入可以很好的解决这个问题，不止如此，操作系统还可以管理处理器资源、管理系统存储资源、管理系统外设及提供操作系统与用户之间的接口。综合来说操作系统就是管理系统的软、硬件资源，向下驱动硬件设备,向上为用户提供调用接口的一套管理机制。本节主要介绍嵌入式linux系统的组成。</div>

# 一、嵌入式linux系统的启动流程
<div class="markdown-body">
&emsp;&emsp;通过前面章节我们了解到，CPU的运行过程就是获取软件指令并执行的过程，操作系统本质上也是一种软件，所以操作系统的启动过程也是CPU启动、运行的过程。此处以AARCH64架构CPU为例，嵌入式Linux启动流程主要分3部分：CPU片内firmware、boot loader（此处以u-boot为例）、linux系统，见下图：</div>

#### 
![图1.1](emb_sys1.png)
<div class="markdown-body">
&emsp;&emsp;firmware一般为CPU厂家在出厂时烧写的固件程序，其功能主要是CPU的初始化及CPU启动方式功能的实现，此部分视CPU厂家不同，实现的功能也不尽相同，有的CPU厂家就没有这部分，但有些NXP的CPU，通过CPU管教配置决定开发板是通过什么方式启动（Flash启动、SD卡启动、USB启动甚至网络启动），这个firmware就是要实现相应启动方式的硬件设备驱动及程序代码的拷贝。
&emsp;&emsp;U-boot是boot loader的一种，是操作系统的引导程序，其主要作用可以分为三部分：CPU初始化、板级设备初始化、引导操作系统内核。U-boot本质上是一段裸机程序，按流程依次实现上述功能。
&emsp;&emsp;Linux系统主要有两部分组成：linux kernel、filesystem，启动时先要启动内核，然后挂载根文件系统rootfs，然后再运行一系列应用进程。嵌入式系统中根文件系统的制作可以采用busybox。
</div>

# 二、嵌入式Linux系统架构
<div class="markdown-body">
&emsp;&emsp;嵌入式linux系统相对于通用计算机系统的桌面linux系统，其基本架构一致，但组件比桌面系统要少，前面我们了解嵌入式系统是软、硬件可裁剪，Linux系统的可裁剪特性完美切合了嵌入式系统的需求，所以在嵌入式系统中选择嵌入式Linux系统在系统裁剪方面有着先天优势，同时linux的系统调用遵循posix规范，方便Linux系统的应用在嵌入式系统中移植、适配。下图是嵌入式linux系统的整体结构图：</div>

#### 
![图2.1](emb_sys2.png)
<div class="markdown-body">
&emsp;&emsp;嵌入式Linux系统主要由用户态应用程序APP和内核组成，内核大方向上分两层：内核组件和设备驱动。用户态APP通过系统调用与内核态的设备驱动程序进行交互，由驱动程序对外设进行控制，协调外设工作。<br>

# 三、结束语 
<div class="markdown-body">
&emsp;&emsp;至此，嵌入式部分的内容基本就分享完了，后续章节将着重探讨linux驱动程序开发及内核分析。
<div>
