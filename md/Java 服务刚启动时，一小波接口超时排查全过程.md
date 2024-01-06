> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7258847745527332924?utm_source=gold_browser_extension)

> 简介 我们组有一个流量较大的Java服务，每次发代码时，服务都会有一小波接口超时，之前简单分析过，发现这些超时的case仅发生在服务刚启动时，少量请求会耗时好几秒，但之后又马上恢复正常。

> 原创：扣钉日记（微信公众号 ID：codelogs），欢迎分享，非公众号转载保留此声明。

### 简介

我们组有一个流量较大的 Java 服务，每次发代码时，服务都会有一小波接口超时，之前简单分析过，发现这些超时的 case 仅发生在服务刚启动时，少量请求会耗时好几秒，但之后又马上恢复正常。

### 问题发生

如下，是我们服务的一次上线，可以看到，上线期间 (21:10 左右) 会有一小波 499 超时。  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1fdf153d9cb4c5f892f75372f1467e0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

而从我们全链路日志平台查看这些超时的调用，会发现外部网络操作 (如：rpc 调用、查询数据库等) 耗时不高，所以耗时来源于执行 java 代码而非外部调用。

但为啥就刚启动完成那会比较耗时，之后又正常了呢，有点经验的话，肯定会想到这里面估计发生了什么隐式操作，那 Java 代码执行时会有哪些隐式操作可能导致耗时高呢？ 我想到了如下几种情况：

1.  懒加载操作，如连接池初始化、缓存加载？

经过检查，发现这些都已在启动时加载，不会延迟到请求时。

2.  发生了 GC？

经过检查，启动时 GC 正常，耗时不高。

3.  JIT 即时编译功能导致？

java 代码默认是解释执行的，当某些代码被多次执行后，会被 JIT 编译成原生指令执行，执行性能相应提升，但我通过 JVM 参数`-Xint`关闭了 JIT 后，发现问题依然存在，故排除了此原因。

4.  执行过程中有锁？

经过检查代码，未发现锁的存在。

5.  操作系统相关隐式操作，上下文切换、缺页中断、文件 io 慢？

经初步检查，CPU、内存、磁盘使用率都正常，这部分深入排查比较费力，且有权限限制，暂先跳过。

那会是什么原因导致的？

### 问题排查

暂时没啥头绪，我打算先用 arthas 的`profile`命令，收集一些 CPU 火焰图看看。

由于超时仅发生在刚启动完成后的部分请求，之后又恢复正常，故我计划在启动完成后开始收集火焰图，每次收集 10s 的火焰图，收集 3 次，然后对比前后的火焰图，看看它们有什么不同，收集脚本如下：

```
function flamegraph_sample(){
    # 不断检测服务直到它启动完成
    while sleep 1; do curl -sS --connect-timeout 3 -m3 http://127.0.0.1:8080/health | grep ok && break; done
    pid=`pgrep -n java`
    for i in {1..3}; do
        java -jar arthas-boot.jar -c "profiler start --alluser" "$pid";
        sleep 10s;
        java -jar arthas-boot.jar -c "profiler stop --file /tmp/flamegraph_cpu_%t.html " "$pid";
    done
    java -jar arthas-boot.jar -c "stop" "$pid";
}
```

生成的前 2 个火焰图如下：  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd36505811ab47c4964721af03a63dfe~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96235d6b9a5e459dadf4c8f95b79d0fe~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)  
乍一看，火焰图中没有明显的瓶颈点，但经过仔细查看，在第一张火焰图中搜索 ClassLoader，可以搜到不少类加载操作 (红色部分)，而第二张则基本没有！

难道是类加载导致的？目前我有 80% 信心怀疑就是它导致的，但类加载有那么慢？

为此，我计划使用 profile 命令的`-e wall`模式收集刚启动完成时的调用栈，并使用`jfr`格式保存数据，其中`wall`模式适合诊断高耗时问题，而`jfr`格式数据会保存时间戳与线程名称，适合 case by case 分析，命令如下：

```
profiler start -e wall --file /tmp/result.jfr
```

收集到 jfr 文件后，使用 jmc 工具打开，然后我在日志平台上找到一个慢调用日志，它显示`http-nio-8080-exec-28`线程在`21:14:10`到`21:14:18`时间段是一次耗时近 8s 的慢调用，所以我用此条件在 jmc 里过滤出此 case 的调用栈数据，如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a426e2069964f769231b3c4e071f2e3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)  
可以发现，确实绝大多数耗时发生在类加载上，类加载之所以慢是因为加载类有锁竞争，而我们接口由于查表较多，确实会触发非常多类的加载，所以问题比较明显。

### 问题解决

知道原因后，解决起来就简单了，把类提前加载到 JVM 即可，为了简单，我直接使用了 spring 中的工具方法，如下：

```
private static final String[] CLASS_PREFIX_ARR = new String[] {
                "org.apache", "com.thoughtworks", "io.netty", "com.google", "io.grpc",
                "com.alibaba", "org.springframework", "cn.hutool", "com.fasterxml", "org.hibernate", 
                "io.opencensus", "org.redisson", "io.micrometer", "io.prometheus",
        };

PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
for (String classPrefix : CLASS_PREFIX_ARR) {
    Resource[] resources;
    try {
        resources = resolver.getResources(
                "classpath*:" + StringUtils.replaceChars(classPrefix, '.', '/') + "/**/*.class");
    } catch (IOException e) {
        ExceptionUtils.rethrow(e);
        return;
    }
    for (Resource resource : resources) {
        String className = null;
        try (InputStream is = resource.getInputStream()) {
            ClassReader cr = new ClassReader(is);
            className = StringUtils.replaceChars(cr.getClassName(), '/', '.');
            Class<?> clz = Class.forName(className);
            log.info("preLoadClass success: " + className + ", classLoader: " + clz.getClassLoader());
        } catch (Throwable e) { 
            log.warn("preLoadClass failed: " + className);
        }
    }
}
```

类预加载上线后，后面又进行过多次代码发布，发布过程中几乎不会再产生超时情况，问题确认已解决。

### 总结

此次问题的排查过程，还是用到了不少排查技巧的，总结一下：

1.  当看起来不应该慢的代码执行慢时，可以想想有哪些可能的隐式操作存在，此次 case 的隐式操作就是类加载。
2.  当诊断问题没有头绪时，可考虑使用 arthas 的`profile`命令来绘制火焰图，看从火焰图中能不能找到线索，尽管不会总是有效。
3.  当从 CPU 火焰图中看不出明显问题时，可通过对比问题前后的火焰图来找不同点。
4.  理解 profile 的`-e cpu`(默认) 与`-e wall`选项的差异，一般`-e cpu`诊断高 cpu 问题，而`-e wall`诊断高耗时问题，但如果是偶尔慢一下，需要 case by case 分析，可考虑使用`jfr`格式保存诊断数据。