> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/gaosong0623/article/details/130838823)

#### **前言：**

 最近用到了 Elasticsearch+kibana+IK 分词器，但是基本上能搜到的所有教程、视频都用的是老版本的，奈何我又空有一身反骨，我就不爱用老版本的，结果就一个一个的踩坑。

 Elasticsearch 是我用过的最坑的软件之一了，我从来没有见过安装这么繁琐这么麻烦，平均报错三个起的软件。

 折腾了一天，搞的我直骂娘，差点儿就在淘宝上找外包安装这软件，要不是太贵（只装 Elasticsearch 就要 90RMB）我就装了……

 幸好我有一身反骨，我就不信自己搞不定，研究了一天头发都秃了终于搞定了，为了避免大家遇到跟我一样的坑，特意写个教程出来，就让我做个布道者扒~ 骄傲 ing~

       创作不易，麻烦多多点赞！！爱你们！！

#### **安装环境及版本：**

Elasticsearch-8.7.0-linux-x86_64

kibana8.7.0  Windows-x86_64

IK 分词器 8.7.0

安装前提：

**你的 Windows 电脑需要装好 xshell 和 xftp，不然安装会很麻烦的！！先把这俩生产力工具搞好！！不会装的随便搜一下，教程一大把。**

我的 Elasticsearch 和 IK 分词器是装在 Linux 虚拟机上的！kibana 是装在 Windows 系统上的！这样方便用！！

需要注意的是，**Elasticsearch 和 kibana 和 IK 分词器的版本必须都是相同的！！！不然就会报错！！**别问我怎么知道的！！他甚至连 0.1 的版本不同都兼容不了 ！！

