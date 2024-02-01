> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6989226070889201695)

Kubernetes+Docker+Jenkins 持续集成架构图
=================================

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a1dacc8080d44289f8c9699c23aec12~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37bbb18f15a7450e9414f41fb77ae1b2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

1.  构建 K8S 集群
2.  Jenkins 调度 K8S API
3.  动态生成 Jenkins Slave pod
4.  Slave pod 拉取 Git 代码／编译／打包镜像
5.  推送到镜像仓库 Harbor
6.  Slave 工作完成，Pod 自动销毁
7.  部署到测试或生产 Kubernetes 平台

Kubernetes+Docker+Jenkins 持续集成方案好处
----------------------------------

*   服务高可用
    
    *   当 Jenkins Master 出现故障时，Kubernetes 会自动创建一个新的 Jenkins Master 容器，并且将 Volume 分配给新创建的容器，保证数据不丢失，从而达到集群服务高可用
*   动态伸缩，合理使用资源
    
    *   每次运行 Job 时，会自动创建一个 Jenkins Slave，Job 完成后，Slave 自动注销并删除容器，资源自动释放
    *   而且 Kubernetes 会根据每个资源的使用情况，动态分配 Slave 到空闲的节点上创建，降低出现因某节点资源利用率高，还排队等待在该节点的情况。
*   扩展性好 当 Kubernetes 集群的资源严重不足而导致 Job 排队等待时，可以很容易的添加一个 Kubernetes Node 到集群中，从而实现扩展。
    

Kubeadm 安装 Kubernetes
=====================

K8S 详细可以参考：[Kubernetes](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2FCantevenl%2Farticle%2Fdetails%2F116481831 "https://blog.csdn.net/Cantevenl/article/details/116481831")

Kubernetes 的架构
--------------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54595e4d3646477087f46ccb37231904~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

*   API Server：用于暴露 Kubernetes API，任何资源的请求的调用操作都是通过 kube-apiserver 提供的接口进行的。
*   Etcd：是 Kubernetes 提供默认的存储系统，保存所有集群数据，使用时需要为 etcd 数据提供备份计划。
*   Controller-Manager：作为集群内部的管理控制中心，负责集群内的 Node、Pod 副本、服务端点（Endpoint）、命名空间（Namespace）、服务账号（ServiceAccount）、资源定额（ResourceQuota）的管理，当某个 Node 意外宕机时，Controller Manager 会及时发现并执行自动化修复流程，确保集群始终处于预期的工作状态。
*   Scheduler：监视新创建没有分配到 Node 的 Pod，为 Pod 选择一个 Node。
*   Kubelet：负责维护容器的生命周期，同时负责 Volume 和网络的管理
*   Kube proxy：是 Kubernetes 的核心组件，部署在每个 Node 节点上，它是实现 Kubernetes Service 的通信与负载均衡机制的重要组件

安装环境准备
------

<table><thead><tr><th>主机</th><th>ip</th><th>安装的软件</th></tr></thead><tbody><tr><td>k8s-master</td><td>192.168.188.116</td><td>kube-apiserver、kube-controller-manager、kube-scheduler、docker、etcd、calico，NFS</td></tr><tr><td>k8s-node1</td><td>192.168.188.117</td><td>kubelet、kubeproxy、Docker</td></tr><tr><td>k8s-node2</td><td>192.168.188.118</td><td>kubelet、kubeproxy、Docker</td></tr><tr><td>harbor 服务器</td><td>192.168.188.119</td><td>Harbor</td></tr><tr><td>gitlab 服务器</td><td>192.168.188.120</td><td>Gitlab</td></tr></tbody></table>

**三台 k8s 服务器都需要完成**

```
关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

关闭selinux
# 临时关闭
setenforce 0
# 永久关闭
sed -i 's/enforcing/disabled/' /etc/selinux/config  
  
关闭swap
# 临时
swapoff -a 
# 永久关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 根据规划设置主机名【master节点上操作】
hostnamectl set-hostname master
# 根据规划设置主机名【node1节点操作】
hostnamectl set-hostname node1
# 根据规划设置主机名【node2节点操作】
hostnamectl set-hostname node2

添加ip到hosts
cat >> /etc/hosts << EOF
192.168.188.116 master
192.168.188.117 node1
192.168.188.118 node2
EOF

设置系统参数
设置允许路由转发，不对bridge的数据进行处理
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
EOF

执行文件
sysctl -p /etc/sysctl.d/k8s.conf

kube-proxy开启ipvs的前置条件

cat > /etc/sysconfig/modules/ipvs.modules <<EOF 
#!/bin/bash 
modprobe -- ip_vs 
modprobe -- ip_vs_rr 
modprobe -- ip_vs_wrr 
modprobe -- ip_vs_sh 
modprobe -- nf_conntrack_ipv4 
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash 

/etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4


```

安装 Docker、kubelet、kubeadm、kubectl
---------------------------------

所有节点安装 Docker、kubeadm、kubelet，==Kubernetes 默认 CRI（容器运行时）为 Docker==，因此先安装 Docker

### 安装 Docker

```
首先配置一下Docker的阿里yum源
安装需要的安装包
yum install -y yum-utils

设置镜像仓库
我们用阿里云

yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

更新yum软件包索引
yum makecache fast

安装docker	docker-ce 社区
yum -y install docker-ce

查看版本
docker version

设置开机启动
systemctl enable docker --now

配置docker的镜像源
mkdir -p /etc/docker

这个是我自己阿里云的加速 每个人都不一样 可以去阿里云官方查看

tee /etc/docker/daemon.json <<-EOF
 {
   "registry-mirrors": ["https://m0rsqupc.mirror.aliyuncs.com"]
 }
EOF

验证
[root@master ~]# cat /etc/docker/daemon.json 
 {
   "registry-mirrors": ["https://m0rsqupc.mirror.aliyuncs.com"]
 }

然后重启docker
systemctl restart docker


```

### 安装 kubelet、kubeadm、kubectl

*   kubeadm: 用来初始化集群的指令
*   kubelet: 在集群中的每个节点上用来启动 pod 和 container 等
*   kubectl: 用来与集群通信的命令行工具

```
添加kubernetes软件源

cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

安装kubelet、kubeadm、kubectl，同时指定版本

yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0

systemctl enable kubelet --now


```

部署 Kubernetes Master（master 节点）
-------------------------------

```
kubeadm init --kubernetes-version=1.18.0 --apiserver-advertise-address=192.168.188.116 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址，（执行上述命令会比较慢，因为后台其实已经在拉取镜像了）
我们 docker images 命令即可查看已经拉取的镜像


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48af77c868a54e1cb960c29e3b05f1cb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

表示 kubernetes 的镜像已经安装成功 红色圈出来的部分 是下面 == 加入从节点 == 需要使用的命令

```
kubeadm join 192.168.188.116:6443 --token ic49lg.zuwab84r0zfs6bbr \
    --discovery-token-ca-cert-hash sha256:270285cba2080b1e291a3a2b3b21730616b59c95c55ca6f950fecf7b68869b97


```

```
使用kubectl工具
mkdir -p $HOME/.kube

cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config

查看运行的节点
kubectl get nodes

目前有一个master节点已经运行了，但是还处于未准备状态


```

**安装 Calico** Calico 是一个网络组件，作用是实现 master 和子节点实现网络通讯功能

```
mkdir k8s 
cd k8s 

wget https://docs.projectcalico.org/v3.10/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml

sed -i 's/192.168.0.0/10.244.0.0/g' calico.yaml 

安装calico的组件
kubectl apply -f calico.yaml

查看pod
kubectl get pod --all-namespaces
必须是全部READY 状态必须是Running

查看更详细的信息
kubectl get pod --all-namespaces -o wide


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e943cec4d20409cafbbaaa23aeb48a8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

加入 Kubernetes Node（Slave 节点）
----------------------------

需要到 node1 和 node2 服务器，向集群添加新节点

执行在 kubeadm init 输出的 kubeadm join 命令

以下的命令是在 master 初始化完成后，每个人的 == 都不一样 ==！！！需要复制自己生成的

