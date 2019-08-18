# Nginx配置详解



### **序言**

### Nginx是lgor Sysoev为俄罗斯访问量第二的rambler.ru站点设计开发的。从2004年发布至今，凭借开源的力量，已经接近成熟与完善。

### Nginx功能丰富，可作为HTTP服务器，也可作为反向代理服务器，邮件服务器。支持FastCGI、SSL、Virtual Host、URL Rewrite、Gzip等功能。并且支持很多第三方的模块扩展。

### Nginx的稳定性、功能集、示例配置文件和低系统资源的消耗让他后来居上。

### **Nginx常用功能**

### 1、Http代理，反向代理：作为web服务器最常用的功能之一，尤其是反向代理。

### 这里我给来2张图，对正向代理与反响代理做个诠释，具体细节，大家可以翻阅下资料。

### ![img](https://img-blog.csdn.net/2018081712345975?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3ODg3NzY0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### Nginx在做反向代理时，提供性能稳定，并且能够提供配置灵活的转发功能。Nginx可以根据不同的正则匹配，采取不同的转发策略，比如图片文件结尾的走文件服务器，动态页面走web服务器，只要你正则写的没问题，又有相对应的服务器解决方案，你就可以随心所欲的玩。并且Nginx对返回结果进行错误页跳转，异常判断等。如果被分发的服务器存在异常，他可以将请求重新转发给另外一台服务器，然后自动去除异常服务器。

### 2、负载均衡

### Nginx提供的负载均衡策略有2种：内置策略和扩展策略。内置策略为轮询，加权轮询，Ip hash。扩展策略，就天马行空，只有你想不到的没有他做不到的啦，你可以参照所有的负载均衡算法，给他一一找出来做下实现。

### Ip hash算法，对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题。

### 3、web缓存

### Nginx可以对不同的文件做不同的缓存处理，配置灵活，并且支持FastCGI_Cache，主要用于对FastCGI的动态程序进行缓存。配合着第三方的ngx_cache_purge，对制定的URL缓存内容可以的进行增删管理。

### **Nginx配置文件结构**

### 默认的config

```
#user nobody;worker_processes 1;#error_log logs/error.log;#error_log logs/error.log notice;#error_log logs/error.log info;#pid    logs/nginx.pid;events {  worker_connections 1024;}http {  include    mime.types;  default_type application/octet-stream;  #log_format main '$remote_addr - $remote_user [$time_local] "$request" '  #         '$status $body_bytes_sent "$http_referer" '  #         '"$http_user_agent" "$http_x_forwarded_for"';  #access_log logs/access.log main;  sendfile    on;  #tcp_nopush   on;  #keepalive_timeout 0;  keepalive_timeout 65;  #gzip on;  server {    listen    80;    server_name localhost;    #charset koi8-r;    #access_log logs/host.access.log main;    location / {      root  html;      index index.html index.htm;    }    #error_page 404       /404.html;    # redirect server error pages to the static page /50x.html    #    error_page  500 502 503 504 /50x.html;    location = /50x.html {      root  html;    }    # proxy the PHP scripts to Apache listening on 127.0.0.1:80    #    #location ~ \.php$ {    #  proxy_pass  http://127.0.0.1;    #}    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000    #    #location ~ \.php$ {    #  root      html;    #  fastcgi_pass  127.0.0.1:9000;    #  fastcgi_index index.php;    #  fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;    #  include    fastcgi_params;    #}    # deny access to .htaccess files, if Apache's document root    # concurs with nginx's one    #    #location ~ /\.ht {    #  deny all;    #}  }  # another virtual host using mix of IP-, name-, and port-based configuration  #  #server {  #  listen    8000;  #  listen    somename:8080;  #  server_name somename alias another.alias;  #  location / {  #    root  html;  #    index index.html index.htm;  #  }  #}  # HTTPS server  #  #server {  #  listen    443 ssl;  #  server_name localhost;  #  ssl_certificate   cert.pem;  #  ssl_certificate_key cert.key;  #  ssl_session_cache  shared:SSL:1m;  #  ssl_session_timeout 5m;  #  ssl_ciphers HIGH:!aNULL:!MD5;  #  ssl_prefer_server_ciphers on;  #  location / {  #    root  html;  #    index index.html index.htm;  #  }  #}}
```

 

 

### nginx文件结构

```
...       #全局块events {     #events块  ...}http   #http块{  ...  #http全局块  server    #server块  {     ...    #server全局块    location [PATTERN]  #location块    {      ...    }    location [PATTERN]     {      ...    }  }  server  {   ...  }  ...   #http全局块}
```

