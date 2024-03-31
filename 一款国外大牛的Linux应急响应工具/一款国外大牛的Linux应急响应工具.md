---
title: 一款国外大牛的 Linux 应急响应工具
url: https://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247521277&idx=1&sn=1ba60746ef7251039f259f7f5c368789&chksm=9acd4362adbaca74d6071583ec3fa654ac2d889016311cc77101ea32b5f5015bca824799d9b9&mpshare=1&scene=1&srcid=0228HGnCjA3CBwAAm6EWS2TL&sharer_shareinfo=009497813a776fa217d9e6e74140ef59&sharer_shareinfo_first=009497813a776fa217d9e6e74140ef59#rd
clipped_at: 2024-03-31 19:31:41
category: temp
tags: 
 - mp.weixin.qq.com
---


# 一款国外大牛的 Linux 应急响应工具

MR.Handler 是一款专门为响应 Linux 系统上的安全事件而设计的工具。它通过 SSH 连接到目标系统以执行一系列诊断命令，收集关键信息，例如网络配置、系统日志、用户帐户和正在运行的进程。在操作结束时，该工具会将所有收集到的数据编译成综合的 HTML 报告。该报告详细介绍了事件响应过程的细节和系统的当前状态，使安全分析师能够更有效地评估和响应事件。

**安装**  

```plain
  pip3 install colorama
  pip3 install paramiko
  git clone https://github.com/emrekybs/BlueFish.git
  cd MrHandler
  chmod +x MrHandler.py
  python3 MrHandler.py
```

# 报告

![图片](assets/1711884701-aaccc22f7353224ad768f05902a31672.webp)

  

下载地址：  

https://github.com/emrekybs/MrHandler