```
kubeadm join 192.168.188.116:6443 --token ic49lg.zuwab84r0zfs6bbr \
    --discovery-token-ca-cert-hash sha256:270285cba2080b1e291a3a2b3b21730616b59c95c55ca6f950fecf7b68869b97

查看kubelet是否开启
systemctl status kubelet

然后查看node信息
kubectl get nodes

[root@master k8s]# kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
master   Ready    master   17m     v1.18.0
node1    Ready    <none>   2m27s   v1.18.0
node2    Ready    <none>   2m25s   v1.18.0

如果Status全部为Ready，代表集群环境搭建成功


```

kubectl 常用命令
------------

```
kubectl get nodes 查看所有主从节点的状态 
kubectl get ns 获取所有namespace资源 
kubectl get pods -n {$nameSpace} 获取指定namespace的pod 
kubectl describe pod的名称 -n {$nameSpace} 查看某个pod的执行过程 
kubectl logs --tail=1000 pod的名称 | less 查看日志 
kubectl create -f xxx.yml 通过配置文件创建一个集群资源对象 
kubectl delete -f xxx.yml 通过配置文件删除一个集群资源对象 
kubectl delete pod名称 -n {$nameSpace} 通过pod删除集群资源 
kubectl get service -n {$nameSpace} 查看pod的service情况


```

基于 Kubernetes 构建 Jenkins 持续集成平台
===============================

*   在 K8S 的基础上创建一个 Jenkins 主节点
*   在 K8S 上再创建 Jenkins 的从节点，来帮助我们进行项目的构建

安装和配置 NFS
---------

NFS（Network File System），它最大的功能就是可以通过网络，让不同的机器、不同的操作系统可以共享彼此的文件。我们可以利用 NFS 共享 ==Jenkins 运行的配置文件 ==、==Maven 的仓库依赖文件 == 等

我们把 NFS 服务器安装在 K8S 主节点上

```
安装NFS服务（这个需要在所有K8S的节点上安装）
yum install -y nfs-utils

创建共享目录（这个只需要在master节点）
mkdir -p /opt/nfs/jenkins
编写NFS的共享配置
vim /etc/exports 

/opt/nfs/jenkins	*(rw,no_root_squash) 

*代表对所有IP都开放此目录，rw是读写，no_root_squash不压制root权限

启动服务
systemctl enable nfs --now

查看NFS共享目录
showmount -e 192.168.188.116

[root@node1 ~]# showmount -e 192.168.188.116
Export list for 192.168.188.116:
/opt/nfs/jenkins *


```

在 Kubernetes 安装 Jenkins-Master
------------------------------

### 创建 NFS client provisioner

nfs-client-provisioner 是一个 Kubernetes 的简易 NFS 的外部 provisioner，本身不提供 NFS，需要现有的 NFS 服务器提供存储

需要编写 yaml 了

*   StorageClass.yaml
*   持久化存储 Storageclass
*   `kubectl explain StorageClass`查看 kind 和 version
*   定义名称为`managed-nfs-storage`

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "true"


```

*   deployment.yaml
*   部署 NFS client provisioner

```
apiVersion: storage.k8s.io/v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: lizhenliang/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 192.168.188.116
            - name: NFS_PATH
              value: /opt/nfs/jenkins/
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.188.116
            path: /opt/nfs/jenkins/


```

最后是一个权限文件 rbac.yaml

```
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io


```

构建 pod

```
将三个文件都写在同一个目录里面
kubectl create -f .

[root@master nfs-client]# kubectl create -f .
storageclass.storage.k8s.io/managed-nfs-storage created
serviceaccount/nfs-client-provisioner created
deployment.apps/nfs-client-provisioner created
clusterrole.rbac.authorization.k8s.io/nfs-client-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/run-nfs-client-provisioner created
role.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner created
rolebinding.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner created
Error from server (AlreadyExists): error when creating "rbac.yaml": serviceaccounts "nfs-client-provisioner" already exists

查看pod
kubectl get pod

[root@master nfs-client]# kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
nfs-client-provisioner-795b4df87d-zchrq   1/1     Running   0          2m4s


```

### 安装 Jenkins-Master

还是需要我们自己写 Jenkins 的 Yaml 文件

*   rbac.yaml
*   和权限有关的信息，把 jenkins 主节点的一些权限加入到 k8s 的管理下

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: jenkins
  namespace: kube-ops
rules:
  - apiGroups: ["extensions", "apps"]
    resources: ["deployments"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create","delete","get","list","patch","update","watch"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create","delete","get","list","patch","update","watch"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get","list","watch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: jenkins
  namespace: kube-ops
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: kube-ops
    
---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: jenkinsClusterRole
  namespace: kube-ops
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get","list","watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
 
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: jenkinsClusterRuleBinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkinsClusterRole
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: kube-ops


```

*   ServiceaAcount.yaml
*   jenkins 主节点的一些 ServiceaAcount 信息

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: kube-ops


```

*   StatefulSet.yaml
*   是一个有状态应用，里面就定义了之前 NFS 的信息

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: jenkins
  labels:
    name: jenkins
  namespace: kube-ops
spec:
  serviceName: jenkins
  selector:
    matchLabels:
      app: jenkins
  replicas: 1
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      name: jenkins
      labels:
        app: jenkins
    spec:
      terminationGracePeriodSeconds: 10
      serviceAccountName: jenkins
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts-alpine
          imagePullPolicy: IfNotPresent
          ports:
          - containerPort: 8080
            name: web
            protocol: TCP
          - containerPort: 50000
            name: agent
            protocol: TCP
          resources:
            limits:
              cpu: 1
              memory: 1Gi
            requests:
              cpu: 0.5
              memory: 500Mi
          env:
            - name: LIMITS_MEMORY
              valueFrom:
                resourceFieldRef:
                  resource: limits.memory
                  divisor: 1Mi
            - name: JAVA_OPTS
              value: -Xmx$(LIMITS_MEMORY)m -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
          livenessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12
          readinessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12
      securityContext:
        fsGroup: 1000
  volumeClaimTemplates:
  - metadata:
      name: jenkins-home
    spec:
      storageClassName: "managed-nfs-storage"
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi


```

*   Service.yaml
*   对外暴露一些端口

```
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: kube-ops
  labels:
    app: jenkins
spec:
  selector:
    app: jenkins
  type: NodePort
  ports:
  - name: web
    port: 8080
    targetPort: web
  - name: agent
    port: 50000
    targetPort: agent


```

```
cd jenkins-master/
[root@master jenkins-master]# ls
rbac.yaml  ServiceaAcount.yaml  Service.yaml  StatefulSet.yaml

创建一个新的命名空间
kubectl create namespace kube-ops 

创建jenkins主节点的pod
kubectl create -f .


[root@master jenkins-master]# kubectl create namespace kube-ops
namespace/kube-ops created
[root@master jenkins-master]# kubectl create -f .
service/jenkins created
serviceaccount/jenkins created
statefulset.apps/jenkins created
role.rbac.authorization.k8s.io/jenkins created
rolebinding.rbac.authorization.k8s.io/jenkins created
clusterrole.rbac.authorization.k8s.io/jenkinsClusterRole created
rolebinding.rbac.authorization.k8s.io/jenkinsClusterRuleBinding created

我们可以通过很多个命令来查看pod和service的信息
查看的时候需要加上我们自己的命名空间
kubectl get pod --namespace kube-ops

[root@master jenkins-master]# kubectl get pod --namespace kube-ops
NAME        READY   STATUS    RESTARTS   AGE
jenkins-0   1/1     Running   0          7m38s

查看服务的信息
kubectl get service --namespace kube-ops

[root@master jenkins-master]# kubectl get service --namespace kube-ops
NAME      TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
jenkins   NodePort   10.101.126.122   <none>        8080:30942/TCP,50000:31796/TCP   8m2s

-o wide查看更详细的信息
kubectl get service --namespace kube-ops -o wide

[root@master jenkins-master]# kubectl get service --namespace kube-ops -o wide
NAME      TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE     SELECTOR
jenkins   NodePort   10.101.126.122   <none>        8080:30942/TCP,50000:31796/TCP   8m20s   app=jenkins

查看pod更详细的信息
kubectl get pod --namespace kube-ops -o wide

[root@master jenkins-master]# kubectl get pod --namespace kube-ops -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
jenkins-0   1/1     Running   0          8m48s   10.244.166.129   node1   <none>           <none>

通过详细信息发现，我们的jenkins是部署在node1节点上
我们可以通过浏览器访问
node1的ip+端口


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/498c8a2a38eb425e850a64044cddc49d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 这里需要解锁 jenkins，我们需要去 jenkins 的共享目录，NFS 下寻找

```
nfs目录在/opt/nfs
[root@master jenkins-master]# cd /opt//nfs/
[root@master nfs]# ls
jenkins
[root@master nfs]# cd jenkins/
[root@master jenkins]# ll
total 4
drwxrwxrwx 13 root root 4096 May 13 16:26 kube-ops-jenkins-home-jenkins-0-pvc-0d59af09-0339-46eb-a6a0-550c40365efb

