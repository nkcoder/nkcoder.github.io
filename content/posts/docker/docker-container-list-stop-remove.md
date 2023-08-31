---
title: Docker容器：查看、停止、删除容器
date: 2020-07-09T06:50:13+08:00
categories:
- docker
draft: false
---

## 查看容器

列出容器的命令为：`docker container ls`，等价的别名为：

```bash
docker container ps
docker container list
docker ps
```

常用的参数说明如下：

- `-a, --all`：列出所有的容器，包括停止运行的容器
- `-s, --size`：显示容器的大小
- `-q, --quiet`：仅显示容器ID
- `-f, --filter`：过滤器，支持`key=value`的格式进行过滤，多个过滤器使用`-f "key=value" -f "key=value"`格式

`-a`列出所有容器：

```bash
➜  ~ docker ps -a
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS                    PORTS                               NAMES
1c4bc2440cff        docker.elastic.co/elasticsearch/elasticsearch:7.7.0   "/tini -- /usr/local…"   7 days ago          Exited (130) 5 days ago                                       sweet_lovelace
da242f09324e        mysql:8.0.19                                          "docker-entrypoint.s…"   7 weeks ago         Up 15 minutes             0.0.0.0:3306->3306/tcp, 33060/tcp   freeimmi_mini_db_1
```

`-s`列出容器的大小：

```bash
➜  ~ docker ps -s
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES                SIZE
da242f09324e        mysql:8.0.19        "docker-entrypoint.s…"   7 weeks ago         Up 16 minutes       0.0.0.0:3306->3306/tcp, 33060/tcp   freeimmi_mini_db_1   105B (virtual 547MB)
```

> `SIZE`列显示的是容器的**可写层**占用的空间，括号中的virtual表示容器镜像的**只读层**和容器的**可写层**总共占用的空间。

`-q`仅显示容器ID：

```bash
➜  ~ docker ps -q
da242f09324e
➜  ~ docker stop $(docker ps -a -q)
1c4bc2440cff
da242f09324e
```

`-f`可以通过容器名称（name）、退出状态（exited）、容器状态（status）、创建时间（before|since|after）等进行过滤：

```bash
➜  ~ docker ps -f 'name=freeimmi_mini_db_1'
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
da242f09324e        mysql:8.0.19        "docker-entrypoint.s…"   7 weeks ago         Up 12 seconds       0.0.0.0:3306->3306/tcp, 33060/tcp   freeimmi_mini_db_1

➜  ~ docker stop freeimmi_mini_db_1
freeimmi_mini_db_1
➜  ~ docker ps -a -f 'exited=0'
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
da242f09324e        mysql:8.0.19        "docker-entrypoint.s…"   7 weeks ago         Exited (0) 48 seconds ago                       freeimmi_mini_db_1

➜  ~ docker start freeimmi_mini_db_1
freeimmi_mini_db_1
➜  ~ docker kill freeimmi_mini_db_1
freeimmi_mini_db_1
➜  ~ docker ps -a -f 'exited=137'
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS               NAMES
da242f09324e        mysql:8.0.19        "docker-entrypoint.s…"   7 weeks ago         Exited (137) 5 seconds ago                       freeimmi_mini_db_1
```

## 停止容器

停止容器使用`docker container stop`或者`docker stop`命令，该命令会向容器内的主进程发送`SIGTERM`信号，等待一段时间后，再发送`SIGKILL`命令。

可以使用`-t, --time`设置kill之前的等待时间，默认是10s：

```
➜  ~ docker container start freeimmi_mini_db_1
freeimmi_mini_db_1
➜  ~ docker stop -t 20 freeimmi_mini_db_1
freeimmi_mini_db_1
```

## 删除容器

删除容器使用`docker container rm`或者`docker rm`命令。

```bash
➜  ~ docker ps -a
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS                     PORTS               NAMES
58a0b9865684        alpine                                                "/bin/sh"                44 seconds ago      Up 43 seconds                                  my_alpine3
1c4bc2440cff        docker.elastic.co/elasticsearch/elasticsearch:7.7.0   "/tini -- /usr/local…"   7 days ago          Exited (130) 5 days ago                        sweet_lovelace
da242f09324e        mysql:8.0.19                                          "docker-entrypoint.s…"   7 weeks ago         Exited (0) 8 minutes ago                       freeimmi_mini_db_1
➜  ~ docker rm 1c4bc2440cff
1c4bc2440cff
```

`docker rm`不能直接删除运行中的容器，可是使用`-f, --force`参数，表示强制删除，即直接向容器中的主进程发送`SIGKILL`信号，一般不推荐这么做，建议先使用`docker stop`停止容器，然后再使用`docker rm`删除容器，给容器留出一些时间进行清理等工作：

```bash
➜  ~ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
58a0b9865684        alpine              "/bin/sh"           2 minutes ago       Up 2 minutes                            my_alpine3
➜  ~ docker rm -f 58a0b9865684
58a0b9865684
```

删除所有停止的容器：

```bash
➜  ~ docker rm $(docker ps -a -q)
da242f09324e
```

`-v`参数表示删除与容器关联的所有匿名的卷：

```bash
➜  ~ docker create -v awesome:/foo -v /bar --name hell-redis redis
7da82099f278dd15c6ee16ea5a1e347feb1f3f45b9c8890c830cd7f095e1c63f
➜  ~ docker ps  -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
7da82099f278        redis               "docker-entrypoint.s…"   7 seconds ago       Created                                 hell-redis
➜  ~ docker rm -v hell-redis
hell-redis
➜  ~ docker volume ls | grep awesome
local               awesome
```

删除容器，还可以使用`docker container prune`，表示删除所有停止的容器，`-f, --force`参数表示不需要交互式确认：

```bash
➜  ~ docker container prune -f
Deleted Containers:
72a9df4be93f1fa11dc61d73ef48211b507949a1aaa0231a5ef0383cd61c4d45
73d22e04a8ae342174e0069029066d4f03f3a94b373b6d0c253654ee926592fb
44c829e75011bb913c7af9bf095ccec865084db1905a80dfacbb3e4b47209b17

Total reclaimed space: 0B
```

`--filter`支持过滤，目前支持的过滤器有：`unitl(删除指定时间之前的容器)`和`label(删除指定标签的容器)`:

```
➜  ~ docker container prune -f --filter "until=1m"
Deleted Containers:
633cb0d9b525e7e595442e8545ef6458c9dbf342fd12c00616acd63ee73bcafb

Total reclaimed space: 0B
```

### reference docs:

- [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)
- [docker stop](https://docs.docker.com/engine/reference/commandline/stop/)
- [docker rm](https://docs.docker.com/engine/reference/commandline/rm/)
- [docker container prune](https://docs.docker.com/engine/reference/commandline/container_prune/)