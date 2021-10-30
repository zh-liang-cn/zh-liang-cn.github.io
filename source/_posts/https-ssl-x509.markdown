---
layout: post
title: HTTPS与SSL(二)：X.509
excerpt: |+
  读完上一篇<a href="http://zhouliang.com/2013/08/26/https-ssl-rsa/">《HTTPS与SSL(一)：RSA》</a>，你应该大概了解了RSA加密在整个软件生态中起到的巨大作用。然而，现实生活中仅有加密还远远不够，假设你正在和小明打电话，即使电话传输中使用了万无一失的加密方式，你怎么保证正在和你对话的就是小明本人呢？你说，我听得出来小明的声音，但是声音可以造假；你说，我问小明几个只有我们两个知道答案的问题，呵呵，这种现实生活中也许可以，但是在互联网中世界中不可能（这种方式到是非常好地用在了一些网站的找回密码功能中）。
  肿么办呢？X.509正是为此而生。
date: '2013-09-13 00:06:36 +0800'
categories:
- Linux
tags:
- SSH
- HTTPS
- SSL
- Algorithm
---
读完上一篇<a href="http://zhouliang.com/2013/08/26/https-ssl-rsa/">《HTTPS与SSL(一)：RSA》</a>，你应该大概了解了RSA加密在整个软件生态中起到的巨大作用。然而，现实生活中仅有加密还远远不够，假设你正在和小明打电话，即使电话传输中使用了万无一失的加密方式，**你怎么保证正在和你对话的就是小明本人呢**？你说，我听得出来小明的声音，但是声音可以造假；你说，我问小明几个只有我们两个知道答案的问题，呵呵，这种现实生活中也许可以，但是在互联网中世界中不可能（这种方式到是非常好地用在了一些网站的“找回密码”功能中）。

肿么办呢？X.509正是为此而生。

## 使用RSA对消息签名

先补充一点本来应该属于<a href="http://zhouliang.com/2013/08/26/https-ssl-rsa/">前一篇</a>的内容，写<a href="http://zhouliang.com/2013/08/26/https-ssl-rsa/">前一篇</a>的时候我忘记了，当我想起来的时候发现写在这里也许更加合适。
看了前面的RSA算法后就知道，其实公钥和私钥是可以**相互**（注意这个词）加密和解密的，即同样可以使用私钥对信息进行加密，然后只能用公钥才能解密。这个原理可以用于消息签名。
所谓消息签名指的是：

> 假如甲想给乙传递一个署名的消息的话，那么她可以为她的消息计算一个散列值(Message digest)，然后用她的密钥(private key)加密这个散列值并将这个“署名”加在消息的后面。这个消息只有用她的公钥才能被解密。乙获得这个消息后可以用甲的公钥解密这个散列值，然后将这个数据与他自己为这个消息计算的散列值相比较。假如两者相符的话，那么他就可以知道发信人持有甲的密钥，以及这个消息在传播路径上没有被篡改过。

好了，关于这点就到这里，先看看其他的内容，待会儿还有机会回头来看这里。

## 解决关于信任的问题

其实解决问题的方法很简单，和我们国家实行的<a href="http://zh.wikipedia.org/wiki/%E4%B8%AD%E5%8D%8E%E4%BA%BA%E6%B0%91%E5%85%B1%E5%92%8C%E5%9B%BD%E5%B1%85%E6%B0%91%E8%BA%AB%E4%BB%BD%E8%AF%81">身份证制度</a>一样，你可以凭借你的身份证来证明你是你自己。如果你弄丢了身份证，你说你是你自己没有用，你妈说你就是你也没有用，你必须靠身份证才能证明你是你自己。

这种方法的确是一种行之有效的方法。首先，我们设立一个公安部专门用于发放证书，和我们理解的充满了贪污腐败的生活不一样，我们完全信任这个公安局和他发放的证书。但是如果每个人都来找公安部，公安部就要忙疯了，因此公安部授权成立了省公安厅，依次往下推，一直到县级公安局。我们信任公安部，公安部信任省公安厅，省公安厅信任地级公安局，地级公安局信任县级公安局，因此我们也信任县级公安局。

