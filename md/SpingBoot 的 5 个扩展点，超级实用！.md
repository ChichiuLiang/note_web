> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/YfELU1Tpdh5AJu2YjGwCcw)

来源：https://juejin.cn/post/7263479207124942907
---------------------------------------------

**1、初始化器 ApplicationContextInitializer**
----------------------------------------

我们在启动 Spring Boot 项目的时候，是执行这样一个方法来启动的

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPpFoIUD6X2X2J1dCiauibgrhxcpYoXZTrVRqAalYpU8iaUQvFFM0Wn8I2w/640?wx_fmt=other&from=appmsg)

我们一层一层往下点，最终发现执行的是这个方法

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPw2AIFx6AkMxL8OygfeJSkBvibHpXM9UdWsCibhj7tWy1nTJUaLoyBk1w/640?wx_fmt=other&from=appmsg)

所以我们在启动项目的时候也可以这样启动 new SpringApplication(SpringbootExtensionPointApplication.class).run(args); 老的只是包装了一个静态方法，实际底层就是实例化一个 SpringApplication 对象，然后调用它的 run 方法。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPpOzvEgNNtJPa2yeyT81SwnbdkdFZO6Z2r1bjjENeibnyMx2IiaOmT62g/640?wx_fmt=other&from=appmsg)

我们进到构造方法里看下，红框里面标出来的，一个是设置初始化器，一个是设置监听器。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPicJtW5cvw2jAzfrBUXlNhnLMMZtBLJbBIxEkS7Hx1IZ9qcKWgLgFxAA/640?wx_fmt=other&from=appmsg)

点进去看下，这两个底层调的方法是一样，就是传入一个类型，通过 Spring SPI 的方式查找这个类型的实现类

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pP5gEIkuD5nEM67XTBx3MYaIZrOVI85cayTSomy0ZudE0dYMAJsD77vw/640?wx_fmt=other&from=appmsg)

打个断点，启动一下，此时有 7 个上下文初始器，这是系统自带的，配置在不同的 spring.factories 文件中。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPiaRm6qVMYibQPzARsejDmKdXTWfDs5wk9Ksna8Cegj0eiaibS3VUM7ICbw/640?wx_fmt=other&from=appmsg)

现在我要新建一个自己的初始化器

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPKljjmIDnEePSWdX2nDfsa0ppxdhO9TRrruJSocutgGhI4flKOq5c7Q/640?wx_fmt=other&from=appmsg)

此时为了能够让 Spring Boot 在启动的时候能够扫描到我创建的初始化器应该怎么办？就是在 spring.factories 文件中添加一下，注册一下，这样就能扫描到，这个就是 SPI。SPI 全称为 Service Provider Interface, 是一种服务发现机制。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPthCPAFZg4DW0Qemp3UOqkPVXoerLXzKJ2J951KGURv2D3BlCJXkNCQ/640?wx_fmt=other&from=appmsg)

那么这时候我们再启动一下 Spring Boot，发现自己创建的 ApplicationContextInitializer 也已经注册上来了，变成 8 个了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pP7YlALYuD7PnCeOClhv7iadIibXbFibEt42ThtGlMLQlNqXibuxRHqSS8fg/640?wx_fmt=other&from=appmsg)

把断点放掉，在控制台中也打印出了这句话，那么这个就是第一个扩展点 ApplicationContextInitializer

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPlcC9ZxIm8oogLAMqliaxibnrumPC5RIUdNaJXeE9odxNduIl38echHow/640?wx_fmt=other&from=appmsg)

定义了这 8 个初始化器，那一定是有地方在调它们的，不然怎么会打印出来呢，那具体在什么地方调的，我们在自己的初始化器的地方打断点，看到已经进来了，然后看下方的堆栈信息，这个就是调用的地方。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPGHTGZDDyvbOO62ujfjicSWEic7FwKd3wmezic9a60uXJia60iaRhaXicDVAw/640?wx_fmt=other&from=appmsg)

原来是调用了 run() 方法中的 prepareContext() 方法中的 applyInitializers() 方法，在这个方法中 for 循环的调用各个初始化器的 initialize() 方法，从而我们就能看到把 Jack 的 ApplicationContextInitializer 这句话给打印出来了。那么这个查找的方法就是以结果为导向来反查调用方，因为你正查的话是很难找到，很难记住的，这个方法希望大家学习到。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPzNU64UBROvwrTHCJnDmjJARu3H0ZSic656OGRAbY6wcCB7jazRib1WAA/640?wx_fmt=other&from=appmsg)

```
专属福利
👉点击领取：Java资料合集！650G！



```

那么最后来看下我们第一个扩展点所处的位置

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pP2xygeNaGH3T2mcjdoXT64FoMZFtgr2O1no1c3lIN15Z4vpAOBqS75Q/640?wx_fmt=other&from=appmsg)

