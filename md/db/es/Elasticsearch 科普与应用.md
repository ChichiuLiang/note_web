> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7299493010136219698)

前言
==

本次主要目的是科普下 Elasticsearch（以下简称：ES），同时分享下目前自己使用到的 ES 的情况。

章一：什么是 ES
=========

简介：
---

Elasticsearch 是一个分布式、高扩展、高实时的搜索与数据分析引擎。它能很方便的使大量数据具有搜索、分析和探索的能力。充分利用 Elasticsearch 的水平伸缩性，能使数据在生产环境变得更有价值。Elasticsearch 的实现原理主要分为以下几个步骤，首先用户将数据提交到 Elasticsearch 数据库中，再通过分词控制器去将对应的语句分词，将其权重和分词结果一并存入数据，当用户搜索数据时候，再根据权重将结果排名，打分，再将返回结果呈现给用户。

Elasticsearch 是与名为 Logstash 的数据收集和日志解析引擎以及名为 Kibana 的分析和可视化平台一起开发的。这三个产品被设计成一个集成解决方案，称为 “Elastic Stack”（以前称为 “ELK stack”）。

Elasticsearch 可以用于搜索各种文档。它提供可扩展的搜索，具有接近实时的搜索，并支持多租户。Elasticsearch 是分布式的，这意味着索引可以被分成分片，每个分片可以有 0 个或多个副本。每个节点托管一个或多个分片，并充当协调器将操作委托给正确的分片。再平衡和路由是自动完成的。相关数据通常存储在同一个索引中，该索引由一个或多个主分片和零个或多个复制分片组成。一旦创建了索引，就不能更改主分片的数量。

Elasticsearch 使用 Lucene，并试图通过 JSON 和 Java API 提供其所有特性。它支持 facetting 和 percolating，如果新文档与注册查询匹配，这对于通知非常有用。另一个特性称为 “网关”，处理索引的长期持久性；例如，在服务器崩溃的情况下，可以从网关恢复索引。Elasticsearch 支持实时 GET 请求，适合作为 NoSQL 数据存储，但缺少分布式事务。 

ES 整体架构：
--------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d5bb62a9f66478988b0490c5735419c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1110&h=1271&s=143798&e=png&b=def1fb)

结构：
---

*   为方便理解，可以对照下 ES 和 MySQL 的关系：

ES 数据架构的主要概念与关系数据库 Mysql 对比表

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d272ce1c0a0a4594801bcb99f398a484~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=822&h=486&s=37738&e=png&b=fcfcfc)

Type 的概念，是适用于早期的 ES 版本，后续的 ES 版本，未使用到此概念。

章二：ES 中的核心概念
============

集群

一个集群，集群中有多个节点，其中有一个为主节点，这个主节点是可以通过选举产生的，主从节点是对于集群内部来说的。es 的一个概念就是去中心化，字面上理解就是无中心节点，这是对于集群外部来说的，因为从外部来看 es 集群，在逻辑上是个整体，你与任何一个节点的通信和与整个 es 集群通信是等价的。

节点

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c2ae19d879f44ebbdc03a231c30f6fd~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=922&h=894&s=48512&e=png&b=fbfbfb)

Elasticsearch 集群中服务分多个角色

### Master 节点：

负责集群内调度决策，集群状态、节点信息、索引映射、分片信息、路由信息，Master 真正主节点是通过选举诞生的，一般一个集群内至少要有三个 Master 可竞选成员，防止主节点损坏。

### Data 存储节点：

用于存储数据及计算，分片的主从副本，热数据节点，冷数据节点；

### Client 协调节点：

协调多个副本数据查询服务，聚合各个副本的返回结果，返回给客户端；

Elasticsearch 的两次查询:

1.  多节点及多分片能够提高系统的写性能，但是这会让数据分散在多个 Data 节点当中，Elasticsearch 并不知道我们要找的文档，到底保存在哪个分片的哪个 segment 文件中。
2.  所以, 为了均衡各个数据节点的性能压力，Elasticsearch 每次查询都是请求 所有 索引所在的 Data 节点，查询请求时协调节点会在相同数据分片多个副本中，随机选出一个节点发送查询请求，从而实现负载均衡。
3.  而收到请求的副本会根据关键词权重对结果先进行一次排序，当协调节点拿到所有副本返回的文档 ID 列表后，会再次对结果汇总排序，最后才会用 DocId 去各个副本 Fetch 具体的文档数据将结果返回。
4.  可以说，Elasticsearch 通过这个方式实现了所有分片的大数据集的全文检索，但这种方式也同时加大了 Elasticsearch 对数据查询请求的耗时。

