---
title: shell 脚本 main 函数中$#获取不到脚本传入参数个数浅析
url: https://mp.weixin.qq.com/s?__biz=Mzg2OTAwMTE3NQ==&mid=2247488822&idx=1&sn=12ad368fb755cca946d82642f7747d4f&chksm=cea2e78ef9d56e98adde3cbc2f846b1b966e4afc0e6e95afa44ff664dfc67f9c4336c73102fe&mpshare=1&scene=1&srcid=0314Zg7ZBZnBcmmqk3nBczaI&sharer_shareinfo=2619112c90c40597c961f76dcacd21fb&sharer_shareinfo_first=2619112c90c40597c961f76dcacd21fb#rd
clipped_at: 2024-03-14 12:06:56
category: default
tags: 
 - mp.weixin.qq.com
---


# shell 脚本 main 函数中$#获取不到脚本传入参数个数浅析

Linux 的 shell 脚本，有时候我们在运行 shell 脚本时会给脚本传入参数，出于逻辑上的严谨，在脚本中可能会做一些逻辑判断或处理，例如判断脚本传入参数的个数。一般我们会用 $# 获取传入参数的个数，假如，我们在 shell 脚本的 main 函数中去判断脚本传入参数的个数，类似如下所示：

```bash
.........
function main()
{
    if [ $# != 1 ]; then
      echo "This script must be run with one parameter"
      echo "Usage: mysql_slowlog_monitor.sh 6h"
      exit 1
    fi

    check_enviroment;
    send_slow_rpt;
    return 0;
}

main;
```

如果你去调试这个 shell 脚本的话，就会发现 **main 函数中 $# 的值永远是 0**，如果将脚本调整一下，将判断传入参数个数的脚本放到 main 函数外 (不能放在其它函数中)，如下所示，这样就 Ok 了

```bash
.............

if [ $# != 1 ]; then
  echo "This script must be run with one parameter"
  echo "Usage: mysql_slowlog_monitor.sh 6h"
  exit 1
fi

.............
function main()
{
    check_enviroment;
    send_slow_rpt;
    return 0;
}

main;
```

那么为什么会出现这种情况呢？在解答这个问题前，我们先来了解一下 $#的用途，$#表示脚本传入参数的个数，也表示一个函数 (function) 调用时，传入函数的参数 (arguments) 个数，而且它也是有作用域范围，如果在函数 (function) 内部的话，它表示的函数调用时，传入参数的个数。

那么再来解答这个问题，上面 shell 脚本中，main 函数调用时写法为 main; 意味着函数调用时没有传入任何参数，所以 $# 在 main 中值为 0，而在脚本 mysql\_slowlog\_monitor.sh 中获取传入的参数个数，应该在脚本中，而且在脚本中的函数外面获取它的值。

那么怎么解决这个问题呢？

### 解决方案 1：

将判断脚本调用时传入的参数的脚本放到函数外面，就像上面示例脚本那样处理。

### 解决方案 2：

借助全局变量，先在函数外获取脚本传入参数的个数，将其赋值为全局变量，然后在 mian 函数中，进行逻辑判断和处理。

```bash
.............
ARGS=$#
.............
function main()
{
    if [ $ARGS != 1 ]; then
      echo "This script must be run with one parameter"
      echo "Usage: mysql_slowlog_monitor.sh 6h"
      exit 1
    fi
    check_enviroment;
    send_slow_rpt;
    return 0;
}

main;
```
