---
title: Linux4.6.0下的网络设备驱动
index_img: /img/post/Linux4.6.0下的网络设备驱动/OSI.png
date: 2017-07-25 15:24:11
categories:
- Linux Driver
tags:
- 网络设备
comment: 'valine'
excerpt:  Linux下网络协议模型主要分四层：网络接口层、网络层、传输层、应用层，与OSI七层协议参考模型
---

<!--more-->

# 1. Linux网络协议模型
　    Linux下网络协议模型主要分四层：网络接口层、网络层、传输层、应用层，与OSI七层协议参考模型的对比见图1.1：

![图1.1](1.1.png)

下面分别描述TCP/IP分层模型的四个协议层分别完成的功能。


## 1.1 网络接口层
　    网络接口层包括用于协作IP数据在已有网络介质上传输的协议。实际上TCP/IP标准并不定义与ISO数据链路层和物理层相对应的功能。相反，它定义像 地址解析协议（Address Resolution Protocol,ARP）这样的协议，提供TCP/IP协议的数据结构和实际物理硬件之间的接口。

## 1.2 网络层
　    网络层对应于OSI七层参考模型的网络层。本层包含IP协议、RIP协议（Routing Information Protocol，路由信息协议），负责数据的包装、寻址和路由。同时还包含网间控制报文协议（Internet ControlMessage Protocol,ICMP）用来提供网络诊断信息。

## 1.3 传输层
　    传输层对应于OSI七层参考模型的传输层，它提供两种端到端的通信服务。其中TCP协议（Transmission Control Protocol）提供可靠的数据流运输服务，UDP协议（Use DatagramProtocol）提供不可靠的用户数据报服务。

## 1.4 应用层
　　应用层对应于OSI七层参考模型的应用层和表达层。因特网的应用层协议包括Finger、Whois、FTP（文件传输协议）、Gopher、HTTP（超文本传输协议）、Telent（远程终端协议）、SMTP（简单邮件传送协议）、IRC（因特网中继会话）、NNTP（网络新闻传输协议）等。

# 2. Linux网络子系统
        网络子系统在Linux内核中主要负责管理各种网络设备，并实现各种网络协议栈，最终实现通过网络连接其它系统的功能。在Linux内核中，网络子系统几乎是自成体系，它包括5个子模块，见图2.1：

![图2.1](2.1.gif)

其各部分的功能如下：

1）Network Device Drivers，网络设备的驱动。

2）Device Independent Interface，该模块定义了描述硬件设备的统一方式即统一设备模型，所有的设备驱动都遵守这个定义，可以降低开发的难度。同时可以用一致的形势向上提供接口。

3）Network Protocols，实现各种网络传输协议，例如IP, TCP,UDP，ICMP等。

4）Protocol Independent Interface，屏蔽不同的硬件设备和网络协议，以相同的格式提供接口（socket\)。

5）System Call interface，系统调用接口，向用户空间提供访问网络设备的统一的接口。

 

# 3. Linux网络设备驱动的结构

        Linux下网络设备驱动可以划分为四层，由上到下依次为：网络协议接口层、网络设备接口层、设备驱动功能层、网络设备和媒介层，其结构图见图3.1:

![图3.1 网络设备驱动结构框图](3.1.bmp)

各层作用如下所示：

1）网络协议接口层向网络层协议提供提供统一的数据包收发接口，不论上层协议为ARP还是IP，都通过dev\_queue\_xmit\(\)函数发送数据，并通过netif\_rx\(\)函数接受数据。这一层的存在使得上层协议独立于具体的设备。

2）网络设备接口层向协议接口层提供统一的用于描述具体网络设备属性和操作的结构体net\_device，该结构体是设备驱动功能层中各函数的容器。实际上，网络设备接口层从宏观上规划了具体操作硬件的设备驱动功能层的结构。

3）设备驱动功能层各函数是网络设备接口层net\_device数据结构的

具体成员，是驱使网络设备硬件完成相应动作的程序，他通过hard\_start\_xmit\(\)函数启动发送操作，并通过网络设备上的中断触发接受操作。

4）网络设备与媒介层是完成数据包发送和接受的物理实体，包括网络适配器和具体的传输媒介，网络适配器被驱动功能层中的函数物理上驱动。对于Linux系统而言，网络设备和媒介都可以是虚拟的。

## 3.1 网络协议接口层

        网络协议接口层提供函数dev\_queue\_xmit\(struct sk\_buff \*skb\)供上层调用用以发送数据，提供函数int netif\_rx\(struct sk\_buff \*skb\)来传递一个struct sk\_buff数据结构的指针来完成数据包接收。

        此处用了一个sk\_buff结构体，含义为“套接字缓冲区”，此结构体定义于/include/linux/skbuff.h文件中，用于在Linux网络子系统中的各层之间传递数据。下面是sk\_buff结构体的关键成员：

