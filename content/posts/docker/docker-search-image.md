---
title: Docker镜像：搜索镜像
date: 2020-06-16T06:37:36+08:00
categories:
- docker
draft: false
---

搜索Docker Hub中的镜像使用`docker search`命令：

```bash
➜  ~ docker search --help

Usage:	docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
```

## 搜索包含关键字的镜像

```bash
➜  ~ docker search redis
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
redis                            Redis is an open source key-value store that…   8140                [OK]
bitnami/redis                    Bitnami Redis Docker Image                      145                                     [OK]
sameersbn/redis                                                                  80                                      [OK]
...
```

`--no-trunc`显示完整的镜像描述：

```bash
➜  ~ docker search redis --no-trunc
NAME                             DESCRIPTION                                                                            STARS               OFFICIAL            AUTOMATED
redis                            Redis is an open source key-value store that functions as a data structure server.     8140                [OK]
bitnami/redis                    Bitnami Redis Docker Image                                                             145                                     [OK]
sameersbn/redis                                                                                                         80                                      [OK]
```

## 过滤

`-f`或`--filter`可以用于过滤搜索结果，常用的过滤器有：

- stars=数字：镜像至少拥有的star数
- is-automated=true|false：镜像是否自动构建
- is-official=true|false：是否为官方镜像

```bash
➜  ~ docker search redis -f stars=100
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
redis               Redis is an open source key-value store that…   8140                [OK]
bitnami/redis       Bitnami Redis Docker Image                      145                                     [OK]
```

```bash
➜  ~ docker search redis -f is-official=true
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
redis               Redis is an open source key-value store that…   8140                [OK]
```

```bash
➜  ~ docker search redis -f is-automated=true
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
bitnami/redis                    Bitnami Redis Docker Image                      145                                     [OK]
sameersbn/redis                                                                  80                                      [OK]
...
```

### references 
- [docker search](https://docs.docker.com/engine/reference/commandline/search/)

