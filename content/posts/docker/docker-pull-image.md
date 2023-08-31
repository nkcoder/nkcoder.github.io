---
title: Docker镜像：拉取镜像
date: 2020-06-16T06:25:07+08:00
categories:
- docker
draft: false
---

Docker的镜像存储在镜像注册中心（image registry）中，默认是[Docker Hub](https://hub.docker.com/)，在拉取镜像的时候也可以指定其它注册中心。

我们使用的大多数镜像都是在Docker Hub中已有镜像的基础上构建的。

拉取镜像，使用`docker image pull`或`docker pull`命令。

```bash
➜  ~ docker image pull --help

Usage:	docker image pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
  -q, --quiet                   Suppress verbose output
```

## 从Docker Hub拉取镜像

镜像的命名是：`repo:tag`，如果没有指定tag，则使用默认的`latest`标签。

```bash
➜  ~ docker image pull alpine
Using default tag: latest
latest: Pulling from library/alpine
cbdbe7a5bc2a: Pull complete
Digest: sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
```

> 注意：`latest`标签并不保证一定是最新的镜像，所以谨慎使用。

## 从其它注册中心拉取镜像

镜像默认都是从[Docker Hub](https://hub.docker.com/)上拉取，你也可以使用其它仓库，只需要在镜像名称前加上仓库地址，仓库地址与URL类似，但是不需要协议，默认使用**https**。

```bash
➜  ~ docker image pull myregistry.local:5000/alpine
```

上面的命令从`myregistry.local:5000`拉取`alpine:latest`镜像，镜像仓库的访问权限通过[docker login](https://docs.docker.com/engine/reference/commandline/login/)配置。

## 拉取一个仓库的多个镜像

`docker image pull`默认拉取一个镜像，使用`-a`或`--all-tags`可以拉取所有镜像：

```bash
➜  ~ docker image pull -a alpine
2.6: Pulling from library/alpine
Image docker.io/library/alpine:2.6 uses outdated schema1 manifest format. Please upgrade to a schema2 image for better future compatibility. More information at https://docs.docker.com/registry/spec/deprecated-schema-v1/
2a3ebcb7fbcc: Pull complete
Digest: sha256:e9cec9aec697d8b9d450edd32860ecd363f2f3174c8338beb5f809422d182c63
2.7: Pulling from library/alpine
...
```

## 使用摘要拉取镜像

通过`name:tag`标识一个镜像，方便且容易记忆。但是tag是可以被覆盖的，`docker image pull alpine:3.11`拉取的总是最新版本的`alpine:3.11`。

有些情况下，你希望使用一个固定版本的镜像，可以通过摘要（digest）拉取镜像。摘要对于镜像是唯一的。

想知道镜像的digest，需要先拉取镜像：

```bash
➜  ~ docker image pull alpine
Using default tag: latest
latest: Pulling from library/alpine
cbdbe7a5bc2a: Pull complete
`Digest: sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54`
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
```

通过digest拉取镜像：

```bash
➜  ~ docker image pull alpine@sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54
sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54: Pulling from library/alpine
Digest: sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54
Status: Image is up to date for alpine@sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54
docker.io/library/alpine@sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54
```

## 配置代理

如果注册中心通过代理才能访问，则需要配置Docker Daemon的代理。设置`HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`环境变量即可。

```bash
HTTPS_PROXY=https://my-proxy.com
```

### references 

- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [Docker Deep Dive](https://www.amazon.com/Docker-Deep-Dive-Nigel-Poulton-ebook/dp/B01LXWQUFF)