---
title: Kerberos认证相关
date: 2018-01-18 19:09:00
categories:
   - 技术相关
tags:
   - KDC
   - Kerberos
   - 笔记
---
<!-- more -->
## 0x01 Kerberos认证过程 (简易版)

认证或请求服务 的过程如下: 
simple.jpg

`简义`

```
`KRB_AS_REQ (Kerberos Authentication Service Request)`
1. Client-A ---------------------------------------------------------==> KDC-AS (客户端执行散列运算加密一个时间戳,然后发送给身份验证服务(KDC-AS)) 此过程叫KRB_AS_REQ


`KRB_AS_REP (Kerberos Authentication Service Response)`
2. Client-A <==--------------------------------------------------------- KDC-AS (返回TGT,TGT票据使用KDC的krbtgt密钥进行加密) 此过程叫KRB_AS_REP


`KRB_TGS_REQ (Kerberos Ticket Granting Service Request)`
3. Client-A ----------------------------------------------------------==> KDC-TGS (Client-A使用AS返回的"会话密钥"构建访问特定服务的请求,再把AS返回的"TGT"连同请求一起发送到票据授予服务TGS) 此过程叫KRB_TGS_REP


`KRB_TGS_REP (Kerberos Ticket Granting Service Response)`
4. Client-A <==---------------------------------------------------------- KDC-TGS (TGS解密TGT和服务请求,并向Client-A发送一个服务票据ST(Service Ticket)


`KRB_AP_REQ (Kerberos Application Request)`
5. Client-A ----------------------------------------------------------==> Server-B (Client-A把服务票据中的服务器部分和请求一起发送到Server-B(Client-A要访问活动目录中的主机),远程服务器将直接接受该服务器票据,并不需要和KDC直接通信,因为该票据是用远程服务器和KDC共享的长期密钥加密过的)
```



## 0x02 Kerberos认证过程 (详细版)
Kerberos.jpg

认证或请求服务 的过程如下: 
#### 1. Client-A ---> KDC-AS 
```
KRB_AS_REQ (Kerberos Authentication Service Request) - 客户端执行散列运算加密一个时间戳,然后发送给身份验证服务(KDC-AS)
	a. 客户端Client对用户口令执行散列运算转换为NTLM散列。此散列值(即用户密钥)成为客户端和KDC共享的长期密钥(long term key)。 
	b. KRB_AS_REQ (Kerberos Authentication Service Request) - 客户端执行散列运算加密一个时间戳,然后发送给身份验证服务(KDC-AS)。
```

#### 2. Client-A <--- KDC-AS 
```
KRB_AS_REP (Kerberos Authentication Service Response) - 
身份验证服务(KDC-AS)会解密时间戳,若解密成功(KDC-AS检查用户的信息(登录限制.组成员身份等)并创建票据授予票据(Ticket-Granting Ticket,TGT),
并向本地LSA (Local Security Authority)请求生成一个特殊的数据PAC,表明了客户端获得某个特定用户的口令(即验证了用户的身份)。
身份验证服务(KDC-AS)向客户端回复两条信息: 
	a. 短期会话密钥SessionKeya-kdc,用于客户端向KDC发起后续的请求 ,该消息经客户端的长期密钥(long term key)加密。(此短期会话密钥仅适用于该客户端和KDC之间)
	b. 票据授予票据(Ticket Granting Ticket,简称TGT),包含有关用户名.域名.时间和组成员资格等信息。
TGT票据使用KDC的krbtgt密钥进行加密,PAC使用krbtgt密钥进行进行签名,并且系统很少会验证PAC数据(在Windows环境中为krbtgt账户的NT-Hash)。
```

#### 3. Client-A ---> KDC-TGS

``` 
KRB_TGS_REQ (Kerberos Ticket Granting Service Request) -
Client使用AS返回的”短期会话密钥”构建访问特定服务的请求,再把AS返回的”票据授予票据(TGT)”连同请求一起发送到票据授予服务TGS) 此过程叫KRB_TGS_REQ
Client-A使用AS返回的会话密钥SessionKeya-kdc构建访问特定服务的请求。客户端Client再把请求连同TGT一起发送到票据授予服务TGS。
(TGT是被KDC的krbtgt密钥加密的，所以Client-A无法解密)
黄金票据 - 此过程3可以伪造TGT(前提是获取krbtgt账号的口令散列值)，宣称自己是域内任何账号，包括域管或者不存在的用户，这是黄金票据的原理。
```
	
#### 4. Client-A <--- KDC-TGS

