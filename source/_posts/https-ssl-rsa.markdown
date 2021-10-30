---
layout: post
title: HTTPS与SSL(一)：RSA
excerpt: |+
  工作中常遇到一些关于安全方面的名词，如SSL、HTTPS、证书、pem、keystore、PGP等，作为不是对安全不甚了解的程序员而言，深感压力巨大，于是趁周末得闲查阅了相关资料，做简要记录。
date: '2013-08-26 01:06:36 +0800'
categories:
- Linux
tags:
- SSH
- HTTPS
- SSL
- Algorithm
---
工作中常遇到一些关于安全方面的名词，如SSL、HTTPS、证书、pem、keystore、PGP等，作为不是对安全不甚了解的程序员而言，深感压力巨大，于是趁周末得闲查阅了相关资料，做简要记录。

## 非对称加密与RSA

一切的一切还得从 **非对称加密** 说起。在信息安全领域，加密方式分为对称加密与非对称加密。对称加密中，加密秘钥和解密秘钥完全相同，安全性不高。而非对称加密的加密秘钥和解密秘钥不同，具有更高的安全性，应用更加广泛.

在非对称加密中，加密秘钥称为 **公钥**(Public Key)，而解密秘钥称为 **私钥**(Private)，而且他们总是成双成对的。一般情况下，我们通过一些工具生成一对公钥和私钥后，将公钥发布给其他人，这样其他人就可以将消息使用公钥加密后发给给我，我接受到消息后，使用我的私钥将消息解密。

<a href="https://www.flickr.com/photos/zhlwish/14023088500/" title="Flickr 上 zhlwish 的 rsa"><img src="https://farm3.staticflickr.com/2897/14023088500_da1890e07c_o.png" width="795" height="259" alt="rsa"></a>

非对称加密有很多种，详见维基百科 <a href="http://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86">公开秘钥加密</a>，使用最广泛的是<a href="https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95">RSA算法</a>。
想想就觉得很神奇，这是怎么做到的呢。我大概了解了一下<a href="https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95">RSA算法</a>，主要是基于数论中的<a href="https://zh.wikipedia.org/wiki/%E8%B4%B9%E9%A9%AC%E5%B0%8F%E5%AE%9A%E7%90%86">费马小定理</a>。虽然我暂时还不能理解为什么是这样，但是经过我的验证，的确很神奇，大概步骤如下：

1. 随意选两个大的质数`p`和`q`，`p`不等于`q`，计算`N = pq`；我选择了`p = 97`，`q = 73`，`n = p * q = 7081`；
2. 根据<a href="https://zh.wikipedia.org/wiki/%E6%AC%A7%E6%8B%89%E5%87%BD%E6%95%B0">欧拉函数</a>，求得小于或者等于`N`的正整数中与`N`互质的数个数：`r = &phi;(N) = &phi;(p)&phi;(q) = (p-1)(q-1)`；本例中`r = 96 * 72 = 6912`；
3. 选择一个小于`r`并且与`r`互质的整数`e`，求`e`关于模`r`的<a href="https://zh.wikipedia.org/wiki/%E6%A8%A1%E5%8F%8D%E5%85%83%E7%B4%A0">模反元素</a>`d`；我选择了质数`13`，肯定与`6912`互质，求得`d = 5317`；
4. 将`(N, e)`作为公钥用于加密，将`(N, d)`作为私钥用于解密；即`(7081, 13)`是公钥，可以分发给其他人，而`(7081, 5317)`是私钥，需要保密；
5. 如果传送的数字为`m`，加密公式为：`m ^ e % N`；假设需要传送数字`100`，`100 ^ 13 % 7081 = 1195`；
6. 假如收到的数字为`n`，解密公式为：`n ^ d % N`；如果收到的数字为`1195`，则解密后得到：`1195 ^ 5317 % 7081 = 100`。

总之，过程很奇妙，而且鉴于实现这个过程的算法和数论理论都是在几百年前发现的，让人不得惊叹人类的智慧和数字的奇妙！<a href="https://gist.github.com/zhlwish/6334261">这里</a>是我检验上面算法所使用的Ruby程序，有兴趣的话也实践一下吧。

在计算机中，字符串经过编码（如ASIIC、Unicode）后实际上是数字。也就是说如果要加密字符串，可以将字符串编码成数字，然后对数字进行加密。解密后，再将数字转换成字符串。

我们现在常用的RSA的秘钥的长度是1024位，不过据称这个长度的RSA已经有被破解的危险，因此提倡使用2048位长度的RSA，然而，Windows XP等早期的系统不支持2048位的RSA算法。

## SSH与RSA

我们在使用<a href="http://zh.wikipedia.org/wiki/Secure_Shell">SSH</a>连接到服务器时也会用到RSA。当然，SSH协议本身设计得具有很强大的扩展性，支持很多种加密算法，RSA只是其中的一种，不过也是应用最广泛的一种。下面是使用<a href="http://en.wikipedia.org/wiki/Ssh-keygen">ssh-keygen</a>工具生成公钥和私钥的过程：

    $ ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/liang/.ssh/id_rsa):
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /Users/liang/.ssh/id_rsa.
    Your public key has been saved in /Users/liang/.ssh/id_rsa.pub.
    The key fingerprint is:
    19:03:9f:99:ef:65:cc:44:d9:f4:89:89:47:89:da:ff liang@liang-macpro
    The key's randomart image is:
    # 后面的输出省略……

这个命令会生成两个文件：`/Users/liang/.ssh/id_rsa.pub`和`/Users/liang/.ssh/id_rsa`。前者是公钥，在建立SSH连接时会发送给服务器，服务器将数据使用此公钥加密后传送给客户端（本机）。后者是私钥，客户端（本机）使用此秘钥对服务器发送的数据进行解密，这个文件很重要，绝对不能外泄，因此最好是能对此文件进行加密，上面命令执行时中间有一个步骤会请你输入密码，这个密码就用于<a href="https://help.github.com/articles/working-with-ssh-key-passphrases">加密私钥</a>。

SSH除了使用用户名密码进行身份验证外，也可以使用公钥进行身份验证。客户端连接服务器时，将客户端的公钥传送给服务器，服务器收到请求后，首先在服务器用户根目录`~/.ssh/authorized_keys`寻找相应的公钥，如果两个秘钥一致，则验证通过。服务器会使用该公钥将数据加密进行传送。

而当客户端连接到服务器时，服务器也会将其公钥传送给客户端，可以在`~/.ssh/known_hosts`文件中找到，从而实现双向数据加密。

讲到这里，就不能不提到<a href="http://rcsg-gsir.imsb-dsgi.nrc-cnrc.gc.ca/documents/internet/node31.html">Password-less SSH</a>了，也就是常说的不用密码登录SSH，实现的步骤如下：

1. 在客户端使用`ssh-keygen -t rsa`生成公钥和秘钥文件，在输入密码的阶段直接输入回车
2. 将`~/.ssh/id_rsa.pub`文件的内容添加到服务器上`~/.ssh/authorized_keys`后面并保存

理解了原理后，深感这个过程太简单了。但是需要注意的是这样其实并不安全，因为私钥有暴露的危险，除非将`~/.ssh/id_rsa`文件的权限设为`600`，还可以用<a href="http://en.wikipedia.org/wiki/Ssh-agent">ssh-agent</a>来实现同样的效果。

到这里，作为一篇博文的篇幅算是差不多了，《HTTPS与SSL》未完待续，敬请期待！
