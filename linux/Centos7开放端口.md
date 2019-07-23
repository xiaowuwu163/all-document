## Centos7开放端口

Centos升级到7之后，发现无法使用iptables控制Linuxs的端口，google之后发现Centos 7使用firewalld代替了原来的iptables。下面记录如何使用firewalld开放Linux端口： 
开启端口 

命令含义： 
--zone #作用域 
--add-port=80/tcp #添加端口，格式为：端口/通讯协议 
--permanent #永久生效，没有此参数重启后失效 
重启防火墙 

　　firewall-cmd --reload 

`# 清空现有规则`

`iptables -F`

`# 对于外网（WAN）到内网（LAN）的封包，至允许那些回应包`

`iptables -A FORWARD -i eth0 -o eht1 -m state --state ESTABLISHED,RELATED -j ACCEPT`

`# 对于所有内网（LAN）到外网（WAN）的封包都予以放行`

`iptables -A FORWARD -i eht0 -o eth1 -j ACCEPT`

`#启用转发日志`

`iptables -A FORWARD -j LOG`

`# 启用IP伪装，使得内网中所有转发出去的封包都是Linux服务器一台机子发出的`

`iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE`

`# 启用地址转换（NAT）将内网的Web服务映射到外网的特定端口上`

`iptables -A PREROUTING -t nat -p tcp --dport 8080 -j DNAT --to-destination 192.168.0.2:8080`