> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247508037&idx=1&sn=a32b3bcf0a6004cd659c7735c90b212f&chksm=e9c659f4deb1d0e2074fa2d106720b463f244c9b3a557e64a2f51f85d49472a4a36216ef912d&scene=21#wechat_redirect)

```
往期热门文章：
1、JetBrains Fleet公测，下一代轻量级全能IDE
2、如果按代码量算工资，也许应该这样写！
3、听我一句劝，业务代码别用多线程！
4、Java Random可破解，随机数不再随机，更不安全
5、为什么 idea 建议去掉 StringBuilder，使用“+”拼接字符串

```

近日公司有一个小需求，需要对之前已有的试用用户申请规则进行拓展。我们的场景大概如下所示:

```
if (是否海外用户) {  
 return false;  
}  
  
if (刷单用户) {  
  return false;  
}  
  
if (未付费用户 && 不再服务时段) {  
  return false  
}  
  
if (转介绍用户 || 付费用户 || 内推用户) {  
  return true;  
}  

```

按照上述的条件我们可以得出的结论是：

1.  咱们的的主要流程主要是基于`and` 或者`or` 的关系。
    
2.  如果有一个不匹配的话，其实咱们后续的流程是不用执行的，就是需要具备一个`短路`的功能。
    
3.  对于目前的现状来说，我如果在原有的基础上来该，只要稍微注意一下解决需求不是很大的问题，但是说后面可维护性非常差。
    

后面进过权衡过后，我还是决定将这个部分进行重构一下。

2 规则执行器
-------

针对这个需求，我首先梳理了一下咱们规则执行器大概的设计， 然后我设计了一个V1版本和大家一起分享一下，如果大家也有这样的 case 可以给我分享留言，下面部分主要是设计和实现的流程和 code。

### 规则执行器的设计

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### 对于规则的抽象并实现规则

```
// 业务数据  
@Data  
public class RuleDto {  
  private String address;  
 private int age;  
}  
  
// 规则抽象  
public interface BaseRule {  
  
    boolean execute(RuleDto dto);  
}  
  
// 规则模板  
public abstract class AbstractRule implements BaseRule {  
  
    protected <T> T convert(RuleDto dto) {  
        return (T) dto;  
    }  
  
    @Override  
    public boolean execute(RuleDto dto) {  
        return executeRule(convert(dto));  
    }  
    
    protected <T> boolean executeRule(T t) {  
        return true;  
    }  
}  
  
// 具体规则- 例子1  
public class AddressRule extends AbstractRule {  
  
    @Override  
    public boolean execute(RuleDto dto) {  
        System.out.println("AddressRule invoke!");  
        if (dto.getAddress().startsWith(MATCH_ADDRESS_START)) {  
            return true;  
        }  
        return false;  
    }  
}  
  
// 具体规则- 例子2  
public class NationalityRule extends AbstractRule {  
  
    @Override  
    protected <T> T convert(RuleDto dto) {  
        NationalityRuleDto nationalityRuleDto = new NationalityRuleDto();  
        if (dto.getAddress().startsWith(MATCH_ADDRESS_START)) {  
            nationalityRuleDto.setNationality(MATCH_NATIONALITY_START);  
        }  
        return (T) nationalityRuleDto;  
    }  
  
  
    @Override  
    protected <T> boolean executeRule(T t) {  
        System.out.println("NationalityRule invoke!");  
        NationalityRuleDto nationalityRuleDto = (NationalityRuleDto) t;  
        if (nationalityRuleDto.getNationality().startsWith(MATCH_NATIONALITY_START)) {  
            return true;  
        }  
        return false;  
    }  
}  
  
// 常量定义  
public class RuleConstant {  
    public static final String MATCH_ADDRESS_START= "北京";  
    public static final String MATCH_NATIONALITY_START= "中国";  
}  

```

### 执行器构建

```
public class RuleService {  
  
    private Map<Integer, List<BaseRule>> hashMap = new HashMap<>();  
    private static final int AND = 1;  
    private static final int OR = 0;  
  
    public static RuleService create() {  
        return new RuleService();  
    }  
  
  
    public RuleService and(List<BaseRule> ruleList) {  
        hashMap.put(AND, ruleList);  
        return this;  
    }  
  
    public RuleService or(List<BaseRule> ruleList) {  
        hashMap.put(OR, ruleList);  
        return this;  
    }  
  
    public boolean execute(RuleDto dto) {  
        for (Map.Entry<Integer, List<BaseRule>> item : hashMap.entrySet()) {  
            List<BaseRule> ruleList = item.getValue();  
            switch (item.getKey()) {  
                case AND:  
                    // 如果是 and 关系，同步执行  
                    System.out.println("execute key = " + 1);  
                    if (!and(dto, ruleList)) {  
                        return false;  
                    }  
                    break;  
                case OR:  
                    // 如果是 or 关系，并行执行  
                    System.out.println("execute key = " + 0);  
                    if (!or(dto, ruleList)) {  
                        return false;  
                    }  
                    break;  
                default:  
                    break;  
            }  
        }  
        return true;  
    }  
  
    private boolean and(RuleDto dto, List<BaseRule> ruleList) {  
        for (BaseRule rule : ruleList) {  
            boolean execute = rule.execute(dto);  
            if (!execute) {  
                // and 关系匹配失败一次，返回 false  
                return false;  
            }  
        }  
        // and 关系全部匹配成功，返回 true  
        return true;  
    }  
  
    private boolean or(RuleDto dto, List<BaseRule> ruleList) {  
        for (BaseRule rule : ruleList) {  
            boolean execute = rule.execute(dto);  
            if (execute) {  
                // or 关系匹配到一个就返回 true  
                return true;  
            }  
        }  
        // or 关系一个都匹配不到就返回 false  
        return false;  
    }  
}  

```

