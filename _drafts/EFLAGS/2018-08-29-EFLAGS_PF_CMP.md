---
layout: post
title:  "CMP 比较运算引起的 PF 置位"
date:   2018-08-29 17:18:30 +0800
categories: [MMU]
excerpt: CMP 比较运算引起的 PF 置位.
tags:
  - EFLAGS
  - PF
---

## 原理

Intel X86 提供了 CMP 指令，该指令用于两个数的比较运算，CMP 执行完之后会，
如果结果的低八位包含偶数个 1，那么 PF 置位，反之如果结果低八位包含奇数个 1，
那么 CF 清零。(CMP 并不会将结果存储到任一个寄存器，但会根据 CMP 的结果设置 
EFLAGS 寄存器)

## 实践

BiscuitOS 提供了 CMP 相关的实例代码，开发者可以使用如下命令：

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

选择 **PF    Parity flag (bit 2)**.

选择 **CMP   Compare two operands**.

![Menuconfig6](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000186.png)

运行实例代码，使用如下代码：

{% highlight ruby %}
cd BiscuitOS/kernel/linux_1.0.1.2/
make 
make start
{% endhighlight %}

![Menuconfig7](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000118.png)

## 源码分析

源码位置：

{% highlight ruby %}
BiscuitOS/kernel/linux_1.0.1.2/tools/demo/mmu/storage/register/EFLAGS/eflags.c
{% endhighlight %}

![Menuconfig8](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000119.png)

源码如上图，将立即数 4 存储到寄存器 AL 中，将立即数 1 存储到寄存器 BL 中，调
用 CMP 指令进行寄存器 BL 和寄存器 AL 的比较运算，产生的结果并不存储到任何寄
存器，但 EFLAGS 会根据这个结果设置特定标志。如果 产生的结果的低 8 位含有偶数
个 1，则 PF 置位，跳转到分支 PF_S7 并将立即数 1 存储到 DX 寄存器； 如果 产生
结果的低 8 位的值含有奇数个 1，则 PF 清零，跳转到 PF_C7 分支，并将立即数 0 
存储到 DX 中，最后将 DX 寄存器的值存储到 PF 变量里。

#### 运行结果如下：

![Menuconfig9](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000120.png)

#### 运行分析：

将立即数 4 存储到寄存器 AL 中，将立即数 1 存储到寄存器 BL 中，调用 CMP 指令
进行寄存器 BL 和寄存器 AL 的比较运算，产生的结果并不存储到任何寄存器，但 
EFLAGS 会根据这个结果设置特定标志。此时 0x4 CMP 0x1 之后的值为 0x3, 结果 0x3 
的低 8 位含有 2 个 1，所以 PF 置位。PF 置位，跳转到分支 PF_S7 并将立即数 1 
存储到 DX 寄存器，最后将 DX 寄存器的值 1 存储到 PF 变量里。

#### 实践结论：

调用 CMP 进行比较运算时，只要产生的结果的低 8 位含有偶数个 1，PF 就置位；
同理，只要产生的结果的低 8 位含有奇数个 1，PF 就清零。

## 运用场景分析

## 附录

[1. CMP 指令: Intel Architectures Software Developer's Manual: Combined Volumes: 2 Instruction Set Reference,A-Z-- Chapter 3 Instruction Set Reference,A-L: 3.2 Instruction(A-L) : CMP -- Compare two operands](https://software.intel.com/en-us/articles/intel-sdm)

[2. Intel Architectures Software Developer's Manual](https://github.com/BiscuitOS/Documentation/blob/master/Datasheet/Intel-IA32_DevelopmentManual.pdf)
