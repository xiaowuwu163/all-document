# Grafana 允许匿名访问的配置方法

在 K8S 系统中，Grafana 往往扮演监控数据展示层的角色，

有些场景下，需要暂时关闭 grafana 的登陆验证功能，方便与其他系统集成

开启匿名访问功能

对于使用 grafana/grafana:5.1.0 镜像部署的 grafana

可以配置以下环境变量，关闭验证
--------------------- 
GF_AUTH_PROXY_ENABLED: true

GF_AUTH_ANONYMOUS_ENABLED: true

#### 修改yaml文件变量可配置开启验证

