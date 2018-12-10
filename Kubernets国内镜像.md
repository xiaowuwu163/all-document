# Kubernets国内镜像

### 阿里云提供了Kubernetes国内镜像来安装`kubelet`、`kubectl` 和 `kubeadm`。

登陆阿里云镜像网站：<https://opsx.alibaba.com/mirror>

查找关键字“kubernetes”，点击【帮助】按钮。



## Debian / Ubuntu

安装https下载格式，增加apt-get索引仓库，阿里云

```
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

构建k8s集群的镜像，利用github服务器和阿里云服务器，构建到dockerhub服务器上，最后用docker下载下来

这里我就为大家推荐一个免费好用的方法：最开始我也是在[docker ub](https://hub.docker.com/)上看到的，通过Create Automated Build我们能很轻松的通过dockerfile来构建自己的镜像，这样我们就可以通过编写dockerfile来同步gcr.io上的镜像到

docker hub，但是docker hub的服务器也是位于国外，因此它的网速依旧是很感人的啊。

网站地址：https://blog.csdn.net/yjf147369/article/details/80290881

  ![img](https://img-blog.csdn.net/20180512125106962?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lqZjE0NzM2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 后来转念一想，国内的云服务商是不是也有类似的服务呢，很自然的就想到了阿里云，果然马爸爸没让我失望。  首先我们登陆阿里云的[容器镜像服务](https://cr.console.aliyun.com/?spm=5176.2020520001.aliyun_topbar.7.69864bd3tTO0oq#/imageList)然后点击*创建镜像仓库*  ![这里写图片描述](https://img-blog.csdn.net/20180512131506832?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lqZjE0NzM2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 在弹出的下面的页面中，我们设置好：地域，命名空间，仓库名称，摘要，仓库类型，代码源（存储dockerfile文件），构建设置。然后选择好github 仓库，项目分支，dockerfile文件的路径，填写版本号，点击*创建镜像仓库*。  **注意：**设置代码源时，我选择的代码源时github，如果第一次做这种构建操作，阿里云会要求绑定到你的github账号，另外下面的构建设置一定要把*海外机器构建*选上。  这里是我的github上的[dockderfile仓库](https://github.com/1912294464/docker-library)，大家可以follow我的项目（当然我也是follow了[别人](https://github.com/mritd)的项目的，这里对作者表示真诚的感谢）  ![这里写图片描述](https://img-blog.csdn.net/2018051213292061?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lqZjE0NzM2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  接下来我们就会发现在镜像列表中多了一行，点击改行右侧的*管理*  ![这里写图片描述](https://img-blog.csdn.net/20180512134044488?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lqZjE0NzM2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 选择*构建*，点击*立即构建*  ![这里写图片描述](https://img-blog.csdn.net/20180512134059120?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lqZjE0NzM2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 在等待一段时间后，下方会提示镜像构建成功  ![这里写图片描述](https://img-blog.csdn.net/20180512134109193?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lqZjE0NzM2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 在点击左侧*基本信息*，我们可以看到我们新构建的仓库的使用方法。  ![这里写图片描述](https://img-blog.csdn.net/20180512134022523?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lqZjE0NzM2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 然后我可以重复相同的操作来构建我们需要的别的镜像。  如果镜像过多这个操作可能相对枯燥复杂了一些，但是通过这一次构建以后就可以随时使用了。而且镜像下载速度绝对是让人满意的。 



# 拉取新构建的镜像
`docker pull registry.cn-shenzhen.aliyuncs.com/cookcodeblog/kube-apiserver-amd64:v1.10.3`
# 打上gcr.io同名标签
`docker tag registry.cn-shenzhen.aliyuncs.com/cookcodeblog/kube-apiserver-amd64:v1.10.3 k8s.gcr.io/kube-apiserver-amd64:v1.10.3`
# 查看镜像
`docker images`
# 删除新构建的镜像，只保留gcr.io镜像
`docker rmi registry.cn-shenzhen.aliyuncs.com/cookcodeblog/kube-apiserver-amd64:v1.10.3`
# 再次查看镜像
`docker images`

## 二、构建集群步骤

#### \#kubeadm init 时从本地私有仓库下载镜像

 

`images=(

gcr.io/google_containers/kube-proxy-amd64:v1.6.1 
gcr.io/google_containers/kube-apiserver-amd64:v1.6.1 
gcr.io/google_containers/kube-scheduler-amd64:v1.6.1 
gcr.io/google_containers/kube-controller-manager-amd64:v1.6.1 
gcr.io/google_containers/kube-proxy-amd64:v1.6.0 
gcr.io/google_containers/kube-controller-manager-amd64:v1.6.0 
gcr.io/google_containers/kube-apiserver-amd64:v1.6.0 
gcr.io/google_containers/kube-scheduler-amd64:v1.6.0 
gcr.io/google_containers/kubernetes-dashboard-amd64:v1.6.0 
gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.1 
gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.1 
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.1 
gcr.io/google_containers/etcd-amd64:3.0.17 
quay.io/coreos/flannel:v0.7.0-amd64 
gcr.io/google_containers/pause-amd64:3.0

)`



 

`for imageName in ${images[@]} ;   do