这个就是jenkins的运行目录
[root@master jenkins]# cd kube-ops-jenkins-home-jenkins-0-pvc-0d59af09-0339-46eb-a6a0-550c40365efb/
[root@master kube-ops-jenkins-home-jenkins-0-pvc-0d59af09-0339-46eb-a6a0-550c40365efb]# ls
config.xml                     jenkins.telemetry.Correlator.xml  nodes                     secrets      war
copy_reference_file.log        jobs                              plugins                   updates
hudson.model.UpdateCenter.xml  logs                              secret.key                userContent
identity.key.enc               nodeMonitors.xml                  secret.key.not-so-secret  users
[root@master kube-ops-jenkins-home-jenkins-0-pvc-0d59af09-0339-46eb-a6a0-550c40365efb]# cat secrets/initialAdminPassword 
5496a0ee3afd449fb65c709d6d721c5d

这个就是管理员密码 将其复制到浏览器解锁jenkins


```

跳过插件安装，我们一会儿修改下载源之后再安装插件 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47a01be4e69c4d9bbbbd460c5d132259~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 选择无 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32a5ead829c24a2aacc475039391c7e4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 创建用户 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7d5b77651054e96b5f86ba8bc713ad3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00e6bb96f1b1402786b36da93215bca1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97626d7b2544404786899ce4265e5b80~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) Jenkins 在 k8s 的环境下运行成功

### Jenkins 插件管理

Jenkins 国外官方插件地址下载速度非常慢，所以可以修改为国内插件地址 Jenkins->Manage Jenkins->Manage Plugins，点击 Available ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/581a8dea1a6c4d82a337644e556466a7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 这样做是为了把 Jenkins 官方的插件列表下载到本地，接着修改地址文件，替换为国内插件地址

```
[root@master kube-ops-jenkins-home-jenkins-0-pvc-0d59af09-0339-46eb-a6a0-550c40365efb]# pwd
/opt/nfs/jenkins/kube-ops-jenkins-home-jenkins-0-pvc-0d59af09-0339-46eb-a6a0-550c40365efb
[root@master kube-ops-jenkins-home-jenkins-0-pvc-0d59af09-0339-46eb-a6a0-550c40365efb]# cd updates/
[root@master updates]# ls
default.json  hudson.tasks.Maven.MavenInstaller

default.json就是插件下载地址
我们修改插件下载地址

sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json

sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g'default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json


```

最后，Manage Plugins 点击 Advanced，把 Update Site 改为国内插件下载地址

> [mirrors.tuna.tsinghua.edu.cn/jenkins/upd…](https://link.juejin.cn?target=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fjenkins%2Fupdates%2Fupdate-center.json "https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f34789834264e8eac1e3c48f86eacf1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 在浏览器 ip 后面 / restart 然后重启 jenkins ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c8d57f7b27d45bcba642acc67120137~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb06eacd8a0a41e6897b9de11738f68b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 安装基本插件

*   Chinese
*   Git
*   Pipeline
*   Extended Choice Parameter

等待安装 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/068e664c4087494a863d58b583f28585~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

Jenkins 与 Kubernetes 整合
-----------------------

安装`Kubernetes`插件 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f19b7f58db834bde948d876dee1e4921~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

安装完成之后前往全局配置最下面 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9373ee9d2964a29afa81e2136f66ada~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8cb686ff74034275a3bb86eac61c3f1e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

*   kubernetes 地址采用了 kube 的服务器发现：[kubernetes.default.svc.cluster.local](https://link.juejin.cn?target=https%3A%2F%2Fkubernetes.default.svc.cluster.local "https://kubernetes.default.svc.cluster.local")
*   namespace 填 kube-ops，然后点击 Test Connection，如果出现 Connection test successful 的提示信息证明 Jenkins 已经可以和 Kubernetes 系统正常通信
*   Jenkins URL 地址：[jenkins.kube-ops.svc.cluster.local:8080](https://link.juejin.cn?target=http%3A%2F%2Fjenkins.kube-ops.svc.cluster.local%3A8080 "http://jenkins.kube-ops.svc.cluster.local:8080")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f95e4863fec442d8003398c8e3be687~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 点击连接测试 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b78e8e0bbd654cc5bf7c80deb2a622e6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec7ab54abcb74d97abca7374dddde45d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

构建 Jenkins-Slave 自定义镜像
----------------------

Jenkins-Master 在构建 Job 的时候，Kubernetes 会创建 Jenkins-Slave 的 Pod 来完成 Job 的构建。我们选择运行 Jenkins-Slave 的镜像为官方推荐镜像：`jenkins/jnlp-slave:latest`，但是这个镜像里面并没有 Maven 环境，为了方便使用，我们需要 == 自定义 == 一个新的镜像

所以我们要编写 Dockerfile 自定义一个镜像

```
vim Dockerfile

FROM jenkins/jnlp-slave:latest

MAINTAINER xiaotian

# 切换到 root 账户进行操作
USER root

# 安装 maven
COPY apache-maven-3.6.2-bin.tar.gz .

RUN tar -zxf apache-maven-3.6.2-bin.tar.gz && \
    mv apache-maven-3.6.2 /usr/local && \
    rm -f apache-maven-3.6.2-bin.tar.gz && \
    ln -s /usr/local/apache-maven-3.6.2/bin/mvn /usr/bin/mvn && \
    ln -s /usr/local/apache-maven-3.6.2 /usr/local/apache-maven && \
    mkdir -p /usr/local/apache-maven/repo

COPY settings.xml /usr/local/apache-maven/conf/settings.xml

USER jenkins


