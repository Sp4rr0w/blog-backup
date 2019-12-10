---
title: Windows身份验证体系结构
date: 2019-12-04 14:55:00
categories:
   - 技术相关
tags:
   - Windows
   - 身份验证
   - 笔记
   - SSP&SSPI
summary_img: /images/pic3.jpg
---

# 0x00 Windows 身份验证体系结构
<!-- more -->
https://docs.microsoft.com/zh-cn/windows-server/security/windows-authentication/windows-authentication-architecture

身份验证是指系统验证用户的登录或登录信息时所使用的过程。 用户的名称和密码与授权列表进行比较，如果系统检测到匹配项，则将访问权限授予该用户的权限列表中指定的范围。

作为可扩展体系结构的一部分，Windows Server 操作系统将实现一组默认身份验证安全支持提供程序(SSP)，其中包括 Negotiate、Kerberos 协议、NTLM、Schannel （安全通道）和Digest(摘要)。 

**authn_lsa_architecture_client**
![authn_lsa_architecture_client](https://docs.microsoft.com/zh-cn/windows-server/security/media/credentials-processes-in-windows-authentication/authn_lsa_architecture_client.gif)


这些提供程序使用的协议允许用户、计算机和服务进行身份验证，并且身份验证过程使授权的用户和服务能够安全地访问资源。
 
## 0x01 本地安全机构 LSA 

本地安全机构（LSA）是一个受保护的子系统，用于对用户进行身份验证并登录到本地计算机。 

此外，LSA 还维护有关计算机上的本地安全所有方面的信息（这些方面统称为本地安全策略）。 它还提供了用于在名称和安全标识符（Sid）之间进行转换的各种服务。

LSA管理本地安全策略、管理审计策略和设置、为用户生成包含SID和组权限关系的令牌。

LSA验证的过程: LSA通过访问本地SAM(Security Accounts Manager)数据库,可以完成本地用户的验证。

**LSA的处理流程** 

    1. LSA首先会把身份凭据交给SSPI,由该接口负责与Kerberos和NTLM服务沟通。
    2. SSPI不能确定用户是本地登录还是域账户进行域登录。所以他会先把身份认证请求传递到Kerberos SSP。
    3. Kerberos SSP会验证用户的登入目标是本地计算机还是域。如果是登录域,Kerberos SSP将继续处理。如果是本地计算机,即用户不是登录域,Kerberos SSP返回一个错误消息到SSPI,交回给GINA处理,使服务器登录不可用。
    4. SSPI现在发送请求到下一个安全提供程序——NTLM。NTLM SSP会将请求交给Netlogon服务针对LSAM (Local Security Account Manager,本地安全账户管理器)数据库进行身份认证。使用NTLM SSP的身份认证过程与Windows NT系统的身份认证方法是相同的。   


## 0x02 身份验证安全支持提供程序 SSP & SSPI

SSPI(Security Support Provider Interface) 安全支持提供接口

    是通用安全服务 API （GSSAPI）的实现，也是 Windows 定义的一套接口，此接口定义了与安全有关的功能函数， 用来获得验证、信息完整性、信息隐私等安全功能，但是没有具体的实现。

SSP(Security Support Provider)直译为安全支持提供者，又名Security Package，SSPI 的实现者，对SSPI相关功能函数的具体实现。

简单的理解为SSP就是一个DLL，来实现身份验证等安全功能，实现的身份验证机制是不一样的，例如：

    NTLM SSP    ( Challenge/Response 验证机制 )
    Kerberos SSP    ( 基于 ticket 的身份验证机制 )
    Negotiate SSP (协商安全支持提供程序)
    Secure Channel (Schannel) SSP
    Digest SSP (摘要式安全支持提供程序)
    Credential (CredSSP 凭据安全支持提供程序 )

所以可以编写自己的 SSP，然后注册到操作系统中。例如mimikatz的SSP(mimilib.dll),通过mimilib伪造SSP记录明文密码。

    copy mimilib.dll C:\windows\System32\
    HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa\
        Security Packages
            kerberos
            msv1_0      
            schannel
            wdigest
            tspkg
            pku2u
            mimilib.dll (编辑键Security Packages添加)
    域控重启后在C:\windows\System32\可看到新生成的文件kiwissp.log (这就是记录的密码)
    或者
    mimikatz同时还支持通过内存更新ssp，这样就不需要重启再获取账户信息
        privilege::debug
        misc::memssp

**SSPI Layer**

https://docs.microsoft.com/zh-cn/windows-server/security/windows-authentication/security-support-provider-interface-architecture

    Application Layer                           SSPI Layer      SSP Layer
    Application  RPC                        }                   {   kerberos   
    .Net Application / .Net Framework       }<->     SSPI   <-> {   NTLM        
    Internet Explorer                       }                   {   Digest
                                                                {   Schannel
                                                                {   Negotiate
                                                                {   Other




### 0x02-1  msv - NTLM SSP

https://github.com/gentilkiwi/mimikatz/blob/master/mimikatz/modules/sekurlsa/kuhl_m_sekurlsa.c

    msv1_0.dll     身份验证包 Lists LM & NTLM credential

NTLM 身份验证是 Windows msv1_0.dll 中包含的一系列身份验证协议。 

    NTLM 身份验证协议包括 LAN Manager 版本 1 和 2 以及 NTLM 版本 1 和 2。 
    NTLM 身份验证协议基于质询/响应机制对用户和计算机进行身份验证，该机制向服务器或域控制器证明用户知道与帐户关联的密码。 
    在使用 NTLM 协议时，每当需要新的访问令牌时，资源服务器必须执行以下操作之一来验证计算机或用户的身份：
        如果计算机或用户的帐户是域帐户，请联系域控制器的部门域认证服务来获取该帐户的域。
        如果该计算机或用户的帐户是本地帐户，请在本地帐户数据库中查找该帐户。

NTLM SSP
NTLM 安全支持提供程序（NTLM SSP）是安全支持提供程序接口（SSPI）使用的一种二进制消息传递协议，可用于实现 NTLM 质询-响应身份验证以及协商完整性和机密性选项。 NTLM SSP 包括 NTLM 和 NTLM 版本2（NTLMv2）身份验证协议。
受支持的 Windows 操作系统可将 NTLM SSP 用于以下内容：

    客户端/服务器身份验证
    打印服务
    使用 CIFS （SMB）进行文件访问
    安全远程过程调用服务或 DCOM 服务
    Location：%windir%\Windows\System32\msv1_0.dll

Microsoft为本地计算机登录提供 MSV1_0 身份验证包,不需要自定义身份验证.本地安全机构(LSA)调用MSV1_0身份验证包来处理GINA为Winlogon登录过程收集的登录数据. 

MSV1_0程序包检查本地安全帐户管理器(SAM)数据库,以确定登录数据是否属于有效的安全主体,然后将登录尝试的结果返回给LSA.

MSV1_0还支持域登录. MSV1_0使用传递身份( pass-through )验证处理域登录.

	在传递身份(pass-through)验证中,MSV1_0的本地实例使用Netlogon服务来调用在域控制器上运行的MSV1_0实例.
	然后,域控制器的MSV1_0实例检查域控制器的SAM数据库,并将登录结果返回到本地计算机上的MSV1_0实例. 
	本地MSV1_0将登录结果转发到本地计算机上的LSA.(如果域控制器不可用,并且LSA缓存了用户证书, MSV1_0的本地实例就可以使用缓存的数据来验证登录数据.)

Authentication packages是一些执行认证检查的DLL。

	Kerberos 是用于域登陆的 Windows Authentication package.
	MSV1_0是用于交互式登陆到本地计算机的 Windows Authentication package.(默认)

交互式登录 (登录本机,没有域)
本地计算机登录身份验证过程:

	0-> 用户按下Ctrl+Alt+Del组合键，Winlogon 检测到用户按下SAS组合键，调用GINA（msgina.dll）显示登录对话框，以便输入账号密码。
	1-> GINA为Winlogon登录过程收集的登录数据通过 Secur32.dll 传递到LSA, LSA 使用 LsaLogonUser API 为各种各样的用户身份验证 ( LsaLogonUser 支持交互式登录 / 服务登录 / 网络登录 ).
	 2-> (登录本机情况下)LSA(Secur32.dll LsaLogonUser API)默认调用 MSV1_0.dll 身份验证包,将用户信息处理后生成一个密钥，同SAM数据库中存储的密钥进行对比。 
	  3-> 请求LSA 里面的 Samsrv.dll 向HKLM\SAM检查( 使用注册表中的 SAM 数据库进行存储,所以要通过注册表中的HKLM\SAM检查 )
	   4-> HKLM\SAM ( 若登陆数据是属于有效的安全主体 ) 将结果返回给 MSV1_0 .
	    5-> MSV1_0 为该登录用户会话生成 本地唯一标识符LUID ( locally unique identifier /SID(Security Identifier)/组SID)将结果返回给 LSA (LSASS.exe ).
	     6-> LSA 调用Lsass.exe来创建此登录会话,将 LUID与此会话关联起来 ,将收到的SID信息为该用户创建一个安全访问令牌 ( 用户的安全标识SID(Security Identifier)/组SID/和分配的特权(特权是由LUID 对象来标识的))
	      7->  然后将令牌的句柄和登录信息发送给Winlogon，完成整个登录过程。

交互式登录 (登录本机,有域)

    若存在域,MSV1_0 使用传递身份(pass-through)验证处理域登录:
	1-> 本地MSV1_0 使用Netlogon服务来调用在域控制器上运行的MSV1_0.
		2-> 域控制器的MSV1_0实例检查域控制器的SAM数据库(上面的3步骤),并将登录结果返回到本地计算机上的MSV1_0. 

交互式登录 (登录域)	
	
    身份验证过程
	0-> 用户按下Ctrl+Alt+Del组合键，Winlogon 检测到用户按下SAS组合键，调用GINA显示登录对话框，以便输入账号密码。
	 1-> GINA为Winlogon登录过程收集的登录数据通过 Secur32.dll 传递到LSA, LSA 使用 LsaLogonUser API 为各种各样的用户身份验证 ( LsaLogonUser 支持交互式登录 / 服务登录 / 网络登录 ).
	  2-> (登录域情况下)LSA 将请求发送给Kerberos (Kerberos.dll)身份验证,通过散列算法，将用户信息生成一个密钥，并将密钥存储在证书缓存区中。 
	   3-> 
            `KRB_AS_REQ (Kerberos Authentication Service Request)`
            1. Client-A ---------------------------------------------------------==> KDC-AS (客户端执行散列运算加密一个时间戳,然后（包含用户证书）发送给身份验证服务(KDC-AS)) 此过程叫KRB_AS_REQ

            `KRB_AS_REP (Kerberos Authentication Service Response)`
            2. Client-A <==--------------------------------------------------------- KDC-AS (返回TGT,KDC利用自己的密钥对请求中时间戳进行解密，若解密成功(KDC-AS检查用户的信息(登录限制.组成员身份等)并创建票据授予票据TGT，且向本地LSA请求生成一个特殊的数据PAC。返回会话密钥和TGT) 此过程叫KRB_AS_REP

            `KRB_TGS_REQ (Kerberos Ticket Granting Service Request)`
            3. Client-A ----------------------------------------------------------==> KDC-TGS (Client-A使用AS返回的"会话密钥"构建访问特定服务的请求,再把AS返回的"TGT"连同请求一起发送到票据授予服务TGS) 此过程叫KRB_TGS_REP

            `KRB_TGS_REP (Kerberos Ticket Granting Service Response)`
            4. Client-A <==---------------------------------------------------------- KDC-TGS (TGS解密TGT和服务请求,并向Client-A发送一个服务票据ST(Service Ticket)

            `KRB_AP_REQ (Kerberos Application Request)`
            5. Client-A ----------------------------------------------------------==> Server-B (Client-A把服务票据ST中的服务器部分和请求一起发送到Server-B(Client-A要访问活动目录中的主机),远程服务器将直接接受该服务器票据,并不需要和KDC直接通信,因为该票据是用远程服务器和KDC共享的长期密钥加密过的)

				


### 0x02-2 kerberos

是Windows活动目录中使用的客户/服务器认证协议(windows中的认证协议有两种NTLM和Kerberos),为通信双方提供双向身份认证。

相互认证或请求服务的实体被称为委托人(principal)。参与的中央服务器被称为密钥分发中心(简称KDC)。

KDC(密钥分发中心　Key Distribution Center)有两个服务组成 : 

	1. AS 身份验证服务(Authentication Server)
	2. TGS 票据授予服务(Ticket Granting Server)
    该认证过程的实现不依赖于主机操作系统的认证,无需基于主机地址的信任,不要求网络上所有主机的物理安全,并假定网络上传送的数据包可以被任意地读取.修改和插入数据。
    在以上情况下, Kerberos 作为一种可信任的第三方认证服务,是通过传统的密码技术(如:共享密钥)执行认证服务的。

身份验证过程在上。

由于从 Windows 2000 开始，Kerberos 协议是默认的身份验证协议，因此所有域服务都支持 Kerberos SSP。 这些服务包括：

    使用轻型目录访问协议（LDAP） Active Directory 查询
    使用远程过程调用服务的远程服务器或工作站管理
    打印服务
    客户端-服务器身份验证
    使用服务器消息块（SMB）协议（也称为通用 Internet 文件系统或 CIFS）的远程文件访问
    分布式文件系统管理和引用
    Intranet 到 Internet Information Services 的身份验证（IIS）
    Internet 协议安全（IPsec）的安全机构身份验证
    用于域用户和计算机 Active Directory 证书服务的证书请求
    
### 0x02-3 tspkg

TSPKG - Terminal Services Package  身份验证提供程序
TSSSP - Terminal Services SSP  终端服务SSP
SSP - 安全支持提供程序

RDP SSO功能依赖于允许“凭据委派(Credential Delegation)”的 CredSSP / TSSSP / TSPKG 组件。

expend read

    [通过滥用CredSSP / TSPKG（RDP SSO）即凭据委派，无需管理员或通过Kekeo接触LSASS即可进行凭据盗窃](https://clement.notin.org/blog/2019/07/03/credential-theft-without-admin-or-touching-lsass-with-kekeo-by-abusing-credssp-tspkg-rdp-sso/)
 
凭据委派 Allow delegating default credentials  （类似于记住远程登陆过的密码）

	Local Computer Policy –> Computer Configuration –> Administrative Templates –> System –> Credentials Delegation
	双击打开"Allow Delegating Saved Credentials with NTLM-only Server Authentication"， 默认是 "Not configured" 选择"Enabled"
	单击 "Show"按钮，并输入"TERMSRV/*" 到列表中，单击OK保存。

同样的方法，修改如下的策略：

    Allow Delegating Saved Credentials
    Allow Delegating Default Credentials with NTLM-only Server Authentication
    Allow Delegating Default Credentials

tspkg.DLL: TSSSP (Terminal Services SSP)

    C:\Windows\System32\tspkg.dll - Web Service Security Package 

### 0x02-4 CredSSP 

配置远程登陆RDP，当选中“Network Layer Authentication”（NLA是网络级身份验证）选项时，它允许在打开图形会话之前进行身份验证。

凭据安全服务提供程序（CredSSP）提供启动新终端服务和远程桌面服务会话时的单一登录（SSO）用户体验。 

使用 CredSSP，应用程序可以根据客户端的策略，将用户的凭据从客户端计算机（通过使用客户端 SSP）委托给目标服务器（通过服务器端 SSP）。 

CredSSP 策略通过使用组策略进行配置，并且默认情况下禁用凭据的委派。

    位置：%windir%\Windows\System32\credssp.dll


### 0x02-5 Schannel

[TLS/SSL 概述（Schannel SSP）](https://docs.microsoft.com/zh-cn/windows-server/security/tls/tls-ssl-schannel-ssp-overview?redirectedfrom=MSDN)


安全通道（Schannel）用于基于 web 的服务器身份验证，例如，当用户尝试访问安全 web 服务器时。

    位置：%windir%\Windows\System32\Schannel.dll

Schannel 是安全支持提供程序 (SSP)，可实现安全套接字层 (SSL) 和传输层安全 (TLS) Internet 标准身份验证协议。

TLS 协议、SSL 协议、专用通信技术（PCT）协议和数据报传输层（DTLS）协议基于公钥加密。 Schannel 提供所有这些协议。所有 Schannel 协议均使用协议使用客户端/服务器模型，且基于需要公钥基础结构的证书身份验证。

对参与方进行身份验证时，Schannel SSP 按以下优先顺序选择协议：

    传输层安全性（TLS）版本1.0
    传输层安全性（TLS）版本1.1
    传输层安全性（TLS）版本1.2
    安全套接字层（SSL）版本2.0
    安全套接字层（SSL）版本3.0
    专用通信技术（百分比,默认PCT 处于禁用状态）

选择的协议是客户端和服务器可以支持的首选身份验证协议。 例如，如果服务器支持所有 Schannel 协议，而客户端仅支持 SSL 3.0 和 SSL 2.0，则身份验证过程使用 SSL 3.0。

管理网络时的一个问题是保护跨未受信任的网络在应用程序之间发送的数据的安全。 

你可以使用 TLS 和 SSL 对服务器和客户端计算机进行身份验证，然后使用协议对经过身份验证的参与方之间的消息进行加密。
例如，你可以将 TLS/SSL 用于：

    电子商务网站受 SSL 保护的交易
    对受 SSL 保护的网站的经身份验证的客户端访问
    远程访问
    SQL 访问
    电子邮件


### 0x02-6 WDigest

Digest 与 NTLM 类似也一种挑战认证的协议，用于轻型目录访问协议（LDAP）和 web 身份验证(IIS)。 摘要式身份验证通过网络以 MD5 哈希或消息摘要形式传输凭据。

Digest SSP （Wdigest）用于以下内容：

    Internet Explorer 和 Internet Information Services （IIS）访问
    LDAP 查询
    位置：%windir%\Windows\System32\Digest.dll

挑战认证的基本流程就是：

    客户端访问服务端发起认证
    服务端返回一个随机值
    客户端利用自己知道的密码或一些其他信息来对这个随机值做一些计算，得运算结果 response，将 response 发送至服务端
    服务端利用同样的方法也计算出一个 response，并将自己计算出的这个与客户端发送过来的 response 进行对比，如果一致，则证明客户端确实是知道密码，认证成功。

所以，在用户进行了交互式登陆后，为了实现 Digest 认证机制的 SSO，Digest SSP 只能将密码加密后保存在内存里面，在有任何客户端软件需要与远程服务器进行 Digest 认证的时候解密还原成明文用于计算 Digest response。

从 Windows 2003 开始，引入了Advanced Digest（就是 Digest 的升级版）。

wdigest.dll 实现的就是 Advanced digest，而不再是老的 digest 认证协议了。其中一项改变就是，它允许不再存储账号的明文密码。

    比如域的 FQDN 为 aa.COM，域账号为 user01，密码为 123456。那么域控并不会保存 user01 账号的明文密码 123456，而是将 MD5(user01:aa.COM:123456) 后的29种HASH保存在域数据库中，这样就绕过了需要明文密码的问题。

    使用 mimikatz 的 dcsync 或其他功能看到的一大堆 Wdigest 的 Hash，就是这 29 种 Hash。而如果给账号开了 Reversible Encryption ，则 dcscync 会将明文密码读出来

Reversible Encryption （可逆加密）

    默认情况下，域里的所有账号都是关闭这个功能的（在没有 Advanced digest 的时候，关闭了这个功能的域账号将不能使用 digest 进行认证）。所以，域数据库里面默认只保存了密码的 Hash 而没有保存密码本身。

开启 Reversible Encryption 的三种方法：

    在域控的组策略里面对所有域成员开启，此时除了 HASH 外，所有域成员的明文密码都会以一种可逆加密的形式保存在域数据库中。以可逆加密形式存储的明文密码，跟直接存储明文没有什么区别。（Group Policy -> Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Store password using Reversible Encryption √ )

    在域用户管理里面，单独为某些用户启用 Reversible Encryption 功能。启动了此功能的用户会在它们的 UserAccessControl 属性里面留下标志。（右键用户属性-> Account -> Store password using Reversible Encryption √ )

    利用多元密码策略（Fine-Grained Password Policies）启用 (Windows Server 2008 or higher)

    [Dump Clear-Text Passwords for All Admins in the Domain Using Mimikatz DCSync](https://adsecurity.org/?p=2053)

禁用WDigest

    HKLM\System\CurrentControl\SetControl\LsaSecurityPack中删除WDigest，然后重新启动服务器 (Windows Server 2008)
    HKLM\System\CurrentControlSet\Control\SecurityProviders\WDigest中创建/设置UseLogonCredential= dword：00000000 (要明文形式保存在 LSA 内存中则设为1)
    Windows Server 2012 R2-2016：默认情况下禁用WDigest，不执行任何操作


Mimikatz中sekurlsa::wdigest 实现思路

    1.提升至Debug权限。
    2.获得lsass进程句柄。
    3.枚举lsass进程中全部模块的句柄，定位wdigest.dll和lsasrv.dll在内存中的位置。
    4.从lsasrv.dll中获得InitializationVector，AES和3DES的值，用于解密。
    5.从wdigest.dll中获得每条凭据的信息，判断加密算法，解密获得明文口令。
    (Windows Server 2008 R2及更高版本的系统需要修改注册表)
    位置：%windir%\Windows\System32\kerberos.dll

### 0x02-7 Negotiate - SPNEGO

简单且受保护的 GSS-API 协商机制（ SPNEGO ）构成协商 SSP 的基础，whichcan 用于协商特定身份验证协议。

当应用程序调用 SSPI 登录到网络时，它可以指定 SSP 来处理请求。 如果应用程序指定 Negotiate SSP，则它会分析请求，并根据客户配置的安全策略选择相应的提供程序来处理请求。

在支持的 Windows 操作系统版本中，协商安全支持提供程序在 Kerberos 协议和 NTLM 之间选择。 

默认情况下，协商将选择 Kerberos 协议，除非该协议不能由身份验证中涉及的系统之一使用，或调用应用程序没有提供足够的信息来使用 Kerberos 协议。

    位置：%windir%\Windows\System32\lsasrv.dll

### 0x02-8 PKU2U

Windows 7 和 Windows Server 2008 R2 中引入了 PKU2U 协议，并将其作为 SSP 实现。 

此 SSP 启用对等身份验证，特别是在 Windows 7 中引入了名为 "家庭组" 的媒体和文件共享功能。 此功能允许在非域成员的计算机之间共享。

    位置：%windir%\Windows\System32\pku2u.dll

### 0x02-9 Exchange 认证

身份验证是 Exchange Web Services (EWS) 应用程序的关键部分。

https://docs.microsoft.com/zh-cn/exchange/client-developer/exchange-web-services/authentication-and-ews-in-exchange

Exchange 提供以下身份验证选项：

    OAuth 2.0 (Exchange Online 仅 ， OAuth2.0应用于Office 365应用程序接口)
    NTLM （Exchange 内部部署仅）
    Basic （基本身份验证 - 不再推荐）


## 0x03 Windows 身份验证中的凭据进程

Windows 凭据管理是操作系统从**服务或用户接收凭据**的过程，并保护该信息以供将来呈现到身份验证目标。 

对于已加入域的计算机，身份验证目标为域控制器。身份验证中使用的凭据是将用户标识关联到某种形式的身份验证（如证书、密码或 PIN）的数字文档。

默认情况下，将根据本地计算机上的安全帐户管理器（SAM）数据库或通过 Winlogon 服务在加入域的计算机上 Active Directory 验证 Windows 凭据。 

凭据是通过登录用户界面上的用户输入收集的，也可以通过应用程序编程接口（API）以编程方式通过向身份验证目标提供。

本地安全信息存储在HKEY_LOCAL_MACHINE\SECURITY下的注册表中。 存储的信息包括策略设置、默认安全值和帐户信息，如缓存的登录凭据。 SAM 数据库的副本也存储在此处，尽管它是写保护的。




## 0x04 Windows 身份验证中使用的组策略设置

https://docs.microsoft.com/zh-cn/windows-server/security/windows-authentication/group-policy-settings-used-in-windows-authentication