### 1、全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。

### 2、events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。

### 3、http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。

### 4、server块：配置虚拟主机的相关参数，一个http中可以有多个server。

### 5、location块：配置请求的路由，以及各种页面的处理情况。

### 下面给大家上一个配置文件，作为理解，同时也配入我搭建的一台测试机中，给大家示例。

```
########### 每个指令必须有分号结束。##################user administrator administrators; #配置用户或者组，默认为nobody nobody。#worker_processes 2; #允许生成的进程数，默认为1#pid /nginx/pid/nginx.pid;  #指定nginx进程运行文件存放地址error_log log/error.log debug; #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emergevents {  accept_mutex on;  #设置网路连接序列化，防止惊群现象发生，默认为on  multi_accept on; #设置一个进程是否同时接受多个网络连接，默认为off  #use epoll;   #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport  worker_connections 1024;  #最大连接数，默认为512}http {  include    mime.types;  #文件扩展名与文件类型映射表  default_type application/octet-stream; #默认文件类型，默认为text/plain  #access_log off; #取消服务日志    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式  access_log log/access.log myFormat; #combined为日志格式的默认值  sendfile on;  #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。  sendfile_max_chunk 100k; #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。  keepalive_timeout 65; #连接超时时间，默认为75s，可以在http，server，location块。   upstream mysvr {     server 127.0.0.1:7878;   server 192.168.10.121:3333 backup; #热备  }  error_page 404 https://www.baidu.com; #错误页  server {    keepalive_requests 120; #单连接请求上限次数。    listen    4545;  #监听端口    server_name 127.0.0.1;  #监听地址        location ~*^.+$ {    #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。      #root path; #根目录      #index vv.txt; #设置默认页      proxy_pass http://mysvr; #请求转向mysvr 定义的服务器列表      deny 127.0.0.1; #拒绝的ip      allow 172.18.5.54; #允许的ip          }   }}
```

### **上面是nginx的基本配置，需要注意的有以下几点：**

### 1.$remote_addr 与$http_x_forwarded_for 用以记录客户端的ip地址；

### 2.$remote_user ：用来记录客户端用户名称；

### 3.$time_local ： 用来记录访问时间与时区；

### 4.$request ： 用来记录请求的url与http协议；

### 5.$status ： 用来记录请求状态；成功是200，

### 6.$body_bytes_s ent ：记录发送给客户端文件主体内容大小；

### 7.$http_referer ：用来记录从那个页面链接访问过来的；

### 8.$http_user_agent ：记录客户端浏览器的相关信息；

### 2、惊群现象：一个网路连接到来，多个睡眠的进程被同事叫醒，但只有一个进程能获得链接，这样会影响系统性能。

### 3、每个指令必须有分号结束。

### location

### 语法    location [=|~*|^~|@]/uri/{…}

### location会尝试根据用户请求中的uri来匹配上面的uri表达式，如果可以匹配，就选择lcoation块中的配置来处理用户请求。当然，匹配方式是多样的。

## = 表示完全匹配

## ~表示匹配URI时时大小写敏感的

## ~*表示匹配URI时忽略大小写

## ^~表示匹配URI时只需要其前半部分匹配即可

## @表示仅用于Nginx服务内部请求之间的重定向

### location是有顺序的，如果一个请求有可能被多个location匹配，实际上这个请求会被第一个location处理。  最后：location / {}会处理所有的请求。

### root和alias的区别：

```
location /img/ {    alias /var/www/image/;}#若按照上述配置的话，则访问/img/目录里面的文件时，ningx会自动去/var/www/image/目录找文件location /img/ {    root /var/www/image;}#若按照这种配置的话，则访问/img/目录下的文件时，nginx会去/var/www/image/img/目录下找文件。]
```

# [nginx之配置proxy_set_header](https://www.cnblogs.com/jsonhc/p/7199295.html)

### 使用Nginx后如何在web应用中获取用户ip及原理解释

### **问题背景：**

### 在实际应用中，我们可能需要获取用户的ip地址，比如做异地登陆的判断，或者统计ip访问次数等，通常情况下我们使用request.getRemoteAddr()就可以获取到客户端ip，但是当我们使用了nginx作为反向代理后，使用request.getRemoteAddr()获取到的就一直是nginx服务器的ip的地址，那这时应该怎么办？

### **part1：解决方案**

