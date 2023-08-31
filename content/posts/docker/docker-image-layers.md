---
title: Docker镜像：镜像分层
date: 2020-06-16T06:41:46+08:00
categories:
- docker
draft: false
---

Docker镜像是由一系列的镜像层构成的。

## 查看镜像层

有多种方式可以查看镜像的分层信息。第一种方式，在拉取镜像的时候：

```bash
➜  ~ docker pull redis
Using default tag: latest
latest: Pulling from library/redis
54fec2fa59d0: Pull complete
9c94e11103d9: Pull complete
04ab1bfc453f: Pull complete
a22fde870392: Pull complete
def16cac9f02: Pull complete
1604f5999542: Pull complete
Digest: sha256:f7ee67d8d9050357a6ea362e2a7e8b65a6823d9b612bc430d057416788ef6df9
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

以`Pull complete`结尾的每一行表示一个镜像分层，前面的`def16cac9f02`是镜像层的ID。

如果镜像已经拉取了，可以通过`docker inspect`查看镜像分层：

```bash
➜  ~ docker inspect redis
[
    {
            "Id": "sha256:f9b9909726890b00d2098081642edf32e5211b7ab53563929a47f250bcdc1d7c",
        "RepoTags": [
            "redis:latest"
        ],
        ...
                "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:c2adabaecedbda0af72b153c6499a0555f3a769d52370469d8f6bd6328af9b13",
                "sha256:744315296a49be711c312dfa1b3a80516116f78c437367ff0bc678da1123e990",
                "sha256:379ef5d5cb402a5538413d7285b21aa58a560882d15f1f553f7868dc4b66afa8",
                "sha256:d00fd460effb7b066760f97447c071492d471c5176d05b8af1751806a1f905f8",
                "sha256:4d0c196331523cfed7bf5bafd616ecb3855256838d850b6f3d5fba911f6c4123",
                "sha256:98b4a6242af2536383425ba2d6de033a510e049d9ca07ff501b95052da76e894"
            ]
        },
    }
]
```

其中`Layers`节点表示该镜像的镜像层，只不过这里显示的是镜像摘要。

## 镜像与镜像层

所有的Docker镜像都起始于一个基础镜像层，但进行修改或增加新的内容时，就会在当前镜像层之上创建新的镜像层。

具体到Dockerfile，每个Docker镜像层都对应于Dockerfile中的一条指令。

镜像层以堆栈结构排列，每个镜像层的内容，是与上一个镜像层相比不同部分的集合。

所有的镜像层都是只读的。当创建一个容器时，会在对顶层创建一个可写的镜像层，也叫`容器层`，任何的修改操作（如增加、删除文件等）都是在顶层的可写镜像层中进行的。如果容器被删除，只有最顶层的可写镜像层被删除，其它只读镜像层都将被保留。

![conainter-layers](./img/container-layers.jpg)

> 图片来自Docker官网: https://docs.docker.com/storage/storagedriver/

不同镜像层之间的交互是由存储引擎（storage driver）实现的，有多个不同的存储引擎可以选择，他们有各自的适用场景。

### 共享镜像层

多个镜像之间可以并且确实在共享镜像层，这样可以有效节省时间并提升性能。比如拉取redis的两个不同tag的镜像：

```bash
➜  ~ docker pull redis:6-alpine
6-alpine: Pulling from library/redis
cbdbe7a5bc2a: Pull complete
dc0373118a0d: Pull complete
cfd369fe6256: Pull complete
e5396613619b: Pull complete
6809b5ad2cd4: Pull complete
386ecfe54d06: Pull complete
Digest: sha256:2586f31f74ac1d7dc6f6c7eabca42f09bba5ec9911fc519d55fbd7508a9c4f01
Status: Downloaded newer image for redis:6-alpine
docker.io/library/redis:6-alpine
➜  ~ docker pull redis:5-alpine
5-alpine: Pulling from library/redis
cbdbe7a5bc2a: Already exists
dc0373118a0d: Already exists
cfd369fe6256: Already exists
3e45770272d9: Pull complete
558de8ea3153: Pull complete
a2c652551612: Pull complete
Digest: sha256:83a3af36d5e57f2901b4783c313720e5fa3ecf0424ba86ad9775e06a9a5e35d0
Status: Downloaded newer image for redis:5-alpine
docker.io/library/redis:5-alpine
```

`Already exists`的行表示镜像层本地已经存在，无需拉取，直接复用。


### reference: 

- [storage-driver](https://docs.docker.com/storage/storagedriver/)
- [Docker Deep Dive](https://www.amazon.com/Docker-Deep-Dive-Nigel-Poulton-ebook/dp/B01LXWQUFF)