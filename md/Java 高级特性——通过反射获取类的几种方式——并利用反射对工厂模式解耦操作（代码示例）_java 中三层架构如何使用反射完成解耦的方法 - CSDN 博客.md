> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/A_hxy/article/details/105992426)

**第一种方式：getClass  
getClass 获取类首先需要有实例化对象**

```
Date date = new Date();
 Class<?> cls = date.getClass();
 System.out.println(cls.getName());
```

**第二种方式：.class  
.class 获取类不需要有实例化对象，但是需要完整类名**

```
Class<?> cls2 = java.util.Date.class;
  System.out.println(cls2.getName());
```

**第三种方式：.forName()  
调用. forName() 需要有完整的类名**

```
Class<?> cls3 = Class.forName("java.util.Date");
System.out.println(cls3.getCanonicalName());
```

运行结果：  
![](https://img-blog.csdnimg.cn/20200508172932378.png)  
学了上面的三种主要获取类方式，那么来实践玩玩吧

把之前写的工厂方法模式拿出来看看存在的问题，还没看的或者有兴趣的可以看看，当然不看也行，因为我这里会重新敲一遍工厂模式（为什么要重新敲呢？相当于一个简单的重构吧，让代码更健壮一点点，哈哈…）  
好了，链接放就这，然后来看看下面的问题

> [**Java 设计模式：工厂模式——图文 + 代码示例（通俗易懂）**](https://blog.csdn.net/A_hxy/article/details/105758657)

简单工厂模式里的工厂类（也是静态工厂模式）：每次出新的手机牌子就需要修改工厂类违反开闭原则  
![](https://img-blog.csdnimg.cn/20200508210159113.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FfaHh5,size_16,color_FFFFFF,t_70)  
工厂方法模式呢？看看消费者类获取手机需要 new，我们都知道解决耦合的关键是取消 new，因为 new 是耦合的最大元凶，所以我们接下来用[反射](https://so.csdn.net/so/search?q=%E5%8F%8D%E5%B0%84&spm=1001.2101.3001.7020)来帮助我们解决设计上的缺陷。  
![](https://img-blog.csdnimg.cn/20200508183058721.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FfaHh5,size_16,color_FFFFFF,t_70)  
来，利用反射对工厂模式[解耦](https://so.csdn.net/so/search?q=%E8%A7%A3%E8%80%A6&spm=1001.2101.3001.7020)操作！

Phone 接口

```
public interface Phone {
    void getBrand();
}
```

Meizu 品牌类

```
public class Meizu implements Phone {
    @Override
    public void getBrand() {
        System.out.println("魅族");
    }
}
```

Xiaomi 品牌类

```
public class Xiaomi implements Phone {
    @Override
    public void getBrand() {
        System.out.println("小米");
    }
}
```

PhoneFactory 工厂类  
利用最开始讲的反射三种获取对象之一的 Class.forName(传入完整的包类名) 找到它的类，然后使用 newInstance() 方法反射的对象实例化

```
public class PhoneFactory {
    public static Phone getInstance(String orgin) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        Class<?> cls  = Class.forName(orgin);
        Phone brand = (Phone)cls.newInstance();
        return brand;
    }
}
```

Customer 消费者类，直白一点说就是可以传值了，而无需用 new 写死

```
public class Customer {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException, ClassNotFoundException {
        PhoneFactory.getInstance("Factory.Xiaomi").getBrand();
        PhoneFactory.getInstance("Factory.Meizu").getBrand();
    }
}
```

运行结果：  
![](https://img-blog.csdnimg.cn/20200508231053368.png)  
可以看到上面代码可以说和静态工厂模式一样，只不过工厂模式利用了反射机制，不再只是用 new 关键字而是用反射机制来。

**最后**

> **实践一波后会发现，对象实例化不再是单独用 new 来，反射也是可以完成的（当然反射还是需要考虑到性能的，这是后话，到时可以一起来探究一下原理），当然这不表示 new 会完全被取代，在一些开发中也会经常用到 new 这个关键字**