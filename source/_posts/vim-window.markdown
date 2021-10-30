---
layout: post
title: Vim的Window/窗口
excerpt: |+
  Vim实际上是一个多窗口的编辑器，启动Vim后实际上默认只打开了一个窗口，如果需要在编辑当前文件的时候参考其他的文件，就需要使用到多窗口了。Vim窗口中打开的是一个Vim缓冲区（Buffer），请参考我的另一篇博客<a href="http://zhouliang.pro/2012/06/28/vim-buffer/">Vim的Buffer/缓冲区</a>，实际上也可以说不谈窗口而只谈缓冲区是不负责任的，因此就有了这篇小短文。额外提一下，Vim中的标签页（Tab）可以包含多个窗口，不像我们常用的<a href="http://www.eclipse.org">Eclipse</a>、<a href="http://macromates.com/">TextMate</a>或者<a href="http://www.sublimetext.com/">Sublime</a>等编辑器，一个标签页只是对应一个窗口。学习一个工具，也要学习其背后的设计原理和设计哲学。
date: '2014-01-13 01:09:51 +0800'
categories:
- Linux
tags:
- Linux
- Vim
---
Vim实际上是一个**多窗口**的编辑器，启动Vim后实际上默认只打开了一个窗口，如果需要在编辑当前文件的时候参考其他的文件，就需要使用到多窗口了。

Vim窗口中打开的是一个Vim缓冲区（Buffer），请参考我的另一篇博客<a href="http://zhouliang.pro/2012/06/28/vim-buffer/">Vim的Buffer/缓冲区</a>，实际上也可以说不谈**窗口**而只谈**缓冲区**

是不负责任的，因此就有了这篇小短文。额外提一下，Vim中的标签页（Tab）可以包含多个窗口，不像我们常用的<a href="http://www.eclipse.org">Eclipse</a>、<a href="http://macromates.com/">TextMate</a>或者<a href="http://www.sublimetext.com/">Sublime</a>等编辑器，一个标签页只是对应一个窗口。

> 学习一个工具，也要学习其背后的设计原理和设计哲学。

## 打开/关闭窗口

打开文件的时候，可以用 `vim file1 file2 file3` 命令一次性打开多个文件，不过Vim默认只会打开一个窗口，显示的第一个文件`file1`对应的Buffer（注意这句话的表达，显示的不是文件`file1`本身，而是`file1`文件对应的Buffer）。其他两个文件对应的Buffer处于_hidden_的状态
如果加上`-o`或者`-O`参数，Vim就为每个Buffer打开一个窗口，如果是`-o`则窗口会从上到下排列，如果是`-O`则窗口从左到右排列。

    vim -o file1 file2 file3

打开的窗口排列方式如下图所示，读者可以自行尝试大写的`-O`参数。

<a href="https://www.flickr.com/photos/zhlwish/14022883140/" title="Flickr 上 zhlwish 的 vim-window-vertical"><img src="https://farm6.staticflickr.com/5039/14022883140_34ba4f36f0_o.png" width="641" height="382" alt="vim-window-vertical"></a>

也可以在Vim启动后自行创建新窗口，`:new`命令和`:split`命令都可以实现这样的功能，其区别在与前者是打开一个空的缓冲区，后者会打开当前所在窗口对应的缓冲区，也就是说拆分后的窗口和原来的窗口内容一样。
这两个命令都会将当前窗口拆分成上下排列的两个窗口，因此还有`:vnew`和`:vsplit`两个命令可以将当前窗口拆分成左右排列的窗口。

另外，快捷键`Ctrl+w+s`和`Ctrl+w+v`分别对应`:split`和`:vsplit`命令。

打开窗口后，可以使用`:q`来退出窗口，用`:w`来保存窗口显示的缓冲区中的内容，这和平常是一样的。也可以使用`ZZ`或者`Ctrl+w+c`快捷键或者`:hide`命令来关闭当前窗口，`Ctrl+w+o`快捷键或者`:only`命令可以关闭当前窗口外的其他所有窗口，在很多其他编辑器的标签页上点击右键都可以找到这个功能。

## 切换窗口

虽然可以有多个窗口，当时处于激活状态的窗口只能有一个。可以使用`Ctrl+w+w`快捷键在多个窗口中来回切换，也可以使用`Ctrl+w`再加上上下左右键的来在不同的窗口中切换。
`Ctrl+w+j`可以切换到下一个窗口中，`Ctrl+w+k`可以将切换到在上一个窗口中，`Ctrl+w+t`可以直接切换到最顶部的窗口，`Ctrl+w+b`可以直接切换到最底部的窗口，而`Ctrl+w+p`可以切换到最近一次所在的窗口。

## 移动窗口

说实在话，我使用Vim的经历中很少用到移动窗口的功能，列在这里只是保证这篇短文的完整性而已，所以你要是记不住这些命令就直接跳过好了，直接到下面_调整窗口大小_一节，这个功能非常常用。

快捷键`Ctrl+w+r`可以将当前的窗口向下进行循环移动。这个命令可以带一个数字作为参数，指明向下循环移动所执行的次数。对应的有`Ctrl+w+R`快捷键，可以使得窗口向上循环移动。

快捷键`Ctrl+w+x`可以将当前窗口与下一窗口进行位置对换。如果当前窗口在底部，而且没有下一个窗口，这个快捷键将使当前窗口与上一个窗口进行位置对换。 利用`Ctrl+w+K`可以将当前窗口放到最顶端，而`Ctrl+w+J`可以把当前窗口放到最底部。

## 调整窗口大小

快捷键`Ctrl+w+>`可以增加窗口的宽度，快捷键`Ctrl+w+<`可以减少窗口的宽度，快捷键`Ctrl+w++`可以增加窗口的高度，快捷键`Ctrl+w+-`可以减少窗口的高度，这几个快捷键都很形象。需要注意的是`Ctrl+w+>`和`Ctrl+w+<`只会增加或者减少一个单位，基本毫无用处，可以考虑使用`10 Ctrl+w+>`将宽度增加10个单位，其他的可以以此类推，详见 <a href="http://stackoverflow.com/questions/4368690/how-to-increase-the-vertical-split-window-size-in-vim">How to increase the vertical split window size in Vim</a>。

快捷键`Ctrl+w+=`可以使窗口的宽度或者高度变成相同。

最后，这个快捷键`Ctrl+W+_`可以将当前的窗口最大化，是不是挺有意思的。:)

## 参考

* <a href="http://www.cs.swarthmore.edu/help/vim/windows.html">http://www.cs.swarthmore.edu/help/vim/windows.html</a>
* <a href="http://www.pythonclub.org/linux/vim/window">http://www.pythonclub.org/linux/vim/window</a>
* <a href="http://stackoverflow.com/questions/4368690/how-to-increase-the-vertical-split-window-size-in-vim">http://stackoverflow.com/questions/4368690/how-to-increase-the-vertical-split-window-size-in-vim</a>
* <a href="http://www.cs.swarthmore.edu/help/vim/windows.html">http://www.cs.swarthmore.edu/help/vim/windows.html</a>