初始化器可以做一些事情，比如 Environment 对象设置一些变量配置。

**2、监听器 ApplicationListener**
-----------------------------

在上面构造函数里我们发现除了有 setInitializers，还有 setListeners，那么这个 listeners 其实也是一个扩展点。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPeg20vmzFxAQsUeXDmUHzP1PibuYc42R3AzYBxY3kgmiaz0pVcWHsa5Sw/640?wx_fmt=other&from=appmsg)

那么什么是监听器，就是这样的，这个其实就是观察者模式，ApplicationEventMulticaster 发布事件，各个 Listener 监听事件。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPAxMHXqD0L0KGWPFnzENVEDVNiaZLItB5dwKaJh27X2KtBZtpMB7Ftag/640?wx_fmt=other&from=appmsg)

和初始化器一样，现在我们自定义两个监听器，一个是 Starting，一个是 Started，括号里面的是泛型，这个是一定要写的，如果不写的话就是不管什么类型的 Event 都会监听。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPIDP8j0I6vicXO45nFYJS27keI8vFCY1C3RXZU3iaTaLHjz8ksIHe2YkQ/640?wx_fmt=other&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPDQgnsTsa9blLGgzUH1ibCL26ffXdiajUPPWQE2sobibiayM4s0b9OmciaLQ/640?wx_fmt=other&from=appmsg)

这个泛型是上限为 ApplicationEvent 类型的 Event，具体的实现类有很多个，Starting 和 Started 只是其中两个。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPTg8we3DFae2Lj4a6XtRO2Y7iaDqGHiabvC56pjcVXAFSHgLM3nRFTcEw/640?wx_fmt=other&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPficTiafhy0FYM3tVRGb3LmbcOHrMb1F275aiciaCbdjsLdk5C4H1ASGVTQ/640?wx_fmt=other&from=appmsg)

现在我们还是把这两个监听器通过 SPI 的方式加到配置中去

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPEZrndYJlAT5Tq2zh931ePO2rJD5193icJNpBTSQ6YEIaIhGT8cUjqhg/640?wx_fmt=other&from=appmsg)

好，运行一下，我们看到这两句话已经打印出来了

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPKscMvEcmaYibwvc9T7RY5wd7icsq8iczYhjL0YyNTKB8rMfP3ibBp1RdUw/640?wx_fmt=other&from=appmsg)

和监听器一样，既然能够打印出来，那肯定是有地方在调用，所以我们在 JackStartingApplicationListener 打个断点，然后看下堆栈信息

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pP00T7F3KYmRuvgNLicOTCBtDNiaibMpHdmdaA8ffZ71JROfz5iaHhHBrYxw/640?wx_fmt=other&from=appmsg)

我们可以看到在 SpringApplication run() 方法的 listeners.starting() 开始进去发送 ApplicationStartingEvent 广播事件，最后发布出去，由我们自己编写的事件监听器接收到。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPH6Kjtwa4QibPbVods9y4vcgbWdHkAwKYo9VujFFvwdYxAKc6K2HQibJw/640?wx_fmt=other&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPR6BzK1eeInBp6Z7iaADBzMqDAXr4TcicicxTACpC7CtKNYJcz96pNwibAg/640?wx_fmt=other&from=appmsg)

那么 ApplicationStartedEvent 事件也是一样的道理，通过打断点的方式来找到它的调用方，最后我们再来看下此时的扩展点图

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPic6l4PPDwmG7qc0OkG4HBZFdWvicwy2I5yia4ibvOcT5dFg3LdJ0f2ia4XA/640?wx_fmt=other&from=appmsg)

**3、Runner**
------------

我们看到在 listeners.started() 后面有个 callRunners

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPTpyiaG0ovd8Vibiaibicy3ibb2XPLzse1f0icNFH7YMjGmJXWeRumeupnBdicA/640?wx_fmt=other&from=appmsg)

我们点进去看下，它其实就是从容器中获取两种类型的 Runner，一种是 ApplicationRunner，还有一种是 CommandLineRunner，然后 for 循环的对它们进行调用，那么其实这个也是一个扩展点

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPGWXHjwoRgtIfKNPPjyombQXKREGCia5Fl6QnRZE9ZaZKoJDf8dNRESw/640?wx_fmt=other&from=appmsg)

我们来写一个自己的 Runner

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPuTGahtSn5Qejtgl8VmsqHVYZSUHicPjkRsnqwk3zWoynlXQqcic0ibB0w/640?wx_fmt=other&from=appmsg)

运行一下，看下打印出来了

  
![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPBCm0ZrrXrCr6CFLCzicDpXnECB9pKBaobOYZYKHL61sXv4icubchVvYA/640?wx_fmt=other&from=appmsg)

