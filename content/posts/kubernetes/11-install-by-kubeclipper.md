---
title: "Kubernetes教程(十一)---使用 KubeClipper 通过一条命令快速创建 k8s 集群"
description: "使用 KubeClipper 通过一条命令快速创建 k8s 集群"
date: 2022-08-20
draft: false
categories: ["Kubernetes"]
tags: ["Kubernetes"]
---

本文主要记录了如何使用开源项目 KubeClipper 通过一条命令快速创建 k8s 集群。

<!--more-->

之前写了一篇使用 kubeadm 创建集群的文章： [Kubernetes教程(一)---使用 kubeadm 创建 k8s 集群(containerd)](https://www.lixueduan.com/posts/kubernetes/01-install/)，整个流程下来其实还是比较麻烦的，体验并不是太好，今天给大家推荐一个优秀的开源项目：[KubeClipper ](https://github.com/kubeclipper-labs/kubeclipper)。



## 1. 什么是 KubeClipper

KubeClipper 旨在提供易使用、易运维、极轻量、生产级的 Kubernetes 多集群全生命周期管理服务，让运维工程师从繁复的配置和晦涩的命令行中解放出来，实现一站式管理跨区域、跨基础设施的多 K8S 集群。

KubeClipper 的 3个优势：

* 1）易使用：图形化页面
  * 提供 Web 控制台，通过点点鼠标即可创建一个 k8s 集群
* 2）极轻量：架构简单，少依赖
  * 不依赖 ansible，使用方便
* 3）生产级：易用性和专业性兼顾
  * 提供在线、离线、代理方式部署以及多版本 K8S、CRI、CNI 选择
  * 提供了离线部署方式对**国内用户极为友好**，毕竟网络才是国内用户装 k8s 的一大难题。

关于该项目的更多信息见：[Github Repo](https://github.com/kubeclipper-labs/kubeclipper) 以及官方的这篇介绍文章：[KubeClipper——轻量便捷的 Kubernetes 多集群全生命周期管理工具](https://mp.weixin.qq.com/s/RVUB5Pw6-A5zZAQktl8AcQ)

一句话概括：**KubeClipper 是一个轻量便捷的 Kubernetes 多集群全生命周期管理工具**。



## 2. 快速开始

### 准备工作

KubeClipper 本身并不会占用太多资源，但是为了后续更好的运行 Kubernetes 建议硬件配置不低于最低要求。

您仅需参考以下对机器硬件和操作系统的要求准备一台主机。

#### 硬件推荐配置

- 确保您的机器满足最低硬件要求：CPU >= 2 核，内存 >= 2GB。
- 操作系统：CentOS 7.x / Ubuntu 18.04 / Ubuntu 20.04。

#### 节点要求

- 节点必须能够通过 `SSH` 连接。
- 节点上可以使用 `sudo` / `curl` / `wget` / `tar` / `gzip`命令。

> 建议您的操作系统处于干净状态（不安装任何其他软件），否则可能会发生冲突。



实际上这些要求基本上不用特别处理，随便找一台机器都满足条件，本文就使用一台 2C4G 的 CentOS 7.9 机器来部署。



### 安装 KubeClipper

#### 下载 kcctl

KubeClipper 提供了命令行工具🔧 kcctl，我们先安装 kcctl，安装命令如下：

```bash
curl -sfL https://oss.kubeclipper.io/kcctl.sh | KC_REGION=cn bash -
```

通过以下命令检测是否安装成功:

```bash
kcctl version
```



#### 开始安装

然后使用 kcctl 安装 kubeclipper，一条命令即可,模板如下：

```bash
kcctl deploy  [--user root] (--passwd SSH_PASSWD | --pk-file SSH_PRIVATE_KEY)
```

> 只需要提供 ssh user 以及 ssh passwd 或者 ssh 私钥即可在本机部署 KubeClipper。

若使用 ssh passwd 方式则命令如下所示:

```bash
kcctl deploy --user root --passwd $SSH_PASSWD
```

私钥方式如下：

```bash
kcctl deploy --user root --pk-file $SSH_PRIVATE_KEY
```

本文使用密码进行部署，具体命令如下：

```bash
kcctl deploy --user root --passwd root
```



执行该命令后，Kcctl 将检查安装环境，若满足条件将会进入安装流程。在打印出如下的 KubeClipper banner 后即表示安装完成。

```
 _   __      _          _____ _ _
| | / /     | |        /  __ \ (_)
| |/ / _   _| |__   ___| /  \/ |_ _ __  _ __   ___ _ __
|    \| | | | '_ \ / _ \ |   | | | '_ \| '_ \ / _ \ '__|
| |\  \ |_| | |_) |  __/ \__/\ | | |_) | |_) |  __/ |
\_| \_/\__,_|_.__/ \___|\____/_|_| .__/| .__/ \___|_|
                                 | |   | |
                                 |_|   |_|
```



> 安装过程中需要去阿里云下载离线安装包，速度有点慢(后续会优化)，需要大概 3分钟时间。



### 登录控制台

安装完成后，打开浏览器，访问 `http://$IP` 即可进入 KubeClipper 控制台。

![](../../../img/kubernetes/kubeclipper/kc-console-login.png)

您可以使用默认帐号密码 `admin / Thinkbig1` 进行登录。

> 您可能需要配置端口转发规则并在安全组中开放端口，以便外部用户访问控制台。

> 文中所有操作均通过命令行工具完成，至于控制台相关功能则由大家自行探索了~



###  安装 K8S 集群

部署成功后可以使用 **kcctl 工具**或者通过**控制台**创建 k8s 集群， 这里咱们使用 kcctl 工具进行创建。

> 命令行，程序猿最后的倔强。

首先使用默认账号密码进行登录获取 token，便于后续 kcctl 和 kc-server 进行交互。

```bash
kcctl login -H http://localhost -u admin -p Thinkbig1
```

查看当前 agent 节点

```bash
[root@iZbp12pyvyollw222dekjmZ ~]# kcctl get node
+--------------------------------------+-------------------------+---------+----------------+-------------+-----+--------+
|                  ID                  |        HOSTNAME         | REGION  |       IP       |   OS/ARCH   | CPU |  MEM   |
+--------------------------------------+-------------------------+---------+----------------+-------------+-----+--------+
| ca22ddb8-5a61-4ca3-87bd-91fa71080cb7 | kc-1 | default | 192.168.10.217 | linux/amd64 |   2 | 3646Mi |
+--------------------------------------+-------------------------+---------+----------------+-------------+-----+--------+

```

然后使用以下命令创建 k8s 集群:

```bash
# 其中 IP 为上一步中的 kc-1 节点 IP
kcctl create cluster --name demo --master 192.168.10.217 --untaint-master
```

大概 3 分钟左右即可完成集群创建,也可以使用以下命令查看集群状态

```bash
kcctl get cluster -o yaml|grep status -A5
```

> 也可以进入控制台查看**实时日志**。

进入 Running 状态即表示集群安装完成,您可以使用 `kubectl get cs` 命令来查看集群健康状况。

```bash
$ kubectl get cs
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok                              
controller-manager   Healthy   ok                              
etcd-0               Healthy   {"health":"true","reason":""} 
```



至此，我们的单节点版本就算是结束了，相比之下确实比使用原生 kubeadm 体验好多了。



## 3. 多节点版本

为了保证高可用，k8s 集群至少需要 3个 master 节点，同时 kubeclipper 使用 etcd 作为后端存储，为了保证高可用，也建议至少使用 3 节点及以上来部署。

>  同时生产环境一般建议 kubeclipper server 节点单独存在，不建议将 server 节点同时作为 agent 节点使用。

节点规划如下：

* 3 台 作为 kubeclipper server 部署使用
* 5 台 作为 kubeclipper agent 节点，部署 k8s 使用。

> 资源有限，依旧是 2C4G 的 CentOS 7.9 机器



### 安装 KubeClipper

同样的，还是需要先安装 kcctl 工具，安装方式和之前是一样的，一条命令搞定：

> kcctl 工具只需要在任意一台节点上安装即可，只要能使用 ssh 访问其他节点即可。
>
> 为了和本教程保持一致，建议在其中的一台 server 节点上安装。

```bash
curl -sfL https://oss.kubeclipper.io/kcctl.sh | KC_REGION=cn bash -
```



接着使用 kcctl 部署 KubeClipper，其模板如下所示：

```bash
kcctl deploy  [--user root] (--passwd SSH_PASSWD | --pk-file SSH_PRIVATE_KEY) (--server SERVER_NODES) (--agent AGENT_NODES)
```

> 需要手动指定 server 节点以及 agent 节点。

若使用 密码 方式则命令如下所示:

```bash
kcctl deploy --user root --passwd $SSH_PASSWD --server SERVER_NODES --agent AGENT_NODES
```

私钥 方式如下：

```bash
kcctl deploy --user root --pk-file $SSH_PRIVATE_KEY --server SERVER_NODES --agent AGENT_NODES
```

本教程使用 密码 方式进行部署，具体命令如下：

```bash
kcctl deploy --server 192.168.10.217,192.168.10.196,192.168.10.136 --agent 192.168.10.58,192.168.10.225,192.168.10.19,192.168.10.27,192.168.10.4 --passwd root --pkg https://oss.kubeclipper.io/release/v1.1.0/kc-amd64.tar.gz
```

> 3 个 server 节点，5 个 agent 节点，多个 IP 之间使用逗号`,`分隔

同样的，在打印出如下的 KubeClipper banner 后即表示安装完成。

```text
 _   __      _          _____ _ _
| | / /     | |        /  __ \ (_)
| |/ / _   _| |__   ___| /  \/ |_ _ __  _ __   ___ _ __
|    \| | | | '_ \ / _ \ |   | | | '_ \| '_ \ / _ \ '__|
| |\  \ |_| | |_) |  __/ \__/\ | | |_) | |_) |  __/ |
\_| \_/\__,_|_.__/ \___|\____/_|_| .__/| .__/ \___|_|
                                 | |   | |
                                 |_|   |_|
```



### 创建高可用 K8S 集群

这里我们同样使用 kcctl 工具，通过命令行方式进行创建。老规矩，使用默认账号密码进行登录：

```bash
kcctl login -H http://localhost -u admin -p Thinkbig1
```

然后查看一下当前的 agent 节点信息：

```bash
[root@server-1 ~]# kcctl get node
+--------------------------------------+----------+---------+----------------+-------------+-----+--------+
|                  ID                  | HOSTNAME | REGION  |       IP       |   OS/ARCH   | CPU |  MEM   |
+--------------------------------------+----------+---------+----------------+-------------+-----+--------+
| ba6a5399-387f-4abd-a1b8-585e0094bf25 | agent-5  | default | 192.168.10.4   | linux/amd64 |   2 | 3938Mi |
| 2c3bb128-0081-4efa-aced-9d5560a5aa63 | agent-4  | default | 192.168.10.27  | linux/amd64 |   2 | 3938Mi |
| e5a79dbe-d872-4118-90f5-5d86bbf34392 | agent-3  | default | 192.168.10.19  | linux/amd64 |   2 | 3938Mi |
| 74aed462-221c-4053-a897-e2166f805285 | agent-2  | default | 192.168.10.225 | linux/amd64 |   2 | 3938Mi |
| c78a59a3-3907-4d2e-99df-5fc85560bd1d | agent-1  | default | 192.168.10.58  | linux/amd64 |   2 | 3938Mi |
+--------------------------------------+----------+---------+----------------+-------------+-----+--------+
```



接下来就可以使用 `kcctl create cluster` 命令创建集群了,命令模版如下：

```bash
kcctl create cluster (--name CLUSTER_NAME) (--master NODES) [--worker NODES][ --untaint-master] 
```

> 更多信息见 `kcctl create cluster -h`

在本教程中，我们将 agent1、2、3 作为 master 节点，agent4、5作为 worker 节点，完整命令如下：

```bash
kcctl create cluster --name demo --master 192.168.10.58,192.168.10.225,192.168.10.19 --worker 192.168.10.27,192.168.10.4
```



这样一个 5 节点的集群大概需要 5 分钟分钟左右可以完成创建,也可以使用以下命令查看集群状态

```bash
kcctl get cluster -o yaml|grep status -A5
```

> 您也可以进入控制台查看实时日志。



进入 Running 状态即表示集群安装完成,您可以使用 `kubectl get cs` 命令来查看集群健康状况。

**ssh 到任意 master 节点**上查看集群状态：

```bash
[root@agent-1 ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok                              
controller-manager   Healthy   ok                              
etcd-0               Healthy   {"health":"true","reason":""} 
[root@agent-1 ~]# kubectl get node
NAME      STATUS   ROLES                  AGE     VERSION
agent-1   Ready    control-plane,master   6m41s   v1.23.6
agent-2   Ready    control-plane,master   6m5s    v1.23.6
agent-3   Ready    control-plane,master   6m14s   v1.23.6
agent-4   Ready    <none>                 5m17s   v1.23.6
agent-5   Ready    <none>                 5m18s   v1.23.6
```

可以看到，5 节点都处于 Ready 状态，说明集群安装成功。



## 4. 小结

本文演示了如何通过 KubeClipper 使用简单的命令创建 K8S 集群。

整个流程可以简单的分为 3 个步骤：

* 1）安装 kcctl

```bash
curl -sfL https://oss.kubeclipper.io/kcctl.sh | KC_REGION=cn bash -
```

* 2）使用 kcctl 部署 kubeclipper

```bash
kcctl deploy  [--user root] (--passwd SSH_PASSWD | --pk-file SSH_PRIVATE_KEY) (--server SERVER_NODES) (--agent AGENT_NODES)
```

* 3）使用 kcctl 或控制台创建 K8S 集群

```bash
kcctl create cluster (--name CLUSTER_NAME) (--master NODES) [--worker NODES][ --untaint-master] 
```

文中所有操作均通过命令行工具完成，至于控制台相关功能则由大家自行探索了~



> 如果觉得体验还不错的话，请不要吝啬你的 star ，[KubeClipper](https://github.com/kubeclipper-labs) 还处于快速成长阶段，欢迎大家积极参与~