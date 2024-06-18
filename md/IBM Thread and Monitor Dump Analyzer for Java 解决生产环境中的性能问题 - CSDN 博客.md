> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_30657541/article/details/98034829)

这个工具的使用和 HeapAnalyzer 一样，非常容易，同样提供了详细的 readme 文档，这里也简单举例如下：

```
#/usr/java50/bin/java -Xmx1000m -jar jca37.jar

```

###### 图 2. 通过 xManager 工具登录到 AIX 服务器上打开 jca 的效果图

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

笔者直接在生产环境下直接通过它对产生的 javacore 文件进行分析，令人惊喜的是，其分析结果非常明了，笔者心头的疑云在对结果进行进一步分析核实后也渐渐散去。

###### 图 3. jca 对 javacore.20090602.134015.430370.txt 的分析结果 —— 第一部分

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

从图中，我们可以清楚地看到引发错误的原因 —— The failure was caused because the class loader limit was exceeded。同时我们还能看出当前生产环境使用的 JRE 版本是：J2RE 5.0 IBM J9 2.3 AIX ppc-32 build j9vmap3223-20070201 ，这个 SR4 的版本有个问题，我们可以从分析结果图的第二部分的 NOTE 小节清楚地看到：

###### 图 4. jca 对 javacore.20090602.134015.430370.txt 的分析结果 —— 第二部分

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

_NOTE: Only for Java 5.0 Service Refresh 4 (build date:February 1st, 2007) and older. When you use delegated class loaders, the JVM can create a large number of ClassLoader objects. On IBM Java 5.0 Service Refresh 4 and older, the number of class loaders that are permitted is limited to 8192 by default and an OutOfMemoryError exception is thrown when this limit is exceeded. Use the -Xmxcl parameter to increase the number of class loaders allowed to avoid this problem, for example to 25000, by setting -Xmxcl25000, until the problem is resolved._

原来，OOM 竟然是因为这个原因产生的。那么到底是哪里加载的对象超过了这个 8192 的限制呢？接下来笔者结合分析结果和应用系统 log 进行了进一步的分析，更加验证了 JCA 分析结果的正确性。

在分析结果中可以看到 Current Thread ，就是对应引起 OOM 的应用服务器的线程，使用该线程的名称作为关键字在我们的 log 里进行搜索，迅速定位到了业务点——这是一个定时的调度，其功能是按固定时间间隔以 DBLink 的方式从外部应用的数据库系统中抽取数据，通过应用层逻辑转换后保存到内网数据库系统中。应用层的逻辑是对所有的业务数据进行循环，对每条业务数据进行到 POJO 的一一对照转换，这意味这一条业务数据需要 10 几个 POJO 进行组合对照，也就是说，如果外网在指定的时间间隔内数据量稍大，就会加载大量的对象，使 JVM 的 class loaders 瞬间超过 8192 的限制，而产生 OOM 的错误，从而使内存不断被消耗，直至宕机。

jca 还提供了对垃圾回收和线程状态的详细分析数据，可以参考如下两图：

###### 图 5. jca 对垃圾回收状态的分析数据

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

###### 图 6. jca 对垃圾线程状态的分析数据

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

```
在IBM Thread and Monitor Dump Analyzer for Java工具中，请求线程可分为以下几种状态：
　　1.死锁，Deadlock（重点关注）
　　2.执行中，Runnable（重点关注）
　　3.等待资源，Waiting on condition（重点关注）
　　4.等待监控器检查资源，Waiting on monitor
　　5.暂停，Suspended
　　6.对象等待中，Object.wait()
　　7.阻塞，Blocked（重点关注）
　　8.停止，Parked
　　Deadlock：死锁线程：一般指多个线程调用间，进入相互资源占用，导致一直等待无法释放的情况。
　　Runnable：一般指该线程正在执行状态中，该线程占用了资源，正在处理某个请求，有可能正在传递SQL到数据
　　库执行，有可能在对某个文件操作，有可能进行数据类型等转换。
　　Waiting on condition：等待资源，如果堆栈信息明确是应用代码，则证明该线程正在等待资源，一般是大
　　量读取某资源，且该资源采用了资源锁的情况下，线程进入等待状态，等待资源的读取。又或者，
　　正在等待其他线程的执行等。
　　Blocked：线程阻塞，是指当前线程执行过程中，所需要的资源长时间等待却一直未能获取到，被容器的线程管



```

问题核实后，如何进行解决呢？分析结果中也给出了相应的建议，从图 4 的 Recommended 小节我们可以看到：

Recommended -Xmxcl setting (only for IBM Java 5.0, up to and including Service Refresh 4 (build date:February 1st ,2007)) : 10,649 or greater。

为了考虑到对既有旧系统不产生影响，我们没有对 JRE 进行升级，而是采用了分析结果给出的建议，在应用服务器启动后设置 -Xmxcl 参数为 25000，为验证我们的修改是不是成功解决掉了 OOM 宕机的问题，笔者取消了对应用服务器的自动重启调度，通过一段时间的监控发现，系统运行良好，一直没有再出现 OOM 而宕机的问题，问题得以最终解决

参考：https://www.ibm.com/developerworks/cn/java/j-lo-javacore/

工具官方文档：http://www-01.ibm.com/support/docview.wss?uid=swg27011855&aid=1

https://www.ibm.com/support/knowledgecenter/en/SSLLVC_5.0.0/com.ibm.esupport.tool.tmda.doc/docs/readme.htm

相关：

#### [IBM 之 java 性能诊断工具初探 - IBM Support assitant 的使用](http://www9.iteye.com/blog/461462)

[使用 IBM heapAnalyzer 分析内存泄露的原因](http://blog.csdn.net/xu1314/article/details/7711768)

转载于: https://www.cnblogs.com/langtianya/p/6606118.html