# linux搭建k8s多节点集群

### 准备工作：

​	master主机：本机

​	node-1节点：virtualbox虚拟机

​	node-2节点：virtualbox虚拟机

#### 1.1 安装docker

​	最快速linux系统install方法:

`curl -sSL https://get.daocloud.io/docker | sh`

- 需要提示的是，如果你使用桥接模式后虚拟机无法上网(Ping不通www.baidu.com，我遇到了这个问题，目前还没解决，但依然可以建立集群，尚不知对Kubernetes集群的影响)，但宿主机可以上网，你可以在设置中将网络模式改为`NAT`模式，安装完Docker后再改回桥接模式(NAT模式下，宿主机无法访问虚拟机，不能建立集群). 

#### 1.2 下来配置Master到两个Minion的SSH无密码访问 

一直按回车键跳过输入的部分 ,**注意，配置的登陆用户是root用户**。 

`ssh-keygen -t rsa `

#### 1.3下载kubernetes源码

` git clone "https://gitee.com/lyric/kubernetes.git" `





