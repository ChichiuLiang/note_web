> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7360984753463803930?utm_source=gold_browser_extension)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/436b1e1024e4418bb69323ebd10afb78~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.jpg#?w=1000&h=988&s=836860&e=jpg&b=041d33) 题图来自 APOD

首先要声明的就是，千万级数据对于 MySQL 来说就是不太合理的一个存在。

优化 MySQL 千万级数据策略还是比较多的。

*   分表分库
*   创建中间表，汇总表
*   修改为多个子查询

这里讨论的情况是在 MySQL 一张表的数据达到千万级别。表设计很烂，业务统计规则又不允许把 sql 拆成多个子查询。

在这样的情况下，开发者可以尝试通过优化 SQL 来达到查询的目的。

当 MySQL 一张表的数据达到千万级别，会出现一些特殊的情况。这里主要是讨论在比较极端的情况下 SQL 的优化策略。

先来个千万级数据
--------

通过存储过程传递函数制造 1000 万条数据。

表结构如下：

```
CREATE TABLE `orders` (
  `order_id` int NOT NULL AUTO_INCREMENT,
  `user_id` int DEFAULT NULL,
  `order_date` date NOT NULL,
  `total_amount` decimal(10,2) NOT NULL,
  PRIMARY KEY (`order_id`),
  KEY `idx_user_id` (`user_id`) USING BTREE,
  KEY `idx_user_amount` (`user_id`,`total_amount`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

CREATE TABLE `users` (
  `user_id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `email` varchar(100) COLLATE utf8mb4_general_ci NOT NULL,
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`user_id`),
  KEY `idx_user_id` (`user_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;


```

造数据的存储过程如下。

用户数据：

```
-- 产生用户存储过程，1000个
CREATE DEFINER=`root`@`localhost` PROCEDURE `create_users`()
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE total_users INT DEFAULT 1000; -- 调整用户数量
    DECLARE rnd_username VARCHAR(50);
    DECLARE rnd_email VARCHAR(100);

    WHILE i < total_users DO
        -- 生成随机用户名和邮箱
        SET rnd_username = CONCAT('User', FLOOR(1 + RAND() * 10000000)); -- 假设用户名唯一
        SET rnd_email = CONCAT(rnd_username, '@example.com'); -- 假设邮箱唯一
        -- 将数据插入用户表
        INSERT INTO users (username, email) VALUES (rnd_username, rnd_email);

        SET i = i + 1;
    END WHILE;
END


```

订单数据生成存储过程如下：

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `generate_orders`()
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE total_users INT DEFAULT 1000; -- 用户数量
    DECLARE total_orders_per_user INT DEFAULT 1000; -- 每个用户的订单数量
    DECLARE rnd_user_id INT;
    DECLARE rnd_order_date DATE;
    DECLARE rnd_total_amount DECIMAL(10, 2);
    DECLARE j INT DEFAULT 0;

    WHILE i < total_users DO
        -- 获取用户ID
        SELECT user_id INTO rnd_user_id FROM users LIMIT i, 1;

        WHILE j < total_orders_per_user DO
            -- 生成订单日期和总金额
            SET rnd_order_date = DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 1096) DAY); -- 2020-01-01和2022-12-31之间的随机日期
            SET rnd_total_amount = ROUND(RAND() * 1000, 2); -- 0到1000之间的随机总金额
            -- 将数据插入订单表
            INSERT INTO orders (user_id, order_date, total_amount) VALUES (rnd_user_id, rnd_order_date, rnd_total_amount);

            SET j = j + 1;
        END WHILE;
        SET j = 0;

        SET i = i + 1;
    END WHILE;
END


```

将 users 和 orders 的数据生成分开，这样可以通过多次调用 orders 存储过程多线程参数数据。

调用一次`call create_users()`，然后开 15 个窗口调用 orders 存储过程`call generate_orders()`。

整个过程会产生 1000 个用户，`15*1000*1000`也就是 1500 万条订单数据。

原始 SQL
------

这是一个很简单的 sql，统计每个用户的订单总额。

在默认情况下，什么索引都没有创建，需要花费 190+s 的时间。

```
-- 第一个版本
SELECT a.*,sum(b.total_amount) as total from users a left join orders b  on a.user_id = b.user_id
group by a.user_id;


