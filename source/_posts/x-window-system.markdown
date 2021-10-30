---
title: X Window System简介
excerpt: "1984年，MIT开始研究X Window System，用于在Unix上支持GUI界面，X Window System研究时是作为应用软件进行开发的，而不是操作系统。而且X Window System是作为架构规范进行研究，因此需要人和组织对其进行实现和包装（和linux一样，要有发行版）。1987年，X版本更新到X11，这一版有非常明显的进步，因此后面X Window System也被称为X11，X11有通过网络功能访问GUI界面的功能。"
date: '2011-12-15 00:39:48 +0800'
categories:
- Linux
tags:
- linux
- ubuntu
- x window
---

### 历史

1984年，MIT开始研究X Window System，用于在Unix上支持GUI界面，X Window System研究时是作为应用软件进行开发的，而不是操作系统。而且X Window System是作为架构规范进行研究，因此需要人和组织对其进行实现和包装（和linux一样，要有发行版）。

1987年，X版本更新到X11，这一版有非常明显的进步，因此后面X Window System也被称为X11，X11有通过网络功能访问GUI界面的功能，

1994年，X11R6发布，后来的架构都基于此版本。

1995年发布X11R6.3。

前面提到X Window System是作为架构规范进行研究的，需要有人去实现，而1992年开始的XFree86项目就是这样一个被广泛使用的实现，名称来源于X+Free+X86架构。

2004年的时候，XFree86不在以GPL协议发布，而是另外成立了公司。X.Org基金会就从XFree86的派生出了另一个窗口系统，称为X.Org Server的X Window System。

现在X11最新的版本是X11R7.6，X.Org发布的X Window实现最新版本为1.11。

因此，我们称X、X Window、X11、xf86都是指代X Window System。

### X的作用

X能为GUI环境提供基本的框架：在屏幕上描绘、体现图像与移动程序窗口，同时也受理、运行、及管理电脑与鼠标、键盘的交互程序。不过，X并没有管辖到用户界面的部份（可以理解为界面样式，如gnome和kde就外观完全不同），而是由其他以X为基础的界面实现来负责，也因为如此，以X为基础环境所开发成的视觉样式非常地多；不同的程序可能有完全不同的用户界面。

### X的架构

X Window System由X Server（服务器）和X Client（客户端）两部分组成。X采行C/S的架构模型，由一个X服务器与多个X客户端程序进行通讯，服务器接受对于图形输出（窗口）的请求并反馈用户输入（键盘、鼠标、触摸屏），服务器可能是一个能显示到其他显示系统的应用程序，也可能是控制某个PC的视频输出的系统程序，也可能是个特殊硬件。前面提到的Xorg基金会发布的X.org Server就是一个服务端。

![X Window System](https://farm4.staticflickr.com/3866/14756367460_48d3b7db34.jpg)

Linux系统在`/etc/sysconfig`目录中有键盘、鼠标等硬件的配置文件，但是因为X Server只是一个应用软件，因此他有自己的配置文件。X Server只有在run level 3的时候才会启动，因此只有在这个时候才会去用这些配置文件。

X的一大特点在于“网络透明性”：应用程序（“客户端”应用程序）所运行的机器，不一定是用户本地的机器（显示的“服务器”）。X中所提及的“客户端”和“服务器”等字眼用词也经常与人们一般想定的相反，“服务器”反而是在用户本地端的自有机器上运行，“客户端”是运行与远程服务器上的。客户端和服务器可以在同一台计算机上，也可以不是，或许其架构和操作系统也不同，但都能运行。

### X Window Manager

这是一个特殊的X Client，负责管理所有的X Client。X Client之间是相互平等的，相互也不知道对方的存在。因此需要有个Window Manager对他们进行管理。主要负责：

* 提供许多的配置选项，包括任务栏、背景桌面的设置等等；
* 管理虚拟桌面 (virtual desktop)，Ubuntu里面称为工作区；
* 提供窗口控制参数，如窗口的大小、重叠次序、窗口移动、窗口最小化等。

Gnome，KDE，XFCE就是所谓的X Window Manager
因此，在一台Linux机器上，我们必须要安装Xorg的X Server，才能接收到键盘等交互设备的输入、才能在窗口上绘制图形界面。为了更好的管理图形界面，于是还需要安装Gnome这样的X Window Manager。

### Display Manager

如果已经登录了，在文字界面下输入startx就可以启动X图形系统。但是一般的图形界面linux系统会在系统启动后让你进行登录，这个登录界面就是Display Manager了，主要提供登录的功能，并且载入用户选择的X Window Manager的配置（我们可以在ubuntu启动的时候选择使用Gnome3还是Gnome Classical，或者可能还有Xfce，如果你安装了XFCE的话）。
gdm就是Gnome Display Manager了。

### 启动X Window System的流程

文字界面下使用startx启动X图形系统，startx实际上是一个Shell脚本，其作用是使用当前用户默认的X Server的配置启动X Window System。startx实际上是通过执行xinit来启动X图形系统的。
X Server的启动参数才`/etc/X11/xinit/xserverrc`中，X Client的参数在`/etc/X11/xinit/xinitrc`。具体的参数另行撰文。

### 参考

* <a title="维基百科：X.org" href="http://zh.wikipedia.org/wiki/X.Org_%E6%9C%8D%E5%8A%A1%E5%99%A8" target="_blank">维基百科：X.org</a>
* <a title="维基百科：X Window" href="http://zh.wikipedia.org/wiki/X_Window%E7%B3%BB%E7%B5%B1" target="_blank">维基百科：X Window</a>

