> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/eV_9-aUb8VVk007YvEQ0xw)

相信我们对策略模式都有耳闻，但是可能不知道它在项目中具体能有什么作用，我们需要在什么场景下才能去尽可能得去使用策略模式。

这里我简单的列出一个我之前在公司做的一个需求：跟第三方 oa 系统对接接口，对方需要回调我们当前系统，但是是不同的业务接口回调，我们系统可以根据一个字段来区分需要走哪个业务分支，可能初级程序员刚接触这个需求的时候想法是，多个接口回调，那就写多个接口罢了，强调接口隔离；

或者直接一个接口也行，if...else if 也很不错，这里呢，为了彰显我们开发人员的逼格，我们可以基于一个接口外加设计模式之策略模式 + 简单工厂模式。

**下面是一个简单的实现 demo：**

首先是我们定义一个接口即起路由作用，我们具体的不同业务实现类来实现这个接口就可以；

```
public interface CalculationStrategy {
    /**
     * 策略接口
     */
    int operate(int num1, int num2);
}
@Component("add")
class AddCalculationStrategyImpl implements CalculationStrategy {
    @Override
    public int operate(int num1, int num2) {
        return num1 + num2;
    }
}
@Component("Division")
class DivisionStrategyImpl implements CalculationStrategy {
    @Override
    public int operate(int num1, int num2) {
        return num1 / num2;
    }
}
@Component("multiple")
class MultiplicationStrategyImpl implements CalculationStrategy {
    @Override
    public int operate(int num1, int num2) {
        return num1 * num2;
    }
}
@Component("subtract")
class SubtractionStrategyImpl implements CalculationStrategy {

    @Override
    public int operate(int num1, int num2) {
        return num1 - num2;
    }
}

/**
 * 如果Component注解中不写标识会默认加载驼峰类名：testStrategyImpl
 */
@Component
class TestStrategyImpl implements CalculationStrategy {

    @Override
    public int operate(int num1, int num2) {
        return num1 - num2;
    }
}


```

不同的业务分支我就使用加减乘除来代替，也能起到同样的效果。

第二步便是我们的策略上下文，我将其理解为策略工厂，这也是最核心的一个类；

这里我们项目启动的时候，第一步是初始化所有加了`@component`等类，我们的策略工厂的构造函数中有获取所有实现了路由规则的实现类名称，第二步便是将获取到的实现类名称放入到我们初始化的一个空的 map 中

```
@Component
public class CalculationFactory{
    /**
     *  把策略角色（类型）key,和参数value放到Map中
     *  key就是beanName(具体策略实现类中@Component的名字)，value就是接口（具体的实现类）
     *  Maps是guava下的封装类型，实则是静态的创建了一个HashMap的对象，Maps可以根据key去获取value对象
     */
    public final Map<String, CalculationStrategy> calculationStrategyMap = Maps.newHashMapWithExpectedSize(4);

    /**
     * 利用构造函数在项目启动的时候将策略实现类注册到 map里
     * @param strategyMap
     */
    public CalculationFactory(Map<String, CalculationStrategy> strategyMap) {
        this.calculationStrategyMap.clear();
        this.calculationStrategyMap.putAll(strategyMap);
    }


    //可以使用@Getter注解代替，这样写方便读者理解在Service层调用Context执行策略
    public Map<String, CalculationStrategy> getCalculationStrategyMap() {
        return calculationStrategyMap;
    }
}


```

第三步便是我们的路由接口实现，这一步便是具体路由的规则判断了 这里有一步我们需要对代码进行健壮性判断，以防`map.get()`结果为空导致系统报错，这里大家可以根据业务情况自行去处理。

```
@Service
public class CalculationService {

    @Autowired
    private CalculationFactory calculationFactory;

    public int operateByStrategy(String strategy, int num1, int num2) {
        // 获取入参，根据不同的参数类型去执行不同的策略，Context的get方法是在这个地方用到的，operate方法就是一开始定义的策略接口
        //calculationFactory.getCalculationStrategyMap().get(strategy)这里可能会出现空，所以要做一个容错处理
        return calculationFactory.getCalculationStrategyMap().get(strategy).operate(num1, num2);
    }
}


```

最后便是我们的测试接口了

```
@RestController
@RequestMapping("/strategy")
public class TestStrategyController {
    @Autowired
    private CalculationService calculationService;

    @GetMapping("/test/{operation}/{num1}/{num2}")
    public int testCalculation(@PathVariable String operation,@PathVariable  int num1, @PathVariable int num2) {
        // 省略参数判空
        return calculationService.operateByStrategy(operation, num1, num2);
    }
}


```

结果如图所示，大家可自行下去测试，粘贴即可运行~

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbuf9OPJd5NxiatLbIJ0k0QP336FvdtfArW2BgxRgfjChZ0HHENsyMia3eT07Cd0q9jAvQ8aG7zgz3K5w/640?wx_fmt=png&from=appmsg)

这样写的好处就是，如果系统还需要扩展其他业务类型的分支处理，那我们只需要将业务处理的实现类实现我们的路由接口，将这个实现类注册进去即可，其他地方都不用改，只需关注我们这个自身的业务分支的逻辑处理。方便了我们系统的后续扩展。

我这也只是一个简单的 demo 案例，我目前自身也是一个初窥门径的程序员，希望能得到大家更多更优秀的思想来提升自己。喜欢的话点个赞再走~

> _感谢阅读，希望对你有所帮助 :)_ _来源：_
> 
> _juejin.cn/post/7304594468158537768_

```
程序汪资料链接

程序汪接私活项目目录，2023年总结


Java项目分享  最新整理全集，找项目不累啦 07版

欢迎添加程序汪微信 itwang007  进粉丝群

```