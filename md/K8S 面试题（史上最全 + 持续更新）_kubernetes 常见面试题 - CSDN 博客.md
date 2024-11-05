> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/crazymakercircle/article/details/128671196)

### 尼恩面试宝典专题 38：[K8S](https://so.csdn.net/so/search?q=K8S&spm=1001.2101.3001.7020) 面试题（史上最全、持续更新）

##### 本文版本说明：V26

### 《尼恩面试宝典》升级规划为：

后续基本上，**每一个月，都会发布一次**，最新版本，可以联系构师尼恩获取， 发送 “领取电子书” 获取。

### 什么是 k8s？说出你的理解

K8s 是 kubernetes 的简称，其本质是一个开源的容器编排系统，主要用于管理容器化的应用，

其目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes 提供了应用部署，规划，更新，维护的一种机制。

说简单点：k8s 就是一个编排容器的系统，一个可以管理容器应用全生命周期的工具，从创建应用，应用的部署，应用提供服务，扩容缩容应用，应用更新，都非常的方便，而且还可以做到故障自愈，

所以，k8s 是一个非常强大的容器编排系统。

### k8s 的组件有哪些，作用分别是什么？

k8s 主要由 master 节点和 node 节点构成。

master 节点负责管理集群，node 节点是容器应用真正运行的地方。

