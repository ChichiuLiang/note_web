> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_40927436/article/details/109148648)

### 常用的 JVM 配置参数有哪些？

*   -Xms: 初始大小内存，默认为物理内存的 1/64 等价于 - XX:InitialHeapSize
    
*   -Xmx: 最大分配内存，默认为物理内存的 1/4 等价于 - XX:MaxHeapSize
    
*   -Xss: 设置单个线程栈的大小，一般默认为 512k~1024k 等价于 - XX:ThreadStackSize
    
    *   当值等于 0 的时候，代表使用得是默认大小
*   -Xmn：设置年轻代大小
    
*   -XX:MetaspaceSize：设置元空间大小（**元空间与永久代最大的区别为：元空间并不在虚拟机中，而使用的是本地内存，因此，元空间只收本地内存的限制**）
    
    *   手动设置：-XX：MetaspaceSize=1024m
*   典型设置案例
    
*   -XX：+PrintGCDetails：输出详细 GC 收集日志信息
    

![](https://img-blog.csdnimg.cn/20201018191535364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDkyNzQzNg==,size_16,color_FFFFFF,t_70#pic_center)

![](https://img-blog.csdnimg.cn/20201018191610878.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDkyNzQzNg==,size_16,color_FFFFFF,t_70#pic_center)

当堆内存不够的话，会**爆出 OOM 错误**

![](https://img-blog.csdnimg.cn/20201018191625943.png#pic_center)

*   -XX:SurvivorRatio：设置新生代中 eden 和 S0/S1 空间比例，默认 -XX:SurvivorRatio=8，Eden : S0 : S1 = 8 : 1 : 1
    
    ​ -XX:SurvivorRatio=4==》Eden : S0 : S1 = 4 : 1 : 1
    
*   -XX:NewRatio：配置年轻代和老年代在堆结构的占比，默认 -XX:NewRatio=2 新生代占 1，老年代占 2，年轻代占整个堆的 1/3
    
    ​ -XX:NewRatio=4 新生代占 1，老年代占 4，年轻代占整个堆的 1/5
    
*   -XX:MaxTenuringThreshold：设置垃圾最大年龄。**默认是 15**。
    
    -XX:MaxTenuringThreshold=0：设置垃圾最大年龄。如果设置为 0 的话，则年轻代对象不经过 Survivor 区，直接进入老年代。对于老年代比较多的应用，可以提高效率。如果此值设置为一个较大的值，则年前对象会在 Survivor 区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代被回收的概率！
    

![](https://img-blog.csdnimg.cn/20201018191655978.png#pic_center)

**因为是由四位二进制数组成，所以垃圾最大年龄为 15. 因为 0-15 是 16 个数字**

![](https://img-blog.csdnimg.cn/20201018191711983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDkyNzQzNg==,size_16,color_FFFFFF,t_70#pic_center)