```

还需要一个带有阿里源镜像的 settings 文件

```
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.conf}/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>/usr/local/apache-maven/repo</localRepository>

  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->

  <!-- offline
   | Determines whether maven should attempt to connect to the network when executing a build.
   | This will have an effect on artifact downloads, artifact deployment, and others.
   |
   | Default: false
  <offline>false</offline>
  -->

  <!-- pluginGroups
   | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
   | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
   | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
   |-->
  <pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <!-- servers
   | This is a list of authentication profiles, keyed by the server-id used within the system.
   | Authentication profiles can be used whenever maven must make a connection to a remote server.
   |-->
  <servers>
    <!-- server
     | Specifies the authentication information to use when connecting to a particular server, identified by
     | a unique name within the system (referred to by the 'id' attribute below).
     |
     | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are
     |       used together.
     |
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->

    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>

  <!-- mirrors
   | This is a list of mirrors to be used in downloading artifacts from remote repositories.
   |
   | It works like this: a POM may declare a repository to use in resolving certain artifacts.
   | However, this repository may have problems with heavy traffic at times, so people have mirrored
   | it to several places.
   |
   | That repository definition will have a unique id, so we can create a mirror reference for that
   | repository, to be used as an alternate download site. The mirror site will be the preferred
   | server for that repository.
   |-->
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
    <mirror>     
      <id>central</id>     
      <mirrorOf>central</mirrorOf>     
      <name>aliyun maven</name>
      <url>https://maven.aliyun.com/repository/public</url>     
    </mirror>
  </mirrors>

  <!-- profiles
   | This is a list of profiles which can be activated in a variety of ways, and which can modify
   | the build process. Profiles provided in the settings.xml are intended to provide local machine-
   | specific paths and repository locations which allow the build to work in the local environment.
   |
   | For example, if you have an integration testing plugin - like cactus - that needs to know where
   | your Tomcat instance is installed, you can provide a variable here such that the variable is
   | dereferenced during the build process to configure the cactus plugin.
   |
   | As noted above, profiles can be activated in a variety of ways. One way - the activeProfiles
   | section of this document (settings.xml) - will be discussed later. Another way essentially
   | relies on the detection of a system property, either matching a particular value for the property,
   | or merely testing its existence. Profiles can also be activated by JDK version prefix, where a
   | value of '1.4' might activate a profile when the build is executed on a JDK version of '1.4.2_07'.
   | Finally, the list of active profiles can be specified directly from the command line.
   |
   | NOTE: For profiles defined in the settings.xml, you are restricted to specifying only artifact
   |       repositories, plugin repositories, and free-form properties to be used as configuration
   |       variables for plugins in the POM.
   |
   |-->
  <profiles>
    <!-- profile
     | Specifies a set of introductions to the build process, to be activated using one or more of the
     | mechanisms described above. For inheritance purposes, and to activate profiles via <activatedProfiles/>
     | or the command line, profiles have to have an ID that is unique.
     |
     | An encouraged best practice for profile identification is to use a consistent naming convention
     | for profiles, such as 'env-dev', 'env-test', 'env-production', 'user-jdcasey', 'user-brett', etc.
     | This will make it more intuitive to understand what the set of introduced profiles is attempting
     | to accomplish, particularly when you only have a list of profile id's for debug.
     |
     | This profile example uses the JDK version to trigger activation, and provides a JDK-specific repo.
    <profile>
      <id>jdk-1.4</id>

      <activation>
        <jdk>1.4</jdk>
      </activation>

      <repositories>
        <repository>
          <id>jdk14</id>
          <name>Repository for JDK 1.4 builds</name>
          <url>http://www.myhost.com/maven/jdk14</url>
          <layout>default</layout>
          <snapshotPolicy>always</snapshotPolicy>
        </repository>
      </repositories>
    </profile>
    -->

    <!--
     | Here is another profile, activated by the system property 'target-env' with a value of 'dev',
     | which provides a specific path to the Tomcat instance. To use this, your plugin configuration
     | might hypothetically look like:
     |
     | ...
     | <plugin>
     |   <groupId>org.myco.myplugins</groupId>
     |   <artifactId>myplugin</artifactId>
     |
     |   <configuration>
     |     <tomcatLocation>${tomcatPath}</tomcatLocation>
     |   </configuration>
     | </plugin>
     | ...
     |
     | NOTE: If you just wanted to inject this configuration whenever someone set 'target-env' to
     |       anything, you could just leave off the <value/> inside the activation-property.
     |
    <profile>
      <id>env-dev</id>

      <activation>
        <property>
          <name>target-env</name>
          <value>dev</value>
        </property>
      </activation>

      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>
    -->
  </profiles>

  <!-- activeProfiles
   | List of profiles that are active for all builds.
   |
  <activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
  -->
</settings>


```

最后需要一个 maven 的安装包 我这里使用 apache-maven-3.6.2-bin.tar

```
[root@master jenkins-slave]# ls
apache-maven-3.6.2-bin.tar.gz  Dockerfile  settings.xml

创建镜像
docker build -t jenkins-slave-maven:latest .


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d92047a65f784b38b8cde0dc05754ad8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

```
查看镜像
docker images 


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b59a408ff7264985ad457eccc443b478~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 然后把镜像上传到 Harbor 的公共库 library 中（因为 k8s 从节点都需要下载此镜像，因此放在公共库最方便）

```
打标签
docker tag jenkins-slave-maven:latest 192.168.188.119:85/library/jenkins-slave-maven:latest

vim /etc/docker/daemon.json

 {
   "registry-mirrors": ["https://m0rsqupc.mirror.aliyuncs.com"],
   "insecure-registries": ["192.168.188.119:85"]
 }

登录
docker login -u admin -p Harbor12345 192.168.188.119:85

上传镜像
docker push 192.168.188.119:85/library/jenkins-slave-maven:latest


```

镜像上传成功 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98f70397e87645ac8b555f1c6536dccd~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

### 测试 Jenkins-Slave 是否可以创建

创建一个 Jenkins 流水线项目 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13fae65c5c4345f3b41128e78dd111c8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

添加 gitlab 的凭据，一会儿流水线脚本需要使用 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc7105b9cf6f49cfa1bae21d3a6c4f24~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

编写 Pipeline，从 GItlab 拉取代码

```
def git_address = "http://47.108.13.86:82/maomao_group/tensquare_back.git" 
def git_auth = "bb5a1b2e-8dfa-4557-a79b-66d0f1f05f5c" 

//创建一个Pod的模板，label为jenkins-slave 
podTemplate(label: 'jenkins-slave', cloud: 'kubernetes', containers: [ 
    containerTemplate( 
        name: 'jnlp', 
        image: "192.168.188.119:85/library/jenkins-slave-maven:latest"
    ) 
  ]
)
{ 
  //引用jenkins-slave的pod模块来构建Jenkins-Slave的pod
  node("jenkins-slave"){ 
      // 第一步 
      stage('拉取代码'){ 
          checkout([$class: 'GitSCM', branches: [[name: 'master']], 
          userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]])
      } 
  }
}


```

*   podTemplate 就是 k8s 的 pod 模板
*   label 是模板名字
*   cloud 是云名字 在全局配置里的 Cloud 里自己填写的
*   containerTemplate 指定在 pod 里运行的容器

构建成功

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/339ad27b9c624f56a0cab40eb6be9190~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 中间遇到的错误 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86e7dbf602d64baa83dbf7633feb14eb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 所有 k8s 节点必须授权 harbor 仓库地址

```
vi /etc/docker/daemon.json

{
"registry-mirrors": ["https://zydiol88.mirror.aliyuncs.com"],
"insecure-registries": ["harbor-ip:85"]
}


```

Jenkins+Kubernetes+Docker 完成微服务持续集成
-----------------------------------

### 拉取代码，构建镜像

#### 创建 NFS 共享目录

让所有 Jenkins-Slave 构建指向 NFS 的 Maven 的共享仓库目录

```
mkdir -p /opt/nfs/maven
vi /etc/exports

/opt/nfs/jenkins        *(rw,no_root_squash)
/opt/nfs/maven          *(rw,no_root_squash)

systemctl restart nfs

showmount -e 192.168.188.116 
Export list for 192.168.188.116:
/opt/nfs/maven   *
/opt/nfs/jenkins *



```

#### 创建项目，编写构建 Pipeline

在流水线里添加多选项参数 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/141afc1af5704ee79d5283610a01d5f9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6463370dcd674f799ebc48f256bf4c3c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6ebedd58bbf4d8499575da8bca29cf7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4958aa84abf649faa7c3dacb66e5d0bf~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c95e633b5354283abaad8e15f67a8d0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 再添加字符参数 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe3fc2c35a204b8abb7aa50cd5d078c1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dc0b836cbc74f2f9ed4bca116b6c92e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

保存

接着去添加 harbor 的凭证 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e920dc9a1d5341d3ae87209ceb873cab~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 编写流水线脚本

```
def git_address = "http://192.168.188.120:82/maomao_group/tensquare_back.git"
def git_auth = "bb5a1b2e-8dfa-4557-a79b-66d0f1f05f5c" 
//构建版本的名称 
def tag = "latest" 
//Harbor私服地址 
def harbor_url = "192.168.188.119:85" 
//Harbor的项目名称 
def harbor_project_name = "tensquare" 
//Harbor的凭证 
def harbor_auth = "4fe90544-8b7b-4f81-b282-d20a2eb6e437" 

