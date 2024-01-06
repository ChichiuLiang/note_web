> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pDhjg_jLLfvrPhLRPCzGgA)

点击 “IT 码徒”，关注，置顶公众号

```
每日技术干货，第一时间送达！

```

前两天在工作中忙的焦头烂额，涉及到`@Transactional`对于事务的控制，便仔细研究了一下，颇有所获，花费好了几天测试整理，今天才发表出来，希望看到博客的老铁们能有所获吧。  

话不多说直奔正题。

**先简单介绍一下 Spring 事务的传播行为：**

所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在`TransactionDefinition`定义中包括了如下几个表示传播行为的常量：

*   `TransactionDefinition.PROPAGATION_REQUIRED`：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。
    
*   `TransactionDefinition.PROPAGATION_REQUIRES_NEW`：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
    
*   `TransactionDefinition.PROPAGATION_SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
    
*   `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
    
*   `TransactionDefinition.PROPAGATION_NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。
    
*   `TransactionDefinition.PROPAGATION_MANDATORY`：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
    
*   `TransactionDefinition.PROPAGATION_NESTED`：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`。
    
    ‍  
    

**然后说一下 Spring 事务的回滚机制：**

Spring 的 AOP 即声明式事务管理默认是针对`unchecked exception`回滚。Spring 的事务边界是在调用业务方法之前开始的，业务方法执行完毕之后来执行`commit or rollback`(Spring 默认取决于是否抛出`runtimeException`)。

如果你在方法中有`try{}catch(Exception e){}`处理，那么 try 里面的代码块就脱离了事务的管理，若要事务生效需要在 catch 中`throw new RuntimeException ("xxxxxx");`这一点也是面试中会问到的事务失效的场景。

再简单介绍一下`@Transactional`注解底层实现方式吧，毫无疑问，是通过动态代理，那么动态代理又分为 JDK 自身和 CGLIB，这个也不多赘述了，毕竟今天的主题是如何将`@Transactional`对于事物的控制应用到炉火纯青。哈哈~

第一点要注意的就是在`@Transactional`注解的方法中，再调用本类中的其他方法 method2 时，那么 method2 方法上的`@Transactional`注解是不！会！生！效！的！但是加上也并不会报错，拿图片简单帮助理解一下吧。这一点也是面试中会问到的事务失效的场景。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e1IsRMuLpicLv1HHtp45BHZLCN4uOAkeeJZDygY3HJUAFRekg6ef9XibWfz8D85FCIFYWFAp0nl4pnnUBatE4yiaw/640?wx_fmt=jpeg&from=appmsg)

通过代理对象在目标对象前后进行方法增强，也就是事务的开启提交和回滚。那么继续调用本类中其他方法是怎样呢，如下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/e1IsRMuLpicLv1HHtp45BHZLCN4uOAkee51qWDQR5qASsHdlTX2tdWoX7RsHFMcxPb4iaEicRQWK8OCkNCjWVERKg/640?wx_fmt=jpeg&from=appmsg)

可见目标对象内部的自我调用，也就是通过 this. 指向的目标对象将不会执行方法的增强。

先说第二点需要注意的地方，等下说如何解决上面第一点的问题。第二点就是`@Transactional`注解的方法必须是公共方法，就是必须是 public 修饰符！！！

至于这个的原因，发表下个人的理解吧，因为 JVM 的动态代理是基于接口实现的，通过代理类将目标方法进行增强，想一下也是啦，没有权限访问那么你让我怎么进行，，，好吧，这个我也没有深入研究底层，个人理解个人理解。

在这里我也放个问题吧，希望有高手可以回复指点指点我，因为 JVM 动态代理是基于接口实现的，那么是不是 service 层都要按照接口和实现类的开发模式，注解才会生效呢，就是说`controller`层直接调用没有接口的 service 层，加了注解也一样不起作用吧，这个懒了，没有测试，其一是因为没有人会这么开发吧，其二是我就认为是不起作用的，哈哈。

**下面来解决一下第一点的问题，如何在方法中调用本类中其他方法呢。**

通过`AopContext.currentProxy ()`获取到本类的代理对象，再去调用就好啦。因为这个是 CGLIB 实现，所以要开启 AOP，当然也很简单，在 springboot 启动类上加上注解`@EnableAspectJAutoProxy(exposeProxy = true)`就可以啦，这个依赖大家自行搜一下就好啦。要注意，注意，代理对象调用的方法也要是 public 修饰符，否则方法中获取不到注入的 bean，会报空指针错误。

emmmm，我先把调用的方式和结果说下吧。自己简单写了代码，有点粗糙，就不要介意啦，嘿嘿。。。

Controller 中调用 Service

```
@RestController
public class TransactionalController {

    @Autowired
    private TransactionalService transactionalService;

    @PostMapping("transactionalTest")
    public void transacionalTest(){
        transactionalService.transactionalMethod();
    }
}


```

