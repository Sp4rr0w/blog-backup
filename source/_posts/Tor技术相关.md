---
title: Tor技术相关
date: 2018-04-07 11:58:00
categories:
   - 技术相关
tags:
   - Tor
   - 笔记
   - 网桥

---
.....
<!-- more -->

## Tor的工作原理

基于TCP连接的匿名通信系统,它可以用于网页浏览、即时消息等通信。

它首先从目录服务器(Directory Server,DS)中获得所有Tor节点信息,然后在Tor节点集合中随机选取一个节点,与它协商密钥,最后与它建立一个安全信道。

## Tor 技术相关

### 目录服务器(Directory Server)

    存放Tor 节点的列表
    
#### 无网桥

    1. 本地Tor客户端A从目录服务器Directory Server获取Tor受信节点机器列表 
    2. 本地Tor客户端A挑选随机3条加密节点访问B(web)  (当经过一段时间后,tor将重新为你的访问建立一条线路访问B(web))


### 网桥(Bridge relays / Bridges) 
    
    中继节点都是通过从目录服务器或者目录镜像得到,GFW只需把中继节点IP加入黑名单即可
    网桥方式或许可以绕过这种审查限制
    
    作为本地网和Tor网络之间的支持物,Tor 中继节点,用于帮助用户绕过审查或封锁。
    

#### 有网桥

    1. 本地Tor客户端A连接网桥 (从获取bridges.torproject.org或者搭建)
    2. 本地Tor客户端A通过网桥从目录服务器Directory Server获取Tor受信节点机器列表 
    3. 本地Tor客户端A通过网桥挑选随机3条加密节点访问B(web)

    本地 -> Bridges -> Directory Server (3 encrypted nodes) -> web

