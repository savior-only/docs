---
title: 如何完美的在渗透测试中做好信息收集？ - FreeBuf 网络安全行业门户
url: https://www.freebuf.com/articles/web/255006.html
clipped_at: 2024-04-01 17:42:15
category: default
tags: 
 - www.freebuf.com
---


# 如何完美的在渗透测试中做好信息收集？ - FreeBuf 网络安全行业门户

**信息收集**

搞渗透的人应该都清楚，信息收集对于渗透测试来说是非常重要的，我们手上掌握的目标的信息越多，成功渗透的概率就越大，而信息收集又分为两类。

第一类：主动信息收集：通过直接访问、扫描网站，这种流量将流经网站

第二类：被动信息收集：利用第三方的服务对目标进行访问了解，比例：Google 搜索、Shodan 搜索等

正所谓知己知彼百战百胜，下面就来介绍一些信息收集的常用手段。

## **域名**

### **域名注册人信息收集**

访问 https://www.chinaz.com/  站长之家这个网站

![1599268779.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-69b8711827f9013b956d1997ef4f9206.png)

![1599268793.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-e5d5220c179e3d77be09eb5c6d4859d4.png)

大家还可以点击上面的 whois 反查，查看更多信息。

### **子域名爆破**

**1、利用工具如（wydomain、layer 子域名挖掘机、dnsenum）**

这里演示一下 layer 子域名挖掘机使用，工具大同小异

下载链接链接：https://pan.baidu.com/s/126WRnM2du8nm5LUJLWTTbg 提取码：cy0w

打开软件

![1599268817.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-9418cb3e9428e2c630e32306267c1f11.png)![1599268827.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-3771a19fd1794b2e80ece786be8ef1df.png)

**2、域传送漏洞**

什么是 DNS 域传送

1、DNS（域名管理系统）万维网重要基础，建立在一个分布式数据库基础上，数据库里保存了 ip 地址和域名的相互映射关系。

用户在浏览器输入域名，浏览器将向 DNS 服务器发送查询，得到目标主机 ip 地址，再与对应的主机建立 http 链接，请求网页。

常用 DNS 记录

```plain
A 记录         IP 地址记录，记录一个域名对应的 ip 地址
NS 记录		域名服务器记录，记录该域名由哪台域名服务器解析
PTR 记录		反向记录，从 ip 地址到域名的一条记录
MX 记录		电子邮件交换记录，记录一个邮件域名对应的 ip 地址
```

2、域传送：DNS Zone Transfe

DNS 服务器分为：主服务器、备份服务器和缓存服务器。  
域传送是指后备服务器从主服务器拷贝数据，并用得到的数据更新自身数据库。  
在主备服务器之间同步数据库，需要使用“DNS 域传送”。

**探测漏洞方法  
**

nslookup 探测漏洞

输入 nslookup 命令进入交互式 shell;

server 命令参数设定查询将要使用的 DNS 服务器；

ls 命令列出某个域中的所有域名；

exit 命令退出；

![1599268855.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-0be265b5e557312d9cf34f2ff5df14a5.png)当然这里只是使用方法，我测试的网站不存在域传送漏洞，所以爆出的是这个，大家可以自己去 fofa、钟馗之眼等去搜索一下，然后测试一下。

![1599268877.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-b3bd5fac44be9322d16f6f22028e4b0e.png)

### **判断网站是否存在 CDN，若存在的绕过方式  
**

**什么是 CDN  
**

CDN 的全称是 Content Delivery Network，即内容分发网络。CDN 是构建在现有网络基础之上的智能虚拟网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN 的关键技术主要有内容存储和分发技术。通俗点就是一种缓存技术，提高用户上网体验，但是 cdn 对渗透测试者的渗透工作就有一定阻碍，所以我们要判断 cdn 的存在与否，和绕过 cdn。

**检测是否存在 cdn 的方法**

还是利用站长之家这个网站，如下图

点击站长工具![1599268901.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-f1c04caed74782b44953ac74eafb75f9.png)

输入网址执行 ping 检测![1599268925.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-ef5b7bb5d7d128b4cbf082767a78a58d.png)原理是实现多地 ping 一个网址，假如 ping 出来的 ip 地址都一样那么将不存在 cdn

上图是不存在 cdn 的情况。

下面测试一下百度

![1599268983.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-f4726ef57865509da5f677c1e9645d06.png)

ip 地址不一样，证明是存在 cdn 的。

**存在 cdn 的绕过方法**

请参考如下网址

> https://www.cnblogs.com/qiudabai/p/9763739.html
> 
> https://www.cnblogs.com/xiaozi/p/12963549.html
> 
> https://blog.csdn.net/qq\_36119192/article/details/89151336

**一、google hack 语法  
**

**googlehack 常用语法**

site       指定域名

intext   正文中存在关键字的网页

intitle   标题中存在关键字的网页

info      一些基本信息

inurl    URL 存在关键字的网页

filetype 搜索指定文件类型

**google 语法利用实例  
**

1、​site:baidu.com# 收集百度子域名

![1599269648.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-c0b1b09cbc77e0ae2d810d4a39570262.png)

2、intitle: 管理登录  #查找后台管理登陆界面

![1599270628.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-df187a705017e8484504e64cf04191ed.png)

3.filetype:php   #查找 php 类型主页

![1599270669.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-92160df43250e17ffbfc361164c644ac.png)

4、inurl:file   #查找 url 上含 file 的网址寻找上传漏洞​

![1599271023.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-7caca482c64a9e830ea6b4b16d4e9c0d.png)

当让这些语法还可以一起组合使用起到更大的作用，如下

site:xx.com filetype:txt 查找 TXT 文件 其他的以此类推

查找后台

site:xx.com intext: 管理

site:xx.com inurl:login

