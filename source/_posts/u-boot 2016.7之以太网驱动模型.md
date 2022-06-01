---
title: u-boot 2016.7之以太网驱动模型
index_img: /img/post/uboot以太网/uboot.jpg
date: 2017-05-18 14:42:11
categories:
- 嵌入式软件开发
tags:
- 嵌入式linux
- uboot
comment: 'valine'
excerpt: u-boot 2016.7引入了设备树（device tree） 和 驱动模型DM（driver model），这为设备的驱动的定义和访问
---

<!--more-->

# 背景
        u-boot 2016.7引入了设备树（device tree） 和 驱动模型DM（driver model），这为设备的驱动的定义和访问接口提供了统一的方法，提高了驱动之间的兼容性和可移植性。具体建议参考/doc/driver-model/README.txt。

        对于u-boot2016.7的以太网络驱动，也属于DM应用的一个范例，此篇着重介绍u-boot2016.7的DM 模型及网络驱动模型的细节，关于设备树（device tree）的相关细节，在后续篇章中进行介绍。

# 1、使能DM功能  
在配置u-boot时，使能DM功能，即在/configs/xxx\_defconfig中定义了：CONFIG\_DM=y；

在配置u-boot时，使能网络设备的DM功能，即在/configs/xxx\_defconfig中定义了：CONFIG\_DM\_NET=y； 

# 2、DM整体框架
      DM 主要有udevice、dirver、uclass、uclass\_dirver四部分组成，其中：

udevice：是指设备对象，类似可以理解为kernel中的device。

dirver：是udevice的驱动，和底层硬件设备通信，并且为上层提供设备操作接口。

uclass：使用相同方式的操作集的device的组。相当于是一种抽象。uclass为使用相同接口的设备提供了统一的接口。

uclass\_dirver：对应uclass的驱动程序。主要提供uclass操作时，如bind和probe  udevice时的一些操作。 

其调用关系见图1：

![图1](1.bmp)

# 3、数据结构和dirver声名
      DM 主要有udevice、dirver、uclass、uclass\_dirver四部分组成，相应与之对应的有四个数据结构struct udevice、struct dirver、struct uclass、struct uclass\_dirver。

对于dirver和uclass\_dirver的声明，u-boot提供了： 
```c
  U_BOOT_DRIVER(xxx_gmac) = {
    .name    = "xxx_gmac",  
    .id    = UCLASS_ETH,  
    .of_match = xxx_gmac_ids,  
    .ofdata_to_platdata = xxx_gmac_ofdata_to_platdata,  
    .probe    = xxx_gmac_probe,  
    .remove    = xxxgemac_remove,  
    .ops    = &xxx_gmac_ops,  
    .priv_auto_alloc_size = sizeof(struct xxx_priv),  
    .platdata_auto_alloc_size = sizeof(struct eth_pdata),  
  };
```

和 

``` c
 UCLASS_DRIVER(eth) = {  
    .name        = "eth",  
    .id        = UCLASS_ETH,  
    .post_bind    = eth_post_bind,  
    .pre_unbind    = eth_pre_unbind,  
    .post_probe    = eth_post_probe,  
    .pre_remove    = eth_pre_remove,  
    .priv_auto_alloc_size = sizeof(struct eth_uclass_priv),  
    .per_device_auto_alloc_size = sizeof(struct eth_device_priv),  
    .flags        = DM_UC_FLAG_SEQ_ALIAS,  
  };
```

# 4、u-boot 2016.7网络驱动模型

      一般以太网的硬件原理框图见图2：
![ 图2](2.bmp)

      以太网的网络驱动模型是DM模型的具体体现，其模型框图见图3：
![图3](3.bmp) 

# 5、u-boot 2016.7网络初始化流程
     u-boot 2016.7的网络初始化流程见图4：    
![图4](4.png) 
                                                               
      其中xxx\_gmac.c和xxx\_phy\_device.c是硬件平台相关的文件，xxx\_gmac.c提供了gmac的对gmac的初始化和对gmac的操作接口及对phy芯片的配置，xxx\_phy\_device.c提供phy芯片的操作接口以及向u-boot注册phy设备。

      当把u-boot移植到新平台或新的板级平台时，若u-boot中没有集成相应的mac控制器驱动或phy芯片驱动时，需手动完成xxx\_gmac.c和xxx\_phy\_device.c并添加。
