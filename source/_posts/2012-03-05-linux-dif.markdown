---
title: Linux上diff命令详解
excerpt: "<a href=\"http://en.wikipedia.org/wiki/Diff\" target=\"_blank\">diff</a>程序是linux上非常重要的工具，用于比较文件的内容，特别是比较两个版本不同的文件(本文中的a.c、b.c可以理解为两个版本的同一个文件，b.c是在a.c的基础上的修正版)以找到改动的地方。diff在命令行中打印每一个行的改动。最新版本的diff还支持二进制文件。diff程序的输出被称为补丁(patch)，因为unix系统中还有一个<a href=\"http://en.wikipedia.org/wiki/Patch_(Unix)\" target=\"_blank\">patch</a>程序，可以根据diff的输出将a.c的文件内容更新为b.c。diff是svn、cvs、git等版本控制工具不可或缺的一部分。"
date: '2012-03-05 23:13:32 +0800'
categories:
- Linux
tags:
- shell
- diff
---
<a href="http://en.wikipedia.org/wiki/Diff" target="_blank">diff</a>程序是linux上非常重要的工具，用于比较文件的内容，特别是比较两个版本不同的文件(本文中的a.c、b.c可以理解为两个版本的同一个文件，b.c是在a.c的基础上的修正版)以找到改动的地方。diff在命令行中打印每一个行的改动。最新版本的diff还支持二进制文件。diff程序的输出被称为补丁(patch)，因为unix系统中还有一个<a href="http://en.wikipedia.org/wiki/Patch_(Unix)" target="_blank">patch</a>程序，可以根据diff的输出将a.c的文件内容更新为b.c。diff是svn、cvs、git等版本控制工具不可或缺的一部分。

## diff和patch实例

下面用一个实例来讲述diff和patch的使用。

文件a.c的内容如下：

```c
#include <stdio.h>
int main(int argc, char *argv[])
{
    // add code here
    return 0;
}
```

文件b.c的内容如下:

```c
#include <stdio.h>
int main(int argc, char *argv[])
{
    printf("Hello world");
    return 0;
}
```

执行命令：

```bash
diff a.c b.c > b.patch
```

输出b.patch的内容如下：

    5c5
    <     // add code here
    ---
    printf("Hello world");

接着执行下面的命令就可以将patch文件b.patch应用于a.c文件上，将a.c文件的内容更新为新的版本(即b.c)

```bash
patch a.c b.patch
```

a.c的内容被更新为b.c的内容：

```c
#include <stdio.h>
int main(int argc, char *argv[])
{
    printf("Hello world");
    return 0;
}
```

不过，diff的输出让人读起来很废劲，因为这个输出本来就不是给人看的，如果你知道一点关于<a href="http://en.wikipedia.org/wiki/Ed_(Unix)" target="_blank">ed</a>编辑器的用法，就容易理解了。最开始的时候diff的输出不是这种格式的，后来受到<a href="http://en.wikipedia.org/wiki/Ed_(Unix)" target="_blank">ed</a>编辑器的影响才变成这个样子，至于其发展历程请移步<a href="http://en.wikipedia.org/wiki/Diff#History" target="_blank">维基百科</a>，因此在解释diff的输出之前，先简单介绍一下<a href="http://en.wikipedia.org/wiki/Ed_(Unix)" target="_blank">ed</a>编辑器。

## ed编辑器

ed是一个交互式的文本编辑器，类Unix系统中有很多很神奇的编辑器，号称神器的有vi、vim、emacs，还有常规意义的编辑器nano、gedit，还有看起来很诡异的sed(**S**tream **Ed**itor)、awk，ed这是另一个很诡异的编辑器，不同于前面的任何一种。

打开shell，按次序输入如下的命令(需要一句一句的敲)：

    touch c.c
    ed c.c
    0a
    
    #include <stdio.h>
    int main(int argc, char *argv[])
    {
        // add code here
        return 0;
    }
    .
    w
    q

使用cat命令查看c.c，该文件的内容就是上面代码中“0a”和“.”之前代码。

和vim一样，ed通过一系列的命令来编辑文件：

* “ed c.c”表示用ed对文件c.c进行编辑
* “0a”表示在第0行后进行添加(a表示add，同样，d表示删除，c表示修改)
* 接着输入需要添加的内容，输入完成后换一行，输入“.<CR>”(<CR>表示回车)，表示输入完成
* 输入“w<CR>”表示保存文件
* 输入“q<CR>”表示退出编辑器

ed编辑器的好处在于可以将一系列命令写成一个文件，然后进行批量执行。比如需要在所有的源代码开头加上版权信息，则可以编写如下的ed文件，保存为add_header:

    0a
    /**
     * &copy; 2012 zhlwish.com
     */
    .
    w
    q

然后写一段shell脚本对源代码文件进行遍历，对每个代码执行如下代码即可批量对源代码文件添加版权信息：

```bash
ed a.c < add_header
```

关于ed的命令和其他方法请参考<a href="http://www.softpanorama.org/Editors/Exorama/index.shtml" target="_blank">Unix ed Editor Command Set</a>本文不详述。

## diff的输出

diff的输出格式是对ed脚本(ed script)的一种扩展，遵循ed的命令语法，添加了"<"和">"分别用于表示新文件的内容和旧文件的内容，比如前文中diff的输出可以按如下方式理解：

* “5c5”表示a.c的第5行有改动(**c**hange)，改动后为b.c的第5行
* “<     // add code here”表示a.c的第5行的内容
* “--- ”是分隔符
* “>     printf("Hello world");”表示b.c文件的第5行的内容

当然还有更复杂的情况，如“3c3,6”、“6d8”，前者表示旧版本文件中的第3行被修改，对应新文件中的第3-6行，后者表示旧版本文件的第6行被删除，在新文件中是第8行。

可以通过参数指定diff输出格式，有兴趣的笔者可以分别进行尝试：

* -e --ed 输出为ed命令格式
* -n --rcs 输出为rcs命令格式
* -y 输出为两列对照模式
* -c 输出为上下文模式

## diff的选项

除以上选项外，diff的有用的选项还包括：

* -r：当diff的参数为文件夹时，diff会遍历整个文件夹对新旧文件夹下同名的文件进行比较
* -w：忽略所有空格和制表符，将所有其他空白字符串视为一致。例如，if ( a == b ) 与 if(a==b) 相等。
* -i：忽略字母大小写。例如，小写 a 被认为同大写 A 一样。

因为成文仓促，不免有不对之处，敬请读者指正。(全文完)

## 参考资料

* <a href="http://baike.baidu.com/view/1374858.htm" target="_blank">Linux命令diff</a>
* <a href="http://www.tsnc.edu.cn/tsnc_wgrj/doc/sed.htm" target="_blank">Sed学习笔记</a>
* <a href="http://wiki.bash-hackers.org/howto/edit-ed" target="_blank">Editing files with the ed text editor from scripts</a>
* <a href="http://www.softpanorama.org/Editors/Exorama/index.shtml" target="_blank">Unix ed Editor Command Set</a>
* <a href="http://www.youtube.com/watch?gl=JP&v=jxlcIMPyAt4" target="_blank">Hello World" with ed editor</a>(Youtube)
* <a href="http://www.ibm.com/developerworks/cn/linux/l-diffp/" target="_blank">用Diff和Patch工具维护源码</a>

