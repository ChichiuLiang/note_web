> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DgzPuw5TlKtiCrlYU8353g)

大家好，我是君哥。

RabbitMQ 这个消息队列相信很多程序员都用过，我第一次使用是在 2016 年，确实是一个老牌的消息队列了，但是为什么一直没有被淘汰呢？今天来聊一聊这个话题。

老旧差
---

### 发布历史

为什么说 RabbitMQ 老呢？下图是 RabbitMQ 最早的发布记录，可以看到 RabbitMQ 在 2007 年已经发布，已经有 16 年多的使用历史了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPH0ziaxNQ4se4KcwYnAWvHSibPJcZPgso0UvyxtRjIt79sHXHJicySKEYbw/640?wx_fmt=jpeg&from=appmsg)

### 小众

为什么说 RabbitMQ 比较小众呢？

一方面 RabbitMQ 使用 Erlang 语言编写，这是一个比较小众的编程语言，学习成本非常高，不像 Java、Scala、C 等编程语言学起来简单。所以虽然 RabbitMQ 也是开源的消息队列，但基于 RabbitMQ 做扩展和二次开发的情况是很少。

另一方面从使用的协议来看，RabbitMQ 支持 AMQP（Advanced Message Queuing Protocol） 协议，这也是主流消息队列不支持的。

AMQP 协议如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPHO1HZBuQspwiaOo8ZicZquNb9ImI3rzDOUgNYWwTYFrg4jWo2PzhaGbOQ/640?wx_fmt=png&from=appmsg)

有几个概念介绍一下：

*   Connection：一个网络连接，AMQP 协议通常使用长连接；
    
*   Channel：网络信道，建立在 Connection 之上的轻量级的连接，一个 Connection 可以有多个 Channel；
    
*   Exchange：交换器，接收消息后将消息路由转发给绑定（Binding）的 Queue；
    
*   Binding：Exchange 和 Queue 之间的虚拟连接；
    
*   Routing Key：这个概念在图中没有画，是指路由规则，用来确定 Exchange 将消息路由到哪些 Queue。
    

可以看到，好多概念在主流的消息队列比如 Kafka、RocketMQ 是没有的，所以说 RabbitMQ 比较小众。

### 性能差

在底层消息持久化的方式上，RabbitMQ 并没有使用 MMAP、Sendfile 等零拷贝技术，这是性能差的一个重要原因。

在架构上，RabbitMQ 提供了镜像队列来做 Master 的备份。如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPHFTYQDguCmSodL13rP2CBJy3ylNqBibcBr9OrkjvwgEcET1sL0WPyM3Q/640?wx_fmt=png&from=appmsg)

无论生产者发送消息，还是消费者拉取消息，如果请求发送到镜像队列，则镜像队列需要把请求转发到 Master 进行处理，Master 处理后再把结果回复给镜像节点，镜像队列回复给请求者。

在特定硬件环境下，RabbitMQ 支持的消息吞吐量在万级~ 十万级，相比 RocketMQ 的十万级~ 百万级和 Kafka 的百万级以上，吞吐量还是差一些。

受欢迎
---

从我过往的公司、身边的一些朋友、面试过的候选人简历可以看出，好多公司消息队列技术选型时选择了 RabbitMQ，这跟 RabbitMQ 老旧和性能差形成鲜明对比。

RabbitMQ 为什么这么受欢迎呢？

### 持续更新

虽然 RabbitMQ 老旧，但是并没有停止更新，而且更新还挺频繁，下图是 2023 年最近发布的几个版本：

