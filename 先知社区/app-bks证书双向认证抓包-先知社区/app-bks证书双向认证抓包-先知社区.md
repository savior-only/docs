

# app bks 证书双向认证抓包 - 先知社区

app bks 证书双向认证抓包

- - -

### 0x1 概述

首先，网上关于双向认证的文章还是挺多的，关于某灵魂双向认证的文章大家应该都看过，原理就是将 p12 证书和分析 APP 源码得到密码导入 burp 即可抓到包。而本次分享的双向认证是关于 bks 证书抓包的，网上关于 bks 证书双向认证抓包的文章好像挺少的，在查阅时，偶然看到一篇大佬的文章也是关于 bks 的，并且本篇文章后续也用到了大佬推荐的工具，文章附上，大家可以查阅：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1637354&extra=&highlight=%CB%AB%CF%F2%C8%CF%D6%A4&page=1](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1637354&extra=&highlight=%CB%AB%CF%F2%C8%CF%D6%A4&page=1)

### 0x2 获取证书及密码

这里拿到的是一个\*\*\*加固的 APP，并且经测试 frida 可以直接挂上，那么脱壳的过程就不必再说了，网上文章也挺多，这里将壳脱出来后先备用。我们先使用 WIFI 代理进行抓包，抓包发现返回 400 错误

[![](assets/1699406748-0e35f73e1f0dc4127cb4e764060abde8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107122700-e5321fcc-7d25-1.png)

并且可以在上图中观察到经典的 No required SSL certificate was sent，到这里我们就比较清晰地了解到了这是做了双向认证的意思。那么我们现在要做的就是找到证书以及密码，通常来说，证书会存放在 APP 安装包里，密码会存在源代码里或者 SO 文件里。通过查阅资料，了解到 APP 的双向认证会存放的证书格式有：p12、pfx、bks。在翻找 APP 资源文件时，确实也找到了 bks 后缀的文件，以及使用 keytool 将其打包的 jar 包，但是这里踩坑了一晚上，原因是 bks 文件的文件内容并非 bks 真实的内容，分析 APP 代码也未见有对其加解密类似的操作。

因为 APP 安装包里有 bks 文件，因此搜索.bks 定位到这里，可以看到右边 KayakSC.getClientBksPassword() 用来获取密码

[![](assets/1699406748-1d5245286a49fb4d0d7d5b43ed599185.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107122724-f364801c-7d25-1.png)

定位到 getClientBksPassword 函数，可以观察到是 native

[![](assets/1699406748-cf59edc84dafd97ad1c3287dbc8204a0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107122805-0bc41e9c-7d26-1.png)

那么此时直接 hook 这个函数拿返回值就完事了，so 分析就不再去看了。hook 的方法也不再说了，这里我们采用上面大佬文章中的 tracer-keystore 进行 hook

[https://github.com/FSecureLABS/android-keystore-audit/blob/master/frida-scripts/tracer-keystore.js](https://github.com/FSecureLABS/android-keystore-audit/blob/master/frida-scripts/tracer-keystore.js)

下载这个脚本，修改脚本中的 hookKeystoreLoadStream 函数为如下图所示样式：

[![](assets/1699406748-d75d797a57f8922c19bde7a932e8be43.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107122823-16b60a86-7d26-1.png)

也可以直接使用下方修改好的函数替换

```bash
function hookKeystoreLoadStream(dump) {
    var keyStoreLoadStream = Java.use('java.security.KeyStore')['load'].overload('java.io.InputStream', '[C');
    /* following function hooks to a Keystore.load(InputStream stream, char[] password) */
    keyStoreLoadStream.implementation = function (stream, charArray) {
        //console.log("[Call] Keystore.load(InputStream stream, char[] password)")
        var hexString = readStreamToHex (stream);
        console.log("[Keystore.load(InputStream, char[])]: keystoreType: " + this.getType() + ", password: '" + charArrayToString(charArray) + "', inputSteam: " + hexString);
        this.load(stream, charArray);
        if (dump) console.log(" Keystore loaded aliases: " + ListAliasesObj(this));
    }
}
```

最后 frida 启动，如下图所示，获取到证书的密码以及证书的十六进制。

[![](assets/1699406748-bde80ce17e2a72a5c6a930f10be20c84.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107122852-27dda602-7d26-1.png)

这里我对比了一下 APP 里的证书十六进制，发现这根本对不上号，折腾了我一晚上

[![](assets/1699406748-65e9ccffe04d714e2983659969aa8476.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107122934-40fd1f8c-7d26-1.png)

那么我们直接用 hook 到的证书十六进制导入 winhex，然后保存文件为 bks 证书即可，紧接着用大佬里的工具 keystore-explorer，[https://keystore-explorer.org/downloads.html](https://keystore-explorer.org/downloads.html)

使用该工具将导出的 bks 证书转换为 p12，要求输入密码时，输入 hook 获取到的密码即可，最后点击保存按钮

[![](assets/1699406748-5ca1f420523fbc44c39a3b3e1f0adcaf.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107122956-4da6c83c-7d26-1.png)

### 0x3 导入证书抓包

最后就是导入 burp 或 charles 愉快地抓包了，burp 找到设置里的 network-tls 页面，点击下方的 add 按钮

[![](assets/1699406748-cf95530d685c9723e05b02c66779a00a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107123028-60d0464a-7d26-1.png)

随后按后续的要求导入转换为 p12 格式的证书（还是原先 winhex 保存的 bks 文件）以及密码，导入成功后直接抓包，可以看到心爱的 HTTP 报文了

[![](assets/1699406748-2b2bbb6bc41d6bb4cdd9796c41347588.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231107123041-68d12ee0-7d26-1.png)
