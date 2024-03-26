---
title: 内网渗透瑞士军刀-impacket工具解析（七）
url: https://mp.weixin.qq.com/s/iDXaectvIUPGkcE3A7cZhw
clipped_at: 2024-03-27 00:34:54
category: default
tags: 
 - mp.weixin.qq.com
---


# 内网渗透瑞士军刀-impacket工具解析（七）

  

  

  

**前言**

preface

Atexec是Impacket中被高频使用的一种横向工具，该技术手段主要通过任务计划实现，本文带大家对其中的技术细节进行简单的介绍。

  

**0x1 MS-TSCH**  

  

Atexec的原理并不复杂，通过远程调用对方主机的**任务计划程序**（Task Scheduler）创建一个定时任务，然后通过手动触发该定时任务将执行结果写入文本中，再利用smb共享去读取执行结果。

  

MS-TSCH：任务调度程序服务远程协议，该协议用于注册和配置任务以及查询远程计算机上运行的任务的状态，主要由三个独立的RPC接口组成

  

-   网络时间表（ATSvc）
    
-   任务计划程序代理 ( SASec)
    
-   Windows Vista 操作系统任务远程协议 ( ITaskSchedulerService )
    

  

这三个接口都使用RPC作为远程配置和管理任务的传输，且它们的功能是逐级递增的。现如今，ITaskSchedulerService 成为了大部分计划任务操作的首选。同时，ITaskSchedulerService 接口不通过远程注册表和文件系统协议，而是通过XML来指定任务的配置。

  

该三个接口在Impacket中均有实现

**Atsvc**  

在atsvc.py中实现

接口的UUID：

  

![图片](assets/1711470894-fc707cb91388b09b223b7e3b7b9770d7.webp)

  

相关的增删改查的结构体和请求：

  

![图片](assets/1711470894-dbff2d37785241cabd8bb0407647654f.webp)

**SASec**  

在sasec.py中实现

接口的UUID和相关的结构体

  

![图片](assets/1711470894-5a754401aa1af0b420473922904df78f.webp)

  

![图片](assets/1711470894-2ad3e019f16e408e7b3a112e065c38b6.webp)

**ITaskSchedulerService**   

在tsch.py中实现

这是平时红队工具中最常用也是功能最完善的计划任务接口，Atexec中默认的也是这个接口

  

![图片](assets/1711470894-79444ee3c1e4fed56ac75ab81b214b76.webp)

  

  

**0x2 流程分析**  

  

首先从主函数入口开始运行，打印版本和banner信息，利用argparse库解析输入的参数。中间经过一系列的参数解析和日志DEBUG判断，最后来到最后两个最关键的函数

  

![图片](assets/1711470894-5b994fac0da5ec97f511e718455cb3d9.webp)

  

第一个函数用于初始化定义的TSCH\_EXEC类对象，第二个调用实例的play方法，我们来到这个类对象的play方法，

首先格式化连接参数，使用SMB传输的RPC服务通过连接命名管道\\pipe\\atsvc来创建和删除计划任务，然后利用DCERPCTransportFactory工厂来实例一个rpctransport

  

![图片](assets/1711470894-af3d179a32cd14fad72a91d0b82b4241.webp)

  

![图片](assets/1711470894-d4ee56aa4440a9225aefbfc18396a0c6.webp)

  

然后根据凭据的种类来设置凭据，进入doStuff方法。在doStuff方法中，获取了一个dce对象，并绑定相应的UUID接口，也就是ITaskSchedulerService 接口。随机生成一个文件名并将这个文件名与tmp拼接作为临时文件，将后续执行结果写入，同时这个临时文件名也将作为后续创建的计划任务名。

  

读取我们输入的命令并拼接，可以看到利用cmd执行结果，并写入Temp目录下的临时文件中

  

![图片](assets/1711470894-a241fd0eddf797649c7ab9ce9a93bb8b.webp)

  

后面定义的xml则是创建的计划任务的具体配置，我们主要关注它的触发器和执行的命令

触发器：

  

![图片](assets/1711470894-6912db5ce5298c672d234443ad9557d3.webp)

  

它指定了起始时间，从2015年7月15号开始每隔一天来周期性的触发这个任务

执行命令：

  

![图片](assets/1711470894-def6fc098abc5793e20ee2d00a55905a.webp)

  

第一个参数是执行的命令，第二个参数是给命令的参数

  

![图片](assets/1711470894-f871927bd141dfb5a460d208b8a73671.webp)

  

后续利用hSchRpcRegisterTask来注册计划任务同时打印日志输出

  

![图片](assets/1711470894-372587a6dea4a60ea9f7d3e5913d6875.webp)

  

前面我们知道，设置的触发器是每天一次的执行，这并不符合我们的需求，于是后续利用hSchRpcRun来直接进行触发

  

![图片](assets/1711470894-9e64892f2cf7f7c408559c1281591bf9.webp)

  

后续就是很常规的利用hSchRpcGetLastRunInfo获取计划任务的详细信息，利用hSchRpcDelete删除执行之后的计划任务，然后进行一次SMB连接读取输出的结果，在读取成功之后删除临时tmp文件

  

![图片](assets/1711470894-ca2d9f802d2566824dd19d629d90e27a.webp)

  

最后整个脚本的流程结束

  

**0x3 总结**  

  

Atexec通过创建计划任务来巧妙的执行命令，但相较于Impacket中的其他的命令执行工具，它也有很多不足。例如它不能获取一个交互式的shell只能单步执行命令。同时后面文件逻辑处理也有些许不足，如果计划任务中间过程出现问题，生成的结果文件则不会被删除，如下：

  

![图片](assets/1711470894-a92c15ee989d58cb05e1a6ed0e00ca1a.webp)

  

![图片](assets/1711470894-5c0ab4974c6a59c7c5bfb3fdb462ff52.webp)

  

除此之外，远程创建计划任务本身也是一个比较敏感的操作，很容易被杀软监控或者阻拦。我们可以通过其他方法来创建计划任务，例如通过COM组件等。

  

  

[](http://mp.weixin.qq.com/s?__biz=MzkxNTEzMTA0Mw==&mid=2247494259&idx=1&sn=fabc3b5aa92e25b64e38dfd54468b70a&chksm=c16175bff616fca95206483b15c4679ad12c159ab15bdd10bd9b6372ab71d070f6327bf476fc&scene=21#wechat_redirect)

  

[](http://mp.weixin.qq.com/s?__biz=MzkxNTEzMTA0Mw==&mid=2247494429&idx=1&sn=9f2bca983297a5fc968547f4663d2ac7&chksm=c16174d1f616fdc77bc935ac63229ac7f49ffd27686736755e989fdcc28764daa9bb548813ec&scene=21#wechat_redirect)

  

[](http://mp.weixin.qq.com/s?__biz=MzkxNTEzMTA0Mw==&mid=2247494508&idx=1&sn=f9e58ae8f443688da2ca34d7c0bccf60&chksm=c16174a0f616fdb63dd79b34dc047b0f4fbaf6a88c02f3789a7ad9df54709096b7a98fd9806b&scene=21#wechat_redirect)