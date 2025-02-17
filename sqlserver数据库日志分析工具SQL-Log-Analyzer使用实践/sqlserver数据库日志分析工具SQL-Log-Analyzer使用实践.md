---
title: sqlserver 数据库日志分析工具 SQL Log Analyzer 使用实践
url: https://mp.weixin.qq.com/s?__biz=MzUxNTYzMjA5Mg==&mid=2247547006&idx=1&sn=6b8acc05a999e87b367e81cb7ccfd09b&chksm=f9b1c6cdcec64fdb55dc84592f2f602f23b397f8c8689d3137f8ce597ae9182b47812f8ca234&mpshare=1&scene=1&srcid=01206CIz1M7sNnGgQRkjRXiw&sharer_shareinfo=84773dfeb6cf7f2b5dc55efea72b4b75&sharer_shareinfo_first=84773dfeb6cf7f2b5dc55efea72b4b75#rd
clipped_at: 2024-03-31 15:40:50
category: temp
tags: 
 - mp.weixin.qq.com
---


# sqlserver 数据库日志分析工具 SQL Log Analyzer 使用实践

  

背 景

某公司某账号突然间被删除，客户紧急召开会议，需我侧协助分析并查明 sqlserver 中的数据是何时被谁删除。  

由于 sqlserver 本身未开审计，无法从字典进行溯源，只能分析当时的事务日志，但是微软自带的分析软件收费也无法使用，于是从第三方 sqlserver 分析软件着手，在整个过程试了 log explorer，但由于客户版本是 sqlserver2012,log explorer 低版本不支持 sqlserver2012 的解析，log explorer 高版本支持同时不能在线分析，后面发现工具 SysToolsSQLLogAnalyzer 支持 2008 及 2012 等数据库版本，可以做在线查看，并且可以恢复被删除的数据。

**本案主要就是**使用 sql-log-analyzer 工具对 sqlserver2008 的实践，至于客户 sqlserver2012 的分析，涉密，但分析雷同，下面就是对该工具的使用介绍。

  

SQL Log Analyzer 工具使用

  

**2.1 环境准备**  

安装 windows server 2008 并同时安装好 sqlserver 2008 数据库。  

![图片](assets/1711870850-2d3e514b156eb14235b97566c04d25a4.webp)

# **2.2 创建 SqlServer 数据库**

创建 sqlserver 数据库 ywdb，并创建表 t1，模拟插入二条记录和删除一条记录：

![图片](assets/1711870850-7c112a04f4d454e70c9b52c91949ac37.webp)

# **2.3 SQLLogAnalyzer 分析工具安装及使用**

## **1）**下载 SysTools SQL Log Analyzer 并安装，安装完后，在桌面点击

![图片](assets/1711870850-62694f60fdcc112af7fcb814ec36a985.webp)

## **2）**以 sa 账号连接被分析数据库

-   连接数据库实例
    

![图片](assets/1711870850-5005f86d663f10765d45c5a44beaf822.webp)

-   选择好将要分析的数据库 DB
    

![图片](assets/1711870850-e671d5ec9c4e67f0af6b1170386b32f4.webp)

-   生成分析的结果数据
    

![图片](assets/1711870850-662a39b84d3f03ab5d58c21341bda8d6.webp)

## **3）**分析后的结果数据可以进行导出

![图片](assets/1711870850-c665e8e969c4d72978c50d320f9870c8.webp)

## **4）**结果数据会以指定的需求生成

![图片](assets/1711870850-e86f4544fa3690d5f62cdc1ef7bbe807.webp)

# **总 结：**

在日常工作，若遇见 sqlserver 有误操作或溯源问题时，可通过工具 SysTools SQL Log Analyzer 进行分析，以便能找到引发问题的根因，最终解决问题，本案分析比较简单，希望能给大家提供一个分析思路。