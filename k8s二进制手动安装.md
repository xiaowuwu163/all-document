# k8s二进制手动安装

v1.9.0以上安装网络教程

https://www.cnblogs.com/peterinblog/p/8124746.html



#### 1 下载etcd新版

https://github.com/coreos/etcd/releases

直接下载k8s的二进制包

https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.9.md#downloads-for-v190

下载其中的server和node

### 2 master安装

#### 2.1 将etcd放入/usr/bin，并确认可执行权限

将解压缩的server里面的组件，放入/usr/bin



`ubuntu@k8s-master:``/usr/bin``$ ``ls` `kube*`

`kube-apiserver  kube-controller-manager  kubectl  kubelet  kube-proxy  kube-scheduler`

`ubuntu@k8s-master:``/usr/bin``$　　`

#### 2.2 制作配置文件

etcd的配置文件

`ubuntu@k8s-master:~$ ``cat` `/etc/etcd/etcd``.conf`

`ETCD_NAME=default`

`ETCD_DATA_DIR=``"/var/lib/etcd/"`

`ETCD_LISTEN_CLIENT_URLS=``"http://0.0.0.0:2379"`

`ETCD_ADVERTISE_CLIENT_URLS=``"http://192.168.56.110:2379"`



##### kube的通用配置文件

 

config



`ubuntu@k8s-master:``/usr/bin``$ ``cat` `/etc/kubernetes/config`

`KUBE_LOGTOSTDERR=``"--logtostderr=true"`

`KUBE_LOG_LEVEL=``"--v=0"`

`KUBE_ALLOW_PRIV=``"--allow-privileged=false"`

`KUBE_MASTER=``"--master=http://192.168.56.110:8080"`



#### kube-apiserver 



`ubuntu@k8s-master:``/usr/bin``$ ``cat` `/etc/kubernetes/apiserver`

`#`

`# The following values are used to configure the kube-apiserver`

`#`

 

`# The address on the local server to listen to.`

`#KUBE_API_ADDRESS="--address=0.0.0.0"`

`KUBE_API_ADDRESS=``"--insecure-bind-address=0.0.0.0"`

 

`# The port on the local server to listen on.`

`KUBE_API_PORT=``"--port=8080"`

 

`# Port minions listen on`

`#KUBELET_PORT="--kubelet-port=10250"`

 

`# Comma separated list of nodes in the etcd cluster`

`KUBE_ETCD_SERVERS=``"--etcd-servers=http://192.168.56.110:2379"`

 

`# Address range to use for services`

`KUBE_SERVICE_ADDRESSES=``"--service-cluster-ip-range=192.168.4.0/24"`

 

`# default admission control policies`

`KUBE_ADMISSION_CONTROL=``"--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,ResourceQuota"`

 

`# Add your own!`

`KUBE_API_ARGS=``""`

`ubuntu@k8s-master:``/usr/bin``$`



##### controll-manager

`ubuntu@k8s-master:``/usr/bin``$ ``cat` `/etc/kubernetes/controller-manager`

`KUBE_CONTROLLER_MANAGER_ARGS=``""`

#####  scheduler 

`ubuntu@k8s-master:``/usr/bin``$ ``cat` `/etc/kubernetes/scheduler`

`KUBE_SCHEDULER_ARGS=``""`



### 2.3 启动文件

etcd的启动文件



`ubuntu@k8s-master:~$ ``cat` `/lib/systemd/system/etcd``.service`

`[Unit]`

`Description=Etcd Server`

`Documentation=https:``//github``.com``/coreos/etcd`

`After=network.target`

 

 

`[Service]`

`User=root`

`Type=notify`

`EnvironmentFile=-``/etc/etcd/etcd``.conf`

`ExecStart=``/usr/bin/etcd`

`Restart=on-failure`

`RestartSec=10s`

`LimitNOFILE=40000`

 

`[Install]`

`WantedBy=multi-user.target`

#### apisever 



`ubuntu@k8s-master:``/usr/bin``$ ``cat` `/lib/systemd/system/kube-apiserver``.service`

