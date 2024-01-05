> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/O0GfCDSpqp59HBMslRITcw)

  

*   前言
    
*   启用缓存 @EnableCaching
    
*   自定义缓存管理器
    
*   @Cacheable
    
*   @CachePut
    
*   @CacheEvict
    
*   @Caching
    
*   @CacheConfig
    
*   Condition & Unless
    
*   清理全部缓存
    
*   SpEL 表达式
    
*   最佳实践
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZfe3SWqBd2B1GATxSVMl6b6knOHdgSM811KmDTPPibMuFQb7IaBxLjc126kImODT2U1QBbqQEwOH5lg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

前言
--

缓存可以通过将经常访问的数据存储在内存中，减少底层数据源如数据库的压力，从而有效提高系统的性能和稳定性。我想大家的项目中或多或少都有使用过，我们项目也不例外，但是最近在 review 公司的代码的时候写的很蠢且 low, 大致写法如下：

```
public User getById(String id) {
 User user = cache.getUser();
    if(user != null) {
        return user;
    }
    // 从数据库获取
    user = loadFromDB(id);
    cahce.put(id, user);
 return user;
}


```

其实 Spring Boot 提供了强大的缓存抽象，可以轻松地向您的应用程序添加缓存。本文就讲讲如何使用 Spring 提供的不同缓存注解实现缓存的最佳实践。

启用缓存 @EnableCaching
-------------------

现在大部分项目都是是 SpringBoot 项目，我们可以在启动类添加注解`@EnableCaching`来开启缓存功能。

```
@SpringBootApplication
@EnableCaching
public class SpringCacheApp {

    public static void main(String[] args) {
        SpringApplication.run(Cache.class, args);
    }
}


```

既然要能使用缓存，就需要有一个缓存管理器 Bean，默认情况下，`@EnableCaching` 将注册一个`ConcurrentMapCacheManager`的 Bean，不需要单独的 bean 声明。`ConcurrentMapCacheManage`r 将值存储在`ConcurrentHashMap`的实例中，这是缓存机制的最简单的线程安全实现。

自定义缓存管理器
--------

默认的缓存管理器并不能满足需求，因为她是存储在 jvm 内存中的，那么如何存储到 redis 中呢？这时候需要添加自定义的缓存管理器。

1.  添加依赖
    

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>


```

2.  配置 Redis 缓存管理器
    

```
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory();
    }

    @Bean
    public CacheManager cacheManager() {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
            .disableCachingNullValues()
            .serializeValuesWith(SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        RedisCacheManager redisCacheManager = RedisCacheManager.builder(redisConnectionFactory())
            .cacheDefaults(redisCacheConfiguration)
            .build();

        return redisCacheManager;
    }
}


```

现在有了缓存管理器以后，我们如何在业务层面操作缓存呢？

我们可以使用`@Cacheable`、`@CachePut` 或`@CacheEvict` 注解来操作缓存了。

@Cacheable
----------

该注解可以将方法运行的结果进行缓存，在缓存时效内再次调用该方法时不会调用方法本身，而是直接从缓存获取结果并返回给调用方。

![](https://mmbiz.qpic.cn/mmbiz_png/oH5VKTC5sLjKHsa2rSwMwVlNyzYP1yRVojZZom6USGz3wgRdrdphF3pD7SuCuiaZdgQ3bGOdibS5z7c2KPoIKeGA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

例子 1：缓存数据库查询的结果。

```
@Service
public class MyService {

    @Autowired
    private MyRepository repository;

    @Cacheable(value = "myCache", key = "#id")
    public MyEntity getEntityById(Long id) {
        return repository.findById(id).orElse(null);
    }
}


```

在此示例中，`@Cacheable` 注解用于缓存 `getEntityById()`方法的结果，该方法根据其 `ID` 从数据库中检索 MyEntity 对象。

但是如果我们更新数据呢？旧数据仍然在缓存中？

@CachePut
---------

然后`@CachePut` 出来了, 与 `@Cacheable` 注解不同的是使用 `@CachePut` 注解标注的方法，在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式写入指定的缓存中。`@CachePut` 注解一般用于更新缓存数据，相当于缓存使用的是写模式中的双写模式。

```
@Service
public class MyService {

    @Autowired
    private MyRepository repository;

    @CachePut(value = "myCache", key = "#entity.id")
    public void saveEntity(MyEntity entity) {
        repository.save(entity);
    }
}


