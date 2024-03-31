---
title: 基于Microsoft Graph API上线CS工具
url: https://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247489646&idx=1&sn=aacbe839955ef98ee8510872b54ecdf9&chksm=9036fed6a74177c0e4f8fdd142e69ad2e1e720bec1112d74b86d8ff2eee11086e2871472e68d&mpshare=1&scene=1&srcid=02169pTLYvz3YOzL9FGD2qzD&sharer_shareinfo=1f8bcc78ca35e90c3a123f956c21b590&sharer_shareinfo_first=1f8bcc78ca35e90c3a123f956c21b590#rd
clipped_at: 2024-03-31 15:39:08
category: temp
tags: 
 - mp.weixin.qq.com
---


# 基于Microsoft Graph API上线CS工具

  

  

- - -

SiegeCast发了两篇关于GraphStrike工具的文章，建议可以先去看一下。

```plain
GraphStrike：使用Microsoft Graph API让信标流量消失
https://redsiege.com/blog/2024/01/graphstrike-release

GraphStrike：攻击性工具开发剖析
https://redsiege.com/blog/2024/01/graphstrike-developer
```

  

**工具介绍**

GraphStrike是一套工具，使Cobalt Strike的HTTPS Beacon能够使用Microsoft Graph API进行C2通信。所有Beacon流量将通过攻击者的SharePoint站点中创建的两个文件进行传输，并且来自Beacon的所有通信都将路由到 https://graph.microsoft.com：  

```plain
https://learn.microsoft.com/en-us/graph/use-the-api
```

![图片](assets/1711870748-06932a578c796ba9f21353ff370ae7e5.webp)

  

GraphStrike包含一个配置程序，用于通过Graph API创建Cobalt Strike HTTPS所需的Azure资产：

![图片](assets/1711870748-474ed531ad2dd26fab8e4c7f71476e24.webp)

  

**GraphStrike 不会在 Azure 中创建任何付费资产，因此使用 GraphStrike 或其配置程序不会产生额外费用。**

  

**为什么用Microsoft Graph API？**

已发布关于利用 Microsoft Graph API 和其他 Microsoft 服务进行攻击活动的几种不同 APT 的威胁情报：  

```plain
https://www.volexity.com/blog/2021/08/17/north-korean-apt-inkysquid-infects-victims-using-browser-exploits/
https://malpedia.caad.fkie.fraunhofer.de/details/win.graphite
https://symantec-enterprise-blogs.security.com/blogs/threat-intelligence/flea-backdoor-microsoft-graph-apt15
https://www.elastic.co/security-labs/siestagraph-new-implant-uncovered-in-asean-member-foreign-ministry
```

威胁行为者继续利用合法服务来实现非法目的，利用像 graph.microsoft.com 这样的高声誉域进行C2通信是非常有效和理想的，但从时间和精力的角度来看，通常很复杂且令人望而却步。  

大多数C2框架不支持获取或轮换访问令牌的方法，这使得它们无法使用Graph API。这可能会使红队难以复制这些技术，并剥夺防守者观察和开发此类活动签名的机会。

GraphStrike致力于减轻这一负担，并提供可靠且可重复的流程来利用Microsoft Graph API，同时保持Cobalt Strike用户体验的熟悉度和可靠性。

  

**GraphStrike局限性**

这个工具存在以下限制：  

```plain
仅支持x64信标。
不支持分阶段信标。
GraphStrike仅与WinINet库兼容；不支持信标的新WinHTTP库选项。
不支持通过Beacon的右键菜单发出睡眠命令，睡眠信标改用命令行选项。
GraphStrike仅在Cobalt Strike的Linux实例上受支持，Windows支持当然是可以实现的，实际上只需更改Python文件和Aggressor脚本中的某些路径即可。
```

  

**下载地址**

**点击下方名片进入公众号**

**回复关键字【****240215****】获取**下载链接****

  

- - -