​     docker pull gcr.io/google_containers/$imageName；

​     docker tag gcr.io/google_containers/$imageName docker.cinyi.com:443/senyint/$imageName；

​     docker push docker.cinyi.com:443/senyint/$imageName；

done`





vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

为kubelet添加一个额外的参数 // 这样kubelet就不会在启动pod的时候去墙外的k8s仓库拉取pause-amd64:3.0镜像了

 

--pod-infra-container-image=docker.cinyi.com:443/senyint/pause-amd64:3.0        

 

Environment="KUBELET_INFRA_IMAGE=--pod-infra-container-image=docker.cinyi.com:443/senyint/pause-amd64:3.0"



#### 2.2一定要关闭swap

##### 一、不重启电脑，禁用启用swap，立刻生效

\# 禁用命令

sudo swapoff -a 

\# 启用命令

sudo swapon -a

\# 查看交换分区的状态

sudo free -m

##### 二、重新启动电脑，永久禁用Swap

\# 把根目录文件系统设为可读写

sudo mount -n -o remount,rw /

 

\# 用vi修改/etc/fstab文件，在swap分区这行前加 # 禁用掉，保存退出

vi /etc/fstab

i      #进入insert 插入模式

:wq   #保存退出



### kubeadm init 操作

会下载需要的镜像和相应的版本，提前下载下来

kubernetes v1.11.3版本需要coredns的镜像

##### 1、直接`docker pull coredns/coredns:latest` 拉取

##### 2、tag 改为相应的版本



# 配置科学上网

## 安装

Linux不同的发行版执行的命令如下(最好在root下运行以下命令，原因我下面会说明)：

```
Debian / Ubuntu:
apt-get install python-pip
pip install shadowsocks

CentOS:
yum install python-setuptools && easy_install pip
pip install shadowsocks
1234567
```

## 配置

sudo vim /etc/shadowsocks.json 
//这里的json文件是自己创建的，不是系统自带

配置文件的内容大致如下：

```
{
    "server":"服务器的ip",
    "server_port":服务器的端口,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"密码",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open":false
}
12345678910
```

## 启动ss客户端

前两步很简单，可是有人就纳闷了安装好了不知道怎么用，其实可以用sslocal -help 来查看帮助就知道了

sslocal -c /etc/shadowsocks.json

一条命令代理就可以启动了。 
这里仅仅是启动了shadowsocks还是不行的，我们还需要设置相关的代理。

## 转换HTTP代理

Shadowsocks默认是用Socks5协议的，对于Terminal的get,wget等走Http协议的地方是无能为力的，所以需要转换成Http代理，加强通用性，这里使用的转换方法是基于Polipo的。

输入命令安装Polipo： 
sudo apt-get install polipo

修改配置文件： 
sudo gedit /etc/polipo/config

将下面的内容整个替换到文件中并保存：

```
    # This file only needs to list configuration variables that deviate
    # from the default values. See /usr/share/doc/polipo/examples/config.sample
    # and "polipo -v" for variables you can tweak and further information.
    logSyslog = false
    logFile = "/var/log/polipo/polipo.log"

    socksParentProxy = "127.0.0.1:1080"
    socksProxyType = socks5

    chunkHighMark = 50331648
    objectHighMark = 16384

    serverMaxSlots = 64
    serverSlots = 16
    serverSlots1 = 32

    proxyAddress = "0.0.0.0"
    proxyPort = 8123
