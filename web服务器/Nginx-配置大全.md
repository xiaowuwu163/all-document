# Nginx-配置大全



## 配置`Nginx gzip`压缩实现性能优化

**1.Nginx gzip压缩功能介绍** 
　　Nginx gzip压缩模块提供了压缩文件内容的功能，用户请求的内容在发送出用客户端之前，Nginx服务器会根据一些具体的策略实施压缩，以节约网站出口带宽，同时加快了数据传输效率，提升了用户访问体验。 
**2.Nginx gzip 压缩的优点** 
　1.提升网站用户体验：由于发给用户的内容小了，所以用户访问单位大小的页面就快了，用户体验提升了，网站口碑就好了。 
　2.节约网站带宽成本，由于数据是压缩传输的，因此，此举节省了网站的带宽流量成本，不过压缩会稍微消耗一些CPU资源，这个一般可以忽略。此功能既能让用户体验增强，公司也少花钱。对于几乎所有的Web服务来说，这是一个非常重要的功能，Apache服务也由此功能。 
　3.需要和不需要压缩的对象

1、纯文本内容压缩比很高，因此纯文本内容是最好压缩，例如：html、js、css、xml、shtml等格式的文件 
2、被压缩的纯文本文件必须要大于1KB，由于压缩算法的特殊原因，极小的文件压缩可能反而变大。 
3、图片、视频（流媒体）等文件尽量不要压缩，因为这些文件大多数都是经历压缩的，如果再压缩很坑不会减小或减少很少，或者可能增加。而在压缩时还会消耗大量的CPU、内存资源 
4、参数介绍及配置说明 
　　此压缩功能很类似早起的Apache服务的`mod_defalate`压缩功能，Nginx的gzip压缩功能依赖于`ngx_http_gzip_module`模块，默认已安装。 
**参数说明如下：**

```
 
```

1. `gzip on; #开启gzip压缩功能`
2.  
3. `gzip_min_length 1k;`
4. `#设置允许压缩的页面最小字节数，页面字节数从header头的Content-Length中获取，默认值是0，表示不管页面多大都进行压缩，建议设置成大于1K，如果小于1K可能会越压越大`
5.  
6. `gzip_buffers 4 16k;`
7. `#压缩缓冲区大小，表示申请4个单位为16K的内存作为压缩结果流缓存，默认是申请与原始是数据大小相同的内存空间来存储gzip压缩结果；`
8.  
9. `gzip_http_version 1.1`
10. `#压缩版本（默认1.1 前端为squid2.5时使用1.0）用于设置识别HTTP协议版本，默认是1.1，目前大部分浏览器已经支持GZIP压缩，使用默认即可。`
11.  
12. `gzip_comp_level 2;`
13. `#压缩比率，用来指定GZIP压缩比，1压缩比最小，处理速度最快；9压缩比最大，传输速度快，但处理最慢，也消耗CPU资源`
14.  
15. `gzip_types text/css text/xml application/javascript;`
16. `#用来指定压缩的类型，“text/html”类型总是会被压缩，这个就是HTTP原理部分讲的媒体类型。`
17.  
18. `gzip_vary on;`
19. `#vary hear支持，该选项可以让前端的缓存服务器缓存经过GZIP压缩的页面，例如用缓存经过Nginx压缩的数据。`

配置在`http`标签端

```
 
```

1. `http{`
2. `gzip on;`
3. `gzip_min_length 1k;`
4. `gzip_buffers 4 32k;`
5. `gzip_http_version 1.1;`
6. `gzip_comp_level 9;`
7. `gzip_types text/css text/xml application/javascript;`
8. `gzip_vary on;`
9. `}`

**设置完成之后重启Nginx服务器。** 
并在`360` `火狐|` `谷歌` 等浏览器中安装插件`Firebug`和`YSlow` 进行查看页面压缩率

