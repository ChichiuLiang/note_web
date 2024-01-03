> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KF_7VdTGKr9MgRIGZ2TuNA)

```
胖虎和朋友原创的视频教程有兴趣的可以看看：




（文末附课程大纲）




👏2024 最新，Java成神之路，架构视频（点击查看）


😉超全技术栈的Java入门+进阶+实战！（点击查看）

```

1前言
---

一般初学者学习编码和 _错误处理_ 时，先知道 _编程语言_ 有一种处理错误的形式或约定（如Java就抛异常），然后就开始用这些工具。但却忽视这问题本质：**处理错误是为了写正确程序**。可是

2啥叫“正确”？
--------

**由解决的问题决定的。问题不同，解决方案不同。**

如一个web接口接受用户请求，参数age，也许业务要求字段是0～150之间整数。如输入字符串或负数就肯定不接受。一般在后端某地做输入合法性检查，不过就抛异常。

但归根到底这问题“正确”解决方法总是要以某种形式提示用户。而提示用户是某种前端工作，就要看界面是app，H5+AJAX还是类似于[jsp]的服务器产生界面。不管啥，你要根据需求去”设计一个修复错误“的流程。

如一个常见的流程要后端抛异常，然后一路到某个集中处理错误的代码，将其转换为某个HTTP的错误（业务错误码）提供给前端，前端再映射做”提示“。如用户输入非法请求，从逻辑上后端都没法自己修复，这是个“正确”的策略。

3报500了嘞！
--------

如用户上传一个头像，后端将图片发给[云存储]，结果云存储报500，咋办？你可能想重试，因为也许仅是[网络抖动]，重试就能正常执行。但若重试多次无效，若设计了某种热备方案，可能改为发到另一个服务器。“重试”和“使用备份的依赖”都是“立刻处理“。

但若重试无效，所有的[备份服务]也无效，也许就能像上面那样把错误抛给前端，提示用户“服务器开小差”。从这方案易看出，你想把错误抛到哪里是因为那个catch的地方是处理问题最方便的地方。一个问题的解决方案可能要几个不同的错误处理组合起来才能办到。

4NPE了！
------

你的程序抛个NPE。这一般就是程序员的bug：

*   要不就是程序员想表达一个东西”没有“，结果在后续处理中忘判断是否为null
    
*   要不就是在写代码时觉得100%不可能为null的地方出现了一个null
    

不管哪种，这错误用户总会看到一个很含糊的报错信息，这远远不够。“正确”办法是程序员自己能尽快发现它，并尽快修复。要做到这点，需要[监控系统]不断爬log，把问题报警出来。而非等用户找客服投诉。

5OOM了！
------

比如你的[后端程序]突然OOM挂了。挂的程序没法恢复自己。要做到“正确”，须在服务之外的容器考虑这问题。

如你的服务跑在[k8s]，他们会监控你程序状态，然后重启新的服务实例弥补挂掉的服务，还得调整流量，把去往宕机服务的流量切换到新实例。这的恢复因为跨系统所以不能仅用异常实现，但道理一样。

但光靠重启就“正确”了？若服务是完全无状态，问题不大。但若有状态，部分用户数据可能被执行一半的请求搞乱。因此重启要留意先“恢复数据到合法状态”。这又回到你要知道咋样才是“正确”的做法。只依靠简单的语法功能不能无脑解决这事。

```
2024最新架构课程，对标培训机构

👉点击查看：Java成神之路-进阶架构视频！

```

6提升维度
-----

*   一个工作线程的“外部容器“是管理工作线程的“master”
    
*   一个网络请求的“外部容器”是一个Web Server
    
*   一个用户进程的“外部容器”是[操作系统]
    
*   Erlang把这种supervisor-worker的机制融入到语言的设计
    

Web程序很大程度能把异常抛给顶层，是因为：

*   请求来自前端，对因为用户请求有误（数据合法性、权限、用户上下文状态）造成的问题，最终基本只能告诉用户。因此抛异常到一个集中处理错误的地方，把异常转换为某个业务错误码的方法，合理
    
*   后端服务一般无状态。这也是软件系统设计的一般原则。无状态才意味着可随时随地安心重启。用户数据不会因为因为下一条而会出问题
    
*   后端对数据的修改依赖DB的事务。因此一个改一半的、没提交的事务不会造成副作用。
    

但这3条件并非总成立。总能遇到：

*   一些处理逻辑并非无状态
    
*   也并非所有的数据修改都能用一个事务保护
    

尤其要注意对[微服务]的调用，对内存状态**「的修改是没有事务保护的」**，一不留神就会搞乱用户数据。比如下面代码段

7难以排查的代码段
---------

```
try {    
   int res1 = doStep1();    
   this.status1 += res1;    
   int res2 = doStep2();    
   this.status2 += res2;    
   // 抛个异常    
   int res3 = doStep3();    
   this.status3 = status1 + status2 + res3;    
} catch ( ...) {     
   // ...    
}    


```

先假设status1、status2、status3之间需维护某种不变的约束（invariant）。然后执行这段代码时，如在doStep3抛异常，下面对status3的赋值就不会执行。这时如不能将status1、status2的修改rollback，就会造成数据违反约束的问题。

