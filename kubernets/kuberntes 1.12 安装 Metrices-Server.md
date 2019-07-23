# kuberntes 1.12 安装 Metrices-Server

2019年03月06日 18:36:42 [ameng734086045](https://me.csdn.net/ameng734086045) 阅读数：146

从几个帖子中部分截取了少量描述，加上了自己的修改，仅做为自己的部署记录。

**一、github资源下载**

<https://github.com/kubernetes/kubernetes/tree/release-1.12> 

目录 /cluster/addons/metrics-server

**二、环境准备**

1、**使用cfssl工具生成 证书**

vi  metrics-server-csr.json

```
{
  "CN": "aggregator",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

生成证书： 

```
 cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes metrics-server-csr.json | cfssljson -bare metrics-server
```

(其中的ca.pem, ca-key.pem ca-config.json是在部署1.12环境时生成的ca证书)

将生成的证书(metrics-server*.pem) 拷贝到集群各个节点的证书目录中

```
scp metrics-server*.pem 192.168.8.93:/data1/k8s_data/etc/kubernetes/ssl/
```

2、**开启聚合层** 

设置 kube-apiserver 启动参数

```
--requestheader-client-ca-file:ca 证书
--requestheader-allowed-names:  客户端证书常用名称列表。允许在--requestheader-username-headers指定的标头中提供用户名，如果为空，则允许在--requestheader-client-ca文件中通过当局验证的任何客户端证书
 --requestheader-extra-headers-prefix:  要检查的请求标头前缀列表
 --requestheader-group-headers:  要检查组的请求标头列表
--requestheader-username-headers:  要检查用户名的请求标头列表
--proxy-client-cert-file:  用于证明aggregator或kube-apiserver在请求期间发出呼叫的身份的客户端证书
--proxy-client-key-file:  用于证明聚合器或kube-apiserver的身份的客户端证书的私钥，当它必须在请求期间调用时使用。包括将请求代理给用户api-server和调用webhook admission插件
--enable-aggregator-routing=true:  打开aggregator路由请求到endpoints IP，而不是集群IP
```

我的配置如下：

```
  --requestheader-client-ca-file=/data1/k8s_data/etc/kubernetes/ssl/ca.pem \
  --requestheader-allowed-names= \
  --requestheader-extra-headers-prefix=X-Remote-Extra- \
  --requestheader-group-headers=X-Remote-Group \
  --requestheader-username-headers=X-Remote-User \
  --enable-aggregator-routing=true \
  --proxy-client-cert-file=/data1/k8s_data/etc/kubernetes/ssl/metrics-server.pem \
  --proxy-client-key-file=/data1/k8s_data/etc/kubernetes/ssl/metrics-server-key.pem \
```

 

重启 kube-apiserver 服务

```
systemctl daemon-reload && systemctl restart kube-apiserver
```

**3、修改部署文件，提前load 所需镜像。**

进入 metrics-server 目录

![img](https://img-blog.csdnimg.cn/20190306175836917.png)

 

修改 metrics-server-deployment.yaml 文件

![img](https://img-blog.csdnimg.cn/20190306180205463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FtZW5nNzM0MDg2MDQ1,size_16,color_FFFFFF,t_70)

--kubelet-insecure-tls Do not verify CA of serving certificates presented by Kubelets. For testing purposes only.

不验证kubelet的证书

\- --kubelet-preferred-address-types=InternalIp   使用独立ip而不是集群ip

![img](https://img-blog.csdnimg.cn/20190306181546166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FtZW5nNzM0MDg2MDQ1,size_16,color_FFFFFF,t_70)

填入实际数字

我的部署：

```
spec:
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server-amd64:v0.3.1
        imagePullPolicy: IfNotPresent
        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
        ports:
        - containerPort: 443
          name: https
          protocol: TCP
      - name: metrics-server-nanny
        image: k8s.gcr.io/addon-resizer:1.8.3
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 5m
            memory: 50Mi
        env:
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: MY_POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
        volumeMounts:
        - name: metrics-server-config-volume
          mountPath: /etc/config
        command:
          - /pod_nanny
          - --config-dir=/etc/config
          - --cpu=100m
          - --extra-cpu=0.5m
          - --memory=100MI
          - --extra-memory=10Mi
          - --threshold=5
          - --deployment=metrics-server-v0.3.1
          - --container=metrics-server
          - --poll-period=300000
          - --estimator=exponential
```

 

 

**三、部署**

进入 metrics-server 目录

```
kubectl apply -f *
```

输入命令测试：

![img](https://img-blog.csdnimg.cn/20190306183306849.png)

![img](https://img-blog.csdnimg.cn/20190306183400211.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FtZW5nNzM0MDg2MDQ1,size_16,color_FFFFFF,t_70)

**四、遇到的问题**

 

Error from server (Forbidden): nodes.metrics.k8s.io is forbidden: User "system:anonymous" cannot list nodes.metrics.k8s.io at the cluster scope: no RBAC policy matched

 

配置  

--proxy-client-cert-file=/data1/k8s_data/etc/kubernetes/ssl/metrics-server.pem \
 --proxy-client-key-file=/data1/k8s_data/etc/kubernetes/ssl/metrics-server-key.pem \

证书可解决