### Bridges 和 Relay 有啥区别?

    (1). Tor 的节点分两种 :
    
        1. 第一种是中继节点,而中继节点又分两种：
        
            Tor Relay 中继 - 需要加入 Directory ,GFW 是通过流量特征屏蔽 Tor,但作为第一跳的 IP 还是可能会被探测屏蔽。
            Tor Bridges 网桥(Bridge relays) - tor 的网桥是一种特殊的中转节点。
                它跟其它中转节点的差异在于：普通的中转节点,其信息会被加入到 TOR 在全球的目录服务器(Directory server),
                所以普通的中转节点会被所有的人看到,并用来进行流量中转。
                而你自己搭建的网桥,其信息不会被加入到全球的目录服务器——只会被你自己所用。
                除了使用桥梁作为其第一跳,用户还可以使用它们来获取 directory 更新
            他们配置都差不多的,取决于你是否公开。(https://www.torproject.org/docs/faq.html.en#RelayOrBridge)
            (Being a normal relay vs being a bridge relay is almost the same configuration: it's just a matter of whether your relay is listed publicly or not.)	
		
        2. 出口节点(Exit Nodes),所谓蜜罐也都是部署在出口节点上的。
        
            出口节点以外的访问的目标网站如果不是 SSL,那么从出口到目标网站就是明文的,会暴露 DNS 记录和明文密码。
		
    (2). 避免蜜罐节点
    
        如果你使用的线路中,出口节点正好是蜜罐,那么该蜜罐就会窥探到你的上网行为,
        并且,假如你访问的目标网站没有 HTTPS 加密,蜜罐就会知道你浏览的页面内容。
        修改 TOR 的配置文件,规避这些不安全国家的节点。。
        在该文件末尾,加入下面这行(ExcludeNodes 表示排除这些国家/地区的节点,StrictNodes 表示强制执行)。
        
            ExcludeNodes {cn},{hk},{mo},{kp},{ir},{sy},{pk},{cu},{vn}
            StrictNodes 1
            ExitNodes {us}#出口节点
            
        关于 StrictNodes 
            如果不设置 StrictNodes 1,TOR 客户端首先也会规避 ExcludeNodes 列出的这些国家。但如果 TOR 客户端找不到可用的线路,就会去尝试位于排除列表中的节点。
            如果设置了 StrictNodes 1,即使 TOR 客户端找不到可用的线路,也不会去尝试这些国家的节点。
    
    (3). 多重代理
    
        TOR + 其它翻墙工具( VPN)
        TOR 本身就是一个多重代理！既然这样,为啥还要拿 TOR 跟其它翻墙工具搭配捏?
            ==>主要因为 GFW 把全球大部分的 TOR 节点都进行了IP封锁
            
DataDirectory /var/lib/tor
#此目录为tor主要的运行数据存放目录包括key文件等


    
### Tor的客户端是如何知道Tor网络中的中继节点有哪些的呢? 

    这里有一个概念,叫目录服务器(Directory Server)。
    目前,官方的目录服务器一共有9台,它们之间会定期同步数据。
    如果想成为中继节点,Tor客户端必须要向其中一台目录服务器注册自己。
    而且Tor的客户端是不用直接连接到官方的目录服务器来获得所有的中继节点信息的,
    而是通过所谓的目录镜像(Directory Mirror)。这些目录镜像会定期的从官方的目录服务器上拷贝一份最新的数据。
    这样做的好处是分担了目录服务器的负担,事实上,大部分中继节点都被配置成了目录镜像。

### 如何使用自己的节点机器列表?

要设置自己的Tor网络,您需要运行自己的authoritative directory servers,

并且必须配置客户端和中继(relays),以便他们了解您的directory servers,而不是默认的公用directory servers。

    1.搭建一个网桥(Bridge relays)
    2.搭建一个目录服务器Directory Server 
    3.搭建很多个中继节点(Relays Nodes)及出口节点(Exit Nodes)

or 

    1.搭建一个网桥权威(Bridge authorities)	#就像正常的v3目录权威(v3 directory authorities)
    2.搭建很多个中继节点(Relays Nodes)及出口节点(Exit Nodes) 

运行一个Tor中继只需满足以下几个条件：

	你的互联网连接的上下行带宽至少为20KB/S(而且在你电脑开机期间都要保证稳定的连接)。
	你的互联网连接必须拥有一个可以路由传送的公网IP地址。
	如果你的电脑处于网络地址转换( NAT )防火墙内,没有公网IP地址,你必须在路由器上设定端口映射规则。你可以借助Tor的通用即插即用设备完成这一步骤,也可以通过查看你的路由器使用说明,或者根据portforward.com(http://portforward.com/english/applications /port_forwarding/HTTPS/HTTPSindex.htm)上的说明手动设置它。

非必需条件有：

    电脑不必一直开机联网(Tor目录会在开机并联网的时候设置好一切)。
    不是一个静态IP地址。

小知识 ： 
除了手动配置几个directory servers,relays和客户端之外,还有两个单独的工具可以帮助您。一个是Chutney,另一个是Shadow

    https://ritter.vg/blog-run_your_own_tor_network.html
    Chutney和Shadow大多是专为在labratory条件下运行experiements建立测试网络工具.
    Shadow是专门为跨大型TOR网络运行带宽测试而设计的。所以,如果你想运行建模50000节点Tor网络-Shadow的你再合适不过了。

    Chutney 是一种配置工具,用于测试/控制/运行Tor网络.
    Shadow 是一个网络模拟器,可以通过其Scallion插件运行Tor
        虽然它通常用于在基本上较大的Tor测试网络上运行负载和性能测试,而不是Chutney可行的测试网络,
        但由于您可以运行完全确定性的实验,因此它也可以作为一个出色的调试工具
        一个大型的Shadow网络有数千个Tor,Shadow可以在没有root的任何Linux机器上运行,也可以使用预配置的镜像EC2上运行
        
    Swarm - 简单的工具,用来管理Docker集群,它将一群Docker宿主机变成一个单一的/虚拟的主机.
        一个swarm集群由很多个机器组成,不管是虚拟机还是物理机都可以。
        最简单的方式就是执行docker swarm init来开启一个swarm节点,并且把当前执行命令的节点作为管理节点,
        然后在其他机器上执行docker swarm join来加入这个swarm集群。
    stack - Docker层级关系中的最高一个层级


### Configuring a Tor relay(中继)

https://www.torproject.org/docs/tor-doc-relay.html.en
https://www.torproject.org/docs/tor-relay-debian
https://www.torproject.org/docs/faq.html.en#ExitPolicies

gedit /etc/tor/torrc ：

	ORPort 443
    Exitpolicy reject *:*		# avoid abuse, 此设置意味着您的中继将用于中继(relaying traffic)Tor网络中的流量,但不用于连接到外部网站或其他服务。
    Nickname ididntedittheconfig
    ContactInfo human@...
	PublishServerDescriptor 0	# not publish anywhere, private relay
	#BridgeRelay 1  			# only add this line if you want to be a bridge
	FallbackDir ipv4address:port orport=port id=fingerprint [weight=num] [ipv6=[ipv6address]:orport]  #
	DirAuthority [nickname] [flags] ipv4address:port fingerprint   
	#公共中继默认为退出中继(exit node出口节点)(要么更改ExitPolicy行:Exitpolicy reject *:*)
	一旦您的中继设备连接到网络,它将尝试确定您配置的端口是否可从外部访问。这一步通常很快,但可能需要20分钟
	当您的中继确定可达时,它会将"server descriptor"上传到目录(directories)中,让客户端知道您的中继使用的地址,端口,密钥等。
	您可以搜索Atlas或Globe获取您配置的昵称
    
------------------------------------------------------------------------------------------------------------

### Configuring a Tor Bridge relay(Bridges网桥)

https://github.com/Yawning/obfs4
https://hub.docker.com/r/vimagick/tor/
https://tor.stackexchange.com/questions/6370/how-to-run-an-obfs4-bridge
https://trac.torproject.org/projects/tor/wiki/doc/PluggableTransports/obfs4proxy 
https://gitweb.torproject.org/torspec.git/tree/attic/bridges-spec.txt

**Ubuntu**
`must have External IP` #如果你的电脑处于网络地址转换( NAT )防火墙内,没有公网IP地址,你必须在路由器上设定端口映射规则

apt-get install tor
apt-get install obfs4proxy

gedit /etc/tor/torrc   

	RunAsDaemon 1
	SOCKSPort 0 					#
	ORPort auto #9004				#定义一个ORPort ,不作为出口节点,设置成Bridge .ORPort default to 443 on Windows, and 9001 on other platforms.
	ExtORPort auto
	BridgeRelay 1
	UpdateBridgesFromAuthority 1	#尝试查询网络授权(the bridge authority), 建议桥梁用户始终设置UpdateBridgeFromAuthority
	ServerTransportPlugin obfs4 exec /usr/bin/obfs4proxy   
	Exitpolicy reject *:*			
	PublishServerDescriptor 0		#1 open 
	Nickname Sp4rr0w
	ContactInfo Sp4rr0w@Sp4rr0w.com 
	
sudo ls -al /var/lib/tor
sudo cat /var/lib/tor/fingerprint	#obfs4 CODE
sudo tail -F /var/log/tor/log		#日志文件 输出成功标志
	
https://bridges.torproject.org/bridges?transport=obfs4 :

    obfs4 104.236.237.76:9443 F68BBB77DA245790969057EE608E6B14283655EB cert=lxPxDTtghn56dGKbQ2vicevpXiUHNzJpSxCLMd76A8MhbKPVQqurd9Uf4sx7yOfYDA5dBw iat-mode=0
    obfs4 77.81.108.138:443 F4B33297F39EC5CF2EFD0E3D2F3EB87EC8428237 cert=Ax+LafRJEU1v5BGdXV0To86Vs+gr+EPbIvOZE3iL/fAY3pxScQppxW5y8e03DSCJuqZlCA iat-mode=0
    obfs4 104.153.209.217:21567 D28E0345809AE4BAC903EF7FC78CAAF111A63C58 cert=DtNNYXeRG4ds+iTM7sdbJHJgH7RmxDb1lt8JR17BiT7eHnORyn+4y+RcoqAI65XGvhXKJg iat-mode=0

https://www.torproject.org/docs/tor-manual-dev.html.en

	-> DirAuthority [nickname] [flags] address:port fingerprint  # DirServer replaced by DirAuthority
	如果没有给出 DirAuthority 行,Tor将使用默认目录权威(default directory authorities).
	注意：此选项用于使用自己的own directory authorities设置私有Tor网络。如果您使用它,您将与其他用户区分开来,因为您不会相信他们所做的相同的权限。
	

架设隐藏服务,如Deep Web暗网：

    HiddenServiceDir /var/lib/tor/hidden_service/
    HiddenServicePort 80 127.0.0.1:80 #将访问的80端口定向到Nginx/Apache,Nginx监听127.0.0.1:80

bridge descriptor ： by bridge -> /tor/server/authority    or go to bridge authority and asking for "/tor/server/fp/ID

### Configuring a directory authority 

https://www.torproject.org/docs/tor-manual-dev.html.en

gedit /etc/tor/torrc

	AuthoritativeDirectory 1 	##当此选项设置为1时,Tor将作为权威目录服务器运行
	V3AuthoritativeDirectory 1
	VersioningAuthoritativeDirectory 1
	RecommendedVersions STRING
    
### Configuring a Bridge authorities (or v3 directory authorities
https://gitweb.torproject.org/torspec.git/tree/attic/bridges-spec.txt

Bridge authorities are like normal v3 directory authorities

网桥权威(Bridge authorities)就像正常的v3目录权威(v3 directory authorities)
become a bridge authority
 -> gedit /etc/tor/torrc  :
		
	AuthoritativeDirectory 1
	BridgeAuthoritativeDir 1


### Centos7 setup Bridge relay(Bridges)

yum install tor -y

yum install git mercurial golang -y8t9xd4g

install obfs4proxy :

	export GOPATH=`mktemp -d`
	go get git.torproject.org/pluggable-transports/obfs4.git/obfs4proxy
	cp $GOPATH/bin/obfs4proxy /usr/local/bin
	
	之后 ServerTransportPlugin obfs4 exec /usr/local/bin	#换一下就好
	
	
### UseBridges 

UseBridges 1

Bridge obfs4 192.168.18.111:9001 4D33C1FCB32E69093E36B8BAD53291A3E850C804

ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy



写于 : 2017-10-16 16:09:00

参考：
    http://m.it610.com/article/3961692.htm
    http://www.haiyun.me/archives/1059.html
    https://program-think.blogspot.com/2013/11/tor-faq.html
    https://dreamcreator108.com/dreams/tor-bridge/index.html
    https://www.torproject.org/docs/running-a-mirror.html.en
    https://www.torproject.org/docs/tor-manual-dev.html.en
    https://github.com/Yawning/obfs4
    https://hub.docker.com/r/vimagick/tor/
    https://tor.stackexchange.com/questions/6370/how-to-run-an-obfs4-bridge
    https://trac.torproject.org/projects/tor/wiki/doc/PluggableTransports/obfs4proxy 
    https://gitweb.torproject.org/torspec.git/tree/attic/bridges-spec.txt
    https://www.torproject.org/docs/tor-doc-relay.html.en
    https://www.torproject.org/docs/tor-relay-debian
    https://www.torproject.org/docs/faq.html.en#ExitPolicies