### 执行器的调用

```
public class RuleServiceTest {  
  
    @org.junit.Test  
    public void execute() {  
        //规则执行器  
        //优点：比较简单，每个规则可以独立，将规则，数据，执行器拆分出来，调用方比较规整  
        //缺点：数据依赖公共传输对象 dto  
  
        //1. 定义规则  init rule  
        AgeRule ageRule = new AgeRule();  
        NameRule nameRule = new NameRule();  
        NationalityRule nationalityRule = new NationalityRule();  
        AddressRule addressRule = new AddressRule();  
        SubjectRule subjectRule = new SubjectRule();  
  
        //2. 构造需要的数据 create dto  
        RuleDto dto = new RuleDto();  
        dto.setAge(5);  
        dto.setName("张三");  
        dto.setAddress("北京");  
        dto.setSubject("数学");;  
  
        //3. 通过以链式调用构建和执行 rule execute  
        boolean ruleResult = RuleService  
                .create()  
                .and(Arrays.asList(nationalityRule, nameRule, addressRule))  
                .or(Arrays.asList(ageRule, subjectRule))  
                .execute(dto);  
        System.out.println("this student rule execute result :" + ruleResult);  
    }  
}  

```

3 总结
----

规则执行器的优点和缺点

*   优点：
    

1.  比较简单，每个规则可以独立，将规则，数据，执行器拆分出来，调用方比较规整；
    
2.  我在 Rule 模板类中定义 convert 方法做参数的转换这样可以能够，为特定 rule 需要的场景数据提供拓展。
    

*   缺点：上下 rule 有数据依赖性，如果直接修改公共传输对象 dto 这样设计不是很合理，建议提前构建数据。
    

> 来源：juejin.cn/post/6951764927958745124

  

```


**往期热门文章：**

#### 

1、[基于 Sentinel 实现历史监控数据回看](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507953&idx=1&sn=a85d90e1efe8a8bf12b694e304957233&chksm=e9c62640deb1af561e5dfd9952d4436137b127ec51e819dc0d9fba8135a145c15a6361c95f07&scene=21#wechat_redirect)

2、[为什么游戏公司不使用微服务架构](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507925&idx=1&sn=aed24d6c0cb96f50703321d2c293fff3&chksm=e9c62664deb1af72d44e772c0bc511acbd9cdca0fa864d3d2351875aad7de271286d403dfa86&scene=21#wechat_redirect)

3、[微信群聊功能，原来是这样设计的！](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507915&idx=1&sn=ae1021f38c2d2f7d6d5a6ad3b20120ea&chksm=e9c6267adeb1af6cb5a745cea5b8c2dc8fd3e91db974bd38e2e635121e58b0f53ce69a7f3a82&scene=21#wechat_redirect)

4、[工作 3 年的同事不懂 isEmpty 和 isBlank 的区别，我真是醉了。](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507909&idx=1&sn=61d65b909ed24e9e1865e3774510b1ee&chksm=e9c62674deb1af620013494dccdf9e1d892bacc349c98e1acfd9a934c43d920287f135cd0f1b&scene=21#wechat_redirect)

5、[只改了五行代码吞吐量提升了10多倍！爽！](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507904&idx=1&sn=3d2e45779ad4cdebf9f135bbd72b9265&chksm=e9c62671deb1af67d87f2bdbaa2f5fa2b898927b432eb5fef0997ff5913a90e11c8f59d4a719&scene=21#wechat_redirect)

6、[为什么很多程序员讨厌低代码](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507865&idx=1&sn=c6844481e342b5cdc411be7b5315561b&chksm=e9c62628deb1af3e5f03ef30e240d6352b6183dd4599f59a4cba0fe6c65f5d55a13076269618&scene=21#wechat_redirect)

7、[人肉运维100次后，终究还是翻车了！](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507854&idx=1&sn=a49977389c9ed9ce371ddcc53ba22b90&chksm=e9c6263fdeb1af297895aa620fced602183a785033fab48289dc10a0cfa867fb565c34ab38db&scene=21#wechat_redirect)

8、[公司用了 3 年多的多账号统一登录方案，万能通用，稳的一批！](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507847&idx=1&sn=e87809677be21141dac73fb192e68bf9&chksm=e9c62636deb1af20227276102716bd2c0d16e587fcd03c83751b8d435efb833badd796b13ec3&scene=21#wechat_redirect)

9、[99%的时间里使用的14个git命令](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507834&idx=1&sn=b4ca68a518bea33ab5f3e43e516545a4&chksm=e9c626cbdeb1afdd4137377f9837bf338b9d03d30e150cbe7492c9d3eab2e5662e25873e180e&scene=21#wechat_redirect)

10、[通过 Arthas Trace 命令将接口性能优化十倍](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247507828&idx=1&sn=805e3ff4dc49ed46e361b875746aab22&chksm=e9c626c5deb1afd39411c99bb4e0d91b6f69ba418028b956ada9fa8246d74f7ed49c5f8b8857&scene=21#wechat_redirect)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)


```