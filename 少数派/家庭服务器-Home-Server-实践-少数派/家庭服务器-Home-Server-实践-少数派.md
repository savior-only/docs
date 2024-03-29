---
title: 家庭服务器 Home Server 实践 - 少数派
url: https://sspai.com/post/82512
clipped_at: 2024-03-26 17:24:34
category: default
tags: 
 - sspai.com
---


# 家庭服务器 Home Server 实践 - 少数派

- - -

## 前言

尽管网上已经有很多 NAS、主机用于个人或家庭服务的文章了，但多数侧重于硬件和 NAS 厂商定制软件体验。很多朋友入手 NAS 后，费了老大劲部署好一个服务，却发现体验一般，可能还不如互联网大厂提供的应用，最后折腾下来，也就用于备份、看剧了。因此，我斗胆以广而不深的方式介绍一下这些年来我在家庭服务器 Home Server 上的实践。

### 本文有什么？没有什么？

我希望结合具体使用场景，介绍如何发挥家庭服务器的价值。

本文将会涉及：

-   我的家庭服务器自托管服务概况；
-   服务与用户终端软件的联动使用；

本文不会详细介绍：

-   服务器的硬件选择与架设；
-   如何部署服务；

## 服务器硬件

虽然本文不会涉及硬件的选择，但还是简单地介绍一下我自己的家庭服务器硬件概况。我有两台服务器硬件，分别是华擎 ASRock DeskMini X300、威联通 QNAP TS-453Bmini。X300 是 STX 机箱准系统，运行 Ubuntu 20.04 系统、453Bmini 是威联通出品的 NAS，运行它自家的 QTS 5.0 系统。以下是两台服务器的硬件配置，其中，NAS 上的大容量卷通过 NFS 挂载在 Ubuntu 上。

| 项目  | DeskMini X300 | TS-453Bmini |
| --- | --- | --- |
| 主板  | ASRock X300M-STX | ？   |
| CPU | AMD 5600g | Intel® Celeron® J3455 |
| 内存  | 32G+8G, 3200MHz | 8G DDR3 |
| 显卡  | 核显 Vega7 | 核显 Intel® HD 500 |
| 存储  | SSD 1T + 500G | HDD 18T + 12T + 8T |
| 接口  | **2xM.2 / 2x2.5"SATA** / Wi-Fi / USB3.2 / DP, HDMI / GB 网口 | **4x3.5"SATA** / 4xUSB3.2 / HDMI / 2xGB 网口 |

近年来，各种迷你主机、NAS、轻 NAS 推陈出新，以上配置仅供参考。对于本文介绍的服务而言，我这一套硬件是能很从容地应对的。

## 服务概况

一般来说，家庭服务器可用于以下需求：

-   存储：BT/PT 下载、文件同步 / 备份；
-   多媒体：电影 / 音乐 / 书籍管理和在线播放；
-   自动化：智能家居、自动化流程、爬虫；
-   游戏：私有服务器；
-   网站：个人网站；
-   网络：软路由、流量审计；
-   实验室：「有些人在生产 / 娱乐中使用它们来学习新技术」

首先对我的家庭服务器上部署的服务进行一个总览。按照功能归为六个大类：**存储与同步、RSS、多媒体、自动化、监控与其他应用**。当然，还有一类我称之为「基础设施」，并没有列在其中，但为了实现上述的六类功能，它们是不可或缺的，比如，Nginx 反向代理、Let's Encrypt SSL 证书更新、DDNS、Wireguard VPN、容器、虚拟机等等。

大多服务都是免费、开源的，也可以很方便地通过 Docker 部署，作为两个特例：Roon 是必须付费使用的，Plex 可选择付费提升移动端体验。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-6521a89e39624556eb0a2f9cd609d7a8.png)

服务总览

### 存储与同步

我的同步 / 备份需求有 7 类：

-   笔记：Markdown 格式文本、各类本地附件，终端软件为 Obsidian；
-   文献：科研文献 PDF 文件、数据库，终端软件为 Zotero；
-   密码：重要密钥，网站 / 应用密码，自动填充；
-   相册：手机 / 平板拍摄的照片 + 相机拍摄的照片
-   截图：手机 / 平板的截图实时同步到其他设备；
-   工作文件：大量的、各类格式的文件，需要版本控制，便捷分享；
-   中转站：阅后即焚的文本 / 文件中转，接收其他人发给我的文件；

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-699d24b6b741d8125abbf9fbc7cba0e9.png)