```

explain 分析如下：

<table><thead><tr><th>id</th><th>select_type</th><th>table</th><th>partitions</th><th>type</th><th>possible_keys</th><th>key</th><th>key_len</th><th>ref</th><th>rows</th><th>filtered</th><th>Extra</th></tr></thead><tbody><tr><td>1</td><td>SIMPLE</td><td>a</td><td></td><td>ALL</td><td>PRIMARY</td><td></td><td></td><td></td><td>1000</td><td>100.0</td><td>Using temporary</td></tr><tr><td>1</td><td>SIMPLE</td><td>b</td><td></td><td>ALL</td><td></td><td></td><td></td><td></td><td>13016086</td><td>100.0</td><td>Using where; Using join buffer (hash join)</td></tr></tbody></table>

可以看到什么索引也没使用，type 为 all，直接全表扫描。

**用时 191s。**

第一次优化：普通索引
----------

把查询条件用到的 sql 条件都创建索引。也就是 where 和 join、sum 涉及到的知道。

```
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_orders_total_amount ON orders (total_amount);
CREATE INDEX idx_users_user_id ON users (user_id);


```

查询 sql 仍然是第一个版本。

```
-- 第一个版本
SELECT a.*,sum(b.total_amount) as total from users a left join orders b  on a.user_id = b.user_id
group by a.user_id;


```

先看看 expalin 的结果：

<table><thead><tr><th>id</th><th>select_type</th><th>table</th><th>partitions</th><th>type</th><th>possible_keys</th><th>key</th><th>key_len</th><th>ref</th><th>rows</th><th>filtered</th><th>Extra</th></tr></thead><tbody><tr><td>1</td><td>SIMPLE</td><td>a</td><td></td><td>index</td><td>PRIMARY,idx_users_user_id</td><td>PRIMARY</td><td>4</td><td></td><td>1</td><td>100.0</td><td></td></tr><tr><td>1</td><td>SIMPLE</td><td>b</td><td></td><td>ref</td><td>idx_orders_user_id</td><td>idx_orders_user_id</td><td>5</td><td>test2.a.user_id</td><td>13003</td><td>100.0</td><td></td></tr></tbody></table>

type 为 index 或者 ref，全部走的索引。

查询结果却让人失望，这次用的时间更多，用了 460+s。也就是说查询变慢了。

推测是由于 mysql 的回表机制导致查询变得更慢了。所以接下来继续优化索引。

第二次优化：覆盖索引
----------

覆盖索引是指一个索引包含了查询所需的所有列，从而可以满足查询的要求，而不需要访问实际的数据行。

通常情况下，数据库查询需要根据索引定位到对应的数据行，然后再从数据行中获取所需的列值。

而当索引中包含了查询所需的所有列时，数据库引擎可以直接通过索引就能够满足查询的要求，无需访问实际的数据行，这样就可以提高查询性能。

这也是普通索引添加了还是查询慢的原因，因为普通索引命中了还是会去找主键，通过主键找到关联字段的值做过滤。

```
-- 先不删除普通索引
-- drop INDEX idx_orders_user_id ON orders;
-- drop INDEX idx_orders_total_amount ON orders;
CREATE INDEX idx_orders_total_amount_user_id ON orders (total_amount,user_id);
CREATE INDEX idx_orders_user_id_total_amount ON orders (user_id,total_amount);


```

1500 万数据创建索引就花费了 300+s。所以创建索引得适度。

查询 sql 还是第一个版本。

```
-- 第一个版本
SELECT a.*,sum(b.total_amount) as total from users a left join orders b  on a.user_id = b.user_id
group by a.user_id;


```

先看看 expalin 的结果：

<table><thead><tr><th>id</th><th>select_type</th><th>table</th><th>partitions</th><th>type</th><th>possible_keys</th><th>key</th><th>key_len</th><th>ref</th><th>rows</th><th>filtered</th><th>Extra</th></tr></thead><tbody><tr><td>1</td><td>SIMPLE</td><td>a</td><td></td><td>index</td><td>PRIMARY,idx_users_user_id</td><td>PRIMARY</td><td>4</td><td></td><td>1</td><td>100.0</td><td></td></tr><tr><td>1</td><td>SIMPLE</td><td>b</td><td></td><td>ref</td><td>idx_orders_user_id,idx_orders_user_id_total_amount</td><td>idx_orders_user_id_total_amount</td><td>5</td><td>test2.a.user_id</td><td>874</td><td>100.0</td><td>Using index</td></tr></tbody></table>

可以看到 orders 表的 type 从 index 提升到了 ref。

此时的查询时间为从 460s + 降低到 10s 了。

结果证明覆盖索引能提升查询速度。

问题就在于这次建的两个覆盖索引，只有 idx_orders_user_id_total_amount 降低了查询时间，而 idx_orders_total_amount_user_id 没有。

这个和 mysql 的关键词执行顺序有一定关系（推测，没找到资料）。

mysql 执行顺序如下：

```
from
on
join
where
group by
having
select
distinct
union （all）
order by
limit


