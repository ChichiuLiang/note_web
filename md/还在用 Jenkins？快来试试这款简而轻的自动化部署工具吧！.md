> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/QBJpN5AFB6fiCmUcOGY-iw?poc_token=HGJZtGWjeU2N9N3dvtaSEZuKkOCno_FMgUzYdNdI)

点击 “IT 码徒”，关注，置顶公众号

每日技术干货，第一时间送达！

最近发现了一个比 Jenkins 使用更简单的项目构建和部署工具，完全可以满足个人以及一些小企业的需求，分享一下。

1

项目介绍

**Jpom** 是一款 Java 开发的简单轻量的低侵入式在线构建、自动部署、日常运维、项目监控软件。

日常开发中，Jpom 可以解决下面这些常见的痛点：

*   团队中没有专业的运维，开发还要做运维的活，需要自己手动构建、部署项目。
    
*   不同的项目有不同的构建、部署命令。
    
*   有开发、测试、生产等多环境打包的需求。
    
*   需要同时监控多个项目的运行状态。
    
*   需要下载 SSH 工具远程连接服务器。
    
*   需要下载 FTP 工具传输文件到服务器。
    
*   多台服务器时，在不同电脑之间账号密码同步不方便。
    
*   想使用一些自动化工具，但是对服务器性能太高，搭建太麻烦。
    
*   对自动化工具有个性化的需求，想自己修改项目，但是市面上的工具太复杂了。
    

2

功能特性

![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Ty3ldLIBSFLHEx9XEZ6hmXRzrsSCWMp7wa7T7IwAjbcU46OnVdibOCBpwic6dvojbs2Ft8Supzwaoxw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic&random=0.7405867797513952&random=0.8532751676850849&random=0.5616697346217521&random=0.34908896393503186)

*   节点管理：集群节点，统一管理多节点的项目，实现快速一键分发项目文件
    
*   项目管理：创建、启动、停止、实时查看项目控制台日志，管理项目文件
    
*   SSH 终端：在浏览器中执行 SSH 终端，方便进行日常运维, 记录执行命令记录
    
*   在线构建：在线拉取 GIT、SVN 仓库快速构建项目包，不用运维人员手动上传项目包
    
*   在线脚本：在线管理脚本、定时执行脚本、webhook 钩子执行、执行日志等
    
*   DOCKER 管理：在线管理镜像、容器、SWARM 集群。界面化管理 DOCKER
    
*   用户管理：多用户管理, 实现不同用户不同权限，用户操作、管理日志完善记录
    
*   项目监控：实时监控项目当前状态、如果异常自动触发邮件、钉钉报警通知
    
*   NGINX 配置、SSL 证书：在线快速方便的修改 NGINX 配置文件，SSL 证书统一管理
    

3

整体架构

![](https://mmbiz.qpic.cn/mmbiz_jpg/iaIdQfEric9Ty3ldLIBSFLHEx9XEZ6hmXRm7y43OJ91nuZl7LmibWlbx95L8Ze892ziaBgD5nGwq1DSwCgV6q2sffQ/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic&random=0.43391108929763966&random=0.04559647079645379&random=0.45596712342835133)

4

效果展示

*   演示地址：https://demo.jpom.top
    
*   账号：demo
    
*   密码：jpom666
    

**逻辑节点**

节点简单理解为服务器就可以，点击节点管理 > 逻辑节点 > 快速绑定，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Ty3ldLIBSFLHEx9XEZ6hmXRDicuZln2SDduve2uK5pMaArGgJoxvS4hfYoSubQKyKCJEXzSxvpgPLg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic&random=0.09560973761108937&random=0.6411160872223709)

**仓库信息**

需要构建的项目（仓库）信息，需要手动添加，构建支持 git 仓库的拉取。

![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Ty3ldLIBSFLHEx9XEZ6hmXRicbvTwtjziaWvBqOXAN3qB2ib1nmnqEWKuR7Ew1LIhVV8G8RQGJOhiaibJw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic&random=0.430824219073912&random=0.5425872032010031)

**构建列表**

构建列表这里展示了所有的构建的项目。

![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Ty3ldLIBSFLHEx9XEZ6hmXRnQRTKnYZtvySztVgm34CqsVQ8GtOD4QfOwrNxmWLTReO0PfPR6N9Zg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic&random=0.47100401605732745&random=0.02298380953381729)

**SSH 管理**

