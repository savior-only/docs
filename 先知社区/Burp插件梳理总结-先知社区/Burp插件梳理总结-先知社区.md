---
title: Burp 插件梳理总结 - 先知社区
url: https://xz.aliyun.com/t/12594?time__1311=mqmhD50KBIqIxeqGNDQbB44xcjTnpxD8QeD&alichlgref=https%3A%2F%2Fwww.google.com%2F
clipped_at: 2024-03-28 00:15:28
category: default
tags: 
 - xz.aliyun.com
---


# Burp 插件梳理总结 - 先知社区

# 前言

​ 很多师傅在搞渗透测试时，离不开 burpsuite，有一些好的插件如虎添翼，我把常用的 burp 插件整理分享出来，若各位师傅有好用的插件欢迎留言交流！

# 插件

### Log4j2Scan

**描述**

```plain
该工具为被动扫描 Log4j2 漏洞 CVE-2021-44228 的 BurpSuite 插件，具有多 DNSLog（后端）平台支持，支持异步并发检测、内网检测、延迟检测等功能。
```

**下载地址**

```plain
https://github.com/whwlsfb/Log4j2Scan
```

### BurpFastJsonScan

**描述**

```plain
一款基于 BurpSuite 的被动式 FastJson 检测插件
```

**下载地址**

```plain
https://github.com/pmiaowu/BurpFastJsonScan
```

### BurpJSLinkFinder

**描述**

```plain
用于端点链接的被动扫描 JS 文件的 Burp 扩展。
```

**下载地址**

```plain
https://github.com/InitRoot/BurpJSLinkFinder
```

### BurpShiroPassiveScan

**描述**

```plain
一款基于 BurpSuite 的被动式 shiro 检测插件
```

**下载地址**

```plain
https://github.com/pmiaowu/BurpShiroPassiveScan
```

### BurpSuite\_403Bypasser

**描述**

```plain
绕过 403 限制目录的 Burpsuite 扩展
```

**下载地址**

```plain
https://github.com/sting8k/BurpSuite_403Bypasser
```

### Fiora

**描述**

```plain
漏洞 PoC 框架 Nuclei 的图形版。快捷搜索 PoC、一键运行 Nuclei。即可作为独立程序运行，也可作为 burp 插件使用
```

**下载地址**

```plain
https://github.com/bit4woo/Fiora/releases
```

### HackBar

**描述**

```plain
Burpsuite 的 HackBar 插件
```

**下载地址**

```plain
https://github.com/d3vilbug/HackBar
```

### HaE

**描述**

```plain
HaE 是基于 BurpSuite Java 插件 API 开发的请求高亮标记与信息提取的辅助型框架式插件，该插件可以通过自定义正则的方式匹配响应报文或请求报文，并对满足正则匹配的报文进行信息高亮与提取。
```

**下载地址**

```plain
https://github.com/gh0stkey/HaE
```

### sqlmap4burp++

**描述**

```plain
sqlmap4burp++ 是一款兼容 Windows，mac，linux 多个系统平台的 Burp 与 sqlmap 联动插件
```

**下载地址**

```plain
https://github.com/c0ny1/sqlmap4burp-plus-plus
```

### TsojanScan

**描述**

```plain
一个集成的 BurpSuite 漏洞探测插件
```

**下载地址**

```plain
https://github.com/Tsojan/TsojanScan
```

### wooyu 同类漏洞查询

**描述**

```plain
从 wooyun 中提取的 payload，以及 burp 插件
```

**下载地址**

```plain
https://github.com/boy-hack/wooyun-payload
```

### 分块传输

**描述**

```plain
Burp suite 分块传输辅助插件
```

**下载地址**

```plain
https://github.com/c0ny1/chunked-coding-converter
```

### 伪造 IP

**描述**

```plain
服务端配置错误情况下用于伪造 ip 地址进行测试的 Burp Suite 插件
```

**下载地址**

```plain
https://github.com/TheKingOfDuck/burpFakeIP
```

### 信息收集管理

**描述**

```plain
domain_hunter 的高级版本，SRC 挖洞、HW 打点之必备！自动化资产收集；快速 Title 获取；外部工具联动；等等
```

**下载地址**

```plain
https://github.com/bit4woo/domain_hunter_pro
```

### 验证码爆破

**描述**

```plain
captcha-killer 的修改版，支持关键词识别 base64 编码的图片，添加免费 ocr 库，用于验证码爆破，适配新版 Burpsuite
```

**下载地址**

```plain
https://github.com/f0ng/captcha-killer-modified
```

### 越权检测

**描述**

```plain
AuthMatrix 是一个 Burp Suite 扩展，它提供了一种简单的方法来测试 Web 应用程序和 Web 服务中的授权。
```

**下载地址**

```plain
https://github.com/SecurityInnovation/AuthMatrix
```

# 螺丝鸽安全公众号

```plain
螺丝鸽安全团队是由一群青春活力的年轻人组成的，我们专注研究 web 渗透、代码审计、免杀、社工钓鱼、内网渗透、应急响应、红蓝对抗。非常欢迎各位师傅关注螺丝鸽安全公众号！让我们一起交流探讨技术。
```

# 下载

​ 因考虑插件比较多，全部下载有点费时，我已全部下载并上传到百度网盘，在公众号后台回复：【burp 插件】即可获得链接。