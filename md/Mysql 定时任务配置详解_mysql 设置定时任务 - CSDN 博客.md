> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/dong_love_me520/article/details/121654462)

#### 定时任务的配置

*   [1、操作](#1_4)
*   [2、详细讲解](#2_91)

这篇文章可以解决 mysql 的一些定时或是循环操作的工作。

1、操作
----

（1）、查看数据库定时策略是否开启

```
show variables like '%event_sche%';

```

运行结果  
![](https://img-blog.csdnimg.cn/d87163c252764eb8b099c0c111633338.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZOs5Lu7,size_20,color_FFFFFF,t_70,g_se,x_16)  
OFF 表示没有开启，修改成 ON（修改后查再看一次确保修改成功）

```
set global event_scheduler=1;

```

![](https://img-blog.csdnimg.cn/7fb596fb7ad1476582ca032e5ce6c006.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZOs5Lu7,size_20,color_FFFFFF,t_70,g_se,x_16)  
（2）、创建 procedure(存储过程)

```
use demo;
delimiter //
create procedure demo_pro()
begin
-- (begin后end//前放要定时处理的sql用;结尾，可以放多个sql)
update app01_dnsinfo set sniff_time = DATE_ADD(NOW(),  INTERVAL  FLOOR(1 + (RAND() * 10800))   SECOND );
end//
delimiter ;

```

![](https://img-blog.csdnimg.cn/45cb7042c5c946b8b87317758460440a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZOs5Lu7,size_20,color_FFFFFF,t_70,g_se,x_16)  
（3）、创建定时任务

```
-- 创建名为demo01的定时任务
create event demo01
-- 每天凌晨一点开始执行
ON SCHEDULE EVERY 1 DAY STARTS DATE_ADD(DATE_ADD(CURDATE(), INTERVAL 1 DAY), INTERVAL 1 HOUR)
on completion preserve disable
-- 任务使用demo_pro()作为执行体
do call demo_pro();

```

![](https://img-blog.csdnimg.cn/6b76179e066b42b8ba2ded3eed65b7db.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZOs5Lu7,size_20,color_FFFFFF,t_70,g_se,x_16)

（4）查看任务建立情况并开启任务  
查看任务是否创建完毕

```
SELECT event_name,event_definition,interval_value,interval_field,status FROM information_schema.EVENTS;

```

![](https://img-blog.csdnimg.cn/9303de3e654f46dfa8263911eef75dd4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZOs5Lu7,size_20,color_FFFFFF,t_70,g_se,x_16)  
开启和关闭任务

```
-- 开启和关闭创建好的事件
-- 开启定时任务
ALTER EVENT demo01 ON COMPLETION PRESERVE ENABLE;
-- 关闭定时任务
ALTER EVENT demo01 ON COMPLETION PRESERVE DISABLE;

```

删除定时任务

```
drop event demo01；

```

业务原 sql（需求是每天更新表中时间为当天的随机时间）

```
-- 查看定时策略是否开启
show variables like '%event_sche%';
-- 开启定时策略（ON是开启状态）
set global event_scheduler=1;
-- 创建procedure(存储过程)
use 123;
delimiter //
create procedure updateTime()
begin
update demo_table set demo_time = DATE_ADD(NOW(),  INTERVAL  FLOOR(1 + (RAND() * 10800))   SECOND );
end//
delimiter ;
-- 创建定时任务
create event updateTime_LY
ON SCHEDULE EVERY 1 DAY STARTS DATE_ADD(DATE_ADD(CURDATE(), INTERVAL 1 DAY), INTERVAL 1 HOUR)
on completion preserve disable
do call updateTime();
-- 查看定时任务事件
SELECT event_name,event_definition,interval_value,interval_field,status FROM information_schema.EVENTS;
-- 开启和关闭创建好的事件
alter event updateTime_LY on completion preserve enable;//开启定时任务
alter event updateTime_LY on completion preserve disable;//关闭定时任务
-- 删除
drop event updateTime_LY;


```

2、详细讲解
------

（1）event_scheduler 的修改会随着数据库服务器重启恢复到原来状态，设置开机自动开启需要配置 mysql 的配置文件 my.ini 添加下面的一行配置

```
[mysqld]
event_scheduler=ON

```

（2）、procedure  
delimiter // 这个的意思是将语句结尾的；修改为以 // 结尾，这样可以保证多条语句的完整执行，并且在结尾我们用 delimiter ；把结尾字符修改回来了。  
（3）、对于任务的执行方式再此追加几种时间控制

```
-- 单位有：SECOND,MINUTE,HOUR,DAY,WEEK(周),QUARTER(季度),MONTH,YEAR
-- 每秒执行1次
ON SCHEDULE EVERY 1 SECOND
-- 每两分钟执行1次
ON SCHEDULE EVERY 2 MINUTE
-- 每3天执行1次
ON SCHEDULE EVERY 3 DAY
-- 5天后执行
ON SCHEDULE AT CURRENT_TIMESTAMP()+INTERVAL 5 DAY
-- 10分钟后执行
ON SCHEDULE AT CURRENT_TIMESTAMP()+INTERVAL 10 MINUTE
-- 在2016年10月1日，晚上9点50执行
ON SCHEDULE AT '2021-12-01 1:50:00'
-- 5天后开始每天都执行执行到下个月底
ON SCHEDULE EVERY 1 DAY STARTS CURRENT_TIMESTAMP()+INTERVAL 5 DAY ENDS CURRENT_TIMESTAMP()+INTERVAL 1 MONTH
-- 从现在起每天执行，执行5天
ON SCHEDULE EVERY 1 DAY ENDS CURRENT_TIMESTAMP()+INTERVAL 5 DAY
-- 每天凌晨一点执行
ON SCHEDULE EVERY 1 DAY STARTS DATE_ADD(DATE_ADD(CURDATE(), INTERVAL 1 DAY), INTERVAL 1 HOUR)
-- 每个月的五号一点执行一次
ON SCHEDULE EVERY 5 MONTH STARTS DATE_ADD(DATE_ADD(DATE_SUB(CURDATE(),INTERVAL DAY(CURDATE())-1 DAY), INTERVAL 1 MONTH),INTERVAL 1 HOUR)
-- 每年一月一号凌晨三点执行一次
ON SCHEDULE  EVERY 1 YEAR STARTS DATE_ADD(DATE(CONCAT(YEAR(CURDATE()) + 1,'-',1,'-',1)),INTERVAL 3 HOUR)

```

所触及的领域是星辰大海，初心就想问个为什么，不懂的越来越多，尽心为之吧。