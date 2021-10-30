---
layout: post
title: '[Rails代码解读]ActiveSupport::Concern'
excerpt: |+
  模块是方法和常量的集合，模块和类一样，其中的可以包括两种方法：实例方法(Instance Method)、模块方法(Module Method)。当一个类混入(Mixin，本文中称混入)一个模块时，模块中的实例方法会成为该类的方法(具体是类的类方法还是实例方法，取决于是include还是extend)。但是模块方法会被忽略，而且，模块方法可以虽然定义在模块中，但是可以直接调用。
date: '2012-07-23 00:49:01 +0800'
categories:
- Ruby
tags:
- ruby
- rails
---
Ruby的模块是一种类型，因此也有模块方法（相当于类方法）和实例方法（相当于实例方法），模块方法可以直接调用，而实例方法必须通过_include_混入到一个类中，才能由一个对象实例调用。

## 模块方法和实例方法

模块是方法和常量的集合，模块和类一样，其中的可以包括两种方法：实例方法(Instance Method)、模块方法(Module Method)。当一个类混入(Mixin，本文中称混入)一个模块时，模块中的实例方法会成为该类的方法(具体是类的类方法还是实例方法，取决于是include还是extend)。但是模块方法会被忽略，而且，模块方法可以虽然定义在模块中，但是可以直接调用。

下面代码定义了Mod模块以及Clz类，Clz类include模块Mod

```ruby
module Mod
    def func_instance
        puts 'Call: Instance Method'
    end
    def self.func_module
        puts 'Call: Module Method'
    end
end

class Clz
    include Mod
end

puts "Class Methods: #{ Clz.methods.grep /^func/ }"
puts "Instance Methods: #{ Clz.instance_methods.grep /^func/ }"
```

输出为：

    Class Methods: []
    Instance Methods: [:func_instance]

模块中定义的实例方法和模块方法的调用方法如下：

```ruby
# 直接调用模块的模块方法
Mod::func_module
# 调用模块的实例方法必须创建Clz对象
Clz.new.func_instance
```

以上代码输出为：

    Call: Instance Method
    Call: Module Method

## extend和include

前节中定义的Clz类include混入了模块Mod，现在创建一个新类ClzNew，使用extend混入模块Mod。

```ruby
class ClzNew
    extend Mod
end
puts "Class Methods: #{ ClzNew.methods.grep /^func/ }"
puts "Instance Methods: #{ ClzNew.instance_methods.grep /^func/ }"

# func_isntance为ClzNew的类方法
ClzNew.func_instance
```

同样，打印ClzNew的类方法和实例方法，输出结果为：

    Class Methods: [:func_instance]
    Instance Methods: []
    Call: Instance Method

从上面的输出可以得到两点：

* 模块方法`func_module`在混入的时候仍然被忽略
* 使用extend混入时，模块中的方法成为类的类方法，而使用include混入时，模块中的方法成为类的实例方法

## included和append_features

当一个模块被其他类或者模块include时，`Module#included(parent)`会被调用，这个方法可以认为是一个回调函数(Callback)。而对于`Module#append_features`则是当include或者extend语句被执行时被调用，真正的将一个模块混入到另一个类或者模块中，不是回调函数，因此一个模块中覆盖(Override)`Module#append_features`时，必须使用`super`语句去真正的执行模块混入的操作，否则模块混入不会被真正的执行。

下面的语句，为之前定义的Mod模块添加了`included`方法，然后将其混入`ClzParent`中。

```ruby
def Mod.included(parent)
    puts "#{self} is included in #{parent}"
end
class ClzParent
  include Mod
end
puts "Class Methods: #{ ClzParent.methods.grep /^func/ }"
puts "Instance Methods: #{ ClzParent.instance_methods.grep /^func/ }"
```

输出如下，其中第一句证实了`included`方法在ClzParent类include模块Mod是被调用，而且`func_instance`成为了ClzParent类的实例方法。

    include: Mod is included in Clz
    Class Methods: []
    Instance Methods: [:func_instance]

