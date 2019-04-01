---
layout:             post
title:              "perf"
date:               2019-04-01 18:55:30 +0800
categories:         [MMU]
excerpt:            perf tools.
tags:
  - MMU
---

> [GitHub ASM code: .endm](https://github.com/BiscuitOS/HardStack/tree/master/Language/Assembly/ARM-GNU-Assembly/Instruction/%5B.endm%5D)
>
> Email: BuddyZhang1 <buddy.zhang@aliyun.com>

# 目录

> - [工具原理](#工具原理)
>
> - [工具安装](#工具安装)
>
> - [工具使用](#工具使用)
>
> - [附录](#附录)

--------------------------------------------------------------
<span id="工具原理"></span>

![MMU](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/IND00000B.jpg)

# 工具原理

Perf 是 Linux kernel 自带的系统性能优化工具，Perf 显现出它强大的实力，足以与
目前 Linux 流行的 OProfile 相媲美了。

Perf 的优势在于与 Linux Kernel 的紧密结合，它可以最先应用到加入 Kernel 的
new feature。而像 OProfile, GProf 等通常会 “慢一拍”。Perf 的基本原理跟
OProfile 等类似，也是在 CPU 的 PMU registers 中 Get/Set performance 
counters 来获得诸如 instructions executed, cache-missed suffered, branches 
mispredicted 等信息。Linux kernel 对这些 registers 进行了一系列抽象，所以你
可以按进程，按 CPU 或者按 counter group 等不同类别来查看 Sample 信息.

-------------------------------------------------------------
<span id="工具安装"></span>

![MMU](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/IND00000A.jpg)

本教程安装基于 BiscuitOS 制作的 Linux 5.0 系统，其他平台参照安装。如需要安装基于 BiscuitOS
的 Linux 5.0 开发环境，请参考下面文章：

> [Linux 5.0 arm32 开发环境搭建教程](https://biscuitos.github.io/blog/Linux-5.0-arm32-Usermanual/)

##### 获取源码

首先从 blktrace 的站点获取相应的源码，源码地址如下：

> [blktrack: http://brick.kernel.dk/snaps/](http://brick.kernel.dk/snaps/)

从 blktrack 站点上根据需求下载一个版本，例如本教程中选择下载 "blktrace-1.2.0.tar.gz"。
将下载好的源码压缩包放到 BiscuitOS 项目的 dl 目录下，例如使用如下命令：

{% highlight bash %}
cp ~/Download/blktrace-1.2.0.tar.gz BiscuitOS/dl
{% endhighlight %}

##### 解压源码

由于本教程是基于 BiscuitOS 制作的 Linux 5.0 开发环境，因此参考如下命令进行操作：

{% highlight bash %}
mkdir -p BiscuitOS/output/linux-5.0-arm32/package/blktrace
cp -rf BiscuitOS/dl/blktrace-1.2.0.tar.gz  BiscuitOS/output/linux-5.0-arm32/package/blktrace
cd BiscuitOS/output/linux-5.0-arm32/package/blktrace
tar xf blktrace-1.2.0.tar.gz
cd blktrace-1.2.0
make clean
{% endhighlight %}

##### 编译源码

由于只需要 blktrace 和 blkparse 两个工具，开发者可以参考一下命令进行编译：

{% highlight bash %}
make CC=BiscuitOS/output/linux-5.0-arm32/arm-linux-gnueabi/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc blktrace blkparse
{% endhighlight %}

这里由于工具运行在 arm32 平台上，所以需要使用交叉编译工具，开发者根据实际情况进行调整。

##### 工具安装

由于本教程是基于 BiscuitOS 制作的 Linux 5.0 开发环境，因此参考如下命令进行行安装：

{% highlight bash %}
cp -rf blktrace blkparse BiscuitOS/output/linux-5.0-arm32/rootfs/rootfs/usr/bin
{% endhighlight %}

##### 更新 rootfs

接着更新 rootfs，并在 BiscuitOS 中使用这个工具，开发者根据实际情况进行更新，BiscuitOS
可以参考如下命令：

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/
./RunQemuKernel.sh pack
{% endhighlight %}

##### 运行工具


-------------------------------------------------------------
<span id="工具使用"></span>

![MMU](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/IND00000A.jpg)

### 统计事件 stat: statistics

{% highlight bash %}
# CPU counter statistics for the specified command:
perf stat command
# Detailed CPU counter statistics (includes extras) for the specified command:
perf stat -d command
# CPU counter statistics for the specified PID, until Ctrl-C:
perf stat -p PID
# CPU counter statistics for the entire system, for 5 seconds:
perf stat -a sleep 5
# Various basic CPU statistics, system wide, for 10 seconds:
perf stat -e cycles,instructions,cache-references,cache-misses,bus-cycles -a sleep 10
{% endhighlight %}

### 剖析 Profiling

{% highlight bash %}
# Sample on-CPU functions for the specified command, at 99 Hertz:
perf record -F 99 command
# Sample on-CPU functions for the specified PID, at 99 Hertz, until Ctrl-C:
perf record -F 99 -p PID
# Sample on-CPU functions for the specified PID, at 99 Hertz, for 10 seconds:
perf record -F 99 -p PID sleep 10
# Sample CPU stack traces (via frame pointers) for the specified PID, at 99 Hertz, for 10 seconds:
perf record -F 99 -p PID -g -- sleep 10
常用参数
	-e：Select the PMU event.
	-a：System-wide collection from all CPUs.
	-p：Record events on existing process ID (comma separated list).
	-A：Append to the output file to do incremental profiling.
	-f：Overwrite existing data file.
	-o：Output file name.
	-g：Do call-graph (stack chain/backtrace) recording.
	-C：Collect samples only on the list of CPUs provided.
{% endhighlight %}

### Static Tracing

{% highlight bash %}
# Trace new processes, until Ctrl-C:
perf record -e sched:sched_process_exec -a
# Trace all context-switches, until Ctrl-C:
perf record -e context-switches -a
# Trace context-switches via sched tracepoint, until Ctrl-C:
perf record -e sched:sched_switch -a
# Trace all context-switches with stack traces, until Ctrl-C:
perf record -e context-switches -ag
# Trace all context-switches with stack traces, for 10 seconds:
perf record -e context-switches -ag -- sleep 10
{% endhighlight %}

### Dynamic Tracing

{% highlight bash %}
# Add a tracepoint for the kernel tcp_sendmsg() function entry ("--add" is optional):
perf probe --add tcp_sendmsg
# Remove the tcp_sendmsg() tracepoint (or use "--del"):
perf probe -d tcp_sendmsg
# Add a tracepoint for the kernel tcp_sendmsg() function return:
perf probe 'tcp_sendmsg%return'
# Show available variables for the kernel tcp_sendmsg() function (needs debuginfo):
perf probe -V tcp_sendmsg
# Show available variables for the kernel tcp_sendmsg() function, plus external vars (needs debuginfo):
perf probe -V tcp_sendmsg --externs
{% endhighlight %}

### Mixed

{% highlight bash %}
# Sample stacks at 99 Hertz, and, context switches:
perf record -F99 -e cpu-clock -e cs -a -g 
# Sample stacks to 2 levels deep, and, context switch stacks to 5 levels (needs 4.8):
perf record -F99 -e cpu-clock/max-stack=2/ -e cs/max-stack=5/ -a -g 
{% endhighlight %}

### Reporting

{% highlight bash %}
# Show perf.data in an ncurses browser (TUI) if possible:
perf report
# Show perf.data with a column for sample count:
perf report -n
# Show perf.data as a text report, with data coalesced and percentages:
perf report --stdio
# Report, with stacks in folded format: one line per stack (needs 4.4):
perf report --stdio -n -g folded
# List all events from perf.data:
perf script
# List all perf.data events, with data header (newer kernels; was previously default):
perf script --header
{% endhighlight %}

-----------------------------------------------

# <span id="附录">附录</span>

> [The GNU Assembler](http://tigcc.ticalc.org/doc/gnuasm.html)
>
> [Debugging on ARM Boot Stage](https://biscuitos.github.io/blog/BOOTASM-debuggingTools/#header)
>
> [BiscuitOS Home](https://biscuitos.github.io/)
>
> [BiscuitOS Driver](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/)
>
> [BiscuitOS Kernel Build](https://biscuitos.github.io/blog/Kernel_Build/)
>
> [Linux Kernel](https://www.kernel.org/)
>
> [Bootlin: Elixir Cross Referencer](https://elixir.bootlin.com/linux/latest/source)
>
> [搭建高效的 Linux 开发环境](https://biscuitos.github.io/blog/Linux-debug-tools/)

## 赞赏一下吧 🙂

![MMU](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/HAB000036.jpg)