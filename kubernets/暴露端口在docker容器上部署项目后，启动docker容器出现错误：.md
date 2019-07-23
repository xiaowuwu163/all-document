# 暴露端口在docker容器上部署项目后，启动docker容器出现错误：

# [iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 8001 -j DNAT --to-destination 172.17.0.5:8080 ! -i docker0: iptables: No chain/target/match by that name.](https://www.cnblogs.com/tongxuping/p/7297817.html)

在docker容器上部署项目后，启动docker容器，出现

**iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 8001 -j DNAT --to-destination 172.17.0.5:8080 ! -i docker0: iptables: No chain/target/match by that name.**

解决方案：

　　1、先看能不能ping通网络。若能依次执行以下命令；

　　2、pkill docker

　　3、iptables -t nat -F

　　4、ifconfig docker0 down

　　5、brctl delbr docker0

　　6、docker -d

　　7、systmctl restart docker 

　　8、重启docker服务