接着对`append_features()`进行实验，需要注意的是`append_features()`方法中真正执行混入的`super`语句被注释了。因此可以预见，`Mod#func_instance()`不会成为类ClzParentNew的实例方法，而后面的输出也证实了这一点。输出也证明，`append_features()`被先调用，`include`被后调用。实际上，`include()`是在`append_features()`被调用的。

```ruby
def Mod.included(parent)
    puts "#{self} is included in #{parent}"
end

def Mod.append_features(parent)
  puts "append_features: #{self} is included in #{parent}"
  # super
end

class ClzParentNew
  include Mod
end

puts "Class Methods: #{ ClzParentNew.methods.grep /^func/ }"
puts "Instance Methods: #{ ClzParentNew.instance_methods.grep /^func/ }"
```

输出为：

    append_features: Mod is included in ClzParentNew
    include: Mod is included in ClzParentNew
    Class Methods: []
    Instance Methods: []

将上面代码中的super语句取消注释，`Mod#func_instance()`成为了类ClzParentNew的实例方法，输出变为如下：

    append_features: Mod is included in ClzParentNew
    include: Mod is included in ClzParentNew
    Class Methods: []
    Instance Methods: [:func_instance]

因此，在通过append_features加入对include回调时，千万不能忘记用`super`来实现真正的include操作。

## Ruby混入的惯用方法

为了将代码更好的模块化，Ruby的提供了不同类型的混入(include和extend)来实现混入类方法和实例方法。但是不管是类的类方法还是实例方法，最终都是模块的实例方法，因此，为了同时实现类方法和实例方法的混入，必须定义两个模块，然后分别include和extend，如下：

```ruby
module ClassMethods
end

module InstanceMethods
end

class Foo
    extend ClassMethods
    include InstanceMethods
end
```

可能是觉得这种方法将模块的粒度分得太开(模块的划分粒度的确是个经常遇到问题)，因此在Ruby中出现了下面一种惯例，将类方法定义在一个子模块中，使用`included()`回调，这个子模块常常命名为`ClassMethods`。

```ruby
module Mod
    def self.included(base)
        base.extend(ClassMothods)
    end
    module ClassMethods
        # 类方法定义
    end
    #实例方法定义
end

class Bar
    include Mod
end
```

上面代码将实现了在Mod中同时定义类方法和实例方法。需要注意的是，即使在ClassMethods中定义的是类方法，也不能在方法前加`self`，否者，该方法会成为模块方法，混入时会被自动忽略了。