`[Unit]`

`Description=Kubernetes API Server`

`Documentation=https:``//github``.com``/GoogleCloudPlatform/kubernetes`

`After=network.target`

`After=etcd.service`

`Wants=etcd.service`

 

`[Service]`

`User=root`

`EnvironmentFile=-``/etc/kubernetes/config`

`EnvironmentFile=-``/etc/kubernetes/apiserver`

`ExecStart=``/usr/bin/kube-apiserver` `\`

`        ``$KUBE_LOGTOSTDERR \`

`        ``$KUBE_LOG_LEVEL \`

`        ``$KUBE_ETCD_SERVERS \`

`        ``$KUBE_API_ADDRESS \`

`        ``$KUBE_API_PORT \`

`        ``$KUBELET_PORT \`

`        ``$KUBE_ALLOW_PRIV \`

`        ``$KUBE_SERVICE_ADDRESSES \`

`        ``$KUBE_ADMISSION_CONTROL \`

`        ``$KUBE_API_ARGS`

`Restart=on-failure`

`Type=notify`

`LimitNOFILE=65536`

 

`[Install]`

`WantedBy=multi-user.target`



#### controller-manager 

`ubuntu@k8s-master:``/usr/bin``$ ``cat` `/lib/systemd/system/kube-controller-manager``.service`

`[Unit]`

`Description=Kubernetes Controller Manager`

`Documentation=https:``//github``.com``/GoogleCloudPlatform/kubernetes`

`After=etcd.service`

`After=kube-apiserver.service`

`Requires=etcd.service`

`Requires=kube-apiserver.service`

 

`[Service]`

`User=root`

`EnvironmentFile=-``/etc/kubernetes/config`

`EnvironmentFile=-``/etc/kubernetes/controller-manager`

`ExecStart=``/usr/bin/kube-controller-manager` `\`

`        ``$KUBE_LOGTOSTDERR \`

`        ``$KUBE_LOG_LEVEL \`

`        ``$KUBE_MASTER \`

`        ``$KUBE_CONTROLLER_MANAGER_ARGS`

`Restart=on-failure`

`LimitNOFILE=65536`

 

`[Install]`

`WantedBy=multi-user.target`



##### scheduler 

`ubuntu@k8s-master:``/usr/bin``$ ``cat` `/lib/systemd/system/kube-scheduler``.service`

`[Unit]`

`Description=Kubernetes Scheduler`

`Documentation=https:``//github``.com``/kubernetes/kubernetes`

 

`[Service]`

`User=root`

`EnvironmentFile=-``/etc/kubernetes/config`

`EnvironmentFile=-``/etc/kubernetes/scheduler`

`ExecStart=``/usr/bin/kube-scheduler` `\`

`        ``$KUBE_LOGTOSTDERR \`

`        ``$KUBE_MASTER`

`Restart=on-failure`

`LimitNOFILE=65536`

 

`[Install]`

`WantedBy=multi-user.target`



#### 2.4 配置完成后启动 

`sudo` `systemctl daemon-reload`

`sudo` `systemctl ``enable` `kube-apiserver kube-controller-manager kube-scheduler`

`sudo` `systemctl start kube-apiserver kube-controller-manager kube-scheduler`

### 3 node安装

3.1 将kube的二进制包解压，将组件放入/usr/bin



`ubuntu@k8s-node:~$ ``ls` `/usr/bin/kube``*`

`/usr/bin/kubectl`  `/usr/bin/kubelet`  `/usr/bin/kube-proxy`

#### 3.2 制作配置文件

全局配置文件，注意1.9版本已经不支持master命令

`ubuntu@k8s-node:~$ ``cat` `/etc/kubernetes/config`

`KUBE_LOGTOSTDERR=``"--logtostderr=true"`

`KUBE_LOG_LEVEL=``"--v=3"`

`KUBE_ALLOW_PRIV=``"--allow-privileged=false"`

`KUBE_MASTER="--master=http://192.168.56.110:8080"`



#### kubelet,需要注意

​     a. kubelet和apiserver连接方式改变，需要额外一个yaml的配置文件kubeconfig

​     b. 选项使能swap和并指定kubeconfig

kubeconfig



`ubuntu@k8s-node:~$ ``cat` `/var/lib/kubelet/kubeconfig`

`apiVersion: v1`

`kind: Config`

`users``:`

`- name: kubelet`

`clusters:`

`- name: kubernetes`

`  ``cluster:`

`    ``server: http:``//192``.168.56.110:8080`