```

可以看到在覆盖索引使用过程先是 where, 再是到 select 的 sum 函数。这也是 idx_orders_user_id_total_amount 索引的创建顺序。

```
drop INDEX idx_orders_user_id ON orders;
drop INDEX idx_orders_total_amount ON orders;
drop INDEX idx_orders_total_amount_user_id ON orders;


```

drop 掉相关的多余索引可以发现执行查询时间没有变化，仍然为 10s。

索引优化这块差不多就是通过覆盖索引来命中索引。

第三次优化：减少数据量
-----------

减少数据量在业务上来说就是移除不必要的数据，或者可以在架构设计这块做一些工作。

分表就是这个原则。

通过这个方式能把千万的数据量减少到百万甚至几十万的量。提升的查询速度是可以想象的。

```
-- 第三次优化：减少数据量
SELECT a.*,sum(b.total_amount) as total from users a left join orders b  on a.user_id = b.user_id
where a.user_id > 1033
group by a.user_id;


```

expain 结果如下：

<table><thead><tr><th>id</th><th>select_type</th><th>table</th><th>partitions</th><th>type</th><th>possible_keys</th><th>key</th><th>key_len</th><th>ref</th><th>rows</th><th>filtered</th><th>Extra</th></tr></thead><tbody><tr><td>1</td><td>SIMPLE</td><td>a</td><td></td><td>range</td><td>PRIMARY,idx_users_user_id</td><td>PRIMARY</td><td>4</td><td></td><td>685</td><td>100.0</td><td>Using where</td></tr><tr><td>1</td><td>SIMPLE</td><td>b</td><td></td><td>ref</td><td>idx_orders_user_id_total_amount</td><td>idx_orders_user_id_total_amount</td><td>5</td><td>test2.a.user_id</td><td>874</td><td>100.0</td><td>Using index</td></tr></tbody></table>

可以看到 users 表的 type 为 range。能过滤一部分数据量。

查询时间从 10s 降低到 7s，减少数据量证明有效。

第四次优化：小表驱动大表
------------

在 MySQL 中，通常情况下，优化器会根据查询条件和表的大小选择合适的驱动表（即主导表）。

小表驱动大表是一种优化策略，它指的是在连接查询中，优先选择小表作为驱动表，以减少连接操作所需的内存和处理时间。

在第三次优化的结果上，可以尝试使用小表驱动大表优化策略。

```
-- 第三个版本，小标驱动大表  没啥效果
SELECT a.*,sum(b.total_amount) as total from users a
left join (select user_id,total_amount from orders c where c.user_id > 1033 ) b  on a.user_id = b.user_id
where a.user_id > 1033
group by a.user_id;


```

将 left join 的表修改为子查询，能提前过滤一部分数据量。

expain 结果如下：

<table><thead><tr><th>id</th><th>select_type</th><th>table</th><th>partitions</th><th>type</th><th>possible_keys</th><th>key</th><th>key_len</th><th>ref</th><th>rows</th><th>filtered</th><th>Extra</th></tr></thead><tbody><tr><td>1</td><td>SIMPLE</td><td>a</td><td></td><td>range</td><td>PRIMARY,idx_users_user_id</td><td>PRIMARY</td><td>4</td><td></td><td>685</td><td>100.0</td><td>Using where</td></tr><tr><td>1</td><td>SIMPLE</td><td>c</td><td></td><td>ref</td><td>idx_orders_user_id_total_amount</td><td>idx_orders_user_id_total_amount</td><td>5</td><td>test2.a.user_id</td><td>874</td><td>100.0</td><td>Using where; Using index</td></tr></tbody></table>

可以看到 explain 没什么变化。实际执行效果也没啥变化。

小表驱动大表在这里无效，但是可以结合具体的业务进行优化 sql。这个策略是没问题的。

第五次优化：强制索引
----------

当 MySQL 中的 IN 子句用于查询千万级数据时，如果未正确设计和使用索引，可能导致索引失效，从而影响查询性能。

通常情况下，MySQL 的优化器会根据查询条件选择最优的执行计划，包括选择合适的索引。然而，对于大数据量的 IN 子句查询，MySQL 可能无法有效使用索引，从而导致全表扫描或索引失效。

查询 sql 如下，由于 in 的数据量不是很稀疏，实际查询强制索引和普通索引效果一致

```
-- 第五个版本，强制索引 
SELECT a.*,sum(b.total_amount) as total from users a left join orders b force index (idx_orders_user_id_total_amount)  on a.user_id = b.user_id
where b.user_id in (1033,1034,1035,1036,1037,1038)
group by a.user_id;
-- 第五个版本，不走强制索引 
SELECT a.*,sum(b.total_amount) as total from users a left join orders b  on a.user_id = b.user_id
where b.user_id in (1033,1034,1035,1036,1037,1038)
group by a.user_id;


