> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/shi_hong_fei_hei/article/details/115337684)

[docker 使用教程相关系列 目录](https://blog.csdn.net/shi_hong_fei_hei/article/details/113954640 "docker使用教程相关系列 目录")
----------------------------------------------------------------------------------------------------------

原博文没有截图操作，且有细微的问题。现已纠正重新整理，并重现问题，解决问题。

问题重现：
-----

开始接触 Docker 的朋友，可能会遇到这么一个问题，使用 [centos7 镜像](https://so.csdn.net/so/search?q=centos7%E9%95%9C%E5%83%8F&spm=1001.2101.3001.7020)创建容器后，在里面使用 systemctl 启动服务报错。针对这个报错，我们接下来就分析下！

```
# docker run -itd --name centos7 centos:7
 
# docker attach centos7
 
# yum install vsftpd
 
```

![](https://img-blog.csdnimg.cn/20210330222128441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoaV9ob25nX2ZlaV9oZWk=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210330222217734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoaV9ob25nX2ZlaV9oZWk=,size_16,color_FFFFFF,t_70)

```
# systemctl start vsftpd

```

 报错：Failed to get D-Bus connection: Operation not permitted

![](https://img-blog.csdnimg.cn/2021033022230996.png)

解决方案：
-----

先把原先创建的容器停止服务并移除

```
docker stop centos7
 
docker rm centos7
```

![](https://img-blog.csdnimg.cn/2021033022443569.png)

以特权模式运行容器。

```
docker run -d --name centos7 --privileged=true centos:7 /usr/sbin/init


```

![](https://img-blog.csdnimg.cn/20210330223101523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoaV9ob25nX2ZlaV9oZWk=,size_16,color_FFFFFF,t_70)

进入容器：

```
docker exec -it centos7 /bin/bash

```

```
# yum install vsftpd
 
# systemctl start vsftpd
```

![](https://img-blog.csdnimg.cn/20210330223414503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoaV9ob25nX2ZlaV9oZWk=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210330223933768.png)

分析原因：
-----

Docker 的设计理念是在容器里面不运行后台服务，容器本身就是宿主机上的一个独立的主进程，也可以间接的理解为就是容器里运行服务的应用进程。一个容器的生命周期是围绕这个主进程存在的，所以正确的使用容器方法是将里面的服务运行在前台。

再说到 systemd，这个套件已经成为主流 Linux 发行版（比如 CentOS7、Ubuntu14+）默认的服务管理，取代了传统的 SystemV 风格服务管理。systemd 维护系统服务程序，它需要特权去会访问 Linux 内核。而容器并不是一个完整的操作系统，只有一个文件系统，而且默认启动只是普通用户这样的权限访问 Linux 内核，也就是没有特权，所以自然就用不了！

因此，请遵守容器[设计原则](https://so.csdn.net/so/search?q=%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99&spm=1001.2101.3001.7020)，一个容器里运行一个前台服务！