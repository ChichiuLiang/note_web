> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_21108311/article/details/82559119)

最近 MySQL 的笔记有点多了，主要是公司 [Oracle](https://so.csdn.net/so/search?q=Oracle&spm=1001.2101.3001.7020) 比较稳定维护较少，上周被安排做了一个 MySQL 亿级数据的迁移，趁此记录下学习笔记；

数据迁移，工作原理和技术支持数据导出、BI 报表之类的相似，差异较大的地方是导入和导出数据量区别，一般报表数据量不会超过几百万，而做数据迁移，如果是互联网企业经常会涉及到千万级、亿级以上的数据量。

导入和导出是两个过程，即使做数据迁移我们也要分开来看，同时，导入 / 导出方式又分为：

1、MySQL 自带导入 / 导出方式

2、各类客户端导入 / 导出方式

先总结下导出：

1、导出对于字段较少 / 字段内容较少的数据，通过客户端方式可以采用 navicat 等工具导出，我这里本次导出三个字段，都是 11 位数字以内的值，用 navicat 导出每分钟大约 250 万数据，

2、MySQL 自带的导出语句：select into outfile 语句；

```
SELECT ... FROM TABLE_A --可以加where条件
INTO OUTFILE "/path/to/file" --导出文件位置
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' -- 字段分割符和包含符
LINES TERMINATED BY '\n';--换行符
```

这里 fields 之前很简单都可看懂，不做说明，讲下 fields 之后的：

FIELDS TERMINATED BY ',' 代表我字段和字段之间用 逗号 分开 ，如：字段 A 字段 B，导出时候显示格式为：A,B

OPTIONALLY ENCLOSED BY '"'代表字段内容用双引号包含，导出格式如："A","B"

LINES TERMINATED BY '\n'; 每条数据换行区分，导出格式如：

"A","B"

"A1","B1"

当然，字段区分和包含符号可以自行定义，定义为：'    #  都可以

用 MySQL 自带导出 / 导入优点是速度极快，缺点是：只能导出文件是在服务器主机所在的本机地址，对于 bi 之类拿到不数据库主机权限的同事这个方式可能奢望了。不过好在对于字段 / 内容较少的报表第三方客户端工具导出速度也不算特别慢；

导入：

重点记录导入，导入主要是 dba 做数据迁移了，方式也分客户端