![](https://img-blog.csdnimg.cn/73983579c7b64421b99a7645c5f707ac.png)

**以及，老版本和新版本的配置完全 tmd 不一样！在新版本用老版本的配置会报错！！别问我怎么知道的！！**

**正式开搞：**
---------

好了废话不多说了，我们正式开搞，先装 Elasticsearch，因为其他俩软件都是依赖他运行的。

### **Elasticsearch：**

先去他的官网下载个安装包：[Past Releases of Elastic Stack Software | Elastic](https://www.elastic.co/cn/downloads/past-releases#elasticsearch "Past Releases of Elastic Stack Software | Elastic")

 **选择跟我一样的 8.7.0！千万别下载最新的 8.7.1 因为 IK 分词器没有 8.7.1 版本的！！后续还得卸载了重新装！！**![](https://img-blog.csdnimg.cn/b46a6361d7de4a3086c6a4bba0aee929.png)​

 **这里需要注意的是，要先查询清楚你的 Linux 是啥内核再下载相应内核的版本！！**

**可以这样查询内核：**

> **uname -r**

![](https://img-blog.csdnimg.cn/d62a9d7ecb5c423882ec16bb88af15b8.png)​

#### 创建用户：

因为 Elasticsearch 这个变态的软件他不允许用 root 用户允许，所以我们需要先创建个用户：

创建用户：

> useradd **xx** **这里的 xx 替换成你的用户名**

设置密码：

> passwd **xx**  **这里的 xx 替换成你的密码，八位以上的！**

密码要输两遍！！

这里先用 root 用户安装，安好了再切换用户运行。

#### 上传安装包, 并解压

 打开你的 xftp，连上虚拟机，在 home 文件下先创建个文件夹，随便你叫什么！！

![](https://img-blog.csdnimg.cn/7c959e6693e64e4db626fa39086ed741.png)​

 然后把安装包上传到你刚创建的目录下！

**请注意，如果你的安装包下载正确，他的名称应该是跟我的一模一样的！！（如果你用的也是 Linux 系统）**

![](https://img-blog.csdnimg.cn/749cd4ff6ce74770bba74aa583c5b926.png)​

 在虚拟机上**切换到上传的目录下**进行解压缩：

> cd **/home/leyou**  **这里的路径需要替换成你的实际路径**

解压： 

> tar -zxvf elasticsearch-8.7.0-linux-x86_64.tar.gz

为了方便操作，把目录重命名：

> mv elasticsearch-8.7.0-linux-x86_64/ elasticsearch

 进入改完名的目录：

> cd elasticsearch

进入配置目录，修改必要的配置：

>  cd config

 好多博主教需要修改内存占用，我的建议是不需要，默认配置的内存一般够用，修多了容易报错，修少了也容易报错，我们这里就不修了！！（主打一个傻瓜式）

主要修改的配置文件：**elasticsearch.yml**

> vim elasticsearch.yml

**讲一些基础操作，按 I 开始修改！！** 

 修改数据和日志目录：

> path.data: **/home/leyou/elasticsearch/data** # 数据目录位置    
> path.logs: **/home/leyou/elasticsearch/logs** # 日志目录位置
> 
> 这里的路径替换成你的实际路径！！data 和 logs 文件夹没创建不要紧一会我们就创建！！

 修改绑定的 ip：

> **network.host: 0.0.0.0** # 绑定到 0.0.0.0，允许任何 ip 来访问

 默认只允许本机访问，修改为 0.0.0.0 后则可以远程访问

改一些没用的配置，主要就是关闭一些安全限制之类的，需要注意的是，**这些必须得改，要不然新版本运行会报错！！这都是我的血泪史！！我把要改动的地方都标红了，你们也****直接复制我的完整代码！！**

>   
> # Enable security features  
> xpack.security.enabled: **false**
> 
> xpack.security.enrollment.enabled: true
> 
> # Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents  
> xpack.security.http.ssl:  
>   enabled: **false**  
>   keystore.path: certs/http.p12
> 
> # Enable encryption and mutual authentication between cluster nodes  
> xpack.security.transport.ssl:  
>   enabled: true  
>   verification_mode: certificate  
>   keystore.path: certs/transport.p12  
>   truststore.path: certs/transport.p12  
> # Create a new cluster with the current node only  
> # Additional nodes can still join the cluster later  
> **cluster.initial_master_nodes: ["localhost.localdomain"]**

**千万别听信别人的加一些其他配置，新版本会报错！！不要瞎加！！别问我怎么知道的！都是自己踩过的坑！！**

改好了按 esc 退出编辑模式，输入: wq 然后再按回车保存。

返回上一层目录，创建需要的文件夹

> cd ..

> mkdir data  
> mkdir logs

 当然也可以用 xftp 操作。

![](https://img-blog.csdnimg.cn/1b24bb93d64044ea9758a3e07f47bfe3.png)

 为你的其他用户添加权限：

> vim /etc/security/limits.conf

添加下面的内容：

> * soft nofile 65536
> 
> * hard nofile 65536
> 
> * hard nofile 131072
> 
> * soft nproc 4096
> 
> * hard nproc 4096

上面的星号别搞丢了！！

给你的文件夹分配用户组：

> chown 你的用户名: 跟你用户名一样的用户组 elasticsearch

分配完了 ll 看一下。

给文件夹改权限：

> chmod -R 777 elasticsearch/

用 cd .. 返回上层目录，同样的操作对 home 目录再来一遍：

>  chown 你的用户名: 跟你用户名一样的用户组 home

> chmod -R 777 home/

都搞好了用 ll 检查一下，变绿了，用户组也变了就可以运行了。

![](https://img-blog.csdnimg.cn/f5e6cf9cc5d644ca8966475540942a95.png)

运行前把 xshell 关掉重启一下，确保配置生效！！

重启之后：

切换用户：

> su - xx  **这里的 xx 替换成你的用户名**

创建好了之后进入 bin 目录开始运行：

> cd **/home/leyou/elasticsearch/bin/ 这里的目录替换成你自己的。**

> ./elasticsearch

 如果你跟着我一步一步走，都没有报错，这里的运行基本上都会成功！！！！

![](https://img-blog.csdnimg.cn/1732606764114ce08d0840ebef12bd35.png)

 没有报错基本上就是启动成功了，可以再用电脑上的游览器检查一下：

> 打开相应的网页：[http://192.168.5.10:9200/](http://192.168.5.10:9200/ "http://192.168.5.10:9200/")

这里需要替换成你自己虚拟机的地址！！如果不知道的话可以用 ifconfig 命令查询！

![](https://img-blog.csdnimg.cn/cb9dc72812774c5ba01d89613db1e6e7.png)

后面的端口号是固定的！！

网页上显示这些就代表你启动成功了！

 ![](https://img-blog.csdnimg.cn/41ad30a20c804e35a93bff0cd4470251.png)

 事已至此 Elasticsearch 就安装成功啦！！我们已经完成 90% 了！

接下来我们安装 ik 分词器！！

### 安装 ik 分词器

首先下载：[https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v8.7.0](https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v8.7.0 "https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v8.7.0")

就下载 8.7.0 的，跟 elasticsearch 版本一样！github 有时候网页打不开，可以尝试一下魔法上网。

然后你是什么电脑就下载啥版本，我是 Windows 系统。

下载好了之后先别着急！！先解压，然后用 idea 打开这个代码包进行构建！！！构建完了之后再上传到虚拟机！！**别问我怎么知道的，不构建就是莫名其妙报错！！！！**

![](https://img-blog.csdnimg.cn/23b9572154a64bddaa28710024ece543.png)

 构建就是这个小按钮哈！！像个锤子一样的图标！！

构建好了 打开 xftp，在 / home/ **这里替换成你的文件夹** /elasticsearch/plugins 目录下创建一个 **analysis-ik** 文件夹，这里的文件夹名字必须跟我一模一样！！不然运行就会报错！！别问我怎么知道的！血泪史！！！

![](https://img-blog.csdnimg.cn/ac396d35523b41fbb95ec0deee888dc6.png)

 都搞好了之后，把代码包上传到 analysis-ik 目录下，由于我们刚刚已经解压了所以这里就不用解压了！！

然后退出 xshell 和 xftp，重启 elasticsearch：

> cd **/home/leyou/elasticsearch/bin/ 这里的目录替换成你自己的。**
> 
> ./elasticsearch

如果这里启动 elasticsearch 没有报错，就说明你的 ik 分词器运行成功了！！！

再打开刚刚的网页检查下：

![](https://img-blog.csdnimg.cn/98e0c6a5c9d24009afa11babbb80b626.png)

最后我们安装 kibana！！

### 安装 kibana

依然是通过官网下载！点这里然后选你的系统就 OK！！记住 kibana 是安在主机上的不是虚拟机上的，所以不是 mac 就是 Windows 系统！！

[Kibana 8.7.0 | Elastic](https://www.elastic.co/downloads/past-releases/kibana-8-7-0 "Kibana 8.7.0 | Elastic")

这个安装就非常非常简单了！！下载好了之后解压！！解压完了之后改配置文件：

![](https://img-blog.csdnimg.cn/492215e4a82748fca2c0a43b770b39d9.png)安装我下面的配置改： 

> ```
> server.host: "192.168.5.174" 这里需要改成你的主机ip
> elasticsearch.hosts: ["http://192.168.5.10:9200"]这里需要改成你的虚拟机ip
> 
> xpack.reporting.roles.enabled: false 这里是没用的配置但是不写就报错！
> 
> i18n.locale: "zh-CN" 这个配置用于显示中文的界面，如果你是个外国人你就当我没说，这个配置估计全网只有我一个人出了教程（我英语差） 哈哈哈哈
> ```

都配置好了之后保存！然后打开你主机电脑的 cmd，以管理员身份运行！

然后 cd 到你安装 kibana 的路径，然后输入 kibana.bat 运行！

**这里一定要以用 cmd 运行，不然如果报错了你都看不到就闪退了！！别问我怎么知道的！！血泪史！！**

**运行好了之后，就可以打开游览器检测是否成功了~~~**

**这里可以访问** [http://127.0.0.1:5601](http://127.0.0.1:5601 "http://127.0.0.1:5601") ，或者你刚刚配置的 **server.host:5601（server.host 就是你的主机 ip，比如 192.168.5.174）**，第一次启动比较慢，估计要等个几分钟才行，没有报错 error 就是启动成功~ 访问检查就 ok！

哦对了，差点忘了说，必须是 elasticsearch 启动的情况下 kibana 才可以正常运行！！没有启动 elasticsearch 先启动再启动 kibana！！！

正常运行之后显示的页面：

![](https://img-blog.csdnimg.cn/98a92ad9a7264bdf9c9bfcacf3327ff2.png)

 需要注意的是！！这才是新界面！好早之前应该就改版了！！但是愣是没见国内任何一个人用！！还得是我当布道者，哦对了，我不确定你们是不是显示的中文，如果不是的话，需要在高级设置那里改成中文哈！（让改中文的教程我估摸是第一个 哈哈哈哈哈）

设置 - Advanced Settings（高级设置）- 搜索 local，然后找到 chinese~~

![](https://img-blog.csdnimg.cn/b341c70c10d143b0ba8d9c8cd759beda.png)

 好啦，就这样，完结撒花！！！！记得点赞评论收藏鼓励三连~~ 爱你们！！