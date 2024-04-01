---
title: 宸极实验室 —『红蓝对抗』EwoMail 结合 Gophish 的钓鱼邮件姿势
url: https://zhuanlan.zhihu.com/p/636383100
clipped_at: 2024-04-01 17:24:00
category: default
tags: 
 - zhuanlan.zhihu.com
---


# 宸极实验室 —『红蓝对抗』EwoMail 结合 Gophish 的钓鱼邮件姿势

> 介绍：`EwoMail` 结合 `Gophish` 的钓鱼邮件姿势利用。

## **0x00 前言**

`EwoMail` 是基于 `Linux` 的开源邮件服务器软件，集成了众多优秀稳定的组件，是一个快速部署、简单高效、多语言、安全稳定的邮件解决方案，帮助你提升运维效率，降低 `IT` 成本，兼容主流的邮件客户端，同时支持电脑和手机邮件客户端。

`Gophish` 是为企业和渗透测试人员设计的开源网络钓鱼工具包。它提供了快速，轻松地设置和执行网络钓鱼攻击以及安全意识培训的能力。

  

## **0x01 EwoMail**

### **1.1 安装**

1.`Centos` 安装 `telnet`：

```text
yum install telnet -y
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-125a56c8165353a9a35abbd69176c036.jpg)

2\. 测试 `25` 端口连通性：

一般服务器默认不开放 `25`，国内外服务器自行选择，各有利弊。

```text
telnet smtp.qq.com 25
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-786afe593a15306b6084cf747d4e4ae1.jpg)

3\. 配置建议：

```text
centos7/8系统 64 位，服务器需要干净环境，要求全新干净系统，不能安装在已有的 nginx,mysql 的环境中。

如需在已有配置数据环境安装，请自行参考安装代码修改和维护。

安装前请仔细看文档，建议使用 centos7 安装

最低配置要求（云服务器的最低建议配置）
CPU：1 核
内存：2G
硬盘：40G
带宽：1-3M
```

4\. 关闭 `selinux`：

```text
vi /etc/sysconfig/selinux
SELINUX=enforcing 改为 SELINUX=disabled
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-d8b4152b3820b0b666bbc46653e34727.jpg)

5.`http://www.ewomail.com/list-11.html`，获取安装代码：  

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-20c884cd6cb97c253aec5dd7ff246ef1.jpg)

6\. 安装成功后将会输出 `Complete installation`，查看安装的域名和数据库密码：

```text
cat /ewomail/config.ini
```

7\. 将你安装的域名，例如安装的域名是 `xxx.com`，就将这行加在服务器的 `hosts` 文件里 `/etc/hosts`：

```text
127.0.0.1 mail.xxx.com smtp.xxx.com imap.xxx.com
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-c3794ce7ee0e9f57925c1f155c99048e.jpg)

8\. 域名添加 `7` 条解析记录：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-c0dcab278b06d887ca690bb2c801c0f6.jpg)

05

9\. 访问地址（将 `IP` 更换成你服务器 `IP` 即可）：

```text
邮箱管理后台：http://IP:8010（默认账号 admin，密码 ewomail123）
ssl 端口 https://IP:7010

web 邮件系统：http://IP:8000
ssl 端口 https://IP:7000

