> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PG6GN2C6m46LvLg28RdpBg)

关注公众号回复「**1024**」获取架构师资料↓

推荐学习：[Spring Boot 3.x 最新教程](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247594996&idx=2&sn=63271ef2953150a707813872acd1ff4d&chksm=eb51f6c2dc267fd4530624c5ceb5ca87e3c90dd9103ab0eaa4a8306206070f602ff275127a1a&scene=21#wechat_redirect)

前言
--

我们生活中都听说了 DDD，也了解了 DDD，那么怎么将一个新项目从头开始按照 DDD 的过程进行划分与架构设计呢？

一、专业术语
------

各种服务

*   IAAS：基础设施服务，Infrastructure-as-a-service
    
*   PAAS：平台服务，Platform-as-a-service
    
*   SAAS：软件服务，Software-as-a-service
    

推荐一个开源免费的 Spring Boot 实战项目：

> https://github.com/javastacks/spring-boot-best-practice

二、架构演变
------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TNUwKhV0JpQekuq7qsKXiaKbrsBoA06L6pEcCmuFbbFJ0QKrCvEnTFjia7icmtibXFaxPFxicNT5uJJxTEIDJ7cRGlg/640?wx_fmt=png)

从图中已经可以很容易看出架构的演进过程，通过对三个层的举例来进行说明：

*   SAAS：比如我们最早的就是单体应用，多个业务之间可能都没有进行分层，之后我们业务多了，都各自混淆在一起，后来我们就通过 MVC、SSM、分层等方式进行业务拆分，保证业务与业务之间解耦
    
*   PAAS：随者业务的增长，我们打算分离出一个子系统，但是成本太高，每次都需要从头搭建一个子系统，效率低下。这时我们就抽取除了一些通用技术，比如 mesh、SOA、微服务等方式来隔离系统，且对通用技术复用来快速搭建一个系统
    
*   IAAS：比如订单服务并发量高，单台服务器已经无法满足要求，这时我们需要多台服务器，可能有 windows 的、linux、mac，想要快速部署就需要屏蔽 OS，于是就有了 VM、Docker、K8S 等技术来屏蔽 OS
    
*   另外，如果你近期准备面试跳槽，建议在 Java 面试库小程序在线刷题，涵盖 2000+ 道 Java 面试题，几乎覆盖了所有主流技术面试题。
    

三、限界上下文
-------

限界上下文概念

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TNUwKhV0JpQekuq7qsKXiaKbrsBoA06L69xHCBPOibmX48QbhfiaG1SbtkKZpHGY71cVXhG3nmiaofYo51dqia0zoDA/640?wx_fmt=png)

**BC 与业务的关系：**

通过对业务的划分，比如订单系统，订单是一个子域；库存是一个子域；其中商品再不同的子域中所表示的意义也不同，比如在订单上下文中的商品表示商品的单价、折扣等等；而在库存的上下文中商品表示商品的库存量、成本、存放位置等。

**BC 与技术的关系：**

多个子域之间必须需要在应用层进行聚合，而聚合的过程中就引出了技术方案，比如订单到库存到支付，他们应该采用同步方式；这几个子域调用通知都应该是异步，那么可能就需要消息中间件或其它技术方案

限界上下文划分规则

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TNUwKhV0JpQekuq7qsKXiaKbrsBoA06L60Svw9IpBCjJn1foHFrlAXrcbo2SnECrqZh76PQMX09IsCMSA1JQa4A/640?wx_fmt=png)

一般来说，先考虑团队规模，来决定最终需要划分到多细粒度的 BC，如果团队规模过小而 BC 过细，则对后期的运维、部署、上线都会造成很大的负担；在确定好粒度后，可以对语义相关性、功能相关性 - 业务方向、功能相关性 - 非业务方向进行划分按照以上的规则划分之后就得到了多个 BC 啦。

推荐学习：[Spring Cloud 微服务实战（2023 最新版）！](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247587660&idx=2&sn=c5a3280ac42ba6afe8a2fe6f5ada3f03&chksm=eb51127adc269b6c92702c3a359dd10c95bcffdda76c72096fd0e11233989c61386d70e7938e&scene=21#wechat_redirect)

**一个 BC 代表一个微服务吗？**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TNUwKhV0JpQekuq7qsKXiaKbrsBoA06L67WTnAXxhhyVQSOtwSbjdYcfRdynwPKHhsJ9xt2Wn7f3nRR8XbgweGg/640?wx_fmt=png)

**概念：** 微服务一般是指将高度相关功能的一个开发部署单元，有自己的技术自治性、技术选型、弹性扩缩容、发布上下频率等，说白了就是各自维护一个业务，然后多个业务组成一个系统，多个业务之间各自管理

**关系：** 这里的 BC 其实就是一个领域或一个模块或一个业务，如果两个领域相关性很高，就可以包含多个 BC，或者如果一个领域访问量非常大，则需要部署在一个微服务中以提高性能。

另外，如果你近期准备面试跳槽，建议在 Java 面试库小程序在线刷题，涵盖 2000+ 道 Java 面试题，几乎覆盖了所有主流技术面试题。

四、领域驱动设计的四重边界
-------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TNUwKhV0JpQekuq7qsKXiaKbrsBoA06L6rXzb3AMtYRYdrd7NB12rwN2ItOOYSY034sibKURQjYswjg0jyGMPbog/640?wx_fmt=png)

