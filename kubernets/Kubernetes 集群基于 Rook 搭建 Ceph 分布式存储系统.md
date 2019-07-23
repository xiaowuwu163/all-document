# Kubernetes 集群基于 Rook 搭建 Ceph 分布式存储系统

2019年01月10日 09:52:50 [哎_小羊_168](https://me.csdn.net/aixiaoyang168) 阅读数：1353

> Ceph是一种流行的开源SDS，可以提供流行的许多类型的存储系统，例如对象、块和文件系统，并在商用硬件上运行。Rook目前是CNCF的孵化级项目，也可以与其他存储provider一起使用，包括CockroachDB、EdgeFS、Minio和Cassandra。
>
> 至于Rook如何帮助你更好地使用Ceph，Trost说主要的好处是MONs的健康检查，带有自动故障转移，通过Kubernetes对象以及在一个中心位置提供存储选择实现对Ceph集群、池、文件系统和RGW的简单管理。
>
> 要了解和掌握Rook，你可以查看：
>
> 快速入门指南（https://rook.io/docs/rook/v0.9/quickstart-toc.html）
>
> GitHub存储库（https://github.com/rook/rook）
>
> 加入论坛（https://groups.google.com/forum/#!forum/rook-dev）
>
> Slack频道（https://slack.rook.io/）
>
> FOSDEM上的相关幻灯片
>
> （https://fosdem.org/2019/schedule/event/ceph_storage_with_rook/attachments/slides/3272/export/events/attachments/ceph_storage_with_rook/slides/3272/Ceph_storage_with_Rook_Slides.pdf）
>
> 演讲
>
> （https://fosdem.org/2019/schedule/event/ceph_storage_with_rook/）
> --------------------- 



### 文章目录

- - [1、Rook & Ceph 介绍](https://blog.csdn.net/aixiaoyang168/article/details/86215080#1Rook__Ceph__3)

  - - [1.1、Rook](https://blog.csdn.net/aixiaoyang168/article/details/86215080#11Rook_5)
    - [1.2、Ceph](https://blog.csdn.net/aixiaoyang168/article/details/86215080#12Ceph_9)

  - [2、环境、软件准备](https://blog.csdn.net/aixiaoyang168/article/details/86215080#2_29)

  - [3、Kubernetes HA 集群搭建](https://blog.csdn.net/aixiaoyang168/article/details/86215080#3Kubernetes_HA__45)

  - [4、部署 Rook Operator](https://blog.csdn.net/aixiaoyang168/article/details/86215080#4_Rook_Operator_90)

  - [5、配置 Rook Dashboard](https://blog.csdn.net/aixiaoyang168/article/details/86215080#5_Rook_Dashboard_214)

  - [6、部署 Rook toolbox 并测试](https://blog.csdn.net/aixiaoyang168/article/details/86215080#6_Rook_toolbox__266)

  - [7、部署 Ceph Monitoring Prometheus 监控](https://blog.csdn.net/aixiaoyang168/article/details/86215080#7_Ceph_Monitoring_Prometheus__352)

  - - [7.1、部署 Prometheus Operator](https://blog.csdn.net/aixiaoyang168/article/details/86215080#71_Prometheus_Operator_355)
    - [7.2、部署 Prometheus 实例](https://blog.csdn.net/aixiaoyang168/article/details/86215080#72_Prometheus__368)
    - [7.3、访问 Prometheus Dashboard](https://blog.csdn.net/aixiaoyang168/article/details/86215080#73_Prometheus_Dashboard_386)

## 1、Rook & Ceph 介绍

### 1.1、Rook

> Rook 是专用于 Cloud-Native 环境的文件、块、对象存储服务。它实现了一个自动管理的、自动扩容的、自动修复的分布式存储服务。Rook 支持自动部署、启动、配置、分配、扩容/缩容、升级、迁移、灾难恢复、监控以及资源管理。为了实现所有这些功能，Rook 需要依赖底层的容器编排平台，例如 kubernetes、CoreOS 等。Rook 目前支持 Ceph、NFS、Minio Object Store、Edegefs、Cassandra、CockroachDB 存储的搭建，后期会支持更多存储方案。

### 1.2、Ceph

> Ceph 是一个开源的分布式存储系统，包括对象存储、块设备、文件系统。它具有高可靠性、安装方便、管理简便、能够轻松管理海量数据。Ceph 存储集群具备了企业级存储的能力，它通过组织大量节点，节点之间靠相互通讯来复制数据、并动态地重分布数据，从而达到高可用分布式存储功能

使用 Rook 可以轻松实现在 Kubernetes 上部署并运行 Ceph 存储系统，并且提供 Dashboard 供用户查看存储系统信息，Rook 跟 Kubernetes 集成关系示意图如下：

![kubernetes+rook](https://gmem.site/wp-content/uploads/2018/02/rook-architecture-0.png)

说明一下：

- Rook 提供了卷插件，来扩展了 K8S 的存储系统，使用 Kubelet 代理程序 Pod 可以挂载 Rook 管理的块设备和文件系统。
- Rook Operator 负责启动并监控整个底层存储系统，例如 Ceph Pod、Ceph OSD 等，同时它还管理 CRD、对象存储、文件体统。
- Rook Agent 代理部署在 K8S 每个节点上以 Pod 容器运行，每个代理 Pod 都配置一个 Flexvolume 驱动，该驱动 主要用来跟 K8S 的卷控制框架集成起来，每个节点上的跟操作相关的操作，例如添加存储设备、挂载、格式化、删除存储等操作，都有该代理来完成。

使用 Rook 部署并管理 Ceph 存储系统，其架构图如下：

![rook + ceph](https://gmem.site/wp-content/uploads/2018/02/rook-architecture-2.png)

## 2、环境、软件准备

本次演示环境，我是在虚拟机上安装 Linux 系统来执行操作，通过虚拟机完成 Kubernetes HA 集群的搭建，以下是安装的软件及版本：

- **Oracle VirtualBox**: 5.1.20 r114628 (Qt5.6.2)
- **System**: CentOS Linux release 7.3.1611 (Core)
- **kubernetes**: 1.12.1
- **docker**: 18.06.1-ce
- **etcd**: 3.3.8
- **Keepalived**: v1.3.5
- **HaProxy**: version 1.5.18
- **cfssl** version 1.2.0
- **Rook** version 0.9

注意：Rook 支持的 Kubernetes 版本 `>= 1.8`，所以我们搭建的 Kubernetes 集群版本要在该版本以上，这里我们使用 `1.12.1` 版本。Rook 如果配置了 `dataDirHostPath` 参数来存储 Rook 数据到 Kubernetes 集群主机上，那么需要分配至少 5G 磁盘空间给该路径。

## 3、Kubernetes HA 集群搭建

Kubernetes HA 集群搭建，主要包含 Etcd HA 和 Master HA。Etcd HA 这个很容易办到，通过搭建 Etcd 集群即可（注意 Etcd 集群只能有奇数个节点）。Master HA 这个稍微麻烦一些，多主的意思就是多个 Kubernetes Master 节点组成，任意一个 Master 挂掉后，自动切换到另一个备用 Master，而且整个集群 Cluster-IP 不发生变化，从而实现高可用。而实现切换时 Cluster-IP 不变化，目前采用最多的方案就是 `haproxy + keepalived` 实现负载均衡，然后使用 VIP(虚地址) 方式来实现的。

这里推荐使用 [kubeasz](https://github.com/gjmzj/kubeasz) 项目，该项目致力于提供快速部署高可用 k8s 集群的工具，它基于二进制方式部署和利用 [`ansible-playbook`](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)实现自动化安装，同时集成了很多常用插件，而且都有对应的文档说明，非常方便操作。它提供了单节点、单主多节点、多主多节点、在公有云上部署等方案，通过它很容易就能完成各种类型版本 k8s 集群的搭建。

本次，我们要搭建的就是多主多节点 HA 集群，请参照 [kubeasz 文档](https://github.com/gjmzj/kubeasz/blob/master/docs/setup/00-planning_and_overall_intro.md) 来执行，非常方便，亲测可行，这里就不在演示了。说明一下：同时由于本机内存限制，共开启了 4 个虚拟机节点，每个节点都分别充当了一个或多个角色。

高可用集群节点配置：

- 部署节点 x 1: 10.222.78.60
- etcd 节点 x 3: 10.222.78.63、10.222.78.64、10.222.78.86
- master 节点 x 2: 10.222.78.63、10.222.78.64
- lb 节点 x 2: 10.222.78.63（主）、10.222.78.64（备）
- node 节点 x 1 : 10.222.77.86

部署完成后，通过如下命令查看下是否部署成功。

```
$ kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-2               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"} 

$ kubectl get nodes
NAME            STATUS                     ROLES    AGE   VERSION
10.222.78.63    Ready,SchedulingDisabled   master   1h    v1.12.1
10.222.78.64    Ready,SchedulingDisabled   master   1h    v1.12.1
10.222.78.86    Ready                      node     1h    v1.12.1
12345678910111213
```

默认 K8S Master 节点不参入调度的，不过为了下边每个节点都能部署相应的 Pod，所以这里将两个 Master 节点设置为参与调用。

```
$ kubectl uncordon 10.222.78.63
$ kubectl uncordon 10.222.78.64
$ kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
10.222.78.63   Ready    master   1h    v1.12.1
10.222.78.64   Ready    master   1h    v1.12.1
10.222.78.86   Ready    node     1h    v1.12.1
1234567
```

## 4、部署 Rook Operator

环境准备就绪，现在可以开始部署 Rook Operator，上来我先填个坑！

```
$ git clone https://github.com/rook/rook.git
$ cd rook/cluster/examples/kubernetes/ceph
$ kubectl create -f operator.yaml 
$ kubectl create -f cluster.yaml
1234
```

如果正常的话，Rook 会创建好所有需要的资源，但是很遗憾，你会发现当 `cluster.yaml` 创建完毕后，不会创建 `rook-ceph-mgr`、`rook-ceph-mon`、`rook-ceph-osd` 等资源。这是个坑，可以参考 [Rook Github Issues 2338](https://github.com/rook/rook/issues/2338) 这里，我们可以通过查看 `rook-ceph-operator` Pod 的日志来分析下：

```
$ kubectl logs -n rook-ceph-system rook-ceph-operator-68576ff976-m9m6l
......
E0107 12:06:23.272607       6 reflector.go:205] github.com/rook/rook/vendor/github.com/rook/operator-kit/watcher.go:76: Failed to list *v1beta1.Cluster: the server could not find the requested resource (get clusters.ceph.rook.io)
E0107 12:06:24.274364       6 reflector.go:205] github.com/rook/rook/vendor/github.com/rook/operator-kit/watcher.go:76: Failed to list *v1beta1.Cluster: the server could not find the requested resource (get clusters.ceph.rook.io)
E0107 12:06:25.288800       6 reflector.go:205] github.com/rook/rook/vendor/github.com/rook/operator-kit/watcher.go:76: Failed to list *v1beta1.Cluster: the server could not find the requested resource (get clusters.ceph.rook.io)
12345
```

类似以上日志输出，这是因为创建的 CRDs 资源版本不匹配导致的。正确的方法就是切换到最新固定版本，正确的操作如下：

```
$ git clone https://github.com/rook/rook.git
$ git checkout -b release-0.9 remotes/origin/release-0.9
$ git branch -a
  master
* release-0.9
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/release-0.4
  remotes/origin/release-0.5
  remotes/origin/release-0.6
  remotes/origin/release-0.7
  remotes/origin/release-0.8
  remotes/origin/release-0.9
$ cd rook/cluster/examples/kubernetes/ceph

# 部署 Rook Operator
$ kubectl create -f operator.yaml 
namespace/rook-ceph-system created
customresourcedefinition.apiextensions.k8s.io/cephclusters.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephfilesystems.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephobjectstores.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephobjectstoreusers.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/cephblockpools.ceph.rook.io created
customresourcedefinition.apiextensions.k8s.io/volumes.rook.io created
clusterrole.rbac.authorization.k8s.io/rook-ceph-cluster-mgmt created
role.rbac.authorization.k8s.io/rook-ceph-system created
clusterrole.rbac.authorization.k8s.io/rook-ceph-global created
clusterrole.rbac.authorization.k8s.io/rook-ceph-mgr-cluster created
serviceaccount/rook-ceph-system created
rolebinding.rbac.authorization.k8s.io/rook-ceph-system created
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-global created
deployment.apps/rook-ceph-operator created

$ kubectl get pods -n rook-ceph-system
NAME                                  READY   STATUS              RESTARTS   AGE
rook-ceph-operator-68576ff976-m9m6l   0/1     ContainerCreating   0          27s

$ kubectl get pods -n rook-ceph-system
NAME                                  READY   STATUS    RESTARTS   AGE
rook-ceph-agent-mjzz5                 1/1     Running   0          5m35s
rook-ceph-agent-tsk9l                 1/1     Running   0          5m35s
rook-ceph-agent-w6vfx                 1/1     Running   0          5m35s
rook-ceph-operator-68576ff976-m9m6l   1/1     Running   0          6m17s
rook-discover-44zh5                   1/1     Running   0          5m35s
rook-discover-dzpts                   1/1     Running   0          5m35s
rook-discover-txl96                   1/1     Running   0          5m35s
12345678910111213141516171819202122232425262728293031323334353637383940414243444546
```

说明一下，这里先创建了 `rook-ceph-operator`，然后在由它在每个节点创建 `rook-ceph-agent` 和 `rook-discover`。接下来，就可以部署 `CephCluster` 了。

```
$ kubectl create -f cluster.yaml 
namespace/rook-ceph created
serviceaccount/rook-ceph-osd created
serviceaccount/rook-ceph-mgr created
role.rbac.authorization.k8s.io/rook-ceph-osd created
role.rbac.authorization.k8s.io/rook-ceph-mgr-system created
role.rbac.authorization.k8s.io/rook-ceph-mgr created
rolebinding.rbac.authorization.k8s.io/rook-ceph-cluster-mgmt created
rolebinding.rbac.authorization.k8s.io/rook-ceph-osd created
rolebinding.rbac.authorization.k8s.io/rook-ceph-mgr created
rolebinding.rbac.authorization.k8s.io/rook-ceph-mgr-system created
rolebinding.rbac.authorization.k8s.io/rook-ceph-mgr-cluster created
cephcluster.ceph.rook.io/rook-ceph created

$ kubectl get cephcluster -n rook-ceph
NAME        DATADIRHOSTPATH   MONCOUNT   AGE   STATE
rook-ceph   /var/lib/rook     3          29m   Created

$ kubectl get pod -n rook-ceph
NAME                                       READY   STATUS      RESTARTS   AGE
rook-ceph-mgr-a-79564676c7-5crc5           1/1     Running     0          30m
rook-ceph-mon-a-56bfd58fdd-sql6w           1/1     Running     0          31m
rook-ceph-mon-b-68df678588-djj5v           1/1     Running     0          31m
rook-ceph-mon-c-65d8945f5-n4qsv            1/1     Running     0          30m
rook-ceph-osd-0-6ccbd4dc4d-2w9jt           1/1     Running     0          30m
rook-ceph-osd-1-647cbb4b84-65ttr           1/1     Running     0          30m
rook-ceph-osd-2-7b8ff9fc47-g8l6q           1/1     Running     0          30m
rook-ceph-osd-prepare-10.222.78.63-vvv9r   0/2     Completed   1          30m
rook-ceph-osd-prepare-10.222.78.64-nbfwz   0/2     Completed   0          30m
rook-ceph-osd-prepare-10.222.78.86-gp6fj   0/2     Completed   0          30m
123456789101112131415161718192021222324252627282930
```

说明一下，从 `cluster.yaml` 文件描述可以看出

```
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
......
  dataDirHostPath: /var/lib/rook
  mon:
    count: 3
    allowMultiplePerNode: true
  dashboard:
    enabled: true
  network:
    hostNetwork: false
  rbdMirroring:
    workers: 0
12345678910111213141516
```

`cephcluster` 是一个 CRD 自定义资源类型，通过它来创建一些列 ceph 的 mgr、osd 等。我们可以直接使用默认配置，默认开启 3 个 mon 资源，`dataDirHostPath` 存储路径在 `/var/lib/rook`，当然也可以自定义配置，例如 `DATADIRHOSTPATH`、`MONCOUNT` 等，可以参考官网文档 [ceph-cluster-crd](https://rook.github.io/docs/rook/v0.9/ceph-cluster-crd.html)。

## 5、配置 Rook Dashboard

创建完毕，我们可以使用默认启动的 Dashboard 来查看一下，默认 `cluster.yaml` 中
`dashboard: enabled: true` 配置为 `true`，那么就会自动创建 Dashboard 服务了。

```
$ kubectl get svc -n rook-ceph |grep mgr-dashboard
rook-ceph-mgr-dashboard                  ClusterIP   10.68.18.172    <none>        8443/TCP         33m
12
```

但是默认服务类型为 `ClusterIP` 类型，只能集群内部访问，通过
`https://rook-ceph-mgr-dashboard-https:8443` 或者 `https://10.68.18.172:8443` 地址访问，如果外部访问的话，就需要使用 `NodePort` 服务暴漏方式。

```
$ kubectl create -f dashboard-external-https.yaml 
service/rook-ceph-mgr-dashboard-external-https created
$ kubectl get svc -n rook-ceph |grep mgr-dashboard
rook-ceph-mgr-dashboard                  ClusterIP   10.68.18.172    <none>        8443/TCP         23m
rook-ceph-mgr-dashboard-external-https   NodePort    10.68.95.24     <none>        8443:33665/TCP   6m58s

# 查看 K8S 集群 IP
$ kubectl cluster-info |grep master
Kubernetes master is running at https://10.222.78.63:8443
123456789
```

外部通过 `https://<Cluster_ip>:<NodePort>` 地址访问，这里我可以通过 `https://10.222.78.63:33665` 地址访问页面，默认跳转到了登录页面 `https://10.222.78.63:33665/#/login` 登录页面。
![rook-dashboard](https://img-blog.csdnimg.cn/20190110093913170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpeGlhb3lhbmcxNjg=,size_16,color_FFFFFF,t_70)
但是需要用户名和密码，这里有两种方式获取：

方式一：`rook-ceph` 默认创建了一个 `rook-ceph-dashboard-password` 的 `secret`，可以用这种方式获取 password。

```
$ kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath='{.data.password}'  |  base64 --decode
21rWFIGtRG
12
```

注意：这里的 password 使用了 base64 加密，需要解密一下才可以使用。

方式二，从 `rook-ceph-mgr` Pod 的日志中获取，日志会打印出来用户名和密码。

```
$ kubectl get pod -n rook-ceph | grep mgr
rook-ceph-mgr-a-79564676c7-5crc5           1/1     Running     0          24m
$ kubectl -n rook-ceph logs rook-ceph-mgr-a-79564676c7-5crc5 | grep password
2019-01-07 13:32:02.315 7fea0989a700  0 log_channel(audit) log [DBG] : from='client.4168 172.20.0.4:0/4240445010' entity='client.admin' cmd=[{"username": "admin", "prefix": "dashboard set-login-credentials", "password": "21rWFIGtRG", "target": ["mgr", ""], "format": "json"}]: dispatch
1234
```

使用 admin 账户和上边的密码登录一下，可以登录成功。
![rook-dashboard](https://img-blog.csdnimg.cn/20190110093952829.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpeGlhb3lhbmcxNjg=,size_16,color_FFFFFF,t_70)
不过目前只能查看 cluster 中的 `Hosts`、`Monitors`、`OSDS` 信息，其他 `Pools (数据池)`、`Block (块设备)`、`Filesystem (文件系统)`、`Object Gateway (对象存储)`暂时没有涉及到，下边会逐步演示到。如果对 Ceph 的块设备、文件系统、对象存储不太清楚的，可以参考之前文章 [初试 Ceph 存储之块设备、文件系统、对象存储](https://blog.csdn.net/aixiaoyang168/article/details/78825850)。
![rook-dashboard-hosts](https://img-blog.csdnimg.cn/20190110094014264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpeGlhb3lhbmcxNjg=,size_16,color_FFFFFF,t_70)

![rook-dashboard-monitor](https://img-blog.csdnimg.cn/20190110094038663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpeGlhb3lhbmcxNjg=,size_16,color_FFFFFF,t_70)

![rook-dashboard-osds](https://img-blog.csdnimg.cn/2019011009410864.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpeGlhb3lhbmcxNjg=,size_16,color_FFFFFF,t_70)

## 6、部署 Rook toolbox 并测试

我们知道 Ceph 是有 CLI 工具来查看集群信息的，但是默认启动的 Ceph 集群时开启了 `cephx` 认证，登录 Ceph 各组件所在的 Pod 是没法执行 CLI 命令的，我们来实验一下：

```
$ kubectl -n rook-ceph get pod | grep rook-ceph-mon
rook-ceph-mon-a-56bfd58fdd-sql6w           1/1     Running     0          43m
rook-ceph-mon-b-68df678588-djj5v           1/1     Running     0          43m
rook-ceph-mon-c-65d8945f5-n4qsv            1/1     Running     0          42m

# 登录任意一个 Pod，执行 Ceph 命令
$ kubectl -n rook-ceph exec -it rook-ceph-mon-a-56bfd58fdd-sql6w bash
[root@rook-ceph-mon-a-56bfd58fdd-sql6w /]# ceph status
2019-01-07 15:06:03.720 7f7df9d4b700 -1 auth: unable to find a keyring on /etc/ceph/ceph.client.admin.keyring,/etc/ceph/ceph.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin,: (2) No such file or directory
2019-01-07 15:06:03.720 7f7df9d4b700 -1 auth: unable to find a keyring on /etc/ceph/ceph.client.admin.keyring,/etc/ceph/ceph.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin,: (2) No such file or directory
2019-01-07 15:06:03.722 7f7df9d4b700 -1 monclient: authenticate NOTE: no keyring found; disabled cephx authentication
2019-01-07 15:06:03.722 7f7df9d4b700 -1 monclient: authenticate NOTE: no keyring found; disabled cephx authentication
[errno 95] error connecting to the cluster
12345678910111213
```

从日志可以看到，`auth: unable to find a keyring on /etc/ceph/`，是没有 `cephx` 认证文件的。此时需要部署一个 `Ceph toolbox`，`toolbox` 以容器的方式在 K8S 内运行，它包含了 `cephx` 认证文件以及各种 `Ceph clients` 工具的。我们可以用它来执行一些 Ceph 相关的测试或调试操作。

```
$ kubectl create -f toolbox.yaml 
deployment.apps/rook-ceph-tools created
$ kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE
rook-ceph-tools-5bd5cdb949-d22ql   1/1     Running   0          2m15s   10.222.78.64   10.222.78.64   <none>

# Pod 被分配到 node2 上了。
$ kubectl -n rook-ceph exec -it rook-ceph-tools-5bd5cdb949-d22ql bash
[root@node2 /]# ceph status
  cluster:
    id:     6c117372-a462-447c-bfd4-a0378393f69e
    health: HEALTH_WARN
            clock skew detected on mon.a, mon.c
  services:
    mon: 3 daemons, quorum b,a,c
    mgr: a(active)
    osd: 3 osds: 3 up, 3 in
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   21 GiB used, 75 GiB / 96 GiB avail
    pgs:
    
[root@node2 /]# ceph df
GLOBAL:
    SIZE       AVAIL      RAW USED     %RAW USED 
    96 GiB     75 GiB       21 GiB         21.52 
POOLS:
    NAME     ID     USED     %USED     MAX AVAIL     OBJECTS 

[root@node2 /]# ceph osd status
+----+----------------------------------+-------+-------+--------+---------+--------+---------+-----------+
| id |               host               |  used | avail | wr ops | wr data | rd ops | rd data |   state   |
+----+----------------------------------+-------+-------+--------+---------+--------+---------+-----------+
| 0  | rook-ceph-osd-0-6ccbd4dc4d-2w9jt | 7893M | 24.2G |    0   |     0   |    0   |     0   | exists,up |
| 1  | rook-ceph-osd-1-647cbb4b84-65ttr | 5148M | 26.9G |    0   |     0   |    0   |     0   | exists,up |
| 2  | rook-ceph-osd-2-7b8ff9fc47-g8l6q | 8096M | 24.0G |    0   |     0   |    0   |     0   | exists,up |
+----+----------------------------------+-------+-------+--------+---------+--------+---------+-----------+    

[root@node2 /]# rados df
POOL_NAME USED OBJECTS CLONES COPIES MISSING_ON_PRIMARY UNFOUND DEGRADED RD_OPS RD WR_OPS WR 

total_objects    0
total_used       21 GiB
total_avail      75 GiB
total_space      96 GiB

# 创建一个新的 pool 
[root@node2 /]# ceph osd pool create test_pool 64
pool 'test_pool' created
[root@node2 /]# ceph osd pool get test_pool size
size: 1

# 再次执行 ceph df 查看是否显示创建的 pool
[root@node2 /]# ceph df
GLOBAL:
    SIZE       AVAIL      RAW USED     %RAW USED 
    96 GiB     75 GiB       21 GiB         21.52 
POOLS:
    NAME          ID     USED     %USED     MAX AVAIL     OBJECTS 
    test_pool     1       0 B         0        67 GiB           0 
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061
```

可以看到在 `toolbox` 内部可以执行 CLI 相关命令，此时我们在 Dashboard 上就可以看到创建的 pool 了。
![rook-dashboard-pools](https://img-blog.csdnimg.cn/20190110094159390.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpeGlhb3lhbmcxNjg=,size_16,color_FFFFFF,t_70)

## 7、部署 Ceph Monitoring Prometheus 监控

服务起来了，我们需要实时监控它，这里可以选择 Prometheus 来作为监控组件，部署 Prometheus 可以采用 `Prometheus Operator`来部署。

### 7.1、部署 Prometheus Operator

首先需要部署 `Prometheus Operator`，部署完毕后，直到 `prometheus-operator` Pod 状态为 `Running` 时，才能执行下一步操作。

```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.26.0/bundle.yaml
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
serviceaccount/prometheus-operator created
# kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
prometheus-operator-544f649-bhfxz   1/1     Running   0          13m
12345678
```

### 7.2、部署 Prometheus 实例

Rook 为我们提供好了部署 Prometheus 实例的 yaml 文件。

```
$ cd cluster/examples/kubernetes/ceph/monitoring
$ kubectl create -f ./
service/rook-prometheus created
serviceaccount/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
prometheus.monitoring.coreos.com/rook-prometheus created
servicemonitor.monitoring.coreos.com/rook-ceph-mgr created

$ kubectl -n rook-ceph get pod |grep prometheus-rook
prometheus-rook-prometheus-0               3/3     Running     1          10m
1234567891011
```

等到 `prometheus-rook-prometheus-0` 状态为 `Running` 后，我们就可以访问它了。

### 7.3、访问 Prometheus Dashboard

Prometheus Dashboard 服务启动后，默认暴漏服务类型为 `NodePort`，端口号为 `30900`，那么我们可以通过访问 `http://<Cluster_IP>:30900` 页面，这里我可以通过 `http://10.222.78.63:30900` 地址访问。

```
$ kubectl -n rook-ceph get svc |grep rook-prometheus
rook-prometheus                          NodePort    10.68.152.50    <none>        9090:30900/TCP   13m
12
```

![rook-prometheus-graph](https://img-blog.csdnimg.cn/20190110094342750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpeGlhb3lhbmcxNjg=,size_16,color_FFFFFF,t_70)

![rook-promethues-targets](https://img-blog.csdnimg.cn/20190110094416475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpeGlhb3lhbmcxNjg=,size_16,color_FFFFFF,t_70)

下一篇文章，将基于已经搭建好的 Rook 系统，验证一下其提供 Block 块存储、FileSystem 文件存储、Object Gateway 对象存储功能。

**参考资料**

- [Rook Ceph Quickstat Doc](https://rook.github.io/docs/rook/v0.9/ceph-quickstart.html)
- [Rook Github Project](https://github.com/rook/rook)