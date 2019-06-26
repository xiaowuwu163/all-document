# Kubernetes集群搭建踩坑记录（基于AWS）

Aug 26, 2018 • [Docker](http://blog.allen-mo.com/categories/docker/)

本文总结了在AWS线上搭建Kubernetes新集群过程中所踩的坑及对应的解决方法，作为前车之鉴。

## 0. 结论

对于大忙人，只需阅读下面的结论即可。

- 需要相互访问的节点（如master与node之间、node之间），如果在不同安全组或不同网段，应**确保端口能相互访问**，**确保入站规则没有限制通信**，以保证正常的网络通信。
- 如果采用Calico，应**确保所有节点的EC2 Source/Destination Checks已经关闭**，否则calico数据包会被丢掉。

## 1. 环境

### 1.1 机器

- master机器3台。kube-apiserver运行在所有master上，监听6444端口。kube-apiserver负载均衡器，监听6443端口。
- node机器多台。

### 1.2 软件

- Linux内核：3.13
- docker：1.12.6
- kubernetes：v1.9.6
- ETCD：v3.1.11
- harbor：1.2.2

## 2. Troubleshooting

### 2.0 问题：EC2 Source/Destination Checks导致Calico网络NAT失效

#### 2.0.1 问题表现

组件运行在两个节点上（下文称为节点1和节点2），service绑定externalIps到这两个节点的端口。测试过程发现如下问题：

- 通过externalIps+绑定端口只能访问externalIps所在节点上运行的Pod。例如，通过节点2绑定端口只能访问到运行在节点2上的容器，访问运行在节点1上的容器会超时。
- 在节点2上无法通过Pod IP访问运行在节点1上的容器，同样通过节点1上无法通过Pod IP访问运行在节点2上的容器。

经过检查：

- 节点上calico生成的路由表正常；在master上可以通过Pod IP访问两个节点上的容器，在节点2和节点1上也可访问master上的容器。说明calico正常工作。
- 在节点2上ping 节点1运行的容器IP时，节点2上eth0可抓到request数据包（但没有reply数据包），但节点1的eth0抓不到数据包。说明数据经过发送端的eth0后，没能成功到达目的地的eth0。反之，在节点1上ping节点2运行的Pod IP，结果相似。

#### 2.0.1 原因

仔细阅读[calico的安装文档](https://docs.projectcalico.org/v2.6/reference/public-cloud/aws)：

```
Routing Traffic Within a Single VPC Subnet
Since Calico assigns IP addresses outside the range used by AWS for EC2 instances, 
you must disable AWS src/dst checks on each EC2 instance in your cluster as described in the AWS documentation.
This allows Calico to route traffic natively within a single VPC subnet
without using an overlay or any of the limited VPC routing table entries.
```

节点必须关闭AWS Source/Destination Checks。

#### 2.0.2 解决方案

联系运维同事，关闭节点的[Source/Destination Checks](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck)，问题解决。

### 2.1 问题 ：EC2安全策略导致node上运行的容器访问kube-apiserver失败

#### 2.1.1 问题表现

首先，发现node节点上运行的calico-node容器无法访问api-server。log如下：

```
E0323 14:57:15.837393       1 reflector.go:201] github.com/projectcalico/kube-controllers/pkg/controllers/networkpolicy/policy_controller.go:192: Failed to list *extensions.NetworkPolicy: Get 10.32.0.1:443/apis/extensions/v1beta1/networkpolicies?timeout=10s: dial tcp 10.32.0.1:443: getsockopt: connection timed out
E0323 14:57:15.877799       1 reflector.go:201] github.com/projectcalico/kube-controllers/pkg/controllers/pod/pod_controller.go:203: Failed to list *v1.Pod: Get 10.32.0.1:443/api/v1/pods?resourceVersion=0: dial tcp 10.32.0.1:443: getsockopt: connection timed out
E0323 14:57:17.202557       1 reflector.go:201] github.com/projectcalico/kube-controllers/pkg/controllers/namespace/namespace_controller.go:151: Failed to list *v1.Namespace: Get 10.32.0.1:443/api/v1/namespaces?resourceVersion=0: dial tcp 10.32.0.1:443: getsockopt: connection timed out
E0323 14:57:17.313846       1 reflector.go:201] github.com/projectcalico/kube-controllers/pkg/controllers/node/node_controller.go:155: Failed to list *v1.Node: Get 10.32.0.1:443/api/v1/nodes?resourceVersion=0: dial tcp 10.32.0.1:443: getsockopt: connection timed out
```

然后，发现node节点上运行的kube-dns容器也无法访问api-server。

#### 2.1.2 原因

根据log提示，容器连接`10.32.0.1:443` 失败。`10.32.0.1`是kubernetes api-server 的service虚拟IP，`443`是service端口。该service是kubernetes自动创建，作为集群内运行的容器访问kubernetes api-server的入口。

再查看kubernetes service的配置，发现443端口实质上是转发给各个master上kube-apiserver的6444端口。

```
# kubectl get svc kubernetes -o yaml
apiVersion: v1
kind: Service
... ...
spec:
  clusterIP: 10.32.0.1
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: 6444
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  type: ClusterIP
```

kube-apiserver网络是hostNetwork模式，因此kube-apiserver容器的6444端口就是master机器的6444端口。

尝试在node机器上连接master机器的6444端口，失败。

再尝试在master机器上连接其他master的6444端口，成功。

#### 2.1.3 解决方案

联系运维同事，修改EC2安全策略，开放master的6444端口供node机器访问。问题解决。

### 2.2 EC2入站规则导致node组与master组容器网络不互通

#### 2.2.1 问题表现

kubernetes 网络是在aws网络之上架的一层容器虚拟网络。kubernetes网络搭建好之后，发现如下问题。

1）master组机器(网段为10.8.*.*)内部网络正常：在master组host机器上可访问运行在master组机器上的容器。

2）node组机器(网段为10.22.*.*)内部网络正常：在node组host机器上可访问运行在node组机器上的容器。

3）在master组host机器也可访问node组容器。

以上，说明容器网络应该没有问题。

4）**但是，node组host机器访问不了master组上运行的容器**。

#### 2.2.2 原因

经过测试，发现：master组机器可ping通master组其他机器的的docker0网桥；master组机器可ping通node组的docker0网桥；node组机器也可ping通node组其他机器的的docker0网桥。且被ping的机器的flannel0网桥上可以抓到icmp包，如下：

```
# tcpdump -i flannel0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on flannel0, link-type RAW (Raw IP), capture size 65535 bytes
07:10:33.918296 IP ip-10-200-95-0.ap-southeast-1.compute.internal > ip-10-200-11-1.ap-southeast-1.compute.internal: ICMP echo request, id 8008, seq 1, length 64
07:10:33.918346 IP ip-10-200-11-1.ap-southeast-1.compute.internal > ip-10-200-95-0.ap-southeast-1.compute.internal: ICMP echo reply, id 8008, seq 1, length 64
... ...
```

但**node组机器无法ping通master组机器的的docker0网桥，且在master上也抓不到数据包。**

检查master和node的 iptables和路由，未发现问题。所以怀疑master组ec2机器是不是还其他的策略配置，导致node机器发给master机器的数据包被悄咪咪的DROP掉？

#### 2.2.3 解决方案

经过运维同事检查，发现是master组EC2入站规则的原因。将master组改为允许node组机器的全部请求，问题解决。

### 2.3 问题：kubelet默认cgroup配置错误

#### 2.3.1 问题表现

各个节点的kubelet不断打出如下异常log。

```
failed to get container info  for "/user/9999.user/16.session": unknown container "/user/9999.user/16.session"
E0329 03:45:34.178403    4458 summary.go:92] Failed to get system container stats for "/user/9999.user/16.session": failed to get cgroup stats for "/user/9999.user/16.session": failed to get container info for "/user/9999.user/16.session": unknown container "/user/9999.user/16.session"
E0329 03:45:44.195805    4458 summary.go:92] Failed to get system container stats for "/user/9999.user/16.session": failed to get cgroup stats for "/user/9999.user/16.session": failed to get container info for "/user/9999.user/16.session": unknown container "/user/9999.user/16.session"
E0329 03:45:54.213220    4458 summary.go:92] Failed to get system container stats for "/user/9999.user/16.session": failed to get cgroup stats for "/user/9999.user/16.session": failed to get container info for "/user/9999.user/16.session": unknown container "/user/9999.user/16.session"
```

#### 2.3.2 原因

看log提示，kubelet默认采用了`/user/9999.user/16.session` 这个cgroup。

参考 [CGROUP 的默认层级](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/resource_management_guide/sec-default_cgroup_hierarchies) 与 [Recommended Cgroups Setup](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/node-allocatable.md#recommended-cgroups-setup) ，kubelet的cgroup应为`/systemd/system.slice`。

#### 2.3.3 解决方案

将所有节点的kubelet cgroup配置为`/systemd/system.slice`：

```
 --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice
```

重启kubelet，则该log不再出现。问题解决。

### 2.4 问题：节点kubelet与kube-aggregator相互访问失败

#### 2.4.1 问题表现

kube-api-server不断打出如下warning。

```
I0328 09:34:23.531708       1 wrap.go:42] GET /api/v1/namespaces/kube-system/endpoints/kube-controller-manager: (2.469136ms) 200 [[kube-controller-manager/v1.9.6 (linux/amd64) kubernetes/9f8ebd1/leader-election] 127.0.0.1:54714]
W0328 09:34:23.985148       1 x509.go:172] x509: subject with cn=system:node:[node IP] is not in the allowed list: [aggregator]
I0328 09:34:23.986038       1 wrap.go:42] GET /api/v1/nodes/[node IP]?resourceVersion=0: (1.090242ms) 200 [[kubelet/v1.9.6 (linux/amd64) kubernetes/9f8ebd1] [master IP]:11060]
W0328 09:34:23.989564       1 x509.go:172] x509: subject with cn=system:node:[node IP] is not in the allowed list: [aggregator]
I0328 09:34:23.997619       1 wrap.go:42] PATCH /api/v1/nodes/[node IP]/status: (8.170747ms) 200 [[kubelet/v1.9.6 (linux/amd64) kubernetes/9f8ebd1] [master IP]:11060]
I0328 09:34:24.078566       1 wrap.go:42] GET /apis/admissionregistration.k8s.io/v1alpha1/initializerconfigurations: (156.577µs) 404 [[kube-apiserver/v1.9.6 (linux/amd64) kubernetes/9f8ebd1] 127.0.0.1:35799]
```

其中，`cn=system:node:[节点IP]`是节点的角色。根据log，猜测是节点的kubelet访问master上的kube-aggregator时未能通过RBAC，导致被拒。

#### 2.4.2 原因

该问题与kube-apiservrer的`--requestheader-allowed-names`配置项有关。

```
--requestheader-allowed-names strings                     List of client certificate common names to allow to provide usernames in headers specified by --requestheader-username-headers. If empty, any client certificate validated by the authorities in --requestheader-client-ca-file is allowed.
```

原错误配置为`--requestheader-allowed-names=aggregator`。也就是说只有以`aggregator`开头的Role才能通过认证，导致各个节点（角色名为cn=system:node:[节点IP]）被决绝访问。

另外，kube-aggregator访问kubelet所用证书原为aggregator，有可能会因权限不够而被拒绝。

```
  - --proxy-client-cert-file=/var/lib/kubernetes/aggregator.pem
  - --proxy-client-key-file=/var/lib/kubernetes/aggregator-key.pem
```

#### 2.4.3 解决方案

首先，将`requestheader-allowed-names`配置改为：

```
--requestheader-allowed-names=
```

即不验证证书中角色名字，直接验证角色权限。

其次，将访问kubelet所用的证书改为admin证书，让kube-aggregator获取最高权限，保证能够正常访问kubelet。

```
  - --proxy-client-cert-file=/var/lib/kubernetes/admin.pem
  - --proxy-client-key-file=/var/lib/kubernetes/admin-key.pem
```

至此，上述log消失。问题解决。

### 2.5 问题：fabric8的base64解码BUG导致创建kubernetes客户端失败

#### 2.5.1 问题表现

新集群搭建完成后，通过命令行和curl均可访问k8s api-server，证明k8s集群运行正常。 **但在Gears填入新集群的公钥证书、私钥后，Gears后端连接k8s apiserver失败。**

#### 2.5.2 原因

经调试跟踪代码，发现fabric8客户端base64类库存在一个bug。

我们生成的证书数据格式如下：

```
-----BEGIN CERTIFICATE-----
//经base64编码的证书数据
-----END CERTIFICATE-----
```

头尾两行是明文字符串，也应作为证书的一部分，提供给kubernetes client。

再看fabric8的处理逻辑（io.fabric8.kubernetes.client.internal.CertUtils）：

```
  public static InputStream getInputStreamFromDataOrFile(String data, String file) throws IOException {
    if (data != null) {
      byte[] bytes = null;
      //尝试对证书数据进行base64解码
      ByteString decoded = ByteString.decodeBase64(data);
      //若解码出来不为null，则认为证书数据是经过base64编码
      //否则，将认为未经过base64编码
      if (decoded != null) {
          bytes = decoded.toByteArray();
      } else {
          bytes = data.getBytes();
      }

      return new ByteArrayInputStream(bytes);
    }
    if (file != null) {
      return new ByteArrayInputStream(new String(Files.readAllBytes(Paths.get(file))).trim().getBytes());
    }
    return null;
  }
```

其中，所调用解码okio#ByteString.decodeBase64(data)方法。

该方法对我们线上环境的证书进行base64解码，解码结果居然并不是null，因此fabric8误以为线上环境的证书是经过base64加密，从而使用了解码出来的错误信息作为证书，因导致客户端创建失败。

#### 2.5.3 解决方案

临时解决方案是，在使用证书创建fabric8客户端前，对证书进行一次base64加密。这样fabric8解密出来的就是正确的证书。

至此，问题解决。

### 2.6 问题：harbor的k8s部署方案存在bug，导致ui无法正常显示镜像

#### 2.6.1 问题表现

采用kubernetes方式部署harbor完毕，测试发现问题：推送镜像到镜像仓库后，在harbor的界面上无法看到镜像。但如果重启harbor的UI组件后，则重启后镜像可显示出来。

#### 2.6.2 原因

harbor的k8s部署方案存在bug，官方github已有相关[issue](https://github.com/vmware/harbor/issues/3825)， 导致harbor的UI组件与registry通信出现问题。

#### 2.6.3 解决方案

考虑harbor的k8s部署方案并未成熟，改为使用官方支持、更成熟的docker-compose部署。

问题解决。