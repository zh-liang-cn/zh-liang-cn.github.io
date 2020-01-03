---
layout: post
title: 用EasyMock进行单元测试：什么是EasyMock
excerpt: |+
  我们都知道单元测试，也常常写单元测试，但是对于什么是单元测试却没有仔细思考过，大概觉得创建一个类继承TestCase，然后写一些assert语句就算是了吧。当然我也常常遇到有人认为在Java类中写main函数也算是单元测试的。总之，归结到一句话，我们可能觉得写了一段代码对我的代码中的一个方法进行了验证，就算是单元测试了。一直到前不久之前，我也算是这大部分人中的一个。由于我之前写SSH比较多，因此我也想当然得认为要测试一个DAO，就必须往数据库里面插入一些数据，然后调用DAO，看能否返回正确的结果这是单元测试。 事实上，我错了。
date: '2012-06-29 00:49:12 +0800'
categories:
- Java
tags:
- DbUnit
- junit
- easymock
---
我们都知道单元测试，也常常写单元测试，但是对于什么是单元测试却没有仔细思考过，大概觉得创建一个类继承TestCase，然后写一些assert语句就算是了吧。当然我也常常遇到有人认为在Java类中写main函数也算是单元测试的。总之，归结到一句话，我们可能觉得写了一段代码对我的代码中的一个方法进行了验证，就算是单元测试了。一直到前不久之前，我也算是这大部分人中的一个。由于我之前写SSH比较多，因此我也想当然得认为要测试一个DAO，就必须往数据库里面插入一些数据，然后调用DAO，看能否返回正确的结果这是单元测试。 根据<a href="http://book.douban.com/subject/5326182/">《测试驱动开发的艺术》</a>一书中的定义，在以下情况中一个测试不是单元测试：

* 访问了数据
* 有网络通讯
* 访问了文件系统
* 不与其他任何单元测试同时运行
* 必须配置好环境后才能运行

因为以上情况下，会使单元测试的速度降低，在大型的项目中，时常会有上千个甚至上万个单元测试用例，这个时间的消耗是我们承担不起的，特别是要求修改了代码必须使所有的单元测试都通过才能提交代码的情况。以上情况中的测试都应该规划到集成测试的范畴。 最近在工作中遇到需要通过Web Service调用远程服务的情况（大家都知道亚马逊是以贯彻SOA著称的公司），在对业务代码进行单元测试的时候遇到了问题，因为自己去搭建一个测试的WebService太麻烦了，而且还得保证数据是固定，很显然，如果每次运行单元测试的时候Web Service都返回不同的结果是不行的。 在这个时候我们就需要<a href="http://easymock.org/">EasyMock</a>这样的工具来帮我们了。顾名思义，<a href="http://easymock.org/">EasyMock</a>可以让我们很Easy地创建Mock对象来模拟外部依赖。

在我的工作中，<a href="http://easymock.org/">EasyMock</a>使我们不必要真正的通过网络去远程调用事先搭建好的测试Web Service。**一般我们先创建一个Mock对象，然后record（录制）这个对象的行为，接着将对象设置为replay（回放）状态，之后我们执行需要测试的业务代码并检验代码是否返回正确的结果，最后，我们可以verify（验证）整个过程中，Mock对象是否完成了record阶段的设定。**

看起来非常复杂，下面是一个实例，Web Service本身不是重点。 一般，Web Service提供给外界的是通过WSDL定义的接口，通过一些工具可以生成_stub class_，包括一个`ServiceClientBuilder`，一个`ServiceClient`接口和一些`Model`，比如下面的`ServiceClient`接口：

```java
public interface ServiceClient {
    public List<String> getGroupsByUser(String username);
}
```

在客户端代码中，使用下面的方式实现一个业务逻辑：

```java
public class UserDao {
private ServiceClient client;
public boolean validate(String username) {
    List<String> groups = client.getGroupsByUser(username);
    for (String group : groups) {
        if (group.equals("admin")) {
            return true;
        }
    }
    return false;
}
/**
 * Used by Unit Test and other injection work
 * @param client
 */
public void setClient(ServiceClient client) {
    this.client = client;
}
}
```

