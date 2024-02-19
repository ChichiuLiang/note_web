> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gxaECvqR3yCql34n6yRzHg)

> 👉 **这是一个或许对你有用****的社群**
> 
> 🐱 一对一交流 / 面试小册 / 简历优化 / 求职解惑，欢迎加入「[**芋道快速开发平台**](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576697&idx=1&sn=a5f8a37fe0c6f05509c5bed6244471a8&chksm=fa4bd3c8cd3c5aded28a6b68a9944ce671f3d2748644f71550c0468d75058e90dd378a1babe8&scene=21#wechat_redirect)」知识星球。下面是星球提供的部分资料： 
> 
> *   [《项目实战（视频）》](http://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247561735&idx=1&sn=1b0a95d87fc647c3cf5e2b88576a8f55&chksm=f9f7e9e3ce8060f5809daa189fea465e95fa5445797f96fd8023f424c2acf4d31751a62792fc&scene=21#wechat_redirect)：从书中学，往事中 **“练”**
>     
> *   [《互联网高频面试题》](http://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247561735&idx=1&sn=1b0a95d87fc647c3cf5e2b88576a8f55&chksm=f9f7e9e3ce8060f5809daa189fea465e95fa5445797f96fd8023f424c2acf4d31751a62792fc&scene=21#wechat_redirect)：面朝简历学习，春暖花开
>     
> *   [《架构 x 系统设计》](http://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247561735&idx=1&sn=1b0a95d87fc647c3cf5e2b88576a8f55&chksm=f9f7e9e3ce8060f5809daa189fea465e95fa5445797f96fd8023f424c2acf4d31751a62792fc&scene=21#wechat_redirect)：摧枯拉朽，掌控面试高频场景题
>     
> *   [《精进 Java 学习指南》](http://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247561735&idx=1&sn=1b0a95d87fc647c3cf5e2b88576a8f55&chksm=f9f7e9e3ce8060f5809daa189fea465e95fa5445797f96fd8023f424c2acf4d31751a62792fc&scene=21#wechat_redirect)：系统学习，互联网主流技术栈
>     
> *   [《必读 Java 源码专栏》](http://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247561735&idx=1&sn=1b0a95d87fc647c3cf5e2b88576a8f55&chksm=f9f7e9e3ce8060f5809daa189fea465e95fa5445797f96fd8023f424c2acf4d31751a62792fc&scene=21#wechat_redirect)：知其然，知其所以然
>     

![](https://mmbiz.qpic.cn/mmbiz_gif/JdLkEI9sZfdWPYr0VKaXztEGHacpRyle7tbZkryrsxIpnAfjRt03ibrcloEZqlRPaVKcb0nD2PrYjtovwOAaFlA/640?wx_fmt=gif)

> 👉**这是一个或许对你有用的开源项目**
> 
> 国产 Star 破 10w+ 的开源项目，前端包括管理后台 + 微信小程序，后端支持单体和微服务架构。
> 
> 功能涵盖 RBAC 权限、SaaS 多租户、数据权限、商城、支付、工作流、大屏报表、微信公众号等等功能：
> 
> *   Boot 仓库：https://gitee.com/zhijiantianya/ruoyi-vue-pro
>     
> *   Cloud 仓库：https://gitee.com/zhijiantianya/yudao-cloud
>     
> *   视频教程：https://doc.iocoder.cn
>     
> 
> 【国内首批】支持 JDK 21 + SpringBoot 3.2.2、JDK 8 + Spring Boot 2.7.18 双版本 

[来源：blog.csdn.net/u010285974  
/article/details/85696239](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

*   [1. 背景](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    
*   [2. 分析](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    

*   [2.1 httpclient 反复创建开销](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    
*   [2.2 反复创建 tcp 连接的开销](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    
*   [2.3 重复缓存 entity 的开销](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    

*   [3. 实现](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    

*   [3.1 定义一个 keep alive strategy](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    
*   [3.2 配置一个 PoolingHttpClientConnectionManager](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    
*   [3.3 生成 httpclient](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    
*   [3.4 使用 httpclient 执行 method 时降低开销](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    

*   [4. 其他](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    

*   [4.1 httpclient 的一些超时配置](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    
*   [4.2 如果配置了 nginx 的话，nginx 也要设置面向两端的 keep-alive](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/6mychickmupUjNwwbOJP8GGFgFicUMbdmS3r8bUtm6nhvFaIXNtjWEK9opuAic1icXLZZjGPfa4G4F67qHjibszUjyQ/640?wx_fmt=jpeg&from=appmsg)

* * *

HttpClient 优化思路：

1.  池化
    
2.  长连接
    
3.  httpclient 和 httpget 复用
    
4.  合理的配置参数（最大并发请求数，各种超时时间，重试次数）
    
5.  异步
    
6.  多读源码
    

[1. 背景](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)
--------------------------------------------------------------------------------------------------------------------------------------------

我们有个业务，会调用其他部门提供的一个基于 http 的服务，日调用量在千万级别。使用了 httpclient 来完成业务。之前因为 qps 上不去，就看了一下业务代码，并做了一些优化，记录在这里。

先对比前后：优化之前，平均执行时间是 250ms；

优化之后，平均执行时间是 80ms，降低了三分之二的消耗，容器不再动不动就报警线程耗尽了，清爽~

> 基于 Spring Boot + MyBatis Plus + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能
> 
> *   项目地址：https://github.com/YunaiV/ruoyi-vue-pro
>     
> *   视频教程：https://doc.iocoder.cn/video/
>     

[2. 分析](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)
--------------------------------------------------------------------------------------------------------------------------------------------

项目的原实现比较粗略，就是每次请求时初始化一个 httpclient，生成一个 httpPost 对象，执行，然后从返回结果取出 entity，保存成一个字符串，最后显式关闭 response 和 client。

我们一点点分析和优化：

### [2.1 httpclient 反复创建开销](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

httpclient 是一个线程安全的类，没有必要由每个线程在每次使用时创建，全局保留一个即可。

### [2.2 反复创建 tcp 连接的开销](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

tcp 的三次握手与四次挥手两大裹脚布过程，对于高频次的请求来说，消耗实在太大。试想如果每次请求我们需要花费 5ms 用于协商过程，那么对于 qps 为 100 的单系统，1 秒钟我们就要花 500ms 用于握手和挥手。又不是高级领导，我们程序员就不要搞这么大做派了，改成 keep alive 方式以实现连接复用！

### [2.3 重复缓存 entity 的开销](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

原本的逻辑里，使用了如下代码：

```
HttpEntity entity = httpResponse.getEntity();

String response = EntityUtils.toString(entity);


```

这里我们相当于额外复制了一份 content 到一个字符串里，而原本的 httpResponse 仍然保留了一份 content，需要被 consume 掉，在高并发且 content 非常大的情况下，会消耗大量内存。并且，我们需要显式的关闭连接，ugly。

> 基于 Spring Cloud Alibaba + Gateway + Nacos + RocketMQ + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能
> 
> *   项目地址：https://github.com/YunaiV/yudao-cloud
>     
> *   视频教程：https://doc.iocoder.cn/video/
>     

[3. 实现](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)
--------------------------------------------------------------------------------------------------------------------------------------------

按上面的分析，我们主要要做三件事：一是单例的 client，二是缓存的保活连接，三是更好的处理返回结果。一就不说了，来说说二。

提到连接缓存，很容易联想到数据库连接池。httpclient4 提供了一个 PoolingHttpClientConnectionManager 作为连接池。接下来我们通过以下步骤来优化：

### [3.1 定义一个 keep alive strategy](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

关于 keep-alive，本文不展开说明，只提一点，是否使用 keep-alive 要根据业务情况来定，它并不是灵丹妙药。还有一点，keep-alive 和 time_wait/close_wait 之间也有不少故事。

在本业务场景里，我们相当于有少数固定客户端，长时间极高频次的访问服务器，启用 keep-alive 非常合适

再多提一嘴，http 的 keep-alive 和 tcp 的 KEEPALIVE 不是一个东西。回到正文，定义一个 strategy 如下：

```
ConnectionKeepAliveStrategy myStrategy = new ConnectionKeepAliveStrategy() {
    @Override
    public long getKeepAliveDuration(HttpResponse response, HttpContext context) {
        HeaderElementIterator it = new BasicHeaderElementIterator
            (response.headerIterator(HTTP.CONN_KEEP_ALIVE));
        while (it.hasNext()) {
            HeaderElement he = it.nextElement();
            String param = he.getName();
            String value = he.getValue();
            if (value != null && param.equalsIgnoreCase
               ("timeout")) {
                return Long.parseLong(value) * 1000;
            }
        }
        return 60 * 1000;//如果没有约定，则默认定义时长为60s
    }
};


```

### [3.2 配置一个 PoolingHttpClientConnectionManager](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

```
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
connectionManager.setMaxTotal(500);
connectionManager.setDefaultMaxPerRoute(50);//例如默认每路由最高50并发，具体依据业务来定


```

也可以针对每个路由设置并发数。

### [3.3 生成 httpclient](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

```
httpClient = HttpClients.custom()
     .setConnectionManager(connectionManager)
     .setKeepAliveStrategy(kaStrategy)
     .setDefaultRequestConfig(RequestConfig.custom().setStaleConnectionCheckEnabled(true).build())
     .build();


```

> ❝
> 
> 注意：使用 setStaleConnectionCheckEnabled 方法来逐出已被关闭的链接不被推荐。更好的方式是手动启用一个线程，定时运行 closeExpiredConnections 和 closeIdleConnections 方法，如下所示。
> 
> ❞

```
public static class IdleConnectionMonitorThread extends Thread {
    
    private final HttpClientConnectionManager connMgr;
    private volatile boolean shutdown;
    
    public IdleConnectionMonitorThread(HttpClientConnectionManager connMgr) {
        super();
        this.connMgr = connMgr;
    }
 
    @Override
    public void run() {
        try {
            while (!shutdown) {
                synchronized (this) {
                    wait(5000);
                    // Close expired connections
                    connMgr.closeExpiredConnections();
                    // Optionally, close connections
                    // that have been idle longer than 30 sec
                    connMgr.closeIdleConnections(30, TimeUnit.SECONDS);
                }
            }
        } catch (InterruptedException ex) {
            // terminate
        }
    }
    
    public void shutdown() {
        shutdown = true;
        synchronized (this) {
            notifyAll();
        }
    }
    
}


```

### [3.4 使用 httpclient 执行 method 时降低开销](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

> ❝
> 
> 这里要注意的是，不要关闭 connection。
> 
> ❞

一种可行的获取内容的方式类似于，把 entity 里的东西复制一份：

```
res = EntityUtils.toString(response.getEntity(),"UTF-8");
EntityUtils.consume(response1.getEntity());


```

但是，更推荐的方式是定义一个 ResponseHandler，方便你我他，不再自己 catch 异常和关闭流。在此我们可以看一下相关的源码：

```
public <T> T execute(final HttpHost target, final HttpRequest request,
        final ResponseHandler<? extends T> responseHandler, final HttpContext context)
        throws IOException, ClientProtocolException {
    Args.notNull(responseHandler, "Response handler");

    final HttpResponse response = execute(target, request, context);

    final T result;
    try {
        result = responseHandler.handleResponse(response);
    } catch (final Exception t) {
        final HttpEntity entity = response.getEntity();
        try {
            EntityUtils.consume(entity);
        } catch (final Exception t2) {
            // Log this exception. The original exception is more
            // important and will be thrown to the caller.
            this.log.warn("Error consuming content after an exception.", t2);
        }
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        }
        if (t instanceof IOException) {
            throw (IOException) t;
        }
        throw new UndeclaredThrowableException(t);
    }

    // Handling the response was successful. Ensure that the content has
    // been fully consumed.
    final HttpEntity entity = response.getEntity();
    EntityUtils.consume(entity);//看这里看这里
    return result;
}


```

可以看到，如果我们使用 resultHandler 执行 execute 方法，会最终自动调用 consume 方法，而这个 consume 方法如下所示：

```
public static void consume(final HttpEntity entity) throws IOException {
    if (entity == null) {
        return;
    }
    if (entity.isStreaming()) {
        final InputStream instream = entity.getContent();
        if (instream != null) {
            instream.close();
        }
    }
}


```

可以看到最终它关闭了输入流。

[4. 其他](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)
--------------------------------------------------------------------------------------------------------------------------------------------

通过以上步骤，基本就完成了一个支持高并发的 httpclient 的写法，下面是一些额外的配置和提醒：

### [4.1 httpclient 的一些超时配置](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

CONNECTION_TIMEOUT 是连接超时时间，SO_TIMEOUT 是 socket 超时时间，这两者是不同的。连接超时时间是发起请求前的等待时间；socket 超时时间是等待数据的超时时间。

```
HttpParams params = new BasicHttpParams();
//设置连接超时时间
Integer CONNECTION_TIMEOUT = 2 * 1000; //设置请求超时2秒钟 根据业务调整
Integer SO_TIMEOUT = 2 * 1000; //设置等待数据超时时间2秒钟 根据业务调整
 
//定义了当从ClientConnectionManager中检索ManagedClientConnection实例时使用的毫秒级的超时时间
//这个参数期望得到一个java.lang.Long类型的值。如果这个参数没有被设置，默认等于CONNECTION_TIMEOUT，因此一定要设置。
Long CONN_MANAGER_TIMEOUT = 500L; //在httpclient4.2.3中我记得它被改成了一个对象导致直接用long会报错，后来又改回来了
 
params.setIntParameter(CoreConnectionPNames.CONNECTION_TIMEOUT, CONNECTION_TIMEOUT);
params.setIntParameter(CoreConnectionPNames.SO_TIMEOUT, SO_TIMEOUT);
params.setLongParameter(ClientPNames.CONN_MANAGER_TIMEOUT, CONN_MANAGER_TIMEOUT);
//在提交请求之前 测试连接是否可用
params.setBooleanParameter(CoreConnectionPNames.STALE_CONNECTION_CHECK, true);
 
//另外设置http client的重试次数，默认是3次；当前是禁用掉（如果项目量不到，这个默认即可）
httpClient.setHttpRequestRetryHandler(new DefaultHttpRequestRetryHandler(0, false));


```

### [4.2 如果配置了 nginx 的话，nginx 也要设置面向两端的 keep-alive](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

现在的业务里，没有 nginx 的情况反而比较稀少。nginx 默认和 client 端打开长连接而和 server 端使用短链接。

> ❝
> 
> 注意 client 端的 keepalive_timeout 和 keepalive_requests 参数，以及 upstream 端的 keepalive 参数设置，这三个参数的意义在此也不再赘述。
> 
> ❞

以上就是我的全部设置。通过这些设置，成功地将原本每次请求 250ms 的耗时降低到了 80 左右，效果显著。

JAR 包如下：

```
<!-- httpclient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.6</version>
</dependency>


```

代码如下：

```
//Basic认证
private static final CredentialsProvider credsProvider = new BasicCredentialsProvider();
//httpClient
private static final CloseableHttpClient httpclient;
//httpGet方法
private static final HttpGet httpget;
//
private static final RequestConfig reqestConfig;
//响应处理器
private static final ResponseHandler<String> responseHandler;
//jackson解析工具
private static final ObjectMapper mapper = new ObjectMapper();
static {
    System.setProperty("http.maxConnections","50");
    System.setProperty("http.keepAlive", "true");
    //设置basic校验
    credsProvider.setCredentials(
            new AuthScope(AuthScope.ANY_HOST, AuthScope.ANY_PORT, AuthScope.ANY_REALM),
            new UsernamePasswordCredentials("", ""));
    //创建http客户端
    httpclient = HttpClients.custom()
            .useSystemProperties()
            .setRetryHandler(new DefaultHttpRequestRetryHandler(3,true))
            .setDefaultCredentialsProvider(credsProvider)
            .build();
    //初始化httpGet
    httpget = new HttpGet();
    //初始化HTTP请求配置
    reqestConfig = RequestConfig.custom()
            .setContentCompressionEnabled(true)
            .setSocketTimeout(100)
            .setAuthenticationEnabled(true)
            .setConnectionRequestTimeout(100)
            .setConnectTimeout(100).build();
    httpget.setConfig(reqestConfig);
    //初始化response解析器
    responseHandler = new BasicResponseHandler();
}
/*
 * 功能：返回响应
 * @author zhangdaquan
 * @param [url]
 * @return org.apache.http.client.methods.CloseableHttpResponse
 * @exception
 */
public static String getResponse(String url) throws IOException {
    HttpGet get = new HttpGet(url);
    String response = httpclient.execute(get,responseHandler);
    return response;
}
 
/*
 * 功能：发送http请求，并用net.sf.json工具解析
 * @author zhangdaquan
 * @param [url]
 * @return org.json.JSONObject
 * @exception
 */
public static JSONObject getUrl(String url) throws Exception{
    try {
        httpget.setURI(URI.create(url));
        String response = httpclient.execute(httpget,responseHandler);
        JSONObject json = JSONObject.fromObject(response);
        return json;
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
/*
 * 功能：发送http请求，并用jackson工具解析
 * @author zhangdaquan
 * @param [url]
 * @return com.fasterxml.jackson.databind.JsonNode
 * @exception
 */
public static JsonNode getUrl2(String url){
    try {
        httpget.setURI(URI.create(url));
        String response = httpclient.execute(httpget,responseHandler);
        JsonNode node = mapper.readTree(response);
        return node;
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
/*
 * 功能：发送http请求，并用fastjson工具解析
 * @author zhangdaquan
 * @param [url]
 * @return com.fasterxml.jackson.databind.JsonNode
 * @exception
 */
public static com.alibaba.fastjson.JSONObject getUrl3(String url){
    try {
        httpget.setURI(URI.create(url));
        String response = httpclient.execute(httpget,responseHandler);
        com.alibaba.fastjson.JSONObject jsonObject = com.alibaba.fastjson.JSONObject.parseObject(response);
        return jsonObject;
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}


```

* * *

欢迎加入我的知识星球，全面提升技术能力。

👉 加入方式，**“**长按**” 或 “**扫描**” 下方二维码噢**：

![](https://mmbiz.qpic.cn/mmbiz_png/6mychickmupUyQRGCOLQlJYyS47picY3OzaoUUJvwwrgEyR5U5fcYrCggDvjuDmAtibib1Zf6JjXlOic1PfvRCF2l4A/640?wx_fmt=png)

星球的**内容包括**：项目实战、面试招聘、源码解析、学习路线。

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdWPYr0VKaXztEGHacpRyle64G451XsWx4Ufc9FHYmNTqFdKdtNhyjlhrEC7Pic1dRfue78ib3TiaKBg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdWPYr0VKaXztEGHacpRylewlfAU6nBKS8tWSIrLcmSicsq0ECUIVnf0JHibsLARrp4Z3ZCh5oSvYMQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdWPYr0VKaXztEGHacpRyleptHSrmBV71SbQSolzZpmdqN2q03yLwk1bwZYNu2lr17gFjO0rh1ZdQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdWPYr0VKaXztEGHacpRyleYQLfl6qLTABQx5UrP0dXXv6cSv2TmBoouiciaZIXwyD88h5OIsM8lpkw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdWPYr0VKaXztEGHacpRyleicAvrkTkdGYRubqZdPNkqpoTmtOCQjkItotwtAbCxibsWUI4Dz0ILTEg/640?wx_fmt=png)

```
文章有帮助的话，在看，转发吧。

谢谢支持哟 (*^__^*）

```