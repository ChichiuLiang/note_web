> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JbK7-t0hU_TZO4ODNTJS7Q)

  

背景

  

之前一直使用的是 Jenkins+Gitlab 作 CI/CD，Jenkins 是一款强大的持续集成 / 持续部署工具，广泛应用于软件开发和部署领域。但是当今互联网逐渐走向容器化， Jenkins 的一些弊端也凸显了出来：

**重量级：**Jenkins 功能十分齐全，几乎可以做所有的事情。但是这也是他的一个弊端，过于重量级，有时候往往一个小的修改需要改动许多地方，升级 \ 下载插件后需要进行重启等。

配置复杂：Jenkins 的配置相对比较复杂，尤其是在安装和配置插件时。有时候可能会遇到配置错误或兼容性问题，导致工具无法正常工作。

**初学者学习困难：**Jenkins 是一款功能强大的工具，包含了大量的特性和插件。对于初学者来说，可能会觉得它的学习曲线比较陡峭，需要花费大量的时间和精力去学习和掌握。

所以，在当今容器化时代，选择一个合适的工具来做 CICD 是一个比较重点关注的事，那么本文说的这个工具就是——Drone。  

  

Drone 简介

  

Drone 是一个基于 Go 语言开发的持续集成 / 持续部署（CI/CD）工具，由前 Google 员工于 2013 年开源。Drone 通过引入 pipline 的概念，将整个构建过程由多个阶段（stage）组成，每个阶段都是一个 Docker。这些阶段间可以通过共享宿主机的磁盘目录实现数据共享和缓存基础数据，以加速下次构建。此外，各阶段也可以共享宿主机的 Docker 环境，实现共享宿主机的 Docker 镜像，从而避免每次构建都重新拉取基础镜像，减少构建时间。

Drone 的优势在于其并发运行能力。多个构建可以并发运行，单机并发数量由服务器 CPU 数决定。此外，Drone 由开发者负责打包镜像和流程控制，无需运维参与在 CI 服务器上部署各种语言编译需要的环境。这使得 Drone 成为 DevOps 的最佳实践。

它主要具有以下方面优点：

**功能强大且灵活：**Drone 通过简单的 YAML 配置文件来定义和执行 Docker 容器中的管道，可以完成构建、测试、发布和部署等任务，具有高度的可配置性。

**兼容性好：**Drone 支持所有 SCM、所有平台和所有语言环境，可以轻松集成到现有的工作流程中。

**部署简单：**Drone 原生支持 Docker 容器，启动两个容器即可完成部署，其他构建、测试、部署工具在使用时会自动从 Docker 仓库拉取。

**扩展性强：**Drone 拥有强大的插件系统，丰富的插件可以免费使用，也可以自定义配置，满足各种扩展需求。

**维护简单：**Drone 直接复用 SCM 的账号体系和权限管理，无需注册用户、分配权限，大大简化了维护工作

  

Drone 自动部署

  

前置条件：服务器得先安装好 Docker（本文不讲述 Docker 的安装）  

配置 GitLab

  

  

在这里，使用 Gitlab 作为 Drone 的账号和权限管理，所以首先在你自己的 Gitlab 服务上面新增一个应用：  

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqhtP8btRBbuXiaOjiaaH2jPWYUsN9Wcd25wabQvDBBXQm8CTXB5s0loPQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqG7QQKSus4HCdWVhticAeOoZ7OyHFIWaHhFzfcVkYXPptCDeSP1QjA6w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

记录下此处的应用程序 id 及密码，后续安装 Drone 时会使用该 ID

安装 Drone

  

  

