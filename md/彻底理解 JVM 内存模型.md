> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/265664787)

一、JDK，JRE，JVM 三者之间的关系
---------------------

![](https://pic1.zhimg.com/v2-342eef4813df0a0c301b0bbe9e32e428_r.jpg)

上面是我从网上找到的一张 JDK 体系图，这张图上显而易见，三者的关系很清晰。

JDK(Java Development Kit) 就是 java 开发工具，从图中可以看到，JDK 包含 JRE，Java 基础类库和 Java 工具，这是给开发者使用的产品。

JRE(Java Runtime Environment)Java 运行时环境，在运行 java 程序代码时，提供 Java 运行的所有环境集合，包括核心类库，JVM 实现等，在安装 jdk 时，安装目录下会有一个 jre 的目录，jre 目录下有一个 lib 目录和一个 bin 目录，lib 目录中存放的是 Java 运行是所需要的核心类库，bin 中的就是 JVM 的实现。

JVM 是包含在 Jre 中的 bin 目录下，他里面为我们封装了一些命令，比如：javac，是 Java 语言运行的虚拟机，提供了编译器，执行器等等。市面上最常用的 JVM 是 HotSpot。

Java 号称一款跨平台的语言，一次编译到处运行，原因就是 JVM 在软件层面帮我们封装了不同操作系统的指令，可以让 Java 开发者无需关注底层操作系统实现。

换句话说，你只需要按照 Java 的语法开发你的程序就行了，至于怎么在操作系统上运行起来，怎么与操作系统交互等等交给 JVM 去做就行了。

当下载 jdk 时，如果你是 windows 系统就要下载 windows 版本的 jdk，如果你是 linux 系统，你就要下载 linux 版本的 jdk，不同的操作系统有不同的 jdk 实现。

![](https://pic1.zhimg.com/v2-b49d3cbe277ec6de841d74b07c5805e8_r.jpg)

本篇主要研究一下 JVM 的内存模型。

二、JVM 内存模型
----------

在此之前先来看看两个概念：JVM 内存模型和 Java 内存模型 (JMM)。

不知道有没有把这两个概念搞混的，甚至认为是一个东西的？

注意：这两个不是一个东西，不是一个概念，先来说一下这两个东西的区别。

Java 内存模型 (JMM) 是什么？答：这是一个规范。

规范了什么？

大家都知道 Java 中真正做事的是什么？线程，Java 也是支持多线程的。那如果在多线程环境中，对共享变量访问，多个线程是如何操作的？随意访问 随意操作？这岂不是很混乱，程序还怎么执行？JMM 就是用来规范这玩意的，让大家 (多线程) 按照我的规范来操作。

JMM 规范了多线程环境中，对共享变量访问的方式。通过这个规范解决多线程的可见性，原子性，访问共享变量的冲突问题。

怎么规范的呢？

有一块区域，叫做主存，把共享变量放到主存中，这块区域大家共享。

线程 A 需要访问共享变量，好，给线程 A 开辟一块独立的空间，把共享变量拷贝一份到这个独立空间中（这块空间叫做工作空间），随你操作。

此时线程 B 也需要访问共享变量，好，也给线程 B 开辟一块独立的空间，把共享变量也拷贝一份到这个独立空间中，随你操作。

以上就是本人对 JMM 的理解。

下面就是本篇的重点了，JVM 内存模型。（本篇以 hotspot 为主）

JVM 内存模型主要是指运行时数据区。JVM 主要包含三块：运行时数据区，类装载子系统，字节码执行引擎。

![](https://pic3.zhimg.com/v2-285535960c024b9c8e16b98f695211ae_r.jpg)

运行时数据区又分为堆，方法区，线程栈 (虚拟机栈)，本地方法栈，程序计数器。

**方法区：**主要用来存放类元信息，类的方法代码，变量名，方法名，访问权限，返回值，静态变量，运行时常量池等等，注意 class 对象不是存在方法区的，是存在堆区的，class 的对象是类加载的最终结果。

方法区是所有线程共享区域

有没有人会有这样一个疑问，方法区和永久代 (元空间) 有什么区别？

答：是有区别的，这是两个不同层次的东西，具体说明，我准备在下一篇文章中做个详细的说明，这里不再赘述了。

**堆：**我们创建的对象，一般都是在堆中分配内存的，在 jdk1.7 以后字符串常量池也挪到了堆中。堆分为年轻代和老年代，年轻代又分为 eden，s1，s2 三个区域 正常情况是 8:1:1 的比例 (这个比例是可以通过参数修改的)。为什么把堆划分为这几个区域？这是为例方便对堆内存的管理，对垃圾回收器分代回收的一个优化策略。后面文章会详细学习一下垃圾回收器方面的知识。堆是所有线程共享的一块区域。

**线程栈 (虚拟机栈):** 这是一块线程独享的内存区域，如果一个线程被创建了，那么就会给线程分配一块内存区域 (线程栈)，默认 512k，可以通过 - Xss 参数修改，线程每调用一个方法，就会在线程栈内存中开辟一个栈帧出来，栈帧中包含局部变量表 (存局部变量的)，操作数栈 (一些取值赋值操作的地方)，动态链接，方法出口等，每个方法都有自己的栈帧哦。

局部变量表：保存线程在执行方法时所有的局部变量，在局部变量表的第 0 位索引位置存放的是当前对象的实例引用，因此你可以在实例方法中通过 [http://this.xxx](http://this.xxx) 来访问本实例的变量或者方法等，其他位置就是存放方法中定义的局部变量。

这个局部变量与实例的成员变量还不一样，成员变量在对象的创建过程中是有两次赋值机会的，一次是在准备阶段给成员变量赋值零值，也就是初始值，第二次在初始化对象时，会为成员变量赋值程序员指定的值，而局部变量 ，因为编译器 javac 在编译的时候已经清楚的知晓局部变量的取值赋值顺序，这个顺序在运行期是不可以变的，因此为了防止程序员忘记赋值使用导致不可预估的错误，javac 要求局部变量必须赋值使用，否则无法通过编译，这是一种约束，为了避免出错的一种约束。

操作数栈：代码指令操作的一个区域，例如一个方法：

```
public static void main(String[] args) {
  Math math = new Math();
  math.compute();
}

public int compute() {
        int a = 2;
        int b = 5;
        return (a + b) * 10;
} 
===>编译后的代码
  public int compute();
    Code:
       0: iconst_2            将int类型2推送至栈顶
       1: istore_1            将int类型数值存入第二个(索引为1)本地变量    
       2: iconst_5            将int类型5推送至栈顶
       3: istore_2            将int类型数值存入第三个(索引为2)本地变量   
       4: iload_1             将第二个int类型本地变量数值推送至栈顶
       5: iload_2             将第三个int类型本地变量数值推送至栈顶
       6: iadd                将栈顶两个int类型的数值相加并将结果压入栈顶
       7: bipush        10    将单字节的常量值(-128~127)推送至栈顶
       9: imul                将栈顶两个int类型的数值相乘并将结果压入栈顶
      10: ireturn              从当前方法中返回int类型的数值

 

public static void main(java.lang.String[]);
    Code:
       0: new           #2                       // class main/jvm/classLoad/Math
       3: dup
       4: invokespecial #3                  // Method "<init>":()V
       7: astore_1
       8: aload_1
       9: invokevirtual #4                  // Method compute:()I
      12: pop
      13: return

```

以上 compute 方法中的操作指令都是在操作数栈中完成的，这里的每一步操作都会调用底层操作系统的指令，从而调用 cpu 完成的。

动态链接: 在代码执行过程中，所有的代码指令，方法名，变量等都存在方法区，线程栈中保存的是其对应的动态链接，通过动态链接可以找到对应的方法和变量等。

方法出口：方法返回退出就等于出栈，方法的退出方式一般有两种：1. 遇到返回指令 例如 return。2. 遇到异常退出

遇到返回指令是一种程序的正常退出，退出后要恢复上一个方法栈帧执行的位置，以便继续执行，这个一般是通过程序计数器实现的。

异常退出，需要交由异常处理器处理。

**本地方法栈：**分配给本地 native 方法一块内存区域，线程独享。

**程序计数器:** 记录程序执行到的指令行号，jvm 每执行完一行指令后，都需要给程序计数器 + 1 来获取下一行的指令代码，当线程上线文切换时，获取执行权的线程要方法开始执行吗？当然不用，因为程序计数器已经帮忙记录被挂起前你执行到了那个位置，线程恢复后重新从这个位置执行就可以了。不仅如此分支，跳转，循环呀等一些基础功能都需要依赖程序计数器。

这是 jvm 中一块绝对不会发生内存溢出的内存区域。

它的功能决定了，程序计数器只能是线程私有的。

在 JVM 调优的学习过程中，我们会重点关注方法区，堆，栈三块区域。

**而 JVM 调优主从三个方面考虑**：

1. 根据业务场景，分配不同的堆内存大小，从而尽量尽量减少 Full GC 的发生

2. 依据业务场景，选择合适的垃圾收集器，从而减少在垃圾回收过程中 STW（用户线程停顿）的时间

3.cpu 飙高，内存飙高等问题排查解决。

看一下内存区域的配置参数

![](https://pic3.zhimg.com/v2-bd53f00d2a7f11199a4d2e08fb9d5d4a_r.jpg)

**-XX：MaxMetaspaceSize**： 设置元空间最大值， 默认是 - 1， 即不限制， 或者说只受限于本地内存大小。

**-XX：MetaspaceSize**： 指定元空间的初始空间大小， 以字节为单位，默认是 21M，达到该值就会触发 full gc 进行类型卸载， 同时收集器会对该值进行调整： 如果释放了大量的空间， 就适当降低该值； 如果释放了很少的空间， 那么在不超过 - XX:MaxMetaspaceSize（如果设置了的话） 的情况下， 适当提高该值。 由于调整元空间的大小需要 Full GC，这是非常昂贵的操作，如果应用在启动的时候发生大量 Full GC，通常都是由于永久代或元空间发生 了大小调整，基于这种情况，一般建议在 JVM 参数中将 MetaspaceSize 和 MaxMetaspaceSize 设置成一样的值，并设置得比初始值要大， 对于 8G 物理内存的机器来说，一般我会将这两个值都设置为 256M。

拓展:

```
/**
 * 栈内存测试
 */
public class StackOverflowError {

    static int count=0;

    //JVM设置-Xss160k(默认512k) 栈内存配置为160k
    public static void redo(){
        count++;


        /**
         * 每调用一次方法都需要在线程栈内存空间中开辟一块内存空间给方法使用，
         * -Xss配置的越小，线程栈内存越小，分配给方法的内存快越小  则递归调用的次数越小
         *
         * 默认线程栈配置为1M  可以调用2万多次
         *
         * 配置160k  只可以调用800多次
         */
        redo();//递归执行
    }

    public static void main(String[] args) {
        try {
            redo();
        }catch (Throwable e){
            System.out.println("发生异常:"+e.toString());
            System.out.println(count);
        }
    }
}

结果：
-Xss1m时：
发生异常:java.lang.StackOverflowError
22436
-Xss160k时
发生异常:java.lang.StackOverflowError
2718

```

结论:-Xss 配置越大 栈空间就越大，栈帧分配的就越多, 当栈空间用完了，就栈溢出了，反之 -Xss 越小，每个线程分配的栈空间就会越小，可以开启的线程数就会越多。

当有人问你的程序可以开启多少个线程时，这块也可以提出来说说，毕竟也是影响的一个点。