根据上图所示，我们通过四重来进行架构设计：

分而治之：DDD 通过规划四重边界，把领域知识做了合理的固化和分层。业务有核心领域和支持域、业务域中又拆分成多个限界上下文（BC），一个 BC 中又根据领域知识核心与否进行分层，领域层中按照多个业务（子域）的强相关性进行聚合成一个子域。

*   【第一重边界】确定项目的愿景与目标，确定问题空间，确定核心子领域、通用子领域（多个子领域可以复用）、支撑子领域（额外功能，如数据统计、导出报表）
    
*   【第二重边界】解决方案空间里的限界上下文就是一道进程隔离层面的物理边界
    
*   【第三重边界】每个限界上下文内，使用分层架构划分为：接口层、领域层、应用层、基础设施层之间的最小隔离
    
*   【第四重边界】领域层里为了保证各个领域的完整性和一致性，引入聚合的设计作为隔离领域模型的最小单元
    

五、整洁分层架构
--------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TNUwKhV0JpQekuq7qsKXiaKbrsBoA06L65VjqyrW0yJ38lMBniaB8n1YsE0ZwYJ3H30KsWjKicVWPTpJZJKIyRJzg/640?wx_fmt=png)

具体说明看图中备注，总的来说就是通过实现与接口分离，让 domain 层尽量独立，而不耦合与任何模块，这里面包含了领域模型的业务逻辑代码，但不会依赖于具体技术实现，可以很方便更换基础设施层，提供给第三方 web 调用 service

六、六边形架构
-------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TNUwKhV0JpQekuq7qsKXiaKbrsBoA06L6Lp48rQrBPavTibU44XburCWhoNQ6PPRmLLXvNQ7OPqogI6FyzmODsvg/640?wx_fmt=png)

**主动适配：** 指来⾃于 UI、命令⾏等输⼊型命令， controller 就是⼀种端⼝，端⼝的具体实现就是应⽤逻辑⾃身。因此端⼝和具体实现都在应⽤系统的内部。

**被动适配：** 指访问存储设备，外部服务等。每种访问就是⼀种端⼝，具体实现是各个具体的中间件。因此端⼝在整个应⽤系统的⾥部，具体实现在系统的外部。每⼀种输⼊和输出都是⼀个端⼝，每个端⼝都有具体的实现逻辑，因此整个应⽤系统的架构就是⼀些列的端⼝ + 适配逻辑组成，架构图就是⼀个多边形形状。有⼏个端⼝需要根据应⽤系统的具体情况⽽定，只是六个端⼝⽐较形象⽽得名为六边形架构。

特点：

1.  外层依赖内层使得依赖更合理。端⼝就是接⼝，依赖接⼝编程。借此保证了应⽤和实现细节之间的隔离。
    
2.  可测试更好
    

七、洋葱架构
------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TNUwKhV0JpQekuq7qsKXiaKbrsBoA06L6fL6JrvwaYrjASFNMPgRY6sEPcGC69WIhszH1UT60KaJeWm5gO1QYwg/640?wx_fmt=png)

洋葱架构针对六边形架构更进⼀步把内层的业务逻辑分为了 DDD 概念的应⽤服务层、领域服务层和领域模型层。

特点：

*   围绕独⽴的领域模型构建应⽤
    
*   内层定义接⼝，外层实现接⼝
    
*   依赖的⽅向指向圆⼼（注意：洋葱架构提倡不破坏耦合⽅向的依赖都是合理的，外层可以依赖直接内层，也可以依赖更⾥⾯的层）
    
*   所有的应⽤代码可以独⽴于基础设施编译和运⾏
    

总结
--

目前领域驱动设计是目前比较流行的一种架构设计，只需要按照领域驱动设计的四重边界进行架构设计，就能够很好的对各个领域解耦，对后期的业务垂直扩展、功能的水平扩展提供了良好的基础。

> 版权声明：本文为 CSDN 博主「我思知我在」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。原文链接：https://blog.csdn.net/qq_32828253/article/details/110673205

**推荐阅读：**  

[上周，被裁员了。。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247596220&idx=1&sn=6af52a0b0b8a7ac5786cfd90b81d3dc7&chksm=eb51fd8adc26749c032c59366253669db64befb90fa19c1a204fa8a19ce8e5e0b332e58295cc&scene=21#wechat_redirect)

[创业失败准备回大厂。。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247596106&idx=1&sn=cb04ea11bdef1fe7e297cf881d645b3b&chksm=eb51fd7cdc26746a1471fb32496f11401c8a2bf459becbc68e64b5e57d578776436f65519fe9&scene=21#wechat_redirect)

[阿里 IDEA 新插件，好用到爆！](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247595985&idx=1&sn=a0c5df759d1079f9931b5f8b9ac85670&chksm=eb51f2e7dc267bf1b7206b68d882d78b993c1a9014e880d5b61828ce7c69c85e8346f864ddd8&scene=21#wechat_redirect)

[震撼！年终奖，到账 495774 元](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247596240&idx=1&sn=8851c987a279e30ad37d17e4bc204e18&chksm=eb51fde6dc2674f02790757b1540851fabac6c7a70c48c16ce7b7a681d1383257c903b625b35&scene=21#wechat_redirect)  

关注公众号回复「**1024**」获取一份架构师资料↓