存储与同步路径

这是一套可以覆盖全平台的存储与同步方案，虽然部署的服务和需要安装的终端软件较多，但这些软件基本可以做到无感地后台运行。家庭服务器上部署的服务有：

-   [Nextcloud](https://sspai.com/link?target=https%3A%2F%2Fnextcloud.com%2F): 私有云盘，完成 PC 端文件同步、版本控制，提供 web 端、移动端 app；
-   [Immich](https://sspai.com/link?target=https%3A%2F%2Fdocumentation.immich.app%2F): 相册备份、浏览，提供 web 端、移动端 app；
-   [Bitwarden](https://sspai.com/link?target=https%3A%2F%2Fbitwarden.com%2F): 密码同步与自动填充，提供 web 插件、移动端 app；
-   [Resilio](https://sspai.com/link?target=https%3A%2F%2Fwww.resilio.com%2Findividuals%2F): P2P 文件同步，全平台文件同步；（也可以用*微力同步*替代）
-   [MicroBin](https://sspai.com/link?target=https%3A%2F%2Fmicrobin.eu%2F): 文本和文件共享 Web 应用程序、阅后即焚；

移动端需要安装的软件有：Resilio、Immich 和 Bitwarden。PC 端需要安装的软件有 Resilio、Bitwarden 插件和 Nextcloud。

**笔记与文献**。我的笔记系统是用 Obsidian 构建的，大多是直接在 Obsidian 内书写的，其他的少量来源有移动端手写笔记、在线多维表格、电子书阅读笔记等，但可以很方便地通过导入或嵌入 Obsidian 中。我把 Zotero 的附件文件夹作为子文件夹囊括在了 Obsidian 的 Vault 内，因此，我在整理一些总结 / 想法的时候可以随意地引用 Zotero 内的科研文献。使用 Resilio 实时同步 Obsidian 文件夹，就可以做到在电脑 1、电脑 2、手机、平板上随时打开 Obsidian 都能获取到最新版本并做修改和同步。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-99e879e32a98036d13c0a0e91c7d2f40.gif)

Obsidian PC 与 Mobile 间同步

**密码**。使用 Bitwarden 进行同步，可以做到在电脑浏览器 / 手机 app 内自动填充网站和应用密码、自动生成复杂密码，存储重要密钥等。

**相册**。照片的来源主要有两处：手机照片和相机照片。通过手机上的 Immich 软件，可以监控 `/Camera` 相册并实时备份至服务器，手机上也可以进行「仅删除本地照片」/「删除所有备份」等操作。相机拍摄的照片需要手动导出并上传至服务器（服务器文件夹通过「映射网络驱动器」挂载在电脑的资源管理器上），Immich 可以监测该文件夹并通过定时任务更新至数据库。移动端可以通过 app，电脑可以通过浏览器，浏览、下载数据库中的所有照片，完美地解决了我手机和相机照片分居两地的尴尬情况。当然，Immich 的功能不止于此：

-   选择性备份；创建专辑；多用户支持和相册共享；公开分享；人脸识别与聚类（很精准）；按地图预览

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-30be901194ea7ecb95b622cbddcef82d.png)

Immich 相册服务

**截图**。很简单的一个功能，使用 Resilio 监控移动端截图文件夹，同步至服务器与电脑端。

**工作文档**。包含大量的、各类格式的文件，主要是在电脑上使用。使用 Nextcloud 等进行备份，支持版本控制、在线预览、Office 在线编辑、快速分享。使用快速分享功能，可以直接在资源管理器内右键选择「复制公开链接」获取文件 / 文件夹的分享链接，还可以设置「允许上传」「允许编辑」「密码保护」「自动过期」等选项。后台也可以查看下载历史。因此，给别人发送文件夹时不用再进行`压缩->上传`的操作了。

**中转站**。MicroBin 提供 Web，可以接收他人发给我的文件；未安装上述任何同步软件情况下的文件中转；阅后即焚的文本 / 文件中转。可以设置「过期时间」「自动焚毁」「代码高亮」「权限设置」「密码访问」。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-9544c34fae07cab03832316deca4ea35.png)

MicroBin 界面

### RSS

站内已经有很多介绍 RSS（Really Simple Syndication, 简易信息聚合）的文章了，需要解决的痛点就两个：RSS 订阅源和 RSS 阅读器。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-82022412028a430c99b981ccb80ea78c.png)

我的 RSS 订阅源分为 4 个主要来源：网站提供、RSSHub、微信公众号和爬虫。

**网站提供**。感谢在 2023 年还愿意提供 RSS 订阅的网站们，安装 RSSHub Radar 插件即可便捷地获取当前浏览网站的 RSS 订阅源。

**RSSHub**。大名鼎鼎的 RSSHub 也提供了很多订阅源，为了有效地使用它们，我也部署在了自己的家庭服务器上。

**微信公众号**。我很讨厌在微信客户端内阅读公众号：一是消息在时间排序上是乱序的；二是在内容上无法按照类别浏览，我无法让大脑做到在科研内容和娱乐内容间迅速切换；三是在浏览时会插播很多推荐内容，但我并不感兴趣；四是没有阅读记录，我无法判断哪些文章读过了，也无法标记稍后阅读。

但不可否认，微信公众号内还是有很多有价值的文章、资讯。也有很多服务提供了微信公众号的内容订阅，比如 Feeddd 和 WeRSS 的订阅服务。但这些服务或是实时性不高，比如几天甚至几周才会更新一次；或是价格昂贵。

*（2023.8 更新，Feeddd 已无法使用，WeRSS 暂不开放新订阅）*

当然，我还能在服务器上完成 RSS 内容的翻译、提炼，特别是对于外文信息，有助于我更快判断信息价值。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-d0c88be16ed00a3af8c081240ad83416.png)