master 节点包含的组件有：kube-api-server、kube-controller-manager、kube-scheduler、[etcd](https://so.csdn.net/so/search?q=etcd&spm=1001.2101.3001.7020)。

node 节点包含的组件有：kubelet、kube-proxy、container-runtime。

**kube-api-server：**

以下简称 api-server，api-server 是 k8s 最重要的核心组件之一，它是 k8s 集群管理的统一访问入口，提供了 RESTful API 接口, 实现了认证、授权和准入控制等安全功能；api-server 还是其他组件之间的数据交互和通信的枢纽，其他组件彼此之间并不会直接通信，其他组件对资源对象的增、删、改、查和监听操作都是交由 api-server 处理后，api-server 再提交给 etcd 数据库做持久化存储，只有 api-server 才能直接操作 etcd 数据库，其他组件都不能直接操作 etcd 数据库，其他组件都是通过 api-server 间接的读取，写入数据到 etcd。

**kube-controller-manager：**

以下简称 controller-manager，controller-manager 是 k8s 中各种控制器的的管理者，是 k8s 集群内部的管理控制中心，也是 k8s 自动化功能的核心；controller-manager 内部包含 replication controller、node controller、deployment controller、endpoint controller 等各种资源对象的控制器，每种控制器都负责一种特定资源的控制流程，而 controller-manager 正是这些 controller 的核心管理者。

**kube-scheduler：**

以下简称 scheduler，scheduler 负责集群资源调度，其作用是将待调度的 pod 通过一系列复杂的[调度算法](https://so.csdn.net/so/search?q=%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95&spm=1001.2101.3001.7020)计算出最合适的 node 节点，然后将 pod 绑定到目标节点上。shceduler 会根据 pod 的信息，全部节点信息列表，过滤掉不符合要求的节点，过滤出一批候选节点，然后给候选节点打分，选分最高的就是最佳节点，scheduler 就会把目标 pod 安置到该节点。

**Etcd：**

etcd 是一个分布式的键值对存储数据库，主要是用于保存 k8s 集群状态数据，比如，pod，service 等资源对象的信息；etcd 可以是单个也可以有多个，多个就是 etcd 数据库集群，etcd 通常部署奇数个实例，在大规模集群中，etcd 有 5 个或 7 个节点就足够了；另外说明一点，etcd 本质上可以不与 master 节点部署在一起，只要 master 节点能通过网络连接 etcd 数据库即可。

**kubelet：**

每个 node 节点上都有一个 kubelet 服务进程，kubelet 作为连接 master 和各 node 之间的桥梁，负责维护 pod 和容器的生命周期，当监听到 master 下发到本节点的任务时，比如创建、更新、终止 pod 等任务，kubelet 即通过控制 docker 来创建、更新、销毁容器；  
每个 kubelet 进程都会在 api-server 上注册本节点自身的信息，用于定期向 master 汇报本节点资源的使用情况。

**kube-proxy：**

kube-proxy 运行在 node 节点上，在 Node 节点上实现 Pod 网络代理，维护网络规则和四层负载均衡工作，kube-proxy 会监听 api-server 中从而获取 service 和 endpoint 的变化情况，创建并维护路由规则以提供服务 IP 和负载均衡功能。简单理解此进程是 Service 的透明代理兼负载均衡器，其核心功能是将到某个 Service 的访问请求转发到后端的多个 Pod 实例上。

container-runtime：容器运行时环境，即运行容器所需要的一系列程序，目前 k8s 支持的容器运行时有很多，如 docker、rkt 或其他，比较受欢迎的是 docker，但是新版的 k8s 已经宣布弃用 docker。

### 简述 Kubernetes 相关基础概念?

答：

**master：**

k8s 集群的管理节点，负责管理集群，提供集群的资源数据访问入口。拥有 Etcd 存储服务（可选），运行 Api Server 进程，Controller Manager 服务进程及 Scheduler 服务进程；

**node（worker）：**

Node（worker）是 Kubernetes 集群架构中运行 Pod 的服务节点，是 Kubernetes 集群操作的单元，用来承载被分配 Pod 的运行，是 Pod 运行的宿主机。运行 docker eninge 服务，守护进程 kunelet 及负载均衡器 kube-proxy；

**pod：**

运行于 Node 节点上，若干相关容器的组合。Pod 内包含的容器运行在同一宿主机上，使用相同的网络命名空间、IP 地址和端口，能够通过 localhost 进行通信。Pod 是 Kurbernetes 进行创建、调度和管理的最小单位，它提供了比容器更高层次的抽象，使得部署和管理更加灵活。一个 Pod 可以包含一个容器或者多个相关容器；

**label：**

Kubernetes 中的 Label 实质是一系列的 Key/Value 键值对，其中 key 与 value 可自定义。Label 可以附加到各种资源对象上，如 Node、Pod、Service、RC 等。一个资源对象可以定义任意数量的 Label，同一个 Label 也可以被添加到任意数量的资源对象上去。Kubernetes 通过 Label Selector（标签选择器）查询和筛选资源对象；

**Replication Controller：**

Replication Controller 用来管理 Pod 的副本，保证集群中存在指定数量的 Pod 副本。

集群中副本的数量大于指定数量，则会停止指定数量之外的多余容器数量。反之，则会启动少于指定数量个数的容器，保证数量不变。

Replication Controller 是实现弹性伸缩、动态扩容和滚动升级的核心；

**Deployment：**

Deployment 在内部使用了 RS 来实现目的，Deployment 相当于 RC 的一次升级，其最大的特色为可以随时获知当前 Pod 的部署进度；

**HPA（Horizontal Pod Autoscaler）：**

Pod 的横向自动扩容，也是 Kubernetes 的一种资源，通过追踪分析 RC 控制的所有 Pod 目标的负载变化情况，来确定是否需要针对性的调整 Pod 副本数量；

**Service：**

Service 定义了 Pod 的逻辑集合和访问该集合的策略，是真实服务的抽象。

Service 提供了一个统一的服务访问入口以及服务代理和发现机制，关联多个相同 Label 的 Pod，用户不需要了解后台 Pod 是如何运行；

**Volume：**

Volume 是 Pod 中能够被多个容器访问的共享目录，Kubernetes 中的 Volume 是定义在 Pod 上，可以被一个或多个 Pod 中的容器挂载到某个目录下；

**Namespace：**

Namespace 用于实现多租户的资源隔离，可将集群内部的资源对象分配到不同的 Namespace 中，形成逻辑上的不同项目、小组或用户组，便于不同的 Namespace 在共享使用整个集群的资源的同时还能被分别管理；

### 简述 Kubernetes 和 Docker 的关系?

答：

Docker 开源的容器引擎，一种更加轻量级的虚拟化技术；

K8s，容器管理工具，用来管理容器 pod 的集合，它可以实现容器集群的自动化部署、自动扩缩容、维护等功能；

### 简述 Kubernetes 如何实现集群管理?

答：在集群管理方面，Kubernetes 将集群中的机器划分为一个 Master 节点和一群工作节点 Node。

其中，在 Master 节点运行着集群管理相关的一组进程 kube-apiserver、kube-controller-manager 和 kube-scheduler，这些进程实现了整个集群的资源管理、Pod 调度、弹性伸缩、安全控制、系统监控和纠错等管理能力，并且都是全自动完成的；

### 简述 Kubernetes 的优势、适应场景及其特点?

答：

优势：容器编排、轻量级、开源、弹性伸缩、负载均衡；

场景：快速部署应用、快速扩展应用、无缝对接新的应用功能、节省资源，优化硬件资源的使用；

特点：

可移植: 支持公有云、私有云、混合云、多重云（multi-cloud）、

可扩展: 模块化,、插件化、可挂载、可组合、

自动化: 自动部署、自动重启、自动复制、自动伸缩 / 扩展；

### 简述 Kubernetes 的缺点或当前的不足之处?

​ 答：

安装过程和配置相对困难复杂、管理服务相对繁琐、运行和编译需要很多时间、它比其他替代品更昂贵、对于简单的应用程序来说，可能不需要涉及 Kubernetes 即可满足；

### 简述 Kubernetes 中什么是 Minikube、Kubectl、Kubelet?

答：

Minikube 是一种可以在本地轻松运行一个单节点 Kubernetes 群集的工具；

Kubectl 是一个命令行工具，可以使用该工具控制 Kubernetes 集群管理器，如检查群集资源，创建、删除和更新组件，查看应用程序；

Kubelet 是一个代理服务，它在每个节点上运行，并使从服务器与主服务器通信；

### kubelet 的功能、作用是什么？（重点，经常会问）

答：kubelet 部署在每个 node 节点上的，它主要有 4 个功能：  
1、节点管理。

kubelet 启动时会向 api-server 进行注册，然后会定时的向 api-server 汇报本节点信息状态，资源使用状态等，这样 master 就能够知道 node 节点的资源剩余，节点是否失联等等相关的信息了。master 知道了整个集群所有节点的资源情况，这对于 pod 的调度和正常运行至关重要。  
2、pod 管理。

kubelet 负责维护 node 节点上 pod 的生命周期，当 kubelet 监听到 master 的下发到自己节点的任务时，比如要创建、更新、删除一个 pod，kubelet 就会通过 CRI（容器运行时接口）插件来调用不同的容器运行时来创建、更新、删除容器；常见的容器运行时有 docker、containerd、rkt 等等这些容器运行时，我们最熟悉的就是 docker 了，但在新版本的 k8s 已经弃用 docker 了，k8s1.24 版本中已经使用 containerd 作为容器运行时了。

3、容器健康检查。

pod 中可以定义启动探针、存活探针、就绪探针等 3 种，我们最常用的就是存活探针、就绪探针，kubelet 会定期调用容器中的探针来检测容器是否存活，是否就绪，如果是存活探针，则会根据探测结果对检查失败的容器进行相应的重启策略；

4、Metrics Server 资源监控。

在 node 节点上部署 Metrics Server 用于监控 node 节点、pod 的 CPU、内存、文件系统、网络使用等资源使用情况，而 kubelet 则通过 Metrics Server 获取所在节点及容器的上的数据。

### kube-api-server 的端口是多少？各个 pod 是如何访问 kube-api-server 的？

kube-api-server 的端口是 8080 和 6443，前者是 http 的端口，后者是 https 的端口，以我本机使用 kubeadm 安装的 k8s 为例：

在命名空间的 kube-system 命名空间里，有一个名称为 kube-api-master 的 pod，

这个 pod 就是运行着 kube-api-server 进程，它绑定了 master 主机的 ip 地址和 6443 端口，但是在 default 命名空间下，存在一个叫 kubernetes 的服务，该服务对外暴露端口为 443，目标端口 6443，

这个服务的 ip 地址是 clusterip 地址池里面的第一个地址，同时这个服务的 yaml 定义里面并没有指定标签选择器，

也就是说这个 kubernetes 服务所对应的 endpoint 是手动创建的，该 endpoint 也是名称叫做 kubernetes，该 endpoint 的 yaml 定义里面代理到 master 节点的 6443 端口，也就是 kube-api-server 的 IP 和端口。

这样一来，其他 pod 访问 kube-api-server 的整个流程就是：pod 创建后嵌入了环境变量，pod 获取到了 kubernetes 这个服务的 ip 和 443 端口，请求到 kubernetes 这个服务其实就是转发到了 master 节点上的 6443 端口的 kube-api-server 这个 pod 里面。

### k8s 中命名空间的作用是什么？

amespace 是 kubernetes 系统中的一种非常重要的资源，namespace 的主要作用是用来实现多套环境的资源隔离，或者说是多租户的资源隔离。

k8s 通过将集群内部的资源分配到不同的 namespace 中，可以形成逻辑上的隔离，以方便不同的资源进行隔离使用和管理。

不同的命名空间可以存在同名的资源，命名空间为资源提供了一个作用域。

可以通过 k8s 的授权机制，将不同的 namespace 交给不同的租户进行管理，这样就实现了多租户的资源隔离，还可以结合 k8s 的资源配额机制，限定不同的租户能占用的资源，例如 CPU 使用量、内存使用量等等来实现租户可用资源的管理。

### k8s 提供了大量的 REST 接口，其中有一个是 Kubernetes Proxy API 接口，简述一下这个 Proxy 接口的作用，已经怎么使用。

kubernetes proxy api 接口，从名称中可以得知，proxy 是代理的意思，其作用就是代理 rest 请求；

Kubernets API server 将接收到的 rest 请求转发到某个 node 上的 kubelet 守护进程的 rest 接口，由该 kubelet 进程负责响应。

我们可以使用这种 Proxy 接口来直接访问某个 pod，这对于逐一排查 pod 异常问题很有帮助。  
下面是一些简单的例子：

```
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/pods  	#查看指定node的所有pod信息
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/stats  	#查看指定node的物理资源统计信息
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/spec  	#查看指定node的概要信息

http://<kube-api-server>:<api-sever-port>/api/v1/namespace/命名名称/pods/pod名称/pod服务的url/  	#访问指定pod的程序页面
http://<kube-api-server>:<api-sever-port>/api/v1/namespace/命名名称/servers/svc名称/url/  	#访问指定server的url程序页面


```

### pod 是什么？

在 kubernetes 的世界中，k8s 并不直接处理容器，而是使用多个容器共存的理念，这组容器就叫做 pod。

pod 是 k8s 中可以创建和管理的最小单元，是资源对象模型中由用户创建或部署的最小资源对象模型，其他的资源对象都是用来支撑 pod 对象功能的，比如，pod 控制器就是用来管理 pod 对象的，service 或者 imgress 资源对象是用来暴露 pod 引用对象的，persistentvolume 资源是用来为 pod 提供存储等等，

简而言之，k8s 不会直接处理容器，而是 pod，pod 才是 k8s 中可以创建和管理的最小单元，也是基本单元。

### pod 的原理是什么？

在微服务的概念里，一般的，一个容器会被设计为运行一个进程，除非进程本身产生子进程，

这样，由于不能将多个进程聚集在同一个单独的容器中，所以需要一种更高级的结构将容器绑定在一起，并将它们作为一个单元进行管理，这就是 k8s 中 pod 的背后原理。

### pod 有什么特点？

1、每个 pod 就像一个独立的逻辑机器，k8s 会为每个 pod 分配一个集群内部唯一的 IP 地址，所以每个 pod 都拥有自己的 IP 地址、主机名、进程等；  
2、一个 pod 可以包含 1 个或多个容器，1 个容器一般被设计成只运行 1 个进程，1 个 pod 只可能运行在单个节点上，即不可能 1 个 pod 跨节点运行，pod 的生命周期是短暂，也就是说 pod 可能随时被消亡（如节点异常，pod 异常等情况）；  
2、每一个 pod 都有一个特殊的被称为 "根容器" 的 pause 容器，也称 info 容器，pause 容器对应的镜像属于 k8s 平台的一部分，除了 pause 容器，每个 pod 还包含一个或多个跑业务相关组件的应用容器；  
3、一个 pod 中的容器共享 network 命名空间；  
4、一个 pod 里的多个容器共享 pod IP，这就意味着 1 个 pod 里面的多个容器的进程所占用的端口不能相同，否则在这个 pod 里面就会产生端口冲突；既然每个 pod 都有自己的 IP 和端口空间，那么对不同的两个 pod 来说就不可能存在端口冲突；  
5、应该将应用程序组织到多个 pod 中，而每个 pod 只包含紧密相关的组件或进程；  
6、pod 是 k8s 中扩容、缩容的基本单位，也就是说 k8s 中扩容缩容是针对 pod 而言而非容器。

### pod 的重启策略有哪些？

pod 重启容器策略是指针对 pod 内所有容器的重启策略，不是重启 pod，其可以通过 restartPolicy 字段配置 pod 重启容器的策略，如下：

*   Always: 当容器终止退出后，总是重启容器，默认策略就是 Always。
    
*   OnFailure: 当容器异常退出，退出状态码非 0 时，才重启容器。
    
*   Never: 当容器终止退出，不管退出状态码是什么，从不重启容器。
    

### pod 的镜像拉取策略有哪几种？

pod 镜像拉取策略可以通过 imagePullPolicy 字段配置镜像拉取策略，

主要有 3 中镜像拉取策略，如下：

*   IfNotPresent: 默认值，镜像在 node 节点宿主机上不存在时才拉取。
*   Always: 总是重新拉取，即每次创建 pod 都会重新从镜像仓库拉取一次镜像。
*   Never: 永远不会主动拉取镜像，仅使用本地镜像，需要你手动拉取镜像到 node 节点，如果 node 节点不存在镜像则 pod 启动失败。

### kubenetes 针对 pod 资源对象的健康监测机制?（必须记住 3 重探测方式，重点，经常问）

提供了三类 probe（探针）来执行对 pod 的健康监测：

*   livenessProbe 探针 （存活探针）:

可以根据用户自定义规则来判定 pod 是否健康，用于判断容器是否处于 Running 状态，

如果不是，kubelet 就会杀掉该容器，并根据重启策略做相应的处理。如果容器不包含该探针，那么 kubelet 就会默认返回值都是 success;

*   ReadinessProbe 探针:

同样是可以根据用户自定义规则来判断 pod 是否健康，容器服务是否可用（Ready），如果探测失败，控制器会将此 pod 从对应 service 的 endpoint 列表中移除，从此不再将任何请求调度到此 Pod 上，直到下次探测成功;

*   startupProbe 探针:

启动检查机制，应用一些启动缓慢的业务，避免业务长时间启动而被上面两类探针 kill 掉，

这个问题也可以换另一种方式解决，就是定义上面两类探针机制时，初始化时间定义的长一些即可;

备注：每种探测方法能支持以下几个相同的检查参数，用于设置控制检查时间：

*   initialDelaySeconds：初始第一次探测间隔，用于应用启动的时间，防止应用还没启动而健康检查失败；
    
*   periodSeconds：检查间隔，多久执行 probe 检查，默认为 10s；
    
*   timeoutSeconds：检查超时时长，探测应用 timeout 后为失败；
    
*   successThreshold：成功探测阈值，表示探测多少次为健康正常，默认探测 1 次。
    

### 就绪探针（ReadinessProbe 探针）与存活探针（livenessProbe 探针）区别是什么？

两者作用不一样，

存活探针是将检查失败的容器杀死，创建新的启动容器来保持 pod 正常工作；

就绪探针是，当就绪探针检查失败，并不重启容器，而是将 pod 移出 endpoint，就绪探针确保了 service 中的 pod 都是可用的，确保客户端只与正常的 pod 交互并且客户端永远不会知道系统存在问题。

### 存活探针的属性参数有哪几个？

存活探针的附加属性参数有以下几个：

*   initialDelaySeconds：表示在容器启动后延时多久秒才开始探测；
    
*   periodSeconds：表示执行探测的频率，即间隔多少秒探测一次，默认间隔周期是 10 秒，最小 1 秒；
    
*   timeoutSeconds：表示探测超时时间，默认 1 秒，最小 1 秒，表示容器必须在超时时间范围内做出响应，否则视为本次探测失败；
    
*   successThreshold：表示最少连续探测成功多少次才被认定为成功，默认是 1，对于 liveness 必须是 1，最小值是 1；
    
*   failureThreshold：表示连续探测失败多少次才被认定为失败，默认是 3，连续 3 次失败，k8s 将根据 pod 重启策略对容器做出决定；
    

注意：定义存活探针时，一定要设置 initialDelaySeconds 属性，该属性为初始延时，如果不设置，默认容器启动时探针就开始探测了，这样可能会存在  
应用程序还未启动就绪，就会导致探针检测失败，k8s 就会根据 pod 重启策略杀掉容器然后再重新创建容器的莫名其妙的问题。  
在生产环境中，一定要定义一个存活探针。

### pod 的就绪探针有哪几种？

我们知道，当一个 pod 启动后，就会立即加入 service 的 endpoint ip 列表中，并开始接收到客户端的链接请求，

假若此时 pod 中的容器的业务进程还没有初始化完毕，那么这些客户端链接请求就会失败，为了解决这个问题，kubernetes 提供了就绪探针来解决这个问题的。

在 pod 中的容器定义一个就绪探针，就绪探针周期性检查容器，

如果就绪探针检查失败了，说明该 pod 还未准备就绪，不能接受客户端链接，则该 pod 将从 endpoint 列表中移除，

pod 被剔除了, service 就不会把请求分发给该 pod，

然后就绪探针继续检查，如果随后容器就绪，则再重新把 pod 加回 endpoint 列表。

kubernetes 提供了 3 种探测容器的存活探针，如下：

*   httpGet：通过容器的 IP、端口、路径发送 http 请求，返回 200-400 范围内的状态码表示成功。
    
*   exec：在容器内执行 shell 命令，根据命令退出状态码是否为 0 进行判断，0 表示健康，非 0 表示不健康。
    
*   TCPSocket：与容器的 IP、端口建立 TCP Socket 链接，能建立则说明探测成功，不能建立则说明探测失败
    

### pod 的就绪探针的属性参数有哪些

就绪探针的附加属性参数有以下几个：

*   initialDelaySeconds：延时秒数，即容器启动多少秒后才开始探测，不写默认容器启动就探测；
    
*   periodSeconds ：执行探测的频率（秒），默认为 10 秒，最低值为 1；
    
*   timeoutSeconds ：超时时间，表示探测时在超时时间内必须得到响应，负责视为本次探测失败，默认为 1 秒，最小值为 1；
    
*   failureThreshold ：连续探测失败的次数，视为本次探测失败，默认为 3 次，最小值为 1 次；
    
*   successThreshold ：连续探测成功的次数，视为本次探测成功，默认为 1 次，最小值为 1 次；
    

### pod 的重启策略是什么？

答：通过命令 “[kubectl](https://so.csdn.net/so/search?q=kubectl&spm=1001.2101.3001.7020) explain pod.spec” 查看 pod 的重启策略;

*   Always：但凡 pod 对象终止就重启，此为默认策略;
    
*   OnFailure：仅在 pod 对象出现错误时才重启;
    

### 简单讲一下 pod 创建过程

情况一、使用 kubectl run 命令创建的 pod：

```
注意：
kubectl run 在旧版本中创建的是deployment，
但在新的版本中创建的是pod则其创建过程不涉及deployment

```

如果是单独的创建一个 pod，则其创建过程是这样的：  
1、首先，用户通过 kubectl 或其他 api 客户端工具提交需要创建的 pod 信息给 apiserver；  
2、apiserver 验证客户端的用户权限信息，验证通过开始处理创建请求生成 pod 对象信息，并将信息存入 etcd，然后返回确认信息给客户端；  
3、apiserver 开始反馈 etcd 中 pod 对象的变化，其他组件使用 watch 机制跟踪 apiserver 上的变动；  
4、scheduler 发现有新的 pod 对象要创建，开始调用内部算法机制为 pod 分配最佳的主机，并将结果信息更新至 apiserver；  
5、node 节点上的 kubelet 通过 watch 机制跟踪 apiserver 发现有 pod 调度到本节点，尝试调用 docker 启动容器，并将结果反馈 apiserver；  
6、apiserver 将收到的 pod 状态信息存入 etcd 中。  
至此，整个 pod 创建完毕。

情况二、使用 deployment 来创建 pod：

1、首先，用户使用 kubectl create 命令或者 kubectl apply 命令提交了要创建一个 deployment 资源请求；  
2、api-server 收到创建资源的请求后，会对客户端操作进行身份认证，在客户端的~/.kube 文件夹下，已经设置好了相关的用户认证信息，这样 api-server 会知道我是哪个用户，并对此用户进行鉴权，当 api-server 确定客户端的请求合法后，就会接受本次操作，并把相关的信息保存到 etcd 中，然后返回确认信息给客户端。  
3、apiserver 开始反馈 etcd 中过程创建的对象的变化，其他组件使用 watch 机制跟踪 apiserver 上的变动。  
4、controller-manager 组件会监听 api-server 的信息，controller-manager 是有多个类型的，比如 Deployment Controller, 它的作用就是负责监听 Deployment，此时 Deployment Controller 发现有新的 deployment 要创建，那么它就会去创建一个 ReplicaSet，一个 ReplicaSet 的产生，又被另一个叫做 ReplicaSet Controller 监听到了，紧接着它就会去分析 ReplicaSet 的语义，它了解到是要依照 ReplicaSet 的 template 去创建 Pod, 它一看这个 Pod 并不存在，那么就新建此 Pod，当 Pod 刚被创建时，它的 nodeName 属性值为空，代表着此 Pod 未被调度。  
5、调度器 Scheduler 组件开始介入工作，Scheduler 也是通过 watch 机制跟踪 apiserver 上的变动，发现有未调度的 Pod，则根据内部算法、节点资源情况，pod 定义的亲和性反亲和性等等，调度器会综合的选出一批候选节点，在候选节点中选择一个最优的节点，然后将 pod 绑定该该节点，将信息反馈给 api-server。  
6、kubelet 组件布署于 Node 之上，它也是通过 watch 机制跟踪 apiserver 上的变动，监听到有一个 Pod 应该要被调度到自身所在 Node 上来，kubelet 首先判断本地是否在此 Pod，如果不存在，则会进入创建 Pod 流程，创建 Pod 有分为几种情况，第一种是容器不需要挂载外部存储，则相当于直接 docker run 把容器启动，但不会直接挂载 docker 网络，而是通过 CNI 调用网络插件配置容器网络，如果需要挂载外部存储，则还要调用 CSI 来挂载存储。kubelet 创建完 pod，将信息反馈给 api-server，api-servier 将 pod 信息写入 etcd。  
7、Pod 建立成功后，ReplicaSet Controller 会对其持续进行关注，如果 Pod 因意外或被我们手动退出，ReplicaSet Controller 会知道，并创建新的 Pod，以保持 replicas 数量期望值。

### k8s 创建一个 pod 的详细流程，涉及的组件怎么通信的？

答：

1）客户端提交创建请求，可以通过 api-server 提供的 restful 接口，或者是通过 kubectl 命令行工具，支持的数据类型包括 JSON 和 YAML；

2）api-server 处理用户请求，将 pod 信息存储至 etcd 中；

3）kube-scheduler 通过 api-server 提供的接口监控到未绑定的 pod，尝试为 pod 分配 node 节点，主要分为两个阶段，预选阶段和优选阶段，其中预选阶段是遍历所有的 node 节点，根据策略筛选出候选节点，而优选阶段是在第一步的基础上，为每一个候选节点进行打分，分数最高者胜出；

4）选择分数最高的节点，进行 pod binding 操作，并将结果存储至 etcd 中；

5）随后目标节点的 kubelet 进程通过 api-server 提供的接口监测到 kube-scheduler 产生的 pod 绑定事件，然后从 etcd 获取 pod 清单，下载镜像并启动容器；

### 简单描述一下 pod 的终止过程

1、用户向 apiserver 发送删除 pod 对象的命令；  
2、apiserver 中的 pod 对象信息会随着时间的推移而更新，在宽限期内（默认 30s），pod 被视为 dead；  
3、将 pod 标记为 terminating 状态；  
4、kubectl 在监控到 pod 对象为 terminating 状态了就会启动 pod 关闭过程；  
5、endpoint 控制器监控到 pod 对象的关闭行为时将其从所有匹配到此 endpoint 的 server 资源 endpoint 列表中删除；  
6、如果当前 pod 对象定义了 preStop 钩子处理器，则在其被标记为 terminating 后会意同步的方式启动执行；  
7、pod 对象中的容器进程收到停止信息；  
8、宽限期结束后，若 pod 中还存在运行的进程，那么 pod 对象会收到立即终止的信息；  
9、kubelet 请求 apiserver 将此 pod 资源的宽限期设置为 0 从而完成删除操作，此时 pod 对用户已不可见。

### pod 的生命周期有哪几种？

pod 生命周期有的 5 种状态（也称 5 种相位），如下：

*   Pending（挂起）：API server 已经创建 pod，但是该 pod 还有一个或多个容器的镜像没有创建，包括正在下载镜像的过程；
    
*   Running（运行中）：Pod 内所有的容器已经创建，且至少有一个容器处于运行状态、正在启动括正在重启状态；
    
*   Succeed（成功）：Pod 内所有容器均已退出，且不会再重启；
    
*   Failed（失败）：Pod 内所有容器均已退出，且至少有一个容器为退出失败状态
    
*   Unknown（未知）：某于某种原因 apiserver 无法获取该 pod 的状态，可能由于网络通行问题导致；
    

### pod 一致处于 pending 状态一般有哪些情况，怎么排查？（重点，持续更新）

（这个问题被问到的概率非常大）  
一个 pod 一开始创建的时候，它本身就是会处于 pending 状态，这时可能是正在拉取镜像，正在创建容器的过程。

如果等了一会发现 pod 一直处于 pending 状态，

那么我们可以使用 kubectl describe 命令查看一下 pod 的 Events 详细信息。一般可能会有这么几种情况导致 pod 一直处于 pending 状态：  
1、调度器调度失败。

Scheduer 调度器无法为 pod 分配一个合适的 node 节点。

而这又会有很多种情况，比如，node 节点处在 cpu、内存压力，导致无节点可调度；pod 定义了资源请求，没有 node 节点满足资源请求；node 节点上有污点而 pod 没有定义容忍；pod 中定义了亲和性或反亲和性而没有节点满足这些亲和性或反亲和性；以上是调度器调度失败的几种情况。  
2、pvc、pv 无法动态创建。

如果因为 pvc 或 pv 无法动态创建，那么 pod 也会一直处于 pending 状态，比如要使用 StatefulSet 创建 redis 集群，因为粗心大意，定义的 storageClassName 名称写错了，那么会造成无法创建 pvc，这种情况 pod 也会一直处于 pending 状态，或者，即使 pvc 是正常创建了，但是由于某些异常原因导致动态供应存储无法正常创建 pv，那么这种情况 pod 也会一直处于 pending 状态。

### DaemonSet 资源对象的特性？

答：

DaemonSet 这种资源对象会在每个 k8s 集群中的节点上运行，并且每个节点只能运行一个 pod，这是它和 deployment 资源对象的最大也是唯一的区别。

所以，在其 yaml 文件中，不支持定义 replicas，

除此之外，与 Deployment、RS 等资源对象的写法相同,

DaemonSet 一般使用的场景有

*   在去做每个节点的日志收集工作；
*   监控每个节点的的运行状态;

### 删除一个 Pod 会发生什么事情？

答：

Kube-apiserver 会接受到用户的删除指令，默认有 30 秒时间等待优雅退出，超过 30 秒会被标记为死亡状态，

此时 Pod 的状态 Terminating，kubelet 看到 pod 标记为 Terminating 就开始了关闭 Pod 的工作;

关闭流程如下：

1）pod 从 service 的 endpoint 列表中被移除；

2) 如果该 pod 定义了一个停止前的钩子，其会在 pod 内部被调用，停止钩子一般定义了如何优雅的结束进程；

3) 进程被发送 TERM 信号（kill -14）;

4) 当超过优雅退出的时间后，Pod 中的所有进程都会被发送 SIGKILL 信号（kill -9）;

### pod 的共享资源？

​ 答：

1）PID 命名空间：Pod 中的不同应用程序可以看到其他应用程序的进程 ID；

2）网络命名空间：Pod 中的多个容器能够访问同一个 IP 和端口范围；

3）IPC 命名空间：Pod 中的多个容器能够使用 SystemV IPC 或 POSIX 消息队列进行通信；

4）UTS 命名空间：Pod 中的多个容器共享一个主机名；

5）Volumes（共享存储卷）：Pod 中的各个容器可以访问在 Pod 级别定义的 Volumes；

### pod 的初始化容器是干什么的？

init container，初始化容器用于在启动应用容器之前完成应用容器所需要的前置条件，

初始化容器本质上和应用容器是一样的，但是初始化容器是仅允许一次就结束的任务，初始化容器具有两大特征：

1、初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么 kubernetes 需要重启它直到成功完成；  
2、初始化容器必须按照定义的顺序执行，当且仅当前一个初始化容器成功之后，后面的一个初始化容器才能运行；

### pod 的资源请求、限制如何定义？

