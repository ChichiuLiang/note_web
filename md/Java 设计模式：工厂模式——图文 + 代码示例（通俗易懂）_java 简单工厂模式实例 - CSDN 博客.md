> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/A_hxy/article/details/105758657)

工厂模式：
-----

每一种设计模式都有它要解决的问题：  
工厂模式最主要解决的问题就是创建者和调用者的耦合，那么代码层面其实就是取消对 new 的使用。

##### 工厂模式有三种：

**1. 简单工厂模式  
2. 工厂方法模式  
3. 抽象方法模式**

先来看看，简单工厂模式——也叫静态工厂模式，这里举个例子：你要去买一台手机，你不用关心手机是怎么生产出来的，里面的零件具体又是怎么制造的，这些通通都交给工厂去处理，你尽管去买手机就好了。

**简单工厂模式图**  
![](https://img-blog.csdnimg.cn/20200426164316984.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FfaHh5,size_16,color_FFFFFF,t_70)

### 代码示例：

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

```
public class PhoneFactory{
    public static Phone getPhone(String phone){
        if("小米".equals(phone)){
           return new Xiaomi();
        }else if ("魅族".equals(phone)){
          return new Meizu();
        }else {
            return null;
        }
    }
}
```

Customer 消费者类

```
public class Customer {
    public static void main(String[] args) {
        PhoneFactory.getPhone("Xiaomi").getBrand();
        PhoneFactory.getPhone("Meizu").getBrand();
    }
}
```

### 运行效果图：

![](https://img-blog.csdnimg.cn/20200426190209484.png)  
**那么简单工厂模式会遇到什么问题呢？**  
随着手机品牌增多，工厂生产也需要对应的增加，工厂内部就需要不断的调整。  
从代码层面——对内部代码需要增加 (也就是需要修改内部代码：那么就会违反 OOP 原则—开闭原则：一个软件实体应当对扩展开放，对修改关闭。那怎么解决呢？看看下面

**工厂方法模式**  
很简单，先看还是举手机例子，当新的手机品牌出现，不是放在同一个工厂生产，而是自己拥有独立工厂生产。那么就解决了上面静态工厂模式违反关闭原则的问题。

**工厂方法模式图**  
![](https://img-blog.csdnimg.cn/20200426164343845.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FfaHh5,size_16,color_FFFFFF,t_70)

### 代码示例

Phone 接口

```
public interface Phone {
    void getBrand();
}
```

PhoneFactory 接口

```
public interface PhoneFactory {
    Phone getPhone();
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

XiaomiFactory 工厂类

```
public class XiaomiFactory implements PhoneFactory {
    @Override
    public Phone getPhone() {
        return new Xiaomi();
    }
}
```

Meizu 品牌类

```
public class XiaomiFactory implements PhoneFactory {
    @Override
    public Phone getPhone() {
        return new Xiaomi();
    }
}
```

MeizuFactory 工厂类

```
public class MeizuFactory implements PhoneFactory{
    @Override
    public Phone getPhone() {
        return new Meizu();
    }
}
```

消费者类

```
public class Customer {
    public static void main(String[] args) {
        Phone xiaomi = new XiaomiFactory().getPhone();
        Phone meizu = new MeizuFactory().getPhone();
        xiaomi.getBrand();
        meizu.getBrand();
    }
}
```

### 运行效果图：

![](https://img-blog.csdnimg.cn/20200426215113527.png)  
**这里需要注意  
工厂方法模式解决简单工厂模式是需要付出代价的！**  
看到上图工厂方法模式图里新增用虚线画的 Huawei 品牌，每新增一个品牌就需要增加，对应新的工厂，会发现需要花费很大的成本，现在才三个新的品牌，那么等到十个、一百个的时候就会变得更加的复杂和难以维护。

工厂方法模式这里还有一个点：看上面工厂方法的消费者代码示例里用到 new 是我们耦合的最大元凶，可以看看这篇文章里写的，利用反射对工厂模式的解耦操作：  
[Java 高级特性——反射 Reflection 获取类的几种方式和对工厂模式解耦操作（代码示例）](https://blog.csdn.net/A_hxy/article/details/105992426)

### 以上总结：

> **第一个静态工厂模式：在实际去开发中会发现比较常用，尽管上面讲了不符合（面向对象）OOP 原则。**  
> **第二个工厂方法模式：不修改工厂类的前提，也就是说不修改已有类，实现对扩展是开发，对修改关闭。**  
> **还有三个工厂模式：抽象工厂模式，下次单独拿出来讲。**

**最后：看到这里是不是有一连串的疑问：怎么看用哪一个都有问题，那到底要用哪一个最好？  
其实都是同一个问题，我们会发现世间万物，并没有十全十美的，事物都会有两面性，有利就会有弊，选择适合自己项目的，就是最好的！**