---
layout: post
title: Java工作流引擎：jBPM、Activiti以及SWF
excerpt: |+
  这只是一篇非常粗浅的记录我对工作流引擎认识的文章。知道工作流引擎是很久之前了，但是一直都没有机会尝试，一是没有业务上的需要，二是感觉工作流入门不容易。最近，项目中用到了一点工作流的东西，虽然我没有具体参与，但是了解一下还是好的。于是抽周末两天时间读了一些文章和jBPM以及Activiti的User Reference，本文做一下记录。SWF指的是Amazon Simple Workflow。
date: '2013-07-06 11:19:25 +0800'
categories:
- Java
tags: []
---
这只是一篇非常粗浅的记录我对工作流引擎认识的文章。
知道工作流引擎是很久之前了，但是一直都没有机会尝试，一是没有业务上的需要，二是感觉工作流入门不容易。最近，项目中用到了一点工作流的东西，虽然我没有具体参与，但是了解一下还是好的。于是抽周末两天时间读了一些文章和jBPM以及Activiti的User Reference，本文做一下记录。SWF指的是Amazon Simple Workflow。

## 工作流引擎是什么？

“工作流引擎”这个名字听起来很吓人（“引擎”这两个字眼总是能吓人的），我之前就是被吓着了的程序员中的一个。百度词条<a href="http://baike.baidu.com/view/1636259.htm">工作流引擎</a>也很难懂。
我们经常做一些程序，比如用户A填一张表，提交后，会给另一个用户B（通常是另一类较色）审核，他们觉得没有问题就确定，最后给原来A用户发送一封邮件。在实现这一类系统时我们会设计一张任务表，这个表中有一列成为Status(状态)：用户提交后状态是0，审核通过后状态是1，审核没通过状态是2。但是这样设计会有一些扩展性的问题，比如：

* 我需要知道某表单的历史信息：什么时候由谁提交、什么时候被审核通过、被谁审核通过等
* 我需要扩展或者改动流程：A用户提交表单后，B用户希望能收到邮件提醒等
* 我需要定时执行一些任务：为A用户提交表单设置截止时间，提前截止时间一天发送邮件通知

> 这些需求用土鳖的方式是都可以实现，我以前在学校工作的时候，买设备走学校的采购流程系统，我就亲眼看到一个工作人员打开SQL Server去数据库中查询这个订单是什么时候下的。

当然，这样的系统做得多了，就会对这些进行归纳抽象，比如:

* 将任务状态表和历史记录表抽象出来成为 **TaskService** 和 **HistoryService** 模块
* 将定时任务以及抽象成 **BusinessCalendar** 模块
* 将用户管理的部分抽取出来成为 **Identity** 模块
* 将发送邮件抽象成 **EmailTask** 模块，使用的时候只需要配置一下收件人和内容即可
* 将需要调用Java类处理业务逻辑功能抽象成 **Action**，使用时配置一下具体调用哪一个Java类
* 最后做一个流程管理的通用界面，能实时监控流程的执行情况

这些也都是jPBM以及Activiti等工作流引擎的核心模块。

## 工作流引擎的基本概念

上一节所述A和B参与的工作流的例子，很好的描述了一个A和B的工作流程，我们用一种语言来将这种模式描述出来，jBPM5之前版本用的是jPDL，现在大家都用<a href="http://zh.wikipedia.org/wiki/%E4%B8%9A%E5%8A%A1%E6%B5%81%E7%A8%8B%E5%BB%BA%E6%A8%A1%E6%A0%87%E8%AE%B0%E6%B3%95">BMPN</a>，两者大同小异，都是用XML来描述流程，也都有可视化设计器支持，不过BMPN是行业标准。
编写好流程定义（Process Defintion）文件以及相关的Java类后，就可以部署到引擎中，每一次执行称为一个流程实例（Process Instance）。

工作流有版本的概念，jBPM和Activiti上传一个新的版本后，版本号会增加1，旧版本还没执行完的流程实例还会继续执行。SWF的版本是个字符串，随意指定好了，这样也很好，字符串名称更明确。
前面提到工作流抽象出来的各个模块，他们之间也是需要相互交互的，比如 **EmailTask** 就需要调用 **Identity** 模块来查找用户的Email信息、用户的姓名等。因此需要一个流程上下文（Process Context）来协调。

最后，流程的各个Task（或者称为Activity）之间可能要共享一点信息，jBPM和Actviti都有流程上下文实例（Process Context Instance）的概念，很像一个Hash，存放key-value信息，当然比Hash更强一点的是，流程上下文实例支持作用域的概念。

## 工作流引擎的组成部分