[Service 中实现对事务的控制：接口](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247565405&idx=2&sn=5a410918a4bbf827cd309071ef7d3272&chksm=eb51456bdc26cc7d1684d22b9cad96098e8229681505e4dd7eec807616562b0d53a5e9fbca05&scene=21#wechat_redirect)

```
public interface TransactionalService {
    void transactionalMethod();
}


```

[Service 中实现对事务的控制：实现类（各种情况的说明都写在图片里了，这样方便阅读，有助于快速理解吧）](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247565405&idx=2&sn=5a410918a4bbf827cd309071ef7d3272&chksm=eb51456bdc26cc7d1684d22b9cad96098e8229681505e4dd7eec807616562b0d53a5e9fbca05&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/e1IsRMuLpicLv1HHtp45BHZLCN4uOAkeekeJeMpV8a70dyPZ5e8EIsqUd0iawlRStKm3VAYNpAP8qiaOk5tQibfbSg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/e1IsRMuLpicLv1HHtp45BHZLCN4uOAkeeyaM5BpaALLGkDBV9f1vuksxZxOYxpTicoiaFV76HhMVVKPdQ4RzbAgMw/640?wx_fmt=png&from=appmsg)

[上面两种情况不管使不使用代理调用方法 1 和方](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247565405&idx=2&sn=5a410918a4bbf827cd309071ef7d3272&chksm=eb51456bdc26cc7d1684d22b9cad96098e8229681505e4dd7eec807616562b0d53a5e9fbca05&scene=21#wechat_redirect)法 2，方法`transactionalMethod`都处在一个事务中，四条更新操作全部失败。

那么有人可能会有疑问了，在方法 1 和方法 2 上都加`@Transactional`注解呢？答案是结果和上面是一致的。

小结只要方法`transactionalMethod`上有注解，并且方法 1 和方法 2 都处于当前事务中（不使用代理调用，方法 1 和方法 2 上的`@Transactional`注解是不生效的；使用代理，需要方法 1 和方法 2 都处在`transactionalMethod`方法的事务中，默认或者嵌套事务均可，当然也可以不加`@Transactional`注解），那么整体保持事务一致性。

如果想要方法 1 和方法 2 均单独保持事务一致性怎么办呢，刚说过了，如果不是用代理调用`@Transactional`注解是不生效的，所以一定要使用代理调用实现，然后让方法 1 和方法 2 分别单独开启新的事务，便 OK 啦。下面摆上图片。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/e1IsRMuLpicLv1HHtp45BHZLCN4uOAkee6sGBNficD3sK17qm02jsEMEq8YlfkqCjnsvWAlShW3GK3C5vcWQ12CA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/e1IsRMuLpicLv1HHtp45BHZLCN4uOAkeelFPzDexnYthU425iaH42Ful9EgCEqmTqlyGU6g0LoOsEoNAwAPJOmibQ/640?wx_fmt=png&from=appmsg)

这两种情况都是方法 1 和方法 2 均处在单独的事务中，各自保持事务的一致性。

接下来进行进一步的优化，可以在`transactionalMethod`方法中分别对方法 1 和方法 2 进行控制。要将代码的艺术发挥到极致嘛，下面装逼开始。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/e1IsRMuLpicLv1HHtp45BHZLCN4uOAkeeWmwY8Fd7nSic4jZibia0zlkMbfibDVeE4qNVESSGXtic6ib1VwOcnu6QAKfQ/640?wx_fmt=png&from=appmsg)

代码太长了，超过屏幕了，粘贴出来截的图，红框注释需要仔细看，希望不要影响你的阅读体验，至此，本篇关于`@Transactioinal`注解的使用就到此为止啦，

简单总结一下吧：

1、就是`@Transactional`注解保证的是每个方法处在一个事务，如果有 try 一定在 catch 中抛出运行时异常。

2、方法必须是 public 修饰符。否则注解不会生效，但是加了注解也没啥毛病，不会报错，只是没卵用而已。

3、this. 本方法的调用，被调用方法上注解是不生效的，因为无法再次进行切面增强。

—END—

[【福利】2023 高薪课程，全面来袭（视频 + 笔记 + 源码）](http://mp.weixin.qq.com/s?__biz=MzU2OTMyMTAxNA==&mid=2247516711&idx=1&sn=e54d98dc55dd5dd2ff2c90935a60ae97&chksm=fc82b17ecbf53868aece859626913a43ba5b592398dcda1847e497df47632b180fb1fbd3a3d3&scene=21#wechat_redirect)
=========================================================================================================================================================================================================================================================

**PS：防止找不到本篇文章，可以收藏点赞，方便翻阅查找哦。**

  

往期推荐

  

  

[

Spring 中 Bean 的生命周期，让你瞬间通透~



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503525&idx=1&sn=bcd6cbba1e7bb42e7cbc63b6a445eb33&chksm=ead000a4dda789b22ccb1157274c59cb6aae8426d8472db56c48f440823b00e53f276debc721&scene=21#wechat_redirect)

[

写了个工具，让 CRUD 开发效率提升 100 倍，开源咯！



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503515&idx=1&sn=44aa91e4c5bbb23adc1db09f113398d9&chksm=ead0009adda7898c9c8a67802dc7f10165a48a2c47f43fdc9799a939050b79100304f30de3c6&scene=21#wechat_redirect)

[

推荐几个软件做笔记，看看你用的哪个？



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503479&idx=1&sn=00da14701e206ecad3e12eb3e749595a&chksm=ead00076dda78960523588969cacb7ae21559e12b6d007ece6382aa3dafc586dc3b2a630022b&scene=21#wechat_redirect)

[

推荐一个 yyds 开源项目任务管理工具



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503468&idx=1&sn=6fac25fae1929390f8cf948bf2341b11&chksm=ead0006ddda7897b44cc3f2a752cd4c9def79e4cc71207631d69a30112556265dc16b5885202&scene=21#wechat_redirect)

[

看了我的 MyBatis-Plus 用法，同事也开始悄悄模仿了



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503452&idx=1&sn=1e6222085f34876b15d5e8b8b7d1969c&chksm=ead0005ddda7894b27528cb22cf8a4fd2fc66aea4f5f61a2a06e1d6a503c39bbe21018812ed1&scene=21#wechat_redirect)

[

用了 6 年的 SpringBoot 项目部署方案，稳得一批！



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503452&idx=2&sn=53e883adf2aabd988c0727d32642e600&chksm=ead0005ddda7894b3e7b62f47c82175dd7647614f454b77934b19c2e2c874053cd054ce264f1&scene=21#wechat_redirect)