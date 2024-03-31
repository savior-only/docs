---
title: 利用 Masscan 批量生成 IP 地址表
url: https://mp.weixin.qq.com/s?__biz=Mzg5NTcwNDcyMg==&mid=2247487834&idx=1&sn=085b392cdf16e9716848938a869362ea&chksm=c00d1aa7f77a93b13ba70a05587380c5af6c48df45218b16cdfc9c9e849c322cce5f1fe2bd99&mpshare=1&scene=1&srcid=0217Ujr1RH48ZHsXAjLgQ7Mu&sharer_shareinfo=b1100c037d977c18476edfd3996076ab&sharer_shareinfo_first=b1100c037d977c18476edfd3996076ab#rd
clipped_at: 2024-03-31 15:49:40
category: temp
tags: 
 - mp.weixin.qq.com
---


# 利用 Masscan 批量生成 IP 地址表

# 简介

Masscan 是 Kali 下集成的高效扫描器，和 nmap 命令有很多相似之处，本文主要记录一下 Masscan 的另类用法。

# 命令生成随机 ip

```plain
BASH

masscan -sL 10.0.0.0/24 > c 段.txt
masscan -sL 10.0.0.0/16 > b 段.txt
masscan -sL 10.0.0.0/8  > a 段.txt
```

  

`sL`：显示扫描的所有主机的列表  
`> xx.txt`：把终端命令行中的结果保存在 `xx.txt` 文件中

```plain

版权声明：本文内容来自个人博客：国光，遵循 CC 4.0 BY-SA 版权协议上原文出接及本声明。
本作品采用知识共享署名 - 非商业性使用 - 禁止演绎 2.5 中国大陆许可协议进行可。
原文链接：https://www.sqlsec.com/2017/07/masscan.html
如有涉及到侵权，请联系，将立即予以删除处理。
在此特别鸣谢原作者：国光的创作，Powered by Butterfly/By 国光。
本文发布已获原作者国光授权转载。
此篇文章的所有版权归原作者所有，与本公众号无关，商业转载建议请联系原作权，非商业转载请注明出处。
```