pod 的资源请求、资源限制可以直接在 pod 中定义

主要包括两块内容，

*   limits，限制 pod 能使用的最大 cpu 和内存，
*   requests，pod 启动时申请的 cpu 和内存。

```
 resources:					#资源配额
      limits:					#限制最大资源，上限
        cpu: 2					#CPU限制，单位是code数
        memory: 2G				#内存最大限制
      requests:					#请求资源（最小，下限）
        cpu: 1					#CPU请求，单位是code数
        memory: 500G			#内存最小请求


```

### pod 的定义中有个 command 和 args 参数，这两个参数不会和 docker 镜像的 entrypointc 冲突吗？

不会。

在 pod 中定义的 command 参数用于指定容器的启动命令列表，如果不指定，则默认使用 Dockerfile 打包时的启动命令，args 参数用于容器的启动命令需要的参数列表；

特别说明：

kubernetes 中的 command、args 其实是实现覆盖 dockerfile 中的 ENTRYPOINT 的功能的。

```
1、如果command和args均没有写，那么使用Dockerfile的配置；
2、如果command写了但args没写，那么Dockerfile默认的配置会被忽略，执行指定的command；
3、如果command没写但args写了，那么Dockerfile中的ENTRYPOINT的会被执行，使用当前args的参数；
4、如果command和args都写了，那么Dockerfile会被忽略，执行输入的command和args。



```

### pause 容器作用是什么？

每个 pod 里运行着一个特殊的被称之为 pause 的容器，也称根容器，而其他容器则称为业务容器；

创建 pause 容器主要是为了为业务容器提供 Linux 命名空间，共享基础：包括 pid、icp、net 等，以及启动 init 进程，并收割僵尸进程；

这些业务容器共享 pause 容器的网络命名空间和 volume 挂载卷，

当 pod 被创建时，pod 首先会创建 pause 容器，从而把其他业务容器加入 pause 容器，从而让所有业务容器都在同一个命名空间中，这样可以就可以实现网络共享。

pod 还可以共享存储，在 pod 级别引入数据卷 volume，业务容器都可以挂载这个数据卷从而实现持久化存储。

### 标签及标签选择器是什么，如何使用？

标签是键值对类型，标签可以附加到任何资源对象上，主要用于管理对象，查询和筛选。

标签常被用于标签选择器的匹配度检查，从而完成资源筛选；一个资源可以定义一个或多个标签在其上面。

标签选择器，标签要与标签选择器结合在一起，标签选择器允许我们选择标记有特定标签的资源对象子集，如 pod，并对这些特定标签的 pod 进行查询，删除等操作。

标签和标签选择器最重要的使用之一在于，在 deployment 中，在 pod 模板中定义 pod 的标签，然后在 deployment 定义标签选择器，这样就通过标签选择器来选择哪些 pod 是受其控制的，service 也是通过标签选择器来关联哪些 pod 最后其服务后端 pod。

### service 是如何与 pod 关联的？

答案是通过标签选择器，每一个由 deployment 创建的 pod 都带有标签，这样，service 就可以定义标签选择器来关联哪些 pod 是作为其后端了，就是这样，service 就与 pod 管联在一起了。

### service 的域名解析格式、pod 的域名解析格式

service 的 DNS 域名表示格式为`<servicename>.<namespace>.svc.<clusterdomain>`，

servicename 是 service 的名称，namespace 是 service 所处的命名空间，clusterdomain 是 k8s 集群设置的域名后缀，一般默认为 cluster.local

pod 的 DNS 域名格式为：`<pod-ip>.<namespace>.pod.<clusterdomain>` ，

其中，pod-ip 需要使用 - 将 ip 直接的点替换掉，namespace 为 pod 所在的命名空间，clusterdomain 是 k8s 集群设置的域名后缀，一般默认为 cluster.local ，

演示如下：`10-244-1-223.default.pod.cluster.local`

对于 deployment、daemonsets 等创建的 pod，还还可以通过`<pod-ip>.<deployment-name>.<namespace>.svc.<clusterdomain>` 这样的域名访问。

### service 的类型有哪几种

service 的类型一般有 4 中，分别是：

*   ClusterIP：表示 service 仅供集群内部使用，默认值就是 ClusterIP 类型
    
*   NodePort：表示 service 可以对外访问应用，会在每个节点上暴露一个端口，这样外部浏览器访问地址为：任意节点的 IP：NodePort 就能连上 service 了
    
*   LoadBalancer：表示 service 对外访问应用，这种类型的 service 是公有云环境下的 service，此模式需要外部云厂商的支持，需要有一个公网 IP 地址
    
*   ExternalName：这种类型的 service 会把集群外部的服务引入集群内部，这样集群内直接访问 service 就可以间接的使用集群外部服务了
    

一般情况下，service 都是 ClusterIP 类型的，通过 ingress 接入的外部流量。

### Pod 到 Service 的通信？

​

1）k8s 在创建服务时为服务分配一个虚拟 IP，客户端通过该 IP 访问服务，服务则负责将请求转发到后端 Pod 上；

2）Service 是通过 kube-proxy 服务进程实现，该进程在每个 Node 上均运行可以看作一个透明代理兼负载均衡器；

3）对每个 TCP 类型 Service，kube-proxy 都会在本地 Node 上建立一个 SocketServer 来负责接受请求，然后均匀发送到后端 Pod 默认采用 Round Robin 负载均衡算法；

4）Service 的 Cluster IP 与 NodePort 等概念是 kube-proxy 通过 Iptables 的 NAT 转换实现，kube-proxy 进程动态创建与 Service 相关的 Iptables 规则；

5）kube-proxy 通过查询和监听 API Server 中 Service 与 Endpoints 的变化来实现其主要功能，包括为新创建的 Service 打开一个本地代理对象，接收请求针对针对发生变化的 Service 列表，kube-proxy 会逐个处理；

### 一个应用 pod 是如何发现 service 的，或者说，pod 里面的容器用于是如何连接 service 的？

答：有两种方式，一种是通过环境变量，另一种是通过 service 的 dns 域名方式。

1、环境变量：

当 pod 被创建之后，k8s 系统会自动为容器注入集群内有效的 service 名称和端口号等信息为环境变量的形式，

这样容器应用直接通过取环境变量值就能访问 service 了，

如`curl http://${WEBAPP_SERVICE_HOST}:{WEBAPP_SERVICE_PORT}`

2、DNS 方式：

使用 dns 域名解析的前提是 k8s 集群内有 DNS 域名解析服务器，

默认 k8s 中会有一个 CoreDNS 作为 k8s 集群的默认 DNS 服务器提供域名解析服务器；

service 的 DNS 域名表示格式为`<servicename>.<namespace>.svc.<clusterdomain>`，

servicename 是 service 的名称，namespace 是 service 所处的命名空间，clusterdomain 是 k8s 集群设置的域名后缀，一般默认为 cluster.local ，

这样容器应用直接通过 service 域名就能访问 service 了，

如`wget http://svc-deployment-nginx.default.svc.cluster.local:80`，

另外，service 的 port 端口如果定义了名称，那么 port 也可以通过 DNS 进行解析，

格式为：`_<portname>._<protocol>.<servicename>.<namespace>.svc.<clusterdomain>`

### 如何创建一个 service 代理外部的服务，或者换句话来说，在 k8s 集群内的应用如何访问外部的服务，如数据库服务，缓存服务等?

答：可以通过创建一个没有标签选择器的 service 来代理集群外部的服务。

1、创建 service 时不指定 selector 标签选择器，但需要指定 service 的 port 端口、端口的 name、端口协议等，这样创建出来的 service 因为没有指定标签选择器就不会自动创建 endpoint；

2、手动创建一个与 service 同名的 endpoint，endpoint 中定义外部服务的 IP 和端口，endpoint 的名称一定要与 service 的名称一样，端口协议也要一样，端口的 name 也要与 service 的端口的 name 一样，不然 endpoint 不能与 service 进行关联。

完成以上两步，k8s 会自动将 service 和同名的 endpoint 进行关联，

这样，k8s 集群内的应用服务直接访问这个 service 就可以相当于访问外部的服务了。

### service、endpoint、kube-proxys 三种的关系是什么？

**service**：

在 kubernetes 中，service 是一种为一组功能相同的 pod 提供单一不变的接入点的资源。

当 service 被建立时，service 的 IP 和端口不会改变，这样外部的客户端（也可以是集群内部的客户端）通过 service 的 IP 和端口来建立链接，这些链接会被路由到提供该服务的任意一个 pod 上。

通过这样的方式，客户端不需要知道每个单独提供服务的 pod 地址，这样 pod 就可以在集群中随时被创建或销毁。

**endpoint**：

service 维护一个叫 endpoint 的资源列表，endpoint 资源对象保存着 service 关联的 pod 的 ip 和端口。

从表面上看，当 pod 消失，service 会在 endpoint 列表中剔除 pod，当有新的 pod 加入，service 就会将 pod ip 加入 endpoint 列表；

但是正在底层的逻辑是，endpoint 的这种自动剔除、添加、更新 pod 的地址其实底层是由`endpoint controller`控制的，`endpoint controller`负责监听 service 和对应的 pod 副本的变化，如果监听到 service 被删除，则删除和该 service 同名的 endpoint 对象，如果监听到新的 service 被创建或者修改，则根据该 service 信息获取得相关 pod 列表，然后创建或更新 service 对应的 endpoint 对象，如果监听到 pod 事件，则更新它所对应的 service 的 endpoint 对象。

**kube-proxy**：

kube-proxy 运行在 node 节点上，在 Node 节点上实现 Pod 网络代理，维护网络规则和四层负载均衡工作，

`kube-proxy`会监听`api-server`中从而获取 service 和 endpoint 的变化情况，创建并维护路由规则以提供服务 IP 和负载均衡功能。

简单理解此进程是 Service 的透明代理兼负载均衡器，其核心功能是将到某个 Service 的访问请求转发到后端的多个 Pod 实例上。

### 无头 service 和普通的 service 有什么区别，无头 service 使用场景是什么？

答：

**无头 service** 没有 cluster ip，在定义 service 时将 `service.spec.clusterIP：None`，就表示创建的是无头 service。

**普通的 service** 是用于为一组后端 pod 提供请求连接的负载均衡，让客户端能通过固定的 service ip 地址来访问 pod，这类的 pod 是没有状态的，同时 service 还具有负载均衡和服务发现的功能。普通 service 跟我们平时使用的 nginx 反向代理很相识。

试想这样一种情况，有 6 个 redis pod , 它们相互之间要通信并要组成一个 redis 集群，

不需要所谓的 service 负载均衡，这时无头 service 就是派上用场了，

无头 service 由于没有 cluster ip，kube-proxy 就不会处理它也就不会对它生成规则负载均衡，无头 service 直接绑定的是 pod 的 ip。无头 service 仍会有标签选择器，有标签选择器就会有 endpoint 资源。

**无头 service 使用场景：**

无头 service 一般用于有状态的应用场景，如 Kaka 集群、Redis 集群等，这类 pod 之间需要相互通信相互组成集群，不在需要所谓的 service 负载均衡。

### deployment 怎么扩容或缩容？

答：直接修改 pod 副本数即可，可以通过下面的方式来修改 pod 副本数：

1、直接修改 yaml 文件的 replicas 字段数值，然后`kubectl apply -f xxx.yaml`来实现更新；

2、使用`kubectl edit deployment xxx` 修改 replicas 来实现在线更新；

3、使用`kubectl scale --replicas=5 deployment/deployment-nginx`命令来扩容缩容。

### deployment 的更新升级策略有哪些？

答：deployment 的升级策略主要有两种。

1、Recreate 重建更新：这种更新策略会杀掉所有正在运行的 pod，然后再重新创建的 pod；

2、rollingUpdate 滚动更新：这种更新策略，deployment 会以滚动更新的方式来逐个更新 pod，同时通过设置滚动更新的两个参数`maxUnavailable、maxSurge`来控制更新的过程。

### deployment 的滚动更新策略有两个特别主要的参数，解释一下它们是什么意思？

答：deployment 的滚动更新策略，rollingUpdate 策略，主要有两个参数，maxUnavailable、maxSurge。

*   maxUnavailable：最大不可用数，maxUnavailable 用于指定 deployment 在更新的过程中不可用状态的 pod 的最大数量，maxUnavailable 的值可以是一个整数值，也可以是 pod 期望副本的百分比，如 25%，计算时向下取整。
    
*   maxSurge：最大激增数，maxSurge 指定 deployment 在更新的过程中 pod 的总数量最大能超过 pod 副本数多少个，maxUnavailable 的值可以是一个整数值，也可以是 pod 期望副本的百分比，如 25%，计算时向上取整。
    

### deployment 更新的命令有哪些？

答：可以通过三种方式来实现更新 deployment。

1、直接修改 yaml 文件的镜像版本，然后`kubectl apply -f xxx.yaml`来实现更新；

2、使用`kubectl edit deployment xxx` 实现在线更新；

3、使用`kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1` 命令来更新。

### 简述一下 deployment 的更新过程?

deployment 是通过控制 replicaset 来实现，由 replicaset 真正创建 pod 副本，每更新一次 deployment，都会创建新的 replicaset，下面来举例 deployment 的更新过程：

假设要升级一个 nginx-deployment 的版本镜像为 nginx:1.9，deployment 的定义滚动更新参数如下：

```
replicas: 3
deployment.spec.strategy.type: RollingUpdate
maxUnavailable：25%
maxSurge：25%


```

通过计算我们得出，3*25%=0.75，maxUnavailable 是向下取整，则 maxUnavailable=0，maxSurge 是向上取整，则 maxSurge=1，所以我们得出在整个 deployment 升级镜像过程中，不管旧的 pod 和新的 pod 是如何创建消亡的，pod 总数最大不能超过 3+maxSurge=4 个，最大 pod 不可用数 3-maxUnavailable=3 个。

现在具体讲一下 deployment 的更新升级过程：

使用`kubectl set image deployment/nginx nginx=nginx:1.9 --record` 命令来更新；

1、deployment 创建一个新的 replaceset，先新增 1 个新版本 pod，此时 pod 总数为 4 个，不能再新增了，再新增就超过 pod 总数 4 个了；旧 = 3，新 = 1，总 = 4；

2、减少一个旧版本的 pod，此时 pod 总数为 3 个，这时不能再减少了，再减少就不满足最大 pod 不可用数 3 个了；旧 = 2，新 = 1，总 = 3；

3、再新增一个新版本的 pod，此时 pod 总数为 4 个，不能再新增了；旧 = 2，新 = 2，总 = 4；

4、减少一个旧版本的 pod，此时 pod 总数为 3 个，这时不能再减少了；旧 = 1，新 = 2，总 = 3；

5、再新增一个新版本的 pod，此时 pod 总数为 4 个，不能再新增了；旧 = 1，新 = 3，总 = 4；

6、减少一个旧版本的 pod，此时 pod 总数为 3 个，更新完成，pod 都是新版本了；旧 = 0，新 = 3，总 = 3；

### deployment 的回滚使用什么命令

在升级 deployment 时 kubectl set image 命令加上 --record 参数可以记录具体的升级历史信息，

使用`kubectl rollout history deployment/deployment-nginx` 命令来查看指定的 deployment 升级历史记录，

如果需要回滚到某个指定的版本，可以使用`kubectl rollout undo deployment/deployment-nginx --to-revision=2` 命令来实现。

### 讲一下都有哪些存储卷，作用分别是什么?

<table><thead><tr><th>卷</th><th>作用</th><th>常用场景</th></tr></thead><tbody><tr><td>emptyDir</td><td>用于存储临时数据的简单空目录</td><td>一个 pod 中的多个容器需要共享彼此的数据 ，emptyDir 的数据随着容器的消亡也会销毁</td></tr><tr><td>hostPath</td><td>用于将目录从工作节点的文件系统挂载到 pod 中</td><td>不常用，缺点是，pod 的调度不是固定的，也就是当 pod 消失后 deployment 重新创建一个 pod，而这 pod 如果不是被调度到之前 pod 的节点，那么该 pod 就不能访问之前的数据</td></tr><tr><td>configMap</td><td>用于将非敏感的数据保存到键值对中，使用时可以使用作为环境变量、命令行参数 arg，存储卷被 pods 挂载使用</td><td>将应用程序的不敏感配置文件创建为 configmap 卷，在 pod 中挂载 configmap 卷，可是实现热更新</td></tr><tr><td>secret</td><td>主要用于存储和管理一些敏感数据，然后通过在 Pod 的容器里挂载 Volume 的方式或者环境变量的方式访问到这些 Secret 里保存的信息了，pod 会自动解密 Secret 的信息</td><td>将应用程序的账号密码等敏感信息通过 secret 卷的形式挂载到 pod 中使用</td></tr><tr><td>downwardApi</td><td>主要用于暴露 pod 元数据，如 pod 的名字</td><td>pod 中的应用程序需要指定 pod 的 name 等元数据，就可以通过 downwardApi 卷的形式挂载给 pod 使用</td></tr><tr><td>projected</td><td>这是一种特殊的卷，用于将上面这些卷一次性的挂载给 pod 使用</td><td>将上面这些卷一次性的挂载给 pod 使用</td></tr><tr><td>pvc</td><td>pvc 是存储卷声明</td><td>通常会创建 pvc 表示对存储的申请，然后在 pod 中使用 pvc</td></tr><tr><td>网络存储卷</td><td>pod 挂载网络存储卷，这样就能将数据持久化到后端的存储里</td><td>常见的网络存储卷有 nfs 存储、glusterfs 卷、ceph rbd 存储卷</td></tr></tbody></table>

