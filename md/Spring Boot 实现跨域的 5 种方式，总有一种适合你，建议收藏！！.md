> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-Kyo0OCzIVVLO1WP5FJzuA)

关注公众号回复「**1024**」获取一份架构师资料↓

推荐学习：

[疯了，2 年经验问这些东西。。](http://mp.weixin.qq.com/s?__biz=MzUyNDc0NjM0Nw==&mid=2247504068&idx=1&sn=537b3395e5c7a11ac09e813c7121f1cc&chksm=fa2a3bc0cd5db2d69bd4b657f16207e19b972a3a546e59c6a4148291e5453d867fb49c9f6cfd&scene=21#wechat_redirect)  

[Spring Boot 框架正确的学习姿势！](http://mp.weixin.qq.com/s?__biz=MzUyNDc0NjM0Nw==&mid=2247503443&idx=1&sn=8dec2a4edbbd2e89fed614ce5ef8ebe2&chksm=fa2a2557cd5dac41e29e1e45761ecc2b25f84c0a059e61103d14292ddaddc2874c1a33e1109e&scene=21#wechat_redirect)

#### 

* * *

  

点下面 ↓↓↓ 小程序刷题进大厂        

一、为什么会出现跨域问题
------------

出于浏览器的同源策略限制。同源策略（Sameoriginpolicy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说 Web 是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。

同源策略会阻止一个域的 javascript 脚本和另外一个域的内容进行交互。所谓同源（即指在同一个域）就是两个页面具有相同的协议（protocol），主机（host）和端口号（port）

二、什么是跨域
-------

当一个请求 url 的协议、域名、端口三者之间任意一个与当前页面 url 不同即为跨域![](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpTBq37BAyzPeibmaDkfng3VA1qsJmaQ0qVQR19oexr8kVznsXTfqRuXDj3tkHFOCiaq8e7HmsP1KZKw/640?wx_fmt=png)

三、非同源限制
-------

1.  无法读取非同源网页的 Cookie、LocalStorage 和 IndexedDB
    
2.  无法接触非同源网页的 DOM
    
3.  无法向非同源地址发送 AJAX 请求
    

四、java 后端 实现 CORS 跨域请求的方式
-------------------------

[对于 CORS 的跨域请求，主要有以下几种方式可供选择：](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)

1.  [返回新的 CorsFilter](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)
    
2.  [重写 WebMvcConfigurer](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)
    
3.  [使用注解 @CrossOrigin](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)
    
4.  [手动设置响应头 (HttpServletResponse)](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)
    
5.  [自定 web filter 实现跨域](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)
    

[注意:](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)

*   [CorFilter / WebMvConfigurer / @CrossOrigin 需要 SpringMVC 4.2 以上版本才支持，对应 springBoot 1.3 版本以上](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)
    
*   [上面前两种方式属于全局 CORS 配置，后两种属于局部 CORS 配置。如果使用了局部跨域是会覆盖全局跨域的规则，所以可以通过 @CrossOrigin 注解来进行细粒度更高的跨域资源控制。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)
    
*   [其实无论哪种方案，最终目的都是修改响应头，向响应头中添加浏览器所要求的数据，进而实现跨域](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561837&idx=1&sn=2e493ae330534aae31db5b85165886be&chksm=eb51775bdc26fe4d17d416d0b22944dc6f2bda8c6e00e49aa1ddef6f9dc5c950f31ffb41227e&scene=21#wechat_redirect)。
    
*   最新面试题整理好了，大家可以在 Java 面试库小程序在线刷题。
    

### 1. 返回新的 CorsFilter(全局跨域)

Spring Boot 基础就不介绍了，推荐下这个实战教程：

> https://github.com/javastacks/spring-boot-best-practice

在任意配置类，返回一个 新的 CorsFIlter Bean ，并添加映射路径和具体的 CORS 配置路径。[](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561798&idx=2&sn=b00ee1c2ee5ede29108b5bfbf292c6be&chksm=eb517770dc26fe668a715aff402dc7ff8f23271cbcf33df44ab8d95cc3c3f43b6e16bba1d46c&scene=21#wechat_redirect)

```
@Configuration
public class GlobalCorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        //1. 添加 CORS配置信息
        CorsConfiguration config = new CorsConfiguration();
        //放行哪些原始域
        config.addAllowedOrigin("*");
        //是否发送 Cookie
        config.setAllowCredentials(true);
        //放行哪些请求方式
        config.addAllowedMethod("*");
        //放行哪些原始请求头部信息
        config.addAllowedHeader("*");
        //暴露哪些头部信息
        config.addExposedHeader("*");
        //2. 添加映射路径
        UrlBasedCorsConfigurationSource corsConfigurationSource = new UrlBasedCorsConfigurationSource();
        corsConfigurationSource.registerCorsConfiguration("/**",config);
        //3. 返回新的CorsFilter
        return new CorsFilter(corsConfigurationSource);
    }
}


```

### 2. 重写 WebMvcConfigurer(全局跨域)

```
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                //是否发送Cookie
                .allowCredentials(true)
                //放行哪些原始域
                .allowedOrigins("*")
                .allowedMethods(new String[]{"GET", "POST", "PUT", "DELETE"})
                .allowedHeaders("*")
                .exposedHeaders("*");
    }
}


```

### 3. 使用注解 (局部跨域)

在控制器 (类上) 上使用注解 @CrossOrigin:，表示该类的所有方法允许跨域。Spring Boot 系列最全教程看这里：https://www.javastack.cn/springboot/

```
@RestController
@CrossOrigin(origins = "*")
public class HelloController {
    @RequestMapping("/hello")
    public String hello() {
        return "hello world";
    }
}


```

[在方法上使用注解 @CrossOrigin:](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561798&idx=2&sn=b00ee1c2ee5ede29108b5bfbf292c6be&chksm=eb517770dc26fe668a715aff402dc7ff8f23271cbcf33df44ab8d95cc3c3f43b6e16bba1d46c&scene=21#wechat_redirect)

```
@RequestMapping("/hello")
@CrossOrigin(origins = "*")
 //@CrossOrigin(value = "http://localhost:8081") //指定具体ip允许跨域
public String hello() {
    return "hello world";
}


```

### 4. 手动设置响应头 (局部跨域)

[使用 HttpServletResponse 对象添加响应头 (Access-Control-Allow-Origin) 来授权原始域，这里 Origin 的值也可以设置为 “*”, 表示全部放行。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561798&idx=2&sn=b00ee1c2ee5ede29108b5bfbf292c6be&chksm=eb517770dc26fe668a715aff402dc7ff8f23271cbcf33df44ab8d95cc3c3f43b6e16bba1d46c&scene=21#wechat_redirect)

```
@RequestMapping("/index")
public String index(HttpServletResponse response) {
    response.addHeader("Access-Allow-Control-Origin","*");
    return "index";
}


```

### 5. 使用自定义 filter 实现跨域

[首先编写一个过滤器，可以起名字为 MyCorsFilter.java](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561798&idx=2&sn=b00ee1c2ee5ede29108b5bfbf292c6be&chksm=eb517770dc26fe668a715aff402dc7ff8f23271cbcf33df44ab8d95cc3c3f43b6e16bba1d46c&scene=21#wechat_redirect)

```
package com.mesnac.aop;

import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
@Component
public class MyCorsFilter implements Filter {
  public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
    HttpServletResponse response = (HttpServletResponse) res;
    response.setHeader("Access-Control-Allow-Origin", "*");
    response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
    response.setHeader("Access-Control-Max-Age", "3600");
    response.setHeader("Access-Control-Allow-Headers", "x-requested-with,content-type");
    chain.doFilter(req, res);
  }
  public void init(FilterConfig filterConfig) {}
  public void destroy() {}
}


```

[在 web.xml 中配置这个过滤器，使其生效](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247561798&idx=2&sn=b00ee1c2ee5ede29108b5bfbf292c6be&chksm=eb517770dc26fe668a715aff402dc7ff8f23271cbcf33df44ab8d95cc3c3f43b6e16bba1d46c&scene=21#wechat_redirect)

```
<!-- 跨域访问 START-->
<filter>
 <filter-name>CorsFilter</filter-name>
 <filter-class>com.mesnac.aop.MyCorsFilter</filter-class>
</filter>
<filter-mapping>
 <filter-name>CorsFilter</filter-name>
 <url-pattern>/*</url-pattern>
</filter-mapping>
<!-- 跨域访问 END  -->


```

都学会了吧？建议收藏备用！

> 版权声明：本文为 CSDN 博主「T-OPEN」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。原文链接：https://blog.csdn.net/weter_drop/article/details/112135940

  

* * *

课程推荐
----

如果你想系统学习 Spring Boot，推荐下我的《[Spring Boot 核心技术课](http://mp.weixin.qq.com/s?__biz=MzUyNDc0NjM0Nw==&mid=2247503560&idx=1&sn=3305c997bce738ec6b66e17f90e1e518&chksm=fa2a25cccd5dacdaa8c7d6da04bc6f8691ae03cc98bedad403b57ce05ff8d14afb29e546054d&scene=21#wechat_redirect)》，基于最新 3.x 版本，**几乎覆盖 Spring Boot 所有核心知识点**，一次付费，永久学习…

**感兴趣的扫码订阅学习：**

![](https://mmbiz.qpic.cn/mmbiz_png/mR4CwoLXicg3JIFvtzAtCQP9rPibV9qFsYqbmiaLKZYlZGicp3DLZI0fdicfVb2ACcrVCHUYWVyO2nPPh0BQbvHCasQ/640?wx_fmt=png)

👉 **课程详细介绍：**[Spring Boot 核心技术](http://mp.weixin.qq.com/s?__biz=MzUyNDc0NjM0Nw==&mid=2247503560&idx=1&sn=3305c997bce738ec6b66e17f90e1e518&chksm=fa2a25cccd5dacdaa8c7d6da04bc6f8691ae03cc98bedad403b57ce05ff8d14afb29e546054d&scene=21#wechat_redirect)[课](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247588807&idx=2&sn=6e57cd2b6c589dfd3fa3cb4e5ddc6ae9&chksm=eb511ef1dc2697e7cb437747236ac7cfbeb1589858f53c0362c1096aeb793d07c322db963c5f&scene=21#wechat_redirect)

课程原价 **999** 元，现在只要 **399** 元即可上车，想系统学习 Spring Boot 技术的尽快上车，不管是用来**面试跳槽**，还是**工作所需，**都是相当有必要的。