而程序员很难发现这个数据被改坏了。坏数据还可能导致其他依赖这数据的代码逻辑出错（如原本应该给积分的，却没给）。而这种错误一般很难排查，从大量数据里找到不正确的那一小段何其困难。

8更难搞定的代码段
---------

```
// controller    
void controllerMethod(/* 参数 */) {    
  try {    
    return svc.doWorkAndGetResult(/* 参数 */);    
  } catch (Exception e) {    
    return ErrorJsonObject.of(e);    
  }    
}    
    
// svc    
void doWorkAndGetResult(/* some params*/) {    
    int res1 = otherSvc1.doStep1(/* some params */);    
    this.status1 += res1;    
    int res2 = otherSvc2.doStep2(/* some params */);    
    this.status2 += res2;    
    int res3 = otherSvc3.doStep3(/* some params */);    
    this.status3 = status1 + status2 + res3;    
    return SomeResult.of(this.status1, this.status2, this.status3);    
}    


```

难搞在于你写的时候可能以为doStep1～3这种东西即使抛异常也能被Controller里的catch。

在svc这层是不用处理任何异常，**因此不写[try……catch]天经地义**。但实际上doStep1、doStep2、doStep3任何一个抛异常都会造成svc的数据状态不一致。甚至你一开始都可以通过文档或其他沟通确定doStep1、doStep2、doStep3一开始都是必然可成功，不会抛错的，因此你写的代码一开始是对的。

但你可能无法控制他们的实现（如他们是另外一个团队开发的[jar]提供的），而他们的实现可能会改成抛错。你的代码可能在完全不自知情况下从“不会出问题”变成“可能出问题”…… 更可怕的类似代码不能正确工作：

```
void doWorkAndGetResult(/* some params*/) {    
    try {    
       int res1 = otherSvc1.doStep1(/* some params */);    
       this.status1 += res1;    
       int res2 = otherSvc2.doStep2(/* some params */);    
       this.status2 += res2;    
       int res3 = otherSvc3.doStep3(/* some params */);    
       this.status3 = status1 + status2 + res3;    
       return SomeResult.of(this.status1, this.status2, this.status3);    
   } catch (Exception e) {    
     // do rollback    
   }    
}    


```

你以为这样就会处理好数据rollback，甚至**「觉得这种代码优雅」**。但实际上doStep1～3每一个地方抛错，rollback的代码都不一样。

### 得这么写

```
void doWorkAndGetResult(/* some params*/) {    
    int res1, res2, res3;    
    try {    
       res1 = otherSvc1.doStep1(/* some params */);    
       this.status1 += res1;    
    } catch (Exception e) {    
       throw e;    
    }    
    
    try {    
      res2 = otherSvc2.doStep2(/* some params */);    
      this.status2 += res2;    
    } catch (Exception e) {    
      // rollback status1    
      this.status1 -= res1;    
      throw e;    
    }    
      
    try {    
      res3 = otherSvc3.doStep3(/* some params */);    
      this.status3 = status1 + status2 + res3;    
    } catch (Exception e) {    
      // rollback status1 & status2    
      this.status1 -= res1;    
      this.status2 -= res2;    
      throw e;    
   }     
}    


```

这才是得到正确结果的代码，在任何地方出错都能维护数据一致性。优雅吗？

看起来很丑。比go的`if err != nil`还丑。但要在正确性和优雅性取舍，肯定毫不犹豫选前者。作为程序员不能直接认为抛异常可解决任何问题，须学会写出有正确逻辑的程序，哪怕很难且看起来丑。

为达成高正确性，你不能总将自己大部分注意力放在“一切都OK的流程“，而把错误看作是可随便应付了事的工作或简单的相信exception可自动搞定一切。

9总结
---

希望程序员们对错误处理都要有敬畏之心。Java因为Checked Exception设计问题不得不避免使用，而Uncaughted Exception**实在是太过于弱鸡，是不能给程序员提供更好地帮助的**。

因此，程序员在每次抛错或者处理错误的时候都要三省吾身：

*   这个错误的处理是正确的吗？
    
*   会让用户看到什么？
    
*   会不会搞乱数据？
    

不要以为自己抛了个异常就不管了。在[编译器]不能帮上太多忙的时候，好好写UT来保护代码脆弱的正确性。

**为人为己，请多写正确的代码**。

来源｜JavaEdge

  

```
胖虎联合两位大佬朋友，一位是知名培训机构讲师和科大讯飞架构，联合打造了《Java架构师成长之路》的视频教程。完全对标外面2万左右的培训课程。

除了基本的视频教程之外，还提供了超详细的课堂笔记，以及源码等资料包..




课程阶段：

Java核心 提升阅读源码的内功心法
深入讲解企业开发必备技术栈，夯实基础，为跳槽加薪增加筹码

分布式架构设计方法论。为学习分布式微服务做铺垫
学习NetFilx公司产品，如Eureka、Hystrix、Zuul、Feign、Ribbon等，以及学习Spring Cloud Alibabba体系
微服务架构下的性能优化
中间件源码剖析
元原生以及虚拟化技术
从0开始，项目实战 SpringCloud Alibaba电商项目


点击下方超链接查看详情（或者点击文末阅读原文）：

(点击查看)  2024年，最新Java架构师成长之路 视频教程！

以下是课程大纲，大家可以双击打开原图查看


```