虽然这是一种很好的方法，但是每次都要编写这些代码，而且和DRY(Don't Repeat Yourself])原则违背。

## 模块混入的依赖问题

正常情况下，Ruby会自动解决模块的依赖，比如B模块include模块A，C类include模块B，那么C类也会自动混入模块A，下面的代码验证了这一点。

```ruby
module A
end

module B
    include A
end

module C
    include B
end

puts C.included_modules  # 输出: [B, A, Kernel]
```

但是存在一个问题，比如下面的代码，A模块中定义了类方法`method_injected_by_a()`，在模块B的回调函数`self.included(base)`中调用了模块A中的方法，到目前为止没有任何问题，但是当C类include模块B时，出现了异常"undefined method 'method_injected_by_a' for C:Class (NoMethodError)"。

```ruby
module A
  def self.included(base)
    base.extend ClassMethods
  end

  module ClassMethods
    def method_injected_by_a
      puts 'method_injected_by_a'
    end
  end
end

module B
  include A

  def self.included(base)
      base.method_injected_by_a
  end
end

class C
  include B
end
```

这是因为模块B在include模块A时，直接将模块A中定义的`#method_injected_by_a()`方法include为模块B的模块方法，因此，类C中并不存在这个方法了。我们可以用如下的代码进行验证。

```ruby
module A
  def self.included(base)
    base.extend ClassMethods
  end
  module ClassMethods
    def method_injected_by_a
      puts 'method_injected_by_a'
    end
  end
end

module B
  include A
end

B::method_injected_by_a  # 输出：method_injected_by_a
```

最后一句话调用了B模块中include的A模块中定义的方法，真的调用成功了。

## ActiveSupport::Concern

Rails的ActiveSupport::Concern正是为了解决本文第5节和第6节的解决这个问题而产生的。而且，这个类的代码不长：

```ruby
module Concern
  def self.extended(base) #:nodoc:
    # 创建实例变量@_dependencies，初始化为[]
    base.instance_variable_set("@_dependencies", [])
  end
  def append_features(base)
    # 如果base定义了@_dependencies，则把当前模块加入base类的@_dependencies
    if base.instance_variable_defined?("@_dependencies")
      base.instance_variable_get("@_dependencies") << self
      return false
    else
      # 如果base不是self的子类直接返回
      return false if base < self
      # 依次调用base.include方法include @_dependencies中的类
      @_dependencies.each { |dep| base.send(:include, dep) }
      # 调用父类的append_features()，真正的执行mixin
      super
      # 调用base.extend, 将ClassMethods中定义的方法extend为base的类方法
      base.extend const_get("ClassMethods") if const_defined?("ClassMethods")
      # 同时将直接执行@_included_block中的代码，在base的类上下文中
      base.class_eval(&amp;@_included_block) if instance_variable_defined?("@_included_block")
    end
  end
  # NOTE: 和self.included()不一样
  def included(base = nil, &amp;block)
    if base.nil?
      @_included_block = block
    else
      super
    end
  end
end
```

从上面的代码和注释中可以看出：

* Concern中定义的`self.extended()`告诉我们，必须通过extend混入，而不是include；
* 声明类方法的模块的名字为ClassMethods，如果拼写错误，会直接被忽略；
* Concern类为每个通过extend混入它的类提供了一个included类方法(Class Macro)，可以传入一个块(Block)，从而实现回调。

示例如下，首先需要导入active_support/concern(默认本文的读者以及安装了rails，如果没安装，请再控制台执行`sudo gem install rails`安装Rails)。

```ruby
require 'active_support/concern'
module A
  extend ActiveSupport::Concern
  module ClassMethods
    def method_injected_by_a
      puts 'method_injected_by_a'
    end
  end
end

module B
  include A
  extend ActiveSupport::Concern
  included do
    puts 'include callback in B'
  end
end

class C
  include B
end

C.method_injected_by_a
```

执行以上代码，输出为：

    include callback in B
    method_injected_by_a

我们发现：

* 没有手动的为模块A定义`self.included(base)`也实现了将A::ClassMethods混入为C类的类方法，代码变干净了；
* 对于仍然有需要includ回调的需求，可以直接使用ActiveSupport::Concern中定义的included方法来满足；
* 在模块A中定义的方法`method_injected_by_a()`没有变成模块B的模块方法，而仍然是C类的类方法，依赖问题也被解决了。

## 总结
Ruby中类模型和传统的面向对象语言C++、Java、C#不同，不过看起来和JavaScript到有半分相似。令人动心的是Ruby本身提供了很强大的元编程(Meta Programming，在Java里面叫反射？)、以及Mixin这样的特性，让实现更加方便、代码更加简洁。Ruby中的模块听起来和Java中的包相似，看起来和抽象类相似，但是实际上完全不同，倒是和C中的宏比较相似。

Ruby中的混入有两种方式：include和extend，前者将模块中的实例方法混入为目标模块或类的实例方法，后者将模块中的实例方法混入为目标模块或类的类方法。模块的模块方法会在混入时被忽略。ruby的Module类分别提供了`self.included()`和`self.extended()`两种回调方法，不论使用哪一种混入方式，都是通过Module类的`append_features()`实现真正的混入的。

## 参考

* <a href="http://ruby-doc.org/core-1.9.3/Module.html">Ruby Module Doc</a>
* <a href="http://book.douban.com/subject/4086938/">Metaprogramming Ruby: Program Like the Ruby Pros</a>
* <a href="http://blog.jayfields.com/2006/12/ruby-instance-and-class-methods-from.html">Jay Fields' Thoughts: Ruby: instance and class methods from a module</a>
* <a href="http://juixe.com/techknow/index.php/2006/06/15/mixins-in-ruby/">Ruby Mixin Tutorial</a>
* <a href="http://www.devalot.com/articles/2008/09/ruby-singleton">Understanding Ruby Singleton Classes - Devalot</a>