### Kibana 计算节点：

作用是实时统计分析、聚合分析统计数据、图形聚合展示。

倒排索引

倒排索引是区别于正排索引的概念：

正排索引：是以文档对象的唯一 ID 作为索引，以文档内容作为记录。

倒排索引：Inverted index，指的是将文档内容中的单词作为索引，将包含该词的文档 ID 作为记录。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07fe7f5024ec45bf94c702566c8a330f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1080&h=428&s=85617&e=png&b=f9f9f9)

举例：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d781efdf70a74537a11499324c28cda0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=523&h=149&s=52474&e=png&b=fefdfd)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c464b8d281fb49aaaefa154be652be41~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=414&h=462&s=69592&e=png&b=fdfcfc)

### 倒排索引 - 组成

#### 单词词典（Term Dictionary）

单词词典的实现一般用 B + 树，B + 树构造的可视化过程网址:[B+ Tree Visualization](https://link.juejin.cn?target=https%3A%2F%2Fwww.cs.usfca.edu%2F~galles%2Fvisualization%2FBPlusTree.html "https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b778fb68fa5b46f4acd8cebc6196ab3e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=584&h=178&s=60044&e=png&b=fefdfc)

#### 倒排列表（Posting List）

倒排列表记录了单词对应的文档集合，有倒排索引项（Posting）组成

### 倒排索引项主要包含如下信息：

1.  文档 id 用于获取原始信息
2.  单词频率（TF，Term Frequency），记录该单词在该文档中出现的次数，用于后续相关性算分
3.  位置（Posting），记录单词在文档中的分词位置（多个），用于做词语搜索（Phrase Query）
4.  偏移（Offset），记录单词在文档的开始和结束位置，用于高亮显示

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42d9ffef592641dc9c0a4f7e1a9c0723~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=598&h=310&s=70930&e=png&b=fcfbfb)

### 倒排索引不可变的好处

*   不需要锁，提升并发能力，避免锁的问题
*   数据不变，一直保存在 os cache 中，只要 cache 内存足够
*   filter cache 一直驻留在内存，因为数据不变
*   可以压缩，节省 cpu 和 io 开销

分片

代表索引分片，es 可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上。构成分布式搜索。分片的数量只能在索引创建前指定，并且索引创建后不能更改。

副本

代表索引副本，es 可以设置多个索引的副本，副本的作用一是提高系统的容错性，当某个节点某个分片损坏或丢失时可以从副本中恢复。二是提高 es 的查询效率，es 会自动对搜索请求进行负载均衡。

分词

分词是将文本转换成一系列单词（Term or Token）的过程，也可以叫文本分析，在 ES 里面称为 Analysis

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65d9b091baa94a83a5f5f71fce33e37c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=636&h=285&s=63209&e=png&b=fffcfb)

### 分词器

分词器是 ES 中专门处理分词的组件，英文为 Analyzer，它的组成如下：

*   Character Filters：针对原始文本进行处理，比如去除 html 标签
*   Tokenizer：将原始文本按照一定规则切分为单词
*   Token Filters：针对 Tokenizer 处理的单词进行再加工，比如转小写、删除或增新等处理

### 分词器调用顺序

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18d90be35ec84276944e84dc78b67e6d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=652&h=408&s=106502&e=png&b=fefcfb)

### 预定义的分词器

#### Standard Analyzer

默认分词器

按词切分，支持多语言

小写处理

#### Simple Analyzer

按照非字母切分

小写处理

#### Whitespace Analyzer

空白字符作为分隔符

#### Stop Analyzer

相比 Simple Analyzer 多了去除请用词处理

停用词指语气助词等修饰性词语，如 the, an, 的， 这等

#### Keyword Analyzer

不分词，直接将输入作为一个单词输出

#### Pattern Analyzer

通过正则表达式自定义分隔符

默认是 \ W+，即非字词的符号作为分隔符

#### Language Analyzer 提供了 30 + 种常见语言的分词器

### 自定义分词

#### Character Filters

在 Tokenizer 之前对原始文本进行处理，比如增加、删除或替换字符等

自带的如下:

1.HTML Strip Character Filter：去除 HTML 标签和转换 HTML 实体

2.Mapping Character Filter：进行字符替换操作

3.Pattern Replace Character Filter：进行正则匹配替换

