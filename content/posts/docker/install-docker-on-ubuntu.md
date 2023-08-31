---
title: Docker安装：在Ubunut上安装
date: 2020-06-07T15:06:07+08:00
categories:
- docker
draft: false
---

在Linux上安装Docker可以有以下几种方式：

- 设置docker仓库，然后从仓库安装。这种方式的好处是易于安装和升级，这也是官方推荐的安装方式
- 下载对应平台的安装包，手动安装和升级。这种方式比较适合无网络访问的环境。
- 使用自动化脚本安装。这种方式主要用于开发和测试。

## 从仓库安装

更新`apt`，并安装可以使用HTTPS仓库的安装包：

```bash
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

安装Docker官方的GPG密钥：

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

设置stable仓库：

```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

最后，安装Docker Engine：

```bash
 $ sudo apt-get update
 $ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## 使用安装包安装

从[ubuntu-仓库](https://download.docker.com/linux/ubuntu/dists/)中选择对应的Ubuntu版本，然后进入`/pool/stable`目录下载对应的DEB安装包，安装：

```bash
$ sudo dpkg -i /path/to/package.deb
```

## 使用脚本安装

使用脚本方式安装，有以下几点需要注意：

- 脚本的运行需要root或sudo权限，所以运行前务必检查脚本的内容
- 脚本安装不支持任何安装参数的配置
- 不需要你的确认，会安装所有的依赖和推荐的包，因此可能会安装大量不需要的安装包
- 安装的是最新版的Edge版，而不是Stable版

因此，脚本安装方式主要用于开发和测试，不应该用于生产环境。

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

安装完后，都以使用命令确认下安装是否成功：

```bash
➜  ~ docker -v
Docker version 19.03.8, build afacb8b
➜  ~ docker-compose -v
docker-compose version 1.25.4, build 8d51620a
```

## 使用非root用户管理Docker

docker守护进程（daemon）是绑定到Unix socket上的，而不是TCP端口。Unix socket默认是属于`root`的，所以在执行Docker命令时需要使用`sudo`。

如果不希望执行每个Docker命令都输入`sudo`，可以创建一个Unix用户组`docker`，然后将普通用户添加到该用户组中。当Docker daemon启动时，它会创建一个可以被`docker`用户组中的用户访问的Unix socket。

创建`docker`用户组并将添加用户（以上安装方式默认都创建了`docker`用户组，只是其中没有任何用户）：

```bash
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

注销然后重新登录可以让用户组的修改生效。在Linux上也可以执行以下命令直接使用户组的修改成效：

```bash
$ newgrp docker 
```

## 卸载

首先卸载Docker engine和cli：

```bash
$ sudo apt-get purge docker-ce docker-ce-cli containerd.io
```

然后删除所有的image、container和volume：

```bash
$ sudo rm -rf /var/lib/docker
```

### references:

- [docker-install-on-ubuntu](https://docs.docker.com/engine/install/ubuntu/)

