---
layout: post
title:  "AND 与运算引起的 SF 置位"
date:   2018-10-17 08:06:30 +0800
categories: [MMU]
excerpt: AND 与运算引起的 SF 置位.
tags:
  - EFLAGS
  - SF
---

## 原理

Intel X86 提供了 AND 指令，该指令用于两个数的二进制与运算，结果存储在 
AL/AX/EAX 寄存器中，如果运算结果 AL 寄存器的值的最高有效位为 1，则 SF 置
位；如果运算结果 AL 寄存器的值最高有效位为零，则 SF 置位。

## 实践

BiscuitOS 提供了 AND 相关的实例代码，开发者可以使用如下命令：

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

选择 **SF  Sing flag (bit 7)**.

选择 **AND Logical AND**

![Menuconfig6](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000371.png)

运行实例代码，使用如下代码：

{% highlight ruby %}
cd BiscuitOS/kernel/linux_1.0.1.2/
make 
make start
{% endhighlight %}

![Menuconfig7](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000372.png)

## 源码分析

源码位置：

{% highlight ruby %}
BiscuitOS/kernel/linux_1.0.1.2/tools/demo/mmu/storage/register/EFLAGS/eflags.c
{% endhighlight %}

![Menuconfig8](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000373.png)

AND 用于两个操作数的逻辑与运算。首先将立即数 0x81 存储到 AX 寄存器中，在将立
即数 0x80 存储到寄存器 BX 中，最后将立即数 0 存储到 DL 寄存器。接着调用 AND 
指令进行 BL 寄存器和 AL 寄存器的逻辑与运算，运算的结果存储到 AL 寄存器中，如
果运算结果的最高有效位为 1，则 EFLAGS 寄存器的 SF 置位并且 SETS 指令会将 DL 
寄存器中值设置为 1.最后将 DL 寄存器中的值存储到 SF 变量里，并将 AX 寄存器中
的值存储到 AX 变量里。

#### 运行结果如下：

![Menuconfig9](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000374.png)

#### 运行分析：

立即数 0x81 与 0x80 逻辑与之后的结果为 0x80， 存储到 AL 寄存器中，此时运算结
果的最高有效位为 1，所以 EFLAGS 寄存器的 SF 标志位置位。此时 SETS 指令将 DL 
寄存器中的值设置为 1.最后将 DL 寄存器中的值 1 存储到 SF 变量里，并将 AX 寄存
器的值 0x80 存储到 AX 变量里。

#### 实践结论：

调用 AND 进行加法运算时，无论是 8/16/32 位与运算，只要产生的结果为零，ZF 就
置位，反之清零。

## 运用场景分析

## 附录

[1. AND 指令: Intel Architectures Software Developer's Manual: Combined Volumes: 2 Instruction Set Reference,A-Z-- Chapter 3 Instruction Set Reference,A-L: 3.2 Instruction(A-L) : AND -- Logical AND](https://software.intel.com/en-us/articles/intel-sdm)

[2. Intel Architectures Software Developer's Manual](https://github.com/BiscuitOS/Documentation/blob/master/Datasheet/Intel-IA32_DevelopmentManual.pdf)
