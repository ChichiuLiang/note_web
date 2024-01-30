> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/u_15162069/2873468)

> 千亿级 Mysql 数据迁移 mongodb 成本节省及性能优化实践，线上某 IOT 核心业务集群之前采用 mysql 作为主存储数据库，随着业务规模的不断增加，mysql 已无法满足海量数据存储需求，业务面临着容量痛点、成本痛点问题、数据不均衡问题等。

         线上某 IOT 核心业务集群之前采用 mysql 作为主存储数据库，随着业务规模的不断增加，mysql 已无法满足海量数据存储需求，业务面临着容量痛点、成本痛点问题、数据不均衡问题等。

       400 亿该业务迁移 mongodb 后，同样的数据节省了极大的内存、CPU、磁盘成本，同时完美解决了容量痛点、数据不均衡痛点，并且实现了一定的性能提升。此外，迁移时候的 mysql 数据为 400 亿，3 个月后的现在对应 mongodb 集群数据已增长到 1000 亿，如果以 1000 亿数据规模等比例计算成本，实际成本节省比例会更高。

       当前国内很多 mongod 文档资料、性能数据等还停留在早期的 MMAP_V1 存储引擎，实际上从 mongodb-3.x 版本开始，mongodb 默认存储引擎已经采用高性能、高压缩比、更小锁粒度的 wiredtiger 存储引擎，因此其性能、成本等优势相比之前的 MMAP_V1 存储引擎更加明显。

