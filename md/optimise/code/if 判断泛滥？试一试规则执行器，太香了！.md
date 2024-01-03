> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HQKhziO6W8sW7dEslujJZQ)

```
胖虎和朋友原创的视频教程有兴趣的可以看看：




（文末附课程大纲）




👏2024 最新，Java成神之路，架构视频（点击查看）


😉超全技术栈的Java入门+进阶+实战！（点击查看）

```

只有体验过几百行 if else 折磨的人，才会对本篇产生共鸣！

业务场景
----

近日在公司领到一个小需求，需要对之前已有的试用用户申请规则进行拓展。我们的场景大概如下所示:

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

*   咱们的的主要流程主要是基于 and 或者 or 的关系。
    
*   如果有一个不匹配的话，其实咱们后续的流程是不用执行的，就是需要具备一个短路的功能。
    
*   对于目前的现状来说，我如果在原有的基础上来改，只要稍微注意一下解决需求不是很大的问题，但是说后面可维护性非常差。
    

后面进过权衡过后，我还是决定将这个部分进行重构一下。

规则执行器
-----

针对这个需求，我首先梳理了一下咱们规则执行器大概的设计， 然后我设计了一个 V1 版本和大家一起分享一下，如果大家也有这样的 case 可以给我分享留言，下面部分主要是设计和实现的流程和 code.

### 规则执行器的设计

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/ibnDyTicARr35ZUukYzJn4e6mbAISXkicgib9We7pzdT4p5AJ9KXSbmzDbibsYfU7gSicxqMbXC80NjJgOfVzlfCBdFA/640?wx_fmt=jpeg&from=appmsg&random=0.5951284136116732&wxfrom=5&wx_lazy=1&wx_co=1)

图片

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
2024最新架构课程，对标培训机构

👉点击查看：Java成神之路-进阶架构视频！

```

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

总结
--

规则执行器的优点和缺点

### 优点：

比较简单，每个规则可以独立，将规则，数据，执行器拆分出来，调用方比较规整；

我在 Rule 模板类中定义 convert 方法做参数的转换这样可以能够，为特定 rule 需要的场景数据提供拓展。

### 缺点：

上下 rule 有数据依赖性，如果直接修改公共传输对象 dto 这样设计不是很合理，建议提前构建数据。

```
胖虎联合两位大佬朋友，一位是知名培训机构讲师和科大讯飞架构，联合打造了《Java架构师成长之路》的视频教程。完全对标外面2万左右的培训课程。

除了基本的视频教程之外，还提供了超详细的课堂笔记，以及源码等资料包..




课程阶段：

Java核心 提升阅读源码的内功心法
深入讲解企业开发必备技术栈，夯实基础，为跳槽加薪增加筹码

分布式架构设计方法论。为学习分布式微服务做铺垫
学习NetFilx公司产品，如Eureka、Hystrix、Zuul、Feign、Ribbon等，以及学习Spring Cloud Alibabba体系
微服务架构下的性能优化
中间件源码剖析
元原生以及虚拟化技术
从0开始，项目实战 SpringCloud Alibaba电商项目


点击下方超链接查看详情（或者点击文末阅读原文）：

(点击查看)  2024年，最新Java架构师成长之路 视频教程！

以下是课程大纲，大家可以双击打开原图查看

```