```
KRB_TGS_REP (Kerberos Ticket Granting Service Response) -
票据授予服务TGS解密TGT和服务请求,然后如果请求被允许(KDC会打开票据,进行校验和检查。如果DC能够打开票据,并能通过校验和检查,那么会认为TGT为有效票据。此时TGT中的数据会被复制,以创建TGS票据ST),
票据授予服务TGS向客户端Client发送一个服务票据(Service Ticket,简称ST),包括两个部分: 
	a. 远程服务器的部分 - 包含请求用户的组成员资格、时间戳、用于客户端和远程服务器之间通信的会话密钥。使用远程服务器Server-B和KDC共享的长期密钥(long term key)加密这部分消息。
	b. 客户端的部分 - 包含用于客户端和远程服务器之间通信的会话密钥SessionKeya-b。(使用步骤2中AS回复的短期会话密钥(SessionKeya-kdc)加密这部分消息生成的会话密钥SessionKeya-b。)
```

#### 5. Client-A ---> Server-B

```
KRB_AP_REQ (Kerberos Application Request) - 
Client-A把服务票据(Service Ticket)中的服务器部分和请求一起发送到Server-B(用户要访问活动目录中的主机)。
远程服务器将直接接受该服务器票据,并不需要和KDC直接通信,因为该票据是用远程服务器和KDC共享的长期密钥加密过的。
解密成功(目标服务会使用自己的NTLM密码散列打开TGS票据,并提取用户的授权数据和会话密钥SessionKeya-b。)即表明KDC已经允许了此次通信。
白银票据 - 此过程5可以伪造TGS(前提是获取服务账号的口令散列值)，宣称自己是域内任何账号，例如域管，这是白银票据的原理。
```

## 术语 
#### Kerberos

```
是Windows活动目录中使用的客户/服务器认证协议(windows中的认证协议有两种NTLM和Kerberos),为通信双方提供双向身份认证。
相互认证或请求服务的实体被称为委托人(principal)。参与的中央服务器被称为密钥分发中心(简称KDC)。
KDC(密钥分发中心　Key Distribution Center)有两个服务组成 : 
	1. AS 身份验证服务(Authentication Server)
	2. TGS 票据授予服务(Ticket Granting Server)
	该认证过程的实现不依赖于主机操作系统的认证,无需基于主机地址的信任,不要求网络上所有主机的物理安全,并假定网络上传送的数据包可以被任意地读取.修改和插入数据。
	在以上情况下, Kerberos 作为一种可信任的第三方认证服务,是通过传统的密码技术(如:共享密钥)执行认证服务的。
```

#### krbtgt账户
	每个域控制器DC都有一个"krbtgt"的用户账户,是KDC的服务账户,用来创建票据授予服务(TGS)加密的密钥。

#### Principal　委托人
	一个具有唯一标识的实体,可以是一台计算机或一项服务,通过使用KDC颁发的票据来进行通信。委托人可以分为两类: 用户和服务,分别具有不同种类的标识符。Kerberos信任模型的核心是每个委托人(principal)和KDC的通信是在利用仅双方可知的密钥构建的安全通道中进行。
当委托人(principal)之间需要通信的时候,它们再使用KDC生成的会话密钥。
```
1. 用户 (UPN)
	用户通过如"user@REALM"格式的用户主体名称(User Principal Name,简称UPN)来标识。记住REALM一定是大写的。
2. 服务 (SPN)
	服务主体名称(Service Principal Name,简称SPN)是Kerberos身份验证服务(AS)所必需。
	用于域中的服务和计算机账户。SPN的格式形如"serviceclass/host_port/serviceName"。
	例如, 主机"dc1.bhusa.com"上LDAP服务的SPN可能类似于"ldap/dc1.bhusa.com", "ldap/dc1"和"ldap/dc1.bhusa.com/bhusa.com"。
	参考全限定主机名和仅主机名,一个服务可能注册为多个SPN。(同通常是执行DNS查询来规范化主机名称。这就解释了DNS为什么是微软Kerberos环境中的一个必要组件。查询服务的"规范化"名称,然后生成请求服务的SPN。)
```

`枚举域帐户的SPN`
```
GetUserSPNS.ps1 - https://github.com/nidem/kerberoast/blob/master/GetUserSPNs.ps1
Find-PSServiceAccounts.ps1 - https://github.com/PyroTek3/PowerShell-AD-Recon/blob/master/Find-PSServiceAccounts
PowerView 的 Get-NetUser -SPN - https://github.com/PowerShellMafia/PowerSploit/blob/5690b09027b53a5932e42399f6943e03fa32e549/Recon/PowerView.ps1#L2087-L2089

getspn.png
```
#### PAC (Privilege Access Certificate 特权访问证书)
	KDC在向Kerberos客户端颁发TGT时,会向本地LSA (Local Security Authority)请求生成一个特殊的数据结构,名为"特权访问证书"这个PAC包含为Kerberos客户端构建一个本地访问令牌所需的用户信息,他同时使用域控制器服务器的私钥和KDC服务器的私钥来进行数字签署,以防假的KDC伪造PAC。