site:xx.com intitle: 后台

查看服务器使用的程序

site:xx.com filetype:asp

site:xx.com filetype:php

site:xx.com filetype:jsp

site:xx.com filetype:aspx

查看上传漏洞

site:xx.com inurl:file

site:xx.com inurl:load

查找注射点

site:xx.com filetype:asp

**二、网络组件搜索引擎  
**

网站如 shodanhq.com、zoomeye.org、www.fofa.so

**这里语法以 fofa 为例  
**

**1、搜索页面标题中含有“后台管理”关键词的网站和 IP  
**

title="后台管理"

![1599649260.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-cedced23a0e0707966de5c0bd8f96e21.png)

**2、搜索 HTTP 响应头中含有“flask”关键词的网站和 IP  
**

![1599649270.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-f3bceafd6dfd8f74fffe95acd9c998b4.png)

**3、搜索根域名中带有“baidu.com”的网站  
**

![1599649280.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-71b733e0cf9660ec9ecfb339ab343108.png)

**4、搜索域名中带有 "login" 关键词的网站  
**

![1599649288.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-03701dbe74cfdecb1f7b179992b83561.png)

**5、搜索开放 3389 端口的 ip  
**

![1599649297.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-83d9874983b9c7d663b92bf100c08f09.png)

**三、github 敏感信息泄露  
**

当确定了公司后，我们可以去互联网上查询与该公司有关的任何信息。比如，公司的邮箱格式，公司的员工姓名，以及与该公司有关的任何信息。并且，我们还可以去 Github、码云等代码托管平台上查找与此有关的敏感信息，有些粗心的程序员在将代码上传至代码托管平台后，并没有对代码进行脱敏处理。导致上传的代码中有包含如数据库连接信息、邮箱密码、还有可能有泄露的源代码等。

详细语法请参考一下链接

https://www.cnblogs.com/ichunqiu/p/10149471.html

[https://blog.csdn.net/qq\_36119192/article/details/99690742](https://blog.csdn.net/qq_36119192/article/details/99690742)

**一、判断操作服务器类型**

1、利用 windows 于 linux 对大小写敏感的特性区别

Linux 操作系统大小写敏感，我们将网址 url 一些字母修改成修改大小写看网站是否还能正常访问，能访问就是 windows 服务器，不能则是 Linux。

例访问一个网站，如下图

![1600173544.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-535d32df0ea55ffdc5b9163a8d524f32.png)这是把小写 n 改为 N 看是否还能正常访问![1600173559.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-11057154ea4878b7efbd68f85f7d8dac.png)

正常访问证明服务器是 windows

2、ping 测试

通过 ping 操作系统判断 TTL 返回值来判断操作系统

TTL 起始值：Windows xp (及在此版本之前的 windows) 128 (广域网中 TTL 为 65-128)

Linux/Unix64 (广域网中 TTL 为 1-64)

某些 Unix:255

如下图![1600173578.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-9ba85e60486b495e3c61a704294608ac.png)

通过 TTL 返回值 241 可以判断对方操作系统为 windows

3、利用 nmap 工具对目标进行扫描，判断操作系统

如：nmap -O ip 地址![1600173593.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-8211e10a33dac91a8ac87e02d69689f8.png)

**二、网站指纹识别**

在渗透测试中，对目标服务器进行指纹识别是相当有必要的，因为只有识别出相应的 Web 容器或者 CMS，才能查找与其相关的漏洞，然后才能进行相应的渗透操作。这是我们想要识别这些网站的指纹，我们就可以利用以下一些指纹识别在线网站。

云悉指纹：http://www.yunsee.cn/finger.html

ThreatScan: https://scan.top15.cn/web/

WhatWeb: https://whatweb.net/

BugScaner: http://whatweb.bugscaner.com/look/![1600173613.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-96aef4ad994cf21ff30f253ba4111762.png)

### **三、网站容器、脚本类型**

可以利用 ThreatScan：https://scan.top15.cn/web/ 这个网站获取![1600173630.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-cfe0e6fcfe38142f7c18e5daee883f29.png)或者利用 wappalyzer 插件![1600173646.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-588feae90af604e4168d6917e0745136.png)

**四、敏感文件、目录信息  
**

由于发布网站时，服务器配置问题，导致目录浏览功能打开，从而引起信息泄露，造成安全隐患。在信息收集过程中，需要收集的敏感目录 / 文件信息包括：

1.robots.txt

2.crossdomin.xml

3.sitemap.xml

4\. 后台目录

5\. 网站安装包

6\. 网站上传目录

7.mysql 管理页面

8.phpinfo

9\. 网站文本编辑器

10\. 测试文件

11\. 网站备份文件 (.rar、zip、.7z、.tar.gz、.bak)

12.DS\_Store 文件

13.vim 编辑器备份文件 (.swp)

14.WEB—INF/web.xml 文件

这个时候我们可以利用一些工具对目标网站进行爬行，爬取网站目录

这里用 burpsuite 的 spider 模块进行爬取网站目录演示

首先访问一个网站![1600173662.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-656c0c7f174febdf0119c578751d3be4.png)任意点击抓包![1600173674.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-750d7d25b3121619f964201e5291d124.png)![1600173683.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-b330bf9983ff12ac778efb376b9ed9d0.png)爬行中![1600173696.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-18c02b44760d726239a78ec0dfccd339.png)爬行后再 target 模块中查看网站目录![1600173713.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-e25ac4ff0a02088450698d5b7d203311.png)

**五、端口收集  
**

利用 nmap 工具扫描![1600173725.png!small](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711964535-7b89ff5fb5d5b5b11893199834dbf153.png)

**六、社会工程学  
**

利用一些社工库等等

本文来源：公众号 猪猪谈安全；作者：随风 kali
