---
title: mongoDB 在 windows 下使用密码登录
url: https://zhuanlan.zhihu.com/p/612494679
clipped_at: 2024-03-26 17:52:26
category: default
tags: 
 - zhuanlan.zhihu.com
---


# mongoDB 在 windows 下使用密码登录

看了好多文章，但是都是以前写的。这理解和自己实测的做个记录。

1\. 首先下载 mongo sheel。因为新版没有，所以得单独下载。

[MongoDB Shell Download​www.mongodb.com/try/download/shell![](assets/1711446746-1b396acda8392402099396922410f871.jpg)](https://www.mongodb.com/try/download/shell)

![](assets/1711446746-b03d4a63f430ee6cb636a64f5301127c.jpg)

下载完之后，解压到 mongo 目录。并把解压出来的 bin 目录里面的 mongosh.exe 挪到 mongon 的 bin 目录底下

![](assets/1711446746-f18a60c809fbdcbda4e5a992ac5f5dcd.jpg)

到这里就可以使用 mongo 的命令了

2\. 设置用户及密码

```text
use admin  
db.createUser({
  user: 'admin',  // 用户名
  pwd: '123456',  // 密码
  roles:[{
    role: 'root',  // 角色 不用修改
    db: 'admin'  // 数据库 不用修改
  }]
})
```

设置完成，可以输入 `show users` 查看是否设置成功。

3\. 开启 mongo 的验证

找到 MongoDB 安装目录，打开 `mongod.cfg` 文件

![](assets/1711446746-17184170468cccf8f32d569cbad35a98.jpg)

找到以下这句：

```text
#security:
```

修改为：

```text
security:
  authorization: enabled
```

![](assets/1711446746-ade19e7f8f0849b0cf3251772dd30081.jpg)

3\. 重启 mongo

打开任务管理器

找到 MongoDB 服务，右键重新启动。

![](assets/1711446746-8828c9f259de7762145a7e9c7e9a7dde.jpg)

4\. 连接数据库

这里我用的是 navicat 15，首先创建数据库

![](assets/1711446746-81238cbcc63e28b894d66e6232a615c3.jpg)

![](assets/1711446746-ad00b885f16f8025efb524faaf5ce953.jpg)

![](assets/1711446746-efe647208c9c099ac82552f97ae70060.jpg)

```text
输入：mongodb://admin:123456@localhost:27017/?authSource=admin
```

到这就正式连接上了
