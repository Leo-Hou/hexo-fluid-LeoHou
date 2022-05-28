---
title: Linux驱动开发（一）——驱动何为
index_img: /img/post/linux驱动/linux-driver.jpg
date: 2022-02-16 09:32:52
categories:
- Linux Driver
tags:
- linux driver
comment: 'valine'
excerpt: 上一章节主要介绍了嵌入式Linux系统的基本架构，事实上Linux系统的应用不只局限于嵌入式系统，像Linux桌面系统、Linux服务器系统也是使用率非常高的场景，所以围绕Linux的软件开发门类众多。
---
## 前言：
<div class="markdown-body">
&emsp;&emsp;上一章节主要介绍了嵌入式Linux系统的基本架构，事实上Linux系统的应用不只局限于嵌入式系统，像Linux桌面系统、Linux服务器系统也是使用率非常高的场景，所以围绕Linux的软件开发门类众多。通过前面对linux系统的了解，我们知道Linux系统中一个重要的组成部分——设备驱动，那何为设备驱动？设备驱动又有什么作用呢？
</div>

# 一、Linux设备驱动
<div class="markdown-body">
&emsp;&emsp;从字面理解驱动（driver）是要驾驶什么，其实驱动就是是驱使硬件，是为了能让系统中的硬件设备能够正常工作的一部分代码。众所周知，Linux系统一切皆文件，我们在用户态想操作一个设备时可以以操作文件的方式来和硬件设备交互，但了解硬件设备的小伙伴清楚硬件设备提供的基础交互接口无非是寄存器、DMA、fifo等，那怎么通过文件的方式访问硬件设备呢？驱动程序大吼一声：没错，正是在下！<br>
&emsp;&emsp;驱动程序可以看成系统用户设备文件接口和硬件设备接口中间的翻译官，对下要安照硬件设备的工作方式对设备进行控制，并进行数据交互，对上要将硬件设备独有的工作方式进行抽象，封装成文件的各种访问操作接口及方式。
</div>

# 二、Linux设备驱动分类
<div class="markdown-body">
&emsp;&emsp;传统Linux系统会将设备驱动分为三大类：字符设备驱动、块设备驱动、网络设备驱动。除了以上三种还有一些设备无法纳入到这三种设备驱动模型的杂散（misc）设备，其实杂散设备也是以字符设备为基础，进行了更进步的抽象与封装。除了上面那些，在服务器领域，因为有IOMMU的加持，Linux内核实现了如 UIO，VFIO、USB等用户态驱动接口，基于用户态驱动的DPDK、SPDK等也在蓬勃发展。后续将依次介绍三种传统的Linux驱动程序。
</div>
