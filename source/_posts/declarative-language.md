---
title: 什么是声明式编程语言
date: 2018-05-18 22:08:12
excerpt: |+
  最近了解了neo4j的Cypher语言以及drools的规则语言，这两种语言都是“声明式语言”，因此有必要弄清楚声明式语言究竟是什么。
tags: 编程语言
---

最近了解了neo4j的Cypher语言以及drools的规则语言，这两种语言都是“声明式语言”，因此有必要弄清楚声明式语言究竟是什么。

## 定义

声明式语言（Declarative Language）是相对于命令式语言（Imperative Language）而言的，其实我们天天在用这两类语言。

命令式语言典型的例子是C、Java、PHP等语言，其特点是提供了if-else、foreach等基本逻辑控制命令（英语中成为primitives），使得我们可以通过这些基本单元来描述完成任务的过程。

声明式语言的典型例子是SQL或者XQuery（可能比较少用，但是XPath是XQuery的一部分，应该大部分人都知道的），声明式语言是一种更为高级（Higher level）的语言，使用的时候并不需要去描述具体实现任务的步骤，只需要描述希望达成的目的。比如，在用SQL的时候，你只需要告诉DB希望查询什么数据（如`select * from books`），而DB会执行相应的处理流程来完成任务。DSL也是非常典型的声明式语言。

## 区别

说到这里，可能你还是不明白这两者到底有什么不同。你会说诸如SQL这样的声明式语言也有`foreach`这种基本逻辑控制命令。是的，本质上而言，命令式语言是描述过程，而声明式语言是描述目标。但是站在不同的层次，过程可能也是目标，而目标也可能成为过程。

后来我想到，产品经理写文档就是天然的声明式编程，他们更多的是描述希望达到的目标；而作为程序员，我们写下的每一行代码就是命令式编程。

这是一行产品需求：

```
需求名称：用户第一次购买了铅笔后发送一张削笔刀现金券
场景：用户购买铅笔支付成功结果页
判断条件：用户是第一次购买铅笔
具体动作：给用户发送一张满5减1的削笔刀现金券
```

这个需求用抽象语言（DSL）描述成：

```
in_scenario: 
    pay_result_page
test:
    user_buy_pencil && user_buy_pencil_for_the_first_time
action:
    send_a_5_plus_1_coupon
```

但是当我们来实现的时候我们会做：

```java
user = UserService.getById(uid)
orderItem = OrderService.getById(oiid);
if (this.buyPencilForTheFirstTime(user) 
        && orderItem.getCategory().equals("pencil")) 
    this.sendCoupon(uid, couponId);
}
```

你会发现在声明式语言中，你需要用到`user_buy_pencil`这样的语法，而在命令式语言中，我们使用了更基本的状态判断。所以，声明式编程用到的更多是Problem Domain（问题域）语法，而命令式编程更多的是用到Primitive（基本控制单元）语法。这个例子中的，`in_scenario`也可以认为是产品经理专用语言，用于描述需求。

## 具体的例子

多年以前阅读[《Learn You a Haskell for Great Good!》](http://learnyouahaskell.com/chapters)这本书读到一个例子，题目是：求边长小于10并且是整数的直角三角形。

![](https://zh-liang-cn.oss-cn-hangzhou.aliyuncs.com/images/20200103232628.png)

拿到这个问题，一看挺简单啊，判定直接三角形就是用勾股定律嘛，"Life is short, you need Python"：

```python
for a in range(1, 11):
    for b in range(1, a+1):
        for c in range(1, b+1):
            if b**2 + c**2 == a**2:
                print("a=%d, b=%d, c=%d" % (a, b, c))
```

然而，我们看看在Haskell语言中的实现：

```haskell
main = print [ (a,b,c) | c <- [1..10], 
                         b <- [1..c], 
                         a <- [1..b], 
                         a^2 + b^2 == c^2] 
```

鹅妹子嘤！是不是有点像数学中学习的集合表达式？对，就是描述一下问题，就可以自动求解，这就是一个非常典型的声明式编程的例子。

![](https://zh-liang-cn.oss-cn-hangzhou.aliyuncs.com/images/20200103232733.png)

如果你不信，点击这里来执行一下：https://repl.it/@LiangZhou1/SatisfiedGlassParallelport

