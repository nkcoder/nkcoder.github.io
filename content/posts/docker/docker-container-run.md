---
title: Docker容器：运行容器
date: 2020-07-09T06:46:44+08:00
categories:
- docker
draft: false
---

镜像拉取后，就可以基于镜像运行容器了。

运行镜像的命令为`docker container run `，等价命令为：`docker run`，命令的格式为：

```bash
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

该命令的参数很多，详细参数及说明可以通过`help`查看，这里根据使用场景介绍几个常用且重要的参数。

### 创建交互式终端

```bash
➜  ~ docker run --name my_alpine -it alpine
/ # ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    7 root      0:00 ps -ef
```

上面的例子，运行一个容器，命名为*my_alpine*，并创建一个交互式的伪终端（tty），连接到容器。

- `--name`：设置容器的名称，容器的名称不能重复，后面可以根据容器的名称停止/删除容器。如果没有指定名称，docker会生成一个随机不重复的名称
- `--interactive , -i`：打开容器的stdin，即可以通过输入与容器交互
- `--tty , -t`：分配一个伪终端连接到容器的stdin，`-it`一般同时使用，表示创建一个交互式终端，可以直接与容器交互

这里的`-i`和`-t`参数可能还是有点晕，我举个例子就明白了：如果只使用`-i`参数，那么可以与容器交互，但不是在标准的终端里显示的，如果只使用`-t`参数，虽然是在终端里操作，但因为非交互，所以不会显示命令的执行结果：

```bash
➜  ~ docker run --name my_alpine2 -i alpine
ls
bin
dev
etc
...
ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    7 root      0:00 ps -ef
```

```bash
➜  ~ docker run --name my_alpine3 -t alpine
/ # ps -ef
ls
```

### 指定启动命令

运行容器时，可以指定一个命令，表示容器启动后要执行的命令。一般image都有默认的启动命令，如alpine是/bin/sh，redis是redis-server等，指定的命令会覆盖默认的命令：

```bash
➜  ~ docker container run --name my_alpine -it alpine sh
/ # ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
/ #

➜  ~ docker container run --name my_redis -it redis
1:C 12 May 2020 14:49:06.873 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 12 May 2020 14:49:06.873 # Redis version=6.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 12 May 2020 14:49:06.873 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 6.0.1 (00000000/0) 64 bit
```

### 后台运行

容器启动默认在前台运行，`--detach , -d`参数使容器在后台运行：

```bash
➜  ~ docker container run --name my_alpine -it -d alpine
aa9937e0e4eaab5a1ad7e7b1909a171b4e835d30e72c0ee54b77899af51e0fa8
➜  ~ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
aa9937e0e4ea        alpine              "/bin/sh"                8 seconds ago       Up 7 seconds                                            my_alpine
```

如果想进入容器看看，可以通过`docker container exec`进入容器：

```bash
➜  ~ docker container exec -it my_alpine sh
/ # ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
/ #
```

### 端口映射

如果容器运行的服务需要对外暴露，则需要将容器中的端口与Host端口做映射，将容器的端口暴露出来。使用`--publish , -p`参数：

```bash
➜  ~ docker container run --name my_redis -dit -p 12379:2379 redis
497024cab9374510c3d201e8ab8c54b3dc3319b7ccd0f4b6b3f5b0dfc35f3b04
```

Host访问12379端口，即访问容器中的2379端口。

### 挂载目录

挂载目录就是使容器中的目录和Host的目录共享，对任何一个目录的修改都会反映到另一个目录。

目录挂载可以作为容器中数据持久化的一种方式。

`--volume , -v`参数用于挂载目录

```bash
➜  docker container run --name my_redis -dit -p 12379:2379 -v /Users/ling.guo/Downloads:/data redis
95d25399ef471f30944d3ddbcc86ef9b79b6eab18347c239da097059d65f85b8
➜  ls /Users/ling.guo/Downloads
test.md
➜  docker exec -it my_redis sh
# ls
test.md
#
```

也可以配合`--read-only`参数，表示container的文件系统是只读的，只有挂载的目录是可写的：

```bash
➜  docker container run --name my_redis2 -dit -p 22379:2379 --read-only -v /Users/ling.guo/Downloads:/data -w /data redis
deae39e6c897d639a6a169221292815464ca48c3cee4ab6f34135c28f09fa989
➜  docker exec -it my_redis2 sh
# touch test2.md
# ls
test.md  test2.md
# mkdir /hi
mkdir: cannot create directory '/hi': Read-only file system
```

其中，`-w`参数表示设置工作目录。

### 设置环境变量

使用`--env , -e`参数可以设置环境变量：

```bash
➜  ~ docker container run --name my_alpine -e VAR1=hello --env VAR2=world alpine env | grep VAR
VAR1=hello
VAR2=world
```

也可以从文件中批量读取环境变量，使用`--env-file`参数：

```bash
➜  ~ docker container run --name my_alpine --env-file env.list --rm alpine env | grep VAR
VAR1=hello
VAR2=world
```

### 限制容器的CPU和内存

`--cpus`表示cpu的个数，`--memory -m`表示内存的硬限制：

```bash
➜  ~ docker container run --name my_alpine -it --rm -m 100MB --cpus 1 alpine
/ # cat /sys/fs/cgroup/memory/memory.limit_in_bytes
104857600
```

### 重启策略

有些场景下，容器停止后，我们希望它可以自动重启。`--restart`参数用于控制容器的重启，支持4种策略：

- `no`：不自动重启容器，默认值
- `on-failure[:max-retries]`：只有当容器意外退出（即退出状态为非0）时重启，可以限制最大重启次数
- `unless-stopped`：当容器是主动被停止，或者由于Docker退出或重启导致容器停止时自动重启
- `always`：总是自动重启

```bash
➜  ~ docker run --restart=always redis
```

### reference docs:

- [docker run](https://docs.docker.com/engine/reference/commandline/run/)