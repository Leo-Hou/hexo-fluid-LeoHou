---
title: 电子信息技术（四）——CPU架构
index_img: /img/post/cpu/cpu.png
date: 2022-02-06 13:26:47
categories:
- 电子信息技术
tags:
- CPU架构
comment: 'valine'
excerpt: 上一节聊到了数字电路，那大规模集成的数字电路的巅峰便是：CPU（中央处理器）。本节就聊一下CPU的基本架构和当前最广泛应用的两种处理器架构x86架构和ARM架构。
---
# 前言：
<div class="markdown-body">
&emsp;&emsp;上一节聊到了数字电路，那大规模集成的数字电路的巅峰便是：CPU（中央处理器）。本节就聊一下CPU的基本架构和当前最广泛应用的两种处理器架构x86架构和ARM架构。
</div>

# 一、CPU基本架构
<div class="markdown-body">
&emsp;&emsp;传统CPU的架构主要是由控制器（CU）、逻辑运算（ALU）、寄存器、中断系统及部分组成，其架构图见下图：</div>

#### 
![CPU架构](cpu-arch.jpg)<br>
（1）CU：控制器根据预定的指令执行顺序，从主存中取出一条指令，按照该条指令的功能，控制CPU各部件的操作；
（2）ALU：接受控制器的命令，完成算术和逻辑运算；
（3）寄存器：存放下一指令地址；存放当前指令；存放操作数和计算结果；
（4）中断系统：处理异常情况和特殊请求。

以上是CPU的简单的架构图，但技术发展到当今，CPU的架构要比上述情况复杂的多。

# 二、CPU的工作流程
<div class="markdown-body">
&emsp;&emsp;一个计算机系统中的CPU最基本的工作就是执行存储的指令序列。CPU从存储中取出一条指令，译码并执行这条指令，保存执行结果，然后取下一条指令，周而复始。下图为CPU执行程序的过程：</div>

#### 
![CPU指令执行](cpu-cmd.jpg)

#### 举例：
如果想计算1+2=？的一个运算，CPU的工作过程是怎样的呢？先看如下汇编（以x86为例）代码：
1） mov ax 1
2） mov bx 2
3） add ax bx

1）表示CPU执行mov指令时要将数值1写入寄存器ax；
2）同理是将数值2写入寄存器bx；
3）表示CPU在读取add指令后要将ax中的值和bx中的值用加法器进行加法运算，然后将计算结果放入寄存器。

# 三、x86架构和ARM架构
<div class="markdown-body">
&emsp;&emsp;当前最广泛应用的两种处理器架构x86架构和ARM架构，其中x86架构处理器主要用在PC电脑和服务器，ARM架构处理器主要在嵌入式电子设备中广泛应用，除了上诉CPU架构还有已经没落的 PowerPC、龙芯深耕的MIPS及被CPU架构的后起之秀RISC-V，此处不做进一步说明，我们主要聊一下x86架构和ARM架构。
</div>

## 3.1 x86架构
<div class="markdown-body">
&emsp;&emsp;x86架构CPU厂商的代表就是intel和AMD，intel是x86世界的霸主，在PC电脑和服务器应用领域拥有先起的生态优势，但近些年AMD在高性能服务器CPU推出ZEN架构以来，大有赶超intel的趋势，并且ZEN架构不断优化升级，在2020年10月推出了ZEN3架构，未来服务器市场是否会被AMD蚕食更多市场份额可以静观其变。至于intel架构此处不做多说，有兴趣的朋友可以自行查阅：</div>

#### [https://software.intel.com/sites/default/files/managed/a4/60/253665-sdm-vol-1.pdf](https://software.intel.com/sites/default/files/managed/a4/60/253665-sdm-vol-1.pdf)

## 3.2 ARM架构
<div class="markdown-body">
&emsp;&emsp;ARM指令集属于精简指令集，发展到今天 ARM的Cortex 家族大放异彩，当前 Cortex家族按CPU架构可以分为Cortex-A、Cortex-R、Cortex-M三个系列，其中A系列主要用在智能移动终端、PC、服务器，Cortex-R系列主要用在汽车电子等高精度嵌入式领域，而Cortex-M系列主要是MCU（单片机）用在低端嵌入式领域。
</div>

## 3.3 x86和ARM的对决
<div class="markdown-body">
&emsp;&emsp;x86在架构上属于复杂指令集，其在PC和服务器领域的性能有着无可匹敌的优势，其Soc内部各模块设计复杂，相互耦合性高，片内、片外各总线数据传输性能优秀。
ARM属于精简指令集架构，在嵌入式领域已是绝对霸主，其指令集和架构有条件开源的模式使其生态迅速铺展，近些年ARM在着力提升单核性能，剑指PC和服务器市场。由于ARM采用开源生态，其CPU内各组件模块化，相互间耦合性比较小，便于各厂商DIY，减小开发难度，但也因此会在性能存在瓶颈。
&emsp;&emsp;国际环境复杂多变，国内多厂商在进行ARM处理的研发，小生态已经形成，我们可以拭目以待。
</div>
