---
layout: post
title: JavaScript中的this
excerpt: JavaScript是一门极为灵活的语言，其神奇之处在于其面向对象的实现方式和函数。而函数往往又认为是JavaScript中最Amazing（或者翻译为惊艳？）的地方。<a
  href="http://www.jquery.com">JQuery</a>这样强大的库正是建立于JavaScript灵活的函数之上的。而要正确的使用JavaScript也必须对对象和函数有更深入的了解，笔者在开发JQuery插件的过程中，遇到了很多疑惑，进而得到了一些关于对象和函数的心得，希望能对读者有用。
date: '2012-07-08 17:30:17 +0800'
categories:
- JavaScript
tags:
- web
- jquery
- javascript
---

JavaScript是一门极为灵活的语言，其神奇之处在于其面向对象的实现方式和函数。而函数往往又认为是JavaScript中最Amazing（或者翻译为惊艳？）的地方。<a href="http://www.jquery.com">JQuery</a>这样强大的库正是建立于JavaScript灵活的函数之上的。而要正确的使用JavaScript也必须对对象和函数有更深入的了解，笔者在开发JQuery插件的过程中，遇到了很多疑惑，进而得到了一些关于对象和函数的心得，希望能对读者有用。

## JavaScript中的对象

### 声明类和创建对象

JavaScript是一门纯粹的面向的语言，所有的对象都继承自全局类`Object`，创建类的方式为：

```javascript
function User() {
    var name;
}
```

看起来是声明了一个函数，实际上是定义了类的构造函数，用`new`运算符即可创建该类的对象：

```javascript
var user = new User();
```

创建一个对象除了上面方法外，还有下面简单方法：

```javascript
var me = {
    name = 'my name';
};
```

上面这种方式如下的代码实现了相同的作用：

```javascript
var me = new Object();
me.name = 'my name';
```

### 点（.）运算符

点（`.`）号在JavaScript中是一个运算符，运算符左边是一个对象，右边是一个标识符（即对象的属性名称），因为对象的属性仍然可以是对象，因此在JQuery中常见的`$.fn.jqModel`这样的字符串就表示了JQuery对象`$`的`fn`属性的`jqModel`属性，JQuery使用这样的方式来定义名称空间，避免变量名称的冲突。

另外，如果对象中没有指定的属性，JavaScript不会把这个当成一个错误，而只是返回很普通的`undefined`作为表达式的值，因此我们如果需要声明`a.b.c`这样的名称空间则需要按如下方式：

```javascript
var a = {};
a.b = {};
a.b.c = {};
console.log(a.b.c); //输出：{}
```

当我们不需要这个名称空间时，可以使用`delete`运算符依次删除对象的属性。

## Global全局对象

JavaScript中定义了全局对象，和`Object`、`Function`这些抽象的类不同，全局对象不是一个类，而是一个对象实例，所有的全局变量都是这个全局对象的属性。`Object`、`Function`、`Date`等核心类也定义在全局对象中。**所有直接定义的函数都是全局对象的方法**。如：

```javascript
var name = 'zhlwish';
function sayHello(name) {
    console.log("hello, " + name);
}
```

在<a href="http://www.jslint.com">JSLint</a>.中，可以看到以上代码定义了两个全局属性：

    global
        name, sayHello

另外，如果使用隐式声明变量的方式声明变量(即不使用`var`关键字声明变量)，那么这个变量会被声明为全局变量，比如下面代码，在函数`sayHello()`中隐式声明了`name`变量，该变量被声明为全局变量，因此在调用了`sayHello()`方法后，即使在`sayHello()`方法外仍然能够获得`name`变量的值。关于`var`关键字详见：<a href="http://coolshell.cn/articles/7480.html">Javascript 中的 var</a>。

```javascript
function sayHello() {
    firstName = "Jack";
    var lastName = "Baul";
}
sayHello();
console.log(firstName); //"Jack"
console.log(lastName); //ReferenceError: lastName is not defined.
```

## JavaScript函数

之前有提到JavaScript是一门纯粹的面向对象的编程语言，因此在JavaScript中函数本身也是一个对象，正如在Java中类本身也是一个类一样，因此，函数对象可以被赋值给其他变量，函数通过括号运算符(`()`)进行调用。但是JavaScript中并没有class关键字。函数承担了定义类和函数本身两种角色，这也导致了本文的主题，因为在分别担任这两种角色的时候，其函数体中的`this`表现并不一样。

### 独立的函数

独立定义的函数实际上是定义于全局对象之中，成为全局对象的一个方法，因此在该方法中，`this`关键字是全局对象的引用。

```javascript
function sayHello(name) {
    console.log(this.toString()); // 输出: [object global]
    return "Hello, " + name;
}
```

需要注意的是，即使嵌套定义在函数中，只要这个函数没有绑定（Binding）任何其他对象（即不是任何其他对象的属性），那么这个函数仍然是独立的函数，其函数体内的`this`是全局对象的引用，下面的代码中，在`pFunc()`中声明了`cFunc()`，但是`cFunc()`中的this仍然是全局对象。

