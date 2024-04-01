---
title: windows 域环境下认证和攻击初识
url: https://mp.weixin.qq.com/s?__biz=MzAwMzYxNzc1OA==&mid=2247486939&idx=1&sn=5df5d91671322fa34e7db4e0b8374ff7&chksm=9b392b6aac4ea27c05ad3cbfeed07343bd622eeeec98a2d2eda914186a8284c905f95a5da3e5&mpshare=1&scene=1&srcid=0829WMA5BVu0cmt3E9V4zqiz&sharer_sharetime=1693306427937&sharer_shareid=0d85edf17b8a6f1974b95685bf30e025#rd
clipped_at: 2024-04-01 17:13:27
category: default
tags: 
 - mp.weixin.qq.com
---


# windows 域环境下认证和攻击初识

**Kerberos 认证原理**

Kerberos 是一种认证机制。目的是通过密钥系统为客户端 / 服务器应用程序提供强大的可信任的第三方认证服务：保护服务器防止错误的用户使用，同时保护它的用户使用正确的服务器，即支持双向验证。kerberos 最初由 MIT 麻省理工开发，微软从 Windows 2000 开始支持 Kerberos 认证机制，将 kerberos 作为域环境下的主要身份认证机制，理解 kerberos 是域渗透的基础。

  

**kerberos 认证框架**

kerberos 机制中主要包含三个角色：Client、Server、KDC (Key Distribution Center) 密钥分发中心。Client 代表用户，用户有自己的密码，Server 上运行的服务也有自己的密码，KDC 是受信任的三方认证中心，它拥有用户和服务的密码信息。KDC 服务默认会安装在域控中，Client 想要访问 Server 的服务（xxx service），前提是通过 KDC 认证，再由 KDC 发放的票据决定 Client 是否有权限访问 Server 的服务。框架图如下：

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-3320b7a30e9c5ba162b61b94dcb25a86.webp)

  

**kerberos 认证术语初识**

KDC (Key Distribution center)：密钥分发中心，在域环境中，KDC 服务默认会安装在域控中。

AS (Authentication Service)：认证服务，验证 client 的 credential (身份认证信息)，发放 TGT。

TGT (Ticket Granting ticket)：票据授权票据，由 KDC 的 AS 发放，客户端获取到该票据后，以后申请其他应用的服务票据 (ST) 时，就不需要向 KDC 的 AS 提交身份认证信息 (credential)，TGT 具有一定的有效期。

TGS (Ticket Granting Service)：票据授权服务，验证 TGT，发放 ST。

ST (Service Ticket)：服务票据，由 KDC 的 TGS 发放，是客户端应用程序访问 Server 某个服务的凭证，Server 端验证通过则完成 Client 与 Server 端信任关系的建立。

先由简到繁地去梳理以上术语的关系。首先 Client 想要访问 Server 的某个服务，就需要通过 KDC 的认证，获取到服务票据（ST），服务会验证服务票据（ST）来判断 Client 是否通过了 KDC 认证。为了避免 Client 每次访问 Server 的服务都要向 KDC 认证 (输入密码)，KDC 设计时分成了两个部分，一个是 AS，另一个是 TGS，AS 接收 Client 的认证信息，认证通过后给 Client 发放一个可重复使用的票据 TGT，后续 Client 使用这个 TGT 向 TGS 请求 ST 即可。

Authenticator：验证器，不能重复使用，与票据（时效内能重复使用）结合用来证明 Client 声明的身份，防止票据被冒用。

  

**windows 域 kerberos 认证流程**

##### 第一步 AS 认证 (获取 TGT)

**请求**：Client 向 KDC 的 AS 发起认证请求，身份认证信息包含了用户密码 hash (user\_hash) 加密的 timestamp 预认证信息 pre-authentication data，以及用户名 (user)、客户端信息 (client info)、服务名 (krbtgt) 等未加密信息。

