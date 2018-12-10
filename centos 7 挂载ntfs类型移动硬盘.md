# centos 7 挂载ntfs类型移动硬盘

\1. 切换到系统yum目录并下载阿里的epel

[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# wget http://mirrors.aliyun.com/repo/epel-7.repo

\2. 查找当前源上可用的ntfs-3g软件

[root@localhost yum.repos.d]# yum list ntfs*
已加载插件：fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Repository epel-debuginfo is listed more than once in the configuration
Repository epel-source is listed more than once in the configuration
Loading mirror speeds from cached hostfile
 * elrepo: dfw.mirror.rackspace.com
 * epel: mirrors.aliyun.com
已安装的软件包
ntfs-3g.x86_64                        2:2017.3.23-1.el7                  @epel
可安装的软件包
ntfs-3g-devel.x86_64                  2:2017.3.23-1.el7                  epel 
ntfsprogs.x86_64 

\3. 安装：

[root@localhost yum.repos.d]# yum -y install ntfs-3g

\4. 挂载U盘

插入NTFS格式的U盘，fdisk -l 查看盘符，新建挂载目录并挂载

[root@localhost ~]# fdisk -l

磁盘 /dev/sdb：31.1 GB, 31104958464 字节，60751872 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x00000000
  设备 Boot      Start        End      Blocks  Id  System
/dev/sdb1  *        2048    60751871    30374912    7  HPFS/NTFS/exFAT   **-->这里的U盘是sdb1**

[root@localhost ~]# mkdir /mnt/ukey

[root@localhost ~]# mount ntfs-3g /dev/sdb1 /mnt/ukey

\5. 卸载U盘， 听说不卸载的话到Windows上会提示格式化磁盘 -_-''

[root@localhost ~]# umount /dev/sdb1