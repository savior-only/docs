---
title: [自动化bugbounty] hackerone，bugcrowd赏金资产范围
url: https://mp.weixin.qq.com/s?__biz=MzU2NzcwNTY3Mg==&mid=2247484853&idx=1&sn=97385eaab8996c59fb00f5b445cc2b38&chksm=fc986c92cbefe5849784645e7114a2ac457e3425d2f6c129575b36cb23f400629d3557c715ee&mpshare=1&scene=1&srcid=0216RYF1x0KWyymh1eJP2WVI&sharer_shareinfo=107c0d7f0f426c21b88f3d6f48e838ce&sharer_shareinfo_first=107c0d7f0f426c21b88f3d6f48e838ce#rd
clipped_at: 2024-03-31 19:21:55
category: temp
tags: 
 - mp.weixin.qq.com
---


# [自动化bugbounty] hackerone，bugcrowd赏金资产范围

xscan发布后，已经有不少人获得了赏金了

![图片](assets/1711884115-6a116269fa6af1d432fdd2b846e90a3d.webp)  

一个xss奖金将近5000CNY，加入星球找到一个就回本，甚至有25倍的回报率。  

来自 大佬的评价：  

![图片](assets/1711884115-e20f9a39dfa17e37822c42bc62e2aadb.webp)

有了xscan，还差一个资产范围，这不，提取了hackerone和bugcrowd官方含有漏洞赏金项目的范围，51可以卷起来了～

（xscan将扫描细节的参数抽离出来了，不同参数扫描效果不一样，可以多测试选择适合的参数进行扫描）  

资产范围提取自官方，json格式,包含inscope,outscope,url,name,logo字段,后续会做成监控，用于持续监控更新的项目，更新的范围。

h1，bugcrowd赏金范围公开下载，公众号回复“资产范围”

加入知识星球，公众号回复“知识星球”