```
#创建共享密钥，用于drone服务和runner之间通信
[root@master application]# openssl rand -hex 16
b63a8848e42f3d96108e04a5fb99f41e
#安装Drone
[root@master application]# docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITLAB_SERVER=http://192.168.0.150:9090 \
  --env=DRONE_GITLAB_CLIENT_ID=8bae21f3d3c8b5a722ae1aa55c61eb6dbd14d9e1aca9a7f720fb638a3ca4c5ca \
  --env=DRONE_GITLAB_CLIENT_SECRET=b0927a462d5c19eae39216483ee7a0d83f8c31e5c7b2011c9bb6a8c976a3cd0b \
  --env=DRONE_RPC_SECRET=b63a8848e42f3d96108e04a5fb99f41e \
  --env=DRONE_SERVER_HOST=192.168.0.150:7070 \
  --env=DRONE_SERVER_PROTO=http \
  --env=DRONE_USER_CREATE=username:root,admin:true \
  --publish=7070:80 \
  --publish=8443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:2
Unable to find image 'drone/drone:2' locally
2: Pulling from drone/drone
79e9f2f55bf5: Pull complete 
21e4c61aedb1: Pull complete 
fb8c70c372db: Pull complete 
50712e01f221: Pull complete 
Digest: sha256:32c78ab006ff82cfe327088b763cf64e7435784fd1131284ebd5bcaf5ca5c43e
Status: Downloaded newer image for drone/drone:2
314b72c95949247f6ebec71c5d089dee87ab541100576d0789af415e86cc5ab4
#安装runner
[root@master application]# docker run --detach \
>   --volume=/var/run/docker.sock:/var/run/docker.sock \
>   --env=DRONE_RPC_PROTO=http \
>   --env=DRONE_RPC_HOST=192.168.0.150:7070 \
>   --env=DRONE_RPC_SECRET=b63a8848e42f3d96108e04a5fb99f41e \
>   --env=DRONE_RUNNER_CAPACITY=2 \
>   --env=DRONE_RUNNER_NAME=my-first-runner \
>   --publish=3000:3000 \
>   --restart=always \
>   --name=runner \
>   drone/drone-runner-docker:1
Unable to find image 'drone/drone-runner-docker:1' locally
1: Pulling from drone/drone-runner-docker
97518928ae5f: Pull complete 
4af047b71fe6: Pull complete 
de9b7e52f3f6: Pull complete 
679d1ec5770c: Pull complete 
Digest: sha256:70da970bb76a62567edbea1ac8002d9484664267f4cbb49fbd7c87a753d02260
Status: Downloaded newer image for drone/drone-runner-docker:1
d542e08d542678d37d9d588bb24d2916b575593c448a59c998631a1f25fc76b0

```

--env=DRONE_USER_CREATE=username:root,admin:true   这个必须要配置，标识 Drone 的管理员是哪个, 不然 drone 项目激活后设置页面出现不了 Trusted 选项。会操作不了 Drone 的设置。安装好后首页如图：

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqia6Ad4d6nrvriaYOf9iaoRgXwTVWOCr9qRt9swVE92yx83HlkibWxFLW3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

点击继续，会跳转到 gitlab 页面赋权：  

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqzOODNFUa00RARk5TMB9KIkiarnyianDEDbu8jmOtvoshLA7BK85Kwvpw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

赋权后会出现你有权限的代码分支，选择一个激活，然后在设置页勾选 Trust  

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqN6FX5kdUuFbRXq03zicO5GOh55IofU5egFDSlnXeRU33AwANOreNOibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqBpK3tVlUsq1mpiaYLO3H2fgbUCo9VVIv99ibXx9pyl0yxSaTtJ3ZE6cw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqmgBOy3QtwiaDuogTEGVmDYA25OYbVoBppumn2zPow10icoEomOBnm1Ng/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

做完设置以后，就可以到程序去配置 Drone 了  

  

程序配置 Drone  

  

  

在程序的根路径新增. drone.yml 文件

```
kind: pipeline
name: default
steps:
  - name: depend
    image: maven:3.8.1-jdk-8
    volumes:
      - name: cache
        path: /root/.m2
      - name: study-jar
        path: /root/study-jar
    commands:
      - mvn clean install
      - rm -rf /root/study-jar/*  &&  bash -x copy_to_host.sh
  - name: deploy
    image: appleboy/drone-ssh
    settings:
      host: 192.168.0.150
      port: 22
      command_timeout: 10m
      username: root
      password:
        from_secret: ssh_password
      script:
        - cd /home/study-data/study-jar/ && bash -x build.sh
volumes:
#maven本地仓库缓存路径
  - name: cache
    host:
      path: /root/maven/lib
#编译后lib及jar复制目标路径
  - name: study-jar
    host:
      path: /home/study-data/study-jar
trigger:
  branch:
    - dev

```

  

新增并且配置好后，在 dev 提交代码，就会执行 steps 里的步骤啦！提交代码后，再看 Drone 的页面，已经显示有代码提交上来。  

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqLJUYtJ6xgFWicNMfCuNA4qkWvMSnHGZ7S9R2HmANOvBJ7W4qwicOaa4g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/lvWQYp8ibHWJDHuPsUzlwyFia5PKicGONbqQD5UlCUWS7EYdudJVembwOsBSq0h9UrRiaH5gctYasfvRnFuYlUp76A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