TTRSS 在 Feedme 上的分类阅读，WOS 订阅与翻译

**我部署了 Tiny Tiny RSS 阅读器（TTRSS）**，并将所有订阅源分类为「学术」、更新」、「消费」、「科技」等大类；同时在 TTRSS 上完成信息的过滤和预定义标签，过滤我不感兴趣的关键词，提取我关注的话题并自动贴上与定义标签，供我集中阅读。

**在移动端，我使用 Feedme 作为阅读器。**我曾试用过 InoReader、Feedly、Read You、Feeder、FocusReader、NewsBlur，但还是 Feedme 最合心意：免费、支持 TTRSS 同步、界面美观适配平板、本地过滤。

在 PC 端，使用 TTRSS 的 Web 版阅读。收藏内容沉淀为知识也是在 PC 端，结合 Obsidian 和 Zotero 完成。

### 多媒体

多媒体模块分为书籍、影视、音乐。

**书籍**。服务器上部署了 Calibre 和 Calibre Web 服务，它们共用一个数据库文件 `metadata.db`。

移动端使用静读天下作为阅读器，静读天下提供 webdav 同步阅读进度，刚好可以使用 NextCloud 的 webdav 进行同步。

添加书籍有两种方式。一是通过 Calibre Web 的网页进行上传；二是通过 Resilio 同步静读天下 app 的 `attachment` 文件夹至服务器，服务器一旦监测到新文件，则自动添加至 Calibre 数据库中。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-b8eb3474d1c8e24f05da2bd831ba803a.png)

电子书同步

**影视**。服务器部署 Plex Server，Plex 提供全平台客户端。我购买了 Plex Lifetime。

**音乐**。服务器部署 Roon Server，Roon 提供了全平台客户端。可以说 Roon 是自建数字播放的终极方案了，但是价格不菲，我庆幸是前两年涨价前入手的 Lifetime。Roon 除了支持本地音乐库，还接入了流媒体 Qobuz、Tidal、KKBOX 服务。

对于影视和音乐，资源的获取可能更为困难。这里，只列出所用下载器：Tubesync（油管视频下载）、rtorrent（BT/PT）、pyncm（网易云）。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-1c8bfb5e5ee9f92896bd70ebcc3959ac.jpg)

Roon 与 Roon Arc

### 自动化与监控

自动化与监控放在一起讲。核心是 n8n 和 Node-RED。两者皆是低代码平台，提供了很多模块，可以联动各类软件，创建自己的自动化流程：

