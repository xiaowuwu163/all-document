# [Mysql的read_only 只读属性说明 (运维笔记)](https://www.cnblogs.com/kevingrace/p/10095332.html)

 

在MySQL数据库中，在进行数据迁移和从库只读状态设置时，都会涉及到只读状态和Master-Slave主从关系设置, 以下针对real_only只读属性做些笔记记录:

**1) 对于MySQL单实例数据库和master库，如果需要设置为只读状态，需要进行如下操作和设置：**
将MySQL设置为只读状态的命令（可以登录mysql执行下面命令， 或者在my.cnf配置文件中添加"read_only=1",然后重启mysql服务）:

```
mysql> show global variables like "%read_only%";
mysql> flush tables with read lock;
mysql> set global read_only=1;
mysql> show global variables like "%read_only%";
```

将MySQL从只读状态设置为读写状态的命令:

```
mysql> unlock tables;
mysql> set global read_only=0;
```

**2) 对于需要保证master-slave主从同步的salve库**
将slave从库设置为只读状态，需要执行的命令为 (下面命令中的1 也可以写成 on):

```
mysql> set global read_only=1;
```

将salve库从只读状态变为读写状态，需要执行的命令是:

```
mysql> set global read_only=0;
```

对于Mysql数据库读写状态，主要靠"read_only"全局参数来设定；默认情况下, 数据库是用于**读写操作**的，所以read_only参数也是0或faluse状态，这时候不论是本地用户还是远程访问数据库的用户，都可以进行读写操作；

如需设置为**只读状态**，将该read_only参数设置为1或TRUE状态，但设置 read_only=1 状态有两个需要注意的地方：
1) read_only=1只读模式，不会影响slave同步复制的功能，所以在MySQL slave库中设定了read_only=1后，通过 "show slave status\G" 命令查看salve状态，可以看到salve仍然会读取master上的日志，并且在slave库中应用日志，保证主从数据库同步一致；
2) read_only=1只读模式，限定的是普通用户进行数据修改的操作，但不会限定具有super权限的用户的数据修改操作 (但是如果设置了"**super_read_only=on**"， 则就会限定具有super权限的用户的数据修改操作了)；在MySQL中设置read_only=1后，普通的应用用户进行insert、update、delete等会产生数据变化的DML操作时，都会报出数据库处于只读模式不能发生数据变化的错误，但具有super权限的用户，例如在本地或远程通过root用户登录到数据库，还是可以进行数据变化的DML操作；**(也就是说"real_only"只会禁止普通用户权限的mysql写操作，不能限制super权限用户的写操作； 如果要想连super权限用户的写操作也禁止，就使用"flush tables with read lock;"，这样设置也会阻止主从同步复制！)**

### **锁表操作**

为了确保所有用户，包括具有super权限的用户也不能进行读写操作，就需要执行给所有的表加读锁的命令 "flush tables with read lock;"，这样使用具有super权限的用户登录数据库，想要发生数据变化的操作时，也会提示表被锁定不能修改的报错。这样通过设置"read_only=1"和"flush tables with read lock;"两条命令，就可以确保数据库处于只读模式，不会发生任何数据改变，在MySQL进行数据库迁移时，限定master主库不能有任何数据变化，就可以通过这种方式来设定。

但同时由于加表锁的命令对数据库表限定非常严格，如果再slave从库上执行这个命令后，slave库可以从master读取binlog日志，但不能够应用日志，slave库不能发生数据改变，当然也不能够实现主从同步了，这时如果使用 "unlock tables;"解除全局的表读锁，slave就会应用从master读取到的binlog日志，继续保证主从库数据库一致同步。

为了保证主从同步可以一直进行，在slave库上要保证具有super权限的root等用户只能在本地登录，不会发生数据变化，其他远程连接的应用用户只按需分配为select,insert,update,delete等权限，保证没有super权限，则只需要将salve设定"read_only=1"模式，即可保证主从同步，又可以实现从库只读。相对的，设定"read_only=1"只读模式开启的解锁命令为设定"read_only=0";设定全局锁"flush tables with read lock;"，对应的解锁模式命令为："unlock tables;".当然设定了read_only=1后，所有的select查询操作都是可以正常进行的。