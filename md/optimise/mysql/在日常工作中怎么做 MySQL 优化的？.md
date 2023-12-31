> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7303816445040967743)

前言
==

面试题来自：[社招一年半面经分享 (含阿里美团头条京东滴滴)](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUyOTg1OTkyMA%3D%3D%26mid%3D2247484910%26idx%3D1%26sn%3Dc686a382915e18bfc7bca152aa8590de%26scene%3D21%23wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247484910&idx=1&sn=c686a382915e18bfc7bca152aa8590de&scene=21#wechat_redirect")

**文章内容收录到个人网站，方便阅读**：[hardyfish.top/](https://link.juejin.cn?target=https%3A%2F%2Fhardyfish.top%2F "https://hardyfish.top/")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/211748955e80404ba738a8071120238d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1596&h=778&s=573416&e=png&b=f7f2ee)

**MySQL 常见的优化手段分为下面几个方面：**

SQL 优化、设计优化，硬件优化等，其中每个大的方向中又包含多个小的优化点

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac7541fc156a43bb80bfb97b4c8ca3b8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=948&h=433&s=26886&e=png&b=f2f2f2)

下面我们具体来看看

**觉得有收获，希望帮忙点赞，转发下哈，谢谢，谢谢**

SQL 优化
======

此优化方案指的是通过优化 SQL 语句以及索引来提高 MySQL 数据库的运行效率，具体内容如下：

分页优化
----

例如：

```
select * from table where type = 2 and level = 9 order by id asc limit 190289,10;
```

优化方案：

*   延迟关联
    
    先通过 where 条件提取出主键，在将该表与原数据表关联，通过主键 id 提取数据行，而不是通过原来的二级索引提取数据行
    
    例如：
    

```
select a.* from table a, (select id from table where type = 2 and level = 9 order by id asc limit 190289,10 ) b where a.id = b.id
```

*   书签方式
    
    书签方式说白了就是找到 limit 第一个参数对应的主键值，再根据这个主键值再去过滤并 limit
    
    例如：
    

```
select * from table where id > (select * from table where type = 2 and level = 9 order by id asc limit 190289, 1) limit 10;
```

索引优化
----

**正确使用索引**

假如我们没有添加索引，那么在查询时就会触发全表扫描，因此查询的数据就会很多，并且查询效率会很低，为了提高查询的性能，我们就需要给最常使用的查询字段上，添加相应的索引，这样才能提高查询的性能

> 建立覆盖索引

InnoDB 使用辅助索引查询数据时会回表，但是如果索引的叶节点中已经包含要查询的字段，那它没有必要再回表查询了，这就叫覆盖索引

例如对于如下查询：

```
select name from test where city='上海'
```

我们将被查询的字段建立到联合索引中，这样查询结果就可以直接从索引中获取

```
alter table test add index idx_city_name (city, name);
```

> 在 MySQL 5.0 之前的版本尽量避免使用 or 查询

在 MySQL 5.0 之前的版本要尽量避免使用 or 查询，可以使用 union 或者子查询来替代，因为早期的 MySQL 版本使用 or 查询可能会导致索引失效，在 MySQL 5.0 之后的版本中引入了索引合并

索引合并简单来说就是把多条件查询，比如 or 或 and 查询对多个索引分别进行条件扫描，然后将它们各自的结果进行合并，因此就不会导致索引失效的问题了

如果从 Explain 执行计划的 type 列的值是`index_merge`可以看出 MySQL 使用索引合并的方式来执行对表的查询

关于 Explain 的使用可以参考我之前的文章：[最完整的 Explain 总结，SQL 优化不再困难](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUyOTg1OTkyMA%3D%3D%26mid%3D2247484391%26idx%3D1%26sn%3Dc478efeed839831a5dc806536029f2b7%26scene%3D21%23wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247484391&idx=1&sn=c478efeed839831a5dc806536029f2b7&scene=21#wechat_redirect")

> 避免在 where 查询条件中使用 != 或者 <> 操作符

SQL 中，不等于操作符会导致查询引擎放弃索引索引，引起全表扫描，即使比较的字段上有索引

解决方法：通过把不等于操作符改成 or，可以使用索引，避免全表扫描

例如，把`column<>’aaa’，改成column>’aaa’ or column<’aaa’`，就可以使用索引了

> 适当使用前缀索引

MySQL 是支持前缀索引的，也就是说我们可以定义字符串的一部分来作为索引

我们知道索引越长占用的磁盘空间就越大，那么在相同数据页中能放下的索引值也就越少，这就意味着搜索索引需要的查询时间也就越长，进而查询的效率就会降低，所以我们可以适当的选择使用前缀索引，以减少空间的占用和提高查询效率

比如，邮箱的后缀都是固定的 “`@xxx.com`”，那么类似这种后面几位为固定值的字段就非常适合定义为前缀索引

```
alter table test add index index2(email(6));
```

使用前缀索引，定义好长度，就可以做到既节省空间，又不用额外增加太多的查询成本

需要注意的是，前缀索引也存在缺点，MySQL 无法利用前缀索引做 order by 和 group by 操作，也无法作为覆盖索引

> 查询具体的字段而非全部字段

要尽量避免使用`select *`，而是查询需要的字段，这样可以提升速度，以及减少网络传输的带宽压力

> 优化子查询

尽量使用 Join 语句来替代子查询，因为子查询是嵌套查询，而嵌套查询会新创建一张临时表，而临时表的创建与销毁会占用一定的系统资源以及花费一定的时间，同时对于返回结果集比较大的子查询，其对查询性能的影响更大

关于 Join 语句使用，可以参考我之前的文章：[写出好的 Join 语句，前提你得懂这些](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUyOTg1OTkyMA%3D%3D%26mid%3D2247484851%26idx%3D1%26sn%3D47dd680db2a74cd0c04332ad6a3b8f12%26scene%3D21%23wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247484851&idx=1&sn=47dd680db2a74cd0c04332ad6a3b8f12&scene=21#wechat_redirect")

> 小表驱动大表

我们要尽量使用小表驱动大表的方式进行查询，也就是如果 B 表的数据小于 A 表的数据，那执行的顺序就是先查 B 表再查 A 表，具体查询语句如下：

```
select name from A where id in (select id from B);
```

> 不要在列上进行运算操作

不要在列字段上进行算术运算或其他表达式运算，否则可能会导致查询引擎无法正确使用索引，从而影响了查询的效率

```
select * from test where id + 1 = 50;
select * from test where month(updateTime) = 7;
```

一个很容易踩的坑：隐式类型转换：

```
select * from test where skuId=123456
```

skuId 这个字段上有索引，但是 explain 的结果却显示这条语句会全表扫描

原因在于 skuId 的字符类型是 varchar(32)，比较值却是整型，故需要做类型转换

**适当增加冗余字段**

增加冗余字段可以减少大量的连表查询，因为多张表的连表查询性能很低，所有可以适当的增加冗余字段，以减少多张表的关联查询，这是以空间换时间的优化策略

**正确使用联合索引**

使用了 B+ 树的 MySQL 数据库引擎，比如 InnoDB 引擎，在每次查询复合字段时是从左往右匹配数据的，因此在创建联合索引的时候需要注意索引创建的顺序

例如，我们创建了一个联合索引是`idx(name,age,sex)`，那么当我们使用，姓名 + 年龄 + 性别、姓名 + 年龄、姓名等这种最左前缀查询条件时，就会触发联合索引进行查询；然而如果非最左匹配的查询条件，例如，性别 + 姓名这种查询条件就不会触发联合索引

Join 优化
-------

MySQL 的 join 语句连接表使用的是 nested-loop join 算法，这个过程类似于嵌套循环，简单来说，就是遍历驱动表（外层表），每读出一行数据，取出连接字段到被驱动表（内层表）里查找满足条件的行，组成结果行

要提升 join 语句的性能，就要尽可能减少嵌套循环的循环次数

一个显著优化方式是对被驱动表的 join 字段建立索引，利用索引能快速匹配到对应的行，避免与内层表每一行记录做比较，极大地减少总循环次数。另一个优化点，就是连接时用小结果集驱动大结果集，在索引优化的基础上能进一步减少嵌套循环的次数

如果难以判断哪个是大表，哪个是小表，可以用 inner join 连接，MySQL 会自动选择**小表去驱动大表**

关于 Join 语句使用，可以参考我之前的文章：[写出好的 Join 语句，前提你得懂这些](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUyOTg1OTkyMA%3D%3D%26mid%3D2247484851%26idx%3D1%26sn%3D47dd680db2a74cd0c04332ad6a3b8f12%26scene%3D21%23wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247484851&idx=1&sn=47dd680db2a74cd0c04332ad6a3b8f12&scene=21#wechat_redirect")

**避免使用 JOIN 关联太多的表**

对于 MySQL 来说，是存在关联缓存的，缓存的大小可以由`join_buffer_size`参数进行设置

在 MySQL 中，对于同一个 SQL 多关联（join）一个表，就会多分配一个关联缓存，如果在一个 SQL 中关联的表越多，所占用的内存也就越大

如果程序中大量的使用了多表关联的操作，同时`join_buffer_size`设置的也不合理的情况下，就容易造成服务器内存溢出的情况，就会影响到服务器数据库性能的稳定性

排序优化
----

**利用索引扫描做排序**

MySQL 有两种方式生成有序结果：其一是对结果集进行排序的操作，其二是按照索引顺序扫描得出的结果自然是有序的

但是如果索引不能覆盖查询所需列，就不得不每扫描一条记录回表查询一次，这个读操作是随机 IO，通常会比顺序全表扫描还慢

因此，在设计索引时，尽可能使用同一个索引既满足排序又用于查找行

例如：

```
--建立索引（date,staff_id,customer_id）
select staff_id, customer_id from test where date = '2010-01-01' order by staff_id,customer_id;
```

只有当索引的列顺序和 ORDER BY 子句的顺序完全一致，并且所有列的排序方向都一样时，才能够使用索引来对结果做排序

UNION 优化
--------

MySQL 处理 union 的策略是先创建临时表，然后将各个查询结果填充到临时表中最后再来做查询，很多优化策略在 union 查询中都会失效，因为它无法利用索引

最好手工将 where、limit 等子句下推到 union 的各个子查询中，以便优化器可以充分利用这些条件进行优化

此外，除非确实需要服务器去重，一定要使用 union all，如果不加 all 关键字，MySQL 会给临时表加上 distinct 选项，这会导致对整个临时表做唯一性检查，代价很高

慢查询日志
-----

出现慢查询通常的排查手段是先使用慢查询日志功能，查询出比较慢的 SQL 语句，然后再通过 Explain 来查询 SQL 语句的执行计划，最后分析并定位出问题的根源，再进行处理

慢查询日志指的是在 MySQL 中可以通过配置来开启慢查询日志的记录功能，超过`long_query_time`值的 SQL 将会被记录在日志中

我们可以通过设置`“slow_query_log=1”`来开启慢查询

需要注意的是，在开启慢日志功能之后，会对 MySQL 的性能造成一定的影响，因此在生产环境中要慎用此功能

设计优化
====

**尽量避免使用 NULL**

NULL 在 MySQL 中不好处理，存储需要额外空间，运算也需要特殊的运算符，含有 NULL 的列很难进行查询优化

应当指定列为 not null，用 0、空串或其他特殊的值代替空值，比如定义为 int not null default 0

**最小数据长度**

越小的数据类型长度通常在磁盘、内存和 CPU 缓存中都需要更少的空间，处理起来更快

**使用最简单数据类型**

简单的数据类型操作代价更低，比如：能使用 int 类型就不要使用 varchar 类型，因为 int 类型比 varchar 类型的查询效率更高

**尽量少定义 text 类型**

text 类型的查询效率很低，如果必须要使用 text 定义字段，可以把此字段分离成子表，需要查询此字段时使用联合查询，这样可以提高主表的查询效率

**适当分表、分库策略**

分表是指当一张表中的字段更多时，可以尝试将一张大表拆分为多张子表，把使用比较高频的主信息放入主表中，其他的放入子表，这样我们大部分查询只需要查询字段更少的主表就可以完成了，从而有效的提高了查询的效率

分库是指将一个数据库分为多个数据库。比如我们把一个数据库拆分为了多个数据库，一个主数据库用于写入和修改数据，其他的用于同步主数据并提供给客户端查询，这样就把一个库的读和写的压力，分摊给了多个库，从而提高了数据库整体的运行效率

常见类型选择
------

**整数类型宽度设置**

MySQL 可以为整数类型指定宽度，例如 int(11)，实际上并没有意义，它并不会限制值的范围，对于存储和计算来说，int(1) 和 int(20) 是相同的

**VARCHAR 和 CHAR 类型**

char 类型是定长的，而 varchar 存储可变字符串，比定长更省空间，但是 varchar 需要额外 1 或 2 个字节记录字符串长度，更新时也容易产生碎片

需要结合使用场景来选择：如果字符串列最大长度比平均长度大很多，或者列的更新很少，选择 varchar 较合适；如果要存很短的字符串，或者字符串值长度都相同，比如 MD5 值，或者列数据经常变更，选择使用 char 类型

**DATETIME 和 TIMESTAMP 类型**

datetime 的范围更大，能表示从 1001 到 9999 年，timestamp 只能表示从 1970 年到 2038 年。datetime 与时区无关，timestamp 显示值依赖于时区。在大多数场景下，这两种类型都能良好地工作，但是建议使用 timestamp，因为 datetime 占用 8 个字节，timestamp 只占用了 4 个字节，timestamp 空间效率更高

**BLOB 和 TEXT 类型**

blob 和 text 都是为存储很大数据而设计的字符串数据类型，分别采用二进制和字符方式存储

在实际使用中，要慎用这两种类型，它们的查询效率很低，如果字段必须要使用这两种类型，可以把此字段分离成子表，需要查询此字段时使用联合查询，这样可以提高主表的查询效率

范式化
---

当数据较好范式化时，修改的数据更少，而且范式化的表通常要小，可以有更多的数据缓存在内存中，所以执行操作会更快

缺点则是查询时需要更多的关联

第一范式：字段不可分割，数据库默认支持

第二范式：消除对主键的部分依赖，可以在表中加上一个与业务逻辑无关的字段作为主键，比如用自增 id

第三范式：消除对主键的传递依赖，可以将表拆分，减少数据冗余

硬件优化
====

MySQL 对硬件的要求主要体现在三个方面：磁盘、网络和内存

**磁盘**

磁盘应该尽量使用有高性能读写能力的磁盘，比如固态硬盘，这样就可以减少 I/O 运行的时间，从而提高了 MySQL 整体的运行效率

磁盘也可以尽量使用多个小磁盘而不是一个大磁盘，因为磁盘的转速是固定的，有多个小磁盘就相当于拥有多个并行运行的磁盘一样

**网络**

保证网络带宽的通畅（低延迟）以及够大的网络带宽是 MySQL 正常运行的基本条件，如果条件允许的话也可以设置多个网卡，以提高网络高峰期 MySQL 服务器的运行效率

**内存**

MySQL 服务器的内存越大，那么存储和缓存的信息也就越多，而内存的性能是非常高的，从而提高了整个 MySQL 的运行效率

最后
==

**觉得有收获，希望帮忙点赞，转发下哈，谢谢，谢谢**

**文章内容收录到个人网站，方便阅读**：[hardyfish.top/](https://link.juejin.cn?target=https%3A%2F%2Fhardyfish.top%2F "https://hardyfish.top/")

参考资料：

*   《高性能 MySQL》
*   《MySQL 技术内幕：InnodDB 存储引擎》