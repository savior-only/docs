---
title: 编写masscan报告转换脚本
url: https://mp.weixin.qq.com/s?__biz=MzUyNTkyOTEzMg==&mid=2247483860&idx=1&sn=635aeabb06cb01a9c9687680dcbe7a12&chksm=fa17de17cd605701efccc9d9af81ce7c5493ebb146d8c5934c8fcc4c49f8e09221333df8a099&mpshare=1&scene=1&srcid=021700pGj5R2ZGEQEinFshqD&sharer_shareinfo=5b285bef5837dbecd99d5bd0dd0dbff7&sharer_shareinfo_first=5b285bef5837dbecd99d5bd0dd0dbff7#rd
clipped_at: 2024-03-31 15:50:05
category: temp
tags: 
 - mp.weixin.qq.com
---


# 编写masscan报告转换脚本

由于nmap扫描比较慢，有时候需要使用masscan对大段ip进行快速扫描。为了后续方便数据处理，往往需要将数据以xls的形式进行统计，但是masscan只支持xml,json,list等格式输出,并不支持直接输出xls格式。最近有正好这个需求，于是写了个小脚本来转换一下。

## 一、编码

#### file: masscan-report-converter.py

![图片](assets/1711871405-17a54987c5c69f5beb20a492d7bcdd93.jpg)

#### ![图片](assets/1711871405-ce721c3f442e250b504af7f5f4e0a260.png)

目前脚本已经收集到我的WorkScript项目中，地址如下：

https://github.com/c0ny1/WorkScripts/tree/master/masscan-report-converter

## 二、使用步骤

##### 1.使用masscan进行扫描，扫描结果以xml保存

![图片](assets/1711871405-b8d54d1102e5b8d0c9829fec4a3ef985.png)

##### 2.使用上面写的脚本转换出xls格式的报告

![图片](assets/1711871405-352aac782fc0c77b57dfa4d0a586df9c.png)

最终效果如下：

![图片](assets/1711871405-e6cdf98f54f2814df14a0c80bf927f6f.jpg)

图1-脚本转换后的报告