---
layout: post
title: Linux dig 命令详解
excerpt: |+
  之所以会关注到这个命令，是因为最近在研究 <a href="http://zhouliang.pro/2014/01/19/mysql-master-slave-replication/">MySQL的主从复制</a> 相关的技术，希望能实现当Slave落后Master比较多的时候自动将Slave从数据源中去除掉的功能。找了很多资料，没有比较好的现成办法。只能参考 <a href="http://www.percona.com/software/percona-toolkit">percona-toolkit</a> 中的 <a href="http://www.percona.com/doc/percona-toolkit/2.1/pt-heartbeat.html">pt-heartbeat</a> 命令的实现自己来做，在实现的过程中，发现这个`dig`命令，深感有必要记录一下。
date: '2014-02-12 00:37:28 +0800'
categories:
- Linux
tags:
- MySQL
- Linux
---
之所以会关注到这个命令，是因为最近在研究 <a href="http://zhouliang.pro/2014/01/19/mysql-master-slave-replication/">MySQL的主从复制</a> 相关的技术，希望能实现当Slave落后Master比较多的时候自动将Slave从数据源中去除掉的功能。找了很多资料，没有比较好的现成办法。只能参考 <a href="http://www.percona.com/software/percona-toolkit">percona-toolkit</a> 中的 <a href="http://www.percona.com/doc/percona-toolkit/2.1/pt-heartbeat.html">pt-heartbeat</a> 命令的实现自己来做，这个完全可以写另一篇博客了，暂且不提罢。

我们每台MySQL服务器都部署有监控用的程序，我希望把这个发送heartbeat的程序集成到监控程序中，但这个程序只需要在Master机器上启动，这就涉及到如何确定这台MySQL服务器是Master还是Slave了，有一些<a href="http://dba.stackexchange.com/questions/46011/how-to-determine-master-in-mysql-master-slave">办法</a>，但是都不算太好，因为MySQL的一个Slave机器也可以作为另一个Slave的Master，比较拗口，大意就是你的你的儿子也是你的孙子的爸爸。

好在我们的MySQL服务器都绑定着CNAME，访问一台MySQL服务器的时候，使用的是对应的CNAME，而不是那台机器的IP或者主机名（可以通过`hostname`命令查看，因此下文称为hostname），比如Master的机器是 `abcedfg.xxx.com`，但是我们另外绑定了`master.db.xxx.com` 域名，Slave机器是 `hijklmn.xxx.com`，我们绑定了 `slave-1.db.xxx.com` 域名。

```json
{
    "master" : "master.db.xxx.com",
    "slaves" : ["slave-1.db.xxx.com", "slave-1.db.xxx.com"]
}
```

所以我用了一个比较奇怪的方法：通过CNAME来查找Master的hostname，如果运行该heartbeat程序的机器的hostname和刚查找到的Master的hostname一致，则说明该程序运行于一台Master机器，否者就直接退出。

因此，问题转变成如何通过CNAME来查找一个域名其真正指向的服务器的hostname。这里，我直观的想到可以用Linux的`dig`命令来解决这个问题。需要注意的是，除了这个命令以外，很多编程语言库都提供相应的支持，比如Java的<a href="http://download.java.net/jdk7/archive/b123/docs/api/java/net/InetAddress.html">InetAddress</a>就可以实现<a href="http://stackoverflow.com/questions/3371879/ip-address-to-hostname-in-java">通过IP地址查找对应的域名</a>：

```java
InetAddress addr = InetAddress.getByName("8.8.8.8");
String host = addr.getHostName();
System.out.println(host);
```

## CNAME

可能需要先大概介绍一下什么是CNAME：一个域名可以有两种类型的指向，如果一个**域名指向**称为一个**记录（Record）**的话，那么就有两种**记录类型（Record Type）**，分别是：

* **A记录**：指向一个IP地址
* **CNAME**：指向一个其他的域名

下面这个图是我的域名的配置：

<a href="https://www.flickr.com/photos/zhlwish/14022922767/" title="Flickr 上 zhlwish 的 cname-settings"><img src="https://farm6.staticflickr.com/5080/14022922767_5176acafb9_o.png" width="793" height="166" alt="cname-settings"></a>

这里有两条A记录，一条CNAME。两条A记录指向的就是我的博客所在的VPS：

* 第二条容易理解，就是将 `www.zhouliang.pro` 指向了VPS的IP地址，这样你使用 `http://www.zhouliang.pro` 就可以访问我的博客了；
* 第一条有点奇怪，这里是一个泛域名，也就是将 `zhouliang.pro` 也指向了这个IP地址，也就是说你用 `http://zhouliang.pro` 也可以直接访问我的博客。
* 第三条记录就是一个CNAME指向，也许你已经在浏览器中打开了 `http://i.zhouliang.pro` ，我将 `i.zhouliang.pro` 转向了网易轻博客服务，放了几张照片，你们感受一下，小清新有木有。

