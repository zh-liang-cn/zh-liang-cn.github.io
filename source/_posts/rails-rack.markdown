---
layout: post
title: '[Rails代码解读]Rack在Rails中的使用'
excerpt: "今天工作中遇到Rails的一个问题，最后发现是使用的一个叫Rack包版本不兼容Rails2.3引起的，虽然问题很容易就解决了，但是Rack这个包是干什么的却引发了我的兴趣，经过查资料阅读代码，写了这篇博客。Rack是一个中间件，介于Web应用程序和Web服务器之间，为所有的Web服务器都提供了统一的接口，使用Rack构建的Web应用程序能简单换到其他的Web服务器上，因为Rails在底层用到了Rack，所以我们可以在开发的时候使用Webrick，然后通过fastcgi或者ruby_mod发布到nginx或者Apache。 "
date: '2012-08-01 23:57:40 +0800'
categories:
- Ruby
tags:
- ruby
- rails
---
今天工作中遇到Rails的一个问题，最后发现是使用的一个叫Rack包版本不兼容Rails2.3引起的，虽然问题很容易就解决了，但是Rack这个包是干什么的却引发了我的兴趣，经过查资料阅读代码，写了这篇博客。

Rack是一个中间件，介于Web应用程序和Web服务器之间，为所有的Web服务器都提供了统一的接口，使用Rack构建的Web应用程序能简单换到其他的Web服务器上，因为Rails在底层用到了Rack，所以我们可以在开发的时候使用Webrick，然后通过fastcgi或者ruby_mod发布到nginx或者Apache。 

## Rack简介

使用Rack构建的应用程序比Rails要简单多了，当然，功能也简单多了，只有request, response, session, logger等一些基本的组件，不过对于一些简单的应用，足够了。基于Rack的Web应用太简单了，以至于大家都用一句话来描述：

> A Rack application is any Ruby object that responds to the call method, takes a single hash parameter and returns an array containing the response status code, HTTP response headers and the response body as an array of strings.

意思就是，一个包含`call(env)`方法的对象就能做为一个Rack Web应用，参数env是一个hash，方法的返回值是一个列表，包含三个元素：HTTP状态码(200, 500等)，HTTP响应头(Hash)，HTTP响应内容(字符串数组)。

先安装rack和mongrel：

    sudo gem install rack
    sudo gem install mongrel --pre