```

查询时间都是零点几秒。

笔者在实际业务中是遇到过这种场景的，业务 sql 更加复杂。这里由于临时创建的订单用户表没复现。

当你发现 explain 都是命中索引的，但是查询依然很慢。这个强制索引可以试试。

优化策略
----

*   提前命中索引，小表驱动大表
*   千万级数据 in 索引失效，进行强制索引
*   使用覆盖索引解决回表问题

下次该怎么优化 SQL
-----------

*   数据接近千万级，需要分表，比如按照用户 id 取模分表。
*   用汇总表代替子查询来命中索引，比如把小时表生成日表、月表汇总数据。
*   关联字段冗余、直接放到一张表就是单表查询了。
*   命中索引，空间换时间，这也是本文分析的场景。

关于命中索引核心点就是覆盖索引，再者是千万数据产生的特有场景需要走强制索引。

tips
----

### explain 结果 type 的含义

在 MySQL 的 `EXPLAIN` 查询结果中，`type` 字段表示了查询使用的访问类型，即查询执行过程中的访问方法。

根据不同的访问类型，MySQL 查询优化器将选择不同的执行计划。以下是 `type` 字段可能的取值及其含义：

*   **system：** 这是最好的情况，表示查询只返回一行结果。这通常是通过直接访问表的 `PRIMARY KEY` 或唯一索引来完成的。
*   **const：** 表示 MySQL 在查询中找到了常量值，这是在连接的第一个表中进行的。由于这是常量条件，MySQL 只会读取一次表中的一行数据。例如，通过主键访问一行数据。
*   **eq_ref：** 类似于 `const`，但在使用了索引的情况下。此类型的查询是通过某个唯一索引来访问表的，对于每个索引键值，表中只有一行匹配。常见于使用主键或唯一索引进行连接操作。
*   **ref：** 表示此查询使用了非唯一索引来查找值。返回的是所有匹配某个单独值的行。该类型一般出现在联接操作中，使用了非唯一索引或者索引前缀。
*   **range：** 表示查询使用了索引来进行范围检索，通常出现在带有范围条件的查询语句中，例如 `BETWEEN`、`IN()`、`>、<`等。
*   **index：** 表示 MySQL 将扫描整个索引来找到所需的行。这通常是在没有合适的索引的情况下，MySQL 会选择使用这种访问类型。
*   **all：** 表示 MySQL 将扫描全表以找到所需的行，这是最差的情况。这种情况下，MySQL 将对表中的每一行执行完整的扫描。

通常来说，`type` 字段的排序从最好到最差依次是 `system`、`const`、`eq_ref`、`ref`、`range`、`index`、`all`，当然，实际情况取决于查询的具体情况、表结构和索引的使用情况。更好的查询性能通常对应着更好的 `type` 类型。

### mysql 的回表机制

在 MySQL 中，回表（"ref" or "Bookmark Lookup" in English）是指在使用索引进行查询时，MySQL 首先通过索引找到满足条件的行的位置，然后再回到主表（或称为数据表）中查找完整的行数据的过程。

这个过程通常发生在某些查询中，特别是涉及到覆盖索引无法满足查询需求时。

当一个查询不能完全通过索引满足时，MySQL 就需要回到主表中查找更多的信息。这种情况通常出现在以下几种情况下：

*   **非覆盖索引查询：** 如果查询需要返回主表中未包含在索引中的其他列的数据时，MySQL 就需要回到主表中查找这些额外的列数据。
*   **使用索引范围条件：** 当查询中使用了范围条件（例如 `BETWEEN`、`>`、`<` 等），而索引只能定位到范围起始位置时，MySQL 需要回到主表中检查满足范围条件的完整行。
*   **使用了聚簇索引但需要查找的列不在索引中：** 在使用了聚簇索引的表中，如果需要查询的列不在聚簇索引中，MySQL 需要回到主表中查找这些列的数据。

当 MySQL 需要执行回表操作时，会发生额外的磁盘访问，因为需要读取主表中的数据。这可能会导致性能下降，特别是在大型数据表中或者在高并发环境中。

为了尽量减少回表操作的发生，可以考虑以下几点：

*   创建覆盖索引：确保查询所需的所有列都包含在索引中，从而避免回表操作。
*   优化查询语句：尽量避免使用范围条件，或者确保所有的过滤条件都可以被索引完全匹配。
*   考虑表设计：在设计数据库表结构时，可以考虑将常用的查询字段都包含在索引中，以减少回表操作的发生。

关于作者
----

来自一线全栈程序员 nine 的探索与实践，持续迭代中。

欢迎关注或者点个小红心~