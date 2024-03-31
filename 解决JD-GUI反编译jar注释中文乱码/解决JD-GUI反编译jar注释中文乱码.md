---
title: 解决JD-GUI反编译jar注释中文乱码
url: https://mp.weixin.qq.com/s?__biz=MzUxNTYzMjA5Mg==&mid=2247550851&idx=1&sn=e6ad1bb1a8f33eb04395c1504bbc6721&chksm=f9b1f930cec67026e76b19880f1dc61b4349a29795e10d44be9722e6bf6fb9f71d3d6cd673eb&mpshare=1&scene=1&srcid=0120ulAsxM7uERa1vc306j25&sharer_shareinfo=d770d40e2b745e7ed33f4916efa36401&sharer_shareinfo_first=d770d40e2b745e7ed33f4916efa36401#rd
clipped_at: 2024-03-31 19:19:44
category: temp
tags: 
 - mp.weixin.qq.com
---


# 解决JD-GUI反编译jar注释中文乱码

  

  

  

问题现象

  

  

  

我们平时会使用JD-GUI反编译其他项目jar包来排查源码里有什么问题，如果源码中都是英文注释倒还好，**要是有中文注释，就会遇到中文乱码的情况**。

![图片](assets/1711883984-c36f1be76587f31ee066419a83475a4c.png)

如上图所示，如果出现这种情况就很难通过注释快速了解代码逻辑，影响问题排查进度。如果是在eclipse或者idea中，一般直接在设置里配置编码格式为UTF-8重新打包即可，但目前这里显然不太现实。那么**如何解决这个问题，以下提供两种解决方**案。

  

  

  

解决方案

  

  

  

**2.1 命令行方式**

我们知道JD-GUI也是一个java程序，启动时除了直接双击.exe文件，还可以在cmd里面直接使用java -jar xxx的方式，这样只需要指定file.endocing=UTF-8即可：

![图片](assets/1711883984-814c00c21112bfcd1f01480113d7aaa2.webp)

**2）**在弹出的窗口中输入java -D'file.encoding=UTF-8' -jar %JD\_GUI\_HOME%\\jd-gui.exe，输入完命令后就会弹出jd-gui的页面

![图片](assets/1711883984-a684786f2ccd09102ee11c27efc9f7b5.webp)

**3）**然后选择打开需反编译的jar包即可

![图片](assets/1711883984-fe605701a4533232dd52faf9d0514499.webp)

![图片](assets/1711883984-65ba72da82913ca481a8e43caeaf9aae.webp)

如图所示，可以看到原来乱码的地方都已经显示正常了。

**2.2 脚本方式**

每次打开powershell并输入这一串命令显然很麻烦，其实可以通过脚本的方式，并绑定到jd-gui的快捷方式图标来自动配置并打开jd-gui：

**1）**编写脚本

```plain
java -D'file.encoding=utf-8' -jar C:\ProgramDevs\jd-gui-windows-1.6.6\jd-gui.exe

powershell脚本后缀为.ps1，保存为C:\ProgramDevs\jd-gui-windows-1.6.6\jd-gui.ps1
```

**2）**测试脚本

![图片](assets/1711883984-840daec80c32e774ecd7436d7e7f1a8e.webp)

**3）**绑定至jd-gui快捷方式

![图片](assets/1711883984-8bf2e4f0f8c8e48768a303b5ef79b1dd.webp)

**4）**双击快捷方式即可打开jd-gui窗口，且中文不会乱码

![图片](assets/1711883984-90442835c0bf1efb62101dd08d8c7040.webp)

**5）**然后选择打开需反编译的jar包即可(同第一种方式步骤3)