![](https://mmbiz.qpic.cn/mmbiz_jpg/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPHyBLVdcMtAwODnaDNuraErpyGjwvc6KmXwibtMhcMOicsLTSuVFO3htKA/640?wx_fmt=jpeg&from=appmsg)

从 2007 年开始，RabbitMQ 已经有 16 年的使用历史，可以称得上是一个久经考验的战士，各种问题已经修复，学习资料丰富，性能稳定。

### 运维简单

RabbitMQ 是一个非常轻量级的消息队列，官方宣称开箱即用。在 Docker 上部署 RabbitMQ，三个命令就可以。

1.  拉取镜像
    

```
docker pull rabbitmq:3.8.2-management
```

2.  创建路径
    

```
mkdir /var/lib/rabbitmq
```

3.  启动容器
    

```
docker run -d --name rabbitmq3.8.2 -p 5672:5672 -p 15672:15672 -v `pwd`/data:/var/lib/rabbitmq --hostname myRabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin --privileged=true
```

这种开箱即用的效果，大大降低了学习成本和运维成本。

### 灵活路由

依托于 AMQP 中的 Exchange，RabbitMQ 提供了灵活的路由配置，有 4 种。

1.  Direct Exchange
    

生产者将消息发送给 Exchange 后，Exchange 通过 Routing Key 把消息路由到对应的队列。如下图（来自官网）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPHZ7q1Y12dssSeyV6epOPibGhBhr5tTuV53owuQ6ibvRqB5FCzxjuWGiciaw/640?wx_fmt=jpeg&from=appmsg)

2.  Fanout Exchange
    

生产者将消息发送给 Exchange 后，Exchange 将消息路由到所有绑定的队列，类似于广播模式。如下图（来自官网）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPHF7WesUEh6Mo4oRN8ibrtIXxiapxMRJG0EXZPKfxD0Bmd7aH6ardOjibrA/640?wx_fmt=jpeg&from=appmsg)

3.  Topic Exchange
    

这种路由策略首先定义一个 Topic，topic 中可以包含 `*` 和 `#`。`*` 可以代表一个单词，`#` 可以代表 0 或多个单词。如下图（来自官网）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPHN5yiadCafRcFRy8qN1DMgxK2jib9LIn8QuIx8uBApFqdtBh8sds6biaDw/640?wx_fmt=jpeg&from=appmsg)

图中 Topic 由三个单词`<celerity>.<colour>.<species>`组成，分别代表特征、颜色和物种，单词之间用`.`间隔。这样 Q1 将接收颜色为 orange 的所有消息，Q2 将接收物种为 rabbit 的消息和特征为 lazy 的消息。

4.  Headers Exchange
    

这种路由策略要求消息中需要携带 Headers(类似 Http 中的消息头)，队列跟 Routing Key 绑定时也要定义一个 Headers，只有绑定中定义的 Headers 跟消息中的 Header 匹配，才会路由到相应的队列。匹配规则有两种：

*   ALL：要求两个 Headers 中所有 key 和 value 匹配；
    
*   ANY：要求两个 Headers 任何一个 key 和 value 匹配。
    

如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPHVbuTVMQEicXSCH0kpI3SWvFS5kheUxmOaEuqaicwxGgnRicEU9FPuQWFw/640?wx_fmt=png&from=appmsg)

这种路由方式在定义绑定关系的时候就需要定义 Headers，如下代码：

```
@Beanpublic Binding binding1(HeadersExchange headersExchange,Queue queue1){ HashMap<String, Object> headers = new HashMap<>(); headers.put("key1","aaa"); headers.put("key2","bbb"); return BindingBuilder.bind(queue1).to(headersExchange).whereAll(headers).match();}public Binding binding2(HeadersExchange headersExchange,Queue queue2){ HashMap<String, Object> headers = new HashMap<>(); headers.put("key1","aaa"); headers.put("key2","bbb"); return BindingBuilder.bind(queue2).to(headersExchange).whereAny(headers).match();}
```

### 客户端丰富

RabbitMQ 客户端支持的编程语言是消息队列中最多的，很容易兼容自己系统使用的编程语言。参考下图（来自官网）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/a1gicTYmvicdicPiaIupO1p2qm96CRXCbCPH9n25daTPVZ3ydib3khTHvVyDTa0j7F1AmVhSZhbtPxKOtQxtic28tkvA/640?wx_fmt=jpeg&from=appmsg)

总结
--

RabbitMQ 虽然老旧，但具有运维简单、灵活路由、客户端丰富等特性。虽然吞吐量不高，但性能足够满足中小企业的使用需求。这让 RabbitMQ 成为非常受欢迎的消息队列。

感谢阅读，如果对你有帮助，请**点赞和再看**。欢迎加我微信：zhujinjun86。

号内回复 **seata**，下载《阿里分布式中间件 Seata 从入门到精通》

  

号内回复 **beijing**，下载我总结的北京上百家知名科技公司

  

号内回复 **aqs**，下载《40 张图精通 Java AQS》