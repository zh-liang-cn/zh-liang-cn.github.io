---
layout: post
title: JavaScript Date
excerpt: |+
  今天工作中遇到JavaScript的Date计算和格式化的问题，顺便整理了一下JavaScript中的Date的相关API，试做简单记录，代码均在<a href=\"http://nodejs.org/\">Node.js</a>上测试通过。
date: '2012-09-11 16:26:48 +0800'
categories:
- JavaScript
tags:
- javascript
- web
---

**2013-07-03 更新：**

> 为了解决本文中的关于Date的一些坑，我创建了一个项目<a href="https://github.com/zhlwish/enhanced-jsdate">enhanced-jsdate</a>，非常简单，有<a href="https://github.com/zhlwish/enhanced-jsdate/blob/master/src/enhanced-jsdate.coffee">CoffeeScript</a>和<a href="https://github.com/zhlwish/enhanced-jsdate/blob/master/lib/enhanced-jsdate.js">JavaScript</a>两个版本。

今天工作中遇到JavaScript的Date计算和格式化的问题，顺便整理了一下JavaScript中的Date的相关API，试做简单记录，代码均在<a href="http://nodejs.org/">Node.js</a>上测试通过。

## Date构造函数

```javascript
// 当前时间
// Tue Sep 11 2012 15:37:46 GMT+0800 (中国标准时间)
new Date();
// Fri Jan 02 1970 08:00:00 GMT+0800 (中国标准时间)
new Date(3600 * 24 * 1000);
```

除了构造函数外，`Date.UTC()`也用于构造日期，但是并不返回Date，而是返回从1970-01-01开始的毫秒数，比如：

```javascript
// 1328054400100, 即2012-01-01 00:00:00 0100
Date.UTC(2012, 1, 1, 0, 0, 0, 100);
```

需要注意的是，`Date()` 函数返回日期字符串，比如：

```javascript
// date是字符串：'Tue Sep 11 2012 15:34:48 GMT+0800 (中国标准时间)'
var date = Date();
```

## 字符串转日期

Date.parse(string)方法并 _不是返回Date，而是返回整数_。

```javascript
// 1120752000000, 即2005-07-08
Date.parse("Jul 8, 2005");
Date.parse("2005-07-08");
Date.parse("2005/07/08");
```

## 日期转字符串

```javascript
toString();         // Tue Sep 11 2012 15:09:01 GMT+0800
toTimeString();     // 15:09:34 GMT+0800
toDateString();     // Tue Sep 11 2012
toGMTString();      // Tue, 11 Sep 2012 07:10:05 GMT
toUTCString();      // Tue, 11 Sep 2012 07:10:17 GMT
toLocaleString();   // 2012年9月11日 15:10:31
toLocaleTimeString();   // 15:10:45
toLocaleDateString();   // 2012年9月11日
```

## Get系列方法

Date提供了一系列Get和Set方法，可以获取和设置年份、月份、日期、小时等等信息，并且提供了本地时间和UTC时间两套方案。

获取本地时间方法如下：

```javascript
var date = new Date();
date.getDate();     // 从 Date 对象返回一个月中的某一天 (1 ~ 31)。
date.getDay();      // 从 Date 对象返回一周中的某一天 (0 ~ 6)。
date.getMonth();    // 从 Date 对象返回月份 (0 ~ 11)。
date.getFullYear(); // 从 Date 对象以四位数字返回年份。不用使用getYear()。
date.getHours();    // 返回 Date 对象的小时 (0 ~ 23)。
date.getMinutes();  // 返回 Date 对象的分钟 (0 ~ 59)。
date.getSeconds();  // 返回 Date 对象的秒数 (0 ~ 59)。
date.getMilliseconds(); //返回 Date 对象的毫秒(0 ~ 999)。
```

另外，`date.getTime()`方法返回从1970-01-01开始到现在的毫秒数。

```javascript
date.getTime(); // 返回1970年1月1日至今的毫秒数。
```

Get系列获取本地时间的方法和获取UTC时间的方法一一对应，如

```javascript
date.getUTCDate();
```

## Set系列方法

Set系列方法和Get系列方法一一对应，如：

```javascript
date.setDate(24);
date.setUTCDate(24);
```

同样，`date.setTime(time)`方法以毫秒数设置Date对象，和`new Date(time)`的作用一样。

## Date之间的计算

Date之间的计算实际上是毫秒数的计算，加法除外，加法实际上是字符串连接，比如

```javascript
// Tue Sep 11 2012 15:40:45 GMT+0800 (中国标准时间)
var d = new Date();
// Tue Sep 11 2012 15:40:50 GMT+0800 (中国标准时间)
var b = new Date();
// -5781
console.log(d - b);
// 'Tue Sep 11 2012 15:40:45 GMT+0800 (中国标准时间)Tue Sep 11 2012 15:40:50 GMT+0800 (中国标准时间)'
console.log(d + b);
// 0.999999995709353
console.log(d / b);
// 1.8153499959824195e+24
console.log(d * b);
```

## Date到字符串的自定义格式转换

JavaScript并没有提供像Ruby的`strftime()`这样的方法，如果需要某种格式的日期字符串只能自己实现。比如下面方法实现将日期转化成_yyyy-mm-dd_形式的字符串。

```javascript
Date.prototype.toHyphenDateString = function() {
    var year = this.getFullYear();
    var month = this.getMonth() + 1;
    var date = this.getDate();
    if (month < 10) { month = "0" + month; }
    if (date < 10) { date = "0" + date; }
    return year + "-" + month + "-" + date;
};
var date = new Date();
// 输出2012-09-11
console.log(date.toHyphenDateString());
```

如果需要将日期转换为_3天之前_或者_4小时之前_这样的字符串，可以使用<a href="http://timeago.yarp.com/">JQuery timeago插件</a>。

## 参考

* <a href="http://www.w3school.com.cn/js/jsref_obj_date.asp">JavaScript Date 对象参考手册</a>
* <a href="http://timeago.yarp.com/">JQuery timeago插件</a>
* <a href="http://nodejs.org/">Node.js</a>

