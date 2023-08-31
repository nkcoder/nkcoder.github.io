---
title: Docker镜像：删除镜像
date: 2020-06-16T06:40:26+08:00
categories:
- docker
draft: false
---

删除本地镜像命令为：`docke image rm`，等价的命令为：`docker rmi`和`docker image remove`。

```bash
➜  ~ docker image rm --help

Usage:	docker image rm [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Aliases:
  rm, rmi, remove

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
```

可以使用镜像ID、镜像tag、镜像摘要来标识并删除一个镜像：

```bash
➜  ~ docker rmi nkcoder/freeimmi_mini:4.12
➜  ~ docker rmi 5ec653d76efc
➜  ~ docker rmi sha256:f2e0e5a3263ba1d749768ba173dd769aade7616d5593065904efdbf77742cd87
```

如果一个镜像关联了多个标签，是不能直接删除的：

```bash
➜  ~ docker images alpine
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              950                 f70734b6a266        2 weeks ago         5.61MB
alpine              latest              f70734b6a266        2 weeks ago         5.61MB

➜  ~ docker image rm f70734b6a266
Error response from daemon: conflict: unable to delete f70734b6a266 (must be forced) - image is referenced in multiple repositories
```

必须先移除tag引用才可以删除镜像（最后一个tag被删除时，镜像被自动删除），或者使用`-f`参数强制删除：

```bash
➜  ~ docker rmi alpine:950
Untagged: alpine:950
➜  ~ docker rmi alpine:latest
Untagged: alpine:latest
Untagged: alpine@sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54
Deleted: sha256:f70734b6a266dcb5f44c383274821207885b549b75c8e119404917a61335981a
Deleted: sha256:3e207b409db364b595ba862cdc12be96dcdad8e36c59a03b7b3b61c946a5741a
```

如果镜像当前正被container使用，则必须先停止和删除所有使用该镜像的container：

```bash
➜  ~ docker rmi nkcoder/freeimmi_mini:4.12
Error response from daemon: conflict: unable to remove repository reference "nkcoder/freeimmi_mini:4.12" (must force) - container 5b58606bfe7f is using its referenced image 5ec653d76efc

➜  ~ docker container rm 5b58606bfe7f
5b58606bfe7f
➜  ~ docker rmi nkcoder/freeimmi_mini:4.12
Untagged: nkcoder/freeimmi_mini:4.12
Untagged: nkcoder/freeimmi_mini@sha256:f2e0e5a3263ba1d749768ba173dd769aade7616d5593065904efdbf77742cd87
Deleted: sha256:5ec653d76efccc0f2b235676aa1251dac7d268a11121f9638dac92e07912709e
Deleted: sha256:1730452bc8977999c6be96bf1e518ad4a7a3f89911d57c75163798267f573a52
Deleted: sha256:0fca82a48216c21010bc1f8732e85b33474ca1302610e239d19094db1913c7d8
Deleted: sha256:1bfeebd65323b8ddf5bd6a51cc7097b72788bc982e9ab3280d53d3c613adffa7
```

### reference: 
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)


