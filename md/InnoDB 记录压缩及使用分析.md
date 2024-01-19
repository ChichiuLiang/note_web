> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/24334129)

这篇文章，源于 RDS 组内的一次饭后闲聊，两位小伙伴在探讨 InnoDB 启用压缩后的种种，比如在磁盘上是怎么存放数据的，数据页的大小是多少？怎么知道一个页里面可以写入多少压缩前的数据等等。两年前曾看过 InnoDB 压缩的文档和代码，现在好多东西都模糊了，趁此机会，又重温了一把，下面通过问答的方式来阐述。

**压缩干嘛用呢？**直白点，就是为了省空间。假如你的手机是 16G 版 iphone 6s，那么你肯定明白可用的存储空间是多么宝贵。如果能够把 10G 的数据变成 5G 甚至 2G，而且还能不丢信息，那么你是不是乐翻了，做梦都在笑吧。好了，压缩就是干这事儿了。当然在 iphone 上压缩这招一般是行不通的，不信你试试。

**压缩是百利而无一害的吗？**骚年，如果有这种想法，那真是 too young too simple 了。虽然压缩有可能节省存储空间，但也是有代价的，压缩是要消耗 cpu 资源的，甚至是过多的 cpu 资源。目前已知的压缩算法有很多种，不同算法的压缩比和 cpu 消耗也是不一样，常用的算法中 zlib 的压缩比较高，但 cpu 消耗也偏高，早期程序一般采用 zlib 较多，如 InnoDB 的压缩等。snappy 和 quicklz 在压缩比和 cpu 消耗上保存了较好的平衡，这两种算法在新开发的软件中使用更为广泛，包括 TokuDB、MongoDB 等。当然，如果你选择 JPG 等已经压缩过的图片进行压缩，那就是 no zuo no die 了，谁都救不了你。

**什么样的数据适合做压缩呢？**我们拿 MySQL 表作为例子，通常情况下表结构中包含字符型数据列如 char, varchar, text 或 blob 等时，具有较高的压缩率，而一些二进制数据，如整形或浮点型数据列，或者一些已经压缩的多媒体文档，如 jpeg、jpg、png 等格式图片及 mp4、avi 等格式视频，其压缩率都不会好。你可以从一个非压缩表中拷贝数据到一个相同的压缩表中，观察数据大小，来决定是否适合压缩。

**假如我的 MySQL** **实例的表中就存在较多字符型数据列，我想启用压缩，怎么整？**不急，给你宝典，自己去练。首先你的 MySQL 版本必须高于 5.1，现在都已经进入 5.7 时代了，这个不成问题。接下来把 innodb_file_per_table 设置为 1，即除了系统表外，其他表都是单独存一个文件，还需要把 innodb_file_format 设置为 Barracuda。然后在建新表或修改现有表的语句中加入 row_format=compressed key_block_size=8 就可以了。你可以仅加入 row_format=compressed，这样 key_block_size 就取默认值 8KB 了。你也可以仅加入 key_block_size={1/2/4/8/16}，也会默认开启压缩。那么，key_block_size 到底设置为多大才合理呢，这是一个深奥的问题，官方建议如下：To determine the best value for KEY_BLOCK_SIZE, typically you create several copies of the same table with different values for this clause, then measure the size of the resulting .ibd files and see how well each performs with a realistic workload。

**我的 MySQL** **业务类型会对压缩效益有影响吗？**为了尽量避免压缩、解压带来的额外消耗，InnoDB 在压缩页中新增了 modification log 区，通过记录当前的页的修改日志，来避免频繁压缩解压。不同的业务场景，会对压缩效益产生不同的影响，下面几种场景相对适用压缩：如果业务中查询占了绝大部分，只有很少的更新，那么只有较少的页中会出现 modification log 空间不足，进而需要重组或者重新压缩，这种场景下是个不错的选择；如果应用只是单纯的插入，此时数据的插入按照主键递增的方式组织，即使存在二级索引也基本上不需要频繁重组和重压缩；对于删除操作，由于 InnoDB 删除行采用打标志位的方式来删除，对记录的删除是通过修改页中没有被压缩的元数据的方式实现，所以效率很高；对于更新操作，如果不是对索引列或者存储在 off-page 的 blob，text，长字符串的列的更新，这种场景下使用压缩也是可以接受的。

以上说了那么多，压缩最终还是通过消耗更多的 cpu 资源来换取减少 IO 消耗，最终带来性能的提升，如果应用是 IO 密集型，而不是 cpu 密集型，那么可以利用剩余的 cpu 来提升应用性能。

