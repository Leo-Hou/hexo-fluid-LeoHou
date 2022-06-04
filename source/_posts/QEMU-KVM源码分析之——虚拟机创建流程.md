---
title: QEMU/KVM源码分析之——虚拟机创建流程
index_img: /img/post/qemu-kvm原码分析之虚拟机创建/qemu-kvm.png
date: 2022-03-13 22:11:45
categories:
- Virtualization
tags:
- QEMU
- KVM
comment: 'valine'
excerpt: Linux系统下QEMU/KVM虚拟化架构有两部分：QEMU和KVM。
---
# 一、背景
<div class="markdown-body">
&emsp;&emsp;Linux系统下QEMU/KVM虚拟化架构有两部分：QEMU和KVM。<br>
&emsp;&emsp;QEMU用户态虚拟机管理工具，它是一个普通的Linux进程，为客户机提供设备模拟的功能，包括模拟
BIOS、PCI/PCIE总线、磁盘、网卡、显卡、声卡、键盘、鼠标等。同时它通过ioctl系统调用与内核态的KVM模块进
行交互 。<br>
&emsp;&emsp;KVM内核模块，它属于标准Linux内核的一部分，是一个专门提供虚拟化功能的模块，主要负责CPU和
内存的虚拟化，包括：客户机的创建、虚拟内存的分配、 CPU执行模式的切换、vCPU寄存器的访问、vCPU指令的执行。<br>
&emsp;&emsp;QEMU和KVM通过IOCTL进行交互，Linux系统下QMEU/KVM虚拟化架构见图1.1：</div>

![图1.1](图1.1.png)

# 二、虚拟机创建流程
## 2.1  KVM初始化流程
<div class="markdown-body">
&emsp;&emsp;KVM是Linux系统下的一个内核模块，下面以AMD的x86架构为例介绍KVM内核模块的初始化流程。kvm-amd以内核模块的方式加载入Linux内核，然后执行kvm_init()，在kvm_init()中进行了一系列初始化，见图2.1.1：</div>

![图2.1.1](图2.1.1.png)

## 2.2 QEMU中kvm_init注册流程
<div class="markdown-body">
&emsp;&emsp;kvm_init是QEMU中创建虚拟机的入口，下图主要呈现了kvm_init的注册流程：</div>

![图2.2.1](图2.2.1.png)

## 2.3 虚拟机创建流程
<div class="markdown-body">
&emsp;&emsp;kvm_init发起创建虚拟机的的流程，见图2.3.1：</div>

![图2.3.1](图2.3.1.png)

## 2.4 vCPU创建流程
<div class="markdown-body">
&emsp;&emsp;kvm_init创建虚拟机时同时要创建vCPU，kvm_init发起创建vCPU流程，见图2.4.1：</div>

![图2.4.1](图2.4.1.png)