### pv 的访问模式有哪几种

pv 的访问模式有 3 种，如下：

*   ReadWriteOnce，简写：RWO 表示，只仅允许单个节点以读写方式挂载；
    
*   ReadOnlyMany，简写：ROX 表示，可以被许多节点以只读方式挂载；
    
*   ReadWriteMany，简写：RWX 表示，可以被多个节点以读写方式挂载；
    

### pv 的回收策略有哪几种

主要有 3 中回收策略：retain 保留、delete 删除、 Recycle 回收。

*   Retain：保留，该策略允许手动回收资源，当删除 PVC 时，PV 仍然存在，PV 被视为已释放，管理员可以手动回收卷。
    
*   Delete：删除，如果 Volume 插件支持，删除 PVC 时会同时删除 PV，动态卷默认为 Delete，目前支持 Delete 的存储后端包括 AWS EBS，GCE PD，Azure Disk，OpenStack Cinder 等。
    
*   Recycle：回收，如果 Volume 插件支持，Recycle 策略会对卷执行 rm -rf 清理该 PV，并使其可用于下一个新的 PVC，但是本策略将来会被弃用，目前只有 NFS 和 HostPath 支持该策略。（这种策略已经被废弃，不用记）
    

### 在 pv 的生命周期中，一般有几种状态

pv 一共有 4 中状态，分别是：

创建 pv 后，pv 的的状态有以下 4 种：Available（可用）、Bound（已绑定）、Released（已释放）、Failed（失败）

```
Available，表示pv已经创建正常，处于可用状态；
Bound，表示pv已经被某个pvc绑定，注意，一个pv一旦被某个pvc绑定，那么该pvc就独占该pv，其他pvc不能再与该pv绑定；
Released，表示pvc被删除了，pv状态就会变成已释放；
Failed，表示pv的自动回收失败；


```

### pv 存储空间不足怎么扩容?

一般的，我们会使用动态分配存储资源，

在创建 storageclass 时指定参数 allowVolumeExpansion：true，表示允许用户通过修改 pvc 申请的存储空间自动完成 pv 的扩容，

当增大 pvc 的存储空间时，不会重新创建一个 pv，而是扩容其绑定的后端 pv。

这样就能完成扩容了。但是 allowVolumeExpansion 这个特性只支持扩容空间不支持减少空间。

### 存储类的资源回收策略:

主要有 2 中回收策略，delete 删除，默认就是 delete 策略、retain 保留。  
Retain：保留，该策略允许手动回收资源，当删除 PVC 时，PV 仍然存在，PV 被视为已释放，管理员可以手动回收卷。  
Delete：删除，如果 Volume 插件支持，删除 PVC 时会同时删除 PV，动态卷默认为 Delete，目前支持 Delete 的存储后端包括 AWS EBS，GCE PD，Azure Disk，OpenStack Cinder 等。

注意：使用存储类动态创建的 pv 默认继承存储类的回收策略，当然当 pv 创建后你也可以手动修改 pv 的回收策略。

### 怎么使一个 node 脱离集群调度，比如要停机维护单又不能影响业务应用

使用 kubectl drain 命令

### k8s 生产中遇到什么特别映像深刻的问题吗，问题排查解决思路是怎么样的？（重点）

（此问题被问到的概率高达 90%，所以可以自己准备几个自己在生产环境中遇到的问题进行讲解）

答：前端的 lb 负载均衡服务器上的 keepalived 出现过脑裂现象。

1、当时问题现象是这样的，vip 同时出现在主服务器和备服务器上，但业务上又没受到影响；  
2、这时首先去查看备服务器上的 keepalived 日志，发现有日志信息显示凌晨的时候备服务器出现了 vrrp 协议超时，所以才导致了备服务器接管了 vip；查看主服务器上的 keepalived 日志，没有发现明显的报错信息，继续查看主服务器和备服务器上的 keepalived 进程状态，都是 running 状态的；查看主服务器上检测脚本所检测的进程，其进程也是正常的，也就是说主服务器根本没有成功执行检测脚本（成功执行检查脚本是会 kill 掉 keepalived 进程，脚本里面其实就是配置了检查 nginx 进程是否存活，如果检查到 nginx 不存活则 kill 掉 keepalived，这样来实现备服务器接管 vip）；  
3、排查服务器上的防火墙、selinux，防火墙状态和 selinux 状态都是关闭着的；  
4、使用 tcpdump 工具在备服务器上进行抓取数据包分析，分析发现，现在确实是备接管的 vip，也确实是备服务器也在对外发送 vrrp 心跳包，所以现在外部流量应该都是流入备服务器上的 vip；  
5、怀疑：主服务器上设置的 vrrp 心跳包时间间隔太长，以及检测脚本设置的检测时间设置不合理导致该问题；  
6、修改 vrrp 协议的心跳包时间间隔，由原来的 2 秒改成 1 秒就发送一次心跳包；检测脚本的检测时间也修改短一点，同时还修改检测脚本的检测失败的次数，比如连续检测 2 次失败才认定为检测失败；  
7、重启主备上的 keepalived，现在 keepalived 是正常的，主服务器上有 vip，备服务器上没有 vip；  
8、持续观察：第二天又发现 keepalived 出现过脑裂现象，vip 又同时出现在主服务器和备服务器上，又是凌晨的时候备服务器显示 vrrp 心跳包超时，所以才导致备服务器接管了 vip；  
9、同样的时间，都是凌晨，vrrp 协议超时；很奇怪，很有理由怀疑是网络问题，询问第三方厂家上层路由器是否禁止了 vrrp 协议，第三方厂家回复，没有禁止 vrrp 协议；  
10、百度、看官方文档求解；  
11、百度、看官网文档得知，keepalived 有 2 种传播模式，一种是组播模式，一种是单播模式，keepalived 默认在组播模式下工作，主服务器会往主播地址 224.0.0.18 发送心跳包，当局域网内有多个 keepalived 实例的时候，如果都用主播模式，会存在冲突干扰的情况，所以官方建议使用单播模式通信，单播模式就是点对点通行，即主向备服务器一对一的发送心跳包；

12、将 keepalived 模式改为单播模式，继续观察，无再发生脑裂现象。问题得以解决。

### k8s 生产中遇到什么特别映像深刻的问题吗，问题排查解决思路是怎么样的？（重点）

参考答案二：测试环境二进制搭建 etcd 集群，etcd 集群出现 2 个 leader 的现象。  
1、问题现象就是：刚搭建的 k8s 集群，是测试环境的，搭建完成之后发现，使用 kubectl get nodes 显示没有资源，kubectl get namespace 一会能正常显示全部的命名空间，一会又显示不了命名空间，这种奇怪情况。  
2、当时经验不是很足，第一点想到的是不是因为网络插件 calico 没装导致的，但是想想，即使没有安装网络插件，最多是 node 节点状态是 notready，也不可能是没有资源发现呀；  
3、然后想到 etcd 数据库，k8s 的资源都是存储在 etcd 数据库中的；  
4、查看 etcd 进程服务的启动状态，发现 etcd 服务状态是处于 running 状态，但是日志有大量的报错信息，日志大概报错信息就是集群节点的 id 不匹配，存在冲突等等报错信息；  
5、使用 etcdctl 命令查看 etcd 集群的健康状态，发现集群是 health 状态，但是居然显示有 2 个 leader，这很奇怪（当初安装 etcd 的时候其实也只是简单看到了集群是健康状态，然后没注意到有 2 个 leader，也没太关注 etcd 服务进程的日志报错信息，以为 etcd 集群状态是 health 状态就可以了）  
6、现在 etcd 出现了 2 个 leader，肯定是存在问题的；  
7、全部检测一遍 etcd 的各个节点的配置文件，确认配置文件里面各个参数配置都没有问题，重启 etcd 集群，报错信息仍未解决，仍然存在 2 个 leader；  
8、尝试把其中一个 leader 节点踢出集群，然后再重新添加它进入集群，仍然是报错，仍然显示有 2 个 leader；  
9、尝试重新生成 etcd 的证书，重新颁发 etcd 的证书，问题仍然存在，仍然显示有 2 个 leader；日志仍是报错集群节点的 id 不匹配，存在冲突；  
10、计算 etcd 命令的 MD5 值，确保各个节点的 etcd 命令是相同的，确保在 scp 传输的时候没有损耗等等，问题仍未解决；  
11、无解，请求同事，架构师介入帮忙排查问题，仍未解决；  
12、删除全部 etcd 相关的文件，重新部署 etcd 集群，etcd 集群正常了，现在只有一个 leader，使用命令 kubectl get nodes 查看节点，也能正常显示了；  
13、最终问题的原因也没有定位出来，只能怀疑是环境问题了，由于是刚部署的 k8s 测试环境，etcd 里面没有数据，所以可以删除重新创建 etcd 集群，如果是线上环境的 etcd 集群出现这种问题，就不能随便删除 etcd 集群了，必须要先进行数据备份才能进行其他方法的处理。

### etcd 集群节点可以设置为偶数个吗，为什么要设置为基数个呢？

不能，也不建议这么设置。

底层的原理，涉及到集群的脑裂 ，

具体的答案，请参考 《尼恩 java 面试宝典 专题 14》

