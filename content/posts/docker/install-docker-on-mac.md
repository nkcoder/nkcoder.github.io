---
title: Docker安装：在Mac上安装
date: 2020-06-07T14:11:43+08:00
categories:
- docker
draft: false
---

## 安装Docker Desktop 

Docker提供了Mac版的安装包，在[Docker 网站](https://www.docker.com/products/docker-desktop)下载`docker.dmg`安装即可。

Mac版Docker的底层是基于Linux VM运行的，即Docker daemon是运行在一个轻量级的Linux VM之上的。

Mac版Docker主要用于开发和测试。

安装之后，可以执行相关命令检查下：

```bash
➜  ~ docker --version
Docker version 19.03.8, build afacb8b
➜  ~ docker-compose --version
docker-compose version 1.25.4, build 8d51620a
```


## 故障排除

如果docker出现故障，除了可以卸载重新安装，还可以试试Mac版Docker自带的`故障排除`功能，点击Docker小鲸鱼后，选择`Troubleshoot`：

![docker-troubleshoot](/images/docker/docker-troubleshoot.png)

可以进行诊断、重置磁盘等操作。


## Dashboard

Mac版Docker还提供了一个可视化的界面，可以看到当前运行的容器的状态、例子以及配置等信息，如：

![docker-dashboard](/images/docker/docker-dashboard.png)

![docker-dashboard-log](/images/docker/docker-dashboard-log.png)


## Kubernetes

Mac版Docker包含一个单机的Kubernetes服务器，可以将Docker容器或服务部署到Kubernetes中去测试。

![docker-kubernetes](/images/docker/docker-kubernetes.png)

- Enable Kubernetes：表示启用自带的Kubernetes
- Deploy Docker Stacks to Kubernetes by default：将Kubernetes设置为docker stack默认的服务编排器

首先使Kubernetes客户端`kubectl`连接本地的Kubernetes服务器：

```bash
➜  ~ kubectl config get-contexts
CURRENT   NAME                 CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop       docker-desktop   docker-desktop
          docker-for-desktop   docker-desktop   docker-desktop
➜  ~ kubectl config use-context docker-desktop
Switched to context "docker-desktop".
```

### references

- [docker-for-mac](https://docs.docker.com/docker-for-mac/)


