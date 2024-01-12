> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/631821119)

一、wireshark 是什么？
----------------

wireshark 是非常流行的网络封包分析软件，简称小鲨鱼，功能十分强大。可以截取各种网络封包，显示网络封包的详细信息。

wireshark 是开源软件，可以放心使用。可以运行在 Windows 和 Mac OS 上。对应的，linux 下的抓包工具是 tcpdump。使用 wireshark 的人必须了解网络协议，否则就看不懂 wireshark 了。

![](https://pic2.zhimg.com/v2-10567550ecfafeea4916c2a6473ecc19_r.jpg)

二、Wireshark 常用应用场景
------------------

1. 网络管理员会使用 wireshark 来检查网络问题

2. 软件测试工程师使用 wireshark 抓包，来分析自己测试的软件

3. 从事 socket 编程的工程师会用 wireshark 来调试

4. 运维人员用于日常工作，应急响应等等

总之跟网络相关的东西，都可能会用到 wireshark

三、Wireshark 抓包原理
----------------

Wireshark 使用 WinPCAP 作为接口，直接与网卡进行数据报文交换。

Wireshark 使用的环境大致分为两种，一种是电脑直连网络的单机环境，另外一种就是应用比较多的网络环境，即连接交换机的情况。

「单机情况」下，Wireshark 直接抓取本机网卡的网络流量；

「交换机情况」下，Wireshark 通过端口镜像、ARP 欺骗等方式获取局域网中的网络流量。

端口镜像：利用交换机的接口，将局域网的网络流量转发到指定电脑的网卡上。

ARP 欺骗：交换机根据 MAC 地址转发数据，伪装其他终端的 MAC 地址，从而获取局域网的网络流量。

> 下面是给大家分享的一些网络安全学习资料，需要的下方免费领取哦

[网络安全大礼包：《黑客 & 网络安全入门 & 进阶学习资源包》免费分享](https://mp.weixin.qq.com/s/fCj88Pdb8V5mWpDjkhDAnw)

四、Wireshark 软件安装
----------------

软件下载路径：

[https://www.wireshark.org/](https://www.wireshark.org/)

![](https://pic1.zhimg.com/v2-82e997fe5d92273c102bbcc804dbf7dc_r.jpg)

按照系统版本选择下载，下载完成后，按照软件提示一路 Next 安装。

![](https://pic4.zhimg.com/v2-6ab898e95582424b51944decb7fb25eb_r.jpg)

五、Wireshark 抓包示例
----------------

先介绍一个使用 wireshark 工具抓取 ping 命令操作的示例，可以上手操作感受一下抓包的具体过程。

1、打开 wireshark，主界面如下：

![](https://pic2.zhimg.com/v2-5df88c56ddd3300d49ea4a59762e31a5_r.jpg)

2、选择菜单栏上 捕获 -> 选项，勾选 WLAN 网卡。这里需要根据各自电脑网卡使用情况选择，简单的办法可以看使用的 IP 对应的网卡。点击 Start，启动抓包。

![](https://pic4.zhimg.com/v2-f41c9cb96ff9163b41896572b1ef5cb3_r.jpg)

3、wireshark 启动后，wireshark 处于抓包状态中。

![](https://pic4.zhimg.com/v2-0e7974164c696ecc2d2d03f5e7f75517_r.jpg)

4、执行需要抓包的操作，如在 cmd 窗口下执行 ping [http://www.baidu.com](http://www.baidu.com)。

![](https://pic2.zhimg.com/v2-2f0e847722ff4c547836c1b4bc907e4d_r.jpg)

5、操作完成后相关数据包就抓取到了，可以点击 停止捕获分组 按钮。

![](https://pic4.zhimg.com/v2-0e7974164c696ecc2d2d03f5e7f75517_r.jpg)

6、为避免其他无用的数据包影响分析，可以通过在过滤栏设置过滤条件进行数据包列表过滤，获取结果如下。说明：ip.addr == 183.232.231.172 and icmp 表示只显示 ICPM 协议且主机 IP 为 183.232.231.172 的数据包。说明：协议名称 icmp 要小写。

![](https://pic4.zhimg.com/v2-635833beda1f30e0fe68c8f151611f0b_r.jpg)

7、wireshark 抓包完成，并把本次抓包或者分析的结果进行保存，就这么简单。关于 wireshark 显示过滤条件、抓包过滤条件、以及如何查看数据包中的详细内容在后面介绍。

![](https://pic3.zhimg.com/v2-51c6fba72e0422e9a7b45b9f5d4e7d3a_r.jpg)

六、Wireshakr 抓包界面介绍
------------------

![](https://pic3.zhimg.com/v2-6660685f9a581135f2bc42b23597f82e_r.jpg)

Wireshark 的主界面包含 6 个部分：

**菜单栏**：用于调试、配置

**工具栏**：常用功能的快捷方式

**过滤栏**：指定过滤条件，过滤数据包

**数据包列表**：核心区域，每一行就是一个数据包

**数据包详情**：数据包的详细数据

**数据包字节**：数据包对应的字节流，二进制

说明：数据包列表区中不同的协议使用了不同的颜色区分。协议颜色标识定位在菜单栏 视图 --> 着色规则。如下所示

![](https://pic1.zhimg.com/v2-2a2697b8c2dfa4c56f928511d25c46dc_r.jpg)

WireShark 主要分为这几个界面

1. Display Filter(显示过滤器)
------------------------

用于设置过滤条件进行数据包列表过滤。菜单路径：分析 --> Display Filters。

![](https://pic4.zhimg.com/v2-8266e479bbe153fc89fb53f4e055abfb_r.jpg)

2. Packet List Pane(数据包列表)
--------------------------

显示捕获到的数据包，每个数据包包含编号，时间戳，源地址，目标地址，协议，长度，以及数据包信息。不同协议的数据包使用了不同的颜色区分显示。

![](https://pic3.zhimg.com/v2-122bc20608568a04f1e54c3ee2f8ff22_r.jpg)

3. Packet Details Pane(数据包详细信息)
-------------------------------

在数据包列表中选择指定数据包，在数据包详细信息中会显示数据包的所有详细信息内容。数据包详细信息面板是最重要的，用来查看协议中的每一个字段。各行信息分别为

（1）Frame: 物理层的数据帧概况

（2）Ethernet II: 数据链路层以太网帧头部信息

（3）Internet Protocol Version 4: 互联网层 IP 包头部信息

（4）Transmission Control Protocol: 传输层 T 的数据段头部信息，此处是 TCP

（5）Hypertext Transfer Protocol: 应用层的信息，此处是 HTTP 协议

![](https://pic1.zhimg.com/v2-281f25a0033865346bc7caaf91722a80_r.jpg)

TCP 包的具体内容

从下图可以看到 wireshark 捕获到的 TCP 包中的每个字段。

![](https://pic4.zhimg.com/v2-55813c018ea2d9ca1e9afbcabe63dfd7_r.jpg)![](https://pic4.zhimg.com/v2-e5fcab37c8ee8aaa97b392b7cbd0b087_r.jpg)

4. Dissector Pane(数据包字节区)
-------------------------

报文原始内容。

![](https://pic2.zhimg.com/v2-6c91c426a973ad639c2a674481d5bfa1_r.jpg)

七、Wireshark 过滤器设置
-----------------

初学者使用 wireshark 时，将会得到大量的冗余数据包列表，以至于很难找到自己需要抓取的数据包部分。

wireshark 工具中自带了两种类型的过滤器，学会使用这两种过滤器会帮助我们在大量的数据中迅速找到我们需要的信息。

1. 抓包过滤器
--------

捕获过滤器的菜单栏路径为 捕获 --> 捕获过滤器。用于在抓取数据包前设置。

![](https://pic1.zhimg.com/v2-91fd0a3f8756cc318647c25d6573b8dc_r.jpg)

如何使用呢？设置如下。

![](https://pic1.zhimg.com/v2-951d8feddf3b4b25e9d0f7997aaea634_r.jpg)

ip host 183.232.231.172 表示只捕获主机 IP 为 183.232.231.172 的数据包。获取结果如下：

![](https://pic1.zhimg.com/v2-5c9eff9114600cd354a931d3854d87a8_r.jpg)

2. 显示过滤器
--------

显示过滤器是用于在抓取数据包后设置过滤条件进行过滤数据包。

通常是在抓取数据包时设置条件相对宽泛或者没有设置导致抓取的数据包内容较多时使用显示过滤器设置条件过滤以方便分析。

![](https://pic3.zhimg.com/v2-78a166b79586869c7f73182e6ffc5b7e_r.jpg)

同样上述场景，在捕获时未设置抓包过滤规则直接通过网卡进行抓取所有数据包。

![](https://pic1.zhimg.com/v2-199e3756ebd0f052a951bf3351b23c6c_r.jpg)

执行 ping [http://www.baidu.com](http://www.baidu.com) 获取的数据包列表如下

![](https://pic3.zhimg.com/v2-86797e7a59979ec3950bb7f915e8351e_r.jpg)

观察上述获取的数据包列表，含有大量的无效数据。这时可以通过设置显示器过滤条件进行提取分析信息。ip.addr == 183.232.231.172，并进行过滤。

![](https://pic4.zhimg.com/v2-aef01eacd3a8a2fd6d6b55a1c22ed42f_r.jpg)

上述介绍了抓包过滤器和显示过滤器的基本使用方法。在组网不复杂或者流量不大情况下，使用显示器过滤器进行抓包后处理就可以满足我们使用。下面介绍一下两者间的语法以及它们的区别。

八、wireshark 过滤器表达式的规则
---------------------

1. 抓包过滤器语法和实例
-------------

抓包过滤器类型 Type（host、net、port）、方向 Dir（src、dst）、协议 Proto（ether、ip、tcp、udp、http、icmp、ftp 等）、逻辑运算符（&& 与、|| 或、！非）

（1）协议过滤

比较简单，直接在抓包过滤框中直接输入协议名即可。

tcp，只显示 TCP 协议的数据包列表

http，只查看 HTTP 协议的数据包列表

icmp，只显示 ICMP 协议的数据包列表

（2）IP 过滤

host 192.168.1.104

src host 192.168.1.104

dst host 192.168.1.104

（3）端口过滤

port 80

src port 80

dst port 80

（4）逻辑运算符 && 与、|| 或、！非

src host 192.168.1.104 &&dst port 80 抓取主机地址为 192.168.1.80、目的端口为 80 的数据包

host 192.168.1.104 || host 192.168.1.102 抓取主机为 192.168.1.104 或者 192.168.1.102 的数据包

！broadcast 不抓取广播数据包

2. 显示过滤器语法和实例
-------------

（1）比较操作符

比较操作符有

== 等于、！= 不等于、> 大于、<小于、>= 大于等于、<= 小于等于

（2）协议过滤

比较简单，直接在 Filter 框中直接输入协议名即可。注意：协议名称需要输入小写。

tcp，只显示 TCP 协议的数据包列表

http，只查看 HTTP 协议的数据包列表

icmp，只显示 ICMP 协议的数据包列表

![](https://pic4.zhimg.com/v2-e44c45fef3b7527ad193d1c54eca5df3_r.jpg)

（3） ip 过滤

ip.src ==112.53.42.42 显示源地址为 112.53.42.42 的数据包列表

ip.dst==112.53.42.42, 显示目标地址为 112.53.42.42 的数据包列表

ip.addr == 112.53.42.42 显示源 IP 地址或目标 IP 地址为 112.53.42.42 的数据包列表

![](https://pic4.zhimg.com/v2-9796c9da8bf70e4afcf5e7f0edff4ac3_r.jpg)

（4）端口过滤

tcp.port ==80, 显示源主机或者目的主机端口为 80 的数据包列表。

tcp.srcport == 80, 只显示 TCP 协议的源主机端口为 80 的数据包列表。

tcp.dstport == 80，只显示 TCP 协议的目的主机端口为 80 的数据包列表。

![](https://pic3.zhimg.com/v2-a6e18ccc92746ec3c3024eb55c5023e2_r.jpg)

（5） http 模式过滤

http.request.method=="GET", 只显示 HTTP GET 方法的。

![](https://pic4.zhimg.com/v2-ab8bfd208cb61f250b4854d82b6202e7_r.jpg)

（6）逻辑运算符为 and/or/not

过滤多个条件组合时，使用 and/or。比如获取 IP 地址为 192.168.0.104 的 ICMP 数据包表达式为 ip.addr == 192.168.0.104 and icmp

![](https://pic1.zhimg.com/v2-36edeb6fba460a1d5f981a153c607fbc_r.jpg)

（7）按照数据包内容过滤

假设我要以 ICMP 层中的内容进行过滤，可以单击选中界面中的码流，在下方进行选中数据。

![](https://pic1.zhimg.com/v2-fc93706ca89fdf8e4fd9e3f8eb32b634_r.jpg)

右键单击选中后出现如下界面

![](https://pic4.zhimg.com/v2-de7c59a62bef261d7e0afc0dfdc3de67_r.jpg)

选中后在过滤器中显示如下

![](https://pic1.zhimg.com/v2-1904cc2d905fed692e613ea767e31ff8_r.jpg)

后面条件表达式就需要自己填写。如下我想过滤出 data 数据包中包含 "abcd" 内容的数据流。关键词是 contains，完整条件表达式为 data contains "abcd"

![](https://pic2.zhimg.com/v2-66624f21de3cf4ce3dec4e0d64963d89_r.jpg)

看到这， 基本上对 wireshak 有了初步了解。

3. 常见用显示过滤需求及其对应表达式
-------------------

**数据链路层**：

筛选 mac 地址为 04:f9:38:ad:13:26 的数据包

eth.src == 04:f9:38:ad:13:26

筛选源 mac 地址为 04:f9:38:ad:13:26 的数据包 ----

eth.src == 04:f9:38:ad:13:26

**网络层**：

筛选 ip 地址为 192.168.1.1 的数据包

ip.addr == 192.168.1.1

筛选 192.168.1.0 网段的数据

ip contains "192.168.1"

**传输层：**

筛选端口为 80 的数据包

tcp.port == 80

筛选 12345 端口和 80 端口之间的数据包

tcp.port == 12345 &&tcp.port == 80

筛选从 12345 端口到 80 端口的数据包

tcp.srcport == 12345 &&tcp.dstport == 80

**应用层**：

特别说明: http 中 http.request 表示请求头中的第一行（如 GET index.jsp HTTP/1.1） http.response 表示响应头中的第一行（如 HTTP/1.1 200 OK），其他头部都用 http.header_name 形式。

筛选 url 中包含. php 的 http 数据包

http.request.uri contains ".php"

筛选内容包含 username 的 http 数据包

http contains "username"

九、Wireshark 抓包分析 TCP 三次握手
-------------------------

1. TCP 三次握手连接建立过程
-----------------

Step1：客户端发送一个 SYN=1，ACK=0 标志的数据包给服务端，请求进行连接，这是第一次握手；

Step2：服务端收到请求并且允许连接的话，就会发送一个 SYN=1，ACK=1 标志的数据包给发送端，告诉它，可以通讯了，并且让客户端发送一个确认数据包，这是第二次握手；

Step3：服务端发送一个 SYN=0，ACK=1 的数据包给客户端端，告诉它连接已被确认，这就是第三次握手。TCP 连接建立，开始通讯。

![](https://pic2.zhimg.com/v2-40824425eb3ef63f73841cae15ecefad_r.jpg)

2. Wireshark 抓包获取访问指定服务端数据包
---------------------------

Step1：启动 wireshark 抓包，打开浏览器输入 [http://www.baidu.com](http://www.baidu.com)。

Step2：使用 ping [http://www.baidu.com](http://www.baidu.com) 获取 IP。

![](https://pic3.zhimg.com/v2-b9bfd6b8c414eeb25aed1fe00654134e_r.jpg)

Step3：输入过滤条件获取待分析数据包列表 ip.addr == 183.232.231.172

![](https://pic2.zhimg.com/v2-8da8884d3111e4efc2c08a0b894ef665_r.jpg)

图中可以看到 wireshark 截获到了三次握手的三个数据包。第四个包才是 HTTPS 的， 这说明 HTTPS 的确是使用 TCP 建立连接的。

**第一次握手数据包**

客户端发送一个 TCP，标志位为 SYN，序列号为 0， 代表客户端请求建立连接。

![](https://pic4.zhimg.com/v2-76c94415a9c1885e6a58030c2a5f18e7_r.jpg)

数据包的关键属性如下：

SYN ：标志位，表示请求建立连接

Seq = 0 ：初始建立连接值为 0，数据包的相对序列号从 0 开始，表示当前还没有发送数据

Ack =0：初始建立连接值为 0，已经收到包的数量，表示当前没有接收到数据

**第二次握手的数据包**

服务器发回确认包, 标志位为 SYN，ACK。将确认序号 (Acknowledgement Number) 字段 + 1，即 0+1=1。

![](https://pic2.zhimg.com/v2-5e10999f4d16cb9c8a89db1cf19d6bf5_r.jpg)

数据包的关键属性如下：

[SYN + ACK]: 标志位，同意建立连接，并回送 SYN+ACK

Seq = 0 ：初始建立值为 0，表示当前还没有发送数据

Ack = 1：表示当前端成功接收的数据位数，虽然客户端没有发送任何有效数据，确认号还是被加 1，因为包含 SYN 或 FIN 标志位。（并不会对有效数据的计数产生影响，因为含有 SYN 或 FIN 标志位的包并不携带有效数据）

**第三次握手的数据包**

客户端再次发送确认包 (ACK) SYN 标志位为 0，ACK 标志位为 1。并且把服务器发来 ACK 的序号字段 + 1，放在确定字段中发送给对方，并且在 Flag 段写 ACK 的 + 1：

![](https://pic3.zhimg.com/v2-49d8a5ce01007e746892326e861a9636_r.jpg)

数据包的关键属性如下：

ACK ：标志位，表示已经收到记录

Seq = 1 ：表示当前已经发送 1 个数据

Ack = 1 : 表示当前端成功接收的数据位数，虽然服务端没有发送任何有效数据，确认号还是被加 1，因为包含 SYN 或 FIN 标志位（并不会对有效数据的计数产生影响，因为含有 SYN 或 FIN 标志位的包并不携带有效数据)。

就这样通过了 TCP 三次握手，建立了连接。开始进行数据交互

![](https://pic3.zhimg.com/v2-d2f8b437bf8ed377e809cd338707a98a_r.jpg)

十、Wireshark 分析常用操作
------------------

调整数据包列表中时间戳显示格式。调整方法为 视图 --> 时间显示格式 --> 日期和时间。调整后格式如下：

![](https://pic4.zhimg.com/v2-e3ae25306486c14d4942963c63350563_r.jpg)

一般 Wireshark 软件也可以与各主流厂家的模拟器一起使用，更适合于项目准确配置。

十一、工具安装包
--------

### 最后在这里给小伙伴们准备了黑客 / 网络安全工具包合集，需要的可以下方免费领取哦

[网络安全大礼包：《黑客 & 网络安全入门 & 进阶学习资源包》免费分享](https://mp.weixin.qq.com/s/fCj88Pdb8V5mWpDjkhDAnw)![](https://pic2.zhimg.com/v2-34973a5efbd5cb598908ea1d30591e7d_r.jpg)