> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6997601812904132644)

在系统介绍了如何实际部署一个 K8S 项目后，作为本系列文章的最后一篇，我们一起来看看 Kubernetes 集群内容总览，再对一些更深层次的功能进行总结。

[Docker 容器完整课程](https://link.juejin.cn?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1RX4y1g7uR "https://www.bilibili.com/video/BV1RX4y1g7uR")
--------------------------------------------------------------------------------------------------------------------------------------------------

Kubernetes 总览
-------------

下图是一个 k8s 的总览结构内容 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f57fd323e2f4b7f92c45450efaa2cee~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 可以看到图中提到的这些功能模块中，还有一些是在本文中并没有出现的：

**l ConfigMap**

用来存储用户配置文件定义的，通过其内部的 Volume 投射技术实现，其实也是 Volume 挂载的一种方式。这种方式不仅可以实现应用程序被的复用，而且还可以通过不同的配置实现更灵活的功能。在创建容器时，用户可以将应用程序打包为容器镜像后，通过环境变量或者外接挂载文件的方式进行配置注入。

**l Secret**

Secret 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。 将这些信息放在 Secret 中比放在 Pod 的定义、容器镜像中、相对于 ConfigMap 说更加安全和灵活。 Secret 是标准的 k8s 资源对象，使用方法和 ConfigMap 非常类似。同时我们可以对 Secret 进行访问控制，防止机密数据被访问

**l PV**

PVC 是 Kubernetes 中持久化数据卷的实现方式，它是 StatefulSet 的核心功能，也是 Pod 持久化的必要手段，Kubernetes 通过 PV 和 PVC 拆分，从而达到功能点的解耦。

除了文中提到的内容之外， Kubernetes 集群的内容也远比我们目前看到的复杂的多，也还有很多内容等待着我们探索。

在这里，我们对这些深层次的功能做一个总结，也是一个对深入学习 Kubernetes 的梳理。

Kubernetes 组件
-------------

我们平时做开发的过程中所使用的服务器（即宿主机），在 Kubernetes 集群中被称为 Node 节点。

同时在 Kubernetes 中存在一个或者多个 Master 节点控制多个宿主机实现集群，整个 Kubernetes 的核心调度功能基本都在 Master 节点上。

**Kubernetes 的主要功能通过五个大组件组成：**

1.  kubelet：安装在 Node 节点上，用以控制 Node 节点中的容器完成 Kubernetes 的调度逻辑
2.  ControllerManager：是我们上述所讲的控制器模式的核心管理组件，管理了所有 Kubernetes 集群中的控制器逻辑
3.  API Server：服务处理集群中的 api 请求，我们一直写的 kubectl，其实就是发送给 API Server 的请求，请求会在其内部进行处理和转发
4.  Scheduler：负责 Kubernetes 的服务调度，比如控制器只是控制 Pod 的编排，最后的调度逻辑是由 Scheduler 所完成并且发送请求给 kubelet 执行的
5.  Etcd：这是一个分布式的数据库存储项目，由 CoreOS 开发，最终被 RedHat 收购成为 Kubernetes 的一部分，它里面保存了 Kubernetes 集群中的所有配置信息，比如所有集群对象的 name，IP，secret，configMap 等所有数据，其依靠自己的一致性算法可以保证在系统中快速稳定的返回各种配置信息，因此这也是 Kubernetes 和心中的核心组件

定制化功能
-----

除了各种强大的组件功能之外，Kubernetes 也给用户提供了极高的自由度。

为了实现这种高度的自由，Kubernetes 给用户提供了三个公开的接口，分别是：

l CNI（Container Networking Interface，容器网络接口）：其定义了 Kubernetes 集群所有网络的链接方式，整个集群的网络都通过这个接口实现。只要实现了这个接口内所有功能的网络插件，就可以作为 Kubernetes 集群的网络配置插件，其内部包括宿主机路由表配置、7 层网络发现、数据包转发等等都有各式各样的小插件，这些小插件还可以随意配合使用，用户可以按照自己的需求自由定制化这些功能

l CSI（Container Storage Interface，容器存储接口）定义了集群持久化的一些规范，只要是实现这个接口的存储功能，就可以作为 Kubernetes 的持久化插件 l CRI（Container Runtime Interface，容器运行时接口）：在 Kubernetes 的容器运行时，比如默认配置的 Docker 在这个集群的容器运行时，用户可以自由选择实现了这个接口的其他任意容器项目，比如之前提到过的 containerd 和 rkt

这里讲一个有趣的点：CRI。

Kubernetes 的默认容器是 Docker，但是由于项目初期的竞争关系，Docker 其实并不满足 Kubernetes 所定义的 CRI 规范，那怎么办呢？

为了解决这个问题，Kubernetes 专门为 Docker 编写了一个叫 DockerShim 的组件，即 Docker 垫片，用来把 CRI 请求规范，转换成为 Docker 操作 Linux 的 OCI 规范（对，就是第二部分提到的那个 OCI 基金会的那个规范）。但是这个功能一直是由 Kubernetes 项目维护的，只要 Docker 发布了新的功能 Kubernetes 就要维护这个 DockerShim 组件。

于是，这个近期的消息——Kubernetes 将在明年的版本 v1.20 中删除删除 DockerShim 组件，意味着从明年的新版本开始，Kubernetes 将全面不支持 Docker 容器的更新了。

但其实这对我们普通开发者来说可能并没有什么影响，最坏的结果就是我们的镜像需要从 Docker 换成其他 Kubernetes 支持的容器镜像。

不过根据这这段时间各个云平台放出的消息来看，这些平台都会提供对应的转换措施，比如我们还是提供 Docker 镜像，平台在发布运维的时候会把这些镜像转换成其他镜像；又或者这些平台会自行维护一个 DockerShim 来支持 Docker，都是有解决方案的。

架构总览与总结 这一部分我们来看看 Kubernetes 的架构图： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dee0fb79fda46f7a74f2861bae1efed~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 通过这一系列的学习，作为一个普通程序员，不得不赞叹 Google 对编码这件事得深厚与极致，框架中因为太多仅因为解耦而产生的组件，并且还提供了这么大的自由度，不得不说是我们活字格开发团队学习过程中遇到的一个很有技术深度的框架。

但这种过高的自由度也有负面作用。

在部署过程中，Kubernetes 集群的复杂度非常高，部署一个满足生产环境需求的 Kubernetes 框架更是难上加难，网上还有专门卖 Kubernetes 生产环境集群部署的脚本程序，可见 Kubernetes 系统的庞大。

在学习的过程中可以使用 kinD 或者 minikube 在本地以 Docker 的形式模拟一个 Kubernetes 集群，但是这种程度的学习距离生产环境还是有一定的差距。

总结
--

这个系列文章，详细的描述了我们活字格开发团队上天过程中碰到的几个难啃的神仙。

从云平台的发展到 k8s 的具体使用，一步一步抽丝剥茧地向大家解释了一个云平台，从最初的虚拟机，到 PaaS 雏形，到 Docker 容器化，再到最终 Kubernetes 的形态的过度和演变。

人的记忆是需要依赖于前驱节点的，仅仅通过一篇文章就想把 Kubernetes 中涉及的那些技术点和各种难记的名词一一解释清楚显然是不可能的，我们的想法是让大家通过一步一步了解整个云生态的演化过程，从而最终理解整个项目。