podTemplate(label: 'jenkins-slave', cloud: 'kubernetes', containers: [ 
	containerTemplate( 
		name: 'jnlp',
		image: "192.168.188.119:85/library/jenkins-slave-maven:latest"
	),
	containerTemplate(
		name: 'docker',
		image: "docker:stable",
		ttyEnabled: true,
		command: 'cat'
	),
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath:'/var/run/docker.sock'),
	nfsVolume(mountPath: '/usr/local/apache-maven/repo', serverAddress: '192.168.188.116' , serverPath: '/opt/nfs/maven'),
	],
)
{ 
  node("jenkins-slave"){
	   // 第一步
	   stage('拉取代码'){
		  checkout([$class: 'GitSCM', branches: [[name: '${branch}']],
		  userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]])
	   }
	   // 第二步
	   stage('公共工程代码编译'){
			//编译并安装公共工程
		  sh "mvn -f tensquare_common clean install"
	   }
	   // 第三步
	   stage('构建镜像，部署项目'){
			 //把选择的项目信息转为数组
			 def selectedProjects = "${project_name}".split(',')
			 
			 for(int i=0;i<selectedProjects.size();i++){
				//取出每个项目的名称和端口 
				def currentProject = selectedProjects[i];
				//项目名称
				def currentProjectName = currentProject.split('@')[0]
				//项目启动端口
				def currentProjectPort = currentProject.split('@')[1]
				
				//定义镜像名称
				def imageName = "${currentProjectName}:${tag}"
				
				//编译，构建本地镜像
				sh "mvn -f ${currentProjectName} clean package dockerfile:build"
				
				container('docker') {
				
					//给镜像打标签
					sh "docker tag ${imageName} ${harbor_url}/${harbor_project_name}/${imageName}"
					
					//登录Harbor，并上传镜像
					withCredentials([usernamePassword(credentialsId:"${harbor_auth}", passwordVariable: 'password', usernameVariable: 'username')])
				{
						  //登录
						  sh "docker login -u ${username} -p ${password} ${harbor_url}"
						  
						  //上传镜像
						  sh "docker push ${harbor_url}/${harbor_project_name}/${imageName}"
					}
					//删除本地镜像
					sh "docker rmi -f ${imageName}"
					sh "docker rmi -f ${harbor_url}/${harbor_project_name}/${imageName}"
				}
			}
		}
	}
}


```

注意：在构建过程会发现无法创建仓库目录，是因为 NFS 共享目录权限不足，需更改权限

只用在 k8s 主节点

```
chmod -R 777 /opt/nfs/maven


```

如果出现这种错误 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbc6f09dc7644bd18e98a72280564b66~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

是 Docker 命令执行权限问题，所有 k8s 服务器都需要执行

```
chmod 777 /var/run/docker.sock


```

需要手动上传父工程依赖到 NFS 的 Maven 共享仓库目录中

#### 记录

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/669504ad6afb48deab4f10045b32fb08~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 脚本写好之后，都能错这么多次，真的太难了

错了只能看日志一步一步进行排错

因为这一步实在是太容易错了，我记录一下构建过程，以后遇到问题可以查看

```
Started by user maomao
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Created Pod: kubernetes kube-ops/jenkins-slave-qv8g5-p9qlr
Agent jenkins-slave-qv8g5-p9qlr is provisioned from template jenkins-slave-qv8g5
---
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://jenkins.kube-ops.svc.cluster.local:8080/job/tensquare__back/15/"
    runUrl: "job/tensquare__back/15/"
  labels:
    jenkins: "slave"
    jenkins/label-digest: "5059d2cd0054f9fe75d61f97723d98ab1a42d71a"
    jenkins/label: "jenkins-slave"
  name: "jenkins-slave-qv8g5-p9qlr"
