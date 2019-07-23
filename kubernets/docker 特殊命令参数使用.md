# docker 特殊命令参数使用

启动systemctl 服务需要启动 dbus服务



docker run --privileged  -ti -e "container=docker"  -v /sys/fs/cgroup:/sys/fs/cgroup  centos  /usr/sbin/init 