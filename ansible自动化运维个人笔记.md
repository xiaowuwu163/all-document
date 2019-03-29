# ansible自动化运维个人笔记

步骤一： 

​	修改网络模式，添加为专网，开通windows ping服务端口，自定义防火墙规则，cmpt

​	添加5985winrm管理端口，非加密

步骤二：

​	升级powershell 4.0（以上）

​	1、管理windows server 2008r2 posershell 3.0 出现错误，不够稳定，需要升级为4.0（以上)

​		安装framework .NET 4.5（以上）

 		安装windows Microsoft Framework（4.0)以上

​		安装python，环境错误，需要安装Microsoft V c++ 2008

​	2、windows server 2012r2 管理，未出现错误

​	3、windows server 2016r2 出现ansible2.7.2版本需要安装好依赖pip install pywinrm==2.19 ansible==2.19 kerberos==1.3.0 paramiko PyYAML Jinja2 httplib2 six

​		安装依赖后出现不能执行win_shell 模块，不能启动线程， 多次执行后可以监控主机，未查出原因。。。。



步骤三: 

​	配置winrm， 执行自定义脚本pwershell-winrm.bat



备注：

*此操作为升级windows 2008 r2系统powershell到4.0版本
1、安装dotNetFx45_Full_x86_x64.exe ,net Framework 4.5以上
2、安装Windows6.1-KB2819745-x64-MultiPkg.msu

*若系统为windows2012及以上版本系统，则从第3步开始执行
3、配置winrm 在cmd中执行powershell-winrm.bat脚本

4、若powershell不能执行.ps1脚本则设置策略为 remotesigned 执行命令set-ExecutionPolicy RemoteSigned 
查看当前策略get-executionpolicy
5、修改网络设置，将网络模式改为专用网络
6、添加防火墙规则，添加入站规则==》端口==》5985

7、##2008版本特定步骤，系统可能未自带Microsoft Visual C++ 2008 Redistributable Package包，执行vcredist_x64.exe安装程序

