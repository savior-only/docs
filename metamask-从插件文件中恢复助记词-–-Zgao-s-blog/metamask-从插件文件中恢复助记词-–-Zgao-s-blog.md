---
title: metamask 从插件文件中恢复助记词 – Zgao's blog
url: https://zgao.top/metamask%E4%BB%8E%E6%8F%92%E4%BB%B6%E6%96%87%E4%BB%B6%E4%B8%AD%E6%81%A2%E5%A4%8D%E5%8A%A9%E8%AE%B0%E8%AF%8D/
clipped_at: 2024-03-26 09:53:50
category: default
tags: 
 - zgao.top
---


# metamask 从插件文件中恢复助记词 – Zgao's blog

![](assets/1711418030-ca81cd7facfbd816279e3e26d45a8507.png)

上周研究了一下如何从小狐狸的插件文件中恢复助记词。结合官方描述是在知道密码的前提下，可以从插件文件中提取 vault 字段在官方提供的解密页面进行恢复。

[https://metamask.github.io/vault-decryptor/](https://metamask.github.io/vault-decryptor/)

## 插件目录

以 Chrome 为例，小狐狸插件配置文件目录为：

```plain
C:\Users\Administrator\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\
```

注意：插件源码和配置文件并不在同一个目录下。插件的 ID 可以在 chrome://extensions 中看到。

![](assets/1711418030-854c2924ffe34d52edeaa020d8ce4622.png)

![](assets/1711418030-37c41951b513203434a9f5f38fb95a7b.png)

## 获取 vault

这里的 ldb 文件是 Google 的 leveldb 数据库文件，需要用特定的方式打开。而在 log 文件中也能找到加密后的 vault，直接用记事本打开搜索 KeyringController。

![](assets/1711418030-5f13a01d64706f7423c45f56283c8921.png)

复制 vault 对应完整的 json。

## 解密助记词

![](assets/1711418030-15cf4327c99831aa28436e434bd31679.png)

## 代码解密

Post Views: 15