### 我在查阅资料时，有一本名叫《实战nginx》的书，作者张晏，这本书上有这么一段话“经过反向代理后，由于在客户端和web服务器之间增加了中间层，因此web服务器无法直接拿到客户端的ip，通过$remote_addr变量拿到的将是反向代理服务器的ip地址”。这句话的意思是说，当你使用了nginx反向服务器后，在web端使用request.getRemoteAddr()（本质上就是获取$remote_addr），取得的是nginx的地址，即$remote_addr变量中封装的是nginx的地址，当然是没法获得用户的真实ip的，但是，nginx是可以获得用户的真实ip的，也就是说nginx使用$remote_addr变量时获得的是用户的真实ip，如果我们想要在web端获得用户的真实ip，就必须在nginx这里作一个赋值操作，如下：

### proxy_set_header            X-real-ip $remote_addr;

### 其中这个X-real-ip是一个自定义的变量名，名字可以随意取，这样做完之后，用户的真实ip就被放在X-real-ip这个变量里了，然后，在web端可以这样获取：

### request.getAttribute("X-real-ip")

### 这样就明白了吧。

### **part2：原理介绍**

### 这里我们将nginx里的相关变量解释一下，通常我们会看到有这样一些配置

```
server {         listen       88;         server_name  localhost;         #charset koi8-r;         #access_log  logs/host.access.log  main;         location /{            root   html;            index  index.html index.htm;            proxy_pass                  http://backend;             proxy_redirect              off;            proxy_set_header            Host $host;            proxy_set_header            X-real-ip $remote_addr;            proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;            # proxy_set_header            X-Forwarded-For $http_x_forwarded_for;         }
```

### **1. proxy_set_header    X-real-ip $remote_addr;**

### 这句话之前已经解释过，有了这句就可以在web服务器端获得用户的真实ip

### 但是，实际上要获得用户的真实ip，不是只有这一个方法，下面我们继续看。

### **2.  proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;**

### 我们先看看这里有个X-Forwarded-For变量，这是一个squid开发的，用于识别通过HTTP代理或负载平衡器原始IP一个连接到Web服务器的客户机地址的非rfc标准，如果有做X-Forwarded-For设置的话,每次经过proxy转发都会有记录,格式就是client1, proxy1, proxy2,以逗号隔开各个地址，由于他是非rfc标准，所以默认是没有的，需要强制添加，在默认情况下经过proxy转发的请求，在后端看来远程地址都是proxy端的ip 。也就是说在默认情况下我们使用request.getAttribute("X-Forwarded-For")获取不到用户的ip，如果我们想要通过这个变量获得用户的ip，我们需要自己在nginx添加如下配置：

### **proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;**

### 意思是增加一个$proxy_add_x_forwarded_for到X-Forwarded-For里去，注意是增加，而不是覆盖，当然由于默认的X-Forwarded-For值是空的，所以我们总感觉X-Forwarded-For的值就等于$proxy_add_x_forwarded_for的值，实际上当你搭建两台nginx在不同的ip上，并且都使用了这段配置，那你会发现在web服务器端通过request.getAttribute("X-Forwarded-For")获得的将会是客户端ip和第一台nginx的ip。

### **那么$proxy_add_x_forwarded_for又是什么？**

### $proxy_add_x_forwarded_for变量包含客户端请求头中的"X-Forwarded-For"，与$remote_addr两部分，他们之间用逗号分开。

### 举个例子，有一个web应用，在它之前通过了两个nginx转发，www.linuxidc.com 即用户访问该web通过两台nginx。

### 在第一台nginx中,使用

### proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;

### 现在的$proxy_add_x_forwarded_for变量的"X-Forwarded-For"部分是空的，所以只有$remote_addr，而$remote_addr的值是用户的ip，于是赋值以后，X-Forwarded-For变量的值就是用户的真实的ip地址了。

### 到了第二台nginx，使用

### proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;

### 现在的$proxy_add_x_forwarded_for变量，X-Forwarded-For部分包含的是用户的真实ip，$remote_addr部分的值是上一台nginx的ip地址，于是通过这个赋值以后现在的X-Forwarded-For的值就变成了“用户的真实ip，第一台nginx的ip”，这样就清楚了吧。

### 最后我们看到还有一个$http_x_forwarded_for变量，这个变量就是X-Forwarded-For，由于之前我们说了，默认的这个X-Forwarded-For是为空的，所以当我们直接使用proxy_set_header            X-Forwarded-For $http_x_forwarded_for时会发现，web服务器端使用request.getAttribute("X-Forwarded-For")获得的值是null。如果想要通过request.getAttribute("X-Forwarded-For")获得用户ip，就必须先使用proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;这样就可以获得用户真实ip。

### **Nginx开启Gzip压缩大幅提高页面加载速度**