会影响后续 tokenizer 解析的 position 和 offset 信息

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afcea135c7b24390a2b2d95eb59ef033~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1467&h=378&s=41243&e=png&b=fdfdfd)

#### Tokenizers

将原始文本按照一定规则切分为单词（term or token）

自带的如下：

1.standard 按照单词进行分割

2.letter 按照非字符类进行分割

3.whitespace 按照空格进行分割

4.UAX URL Email 按照 standard 进行分割，但不会分割邮箱和 URL

5.Ngram 和 Edge NGram 连词分割

6.Path Hierarchy 按照文件路径进行分割

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3de9526550a48f182d1c69a83c9316d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1152&h=520&s=54011&e=png&b=fdfdfd)

#### Token Filters

*   对于 tokenizer 输出的单词（term）进行增加、删除、修改等操作
*   自带的如下：  
    1.lowercase 将所有 term 转为小写  
    2.stop 删除停用词  
    3.Ngram 和 Edge NGram 连词分割  
    4.Synonym 添加近义词的 term
*   ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b47ab4bb2554143a88dd005a57ac135~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1380&h=899&s=103218&e=png&b=fcfcfc)

自定义分词

自定义分词需要在索引配置中设定 char_filter、tokenizer、filter、analyzer 等

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f796d0ae2c554fbb9185ea7e5d30f7ca~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1340&h=564&s=75468&e=png&b=f9f9f9)

### 分词使用说明

分词会在如下两个时机使用：

#### 创建或更新文档时 (Index Time)，会对相应的文档进行分词处理

#### 查询时（Search Time），会对查询语句进行分词

1.  查询时通过 analyzer 指定分词器
    
2.  通过 index mapping 设置 search_analyzer 实现
    
3.  一般不需要特别指定查询时分词器，直接使用索引分词器即可，否则会出现无法匹配的情况
    
4.  分词使用建议    
    
    1.  明确字段是否需要分词，不需要分词的字段就将 type 设置为 keyword，可以节省空间和提高写性能
    2.  善用_analyze API，查看文档的分词结果

recovery

代表数据恢复或叫数据重新分布，es 在有节点加入或退出时会根据机器的负载对索引分片进行重新分配，挂掉的节点重新启动时也会进行数据恢复。

章三：ES 的常规操作（具体的查询问题以及优化下一期展开）
=============================

官方文档：

[www.elastic.co/guide/cn/el…](https://link.juejin.cn?target=https%3A%2F%2Fwww.elastic.co%2Fguide%2Fcn%2Felasticsearch%2Fguide%2Fcurrent%2Fforeword_id.html "https://www.elastic.co/guide/cn/elasticsearch/guide/current/foreword_id.html")（基于 Elasticsearch 2.x 版本，有些内容可能已经过时。）

索引写入流程图：
--------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/868375793ae84feeae0a1f432b435eb2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1908&h=1042&s=230177&e=png&b=fefdfd)

1.  客户端选择一个 node 发送请求过去，这个 node 就是 coordinating node (协调节点)
    
2.  coordinating node，对 document 进行路由，将请求转发给对应的 node
    
3.  实际上的 node 上的 primary shard 处理请求，然后将数据同步到 replica node
    
4.  coordinating node，如果发现 primary node 和所有的 replica node 都搞定之后，就会返回请求到客户端
    
5.  ES 使用了一个内存缓冲区 Buffer，先把要写入的数据放进 buffer；同时将数据写入 translog 日志文件（其实是些 os cache）。
    
6.  refresh：
    
    1.  buffer 数据满 / 1s 定时器到期会将 buffer 写入操作系统 segment file 中，进入 cache 立马就能搜索到，所以说 es 是近实时（NRT，near real-time）的
7.  flush：
    
    1.  tanslog 超过指定大小 / 30min 定时器到期会触发 commit 操作
        
        1.  将对应的 cache 刷到磁盘 file，
        2.  commit point 写入磁盘，commit point 里面包含对应的所有的 segment file
8.  translog 默认 5s 把 cache fsync 到磁盘，所以 es 宕机会有最大 5s 窗口的丢失数据
    

索引查询流程图：
--------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d25538d9547c44319f1a412128d4dada~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.png#?w=1918&h=886&s=175331&e=png&b=fefefe)

### 读取数据

1.  客户端发送任何一个请求到任意一个 node，成为 coordinate node
2.  coordinate node 对 document 进行路由，将请求 rr 轮训转发到对应的 node，在 primary shard 以及所有的 replica 中随机选择一个，让读请求负载均衡，
3.  接受请求的 node，返回 document 给 coordinate note
4.  coordinate node 返回给客户端