**例如：没有制作压缩图片** 
![image_1b4t16l2e1hf22ir12sb1ukisd9.png-178.6kB](http://static.zybuluo.com/abcdocker/4gs0btm04zfikm4tynqxwbv3/image_1b4t16l2e1hf22ir12sb1ukisd9.png)
制作后 
![image_1b4t19ftsuh97c5bm2arrndmm.png-181kB](http://static.zybuluo.com/abcdocker/0utgc91nnsntvfsev1yvpk8n/image_1b4t19ftsuh97c5bm2arrndmm.png)

## 配置`Nginx expires`缓存实现性能优化

**1.Nginx expires 功能介绍** 
　　简单地说，`Nginx expires`的功能就是为用户访问的网站内容设定一个国企时间，当用户第一次访问到这些内容时，会把这样内容存储在用户浏览器本地，这样用户第二次及此后继续访问网站，浏览器会检查加载缓存在用户浏览器本地的内容，就不会去服务器下载了。直到缓存的内容过期或被清除为止。 
　　深入理解，expires的功能就是允许通过Nginx 配置文件控制HTTP的“`Expires`”和“`Cache-Contorl`”响应头部内容，告诉客户端刘琦是否缓存和缓存多久以内访问的内容。这个`expires模块`控制Nginx 服务器应答时Expires头内容和Cache-Control头的max-age指定。 
　　这些HTTP头向客户端表名了内容的有效性和持久性。如果客户端本地有内容缓存，则内容就可以从缓存（除非已经过期）而不是从服务器读取，然后客户端会检查缓存中的副本。

**2.Nginx expires作用介绍** 
　　在网站的开发和运营中，对于`图片` `视频` `css` `js`等网站元素的更改机会较少，特别是图片，这时可以将图片设置在客户端浏览器本地缓存`365`天或`3650`天，而降css、js、html等代码缓存`10~30`天，这样用户第一次打开页面后，会在本地的浏览器按照过期日期缓存响应的内容，下次用户再打开类似页面，重复的元素就无需下载了，从而加快了用户访问速度，由于用户的访问请求和数据减少了，因此节省了服务器端大量的带宽。此功能和`apache`的`expire`相似。

**3.Nginx expires 功能优点** 
1.Expires可以降低网站的带宽，节约成本。 
2.加快用户访问网站的速度，提升了用户访问体验。 
3.服务器访问量降低了，服务器压力就减轻了，服务器成本也会降低，甚至可以解决人力成本。 
对于几乎所有Web来说，这是非常重要的功能之一，Apache服务也由此功能。

**4. Nginx expires 配置详解** 
1）根据文件扩展名进行判断，添加expires功能范例。

```
 
```

1. `location ~.*\.(gif|jpg|jpeg|png|bmp|swf)$`
2. `{`
3. `expires 3650d;`
4. `}`

该范例的意思是当前用户访问网站URL结尾的文件扩展名为上述指定的各种类型的图片时，设置缓存`3650`天，即10年。 
**提示：配置可以放在server标签，也可以放在http标签下配置** 
![image_1b4t1es7v3m31jk47pl7sn1pbn13.png-70.4kB](http://static.zybuluo.com/abcdocker/pr349chlihjw8fymqcmv8lia/image_1b4t1es7v3m31jk47pl7sn1pbn13.png)
例如：

```
 
```

1. `[root@web02 /]# curl -I www.jd.com`
2. `HTTP/1.1 200 OK`
3. `Server: jdws`
4. `Date: Mon, 25 Jul 2016 15:15:47 GMT`
5. `Content-Type: text/html; charset=gbk`
6. `Content-Length: 197220`
7. `Connection: keep-alive`
8. `Vary: Accept-Encoding`
9. `Expires: Mon, 25 Jul 2016 15:15:38 GMT #告诉用户什么时候过期`
10. `Cache-Control: max-age=20`
11. `ser: 6.158`
12. `Via: BJ-M-YZ-NX-74(HIT), http/1.1 BJ-UNI-1-JCS-117 ( [cRs f ])`
13. `Age: 16`

2）根据URI中的路径（目录）进行判断，添加expires功能范例。

```
 
```

1. `location ~^/(images|javascript|js|css|flash|media|static)/ {`
2. `expires 360d;`
3. `}`

意思是当用户访问URI中包含上述路径（例：`images` `js` `css` 这些在服务端是`程序目录`）时，把访问的内容设置缓存360天，即1年。如果要想缓存30天，设置30d即可。

```
 
```

1. `HTTP/1.1 200 OK`
2. `Server: JDWS`
3. `Date: Mon, 25 Jul 2016 16:00:32 GMT`
4. `Content-Type: text/html; charset=gbk`
5. `Vary: Accept-Encoding`
6. `Expires: Mon, 25 Jul 2016 16:00:48 GMT #<==缓存的过期时间`
7. `Cache-Control: max-age=20 #<==缓存的总时间按秒，单位`
8. `ser: 130.29`
9. `Via: BJ-Y-NX-104(HIT), http/1.1 HK-1-JCS-70 ( [cRs f ])`
10. `Age: 14`
11. `Content-Length: 197220`

**5.Nginx expires功能缺点及解决方法** 
　　当网站被缓存的页面或数据更新了，此时用户看到的可能还是旧的已经缓存的内容，这样会影响用户体验。 
　　对经常需要变动的图片等文件，可以缩短对象缓存时间，例如：谷歌和百度的首页图片经常根据不同的日期换成一些节日的图，所以这里可以将图片设置为缓存期为1天。 
　　当网站改版或更新内容时，可以在服务器将缓存的对象改名（网站代码程序）。

1.对于网站的图片、软件，一般不会被用户直接修改，用户层面上的修改图片，实际上是重新传到服务器，虽然内容一样但是是一个新的图片名了 
2.网站改版升级会修改JS、CSS元素，若改版的时候对这些元素该了名，会使得前端的CDN以及用户需要重新缓存内容。

**6.企业网站缓存日期曾经的案例参考** 
　　若企业的业务和访问量不同，那么网站的缓存期时间设置也是不同的，比如： 
a.51CTP：1周 
b.sina：15天 
c.京东：25年 
d.淘宝：10年

**7.企业网站有可能不希望被缓存的内容** 
1.广告图片，用于广告服务，都缓存了就不好控制展示了。 
2.网站流量统计工具（js代码）都缓存了流量统计就不准了 
3.更新很频繁的文件（google的logo），如果按天，缓存效果还是显著的。

## Nginx`日志`相关优化与安装

**1.编写脚本脚本实现Nginx access日志轮询** 
　　Nginx目前没有类似Apache的通过`cronlog`或者`rotatelog`对日志分割处理的能力，但是，运维人员可以通过利用脚本开发、Nginx的信号控制功能或reload重新加载，来实现日志自动切割，轮询。

（1）配置日志切割脚本

```
 
```

1. `[root@web02 /]# mkdir /server/scripts/ -p`
2. `[root@web02 /]# cd /server/scripts/`
3. `[root@web02 scripts]# cat cut_nginx_log.sh`
4. `cd /application/nginx/logs && \`
5. `/bin/mv www_access.log www_access_$(data +%F -d -1dy).log #将日志按日志改成前一天的名称`
6. `/application/nginx/sbin/nginx -s reload #重新加载nginx使得重新生成访问日志文件`

**提示：**实际上脚本的功能很简单，就是改名日志，然后加载nginx，重新生成文件记录日志。

（2）将这段脚本保存后加入到定时任务，设置每天凌晨0点进行切割日志

```
 
```

1. `[root@web02 scripts]# crontab -e`
2. `###cut nginx access log`
3. `00 00 * * * /bin/sh /server/scripts/cut_nginx.log.sh >/dev/null 2>&1`

解释：每天0点执行`cut_nginx_log.sh`脚本，将脚本的输出重定向到空。

**2.不记录不需要的访问日志** 
　　对于负载均衡器健康检查节点或某些特定文件(比如图片、`js`、`css`)的日志，一般不需要记录下来，因为在统计PV时是按照页面计算的。而且日志写入频繁会大量消耗磁盘I/O，降低服务的性能。 
具体配置如下：

```
 
```

1. `location ~ .*\.(js|jpg|JPG|jpeg|JPEG|css|bmp|gif|GIF)?$ {`
2. `access_log off;`
3. `}`

这里用`location`标签匹配不记录日志的元素扩展名，然后关掉了日志。

**3.访问日志的权限设置** 
加入日志目录为`/app/logs` 则授权方法为：

```
 
```

1. `chown -R root.root /app/logs/`
2. `chmod -R 700 /app/logs`

不需要在日志目录上给nginx用户读或写的许可。

## Nginx站点目录及文件URL访问控制

**1.根据扩展名限制程序和文件访问** 
　　Web2.0时代，绝大多数网站都是以用户为中心，例如：BBS、blog、sns产品，这几个产品共同特点就是不但允许用户发布内容到服务器，还允许用户发图片甚至附件到服务器，由于为用户打开了上传的功能，因为给服务器带来了很大的安全风险。 
　　下面将利用Nginx配置禁止访问上传资源目录下的PHP、shell、perl、Python程序文件，这样用户即使上传了木马文件也没法去执行，从而加强了网站的安全。 
配置Nginx，限制禁止解析指定目录下的制定程序。

```
 
```

1. `location ~ ^/images/.*\.(php|php5|.sh|.pl|.py)$`
2. `{`
3. `deny all;`
4. `}`
5. `location ~ ^/static/.*\.(php|php5|.sh|.pl|.py)$`
6. `{`
7. `deny all;`
8. `}`
9. `location ~* ^/data/(attachment|avatar)/.*\.(php|php5)$`
10. `{`
11. `deny all;`
12. `}`

Nginx下配置禁止访问*.txt文件

```
 
```

1. `location ~*\.(txt|doc)${`
2. `if (-f $request_filename) {`
3. `root /data/www/www;`
4. `#rewrite ....可以重定向某个URL`
5. `break;`
6. `}`
7. `}`
8. `location ~*\.(txt|doc)${`
9. `root /data/www/www;`
10. `deny all;`
11. `}`

对上述限制需要卸载`php`匹配的前面

```
 
```

1. `　　location ~.*\.(php|php5)?$`
2. `{`
3. `　fastcgi_pass 127.0.0.1:9000`
4. `　fastcgi_index index.php`
5. `　include fcgi.conf;`
6. `}`

对目录访问进行设置

**单目录**

```
 
```

1. `location ~ ^/(static)/ {`
2. `deny all;`
3. `}`
4. `location ~ ^/static {`
5. `deny all;`
6. `}`

**多目录**

```
 
```

1. `location ~ ^/(static)/ {`
2. `deny all;`
3. `}`

范例：禁止访问目录并返回指定的`http`状态码

```
 
```

1. `location /admin/ { return 404; }`
2. `location /templates/ { return 403; }`

**限制网站来源IP访问** 
案例环境：`phpmyadmin` 数据库的Web客户端，内部开发人员使用 
禁止某目录让外界访问，但允许某IP访问该目录，切支持PHP解析

```
 
```

1. `location ~ ^/docker/ {`
2. `allow 202.111.12.211;`
3. `deny all;`
4. `}`

**企业问题案例：** Nginx做反向代理的时候可以限制客户端IP吗？ 
解答：可以，具体方法如下。

```
 
```

1. `方法1：使用if来控制。`
2. `if ( $remote_addr = 10.0.0.7 ) {`
3. `return 403;`
4. `}`
5. `if ( $remote_addr = 218.247.17.130 ) {`
6. `set $allow_access_root 'true';`
7. `}`

## 配置Nginx禁止非法域名解析访问企业网站

`Nginx`如何预防用户IP访问网站（恶意域名解析，相当于是直接IP访问企业网站） 
让使用IP访问的网站用户，或者而已解析域名的用户，收到`501`错误

```
 
```

1. `server {`
2. `listen 80 default_server;`
3. `server_name _;`
4. `return 501;`
5. `}`

通过301跳转到主页。

```
 
```

1. `server {`
2. `listen 80 default_server;`
3. `server_name _;`
4. `rewrite ^(.*) http://www.abcdocker.com/$1 permanent;`
5. `}`

**要放在第一个server**

```
 
```

1. `if ($host !~ ^www/.abcdocker/.com$){`
2. `rewrite ^(.*) http://www.abcdocker.com$1 permanent;`
3. `}`

如果header信息和host主机名字字段非`www.abcdocker.com`，就301跳转到`www.baidu.cn`

## Nginx图片及目录`防盗链`解决方案

**1.什么是资源盗链** 
　　简单的说，就是某些不发的网站未经许可，通过在其自身网站程序里非法调用其他网站的资源吗，然后在自己的网站上显示。达到补充自身网站的效果，这一举动不但浪费了调用网站的流量，还造成服务器的压力，甚至宕机。 
![image_1b4t2o155shp1gh61ru01ef81mgj1g.png-120.7kB](http://static.zybuluo.com/abcdocker/jx8xtmsnixw1bnxc9xruc77h/image_1b4t2o155shp1gh61ru01ef81mgj1g.png)
**2.网站资源被盗链带来的问题** 
　　若网站图片及相关资源被盗链，最直接的影响就是网络带宽占用加大了，带宽费用多了，网站流量也可能忽高忽低，`nagios/zabbix`等报警服务频繁报警。 
![image_1b4t2oua01igeck1c85v0bn1m1t.png-78.9kB](http://static.zybuluo.com/abcdocker/npwq1wdw2z5jaslec89ftfq6/image_1b4t2oua01igeck1c85v0bn1m1t.png)
　　最严重的情况就是网站的资源被非法使用，导致网站带宽成本加大和服务器压力加大，有可能会导致数万元的损失，且网站的正常用户访问也会受到影响。 
**3.网站资源被盗链严重问题企业真实案例** 
公司的CDN源站的流量没有变动，但是CDN加速那边的流量无故超了好几个GB，不知道怎么如理。 
该故障的影响： 
由于是购买的CDN网站加速服务，因此虽然流量多了几个GB，但是业务未受影响。只是，这么大的异常流量，持续下去可直接导致公司无故损失数万元。 
**解决方案：** 
第一，对IDC及CDN带宽做监控报警。 
第二，作为高级运维或者运维经理，每天上班的重要任务，就是经常查看网站流量图，关注流量变化，关注异常流量 
第三，对访问日志做分析，对于异常流量迅速定位，并且和公司市场推广等有比较好的默契沟通 
相关博客：[轻松应对IDC机房带宽突然暴涨问题](https://www.abcdocker.com/wp-content/themes/begin/inc/go.php?url=http://oldboy.blog.51cto.com/2561410/909696)

**4.常见防盗链解决方案的基本原理** 
(1)根据http referer 实现防盗链 
　　在HTTP协议中，有一个表头字段叫referer，使用URL格式来表示哪里的链接用了当前网页的资源。通过referer可以检测目标访问的来源网页，如果是资源文件，可以跟踪到显示它的网页地址，一旦检测出来不是本站，马上进行阻止或返回指定的页面。 
　　HTTP Referer是header的一部分，当浏览器向Web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的，服务器籍此可以获得一些信息用于处理，Apache、Nginx、Lighttpd三者都支持根据http referer实现防盗链referer是目前网站图片、附件、html最常用的盗链手段。

```
 
```

1. `log_format main '$remote_addr - $remote_user [$time_local] "$request" '`
2. `'$status $body_bytes_sent "$http_referer" '`
3. `'"$http_user_agent" "$http_x_forwarded_for"';`
4.  
5. `#--> $http_referer`

![image_1b4t2spm61ti81s9j80p1ooq1eg02a.png-111.9kB](http://static.zybuluo.com/abcdocker/r2i1c1hsrct194qfkeeh9xx4/image_1b4t2spm61ti81s9j80p1ooq1eg02a.png)

(2)根据cookie防盗链 
　　对于一些特殊的业务数据，例如流媒体应用以及通过Active X显示的内容（例如，Flash、Windows Media视频、流媒体的RTSP协议等）因为他们不向服务器提供Referer Header，所以若也采用上述的Referer的防盗链手段就达不到想要的结果。 
　　对于视频这种占用流量较大的业务根据实现防盗链是比较困难的，此时可以采用Cookie技术，来解决对Flash、Windows Media视频等防盗链问题。 
　　例如：Active X插件不传递Referer，但会传递Cookie。可以在显示Active X的页面的 标签内嵌入一段代码，可以用JavaScript 代码来设置一段`Cookie；Cache=av；` 
![image_1b4t2u09vq119ai14f111av1dr22n.png-20.8kB](http://static.zybuluo.com/abcdocker/j6kzfj89st4ln9uwq0c27gm1/image_1b4t2u09vq119ai14f111av1dr22n.png)
然后就可以通过各种手段来判断这个`Cookie`的存在以及验证其值的操作了。

(3)通过加密变换访问路径实现防盗链 
　　此方法比较适合视频以及下载类业务的网站。例如：`Lighttpd` 有类似的插件`mod_secdownload`来实现此功能，现在服务器配置此模块，设置一个固定用于加密的字符串，比如abcdocker，然后设置一个url前缀，比如/abc/，再设置一个过期时间，比如1小时，然后写一段PHP代码，例如加密字符串和系统时间等通过md5算法生产一个加密字符串，最终获取到的文件的URL连接种会带由一个时间戳和一个加密字符的md5数值，在访问时系统会对这两个数据进行验证。如果时间不在预期的时间段内（如1小时）则失效；如果时间戳符合条件，但是加密的字符串不符合条件也失效，从而达到防盗链的效果。

PHP代码示例如下： 
![image_1b4t2v8j117ev4vh18nsf1h1dij34.png-60.3kB](http://static.zybuluo.com/abcdocker/7449dy5dyiju7zsrmf5l0g7p/image_1b4t2v8j117ev4vh18nsf1h1dij34.png)
Nginx实现下载防盗链模块 
[http://nginx.org/en/docs/http/ngx_http_secure_link_module.html](https://www.abcdocker.com/wp-content/themes/begin/inc/go.php?url=http://nginx.org/en/docs/http/ngx_http_secure_link_module.html) 
(4)在产品设计上解决盗链方案 
　　产品设计时，处理盗链问题可将计就计，为网络上传的图片添加水印。 
![image_1b4t2vsfjspa2os13dm1sdf84r3h.png-18.2kB](http://static.zybuluo.com/abcdocker/tdw85yx7io4tj4dcxp3c615i/image_1b4t2vsfjspa2os13dm1sdf84r3h.png)

　　图片添加版权水印，很多网站一般直接转载图片是为了快捷，但是对于有水印的图片，很多站长是不愿意进行转载的。 
　　 
(4)Nginx防盗链演示 
1.利用referer并且针对扩展名rewrite重定向。

```
 
```

1. `#Preventing hot linking of images and other file types`
2. `location ~* ^.+\.(jpg|png|swf|flv|rar|zip)$ {`
3. `valid_referers none blocked *.abcdocker.org abcdocker.org;`
4. `if ($invalid_referer) {`
5. `rewrite ^/ http://bbs.abcdocker.org/img/nolink.gif;`
6. `}`
7. `root html/www;`
8. `}`

**提示：**要根据主机公司实际业务（是否有外联的合作），进行域名设置。 
针对防盗链中设置进行解释 
`jpg` `png` `swf` `flv` `rar` `zip` 表示对`jpg、gif`等zip为后缀的文件实行防盗链处理 
`*.abcdocker.org abcdocker.org`表示这个请求可以正常访问上面指定的文件资源 
`if{}`中内容的意思是：如果地址不是上面指定的地址就跳转到通过rewrite指定的地址，也可以直接通过retum返回`403`错误 
`return 403`为定义的http返回状态码 
`rewrite ^/ http://bbs.abcdocker.org/img/nolink.gif;`表示显示一张防盗链图片 
`access_log off;`表示不记录访问日志，减轻压力 
`expires 3d`指的是所有文件3天的浏览器缓存

**实战模拟演示** 
1）假定blog.abcdocker.com是非法盗链的网站域名，先写一个html程序。

```
 
```

1. `<html>`
2. `<head>`
3. `<title>`
4. `123456789`
5. `</title>`
6. `</head>`
7. `<body bgcolor=green>`
8. `博客<br>我的博客<a href="http://oldboy.blog.etiantian.org" target=_blank“>博客地址</a>`
9. `<img src="http://www.abcdocker.com/stu.jpg">`
10. `</body>`
11. `</html>`

这个非法链接的网站给他用户提供的访问地址是 
`http://blog.abcdocker.com/123.html` 网站里回家再`www.abcdocker.com`网站图片的`stu.jpg` 
Nginx的日志格式为`www.abcdocker.com`，其内容如下

```
 
```

1. `log_format main '$remote_addr - $remote_user [$time_local] "$request" '`
2. `'$status $body_bytes_sent "$http_referer" '`
3. `'"$http_user_agent" "$http_x_forwarded_for"';`

盗链的图片blog.abcdocker.com访问我们站点时，记录的日志如下：

```
 
```

1. `10.0.0.1 - - [30/May/2016:11:13:38 +0800] "GET /stu.jpg HTTP/1.1" 200 68080 "http://blog.abcdocker.com/123.html "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:47.0) Gecko/20100101 Firefox/47.0"`

可在www.abcdocker.com网站下设置防盗链，Nginx的方法如下：

```
 
```

1. `#Preventing hot linking of images and other file types`
2. `location ~* ^.+\.(jpg|png|swf|flv|rar|zip)$ {`
3. `valid_referers none blocked *.abcdocker.org abcdocker.org;`
4. `if ($invalid_referer) {`
5. `rewrite ^/ http://bbs.abcdocker.org/img/nolink.gif;`
6. `}`
7. `root html/www;`
8. `}`

**提示：** referers不是*.abcdocker.org的话，就给他一个跳转

**5.为什么要配置错误页面优化显示？** 
　　在网站的运行过程中，可能由于页面不存在或者系统过载等原因，导致网站无法正常响应用于的请求，此时Web服务默认会返回系统默认的错误码，或者很不友好的页面。影响用户体验 
![image_1b4t36ph5k9t68f1t6b1vkb1qrv3u.png-26.6kB](http://static.zybuluo.com/abcdocker/ouphjg4hquk53r957hcru9cf/image_1b4t36ph5k9t68f1t6b1vkb1qrv3u.png)
对错误代码`404`实行本地页面优雅显示

```
 
```

1. `server {`
2. `listen 80;`
3. `server_name www.etiantian.org;`
4. `location / {`
5. `root html/www;`
6. `index index.php index.html index.htm;`
7. `error_page 404 /404.html`
8. `#当页面出现404错误时，会跳转404.html页面显示给用户`
9. `}`

提示： 
此路径相对于`root html/www;`的

```
 
```

1. `error_page 404 /404.html;`
2. `error_page 403 /403.html;`

另一种 重定向到一个地址

```
 
```

1. `error_page 404 http://www.abcdocker.com;`
2. `#error_page 404 /404.html;`
3. `error_page 404 http://www.abcdocker.com;`

可以写多行。

```
 
```

1. `error_page 404 /404.html;`
2. `error_page 500 502 503 504 /50x.html;`

阿里门户网站天猫的Nginx优雅显示配置案例如下：

```
 
```

1. `error_page 500 501 502 503 504 http://err.tmall.com/error2.html;`
2. `error_page 400 403 404 405 408 410 411 412 413 414 415 http://err.tmall.com/error1.html;`

## Nginx站点目录文件及目录权限优化

　　为了保证网站不遭受木马入侵，所有站点的用户和组都应该为`root`，所有目录权限是`755`；所有文件权限是`644`.设置如下：

```
 
```

1. `-rw-r--r-- 1 root root 20 May 26 12:04 test_info.php`
2. `drw-r--r-- 8 root root 4096 May 29 16:41 uploads`

![image_1b4t3apvf1187ap817u81ahs1b4b.png-143.7kB](http://static.zybuluo.com/abcdocker/nl7179rocxu8cuodq8cvkwj9/image_1b4t3apvf1187ap817u81ahs1b4b.png)

　　可以设置上传只可以`put`不可以`get`，或者使用`location`不允许访问共享服务器的内容，图片服务器禁止访问php|py|sh。这样就算黑客将php木马上传上来也无法进行执行

**集群架构中不同角色的权限具体思路说明** 
![image_1b4t3dafg1bb5186315bt1n2t13c44o.png-57.6kB](http://static.zybuluo.com/abcdocker/w7l0s0z4sn7d6gpjtk84pbfn/image_1b4t3dafg1bb5186315bt1n2t13c44o.png)

## Nginx防`爬虫`优化

**1.robots.txt机器人协议介绍** 
　　`Robots`协议（也成为爬虫协议、机器人协议等）的全称是`网络爬虫排除标准`(Robots Exclusin Protocol)网站通过Robots协议告诉引擎那个页面可以抓取，那些页面不能抓取。

**2.机器人协议八卦** 
![image_1b4t3f0ct13b32ml33c1ob31rfk55.png-50.4kB](http://static.zybuluo.com/abcdocker/47uywtn8hei9cycwd34y8zq4/image_1b4t3f0ct13b32ml33c1ob31rfk55.png)
`2008年9月8日`，淘宝网宣布封杀百度爬虫，百度热痛遵守爬虫协议，因为一旦破坏协议，用户的隐私和利益就无法得到保障。 
![image_1b4t3flbb53fhu81kvtm3keko5i.png-44.1kB](http://static.zybuluo.com/abcdocker/qft17uv4hqp9bo0dyeg0kdsn/image_1b4t3flbb53fhu81kvtm3keko5i.png)
2012年8月。360综合搜索被指违反robots协议 
![image_1b4t3fv0rov91ilj9j616sotf05v.png-84.6kB](http://static.zybuluo.com/abcdocker/6od2u2h2lul7ti3jwb1vuhns/image_1b4t3fv0rov91ilj9j616sotf05v.png)

**3.Nginx防爬虫优化** 
　　我们可以根据客户端的`user-agents`信息，轻松地阻止爬虫取我们的网站防爬虫 
范例：阻止下载协议代理

```
 
```

1. `## Block download agents ##`
2. `if ($http_user_agent ~* LWP::Simple|BBBike|wget) {`
3. `return 403;`
4. `}`

**说明：**如果用户匹配了if后面的客户端(例如wget)就返回403

**范例：添加内容防止N多爬虫代理访问网站**

```
 
```

1. `if ($http_user_agent ~* "qihoobot|Baiduspider|Googlebot|Googlebot-Mobile|Googlebot-Image|Mediapartners-Google|Adsbot-Google|Yahoo! Slurp China|YoudaoBot|Sosospider|Sogou spider|Sogou web spider|MSNBot") {`
2. `return 403;`
3. `}`

测试禁止不同的浏览器软件访问

```
 
```

1. `if ($http_user_agent ~* "Firefox|MSIE")`
2. `{`
3. `rewrite ^(.*) http://blog.etiantian.org/$1 permanent;`
4. `}`
5.  
6. `如果浏览器为Firefox或者IE就会跳转到http:blog.etiantian.org`

**提示：** 
这里主要用了`$remote_addr`这个函数在记录。 
查看更多函数

```
 
```

1. `[root@web02 conf]# cat fastcgi_params`
2. `fastcgi_param QUERY_STRING $query_string;`
3. `fastcgi_param REQUEST_METHOD $request_method;`
4. `fastcgi_param CONTENT_TYPE $content_type;`
5. `fastcgi_param CONTENT_LENGTH $content_length;`
6. `fastcgi_param SCRIPT_NAME $fastcgi_script_name;`
7. `fastcgi_param REQUEST_URI $request_uri;`
8. `fastcgi_param DOCUMENT_URI $document_uri;`
9. `fastcgi_param DOCUMENT_ROOT $document_root;`
10. `fastcgi_param SERVER_PROTOCOL $server_protocol;`
11. `fastcgi_param HTTPS $https if_not_empty;`
12. `fastcgi_param GATEWAY_INTERFACE CGI/1.1;`
13. `fastcgi_param SERVER_SOFTWARE nginx/$nginx_version;`
14. `fastcgi_param REMOTE_ADDR $remote_addr;`
15. `fastcgi_param REMOTE_PORT $remote_port;`
16. `fastcgi_param SERVER_ADDR $server_addr;`
17. `fastcgi_param SERVER_PORT $server_port;`
18. `fastcgi_param SERVER_NAME $server_name;`
19. `# PHP only, required if PHP was built with --enable-force-cgi-redirect`
20. `fastcgi_param REDIRECT_STATUS 200;`

## 利用Nginx限制HTTP的请求方法

　　HTTP最常用的方法为`GET/POST`，我们可以通过Nginx限制http请求的方法来达到提升服务器安全的目的， 
　例如，让HTTP只能使用GET、HEAD和POST方法配置如下：

```
 
```

1. `#Only allow these request methods`
2. `if ($request_method !~ ^(GET|HEAD|POST)$ ) {`
3. `return 501;`
4. `}`
5. `#Do not accept DELETE, SEARCH and other methods`

　　设置对应的用户相关权限，这样一旦程序有漏洞，木马就有可能被上传到服务器挂载的对应存储服务器的目录里，虽然我们也做了禁止PHP、SH、PL、PY等扩展名的解析限制，但是还是会遗漏一些我们想不到的可执行文件。对于这种情况，该怎么办捏？事实上，还可以通过限制上传服务器的web服务（可以具体到文件）使用GET方法，来达到防治用户通过上传服务器访问存储内容，让访问存储渠道只能从静态或图片服务器入口进入。例如，在上传服务器上限制HTTP的GET方法的配置如下：

```
 
```

1. `## Only allow GET request methods ##`
2. `if ($request_method ~* ^(GET)$ ) {`
3. `return 501;`
4. `}`

提示：还可以加一层`location`更具体的限制文件名 
![image_1b4t3q9omg2k1o4hv3e1i8b1jb36c.png-51.5kB](http://static.zybuluo.com/abcdocker/teriyodegw86h8irli5u01m0/image_1b4t3q9omg2k1o4hv3e1i8b1jb36c.png)

## 使用`CDN`做网站内容加速

**1.什么是CDN？** 
　　CDN的全称是`Content Delivery Network` 中文意思是内容分发网络。 
　　通过现有的Internet中增加一层新的网络架构，将网站的内容发布到最接近用户的cache服务器内，通过智能DNS负载均衡技术，判断用户的来源，让用户就近使用和服务器相同线路的带宽访问cache服务器取得所需的内容。 
　　例如：天津网通用户访问天津网通Cache服务器上的内容，北京电信访问北京电信Cache服务器上的内容。这样可以减少数据在网络上传输的事件，提高访问速度。 
　　CDN是一套全国或全球的分布式缓存集群，其实质是通过智能DNS判断用户的来源地域以及上网线路，为用户选择一个最接近用户地狱以及和用户上网线路相同的服务器节点，因为地狱近，切线路相同，所以，可以大幅度提升浏览网站的体验。

**CDN的价值** 
1、为架设网站的企业省钱。 
2、提升企业网站的用户访问体验（相同线路，相同地域，内存访问）。 
3、可以阻挡大部分流量攻击，例如：DDOS攻击 
更多CDN介绍请查看本网相关文章

## Nginx程序架构优化

**1.为网站程序解耦** 
　　解耦是开发人员中流行的一个名词，简单地说就是把一堆程序嗲吗按照业务用途分开，然后提供服务，例如：注册登录、上传、下载、订单支付等都应该是独立的程序服务，只不过在客户端看来是一个整体而已。如果中小公司做不到上述细致的解耦，最起码让下面的几个程序模块独立。 
**1.网站页面服务 2.图片附件及下载服务。 3.上传图片服务** 
　　上述三者的功能尽量分离。分离的最佳方式是分别使用独立的服务器（需要改动程序）如果程序实在不好改，次选方案是在前端负载均衡器`haproxy/nginx`上，根据URI设置

**使用普通用户启动Nginx（监牢模式）**

1.为什么要让Nginx服务使用普通用户 
　　默认情况下，Nginx的`Master`进程使用的是`root`用户，`Worker`进程使用的是`Nginx`指定的普通用户，使用root用户跑Nginx的Master进程由两个最大的问题： 
▲ 管理权限必须是root，这就使得最小化分配权限原则遇到难题 
▲使用root跑Nginx服务，一旦网站出现漏洞，用户就很容易获得服务器root权限

```
 
```

1. `[root@web02 ~]# ps -ef|grep nginx`
2. `root 2155 1 0 03:43 ? 00:00:00 nginx: master process /application/nginx/sbin/nginx`
3. `www 2156 2155 0 03:43 ? 00:00:01 nginx: worker process`
4. `www 3047 2155 0 06:17 ? 00:00:00 nginx: worker process`
5. `www 3051 2155 0 06:17 ? 00:00:00 nginx: worker process`
6. `www 3435 2155 0 11:13 ? 00:00:00 nginx: worker process`

2.给Nginx服务降权解决方案

(1) 给Nginx服务降权，用inca用户跑Nginx服务，给开发及运维设置普通账号，只要和inca同组即可管理Nginx，该方案解决了Nginx管理问题，防止root分配权限过大。 
(2) 开发人员使用普通账户即可管理Nginx服务以及站点下的程序和日志 
(3) 采取项目负责制，即谁负载项目维护处了问题就是谁负责。

3.实时Nginx降权方案

```
 
```

1.  
2.  
3. `[root@web02 ~]# useradd inca`
4. `[root@web02 ~]# su - inca`
5. `[inca@web02 ~]$ pwd`
6. `/home/inca`
7. `[inca@web02 ~]$ mkdir conf logs www`
8. `[inca@web02 ~]$ cp /application/nginx/conf/mime.types ~/conf/`
9. `[inca@web02 ~]$ echo inca >www/index.html`
10. `[inca@web01 ~]$ cat conf/nginx.conf`
11. `worker_processes 4;`
12. `worker_cpu_affinity 0001 0010 0100 1000;`
13. `worker_rlimit_nofile 65535;`
14. `error_log /home/inca/logs/error.log;`
15. `user inca inca;`
16. `pid /home/inca/logs/nginx.pid;`
17. `events {`
18. `use epoll;`
19. `worker_connections 10240;`
20. `}`
21. `http {`
22. `include mime.types;`
23. `default_type application/octet-stream;`
24. `sendfile on;`
25. `keepalive_timeout 65;`
26.  
27. `log_format main '$remote_addr - $remote_user [$time_local] "$request" '`
28. `'$status $body_bytes_sent "$http_referer" '`
29. `'"$http_user_agent" "$http_x_forwarded_for"';`
30.  
31. `#web.fei fa daolian..............`
32. `server {`
33. `listen 8080;`
34. `server_name www.etiantian.org;`
35. `root /home/inca/www;`
36. `location / {`
37. `index index.php index.html index.htm;`
38. `}`
39. `access_log /home/inca/logs/web_blog_access.log main;`
40. `}`
41. `}`

提示，需要关闭root权限的nginx，否则会报错

```
 
```

1. `[root@web02 ~]# /application/nginx/sbin/nginx -s stop`
2. `[root@web02 ~]# lsof -i:80`

切换用户，启动`Nginx`

```
 
```

1. `[root@web02 ~]# su - inca`
2. `[inca@web02 ~]$ /application/nginx/sbin/nginx -c /home/inca/conf/nginx.conf &>/dev/null &`
3. `[1] 3926`
4. `[inca@web02 ~]$ lsof -i:80`
5. `[1]+ Exit 1 /application/nginx/sbin/nginx -c /home/inca/conf/nginx.conf &>/dev/null`

**本解决方案的优点如下：** 
1.给Nginx服务降权，让网站更安全 
2.按用户设置站点权限，使站点更安全（无需虚拟化隔离） 
3.开发不需要用root即可完整管理服务及站点 
4.可实现对责任划分，网络问题属于运维的责任，打开不就是开发责任或共同承担

**控制Nginx并发连接数** 
　　`nginx_http_limit_conn_module`这个模块用于限制每个定义的key值的连接数，特别是单IP的连接数。 
　　不是所有的连接数都会被计数。一个符合要求的连接是整个请求头已经被读取的连接。 
控制Nginx并发连接数量参数的说明如下：

```
 
```

1. `limit_conn_zone参数：`
2. `语法：limit_conn_zone key zone=name:size;`
3. `上下文:http`
4. `用于设置共享内存区域，key可以是字符串，nginx自有变量或前两个组合，如$binary_remote_addr、$server_name。name为内存区域的名称，size为内存区域的大小。`
5. `limit_conn参数：`
6. `语法：limit_conn zone number;`
7. `上下文：http、server、location`

配置文件如下：

```
 
```

1. `[root@oldboy ~]# cat /application/nginx/conf/nginx.conf`
2. `worker_processes 1;`
3. `events {`
4. `worker_connections 1024;`
5. `}`
6. `http {`
7. `include mime.types;`
8. `default_type application/octet-stream;`
9. `sendfile on;`
10. `keepalive_timeout 65;`
11.  
12. `limit_conn_zone $binary_remote_addr zone=addr:10m;`
13.  
14. `server {`
15. `listen 80;`
16. `server_name www.etiantian.org;`
17. `location / {`
18. `root html;`
19. `index index.html index.htm;`
20. `limit_conn addr 1; #<==限制单IP的并发连接为1`
21. `}`
22. `}`
23. `}`

还可以设置某个目录单IP并发连接数

```
 
```

1. `location /download/ {`
2. `limit_conn addr 1;`
3. `}`

在客户端使用Apache的ab测试工具进行测试 
执行`ab -c 1 -n 10 http://10.0.0.3`进行测试 
注意：-c并发数、-n请求总数，`10.0.0.3nginx`的IP地址

![image_1b4t44gou1vvj1t4tjv01lcb8116p.png-163.5kB](http://static.zybuluo.com/abcdocker/2jplm6tjyql8uw4y995lpwn4/image_1b4t44gou1vvj1t4tjv01lcb8116p.png)

## 控制客户端请求Nginx的速率

　　`ngx_http_limit_req_module`模块用于限制每个IP访问定义key的请求速率。 
**limit_req_zone参数说明如下：** 
语法：`limit_req_zonekey zone=name:size rate=rate;` 
用于设置共享内存区域，key可以是字符串、Nginx自有变量或前两个组合，如`$binary_remote_addr` `name`为内存区域的名称，`size`为内存区域的大小，`rate`为速率，单位为`r/s` 每秒一个请求。

```
 
```

1. `[root@oldboy ~]# cat /application/nginx/conf/nginx.conf`
2. `worker_processes 1;`
3. `events {`
4. `worker_connections 1024;`
5. `}`
6. `http {`
7. `include mime.types;`
8. `default_type application/octet-stream;`
9. `sendfile on;`
10. `keepalive_timeout 65;`
11. `limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;`
12. `#<==以请求的客户端IP作为key值，内存区域命名为one，分配10m内存空间，访问速率限制为1秒1次请求(request)`
13. `server {`
14. `listen 80;`
15. `server_name www.etiantian.org;`
16. `location / {`
17. `root html;`
18. `index index.html index.htm;`
19. `limit_req zone=one burst=5;`
20. `#<==使用前面定义的名为one的内存空间，队列值为5，即可以有5个请求排队等待。`
21. `}`
22. `}`
23. `}`