```
# 开启gzip压缩服务gzip on; # gzip压缩是要申请临时内存空间的，假设前提是压缩后大小是小于等于压缩前的。# 例如，如果原始文件大小为10K，那么它超过了8K，所以分配的内存是8 * 2 = 16K;再例如，# 原始文件大小为18K，很明显16K也是不够的，那么按照 8 * 2 * 2 = 32K的大小申请内存。# 如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。  # 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。 # 例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 # 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。# 如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。gzip_buffers 2 8k; # nginx对于静态文件的处理模块。# 该模块可以读取预先压缩的gz文件，这样可以减少每次请求进行gzip压缩的CPU资源消耗。# 该模块启用后，nginx首先检查是否存在请求静态文件的gz结尾的文件，如果有则直接返回该gz文件内容。# 为了要兼容不支持gzip的浏览器，启用gzip_static模块就必须同时保留原始静态文件和gz文件。# 这样的话，在有大量静态文件的情况下，将会大大增加磁盘空间。我们可以利用nginx的反向代理功能实现只保留gz文件。gzip_static on|off # 启用gzip压缩的最小文件，小于设置值的文件将不会压缩gzip_min_length 1k; # gzip压缩基于的http协议版本，默认就是HTTP 1.1 gzip_http_version 1.1; # gzip 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间，后面会有详细说明gzip_comp_level 2; # 需要进行gzip压缩的Content-Type的Header的类型。建议js、text、css、xml、json都要进行压缩；# 图片就没必要了，gif、jpge文件已经压缩得很好了，就算再压，效果也不好，而且还耗费cpu。# javascript有多种形式。其中的值可以在 mime.types 文件中找到。gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png; # 默认值：off# Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头。# off - 关闭所有的代理结果数据的压缩# expired - 启用压缩，如果header头中包含 "Expires" 头信息# no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息# no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息# private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息# no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息# no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息# auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息# any - 无条件启用压缩gzip_proxied [off|expired|no-cache|no-store|private|no_last_modified|no_etag|auth|any] ... # 是否在http header中添加Vary: Accept-Encoding，建议开启# 和http头有关系，加个vary头，给代理服务器用的，有的浏览器支持压缩，# 有的不支持，所以避免浪费不支持的也压缩，所以根据客户端的HTTP头来判断，是否需要压缩gzip_vary on; # 禁用IE 6 gzipgzip_disable "MSIE [1-6]\.";
```

### 用curl测试Gzip是否成功开启

### curl -I -H "Accept-Encoding: gzip, deflate" "http://www.slyar.com/blog/"

# [nginx静态文件缓存的解决方案](https://www.cnblogs.com/wangzhisdu/p/7771069.html)

```
##cache##    proxy_connect_timeout 500;    #跟后端服务器连接的超时时间_发起握手等候响应超时时间    proxy_read_timeout 600;    #连接成功后_等候后端服务器响应的时间_其实已经进入后端的排队之中等候处理    proxy_send_timeout 500;    #后端服务器数据回传时间_就是在规定时间内后端服务器必须传完所有数据    proxy_buffer_size 128k;    #代理请求缓存区_这个缓存区间会保存用户的头信息以供Nginx进行规则处理_一般只要能保存下头信息即可      proxy_buffers 4 128k;    #同上 告诉Nginx保存单个用的几个Buffer最大用多大空间    proxy_busy_buffers_size 256k;    #如果系统很忙的时候可以申请更大的proxy_buffers 官方推荐*2    proxy_temp_file_write_size 128k;    #proxy缓存临时文件的大小    proxy_temp_path /usr/local/nginx/temp;    #用于指定本地目录来缓冲较大的代理请求    proxy_cache_path /usr/local/nginx/cache levels=1:2 keys_zone=cache_one:200m inactive=15d max_size=200g;    #设置web缓存区名为cache_one,内存缓存空间大小为12000M，自动清除超过15天没有被访问过的缓存数据，硬盘缓存空间大小200g
http{    ......    proxy_cache_path/data/nginx/tmp-test levels=1:2 keys_zone=tmp-test:100m inactive=7d max_size=1000g;}
```

### proxy_cache_path 缓存文件路径

### levels 设置缓存文件目录层次；levels=1:2 表示两级目录

### keys_zone 设置缓存名字和共享内存大小

### inactive 在指定时间内没人访问则被删除

### max_size 最大缓存空间，如果缓存空间满，默认覆盖掉缓存时间最长的资源。

### 当配置好之后，重启nginx，如果不报错，则配置的proxy_cache会生效

### 查看  proxy_cache_path /data/nginx/目录，会发现生成了tmp-test文件夹。