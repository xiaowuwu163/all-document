

# ingress nginx 路由重新定义

https://blog.csdn.net/qingyafan/article/details/82692509



nginx-ingress 高级配置

 <https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md> 。 



# Kubernetes - nginx-ingress 配置跳坑指南

2018年09月13日 18:12:01

 

庆祝亚运会

 

阅读数：107

更多

所属专栏： [容器云](https://blog.csdn.net/column/details/28018.html)

版权声明：本文为博主原创文章，未经博主允许不得转载。	https://blog.csdn.net/qingyafan/article/details/82692509

GitHub地址： <https://github.com/QingyaFan/container-cloud/blob/master/kubernetes-ingress-2018-06-e.md>

Ingress是Kubernetes集群对外暴露服务的一种推荐方式，Ingress其实是封装了nginx（也可以是traefic），背后还是nginx在发挥作用，Ingress的作用是不断检测服务的endpoint的变化，然后将变化更新到nginx的配置中。

## 简单配置

Ingress有一个yaml配置文件，Ingress根据这个配置文件生成nginx的配置文件，格式像下面这样（test.yaml）：

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /service/
        backend:
          serviceName: app1
          servicePort: 808012345678910111213
```

执行`kubectl create -f test.yaml`，通过`example.com/app1/`就可以访问到app1的`/service/`目录，这里需要注意，app1中需要定义`/service/`路由，否则会出现错误，以上的配置文件最终的nginx规则是：

> example.com/service/ –> app1:8080/service/

如果app1中没有定义`service`这个路由，那么需要做一个rewrite。例如，app1中定义了`/v1/`，想通过`/service/`访问`v1`，这在nginx很容易做到，在Ingress中需要通过rewrite实现：

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite /service/(.*) /v1/$1 break;
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /v1/
        backend:
          serviceName: app1
          servicePort: 808012345678910111213141516
```

## 高级配置

Ingress背后是Nginx，所以Nginx能实现的Ingress同样可以实现，但是实现方式不太一样，基本都是通过`annotations`实现的，各种配置可以参考文档： <https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md> 。

## 配置HTTPS

首先创建一个secret资源，来存储秘钥和证书。如果你的证书是.pem的，也是一样的，将下面的your_cert.crt改为your_cert.pem即可。

```
kubectl create secret tls tls_secret_name --key your_key.key --cert your_cert.crt1
```

然后在Ingress的配置文件中添加配置使用这个secret，然后更新配置`kubectl replace -f ingress.yaml`，即可。

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite /service/(.*) /v1/$1 break;
spec:
  tls:
    - hosts:
      - example.com
      secretName: tls_secret_name
  rules:
  - host: example.com
    http:
      paths:
      - path: /v1/
        backend:
          serviceName: app1
          servicePort: 80801234567891011121314151617181920
```

## 总结

本文主要介绍了nginx版本的Ingress的配置。



## 部署

http://blog.51cto.com/newfly/2060587