`UserDao.validate(String)`方法对用户组进行了判断，如果用户是_admin_，则返回`true`，否则返回`false`，那么测试代码可以是这样：

```java
import static org.easymock.EasyMock.*;
import static org.junit.Assert.*;
import org.junit.Before;
import org.junit.Test;
public class UserDaoTest {
    private static final String USERNAME = "liang";
    private UserDao userDao = new UserDao();
    private ServiceClient client;
    private List<String> groupContainsAdmin;
    private List<String> groupNotContainAdmin;
    @Before
    public void before() {
        client = createMock(ServiceClient.class);
        groupContainsAdmin = new ArrayList<String>() {
            {
                add("admin");
                add("user");
            }
        };
        groupNotContainAdmin = new ArrayList<String>() {
            {
                add("user");
            }
        };
    }
    @Test
    public void testValidateIsAdmin() {
        expect(client.getGroupsByUser(eq(USERNAME))).andReturn(
                groupContainsAdmin);
        userDao.setClient(client);
        replay(client);
        boolean isAdmin = userDao.validate(USERNAME);
        assertTrue(isAdmin);
        verify(client);
    }
    @Test
    public void testValidateIsNotAdmin() {
        expect(client.getGroupsByUser(eq(USERNAME))).andReturn(
                groupNotContainAdmin);
        userDao.setClient(client);
        replay(client);
        boolean isAdmin = userDao.validate(USERNAME);
        assertFalse(isAdmin);
        verify(client);
    }
}
```

整个代码的执行流程如下：

* 每个测试运行之前，先调用`EasyMock.createMock(Class)`方法创建了`ServiceClient`的`Mock`对象;
* 在每个测试中，使用`EasyMock.expect()`方法录制`Mock`对象的行为，如`testValidateIsAdmin()`中设置`getGroupsByUser()`方法的参数必须是USERNAME，并且返回值是groupContainsAdmin，而`testValidateIsNotAdmin()`中`getGroupsByUser()`则返回groupNotContainAdmin。这里值得注意的是`EasyMock.eq()`方法和`andReturn()`方法；
* `Mock`对象行为录制完成后，调用`EasyMock.replay()`返回标记`Mock`对象为回放状态；
* 调用业务方法，并且验证业务代码的逻辑的正确性；
* 最后调用`EasyMock.verify()`验证业务代码的执行逻辑和我们录制的一样，确保业务逻辑的调用远程服务执行的正确性。 <a href="http://easymock.org/">EasyMock</a>除了上面的方法外，还提供了`EasyMock.isA(Class)`，`EasyMock.andStubReturn()`等方法以便于更好的完成测试。详细参考<a href="http://easymock.org/EasyMock3_1_Documentation.html">http://easymock.org/EasyMock3_1_Documentation.html</a>，我之后的文章也会进行总结。 

不过EasyMock也有不如人意的地方，因为<a href="http://easymock.org/">EasyMock</a>本身用Java的动态代理方式实现，因此无法实现对静态方法的Mock，不过还有更为强大的<a href="https://code.google.com/p/powermock/">PowerMock</a>通过cglib这样的字节码操作库实现了对静态方法的Mock，不过即使再强大的PowerMock也是基于<a href="http://easymock.org/">EasyMock</a>实现的，因此了解<a href="http://easymock.org/">EasyMock</a>实在是很必要的。

## 参考

* <a href="http://easymock.org/EasyMock3_1_Documentation.html">http://easymock.org/EasyMock3_1_Documentation.html</a>
* <a href="http://www.ibm.com/developerworks/cn/opensource/os-cn-easymock/">http://www.ibm.com/developerworks/cn/opensource/os-cn-easymock/</a>
* <a href="http://macrochen.iteye.com/blog/298032">http://macrochen.iteye.com/blog/298032</a>
* <a href="http://www.iteye.com/topic/310294">http://www.iteye.com/topic/310294</a>
* <a href="http://www.iteye.com/topic/310313">http://www.iteye.com/topic/310313</a>

