# ETCD 工具使用

### 1-获取所有key

```
 ETCDCTL_API=3 etcdctl --endpoints=https://172.10.0.210:2379 --cacert=/etc/kubernetes/ssl/ca.pem --key=/etc/etcd/ssl/etcd-key.pem  --cert=/etc/etcd/ssl/etcd.pem  get /registry/ --prefix   --keys-only
```

### 2-删除key

```
 ETCDCTL_API=3 etcdctl --endpoints=https://172.10.0.210:2379 --cacert=/etc/kubernetes/ssl/ca.pem --key=/etc/etcd/ssl/etcd-key.pem  --cert=/etc/etcd/ssl/etcd.pem  del [key值]
```