-   [n8n](https://sspai.com/link?target=https%3A%2F%2Fn8n.io%2F): Workflow automation for technical people. Build complex automations 10x faster, without fighting APIs
-   [Node-Red](https://sspai.com/link?target=https%3A%2F%2Fnodered.org%2F): Low-code programming for event-driven applications

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-2a2952be57d3d2a67b6a436c5acc786b.png)

自动化流程

我接入自动化与监控系统的有 Linux 定时任务、AI（ChatGPT、Bing）、微信机器人（HOOK）、微软 ToDo、Outlook 邮件、HomeAssistant 等。智能硬件包括智能家居类产品和我自制的硬件，通过 HomeAssistant 接入。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-61fea24cef35301024340539e75d74c7.png)

n8n

作为直接交互，软件为 HomeAssistant 或微信，硬件为小爱音箱。

通过任意路径能与 n8n 或 Node-RED 通信的模块，皆可被「自动化」。比如，我家的电动窗帘本身只支持天猫精灵，但通过 433MHz 无线收发模块可以控制它，再通过 esp32 成功接入 HA。目前我只设置了定时开关窗帘和小爱同学语音控制，但理论上，我还可以设置成微信消息控制开合、电视状态控制开合，甚至「今天油价降了 -> 窗帘提前打开」。至此，限制我的只有想象力。

**监控部分**，Dashy 提供导航页面；微信（HOOK）负责异常状态提醒，当然 Gotify 也能很好地胜任。

### 其他应用

除了以上几大块服务外。我的家庭服务器上还部署了这些工具：

-   [Grocy](https://sspai.com/link?target=https%3A%2F%2Fgrocy.info%2F)：自托管家庭杂货和家务管理；
-   [Apitable](https://sspai.com/link?target=https%3A%2F%2Fapitable.com%2F)：多维表格，面向 api 的易用低代码平台；开源版维格表，自托管版 Airtable；
-   [Chatgpt Web](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FChanzhaoyu%2Fchatgpt-web)：用 Express 和 Vue3 搭建的 ChatGPT 演示网页；
-   [Code Server](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fcoder%2Fcode-server)：VS Code in the browser
-   [Gitea](https://sspai.com/link?target=https%3A%2F%2Fgitea.com%2F)：轻量级 DevOps 平台；
-   [Kms Server](https://sspai.com/link?target=https%3A%2F%2Fhub.docker.com%2Fr%2Fteddysun%2Fkms): 微软 kms 激活服务器

重点介绍 Grocy 和 Apitable。首先 Grocy，主要功能是家庭的仓库管理，包括：

-   物品的出库入库，通过条形码交互，支持使用手机或专用条形扫描仪；
-   菜谱与食谱，管理自己的菜谱，并在日历中规划食谱；
-   物品临期提醒；
-   购物清单，通过菜谱食谱自动添加；
-   设备、家务、电量管理；
-   提供 API；

目前，我用的最多的是菜谱记录的功能。虽然中式菜肴自由发挥的空间很大，但是面对「适量」、「一勺」、「断生」等词汇，如果某一道菜长时间不做，真的会茫然。而我家蒸箱做的鲈鱼，16 分钟和 18 分钟真的差很多。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-36e58329a703d78fbe90cf6974a7d565.png)

Grocy 自托管家庭杂货和家务管理，APP，菜谱

我使用 Grocy 的时间并不长，一是在想生活是否也需要这么精细的规划，二是物品的出入库并不是特别地方便。但是 Grocy 提供了一个很好的框架，我计划再研究研究，看看是否可以把叮咚等线上买菜的购物记录导入，把出库通过智能手表、NFC 来实现。

**Apitable** 可以看作是 Airtable 的自托管替代品。本地化的产品也有飞书多维表格和维格表，但自托管的好处是容量不受限、api 使用不受限，能更好地被自动化系统使用。Apitable 通过浏览器访问，因此，嵌入 Obsidian 就是常规操作了。

![](https://kenyons.oss-cn-shenzhen.aliyuncs.com/img/1711445074-21b55f6ae70ecff2c53489f0f8278043.png)

Apitable 嵌入 Obsidian

Apitable 也能生成快速提交的表单格式，对移动端兼容良好。

## 总结

以上是我在家庭服务器上做的实践，希望可以为大家提供一些服务器与 NAS 的应用思路。

更多关于自托管服务，请移步：

-   https://github.com/awesome-selfhosted/awesome-selfhosted
-   https://www.reddit.com/r/selfhosted/
-   https://www.reddit.com/r/homelab/

最后，感谢开源社区的贡献者们。
