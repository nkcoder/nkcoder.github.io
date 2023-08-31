---
title: Docker容器：查看容器状态
date: 2020-07-09T06:52:13+08:00
categories:
- docker
draft: false
---

本文涉及到的主要命令如下：

- `docker container inspect`：查看容器的详细信息
- `docker container log`：查看容器的日志
- `docker container stats`：查看容器的实时统计

## 查看容器的详细信息

`docker container inspect`或`docker inspect`命令会显示容器的详细信息，默认以`JSON`格式输出：

```json
➜  ~ docker container inspect 3f3c3d02c947
[
    {
        "Id": "3f3c3d02c9474ad8d152548d9db81bcd86d956e9686828b537a9a9708b631af2",
        "Created": "2020-06-01T22:58:05.916162Z",
        "Path": "/tini",
        "Args": [
            "--",
            "/usr/local/bin/docker-entrypoint.sh",
            "eswrapper"
        ],
      ...
    }
    ...
]
```

默认的输出信息很多，可以通过`-f, --format`格式化输出：

```bash
➜  ~ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 3f3c3d02c947
172.17.0.2
➜  ~ docker inspect --format='{{.NetworkSettings.Networks.bridge.IPAddress}}' 3f3c3d02c947
172.17.0.2
➜  ~ docker inspect --format='{{.LogPath}}' 3f3c3d02c947
/var/lib/docker/containers/3f3c3d02c9474ad8d152548d9db81bcd86d956e9686828b537a9a9708b631af2/3f3c3d02c9474ad8d152548d9db81bcd86d956e9686828b537a9a9708b631af2-json.log
```

如果只想查看某个子节点，比如`NetworkSettings`下的`Ports`，因为`Ports`本身是个对象，默认只会显示节点中值的集合，可以使用模版函数`json`，将对象以json格式输出：

```bash
➜  ~ docker inspect --format='{{json .NetworkSettings.Ports}}' 3f3c3d02c947
{"9200/tcp":[{"HostIp":"0.0.0.0","HostPort":"9200"}],"9300/tcp":[{"HostIp":"0.0.0.0","HostPort":"9300"}]}
```

## 查看容器的运行日志

`docker container logs`或`docker logs`可以查看容器的日志。

> 该命令仅适用于使用`json-file`和`journald`作为日志驱动的容器。

常用的参数说明如下：

- `--tail`: 显示最新的多少行日志，默认是所有
- `-f, --follow`: 动态显示更新的日志
- `-t, --timestap`: 显示每行日志的时间戳
- `details`: 显示额外的详细信息
- `--since`: 仅显示某个时间点（比如：2020-06-02T10:23:37）之后的日志，或者使用相对值（42m表示42分钟）
- `--until`: 仅显示某个时间点（比如：2020-06-02T10:23:37）之前的日志，或者使用相对值（42m表示42分钟）

```bash
➜  ~ docker logs 3f3c3d02c947
{"type": "server", "timestamp": "2020-06-01T22:58:11,829Z", "level": "INFO", "component": "o.e.e.NodeEnvironment", "cluster.name": "docker-cluster", "node.name": "3f3c3d02c947", "message": "using [1] data paths, mounts [[/ (overlay)]], net usable_space [49.5gb], net total_space [58.4gb], types [overlay]" }
{"type": "server", "timestamp": "2020-06-01T22:58:11,837Z", "level": "INFO", "component": "o.e.e.NodeEnvironment", "cluster.name": "docker-cluster", "node.name": "3f3c3d02c947", "message": "heap size [1gb], compressed ordinary object pointers [true]" }
...

➜  ~ docker logs -f --tail 3 3f3c3d02c947
{"type": "server", "timestamp": "2020-06-01T22:58:27,353Z", "level": "INFO", "component": "o.e.x.i.a.TransportPutLifecycleAction", "cluster.name": "docker-cluster", "node.name": "3f3c3d02c947", "message": "adding index lifecycle policy [slm-history-ilm-policy]", "cluster.uuid": "q8H8WMzpS6qMLzP1IgTjGQ", "node.id": "sTkmBkS4QwG4h4IaWBMzQQ"  }
{"type": "server", "timestamp": "2020-06-01T22:58:27,480Z", "level": "INFO", "component": "o.e.l.LicenseService", "cluster.name": "docker-cluster", "node.name": "3f3c3d02c947", "message": "license [d7cc442a-9316-4f91-8547-5c75384da38b] mode [basic] - valid", "cluster.uuid": "q8H8WMzpS6qMLzP1IgTjGQ", "node.id": "sTkmBkS4QwG4h4IaWBMzQQ"  }
{"type": "server", "timestamp": "2020-06-01T22:58:27,482Z", "level": "INFO", "component": "o.e.x.s.s.SecurityStatusChangeListener", "cluster.name": "docker-cluster", "node.name": "3f3c3d02c947", "message": "Active license is now [BASIC]; Security is disabled", "cluster.uuid": "q8H8WMzpS6qMLzP1IgTjGQ", "node.id": "sTkmBkS4QwG4h4IaWBMzQQ"  }

➜  ~ docker logs -t --since '2020-06-01T22:58:27.483306900Z' 3f3c3d02c947
2020-06-01T22:58:27.483306900Z {"type": "server", "timestamp": "2020-06-01T22:58:27,482Z", "level": "INFO", "component": "o.e.x.s.s.SecurityStatusChangeListener", "cluster.name": "docker-cluster", "node.name": "3f3c3d02c947", "message": "Active license is now [BASIC]; Security is disabled", "cluster.uuid": "q8H8WMzpS6qMLzP1IgTjGQ", "node.id": "sTkmBkS4QwG4h4IaWBMzQQ"  }
```

## 查看容器的实时统计

可以使用`docker container stats`或`docker stats`查看容器的运行状态。

```bash
➜  ~ docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
19469f79b8a4        kafka_zookeeper_1   0.14%               78.54MiB / 3.848GiB   1.99%               5.63kB / 3.15kB     0B / 0B             23
e69d55f4eb91        kafka_kafka_1       0.00%               0B / 0B               0.00%               0B / 0B             0B / 0B             0
3f3c3d02c947        vigilant_merkle     0.49%               1.283GiB / 3.848GiB   33.34%              1.73kB / 0B         0B / 0B             55
```

默认返回所有运行的容器的实时状态，不停刷新。主要参数有：

- `--all, -a`: 返回所有容器的状态，包括已停止的容器，但是停止的容器没有状态数据
- `--no-stream`: 仅返回第一次的结果，不会动态地实时刷新
- `--no-trunc`: 数据不截断

也可以指定具体的容器ID，多个容器ID以空格分隔：

```bash
➜  ~ docker stats --no-trunc --no-stream 3f3c3d02c947
CONTAINER ID                                                       NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
3f3c3d02c9474ad8d152548d9db81bcd86d956e9686828b537a9a9708b631af2   vigilant_merkle     0.46%               1.283GiB / 3.848GiB   33.35%              2.01kB / 0B         0B / 0B             55
```

### reference docs:

- [docker logs](https://docs.docker.com/engine/reference/commandline/logs/)
- [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/)
- [docker stats](https://docs.docker.com/engine/reference/commandline/stats/)