

# IDocView 前台 RCE 漏洞分析 - 先知社区

IDocView 前台 RCE 漏洞分析



## 0X00 概述

IDocView 是一款能够提供在线文档预览、压缩文件预览、图纸预览、图片预览、音视频播放、协作编辑、同步展示 Web 应用，其`html/2word`存在远程代码执行漏洞

## 0X01 验证

vps 上新建`..\..\..\docview\1.jsp`文件，需要注意一下 VPS 仅支持 linux 且 Linux 在新建文件时需要转义`\`例如

[![](assets/1701606536-ce4e832cae9d9195db9ff92ca422da9c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233153-4328268a-894c-1.png)

或者利用单引号包裹创建

[![](assets/1701606536-fae7e4fe3c1841551e72b977b86d414a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233227-576b8b3c-894c-1.png)

由于在文件开头为。默认为隐藏文件，所以我们如果要查看文件，就需要使用`ls -al`命令即可查看是否创建成功

[![](assets/1701606536-ca128373735e04a78a7b6cd9726477b4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233303-6cf6e80c-894c-1.png)

之后在 vps 上新建任意 html 文件，引入我们创建好的 jsp 文件即可

[![](assets/1701606536-0256f49bdb718445b1c29e958a1b1388.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233327-7b4430f4-894c-1.png)

之后开启 web 服务，在受害机上访问`/html/2word?url=html`地址即可

[![](assets/1701606536-21a264fd728e8e2922d0ade500f9a4e9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233347-876978f8-894c-1.png)

之后访问根目录 1.jsp 即可

[![](assets/1701606536-6dd1e1738c73a739bb6624f80456b2f1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233402-906810cc-894c-1.png)

## 0X02 分析

定位到 html/2word 控制器

[![](assets/1701606536-d5fd6ef7c55549e4f864b7e2bc72f2ba.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233424-9d7163f4-894c-1.png)

这个方法的作用大致为

> 1.  创建 urlhtml 目录
> 2.  根据 URL 计算出一个唯一的标识作为文件名。
> 3.  之后通过 isFile() 判断该文件名下面 index.html 是否存在，注意加了！所以不存在则为真，进入 GrabWebPageUtil._downloadHtml_(url, htmlDir);
> 4.  使用外部命令将 index.html 文件转换为 Word 文档
> 5.  设置响应的 Content-Type 为 Word 文档类型。
> 6.  将生成的 Word 文档作为响应的内容发送给客户端。

跟进`downloadHtml`方法

[![](assets/1701606536-b6685017eea67d9f4bbb4770863ad3c4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233553-d2172166-894c-1.png)

> 1.  69-74 行判断了 url 和文件目录是否为空，以及 url 是否 http 开头
> 2.  76 行创建`URL`对象
> 3.  77-83 行判断文件目录是否存在以及文件目录是否是个文件
> 4.  84 行 进入`getwebPage`方法

[![](assets/1701606536-72e5dd6b3802087071506800d01164a5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233620-e28d66b8-894c-1.png)

> 138-156 行无关紧要的东西  
> 155-176 行建立 URL 连接，并判断状态码

[![](assets/1701606536-0d428939c8657b9746c6d63d3729cedb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233638-ed49648a-894c-1.png)

> 180—196 行，如果跳转，建立新的连接  
> 199-106 行，判断文件存在否，不存在创建、203 行判断是否能写入文件

[![](assets/1701606536-4bdc5e6d5702217e1f02f747a9c4ef1f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233654-f6f1fcd6-894c-1.png)

> 209-222 行，判断文件后缀是否为 html，htm，asp... 如果是获取 URL 连接的输入流给 in，按行读 in 给 strResponse，转成 string 给 htmlContent
> 
> 224 行，这个就比较关键了，跟进`_searchForNewFilesToGrab_`

[![](assets/1701606536-aecba858b680438e8559bca8450cca77.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233714-02f1e24e-894d-1.png)

上述代码的意思呢，就是从 htmlcontnet 中获取 link、href、src 等链接，跟进其中的`_addLinkToFrontier_(urlToGrab, fromHTMLPageUrl);`_,_  
获取到的链接会添加进_filesToGrab 中_

[![](assets/1701606536-53c419418c7de272099e54cefc677649.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233730-0c68b38e-894d-1.png)

思路回到上一层，获取完的 htmlContent 就会被写入文件

[![](assets/1701606536-402a08dbc57f4a0f85621fea5c9c597f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233755-1b2422f0-894d-1.png)

之后我们可以看一下最底层的 else 语句，为最开始的 if 语句的分支，这里就很简单明了的写入文件，等会在提这个分支，不然理不顺

[![](assets/1701606536-de0f1cfa3b30627483d391de21bfb676.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233811-249ddd6c-894d-1.png)

至此，整个`getWebPage`就走完了  
[![](assets/1701606536-951d8bf37532cd2900d72716d7fca15d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233828-2ecc1470-894d-1.png)

回到 downloadHtml 中的 84 行

[![](assets/1701606536-507e1ab3206fd5c3896816f203bba75a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233843-37892c9c-894d-1.png)

后续语句就是通过遍历\_filesToGrab，\_并不断调用 getWebPage 来下载文件，这就是整个的执行过程，漏洞发生的原因也是在于第一次执行`getWebPage时`由于文件后缀是以 html、htm、asp、aspx、php、net 结尾的就会去解析其中存在的 link、href、src，并保存到\_filesToGrab\_中，之后在遍历\_filesToGrab \_中的链接，并通过 getWebPage 下载，在回到先前的 else 分支，jsp 后缀的类型文件正好不在 IF 语句所限制中，所以直接通过 else 语句保存在服务器中了

[![](assets/1701606536-08260d1ca43ae1ecc10669acb6877452.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233859-416d4e14-894d-1.png)

而 filename 又是从 URL 中最后的/开始取，并判断了是否包含/，包含则命名为 default.html，而该系统又是 windwos，所以就刚好可以很巧妙的构造......\\1.txt 的文件名，造成命令执行了

[![](assets/1701606536-a5e0f3cd61d98b512435681492f10131.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231122233916-4b896a04-894d-1.png)

## 0X03 总结

才疏学浅，如有什么问题，希望各位师傅多多指正，欢迎大家一起交流，学习经验
