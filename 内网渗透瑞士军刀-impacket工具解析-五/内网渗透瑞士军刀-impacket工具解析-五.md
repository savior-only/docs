---
title: 内网渗透瑞士军刀-impacket工具解析（五）
url: https://mp.weixin.qq.com/s/unv6sY296gfrsASZO_xr2w
clipped_at: 2024-03-27 00:33:32
category: default
tags: 
 - mp.weixin.qq.com
---


# 内网渗透瑞士军刀-impacket工具解析（五）

  

  

  

**前言**

preface

在前面四篇文章中，我们向大家介绍了impacket中的两种认证协议的详细实现原理，以及Windows网络中最常用的协议SMB和RPC，这四种协议都属于impacket库中最基础的底层协议，利用这些底层协议的组合，可以实现各种功能强大的内网渗透工具。

  

**从本篇开始，我们将基于impacket内置工具来逐一介绍其功能和代码细节。**

  

**PSexec和psexec.py**  

  

    Psexec是一款由Mark Russinovich在1996年开发的Sysinternals系列工具之一，主要功能是用于在远程计算机上执行命令。

  

    在当时，主流的远程运维方式主要还是依赖于Telnet和SSH服务，这两种服务在Windows主机中默认没有安装，而在后来出现的Windows远程桌面服务对带宽的要求有比较高，所以只有700Kb且只需要通过SMB服务进行通信的Psexec工具很快就得到了从事Windows运维以及安全从业者们的关注。

  

    Sysinternals中的Psexec是一款闭源工具，在后来开源爱好者们发布了一款模仿Psexec功能的开源项目RemCom，这个项目托管在Github\[https://github.com/kavika13/RemCom\]上，有兴趣的小伙伴可以阅读一下项目的源码。RemCom工具主要由两个部分组成：RemCom和RemComSVC，前者作为客户端运行，后者为一个Windows服务程序，作用是接受客户端指令，在所安装的机器上执行指令并向客户端返回执行结果。

  

    impacket工具出现后，作者*@agsolino*便将RemComSVC的二进制程序嵌入到Python中，并基于impacket中的DCERPC和SMB协议重新实现了RemCom的功能，并将其命名为psexec.py，这也是我们今天介绍的第一个工具。

  

 **RemComSVC解析**  

  

    首先psexec会利用SMB连接将RemComSVC程序上传到服务器上，再利用SCMR服务管理RPC在服务器上创建一个服务，服务的二进制程序指向上传的RemComSVC文件路径。

  

    我们可以从服务端代码来看这个服务所进行的操作。

  

    服务入口为\_ServiceMain函数中，我们可以看到这个服务会启动一个CommunicationPoolThread线程来处理客户端发送的请求，并且在主线程中调用WaitForSingleObject方法监听服务退出事件，如果服务退出就将自身删除掉。

  

![图片](assets/1711470812-378e7ebf4e773fc366880873bd38c594.webp)

  

再跟进一下CommunicationPoolThread，CommunicationPoolThread函数是一个死循环，首先调用CreateNamedPipe创建一个命名管道，管道名称是一个常量，定义如下

  

**代码块**

  

紧接着创建一个新的线程CommunicationPipeThreadProc，并且将上一个管道的句柄作为参数传给CommunicationPipeThreadProc

  

![图片](assets/1711470812-4f800b7901909891e36ca940041c7e30.webp)

  

    再跟进一下CommunicationPipeThreadProc，在这里调用ReadFile函数读取上一个管道的数据并赋值到msg结构体中，ReadFile在这里将阻塞，直到有客户端来连接之前创建的RemComCOMM管道，并向该管道写入数据。读取到客户端发送的数据之后，再调用Execute函数来处理客户端消息msg，最后将处理的返回结果写入emComCOMM管道。

  

![图片](assets/1711470812-64def5e2c8f557de10839e9d94c164ed.webp)

  

  RemCom利用了两个类来进行数据通讯RemComMessage和RemComResponse，两个类的定义如下：

  

**代码块**

  

  RemComMessage表示客户端请求，其中szCommand为执行命令，szWorkingDir表示工作目录，dwPriority为优先级，dwProcessId表示客户端的PID，szMachine为客户端机器名。RemComResponse为服务端响应，包含两个字段：dwErrorCode和dwReturnCode分别表示错误码和命令执行的返回值。

  

  再跟进Execute方法，可以看到开头部分又创建了管道，从注释里可以知道创建三个命名管道分别代表标准输出、标准输入和标准错误。

  

![图片](assets/1711470812-c1e7805da6cdb3557e0b9107bec899bd.webp)

  

  跟进CreateNamedPipes方法可以发现，管道的名称有一定规律，这三个命名管道前缀都是常量，定义如下

  

**代码块**

  

  紧接着将其与客户端传入的参数pMsg->szMachine和pMsg->dwProcessId，也就是机器名和PID拼接

  

![图片](assets/1711470812-5a2b04b260d0a72d5218c7c2bd076e3a.webp)

  

  最后调用CreateProcess方法创建进程，进程的命令参数为客户端消息中的szCommand字段，标准输出、标准输入和错误分别重定向到了刚刚所创建的三个命名管道上。

  

![图片](assets/1711470812-b017fa6c16f2343568d5f332bced9d2b.webp)

  

  从整个过程可以看到，这里实现交互式命令执行主要依赖于4个命名管道，其中RemCom\_communicaton管道主要用于消息传输，其他三个管道用于将命令执行的输出重定向，以便于客户端通过命名管道进行读取。

  

**psexec代码解析**  

  

    impacket工具中的入口参数都比较类似，大致分为几个部分，首先

    PSEXEC类中，主要是一些常规的参数的传递、没有进行特殊的操作。

  

![图片](assets/1711470812-570206d7b2de52f451f0bef2042e8a08.webp)

  

  调用的入口方法为run方法，我们跟进该方法，开头部分主要是对rpc进行了初始化，这里连接的rpc端点为\\pipe\\svcctl，这个是用于SCMR远程服务管理的RPC端点，设置好相关参数之后再调用doStuff进行处理，

  

![图片](assets/1711470812-2a07292d5e9121cd7982f2ddbeb128fc.webp)

  

![图片](assets/1711470812-8b0f0ee83df7ab14f1ce916aaa2a1f88.webp)

  

继续跟进doStuff方法发现这里调用了rpc的connect的方法，在之前的RPC介绍的文章我们说过，connect方法根据RPC的底层传输方式都是不同的，根据前面stringBinding的格式可以知道从RPC工厂返回的是以SMB为底层传输方法的RPC，那这里的connect方法实际上是建立了一个SMB连接

  

![图片](assets/1711470812-0f18bd8f218cf570ce4b3ad8ed9d20c1.webp)

  

继续跟进发现，在这里调用了一个外部库中的类ServiceInstall，分为两种情况，当self.\_\_exeFile为空时参数为remcomsvc.RemComSvc()，不为空时就打开文件，并将文件句柄作为参数，实际上这里就是利用SCMR对RemComSVC进行安装的部分，这里提供了一个替代RemComSVC的选项

  

![图片](assets/1711470812-047bc2980241566f05092a0e0b0b5250.webp)

  

点击RemComSvc可以看到这里是将RemComSVC的二进制程序硬编码到了代码里面。

  

![图片](assets/1711470812-35bfb86d401ee819892c73f19978ad86.webp)

  

跟进下ServiceInstall类，初始化方法中对serviceName和binary\_service\_name参数进行判断。这两个参数分别代表目标机器上安装的服务名称和服务程序落地的文件名，可以看到在没有指定这两个名称的情况下自动生成名称的规则。服务名为4字节长度的ASCII字母，服务落地的名称为8字节长度的ASCII字母。

  

![图片](assets/1711470812-147b4ef32630647f8c75008f71eeb498.webp)

  

查看install方法可以发现这里有一个小细节，psexec会通过rpc来寻找一个可写的共享路径，并将服务文件复制到这个路径中。

  

![图片](assets/1711470812-bd115a6ec9d3ceedd35cf7be30fb8427.webp)

  

随后，打开服务管理器，根据指定的服务名、文件路径远程创建并启动服务。

  

![图片](assets/1711470812-af80908a3027c941f2986e1b6100936c.webp)

  

再回到psexec中，在这里会检查是否需要将文件复制到目标服务器中，并修改命令内容，这里主要作用是可以执行用户自己定义的可执行程序。

  

![图片](assets/1711470812-daa38699dabf542dcd42499489810756.webp)

  

随后可以看到连接了目标服务的RemCom\_communicaton共享，再构造一个RemComMessage数据包，对数据包进行赋值之后写入到命名管道中。

  

![图片](assets/1711470812-faa75266b2b0588483c08fd0a00d980e.webp)

  

我们查看RemComMessage可以看到字段和RemCom中定义的都是一致的

  

![图片](assets/1711470812-4d794b9421d4310118f04c9ffa14bd29.webp)

  

在这里客户端传输的机器名实际上是随机生成的4字节的字母。继续跟进，在这里实例化了三个类，分别为RemoteStdInPipe、RemoteStdOutPipe、RemoteStdErrPipe，并调用了三个类的start方法，从注释中我们可以知道这里是创建了三个管道的线程。

  

![图片](assets/1711470812-0f44603304cb877273f3e0d91668da6b.webp)

  

三个类都是继承于Pipes类，而Pipes类又是继承自Thread类，其中RemoteStdOutPipe和RemoteStdErrPipe类比较简单，都是在run方法中通过一个死循环读取各自管道的数据。

  

![图片](assets/1711470812-bce5b9ccdeb53af740023bc8ce9c9d03.webp)

  

![图片](assets/1711470812-db508396d7120f2a3a7c968f619cfe85.webp)

  

RemoteStdInPipe相对比较复杂，因为这里需要实现一个输入终端的逻辑，所以在内部创建了一个RemoteShell实例，

  

![图片](assets/1711470812-fead37ef0062bca61c149c1b32784f4c.webp)

  

RemoteShell类继承自Cmd，这是impacket中用于实现命令行的一个类，我们就不做具体具体分析了，简单来说，通过这个类创建的命令终端，输入的命令都会被分分配到相应的方法进行处理，例如输入foo bar就会调用do\_foo方法处理bar参数，如果没有找到do\_foo方法则执行default方法。

  

回到RemoteShell类中，do\_\*方法一共又有5个，do\_help对应的是help命令，作用是输出psexec的帮助信息。  

  

![图片](assets/1711470812-e5387820f9767c568ec2b9cfc8a138af.webp)

  

do\_lcd方法对应lcd命令，作用为切换当前工作目录。

  

![图片](assets/1711470812-af359efad9e26962df1cc78bee0d5a8f.webp)

  

do\_lget方法对应lget命令、作用将目标服务器上的文件下载到本地，从代码中可以看到调用的实际上是SMBConnection.getFile方法，传入的共享路径为安装服务的共享路径。

  

![图片](assets/1711470812-b4168d8b050d0d88f2f1014bbde0034d.webp)

  

do\_shell方法很简单，作用就是在本地执行一条系统命令。

  

![图片](assets/1711470812-0faac3217f474cdc1d720e69cd2dd514.webp)

  

再来看默认执行的default方法，再default方法中则是把输入都写到标准输入命名管道中。

  

![图片](assets/1711470812-f0620a3b11fbc88a95d5a1ab37286a24.webp)

  

**总结**  

  

psexec利用DCERPC和命名管道巧妙地实现了Windows平台远程交互式shell，但是工具本身还是存在许多明显的特征。首先是psexec中默认的服务程序为硬编码内容，并且该文件需要落地到服务端，众多杀软和EDR早已将这个文件特征列入到特征库中，其次是服务创建的参数，服务名及可执行文件名称都是随机生成，正常服务中这些参数都应该是一个有意义的字符串，最后是访问共享路径，psexec通过4个命名管道与服务端进行通信，其中用于初始信息交换的命名管道为硬编码名称“RemCom\_communicaton”，用于输入输出重定向的其他三个管道也具有一定的硬编码特征。

  

当然、具有OPSEC意识的攻击者可以通过魔改RemComSVC和psexec的方式来消除以上特征，但仍然绕不开网络登录、创建服务，访问管道等过程，这时可以以用户身份为线索追溯服务创建及共享访问的行为来确定系统是否遭受此类工具进行攻击。

  

  

[](http://mp.weixin.qq.com/s?__biz=MzkxNTEzMTA0Mw==&mid=2247493393&idx=1&sn=8fce0054925cc69d6eabc827042a43f6&chksm=c16178ddf616f1cbf9c6a61d8fcba4b3233d83cf4acb3446c97088efd909a456a356751dc136&scene=21#wechat_redirect)

  

[](http://mp.weixin.qq.com/s?__biz=MzkxNTEzMTA0Mw==&mid=2247493468&idx=1&sn=e5137d7a47ec7de4b39f1a1fe2789b3a&chksm=c1617890f616f1866beea59a3894ddeb4e07107976212376ef7476b0dcf149bd7e8ab26582a0&scene=21#wechat_redirect)

  

[](http://mp.weixin.qq.com/s?__biz=MzkxNTEzMTA0Mw==&mid=2247493609&idx=1&sn=5af3d71755ede7435c10fd280a5fbaf0&chksm=c1617825f616f133ccd617e98534ddea2d43d332ba3540e00bd46a612e583e873c60d4fb63bd&scene=21#wechat_redirect)

  

[](http://mp.weixin.qq.com/s?__biz=MzkxNTEzMTA0Mw==&mid=2247494259&idx=1&sn=fabc3b5aa92e25b64e38dfd54468b70a&chksm=c16175bff616fca95206483b15c4679ad12c159ab15bdd10bd9b6372ab71d070f6327bf476fc&scene=21#wechat_redirect)