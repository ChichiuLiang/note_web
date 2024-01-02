> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwNTI2ODY5OA==&mid=2649938635&idx=1&sn=fc22d218fb1529e8f4c1fcb659838d75&chksm=8f35097eb8428068258b95a652e46ed082499b3952ac11af2027400db758a1ac4b158412aeea&scene=21#wechat_redirect)

![](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/smiley/smiley_83.png) 戳蓝字「TopCoder」关注我们哦！

![](https://mmbiz.qpic.cn/mmbiz_png/jOD3PUUxTibPRnc9oqChpKIhYiaRn4l6PNfHgYMMvnahOtC5iaC6owShCHb1dWYj4a8nx7X2picQA9VrvOJk4jmjZw/640?wx_fmt=png)

> 编者注：Netty 是 Java 领域有名的开源网络库，特点是高性能和高扩展性，因此很多流行的框架都是基于它来构建的，比如我们熟知的 Dubbo、Rocketmq、Hadoop 等，针对高性能 RPC，一般都是基于 Netty 来构建，比如 sock-bolt。总之一句话，Java 小伙伴们需要且有必要学会使用 Netty 并理解其实现原理。

netty 旨在为可维护的高性能、高可扩展性协议服务器和客户端的快速开发提供异步事件驱动的网络应用程序框架和工具。换句话说，Netty 是一个 NIO 客户端服务器框架，可以快速轻松地开发协议服务器和客户端等网络应用程序。它极大地简化并简化了 TCP 和 UDP 套接字服务器开发等网络编程。  

> 学习 netty 原理细节，看 netty 源码是必不可少的，那首先来看下如何编译源码：
> 
> 1.  从 github 下载 netty 4.x 源码
>     
> 2.  如果缺少 XxxObjectHashMap 类，这些类是在编译时自动生成的，可以执行 mvn clean install 或者 cd common && mvn clean install 命令即可。
>     
> 3.  打开 idea，开启源码阅读之旅 :)
>     

除了看源码，可以结合一些书籍来看，学习效果更好。关于 Netty 的书籍，笔者这里推荐一本 李林锋 写的《Netty 权威指南》，这本书对于 Netty 的基础概念和 NIO 部分讲解的还是不错的，不过有点地方感觉有点贴代码凑字数嫌疑，整体来说还算不错。