尾言

  

Drone 提供了类似 jenkins 的插件支持，支持很多额外插件功能的使用，比如：发邮件、SSH、SonarQube 代码质量分析和管理等等。插件市场是应有尽有，而且配置简单没有 jenkins 那么复杂。有兴趣的朋友可以尝试去配置体验下，真的是非常之强大的！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/lvWQYp8ibHWLOribhNJ5E9XKmIlTRBuLZBJbBXNTL01iag5y7vYayicGG2TxH6ib7P9JUYAkXVibJMMj2ic5U0pibwxEfw/640?wx_fmt=png&from=appmsg)

官网地址：drone.io

文档地址：docs.drone.io

热文推荐

  

  

  

  

[4.3K star! 一个轻量优美的 Redis 客户端！有颜也有料！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247485283&idx=1&sn=a431054cbc6cf4237183d7eaafd0e36a&chksm=e950c5f3de274ce5a3ef89f290a55cb3507ec8da64688c7aeb3961d57f3310900ab418297e7f&scene=21#wechat_redirect)  
[Spring Boot 项目 Jar 包加密：防止反编译的安全实践!](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247485269&idx=1&sn=d086afa3d36c000340f3e1592b5798c3&chksm=e950c5c5de274cd3e2e7931cd29c8ad4f42cafec476bf94d15d989bfdf6fdc89d395bcf09341&scene=21#wechat_redirect)  
[编程界的 Goat：你认识哪些？有一个已离开人世!](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247485223&idx=1&sn=6489b447b689108ef5d08a4615a2ab60&chksm=e950c5b7de274ca18be281426b96bb04d31ab93c02cb0a148ba5865ef65ff46d43368b88f774&scene=21#wechat_redirect)  

[MYSQL 优化实战：15 招性能飞升，给力！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247485204&idx=1&sn=eca14df37c9e1740c6519bf8afa34a03&chksm=e950c584de274c92627b70241f0fa53f97eb65533b465bf6fba8e869c997797500630371bf3a&scene=21#wechat_redirect)  

[22.9K star！后端 & 运维必备神器，推荐！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247485197&idx=1&sn=73f92a0ab77d15c27b9e187f35583788&chksm=e950c59dde274c8b49412c696785242bd2eb0f89520b3839e08745a0660410e9ae7f67c9cc71&scene=21#wechat_redirect)  

[10 种实现定时任务的方式，你都知道哪些？都很强大！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247485179&idx=1&sn=cea9c3bf8164a3f8758aa90b2df4b1ca&chksm=e950c46bde274d7d9cd7978afbd8ef755a16a6da3e67a1593de8a0854242fdc00fbe65756a3f&scene=21#wechat_redirect)  

[1.6Kstar! 推荐一款 web 版 Linux、数据库、Redis、MongoDB 统一管理平台! 完美！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247485145&idx=1&sn=e848684dbc02ee4b964ec48e6445c043&chksm=e950c449de274d5ff3ef28d348deb807d475bc653aa0839a08e164158025ea2cc8f1ccfd4a9f&scene=21#wechat_redirect)  

[Nginx 实战：缓存、压缩、黑白名单。。常用 Nginx 运维指南！推荐！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247485140&idx=1&sn=c3f6b45424adfd936aaee42d62ecef70&chksm=e950c444de274d524e84fa17cf9c877a81716d5bcc551471368b5536399247ecebe1854454d4&scene=21#wechat_redirect)  

[36.9K star ! 推荐一个酷炫低代码开发平台！功能真强大！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247484905&idx=1&sn=3fcd1c04742bc74304f466ea6c182bf3&chksm=e950c779de274e6fff7f51fd1480f7e192bed5a38b42a73a6f7a5ba105db70bc95fbc2a30c67&scene=21#wechat_redirect)  

[前端：为什么总是后端研发当领导？原因扎心了！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247484590&idx=1&sn=9035ed0e4351028ef69018fbab734bf0&chksm=e950c63ede274f28457d07a872d85aa913e419ce9a16d9d337199190de20490b07a2a48ea3f8&scene=21#wechat_redirect)  

[放弃 xxl-job：这个分布式调度框架，真强！](http://mp.weixin.qq.com/s?__biz=MzI0NTI0MTg1NQ==&mid=2247484878&idx=1&sn=956db580040225c7f8d2896ead488035&chksm=e950c75ede274e48bedd54b17c75682ab3ffb9fcd50f49fab8247850df68d7b7b796900b938b&scene=21#wechat_redirect)