MYSQL  DOCKER镜像-e参数说明：

`-e`：指定环境参数，`e`是`environment`的缩写，在运行MySQL容器时可以指定的环境参数有：

```
    MYSQL_ROOT_PASSWORD ： root用户的密码，这里设置的初始化密码为`123456`；

    MYSQL_DATABASE ： 运行时需要创建的数据库名称；

    MYSQL_USER ： 运行时需要创建用户名，与MYSQL_PASSWORD一起使用；

    MYSQL_PASSWORD ： 运行时需要创建的用户名对应的密码，与MYSQL_USER一起使用；

    MYSQL_ALLOW_EMPTY_PASSWORD ： 是否允许root用户的密码为空，该参数对应的值为:yes；

    MYSQL_RANDOM_ROOT_PASSWORD：为root用户生成随机密码；

    MYSQL_ONETIME_PASSWORD ： 设置root用户的密码必须在第一次登陆时修改（只对5.6以上的版本支持）。

    MYSQL_ROOT_PASSWORD 和 MYSQL_RANDOM_ROOT_PASSWORD 两者必须有且只有一个。
```

 

 

 

 

 

 