**生成 session key 以及 TGT**：域控中存储了域中所有用户密码 hash（user\_hash），KDC 的 AS 依据用户名查找相应的 user\_hash，成功解密预认证信息，验证客户端通过，然后会生成一个 sessionkey-TGS (后续用于加密 Client 与 TGS 通信)，以及 TGT (由 krbtgt hash 加密的 sessionkey-TGS、user、client info、lifetime、timestamp 等信息)。

注：krbtgt 账户是创建域时系统自动创建的，可以认为是为了 kerberos 认证服务而创建的账号。

注：TGT 是 KDC 加密的，Client 无法解密，并且具有有效期，客户端用其向 TGS 请求 ST。

**响应**：AS 用 user\_hash 加密 sessionkey-TGS，与 TGT 一起生成 REP 响应发送给客户端。客户端解密响应成功说明数据包是 KDC 发送来的，并且获得 sessionkey-TGS 以及 TGT，sessionkey-TGS 用于后续加密通信。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-35abe519e145793fa25c6737fe0ea7ee.webp)

##### 第二步 TGS 认证 (获取 ST)

通过第一步，客户端解密 AS 的响应后，可以得到一个 sessionkey-TGS 以及 TGT。

**请求**：用户想访问 Aservice 服务，于是向 TGS 请求访问 Aservice 的 ST。首先客户端会生成验证器 Authenticator，内容包含 user、client info、lifetime、timestamp 信息，并且用 sessionkey-TGS 加密。客户端将验证器、Aservice 信息、TGT 发送给 TGS 请求 ST。

**生成 session key 以及 ST**：TGS 收到请求，利用 krbtgt hash 解密 TGT，获取到 sessionkey-TGS，user、client info 等信息，然后利用 sessionkey-TGS 解密验证器，校验验证器和 TGT 中的 user 信息，如果一致，则说明该请求符合 TGT 中声明的用户，该用户是通过 AS 认证的。然后 TGS 会为用户 user 和服务 Aservice 之间生成新的 session key sessionkey-Aservice，并用 sessionkey-TGS 加密 sessionkey-Aservice。再生成一个 ST，内容包含 user、client info、lifetime、timestamp、sessionkey-Aservice，ST 用 Aservice 的 service\_hash 加密。

注：验证器 Authenticator 只能使用一次，是为了防止 TGT 被冒用。kerberos 设计之初，产生票据的概念就是为了避免重复的常规密码验证，因为票据在有效期内可以重复使用。为了避免冒用，设计出 session key 以及 Authenticator。session key 只有真正的客户端、服务知道，利用 session key 加密验证器，服务就可以解密对比验证器以及票据中声明的用户、客户端信息是否一致，一致说明票据来自可信客户端。

**响应**：TGS 将 sessionkey-TGS 加密后的 sessionkey-Aservice 以及 service\_hash 加密的 ST 响应给客户端。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-affe687d72d6dc597e94cbd9cde8698f.webp)

##### 第三步 服务认证

通过第二步，Client 获取到 sessionkey-Aservice 以及 ST，接下来 Client 利用 sessionkey-Aservice 加密 Authenticator，连同 ST 去请求 Server 的 Aservice。Aservice 利用自己的 service\_hash 解密 ST，获得 sessionkey-Aservice，再解密 Authenticator 验证 Client 声明的 user 信息，通过认证后 Aservice 还需要用 sessionkey-Aservice 加密一段信息返回给 Client，Client 利用 sessionkey-Aservice 解密成功说明 Aservice 用自己 service\_hash 成功解密出了 sessionkey-Aservice，是可信服务端。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-3d5dfbec199349ff491f0fc8d132a856.webp)

至此，kerberos 认证流程完成，Client 可访问 Aservice 提供的服务。

  

**NTLM 认证**

