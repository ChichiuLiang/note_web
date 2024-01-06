> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7071047469978615815)

大家好，我是大彬~

今天给大家分享 MySQL 的索引下推。

什么是索引下推
-------

索引条件下推，也叫索引下推，英文全称`Index Condition Pushdown`，简称 ICP。

索引下推是 **MySQL5.6** 新添加的特性，用于优化数据的查询。

在 MySQL5.6 之前，通过使用非主键索引进行查询的时候，存储引擎通过索引查询数据，然后将结果返回给 MySQL server 层，**在 server 层判断是否符合条件**。

在 MySQL5.6 及以上版本，可以使用索引下推的特性。当存在索引的列做为判断条件时，MySQL server 将这一部分判断条件传递给存储引擎，然后存储引擎会筛选出**符合 MySQL server 传递条件的索引项**，即在存储引擎层根据索引条件**过滤**掉不符合条件的索引项，然后回表查询得到结果，将结果返回给 MySQL server。

可以看到，**有了索引下推的优化，在满足一定的条件下，存储引擎层会在回表查询之前对数据进行过滤，可以减少存储引擎回表查询的次数**。

举个例子
----

假设有一张用户信息表 user_info，有三个字段`name, level, weapon（装备）`，建立联合索引`(name, level)`，user_info 表初始数据如下：

<table><thead><tr><th>id</th><th>name</th><th>level</th><th>weapon</th></tr></thead><tbody><tr><td>1</td><td>大彬</td><td>1</td><td>键盘</td></tr><tr><td>2</td><td>盖聂</td><td>2</td><td>渊虹</td></tr><tr><td>3</td><td>卫庄</td><td>3</td><td>鲨齿</td></tr><tr><td>4</td><td>大铁锤</td><td>4</td><td>铁锤</td></tr></tbody></table>

假如需要匹配姓名第一个字为 "大"，并且 level 为 1 的用户，SQL 语句如下：

```
SELECT * FROM user_info WHERE name LIKE "大%" AND level = 1;
```

那么这条 SQL 具体会怎么执行呢？

下面分情况进行分析。

**先来看看 MySQL5.6 以前的版本**。

前面提到 MySQL5.6 以前的版本没有索引下推，其执行过程如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfb5a498790545389aba6c44c08c21b4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

查询条件`name LIKE` 不是等值匹配，根据**最左匹配原则**，在`(name, level)`索引树上只用到`name`去匹配，查找到两条记录（id 为 1 和 4），拿到这两条记录的 id 分别回表查询，然后将结果返回给 MySQL server，在 MySQL server 层进行`level`字段的判断。**整个过程需要回表 2 次**。

**然后看看 MySQL5.6 及以上版本的执行过程**，如下图。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5f77f6cd130462f932c7b143adae358~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

相比 5.6 以前的版本，多了索引下推的优化，在索引遍历过程中，对**索引中的字段**先做判断，过滤掉不符合条件的索引项，**也就是判断 level 是否等于 1**，level 不为 1 则直接跳过。因此在`(name, level)`索引树只匹配一个记录，之后拿着此记录对应的 id（id=1）回表查询全部数据，**整个过程回表 1 次**。

可以使用 explain 查看是否使用索引下推，当`Extra`列的值为`Using index condition`，则表示使用了索引下推。

总结
--

从上面的例子可以看出，使用索引下推在某些场景下可以有效**减少回表次数**，从而提高查询效率。

码字不易，如果觉得对你有帮助，可以**点个赞**鼓励一下！