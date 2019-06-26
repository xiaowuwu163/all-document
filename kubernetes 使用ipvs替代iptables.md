# kubernetes 使用ipvs替代iptables

2019年01月20日 22:39:00 [weixin_34202952](https://me.csdn.net/weixin_34202952) 阅读数：138

# 使用ipvs

> 从k8s的1.8版本开始，kube-proxy引入了IPVS模式，IPVS模式与iptables同样基于Netfilter，但是采用的hash表，因此当service数量达到一定规模时，hash查表的速度优势就会显现出来，从而提高service的服务性能。

## 开启内核参数

```
cat >> /etc/sysctl.conf << EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF


sysctl -p
```

## 开启ipvs支持

```
yum -y install ipvsadm  ipset

# 临时生效
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4


# 永久生效



cat > /etc/sysconfig/modules/ipvs.modules <<EOF
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF


# 授权
chmod 755 /etc/sysconfig/modules/ipvs.modules 


# 加载模块
bash /etc/sysconfig/modules/ipvs.modules


# 查看加载
lsmod | grep -e ip_vs -e nf_conntrack_ipv4

# 输出如下:
-----------------------------------------------------------------------
nf_conntrack_ipv4      20480  0 
nf_defrag_ipv4         16384  1 nf_conntrack_ipv4
ip_vs_sh               16384  0 
ip_vs_wrr              16384  0 
ip_vs_rr               16384  0 
ip_vs                 147456  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack          110592  2 ip_vs,nf_conntrack_ipv4
libcrc32c              16384  2 xfs,ip_vs
-----------------------------------------------------------------------

```

## 配置kube-proxy

```
# 添加下面两行

  --proxy-mode=ipvs  \
  --masquerade-all=true \


# 修改服务文件



vim /usr/lib/systemd/system/kube-proxy.service

[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target
[Service]
WorkingDirectory=/data/k8s/kube-proxy
ExecStart=/data/k8s/bin/kube-proxy \
  --bind-address=192.168.1.145 \
  --hostname-override=192.168.1.145 \
  --cluster-cidr=10.254.0.0/16 \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig \
  --logtostderr=true \
  --proxy-mode=ipvs  \
  --masquerade-all=true \
  --v=2
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target 
```

## 重启kube-proxy

```
systemctl daemon-reload
systemctl restart kube-proxy
systemctl status kube-proxy
```

# 测试是否生效

```
[root@k8sNode01 docker]# ipvsadm -Ln



IP Virtual Server version 1.2.1 (size=4096)



Prot LocalAddress:Port Scheduler Flags



  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn



TCP  10.254.0.1:443 rr



  -> 192.168.1.142:6443           Masq    1      0          0         



  -> 192.168.1.143:6443           Masq    1      1          0         



  -> 192.168.1.144:6443           Masq    1      1          0         



TCP  10.254.27.38:80 rr



  -> 172.30.36.4:9090             Masq    1      0          0         



TCP  10.254.72.60:80 rr



  -> 172.30.90.4:8080             Masq    1      0          0         



TCP  10.254.72.247:80 rr



  -> 172.30.36.5:3000             Masq    1      0          0         



TCP  127.0.0.1:27841 rr



  -> 172.30.36.2:80               Masq    1      0          0         



  -> 172.30.90.2:80               Masq    1      0          0         



TCP  127.0.0.1:28453 rr



  -> 172.30.36.5:3000             Masq    1      0          0         



TCP  127.0.0.1:36018 rr



  -> 172.30.36.4:9090             Masq    1      0          0         



TCP  172.30.90.0:27841 rr



  -> 172.30.36.2:80               Masq    1      0          0         



  -> 172.30.90.2:80               Masq    1      0          0    
```