### 流程定义

所谓流程定义即使用XML编写的用于描述流程的文件，你需要掌握一些BMPN的知识。很多工作流引擎都带有可视化流程设计器，但是需要理解的是，背后其实还是XML，下面是一个定义了4个结点的示例。

```xml
<process id="myProcess" name="My process" isExecutable="true">
    <startevent id="start" name="Start"></startevent>
    <endevent id="end" name="End"></endevent>
    <scripttask id="script" name="Script Task" scriptFormat="javascript" activiti:autoStoreVariables="true">
        <script>execution.setVariable("message", "Hello")</script>
    </scripttask>
    <servicetask id="service" name="Service Task" activiti:class="com.zhlwish.activity.demo01.HelloAction"></servicetask>
    <sequenceflow id="flow1" sourceRef="start" targetRef="script"></sequenceflow>
    <sequenceflow id="flow2" sourceRef="script" targetRef="service"></sequenceflow>
    <sequenceflow id="flow3" sourceRef="service" targetRef="end"></sequenceflow>
</process>
{% endhighlight%}

定以好流程后，就可以发布到工作流引擎，工作流引擎负责解析这个XML，并且存到数据库中。我们通过API来启动一个流程。

### API

对于苦逼的程序员来讲，API就是一切，不过经过抽象后的API也不复杂，基本上还是前面前面一节所述的概念的抽象。下面是Activiti工作流引擎部署流程定义文件，并启动一个流程的示例代码：

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
RepositoryService repositoryService = processEngine.getRepositoryService();
repositoryService.createDeployment().addClasspathResource("Demo1.bpmn").deploy();
RuntimeService runtimeService = processEngine.getRuntimeService();
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess");
{% endhighlight%}

流程启动后，可以通过API来对流程进行控制，如触发一个消息等待任务（receiveTask），甚至是将任务分配给某个用户，获取某个用户的所有任务等。

```java
List<Task> tasks = taskService.createTaskQuery()
     .taskAssignee("admin")
     .orderByDueDate().asc()
     .list();
```

这段代码是查询`admin`用户的所有任务，按照截止时间升序排序，也就是最紧急的放在最前面。

## 嵌入式部署与独立

嵌入式部署即将流程引擎嵌入部署于Web应用中，这是最容易也是最简单的方式，通过上面的API就可以实现。
独立部署即流程引擎被独立运行，Web应用通过Rest API或者其他方式调用流程引擎的接口。Activiti引擎实现了一套Rest API，SWF也实现了完整的API结构，包括各个语言的版本。

独立部署的好处就是，引擎独立运行，和外部系统很好的解耦了，外部系统的故障不会导致工作流引擎的崩溃。

## SWF

SWF与其说是工作流引擎，不如说是分布式计算调度框架，SWF中只包括Task和History两部分，甚至是每个Task之间如果要传递一些数据的话，都只能通过第三方存储（比如Message Queue或者Redis），不过这也给了编程更大的灵活性，问题是这种灵活性是不是非常需要。

一个SWF由Worker和Decider组成，Worker执行实际的任务，而Decider进行流程控制，两者严格上来讲没有区别，只是所执行的任务不同罢了。每个Worker和Decider会定期的去SWF的一个Task List取下一个任务。可以看出来这更像是一个“多线程”的结构，而SWF官方网站的Use Case是<a href="http://aws.amazon.com/swf/testimonials/swfnasa/">NASA的火星探索计划</a>中需要处理图片的系统，这其实也是一个更多侧重于计算的系统，流程反而非常简单。

另外，SWF（Simple Workflow）的一个Workflow不能太复杂，因为所有的流程控制都集中于Decider，如果太复杂的话Decider将无比庞大，给维护和扩展带来一定的困扰。

## 参考

* <a href="http://www.kafeitu.me/activiti/2012/03/22/workflow-activiti-action.html">工作流引擎Activiti使用总结</a>
* <a href="http://www.infoq.com/cn/articles/rh-jbpm5-activiti5</task>">纵观jBPM：从jBPM3到jBPM5以及Activiti5</a>
* <a href="http://www.iteye.com/topic/333718">jbpm3与jbpm4实现对比</a>
* <a href="http://blog.csdn.net/james999/article/details/1769592">揭秘jbpm流程引擎内核设计思想及构架</a>
* <a href="http://www.activiti.org/userguide">Activiti UserGuide</a>
* <a href="http://activiti.org/javadocs/">Activit JavaDoc</a>
* <a href="http://docs.aws.amazon.com/amazonswf/latest/developerguide/swf-dg-using-swf-api.html">Using the Amazon SWF API</a>