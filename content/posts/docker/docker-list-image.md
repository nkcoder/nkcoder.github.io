---
title: Docker镜像：查看本地镜像
date: 2020-06-16T06:33:33+08:00
categories:
- docker
draft: false
---

查看本地镜像，使用命令`docker image ls`：

```bash
➜  ~ docker image ls --help

Usage:	docker image ls [OPTIONS] [REPOSITORY[:TAG]]

List images

Aliases:
  ls, images, list

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
```

该命令还有两个别名，即：`docker images`和`docker image list`。

## 查看最近创建的镜像

```bash
➜  ~ docker image ls
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
nkcoder/freeimmi_mini                4.12                5ec653d76efc        3 weeks ago         394MB
mysql                                8.0.19              9228ee8bac7a        5 weeks ago         547MB
docker/desktop-storage-provisioner   v1.0                605a0f683b7b        2 months ago        33.1MB
k8s.gcr.io/kube-controller-manager   v1.15.5             1399a72fa1a9        6 months ago        159MB
```

## 通过仓库名称或tag查看镜像

`docker images`后面可以带可选的仓库名称和版本号：`docker images [repository[:tag]]`

```bash
➜  ~ docker images mysql
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               8.0.19              9228ee8bac7a        5 weeks ago         547MB

➜  ~ docker images mysql:8.0.19
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               8.0.19              9228ee8bac7a        5 weeks ago         547MB

➜  ~ docker images mysql:8
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

- 如果只有`repository`，而没有`tag`，则会列出该仓库的所有镜像
- `repository`和`tag`是精确匹配的

## 过滤

`-f, --filter`参数通过指定的条件过滤，常用的过滤器有：

- dangling=true|false：true表示仅返回悬垂镜像，false表示返回非悬垂镜像。那些没有tag的镜像被称为悬垂镜像，在列表中展示为<none>:<none>

  ```bash
  ➜  ~ docker image ls --filter “dangling=true”
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  alpine              <none>              f70734b6a266        13 days ago         5.61MB
  ```

- before=镜像名称｜镜像ID：返回在之前被创建的全部镜像

  ```bash
  ➜  ~ docker image ls -f “before=fab2dded59dd”
  REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
  docker/kube-compose-controller   v0.4.23             a8c3d87a58e7        11 months ago       35.3MB
  docker/kube-compose-api-server   v0.4.23             f3591b2cb223        11 months ago       49.9MB
  ```

- since=镜像名称｜镜像ID：返回在之后被创建的全部镜像

  ```bash
  ➜  ~ docker image ls -f “since=fab2dded59dd”
  REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
  alpine                               <none>              f70734b6a266        13 days ago         5.61MB
  nkcoder/freeimmi_mini                4.12                5ec653d76efc        3 weeks ago         394MB
  ```

- reference=”repo:tag"：对repo和tag进行模糊匹配，仅返回匹配的镜像

  ```bash
  ➜  ~ docker image ls -f reference="docker/*:*"
  REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
  docker/desktop-storage-provisioner   v1.0                605a0f683b7b        2 months ago        33.1MB
  ```

## 格式化输出

`--format`参数根据go模版输出格式化后的内容，主要的go模版有：

| Placeholder     | Description                              |
| :-------------- | :--------------------------------------- |
| `.ID`           | 镜像ID                                   |
| `.Repository`   | Image repository                         |
| `.Tag`          | 镜像tag                                  |
| `.Digest`       | Image digest                             |
| `.CreatedSince` | Elapsed time since the image was created |
| `.CreatedAt`    | Time when the image was created          |
| `.Size`         | 镜像大小                                 |

```bash
➜  ~ docker images --format "{{.Repository}}:{{.Tag}} \t {{.CreatedAt}}"
nkcoder/freeimmi_mini:4.12 	 2020-04-12 21:56:17 +0800 CST
mysql:8.0.19 	 2020-03-31 11:14:46 +0800 CST
```

格式化时，默认是根据模版来显示内容的，没有表头行，在模版前加上`table`可以显示表头：

```bash
➜  ~ docker images --format "table {{.Repository}}:{{.Tag}} \t {{.CreatedAt}}"
REPOSITORY:TAG                                 CREATED AT
nkcoder/freeimmi_mini:4.12                     2020-04-12 21:56:17 +0800 CST
mysql:8.0.19                                   2020-03-31 11:14:46 +0800 CST
```

## 仅返回镜像ID

`-q`参数使命令仅输出镜像ID，在命令组合时很有用，比如以下命令删除所有悬垂镜像：

```bash
➜  ~ docker rmi $(docker images -f "dangling=true" -q)

8abc22fbb042
48e5f45168b9
```

### reference: 
- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [Docker Deep Dive](https://www.amazon.com/Docker-Deep-Dive-Nigel-Poulton-ebook/dp/B01LXWQUFF)
