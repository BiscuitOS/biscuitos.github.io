---
layout: post
title:  "BiscuitOS User Interface"
date:   2019-05-10 14:55:30 +0800
categories: [HW]
excerpt: BiscuitOS Application().
tags:
  - Userland
---

![DTS](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/IND00000Y.jpg)

> [Github: BiscuitOS HardStack](https://github.com/BiscuitOS/HardStack)
>
> Email: BuddyZhang1 <buddy.zhang@aliyun.com>

# 目录


> - [应用程序移植](#A00)
>
> - [GNU 项目移植](#B00)
>
> - [动态库移植](#C00)
>
> - [静态块移植](#D00)
>
> - [BiscuitOS 项目移植](#E00)
>
> - [游戏移植](#F00)
>
> - [附录](#附录)

-----------------------------------
<span id="A00"></span>

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/IND00000Q.jpg)

## 应用程序移植

开发者有时需要 BiscuitOS 上运行 C/CPP 应用程序，因此需要将应用程序通过交叉
编译之后移植到 BiscuitOS 上运行，目前 BiscuitOS 已经完整支持应用程序的移植
与实践，开发者可以通过参考下面的文章了解更多的详情。

> - [应用程序移植最小实践](#A01)


-----------------------------------
<span id="A01"></span>

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/IND00000K.jpg)

### 应用程序移植最小实践

本节通过一个最简单的例子用于介绍并实践在 BiscuitOS 上运行一个简单的 C
程序，开发者可以参考本节的内容移植自己的应用程序。

> - [前期准备](#A010)
>
> - [项目配置](#A011)
>
> - [生成 Makefile](#A012)
>
> - [获取源码](#A013)
>
> - [源码编译](#A014)
>
> - [程序安装](#A015)
>
> - [生成系统镜像](#A016)
>
> - [程序运行](#A017)

------------------------------------------

##### <span id="A010">前期准备</span>

应用程序移植的核心理念是在主机端使用交叉编译工具，进行交叉编译之后，
将生成的目标文件在目标上运行。因此前期准备包括了一台主机，交叉编译工具，
以及 BiscuitOS。开发者可以在 BiscuitOS linux 5.0 arm32 上进行实践，
因此如果开发者还没有 Linux 5.0 开发环境的，可以参考下面的文档进行搭建：

> [搭建 Linux 5.0 arm32 开发环境](https://biscuitos.github.io/blog/Linux-5.0-arm32-Usermanual/)

-------------------------------------------

#### <span id="A011">项目配置</span>

BiscuitOS 项目中已经包含了默认的配置，开发者可以在 BiscuitOS 中打开
相应的配置，配置完毕后就可以获得对应的文件。因此首先应该基于项目进行
BiscuitOS 配置，步骤如下，首先使用命令启动 Kbuild 配置界面：

{% highlight bash %}
cd BiscuitOS
make linux-5.0-arm32_defconfig
make menuconfig
{% endhighlight %}

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/boot/BOOT000240.png)

Kbuild 编译系统启用之后如上图，对应应用程序，开发者应该选择
"Package" 并按下回车：

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/boot/BOOT000241.png)

此界面是 Package 支持软件的配置界面，开发者将光标移动到 "Demo Application",
按下 "Y" 按键之后再按下回车键，进入 "Demo Application" 配置界面。

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/boot/BOOT000242.png)

上图正是 "Demo" 应用程序的配置界面，"version" 选项代表当前软件的版本。
"tar type" 选项代表应用程序如果是压缩包，则压缩的类型。"url" 选项代表
软件的下载链接。"LDFLAGS" 代表用户可自定义的链接标志，"CFLAGS" 选项代表
用户可自定义的编译标志，"configure" 代表用户自定义的 configure 标志，
"source code list" 代表需要编译的文件。开发者可以使用默认属性，保存并退出，
至此，应用程序的配置已经完成。

------------------------------------------------

#### <span id="A012">生成 Makefile</span>

配置完毕之后，开发者接下来编译并生成 Makefile，使用如下命令：

{% highlight bash %}
cd BiscuitOS
make
{% endhighlight %}

编译完毕之后，输出如下信息：

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/boot/BOOT000243.png)

此时会在 BiscuitOS/output/linux-5.0-arm32/package/ 目录下生成一个目录
"demo-0.11", 进入该目录，可以获得两个文件: Makefile 和 README.md。至此
应用程序的移植前期准备已经结束。

------------------------------------------------

#### <span id="A013">获取源码</span>

进过上面的步骤之后，开发者在 "BiscuitOS/output/linux-5.0-arm32/package/demo-0.11"
目录下获得移植所需的 Makefile，然后开发者接下来需要做的就是下载源码，
使用如下命令：

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/package/demo-0.11
make download
{% endhighlight %}

此时终端输出相关的信息，如下：

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/boot/BOOT000244.png)

此时在当前目录下会获得一个新的目录 "demo-0.11"，里面存储着源码相关的文件，
至此源码下载完毕。

------------------------------------------------

#### <span id="A014">源码编译</span>

获得源码之后，只需简单的命令就可以编译源码，使用如下命令：

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/package/demo-0.11
make
{% endhighlight %}

编译成功输出如下信息：

{% highlight bash %}
"Build .... [OK]"
{% endhighlight %}

------------------------------------------------

#### <span id="A015">程序安装</span>

程序编译成功之后，需要将可执行文件安装到 BiscuitOS rootfs 里，
只需简单的命令就可以实现，使用如下命令：

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/package/demo-0.11
make install
{% endhighlight %}

安装成功输出如下信息：

{% highlight bash %}
"Install .... [OK]"
{% endhighlight %}

------------------------------------------------

#### <span id="A016">生成系统镜像</span>

程序安装成功之后，接下来需要将新的软件更新到 BiscuitOS 使用
的镜像里，只需简单的命令就可以实现，使用如下命令：

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/package/demo-0.11
make pack
{% endhighlight %}

打包成功输出如下信息：

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/boot/BOOT000245.png)

------------------------------------------------

#### <span id="A017">程序运行</span>

程序安装成功之后，接下来就是在 BiscuitOS 中运行程序，
只需简单的命令就可以实现，使用如下命令：

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/
./RunBiscuitOS.sh start
{% endhighlight %}

程序运行成功输出如下信息：

![](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/boot/BOOT000246.png)







> [BiscuitOS 配置介绍](https://github.com/BiscuitOS/HardStack/blob/master/GAME/snake/snake.c)

将源码存储到 BiscuitOS/output/linux-5.0-arm32/package/snake/ 目录下，命名为
snake.c, 由于这个 C 文件比较简单，可以直接使用交叉编译工具直接编译。使用如下命令：

{% highlight ruby %}
cd BiscuitOS/output/linux-5.0-arm32/package/
mkdir -p snake
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/GAME/snake/snake.c
BiscuitOS/output/linux-5.0-aarch/aarch64-linux-gnu/aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc -lncurses snake.c -o snake
{% endhighlight %}

如果开发者的程序不使用任何多余的动态库，执行上面的命令之后，就会生成 ARM32 上运行
的可执行程序；如果开发者的应用程序依赖了特定的动态库，那么请参考下面内容：

> [snake 依赖 ncurses 库移植方法](#APP1)

#### <span id="安装程序">安装程序</span>

编译完生成可执行文件之后，接下来是将应用程序安装到 rootfs 里面，rootfs 位于
BiscuitOS/output/linux-5.0-arm32/rootfs/rootfs/ 目录下，开发者应该将应用程序
安装到 rootfs 目录下的 usr/bin/ 目录里，如本例子中生成的 snake，如下：

{% highlight ruby %}
cd BiscuitOS/output/linux-5.0-arm32/package/snake/
cp snake BiscuitOS/output/linux-5.0-arm32/rootfs/rootfs/usr/bin
{% endhighlight %}

#### <span id="rootfs 打包">rootfs 打包</span>

开发者将应用程序安装到 rootfs 之后，需要更新 rootfs，这步称为 rootfs 打包，使用如下
命令即可完成：

{% highlight ruby %}
cd BiscuitOS/output/linux-5.0-arm32/
./RunQemuKernel.sh pack
{% endhighlight %}

只要更新了 rootfs 都需要执行上面的命令。

#### <span id="运行程序">运行程序</span>

以上步骤执行完之后，最后一步就是运行应用程序，例如本例中的贪吃蛇，请参考如下步骤：

{% highlight ruby %}
cd BiscuitOS/output/linux-5.0-arm32/
./RunQemuKernel.sh start
{% endhighlight %}

执行上面的命令之后，BiscuitOS 就能正常启动，当进入用户终端之后，由于贪吃蛇的运行
需要设置终端，其他应用程序无需这一步。如下：

{% highlight bash %}
Freeing unused kernel memory: 1024K
Run /linuxrc as init process
 ____  _                _ _    ___  ____
| __ )(_)___  ___ _   _(_) |_ / _ \/ ___|
|  _ \| / __|/ __| | | | | __| | | \___ \
| |_) | \__ \ (__| |_| | | |_| |_| |___) |
|____/|_|___/\___|\__,_|_|\__|\___/|____/
Welcome to BiscuitOS

Please press Enter to activate this console.
/ #
/ # export TERM=vt102
/ # export TERMINFO=/usr/share/terminfo
/ # clear
/ # s
{% endhighlight %}

snake 运行效果如下：

![DTS](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/boot/BOOT000065.png)

-----------------------------------
<span id="APP1"></span>

![DTS](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/IND00000L.jpg)

# 移植动态库

开发者在移植应用程序的过程中往往会遇到应用程序依赖特定的动态库，但该动态库交叉工具
并未提供，因此针对这种情况，开发者需要交叉编译并移植动态库，可以参照本节完成。例如
本例中 snake 依赖了 ncurses 库。

#### <span id="动态库源码">动态库源码</span>

动态库移植的第一步是获取动态库的源码，大多数库的源码位于

> [http://ftp.gnu.org/pub/gnu/](http://ftp.gnu.org/pub/gnu/)

例如贪吃蛇需要使用 ncurses 的库，可以从该 ftp 服务器上获得 ncurses 对应的源码，
例如本例子使用的 ncurses-6.1.tar.gz, 将下载好的源码放到 BiscuitOS/output/linux-5.0-arm32/package/ncurses 目录下，并解压，使用如下命令：

{% highlight c %}
cd BiscuitOS/output/linux-5.0-arm32/package/
mkdir -p ncurses
mv ~/Download/ncurses-6.1.tar.gz ncurses
cd ncurses
tar xf ncurses-6.1.tar.gz
cd ncurses-6.1
{% endhighlight %}

#### <span id="配置动态库">配置动态库</span>

编译动态库之前，首先配置动态库，在编译之前，需要将交叉工具导入到全局环境变量里，
使用如下命令：

{% highlight c %}
export PATH=$PATH:BiscuitOS/output/linux-5.0-arm32/arm-linux-gnueabi/arm-linux-gnueabi/bin
{% endhighlight %}

通过执行上面的命令，arm-linux-gnueabi- 交叉编译工具就可以直接使用，接下来就是配置
动态库，使用如下命令：

{% highlight c %}
./configure --prefix=BiscuitOS/output/linux-5.0-arm32/rootfs/rootfs/usr/ --host=arm-linux-gnueabi --with-shared --disable-stripping
{% endhighlight %}

###### --prefix

--prefix 参数用于指定安装目录，这里直接安装到 rootfs 中。

###### --host

--host 参数用于指定交叉编译工具，这里使用 arm-linux-gnueabi

###### --with-shared

--with-shared 参数用于说明编译成动态库

###### --disable-stripping

--disable-stripping 参数用于提示安装的时候。不执行 strip 功能。

成功运行上面的命令之后，动态库源码就已经配置好。

#### <span id="编译动态库">编译动态库</span>

配置好动态库之后，就是编译动态库，命令很简单，如下：

{% highlight c %}
make
{% endhighlight %}

如果没有遇到错误，那么动态库就编译成功。

#### <span id="安装动态库">安装动态库</span>

最后一步就是安装动态库，只需执行简单的命令，动态库就会被安装到指定目录，如下：

{% highlight c %}
make install
{% endhighlight %}

至此，动态库的交叉编译和移植已经完成，接下来就是使用动态库。

#### <span id="使用动态库">使用动态库</span>

动态库的使用分为主机上交叉编译时候使用，以及目标机上使用动态库。例如 snake 的例子中，
引用了 ncurses 的库，原始命令为：

{% highlight ruby %}
BiscuitOS/output/linux-5.0-aarch/aarch64-linux-gnu/aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc -lncurses snake.c -o snake
{% endhighlight %}

现在要在主机上交叉编译 snake 时候，需要重新指定库的位置以及头文件的位置，可以参考如下命令，
从上面的动态库安装可以知道，动态库安装在 BiscuitOS/output/linux-5.0-arm32/rootfs/rootfs/usr/
目录下，因此这里需要使用 -L 指定动态库的路径，以及使用 -I 指定动态库使用的头文件，
如下：

{% highlight c %}
BiscuitOS/output/linux-5.0-arm32/arm-linux-gnueabi/arm-linux-gnueabi/bin/arm-linux-gnueabi-gcc  snake.c -LBiscuitOS/output/linux-5.0-arm32/rootfs/rootfs/usr/lib/ -lncurses -IBiscuitOS/output/linux-5.0-arm32/rootfs/rootfs/usr/include/ncurses -IBiscuitOS/output/linux-5.0-arm32/rootfs/rootfs/usr/include/ -o snake
{% endhighlight %}

通过上面的命令，就可以在主机上交叉编译的时候使用动态库。

-----------------------------------------------

# <span id="附录">附录</span>

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