```
1. 用户的登入时间以及用户会话额到期时间
2. 用户上一次设置密码的时间,以及允许他再次更改密码的时间
3. 用户的经典登入名,domain\user
4. 用户的显示名称
5. 指派给用户账户的经典NT登入脚本的名称(如果有的话)
6. 用户漫游配置文件的UNC路径
7. 客户端主目录的UNC路径
8. 用户的并发登入数
9. 在颁发PAC的KDC处,自从上一次成功登入以来,所允许的不成功登入尝试次
10. 用户的RID
11. 用户的"主要组"的RID,只限在POSIX使用
12. 在域中,将用户作为一个成员的组的数量,以及每个组的RID
13. 适用于用户的已知SID
14. 域的SID
15. 资源域的SID
```

除此之外,PAC中还嵌入了另一个名为`用户账户控制`的数据结构 
如何得到

```
powerview 
	Get-ComputerProperty -Properties distinguishedname,useraccountcontrol,msds-allowedtodelegateto| fl
	Get-UserProperty -Properties distinguishedname,useraccountcontrol,msds-allowedtodelegateto| fl

dsquery
	dsquery * -limit 0 -filter "(&(objectCategory=computer)(objectClass=computer))" -attr cn operatingSystem distinguishedName useraccountcontrol msds-allowedtodelegateto
	dsquery * -l -limit 0 -filter "&(!objectClass=computer)(servicePrincipalName=*)" -attr serviceprincipalname name samaccountname memberof pwdlastset distinguishedname useraccountcontrol
	dsquery * -limit 0 -filter "(&(objectCategory=user)(objectClass=user))" -attr cn distinguishedName useraccountcontrol msds-allowedtodelegateto
```

#### LSA (Local Security Authority)
	LSA管理本地安全策略、管理审计策略和设置、为用户生成包含SID和组权限关系的令牌。LSA验证的过程: LSA通过访问本地SAM(Security Accounts Manager)数据库,可以完成本地用户的验证。
`LSA的处理流程`: 
```
1. LSA首先会把身份凭据交给SSPI,由该接口负责与Kerberos和NTLM服务沟通。
2. SSPI不能确定用户是本地登录还是域账户进行域登录。所以他会先把身份认证请求传递到Kerberos SSP。
3. Kerberos SSP会验证用户的登入目标是本地计算机还是域。如果是登录域,Kerberos SSP将继续处理。如果是本地计算机,即用户不是登录域,Kerberos SSP返回一个错误消息到SSPI,交回给GINA处理,使服务器登录不可用。
4. SSPI现在发送请求到下一个安全提供程序——NTLM。NTLM SSP会将请求交给Netlogon服务针对LSAM (Local Security Account Manager,本地安全账户管理器)数据库进行身份认证。使用NTLM SSP的身份认证过程与Windows NT系统的身份认证方法是相同的。
```

#### 相关 
```
1. krbtgt 密码: 
	它是整个活动目录中唯一的不会自动更新的密码。
	除非:
	第一种情况: 域功能级别(domain functional level,简称DFL)从NT5(2000/2003)升级到NT6(2008/2012)。
	第二种情况: 利用域的恢复数据来实施域的裸机恢复(bare metal recovery)。
2. Kerberos智能卡进行身份认证
	Kerberos也允许使用PKI和智能卡进行身份认证。用户会被提示输入一个智能卡的PIN码,而不是口令。
	Windows使用PIN码来访问智能卡上的公钥证书(public key certification)。利用智能卡的私钥签名该证书,并发送到KDC。
	KDC验证证书上的签名是否源于可信实体。然后KDC发送公钥证书加密过的TGT。既然信息只能被智能卡的私钥解密,用户也就通过了域的身份认证。
	然而,对于使用智能卡进行身份认证的账户来说,密码的散列值仍然存储在域控服务器上。此外,智能卡只能对"交互式会话(interactive sessions)"提供保护。
	也就意味着智能卡认证仅能用于登录域中的计算机。
```

#### 參考
```
深入解读MS14-068漏洞:微软精心策划的后门?
	http://www.freebuf.com/vuls/56081.html
敞开的地狱之门:Kerberos协议的滥用
	http://www.freebuf.com/articles/system/45631.html
你所不知道的Kerberos 整理笔记(三)
	http://www.voidcn.com/article/p-nnpovuml-ng.html
```