反过来，如果小明拿着县级公安局发给他的身份证，因为我信任公安部，因此我就可以确认是小明本人，并且信任他。这正是<a href="http://baike.baidu.com/view/7615.htm">X.509协议</a>协议所设计的信任链模型，在X.509协议中，这一个授权链称为Cetrification Chain，公安部就是CA（Certification Authority），身份证就是证书（Certification），而我们的浏览器中内置了一些权威的根证书，浏览器会自动信任由这些权威机构颁发的其他证书。

<a href="https://www.flickr.com/photos/zhlwish/14023127037/" title="Flickr 上 zhlwish 的 2013-09-12_2332"><img src="https://farm6.staticflickr.com/5582/14023127037_f7b383896f_o.png" width="484" height="295" alt="2013-09-12_2332"></a>

上图是Gmail的证书，可以很清楚的看到这个证书链。Gmail网站的证书在最底层，处于最顶层的称为**根证书**，我们可以认为**根证书**就是公安部的“身份证”（实际上应该称为<a href="http://baike.baidu.com/view/359635.htm">组织机构代码证</a>）。每个国家都只有一个公安部，但是国际上不止一家顶级证书颁发机构（顶级CA）。我们所使用的浏览器默认内置了所有这些顶级CA的根证书，因此只要顶级CA颁发证书我们都可以信任。因此当我们访问了一个经过这些顶级CA认证的网站，我们可以在浏览器上看到绿色图标。

CA发放的证书中包括一个公钥（详见本文第3节），其对应的私钥也会同时发放给服务提供商，用于服务提供商和其客户的通信加密。如果服务提供商不小心将私钥弄泄露了，他可以向CA申请召回（Revoke）这个证书，并申请新的证书，CA维护了一个称为Certificate Revocation List的列表。这个证书被召回后，所有的客户可以通过CA查询到这个证书已经被召回，不再使用泄露了私钥的公钥进行加密通信。下图是Chrome浏览器的HTTPS/SSL选项，默认情况下Chrome会检查服务器的证书是否被召回。

<a href="https://www.flickr.com/photos/zhlwish/14023055658/" title="Flickr 上 zhlwish 的 2013-09-04_0004"><img src="https://farm6.staticflickr.com/5039/14023055658_1baee607fa_o.png" width="529" height="114" alt="2013-09-04_0004"></a>

这个过程和下面的这种情况比较类似：如果小明的身份证丢失了，他需要立即通知公安部废止这个身份证，然后重新办理新的身份证。（注：你能想象么，现实生活中的身份证居然是无法废止的，详见这个新闻<a href="http://npc.people.com.cn/n/2013/0821/c14576-22636564.html">《身份证漏洞帮了犯罪分子的忙》</a>，你基本可以认为设计身份证系统的“砖家”完全没有安全意识，不过，这已经见怪不怪了）

## X.509证书

X.509协议是国际电信组织（ITU）的一个标准，正是**这个标准定义了我们上面所说的这一套流程**，专业术语称为**PKI**（public key infrastructure）以及**PMI**（Privilege Management Infrastructure）。我们更加关心的是，这个标准定义了证书的格式。
我们先看一个例子，这个例子摘自<a href="http://en.wikipedia.org/wiki/X.509">维基百科</a>，是www.freesoft.org网站的证书, 重要的字段分别如下：

* _Issuer_ 字段显示该证书是由<a href="https://www.thawte.com">Thawte</a>发布的，`OU`、`CN`等缩略语和LDAP中的缩略语一样，`OU`表示`Organization Unit`，中文中最能传神的翻译应该是`单位`了，我尚不清楚这些缩略语是在哪个标准中定义；
* _Subject_ 字段显示这个证书是颁发给谁的，其中`CN`的值应该和使用这个证书的网站的域名一致，`CN`表示`Common Name`；
* _Subject Public Key Info_ 字段是公钥信息，_Public Key Algorithm_ 显示这个公钥使用的RSA算法，_RSA Public Key_ 字段中是RSA的公钥了；
* _Signature Algorithm_ 字段是这个证书的签名，用于验证这个证书是不是真的是Issuer（在本例中是<a href="https://www.thawte.com">Thawte</a>）发布的，详见下一节。