![](https://i-blog.csdnimg.cn/blog_migrate/1c7a4f64bdabfd6e8380b645d12a4698.png)

![](https://i-blog.csdnimg.cn/blog_migrate/41eab812eae3b77ff7dd1e1c25cf55da.png)

### etcd 集群节点之间是怎么同步数据的？

总体而言，是 通过 Raft 协议进行节点之间数据同步， 保证节点之间的数据一致性

在正式开始介绍 Raft 协议之间，我们有必要简单介绍一下其相关概念。

在现实的场景中，节点之间的一致性也就很难保证，这样就需要 Paxos、Raft 等一致性协议。

一致性协议可以保证在集群中大部分节点可用的情况下，集群依然可以工作并给出一个正确的结果，从而保证依赖于该集群的其他服务不受影响。

这里的 “大部分节点可用” 指的是集群中超过半数以上的节点可用，例如，集群中共有 5 个节点，此时其中有 2 个节点出现故障宕机，剩余的可用节点数为 3，此时，集群中大多数节点处于可用的状态，从外部来看集群依然是可用的。

常见的一致性算法有 Paxos、Raft 等，

Paxos 协议是 Leslie Lamport 于 1990 年提出的一种基于消息传递的、具有高度容错特性的一致性算法，Paxos 算法解决的主要问题是分布式系统内如何就某个值达成一致。在相当长的一段时间内，Paxos 算法几乎成为一致性算法的代名词，

但是 Paxos 有两个明显的缺点：第一个也是最明显的缺点就是 Paxos 算法难以理解，Paxos 算法的论文本身就比较晦涩难懂，要完全理解 Paxos 协议需要付出较大的努力，很多经验丰富的开发者在看完 Paxos 论文之后，无法将其有效地应用到具体工程实践中，这明显增加了工程化的门槛，也正因如此，才出现了几次用更简单的术语来解释 Paxos 的尝试。

Paxos 算法的第二个缺点就是它没有提供构建现实系统的良好基础，也有很多工程化 Paxos 算法的尝试，但是它们对 Paxos 算法本身做了比较大的改动，彼此之间的实现差距都比较大，实现的功能和目的都有所不同，同时与 Paxos 算法的描述有很多出入。例如，著名 Chubby，它实现了一个类 Paxos 的算法，但其中很多细节并未被明确。本章并不打算详细介绍 Paxos 协议的相关内容，如果读者对 Paxos 感兴趣，则可以参考 Lamport 发表的三篇论文：《The Part-Time Parliament》、《Paxos made simple》、《Fast Paxos》。

Raft 算法是一种用于管理复制日志的一致性算法，其功能与 Paxos 算法相同类似，但其算法结构和 Paxos 算法不同，在设计 Raft 算法时设计者就将易于理解作为其目标之一，这使得 Raft 算法更易于构建实际的系统，大幅度减少了工程化的工作量，也方便开发者此基础上进行扩展。

Raft 协议中，核心就是用于：

*   Leader 选举
*   日志复制。

#### Leader 选举

Raft 协议的工作模式是一个 Leader 节点和多个 Follower 节点的模式，也就是常说的 Leader-Follower 模式。

在 Raft 协议中，每个节点都维护了一个状态机，该状态机有三种状态，分别是 Leader 状态、Follower 状态和 Candidate 状态，在任意时刻，集群中的任意一个节点都处于这三个状态之一。

各个状态和转换条件如图所示。  
![](https://i-blog.csdnimg.cn/blog_migrate/bfbba9b70f9f39dcb0e7a021ca5492b5.png)

在多数情况下，集群中有一个 Leader 节点，其他节点都处于 Follower 状态，下面简单介绍一下每个状态的节点负责的主要工作。

*   Leader 节点负责处理所有客户端的请求，当接收到客户端的写入请求时，Leader 节点会在本地追加一条相应的日志，然后将其封装成消息发送到集群中其他的 Follower 节点。当 Follower 节点收到该消息时会对其进行响应。如果集群中多数（超过半数）节点都已收到该请求对应的日志记录时，则 Leader 节点认为该条日志记录已提交（committed），可以向客户端返回响应。Leader 还会处理客户端的只读请求，其中涉及一个简单的优化，后面介绍具体实现时，再进行详细介绍。Leader 节点的另一项工作是定期向集群中的 Follower 节点发送心跳消息，这主要是为了防止集群中的其他 Follower 节点的选举计时器超时而触发新一轮选举。
    
*   Follower 节点不会发送任何请求，它们只是简单地响应来自 Leader 或者 Candidate 的请求；Follower 节点也不处理 Client 的请求，而是将请求重定向给集群的 Leader 节点进行处理。
    
*   Candidate 节点是由 Follower 节点转换而来的，当 Follower 节点长时间没有收到 Leader 节点发送的心跳消息时，则该节点的选举计时器就会过期，同时会将自身状态转换成 Candidate，发起新一轮选举。选举的具体过程在下面详细描述。
    

了解了 Raft 协议中节点的三种状态及各个状态下节点的主要行为之后，我们通过一个示例介绍 Raft 协议中 Leader 选举的大致流程。为了方便描述，我们假设当前集群中有三个节点（A、B、C），如图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/ea56e65d3208d58b89ae30355c160a09.png)

在 Raft 协议中有两个时间控制 Leader 选举发生，其中一个是选举超时时间（election timeout），每个 Follower 节点在接收不到 Leader 节点的心跳消息之后，并不会立即发起新一轮选举，而是需要等待一段时间之后才切换成 Candidate 状态发起新一轮选举。这段等待时长就是这里所说的 election timeout（后面介绍 etcd 的具体实现时会提到，Follower 节点等待的时长并不完全等于该配置）。之所以这样设计，主要是 Leader 节点发送的心跳消息可能因为瞬间的网络延迟或程序瞬间的卡顿而迟到（或是丢失），因此就触发新一轮选举是没有必要的。election timeout 一般设置为 150ms～300ms 之间的随机数。另一个超时时间是心跳超时时间（heartbeat timeout），也就是 Leader 节点向集群中其他 Follower 节点发送心跳消息的时间间隔。

当集群初始化时，所有节点都处于 Follower 的状态，此时的集群中没有 Leader 节点。当 Follower 节点一段时间（选举计时器超时）内收不到 Leader 节点的心跳消息，则认为 Leader 节点出现故障导致其任期（Term）过期，Follower 节点会转换成 Candidate 状态，发起新一轮的选举。所谓 “任期（Term）”，实际上就是一个全局的、连续递增的整数，在 Raft 协议中每进行一次选举，任期（Term）加一，在每个节点中都会记录当前的任期值（currentTerm）。每一个任期都是从一次选举开始的，在选举时，会出现一个或者多个 Candidate 节点尝试成为 Leader 节点，如果其中一个 Candidate 节点赢得选举，则该节点就会切换为 Leader 状态并成为该任期的 Leader 节点，直到该任期结束。

回到前面的示例中，此时节点 A 由于长时间未收到 Leader 的心跳消息，就会切换成为 Candidate 状态并发起选举（节点 A 的选举计时器（election timer）已被重置）。

在选举过程中，节点 A 首先会将自己的选票投给自己，并会向集群中其他节点发送选举请求（Request Vote）以获取其选票，如图 2-3（1）所示；此时的节点 B 和节点 C 还都是处于 Term=0 的任期之中，且都是 Follower 状态，均未投出 Term=1 任期中的选票，所以节点 B 和节点 C 在接收到节点 A 的选举请求后会将选票投给节点 A，另外，节点 B、C 在收到节点 A 的选举请求的同时会将选举定时器重置，这是为了防止一个任期中同时出现多个 Candidate 节点，导致选举失败，如图 2-3 （2）所示。

注意，节点 B 和节点 C 也会递增自身记录的 Term 值。  
![](https://i-blog.csdnimg.cn/blog_migrate/fddac9c4175afad2acf1db7876bf1d89.png)

在节点 A 收到节点 B、C 的投票之后，其收到了集群中超过半数的选票，所以在 Term=1 这个任期中，该集群的 Leader 节点就是节点 A，其他节点将切换成 Follower 状态，如图 2-4 所示。

另外需要读者了解的是，集群中的节点除了记录当期任期号（currentTerm），还会记录在该任期中当前节点的投票结果（VoteFor）。

![](https://i-blog.csdnimg.cn/blog_migrate/cb9127e51332c5339c53f0891b69caf3.png)

继续前面的示例，成为 Term=1 任期的 Leader 节点之后，节点 A 会定期向集群中的其他节点发送心跳消息，如图 2-5（1）所示，

这样就可以防止节点 B 和节点 C 中的选举计时器（election timer）超时而触发新一轮的选举；当节点 B 和节点 C（Follower）收到节点 A 的心跳消息之后会重置选举计时器，如图 2-5（2）所示，由此可见，心跳超时时间（heartbeat timeout）需要远远小于选举超时时间（election timeout）

![](https://i-blog.csdnimg.cn/blog_migrate/938f01000ad3da93447b4b02e5d98db7.png)

到这里读者可能会问，如果有两个或两个以上节点的选举计时器同时过期，则这些节点会同时由 Follower 状态切换成 Candidate 状态，然后同时触发新一轮选举，在该轮选举中，每个 Candidate 节点获取的选票都不到半数，无法选举出 Leader 节点，那么 Raft 协议会如何处理呢？这种情况确实存在，假设集群中有 4 个节点，其中节点 A 和节点 B 的选举计时器同时到期，切换到 Candidate 状态并向集群中其他节点发出选举请求，如图 2-6（1）所示。

这里假设节点 A 发出的选举请求先抵达节点 C，节点 B 发出的选举请求先抵达节点 D，如图 2-6（2）所示，节点 A 和节点 B 除了得到自身的选票之外，还分别得到了节点 C 和节点 D 投出的选票，得票数都是 2，都没有超过半数。在这种情况下，Term=4 这个任期会以选举失败结束，随着时间的流逝，当任意节点的选举计时器到期之后，会再次发起新一轮的选举。前面提到过 election timeout 是在一个时间区间内取的随机数，所以在配置合理的时候，像上述情况多次出现的概率并不大。

![](https://i-blog.csdnimg.cn/blog_migrate/f8e8e13978ec6f9db8bc0e8cf74b731e.png)

继续上面的示例，这里假设节点 A 的选举计时器再次到期（此次节点 B、C、D 的选举计时器并未到期），它会切换成 Candidate 状态并发起新一轮选举（Term=5），如图 2-7（1）所示，其中节点 B 虽然处于 Candidate 状态，但是接收到 Term 值比自身记录的 Term 值大的请求时，节点会切换成 Follower 状态并更新自身记录的 Term 值，所以该示例中的节点 B 也会将选票投给节点 A，如图 2-7（2）所示。

![](https://i-blog.csdnimg.cn/blog_migrate/b8d70b44823de39f86e2c2376001a62a.png)

在获取集群中半数以上的选票并成为新任期（Term=5）的 Leader 之后，节点 A 会定期向集群中其他节点发送心跳消息；当集群中其他节点收到 Leader 节点的心跳消息的时候，会重置选举定时器，如图 2-8 所示。

![](https://i-blog.csdnimg.cn/blog_migrate/255eaa012caf92f11ed113e1248104c9.png)

介绍完集群启动时的 Leader 选举流程之后，下面分析 Leader 节点宕机之后重新选举的场景。继续上述 4 节点集群的示例，在系统运行一段时间后，集群当前的 Leader 节点（A）因为故障而宕机，此时将不再有心跳消息发送到集群的其他 Follower 节点（节点 B、C、D），一段时间后，会有一个 Follower 节点的选举计时器最先超时，这里假设节点 D 的选举计时器最先超时，然后它将切换为 Candidate 状态并发起新一轮选举，如图 2-9（1）所示。  
![](https://i-blog.csdnimg.cn/blog_migrate/2e3fb8118839a96edd005976bddaf0fe.png)

当节点 B 和节点 C 收到节点 D 的选举请求后，会将其选票投给节点 D，由于节点 A 已经宕机，没有参加此次选举，也就无法进行投票，但是在此轮选举中，节点 D 依然获得了半数以上的选票，故成为新任期（Term=6）的 Leader 节点，并开始向其他 Follower 节点发送心跳消息，如图 2-10 所示。

![](https://i-blog.csdnimg.cn/blog_migrate/5ad9da81a5d4ecde62d06f13127e73df.png)

当节点 A 恢复之后，会收到节点 D 发来的心跳消息，该消息中携带的任期号（Term=6）大于节点 A 当前记录的任期号（Term=5），所以节点 A 会切换成 Follower 状态。在 Raft 协议中，当某个节点接收到的消息所携带的任期号大于当前节点本身记录的任期号，那么该节点会更新自身记录的任期号，同时会切换为 Follower 状态并重置选举计时器，这是 Raft 算法中所有节点最后请读者考虑一个场景：如果集群中选出的 Leader 节点频繁崩溃或是其他原因导致选举频繁发生，这会使整个集群中没有一个稳定的 Leader 节点，这样客户端无法与集群中的 Leader 节点正常交互，也就会导致整个集群无法正常工作。

Leader 选举是 Raft 算法中对时间要求较为严格的一个点，一般要求整个集群中的时间满足如下不等式：  
广播时间 ＜＜ 选举超时时间 ＜＜ 平均故障间隔时间

在上述不等式中，广播时间指的是从一个节点发送心跳消息到集群中的其他节点并接收响应的平均时间；平均故障间隔时间就是对于一个节点而言，两次故障之间的平均时间。为了保证整个 Raft 集群可用，广播时间必须比选举超时时间小一个数量级，这样 Leader 节点才能够发送稳定的心跳消息来重置其他 Follower 节点的选举计时器，从而防止它们切换成 Candidate 状态，触发新一轮选举。在前面的描述中也提到过，选举超时时间是一个随机数，通过这种随机的方式，会使得多个 Candidate 节点瓜分选票的情况明显减少，也就减少了选举耗时。

另外，选举超时时间应该比平均故障间隔时间小几个数量级，这样 Leader 节点才能稳定存在，整个集群才能稳定运行。当 Leader 节点崩溃之后，整个集群会有大约相当于选举超时的时间不可用，这种情况占比整个集群稳定运行的时间还是非常小的。

广播时间和平均故障间隔时间是由网络和服务器本身决定的，但是选举超时时间是可以由我们自己调节的。

一般情况下，广播时间可以做到 0.5ms～50ms，选举超时时间设置为 200ms～1s 之间，而大多数服务器的平均故障间隔时间都在几个月甚至更长，很容易满足上述不等式的时间需求。

#### 日志复制

通过上一节介绍的 Leader 选举过程，集群中最终会选举出一个 Leader 节点，而集群中剩余的其他节点将会成为 Follower 节点。

Leader 节点除了向 Follower 节点发送心跳消息，**还会处理客户端的请求**，并将客户端的更新操作以消息（Append Entries 消息）的形式发送到集群中所有的 Follower 节点。

当 Follower 节点记录收到的这些消息之后，会向 Leader 节点返回相应的响应消息。当 Leader 节点在收到半数以上的 Follower 节点的响应消息之后，会对客户端的请求进行应答。

最后，Leader 会提交客户端的更新操作，该过程会发送 Append Entries 消息到 Follower 节点，通知 Follower 节点该操作已经提交，同时 Leader 节点和 Follower 节点也就可以将该操作应用到自己的状态机中。

上面这段描述仅仅是 Raft 协议中日志复制部分的大致流程，下面我们依然通过一个示例描述该过程，为了方便描述，我们依然假设当前集群中有三个节点（A、B、C），其中 A 是 Leader 节点，B、C 是 Follower 节点，此时有一个客户端发送了一个更新操作到集群，如图 2-11（1）所示。前面提到过，集群中只有 Leader 节点才能处理客户端的更新操作，这里假设客户端直接将请求发给了节点 A。当收到客户端的请求时，节点 A 会将该更新操作记录到本地的 Log 中，如图 2-11（2）所示。

![](https://i-blog.csdnimg.cn/blog_migrate/67458be8e3b38ad73a720d4a1af8582d.png)

之后，节点 A 会向其他节点发送 Append Entries 消息，其中记录了 Leader 节点最近接收到的请求日志，如图 2-12（1）所示。集群中其他 Follower 节点收到该 Append Entries 消息之后，会将该操作记录到本地的 Log 中，并返回相应的响应消息，如图 2-12（2）所示。

![](https://i-blog.csdnimg.cn/blog_migrate/5f6c3e6108567fa382b60ae778856cdc.png)

当 Leader 节点收到半数以上的响应消息之后，会认为集群中有半数以上的节点已经记录了该更新操作，Leader 节点会将该更新操作对应的日志记录设置为已提交（committed），并应用到自身的状态机中。同时 Leader 节点还会对客户端的请求做出响应，如图 2-13（1）所示。同时，Leader 节点也会向集群中的其他 Follower 节点发送消息，通知它们该更新操作已经被提交，Follower 节点收到该消息之后，才会将该更新操作应用到自己的状态机中，如图 2-13（2）所示。

![](https://i-blog.csdnimg.cn/blog_migrate/92cd68b7b63839c17ea8194533e74070.png)

在上述示例的描述中我们可以看到，集群中各个节点都会维护一个本地 Log 用于记录更新操作，除此之外，每个节点还会维护 commitIndex 和 lastApplied 两个值，它们是本地 Log 的索引值，其中 commitIndex 表示的是当前节点已知的、最大的、已提交的日志索引值，lastApplied 表示的是当前节点最后一条被应用到状态机中的日志索引值。当节点中的 commitIndex 值大于 lastApplied 值时，会将 lastApplied 加 1，并将 lastApplied 对应的日志应用到其状态机中。

在 Leader 节点中不仅需要知道自己的上述信息，还需要了解集群中其他 Follower 节点的这些信息，例如，Leader 节点需要了解每个 Follower 节点的日志复制到哪个位置，从而决定下次发送 Append Entries 消息中包含哪些日志记录。为此，Leader 节点会维护 nextIndex[] 和 matchIndex[] 两个数组，这两个数组中记录的都是日志索引值，其中 nextIndex[] 数组记录了需要发送给每个 Follower 节点的下一条日志的索引值，matchIndex[] 表示记录了已经复制给每个 Follower 节点的最大的日志索引值。

这里简单看一下 Leader 节点与某一个 Follower 节点复制日志时，对应 nextIndex 和 matchIndex 值的变化：Follower 节点中最后一条日志的索引值大于等于该 Follower 节点对应的 nextIndex 值，那么通过 Append Entries 消息发送从 nextIndex 开始的所有日志。之后，Leader 节点会检测该 Follower 节点返回的相应响应，如果成功则更新相应该 Follower 节点对应的 nextIndex 值和 matchIndex 值；如果因为日志不一致而失败，则减少 nextIndex 值重试。

下面我们依然通过一个示例来说明 nextIndex[] 和 matchIndex[] 在日志复制过程中的作用，假设集群现在有三个节点，其中节点 A 是 Leader 节点（Term=1），而 Follower 节点 C 因为宕机导致有一段时间未与 Leader 节点同步日志。此时，节点 C 的 Log 中并不包含全部的已提交日志，而只是节点 A 的 Log 的子集，节点 C 故障排除后重新启动，当前集群的状态如图 2-14 所示（这里只关心 Log、nextIndex[]、matchIndex[]，其他的细节省略，另外需要注意的是，图中的 Term=1 表示的是日志发送时的任期号，而非当前的任期号）。

![](https://i-blog.csdnimg.cn/blog_migrate/9c82153cb9354b9f4781e46eb566106d.png)

A 作为 Leader 节点，记录了 nextIndex[] 和 matchIndex[]，所以知道应该向节点 C 发送哪些日志，在本例中，Leader 节点在下次发送 Append Entries 消息时会携带 Index=2 的消息（这里为了描述简单，每条消息只携带单条日志，Raft 协议采用批量发送的方式，这样效率更高），如图 2-15（1）所示。当节点 C 收到 Append Entries 消息后，会将日志记录到本地 Log 中，然后向 Leader 节点返回追加日志成功的响应，当 Leader 节点收到响应之后，会递增节点 C 对应的 nextIndex 和 matchIndex，这样 Leader 节点就知道下次发送日志的位置了，该过程如图 2-15（2）所示。

在上例中，当 Leader 节点并未发生过切换，所以 Leader 节点始终准确地知道节点 C 对应 nextIndex 值和 matchIndex 值。

如果在上述示例中，在节点 C 故障恢复后，节点 A 宕机后重启，并且导致节点 B 成为新任期（Term=2）的 Leader 节点，则此时节点 B 并不知道旧 Leader 节点中记录的 nextIndex[] 和 matchIndex[] 信息，所以新 Leader 节点会重置 nextIndex[] 和 matchIndex[]，其中会将 nextIndex[] 全部重置为其自身 Log 的最后一条已提交日志的 Index 值，而 matchIndex[] 全部重置为 0，如图 2-16 所示。

![](https://i-blog.csdnimg.cn/blog_migrate/aa8de78e3442b74170cf1ebef00a6cb3.png)

![](https://i-blog.csdnimg.cn/blog_migrate/22f3a0ad024e6ea369b0ac86f18425ca.png)

随后，新任期中的 Leader 节点会向其他节点发送 Append Entries 消息，如图 2-17（1）所示，节点 A 已经拥有了当前 Leader 的全部日志记录，所以会返回追加成功的响应并等待后续的日志，而节点 C 并没有 Index=2 和 Index=3 两条日志，所以返回追加日志失败的响应，在收到该响应后，Leader 节点会将 nextIndex 前移，如图 2-17（2）所示。

![](https://i-blog.csdnimg.cn/blog_migrate/bc0cc171575bdf25a7efdd1f0d2242f8.png)

然后新 Leader 节点会再次尝试发送 Append Entries 消息，循环往复，不断减小 nextIndex 值，直至节点 C 返回追加成功的响应，之后就进入了正常追加消息记录的流程，不再赘述。

了解了 Log 日志及节点中基本的数据结构之后，请读者回顾前面描述的选举过程，

其中 Follower 节点的投票过程并不像前面描述的那样简单（先收到哪个 Candidate 节点的选举请求，就将选票投给哪个 Candidate 节点），Follower 节点还需要比较该 Candidate 节点的日志记录与自身的日志记录，拒绝那些日志没有自己新的 Candidate 节点发来的投票请求，确保将选票投给包含了全部已提交（committed）日志记录的 Candidate 节点。

这也就保证了已提交的日志记录不会丢失：Candidate 节点为了成为 Leader 节点，必然会在选举过程中向集群中半数以上的节点发送选举请求，因为已提交的日志记录必须存在集群中半数以上的节点中，这也就意味着每一条已提交的日志记录肯定在这些接收到节点中的至少存在一份。也就是说，记录全部已提交日志的节点和接收到 Candidate 节点的选举请求的节点必然存在交集，如图 2-18 所示。  
![](https://i-blog.csdnimg.cn/blog_migrate/a634125e3fa11e8aa448a117542f7f1a.png)

如果 Candidate 节点上的日志记录与集群中大多数节点上的日志记录一样新，那么其日志一定包含所有已经提交的日志记录，也就可以获得这些节点的投票并成为 Leader。

在比较两个节点的日志新旧时，Raft 协议通过比较两节点日志中的最后一条日志记录的索引值和任期号，以决定谁的日志比较新：首先会比较最后一条日志记录的任期号，如果最后的日志记录的任期号不同，那么任期号大的日志记录比较新；如果最后一条日志记录的任期号相同，那么日志索引较大的 比较新。

这里只是大概介绍一下 Raft 协议的流程和节点使用的各种数据结构，读者需要了解的是 Raft 协议的工作原理，如果对上述数据结构描述感到困惑，在后面介绍 etcd-raft 模块时，还会再次涉及这些数据结构，到时候读者可以结合代码及这里的描述进一步进行分析。

### 请详述 kube-proxy 原理?

​ 答：集群中每个 Node 上都会运行一个 kube-proxy 服务进程，他是 Service 的透明代理兼均衡负载器，其核心功能是将某个 Service 的访问转发到后端的多个 Pod 上。

kube-proxy 通过监听集群状态变更，并对本机 iptables 做修改，从而实现网络路由。

而其中的负载均衡，也是通过 iptables 的特性实现的。

从 V1.8 版本开始，用 IPVS（IP Virtual Server）模式，用于路由规则的配置，主要优势是：

1）为大型集群提供了更好的扩展性和性能。采用哈希表的数据结构，更高效；

2）支持更复杂的负载均衡算法；

3）支持服务器健康检查和连接重试；

4）可以动态修改 ipset 的集合；

### flannel 和 ovs 网络的区别？

​ 答：

1）配置是否自动化：OpenvSwitch（ovs）作为开源的交换机软件，相对比较成熟和稳定，支持各种网络隧道和协议，经历了大型项目 OpenStack 的考验，而 flannel 除了支持建立覆盖网络来实现 Pod 到 Pod 之间的无缝通信之外，还跟 docker、k8s 的架构体系紧密结合，flannel 能感知 k8s 中的 service 对象，然后动态维护自己的路由表，并通过 etcd 来协助 docker 对整个 k8s 集群的 docker0 网段进行规范，而 ovs ，这些操作则需要手动完成，假如集群中有 N 个节点，则需要建立 N(N-1)/2 个 Vxlan 或者 gre 连接，这取决于集群的规模，如果集群的规模很大，则必须通过自动化脚本来初始化，避免出错。

2）是否支持隔离：flannel 虽然很方便实现 Pod 到 Pod 之间的通信，但不能实现多租户隔离，也不能很好地限制 Pod 的网络流量，而 ovs 网络有两种模式：单租户模式和多租户模式，单租户模式直接使用 openvswitch + vxlan 将 k8s 的 pod 网络组成一个大二层，所有的 pod 可以互相通信访问，多租户模式以 Namespace 为维度分配虚拟网络，从而形成一个网络独立用户，一个 Namespace 中的 pod 无法访问其他 Namespace 中的 pod 和 svc 对象；

### k8s 集群外流量怎么访问 Pod？

答：

可以通过 Service 的 NodePort 方式访问，会在所有节点监听同一个端口，比如：30000，访问节点的流量会被重定向到对应的 Service 上面；

### K8S 资源限制 QoS？

​ 答：Quality of Service（Qos）

主要有三种类别：

1）BestEffort：什么都不设置（CPU or Memory），佛系申请资源；

2）Burstable：Pod 中的容器至少一个设置了 CPU 或者 Memory 的请求；

3）Guaranteed：Pod 中的所有容器必须设置 CPU 和 Memory，并且 request 和 limit 值相等；

### k8s 数据持久化的方式有哪些？

​ 答：

1)EmptyDir（空目录）：没有指定要挂载宿主机上的某个目录，直接由 Pod 内保部映射到宿主机上。类似于 docker 中的 manager volume；场景有：a. 只需要临时将数据保存在磁盘上，比如在合并 / 排序算法中；b. 作为两个容器的共享存储，使得第一个内容管理的容器可以将生成的数据存入其中，同时由同一个 webserver 容器对外提供这些页面; emptyDir 的特性：同个 pod 里面的不同容器，共享同一个持久化目录，当 pod 节点删除时，volume 的数据也会被删除。如果仅仅是容器被销毁，pod 还在，则不会影响 volume 中的数据。总结来说：emptyDir 的数据持久化的生命周期和使用的 pod 一致。一般是作为临时存储使用。

2）Hostpath：将宿主机上已存在的目录或文件挂载到容器内部。类似于 docker 中的 bind mount 挂载方式；

3）PersistentVolume（简称 PV）：基于 NFS 服务的 PV，也可以基于 GFS 的 PV。它的作用是统一数据持久化目录，方便管理，PVC 是向 PV 申请应用所需的容量大小，K8s 集群中可能会有多个 PV，PVC 和 PV 若要关联，其定义的访问模式必须一致。定义的 storageClassName 也必须一致，若群集中存在相同的（名字、访问模式都一致）两个 PV，那么 PVC 会选择向它所需容量接近的 PV 去申请，或者随机申请；

### K8S 的基本组成部分？

答：

Master 节点主要有五个组件，分别是 kubectl、api-server、controller-manager、kube-scheduler 和 etcd；

node 节点主要有三个组件，分别是 kubelet、kube-proxy 和 容器运行时 docker 或者 rkt；

kubectl：客户端命令行工具，作为整个系统的操作入口。  
apiserver：以 REST API 服务形式提供接口，作为整个系统的控制入口。  
controller-manager：执行整个系统的后台任务，包括节点状态状况、Pod 个数、Pods 和 Service 的关联等。  
kube-scheduler：负责节点资源管理，接收来自 kube-apiserver 创建 Pods 任务，并分配到某个节点。  
etcd：负责节点间的服务发现和配置共享。  
kube-proxy：运行在每个计算节点上，负责 Pod 网络代理。定时从 etcd 获取到 service 信息来做相应的策略。  
kubelet：运行在每个计算节点上，作为 agent，接收分配该节点的 Pods 任务及管理容器，周期性获取容器状态，反馈给 kube-apiserver。  
DNS：一个可选的 DNS 服务，用于为每个 Service 对象创建 DNS 记录，这样所有的 Pod 就可以通过 DNS 访问服务了。

### K8s 中镜像的下载策略是什么？

​ 答：可通过命令 “kubectl explain pod.spec.containers” 来查看 imagePullPolicy 这行的解释，

K8s 的镜像下载策略有三种：

Always：镜像标签为 latest 时，总是从指定的仓库中获取镜像；

Never：禁止从仓库中下载镜像，也就是说只能使用本地镜像；

IfNotPresent：仅当本地没有对应镜像时，才从目标仓库中下载；

### 标签与标签选择器的作用是什么？

​ 答：标签：是当相同类型的资源对象越来越多的时候，为了更好的管理，可以按照标签将其分为一个组，为的是提升资源对象的管理效率；标签选择器：就是标签的查询过滤条件。

### K8s 的负载均衡器？

​ 答：负载均衡器是暴露服务的最常见和标准方式之一。

根据工作环境使用两种类型的负载均衡器，即内部负载均衡器或外部负载均衡器。内部负载均衡器自动平衡负载并使用所需配置分配容器，而外部负载均衡器将流量从外部负载引导至后端容器；

### kubelet 监控 Node 节点资源使用是通过什么组件来实现的？

​ 答：用 Metrics Server 提供核心指标，包括 Node、Pod 的 CPU 和内存的使用。而 Metrics Server 需要采集 node 上的 cAdvisor 提供的数据资源，

当 kubelet 服务启动时，它会自动启动 cAdvisor 服务，然后 cAdvisor 会实时采集所在节点的性能指标及在节点上运行的容器的性能指标。

kubelet 的启动参数 --cadvisor-port 可自定义 cAdvisor 对外提供服务的端口号，默认是 4194；

### Pod 的状态？

​ 答：

1）Pending：已经创建了 Pod，但是其内部还有容器没有创建；

2）Running：Pod 内部的所有容器都已经创建，只有由一个容器还处于运行状态或者重启状态；

3）Succeeed：Pod 内所有容器均已经成功执行并且退出，不会再重启；

4）Failed：Pod 内所有容器都退出，但至少有一个为退出失败状态；

5）Unknown：由于某种原因不能获取该 Pod 的状态，可能是网络问题；

### deployment/rs 的区别？

​ 答：deployment 是 rs 的超集，提供更多的部署功能，如：回滚、暂停和重启、 版本记录、事件和状态查看、滚动升级和替换升级。

如果能使用 deployment，则不应再使用 rc 和 rs；

### rc/rs 实现原理？

答：

Replication Controller 可以保证 Pod 始终处于规定的副本数，

而当前推荐的做法是使用 Deployment+ReplicaSet，

ReplicaSet 号称下一代的 Replication Controller，当前唯一区别是 RS 支持 set-based selector，

RC 是通过 ReplicationManager 监控 RC 和 RC 内 Pod 的状态，从而增删 Pod，以实现维持特定副本数的功能，RS 也是大致相同；

### kubernetes 服务发现？

答：

1）环境变量： 当你创建一个 Pod 的时候，kubelet 会在该 Pod 中注入集群内所有 Service 的相关环境变量。**需要注意:** 要想一个 Pod 中注入某个 Service 的环境变量，则必须 Service 要先比该 Pod 创建；

2）DNS：可以通过 cluster add-on 方式轻松的创建 KubeDNS 来对集群内的 Service 进行服务发现；

### k8s 发布 (暴露) 服务，servcie 的类型有那些？

答：

kubernetes 原生的，一个 Service 的 ServiceType 决定了其发布服务的方式。

1） ClusterIP：这是 k8s 默认的 ServiceType。通过集群内的 ClusterIP 在内部发布服务。

2）NodePort：这种方式是常用的，用来对集群外暴露 Service，你可以通过访问集群内的每个 NodeIP:NodePort 的方式，访问到对应 Service 后端的 Endpoint。

3）LoadBalancer: 这也是用来对集群外暴露服务的，不同的是这需要 Cloud Provider 的支持，比如 AWS 等。

4）ExternalName：这个也是在集群内发布服务用的，需要借助 KubeDNS(version>= 1.7) 的支持，就是用 KubeDNS 将该 service 和 ExternalName 做一个 Map，KubeDNS 返回一个 CNAME 记录；

### 简述 ETCD 及其特点?

答：etcd 是一个分布式的、高可用的、一致的 key-value 存储数据库，基于 Go 语言实现，主要用于共享配置和服务发现。特点：

1）完全复制：集群中的每个节点都可以使用完整的存档；

2）高可用性：Etcd 可用于避免硬件的单点故障或网络问题；

3）一致性：每次读取都会返回跨多主机的最新写入；

4）简单：包括一个定义良好、面向用户的 API（gRPC）；

5）安全：实现了带有可选的客户端证书身份验证的自动化 TLS；

6）快速：每秒 10000 次写入的基准速度；

7）可靠：使用 Raft 算法实现了强一致、高可用的服务存储目录；

### 简述 ETCD 适应的场景?

​ 答：

1）服务发现：服务发现要解决的也是分布式系统中最常见的问题之一，即在同一个分布式集群中的进程或服务，要如何才能找到对方并建立连接。本质上来说，服务发现就是想要了解集群中是否有进程在监听 udp 或 tcp 端口，并且通过名字就可以查找和连接。

2）消息发布与订阅：在分布式系统中，最实用对的一种组件间的通信方式：消息发布与订阅。构建一个配置共享中心，数据提供者在这个配置中心发布消息，而消息使用者订阅他们关心的主题，一旦主题有消息发布，就会实时通知订阅者。达成集中式管理与动态更新。应用中用到的一些配置信息放到 etcd 上进行集中管理。

3）负载均衡：分布式系统中，为了保证服务的高可用以及数据的一致性，通常都会把数据和服务部署多份，以此达到对等服务，即使其中的某一个服务失效了，也不影响使用。etcd 本身分布式架构存储的信息访问支持负载均衡。

4）分布式通知与协调：通过注册与异步通知机制，实现分布式环境下不同系统之间的通知与协调，从而对数据变更做到实时处理。

5）分布式锁：因为 etcd 使用 Raft 算法保持了数据的强一致性，某次操作存储到集群中的值必然是全局一致的，所以很容易实现分布式锁。锁服务有两种使用方式，一是保持独占，二是控制时序。

6）分布式队列：分布式队列的常规用法与场景五中所描述的分布式锁的控制时序用法类似，即创建一个先进先出的队列，保证顺序。

7）集群监控与 Leader 精选：通过 etcd 来进行监控实现起来非常简单并且实时性强；

### 简述 Kubernetes RC 的机制?

​ 答：Replication Controller 用来管理 Pod 的副本，保证集群中存在指定数量的 Pod 副本。当定义了 RC 并提交至 Kubernetes 集群中之后，Master 节点上的 Controller Manager 组件获悉，并同时巡检系统中当前存活的目标 Pod，并确保目标 Pod 实例的数量刚好等于此 RC 的期望值，若存在过多的 Pod 副本在运行，系统会停止一些 Pod，反之则自动创建一些 Pod；

### 简述 kube-proxy 作用?

答：kube-proxy 运行在所有节点上，它监听 apiserver 中 service 和 endpoint 的变化情况，创建路由规则以提供服务 IP 和负载均衡功能。

简单理解此进程是 Service 的透明代理兼负载均衡器，其核心功能是将到某个 Service 的访问请求转发到后端的多个 Pod 实例上；

### 简述 kube-proxy iptables 原理?

​ 答：Kubernetes 从 1.2 版本开始，将 iptables 作为 kube-proxy 的默认模式。iptables 模式下的 kube-proxy 不再起到 Proxy 的作用，其核心功能：通过 API Server 的 Watch 接口实时跟踪 Service 与 Endpoint 的变更信息，并更新对应的 iptables 规则，Client 的请求流量则通过 iptables 的 NAT 机制 “直接路由” 到目标 Pod；

### 简述 kube-proxy ipvs 原理?

答：IPVS 在 Kubernetes1.11 中升级为 GA 稳定版。

IPVS 则专门用于高性能负载均衡，并使用更高效的数据结构（Hash 表），允许几乎无限的规模扩张，因此被 kube-proxy 采纳为最新模式；

在 IPVS 模式下，使用 iptables 的扩展 ipset，而不是直接调用 iptables 来生成规则链。

iptables 规则链是一个线性的数据结构，ipset 则引入了带索引的数据结构，因此当规则很多时，也可以很高效地查找和匹配；

可以将 ipset 简单理解为一个 IP（段）的集合，这个集合的内容可以是 IP 地址、IP 网段、端口等，iptables 可以直接添加规则对这个 “可变的集合” 进行操作，这样做的好处在于可以大大减少 iptables 规则的数量，从而减少性能损耗；

### 简述 kube-proxy ipvs 和 iptables 的异同?

答：iptables 与 IPVS 都是基于 Netfilter 实现的，但因为定位不同，二者有着本质的差别：

iptables 是为防火墙而设计的；IPVS 则专门用于高性能负载均衡，并使用更高效的数据结构（Hash 表），允许几乎无限的规模扩张。

与 iptables 相比，IPVS 拥有以下明显优势：为大型集群提供了更好的可扩展性和性能；支持比 iptables 更复杂的复制均衡算法（最小负载、最少连接、加权等）；支持服务器健康检查和连接重试等功能；可以动态修改 ipset 的集合，即使 iptables 的规则正在使用这个集合；

### 简述 Kubernetes 中什么是静态 Pod?

答：静态 pod 是由 kubelet 进行管理的仅存在于特定 Node 的 Pod 上，他们不能通过 API Server 进行管理，无法与 ReplicationController、Deployment 或者 DaemonSet 进行关联，并且 kubelet 无法对他们进行健康检查。

静态 Pod 总是由 kubelet 进行创建，并且总是在 kubelet 所在的 Node 上运行；

### 简述 Kubernetes Pod 的常见调度方式?

答：

1）Deployment 或 RC：该调度策略主要功能就是自动部署一个容器应用的多份副本，以及持续监控副本的数量，在集群内始终维持用户指定的副本数量；

2）NodeSelector：定向调度，当需要手动指定将 Pod 调度到特定 Node 上，可以通过 Node 的标签（Label）和 Pod 的 nodeSelector 属性相匹配；

3）NodeAffinity 亲和性调度：亲和性调度机制极大的扩展了 Pod 的调度能力，目前有两种节点亲和力表达：硬规则，必须满足指定的规则，调度器才可以调度 Pod 至 Node 上（类似 nodeSelector，语法不同）；软规则，优先调度至满足的 Node 的节点，但不强求，多个优先级规则还可以设置权重值；

4）Taints 和 Tolerations（污点和容忍）：Taint：使 Node 拒绝特定 Pod 运行；Toleration：为 Pod 的属性，表示 Pod 能容忍（运行）标注了 Taint 的 Node；

### 简述 Kubernetes 初始化容器（init container）?

答：init container 的运行方式与应用容器不同，它们必须先于应用容器执行完成，当设置了多个 init container 时，将按顺序逐个运行，并且只有前一个 init container 运行成功后才能运行后一个 init container。

当所有 init container 都成功运行后，Kubernetes 才会初始化 Pod 的各种信息，并开始创建和运行应用容器；

### 简述 Kubernetes deployment 升级过程?

答：

初始创建 Deployment 时，系统创建了一个 ReplicaSet，并按用户的需求创建了对应数量的 Pod 副本；

当更新 Deployment 时，系统创建了一个新的 ReplicaSet，并将其副本数量扩展到 1，然后将旧 ReplicaSet 缩减为 2；

之后，系统继续按照相同的更新策略对新旧两个 ReplicaSet 进行逐个调整；

最后，新的 ReplicaSet 运行了对应个新版本 Pod 副本，旧的 ReplicaSet 副本数量则缩减为 0；

### 简述 Kubernetes deployment 升级策略?

答：

在 Deployment 的定义中，可以通过 spec.strategy 指定 Pod 更新的策略，

目前支持两种策略：Recreate（重建）和 RollingUpdate（滚动更新），

默认值为 RollingUpdate；

Recreate：设置 spec.strategy.type=Recreate，表示 Deployment 在更新 Pod 时，会先杀掉所有正在运行的 Pod，然后创建新的 Pod；

RollingUpdate：设置 spec.strategy.type=RollingUpdate，表示 Deployment 会以滚动更新的方式来逐个更新 Pod。同时，可以通过设置 spec.strategy.rollingUpdate 下的两个参数（maxUnavailable 和 maxSurge）来控制滚动更新的过程；

### 简述 Kubernetes DaemonSet 类型的资源特性?

答：

DaemonSet 资源对象会在每个 Kubernetes 集群中的节点上运行，并且每个节点只能运行一个 pod，这是它和 deployment 资源对象的最大也是唯一的区别。

因此，在定义 yaml 文件中，不支持定义 replicas。

它的一般使用场景如下：在去做每个节点的日志收集工作。监控每个节点的的运行状态。

### 简述 Kubernetes 自动扩容机制?

答：

Kubernetes 使用 Horizontal Pod Autoscaler（HPA）的控制器实现基于 CPU 使用率进行自动 Pod 扩缩容的功能。

HPA 控制器周期性地监测目标 Pod 的资源性能指标，并与 HPA 资源对象中的扩缩容条件进行对比，在满足条件时对 Pod 副本数量进行调整；

### 简述 Kubernetes Service 分发后端的策略?

答：

1）RoundRobin：默认为轮询模式，即轮询将请求转发到后端的各个 Pod 上；

2）SessionAffinity：基于客户端 IP 地址进行会话保持的模式，即第 1 次将某个客户端发起的请求转发到后端的某个 Pod 上，之后从相同的客户端发起的请求都将被转发到后端相同的 Pod 上；

### 简述 Kubernetes Headless Service?

答：在某些应用场景中，若需要人为指定负载均衡器，不使用 Service 提供的默认负载均衡的功能，或者应用程序希望知道属于同组服务的其他实例。

Kubernetes 提供了 Headless Service 来实现这种功能，即不为 Service 设置 ClusterIP（入口 IP 地址），仅通过 Label Selector 将后端的 Pod 列表返回给调用的客户端；

### 简述 Kubernetes 外部如何访问集群内的服务?

答：

映射 Pod 到物理机：将 Pod 端口号映射到宿主机，即在 Pod 中采用 hostPort 方式，以使客户端应用能够通过物理机访问容器应用；

映射 Service 到物理机：将 Service 端口号映射到宿主机，即在 Service 中采用 nodePort 方式，以使客户端应用能够通过物理机访问容器应用；

映射 Service 到 LoadBalancer：通过设置 LoadBalancer 映射到云服务商提供的 LoadBalancer 地址。这种用法仅用于在公有云服务提供商的云平台上设置 Service 的场景；

### 简述 Kubernetes ingress?

​ 答：

K8s 的 Ingress 资源对象，用于将不同 URL 的访问请求转发到后端不同的 Service，以实现 HTTP 层的业务路由机制。

K8s 使用了 Ingress 策略和 Ingress Controller，两者结合并实现了一个完整的 Ingress 负载均衡器。

使用 Ingress 进行负载分发时，Ingress Controller 基于 Ingress 规则将客户端请求直接转发到 Service 对应的后端 Endpoint（Pod）上，从而跳过 kube-proxy 的转发功能，kube-proxy 不再起作用，

全过程为：ingress controller + ingress 规则 ----> services；

### 简述 Kubernetes 镜像的下载策略?

​ 答：

1）Always：镜像标签为 latest 时，总是从指定的仓库中获取镜像；

2）Never：禁止从仓库中下载镜像，也就是说只能使用本地镜像；

3）IfNotPresent：仅当本地没有对应镜像时，才从目标仓库中下载；默认的镜像下载策略是：当镜像标签是 latest 时，默认策略是 Always；当镜像标签是自定义时（也就是标签不是 latest），那么默认策略是 IfNotPresent；

### 简述 Kubernetes 的负载均衡器?

​ 答：

根据工作环境使用两种类型的负载均衡器，即内部负载均衡器或外部负载均衡器。

内部负载均衡器自动平衡负载并使用所需配置分配容器，而外部负载均衡器将流量从外部负载引导至后端容器；

### 简述 Kubernetes 各模块如何与 API Server 通信?

答：K8s API Server 作为集群的核心，负责集群各功能模块之间的通信。

集群内的各个功能模块通过 API Server 将信息存入 etcd，当需要获取和操作这些数据时，则通过 API Server 提供的 REST 接口（用 GET、LIST 或 WATCH 方法）来实现，从而实现各模块之间的信息交互。

1）kubelet 进程与 API Server 的交互：每个 Node 上的 kubelet 每隔一个时间周期，就会调用一次 API Server 的 REST 接口报告自身状态，API Server 在接收到这些信息后，会将节点状态信息更新到 etcd 中；

2）kube-controller-manager 进程与 API Server 的交互：kube-controller-manager 中的 Node Controller 模块通过 API Server 提供的 Watch 接口实时监控 Node 的信息，并做相应处理

；3）kube-scheduler 进程与 API Server 的交互：Scheduler 通过 API Server 的 Watch 接口监听到新建 Pod 副本的信息后，会检索所有符合该 Pod 要求的 Node 列表，开始执行 Pod 调度逻辑，在调度成功后将 Pod 绑定到目标节点上；

### 简述 Kubernetes Scheduler 作用及实现原理?

​ 答：

Scheduler 是负责 Pod 调度的重要功能模块，负责接收 Controller Manager 创建的新 Pod，为其调度至目标 Node，调度完成后，目标 Node 上的 kubelet 服务进程接管后继工作，负责 Pod 接下来生命周期；

Scheduler 的作用是将待调度的 Pod，按照特定的调度算法和调度策略绑定（Binding）到集群中某个合适的 Node 上，并将绑定信息写入 etcd 中；

Scheduler 通过调度算法调度为待调度 Pod 列表中的每个 Pod 从 Node 列表中选择一个最适合的 Node 来实现 Pod 的调度。随后，目标节点上的 kubelet 通过 API Server 监听到 Kubernetes Scheduler 产生的 Pod 绑定事件，然后获取对应的 Pod 清单，下载 Image 镜像并启动容器；

### 简述 Kubernetes Scheduler 使用哪两种算法将 Pod 绑定到 worker 节点?

​ 答：

1）预选（Predicates）：输入是所有节点，输出是满足预选条件的节点。kube-scheduler 根据预选策略过滤掉不满足策略的 Nodes。如果某节点的资源不足或者不满足预选策略的条件则无法通过预选；

2）优选（Priorities）：输入是预选阶段筛选出的节点，优选会根据优先策略为通过预选的 Nodes 进行打分排名，选择得分最高的 Node。例如，资源越富裕、负载越小的 Node 可能具有越高的排名；

### 简述 Kubernetes kubelet 的作用?

​ 答：

在 Kubernetes 集群中，在每个 Node（又称 Worker）上都会启动一个 kubelet 服务进程。

该进程用于处理 Master 下发到本节点的任务，管理 Pod 及 Pod 中的容器。

每个 kubelet 进程都会在 API Server 上注册节点自身的信息，定期向 Master 汇报节点资源的使用情况，并通过 cAdvisor 监控容器和节点资源；

### 简述 Kubernetes kubelet 监控 Worker 节点资源是使用什么组件来实现的?

​ 答:

kubelet 使用 cAdvisor 对 worker 节点资源进行监控。

在 Kubernetes 系统中，cAdvisor 已被默认集成到 kubelet 组件内，当 kubelet 服务启动时，它会自动启动 cAdvisor 服务，然后 cAdvisor 会实时采集所在节点的性能指标及在节点上运行的容器的性能指标；

### 简述 Kubernetes 如何保证集群的安全性?

​ 答：

1）基础设施方面：保证容器与其所在宿主机的隔离；

2）用户权限：划分普通用户和管理员的角色；

3）API Server 的认证授权：Kubernetes 集群中所有资源的访问和变更都是通过 Kubernetes API Server 来实现的，因此需要建议采用更安全的 HTTPS 或 Token 来识别和认证客户端身份（Authentication），以及随后访问权限的授权（Authorization）环节；

4）API Server 的授权管理：通过授权策略来决定一个 API 调用是否合法。对合法用户进行授权并且随后在用户访问时进行鉴权，建议采用更安全的 RBAC 方式来提升集群安全授权；

5）AdmissionControl（准入机制）：对 kubernetes api 的请求过程中，顺序为：先经过认证 & 授权，然后执行准入操作，最后对目标对象进行操作；

### 简述 Kubernetes 准入机制?

答：

在对集群进行请求时，每个准入控制代码都按照一定顺序执行。

如果有一个准入控制拒绝了此次请求，那么整个请求的结果将会立即返回，并提示用户相应的 error 信息，准入控制（AdmissionControl）准入控制本质上为一段准入代码，在对 kubernetes api 的请求过程中，顺序为：先经过认证 & 授权，然后执行准入操作，最后对目标对象进行操作。

常用组件（控制代码）如下：

AlwaysAdmit：允许所有请求；

AlwaysDeny：禁止所有请求，多用于测试环境；

ServiceAccount：它将 serviceAccounts 实现了自动化，它会辅助 serviceAccount 做一些事情，比如如果 pod 没有 serviceAccount 属性，它会自动添加一个 default，并确保 pod 的 serviceAccount 始终存在；

LimitRanger：观察所有的请求，确保没有违反已经定义好的约束条件，这些条件定义在 namespace 中 LimitRange 对象中；

NamespaceExists：观察所有的请求，如果请求尝试创建一个不存在的 namespace，则这个请求被拒绝；

### 简述 Kubernetes RBAC 及其特点（优势）?

​ 答：

RBAC 是基于角色的访问控制，是一种基于个人用户的角色来管理对计算机或网络资源的访问的方法，

优势：

1）对集群中的资源和非资源权限均有完整的覆盖；

2）整个 RBAC 完全由几个 API 对象完成， 同其他 API 对象一样， 可以用 kubectl 或 API 进行操作；

3）可以在运行时进行调整，无须重新启动 API Server；

### 简述 Kubernetes Secret 作用?

​ 答：

Secret 对象，主要作用是保管私密数据，比如密码、OAuth Tokens、SSH Keys 等信息。

将这些私密信息放在 Secret 对象中比直接放在 Pod 或 Docker Image 中更安全，也更便于使用和分发；

### 简述 Kubernetes Secret 有哪些使用方式?

​ 答：

1）在创建 Pod 时，通过为 Pod 指定 Service Account 来自动使用该 Secret；

2）通过挂载该 Secret 到 Pod 来使用它；

3）在 Docker 镜像下载时使用，通过指定 Pod 的 spc.ImagePullSecrets 来引用它；

### 简述 Kubernetes PodSecurityPolicy 机制?

​ 答：

Kubernetes PodSecurityPolicy 是为了更精细地控制 Pod 对资源的使用方式以及提升安全策略。

在开启 PodSecurityPolicy 准入控制器后，Kubernetes 默认不允许创建任何 Pod，需要创建 PodSecurityPolicy 策略和相应的 RBAC 授权策略（Authorizing Policies），Pod 才能创建成功；

### 简述 Kubernetes PodSecurityPolicy 机制能实现哪些安全策略?

1）特权模式：privileged 是否允许 Pod 以特权模式运行；

2）宿主机资源：控制 Pod 对宿主机资源的控制，如 hostPID：是否允许 Pod 共享宿主机的进程空间；

3）用户和组：设置运行容器的用户 ID（范围）或组（范围）；

4）提升权限：AllowPrivilegeEscalation：设置容器内的子进程是否可以提升权限，通常在设置非 root 用户（MustRunAsNonRoot）时进行设置；

5）SELinux：进行 SELinux 的相关配置；

### 简述 Kubernetes 网络模型?

答：Kubernetes 网络模型中每个 Pod 都拥有一个独立的 IP 地址，不管它们是否运行在同一个 Node（宿主机）中，都要求它们可以直接通过对方的 IP 进行访问；

同时为每个 Pod 都设置一个 IP 地址的模型使得同一个 Pod 内的不同容器会共享同一个网络命名空间，也就是同一个 Linux 网络协议栈。

这就意味着同一个 Pod 内的容器可以通过 localhost 来连接对方的端口；在 Kubernetes 的集群里，IP 是以 Pod 为单位进行分配的。一个 Pod 内部的所有容器共享一个网络堆栈；

### 简述 Kubernetes CNI 模型?

答：

Kubernetes CNI 模型是对容器网络进行操作和配置的规范，通过插件的形式对 CNI 接口进行实现。

CNI 仅关注在创建容器时分配网络资源，和在销毁容器时删除网络资源。

容器（Container）：是拥有独立 Linux 网络命名空间的环境，例如使用 Docker 或 rkt 创建的容器。容器需要拥有自己的 Linux 网络命名空间，这是加入网络的必要条件；

网络（Network）：表示可以互连的一组实体，这些实体拥有各自独立、唯一的 IP 地址，可以是容器、物理机或者其他网络设备（比如路由器）等；

### 简述 Kubernetes 网络策略?

​ 答：

为实现细粒度的容器间网络访问隔离策略，K8s 引入 Network Policy 主要功能是对 Pod 间的网络通信进行限制和准入控制，设置允许访问或禁止访问的客户端 Pod 列表。

Network Policy 定义网络策略，配合策略控制器（Policy Controller）进行策略的实现；

### 简述 Kubernetes 网络策略原理?

​ 答：

Network Policy 的工作原理主要为：policy controller 需要实现一个 API Listener，监听用户设置的 Network Policy 定义，并将网络访问规则通过各 Node 的 Agent 进行实际设置（Agent 则需要通过 CNI 网络插件实现）；

### 简述 Kubernetes 中 flannel 的作用?

​ 答：

1）它能协助 Kubernetes，给每一个 Node 上的 Docker 容器都分配互相不冲突的 IP 地址；

2）它能在这些 IP 地址之间建立一个覆盖网络（Overlay Network），通过这个覆盖网络，将数据包原封不动地传递到目标容器内；

### 简述 Kubernetes Calico 网络组件实现原理?

​ 答：

Calico 是一个基于 BGP 的纯三层的网络方案，与 OpenStack、Kubernetes、AWS、GCE 等云平台都能够良好地集成，Calico 在每个计算节点都利用 Linux Kernel 实现了一个高效的 vRouter 来负责数据转发。每个 vRouter 都通过 BGP 协议把在本节点上运行的容器的路由信息向整个 Calico 网络广播，并自动设置到达其他节点的路由转发规则；Calico 保证所有容器之间的数据流量都是通过 IP 路由的方式完成互联互通的。

Calico 节点组网时可以直接利用数据中心的网络结构（L2 或者 L3），不需要额外的 NAT、隧道或者 Overlay Network，没有额外的封包解包，能够节约 CPU 运算，提高网络效率；

### 简述 Kubernetes 共享存储的作用?

​ 答：

Kubernetes 对于有状态的容器应用或者对数据需要持久化的应用，因此需要更加可靠的存储来保存应用产生的重要数据，以便容器应用在重建之后仍然可以使用之前的数据。因此需要使用共享存储；

### 简述 Kubernetes PV 和 PVC?

答：

PV 是对底层网络共享存储的抽象，将共享存储定义为一种 “资源”；

PVC 则是用户对存储资源的一个 “申请”；

### 简述 Kubernetes PV 生命周期内的阶段?

答：

1）Available：可用状态，还未与某个 PVC 绑定；

2）Bound：已与某个 PVC 绑定；

3）Released：绑定的 PVC 已经删除，资源已释放，但没有被集群回收；

4）Failed：自动资源回收失败；

### 简述 Kubernetes CSI 模型?

答：

CSI 是 Kubernetes 推出与容器对接的存储接口标准，存储提供方只需要基于标准接口进行存储插件的实现，就能使用 Kubernetes 的原生存储机制为容器提供存储服务，CSI 使得存储提供方的代码能和 Kubernetes 代码彻底解耦，部署也与 Kubernetes 核心组件分离；

CSI 包括 CSI Controller：的主要功能是提供存储服务视角对存储资源和存储卷进行管理和操作；Node 的主要功能是对主机（Node）上的 Volume 进行管理和操作；

### 简述 Kubernetes Worker 节点加入集群的过程?

​ 答：在该 Node 上安装 Docker、kubelet 和 kube-proxy 服务； 然后配置 kubelet 和 kubeproxy 的启动参数，将 Master URL 指定为当前 Kubernetes 集群 Master 的地址，最后启动这些服务； 通过 kubelet 默认的自动注册机制，新的 Worker 将会自动加入现有的 Kubernetes 集群中； Kubernetes Master 在接受了新 Worker 的注册之后，会自动将其纳入当前集群的调度范围；

### 简述 Kubernetes Pod 如何实现对节点的资源控制?

​ 答：

Kubernetes 集群里的节点提供的资源主要是计算资源，计算资源是可计量的能被申请、分配和使用的基础资源。当前 Kubernetes 集群中的计算资源主要包括 CPU、GPU 及 Memory。

CPU 与 Memory 是被 Pod 使用的，因此在配置 Pod 时可以通过参数 CPU Request 及 Memory Request 为其中的每个容器指定所需使用的 CPU 与 Memory 量，Kubernetes 会根据 Request 的值去查找有足够资源的 Node 来调度此 Pod；

### 简述 Kubernetes Requests 和 Limits 如何影响 Pod 的调度?

​ 答：

当一个 Pod 创建成功时，Kubernetes 调度器（Scheduler）会为该 Pod 选择一个节点来执行。对于每种计算资源（CPU 和 Memory）而言，每个节点都有一个能用于运行 Pod 的最大容量值。调度器在调度时，首先要确保调度后该节点上所有 Pod 的 CPU 和内存的 Requests 总和，不超过该节点能提供给 Pod 使用的 CPU 和 Memory 的最大容量值；

### 简述 Kubernetes Metric Service?

答：在 Kubernetes 从 1.10 版本后采用 Metrics Server 作为默认的性能数据采集和监控，主要用于提供核心指标（Core Metrics），包括 Node、Pod 的 CPU 和内存使用指标。

对其他自定义指标（Custom Metrics）的监控则由 Prometheus 等组件来完成；

### 简述 Kubernetes 中，如何使用 EFK 实现日志的统一管理？

答：

在 Kubernetes 集群环境中，通常一个完整的应用或服务涉及组件过多，建议对日志系统进行集中化管理，EFK 是 Elasticsearch、Fluentd 和 Kibana 的组合，

Elasticsearch：是一个搜索引擎，负责存储日志并提供查询接口；

Fluentd：负责从 Kubernetes 搜集日志，每个 node 节点上面的 fluentd 监控并收集该节点上面的系统日志，并将处理过后的日志信息发送给 Elasticsearch；

Kibana：提供了一个 Web GUI，用户可以浏览和搜索存储在 Elasticsearch 中的日志；

### 简述 Kubernetes 如何进行优雅的节点关机维护?

​ 答：由于 Kubernetes 节点运行大量 Pod，因此在进行关机维护之前，建议先使用 kubectl drain 将该节点的 Pod 进行驱逐，然后进行关机维护；

### 简述 Kubernetes 集群联邦?

​ 答：Kubernetes 集群联邦可以将多个 Kubernetes 集群作为一个集群进行管理。因此，可以在一个数据中心 / 云中创建多个 Kubernetes 集群，并使用集群联邦在一个地方控制 / 管理所有集群；

### 简述 Helm 及其优势?

​ 答：Helm 是 Kubernetes 的软件包管理工具，Helm 能够将一组 K8S 资源打包统一管理, 是查找、共享和使用为 Kubernetes 构建的软件的最佳方式。 Helm 中通常每个包称为一个 Chart，一个 Chart 是一个目录，优势：1）统一管理、配置和更新这些分散的 k8s 的应用资源文件；2）分发和复用一套应用模板；3）将应用的一系列资源当做一个软件包管理；4）对于应用发布者而言，可以通过 Helm 打包应用、管理应用依赖关系、管理应用版本并发布应用到软件仓库；5）对于使用者而言，使用 Helm 后不用需要编写复杂的应用部署文件，可以以简单的方式在 Kubernetes 上查找、安装、升级、回滚、卸载应用程序；

### 标签与标签选择器的作用是什么?

​ 答：

1）标签可以附加在 kubernetes 任何资源对象之上的键值型数据，常用于标签选择器的匹配度检查，从而完成资源筛选；

2）标签选择器用于表达标签的查询条件或选择标准，Kubernetes API 目前支持两个选择器：基于等值关系（equality-based）的标签选项器以及基于集合关系（set-based）的标签选择器；

### 什么是 Google 容器引擎?

​ 答：Google Container Engine（GKE）是 Docker 容器和集群的开源管理平台。这个基于 Kubernetes 的引擎仅支持在 Google 的公共云服务中运行的群集；

### image 的状态有那些？

​ 答：

1）Running：Pod 所需的容器已经被成功调度到某个节点，且已经成功运行；

2）Pending：APIserver 创建了 pod 资源对象，并且已经存入 etcd 中，但它尚未被调度完成或者仍然处于仓库中下载镜像的过程；

3）Unknown：APIserver 无法正常获取到 pod 对象的状态，通常是其无法与所在工作节点的 kubelet 通信所致；

### Service 这种资源对象的作用是什么?

​ 答：

service 就是将多个 POD 划分到同一个逻辑组中，并统一向外提供服务，POD 是通过 Label Selector 加入到指定的 service 中。

Service 相当于是一个负载均衡器，用户请求会先到达 service，再由 service 转发到它内部的某个 POD 上，通过 services.spec.type 字段来指定：

1）ClusterIP：用于集群内部访问。该类型会为 service 分配一个 IP，集群内部请求先到达 service，再由 service 转发到其内部的某个 POD 上；

2）NodePort：用于集群外部访问。该类型会将 Service 的 Port 映射到集群的每个 Node 节点上，然后在集群之外，就能通过 Node 节点上的映射端口访问到这个 Service；

3）LoadBalancer：用于集群外部访问。该类型是在所有 Node 节点前又挂了一个负载均衡器，作为集群外部访问的统一入口，外部流量会先到达 LoadBalancer，再由它转发到集群的 node 节点上，通过 nodePort 再转发给对应的 service，最后由 service 转发到后端 Pod 中；

4）ExternalName：创建一个 DNS 别名（即 CNAME）并指向到某个 Service Name 上，也就是为某个 Service Name 添加一条 CNAME 记录，当有请求访问这个 CNAME 时会自动解析到这个 Service Name 上；

### 常用的标签分类有哪些?

​ 答：release（版本）：stable（稳定版）、canary（金丝雀版本）、beta（测试版本）、environment（环境变量）：dev（开发）、qa（测试）、production（生产）、application（应用）：ui、as（application software 应用软件）、pc、sc、tier（架构层级）：frontend（前端）、backend（后端）、cache（缓存）、partition（分区）：customerA（客户 A）、customerB（客户 B）、track（品控级别）：daily（每天）、weekly（每周）；

### 说说你对 Job 这种资源对象的了解?

​ 答：

Job 控制一组 Pod 容器，可以通过 Job 这种资源对象定义并启动一个批处理任务的 Job，其中 Job 所控制的 Pod 副本是短暂运行的，可以将其视为一组 Docker 容器，每个 Docker 容器都仅仅运行一次，当 Job 控制的所有 Pod 的副本都运行结束时，对应的 Job 也就结来。

Job 生成的副本是不能自动重启的，对应的 Pod 副本的 RestartPolicy 都被设置为 Never。

Job 所控制的 Pod 副本的工作模式能够多实例并行计算。

### k8s 是怎么进行服务注册的?

​ 答：

1）Service 创建的时候会向 API Server 用 POST 方式提交一个新的 Service 定义，这个请求需要经过认证、鉴权以及其它的准入策略检查过程之后才会放行；

2）CoreDns 会为 Service 创建一个 dns 记录，Service 得到一个 ClusterIP（虚拟 IP 地址），并保存到集群数据仓库；

3）在集群范围内传播 Service 配置；

### Kubernetes 与 Docker Swarm 的区别如何?

​ 答：

1）安装和部署：k8s 安装很复杂; 但是一旦安装完毕，集群就非常强大，Docker Swarm 安装非常简单; 但是集群不是很强大; 2) 图形用户界面：k8s 有，Docker Swarm 无；

3）可伸缩性：k8s 支持，Docker Swarm 比 k8s 快 5 倍；

4）自动伸缩：k8s 有，Docker Swarm 无；

5）负载均衡：k8s 在不同的 Pods 中的不同容器之间平衡负载流量，需要手动干预，Docker Swarm 可以自动平衡集群中容器之间的流量；

6）滚动更新回滚：k8s 支持，Docker Swarm 可以部署滚动更新，但不能自动回滚；

7）数据量：k8s 可以共享存储卷。只能与其他集装箱在同一 Pod，Docker Swarm 可以与任何其他容器共享存储卷；

8）日志记录和监控：k8s 内置的日志和监控工具，Docker Swarm 要用第三方工具进行日志记录和监控；

### 什么是 Container Orchestration?

答：

1）资源编排 - 负责资源的分配，如限制 namespace 的可用资源，scheduler 针对资源的不同调度策略；

2）工作负载编排 - 负责在资源之间共享工作负载，如 Kubernetes 通过不同的 controller 将 Pod 调度到合适的 node 上，并且负责管理它们的生命周期；

3）服务编排 - 负责服务发现和高可用等，如 Kubernetes 中可用通过 Service 来对内暴露服务，通过 Ingress 来对外暴露服务；容器编排常用的控制器有：Deployment 经常被作为无状态实例控制器使用; StatefulSet 是一个有状态实例控制器; DaemonSet 可以指定在选定的 Node 上跑，每个 Node 上会跑一个副本，它有一个特点是它的 Pod 的调度不经过调度器，在 Pod 创建的时候就直接绑定 NodeName；最后一个是定时任务，它是一个上级控制器，和 Deployment 有些类似，当一个定时任务触发的时候，它会去创建一个 Job ，具体的任务实际上是由 Job 来负责执行的；

### 什么是 Heapster?

​ 答：

Heapster 是 K8s 原生的集群监控方案。

Heapster 以 Pod 的形式运行，它会自动发现集群节点、从节点上的 Kubelet 获取监控数据。Kubelet 则是从节点上的 cAdvisor 收集数据；

### k8s Architecture 的不同组件有哪些?

​ 答：

主要有两个组件 – 主节点和工作节点。

主节点具有 kube-controller-manager，kube-apiserver，kube-scheduler 等组件。

而工作节点具有 kubelet 和 kube-proxy 等组件；

### 能否介绍一下 Kubernetes 中主节点的工作情况?

​ 答：

主节点是集群控制节点，负责集群管理和控制，包含：

1）apiserver: rest 接口，资源增删改查入口；

2）controller-manager: 所有资源对象的控制中心；

3）scheduler: 负责资源调度，例如 pod 调度；

4）etcd: 保存资源对象数据；

### kube-apiserver 和 kube-scheduler 的作用是什么？

​ 答：

kube-apiserver: rest 接口，增删改查接口，集群内模块通信；

kube-scheduler: 将待调度的 pod 按照调度算法绑定到合适的 pod，并将绑定信息写入 etcd；

### 你能简要介绍一下 Kubernetes 控制管理器吗？

Kubernetes 控制管理器是集群内部的控制中心，负责 node,pod,namespace 等管理，

控制管理器负责管理各种控制器，每个控制器通过 api server 监控资源对象状态，将现有状态修正到期望状态；

### Kubernetes 有哪些不同类型的服务？

*   ClusterIP、
*   NodePort、
*   LoadBalancer、
*   ExternalName；

### 你对 Kubernetes 的负载均衡器有什么了解？

答：

1）内部负载均衡器: 自动平衡负载并使用所需配置分配容器；

2）外部负载均衡器: 将流量从外部负载引导至后端容器；

### 使用 Kubernetes 时可以采取哪些最佳安全措施?

​

1）确保容器本身安全；

2）锁定容器的 Linux 内核；

3）使用基于角色的访问控制（RBAC）；

4）保守秘密的辛勤工作；5）保持网络安全；

​

### 参考文献：

https://blog.csdn.net/qq_21222149/article/details/89201744

https://blog.csdn.net/warrior_0319/article/details/80073720  
http://www.sel.zju.edu.cn/?p=840  
http://alexander.holbreich.org/docker-components-explained/  
https://www.cnblogs.com/sparkdev/p/9129334.htmls

### 推荐阅读：

*   《[Docker 面试题（史上最全 + 持续更新）](https://blog.csdn.net/crazymakercircle/article/details/128670335)》
    
*   《 [场景题：假设 10W 人突访，你的系统如何做到不 雪崩？](https://blog.csdn.net/crazymakercircle/article/details/128533821)》
    
*   《[尼恩 Java 面试宝典](https://blog.csdn.net/crazymakercircle/article/details/124790425)》
    
*   《[Springcloud gateway 底层原理、核心实战 (史上最全)](https://blog.csdn.net/crazymakercircle/article/details/125057567)》
    
*   《[Flux、Mono、Reactor 实战（史上最全）](https://blog.csdn.net/crazymakercircle/article/details/124120506)》
    
*   《[sentinel （史上最全）](https://blog.csdn.net/crazymakercircle/article/details/125059491)》
    
*   《[Nacos (史上最全)](https://blog.csdn.net/crazymakercircle/article/details/125057545)》
    
*   《[分库分表 Sharding-JDBC 底层原理、核心实战（史上最全）](https://blog.csdn.net/crazymakercircle/article/details/123420859)》
    
*   《[TCP 协议详解 (史上最全)](https://blog.csdn.net/crazymakercircle/article/details/114527369)》
    
*   《[clickhouse 超底层原理 + 高可用实操 （史上最全）](https://blog.csdn.net/crazymakercircle/article/details/126992542)》
    
*   《[nacos 高可用（图解 + 秒懂 + 史上最全）](https://blog.csdn.net/crazymakercircle/article/details/120702536)》
    
*   《[队列之王： Disruptor 原理、架构、源码 一文穿透](https://blog.csdn.net/crazymakercircle/article/details/128264803)》
    
*   《[环形队列、 条带环形队列 Striped-RingBuffer （史上最全）](https://blog.csdn.net/crazymakercircle/article/details/128264508)》
    
*   《[一文搞定：SpringBoot、SLF4j、Log4j、Logback、Netty 之间混乱关系（史上最全）](https://blog.csdn.net/crazymakercircle/article/details/125135726)
    
*   《[单例模式（史上最全）](https://blog.csdn.net/crazymakercircle/article/details/128265067)
    
*   《[红黑树（ 图解 + 秒懂 + 史上最全）](https://blog.csdn.net/crazymakercircle/article/details/125017316)》
    
*   《[分布式事务 （秒懂）](https://blog.csdn.net/crazymakercircle/article/details/109459593)》
    
*   《[缓存之王：Caffeine 源码、架构、原理（史上最全，10W 字 超级长文）](https://blog.csdn.net/crazymakercircle/article/details/128123114)》
    
*   《[缓存之王：Caffeine 的使用（史上最全）](https://blog.csdn.net/crazymakercircle/article/details/113751575)》
    
*   《[Java Agent 探针、字节码增强 ByteBuddy（史上最全）](https://blog.csdn.net/crazymakercircle/article/details/126579528)》
    
*   《[Docker 原理（图解 + 秒懂 + 史上最全）](https://blog.csdn.net/crazymakercircle/article/details/120747767)》
    
*   《[Redis 分布式锁（图解 - 秒懂 - 史上最全）](https://blog.csdn.net/crazymakercircle/article/details/116425814)》
    
*   《[Zookeeper 分布式锁 - 图解 - 秒懂](https://blog.csdn.net/crazymakercircle/article/details/85956246)》
    
*   《[Zookeeper Curator 事件监听 - 10 分钟看懂](https://blog.csdn.net/crazymakercircle/article/details/85922561)》
    
*   《[Netty 粘包 拆包 | 史上最全解读](https://blog.csdn.net/crazymakercircle/article/details/83957259)》
    
*   《[Netty 100 万级高并发服务器配置](https://blog.csdn.net/crazymakercircle/article/details/83758107)》
    
*   《[Springcloud 高并发 配置 （一文全懂）](https://blog.csdn.net/crazymakercircle/article/details/102557988)》