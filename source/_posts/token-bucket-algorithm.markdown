---
layout: post
title: 令牌桶算法/Token Bucket Algorithm
excerpt: |+
  在电信网络中最常用的流量控制算法是 <a href="http://en.wikipedia.org/wiki/Token_bucket">令牌桶算法</a>（Token Bucket Algorithm），在Linux上同样也用了这个算法来 <a href="http://lartc.org/howto/lartc.qdisc.classless.html#AEN690">对带宽进行管理</a>。当然使用场景不仅仅局限于此，凡是涉及到流量控制的地方都可以使用这种方法。例如，我们可以用这种方法对Web Service的访问量进行控制，可以避免Web Service遭受到恶意的攻击，也可以实现较复杂的流量分级计费，比如：调用次数1000次/s以下，每天收费10元；1000-2000次/s以下，每天收费40元等。
date: '2014-01-25 01:40:56 +0800'
categories:
- Linux
tags:
- Linux
- Algorithm
---

在电信网络中最常用的**流量控制算法**就是 <a href="http://en.wikipedia.org/wiki/Token_bucket">令牌桶算法</a>（Token Bucket Algorithm），在Linux上同样也用了这个算法来 <a href="http://lartc.org/howto/lartc.qdisc.classless.html#AEN690">对带宽进行管理</a>。当然使用场景不仅仅局限于此，凡是涉及到流量控制的地方都可以使用这种方法。例如，我们可以用这种方法对Web Service的访问量进行控制，可以避免Web Service遭受到恶意的攻击，也可以实现较复杂的流量分级计费，比如：调用次数1000次/s以下，每天收费10元；1000-2000次/s以下，每天收费40元等。

<a href="http://en.wikipedia.org/wiki/Token_bucket">令牌桶算法</a> 实际上由三部分组成：两个流和一个桶，分别是令牌流、数据流与令牌桶。

## 令牌流与令牌桶

系统会以一定的速度生成令牌，并将其放置到令牌桶中，可以将令牌桶想象成一个缓冲区（可以用队列这种数据结构来实现），当缓冲区填满的时候，新生成的令牌会被扔掉。
这里有两个变量很重要：

* 第一个是生成令牌的速度，一般称为 _rate_ 。比如，我们设定 _rate = 2_，即每秒钟生成 _2_ 个令牌，也就是每 _1/2_ 秒生成一个令牌； 
* 第二个是令牌桶的大小，一般称为 _burst_。比如，我们设定 _burst = 10_，即令牌桶最大只能容纳 _10_ 个令牌。

## 数据流

数据流是真正的进入系统的流量，对于Web Service来讲，如果平均每秒钟会调用2次，则认为速率为 _2次/s_ 。

## 算法原理

系统接收到一个单位数据（对于网络传输，可以是一个包或者一个字节；对于Web Service，可以是一个请求）后，从令牌桶中取出一个令牌，然后对数据或请求进行处理。如果令牌桶中没有令牌了，会直接将数据或者请求丢弃。当然，对于Web Service，就不能是丢弃这么简单了：可以返回一个异常消息，用于提示用户其请求速率超过了系统限制。
整个算法的结构看起来是这样：

<a href="https://www.flickr.com/photos/zhlwish/14206222001/" title="Flickr 上 zhlwish 的 token-bucket"><img src="https://farm3.staticflickr.com/2908/14206222001_9729125a30_o.png" width="851" height="311" alt="token-bucket"></a>

有一下三种情形可能发生：

* 数据流的速率 **等于** 令牌流的速率。这种情况下,每个到来的数据包或者请求都能对应一个令牌,然后无延迟地通过队列；
* 数据流的速率 **小于** 令牌流的速率。通过队列的数据包或者请求只消耗了一部分令牌，剩下的令牌会在令牌桶里积累下来，直到桶被装满。剩下的令牌可以在突发请求的时候消耗掉。
* 数据流的速率 **大于** 令牌流的速率。这意味着桶里的令牌很快就会被耗尽。导致服务中断一段时间，如果数据包或者请求持续到来,将发生丢包或者拒绝响应。

比如前面举的例子，生成令牌的速率和令牌桶的大小分别为 _rate = 2, burst = 10_，则系统能承受的突发请求速率为 _10次/s_，平均请求速率为 _2次/s_。
三种情形中的最后一种情景是这个算法的核心所在，这个算法非常精确，实现非常简单并且对服务器的压力可以忽略不计，因此应用得相当广泛，值得学习和利用。

## 参考

* <a href="http://lartc.org/#download">Linux 的高级路由和流量控制/Linux Advanced Routing &amp; Traffic Control</a>
* <a href="http://en.wikipedia.org/wiki/Token_bucket">Token Bucket from WikiPedia</a>