> 彩蛋：买域名的时候特别注意服务商是不是提供免费的泛域名解析服务，不提供的都是耍流氓，据我所知，万网就是在耍流氓。

## dig 命令

学习Linux命令只有一条路，那就是：`man dig`，到控制台敲一下这个命令，输出略长。本文的目的是先大致介绍一下，深入了解还是得细读`man dig`。
在控制台输入，输出结果如下：

    $ dig i.zhouliang.pro
    ; <<>> DiG 9.8.3-P1 <<>> i.zhouliang.pro
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45515
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0
    ;; QUESTION SECTION:
    ;i.zhouliang.pro.       IN  A
    ;; ANSWER SECTION:
    i.zhouliang.pro.        14400   IN  CNAME   mydomain.lofter.com.
    mydomain.lofter.com.    18000   IN  A       54.248.125.234
    ;; Query time: 211 msec
    ;; SERVER: 192.168.106.1#53(192.168.106.1)
    ;; WHEN: Fri Jan 24 00:43:26 2014
    ;; MSG SIZE  rcvd: 82

输出结果大致分成4个部分，实际上可能还包括更多的内容，总共会有以下6个部分：

1. **Header**: 包括软件版本，全局变量以及除消息头以外的其他部分的信息，比如上例中，显示有1个QUERY，2个ANSWER
2. **QUESTION SECTION**: 请求参数信息，也就是你的输入
3. **ANSWER SECTION**: 从DNS查询到的信息，也就是输出，显示`i.zhouliang.pro`是CNAME，指向`mydomain.lofter.com`，而后者是一个A记录，指向一个IP地址
4. **AUTHORITY SECTION**: 包含DNS域名服务器的授权信息，上例中不包含这一部分，如果用这个命令就可以看到 `dig @ns1.redhat.com redhat.com` ，这里的`@`符号用于指定查询所使用的DNS服务器
5. **ADDITIONAL SECTION**: 包含AUTHORITY SECTION中的域名服务器的IP地址，同样，上例中也不包含这一部分
6. **Stats section**: 最下方的一部分，显示了查询时间等额外信息

另外，上面所有的以 `;` 开头的行实际上都是注释。
可以通过下面的参数来控制显示或者不显示上面的这些部分：

1. +nocomments :  不显示注释
2. +noauthority :  不显示AUTHORITY SECTION
3. +noadditional :  不显示ADDITIONAL SECTION
4. +nostats :  不显示Stats section
5. +noanswer :  不显示ANSWER SECTION
6. +noall : 不显示所有的信息，一般会这样用 `dig zhouliang.pro +noall +answer`

和上面参数对应还有`+comments`，`+answer`等，后文有示例，此处不赘述。另外，还有如下两个参数需要了解：

* +short : 显示简短的信息
* -t &lt;type&gt; : 指定查询的记录类型，可以是CNAME、A、MX、NS，分别表示CNAME、A记录、MX记录、DNS服务器，默认是A
* -x : 表示反向查找，也就是根据IP地址查找域名

## dig命令示例

下面来举几个实用的例子。

### 查看域名

    $ dig i.zhouliang.pro +noall +anwser
    ; <<>> DiG 9.8.3-P1 <<>> i.zhouliang.pro +noall +answer
    ;; global options: +cmd
    i.zhouliang.pro.            10034    IN    CNAME    mydomain.lofter.com.
    mydomain.lofter.com.        9183     IN    A        54.248.125.234

特别注意这里输出了两行，第一行是CNAME，先将`i.zhouliang.pro`解析成`mydomain.lofter.com`，第二行是A记录，将`mydomain.lofter.com`解析成IP地址。这是一个完整的域名解析过程。

### 查找域名的MX记录：

    $ dig zhouliang.pro -t MX +short
    10 mxdomain.qq.com.

从输出可以看出，我用了QQ提供的域名邮箱服务。

### 查找域名对应的CNAME：

    $ dig i.zhouliang.pro -t CNAME +short
    mydomain.lofter.com.

从输出可以看出，我用了网易Loft提供的博客服务。另外，这个方法刚好解答了本文开头所提到的那个问题。

### 根据IP地址反向查找域名

    $ dig -x 8.8.8.8 +short
    ; <<>> DiG 9.8.3-P1 <<>> -x 8.8.8.8 +noall +answer
    ;; global options: +cmd
    8.8.8.8.in-addr.arpa.    79605    IN    PTR    google-public-dns-a.google.com.

从输出可以看出，Google的这个DNS服务器有个域名叫做google-public-dns-a.google.com

### 查询域名的解析DNS服务器地址

    $ dig zhouliang.pro ns +short
    ns15.bigwww.com.
    ns13.bigwww.com.

从输出可以找到我的解析我的域名的DNS服务器地址。

## 参考

* <a href="http://www.thegeekstuff.com/2012/02/dig-command-examples/">10 Linux DIG Command Examples for DNS Lookup</a>