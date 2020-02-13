---
layout: post
title: '几个有意思的Ruby命令行参数 '
excerpt: '最近在做Ruby on Rails往JRuby上的迁移工作，积累了一些关于平常写代码时不太容易注意的环境变量、命令行参数和全局变量，了解这些知识对于进一步学习Ruby有很大的帮助，也有助于阅读一些开源框架如<a href="https://github.com/rails/rails">Rails</a>、<a href="https://github.com/rspec/rspec-core">Rspec</a>的代码。本文算是这个小系列的第一篇。 '
date: '2013-01-07 23:37:17 +0800'
categories:
- Ruby
tags:
- ruby
---
最近在做Ruby on Rails往JRuby上的迁移工作，积累了一些关于平常写代码时不太容易注意的环境变量、命令行参数和全局变量，了解这些知识对于进一步学习Ruby有很大的帮助，也有助于阅读一些开源框架如<a href="https://github.com/rails/rails">Rails</a>、<a href="https://github.com/rspec/rspec-core">Rspec</a>的代码。本文算是这个小系列的第一篇。

## -h

显示帮助，本文的大部分内容都是来自于这里，虽然写了将近一年时间的Ruby代码，但是却很少仔细阅读 `ruby -h` 的信息，更为详细的信息可以查看 `man ruby`。特别列出来，做个提醒。

## -e 'script'

直接执行一段Ruby脚本，比如:

    $ ruby -e 'puts "hello"'
    hello

为了比较不同的Ruby实现和版本的性能，可以会执行一些简单的的脚本来进行对比（测试数据仅共演示用，更好的性能对比使用<a href="http://www.ruby-doc.org/stdlib-1.9.3/libdoc/benchmark/rdoc/Benchmark.html">Benchmark</a>或者<a href="https://github.com/rdp/ruby-prof">ruby-prof</a>）：

    $ time ruby -e '(0..100000000).reduce(:+)'
    7.81s user 0.01s system 99% cpu 7.822 total
    $ time jruby -e '(0..100000000).reduce(:+)'
    5.18s user 0.24s system 108% cpu 4.989 total

## -r 'lib'

强制让Ruby使用require去加载不包含在<a href="http://ruby-doc.org/core-1.9.3/">核心库</a>中的其他库，如<a href="http://ruby-doc.org/stdlib-1.9.3/">标准库</a>中的<a href="http://ruby-doc.org/stdlib-1.9.3/libdoc/date/rdoc/Date.html">date</a>。

比如下面的命令会提示报错，因为Date在标注库，Ruby启动时不会默认加载。

    $ ruby -e "puts Date.today"
    -e:1:in 'main' uninitialized constant Date (NameError)

加上`-r`参数就不会报错了，相当于执行了`require "date""`.

    $ ruby -r date -e "puts Date.today"

## -c

让Ruby进行语法检查，确保没有括号不匹配，`do &hellip; end`不匹配的等语法错误。比如：

    $ ruby -ce '(1..2).each { |i| puts i '
    -e:1: syntax error, unexpected $end, expecting '}'

不过如果你的程序有自动测试的话，一般不需要这一步的。以防万一，万一有偷懒没写自动测试的情况下，可以为你守住最后一条防线。

## -Idirectory

告诉Ruby到哪儿去加载代码。相当于在Ruby代码中执行`$LOAD_PATH < < directory`。我们使用的JRuby是经过自己修改过的，而且我司有自己的Package管理系统，rubygems和rake的代码分开放在独立的包中的，因此这个参数就起到了很大的作用。

另外，使用`RUBYLIB`这个环境变量也可以起到同样的效果

    $ env RUBY_LIB="directory" ruby -e "puts 'hello'"

## -d

显示Ruby的debug信息，相当于设置了全局变量$DEBUG为true，其实没什么用，只是比较好玩 :)

## -S

这个参数如果不是这次从Ruby往JRuby迁移，我也不会觉得有什么用。Ruby有一套自己的命令，如果rake、gem、irb等等，Ruby自己有一个`RUBYPATH`的变量，`ruby -S rake -T`表示执行`RUBYPATH`下的rake命令。平常，我们并不需要了解这些，但是在JRuby中，我们就要这样了：

    jruby -S gem list

如果你的机器安装了多个Ruby版本（不是用rvm），那么这个参数肯定有用的。
