# linux各种环境下内核参数配置，以及解释

2018年05月14日 14:54:23 [脉脉此情](https://me.csdn.net/H_haow) 阅读数 1262

# Oracle环境下

文件/etc/sysctl.conf

```
#同时可以拥有的的异步IO请求数目
fs.aio-max-nr = 1048576
#系统级别的能够打开的文件句柄的数量
fs.file-max = 6815744
#整个系统共享内存段的最大数目
kernel.shmmni = 4096
# 每个信号对象集的最大信号对象数；系统范围内最大信号对象数；每个信号对象支持的最大操作数；系统范围内最大信号对象集数
kernel.sem = 250 32000 100 128
#系统中的程序会选择这个范围内的端口来连接到目的端口（目的端口当然是用户指定的）
net.ipv4.ip_local_port_range = 9000 65500
#接收套接字缓冲区大小的默认值(以字节为单位)
net.core.rmem_default=262144
#接收套接字缓冲区最大值(以字节为单位)
net.core.rmem_max=4194304
# 发送套接字缓冲区大小的默认值
net.core.wmem_default=262144
# 发送套接字缓冲区大小的最大值
net.core.wmem_max=1048586
```

# CDH环境

```
#最大限度地降低了使用swap的可能性
vm.swappiness=0
//
#关闭大内存分页
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

# ES环境

文件/etc/sysctl.conf

```
#进程能拥有的最多内存区域
vm.max_map_count=655360
#最大限度地降低了使用swap的可能性
vm.swappiness=0
```

# OpenStack环境

```
#ip转发功能，0表示禁止数据包转发，1表示允许
net.ipv4.ip_forward=1
#源路由核查功能1是启用，0是禁用
net.ipv4.conf.default.rp_filter=0
#禁用所有IP源路由
net.ipv4.conf.default.accept_source_route=0
#开启允许绑定非本机的IP，1表示允许，0表示禁用
net.ipv4.ip_nonlocal_bind=1
#禁用SysRq（组合键）功能
kernel.sysrq=0
#控制core文件的文件名中是否添加pid作为扩展。文件内容为1，表示添加pid作为扩展名，生成的core文件格式为core.xxxx；为0则表示生成的core文件同一命名为core
kernel.core_uses_pid=1
#开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击
net.ipv4.tcp_syncookies=1
#二层的网桥在转发包时不会会被iptables的FORWARD规则所过滤，值为1的时候则会被过滤
net.bridge.bridge-nf-call-ip6tables=0
net.bridge.bridge-nf-call-iptables=0
net.bridge.bridge-nf-call-arptables=0
#每个消息队列的最大字节限制。该文件指定一个消息队列的最大长度
kernel.msgmnb=65536
#整个系统的最大数量的消息队列。该文件指定消息队列标识的最大数目，即系统范围内最大多少个消息队列
kernel.msgmax=65536
#定义单个共享内存段的最大值
kernel.shmmax=68719476736
#控制共享内存页数，Linux 共享内存页大小为4KB, 共享内存段的大小都是共享内存页大小的整数倍。一个共享内存段的最大大小是16G，那么需要共享内存页数是 16GB/4KB=16777216KB/4KB=4194304
kernel.shmall=4294967296
#
net.nf_conntrack_max=1048576
#内核追踪文件的路径和命名方式
kernel.core_pattern=/tmp/core-%e-%s-%u-%g-%p-%t
#最大限度地降低了使用swap的可能性
vm.swappiness=0
#系统级别的能够打开的文件句柄的数量
fs.file-max=1126400
#预留端口避免占用
net.ipv4.ip_local_reserved_ports=49000,35357,41055,58882,6376
#当探测没有确认时，重新发送探测的频度，3秒
net.ipv4.tcp_keepalive_intvl=3
#当keepalive起用的时候，TCP发送keepalive消息的频度，30秒
net.ipv4.tcp_keepalive_time=30
#如果对方不予应答，探测包的发送次数
net.ipv4.tcp_keepalive_probes=8
#在丢弃激活(已建立通讯状况)的TCP连接之前﹐需要进行多少次重试
net.ipv4.tcp_retries2=5
#为每个TCP连接分配的读、写缓冲区内存大小，单位是Byte，第一个数字表示，为TCP连接分配的最小内存，第二个数字表示，为TCP连接分配的缺省内存，第三个数字表示，为TCP连接分配的最大内存
net.ipv4.tcp_rmem=4096  4096    167772160
#接收套接字缓冲区大小的默认值(以字节为单位)
net.core.rmem_default=671088640
#1表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭
net.ipv4.tcp_tw_reuse=1
#松散模式0，严谨模式为1
net.ipv4.conf.all.rp_filter=0
#发送套接字缓冲区大小的最大值
net.core.wmem_max=671488640
#发送套接字缓冲区大小的默认值
net.core.wmem_default=671488640
#每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目
net.core.netdev_max_backlog=300000
#设置tcp/ip会话的滑动窗口大小是否可变。参数值为布尔值,为1时表示可变,为0时表示不可变
net.ipv4.tcp_window_scaling=1
#开启TCP时间戳
net.ipv4.tcp_timestamps=0
#为每个TCP连接分配的读、写缓冲区内存大小，单位是Byte，第一个数字表示，为TCP连接分配的最小内存，第二个数字表示，为TCP连接分配的缺省内存，第三个数字表示，为TCP连接分配的最大内存
net.ipv4.tcp_wmem=4096  4096    167772160
#禁用ipv6
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.all.disable_ipv6=1
#启用内核追踪
fs.suid_dumpable=2
#启用有选择的应答（1表示启用），通过有选择地应答乱序接收到的报文来提高性能，让发送者只发送丢失的报文段，（对于广域网通信来说）这个选项应该启用，但是会增加对CPU的占用
net.ipv4.tcp_sack=1
#接收套接字缓冲区最大值(以字节为单位)
net.core.rmem_max=671488640
```