NTLM 认证采用质询 / 应答 (Challenge/Response) 的消息交换模式。NTLM 既可用于域环境下的身份认证，也可以用于没有域的工作组环境。主要有本地认证和网络认证两种方式。

**本地认证：**

用户登陆 windows 时，windows 首先会调用 winlogon.exe 进程接收用户输入的密码，之后密码会被传递给 lsass.exe 进程，进程会先在内存中存储一份明文密码，并将密码加密为 NTLM hash，与本地 SAM 数据库中用户的 NTLM hash 对比，一致则登陆成功。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-285ea77e12b646d61f7a40036bb16f71.webp)

  

**网络认证：**

如下为 NTLM 域环境中网络认证流程。

第一步：首先用户输入正确用户密码登陆到客户端主机，用户想要访问某个服务器的服务，客户端先发送一个包含用户名明文的数据包给服务器，发起认证请求。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-de79651b6a8dc6bf802a2317c2321068.webp)

第二步：服务器生成一个随机数，称为 Challenge，返回给客户端。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-3d2d8e8b16db9fc9e82038a4ccf20b66.webp)

第三步：客户端接收到 Challenge 后，用密码 hash 加密，生成 Response，发送给服务。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-c8c716bbf212d5094738f47f74a866a5.webp)

第四步：服务将 Response、用户名、Challenge 发送给域控验证。域控使用本地数据库 (NTDS.dit) 中保存的对应用户的 NTLM hash 对 Challenge 进行加密，得到的结果与 Response 进行对比，一致则认证成功。然后将认证结果返回给服务端。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-d11c55179f184a23ca22d2b706525910.webp)

  

**相关攻击基础**

#### windows 下的用户密码 hash

windows 系统下的用户密码 hash 通常指的是 Security Account Manager 中保存的用户密码 hash，也就是 SAM 文件中的 hash，mimikatz 读取出已登录用户的 NTLM hash 都是同一个 hash，域控中 NTDS.dit 的 hash。如下密码均为 Aa123456，都是 NTLM hash 值。(以下操作均需以管理员权限执行)

###### SAM 中的 hash

先导出 sam，mimikatz 读取 (本地用户 ate/Aa123456)。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-982da41bfe0d670e8ecd583954ff9fbf.webp)

mimikatz 读取。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-232a03ff1d85c1fabba9d36824ce9079.webp)

###### mimikatz 从内存 dump 出的 hash

如下，cmd 运行 mimikatz.exe，在 mimikatz 会话中执行 privilege::debug 和 sekurlsa::logonpasswords。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-b13016875057375df168e073a85fa7ab.webp)

testdomain\\test1 密码 Aa123456 的 hash。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-bf37311d0feef340e5159bb626d76819.webp)

###### 域控中 NTDS.dit 的 hash

如下，testdomain\\test1 密码 Aa123456 的 hash。域中先利用 ntdsutil 导出 NTDS.dit，SYSTEM 和 SECURITY 文件。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-60e6888a1582fae2120456b3f9886664.webp)

导出文件的位置。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-b22adf7daf380789dd2824188296a092.webp)

利用 NTDSDumpEx 查看，如下。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-dc732088dabb2a26162261ce472707b9.webp)

#### PTH

通过前面的内容，可以看到 kerberos、NTLM 认证过程的关键，首先就是基于用户密码 hash 的加密，所以在域渗透中，无法破解用户密码 hash 的情况下，也可以直接利用 hash 来完成认证，达到攻击的目的，这就是 hash 传递攻击（Pass The Hash）。如下，192.168.39.100 为域控的地址，192.168.39.133 为登陆过域管理账号的终端，获取到了域管理的 hash，在 192.168.39.133 模拟 pth 来接管域控。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-8b9799cb5f88c9ba2b224edbf5ac8046.webp)

攻击成功后获取到一个 shell，虽然是本机的，但可以操控域控，如下：

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-214c1dc25dc3276f2ba3f84f3034484d.webp)

#### SPN

