---
layout: post
title: 几个有意思的Ruby环境变量
excerpt: '最近在做Ruby on Rails往JRuby上的迁移工作，积累了一些关于平常写代码时不太容易注意的环境变量、命令行参数和全局变量，了解这些知识对于进一步学习Ruby有很大的帮助，也有助于阅读一些开源框架如<a href="http://rubyonrails.org/">Rails</a>、<a href="https://github.com/rspec">Rspec</a>的代码。本文是这个小系列的第二篇。 '
date: '2013-01-10 11:24:03 +0800'
categories:
- Ruby
tags:
- ruby
---
最近在做Ruby on Rails往JRuby上的迁移工作，积累了一些关于平常写代码时不太容易注意的环境变量、命令行参数和全局变量，了解这些知识对于进一步学习Ruby有很大的帮助，也有助于阅读一些开源框架如<a href="http://rubyonrails.org/">Rails</a>、<a href="https://github.com/rspec">Rspec</a>的代码。本文是这个小系列的第二篇。

## RUBYLIB

这个环境变量告诉Ruby到哪儿去加载代码，在<a href="http://blog.aizatto.com/2007/05/30/what-version-of-ruby-am-i-using/">几个有意思的Ruby命令行参数</a>中提到过，和 `-Idirectory` 是一样的，在Ruby程序中可以通过 `$LOAD_PATH` 或者是 `$:` 这两个全局变量得到，当然也可以在Ruby程序中修改这两个变量来实现动态切换库文件的版本。JRuby通过 `jruby --1.9`或者`jruby --1.8`来实现1.9和1.8版本的切换就是通过这种方式完成的（只是感觉，没经过验证）。

```bash%}
env RUBYLIB=/tmp/lib1:/tmp/lib2 ruby -e "puts $LOAD_PATH"
ruby -I/tmp/lib1:/tmp/lib2 -e "puts $:"
```

## RUBYOPT

这个环境变量告诉Ruby每次执行的时候都要自动加上那些参数，还是上面的例子，可以这样写：

```bash%}
export RUBYOPT="-I/tmp/lib1:/tmp/lib2 -w"
ruby -e "puts $LOAD_PATH"
ruby -e "puts $:"
```

反正，条条大路通罗马。

JRuby默认支持的Ruby版本是是1.8.7，为了不用每次执行`jruby`命令时都指定版本，我用了RUBYOPT，挺好！

```bash%}
jruby -e "puts RUBY_VERSION"
# 1.8.7
export RUBYOPT="--1.9"
jruby -e "puts RUBY_VERSION"
# 1.9.2
```

## RUBYPATH
这个变量我目前没有发现有什么用，`man ruby`中提到是指定 `-S gem`这样的参数时，Ruby寻找gem命令的路径，默认情况下Ruby也会查找PATH路径，但是前者比后者高。
## GEM_HOME
这个环境变量和rubygems相关，由于rubygems已经成为Ruby 1.9的一部分，因为也写在这里。GEM_HOME是通过 `gem install` 命令安装gem时的安装路径，这个路径下一般有5个文件夹

    bin               # 可执行脚本
    cache             # 缓存
    doc               # 文档
    gems              # gem的代码
    specifications    # gem的说明文件

## GEM_PATH

GEM&#95;HOME是安装路径，而GEM&#95;PATH只是gem的查找路径，像<a href="http://ruby-china.org/wiki/rvm-guide">rvm</a>中有 `gemset` 的概念，也就意味着其实有多个GEM_PATH，<a href="https://gist.github.com/668037">Illustrates GEM&#95;HOME vs GEM&#95;PATH</a>这个gist比较详细的解释了GEM&#95;HOME和GEM&#95;PATH的不同，抄录在这里：

```bash%}
export GEM_HOME=a
export GEM_PATH=a
gem install rack
gem list                    # shows rack
export GEM_HOME=b
export GEM_PATH=b
gem install rake
gem list                    # shows rake (not rack)
export GEM_PATH=a:b
gem list                    # shows rake and rack
```

## 小结
<a href="http://ruby-china.org/wiki/rvm-guide">rvm</a> 是一个非常好的工具，开始我并不是很明白他是怎么实现的，后来了解到这些环境变量后，终于明白了。不信的话，你可以看看这个文件：`~/.rvm/bin`。

## 参考

* <a href="http://blog.aizatto.com/2007/05/30/what-version-of-ruby-am-i-using/">What Version of Ruby Am I Using?</a>
* <a href="http://ruby-china.org/wiki/rvm-guide">RVM 实用指南</a>