域名解析完成后，可以用子域名访问，例如下面
http://mail.xxx.com:8000 (http)
https://mail.xxx.com:7000 (ssl)
```

### **1.2 配置邮件服务器**

1\. 邮件服务器首页：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-e37b08484fe337223a49f93fe0dc711a.jpg)

2\. 域名配置：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-4b6c5b7431eecf22d4f6676269b2be7b.jpg)

3\. 添加邮箱：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-12669e56b327156e693e1fba4f28e9bb.jpg)

4\. 访问邮箱系统进行登录使用：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-2de346770895fc969e0791e39185d8e0.jpg)

5\. 设置账户名称以在收件人简称处显示：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-54c623bcb1ff28086e0c22535a91e634.jpg)

### **1.3 邮件发送测试**

发送测试邮件到 `QQ` 邮箱：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-b27a37ab6beed33ebe479f1b10503ee8.jpg)

## **0x02 Gophish**

### **2.1 安装**

1\. 项目地址：*[https://github.com/gophish/gophish](https://github.com/gophish/gophish)*

2.`Gophish` 适用于 `Ubuntu` 系统，因为 `Ewomail` 推荐 `Centos` 搭建，这次使用 `docker` 搭建 `Gophish`：

```text
yum install docker -y
systemctl start docker
docker pull gophish/gophish
docker run -it -d --rm --name gophish -p 3333:3333 -p 89:80 -p 88:8080 gophish/gophish
docker logs gophish
```

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-913ff0ac41617a0a7fb26099f4bd3350.jpg)

3\. 访问映射后的后台端口：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-fda6a9450ba00ba6e31e7331ffde2e63.jpg)

1\. 设置 `smtp` 等配置（`hr@abcd.com` 已经提前在 `EwoMail` 中添加域名并创建邮箱账户、`abcd.com` 创建对应 `hosts` 解析，需要确保你添加的域名未设置 `SPF`，否则会不可用，可用“`nslookup -type=txt` 域名”来检测）:  

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-488b1eb37049280878102008b557bcb0.jpg)

2\. 添加收件人后测试收件效果是否为设置的邮箱地址：  

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-5304f197d56c851b00de070f1716cf11.jpg)

3\. 收件人视角：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-9424264b6e17a71e0737fd5853637dc8.jpg)

4\. 配置钓鱼页面，针对目标特点配置对应风格（保存后直接复制内容导入或利用 `mip22` 导入对应页面）：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-5aeb50940d7870c4ddd13afb40506c4a.jpg)

本次模拟均为测试，以 `steam` 登录为例。

5\. 配置邮件模板，导出选择的邮件格式为 `eml` 格式后，复制到 `HTML` 处：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-673b6e7f1e9ea48c75322af2e1f88906.jpg)

1\. 选择 `Campaigns`，创建新的钓鱼活动：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-4a241a0e7e70fb25db4ae3c3af4bf5f3.jpg)

2\. 选择对应配置好的模板、跳转页，在 `URL` 处填入 `Gophish` 的接口地址与端口，点击发送；目标打开邮件后，后台会显示收件人的对应信息：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-d2424e88c0f1a67447893846c6bd7214.jpg)

3\. 目标收到构造好的邮件后点击按钮：  

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-7ae6dc71add732bad33505d8a6f2693e.jpg)

4\. 跳转到构造好的钓鱼页面：  

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-08907a2e2fde2a417fe631df9d6cd784.jpg)

5\. 在钓鱼页面输入对应信息，如账户、密码后，`Gophish` 会将信息返回至后台：  

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-791b7745672c44025f0468e1bdf46476.jpg)

1\. 针对 `Ewomail` 与 `Gophish` 的钓鱼方式还包括利用未开启 `SPF` 的对应目标企业单位域名，伪造其邮件地址对其进行钓鱼，搭配对应的常用系统或者敏感内容（例如薪资、社保等），引导其打开附件内的导航链接或附件，获取目标计算机的控制权。

2\. 例如利用微信截图名称的文件等可以引起目标好奇心的文件诱导其执行：

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-6c22adc57ce20550a38185b4525a3725.jpg)

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711963440-ed14617cadc908d8b6d83d497babf47e.jpg)

## **0x03 总结**

总结起来，钓鱼邮件是一种非常有效的网络欺诈手段，它利用了人们的社交工程和心理漏洞。通过伪装成可信的实体或组织，攻击者试图获取用户的敏感信息、登录凭据或进行其他恶意活动。钓鱼邮件是攻击者利用社交工程和技术手段欺骗用户的一种常见方式。通过了解攻击者的策略和方法，用户可以更好地保护自己，提高网络安全意识，并采取适当的防范措施来减少成为钓鱼攻击的受害者的风险。
