---
title: Linux字符设备驱动
date: 2022-03-20 16:47:05
index_img: /img/post/linux驱动/char.png
categories:
- Linux Driver
tags:
- 字符设备
comment: 'valine'
excerpt: 字符设备：字符设备是指在I/O传输过程中以字符为单位进行传输的设备，是面向流的设备，常见的字符设备有鼠标、键盘、串口、控制台和LED等。
---
# 一、字符设备
<div class="markdown-body">
&emsp;&emsp;字符设备是指在I/O传输过程中以字符为单位进行传输的设备，是面向流的设备，常见的字符设备有鼠标、键盘、串口、控制台和LED等。</br>
&emsp;&emsp;字符设备会在linux系统的/dev目录下对应一个设备文件。字符设备可以使用与普通文件相同的文件操作命令对字符设备文件进行操作，例如打开、关闭、读、写等。</div>

# 二、字符设备驱动模型
<div class="markdown-body">
&emsp;&emsp;Linux系统下字符设备驱动主要有2部分：用户态设备文件和内核态驱动中文件操作函数（见图2.1）。用户态的进程想和硬件设备交互时，首先要通过系统调用操作（read()、write()、ioctl()等）字符设备文件，字符设备驱动然后调用file_operations 中注册的操作函数和硬件交互。</div>

#### 
![图2.1](图2.1.png)
 
# 三、字符设备驱动构建
<div class="markdown-body">
&emsp;&emsp;Linux系统下构建字符设备驱动主要有部分要实现：驱动初始化、设备操作、驱动注销（见图3.1)。</br>
a. 驱动初始化：</br>
需要完成cdev的分配、cdev初始化、注册cdev及硬件设备初始化。</br>
b. 设备操作：</br>
需要填充 struct file_operations 结构体中断的操作函数，实现 struct file_operations 结构体中</br>
的read()、write()和ioctl()等函数是驱动设计的主体工作。</br>
c. 驱动注销：<br>
即释放设备号、删除、注销cdev。</div>

#### 
![图3.1](图3.1.png)
 
## 3.1 struct cdev

<include/linux/cdev.h> 

```c
struct cdev {   
　　struct kobject kobj;                  //内嵌的内核对象.  
　　struct module *owner;                 //该字符设备所在的内核模块（所有者）的对象指针，一般为THIS_MODULE主要用于模块计数  
　　const struct file_operations *ops;    //该结构描述了字符设备所能实现的操作集（打开、关闭、读/写、...），是极为关键的一个结构体
　　struct list_head list;                //用来将已经向内核注册的所有字符设备形成链表
　　dev_t dev;                            //字符设备的设备号，由主设备号和次设备号构成（如果是一次申请多个设备号，此设备号为第一个）
　　unsigned int count;                   //隶属于同一主设备号的次设备号的个数
　　...
};  
```

## 3.2 cdev初始化
<div class="markdown-body">
&emsp;&emsp;对于struct cdev，内核在头文件linux/cdev.h中提供了相应操作接口：</div>

### 3.2.1 为cdev分配内存
```c
struct cdev *cdev_alloc(void);　　
/* 返回值：
　　　　成功 cdev 对象首地址
　　　　失败：NULL */
```

### 3.2.2 初始化cdev
<div class="markdown-body">
&emsp;&emsp;初始化cdev的成员变量，并建立cdev和file_operations之间的关联。
```c
void cdev_init(struct cdev *p, const struct file_operations *p);　　
/* 参数：
　　　　struct cdev *p - 被初始化的 cdev对象
　　　　const struct file_operations *fops - 字符设备操作方法集 */
```
</div>　　

### 3.2.3 注册cdev设备对象
<div class="markdown-body">
&emsp;&emsp;注册cdev设备对象，添加到系统字符设备列表中。</div>
```c
int cdev_add(struct cdev *p, dev_t dev, unsigned count);
/* 参数：
　　　　struct cdev *p - 被注册的cdev对象
　　　　dev_t dev - 设备的第一个设备号
　　　　unsigned - 这个设备连续的次设备号数量
   返回值：
　　　　成功：0
　　　　失败：负数（绝对值是错误码）*/
```

### 3.2.4 设备号申请
<div class="markdown-body">
&emsp;&emsp;一个字符设备有一个主设备号和一个次设备号。主设备号用来标识与设备文件相连的驱动程序，用来反映设备类型。次设备号被驱动程序用来辨别操作的是哪个设备，用来区分同类型的设备。</br>
&emsp;&emsp;linux内核中，设备号用dev_t来描述：
```c
typedef u_long dev_t;　　// 在32位机中是4个字节，高12位表示主设备号，低20位表示次设备号。 
```
&emsp;&emsp;内核在头文件 linux/fs.h中提供了几个方便操作的宏定义来实现dev_t： </div>
```c
#define MAJOR(dev)    ((unsigned int) ((dev) >> MINORBITS))　　// 从设备号中提取主设备号
#define MINOR(dev)    ((unsigned int) ((dev) & MINORMASK))　　// 从设备号中提取次设备号
#define MKDEV(ma,mi)    (((ma) << MINORBITS) | (mi))</span>　　// 将主、次设备号拼凑为设备号
/* 只是拼凑设备号,并未注册到系统中，若要使用需要竞态申请 */
```

