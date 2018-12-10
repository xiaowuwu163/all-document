# [kubernetes dashboard 二次开发环境配置](https://www.kubernetes.org.cn/4361.html)

2018-07-25 17:53 [王善鹏](https://www.kubernetes.org.cn/author/wsp23212) 分类：[Kubernetes实践分享/开发实战](https://www.kubernetes.org.cn/practice) 阅读(1934)	评论(0) 

# 前言

kubernetes的dashboard可以用来显示集群的状态，通过dashboard可以很方便的管理集群、管理容器等。我们可以对官方的dashboard进行二次开发，部署有特色的dashboard应用。本文作者在实际配置过程中大部分情况使用的是一般账户，偶尔因为需要权限会使用root账户，请读者在阅读的时候注意一下。本文作者使用集群外独立的机器配置，配置时需要保证机器能和集群通信。本文基于CentOS 7.0，kubernetes 1.8，dashboard 1.8.3配置。

# 一、安装docker1.12

在配置dashboard开发环境前需要安装docker，本文基于的docker版本为docker1.12。安装docker有很多方法，本文不在赘述，

# 二、安装go

- 下载# wget -c https://golang.google.cn/dl/go1.10.2.linux-amd64.tar.gz
- 解压# tar -C /usr/lib -xzf go1.10.2.linux-amd64.tar.gz
- 修改环境变量

> \# vi ~/.bash_profile
>
> 添加 export GOROOT**=**/usr/lib/go
>
> 添加export PATH=$PATH: $GOROOT/bin :$GOPATH/bin
>
> 添加export GOPATH=$HOME/goworkspace
>
> \# source ~/.bash_profile

- 测试# go version

# 三、安装 node8.5 和 npm5.3

- 下载# wget -c  <https://npm.taobao.org/mirrors/node/v8.5.0/node-v8.5.0-linux-x64.tar.gz>
- 解压$ sudo tar -C /usr/lib -xzf node-v8.5.0-linux-x64.tar.gz
- 测试版本号$ /usr/lib/node-v8.5.0-linux-x64/bin/node –v
- 创建链接# ln -s /usr/lib/node-v8.5.0-linux-x64/bin/npm /usr/local/bin/npm
- 创建链接# ln -s /usr/lib/node-v8.5.0-linux-x64/bin/node /usr/local/bin/node
- 创建链接# ln -s /usr/local/bin/node /usr/bin/node
- 创建链接# ln -s /usr/local/bin/npm /usr/bin/npm
- 测试命令$ node –v
- 测试命令$ npm –v

# **四、**安装**Gulp.js** **3.9+**

- 安装$ sudo npm install –global gulp-cli

> 显示CLI version 2.0.1，先不用在意。

- 测试版本号$ /usr/lib/node-v8.5.0-linux-x64/bin/gulp –v
- 创建链接$ sudo ln -s /usr/lib/node-v8.5.0-linux-x64/bin/gulp /usr/local/bin/gulp
- 创建链接$ ln -s /usr/local/bin/gulp /usr/bin/gulp
- 测试命令$ gulp –v

# 五、设置dashboard开发环境

## 1.    获取源码

- 创建文件夹$HOME/goworkspace/src/github.com/kubernetes/
- 进入目录$ cd $HOME/goworkspace/src/github.com/kubernetes/
- 下载1.8.3版本$ wget -c https://github.com/kubernetes/dashboard/archive/v1.8.3.tar.gz
- 解压$ tar -zvxf v1.8.3.tar.gz
- 修改目录名称$ mv dashboard-1.8.3 dashboard
- 进入目录$ cd dashboard

### 2.      其他配置

#### *执行前应该做的操作：下载git工具，下载g++工具,否则安装npm依赖报错,留意 npm i --unsafe-perm  命令下出现的一些错误，根据需要安装缺少的插件工具

#### yum install -y git

#### yum install gcc-c++

#### yum install patch

- 下载依赖库$ npm i –unsafe-perm

> 发现本地版本是3.9.1，全局是2.0.1。没关系，继续往下配置。

- 修改环境变量

> $ vi ~/.bash_profile
>
> 添加export KUBE_DASHBOARD_KUBECONFIG=”$HOME/.kube/webconfig”
>
> $ source ~/.bash_profile

- 将集群的kubeconfig文件放到$HOME/.kube/目录中
- 修改build/serve.js

> 找到target: conf.frontend.serveHttps
>
> 将https://localhost:${conf.backend.secureDevServerPort}和http://localhost:${conf.backend.devServerPort}中的localhost修改为127.0.0.1

- 启动服务$ gulp serve
- 在浏览器中输入ip:9090进行访问。

**PS：**开发模式访问如果出现空白考虑使用其他浏览器访问，本文使用火狐不能访问但是chrome可以，开发模式只有英语没有国际化。

# 附录：

# A. 安装heapster组件

如果集群中没有Heapster组件，dashboard后台会提示缺少Heapster组件，但不影响使用。Heapster组件用来使得dashboard能够显示图表。

- 下载源码$ wget -c <https://github.com/kubernetes/heapster/archive/v1.5.3.tar.gz>
- 解压$ tar -zvxf heapster-1.5.3.tar.gz
- 进入目录$ cd heapster-1.5.3/deploy/kube-config/influxdb
- 修改文件

> $ vi influxdb.yaml
>
> 将image那行修改成image: docker.io/wanghkkk/heapster-influxdb-amd64-v1.3.3:v1.3.3或者自己找到的镜像地址
>
> $ vi grafana.yaml
>
> 将image那行修改成image: docker.io/wanghkkk/heapster-grafana-amd64-v4.4.3:v4.4.3或者自己找到的镜像地址
>
> $ vi heapster.yaml
>
> 将image那行修改成image: docker.io/wanghkkk/heapster-amd64-v1.4.0:v1.4.0或者自己找到的镜像地址

- 创建组件

> $ kubectl create -f heapster.yaml
>
> $ kubectl create -f influxdb.yaml
>
> $ kubectl create -f grafana.yaml

# B. 打包成镜像

在dashboard目录中执行$ gulp docker-image:head就打包成kubernetes/kubernetes-dashboard-amd64:head镜像。该镜像可以部署到集群中使用。

# C. 安装go开发插件

虽然vim可以编辑go文件，但是vim本身不具有go的开发工具，go开发的时候比较难于控制，所以为了方便go的开发，需要安装一些vim的go开发插件。我们使用vundle来作为vim的插件管理工具。

## 1.    升级vim到8.0

- \# cd /etc/yum.repos.d/
- 获取yum源 # wget [https://copr.fedorainfracloud.org/coprs/mcepl/vim8/repo/epel-7/mcepl-vim8-epel-7.repo ](https://copr.fedorainfracloud.org/coprs/mcepl/vim8/repo/epel-7/mcepl-vim8-epel-7.repo)也可以不使用这个源，找其他或者自己做一个

- 导入证书 # rpm –import <https://copr-be.cloud.fedoraproject.org/results/mcepl/vim8/pubkey.gpg>
- 卸载vim-minimal # yum erase vim-minimal
- 升级 # yum update vim
- 验证 # vim –version
- 修改 # vim /etc/bashrc

> 其中最后一行添加alias vi=’vim’

- 使设置生效 # source /etc/bashrc

**PS：**卸载vim-minimal是因为升级过程中可能会冲突，并且vim-minimal和vim的版本也不同，但这也会导致卸载sudo。安装 sudo # yum install sudo，这也会安装新版本的vim-minimal，如果此时已经退出了管理员账户可以使用 $ pkexec yum install sudo。

## 2.    安装vundle.vim

- 创建文件夹 ~/.vim/bundle
- 下载vim $ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
- 创建并配置文件$ vi ~/.vimrc

> set nocompatible ” be iMproved, required
> filetype off ” required
> set rtp+=~/.vim/bundle/Vundle.vim
> call vundle#begin()
> Plugin ‘VundleVim/Vundle.vim’
> Plugin ‘fatih/vim-go’
> call vundle#end() ” required
> filetype plugin indent on ” required

## 3.    安装vim-go

- 运行命令$ vim +PluginInstall +qall

## 4.    安装 go.tools Binaries 二进制工具

- 创建文件夹$GOPATH\src\golang.org\x\
- 进入目录$ cd $GOPATH\src\golang.org\x\
- 下载源码$ git clone https://github.com/golang/tools.git tools
- 下载源码$ git clone https://github.com/golang/lint.git lint
- 安装插件在vim下输入:GoInstallBinaries

**PS：**插件支持特性参考<https://github.com/fatih/vim-go/blob/master/doc/vim-go.txt>

## 问题i.          vim 退格键（backspace）不能用

- 编辑~/.vimrc加入下面2行：$ vi ~/.vimrc

> set nocompatible
>
> set backspace=indent,eol,start

# D. delve调试方法

dashboard后台go代码的调试不同于普通go程序的调试，因为它是长时间驻在后台的服务，因而是一个长期不间断运行的程序。详细调试命令请参考[https://github.com/derekparker/delve/blob/master/Documentation/cli/README.md。](https://github.com/derekparker/delve/blob/master/Documentation/cli/README.md%E3%80%82)

## 1.    普通go程序调试

- 编写go文件test-debug.go

![img](https://www.kubernetes.org.cn/img/2018/07/%E6%8D%95%E8%8E%B7.png)

- 用调试器启动程序$ dlv debug test-debug.go
- 设置断点(dlv) break main.main
- 让程序运行到设置断点的地方(dlv) continue
- 程序停在断点的地方之后就可以开始调试了。
- 不断(dlv) continue，程序会正常退出。

 

## 2.    Attach到已经运行的程序调试

其实很多时候，我们调试的代码可能是daemon程序或者需要实现编译好在不同机器运行的程序。这就需要我们attach到一个已经在运行中的程序上。

- 编译程序为二进制文件$ go build test-debug.go
- 运行程序$ ./test-debug
- 查看当前运行程序的进程号$ ps –aux|grep test-debug
- 显示的第二列数字N是进程号，attach上$ dlv attach N
- 设置断点(dlv) break dostuff (dlv) break dostuff:3
- 让程序运行到设置断点的地方(dlv) continue
-  程序停在断点的地方之后就可以开始调试了。
- 不断(dlv) continue，程序会正常退出。

## 3.    Dashboard调试

Dashboard也需要在attach之后进行调试，但是因为它是常驻服务，在人为退出之前程序不会结束，所以continue之后会一直等待，直到程序会停在断点处。

- 参照上述方法启动服务 $ gulp serve
- 查看当前运行程序的进程号$ ps –aux|grep dashboard
- 显示的第二列数字N是进程号，attach上$ dlv attach N
- 设置断点(dlv) break b github.com/kubernetes/dashboard/src/app/backend/auth.AuthHandler.handleLogin:2
- 让程序运行(dlv) continue，这时程序会一直停住等待。
- 在网页上选择完kubeconfig文件执行登录操作，这时程序会停在断点处。这时就可以进行调试了。
- 调试完之后(dlv) continue程序会继续停住等待，直到下次在网页上操作程序会停在断点处。

**PS：**Dashboard的main函数因为是在启动服务时就运行的，因而在服务启动后没有办法调试，在服务启动进行时，可以attach到程序设置断点应该可以调试main函数，可以在main数中增加time.Sleep函数增加启动时间进行调试，其他启动服务时运行的函数也可以照此方法进行调试。

# 结语

本文作者参考了官方dashboard开发的配置方法<https://github.com/kubernetes/dashboard/wiki/Getting-started>。虽然官方dashboard的前后端代码都是放在一起的，但是前端与后端的代码是可以分离的，前端是angular的单页应用，后端采用go的restful框架，通过rest服务进行通信，部署时用gulp对前后端统一进行打包。在开发过程中，如果团队比较大可以采用前后端分开开发和部署的模式，前端只编写和部署angular代码而后端只编写和部署go代码。这样前端和后端的开发人员可以在自己熟悉的平台和开发环境中进行开发和单元测试。部署时可以参考官方的gulp文件编写自己的gulp文件。