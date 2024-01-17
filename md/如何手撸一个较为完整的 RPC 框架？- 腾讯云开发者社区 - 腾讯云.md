> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cloud.tencent.com](https://cloud.tencent.com/developer/article/1950610)

> 点击上方 “芋道源码”，选择 “设为星标” 管她前浪，还是后浪？ 能浪的浪，才是好浪！ 每天 10:33 更新文章，每天掉亿点点头发... 源码精品专栏 原创 | Java 2021 超神之路，很肝~ ......

点击上方 “芋道源码”，选择 “[设为星标](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247486188%26idx%3D3%26sn%3Df160d91ea23e5113e6077c500a2e30c4%26chksm%3Dfa49755dcd3efc4bf4f566fbbbf74c191d0b79f2d3222fd211bc52d80b5ef127f52b1158ed71%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)”

管她前浪，还是后浪？

能浪的浪，才是好浪！

每天 **10:33** 更新文章，每天掉亿点点头发...

源码精品专栏

*   [原创 | Java 2021 超神之路，很肝~](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [中文详细注释的开源项目](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247486264%26idx%3D1%26sn%3D475ac3f1ef253a33daacf50477203c80%26chksm%3Dfa497489cd3efd9f7298f5da6aad0c443ae15f398436aff57cb2b734d6689e62ab43ae7857ac%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [RPC 框架 Dubbo 源码解析](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247484647%26idx%3D1%26sn%3D9eb7e47d06faca20d530c70eec3b8d5c%26chksm%3Dfa497b56cd3ef2408f807e66e0903a5d16fbed149ef7374021302901d6e0260ad717d903e8d4%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [网络应用框架 Netty 源码解析](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247485054%26idx%3D2%26sn%3D9f3b85f7b8454634da6c5c2ded9b4dba%26chksm%3Dfa4979cfcd3ef0d9d2dd92d8d1bd8f1553abc6e2095a5d743e0b2c2afe4955ea2bbbd7a4b79d%26token%3D55862109%26lang%3Dzh_CN%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [消息中间件 RocketMQ 源码解析](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247486256%26idx%3D1%26sn%3D81daccd3fcd2953456c917630636fb26%26chksm%3Dfa497481cd3efd97d9239f5eab060e49dea9876a6046eadba0effb878d2fb51f3ba5733e4c0b%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [数据库中间件 Sharding-JDBC 和 MyCAT 源码解析](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247486257%26idx%3D1%26sn%3D4d3c9c675f8833157641a2e0b48e498c%26chksm%3Dfa497480cd3efd96fe17975b0b8b141e87fd0a62673e6a30b501460de80b3eb997056f09de08%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [作业调度中间件 Elastic-Job 源码解析](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247486258%26idx%3D1%26sn%3Dae5665ae9c3002b53f87cab44948a096%26chksm%3Dfa497483cd3efd950514da5a37160e7fd07f0a96f39265cf7ba3721985e5aadbdcbe7aafc34a%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [分布式事务中间件 TCC-Transaction 源码解析](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247486259%26idx%3D1%26sn%3Db023cf3dbf97e5da59db2f4ee632f5a6%26chksm%3Dfa497482cd3efd9402d71469f71863f71a6998b27e12ca2e00446b8178d79dcef0721d8e570a%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [Eureka 和 Hystrix 源码解析](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247486260%26idx%3D1%26sn%3D8f14c0c191d6f8df6eb34202f4ad9708%26chksm%3Dfa497485cd3efd93937143a648bc1b530bc7d1f6f8ad4bf2ec112ffe34dee80b474605c22db0%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)
*   [Java 并发源码](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247486261%26idx%3D1%26sn%3Dbd69f26aadfc826f6313ffbb95e44ee5%26chksm%3Dfa497484cd3efd92352d6fb3d05ccbaebca2fafed6f18edbe5be70c99ba088db5c8a7a8080c1%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

[来源：juejin.cn/post/](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

[6992867064952127524](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

*   [**缘 起**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
*   [**概 念 篇**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**RPC 是什么？**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**RPC 框架基本架构**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**RPC 框架通信流程以及涉及到的角色**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**具体调用过程**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
*   [**实 战 篇**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**技术选型**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**项目总体结构**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**项目实现介绍**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**自定义消息协议、编解码**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**序列化和反序列化**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**客户端 RPC 调用方式**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**整体架构和流程**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**环境搭建**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**项目测试**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
*   [**项 目 代 码**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)
    *   [**项目代码地址**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26chksm%3Dfa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8%26scene%3D21%26token%3D899450012%26lang%3Dzh_CN%23wechat_redirect&source=article&objectId=1950610)

![](https://ask.qcloudimg.com/http-save/1305760/797a0d4aa51b92481b128916b316a7e1.jpeg)

* * *

### [**缘 起**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

最近在公司分享了手撸 RPC，因此做一个总结。

> 基于 Spring Boot + MyBatis Plus + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能。 项目地址：https://github.com/YunaiV/ruoyi-vue-pro

### [**概 念 篇**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

#### [**RPC 是什么？**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

RPC 称远程过程调用（**Remote Procedure Call** ），用于解决分布式系统中服务之间的调用问题。通俗地讲，就是开发者能够像调用本地方法一样调用远程的服务。所以，RPC 的作用主要体现在这两个方面：

*   屏蔽远程调用跟本地调用的区别，让我们感觉就是调用项目内的方法；
*   隐藏底层网络通信的复杂性，让我们更专注于业务逻辑。

#### [**RPC 框架基本架构**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

下面我们通过一幅图来说说 RPC 框架的基本架构

![](https://ask.qcloudimg.com/http-save/1305760/b4e8fc7a0388c6f35fb13a7caceecf8c.png)

RPC 框架包含三个最重要的组件，分别是**客户端** 、**服务端** 和[**注册中心**](https://cloud.tencent.com/product/tse?from_column=20065&from=20065) 。在一次 RPC 调用流程中，这三个组件是这样交互的：

*   服务端在启动后，会将它提供的服务列表发布到注册中心，客户端向注册中心订阅服务地址；
*   客户端会通过本地代理模块 Proxy 调用服务端，Proxy 模块收到负责将方法、参数等数据转化成网络字节流；
*   客户端从服务列表中选取其中一个的服务地址，并将数据通过网络发送给服务端；
*   服务端接收到数据后进行解码，得到请求信息；
*   服务端根据解码后的请求信息调用对应的服务，然后将调用结果返回给客户端。

#### [**RPC 框架通信流程以及涉及到的角色**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

![](https://ask.qcloudimg.com/http-save/1305760/f3954ee4fbed4b91aea9d678e262d1e2.jpeg)

从上面这张图中，可以看见 RPC 框架一般有这些组件：**服务治理 (注册发现)、****负载均衡****、容错、序列化 / 反序列化、编解码、网络传输、线程池、动态代理等角色，当然有的 RPC 框架还会有连接池、日志、安全** 等角色。

#### [**具体调用过程**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

![](https://ask.qcloudimg.com/http-save/1305760/aebf94078630617fb32ed466ab934658.jpeg)

*   服务消费方 (**client** ) 以本地调用方式调用服务
*   **client stub** 接收到调用后负责将方法、参数等封装成能够进行网络传输的消息体
*   **client stub** 将消息进行编码并发送到服务端
*   **server stub** 收到消息后进行解码
*   **server stub** 根据解码结果调用本地的服务
*   本地服务执行并将结果返回给 **server stub**
*   **server stub** 将返回导入结果进行编码并发送至消费方
*   **client stub** 接收到消息并进行解码
*   服务消费方 (**client** ) 得到结果

##### [**RPC 消息协议**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

RPC 调用过程中需要将参数编组为消息进行发送，接收方需要解组消息为参数，过程处理结果同样需要经编组、解组。消息由哪些部分构成及消息的表示形式就构成了消息协议。

RPC 调用过程中采用的消息协议称为 RPC 消息协议。

> 基于微服务的思想，构建在 B2C 电商场景下的项目实战。核心技术栈，是 Spring Boot + Dubbo 。未来，会重构成 Spring Cloud Alibaba 。 项目地址：https://github.com/YunaiV/onemall

### [**实 战 篇**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

从上面的概念我们知道一个 RPC 框架大概有哪些部分组成，所以在设计一个 RPC 框架也需要从这些组成部分考虑。

从 RPC 的定义中可以知道，RPC 框架需要屏蔽底层细节，让用户感觉调用远程服务像调用本地方法一样简单，所以需要考虑这些问题：

*   用户使用我们的 RPC 框架时如何尽量少的配置
*   如何将服务注册到 ZK(这里注册中心选择 ZK) 上并且让用户无感知
*   如何调用透明 (尽量用户无感知) 的调用服务提供者
*   启用多个服务提供者如何做到动态负载均衡
*   框架如何做到能让用户自定义扩展组件 (比如扩展自定义负载均衡策略)
*   如何定义消息协议，以及编解码
*   ... 等等

上面这些问题在设计这个 RPC 框架中都会给予解决。

#### [**技术选型**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

*   **注册中心** 目前成熟的注册中心有 Zookeeper，Nacos，Consul，Eureka，这里使用 ZK 作为注册中心，没有提供切换以及用户自定义注册中心的功能。
*   **IO 通信框架** 本实现采用 Netty 作为底层通信框架，因为 Netty 是一个高性能事件驱动型的非阻塞的 IO(NIO) 框架，没有提供别的实现，也不支持用户自定义通信框架。
*   **消息协议** 本实现使用自定义消息协议，后面会具体说明。

#### [**项目总体结构**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

![](https://ask.qcloudimg.com/http-save/1305760/aaaab2bd5bbce023fb03c4ed99534a1f.jpeg)

从这个结构中可以知道，以 rpc 命名开头的是 rpc 框架的模块，也是本项目 RPC 框架的内容，而 **consumer 是服务消费者** ，**provider 是服务提供者** ，**provider-api 是暴露的服务** **API** 。

##### [**整体依赖情况**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

![](https://ask.qcloudimg.com/http-save/1305760/c563ea2e4c30a30c1740c72ad0801277.jpeg)

#### [**项目实现介绍**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

要做到用户使用我们的 RPC 框架时尽量少的配置，所以把 rpc 框架设计成一个 starter，用户只要依赖这个 starter, 基本那就可以了。

###### [**为什么要设计成两个 starter （client-starter/server-starter）?**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

这个是为了更好的体现出客户端和服务端的概念，消费者依赖客户端，服务提供者依赖服务端，还有就是最小化依赖。

###### [**为什么要设计成 starter ?**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

基于 spring boot 自动装配机制，会加载 starter 中的 spring.factories 文件，在文件中配置以下代码，这里我们 starter 的配置类就生效了，在配置类里面配置一些需要的 bean。

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.rrtv.rpc.client.config.RpcClientAutoConfiguration

```

**发布服务和消费服务**

*   **对于发布服务** 服务提供者需要在暴露的服务上增加注解 @RpcService，这个自定义注解是基于 @service 的，是一个复合注解，具备 @service 注解的功能，在 @RpcService 注解中指明服务接口和服务版本，发布服务到 ZK 上，会根据这个两个元数据注册。

![](https://ask.qcloudimg.com/http-save/1305760/021c4fb74f2c6f585399adea1ca81d81.jpeg)

*   **发布服务原理：** 服务提供者启动之后，根据 spring boot 自动装配机制，server-starter 的配置类就生效了，在一个 bean 的后置处理器（RpcServerProvider）中**获取被注解 @RpcService** 修饰的 bean，将注解的**元数据** **注册到 ZK** 上。

![](https://ask.qcloudimg.com/http-save/1305760/d436d5c0b5b0f7ae4ec8edb9eebee2e3.jpeg)

*   **对于消费服务** 消费服务需要使用自定义的 @RpcAutowired 注解标识，是一个复合注解，基于 @Autowired。

![](https://ask.qcloudimg.com/http-save/1305760/ddf29f72b11a7e5ffc56d286cfac82e3.jpeg)

*   **消费服务原理** 要让客户端无感知的调用服务提供者，就需要使用动态代理，如上面所示， HelloWordService 没有实现类，需要给它赋值代理类，在代理类中发起请求调用。 基于 **spring boot 自动装配** ，服务消费者启动，bean 后置处理器 **RpcClientProcessor** 开始工作。 它主要是遍历所有的 bean，判断每个 bean 中的属性是否有被 **@RpcAutowired** 注解修饰，有的话把该属性动态赋值代理类，这个再调用时会调用代理类的 **invoke** 方法。

![](https://ask.qcloudimg.com/http-save/1305760/ababedaa9d84ea326ed8cf11763b1da3.jpeg)

代理类 **invoke** 方法通过服务发现获取服务端元数据，封装请求，通过 **netty** 发起调用。

![](https://ask.qcloudimg.com/http-save/1305760/66234302e1eb18f05416213d2e65c067.jpeg)

##### [**注册中心**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

本项目注册中心使用 **ZK** ，由于注册中心被服务消费者和服务提供者都使用。所以把 ZK 放在 rpc-core 模块。

![](https://ask.qcloudimg.com/http-save/1305760/2dc59d0fe3eb92d2122dc62be5a006fa.jpeg)

**rpc-core** 这个模块如上图所示，核心功能都在这个模块。服务注册在 register 包下。

服务注册接口，具体实现使用 ZK 实现。

![](https://ask.qcloudimg.com/http-save/1305760/757997a4dc943cba26ad47a1aea726e8.jpeg)

##### [**负载均衡策略**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

负载均衡定义在 **rpc-core** 中，目前支持轮询（**FullRoundBalance** ）和随机（**RandomBalance** ），默认使用随机策略。由 **rpc-client-spring-boot-starter** 指定。

![](https://ask.qcloudimg.com/http-save/1305760/298356e57862864f8b6d8099fcca677a.jpeg)

通过 ZK 服务发现时会找到多个实例，然后通过负载均衡策略获取其中一个实例

![](https://ask.qcloudimg.com/http-save/1305760/7d01ff630fd1b3a211b834ca04c7bfc5.jpeg)

可以在消费者中配置 **rpc.client.balance=fullRoundBalance** 替换，也可以自定义负载均衡策略，通过实现接口 **LoadBalance** ，并将创建的类加入 **IOC** **容器** 即可。

由于我们配置 **@ConditionalOnMissingBean** ，所以会优先加载用户自定义的 **bean** 。

![](https://ask.qcloudimg.com/http-save/1305760/a680fd50886281086c55ca9f6b070b67.jpeg)

#### [**自定义消息协议、编解码**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

所谓协议，就是通信双方事先商量好规则，服务端知道发送过来的数据将如何解析。

*   自定义消息协议

![](https://ask.qcloudimg.com/http-save/1305760/5faa7d59d3075fe75e207c708d8f73d4.jpeg)

**魔数** ：魔数是通信双方协商的一个暗号，通常采用固定的几个字节表示。魔数的作用是防止任何人随便向服务器的端口上发送数据。 例如 java Class 文件开头就存储了魔数 0xCAFEBABE，在加载 Class 文件时首先会验证魔数的正确性。 **协议版本号** ：随着业务需求的变化，协议可能需要对结构或字段进行改动，不同版本的协议对应的解析方法也是不同的。 **序列化算法** ：序列化算法字段表示数据发送方应该采用何种方法将请求的对象转化为二进制，以及如何再将二进制转化为对象。 如 JSON、Hessian、Java 自带序列化等。 **报文类型** ：在不同的业务场景中，报文可能存在不同的类型。RPC 框架中有请求、响应、心跳等类型的报文。 **状态** ：状态字段用于标识请求是否正常（SUCCESS、FAIL）。 **消息 ID** ：请求唯一 ID，通过这个请求 ID 将响应关联起来，也可以通过请求 ID 做链路追踪。 **数据长度** ：标明数据的长度，用于判断是否是一个完整的数据包。 **数据内容** ：请求体内容。

*   **编解码** 编解码实现在 **rpc-core** 模块，在包 **com.rrtv.rpc.core.codec** 下。 自定义编码器通过继承 netty 的 **MessageToByteEncoder<MessageProtocol>** 类实现消息编码。

![](https://ask.qcloudimg.com/http-save/1305760/d2548c409e20c4606f31787f299e96a4.jpeg)

自定义解码器通过继承 **netty** 的 **ByteToMessageDecoder** 类实现消息解码。

![](https://ask.qcloudimg.com/http-save/1305760/e4ececd633a3b747216ae46d104304a4.jpeg)

![](https://ask.qcloudimg.com/http-save/1305760/e7ef8f6c2a0902d38786cfde05024466.jpeg)

##### [**解码时需要注意 TCP 粘包、拆包问题**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

###### [**什么是 TCP 粘包、拆包**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

**TCP** 传输协议是面向流的，没有数据包界限，也就是说消息无边界。客户端向服务端发送数据时，可能将一个完整的报文拆分成多个小报文进行发送，也可能将多个报文合并成一个大的报文进行发送。因此就有了拆包和粘包。

在网络通信的过程中，每次可以发送的数据包大小是受多种因素限制的，如 **MTU** 传输单元大小、滑动窗口等。

所以如果一次传输的网络包数据大小超过传输单元大小，那么我们的数据可能会拆分为多个数据包发送出去。

如果每次请求的网络包数据都很小，比如一共请求了 **10000** 次，**TCP** 并不会分别发送 **10000** 次。TCP 采用的 **Nagle** （批量发送，主要用于解决频繁发送小数据包而带来的网络拥塞问题） 算法对此作出了优化。

所以，网络传输会出现这样：

![](https://ask.qcloudimg.com/http-save/1305760/d99038803dc2e0db72390b1da40c2ad2.jpeg)

*   服务端恰巧读到了两个完整的数据包 A 和 B，没有出现拆包 / 粘包问题；
*   服务端接收到 A 和 B 粘在一起的数据包，服务端需要解析出 A 和 B；
*   服务端收到完整的 A 和 B 的一部分数据包 B-1，服务端需要解析出完整的 A，并等待读取完整的 B 数据包；
*   服务端接收到 A 的一部分数据包 A-1，此时需要等待接收到完整的 A 数据包；
*   数据包 A 较大，服务端需要多次才可以接收完数据包 A。

###### [**如何解决 TCP 粘包、拆包问题**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

**解决问题的根本手段**

找出消息的边界：

*   消息长度固定 每个数据报文都需要一个固定的长度。当接收方累计读取到固定长度的报文后，就认为已经获得一个完整的消息。当发送方的数据小于固定长度时，则需要空位补齐。 消息定长法使用非常简单，但是缺点也非常明显，无法很好设定固定长度的值，如果长度太大会造成字节浪费，长度太小又会影响消息传输，所以在一般情况下消息定长法不会被采用。
*   特定分隔符 在每次发送报文的尾部加上特定分隔符，接收方就可以根据特殊分隔符进行消息拆分。分隔符的选择一定要避免和消息体中字符相同，以免冲突。 否则可能出现错误的消息拆分。比较推荐的做法是将消息进行编码，例如 base64 编码，然后可以选择 64 个编码字符之外的字符作为特定分隔符。
*   消息长度 + 消息内容 消息长度 + 消息内容是项目开发中最常用的一种协议，接收方根据消息长度来读取消息内容。

本项目就是利用 “**消息长度 + 消息内容** ” 方式**解决 TCP 粘包、拆包问题** 的。

所以在解码时要判断数据是否够长度读取，没有不够说明数据没有准备好，继续读取数据并解码，这里这种方式可以获取一个个完整的数据包。

![](https://ask.qcloudimg.com/http-save/1305760/0fe83a14cdc3c2ac8da4147449d0e300.jpeg)

#### [**序列化和反序列化**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

序列化和反序列化在 **rpc-core** 模块 **com.rrtv.rpc.core.serialization** 包下，提供了 **HessianSerializati** **on** 和 **JsonSerialization** 序列化。

默认使用 **HessianSerialization** 序列化。用户不可以自定义。

**序列化性能：**

*   空间上

![](https://ask.qcloudimg.com/http-save/1305760/ccefc777f9b7ac9a962f5361615bc575.jpeg)

*   时间上

![](https://ask.qcloudimg.com/http-save/1305760/887add0d537045af6557a7c77b1cd651.jpeg)

**网络传输，使用 netty**

**netty** 代码固定的，值得注意的是 **handler** 的顺序不能弄错，以服务端为例，编码是出站操作（可以放在入站后面），解码和收到响应都是入站操作，解码要在前面。

![](https://ask.qcloudimg.com/http-save/1305760/c8fb8abc9e712e4acf5afc2d7767bcce.jpeg)

#### [**客户端 RPC 调用方式**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

成熟的 RPC 框架一般会提供四种调用方式，分别为**同步 Sync** 、**异步 Future** 、**回调 Callback** 和**单向 Oneway** 。

*   Sync 同步调用 客户端线程发起 RPC 调用后，当前线程会一直阻塞，直至服务端返回结果或者处理超时异常。

![](https://ask.qcloudimg.com/http-save/1305760/e08611297c4497cf1ce6b318b26eecf7.jpeg)

*   Future 异步调用 客户端发起调用后不会再阻塞等待，而是拿到 RPC 框架返回的 Future 对象，调用结果会被服务端缓存，客户端自行决定后续何时获取返回结果。当客户端主动获取结果时，该过程是阻塞等待的

![](https://ask.qcloudimg.com/http-save/1305760/b2eca7c75e589ed442e9824dc8e031ee.jpeg)

*   Callback 回调调用 客户端发起调用时，将 Callback 对象传递给 RPC 框架，无须同步等待返回结果，直接返回。当获取到服务端响应结果或者超时异常后，再执行用户注册的 Callback 回调

![](https://ask.qcloudimg.com/http-save/1305760/1cb280116d3c4b2b4cd9d9bb44c54a14.jpeg)

*   Oneway 单向调用 客户端发起请求之后直接返回，忽略返回结果。

![](https://ask.qcloudimg.com/http-save/1305760/607f1c3a46f682515daf7c7e6146c827.jpeg)

这里使用的是第一种：客户端同步调用，其他的没有实现。逻辑在 **RpcFuture** 中，使用 **CountDownLatc** **h** 实现阻塞等待（**超时等待** ）

![](https://ask.qcloudimg.com/http-save/1305760/92f6f1cf831d91c440bf45bbd2b576d6.jpeg)

#### [**整体架构和流程**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

![](https://ask.qcloudimg.com/http-save/1305760/1f70af029b7490f1bc153b47879a7b5a.jpeg)

流程分为三块：**服务提供者启动流程** 、**服务消费者启动** 、**调用过程。**

*   服务提供者启动 **-** 服务提供者 **provider** 会依赖 **rpc-server-spring-boot-starter。** **-** **ProviderApplication** 启动，根据 **springboot** 自动装配机制，**RpcServerAutoConfiguration** 自动配置生效。 **-** **RpcServerProvider** 是一个 bean 后置处理器，会发布服务，将服务元数据注册到 ZK 上。 **-** **RpcServerProvider.run** 方法会开启一个 netty 服务。
*   服务消费者启动 **-** 服务消费者 **consumer** 会依赖 **rpc-client-spring-boot-starter** **-** **ConsumerApplication** 启动，根据 **springboot** 自动装配机制，**RpcClientAutoConfiguration** 自动配置生效 **-** 将服务发现、负载均衡、代理等 bean 加入 **IOC 容器** **-** 后置处理器 **RpcClientProcessor** 会扫描 bean , 将被 **@RpcAutowired** 修饰的属性动态赋值为代理对象
*   调用过程 **-** 服务消费者 发起请求 **http://localhost‍:9090/hello/world?name=hello** **-** 服务消费者 调用 **helloWordService.sayHello()** 方法，会被代理到执行 **ClientStubInvocationHandler.invoke()** 方法 **-** 服务消费者 通过 ZK 服务发现获取服务元数据，找不到报错 404 **-** 服务消费者 自定义协议，封装请求头和请求体 **-** 服务消费者 通过自定义编码器 **RpcEncoder** 将消息编码 **-** 服务消费者 通过 服务发现获取到服务提供者的 ip 和端口， 通过 Netty 网络传输层发起调用 **-** 服务消费者 通过 **RpcFuture** 进入返回结果（超时）等待 **-** 服务提供者 收到消费者请求 **-** 服务提供者 将消息通过自定义解码器 **RpcDecoder** 解码 **-** 服务提供者 解码之后的数据发送到 **RpcRequestHandler** 中进行处理，通过反射调用执行服务端本地方法并获取结果 **-** 服务提供者 将执行的结果通过 编码器 **RpcEncoder** 将消息编码。（由于请求和响应的协议是一样，所以编码器和解码器可以用一套） **-** 服务消费者 将消息通过自定义解码器 **RpcDecoder** 解码 **-** 服务消费者 通过 **RpcResponseHandl** **er** 将消息写入 请求和响应 池中，并设置 **RpcFuture** 的响应结果 **-** 服务消费者 获取到结果

以上流程具体可以结合代码分析，代码后面会给出。

#### [**环境搭建**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

*   操作系统：Windows
*   集成开发工具：IntelliJ IDEA
*   项目技术栈：SpringBoot 2.5.2 + JDK 1.8 + Netty 4.1.42.Final
*   项目依赖管理工具：Maven 4.0.0
*   注册中心：Zookeeeper 3.7.0

#### [**项目测试**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

*   启动 Zookeeper 服务器：**bin/zkServer.cmd**
*   启动 provider 模块 **ProviderApplication**
*   启动 consumer 模块 **ConsumerApplication**
*   测试：浏览器输入 **http://localhost:9090/hello/world?name=hello** ，成功返回 **您好：hello** , rpc 调用成功

### [**项 目 代 码**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

#### [**项目代码地址**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMTA2NTU2Ng%3D%3D%26mid%3D2247487551%26idx%3D1%26sn%3D18f64ba49f3f0f9d8be9d1fdef8857d9%26scene%3D21%23wechat_redirect&source=article&objectId=1950610)

**https://gitee.com/listen_w/rpc**

* * *

* * *

欢迎加入我的知识星球，一起探讨架构，交流源码。加入方式，**长按下方二维码噢**：

![](https://ask.qcloudimg.com/http-save/1305760/8233b7e200cec4256005b1e7127894f3.png)

已在知识星球更新源码解析如下：

![](https://ask.qcloudimg.com/http-save/1305760/d26eb1fddbb0c1a62c0807884270c291.jpeg)

![](https://ask.qcloudimg.com/http-save/1305760/af876da591a29569b68f72878cbd8f49.jpeg)

![](https://ask.qcloudimg.com/http-save/1305760/7024ab18a3cb65d7e295915d85c0e648.jpeg)

![](https://ask.qcloudimg.com/http-save/1305760/9995590822e479ff93eaf2cb746095d2.jpeg)

最近更新《芋道 SpringBoot 2.X 入门》系列，已经 101 余篇，覆盖了 MyBatis、Redis、MongoDB、ES、分库分表、读写分离、SpringMVC、Webflux、权限、WebSocket、Dubbo、RabbitMQ、RocketMQ、Kafka、性能测试等等内容。

提供近 3W 行代码的 SpringBoot 示例，以及超 4W 行代码的电商微服务项目。

获取方式：点 “**在看**”，关注公众号并回复 **666** 领取，更多内容陆续奉上。

```
文章有帮助的话，在看，转发吧。谢谢支持哟 (*^__^*）

```

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan)，分享自微信公众号。

原始发表：2022-02-23，如有侵权请联系 [cloudcommunity@tencent.com](mailto:cloudcommunity@tencent.com) 删除