spec:
  containers:
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "jenkins-slave-qv8g5-p9qlr"
    - name: "JENKINS_NAME"
      value: "jenkins-slave-qv8g5-p9qlr"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.kube-ops.svc.cluster.local:8080/"
    image: "192.168.188.119:85/library/jenkins-slave-maven:latest"
    imagePullPolicy: "IfNotPresent"
    name: "jnlp"
    resources:
      limits: {}
      requests: {}
    tty: false
    volumeMounts:
    - mountPath: "/usr/local/apache-maven/repo"
      name: "volume-1"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - command:
    - "cat"
    image: "docker:stable"
    imagePullPolicy: "IfNotPresent"
    name: "docker"
    resources:
      limits: {}
      requests: {}
    tty: true
    volumeMounts:
    - mountPath: "/usr/local/apache-maven/repo"
      name: "volume-1"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - hostPath:
      path: "/var/run/docker.sock"
    name: "volume-0"
  - name: "volume-1"
    nfs:
      path: "/opt/nfs/maven"
      readOnly: false
      server: "192.168.188.116"
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on jenkins-slave-qv8g5-p9qlr in /home/jenkins/agent/workspace/tensquare__back
[Pipeline] {
[Pipeline] stage
[Pipeline] { (拉取代码)
[Pipeline] checkout
The recommended git tool is: NONE
using credential bb5a1b2e-8dfa-4557-a79b-66d0f1f05f5c
Cloning the remote Git repository
Cloning repository http://192.168.188.120:82/maomao_group/tensquare_back.git
 > git init /home/jenkins/agent/workspace/tensquare__back # timeout=10
Fetching upstream changes from http://192.168.188.120:82/maomao_group/tensquare_back.git
 > git --version # timeout=10
 > git --version # 'git version 2.20.1'
using GIT_ASKPASS to set credentials gitlab-http-auth
 > git fetch --tags --force --progress -- http://192.168.188.120:82/maomao_group/tensquare_back.git +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
Checking out Revision dba0faf11591dc9aa572e43bb0b5134b3ebf195e (origin/master)
 > git config remote.origin.url http://192.168.188.120:82/maomao_group/tensquare_back.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse origin/master^{commit} # timeout=10
 > git config core.sparsecheckout # timeout=10
 > git checkout -f dba0faf11591dc9aa572e43bb0b5134b3ebf195e # timeout=10
Commit message: "牛"
 > git rev-list --no-walk dba0faf11591dc9aa572e43bb0b5134b3ebf195e # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (公共工程代码编译)
[Pipeline] sh
+ mvn -f tensquare_common clean install
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------------< com.tensquare:tensquare_common >-------------------
[INFO] Building tensquare_common 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.0.0:clean (default-clean) @ tensquare_common ---
[INFO] Deleting /home/jenkins/agent/workspace/tensquare__back/tensquare_common/target
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:resources (default-resources) @ tensquare_common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/jenkins/agent/workspace/tensquare__back/tensquare_common/src/main/resources
[INFO] skip non existing resourceDirectory /home/jenkins/agent/workspace/tensquare__back/tensquare_common/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ tensquare_common ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 5 source files to /home/jenkins/agent/workspace/tensquare__back/tensquare_common/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:testResources (default-testResources) @ tensquare_common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/jenkins/agent/workspace/tensquare__back/tensquare_common/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ tensquare_common ---
[INFO] No sources to compile
[INFO] 
[INFO] --- maven-surefire-plugin:2.21.0:test (default-test) @ tensquare_common ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:3.0.2:jar (default-jar) @ tensquare_common ---
[INFO] Building jar: /home/jenkins/agent/workspace/tensquare__back/tensquare_common/target/tensquare_common-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- maven-install-plugin:2.5.2:install (default-install) @ tensquare_common ---
[INFO] Installing /home/jenkins/agent/workspace/tensquare__back/tensquare_common/target/tensquare_common-1.0-SNAPSHOT.jar to /usr/local/apache-maven/repo/com/tensquare/tensquare_common/1.0-SNAPSHOT/tensquare_common-1.0-SNAPSHOT.jar
[INFO] Installing /home/jenkins/agent/workspace/tensquare__back/tensquare_common/pom.xml to /usr/local/apache-maven/repo/com/tensquare/tensquare_common/1.0-SNAPSHOT/tensquare_common-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.162 s
[INFO] Finished at: 2021-05-13T15:27:00Z
[INFO] ------------------------------------------------------------------------
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (构建镜像，部署项目)
[Pipeline] sh
+ mvn -f tensquare_eureka_server clean package dockerfile:build
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------< com.tensquare:tensquare_eureka_server >----------------
[INFO] Building tensquare_eureka_server 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.0.0:clean (default-clean) @ tensquare_eureka_server ---
[INFO] Deleting /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/target
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:resources (default-resources) @ tensquare_eureka_server ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ tensquare_eureka_server ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:testResources (default-testResources) @ tensquare_eureka_server ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ tensquare_eureka_server ---
[INFO] No sources to compile
[INFO] 
[INFO] --- maven-surefire-plugin:2.21.0:test (default-test) @ tensquare_eureka_server ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:3.0.2:jar (default-jar) @ tensquare_eureka_server ---
[INFO] Building jar: /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/target/tensquare_eureka_server-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.0.1.RELEASE:repackage (default) @ tensquare_eureka_server ---
[INFO] 
[INFO] --- dockerfile-maven-plugin:1.3.6:build (default-cli) @ tensquare_eureka_server ---
[INFO] Building Docker context /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server
[INFO] 
[INFO] Image will be built as tensquare_eureka_server:latest
[INFO] 
[INFO] Step 1/5 : FROM openjdk:8-jdk-alpine
[INFO] 
[INFO] Pulling from library/openjdk
[INFO] Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
[INFO] Status: Image is up to date for openjdk:8-jdk-alpine
[INFO]  ---> a3562aa0b991
[INFO] Step 2/5 : ARG JAR_FILE
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> a2a3b3df4f15
[INFO] Step 3/5 : COPY ${JAR_FILE} app.jar
[INFO] 
[INFO]  ---> 656a595f07ab
[INFO] Step 4/5 : EXPOSE 10086
[INFO] 
[INFO]  ---> Running in 868cfdbdd284
[INFO] Removing intermediate container 868cfdbdd284
[INFO]  ---> 4fa7a8297ad1
[INFO] Step 5/5 : ENTRYPOINT ["java","-jar","/app.jar"]
[INFO] 
[INFO]  ---> Running in bee52b92749b
[INFO] Removing intermediate container bee52b92749b
[INFO]  ---> d0ec04357240
[INFO] Successfully built d0ec04357240
[INFO] Successfully tagged tensquare_eureka_server:latest
[INFO] 
[INFO] Detected build of image with id d0ec04357240
[INFO] Building jar: /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/target/tensquare_eureka_server-1.0-SNAPSHOT-docker-info.jar
[INFO] Successfully built tensquare_eureka_server:latest
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  27.874 s
[INFO] Finished at: 2021-05-13T15:27:29Z
[INFO] ------------------------------------------------------------------------
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ docker tag tensquare_eureka_server:latest 192.168.188.119:85/tensquare/tensquare_eureka_server:latest
[Pipeline] withCredentials
Masking supported pattern matches of $username or $password
[Pipeline] {
[Pipeline] sh
Warning: A secret was passed to "sh" using Groovy String interpolation, which is insecure.
		 Affected argument(s) used the following variable(s): [password, username]
		 See https://jenkins.io/redirect/groovy-string-interpolation for details.
+ docker login -u **** -p **** 192.168.188.119:85
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[Pipeline] sh
+ docker push 192.168.188.119:85/tensquare/tensquare_eureka_server:latest
The push refers to repository [192.168.188.119:85/tensquare/tensquare_eureka_server]
c8a7e30e2667: Preparing
ceaf9e1ebef5: Preparing
9b9b7f3d56a0: Preparing
f1b5933fe4b5: Preparing
ceaf9e1ebef5: Layer already exists
9b9b7f3d56a0: Layer already exists
f1b5933fe4b5: Layer already exists
c8a7e30e2667: Pushed
latest: digest: sha256:320a88f1b5efa46bc74643c156652ca724211fff07578c8733ac7208f2ffa8a9 size: 1159
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] sh
+ docker rmi -f tensquare_eureka_server:latest
Untagged: tensquare_eureka_server:latest
[Pipeline] sh
+ docker rmi -f 192.168.188.119:85/tensquare/tensquare_eureka_server:latest
Untagged: 192.168.188.119:85/tensquare/tensquare_eureka_server:latest
Untagged: 192.168.188.119:85/tensquare/tensquare_eureka_server@sha256:320a88f1b5efa46bc74643c156652ca724211fff07578c8733ac7208f2ffa8a9
Deleted: sha256:d0ec043572402382bd3610e7fe9c63642c838edb43f7a16d822c374d231f9e9a
Deleted: sha256:4fa7a8297ad170d46a9b5774b6f6adacd6276e9acae9a4cbbb918286a7617c67
Deleted: sha256:656a595f07ab8eb40c24f787cdff50cbf0082e450251cc007a6bb055e67824e2
Deleted: sha256:caa525b07e2d84ac4c0ef38c69f193cd72d094c285d2971b5017359b9c9cb1d5
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS


```

微服务部署到 k8s
----------

*   需要把构建好的微服务镜像 交给 k8s 的环境来进行运行，因此需要一个`Kubernetes Continuous Deploy`插件
*   这是一个持续集成的插件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a0136c94fc24ff090ea83a97b987821~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 修改后的流水线脚本

*   `deploy.yml`描述了 在 k8s 里怎么部署不同微服务
*   kubeconfigId 是证书，需要在 jenkins 环境里创建出来

添加 k8s 证书，添加凭据 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bd6b05b8c77426c890f235133b46530~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 这个证书在 k8s 主节点

```
cd /root/.kube
cat config

显示的内容就是证书，我们需要把内容原封不动的全部复制过来


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/967786edbf964006a0a269ef70d6687a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 然后再次查看 会产生一个 id，这个 id 添加到脚本里 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53e1b36ca6ec49f7a226d6c123d52c5a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b1e4434237344f8b7b538a335e0f8dd~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

```
def git_address = "http://192.168.188.120:82/maomao_group/tensquare_back.git"
def git_auth = "bb5a1b2e-8dfa-4557-a79b-66d0f1f05f5c" 
//构建版本的名称 
def tag = "latest" 
//Harbor私服地址 
def harbor_url = "192.168.188.119:85" 
//Harbor的项目名称 
def harbor_project_name = "tensquare" 
//Harbor的凭证 
def harbor_auth = "2ec3c8b6-f9b6-4ef6-b4bc-fdba74f99420" 
//k8s的凭证
def k8s_auth = ""9565b450-3899-4892-8aed-d15b6f26f8fd

podTemplate(label: 'jenkins-slave', cloud: 'kubernetes', containers: [ 
	containerTemplate( 
		name: 'jnlp',
		image: "192.168.188.119:85/library/jenkins-slave-maven:latest"
	),
	containerTemplate(
		name: 'docker',
		image: "docker:stable",
		ttyEnabled: true,
		command: 'cat'
	),
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath:'/var/run/docker.sock'),
	nfsVolume(mountPath: '/usr/local/apache-maven/repo', serverAddress: '192.168.188.116' , serverPath: '/opt/nfs/maven'),
	],
)
{ 
  node("jenkins-slave"){
	   // 第一步
	   stage('拉取代码'){
		  checkout([$class: 'GitSCM', branches: [[name: '${branch}']],
		  userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]])
	   }
	   // 第二步
	   stage('公共工程代码编译'){
			//编译并安装公共工程
		  sh "mvn -f tensquare_common clean install"
	   }
	   // 第三步
	   stage('构建镜像，部署项目'){
			 //把选择的项目信息转为数组
			 def selectedProjects = "${project_name}".split(',')
			 
			 for(int i=0;i<selectedProjects.size();i++){
				//取出每个项目的名称和端口 
				def currentProject = selectedProjects[i];
				//项目名称
				def currentProjectName = currentProject.split('@')[0]
				//项目启动端口
				def currentProjectPort = currentProject.split('@')[1]
				
				//定义镜像名称
				def imageName = "${currentProjectName}:${tag}"
				
				//编译，构建本地镜像
				sh "mvn -f ${currentProjectName} clean package dockerfile:build"
				
				container('docker') {
				
					//给镜像打标签
					sh "docker tag ${imageName} ${harbor_url}/${harbor_project_name}/${imageName}"
					
					//登录Harbor，并上传镜像
					withCredentials([usernamePassword(credentialsId:"${harbor_auth}", passwordVariable: 'password', usernameVariable: 'username')])
				{
						  //登录
						  sh "docker login -u ${username} -p ${password} ${harbor_url}"
						  
						  //上传镜像
						  sh "docker push ${harbor_url}/${harbor_project_name}/${imageName}"
					}
					//删除本地镜像
					sh "docker rmi -f ${imageName}"
					sh "docker rmi -f ${harbor_url}/${harbor_project_name}/${imageName}"
				}
				def deploy_image_name = "${harbor_url}/${harbor_project_name}/${imageName}"
				
				//部署到K8S
				sh """
						sed -i 's#\$IMAGE_NAME#${deploy_image_name}#' ${currentProjectName}/deploy.yml 
						sed -i 's#\$SECRET_NAME#${secret_name}#' ${currentProjectName}/deploy.yml 
				   """

				   kubernetesDeploy configs: "${currentProjectName}/deploy.yml", kubeconfigId: "${k8s_auth}"
				
			}
		}
	}
}


```

### 完成 eureka 集群部署到 Kubernetes

我们在 eureka 项目下创建一个`deploy.yml`文件

```
---
apiVersion: v1
kind: Service
metadata:
  name: eureka
  labels:
    app: eureka
spec:
  type: NodePort
  ports:
    - port: 10086
      name: eureka
      targetPort: 10086
  selector:
    app: eureka
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: eureka
spec:
  serviceName: "eureka"
  replicas: 2
  selector:
    matchLabels:
      app: eureka
  template:
    metadata:
      labels:
        app: eureka
    spec:
      imagePullSecrets:
        - name: $SECRET_NAME
      containers:
        - name: eureka
          image: $IMAGE_NAME
          ports:
            - containerPort: 10086
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: EUREKA_SERVER
              value: "http://eureka-0.eureka:10086/eureka/,http://eureka-1.eureka:10086/eureka/"
            - name: EUREKA_INSTANCE_HOSTNAME
              value: ${MY_POD_NAME}.eureka
  podManagementPolicy: "Parallel"


```

接着更改 eruka 的配置文件`application.yml`

```
server:
  port: ${PORT:10086}
spring:
  application:
    name: eureka

eureka:
  server:
    # 续期时间，即扫描失效服务的间隔时间（缺省为60*1000ms）
    eviction-interval-timer-in-ms: 5000
    enable-self-preservation: false
    use-read-only-response-cache: false
  client:
    # eureka client间隔多久去拉取服务注册信息 默认30s
    registry-fetch-interval-seconds: 5
    serviceUrl:
      defaultZone: ${EUREKA_SERVER:http://127.0.0.1:${server.port}/eureka/}
  instance:
    # 心跳间隔时间，即发送一次心跳之后，多久在发起下一次（缺省为30s）
    lease-renewal-interval-in-seconds: 5
    # 在收到一次心跳之后，等待下一次心跳的空档时间，大于心跳间隔即可，即服务续约到期时间（缺省为90s）
    lease-expiration-duration-in-seconds: 10
    instance-id: ${EUREKA_INSTANCE_HOSTNAME:${spring.application.name}}:${server.port}@${random.long(1000000,9999999)}
    hostname: ${EUREKA_INSTANCE_HOSTNAME:${spring.application.name}}


```

提交代码到仓库 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de97136a5b53494482c63bb149014298~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 尝试构建 eruka 服务器，但报错显示找不到`secret_name`这个属性 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7173b9bde3bf40178b75eb1707652a53~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 修改脚本定义一个 secret_name 的变量 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb07ba4398e548ddb8d3a79b07c78308~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 然后我们去制作凭证

```
首先在k8s主节点登录harbor仓库
docker login -u maomao -p Xiaotian123 192.168.188.119:85

然后使用一串命令生成证书
kubectl create secret docker-registry registry-auth-secret --docker-server=192.168.188.119:85 --docker-username=maomao --docker-password=Xiaotian123 --docker-email=123@qq.com



```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a3304c95b934962b60d5ac046fd4306~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 查看证书

```
kubectl get secret


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4469d4ef78aa4c69b7e39f3f4839299d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 然后再次构建 eruka 服务器 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/855d31e68537418a92f1e6d7e1c9352d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 构建成功之后，可以在主节点查看信息

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83fde2aee10546d286651b8adb869f2a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

```
kubectl get pods

查看端口
kubectl get svc


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a438f78cc86d4e6ea202ebc0fa088b91~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03eeba2a1bb04515aaef4e17bfbcccd1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1071371b349e45ac81f6dcfcfa7db7b7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

我们可以通过端口访问 所有 k8s 服务器 ip

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aea31842a7ad4e1184a998b9fd878de7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65c69abada0a428f95ff30bb9a9ed1c3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98c4fbce19d145b3b17582757a16cbed~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c79b91c47374d53823d2793518e4dd4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

### 错误排查

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c9fe77063434475b04b6316fee8b2b2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 经过我的深思熟虑，发现是密钥的问题 因为直接复制，导致有些地方本来是一行的结果变成了两行 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c8cfde10ff14a1694d60b1342caf241~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 将换行删除就行了 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bb0aa3940e64216a704c661cf40b17a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

结果又错了 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/660ed0049f4a4fd1b67429b3644a6417~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 我干脆直接把密钥文件拷贝到自己电脑上进行复制，这样格式就不会错了

记录一下构建过程

```
Started by user maomao
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Created Pod: kubernetes kube-ops/jenkins-slave-dlxsx-7pxm1
Agent jenkins-slave-dlxsx-7pxm1 is provisioned from template jenkins-slave-dlxsx
---
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://jenkins.kube-ops.svc.cluster.local:8080/job/tensquare__back/20/"
    runUrl: "job/tensquare__back/20/"
  labels:
    jenkins: "slave"
    jenkins/label-digest: "5059d2cd0054f9fe75d61f97723d98ab1a42d71a"
    jenkins/label: "jenkins-slave"
  name: "jenkins-slave-dlxsx-7pxm1"
spec:
  containers:
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "jenkins-slave-dlxsx-7pxm1"
    - name: "JENKINS_NAME"
      value: "jenkins-slave-dlxsx-7pxm1"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.kube-ops.svc.cluster.local:8080/"
    image: "192.168.188.119:85/library/jenkins-slave-maven:latest"
    imagePullPolicy: "IfNotPresent"
    name: "jnlp"
    resources:
      limits: {}
      requests: {}
    tty: false
    volumeMounts:
    - mountPath: "/usr/local/apache-maven/repo"
      name: "volume-1"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - command:
    - "cat"
    image: "docker:stable"
    imagePullPolicy: "IfNotPresent"
    name: "docker"
    resources:
      limits: {}
      requests: {}
    tty: true
    volumeMounts:
    - mountPath: "/usr/local/apache-maven/repo"
      name: "volume-1"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - hostPath:
      path: "/var/run/docker.sock"
    name: "volume-0"
  - name: "volume-1"
    nfs:
      path: "/opt/nfs/maven"
      readOnly: false
      server: "192.168.188.116"
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on jenkins-slave-dlxsx-7pxm1 in /home/jenkins/agent/workspace/tensquare__back
[Pipeline] {
[Pipeline] stage
[Pipeline] { (拉取代码)
[Pipeline] checkout
The recommended git tool is: NONE
using credential bb5a1b2e-8dfa-4557-a79b-66d0f1f05f5c
Cloning the remote Git repository
Cloning repository http://192.168.188.120:82/maomao_group/tensquare_back.git
 > git init /home/jenkins/agent/workspace/tensquare__back # timeout=10
Fetching upstream changes from http://192.168.188.120:82/maomao_group/tensquare_back.git
 > git --version # timeout=10
 > git --version # 'git version 2.20.1'
using GIT_ASKPASS to set credentials gitlab-http-auth
 > git fetch --tags --force --progress -- http://192.168.188.120:82/maomao_group/tensquare_back.git +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
Checking out Revision a35a2f630f44b425c52aa483aef1b7dc64539940 (origin/master)
 > git config remote.origin.url http://192.168.188.120:82/maomao_group/tensquare_back.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse origin/master^{commit} # timeout=10
 > git config core.sparsecheckout # timeout=10
 > git checkout -f a35a2f630f44b425c52aa483aef1b7dc64539940 # timeout=10
Commit message: "K8S"
 > git rev-list --no-walk a35a2f630f44b425c52aa483aef1b7dc64539940 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (公共工程代码编译)
[Pipeline] sh
+ mvn -f tensquare_common clean install
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------------< com.tensquare:tensquare_common >-------------------
[INFO] Building tensquare_common 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.0.0:clean (default-clean) @ tensquare_common ---
[INFO] Deleting /home/jenkins/agent/workspace/tensquare__back/tensquare_common/target
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:resources (default-resources) @ tensquare_common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/jenkins/agent/workspace/tensquare__back/tensquare_common/src/main/resources
[INFO] skip non existing resourceDirectory /home/jenkins/agent/workspace/tensquare__back/tensquare_common/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ tensquare_common ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 5 source files to /home/jenkins/agent/workspace/tensquare__back/tensquare_common/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:testResources (default-testResources) @ tensquare_common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/jenkins/agent/workspace/tensquare__back/tensquare_common/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ tensquare_common ---
[INFO] No sources to compile
[INFO] 
[INFO] --- maven-surefire-plugin:2.21.0:test (default-test) @ tensquare_common ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:3.0.2:jar (default-jar) @ tensquare_common ---
[INFO] Building jar: /home/jenkins/agent/workspace/tensquare__back/tensquare_common/target/tensquare_common-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- maven-install-plugin:2.5.2:install (default-install) @ tensquare_common ---
[INFO] Installing /home/jenkins/agent/workspace/tensquare__back/tensquare_common/target/tensquare_common-1.0-SNAPSHOT.jar to /usr/local/apache-maven/repo/com/tensquare/tensquare_common/1.0-SNAPSHOT/tensquare_common-1.0-SNAPSHOT.jar
[INFO] Installing /home/jenkins/agent/workspace/tensquare__back/tensquare_common/pom.xml to /usr/local/apache-maven/repo/com/tensquare/tensquare_common/1.0-SNAPSHOT/tensquare_common-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.069 s
[INFO] Finished at: 2021-05-13T16:34:40Z
[INFO] ------------------------------------------------------------------------
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (构建镜像，部署项目)
[Pipeline] sh
+ mvn -f tensquare_eureka_server clean package dockerfile:build
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------< com.tensquare:tensquare_eureka_server >----------------
[INFO] Building tensquare_eureka_server 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.0.0:clean (default-clean) @ tensquare_eureka_server ---
[INFO] Deleting /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/target
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:resources (default-resources) @ tensquare_eureka_server ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ tensquare_eureka_server ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:testResources (default-testResources) @ tensquare_eureka_server ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ tensquare_eureka_server ---
[INFO] No sources to compile
[INFO] 
[INFO] --- maven-surefire-plugin:2.21.0:test (default-test) @ tensquare_eureka_server ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:3.0.2:jar (default-jar) @ tensquare_eureka_server ---
[INFO] Building jar: /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/target/tensquare_eureka_server-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.0.1.RELEASE:repackage (default) @ tensquare_eureka_server ---
[INFO] 
[INFO] --- dockerfile-maven-plugin:1.3.6:build (default-cli) @ tensquare_eureka_server ---
[INFO] Building Docker context /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server
[INFO] 
[INFO] Image will be built as tensquare_eureka_server:latest
[INFO] 
[INFO] Step 1/5 : FROM openjdk:8-jdk-alpine
[INFO] 
[INFO] Pulling from library/openjdk
[INFO] Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
[INFO] Status: Image is up to date for openjdk:8-jdk-alpine
[INFO]  ---> a3562aa0b991
[INFO] Step 2/5 : ARG JAR_FILE
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> a2a3b3df4f15
[INFO] Step 3/5 : COPY ${JAR_FILE} app.jar
[INFO] 
[INFO]  ---> 1b476689026f
[INFO] Step 4/5 : EXPOSE 10086
[INFO] 
[INFO]  ---> Running in 68e029b5bdac
[INFO] Removing intermediate container 68e029b5bdac
[INFO]  ---> f76e827eb058
[INFO] Step 5/5 : ENTRYPOINT ["java","-jar","/app.jar"]
[INFO] 
[INFO]  ---> Running in 92ad4dcb6596
[INFO] Removing intermediate container 92ad4dcb6596
[INFO]  ---> ce5f4598f452
[INFO] Successfully built ce5f4598f452
[INFO] Successfully tagged tensquare_eureka_server:latest
[INFO] 
[INFO] Detected build of image with id ce5f4598f452
[INFO] Building jar: /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/target/tensquare_eureka_server-1.0-SNAPSHOT-docker-info.jar
[INFO] Successfully built tensquare_eureka_server:latest
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  28.550 s
[INFO] Finished at: 2021-05-13T16:35:10Z
[INFO] ------------------------------------------------------------------------
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ docker tag tensquare_eureka_server:latest 192.168.188.119:85/tensquare/tensquare_eureka_server:latest
[Pipeline] withCredentials
Masking supported pattern matches of $username or $password
[Pipeline] {
[Pipeline] sh
Warning: A secret was passed to "sh" using Groovy String interpolation, which is insecure.
		 Affected argument(s) used the following variable(s): [password, username]
		 See https://jenkins.io/redirect/groovy-string-interpolation for details.
+ docker login -u **** -p **** 192.168.188.119:85
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[Pipeline] sh
+ docker push 192.168.188.119:85/tensquare/tensquare_eureka_server:latest
The push refers to repository [192.168.188.119:85/tensquare/tensquare_eureka_server]
6b8a8e1926e4: Preparing
ceaf9e1ebef5: Preparing
9b9b7f3d56a0: Preparing
f1b5933fe4b5: Preparing
f1b5933fe4b5: Layer already exists
9b9b7f3d56a0: Layer already exists
ceaf9e1ebef5: Layer already exists
6b8a8e1926e4: Pushed
latest: digest: sha256:584ab9d6bd684636dfef55c2a2ac3d36d445b287f5a0a89a6694823655f909b1 size: 1159
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] sh
+ docker rmi -f tensquare_eureka_server:latest
Untagged: tensquare_eureka_server:latest
[Pipeline] sh
+ docker rmi -f 192.168.188.119:85/tensquare/tensquare_eureka_server:latest
Untagged: 192.168.188.119:85/tensquare/tensquare_eureka_server:latest
Untagged: 192.168.188.119:85/tensquare/tensquare_eureka_server@sha256:584ab9d6bd684636dfef55c2a2ac3d36d445b287f5a0a89a6694823655f909b1
Deleted: sha256:ce5f4598f452e300f537deacab64ee958f93a7c39ced0ff71f360f9c4d5d7572
Deleted: sha256:f76e827eb0580d99d60f8709608850589404666611fae1068e4e2231d100e6a3
Deleted: sha256:1b476689026f842092261a4d524043b6c4aa754e6b7b68c25501269fa1e17dce
Deleted: sha256:731b8259434bd270843c034ed36011581a2371daf28f682a6e3d0fa8b713545a
[Pipeline] }
[Pipeline] // container
[Pipeline] sh
+ sed -i s#$IMAGE_NAME#192.168.188.119:85/tensquare/tensquare_eureka_server:latest# tensquare_eureka_server/deploy.yml
+ sed -i s#$SECRET_NAME#registry-auth-secret# tensquare_eureka_server/deploy.yml
[Pipeline] kubernetesDeploy
Starting Kubernetes deployment
Loading configuration: /home/jenkins/agent/workspace/tensquare__back/tensquare_eureka_server/deploy.yml
Finished Kubernetes deployment
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS


```

网关集群部署到 k8s
-----------

更改 zull 的配置，将地址换成这个

```
defaultZone: http://eureka-0.eureka:10086/eureka/,http://eureka-1.eureka:10086/eureka/


```

编写一个 deploy.yml 文件

```
---
apiVersion: v1
kind: Service
metadata:
  name: zuul
  labels:
    app: zuul
spec:
  type: NodePort
  ports:
    - port: 10020
      name: zuul
      targetPort: 10020
  selector:
    app: zuul
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zuul
spec:
  serviceName: "zuul"
  replicas: 2
  selector:
    matchLabels:
      app: zuul
  template:
    metadata:
      labels:
        app: zuul
    spec:
      imagePullSecrets:
        - name: $SECRET_NAME
      containers:
        - name: zuul
          image: $IMAGE_NAME
          ports:
            - containerPort: 10020
  podManagementPolicy: "Parallel"


```

去进行任务构建 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bb85cc5c38c42908fd1594bd40fe6bb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 报错了 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/383f372f6503418286d99f252e2b4bc7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) 原来时需要 == 手动上传父工程依赖到 NFS 的 Maven 共享仓库目录中 ==

```
[root@master tensquare]# pwd
/opt/nfs/maven/com/tensquare

[root@master tensquare]# mv /root/tensquare_parent/ ./
[root@master tensquare]# ls
tensquare_common  tensquare_parent


```

上传完父工程依赖之后再次进行构建 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b103990734e48589f6ab2cbba24dc01~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

已经有两个网关了 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb9b396f2c764e748b5c1104d6c219da~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b9ef81f70474ac8bfa9dbd07081c227~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba0982832b864123a048d489dbbac59f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)