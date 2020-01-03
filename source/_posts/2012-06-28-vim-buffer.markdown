---
layout: post
title: Vim的Buffer/缓冲区
excerpt: |+
  顾名思义，Buffer是内存中的一块缓冲区域，用于临时存放Vim打开过的文件。用Vim打开文件后，文件就自动被加入到Buffer队列中，而且Buffer中永远是最新的版本，修改文件后还未保存时，改动就存在于Buffer中。打开过的文件会一直存在Buffer中，除非手动的删除（bw命令，不过很多时候没这个必要）。在阅读或者编写代码的时候，会经常在多个文件之间跳转，很好的利用Buffer会当然的make
  your life more easier。'
date: '2012-06-28 00:12:50 +0800'
categories:
- Linux
tags:
- Linux
- Vim
---
顾名思义，**Buffer**是内存中的一块缓冲区域，用于临时存放Vim打开过的文件。用Vim打开文件后，文件就自动被加入到Buffer队列中，而且Buffer中永远是最新的版本，修改文件后还未保存时，改动就存在于Buffer中。打开过的文件会一直存在Buffer中，除非手动的删除（bw命令，不过很多时候没这个必要）。在阅读或者编写代码的时候，会经常在多个文件之间跳转，很好的利用Buffer会当然的_make your life more easier_。

## 查看Buffer/缓冲区

使用下面命令可以同时打开多个文件：

    vim test1 test2 test3

之后，可以用Vim命令`:o`继续打开文件

    :o test4

使用VIM命令`:buffers`可以查看当前Buffer中的文件列表：

    :buffers

<a href="https://www.flickr.com/photos/zhlwish/14023090980/" title="Flickr 上 zhlwish 的 vim-buffer-snapshot"><img src="https://farm6.staticflickr.com/5574/14023090980_00dbb91c0a_o.png" width="347" height="128" alt="vim-buffer-snapshot"></a>

`:buffers`命令还有两个更简单一些的别名：`:ls`和`:files`。
上图中第一列是文件编号，第二列是缓冲文件的状态，第三列是文件的名称，第四列是上一次编辑的位置，即在不同文件之间切换的时候Vim会自动跳转到上一次光标所在的位置。 缓冲文件的状态有如下几种，仅供参考：

* - （非活动的缓冲区）
* a （当前被激活缓冲区）
* h （隐藏的缓冲区）
* % （当前的缓冲区）
* # （交换缓冲区）
* = （只读缓冲区）
* + （已经更改的缓冲区）

## 切换缓冲区

最常用的功能是缓冲区之间的切换了，最直接的方式是先用`:buffers`命令查看所有的缓冲区，然后使用`:buffer <编号>`或者`:buffer <文件名>`切换：

    :buffer 1
    :buffer test1

这种方式看起来比较费劲。

另外一种方式是使用切换上一个或者下一个缓冲区，以及直接切换到第一个和最后一个缓冲区的命令。文件比较少的时候的确很管用。

    :bnext
    :bprevious
    :blast
    :bfirst

当然，为了节省敲键盘的时间，可以在**.vimrc**中设置缓冲区前后切换的快捷键：

    nmap <C-b>n  :bnext<CR>;
    nmap <C-b>p  :bprev<CR>;

同时按下`Ctrl+b+n`或者`Ctrl+b+p`时切换到下一个或者前一个缓冲区。

## 维护缓冲区

如果希望维护一个简洁的缓冲区，不希望一些和当前工作不相关的缓冲区存在，或者希望手动的把一些和当前工作相关的文件加入到缓冲区，下面三个命令可以帮助你：

    :badd test5
    :bdelete test4

上面命令添加了一个名为_test5_的缓冲区，删除了缓冲区_test4_。

## bufexplorer插件

<a href="http://www.vim.org/scripts/script.php?script_id=42">bufexplorer插件</a>提供了一些替换上面命令的快捷键，并且提供了一个窗口，可以选择、删除缓冲区，解压下载后的文件到~/.vim/目录，重启VIM就生效了。 常用的快捷键如下：

* \bv  垂直方向打开一个窗口，显示缓冲区列表
* \bs  水平方向打开一个窗口，显示缓冲区列表
* \be  在当前编辑窗口中显示缓冲区列表

bufexplorer有下面的选项设置：

    """"""""""""""""""""""""""""""
    " BufExplorer
    """"""""""""""""""""""""""""""
    let g:bufExplorerDefaultHelp=0       " Do not show default help.
    let g:bufExplorerShowRelativePath=1  " Show relative paths.
    let g:bufExplorerSortBy='mru'        " Sort by most recently used.
    let g:bufExplorerSplitRight=0        " Split left.
    let g:bufExplorerSplitVertical=1     " Split vertically.
    let g:bufExplorerSplitVertSize = 30  " Split width
    let g:bufExplorerUseCurrentWindow=1  " Open in new window.

最后看起来应该是这个样子:

<a href="https://www.flickr.com/photos/zhlwish/14206431341/" title="Flickr 上 zhlwish 的 bufexplorer"><img src="https://farm6.staticflickr.com/5489/14206431341_313faa63c1_o.png" width="839" height="550" alt="bufexplorer"></a>

## 参考

* <a href="http://www.pythonclub.org/linux/vim/buffer">http://www.pythonclub.org/linux/vim/buffer</a>
* <a href="http://stackoverflow.com/questions/102384/using-vims-tabs-like-buffers">http://stackoverflow.com/questions/102384/using-vims-tabs-like-buffers</a>
* <a href="http://www.openfoundry.org/tw/tech-column/2383-vim--buffers-and-windows">http://www.openfoundry.org/tw/tech-column/2383-vim--buffers-and-windows</a>
* <a href="http://vim.wikia.com/wiki/Vim_buffer_FAQ">http://vim.wikia.com/wiki/Vim_buffer_FAQ</a>
* <a href="http://easwy.com/blog/archives/advanced-vim-skills-netrw-bufexplorer-winmanager-plugin/">enter link description here</a>