123456789101112131415161718
```

重启Polipo： 
/etc/init.d/polipo restart

验证代理是否正常工作： 
export http_proxy=”http://127.0.0.1:8123/” 
curl www.google.com

如果正常，就会返回抓取到的Google网页内容。

另外，在浏览器中输入<http://127.0.0.1:8123/>便可以进入到Polipo的使用说明和配置界面。

## 配置浏览器

在firefox中

preference->advanced->network->connection->settings中选择手动设置代理，并将http代理设置为127.0.0.1 端口8123 （就是之前第二步配置的port） 
做到这步应该就能通过shadowsocks访问了，但我遇到的电脑还是不行，后来将http代理下面的“Use this proxy server for all protocols（将代理应用到所有协议）”这个也钩上才可以了。

PS.如果跳过第二步，直接在第三部中配置http代理设置为127.0.0.1 端口1080，有些文章中是这样配置的，但是本人亲测这样无法连接上网。

## Ubuntu开机后自动运行

现在可以科学上网了，可是每次开机都要手动打开终端输入一条命令，虽然这条命令并不长，但是每次都去手动输入，显得自己很low,而且关掉终端代理就关闭了。

写个脚本

我们可以在比如/home下新建个文件叫做shadow.sh，在里面写上我们启动ss客户端需要的命令，然后保存即可。

```
#！/bin/bash
#shadow.sh
sslocal -c /etc/shadowsocks.json
123
```

看可不可以我们到终端执行命令 sh /home/shadow.sh，如果成功的话会有信息输出的。你也可以到浏览器去试试。这个时候你虽然输入的少了，可是关了终端还是会掉的，我们可以让他在后台运行，nohup sh /home/shadow.sh &。

加入开机运行

这里我们需要在/etc下编辑一个叫rc,local的文件，需要root权限，在终端先su获取root权限。

**这里问题来了，因为我们要开机启动，要使用root权限来执行前面写好的脚本，但如果你的shawdocks不是在root下装的话，执行脚本是就会报错：** 
“Traceback (most recent call last): 
File “/home/gaoxw/.local/bin/sslocal”, line 7, in 
from shadowsocks.local import main” 
**使用sudo安装还是会报上面这个错误。**

如果你有root帐号的话，然后vim /etc/rc.local编辑，在exit之前输入nohup bash /home/shadow.sh>/home/d.txt & 保存。

这个时候你可以reboot重启了，测试下看看能不能后台自动运行，重启你可以先去看下我们要他输出d.txt，你竟然发现是 /home/shadow.sh line 3 :sslocal: command not found,打开浏览器果然是无法链接代理服务器。

经过一番搜索我们发现远离linux是找不到sslocal这条命令？需要添加路径，我们发现sslocal和ssserver这两个命令是被存在 /usr/local/bin下面的，其实不用去profile添加了，直接把这两个文件移动到/bin下，就可以了。



### 填坑笔记

kubeadm init报错不能连接https://dl.k8.io.release/stable-1.11.txt解决方法

指定kubernetes = <version>

#### kubelet 不能启动问题

apiserver-amd64镜像没有启动成功

cgroup-driver，docker和kubelet不一致

journalctl -xeu kubelet 查看日志



#### 初始化 master

**等会有个坑，kubeadm 等相关 rpm 安装后会生成 /etc/kubernetes 目录，而 kubeadm init 时候又会检测这些目录是否存在，如果存在则停止初始化，所以要先清理一下，以下清理脚本来源于 官方文档 Tear down 部分，该脚本同样适用于初始化失败进行重置**

```
systemctl stop kubelet;
# 注意: 下面这条命令会干掉所有正在运行的 docker 容器，
# 如果要进行重置操作，最好先确定当前运行的所有容器都能干掉(干掉不影响业务)，
# 否则的话最好手动删除 kubeadm 创建的相关容器(gcr.io 相关的)
docker rm -f -v $(docker ps -q);
find /var/lib/kubelet | xargs -n 1 findmnt -n -t tmpfs -o TARGET -T | uniq | xargs -r umount -v;
rm -r -f /etc/kubernetes /var/lib/kubelet /var/lib/etcd;
```





### 资料的网址：

#### 1、kubeadm安装k8s集群的文档

https://k8s.qikqiak.com/docs/16.%E7%94%A8%20kubeadm%20%E6%90%AD%E5%BB%BA%E9%9B%86%E7%BE%A4%E7%8E%AF%E5%A2%83.html

#### 2、kubeadm 安装k8s集群的文档与1结合使用，k8s中文网

https://www.kubernetes.org.cn/878.html



要注意将上面的加入集群的命令保存下面，如果忘记保存上面的 token 和 sha256 值的话也不用担心，我们可以使用下面的命令来查找：

```
$ kubeadm token list
kubeadm token list
TOKEN                     TTL       EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
i5gbaw.os1iow5tdo17rwdu   23h       2018-05-18T01:32:55+08:00   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token
```

要查看 CA 证书的 sha256 的值的话，我们可以使用`openssl`来读取证书获取 sha256 的值：

```
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
e9ca4d9550e698105f1d8fae7ecfd297dd9331ca7d50b5493fa0491b2b4df40c
```



### 修改apt-get镜像下载地址

1、复制原文件备份 
sudo cp /etc/apt/source.list /etc/apt/source.list.bak

2、编辑源列表文件

sudo vim /etc/apt/source.list

3、将原来的列表删除,添加如下内容

\# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted

deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties

deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted

deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted

deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties

deb http://mirrors.aliyun.com/ubuntu/ xenial universe

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe

deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse

deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties

deb http://archive.canonical.com/ubuntu xenial partner

deb-src http://archive.canonical.com/ubuntu xenial partner

deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted

deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties

deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe

deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse

4、运行sudo apt-get update

5、运行sudo apt-get upgrade