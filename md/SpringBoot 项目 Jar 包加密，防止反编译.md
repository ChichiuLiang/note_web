> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/OZSSiKDZNF5rre_AYbjxBg)

```
胖虎和朋友原创的视频教程有兴趣的可以看看：




（文末附课程大纲）




👏2024 最新，Java成神之路，架构视频（点击查看）


😉超全技术栈的Java入门+进阶+实战！（点击查看）

```

1场景
---

最近项目要求部署到其他公司的服务器上，但是又不想将源码泄露出去。要求对正式环境的启动包进行安全性处理，防止客户直接通过反编译工具将代码反编译出来。

2方案
---

> 第一种方案使用代码混淆

采用proguard-maven-plugin插件

在单模块中此方案还算简单，但是现在项目一般都是多模块，一个模块依赖多个公共模块。那么使用此方案就比较麻烦，配置复杂，文档难懂，各模块之间的调用在是否混淆时极其容易出错。

> 第二种方案使用代码加密

采用classfinal-maven-plugin插件

此方案比对上面的方案来说，就简单了许多。直接配置一个插件就可以实现源码的安全性保护。并且可以对yml、properties配置文件以及lib目录下的maven依赖进行加密处理。若想指定机器启动，支持绑定机器，项目加密后只能在特定机器运行。

ClassFinal项目源码地址[1]

3项目操作
-----

只需要在启动类的pom.xml文件中加如下插件即可，需要注意的是，改插件时要放到spring-boot-maven-plugin插件后面，否则不起作用。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <!--
                1. 加密后,方法体被清空,保留方法参数、注解等信息.主要兼容swagger文档注解扫描
                2. 方法体被清空后,反编译只能看到方法名和注解,看不到方法体的具体内容
                3. 加密后的项目需要设置javaagent来启动,启动过程中解密class,完全内存解密,不留下任何解密后的文件
                4. 启动加密后的jar,生成xxx-encrypted.jar,这个就是加密后的jar文件,加密后不可直接执行
                5. 无密码启动方式,java -javaagent:xxx-encrypted.jar -jar xxx-encrypted.jar
                6. 有密码启动方式,java -javaagent:xxx-encrypted.jar='-pwd= 密码' -jar xxx-encrypted.jar
            -->
            <groupId>net.roseboy</groupId>
            <artifactId>classfinal-maven-plugin</artifactId>
            <version>1.2.1</version>
            <configuration>
                <password>#</password><!-- #表示启动时不需要密码,事实上对于代码混淆来说,这个密码没什么用,它只是一个启动密码 -->
                <excludes>org.spring</excludes>
                <packages>${groupId}</packages><!-- 加密的包名,多个包用逗号分开 -->
                <cfgfiles>application.yml,application-dev.yml</cfgfiles><!-- 加密的配置文件,多个包用逗号分开 -->
                <libjars>hutool-all.jar</libjars> <!-- jar包lib下面要加密的jar依赖文件,多个包用逗号分开 -->
                <code>xxxx</code> <!-- 指定机器启动,机器码 -->
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>classFinal</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>


```

4启动方式
-----

无密码启动

```
java -javaagent:xxx-encrypted.jar -jar xxx-encrypted.jar


```

有密码启动

```
java -javaagent:xxx-encrypted.jar='-pwd=密码' -jar xxx-encrypted.jar


```

5反编译效果
------

启动包加密之后，方法体被清空,保留方法参数、注解等信息.主要兼容swagger文档注解扫描

反编译只能看到方法名和注解,看不到方法体的具体内容

启动过程中解密class,完全内存解密,不留下任何解密后的文件

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

yml配置文件留下空白

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

6绑定机器启动
-------

下载到classfinal-fatjar-1.2.1.jar[2]依赖，在当前依赖下cmd执行`java -jar classfinal-fatjar-1.2.1.jar -C`命令，会自动生成一串机器码

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

将此生成好的机器码，放到maven插件中的code里面即可。这样，打包好的项目只能在生成机器码的机器运行，其他机器则启动不了项目。

### 参考资料

[1]

ClassFinal项目源码地址: _https://gitee.com/roseboy/classfinal.git_

[2]

classfinal-fatjar-1.2.1.jar: _https://repo1.maven.org/maven2/net/roseboy/classfinal-fatjar/1.2.1/classfinal-fatjar-1.2.1.jar_

来源｜juejin.cn/post/7291846601651273769

  

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