SPN 是指服务主体名称 (Service Principal Names)，就是一个具体的服务在域里的唯一标识符，服务要使用 kerberos 认证，就需要正确配置 SPN，服务可以使用别名或者主机名称向域注册 SPN，注册完成后，可在域控使用 ADSI 编辑器连接到 LDAP 目录，查看服务的 SPN。

SPN 分为两种，一种是注册在机器账户上的，一种是注册在域用户账户中的。当服务的权限为 Local System 或 Network Service，则 SPN 注册在机器帐户下。当服务的权限为一个域用户，则 SPN 注册在域用户帐户下。

比如，域控机器 (也是一个机器账户) 里的 DNS 服务（用 ADSI 编辑器连接 LDAP 查看）。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-2f49d8f4469a0d0fd48985e21091dad7.webp)

域用户可向域控 LDAP 目录查询 SPN 信息，从而获取到域内安装了哪些服务。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-7d6da5403405afa345b5beded19d37b7.webp)

抓包可以看到是通过 LDAP 协议查询获得 SPN 信息。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-a6cb5c4c361b7ff14082cc4aec281cf4.webp)

通过 SPN 查询的方式发现域内的服务相比端口扫描更为隐蔽，但是也有缺陷，可能漏掉一些未注册的服务。

  

**黄金票据和白银票据**

###### 黄金票据

黄金票据 (Golden Ticket) 是可换取任意服务票据 (ST) 的票据授权票据 (TGT)，前面 kerberos 认证原理提到 TGT 是由域控 krbtgt 的密码 Hash 加密的，所以伪造金票的前提是控制了域控。

伪造金票需要域名、域 sid、krbtgt 的密码 hash。如下在域控获取 krbtgt hash。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-5609c7316868c97666b6c1ced446a84e.webp)

在 mimikatz.log 中找到其 NTLM hash。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-3a1b7009bf992d76389c48fdd87d9fd5.webp)

用普通用户伪造金票并访问域控，获取域 sid，注意不包含最后 - xxxx。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-743f7dfc34867d92b0fce1dc773ecc5f.webp)

/user 指定伪造用户名，/domain 指定域，/sid 指定 sid，/krbtgt 指定 krbtgt hash，/ptt 直接将票据导入内存。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-8b34dc9c20edd849f219edceee103683.webp)

成功之后可访问域控 C 盘，注意要用主机名 (如下 WIN-xxxxx) 而不是 IP。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-4baceaa7fb1b60ac4e196be2ee3324f1.webp)

###### 白银票据

白银票据 (Silver Tickets) 是指伪造的服务票据 (ST)，只能用来访问特定的服务，通过 kerberos 的认证原理得知 ST 是由 TGS 颁发的，使用了服务的密码 hash 加密，所以在伪造银票的时候需要知道服务的密码 hash。下面通过创建 LDAP 银票访问域控 LDAP 服务来演示银票的伪造和利用。

域控的 LDAP 服务是由网络服务账户运行的，其对应 sid 是 S-1-5-20，域控上通过 mimikatz 获取 hash，执行 mimikatz.exe log privilege::debug sekurlsa::logonpasswords exit。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-4eac74125ca57f559bc3c513e5659b01.png)

普通用户伪造银票并导入内存获取权限，可取到域控 krbtgt hash。/target 指定服务主机名，/rc4 指定服务密码的 hash，/service 指定服务，如下。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962807-6cf743c921e3ea29509da67565bac644.webp)

  

参考：

```plain
https://www.cnblogs.com/felixzh/p/9855029.html
https://blog.csdn.net/dog250/article/details/5468741
https://tools.ietf.org/html/rfc4120.html
https://blog.csdn.net/qq_18501087/article/details/101593642
https://blog.csdn.net/weixin_30532987/article/details/96203552
https://support.microsoft.com/en-au/help/243330/well-known-security-identifiers-in-windows-operating-systems
```
