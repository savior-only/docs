---
title: Hikvision 海康威视数据库账号解密工具
url: https://mp.weixin.qq.com/s?__biz=MzkzNzI4NDQzMA==&mid=2247494813&idx=2&sn=b6093b443b4bd71eaa46ec0be9169dc8&chksm=c2936abcf5e4e3aaef6ffd573351d4d6b8d5d0aafe128219e93b73b82f1e4029f066bd7d96c5&scene=126&sessionid=1697942000&key=debaf56e0df4198f87bb416a8c39e3effed1f6aed5606abd55bf6316632414c7127d8fdcd09276c502a2c7215ee010ad1df78511b9ed66c67ea572b7142d674cb974fa752df849916b6820fe52a689cfcb880638a53959efc25531b54dbfa9a5ac2a72e16dec8bbfff4cf66dc5300c1a52c401b6ec100f57253ed166821f67da&ascene=15&uin=NTY2NTA4NjQ%3D&devicetype=Windows+10+x64&version=63060012&lang=zh_CN&session_us=gh_6f2a00098080&countrycode=AL&exportkey=n_ChQIAhIQm5h3%2FMBH3jXhwF0xuwKvHxLuAQIE97dBBAEAAAAAAEbyGP7WqhgAAAAOpnltbLcz9gKNyK89dVj0VK%2FSYSRcKXeQeTfzIOQl5ggQu3byi1K%2F20PxJglvMTVATl9qeVlVpBTmAxz5LNAb%2Byg4f%2BgjyOVtc7StaeJ4g5BexoqnIncYPRd9PVp8eHytCF0QFBWlDqVn86s79YFaz7rQphAch6PzpAgxb06oW6fLkRZaxlX5Y8EQhiprMDskX%2BE5d85jlUgURQ%2F9KaAUm7MWAnw2pXwhLPIIbC0EoeOhtcnTVMGBUyhYfGWPly%2FRj95TUwrAR0wgYjXTlQ1Gvyf%2BgnBQtz4%3D&acctmode=0&pass_ticket=tb9HPtTGilJ2jU7A916YMQ2r580l0Ms4YgUlkjK8hPiaog12Sh8eKUeXIA6vSUHk&wx_header=0&fontgear=2
clipped_at: 2024-04-01 17:12:54
category: default
tags: 
 - mp.weixin.qq.com
---


# Hikvision 海康威视数据库账号解密工具

**工具简介**

这个工具可以用来解密 Hikvision 海康威视加密的数据库账号和密码，适用版本： 海康威视 ivms-8700 

**使用教程**

找到数据库配置文件，如图：  

![图片](assets/1711962774-e8aec369781a28986405be07c3769a30.webp)

然后输入加密的账号和密码，如图：

```plain
java -jar HikvisionDecode-1.0-SNAPSHOT.jar xxxxx
```

![图片](assets/1711962774-e41a04218728c0e23b77f2c217c441b1.webp)

**下载地址**

1\.  链接：https://pan.quark.cn/s/ee6c94ed056b

2.  https://github.com/baogod404/HikvisionDecode
