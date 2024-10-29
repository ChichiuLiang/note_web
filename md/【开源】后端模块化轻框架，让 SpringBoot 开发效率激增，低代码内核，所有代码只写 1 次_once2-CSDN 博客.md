> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Daniel_Leung/article/details/141131791)

![](https://i-blog.csdnimg.cn/direct/20f18b519bde4cf9b4f92667842fccf0.png)

大家好，欢迎来到停止重构的频道。

**本期介绍我们的新后端框架：Once2**

一个能让所有后端代码只写 1 次，更看重项目过程、工程质量的框架。

我们按这样的顺序展开介绍

1、模块、业务分离

2、Once 只是一种规则

3、工作原理

4、设计愿景

模块、业务分离
-------

首先介绍最[主要的](https://so.csdn.net/so/search?q=%E4%B8%BB%E8%A6%81%E7%9A%84&spm=1001.2101.3001.7020)模块、业务分离。

从宏观上讲，后端应用程序是多个请求接口的集合，而对于**单个接口**而言是**执行多个步骤**。

![](https://i-blog.csdnimg.cn/direct/901fec1583fd4371953c0ab502a0b246.jpeg)

基于这样的观点，**Once 将代码分离了两层：模块代码、业务代码**

**模块代码是需要写 Java 代码实现具体功能的。**只需要关心**通用功能的实现**，例如：文件上传、参数检查、[数据库操作](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E5%BA%93%E6%93%8D%E4%BD%9C&spm=1001.2101.3001.7020)等。

**业务代码指的是具体的后端接口**。只需要关心**业务功能的实现步骤**，也就是对模块使用顺序进行编排，而不需要关心具体功能如何实现。

![](https://i-blog.csdnimg.cn/direct/1c49a66adef742bcb5888fa79a83df05.jpeg)

**模块部分**，每个模块完全独立，且具有统一的使用方式。

模块可以单独运行调试，复制粘贴文件夹即可无条件复用或替换升级。

![](https://i-blog.csdnimg.cn/direct/cdd92072bc1c426c95078b6801629df2.jpeg)

**Once 也提供了模块库**，可以一键下载更新模块。例如：检查参数、上传文件、数据库操作等模块。

后续也会持续扩展模块池。

![](https://i-blog.csdnimg.cn/direct/d23ac72f461d40f89492ade5e6ea235b.jpeg)

这里顺便提一下，大家比较关心的数据库模块。

数据库模块不需要配置数据表 entity，它会在工程启动时自动获取生成。数据操作上也更加简洁。

![](https://i-blog.csdnimg.cn/direct/20c298f96e3a4dc4a40c60aba1b18389.jpeg)

**业务部分**，由于每个后端接口只是对模块使用顺序进行编排。

所以这部分不需要写代码，**只需要通过 Json 配置即可**。这样能免去冗余代码的编写。

工程运行时，Json 配置会自动生成真正的 Java 代码。

![](https://i-blog.csdnimg.cn/direct/d1b9a37c699f4244b2909c2376847eae.jpeg)

Once 只是一种规则
-----------

**Once 只是一种规，则实质上是一个 SpringBoot3 工程。**

发生的错误，都可以以 SpringBoot 工程的视角进行排查。

Once 只是在 SpringBoot 工程的基础上做了 2 件事情。

![](https://i-blog.csdnimg.cn/direct/3cbca16dfc354b0d949ae3b5de9e7dbc.jpeg)

**一是规整化代码结构**，将代码分离为业务代码、模块代码、通用代码。规范了模块调用、日志、错误码等等。

![](https://i-blog.csdnimg.cn/direct/e035cd5522f74a90996a47cfaef6e7a4.jpeg)

**二是加入代码生成器 Christmas**，用于生成拼接代码。

例如，将业务代码从 Json 配置生成为真正的 Java 代码，从模块库下载模块，更新框架等。

![](https://i-blog.csdnimg.cn/direct/64393fea8009421b8c44e651cf4aae54.jpeg)

工作原理
----

接下来是工作原理，为了实现以上模块代码、业务代码分离，**Once 架构加入了数据**。

池数据池可以看作是一个接口中的全局变量，所有模块都可以对其进行添加、删除数据。

![](https://i-blog.csdnimg.cn/direct/f7e50a37377140e69f2aa990aeb73e71.jpeg)

​具体工作原理为：

**在接口逻辑开始时**，会将请求参数及其他重要对象存放在数据池中。

![](https://i-blog.csdnimg.cn/direct/2672c9aeaecd472b8c12ece74266440a.jpeg)

**调用模块时**，需要设置模块参数，以及将数据池传递给模块。

![](https://i-blog.csdnimg.cn/direct/b2c03ea8ca844b9a942e0c99a32dfc93.jpeg)

**模块按模块参数执行任务时**，可从数据池获取数据或对数据池数据进行修改。

![](https://i-blog.csdnimg.cn/direct/fb916e5fd1d64f6eadac9d4ac4089b14.jpeg)

**模块执行任务完毕后**，检查模块是否报错，不报错继续下一步，否则中断逻辑返回客户端。

![](https://i-blog.csdnimg.cn/direct/8f3cb056c8ee4d8bae69de79c918d4b6.jpeg)

**这里的报错返回只是默认行为**。可以添加逻辑选择器以实现更加复杂的逻辑，如发生错误重试、发生错误启动异常逻辑等。

![](https://i-blog.csdnimg.cn/direct/789b14cc5ea241fd878559779234575c.jpeg)

**接口返回时**，自动将数据池中用于记录返回数据的部分转换为 Json 字符串，并返回客户端。

![](https://i-blog.csdnimg.cn/direct/b8b01d06ade748ffa68ca4e12150d52d.jpeg)

设计愿景
----

Once 与其他技术框架的侧重点是不一样的。

**我们更关心项目过程、工程质量**，我们是按一个团队的视角进行设计的。主要是针对中大型项目，更看重整体成本、运维 / 迭代难度。

![](https://i-blog.csdnimg.cn/direct/51d23653e079444a96b058900ad7f16e.jpeg)

Once 可以缩减代码、提升代码复用度，但这并不是 Once 的唯一目的。

我们更希望通过模块积累降低新项目成本，希望团队能更合理地分，工人员流动不会对项目造成影响。

![](https://i-blog.csdnimg.cn/direct/8178948d5ea14209918325cfbf936d76.jpeg)

**模块代码开发**是需要有一定后端开发经验的程序员才能胜任的，我们希望这些工作能**独立开来**。

一方面模块可以**单独调试开发**，且无条件复用在别的项目。

另一方面由于项目进度等原因，某些模块可能是临时开发或者存在**缺陷**的，以后可以**单独替换**这些模块。

![](https://i-blog.csdnimg.cn/direct/50f586bcfa614e7bb2073bcd8ca42557.jpeg)

**业务代码**由于使用 **Json 配置**替代了编码，且**无需了解实际模块**的运行原理。

所以业务代码可以**交由经验尚浅**或者其他领域**的程序员**完成。

这样做除了能**加快开发效率**，更重要的是可以约束业务代码编写，让业务**逻辑更加清晰明了**。避免每次排查 BUG 时都需要先花很长时间理解逻辑，然后再逐一排查。

![](https://i-blog.csdnimg.cn/direct/b3b96b0a57394afbb3e90043164e8489.jpeg)

总结
--

最后，Once2 已经放在了 GitHub、Gitee。

我们也提供了完整的使用文档。

感兴趣的小伙伴可以尝试一下，点个 Star 就更好了。

![](https://i-blog.csdnimg.cn/direct/85be1b48bbe94b9eb4676e4f6a08c51c.jpeg)

以后，我们会不断推出新模块丰富模块池。

当模块池和用户量到达一定规模后，将继续推出图形编辑界面，进一步推动低代码平台。