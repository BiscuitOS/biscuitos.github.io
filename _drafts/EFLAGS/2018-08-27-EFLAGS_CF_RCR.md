---
layout: post
title:  "RCR 带进位的循环右移"
date:   2018-08-27 12:07:30 +0800
categories: [MMU]
excerpt: RCR 带进位的循环右移.
tags:
  - EFLAGS
  - CF
---

## 原理

Intel X86 提供了 RCR 指令，该指令用于带进位的循环右移，RCR 将 LSB (最低有效位) 循环移动到 MSB（最高有效位），如果 CF 清零，CF 也一同循环右移到 MSB 中

## 实践

BiscuitOS 提供了 RCL 相关的实例代码，开发者可以使用如下命令：

首先，开发者先准备 BiscuitOS 系统，内核版本 linux 1.0.1.2。开发可以参照文档
构建 BiscuitOS 调试环境：

{% highlight ruby %}
https://biscuitos.github.io/blog/Linux1.0.1.2_ext2fs_Usermanual/
{% endhighlight %}


接着，开发者配置内核，使用如下命令：

{% highlight ruby %}
cd BiscuitOS
make clean
make update
make linux_1_0_1_2_ext2_defconfig
make
cd BiscuitOS/kernel/linux_1.0.1.2/
make clean
make menuconfig
{% endhighlight %}

由于 BiscuitOS 的内核使用 Kbuild 构建起来的，在执行完 make menuconfig 之后，
系统会弹出内核配置的界面，开发者根据如下步骤进行配置：

![Menuconfig](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000003.png)

选择 **kernel hacking**，回车

![Menuconfig1](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000004.png)

选择 **Demo Code for variable subsystem mechanism**, 回车

![Menuconfig2](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000005.png)

选择 **MMU (Memory Manager Unit) on X86 Architecture**, 回车

![Menuconfig3](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000006.png)

选择 **Data storage： Main  Memory, Buffer, Cache**, 回车

![Menuconfig4](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000007.png)

选择 **Register: X86 Common Register mechanism**, 回车

![Menuconfig5](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000008.png)

选择 **EFLAGS： Current status register of processor**, 回车

选择 **CF    Carry Flag(bit 0)**.

选择 **RCR  Rotate through carry right**.

![Menuconfig6](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000091.png)

运行实例代码，使用如下代码：

{% highlight ruby %}
cd BiscuitOS/kernel/linux_1.0.1.2/
make 
make start
{% endhighlight %}

![Menuconfig7](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000088.png)

## 源码分析

源码位置：

{% highlight ruby %}
BiscuitOS/kernel/linux_1.0.1.2/tools/demo/mmu/storage/register/EFLAGS/eflags.c
{% endhighlight %}

![Menuconfig7](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000089.png)

源码如上图，根据 RCR 指令第二个参数的类型分作三种情况

###### 8 位带进位循环右移

将立即数 0x7 存储到 AL 寄存器中，将立即数 0x4 存储到 CL 寄存器，调用 STC 
命令置位 CF 标志位，接着调用 RCR 命令循环右移 AL 寄存器，将 LSB 的值依次
移到 MSB 中， 循环的次数存储在 CL 寄存器中。最终将循环后的数据存储到 AX 
寄存器中的低 8 位中。

###### 16 位带进位循环右移

将立即数 0x8007 存储到 AX 寄存器中，将立即数 0x4 存储到 CL 寄存器，调用 STC 
命令置位 CF 标志位，接着调用 RCR 命令循环右移 AX 寄存器， 将 LSB 的值依次移
到 MSB 中，循环的次数存储在 CL 寄存器中。最终将循环后的数据存储到 AX 寄存器
中。

###### 32 位带进位循环右移

将立即数 0x80000007 存储到 EAX 寄存器中，将立即数 0x4 存储到 CL 寄存器，
调用 STC 命令置位 CF 标志位，接着调用 RCR 命令循环右移 EAX 寄存器，将 LSB 
的值依次移到 MSB 中， 循环的次数存储在 CL 寄存器中。最终将循环后的数据存储
到 EAX 寄存器中。

#### 运行结果如下：

![Menuconfig7](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000090.png)

#### 运行分析：

根据 RCR 指令第二个参数的类型分作三种情况

###### 8 位带进位循环右移

将立即数 0x7 存储到 AL 寄存器中，将立即数 0x4 存储到 CL 寄存器，调用 STC 
命令置位 CF 标志位，接着调用 RCR 命令循环右移 AL 寄存器中的值 0x7， 循环
的次数存储在 CL 寄存器中，此处向右循环 4 次。寄存器 AL 的 LSB 的值为 1，经
过 1 次循环右移之后，LSB 的值存储到 CF 标志位中，CF 标志位中原始值存储到 
MSB 中，此时 AL 寄存器的值为 0xB. 最终变量 AL 寄存器的值为 0xF0，CF 的置位
影响 RCR 循环右移的结果。

###### 16 位带进位循环右移

将立即数 0x8007 存储到 AX 寄存器中，将立即数 0x4 存储到 CL 寄存器，调用 STC 
命令置位 CF 标志位，接着调用 RCR 命令循环右移 AX 寄存器中的值 0x8007， 循环
的次数存储在 CL 寄存器中，此处向右循环 4 次。寄存器 AX 的 LSB 的值为 1，经过 
1 次循环右移之后，LSB 的值存储到 CF 标志位中，CF 标志位中原始值存储到 MSB 中，
此时 AX 寄存器的值为 0xC003. 最终变量 AX 寄存器的值为 0xF800，CF 的置位影响 
RCR 循环右移的结果。

###### 32 位带进位循环右移

将立即数 0x80000007 存储到 EAX 寄存器中，将立即数 0x4 存储到 CL 寄存器，调用 
STC 命令置位 CF 标志位，接着调用 RCR 命令循环右移 EAX 寄存器中的值 0x7， 
循环的次数存储在 CL 寄存器中，此处向右循环 4 次。寄存器 EAX 的 LSB 的值为 
1，经过 1 次循环右移之后，LSB 的值存储到 CF 标志位中，CF 标志位中原始值存储
到 MSB 中，此时 EAX 寄存器的值为 0xC0000003. 最终变量 EAX 寄存器的值为 
0xF8000000，CF 的置位影响 RCR 循环右移的结果。

#### 实践结论：

CF 的置位会影响 RCR 右移的结果。

## 运用场景分析

## 附录

[1. RCR 指令: Intel Architectures Software Developer's Manual: Combined Volumes: 2 Instruction Set Reference,A-Z-- Chapter 4 Instruction Set Reference,M-U: 4.3 Instruction(M-U) : RCL/RCR/ROL/ROR -- Rotate](https://software.intel.com/en-us/articles/intel-sdm)

[2. Intel Architectures Software Developer's Manual](https://github.com/BiscuitOS/Documentation/blob/master/Datasheet/Intel-IA32_DevelopmentManual.pdf)