### 搜索过程

1.  客户端发送一个请求给 coordinate node
2.  协调节点将搜索的请求转发给所有的 shard 对应的 primary shard 或 replica shard
3.  query phase：每一个 shard 将自己搜索的结果（其实也就是一些唯一标识），返回给协调节点，有协调节点进行数据的合并，排序，分页等操作，产出最后的结果
4.  fetch phase ，接着由协调节点，根据唯一标识去各个节点进行拉去数据，最总返回给客户端

创建索引
----

```
PUT indexName1
{
    "mappings" : {
      "dynamic" : "strict",
      "properties" : {
        "advertiser_name" : {
          "type" : "object",
          "enabled" : false
        },
        "body" : {
          "type" : "text"
        },
        "cdn_url" : {
          "type" : "object",
          "enabled" : false
        },
        "created_at" : {
          "type" : "long"
        },
        "days" : {
          "type" : "short"
        },
        "domain" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword"
            }
          },
        },
        "heat_degree" : {
          "type" : "short"
        },
        "html_url" : {
          "type" : "object",
          "enabled" : false
        },
        "interaction" : {
          "type" : "long"
        },
        "message" : {
          "type" : "text",
        },
        "page_logo" : {
          "type" : "object",
          "enabled" : false
        },
        "sub_domain" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword"
            }
          },
        }
      }
    },
    "settings" : {
      "index" : {
        "refresh_interval" : "300s",
        "indexing" : {
          "slowlog" : {
            "level" : "info",
            "threshold" : {
              "index" : {
                "warn" : "200ms",
                "trace" : "20ms",
                "debug" : "50ms",
                "info" : "100ms"
              }
            },
            "source" : "1000"
          }
        },
        "unassigned" : {
          "node_left" : {
            "delayed_timeout" : "5m"
          }
        },
        "analysis" : {
          "analyzer" : {
            "zf_analyzer" : {
              "filter" : [
                "lowercase",
                "stop",
                "snowball"
              ],
              "char_filter" : [
                "my_mapping"
              ],
              "type" : "custom",
              "tokenizer" : "standard"
            }
          },
          "char_filter" : {
            "my_mapping" : {
              "type" : "mapping",
              "mappings" : [
                ".=> @",
                ":=> @"
              ]
            }
          }
        },
        "priority" : "100",
        "number_of_replicas" : "1",
        "routing" : {
          "allocation" : {
            "require" : {
              "box_type" : "hot"
            }
          }
        },
        "search" : {
          "slowlog" : {
            "level" : "info",
            "threshold" : {
              "fetch" : {
                "warn" : "200ms",
                "trace" : "50ms",
                "debug" : "80ms",
                "info" : "100ms"
              },
              "query" : {
                "warn" : "500ms",
                "trace" : "50ms",
                "debug" : "100ms",
                "info" : "200ms"
              }
            }
          }
        },
        "number_of_shards" : "12"
      }
    }
}
```

打开 / 关闭索引
---------

```
POST indexName1/_close
POST indexName1,indexName2/_close

POST indexName1/_open
POST indexName1,indexName2/_open
```

设置索引刷新时间
--------

```
PUT indexName1/_settings
{
  "refresh_interval": "90s"
}
```

### 查询个数

```
GET indexName1/_count
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "sales": {
            "gte": 1000
          }
        }
      }
    }
  }
}
```

### 查询所有

```
//查询所有
GET /indexName1/_doc/_search
{
  "query":{
    "match_all":{}
  }
}
```

### 查询不区分大小写