```

@CacheEvict
-----------

标注了 `@CacheEvict` 注解的方法在被调用时，会从缓存中移除已存储的数据。`@CacheEvict` 注解一般用于删除缓存数据，相当于缓存使用的是写模式中的失效模式。

![](https://mmbiz.qpic.cn/mmbiz_png/oH5VKTC5sLjKHsa2rSwMwVlNyzYP1yRVoqN9yDaR3CFC79hcVHa4sdHPACzKhe6ViaTnNy62qoc1j7HNJEotibAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```
@Service
public class MyService {

    @Autowired
    private MyRepository repository;

     @CacheEvict(value = "myCache", key = "#id")
    public void deleteEntityById(Long id) {
        repository.deleteById(id);
    }
}


```

@Caching
--------

`@Caching` 注解用于在一个方法或者类上，同时指定多个 Spring Cache 相关的注解。

![](https://mmbiz.qpic.cn/mmbiz_png/oH5VKTC5sLjKHsa2rSwMwVlNyzYP1yRV89bQH4e3ibo49K2SiamgCD7ay1mGktYXkgApuAx6abj7ibZibhkSlLOU3A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

例子 1：`@Caching`注解中的`evict`属性指定在调用方法 `saveEntity` 时失效两个缓存。

```
@Service
public class MyService {

    @Autowired
    private MyRepository repository;

    @Cacheable(value = "myCache", key = "#id")
    public MyEntity getEntityById(Long id) {
        return repository.findById(id).orElse(null);
    }

    @Caching(evict = {
        @CacheEvict(value = "myCache", key = "#entity.id"),
        @CacheEvict(value = "otherCache", key = "#entity.id")
    })
    public void saveEntity(MyEntity entity) {
        repository.save(entity);
    }

}


```

例子 2：调用`getEntityById`方法时，Spring 会先检查结果是否已经缓存在`myCache`缓存中。如果是，`Spring` 将返回缓存的结果而不是执行该方法。如果结果尚未缓存，Spring 将执行该方法并将结果缓存在 `myCache` 缓存中。方法执行后，Spring 会根据`@CacheEvict`注解从`otherCache`缓存中移除缓存结果。

```
@Service
public class MyService {

    @Caching(
        cacheable = {
            @Cacheable(value = "myCache", key = "#id")
        },
        evict = {
            @CacheEvict(value = "otherCache", key = "#id")
        }
    )
    public MyEntity getEntityById(Long id) {
        return repository.findById(id).orElse(null);
    }

}


```

例子 3：当调用`saveData`方法时，Spring 会根据`@CacheEvict`注解先从`otherCache`缓存中移除数据。然后，Spring 将执行该方法并将结果保存到数据库或外部 API。

方法执行后，Spring 会根据`@CachePut`注解将结果添加到 `myCache`、`myOtherCache` 和 `myThirdCache` 缓存中。Spring 还将根据`@Cacheable`注解检查结果是否已缓存在 `myFourthCache` 和 `myFifthCache` 缓存中。如果结果尚未缓存，Spring 会将结果缓存在适当的缓存中。如果结果已经被缓存，Spring 将返回缓存的结果，而不是再次执行该方法。

```
@Service
public class MyService {

    @Caching(
        put = {
            @CachePut(value = "myCache", key = "#result.id"),
            @CachePut(value = "myOtherCache", key = "#result.id"),
            @CachePut(value = "myThirdCache", key = "#result.name")
        },
        evict = {
            @CacheEvict(value = "otherCache", key = "#id")
        },
        cacheable = {
            @Cacheable(value = "myFourthCache", key = "#id"),
            @Cacheable(value = "myFifthCache", key = "#result.id")
        }
    )
    public MyEntity saveData(Long id, String name) {
        // Code to save data to a database or external API
        MyEntity entity = new MyEntity(id, name);
        return entity;
    }

}


```

@CacheConfig
------------

通过`@CacheConfig` 注解，我们可以将一些缓存配置简化到类级别的一个地方，这样我们就不必多次声明相关值：

```
@CacheConfig(cacheNames={"myCache"})
@Service
public class MyService {

    @Autowired
    private MyRepository repository;

    @Cacheable(key = "#id")
    public MyEntity getEntityById(Long id) {
        return repository.findById(id).orElse(null);
    }

    @CachePut(key = "#entity.id")
    public void saveEntity(MyEntity entity) {
        repository.save(entity);
    }