![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Ty3ldLIBSFLHEx9XEZ6hmXRN8XdV96ZA6JVK7UCrBhpxQyJrldajRricJGEuGTqqFUMD2UViabuxzoA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=wxpic&random=0.34547618714155015&random=0.5009052824455684)

5

安装使用

官方提供了多种安装方式，推荐使用「一键安装」，默认会安装到 /usr/local/jpom-server 目录。

```
# 一键默认安装 + 自动配置开机自启服务
curl -fsSL https://jpom.top/docs/install.sh | bash -s Server jdk+default+service

```

可以通过以下命令管理 Jpom 服务端：

*   启动：systemctl start jpom-server
    
*   停止：systemctl stop jpom-server
    
*   重启：systemctl restart jpom-server
    

启动成功后，服务端的端口为 2122，可通过 http://127.0.0.1:2122/ 访问管理页面（如果不是本机访问，需要把 127.0.0.1 换成你安装的服务器 IP 地址）。

如无法访问管理系统，执行命令 systemctl status firewalld 检查下是否开启了防火墙 ，如状态栏看到绿色显示 Active: active (running) 需要放行 2122 端口。

```
# 放行管理系统的 2122 端口
firewall-cmd --add-port=2122/tcp --permanent
# 重启防火墙才会生效
firewall-cmd --reload

```

如果在操作系统上放行了端口仍无法访问，并且你使用的是云服务器，请到云服务器后台中检查安全组规则是否放行 2122 端口。  

⚠️ 注意：Linux 系统中有多种防火墙：Firewall、Iptables、SELinux 等，再检查防火墙配置时候需要都检查一下。

更多 Jpom 服务端安装方式可以查看「安装 Jpom」。

6

相关地址

*   项目地址 : https://gitee.com/dromara/Jpom
    
*   官网 : https://jpom.top/
    

  

—END—

[【福利】2023 高薪课程，全面来袭（视频 + 笔记 + 源码）](http://mp.weixin.qq.com/s?__biz=MzU2OTMyMTAxNA==&mid=2247516711&idx=1&sn=e54d98dc55dd5dd2ff2c90935a60ae97&chksm=fc82b17ecbf53868aece859626913a43ba5b592398dcda1847e497df47632b180fb1fbd3a3d3&scene=21#wechat_redirect)
=========================================================================================================================================================================================================================================================

**PS：防止找不到本篇文章，可以收藏点赞，方便翻阅查找哦。**

  

往期推荐

  

  

[

去了一家不到 20 人的 IT 公司后，真的是大开眼界。。。



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503969&idx=1&sn=41dda738632d03484b2e64ea5cc9024e&chksm=ead01e60dda79776d610801078703ebc8a29ddadc0a043f2e648d129f23de917aaaa4f3fcf0d&scene=21#wechat_redirect)

[

缓存之美：如何选择合适的本地缓存？



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503956&idx=2&sn=ccdd7b7f247b23eb0946120e500f4eba&chksm=ead01e55dda7974354a0cf89a0015ae213a90a6b485856ea2f8e1dd0ad52177b885774364721&scene=21#wechat_redirect)

[

推荐一个灵活可配置的开源监控平台，功能非常强大！



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503946&idx=1&sn=16d121970a80dc5262919841cc4cc5b1&chksm=ead01e4bdda7975dac8e419cfac5bd3a5c8b485c026b9753e86bc79831c49574be982800fd2d&scene=21#wechat_redirect)

[

Spring Boot + Prometheus + Grafana 打造可视化监控，一目了然！



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503928&idx=1&sn=0fdc4240ff782b0915bef6b0cee3d8dc&chksm=ead01e39dda7972f31ee66ff56500002c7cd9500dd65e57a792fc48068eb32c17caa93559879&scene=21#wechat_redirect)

[

SpringBoot 自定义注解 + 反射实现 excel 导入的数据组装及字段校验



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503921&idx=1&sn=a69aedde41f697cae868e6dc2ac9a134&chksm=ead01e30dda79726f188feb3204ff4c895a6c2f1ed979a11662136d51737f8d70cd2c80ecd34&scene=21#wechat_redirect)

[

线上接口慢查询代码优化，性能提升 10 倍，这班加的值了



](http://mp.weixin.qq.com/s?__biz=MzI3MDM0MzAyMg==&mid=2247503906&idx=1&sn=7f96803a56973b5a9b20f8d4f041528f&chksm=ead01e23dda7973501ffbb0a70a8ebb7d306d2ac42018448d33a068ce0d72773542ab4803157&scene=21#wechat_redirect)