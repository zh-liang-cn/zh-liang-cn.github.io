---
layout: post
title: 几个有意思的Ruby全局变量
excerpt: |+
  最近在做Ruby on Rails往JRuby上的迁移工作，积累了一些关于平常写代码时不太容易注意的环境变量、命令行参数和全局变量，了解这些知识对于进一步学习Ruby有很大的帮助，也有助于阅读一些开源框架如<a href="http://rubyonrails.org/">Rails</a>、<a href="https://github.com/rspec">Rspec</a>的代码。本文是这个小系列的第三篇。其他两篇分别是<a href="http://www.zhlwish.com/2013/01/10/ruby-environment-variables-you-should-known/">几个有意思的Ruby环境变量</a>和<a href="http://www.zhlwish.com/2013/01/07/ruby-command-line-params-you-should-know/">几个有意思的Ruby命令行参数</a> 。
date: '2013-01-15 00:39:37 +0800'
categories:
- Ruby
tags:
- ruby
---
最近在做Ruby on Rails往JRuby上的迁移工作，积累了一些关于平常写代码时不太容易注意的环境变量、命令行参数和全局变量，了解这些知识对于进一步学习Ruby有很大的帮助，也有助于阅读一些开源框架如<a href="http://rubyonrails.org/">Rails</a>、<a href="https://github.com/rspec">Rspec</a>的代码。本文是这个小系列的第三篇。其他两篇分别是<a href="http://zhouliang.pro/2013/01/10/ruby-environment-variables-you-should-known/">几个有意思的Ruby环境变量</a>和<a href="http://zhouliang.pro/2013/01/07/ruby-command-line-params-you-should-know/">几个有意思的Ruby命令行参数</a> 。

Ruby中的全局变量指以 `$` 开头的变量，<a href="http://blog.chinaunix.net/uid-26244834-id-2954687.html">Ruby内置的全局变量有55个（1.9版本）</a>，可以使用 `Kernel#global_variables()`方法查看所有的全局变量。

```ruby
global_variables.sort
# => [:$!, :$", :$$, :$&amp;, :$', :$*, :$+, :$,, :$-0, :$-F, :$-I, :$-K, :$-W, :$-a, :$-d, :$-i, :$-l, :$-p, :$-v, :$-w, :$., :$/, :$0, :$1, :$2, :$3, :$4, :$5, :$6, :$7, :$8, :$9, :$:, :$;, :$< , :$=, :$>, :$?, :$@, :$DEBUG, :$FILENAME, :$KCODE, :$LOADED_FEATURES, :$LOAD_PATH, :$PROGRAM_NAME, :$SAFE, :$VERBOSE, :$\, :$_, :$`, :$binding, :$stderr, :$stdin, :$stdout, :$~]
```

里面的大部分都比较少用到，下面是我用到的几个：

## $stdin, $stdout, $stderr

分别代表标准输入、标准输出和错误输出，其变量类型为<a href="http://ruby-doc.org/core-1.9.3/IO.html">IO</a>。

```ruby
$stdin.class
# => IO
```

需要在控制台打印日志时，可以直接将日志输出到标准输出流（大部分时候指控制台）。

```ruby
log = Logger.new($stdout)
```

## $0, $PROGRAM_NAME

都指代程序名称，下面是 `global.rb`文件的代码内容：

```ruby
#!/usr/bin/env ruby
puts $0
puts $PROGRAM_NAME
```

在控制台执行输出结果如下：

    $ ruby global.rb
    global.rb
    global.rb

用处最广的地方可能是判断当前脚本是直接被执行还是被`require`，详见[what-does-if-file-0-mean-in-ruby](http://stackoverflow.com/questions/4687680/what-does-if-file-0-mean-in-ruby)，[FILE](http://ruby-doc.org/docs/keywords/1.9/Object.html#method-i-__FILE__)是一个Object对象的一个方法，返回当前源代码文件的名称，方法名称比较奇怪，不过奇怪的方法名称在Ruby中已经见怪不怪了。

```ruby
if __FILE__ == $0
    puts 'do something'
end
```

## $n 和 $~

`$n`在这里指代`$1`, `$2`, `$3`, ..., `$9`，这个变量是和模式匹配运算符 `=~` 一起使用的，指代匹配到的字符串，比如

```ruby
'1986-11-25' =~ /(\d+)-(\d+)-(\d+)/
# => 0
puts $1
# => 1986
puts $2
# => 11
puts $3
# => 25
```

当然，大部分时候我们并不清楚会返回多少个匹配项，`$~`来拯救了，这个全局变量返回<a href="http://www.ruby-doc.org/core-1.9.3/MatchData.html">MatchData</a>对象，包括了所有的匹配。

```ruby
puts $~
# => # matchdata "1986-11-25" 1:"1986" 2:"11" 3:"25" ;
puts $~[0]
# => 1986
```

最后，`$&amp;`表示最近一次进行匹配的整个字符串，在上例中为`1986-11-25`。$`和`$'`也和匹配相关，分别表示当前匹配之前的字符串(pre-match)和当前匹配之后的字符串(post-match)，具体可以参考<a href="http://www.ruby-doc.org/core-1.9.3/MatchData.html#method-i-post_match">MatchData#post_match</a>，实在是比较少用到。

## $LOAD_PATH 和 $:

我在<a href="http://zhouliang.pro/2013/01/10/ruby-environment-variables-you-should-known/">《几个有意思的Ruby环境变量》</a>中示例`RUBYLIB`和`RUBYOPT`环境变量的时候提到过这两个变量，两个都是数组，数组元素为Ruby的代码加载路径。
除了通过修改环境变量以外修改Ruby代码加载路径外，还可以通过修改这个全局变量来到达相同的效果。也就是说 `env RUBYLIB=/tmp/lib1` 和 `$LOAD_PATH << '/tmp/lib1'`起到的作用一样，各有千秋。

## $$

我们有个网站，后台挂了多个Rails进程，所有Rails的进程打印到同一个文件(production.log)中，我们希望在日志里输出一些信息的时候带上进程的id用于了解每个进程的状态，这个变量就用的上了，`$$`代表进程的id号。

## 其他

下面只是简单列举其他的几个全局变量，我从未在写代码的过程中使用过这些，因此只是简单罗列。

* `$!`, 最近一次的错误信息 
* `$@`, 错误产生的位置 
* `$_`, gets最近读的字符串 
* `$.`, 解释器最近读的行数(line number) 
* `$=`, 是否区别大小写的标志 
* `$/`, 输入记录分隔符 
* `$\`, 输出记录分隔符 
* `$*`, 命令行参数 
* `$?`, 最近一次执行的子进程退出状态