![](https://mmbiz.qpic.cn/mmbiz_jpg/jOD3PUUxTibPRnc9oqChpKIhYiaRn4l6PNphueOia737xgPcvO3rzOnr9jO1CUXReg960Cj78FtPFPwPgOB2xsU1g/640?wx_fmt=jpeg)

什么是 Netty
---------

Netty 是一个事件驱动的高性能 Java 网络库，是一个隐藏了背后复杂性而提供一个易于使用的 API 的客户端 / 服务端框架。Netty 以其高性能和可扩展性，使开发者专注于真正感兴趣的地方。它的一个主要目标就是促进 “关注点分离”：**使业务逻辑从网络基础设施应用程序中分离**。

> 不仅仅是 Netty 框架，其他框架的设计目的也大都是为了使业务程序和底层技术解耦，使程序员更加专注于业务逻辑实现，提高开发质量和效率。Netty 为什么性能如此之高，主要是其内部的 Reactor 模型机制。

### Netty 核心组件

*   **Bootstrap 和 ServerBootstrap**：Netty 应用程序通过设置 bootstrap 引导类来完成，该类提供了一个用于应用程序网络层配置的容器。Bootstrap 服务端的是 ServerBootstrap，客户端的是 Bootstrap。
    
*   **Channel**：Netty 中的接口 Channel 定义了与 socket 丰富交互的操作集：bind, close, config, connect, isActive, isOpen, isWritable, read, write 等等。
    
*   **ChannelHandler**：ChannelHandler 支持很多协议，并且提供用于数据处理的容器，ChannelHandler 由特定事件触发， 常用的一个接口是 ChannelInboundHandler，该类型处理入站读数据（socket 读事件）。
    
*   **ChannelPipeline**：ChannelPipeline 提供了一个容器给 ChannelHandler 链并提供了一个 API 用于管理沿着链入站和出站事件的流动。每个 Channel 都有自己的 ChannelPipeline，当 Channel 创建时自动创建的。下图说明了 ChannelHandler 和 ChannelPipeline 二者的关系：
    

![](https://mmbiz.qpic.cn/mmbiz_png/jOD3PUUxTibPRnc9oqChpKIhYiaRn4l6PNyRFR4r5KM9YU70tB7fAK07cwumDvMIy8IvWoAhgla6zIn4qokGNN8Q/640?wx_fmt=png)

*   **EventLoop**：EventLoop 用于处理 Channel 的 I/O 操作。一个单一的 EventLoop 通常会处理多个 Channel 事件。一个 EventLoopGroup 可以含有多于一个的 EventLoop 和 提供了一种迭代用于检索清单中的下一个。
    
*   **ChannelFuture**：Netty 所有的 I/O 操作都是异步。因为一个操作可能无法立即返回，我们需要有一种方法在以后获取它的结果。出于这个目的，Netty 提供了接口 ChannelFuture, 它的 addListener 方法
    

Netty 是一个非阻塞、事件驱动的网络框架。Netty 实际上是使用 Threads（ 多线程） 处理 I/O 事件的，对于熟悉多线程编程的读者可能会需要关注同步代码。这样的方式不好，因为同步会影响程序的性能，Netty 的设计保证程序处理事件不会有同步。因为某个 Channel 事件是被添加到一个 EventLoop 中的，以后该 Channel 事件都是由该 EventLoop 来处理的，而 EventLoop 是一个线程来处理的，也就是说 Netty 不需要同步 IO 操作，EventLoop 与 EventLoopGroup 的关系可以理解为线程与线程池的关系一样。

### Buffer（缓冲）

ByteBuf 是字节数据的容器，所有的网络通信都是基于底层的字节流传输，ByteBuf 是一个很好的经过优化的数据容器，我们可以将字节数据有效的添加到 ByteBuf 中或从 ByteBuf 中获取数据。为了便于操作，ByteBuf 提供了两个索引：一个用于读，一个用于写。我们可以按顺序读取数据，也可以通过调整读取数据的索引或者直接将读取位置索引作为参数传递给 get 方法来重复读取数据。

**ByteBuf 使用模式**

**堆缓冲区 ByteBuf** 将数据存储在 JVM 的堆空间，这是通过将数据存储在数组的实现。堆缓冲区可以快速分配，当不使用时也可以快速释放。它还提供了直接访问数组的方法，通过 ByteBuf.array() 来获取 byte[] 数据。

堆缓冲区 ByteBuf 使用示例：

```
1ByteBuf heapBuf = ...;2if (heapBuf.hasArray()) {3    byte[] array = heapBuf.array();4    int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();5    int length = heapBuf.readableBytes();6    handleArray(array, offset, length);7}
```

**直接缓冲区 ByteBuf，**在 JDK1.4 中被引入 NIO 的 ByteBuffer 类允许 JVM 通过本地方法调用分配内存，其目的是通过免去中间交换的内存拷贝, 提升 IO 处理速度; 直接缓冲区的内容可以驻留在垃圾回收扫描的堆区以外。DirectBuffer 在`-XX:MaxDirectMemorySize=xx`M 大小限制下, 使用 Heap 之外的内存, GC 对此” 无能为力”，也就意味着规避了在高负载下频繁的 GC 过程对应用线程的中断影响。

### Netty 示例代码

了解了 Netty 基础概念之后，一起看下 Netty 的使用示例，下面以 TCP server、TCP client、http server 为例，由于示例代码不难，所以不再赘述，直接上代码。

#### TCP Server

```
1public static void main(String[] args) { 2    EventLoopGroup bossGroup = new NioEventLoopGroup(1); 3    EventLoopGroup workerGroup = new NioEventLoopGroup(); 4 5    try { 6        ServerBootstrap boot = new ServerBootstrap(); 7        boot.group(bossGroup, workerGroup) 8            .channel(NioServerSocketChannel.class) 9            .localAddress(8080)10            .childHandler(new ChannelInitializer<SocketChannel>() {11                @Override12                protected void initChannel(SocketChannel ch) throws Exception {13                    ch.pipeline().addLast(new EchoHandler());14                }15            });1617        // start18        ChannelFuture future = boot.bind().sync();19        future.channel().closeFuture().sync();20    } catch (Exception e) {21        e.printStackTrace();22    } finally {23        // shutdown24        bossGroup.shutdownGracefully();25        workerGroup.shutdownGracefully();26    }27}2829public class EchoHandler extends ChannelInboundHandlerAdapter {30    @Override31    public void channelRead(ChannelHandlerContext ctx, Object msg) {32        ByteBuf in = (ByteBuf) msg;33        System.out.println(in.toString(CharsetUtil.UTF_8));34        ctx.write(msg);35    }3637    @Override38    public void channelReadComplete(ChannelHandlerContext ctx) {39        ctx.flush();40    }4142    @Override43    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {44        cause.printStackTrace();45        ctx.close();46    }47}
```

#### TCP client

```
1public static void main(String[] args) { 2 3    EventLoopGroup group = new NioEventLoopGroup(); 4    try { 5        Bootstrap b = new Bootstrap(); 6        b.group(group) 7        .channel(NioSocketChannel.class) 8        .option(ChannelOption.TCP_NODELAY, true) 9        .handler(new ChannelInitializer<SocketChannel>() {10            @Override11            public void initChannel(SocketChannel ch) throws Exception {12                ChannelPipeline p = ch.pipeline();13                //p.addLast(new LoggingHandler(LogLevel.INFO));14                p.addLast(new EchoClientHandler());15            }16        });1718        // Start the client.19        ChannelFuture f = b.connect("localhost", 8081).sync();20        f.channel().closeFuture().sync();21    } catch (Exception e) {22      e.printStackTrace();23    } finally {24        group.shutdownGracefully();25    }26}2728public class EchoClientHandler extends ChannelInboundHandlerAdapter {2930    private final ByteBuf message;3132    public EchoClientHandler() {33        message = Unpooled.buffer(256);34        message.writeBytes("hello netty".getBytes(CharsetUtil.UTF_8));35    }3637    @Override38    public void channelActive(ChannelHandlerContext ctx) {39        ctx.writeAndFlush(message);40    }4142    @Override43    public void channelRead(ChannelHandlerContext ctx, Object msg) {44        System.out.println(((ByteBuf) msg).toString(CharsetUtil.UTF_8));45        ctx.write(msg);46        try {47            Thread.sleep(1000);48        } catch (InterruptedException e) {49            e.printStackTrace();50        }51    }5253    @Override54    public void channelReadComplete(ChannelHandlerContext ctx) {55        ctx.flush();56    }5758    @Override59    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {60        // Close the connection when an exception is raised.61        cause.printStackTrace();62        ctx.close();63    }64}
```

netty client 端在什么时候将 channel 注册到 selector 上的呢？是在创建 channel 之后，就注册到 selector 的，相关代码在 initAndRegister 方法中：

```
1final ChannelFuture initAndRegister() { 2    Channel channel = null; 3    try { 4        // 创建(netty自定义)Channel实例，并初始化 5        // channel为 NioServerSocketChannel 实例,NioServerSocketChannel的父类AbstractNioChannel保存有nio的ServerSocketChannel 6        channel = channelFactory.newChannel(); 7        init(channel); 8    } catch (Throwable t) { 9        if (channel != null) {10            // channel can be null if newChannel crashed (eg SocketException("too many open files"))11            channel.unsafe().closeForcibly();12            // as the Channel is not registered yet we need to force the usage of the GlobalEventExecutor13            return new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE).setFailure(t);14        }15        // as the Channel is not registered yet we need to force the usage of the GlobalEventExecutor16        return new DefaultChannelPromise(new FailedChannel(), GlobalEventExecutor.INSTANCE).setFailure(t);17    }1819    // 向Selector注册channel20    ChannelFuture regFuture = config().group().register(channel);21    if (regFuture.cause() != null) {22        if (channel.isRegistered()) {23            channel.close();24        } else {25            channel.unsafe().closeForcibly();26        }27    }2829    // If we are here and the promise is not failed, it's one of the following cases:30    // 1) If we attempted registration from the event loop, the registration has been completed at this point.31    //    i.e. It's safe to attempt bind() or connect() now because the channel has been registered.32    // 2) If we attempted registration from the other thread, the registration request has been successfully33    //    added to the event loop's task queue for later execution.34    //    i.e. It's safe to attempt bind() or connect() now:35    //         because bind() or connect() will be executed *after* the scheduled registration task is executed36    //         because register(), bind(), and connect() are all bound to the same thread.3738    return regFuture;39}
```

initAndRegister 之后会执行 connect 动作，注意，真正的 channel.connect 动作是由 NioEventLoop 线程来完成的，当连接三次握手完成之后，会触发该 channel 的 ACCEPT 事件，也就是 NIOEventLoop 中处理事件的流程。

#### Http server

```
1public static void main(String[] args) { 2    EventLoopGroup bossGroup = new NioEventLoopGroup(1); 3    EventLoopGroup workerGroup = new NioEventLoopGroup(); 4 5    try { 6        ServerBootstrap boot = new ServerBootstrap(); 7        boot.group(bossGroup, workerGroup) 8            .channel(NioServerSocketChannel.class) 9            .localAddress(8080)10            .childHandler(new ChannelInitializer<SocketChannel>() {11                @Override12                protected void initChannel(SocketChannel ch) throws Exception {13                    ch.pipeline()14                            .addLast("decoder", new HttpRequestDecoder())15                            .addLast("encoder", new HttpResponseEncoder())16                            .addLast("aggregator", new HttpObjectAggregator(512 * 1024))17                            .addLast("handler", new HttpHandler());18                }19            });2021        // start22        ChannelFuture future = boot.bind().sync();23        future.channel().closeFuture().sync();24    } catch (Exception e) {25        e.printStackTrace();26    } finally {27        // shutdown28        bossGroup.shutdownGracefully();29        workerGroup.shutdownGracefully();30    }31}3233public class HttpHandler extends SimpleChannelInboundHandler<FullHttpRequest> {34    @Override35    protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest msg) throws Exception {36        DefaultFullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1,37                HttpResponseStatus.OK,38                Unpooled.wrappedBuffer("hello netty".getBytes()));3940        HttpHeaders heads = response.headers();41        heads.add(HttpHeaderNames.CONTENT_TYPE, HttpHeaderValues.TEXT_PLAIN + "; charset=UTF-8");42        heads.add(HttpHeaderNames.CONTENT_LENGTH, response.content().readableBytes()); // 343        heads.add(HttpHeaderNames.CONNECTION, HttpHeaderValues.KEEP_ALIVE);4445        ctx.writeAndFlush(response);46    }47}
```

 _**推荐阅读**_ 

*   [Java 常见几种动态代理的对比](http://mp.weixin.qq.com/s?__biz=MzIwNTI2ODY5OA==&mid=2649938597&idx=1&sn=f646f74287c43aeedb3670029cca80e5&chksm=8f350910b84280060218acc59b2200381cd33fac2c1cf6cf64068dcfd79407babfc8175e23c5&scene=21#wechat_redirect)
    
*   [程序员必看 | mockito 原理浅析](http://mp.weixin.qq.com/s?__biz=MzIwNTI2ODY5OA==&mid=2649938607&idx=1&sn=7e17607eb5a537f7734631030d289351&chksm=8f35091ab842800cc88e928fdedd763334c4e6c4c2f750bfc2a04499d41a629740c2f16e78d4&scene=21#wechat_redirect)
    
*   [Eureka 原理分析](http://mp.weixin.qq.com/s?__biz=MzIwNTI2ODY5OA==&mid=2649938614&idx=1&sn=ded4dc4d889e295b54b45e1393335142&chksm=8f350903b842801560a8ee0bc14b935c9a77cd73bbf508c497fe842bb1c38cbc04fd01f2dda1&scene=21#wechat_redirect)
    
*   [MQ 初窥门径【面试必看的 Kafka 和 RocketMQ 存储区别】](http://mp.weixin.qq.com/s?__biz=MzIwNTI2ODY5OA==&mid=2649938550&idx=1&sn=d8f0f2f4cc43cbc3664ae813f3784219&chksm=8f3509c3b84280d51e6af1baa2f30a184e1ea16d28bda7e942cc02a1c4b03ba780f5af8f13df&scene=21#wechat_redirect)
    
*   [java lambda 深入浅出](http://mp.weixin.qq.com/s?__biz=MzIwNTI2ODY5OA==&mid=2649938626&idx=1&sn=080f215fe8e0489c91132a6b3ca12f27&chksm=8f350977b84280615f284fc6737da6531bc0be6ef7f8bf093fe46b212d474ceb2d0432726065&scene=21#wechat_redirect)
    

  
欢迎小伙伴**关注【TopCoder】**阅读更多精彩好文。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/jOD3PUUxTibP4TMsUhZhl8N3f7mphlhYMhnu8RbooMP7Ab6UbHF4BITSCe6D8hPy3VyzTu2Yp23vRRRTzJj8AQg/640?wx_fmt=jpeg)