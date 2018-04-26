---
title: 搭建私有Tor网络
date: 2018-04-11 19:58:00
categories:
   - 技术相关
tags:
   - Tor
   - 笔记
   - 网桥
   - 中继
   - 权威目录服务器
---
.....
<!-- more -->

# 搭建私有Tor网络 ( private Tor network)

RS1 ： Tor Client (客户端)	

RS2 ： Tor relay	(退出及中继节点)	2个中继+1退出中继

RS3 ： Tor Bridge relay	(网桥)	

RS4 ： Authority Server (权威目录服务器)


权威服务器(Authority Server)必须和洋葱路由器(Tor)在同一时间

    apt-get install tor ntpdate 
    ntpdate time.nist.gov 

## 配置权威目录服务器RS4(Setup Authority Server on RS4)  

(1). 运行以下命令生成的权威密钥(authority keys)

mkdir /var/lib/tor/keys

    sudo tor-gencert --create-identity-key -m 12 -a 192.168.18.111:7000 \
                -i /var/lib/tor/keys/authority_identity_key \
                -s /var/lib/tor/keys/authority_signing_key \
                -c /var/lib/tor/keys/authority_certificate 
or ===> 

    sudo tor-gencert --create-identity-key -m 12 -a 192.168.18.111:7000 -i /var/lib/tor/keys/authority_identity_key -s /var/lib/tor/keys/authority_signing_key -c /var/lib/tor/keys/authority_certificate  -v

当提示输入密码，请使用你的密码 dfgt@#dfd34341sd~~!。该命令将生成以下文件

    authority_identity_key ：长期密钥签署授权证书(authority certificate)
    authority_signing_key ：中期键(medium-term key)(3-12个月)签署目录信息
    authority_certificate ：由权威身份密钥(authority identity key)签名的文件，以证明权威签名密钥。

(2). 其次，生成router keys

https://tor.stackexchange.com/questions/8873/creating-directory-server
http://fengy.me/prog/2015/01/09/private-tor-network/

	tor --list-fingerprint --orport 1 \
    --dirserver "name 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" \
    --datadirectory /var/lib/tor

or ==>

	sudo tor --list-fingerprint --orport 1 --dirserver "name 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" --datadirectory /var/lib/tor/
    (if permission denied : sudo chown -R root:root /var/lib/tor)

该命令将生成以下文件：

    secret_id_key ：长期密钥来签署路由器描述符和TLS证书。
    secret_onion_key ：用于建立一个电路和协商临时密钥中期关键。
    secret_onion_key_ntor ：握手短期关键(short-term key for handshake.)。
    fingerprint : fingerprint of the identity key.

(3) 编辑torrc 
gedit /etc/tor/torrc 

	TestingTorNetwork 1
	DataDirectory /var/lib/tor
	RunAsDaemon 1
	ConnLimit 60
	Nickname RS4
	ShutdownWaitLength 0
	PidFile /var/lib/tor/pid
	Log notice file /var/lib/tor/notice.log
	Log info file /var/lib/tor/info.log
	ProtocolWarnings 1
	SafeLogging 0
	DisableDebuggerAttachment 0
	DirAuthority RS4 orport=5000 v3ident=finger1 192.168.18.111:7000 finger2
    
	SocksPort 0
	OrPort 5000
	Address 192.168.18.111
	DirPort 7000
    
	# An exit policy that allows exiting to IPv4 LAN
	#ExitPolicy accept 192.168.16.0/22:*
	# An exit policy that allows exiting to IPv6 localhost
	#ExitPolicy accept [::1]:*
	#IPv6Exit 1

	AuthoritativeDirectory 1
	V3AuthoritativeDirectory 1
	ContactInfo auth@test.test
	ExitPolicy reject *:*

service tor start

finger1 : /var/lib/tor/keys/authority_certificate 找到 fingerprint 9F891F74141865DD89F1EB7D1C5853AB6188D041

finger2 : /var/lib/tor/fingerprint 

	eg. Your Tor server's identity key fingerprint is 
	'Unnamed C78C7376A09461466F95C56E36199148058FBF84'
	===> finger2 = C78C 7376 A094 6146 6F95 C56E 3619 9148 058F BF84

## 配置客户端(Tor Client) 
(1) 生成router keys

    tor --list-fingerprint --orport 1 \
        --dirserver "x 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" \
        --datadirectory /var/lib/tor
        
(2) 编辑torrc 