**InnoDB** **的压缩是怎么实现的呢？**其实，截止目前 InnoDB 已经包括两种压缩模式，一种是这里提到的对记录（包括索引）的压缩。该压缩从 MySQL 5.1 开始既已存在；另一种是在 MySQL 5.7 中加入的，是对数据块的压缩，本文不对其做介绍，有兴趣的同学可以查看 [http://ks.netease.com/blog?id=4540](http://ks.netease.com/blog?id=4540) 及后续分析。

这里对记录压缩做简单介绍：InnoDB 启用压缩后，btr_page_create 会调用 page_create_zip 从磁盘分配一张空的大小为 zip_size（即 key_block_size）的压缩页，该页在 buffer pool（bp）中驻留，不断接收插入及更新等操作的数据，InnoDB 的 flush 或 checkpoint 机制将该页数据写回磁盘，在回刷前会调用 page_zip_compress 进行压缩后，其实也不会每次写入磁盘都会进行压缩，就像之前问答中已经提到的，InnoDB 压缩页中开辟有 modification log 区，压缩页中一段时间内的记录增删改会先写入该区，再通过一定条件触发合并，合并的过程就是将原来的压缩数据解压，再将 modification log 区的内容合并掉，重新进行压缩。随着数据的不断写入，到了一个时间点，总会出现压缩后的数据大小仍超过了设定的 key_block_size，这种场景下会触发压缩页的分裂，此时就需要分配一个新页，将原页部分数据移到新页中，再分别进行压缩。如此，周而复始。

在内存充足的情况下，InnoDB bp 中包括压缩页的压缩状态 (buf_page_t*)page->zip.data 和解压状态 (buf_block_t*)page->frame。当内存不足时，InnoDB 可能将解压的 frame 回收，保留压缩的 zip.data 在 bp 中。如果一个压缩页在一段时间内没有被使用，压缩格式的 zip.data 也会被写回到磁盘中，以释放内存。

Innodb 使用一个自适应 LRU 算法来维持 bf 内 zip.data 和 frame 的平衡，目的是为了避免在 cpu 繁忙时减少解压页的开销，当 cpu 富余时避免过多的 IO。当系统是 IO 密集型时，倾向于回收 frame，以留下内存给其他从磁盘读入的页。当 cpu 密集型时，InnoDB 选择全部回收 zip.data 和 frame，也就是回收整个压缩页，这样内存可以更多的留给热点数据，并减少内存中需要解压的页。

**最后，如何逼格更高得使用 InnoDB 压缩呢？** InnoDB 一如既往得友好，information_schema 中提供了两组系统表来查看运行时的压缩行为。

系统表 innodb_cmp 和 innodb_cmp_reset 用来分析运行中的压缩状态，包括压缩页的压缩 / 解压次数，所花费时间，压缩成功次数以及压缩页的大小等。其中 innodb_cmp 保存历史汇总数据，而 innodb_cmp_reset 则记录的是一个较为实时的统计值，表结构如下：

mysql> show create table innodb_cmp\G

*************************** 1. row ***************************

Table: INNODB_CMP

Create Table: CREATE TEMPORARY TABLE `INNODB_CMP` (

`page_size` int(5) NOT NULL DEFAULT '0',

`compress_ops` int(11) NOT NULL DEFAULT '0',

`compress_ops_ok` int(11) NOT NULL DEFAULT '0',

`compress_time` int(11) NOT NULL DEFAULT '0',

`uncompress_ops` int(11) NOT NULL DEFAULT '0',

`uncompress_time` int(11) NOT NULL DEFAULT '0'

) ENGINE=MEMORY DEFAULT CHARSET=utf8

1 row in set (0.00 sec)

mysql> show create table innodb_cmp_reset\G

*************************** 1. row ***************************

Table: INNODB_CMP_RESET

Create Table: CREATE TEMPORARY TABLE `INNODB_CMP_RESET` (

`page_size` int(5) NOT NULL DEFAULT '0',

`compress_ops` int(11) NOT NULL DEFAULT '0',

`compress_ops_ok` int(11) NOT NULL DEFAULT '0',

`compress_time` int(11) NOT NULL DEFAULT '0',

`uncompress_ops` int(11) NOT NULL DEFAULT '0',

`uncompress_time` int(11) NOT NULL DEFAULT '0'

) ENGINE=MEMORY DEFAULT CHARSET=utf8

1 row in set (0.00 sec)

如果 compress_ops_ok/compress_ops 的比率很高的话，则表明系统运行的良好，反之则表明 InnoDB 经常进行 reorganize, recompress 和 split，此时对该表禁用压缩或者选择更大的 key_block_size。

InnoDB bp 使用伙伴分配系统（buddy allocator）来分配不同大小的内存页用来缓存压缩数据：1KB 到 16KB（即对应不同的 key_block_size），information_schema 为此提供了 innodb_cmpmem 和 innodb_cmpmem_reset 来记录压缩页使用 bp 的一些信息，包括：bp 为压缩页分配的正在使用的页数，可供使用的空闲页数，伙伴系统重新分配页的次数及时间等，详细表结构如下：

mysql> show create table innodb_cmpmem\G

*************************** 1. row ***************************

Table: INNODB_CMPMEM

Create Table: CREATE TEMPORARY TABLE `INNODB_CMPMEM` (

`page_size` int(5) NOT NULL DEFAULT '0',

`buffer_pool_instance` int(11) NOT NULL DEFAULT '0',

`pages_used` int(11) NOT NULL DEFAULT '0',

`pages_free` int(11) NOT NULL DEFAULT '0',

`relocation_ops` bigint(21) NOT NULL DEFAULT '0',

`relocation_time` int(11) NOT NULL DEFAULT '0'

) ENGINE=MEMORY DEFAULT CHARSET=utf8

1 row in set (0.00 sec)

mysql> show create table innodb_cmpmem_reset\G

*************************** 1. row ***************************

Table: INNODB_CMPMEM_RESET

Create Table: CREATE TEMPORARY TABLE `INNODB_CMPMEM_RESET` (

`page_size` int(5) NOT NULL DEFAULT '0',

`buffer_pool_instance` int(11) NOT NULL DEFAULT '0',

`pages_used` int(11) NOT NULL DEFAULT '0',

`pages_free` int(11) NOT NULL DEFAULT '0',

`relocation_ops` bigint(21) NOT NULL DEFAULT '0',

`relocation_time` int(11) NOT NULL DEFAULT '0'

) ENGINE=MEMORY DEFAULT CHARSET=utf8

1 row in set (0.00 sec)

在使用 InnoDB 压缩时，周期性观察 information_schema 中提供的这两组压缩相关的临时表，对于评估压缩的效果和性能非常有参考价值。