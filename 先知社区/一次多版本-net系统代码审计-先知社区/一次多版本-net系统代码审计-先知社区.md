---
title: 一次多版本.net系统代码审计 - 先知社区
url: https://xz.aliyun.com/t/14180?u_atoken=aa7638b7d038ebb9a3993df843be2e70&u_asession=01wvDka-YZJppSXloc22pi2-dowy_ZLR4MQXtgs4qZozuKO9lilF5zEnn9dQN7dpmRdlmHJsN3PcAI060GRB4YZGyPlBJUEqctiaTooWaXr7I&u_asig=05ScOxeMVmtp-_FXITMQWVuD5o9qyNVs1OAdo7CDak0ajCjfR_-5QX3jTjXhABGJYWPhLOPgHX5xNpm0exTkPBBGzjBGGGUdVbfH2LajnJcpvrTNmwPLL0LO-0YmiCOo7PUgzQ2rmQfaI3kCUH6nrGmBqIiuJCt981IQKnRLUbbgtg2QMxYs6lyXb1lFWKql56wgPqJ-gMVBgy9bydU-CSmjCu9Fzs324A7LFHzI_zGAFRURv06LITWmoubVGPSQpCrgzhUnCUO8kFq22j9o5BxDVZB4MuZHT0qqFIV4ngWlF6gx6UxFgdF3ARCQ86jS_u_XR5hatHQVh06VuUZ-D1wA&u_aref=3Z3vrpee5RJn39MZ6P2ZSitauAU%3D
clipped_at: 2024-03-26 23:09:13
category: default
tags: 
 - xz.aliyun.com
---


# 一次多版本.net系统代码审计 - 先知社区

# 前言

做项目喜闻乐见的获取了bin.rar，当时快速过了config交了数据库权限，然后看代码发现mobileController下方法未授权，交了一些信息泄露，大部分是人员手机相关。项目结束后想交cnvd，写了个nuclei跑了一下结果没几个，当时大为不解，但是感谢他们部署系统的好习惯，顺手测了另两个站的bin.rar，嘿嘿。dnspy导出到vs里，记录一下发现的漏洞。

> 最后发现最开始漏洞少的原因：1.系统版本不同，第一个源码应该是较老的版本，后面系统重构。2.供应商对每个企业多多少少做了功能的个性化开发，导致出现某个企业特有的漏洞。

# 漏洞

## 未授权

别问我什么rce、sql、反序列化，问就是未授权

老版本mobileController下未授权，可利用接口很多，不一一列举，但是大部分需要别的参数。不过有数据能交就行。  
[![](assets/1711465753-013f0373d748d08c2e748bcc2e4b2e8e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312153321-cd6bed66-e042-1.png)

[![](assets/1711465753-dce459f45f3a45982a403848c9afd735.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312153622-39906616-e043-1.png)

[![](assets/1711465753-58f9ecc75773b3d6569fb96c029771dc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312153751-6e7938d0-e043-1.png)

## 文件上传

搜索saveas  
**老版本下两个**

-   一个直接的图片上传getshell
-   一个参数全部可控  
    [![](assets/1711465753-4575d536999a9d85fec7d76cf90595eb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312154851-f798d1ec-e044-1.png)  
    附件目录下不能解析，穿一下目录，传到根目录

[![](assets/1711465753-63e6db4b3e2f1fbfa1f5568a6efb45ae.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312155013-28d51612-e045-1.png)

[![](assets/1711465753-b44cdf98a81e1ef9ad82066d75dbb2f0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312155320-97fd900a-e045-1.png)

**中版本无**  
中版本开始权限使用AllowAnonymous  
ImgFile默认没开  
[![](assets/1711465753-bab5ccdda19514eff23e47a9d24260f7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312155901-63426380-e046-1.png)

[![](assets/1711465753-da3344f6e8f0537cff6d2c862c578b62.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312155937-78aed352-e046-1.png)

**新版本**  
嘿，您猜怎么着，其他洞全都没了。但是系统贴心的加了一个功能

[![](assets/1711465753-677aed71c8c00e1f5f69764c01ae3f50.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312160131-bcc19070-e046-1.png)

## 注入

老版本  
[![](assets/1711465753-5e934cff18ea84d8cf2aef467b27131c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312160205-d11c6798-e046-1.png)

[![](assets/1711465753-fc4f3800e9c8d54554c2dde8e176eb16.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312160300-f1fe145c-e046-1.png)

其他

[![](assets/1711465753-f60c3adb38e18805aafcd84273642f98.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312160926-d82f9676-e047-1.png)

[![](assets/1711465753-4383eb0013dec0636bbf153dbd00b25c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312160615-65ca95cc-e047-1.png)

顺便看了一下反序列化，莫得

## 任意用户登录

某企业定制化漏洞

[![](assets/1711465753-ec6c8c92e37e166af2db1669da644a85.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312161403-7cdbff3e-e048-1.png)

[![](assets/1711465753-55f04b24da9ada76245164f73adad28c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312161627-d31f42a2-e048-1.png)

给个名字，还你个cookie，很棒