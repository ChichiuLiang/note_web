> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/v67yQYSMPqpnAofdBbPZ6A)

```
往期热门文章：
1、为什么不建议在 Docker 中跑 MySQL？
2、推荐一款功能强大的商城系统：Javashop，适合二次开发，100%开源！
3、ObjectMapper，别再像个二货一样一直new了！
4、我用Spring Boot随手封装了一个万能的 Excel 导出工具，传什么都能导出！
5、MyBatis的10种精妙用法，真是妙啊！
5、同事写的让我内心五味杂陈的代码


前言


```

有个老哥昨天被面试官欺负了，但是是被这个问题（标题）欺负的？

其实是个比较基础的问题，只要有了解过，叙述是非常简单OK的。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

**正文**

  

首先 ， 话说在前头，

没有什么 在里面 好 和在外面好  或者 不好的 一说。

本篇文章内容：

1.  使用场景
    
2.  性能分析
    
3.  个人看法
    

###   

### **1. 使用场景**

  

为什么要把 使用场景 摆在第一个 ？

因为本身try catch 放在 for循环 外面 和里面 ，如果出现异常，产生的效果是不一样的。

怎么用，就需要看好业务场景，去使用了。

##### **① try  catch  在 for 循环 外面**

代码示例 ：

```


public static void tryOutside() {    
    try {    
        for (int count = 1; count <= 5; count++) {    
            if (count == 3) {    
                //故意制造一下异常    
                int num = 1 / 0;    
            } else {    
                System.out.println("count:" + count + " 业务正常执行");    
            }    
        }    
    } catch (Exception e) {    
        System.out.println("try catch  在for 外面的情形， 出现了异常，for循环显然被中断");    
    }    
}  


```

结果：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

效果结论：

> try  catch  在 for 循环 外面 的时候， 如果 for循环过程中出现了异常， 那么for循环会终止。

##### **② try  catch  在 for 循环 里面**

代码示例 ：

```


public static void tryInside() {    
    
    for (int count = 1; count <= 5; count++) {    
        try {    
            if (count == 3) {    
                //故意制造一下异常    
                int num = 1 / 0;    
            } else {    
                System.out.println("count:" + count + " 业务正常执行");    
            }    
        } catch (Exception e) {    
            System.out.println("try catch  在for 里面的情形， 出现了异常，for循环显然继续执行");    
        }    
    }    
}    



```

结果：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

效果结论：

> try  catch  在 for 循环 里面 的时候， 如果 for循环过程中出现了异常，异常被catch抓掉，不影响for循环 继续执行。

ps：

在面试的时候，如果真的连上面这个在外面在里面使用效果都没说对，那，真的会去等通知了。

但是 之前不会的看官，看完这一篇， 肯定会了。

###   

### **2. 性能**

###   

### 时间上， 其实算是无差别。

内存上， 如果没出异常，其实也是无差别。

但是如果出现了异常， 那就要注意了。

注意点是什么 ？ 看代码：

我们简单用

```


Runtime runtime = Runtime.getRuntime();    
long memory = runtime.freeMemory();    



```

来统计一下内存消耗情况：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

结论：

> 也就是说， try catch 放在 for 循环里面 ，因为出现异常不会终止 for循环。所以如果真的存在大批量业务处理全是异常，有那么一定的内存消耗情况。

如果说代码没出错的话， try catch 在 for 里面 和 外面 ，都是几乎没区别的。

为啥， 因为 异常try catch 其实一早编译完就标记了 如果从哪儿（from）出现异常，会直接去到（to）的那行代码去。

*   `Exception table` : 当前函数程序代码编译涉及到的异常；
    
*   `type` ：异常类型；
    
*   `target`：表示异常的处理起始位；
    
*   `from`：表示 try-catch 的开始地址；
    
*   `to`：表示 try-catch 的结束地址；
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

所以如果不考虑业出错，是否终止循环， 这个try catch 放里放外没啥区别。

**3. 个人看法**

其实就是看业务。我需要出现异常就终止循环的，就放外头；

不需要终止循环，就搞里头。

但是要注意一点就是，别在for循环里面去 查库调用第三方啥的，这些操作，如果必要，需要慎重考虑了。（别什么都搞里头！！！）

```


转自：blog.csdn.net/qq_35387940/article/details/128406626

  

```


**往期热门文章：**

#### 

1、[再见 Guava ！Spring Boot 拥抱本地缓存之王 Caffeine](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508256&idx=1&sn=f8904dc37929673c33827206e28e38b7&chksm=e9c65891deb1d1875b4f69f018b09bf14d80fb470e6369a799066d904dcbf22a36a4498d28b4&scene=21#wechat_redirect)

2、[如何搭建一台永久运行的个人服务器？](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508250&idx=1&sn=22d6019706c1d6c955e992287fd01ec6&chksm=e9c658abdeb1d1bddee3bbfe017d27b73460e91f2062ef9ed402f4b6c706427fe70242f64735&scene=21#wechat_redirect)

3、[灵魂画手图解 Spring 循环依赖](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508187&idx=1&sn=f5a08f3f82d46f0c6c2f6fe3d76b2cd4&chksm=e9c6596adeb1d07cb657150de0c31e95638dc780abd59a212227950d25fd6abfeadf12b3b47a&scene=21#wechat_redirect)

4、[让同事高血压的8个Bug操作集锦](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508179&idx=1&sn=80c0deebcc3000f067c4352cdc9486b4&chksm=e9c65962deb1d0743567253896e74bbc10d2cb4b2c079efb877badfa8baa8b14cbd9e2fbab2a&scene=21#wechat_redirect)

5、[别再乱用了，Java 21 将弃用、删除这些功能！](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508105&idx=1&sn=4313f0f49f53176de0e1ca264d211f9b&chksm=e9c65938deb1d02e8497f642ae808fd4f5351a7b690149bf9f3f8c2b6a09a9a3eff3a154e54f&scene=21#wechat_redirect)

6[、IDEA 28 个小技巧 ，效率嘎嘎高...](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508041&idx=1&sn=873e44626f0cab85514a0ccfec036ae7&chksm=e9c659f8deb1d0ee7368ee17e51d9054e82f2f7916ffb9e3534e140a2489be76503d22fa7911&scene=21#wechat_redirect)

7、[if 判断泛滥？试一试规则执行器，太香了！](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508037&idx=1&sn=a32b3bcf0a6004cd659c7735c90b212f&chksm=e9c659f4deb1d0e2074fa2d106720b463f244c9b3a557e64a2f51f85d49472a4a36216ef912d&scene=21#wechat_redirect)

8、[JetBrains Fleet公测，下一代轻量级全能IDE](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508021&idx=1&sn=d6e74d732862c2cbcf3a21cb5131a9d8&chksm=e9c65984deb1d092cbdb777dbe6efdcdb9a494ff90f3637b852797886933558f770739d359ec&scene=21#wechat_redirect)

9、[如果按代码量算工资，也许应该这样写！](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508013&idx=1&sn=1028f0fab7017c59afa9dc60eacb59f1&chksm=e9c6599cdeb1d08a25645e8253eb0dcebcf2c891f2f2c84aee68ce137a05fb3c43683fc99339&scene=21#wechat_redirect)

10、[听我一句劝，业务代码别用多线程！](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507999&idx=1&sn=8721274dcbb3ed28ee5d592845bf3f5e&chksm=e9c659aedeb1d0b8e924d9492ee98aebb063e56e81776cbfb43538274e37a80a1bbd734c5105&scene=21#wechat_redirect)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)


```


```