下面代码演示了如何创建一个用Rack来创建一个Web应用，在控制台中执行下面的代码，然后用浏览器访问 [http://localhost:3000](http://localhost:3000)，即可看到显示 **Hello Rack!** 的页面。

```ruby
#!/usr/bin/env ruby
require 'rubygems'
require 'rack'
class HelloRack
    def call(env)
        [
            200,
            {"Content-Type" => "text/html"},
            ["Hello Rack!"]
        ]
    end
end

Rack::Handler::Mongrel.run(HelloRack.new, :Port => 3000)
```

我们可以将后端的Web服务器Ruby标准库中的WEBrick，只需要将上面代码的最后一行改为：

```ruby
Rack::Handler::WEBrick.run(HelloRack.new, :Port => 3000)
```

当然要是Rack只能支持对于根路径的响应就没有啥意义了，Rack还提供了一个称为<a href="http://rack.rubyforge.org/doc/Rack/Builder.html">Rack::Builder</a>的API，提供了简单的DSL，可以定义简单的URL mapping，使对不同路径的请求由不同的程序来处理，不过，这个和Rails暂时无关，有兴趣请阅读<a href="http://m.onkey.org/ruby-on-rack-1-hello-rack">Ruby on Rack #1 - Hello Rack!</a>和<a href="http://m.onkey.org/ruby-on-rack-2-the-builder">Ruby on Rack #2 - The Builder</a>，或者<a href="http://blog.ixti.net/development/ruby/2011/09/03/understanding-rack-builder/">Understanding Rack Builder</a>。

Rack还提供一个有意思的东西，叫rakeup，使得可以将主要的对请求的响应都放在一个名为`config.ru`文件中，和Rails关系不大，这篇文章可能讲解得更加清楚一些：<a href="http://ruby.about.com/od/rack/a/Using-Rack.htm">Using Rack</a>。

我目前发现的，真正和Rails相关的，是<a href="http://rack.rubyforge.org/doc/classes/Rack/Server.html">Rack::Server</a> API。下面是一个简单的示例：

```ruby
Rack::Server.start(
  :app => proc {|env| 200, {"Content-Type" => "text/html"}, ["Hello Rack!"]]},
  :server => 'webrick',
  :Port => 3030
)
```

`Server#start()`的参数是一个Hash，其参数包括`:server`, `:Host`, `:Port`等，其中`:config`可以是一个`.ru`文件的路径，用于覆盖`:app`的配置，比如下面代码，我们从`config.ru`中读取处理请求的函数，比如下面的代码。

```ruby
Rack::Server.start(
  :config => 'config.ru'
  :server => 'webrick',
  :Port => 3030
)
```

_config.ru_的代码如下所示，仅仅一行。

```ruby
run proc {|env| [200, {"Content-Type" => 'text/html'}, ["Hello Rack!"]]}
```

## Rails中的Rack

对Rails的分析采用倒推的方式，我们启动Rails应用的时候，使用`rails server`命令。这个命令会调用<a href="https://github.com/rails/rails/blob/master/railties/lib/rails/commands/server.rb">railties/lib/rails/commands/server.rb</a>代码，下面是代码的节选。

```ruby
module Rails
    class Server < ::Rack::Server
        def start
            # ...
            super
        end

        def default_options
            super.merge({
                :Port        => 3000,
                :DoNotReverseLookup => true,
                :environment => (ENV['RAILS_ENV'] || "development").dup,
                :daemonize   => false,
                :debugger    => false,
                :pid         => File.expand_path("tmp/pids/server.pid"),
                :config      => File.expand_path("config.ru")
            })
        end
    end
end
```

可以看到，`Rails::Server`继承了`Rack::Server`，在前一节已经了解到，`Rack::Server#start()`方法会启动服务，`Rails:Server`覆盖了`start()`方法，并且在做了一些处理之后使用`super`调用了父类的同名方法，因此调用这个方法同样能启动服务。而且，`Rails:Server`也覆盖了父类的`default_options()`，这里的`super`也表示调用父类的同名方法，其返回值为Hash，使用`Hash#merge()`覆盖了父类的一些配置信息，比如将Rack默认的9292端口改为3000，等等。最后是最为关键的配置信息：`:config => File.expand_path("config.ru")`，意味着`Rails::Server`会读取从Rails App根目录的`config.ru`，然后交给Rack执行。

在Rails App的根目录下面找到`config.ru`，里面一如既往的简单，只有两条语句：

```ruby
require ::File.expand_path('../config/environment',  __FILE__)
run AppName::Application
```

第一句是加载`config/environment.rb`，第二句和前一节的最后一段的代码非常相似，我们现在可以勇敢地猜测，`AppName::Application`中肯定定义了`call(env)`方法。

Rails3的`config/environment.rb`文件也很简单，第一句加载相同目录下的`application.rb`，第二句调用了`AppName::Application`的`initialize!()`方法。(注意，Rails2的启动流程不一样)。

```ruby
require File.expand_path('../application', __FILE__)
AppName::Application.initialize!
```

`config/application.rb`还是的定义依然很简单，继承了`Rails::Application`，仅仅做了一些配置工作，比如禁止在log文件中记录`:password`，启用Rails3新引入的SASS和Coffie Script。

```ruby
module AppName
    class Application < Rails::Application
        config.encoding = "utf-8"
        config.filter_parameters += [:password]
        config.active_record.whitelist_attributes = true
        config.assets.enable = true
        config.assets.version = '1.0'
    end
end
```

因此，进一步找到<a href="https://github.com/rails/rails/blob/master/railties/lib/rails/application.rb">railties/lib/rails/application.rb</a>，这个文件定义了`Rails::Application`，而且终于看到了我们预测中的`call(env)`。当然，除此以外Rails做的更多，比如定义了<a href="https://github.com/rails/rails/blob/master/railties/lib/rails/rack/logger.rb">Rails::Rack::Logger</a>用来替代Rack自身的日志系统。

```ruby
module Rails
    class Application < Engine
        def call(env)
            env["ORIGINAL_FULLPATH"] = build_original_fullpath(env)
            super(env)
        end
    end
end
```

另外，`Rails::Application`的父类<a href="https://github.com/rails/rails/blob/master/railties/lib/rails/engine.rb">Rails::Engine</a>是Rails的启动配置的核心所在，一次加载了config/routes.rb、app/views等信息。

## 总结

总的来说，Rails首先加载了`config/application.rb`中定义了`AppName::Application`，然后调用其`initialize!()`方法执行一些初始化工作，最后使用Rack的`run AppName::Application`运行整个应用程序。Rails也通过Rack可以很方便的部署于Apache、nginx、lighttpd等各种服务器，包括Ruby自带的Webrick，以及mongrel等。要更深入的了解Rack需要进一步阅读参考中的链接。

## 参考

* <a href="http://ruby.about.com/od/rack/a/What-Is-Rack.htm">What's Rack</a>
* <a href="http://ruby.about.com/od/rack/a/Using-Rack.htm">Using Rack</a>
* <a href="http://chneukirchen.org/blog/archive/2007/02/introducing-rack.html">Introducing Rack</a>
* <a href="http://rack.rubyforge.org/doc/SPEC.html">Rake Spec</a>
* <a href="http://m.onkey.org/ruby-on-rack-1-hello-rack">Ruby on Rack, Hello Rack</a>
* <a href="http://m.onkey.org/ruby-on-rack-2-the-builder">Ruby on Rack, The builder</a>
* <a href="http://blog.ixti.net/development/ruby/2011/09/03/understanding-rack-builder/">Understanding Rack Builder</a>
* <a href="http://rack.rubyforge.org/doc/">rack doc</a>

