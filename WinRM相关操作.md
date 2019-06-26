- # 		**WinRM相关操作**

开启WinRM服务：

Enable-PSRemoting –Force

阻止本地计算机接收远程命令(不会停止WinRM服务)：

Disable-psremoting –Force

查看WinRM服务监听信息：

winrm enumerate winrm/config/Listener

WinRM2.0默认端口5985（HTTP端口）或5986（HTTPS端口）。

配置https连接方式 , 查看ca证书
ls Cert:\LocalMachine\My\

删除WinRM HTTP侦听：

winrm delete winrm/config/listener?Address=*+Transport=HTTP

重新建立HTTP侦听：

winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Port="6998" ;Hostname="WMSvc-lebangServer" ;CertificateThumbprint=""}

winrm create winrm/config/listener?Address=*+Transport=HTTP

WinRM服务更改监听端口：

set-item -force wsman:\localhost\listener\listener*\port 5985

查看WinRM的配置：

winrm get winrm/config

查看端口监听状态：

netstat -nao | findstr "5985"



sed -i "s#tdout_buffer.append(stdout)#tdout_buffer.append(stdout.decode('gbk').encode('utf-8'))#g" /usr/lib/python2.7/site-packages/winrm/protocol.py

sed -i "s#stderr_buffer.append(stderr)#stderr_buffer.append(stderr.decode('gbk').encode('utf-8'))#g" /usr/lib/python2.7/site-packages/winrm/protocol.py
--------------------- 