```c
struct sk_buff {

  union {

         struct {

                /*These two members must be first. */

                structsk_buff            *next;

                structsk_buff            *prev;

                union{

                       ktime_t          tstamp;

                       structskb_mstamp skb_mstamp;

                };

         };

         structrb_node   rbnode; /* used in netem &tcp stack */

  };

  struct sock            *sk;

  structnet_device       *dev;

  …

  unsigned int              len,

                            data_len;

  __u16                     mac_len,

                            hdr_len;

  …

  union {

         __be16         inner_protocol;

         __u8           inner_ipproto;

   };

   __u16                inner_transport_header;

   __u16                inner_network_header;

   __u16                inner_mac_header;

   __be16               protocol;

   __u16                transport_header;

   __u16                network_header;

   __u16                mac_header;

   …

   /* These elements must be at the end, see alloc_skb()for details.  */

   sk_buff_data_t        tail;

   sk_buff_data_t        end;

   unsigned char         *head,

                         *data;

} 
```

        其中head指向已分配空间开头，data指向有效的octet开头，tail指向有效的octet结尾，而end指向tail可以到达的最大地址。每一层会在head和data之间填充协议头，或在tail和end之间添加新的协议数据。见图3.2：

![图3.2](3.2.bmp)

Linux为操作套接字缓冲区提供了分配、释放、变更等操作函数。

分配：
```c
struct sk\_buff \*alloc\_skb\(unsignedint len, gfo\_t priority\);

struct sk\_buff \*dev\_alloc\_skb\(unsigned int len\);

static inline struct sk\_buff \*netdev\_alloc\_skb\_ip\_align\(struct

net\_device \*dev, unsigned int length\);

```

释放：
```c
void kfree\_skb\(struct sk\_buff \*skb\);

void dev\_kfree\_skb\(struct sk\_buff \*skb\);

voiddev\_kfree\_skb\_irq\(struct sk\_buff \*skb\);

voiddev\_kfree\_skb\_any\(struct sk\_buff \*skb\);

```

变更：
```c
unsigned char \*skb\_put\(struct sk\_buff \*skb, unsigned intlen\);

unsigned char \*skb\_push\(structsk\_buff \*skb, unsigned int len\);

staticinline void skb\_reserve\(struct sk\_buff \*skb, unsigned int len\);

```

       此处着重介绍一下函数\*netdev\_alloc\_skb\_ip\_align\(\)，此函数的作用是申请一个skb描述符及为报文数据buffer分配内存空间。通常内存空间的分配地址以32位处理器来说是四字节对齐的，但由于以太网的帧格式的MAC头没有四字节对齐，这样会导致以太网报文的IP头也不符合四字节对齐，如此会造成IP协议解析校验错误。如果在申请skb描述符时调用\*netdev\_alloc\_skb\_ip\_align\(\)，分配的报文空间做MDA映射时MAC头会自动四字节对其，以使接收到的以太网报文在上传到IP层时能够解析通过。

## 3.2 网络设备接口层

       网络接口层主要为众多的网络设备定义统一的抽象数据结构net\_device结构体。net\_device结构可分为全局信息、硬件信息、接口信息、设备操作方法和辅助成员等五个部分，其主要关键信息如下：

1）全局信息
```c
char name\[INFAMSIZ\]    设备名 
```
2）硬件信息
```c
unsigned long          mem\_end; 

unsigned long          mem\_start;

unsigned long          base\_addr;

int                             irq;

unsigned char          if\_port;

unsigned char          dma;
```
       其中men\_start和men\_end分别定义了设备所使用的共享内存的起始地址和结束地址；base\_addr是网络设备的I/O基地址；irq为设备使用的中断号；if\_port指多端口设备使用哪一端口；dma指分配给设备的DMA通道。

3）接口信息
```c
unsigned int              mtu;

unsigned short          type;

unsigned short          hard\_header\_len;
```
mtu指最大传输单元；

type表示接口的硬件类型；

hard\_header\_len指网络设备硬件头长度。

4）设备操作方法

       结构体struct net\_device\_ops定义了网络设备的一系列硬件操作方法的函数的合集，原型定义于include/linux/netdevice.h中,以下是结构体中定义的主要操作函数：

```c
struct net_device_ops {

int                 (*ndo_init)(struct net_device*dev);

void                (*ndo_uninit)(struct net_device*dev);

int                 (*ndo_open)(struct net_device*dev);

int                 (*ndo_stop)(struct net_device*dev);

netdev_tx_t         (*ndo_start_xmit)(struct sk_buff *skb,

                                      structnet_device *dev);

 

netdev_features_t     (*ndo_features_check)(struct sk_buff *skb,

                                            struct net_device *dev,

                                            netdev_features_t features);

u16                   (*ndo_select_queue)(structnet_device *dev,

                                          struct sk_buff *skb,

                                          void *accel_priv,

                                          select_queue_fallback_t fallback);

void                  (*ndo_change_rx_flags)(structnet_device *dev,

                                             intflags);

void                  (*ndo_set_rx_mode)(structnet_device *dev);

int                   (*ndo_set_mac_address)(struct net_device*dev,

                                             void *addr);

int                   (*ndo_validate_addr)(structnet_device *dev);

int                   (*ndo_do_ioctl)(structnet_device *dev,

                                      struct ifreq *ifr, int cmd);

…

};
```

