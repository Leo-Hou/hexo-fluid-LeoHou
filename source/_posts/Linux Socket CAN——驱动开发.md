---
title: Linux Socket CAN——驱动开发
index_img: /img/post/can驱动开发/can.png
date: 2022-06-02 12:14:44
categories:
- Linux Driver
tags:
- 网络设备
comment: 'valine'
excerpt: CAN是Controller Area Network\(控制器局域网\)的缩写。CAN通信协议在1986年由德国电气商博世公司所开发，主要面向汽车的通信系统
---

<!--more-->

# 1. CAN总线协议

       CAN是Controller Area Network\(控制器局域网\)的缩写。CAN通信协议在1986年由德国电气商博世公司所开发，主要面向汽车的通信系统。现已是ISO国际标准化的串行通信协议。根据不同的距离、不同的网络，可配置不同的速度，最高速度为1MBit/s。

CAN被细分为三个层次：

\(1\)CAN对象层\(the object layer\);

\(2\)CAN传输层\(the transfer layer\);

\(3\)CAN物理层\(the physical layer\);

CAN协议所对应的ISO模型见图1.1：

![图1.1](1.1.png)

       对象层和传输层包括所有由ISO/OSI模型定义的数据链路层的服务和功能。

## 1.1 对象层的作用范围包括：
\(1\)查找被发送的报文。

\(2\)确定由实际要使用的传输层接收哪一个报文。

\(3\)为应用层相关硬件提供接口。

## 1.2 传输层的作用主要：
\(1\)传送规则，也就是控制帧结构、执行仲裁、错误检测、出错标定、故障界定。

\(2\)总线上什么时候开始发送新报文及什么时候开始接收报文均在传输层里确定。

\(3\)位定时的一些普通功能也可以看作是传输层的一部分。

\(4\)传输层的修改是受到限制的。

## 1.3 物理层的作用：
       在不同节点之间根据所有的电气属性进行位信息的实际传输。当然，同一网络内，物理层对于所有的节点必须是相同的。尽管如此，在选择物理层方面还是很自由的。

# 2. Linux下Socket CAN驱动模型
       Linux下Socket CAN驱动属于网络设备的一部分。Linux下Socket CAN分层模型见图2.1：

![图2.1](2.1.png)

                                                                              

Linux下Socket CAN的驱动模型见图2.2：

![图2.2](2.2.png)

# 3. Socket CAN驱动框架的重要数据结构
## 3.1 struct net\_device\_ops结构体
![](3.1.png)   

       struct net\_device\_ops定义了网络设备的操作方法，.ndo\_open开启网络设备的操作，.ndo\_stop停止网络设备，.ndo\_start\_xmit发送网络数据，.ndo\_change\_mtu网络设备一次最大传输单元。

 

## 3.2 struct can\_frame 结构体

![](3.2.png)

       其中can\_id表示can frame的id，can\_dlc表示can frame数据的长度，data\[CAN\_MAX\_DLEN\]表示携带的数据。

## 3.3 struct platform\_driver 结构体
![](3.3.png)

       其中probe是驱动初始化函数入口，初始化本地结构体，remove是驱动卸载函数入口。

# 4. Linux下NAPI机制
       linux下网络数据接收机制NAPI：混合使用中断与轮询，而不使用纯粹的中断事件驱动模型。这样就提高了系统的性能，当设备产生一个数据接收中断后，新机制的软中断处理函数就会轮询设备的入口队列，直到入口队列中没有数据了，再开启中断。

NAPI数据接收的流程为：
    a、接收中断来临
    b、关闭接收中断
    c、以轮询方式接收所有数据包直到收空
    d、开启接收中断

NAPI驱动程序各部分的调用关系见图4.1：

![ 图4.1](4.1.png)


# 5. 数据发送接收流程
       Linux下Socket CAN在用户空间提供socket接口，在内核空间实现CAN Frame协议，并协同CAN控制器驱动控制CAN控制器的驱动，实现CAN通信。

## 5.1 发送流程

![](5.1.png)

## 5.2 接收流程

![](5.2.png)

                                               