那么这个 Runner 的一般应用场景就是资源释放清理或者做注册中心，因为执行到 Runner 的时候项目已经启动完毕了，这时候就可以注册到注册中心上去了。此时我们再看下扩展点图。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPZF2q581BVSicicicODWAOqaVeR3bre2jj1IvXnOQdyfH1MU0CyWFdsUdg/640?wx_fmt=other&from=appmsg)

**4、BeanFactoryPostProcessor**
------------------------------

我们看下 run 方法里的 refreshContext() 方法

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPGtZicMD4giao9cmWhF3AC0o1xRbMakpkxx7OSUW6m3oqJ7Mb1PzhTufA/640?wx_fmt=other&from=appmsg)

这个方法底层会调 spring 里面的 refresh() 方法，这个方法里面就会做对容器的初始化。红框里的 invokeBeanFactoryPostProcessors() 方法，这里也有一个扩展点，就是 BeanFactoryPostProcessor，执行对 BeanFactory 的后置处理。

Spring Boot 解析配置成 BeanDefinition 的操作也是在此方法中。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pP5b0uymT5FwibLjibGFOT8iagcjRBicTfPuDmtnCSs9qTEVmwFFZkkB7RAQ/640?wx_fmt=other&from=appmsg)

现在我们来创建一个自己的 BeanFactoryPostProcessor，这个方法里面可以修改 beanFactory 的属性，beanfactory 里面有 BeanDefinition，可以修改 BeanDefinition 里面的值。BeanDefinition 是一个 bean 的元数据的信息，有多少个 bean 就有多少个 BeanDefinition。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPsic1tKGhFffyU3KsRuKwa3zQRrHic4qICul7OYHDbCZYzngAvkiawmIWQ/640?wx_fmt=other&from=appmsg)

运行一下，也打印出来了

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPW7erF8KkGf1eLoTdsmN3sy5zVEKS1o0ViaOgwkkbm4WteA4icm6IaIpA/640?wx_fmt=other&from=appmsg)

此时我们再看下扩展点图，越来越完善了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPo3Bp8DsZH2vQ5jG3KXgHR0QSonhteiaJ1m274xCibuv6fv9Rmiab6UGKw/640?wx_fmt=other&from=appmsg)

**5、BeanPostProcessor**
-----------------------

最后介绍的是 BeanPostProcessor，它在通过反射构造函数进行 bean 实例化之后执行，那么红框里面标出来的 registerBeanPostProcessors() 方法就是向 BeanFactory 中注册 beanpostprocessor，用于后续 bean 创建的拦截操作。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPlo9OVibkkAicicxeSx9lP3j3AtDjzClFBMT9l2tCxKyzELzB3acibduH2A/640?wx_fmt=other&from=appmsg)

现在我们来创建一个自己的 BeanPostProcessor，一共有两个方法，postProcessBeforeInitialization 和 postProcessAfterInitialization，不过我们一般用 postProcessAfterInitialization，在 bean 调用反射构造函数实例化之后执行。著名的应用场景 AOP 底层就是通过 BeanPostProcessor 来实现的。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPKAXfbvibnHCQWRicHicFH6GibnsYg4tTnkezxJaaPo62HSTGmAmZzqxOYg/640?wx_fmt=other&from=appmsg)

现在我在 postProcessAfterInitialization 上打个断点，看下堆栈信息是在哪里调用的

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pP3BqFxdzhXr5DEib2zoZjn3kg5OU6Av7TcRauqWg2lIcGicPSbsVCGfUA/640?wx_fmt=other&from=appmsg)

是在 finishBeanFactoryInitialization() 方法处调用的

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pP1UWLkLqft8XyNOe7mdoibrgFyYETtW3y01wclnO26rickGX8oiaRwaWAA/640?wx_fmt=other&from=appmsg)

**后记**
------

最后我来把扩展点图补充完整，如下所示，很清晰明了，在什么时候调用了什么，我们自己开发的时候结合应用场景，在什么时候要干什么事，就知道要创建什么类型的扩展点了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd4sEKeDSnANfLCTPDWnd1pPRFVXiaibQteyHqcV9ias8cJpemvDtZE7xUiag3jGXFpl5AyG1Z8NKdIb7w/640?wx_fmt=other&from=appmsg)

本文前三个讲的是 Spring Boot 里面自己有的扩展点，后两个因为 Spring Boot 底层调的是 Spring 的源码，所以属于 Spring 里面的扩展点，所以如果这么算的话 Spring 里面的扩展点还有很多扩展点，比如 InitializeBean、Aware 等等这里都没讲，等待大家去发掘~

```
 
热门推荐
Controller层代码这么写，简洁又优雅！
组装生成复杂父子树形结构， Stream + Lambda优雅搞定！
Markdown 教程-使用扩展Markdown语法

```