# 部署coredns出现容器不能启动，日志显示no such file or directory 

#### plugin/kubernetes: open /var/run/secrets/kubernetes.io/serviceaccount/token: no such file or directory 



该错误是apiserver没有设置启动参数 –admission-control=ServiceAccount 

```
KUBE_API_ARGS="--etcd-servers=http://127.0.0.1:2379 --insecure-bind-address=0.0.0.0   --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=ServiceAccount,NamespaceLifecycle,LimitRanger,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
```

修改kube-controller-manager 启动参数–service-account-private-key-file 

```
KUBE_CONTROLLER_MANAGER_ARGS="--service-account-private-key-file=/var/run/kubernetes/apiserver.key --master=http://172.16.199.55:8080 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"

```

