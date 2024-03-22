

# driftingbules:5 靶场解析及复现 - FreeBuf 网络安全行业门户

**kali 的 IP 地址：192.168.200.14**

**靶机 IP 地址：192.168.200.60**

# 一、信息收集

## 1\. 对利用 nmap 目标靶机进行扫描

由于 arp-scan 属于轻量级扫描，在此直接使用 nmap 进行对目标靶机扫描开放端口

```bash
nmap -A -p 1-65535 192.168.200.60
```

使用 nmap 扫描 开放的端口是 22（ssh）、80（http）端口

![1703208594_6584e692544d0a6888142.png!small?1703208596053](assets/1710207215-8b2d15585e1a18706e079b5637308b14.png)

## 2\. 访问网站

发现框架是 wordpress 框架，或者也可以使用 wappalyzer、whatweb

![1703208602_6584e69a085375b16849d.png!small?1703208604047](assets/1710207215-28c9da1c775a03626fbb7e771f24bf43.png)

既然是 wordpress 系统，那么后台登录页面就是 wp-login.php

![1703208610_6584e6a2cb4bea26b2e46.png!small?1703208612125](assets/1710207215-c4621ec4b94007c956bf22404b340882.png)

# 二、漏洞探测

## 3.wpsan 爆用户名

使用专门扫描 wordpress 得 wpsan

使用 wpscan 扫描下账户信息，发现 gadd、gill、collins、satanic、abuzerkomurcu 账户。

```bash
wpscan --url "http://192.168.200.60/" -e u #对目标网页进行扫描
```

![1703208621_6584e6ad2711aad5ac80a.png!small?1703208623267](assets/1710207215-943eae1a2afcccd217cf4948bb4f88f6.png)

此时，已经拥有了**用户名**将之保存到 username.txt 下，在寻找登陆密码

```bash
gobuster dir -u http://192.168.200.60  -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt  -x php,jpg,txt,html
```

![1703208631_6584e6b7a6b7f5676f055.png!small?1703208633216](assets/1710207215-14238c7642b3d669cfd5316582ea6fc4.png)

![1703208639_6584e6bfccfc52843f188.png!small?1703208641077](assets/1710207215-55b0d4f8748855f79a7557ed98802aba.png)

对网站进行扫描之后，没有发现密码本类似得信息

## 4\. 用 cewl 爬取网站密码：

```bash
cewl -m 6 -w passwd.txt http://192.168.200.60 # 爬取网页信息，保存到 passwd.txt 中
```

![1703208647_6584e6c742cde62233177.png!small?1703208648613](assets/1710207215-e74db637466562c5dfaa25a7a470c71c.png)

## 5.wpscan 爆破后台登录账户密码

对刚才发现的几个账户进行密码爆破，

```bash
wpscan --url http://192.168.200.60 -U username.txt -P passwd.txt
```

![1703208655_6584e6cf8388008fc2f96.png!small?1703208657081](assets/1710207215-7d9b49f9d417fd651005880c68ac1df4.png)

有一组符合条件

```bash
账户名：gill
密码:interchangeable
```

成功进入后台

![1703208662_6584e6d699f4e8de27b47.png!small?1703208664110](assets/1710207215-2f622c1fa0c921dc6a8484c028935141.png)

这个图片在 index.php 页面没有显示被隐藏，那么首先进行图片隐写的研究

![1703208673_6584e6e14930396a443ba.png!small?1703208675186](assets/1710207215-61320e0da96ad44e37c6f8a121c1e04b.png)

右键复制链接下载到 kali 当中。

```bash
wget 192.168.200.60/wp-content/uploads/2021/02/dblogo.png # 下载
```

## 6\. 识别图片隐藏信息

> Exiftool dblogo.png  
> 这个工具可以识别出一些我们在一般情况下无法识别的图片中的信息

  

**有 ssh 的密码，用户名用之前的那五个用户名试试**

![](assets/1710207215-eb823ce8f49bc8ca60ba59385c7d314a)