ndo\_init（）是网络设备初始化；

ndo\_open（）是打开网络接口设备；

ndo\_stop（）是停止网络设备；

ndo\_start\_xmit（）启动数据包发送；

ndo\_do\_ioct（）进行设备特定的I/O控制。

除此之外，net\_device中还提供了ethtool\_ops、header\_ops这样的操作集。

5）辅助成员
```c
unsigned long           last\_rx;

unsigned long           trans\_start;
```
last\_rx 记录最后一次接收到收据包时的时间戳，trans\_start记录最后数据包开始发送时的时间戳。

## 3.3 设备驱动功能层

       net\_device结构体的成员\(属性和函数指针\)需要被设备驱动功能层的具体数值和函数赋予。对具体的设置xxx，工程师应该编写设备驱动功能层的函数，这些函数型如xxx\_open\(\)、xxx\_stop\(\)、xxx\_hard\_header\(\)、xxx\_get\_stats\(\)、xxx\_tx\_timeout\(\)等。其实就是net\_device中相应操作函数的实体映射。

       对于采用中断方式接收数据包的操作，设备驱动功能层中有很大一部分工作是中断处理。

## 3.4 网络设备与媒介层

       网络设备与媒介层直接对应于实际的硬件设备的操作。

# 4. 驱动的实现

      在设计具体的网络设备驱动程序时，我们的主要工作是实现设备驱动功能层的相关函数并填充net\_device数据结构的内容并将net\_device注册入内核

## 4.1 网络设备的注册和注销
     网络设备注册方式与字符驱动不同之处在于它没有主次设备号，并使用下面的函数注册：
```c
intregister\_netdev\(struct net\_deivce\*dev\)；
```
     网络设备的注销，使用下面的函数注销：
```c
void unregister\_netdev\(structnet\_device\*dev\)；
```
## 4.2 网络设备初始化
       设备探测工作在init方法中进行，一般调用一个称之为probe方法的函数初始化的主要工作时检测设备，配置和初始化硬件，最后向系统申请这些资源。此外填充该设备的dev结构，我们调用内核提供的ether\_setup方法来设置一些以太网默认的设置。

## 4.3 网络设备打开和关闭
### 1）网络设备打开
      open这个方法在网络设备驱动程序里是网络设备被激活时被调用的（即设备状态由down变成up），实际上很多在初始化的工作可以放到这里来做。比如说资源的申请，硬件的激活。如果dev->open返回非0，则硬件状态还是down，注册中断、DMA等；设置寄存器，启动设备；启动发送队列。一般注册中断都在init中做，但在网卡驱动程序中，注册中断大部分都是放在open中注册，因为要经常关闭和重启网卡。

### 2）网络设备关闭
       stop方法做和open相反的工作可以释放某些资源以减少系统负担stop是在设备状态由up转为down时被调用。

## 4.4 数据的发送与接收
### 1）数据的发送
       在系统调用的驱动程序的hard\_start\_xmit时，发送的数据放在一个sk\_buff结构中。一般的驱动程序传给硬件发出去。也有一些特殊的设备比如说loopback把数据组成一个接收数据在传送给系统或者dummy设备直接丢弃数据。如果发送成功,hard\_start\_xmit方法释放sk\_buff。如果设备暂时无法处理，比如硬件忙，则返回1。

### 2）数据接收
       Linux下提供了三种收据接收方式：中断方式、poll\_controller轮询方式和NAPI（New API）方式。驱动程序对数据接受的处理就是将接收到的数据填充skb并调用netif\_rx函数将skb交交给设备无关层。通常情况下，网络设备驱动以中断方式接收数据包，当设备收到数据后都会产生一个中断，在中断处理程序中驱动程序申请一块sk\_buff\(skb\)从硬件中读取数据位置到申请号的缓冲区里。接下来填充sk\_buff中的一些信息。中断有可能是收到数据产生也可能是发送完成产生，中断处理程序要对中断类型进行判断，如果是收到数据中断则开始接收数据，如果是发送完成中断，则处理发送完成后的一些操作，比如说重启发送队列。

接收流程：
a、分配skb=dev\_alloc\_skb\(pkt->datalen+2\)
b、从硬件中读取数据到skb
c、调用netif\_rx将数据交给协议栈

如果是NAPI兼容的设备驱动，则可以通过poll方式接收数据。
NAPI数据接收的流程为：
a、接收中断来临
b、关闭接收中断
c、以轮询方式接收所有数据包直到收空
d、开启接收中断

NAPI驱动程序各部分的调用关系见图4.1：

![图4.1](4.1.bmp)

# 5. 总结
       Linux网络设备驱动的层次化设计实现了对上层协议提供统一的接口和对硬件设备多样化的适应。驱动开发人员主要的工作集中在设备驱动功能层，在此之前务必要先熟悉net\_device结构体和sk\_buff结构体，并理解NAPI方式的数据包接收。
       值得注意的是在给sk\_buff分配内存空间时一定要注意到接收数据帧的IP头的字节对齐问题，以防IP校验不过。
