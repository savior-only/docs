---
title: Burp Suite 插件开发（一）
url: https://mp.weixin.qq.com/s?__biz=MzI3OTM3OTAyNw==&mid=2247485775&idx=1&sn=fdfda883b6e49656053f576ca0756a8e&chksm=eb49e473dc3e6d651d27830fb6c9767053f309935d35391c2b21a85b7944197ec7be5c50b7cc&mpshare=1&scene=1&srcid=0311h6kq7SHeaqpsAoeNOxRP&sharer_shareinfo=75a387bb249d18871325db2796272aef&sharer_shareinfo_first=75a387bb249d18871325db2796272aef#rd
clipped_at: 2024-03-31 16:08:21
category: temp
tags: 
 - mp.weixin.qq.com
---


# Burp Suite 插件开发（一）

## Burp Suite 介绍

Burp Suite 是一款用于攻击和测试 Web 应用程序的集成平台，它包含了多种工具，如代理、重发器、扫描器、爬虫、Intruder、Repeater 等，可以协同工作，共享信息，支持各种攻击和检测方法。

Burp Suite 的插件是一种扩展 Burp Suite 功能的方式，可以通过 Burp Extender 模块来安装和管理。插件可以由社区用户创建和维护，也可以由自己编写。插件可以修改 HTTP 请求和响应，发送额外的 HTTP 请求，自定义 Burp Suite 的界面和功能，添加额外的扫描检查，访问 Burp Suite 的信息等。

## 配置环境

首先我们需要安装 IDE 工具，可以下载 IntelliJ IDEA、Atom、Netbeans、Eclipse 等。

**Java 环境**：

Java 环境我们将在后续文章中以使用 IDEA 工具编写为例介绍。

**Python 环境**：

Burp Suite 依赖 Jython 来支持 Python，需要下载 Jython 并配置 Burp Suite 位置。

在 Burp Suite 页面加载 Python 插件需要选择扩展类型为 Python 并指定 Python 插件文件。

## API

**Extender API**：

Burp Suite 的 Extender API 是一套用于创建 Burp Suite 扩展的 Java 接口，它可以让攻击者使用自己或第三方的代码来扩展 Burp Suite 的功能，例如修改 HTTP 请求和响应、发送额外的 HTTP 请求、自定义 Burp Suite 的界面和功能、添加额外的扫描检查，访问 Burp Suite 的信息等。

所有接口可以访问 Burp Suite 的官方文档查看:https://portswigger.net/burp/extender/api/index-all.html。

**Montoya API**：

Montoya AP I 和 Extender API 都是用于创建 Burp Suite 扩展的 Java 接口，但是 Montoya API 是 Extender API 的升级版，它提供了更多的功能和优化，例如：

-   Montoya API 将 Extender API 中的一些接口进行了分类和重构，使得扩展的开发更加清晰和方便。
    
-   Montoya API 增加了一些新的接口，如 IBurpExtension、IExtensionStateListener、IExtensionHelpers 等，可以让扩展更好地与 Burp Suite 的核心功能和状态进行交互。
    
-   Montoya API 支持使用 Maven 或 Gradle 来创建和构建扩展项目，可以方便地管理依赖和版本。该 API 可以访问 Burp Suite 的官方文档查看：
    

https://portswigger.github.io/burp-extensions-montoya-api/javadoc/burp/api/montoya/MontoyaApi.html

## BApp Store

BApp Store 包含由 Burp Suite 用户编写的 Burp 扩展，以扩展 Burp 的功能。

攻击者可以通过 Burp Extender 工具中的 BApp Store 功能直接在 Burp Suite 中安装。也可以访问 Burp Suite 提供的在线网站 (https://portswigger.net/bappstore)，以便离线安装到 Burp Suite 中。

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

## 插件编写

以使用 Java 语言编写为例，启动 IDEA 环境并创建一个名为”Burps”的新项目：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

*注意：需要将语言更改为 Kotlin 来支持*。

接下来需要添加 Burp Suite 的接口，按住 CTRL+ALT+SHIFT+S 打开”项目结构”菜单并点击”+”号选择”From Maven…”：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

搜索“net.portswigger.burp.extender:burp-extender-api:2.1”并安装：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

点击”OK”按钮并将该库应用到创建的插件中：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

接下来就需要编写代码了，首先需要创建名为”Burp”的新包。右键 src 目录，点击”New”按钮并选择”Package”以创建：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

然后我们需要创建一个 Kotlin 的 Class 类，右键”Burp”并点击”New”按钮即可创建：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

在该文件中编写一个简单的 Hellow World：

```plain
package Burp
import burp.IBurpExtender
import burp.IBurpExtenderCallbacks
import java.io.PrintWriter

@Suppress("unused") // Remove warning, the class will be used by burp
class BurpExtender : IBurpExtender {
    override fun registerExtenderCallbacks(callbacks: IBurpExtenderCallbacks) {
        // Let's wrap stdout and stderr in PrintWriter with auto flush
        val stdout = PrintWriter(callbacks.stdout, true)
        val stderr = PrintWriter(callbacks.stderr, true)

        // Set our extension name, this will be display in burp extensions tab
        callbacks.setExtensionName("Wu Hu")
        stdout.println("Hello world!")
        stderr.println("OMG! Error!!")
    }
}
```

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

接下来我们需要构建 Jar 文件，按住 CTRL+ALT+SHIFT+S 打开”项目结构”菜单，选择”Artifacts”并点击”+”号以新建 Jar 文件：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

然后选择目录为插件项目名称：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

然后可以勾选”Include in project build”，来为每个生成自动创建 Jar 文件：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

最后，键入 CTRL+F9 即可构建项目：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

在 Burp Suite 的扩展插件选项卡界面添加自定义的 Burps 插件：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

我们可以在”Output”和”Errors”选项卡中查看自定义的扩展插件消息：

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

![图片](assets/1711872501-37d80127b73f829661c0d17b431e0b18.svg)

## 简单代码示例

如需要更多的代码示例，可以访问以下两个 Burp Suite 官方仓库查看：

https://github.com/PortSwigger/burp-extensions-montoya-api-examples

https://github.com/PortSwigger/example-hello-world

## GitHub 仓库

**Awesome Burp Extensions**：

https://github.com/snoopysecurity/awesome-burp-extensions

\- END -