```
Certificate:
   Data:
       Version: 1 (0x0)
       Serial Number: 7829 (0x1e95)
       Signature Algorithm: md5WithRSAEncryption
       Issuer: C=ZA, ST=Western Cape, L=Cape Town, O=Thawte Consulting cc,
               OU=Certification Services Division,
               CN=Thawte Server CA/emailAddress=server-certs@thawte.com
       Validity
           Not Before: Jul  9 16:04:02 1998 GMT
           Not After : Jul  9 16:04:02 1999 GMT
       Subject: C=US, ST=Maryland, L=Pasadena, O=Brent Baccala,
                OU=FreeSoft, CN=www.freesoft.org/emailAddress=baccala@freesoft.org
       Subject Public Key Info:
           Public Key Algorithm: rsaEncryption
           RSA Public Key: (1024 bit)
               Modulus (1024 bit):
                   00:b4:31:98:0a:c4:bc:62:c1:88:aa:dc:b0:c8:bb:
                   33:35:19:d5:0c:64:b9:3d:41:b2:96:fc:f3:31:e1:
                   66:36:d0:8e:56:12:44:ba:75:eb:e8:1c:9c:5b:66:
                   70:33:52:14:c9:ec:4f:91:51:70:39:de:53:85:17:
                   16:94:6e:ee:f4:d5:6f:d5:ca:b3:47:5e:1b:0c:7b:
                   c5:cc:2b:6b:c1:90:c3:16:31:0d:bf:7a:c7:47:77:
                   8f:a0:21:c7:4c:d0:16:65:00:c1:0f:d7:b8:80:e3:
                   d2:75:6b:c1:ea:9e:5c:5c:ea:7d:c1:a1:10:bc:b8:
                   e8:35:1c:9e:27:52:7e:41:8f
               Exponent: 65537 (0x10001)
    Signature Algorithm: md5WithRSAEncryption
       93:5f:8f:5f:c5:af:bf:0a:ab:a5:6d:fb:24:5f:b6:59:5d:9d:
       92:2e:4a:1b:8b:ac:7d:99:17:5d:cd:19:f6:ad:ef:63:2f:92:
       ab:2f:4b:cf:0a:13:90:ee:2c:0e:43:03:be:f6:ea:8e:9c:67:
       d0:a2:40:03:f7:ef:6a:15:09:79:a9:46:ed:b7:16:1b:41:72:
       0d:19:aa:ad:dd:9a:df:ab:97:50:65:f5:5e:85:a6:ef:19:d1:
       5a:de:9d:ea:63:cd:cb:cc:6d:5d:01:85:b5:6d:c8:f3:d9:f7:
       8f:0e:fc:ba:1f:34:e9:96:6e:6c:cf:f2:ef:9b:bf:de:b5:22:
       68:9f
```

## 证书的消息签名

前一节提到证书的最后一部分是证书的签名，用于验证证书的真实性。这又是什么原理呢？
先谈这段消息签名的生成方法：

* Issuer（本例中的<a href="https://www.thawte.com">Thawte</a>）先计算证书前一部分内容的MD5值；
* Issuer然后使用他自己的RSA私钥加密。

在本文第一节《使用RSA对消息签名》中提到，用私钥加密的消息同样可以使用其对应的公钥进行解密，因为浏览器已经内置了一些权威的根证书（包括本例中的<a href="https://www.thawte.com">Thawte</a>），这个根证书中包括了其公钥，因此浏览器可以使用<a href="https://www.thawte.com">Thawte</a>的公钥对这段签名进行解密，然后验证解密后的内容是否和证书的前一段内容一致，如果一致，就说明了颁发该证书的服务商的确拥有<a href="https://www.thawte.com">Thawte</a>的私钥，从而证明了这个证书的真实性。

## 小结

最后，因为本文为了理论能讲述得形象一些，不得不类比了一些现实生活，因此我是在愤怒写完这篇博文的，你应该能体会我的心情。《HTTPS与SSL》未完待续，敬请期待！
