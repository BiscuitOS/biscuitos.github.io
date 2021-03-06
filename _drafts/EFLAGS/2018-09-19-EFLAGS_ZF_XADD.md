---
layout: post
title:  "XADD 交换加法操作引起 ZF 置位"
date:   2018-09-19 20:22:30 +0800
categories: [MMU]
excerpt: XADD 交换加法操作引起 ZF 置位.
tags:
  - EFLAGS
  - ZF
---

## 原理

Intel X86 提供了 XADD 指令，该指令用于将两个操作数先交换，然后再相加。相加之
后的结果为零，则引起 ZF 置位；反之 ZF 清零。

## 实践

BiscuitOS 提供了 XADD 相关的实例代码，开发者可以使用如下命令：

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

选择 **ZF Zero flag (bit 6)**.

选择 **XADD   Exchanged and add**

![Menuconfig6](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000346.png)

运行实例代码，使用如下代码：

{% highlight ruby %}
cd BiscuitOS/kernel/linux_1.0.1.2/
make 
make start
{% endhighlight %}

![Menuconfig7](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000347.png)

## 源码分析

源码位置：

{% highlight ruby %}
BiscuitOS/kernel/linux_1.0.1.2/tools/demo/mmu/storage/register/EFLAGS/eflags.c
{% endhighlight %}

![Menuconfig8](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000348.png)

源码如上图，将变量 CX 中的值存储到 AX 寄存器中，将变量 DX 中的值存储到 BX 寄
存器，然后调用 XADD 指令，将 AL 寄存器中的值与 BL 寄存器中的值先交换位置，然
后相加，相加之后的和存储到 AL 寄存器中。如果 AL 寄存器中的值为零，则 ZF 置位，
那么跳转到 ZF_S02 分支中，并将立即数 1 存储到 DX 寄存器中；反之如果 AL 寄存
器中的值不为零，则 ZF 清零，那么跳转到 ZF_C02 分支中执行，并将立即数 0 存储
到 DX 寄存器中。最后将 DX 寄存器中的值存储到 ZF 变量中，并将 AX 寄存器中的值
存储到 AX 变量里。

#### 运行结果如下：

![Menuconfig9](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000349.png)

#### 运行分析：

将立即数 0x1 存储到 AX ，将立即数 -1 存储到 BX 寄存器中，两个数交换位置之后，
AX 寄存器的值为 -1， BX 寄存器的值为 1，然后相加，相加的和为零，存储到 AL 寄
存器中。AL 寄存器的值为零，所以 ZF 置位，那么程序跳转到 ZF_S02 分支继续执行，
立即数存储到 DX 寄存器中。最后 DX 寄存器中的值 1 存储到 ZF 变量里，并且 AX 寄
存器的值存储到 AX 变量里。

#### 实践结论：

XADD 指令执行的结果为零时会引起 ZF 置位

## 运用场景分析

## 附录

[1. XADD 指令: Intel Architectures Software Developer's Manual: Combined Volumes: 2 Instruction Set Reference,V-Z-- Chapter 5 Instruction Set Reference,V-Z: 5.2 Instruction(V-Z) : XADD  Exchanged and add](https://software.intel.com/en-us/articles/intel-sdm)

[2. Intel Architectures Software Developer's Manual](https://github.com/BiscuitOS/Documentation/blob/master/Datasheet/Intel-IA32_DevelopmentManual.pdf)
