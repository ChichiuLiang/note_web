> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/BFpQMWjDdhJsuaWtAQIIng)

点击 “IT 码徒”，关注，置顶公众号

每日技术干货，第一时间送达！

1

前言

对于正在运行的 mysql，性能如何，参数设置的是否合理，账号设置的是否存在安全隐患，你是否了然于胸呢？

俗话说工欲善其事，必先利其器，定期对你的 MYSQL 数据库进行一个体检，是保证数据库安全运行的重要手段，因为，好的工具是使你的工作效率倍增！

今天和大家分享几个 mysql 优化的工具，你可以使用它们对你的 mysql 进行一个体检，生成 awr 报告，让你从整体上把握你的数据库的性能情况。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufSDbY2ShvbiahwmOibhicu41WUkhRoDVXl7TOmnMe34JGicbCZzFEGloPbw9Vawh6C1npqargicmzULdQ/640?wx_fmt=png&random=0.7075363564691639)

2

mysqltuner.pl






========================

是 mysql 一个常用的数据库性能诊断工具，主要检查参数设置的合理性包括日志文件、存储引擎、安全建议及性能分析。针对潜在的问题，给出改进的建议。是 mysql 优化的好帮手。

在上一版本中，MySQLTuner 支持 MySQL / MariaDB / Percona Server 的约 300 个指标。

> 项目地址：https://github.com/major/MySQLTuner-perl

#### 1.1、下载

```
[root@localhost ~]#wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/mysqltuner.pl


```

#### 1.2、使用

```
[root@localhost ~]# ./mysqltuner.pl --socket /var/lib/mysql/mysql.sock
 >> MySQLTuner 1.7.4 - Major Hayden <major@mhtx.net>
 >> Bug reports, feature requests, and downloads at http://mysqltuner.com/
 >> Run with '--help' for additional options and output filtering
[--] Skipped version check for MySQLTuner script
Please enter your MySQL administrative login: root
Please enter your MySQL administrative password: [OK] Currently running supported MySQL version 5.7.23
[OK] Operating on 64-bit architecture 


```

#### 1.3、报告分析

1）重要关注 [!!]（中括号有叹号的项）例如 [!!] Maximum possible memory usage: 4.8G (244.13% of installed RAM)，表示内存已经严重用超了。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufSDbY2ShvbiahwmOibhicu41W6pVFN2t4MkUnF6x1jdcKDzwf3dJwa70yL7ZTYTmKH9FIf8eAOTzNpA/640?wx_fmt=png&random=0.7079080955719266)

2）关注最后给的建议 “Recommendations”。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufSDbY2ShvbiahwmOibhicu41WHGnoqKn1NX1RESBMAsoKJdZ0mEsCGqSx1GAtyGRicTL6dBNbTZtmj5g/640?wx_fmt=png&random=0.8505836546677643)

3

tuning-primer.sh






===========================

mysql 的另一个优化工具，针于 mysql 的整体进行一个体检，对潜在的问题，给出优化的建议。

> 项目地址：https://github.com/BMDan/tuning-primer.sh

目前，支持检测和优化建议的内容如下：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufSDbY2ShvbiahwmOibhicu41WLMpl9qcMhLINu8wbtUmM4E1SQS0ZT5oSqvbuFB4mLWRTlHvIth6nfA/640?wx_fmt=png&random=0.9112078574297853)

#### 2.1 下载

```
[root@localhost ~]#wget https://launchpad.net/mysql-tuning-primer/trunk/1.6-r1/+download/tuning-primer.sh


```

#### 2.2、使用

```
[root@localhost ~]# [root@localhost dba]# ./tuning-primer.sh 
 
 -- MYSQL PERFORMANCE TUNING PRIMER --
 - By: Matthew Montgomery -


```

#### 2.3、报告分析

重点查看有红色告警的选项，根据建议结合自己系统的实际情况进行修改，例如：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufSDbY2ShvbiahwmOibhicu41WJicIu2QBMDx1HdgQtsnyuiazQiawobo9JAUUgNXibD8KA5d7dNqWte4Nvg/640?wx_fmt=png&random=0.5710458321466783)

4

pt-variable-advisor






==============================

pt-variable-advisor 可以分析 MySQL 变量并就可能出现的问题提出建议。

#### 3.1、安装

> https://www.percona.com/downloads/percona-toolkit/LATEST/

```
[root@localhost ~]#wget https://www.percona.com/downloads/percona-toolkit/3.0.13/binary/redhat/7/x86_64/percona-toolkit-3.0.13-re85ce15-el7-x86_64-bundle.tar
[root@localhost ~]#yum install percona-toolkit-3.0.13-1.el7.x86_64.rpm 


```

#### 3.2、使用

pt-variable-advisor 是 pt 工具集的一个子工具，主要用来诊断你的参数设置是否合理。

```
[root@localhost ~]# pt-variable-advisor localhost --socket /var/lib/mysql/mysql.sock


```

#### 3.3、报告分析

重点关注有 WARN 的信息的条目，例如：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufSDbY2ShvbiahwmOibhicu41WZLib3M64FM13TW1JliaYgMEicdy3iawJcxfM2mMb62sNSic0iconalMC04vw/640?wx_fmt=png&random=0.25445142124843034)

5