```javascript
function pFunc() {
    function cFunc() {
        console.log(this.toString()); //输出: [object global]
    }
    //调用嵌套函数
    cFunc();
}
//调用父亲函数
pFunc();
```

### 作为构造方法的函数

用函数来定义类，而这个函数就是类的构造函数，在本文之前就有叙述。因此在构造函数体内，`this`是对象本身的引用，如下面代码，`User()`是构造方法，在其函数体内`this`是`user`对象实例的引用，`name`是对象`user`的属性。

```javascript
function User(name) {
    console.log(this.toString);
    this.name = name;
};
var user = new User("my name"); //输出: [object Object]
console.log(user.name); //输出: my name
```

### 匿名函数

JavaScript允许定义匿名函数，如下代码所示，但是如果该匿名函数不赋值给任何变量，这个函数将永远得不到运行的机会，因为没办法调用这个匿名函数。

```javascript
function() {
    console.log("匿名函数");
}
```

我们可以将匿名函数赋值给一个对象，然后通过括号运算符来调用这个匿名函数：

```javascript
var func = function() {
    console.log("匿名函数");
};
func(); //输出：匿名函数
```

还有另外一种方法调用匿名函数，而这种方法在JQuery中非常常见，被称为<a href="http://benalman.com/news/2010/11/immediately-invoked-function-expression/">Immediately-Invoked Function Expression (IIFE)</a>，先声明一个匿名函数，然后通过括号运算符立即调用它。

```javascript
(function(name) {
    console.log("匿名函数: " + name);
})("my name");
```

## 执行上下文

最后回到本文的主题，JavaScript中的`this`和其他语言中不一样，他是函数执行时当前对象的引用，即执行上下文（Execution Context）的引用。不论定义一个函数还是声明一个变量，当我们没有显示的指定该函数属于哪一个对象时，它就属于全局对象，这个函数体中的`this`就被绑定到了全局名称空间。下面的代码，首先创建`obj`对象，这是一个全局对象，接着为`obj`对象声明一个方法`method()`，因为已经明确指定了`method()`绑定到了`obj`对象中，因此在`method()`方法体内，`this`指向当前对象`obj`。而在`method()`方法体内定义的匿名方法（变量`count`所引用的方法）是独立的方法，因此在此方法体内`this`又是全局对象的引用。

```javascript
//声明并创建全局变量obj对象
obj = { };
//将method方法绑定到obj对象
obj.method = function() {
    //this是obj对象的引用
    console.log(this.toString());
    var count = function() {
        //因为function是一个独立的变量
        console.log(this.toString());
    };
    count();
};
obj.method();
```

正是由于上述情况的出现，我们不得不常常编写如下的代码，我们在`method()`方法内将`this`赋值为`that`，在匿名函数中使用`that`引用`obj`对象本身。

```javascript
obj = {};
obj.method = function() {
    var that = this;
    var count = function() {
        console.log(that.toString());
    };
    count();
};
obj.method();
```

有了执行上下文的概念，下面的代码为什么输出不一致也就不难理解了，因为前者是一个函数，默认被绑定到全局变量，而后者是创建一个对象，this被绑定到了该对象上：

```javascript
function a() {
    console.log(this.toString());
}
a(); //输出：[object global]
var obj = new a(); //输出：[object Object]
```

## this和var

在上一段代码中，因为`count`是`method()`方法内部定义的变量，因此在方法体外部无法引用，所以`count`所指向的匿名方法也没办法调用。但是如果我们希望能在`mothod()`方法外也能调用该方法，应该怎么做呢？至少我们可以将`count`变量绑定到`obj`对象，使`count`成为`obj`对象的一个属性，然后调用，如下所示：

```javascript
obj = {};
obj.method = function() {
    this.count = function() {
        console.log('invoking count');
    };
};
obj.method();
obj.count(); // 输出: invoking count
```

需要注意的是，在`method()`方法中定义的方法是obj对象的方法，只能通过`obj.count()`调用，而不能通过`obj.method.count()`调用。

## 总结

JavaScript这么语言对于我们学过的Java、C++等主流面向对象的语言有很大的差异，但是也为我们打开了另外一扇窗口。其`this`关键字在不同的使用场景下引用的对象不同，和JavaScript的面向对象设计有很大的关系，理解了这些知识，对于在什么情况下使用`this`也就不难掌握了，同时也需要记住，分析`this`所引用的对象要动态的看，不能仅仅靠分析静态的代码就能得出结论。

## 参考

* <a href="http://book.douban.com/subject/10875361/">Pro Jquery</a>
* <a href="http://book.douban.com/subject/5252901/">JavaScript Patterns</a>
* <a href="http://book.douban.com/subject/1232061/">JAVASCRIPT权威指南</a>