`contexts:`

`- context:`

`    ``cluster: kubernetes`

`    ``user: kubelet`

`  ``name: service-account-context`

`current-context: service-account-context`



#### kubelet配置文件 

 `ubuntu@k8s-node:~$ ``cat` `/etc/kubernetes/kubelet`

`#KUBELET_ADDRESS="--address=127.0.0.1"`

`KUBELET_HOSTNAME=``"--hostname-override=192.168.56.111"`

`#KUBELET_API_SERVER="--api-servers=http://192.168.56.110:8080"`

`# pod infrastructure container`

`KUBELET_POD_INFRA_CONTAINER=``"--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"`

`KUBELET_ARGS=``"--enable-server=true --enable-debugging-handlers=true --fail-swap-on=false --kubeconfig=/var/lib/kubelet/kubeconfig"`

proxy配置文件 

`ubuntu@k8s-node:~$ ``cat` `/etc/kubernetes/proxy`

`KUBE_PROXY_ARGS=``""`



#### 3.3 启动文件

kubelet

`ubuntu@k8s-node:~$ ``cat` `/lib/systemd/system/kubelet``.service`

`[Unit]`

`Description=Kubernetes Kubelet`

`Documentation=https:``//github``.com``/GoogleCloudPlatform/kubernetes`

`After=docker.service`

`Requires=docker.service`

 

`[Service]`

`WorkingDirectory=``/var/lib/kubelet`

`EnvironmentFile=-``/etc/kubernetes/config`

`EnvironmentFile=-``/etc/kubernetes/kubelet`

`ExecStart=``/usr/bin/kubelet` `\`

`        ``$KUBE_LOGTOSTDERR \`

`        ``$KUBE_LOG_LEVEL \`

`        ``$KUBELET_API_SERVER \`

`        ``$KUBELET_ADDRESS \`

`        ``$KUBELET_PORT \`

`        ``$KUBELET_HOSTNAME \`

`        ``$KUBE_ALLOW_PRIV \`

`        ``$KUBELET_POD_INFRA_CONTAINER \`

`        ``$KUBELET_ARGS`

`Restart=on-failure`

`KillMode=process`

 

`[Install]`

`WantedBy=multi-user.target`

#### 　　proxy 

`ubuntu@k8s-node:~$ ``cat` `/lib/systemd/system/kube-proxy``.service`

`[Unit]`

`Description=Kubernetes Proxy`

`Documentation=https:``//github``.com``/GoogleCloudPlatform/kubernetes`

`After=network.target`

 

`[Service]`

`EnvironmentFile=-``/etc/kubernetes/config`

`EnvironmentFile=-``/etc/kubernetes/proxy`

`ExecStart=``/usr/bin/kube-proxy` `\`

`        ``$KUBE_LOGTOSTDERR \`

`        ``$KUBE_LOG_LEVEL \`

`        ``$KUBE_MASTER \`

`        ``$KUBE_PROXY_ARGS`

`Restart=on-failure`

`LimitNOFILE=65536`

 

`[Install]`

`WantedBy=multi-user.target`



##### 3.4 启动服务 

`sudo` `systemctl daemon-reload`

`sudo` `systemctl ``enable` `kubelet`

`sudo` `systemctl start kubelet`

#### 4 验证服务 

`ubuntu@k8s-master:~$ kubectl get nodes`

`NAME             STATUS    ROLES     AGE       VERSION`

`192.168.56.111   Ready     <none>    47m       v1.9.0`



配置CA证书和网络