```
//新建索引，字段不区分大小写
PUT indexName1
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "store_name": {
        "type": "keyword",
        "normalizer": "my_normalizer"
      },
      "store_id": {
        "type": "keyword"
      }
    }
  },
  "settings": {
    "index": {
      "search": {
        "slowlog": {
          "level": "info",
          "threshold": {
            "fetch": {
              "warn": "200ms",
              "trace": "50ms",
              "debug": "80ms",
              "info": "100ms"
            },
            "query": {
              "warn": "500ms",
              "trace": "50ms",
              "debug": "100ms",
              "info": "200ms"
            }
          }
        }
      },
      "refresh_interval": "10s",
      "indexing": {
        "slowlog": {
          "level": "info",
          "threshold": {
            "index": {
              "warn": "200ms",
              "trace": "20ms",
              "debug": "50ms",
              "info": "100ms"
            }
          },
          "source": "1000"
        }
      },
      "analysis": {
        "normalizer": {
          "my_normalizer": {
            "filter": "lowercase",
            "type": "custom"
          }
        }
      },
      "number_of_shards": "1",
      "unassigned": {
        "node_left": {
          "delayed_timeout": "5m"
        }
      },
      "number_of_replicas": "0"
    }
  }
}



//插入数据


PUT /indexName1/_doc/1
{
    "store_name":"HighSteel",
    "store_id":"com.google.highsteel"
    
}

PUT /indexName1/_doc/2
{
    "store_name":"KKKKKKOOOOO",
    "store_id":"com.google.highsteel"
    
}

//查询所有
GET /indexName1/_doc/_search
{
  "query":{
    "match_all":{}
  }
}

//模糊查询，不区分大小写
GET indexName1/_search
{
  "query":{
        "wildcard" : {
            "store_name": "*KKkkkKOooOO*"
        }
  }
}
```

章四：ES 的优化项
==========

设置分片数
-----

```
#创建索引时指定，指定后，不能进行修改
"number_of_shards" : "12"
```

副本数
---

```
#设置副本数
PUT /indexName1/_settings
{
    "number_of_replicas": 1
}
```

内存
--

内存不用超过节点内存的一半，另外一半给 Lucene 使用。

冷热节点
----

```
PUT indexName1/_settings
{ 
	  "index.routing.allocation.require.box_type": "warm"
}
```

索引模版，索引生命周期
-----------

*   索引生命周期管理策略：[help.aliyun.com/document_de…](https://link.juejin.cn?target=https%3A%2F%2Fhelp.aliyun.com%2Fdocument_detail%2F178307.html "https://help.aliyun.com/document_detail/178307.html")
    
*   可以配置按大小或者按时间 (天) 来生成新的索引，索引名称，自动 + 1。旧索引自动切换为冷索引。新生成的索引为热索引。
    
    *   eg:index-01,index-02

查询索引信息
------

返回的统计信息和 节点统计 的输出很相似：search 、 fetch 、 get 、 index 、 bulk 、 segment counts 等等。

```
GET indexName1/_stats
```

segment 合并
----------

```
#查看集群上索引的删除条目的情况
GET _cat/indices/?v&s=docs.deleted:desc

#对某个索引，进行段合并
POST indexName1/_forcemerge?max_num_segments=1&only_expunge_deletes=true

#查看正在进行的forcemerge任务
GET _tasks?detailed=true&actions=*forcemerge&human

#取消任务
POST _tasks/J2FOnHkDSIGJ43AaTDniJQ:5929987436/_cancel
```

慢写入，慢查询
-------

### 慢查询

```
#查询条件
search_time_ms:[5000 TO *]
```

### 慢写入

```
#查询条件
index_time_ms:[200 TO *]
```

章五：ES 的注意事项
===========

索引字段不能修改，只能新增
-------------

segment 段合并，forcemerge, 磁盘空间问题考虑。
---------------------------------

fielddata 占用内存空间问题
------------------

```
indices.fielddata.memory_size_in_bytes

_cat/segments/indexname?v&h=shard,segment,size,size.memory

#fielddata具体字段占用
GET _cat/fielddata?v&s=size:desc

#fielddata具体索引占用
GET _cat/indices?v&h=index,fielddata.memory_size&s=fielddata.memory_size:desc
```

flush(sync_interval)
--------------------

1.  tanslog 超过指定大小 / 30min 定时器到期会触发 commit 操作
    
    1.  将对应的 cache 刷到磁盘 file，
    2.  commit point 写入磁盘，commit point 里面包含对应的所有的 segment file

refresh(refresh_interval)
-------------------------

*   buffer 数据满 / 1s 定时器到期会将 buffer 写入操作系统 segment file 中，进入 cache 立马就能搜索到，所以说 es 是近实时（NRT，near real-time）的
*   使用建议：对于实时性要求不高且想优化写入的业务场景，建议根据业务实际调大刷新频率。

修改索引非 readonly
--------------

```
PUT /indexName1/_settings
{
    "index.blocks.read_only_allow_delete": null
}
```

热索引、冷索引（自己定义，未用索引模版 + 索引生命周期管理）
-------------------------------

#### 冷热切换索引

```
PUT indexName1*/_settings
{ 
	  "index.routing.allocation.require.box_type": "warm"
}
```