pt-qurey-digest






==========================

pt-query-digest 主要功能是从日志、进程列表和 tcpdump 分析 MySQL 查询。

#### 4.1、安装

具体参考 3.1 节

#### 4.2、使用

pt-query-digest 主要用来分析 mysql 的慢日志，与 mysqldumpshow 工具相比，py-query_digest 工具的分析结果更具体，更完善。

```
[root@localhost ~]# pt-query-digest /var/lib/mysql/slowtest-slow.log 


```

#### 4.3、常见用法分析

1）直接分析慢查询文件:

```
pt-query-digest /var/lib/mysql/slowtest-slow.log > slow_report.log


```

2）分析最近 12 小时内的查询：

```
pt-query-digest --since=12h /var/lib/mysql/slowtest-slow.log > slow_report2.log


```

3）分析指定时间范围内的查询：

```
pt-query-digest /var/lib/mysql/slowtest-slow.log --since '2017-01-07 09:30:00' --until '2017-01-07 10:00:00'> > slow_report3.log


```

4）分析指含有 select 语句的慢查询

```
pt-query-digest --filter '$event->{fingerprint} =~ m/^select/i' /var/lib/mysql/slowtest-slow.log> slow_report4.log


```

5）针对某个用户的慢查询

```
pt-query-digest --filter '($event->{user} || "") =~ m/^root/i' /var/lib/mysql/slowtest-slow.log> slow_report5.log


```

6）查询所有所有的全表扫描或 full join 的慢查询

```
pt-query-digest --filter '(($event->{Full_scan} || "") eq "yes") ||(($event->{Full_join} || "") eq "yes")' /var/lib/mysql/slowtest-slow.log> slow_report6.log


```

#### 4.4、报告分析

**第一部分：总体统计结果**

*   Overall：总共有多少条查询
    
*   Time range：查询执行的时间范围
    
*   unique：唯一查询数量，即对查询条件进行参数化以后，总共有多少个不同的查询
    
*   total：总计
    
*   min：最小
    
*   max：最大
    
*   avg：平均
    
*   95%：把所有值从小到大排列，位置位于 95% 的那个数，这个数一般最具有参考价值
    
*   median：中位数，把所有值从小到大排列，位置位于中间那个数
    

**第二部分：查询分组统计结果**

*   Rank：所有语句的排名，默认按查询时间降序排列，通过 --order-by 指定
    
*   Query ID：语句的 ID，（去掉多余空格和文本字符，计算 hash 值）
    
*   Response：总的响应时间
    
*   time：该查询在本次分析中总的时间占比
    
*   calls：执行次数，即本次分析总共有多少条这种类型的查询语句
    
*   R/Call：平均每次执行的响应时间
    
*   V/M：响应时间 Variance-to-mean 的比率
    
*   Item：查询对象
    

**第三部分：每一种查询的详细统计结果**

*   ID：查询的 ID 号，和上图的 Query ID 对应
    
*   Databases：数据库名
    
*   Users：各个用户执行的次数（占比）
    
*   Query_time distribution ：查询时间分布, 长短体现区间占比。
    
*   Tables：查询中涉及到的表
    
*   Explain：SQL 语句
    

**PS：防止找不到本篇文章，可以收藏点赞，方便翻阅查找哦。**

  

往期推荐

  

  

[

面试官：try catch 应该在 for 循环里面还是外面？



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503632&idx=1&sn=6867524a16f69899fc2b6e5a578af80d&chksm=ead00111dda7880784bdb2dd46a90033e3d241522699cde63b9aecc38f08f95fdaecd963f4cc&scene=21#wechat_redirect)

[

高性能，短 URL 服务，设计及实现！附带源码...



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503632&idx=2&sn=aafad25c58b8a1dc4e0c61ec5e55e0be&chksm=ead00111dda7880747deafd85e7cb11179dfa913f7c33e65fdbc9cb88662f90ff4b0fd251305&scene=21#wechat_redirect)

[

还在苦恼 Maven 依赖冲突，这款 IDEA 插件帮你解决！



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503552&idx=1&sn=8e5b7036de540799c7ed049abf2bdb64&chksm=ead000c1dda789d7340b8ff57a5f8b0a980bd8139e6e61d4085e69085eddd1050a69ef941f92&scene=21#wechat_redirect)

[

公司新来一个同事，把 @Transactional 事务注解运用得炉火纯青。。



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503543&idx=1&sn=ea11e1e245e1583fe30acd0958ac6518&chksm=ead000b6dda789a03f651c201e1a780426766279c00ff639e8587c3fa7159a73a15e63a981ab&scene=21#wechat_redirect)

[

Spring 中 Bean 的生命周期，让你瞬间通透~



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503525&idx=1&sn=bcd6cbba1e7bb42e7cbc63b6a445eb33&chksm=ead000a4dda789b22ccb1157274c59cb6aae8426d8472db56c48f440823b00e53f276debc721&scene=21#wechat_redirect)

[

写了个工具，让 CRUD 开发效率提升 100 倍，开源咯！



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503515&idx=1&sn=44aa91e4c5bbb23adc1db09f113398d9&chksm=ead0009adda7898c9c8a67802dc7f10165a48a2c47f43fdc9799a939050b79100304f30de3c6&scene=21#wechat_redirect)