    @CacheEvict(key = "#id")
    public void deleteEntityById(Long id) {
        repository.deleteById(id);
    }
}


```

Condition & Unless
------------------

*   `condition`作用：指定缓存的条件（满足什么条件才缓存），可用 `SpEL` 表达式（如 `#id>0`，表示当入参 id 大于 0 时才缓存）
    
*   `unless`作用 : 否定缓存，即满足 `unless` 指定的条件时，方法的结果不进行缓存，使用 `unless` 时可以在调用的方法获取到结果之后再进行判断（如 #result == null，表示如果结果为 null 时不缓存）
    

```
//when id >10, the @CachePut works. 
@CachePut(key = "#entity.id", condition="#entity.id > 10")
public void saveEntity(MyEntity entity) {
 repository.save(entity);
}


//when result != null, the @CachePut works.
@CachePut(key = "#id", condition="#result == null")
public void saveEntity1(MyEntity entity) {
 repository.save(entity);
}


```

清理全部缓存
------

通过`allEntries`、`beforeInvocation`属性可以来清除全部缓存数据，不过`allEntries`是方法调用后清理，`beforeInvocation`是方法调用前清理。

```
//方法调用完成之后，清理所有缓存
@CacheEvict(value="myCache",allEntries=true)
public void delectAll() {
    repository.deleteAll();
}

//方法调用之前，清除所有缓存
@CacheEvict(value="myCache",beforeInvocation=true)
public void delectAll() {
    repository.deleteAll();
}


```

SpEL 表达式
--------

Spring Cache 注解中频繁用到 SpEL 表达式，那么具体如何使用呢？

**SpEL 表达式的语法**

![](https://mmbiz.qpic.cn/mmbiz_png/oH5VKTC5sLjKHsa2rSwMwVlNyzYP1yRV8LF4r2rAkicPVrXIVjia34Svmhsp9MXRVUNGZqpOhe6HDeSTskAGNXdg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Spring Cache 可用的变量**

![](https://mmbiz.qpic.cn/mmbiz_png/oH5VKTC5sLjKHsa2rSwMwVlNyzYP1yRVxTahvd6AIgkS72aQ4B49wU4HqJgUOLQlKff7HnoCWreIq4d4BSrbibQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最佳实践
----

通过`Spring`缓存注解可以快速优雅地在我们项目中实现缓存的操作，但是在双写模式或者失效模式下，可能会出现缓存数据一致性问题（读取到脏数据），`Spring Cache` 暂时没办法解决。最后我们再总结下 Spring Cache 使用的一些最佳实践。

*   只缓存经常读取的数据：缓存可以显着提高性能，但只缓存经常访问的数据很重要。很少或从不访问的缓存数据会占用宝贵的内存资源，从而导致性能问题。
    
*   根据应用程序的特定需求选择合适的缓存提供程序和策略。`SpringBoot` 支持多种缓存提供程序，包括 `Ehcache`、`Hazelcast` 和 `Redis`。
    
*   使用缓存时请注意潜在的线程安全问题。对缓存的并发访问可能会导致数据不一致或不正确，因此选择线程安全的缓存提供程序并在必要时使用适当的同步机制非常重要。
    
*   避免过度缓存。缓存对于提高性能很有用，但过多的缓存实际上会消耗宝贵的内存资源，从而损害性能。在缓存频繁使用的数据和允许垃圾收集不常用的数据之间取得平衡很重要。
    
*   使用适当的缓存逐出策略。使用缓存时，重要的是定义适当的缓存逐出策略以确保在必要时从缓存中删除旧的或陈旧的数据。
    
*   使用适当的缓存键设计。缓存键对于每个数据项都应该是唯一的，并且应该考虑可能影响缓存数据的任何相关参数，例如用户 ID、时间或位置。
    
*   常规数据（读多写少、即时性与一致性要求不高的数据）完全可以使用 Spring Cache，至于写模式下缓存数据一致性问题的解决，只要缓存数据有设置过期时间就足够了。
    
*   特殊数据（读多写多、即时性与一致性要求非常高的数据），不能使用 Spring Cache，建议考虑特殊的设计（例如使用 Cancal 中间件等）。
    

  

* * *

```
程序汪资料链接

程序汪接私活项目目录，2023年总结


Java项目分享  最新整理全集，找项目不累啦 07版

欢迎添加程序汪微信 itwang007  进粉丝群

```