关于作者

       前滴滴出行专家工程师，现任 OPPO 文档数据库 mongodb 负责人，负责数万亿级数据量文档数据库 mongodb 内核研发、性能优化及运维工作，一直专注于分布式缓存、高性能服务端、数据库、中间件等相关研发。后续持续分享《MongoDB 内核源码设计、性能优化、最佳运维实践》，Github 账号地址:[https://github.com/y123456yz](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fy123456yz)

序言

       本文是 oschina 专栏《mongodb 源码实现、调优、最佳实践系列》的第 22 篇文章，其他文章可以参考如下链接：

[Qcon - 万亿级数据库 MongoDB 集群性能数十倍提升及机房多活容灾实践](https://my.oschina.net/u/4087916/blog/4907378)

[Qcon 现代数据架构 -《万亿级数据库 MongoDB 集群性能数十倍提升优化实践》核心 17 问详细解答](https://my.oschina.net/u/4087916/blog/4956301)

[百万级高并发 mongodb 集群性能数十倍提升优化实践 (上篇)](https://my.oschina.net/u/4087916/blog/3141909)

[百万级高并发 mongodb 集群性能数十倍提升优化实践 (下篇)](https://my.oschina.net/u/4087916/blog/3155205)

[Mongodb 特定场景性能数十倍提升优化实践 (记一次 mongodb 核心集群雪崩故障)](https://my.oschina.net/u/4087916/blog/4659464)

[常用高并发网络线程模型设计及 mongodb 线程模型优化实践](https://my.oschina.net/u/4087916/blog/4431422)

[为何要对开源 mongodb 数据库内核做二次开发](https://my.oschina.net/u/4087916/blog/3063529)

[盘点 2020 | 我要为分布式数据库 mongodb 在国内影响力提升及推广做点事](https://my.oschina.net/u/4087916/blog/4817081)

 [百万级代码量 mongodb 内核源码阅读经验分享](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fxie.infoq.cn%2Farticle%2Fhttps%3A%2Fmy.oschina.net%2Fu%2F4087916%2Fblog%2F46961047b2c1dc67de82972faac2812c)

[话题讨论 | mongodb 拥有十大核心优势，为何国内知名度远不如 mysql 高？](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fxie.infoq.cn%2Farticle%2F180d98535bfa0c3e71aff1662)

[Mongodb 网络模块源码实现及性能极致设计体验](https://my.oschina.net/u/4087916/blog/4295038)

[网络传输层模块实现二](https://my.oschina.net/u/4087916/blog/4674521)

[网络传输层模块实现三](https://my.oschina.net/u/4087916/blog/4678616)

[网络传输层模块实现四](https://my.oschina.net/u/4087916/blog/4685419)

[command 命令处理模块源码实现一](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fxie.infoq.cn%2Farticle%2Fed28fa03590e58fd88e987c82)

[command 命令处理模块源码实现二](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fxie.infoq.cn%2Farticle%2F6de3aef4978a5078da44a8228)

[mongodb 详细表级操作及详细时延统计实现原理 (快速定位表级时延抖动)](https://my.oschina.net/u/4087916/blog/4809012)

[[图、文、码配合分析]-Mongodb write 写 (增、删、改) 模块设计与实现](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fxie.infoq.cn%2Farticle%2F81d2214c49049f5d70e3eb448)

[Mongodb 集群搭建一篇就够了 - 复制集模式、分片模式、带认证、不带认证等 (带详细步骤说明)](https://my.oschina.net/u/4087916/blog/4661542)

[300 条数据变更引发的血案 - 记某十亿级核心 mongodb 集群部分请求不可用故障踩坑记](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fxie.infoq.cn%2Fedit%2F5932858d57db13d43a8b8d62a)

[记十亿级 Es 数据迁移 mongodb 成本节省及性能优化实践](https://my.oschina.net/u/4087916/blog/write)

[千亿级数据迁移 mongodb 成本节省及性能优化实践](https://my.oschina.net/u/4087916/blog/5065569)

1.  业务迁移背景

        该业务在迁移 mongodb 前已有约 400 亿数据，申请了 64 套 mysql 集群，由业务通过 shardingjdbc 做分库分表，提前拆分为 64 个库，每个库 100 张表。主从高可用选举通过依赖开源 orchestrator 组建，mysql 架构图如下图所示：

![](https://s2.51cto.com/images/blog/202106/02/5ce7a040539a4cb011aa728b0db12957.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

   说明：上图中红色代表磁盘告警，很多节点磁盘使用水位即将 100%。

        如上图所示，业务一年多前一次性申请了 64 套 MySQL 集群，单个集群节点数一主三从，每个节点规格如下：

*   cpu：4
*   mem：16G
*   磁盘：500G
*   总节点数：64*4=256
*   SSD 服务器

       该业务运行一年多时间后，总集群数据量达到了 400 亿，并以每月 200 亿速度增长，由于数据不均衡等原因，造成部分集群数据量大，持续性耗光磁盘问题。由于节点众多，越来越多的集群节点磁盘突破瓶颈，为了解决磁盘瓶颈，DBA 不停的提升节点磁盘容量。业务和 DBA 都面临严重痛点，主要如下：

*   数据不均衡问题
*   节点容量问题
*   成本持续性增加
*   DBA 工作量剧增 (部分磁盘提升不了需要迁移数据到新节点)，业务也提心吊胆

业务

2. 为何选择 mongodb - 附十大核心优势总结

       业务遇到瓶颈后，基于 mongodb 在公司已有的影响力，业务开始调研 mongodb，通过和业务接触了解到，业务使用场景都是普通的增、删、改、查、排序等操作，同时查询条件都比较固定，用 mongodb 完全没任何问题。

       此外，mongodb 相比传统开源数据库拥有如下核心优索：

*   优势一：模式自由

         mongodb 为 schema-free 结构，数据格式没有严格限制。业务数据结构比较固定，该功能业务不用，但是并不影响业务使用 mongodb 存储结构化的数据。

*   优势二：天然高可用支持

         mysql 高可用依赖第三方组件来实现高可用，mongodb 副本集内部多副本通过 raft 协议天然支持高可用，相比 mysql 减少了对第三方组件的依赖。

*   优势三：分布式 - 解决分库分表及海量数据存储痛点

       mongodb 是分布式数据库，完美解决 mysql 分库分表及海量数据存储痛点，业务无需在使用数据库前评估需要提前拆多少个库多少个表，mongodb 对业务来说就是一个无限大的表 (当前我司最大的表存储数千亿数据，查询性能无任何影响)。

       此外，业务在早期的时候一般数据都比较少，可以只申请一个分片 mongodb 集群。而如果采用 mysql，就和本次迁移的 IOT 业务一样，需要提前申请最大容量的集群，早期数据量少的时候严重浪费资源。

*   优势四：完善的数据均衡机制、不同分片策略、多种片建类型支持

       关于 balance：支持自动 balance、手动 balance、时间段任意配置 balance.

      关于分片策略：支持范围分片、hash 分片，同时支持预分片。

      关于片建类型：支持单自动片建、多字段片建

*   优势五：不同等级的数据一致性及安全性保证

       mongodb 在设计上根据不同一致性等级需求，支持不同类型的 Read Concern 、Write Concern 读写相关配置，客户端可以根据实际情况设置。此外，mongodb 内核设计拥有完善的 rollback 机制。

*   优势六：高并发、高性能

         为了适应大规模高并发业务读写，mongodb 在线程模型设计、并发控制、高性能存储引擎等方面做了很多细致化优化。

*   优势七：wiredtiger 高性能存储引擎设计

        网上很多评论还停留在早期 MMAPv1 存储引擎，相比 MMAPv1，wiredtiger 引擎性能更好，压缩比更高，锁粒度更小，具体如下：

*   WiredTiger 提供了低延迟和高吞吐量
*   处理比内存大得多的数据，而不会降低性能或资源
*   系统故障后可快速恢复到最近一个 checkpoint
*   支持 PB 级数据存储
*   多线程架构，尽力利用乐观锁并发控制算法减少锁操作
*   具有 hot-caches 能力
*   磁盘 IO 最大化利用，提升磁盘 IO 能力
*   其他

      更多 WT 存储引擎设计细节可以参考：

[http://source.wiredtiger.com/3.2.1/architecture.html](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fsource.wiredtiger.com%2F3.2.1%2Farchitecture.html)

*   优势八：成本节省 - WT 引擎高压缩比支持

       mongodb 对数据的压缩支持 snappy、zlib 算法，在以往线上真实的数据空间大小与真实磁盘空间消耗进行对比，可以得出以下结论：

*   mongodb 默认的 snappy 压缩算法压缩比约为 2.2-4.5 倍
*   zlib 压缩算法压缩比约为 4.5-7.5 倍 (本次迁移采用 zlib 高压缩算法)

![](https://s2.51cto.com/images/blog/202106/02/d9ff9923025b374c66e7fb226e99cc45.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

       此外，以线上已有的从 mysql、Es 迁移到 mongodb 的真实业务磁盘消耗统计对比，同样的数据，存储在 mongodb、Mysql、Es 的磁盘占比≈1：3.5：6，不同数据存储占比有差距。

*   优势九：天然 N 机房 (不管同城还是异地) 多活容灾支持

        mongodb 天然高可用机制及代理标签自动识别转发功能的支持，可以通过节点不同机房部署来满足同城和异地 N 机房多活容灾需求，从而实现成本、性能、一致性的 “三丰收”。更多机房多活容灾的案例详见 Qcon 分享：

[OPPO 万亿级文档数据库 MongoDB 集群性能优化实践](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fqcon.infoq.cn%2F2020%2Fshenzhen%2Ftrack%2F916)

*   优势十：完善的客户端均衡访问策略

      mongodb 客户端访问路由策略由客户端自己指定，该功能通过 Read Preference 实现，支持 primary 、primaryPreferred 、secondary 、secondaryPreferred 、nearest 五种客户端均衡访问策略。

*   分布式事务支持

       mongodb-4.2 版本开始已经支持分布式事务功能，当前对外文档版本已经迭代到 version-4.2.12，分布式事务功能也进一步增强。此外，从 mongodb-4.4 版本产品规划路线图可以看出，mongodb 官方将会持续投入开发查询能力和易用性增强功能，例如 union 多表联合查询、索引隐藏等

[mongodb 源码分析、更多实践案例细节](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fy123456yz%2Freading-and-annotate-mongodb-3.6)

2. mongodb 资源评估及部署架构

       业务开始迁移 mongodb 的时候，通过和业务对接梳理，该集群规模及业务需求总结如下：

*   已有数据量 400 亿左右
*   数据磁盘消耗总和 30T 左右
*   读写峰值流量 4-5W/s 左右，流量很小
*   同城两机房多活容灾
*   读写分离
*   每月预计增加 200 亿数据
*   满足几个月内 1500 亿新增数据需求

       说明：数据规模和磁盘消耗按照单副本计算，例如 mysql 64 个分片，256 个副本，数据规模和磁盘消耗计算方式为：64 个主节点数据量之和、64 个分片主节点磁盘消耗之和。

3.1 mongodb 资源评估
----------------

    分片数及存储节点套餐规格选定评估过程如下：

*   内存评估

        我司都是容器化部署，以往经验来看，mongodb 对内存消耗不高，历史百亿级以上 mongodb 集群单个容器最大内存基本上都是 64Gb，因此内存规格确定为 64G。

*   分片评估

       业务流量峰值 3-5W/s，考虑到可能后期有更大峰值流量，因此按照峰值 10W/s 写，5w/s 读，也就是峰值 15W/s 评估，预计需要 4 个分片。

*   磁盘评估

         mysql 中已有数据 400 亿，磁盘消耗 30T。按照以网线上迁移经验，mongodb 默认配置磁盘消耗约为 mysql 的 1/3-1/5，400 亿数据对应 mongodb 磁盘消耗预计 8T。考虑到 1500 亿数据，预计 4 个分片，按照每个分片 400 亿规模，预计每个分片磁盘消耗 8T。

        线上单台物理机 10 多 T 磁盘，几百 G 内存，几十个 CPU，为了最大化利用服务器资源，我们需要预留一部分磁盘给其他容器使用。另外，因为容器组套餐化限制，最终确定确定单个节点磁盘在 7T。预计 7T 节点，4 个分片存储约 1500 亿数据。

*   CPU 规格评估

由于容器调度套餐化限制，因此 CPU 只能限定为 16CPU(实际上用不了这么多 CPU)。

*   mongos 代理及 config server 规格评估

       此外，由于分片集群还有 mongos 代理和 config server 复制集，因此还需要评估 mongos 代理和 config server 节点规格。由于 config server 只主要存储路由相关元数据，因此对磁盘、CUP、MEM 消耗都很低；mongos 代理只做路由转发只消耗 CPU，因此对内存和磁盘消耗都不高。最终，为了最大化节省成本，我们决定让一个代理和一个 config server 复用同一个容器，容器规格如下：

8CPU/8G 内存 / 50G 磁盘，一个代理和一个 config server 节点复用同一个容器。

分片及存储节点规格总结：4 分片 / 16CPU、64G 内存、7T 磁盘。

mongos 及 config server 规格总结：8CPU/8G 内存 / 50G 磁盘

![](https://s2.51cto.com/images/blog/202106/02/1f2abaf6c990f27fd145e458c8601d28.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

3.2 集群部署架构
----------

         由于该业务所在城市只有两个机房，因此我们采用 2+2+1(2mongod+2mongod+1arbiter 模式)，在 A 机房部署 2 个 mongod 节点，B 机房部署 2 个 mongod 节点，C 机房部署一个最低规格的选举节点，如下图所示：

![](https://s2.51cto.com/images/blog/202106/02/74b608fc500c1f7a5e01f35aa8c67b40.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

说明：

*   每个机房代理部署 2 个 mongos 代理，保证业务访问代理高可用，任一代理挂掉，对应机房业务不受影响。
*   如果机房 A 挂掉，则机房 B 和机房 C 剩余 2mongod+1arbiter，则会在 B 机房 mongod 中从新选举一个主节点。arbiter 选举节点不消耗资源
*   客户端配置 nearest ，实现就近读，确保请求通过代理转发的时候，转发到最近网络时延节点，也就是同机房对应存储节点读取数据。
*   弊端：如果是异地机房，B 机房和 C 机房写存在跨机房写场景。A B 为同城机房，则没有该弊端，同城机房时延可以忽略。

4. 业务全量 + 增量迁移方式

![](https://s2.51cto.com/images/blog/202106/02/a98dcdc7e0fb146b778829a31b5fa8d9.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

        迁移过程由业务自己完成，通过阿里开源的 datax 工具实现，该迁移工具的更多细节可以参考：[https://github.com/alibaba/DataX](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Falibaba%2FDataX)

5. 性能优化过程

      该集群优化过程按照如下两个步骤优化：数据迁移开始前的提前预优化、迁移过程中瓶颈分析及优化、迁移完成后性能优化。

5.1 数据迁移开始前的提前预操作
-----------------

       和业务沟通确定，业务每条数据都携带有一个设备标识 ssoid，同时业务查询更新等都是根据 ssoid 维度查询该设备下面的单条或者一批数据，因此片建选择 ssoid。

*   分片方式

        为了充分散列数据到 4 个分片，因此选择 hash 分片方式，这样数据可以最大化散列，同时可以满足同一个 ssoid 数据落到同一个分片，保证查询效率。

*   预分片

      mongodb 如果分片片建为 hashed 分片，则可以提前做预分片，这样就可以保证数据写进来的时候比较均衡的写入多个分片。预分片的好处可以规避非预分片情况下的 chunk 迁移问题，最大化提升写入性能。

      sh.shardCollection("xxx.xxx", {ssoid:"hashed"}, false, { numInitialChunks: 8192} )

注意事项：切记提前对 ssoid 创建 hashed 索引，否则对后续分片扩容有影响。

*   就近读

     客户端增加 nearest 配置，从离自己最近的节点读，保证了读的性能。

*   mongos 代理配置

      A 机房业务只配置 A 机房的代理，B 机房业务只配置 B 机房代理，同时带上 nearest 配置，最大化的实现本机房就近读，同时避免客户端跨机房访问代理。

*   禁用 enableMajorityReadConcern

      禁用该功能后 ReadConcern majority 将会报错，ReadConcern majority 功能注意是避免脏读，和业务沟通业务没该需求，因此可以直接关闭。

     mongodb 默认使能了 enableMajorityReadConcern，该功能开启对性能有一定影响，参考：

[MongoDB readConcern 原理解析](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdeveloper.aliyun.com%2Farticle%2F60553)

[OPPO 百万级高并发 MongoDB 集群性能数十倍提升优化实践](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fmongoing.com%2Farchives%2F29934)

*   存储引擎 cacheSize 规格选择

          单个容器规格：16CPU、64G 内存、7T 磁盘，考虑到全量迁移过程中对内存压力，内存碎片等压力会比较大，为了避免 OOM，设置 cacheSize=42G。

5.2 数据全量迁移过程中优化过程
-----------------

![](https://s2.51cto.com/images/blog/202106/02/55b30b9f104d0ab0deb144befd657e82.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

        全量数据迁移过程中，迁移速度较块，内存脏数据较多，当脏数据比例达到一定比例后用户读写请求对应线程将会阻塞，用户线程也会去淘汰内存中的脏数据 page，最终写性能下降明显。

         wiredtiger 存储引擎 cache 淘汰策略相关的几个配置如下:

![](https://s2.51cto.com/images/blog/202106/02/f69f3c4405bc402c2491bfa8a07b33c8.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

     由于业务全量迁移数据是持续性的大流量写，而不是突发性的大流量写，因此 eviction_target、eviction_trigger、eviction_dirty_target、eviction_dirty_trigger 几个配置用处不大，这几个参数阀值只是在短时间突发流量情况下调整才有用。

      但是，在持续性长时间大流量写的情况下，我们可以通过提高 wiredtiger 存储引擎后台线程数来解决脏数据比例过高引起的用户请求阻塞问题，淘汰脏数据的任务最终交由 evict 模块后台线程来完成。

      全量大流量持续性写存储引擎优化如下：

db.adminCommand({ setParameter : 1, "wiredTigerEngineRuntimeConfig" : "eviction=(threads_min=4, threads_max=20)"})

5.3 全量迁移完成后，业务流量读写优化
--------------------

![](https://s2.51cto.com/images/blog/202106/02/0ed5e7658c9b1fb3aeed7ad76ee69744.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

      前面章节我们提到，在容器资源评估的时候，我们最终确定选择单个容器套餐规格为如下：

         16CPU、64G 内存、7T 磁盘。

      全量迁移过程中为了避免 OOM，预留了约 1/3 内存给 mongodb server 层、操作系统开销等，当全量数据迁移完后，业务写流量相比全量迁移过程小了很多，峰值读写 OPS 约 2-4W/s。

      也就是说，前量迁移完成后，cache 中脏数据比例几乎很少，基本上不会达到 20% 阀值，业务读流量相比之前多了很多 (数据迁移过程中读流量走原 mysql 集群)。为了提升读性能，因此做了如下性能调整 (提前建好索引)：

1.  节点 cacheSize 从之前的 42G 调整到 55G，尽量多的缓存热点数据到内存，供业务读，最大化提升读性能。
2.  每天凌晨低峰期做一次 cache 内存加速释放，避免 OOM。

        上面的内核优后后，业务测时延监控曲线变化，时延更加平稳，平均时延也有 25% 左右的性能优后，如下图所示：

![](https://s2.51cto.com/images/blog/202106/02/10b295954269b3d08c96bc8d127add14.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

6. 迁移前后，业务测时延统计对比 (Mysql vs mongodb)

6.1 性能收益对比
----------

*   迁移前业务测时延监控曲线 (平均时延 7ms,  2 月 1 日数据，此时 mysql 集群只有 300 亿数据)：

![](https://s2.51cto.com/images/blog/202106/02/28bbb706bbf93ccdd01e3be2c9fe12fe.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

*   迁移 mongodb 后并且业务流量全部切到 mongodb 后业务测时延监控曲线 (平均 6ms,  3 月 6 日数据，此时 mongodb 集群已有约 500 亿数据))

![](https://s2.51cto.com/images/blog/202106/02/c20cf5845e740367dcadfb5c284f695b.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

总结：

*   mysql(300 亿数据) 时延：约 7ms
*   mongodb(500 亿数据) 时延：约 6ms

6.2 性能质疑解答
----------

     该文有部分同学可能质疑性能数据，认为 mongodb 实例规格是 16CPU/64G 内存 / 7T 磁盘，而 mysql 是 4CPU/16G 内存 / 500G 磁盘。认为 mongodb 规格更高，而 mysql 资源规格低。但是忽略了单节点数据量和流量这个因素，按照单实例对比，总结如下 (由于只记录了 mysql 300 亿时候、mongodb 500 亿时候的业务测时延，因此还是以这两个时间点为例比较)：

     Mysql 和 mongodb 的 CPU 都不是瓶颈，都很空闲，两者之间容器规格唯一区别就是内存，单实例规格、数据量、业务测时延等对比总结 (单实例 mysql 数据量约 300/64=4.7 亿，mongodb 约 125 亿)：

![](https://s2.51cto.com/images/blog/202106/02/85738d77b2bef063ef4bed64cbbbcc53.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

       如果 mysql 采用 mongodb 同样的规格，由于 mysql 同样数据磁盘消耗是 mongodb 3.3 倍，因此需要 22T 左右磁盘，并且承担同样的数据量和流量，性能会不会好于方案 1？这个不是很确定，因为都是线上环境，不可能为了验证这个测试而大费周章。

      如上，方案 3 和方案 1、方案 2 的性能对比有待验证。实际上，mongodb 当前 4 个分片已经 1000 亿数据了，客户端访问时延基本上没有变化，还是约 6ms，因此实际上如果同等资源规格验证，客观数 mysql 单个节点需要承担如下数据量和业务流量：

![](https://s2.51cto.com/images/blog/202106/02/2355b06f8f44e9a2c4627a3677409661.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

7. 迁移成本收益对比

7.1 Mysql 集群规格及存储数据最大量
----------------------

![](https://s2.51cto.com/images/blog/202106/02/10ccfb35add4d181ea92e0f1ad25992e.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

       原 mysql 集群一共 64 套，每套集群 4 副本，每个副本容器规格：4CPU、16G mem、500G 磁盘，总共可以存储 400 亿数据，这时候大部分节点已经开始磁盘 90% 水位告警，DBA 对部分节点做了磁盘容量提升。

      总结如下：

*   集群总套数：64
*   单套集群副本数：4
*   每个节点规格：4CPU、16G mem、500G 磁盘
*   该 64 套集群最大存储数据量：400 亿

7.2 mongodb 集群规格及存储数据最大量

![](https://s2.51cto.com/images/blog/202106/02/1b12c052e2793b42644d6209abca2d91.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

        mongodb 从 mysql 迁移过来后，数据量已从 400 亿增加到 1000 亿，并以每个月增加 200 亿数据。mongodb 集群规格及存储数据量总结如下：

*   分片数：4
*   单分片副本数：4
*   每个节点规格：16CPU、64G mem、7T 磁盘
*   四个分片存储数据量：当前已存 1000 亿，最大可存 1500 亿数据。

7.3 成本对比计算过程
------------

       说明：由于 mysql 迁移 mongodb 后，数据不在往 mysql 中写入，流量切到 mongodb 时候 mysql 中大约存储有 400 亿数据，因此我们以这个时间点做为对比时间点。以 400 亿数据为基准，资源消耗对比如下表 (每个分片只计算主节点资源消耗，因为 mysql 和 mongodb 都是 4 副本)：

![](https://s2.51cto.com/images/blog/202106/02/1d198ecd4b284776fafebddb5db3fead.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

        由于 mongodb 四个分片还有很多磁盘冗余，该四个分片相比 400 亿数据，还可以写 1200 亿数据。如果按照 1600 亿数据计算，如果还是按照 mysql 之前套餐规格，则 mysql 集群数需要再增加三倍，也就是总集群套数需要 64*4=256 套，资源占用对比如下：

![](https://s2.51cto.com/images/blog/202106/02/e0805746f2e3c352064f4a49fedc6034.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

7.4 收益总结 (客观性对比)
----------------

       从上面的内容可以看出，该业务迁移 mongodb 后，除了解决了业务容量痛点、促进业务快速迭代开发、性能提升外，成本还节省了数倍。成本节省总结如下：

*   400 亿维度计算 (mysql 和 mongodb 都存储相同的 400 亿数据)：

     CPU 和内存成本比例：4:1

     磁盘成本比例：3.3:1

*   1500 亿维度计算 (mysql 集群都采用之前规格等比例换算)：

     CPU 和内存成本比例：16:1

     磁盘成本比例：3.3:1

      从上面的分析可以看出，数据量越大，按照等比例换算原则，mongodb 存储成本会更低，原因如下：

*   CPU / 内存节省原因：

     主要是因为 mongodb 海量数据存储及高性能原因，索引建好后，单实例单表即使几百亿数据，读写也是 ms 级返回 (注意：切记查询更新建好索引)。

此外，由于 mongodb 分布式功能，对容量评估更加方便，就无需提前一次性申请很多套 mysql，而是根据实际需要可以随时加分片。

*   磁盘节省原因：

      mongodb 存储引擎 wiredtiger 默认高压缩、高性能。

      最后，鉴于客观性成本评价，CPU / 内存成本部分可能会有争议，比如 mysql 内存和 CPU 是否申请的时候就申请过大。mongodb 对应 CPU 也同样存在该问题，例如申请的单个容器是 16CPU，实际上真实只消耗了几个 CPU。

     但是，磁盘节省是实时在在的，是相同数据情况下 mysql 和 mongodb 的真实磁盘消耗对比。

      当前该集群总数据量已经达到千亿级，并以每个月 200 亿规模增加，单从容器计费层面上换算，1000 亿数据按照等比例换算，预计可节省极大的成本。

8. 最后：千亿级中等规模 mongodb 集群注意事项

      mongodb 无需分库分表，单表可以无限大，但是单表随着数据量的增多会引起以下问题：

*   切记提前建好索引，否则影响查询更新性能 (数据越多，无索引查询扫描会越慢)。
*   切记提前评估好业务需要那些索引，单节点单个表数百亿数据，加索引执行时间较长。
*   服务器异常情况下节点替换时间相比会更长。
*   切记数据备份不要采用 mongodump/mongorestore 方式，而是采用热备或者文件拷贝方式备份。
*   节点替换尽量从备份中拷贝数据加载方式恢复，而不是通过主从全量同步方式，全量同步过程较长。

9. 未来挑战 (该集群未来万亿级实时数据规模挑战)

         随着时间推移，业务数据增长也会越来越多，单月数据量增长曲线预计会直线增加 (当前每月数据量增加 200 亿左右)，预计未来 2-3 年该集群总数据量会达到万亿级，分片数也会达到 20 个分片左右，可能会遇到各自各样的问题。

1.  但是，IOT 业务数据存在明显的冷数问题，一年前的数据用户基本上不会访问，因此我们考虑做如下优后来满足性能、成本的进一步提升：冷数据归档到低成本 SATA 盘
2.  冷数据提升压缩比，最大化减少磁盘消耗
3.  如何解决冷数据归档 sata 盘过程中的性能问题

      冷热归档存储可以参考之前在 Qcon、dbaplus、mongodb 中文社区分享的另一篇文章：

[用最少人力玩转万亿级数据，我用的就是 MongoDB！](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fdbaplus.cn%2Fnews-162-3666-1.html)

[mongodb 源码分析、更多实践案例细节](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fy123456yz%2Freading-and-annotate-mongodb-3.6)

10. 最后说明 (业务场景总结)

      本千亿级 IOT 业务使用场景总结如下：

*   本分享的业务数据读、更新、排序等都可以走索引，包括单字段索引、多字段索引、数组索引，所有查询和更新都能确定走具体的某个最优索引。    
*   查询都是单表查询，不涉及多表联合查询。

       数据库场景非常重要，脱离业务场景谈数据库优劣无任何意义。例如本文的业务场景，业务能确定需要建那些索引，同时所有的更新、查询、排序都可以对应具体的最优索引，因此该场景就非常适合 mongodb。