#### 3.2.4.1 - 静态申请设备号
```c
int register_chrdev_region(dev_t from, unsigned count, const char *name);
/* 参数：
　　　　dev_t from - 要申请的设备号（起始）
　　　　unsigned count - 要申请的设备号数量
　　　　const char *name - 设备名
   返回值：
　　　　成功：0
　　　　失败：负数（绝对值是错误码）*/
```

#### 3.2.4.2 - 动态分配设备号
```c
int alloc_chrdev_region(dev_t *dev, unsigned baseminor, unsigned count, const char *name);
/* 参数：
　　　　dev_t *dev - 用于保存分配到的第一个设备号（起始）
　　　　unsigned baseminor - 起始次设备号
　　　　unsigned count - 要分配设备号的数量
　　　　const char *name - 设备名
   返回值：
　　　　成功：0
　　　　失败：负数（绝对值是错误码）*/
```

#### 3.2.4.3 创建设备文件
<div class="markdown-body">
&emsp;&emsp;利用cat /proc/devices查看申请到的设备名，设备号。</div>

##### a. 使用mknod手工创建：
<div class="markdown-body">
&emsp;&emsp;mknod filename type major minor </div>

##### b. 自动创建设备节点:
<div class="markdown-body">
&emsp;&emsp;利用udev（mdev）来实现设备文件的自动创建，首先应保证支持udev（mdev），由busybox配置。在驱动初始化代码里调用class_create为该设备创建一个class，再为每个设备调用device_create创建对应的设备。
</div> 

## 3.3 file_operations *fops
<div class="markdown-body">
&emsp;&emsp;Linux下一切皆文件，字符设备也是抽象成设备文件，struct cdev 中的file_operations结构体中的成员函数是字符设备程序设计的主题内容，这些函数实际会在用户层程序进行Linux的open()、close()、write()、read()等系统调用时最终被调用。
文件的操作接口结构：</div>
```c
struct file_operations {
　　struct module *owner;　　
　　　　/* 模块拥有者，一般为 THIS——MODULE */
　　ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);　　
　　　　/* 从设备中读取数据，成功时返回读取的字节数，出错返回负值（绝对值是错误码） */
　　ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);　　　
　　　　/* 向设备发送数据，成功时该函数返回写入字节数。若为被实现，用户调层用write()时系统将返回 -EINVAL*/
　　int (*mmap) (struct file *, struct vm_area_struct *);　　
　　　　/* 将设备内存映射内核空间进程内存中，若未实现，用户层调用 mmap()系统将返回 -ENODEV */
　　long (*unlocked_ioctl)(struct file *filp, unsigned int cmd, unsigned long arg);　　
　　　　/* 提供设备相关控制命令（读写设备参数、状态，控制设备进行读写...）的实现，当调用成功时返回一个非负值 */
　　int (*open) (struct inode *, struct file *);　　
　　　　/* 打开设备 */
　　int (*release) (struct inode *, struct file *);　　
　　　　/* 关闭设备 */
　　int (*flush) (struct file *, fl_owner_t id);　　
　　　　/* 刷新设备 */
　　loff_t (*llseek) (struct file *, loff_t, int);　　
　　　　/* 用来修改文件读写位置，并将新位置返回，出错时返回一个负值 */
　　int (*fasync) (int, struct file *, int);　　
　　　　/* 通知设备 FASYNC 标志发生变化 */
　　unsigned int (*poll) (struct file *, struct poll_table_struct *);　　
　　　　/* POLL机制，用于询问设备是否可以被非阻塞地立即读写。当询问的条件未被触发时，用户空间进行select()和poll()系统调用将引起进程阻塞 */
　　...
};
```

## 3.4 cdev注销 

### 3.4.1 释放设备号
```c
void unregister_chrdev_region(dev_t from, unsigned count);
/* 参数：
　　　　dev_t from - 要释放的第一个设备号（起始）
　　　　unsigned count - 要释放的次设备号数量 */
```

### 3.4.2 注销cdev
将cdev对象从系统中移除
```c
void cdev_del(struct cdev *p);
/*参数：　
　　　　struct cdev *p - 要移除的cdev对象 */
```

### 3.4.3 释放cdev内存
```c
void cdev_put(struct cdev *p);
/*参数：
　　　　struct cdev *p - 要移除的cdev对象 */
```
