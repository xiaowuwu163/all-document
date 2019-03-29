## virtualenvwrapper 虚拟python环境

　　鉴于virtualenv不便于对虚拟环境集中管理，所以推荐直接使用virtualenvwrapper。 virtualenvwrapper提供了一系列命令使得和虚拟环境工作变得便利。它把你所有的虚拟环境都放在一个地方。

　　安装virtualenvwrapper(确保virtualenv已安装)

ln -s /usr/local/python3/bin/virtualenv /usr/local/bin/virtualenv

```
pip install virtualenvwrapper
pip install virtualenvwrapper-win　　#Windows使用该命令
```

　　安装完成后，在~/.bashrc写入以下内容

```
export WORKON_HOME=~/Envs
source /usr/local/python3/bin/virtualenvwrapper.sh　　
```

　　第一行：virtualenvwrapper存放虚拟环境目录

　　第二行：virtrualenvwrapper会安装到python的bin目录下，所以该路径是python安装目录下bin/virtualenvwrapper.sh

```
source ~/.bashrc　　　　#读入配置文件，立即生效
```

　

　virtualenvwrapper基本使用

1.创建虚拟环境　**mkvirtualenv**

```
mkvirtualenv venv　　　
```

　　这样会在WORKON_HOME变量指定的目录下新建名为venv的虚拟环境。

　　若想指定python版本，可通过"--python"指定python解释器

```
mkvirtualenv --python=/usr/local/python3.5.3/bin/python venv
```

\2. 基本命令 　

　　查看当前的虚拟环境目录

```
[root@localhost ~]# workon
py2
py3
```

　　切换到虚拟环境

```
[root@localhost ~]# workon py3
(py3) [root@localhost ~]# 
```

　　退出虚拟环境

```
(py3) [root@localhost ~]# deactivate
[root@localhost ~]# 
```

　　删除虚拟环境

```
rmvirtualenv venv
```