gedit /etc/tor/torrc 

	OrPort 5000
	SocksPort 9011
    Exitpolicy reject *:*		
    Nickname RS1
	DirAuthority RS4 orport=5000 v3ident=finger1 192.168.1.4:7000 finger2 #(finger1 and finger2 in RS4)
	
	TestingTorNetwork 1
	DataDirectory /var/lib/tor
	RunAsDaemon 1
	ConnLimit 60
	ShutdownWaitLength 0
	PidFile /var/lib/tor/pid
	Log notice file /var/lib/tor/notice.log
	Log info file /var/lib/tor/info.log
	ProtocolWarnings 1
	SafeLogging 0
	DisableDebuggerAttachment 0
	
service tor start

## 配置中继和退出节点(以及网桥) 
### 配置中继节点 Tor Relay (s)

(1) 生成router keys

    tor --list-fingerprint --orport 1 \
        --dirserver "x 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" \
        --datadirectory /var/lib/tor
	
(2) 编辑torrc 

gedit /etc/tor/torrc 

	OrPort 5000
	SocksPort 0					#RS1 diffrent
    Exitpolicy reject *:*		
    Nickname RS2
	DirAuthority RS4 orport=5000 v3ident=finger1 192.168.1.4:7000 finger2 #(finger1 and finger2 in RS4)
	
	TestingTorNetwork 1
	DataDirectory /var/lib/tor
	RunAsDaemon 1
	ConnLimit 60
	Nickname RS1
	ShutdownWaitLength 0
	PidFile /var/lib/tor/pid
	Log notice file /var/lib/tor/notice.log
	Log info file /var/lib/tor/info.log
	ProtocolWarnings 1
	SafeLogging 0
	DisableDebuggerAttachment 0
	# An exit policy that allows exiting to IPv4 LAN
	ExitPolicy accept 192.168.16.0/22:*
	# An exit policy that allows exiting to IPv6 localhost
	ExitPolicy accept [::1]:*
	IPv6Exit 1
	
service tor start

### Tor Relay (s)配置退出中继节点

只是在"Tor Relay (s)配置中继 " 配置文件中加一个：

    ExitRelay 1


## 配置网桥 Bridge relay
Ubuntu

must have External IP 

apt-get install obfs4proxy

gedit /etc/tor/torrc   

	SOCKSPort 0 					
	ORPort 5000				
	ExtORPort auto
	BridgeRelay 1
	UpdateBridgesFromAuthority 1	
	ServerTransportPlugin obfs4 exec /usr/bin/obfs4proxy   
	Exitpolicy reject *:*			
	RunAsDaemon 1
	PublishServerDescriptor 0		
	Nickname RS3
	ContactInfo test@test.test 
	DirAuthority RS4 orport=5000 v3ident=finger1 192.168.1.4:7000 finger2 #(finger1 and finger2 in RS4)

service tor start

sudo cat /var/lib/tor/fingerprint




## 使用网桥 UseBridges 

    UseBridges 1
    Bridge obfs4 89.46.73.150:41595 BE158D7939B8C95C54F07C50D7EBE50BEDA68C4D cert=3OCxcKIP+9UmUUVJPgjArM95dpZUDJv6+uFR35KlKy6JzkxbN3llrvE1jNzhFPWaX2mgZw iat-mode=0
    Bridge obfs4 13.58.94.90:9443 DDAFCB98850DE23177224F382049A6FCD4A80E4B cert=MvzYIjUby2RfVa5ATIKkf1bC3lQ4JSzzmWtHCDcKZaCxByn4Tp5R30bMiMnOEXrKPP8qeA iat-mode=0
    Bridge obfs4 83.212.97.47:54187 80FCA5A349AE7E5C2C8503BFB908D4204FDB9C3E cert=IfdoBRxcIl/l5YrMUxFrNSOOI5DjU3w8IcZI/CQMbpzBj/UdpdCZsT5yfbZ1MFL6xmTTGw iat-mode=0
    ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy
    MaxCircuitDirtiness 600    # 600 seconds = 10 minutes

firefox : socks5 ---> 127.0.0.1 9050

## Test The Private Network

在RS1，设置火狐为"127.0.0.1:9011" SOCKS5
Wireshark来观看每RS1访问http://192.168.1.4



参考 

https://www.torproject.org/docs/tor-manual-dev.html.en

https://ritter.vg/blog-run_your_own_tor_network.html

http://fengy.me/prog/2015/01/09/private-tor-network/


写于 : 2017-10-30 16:09:00



