---
title: Git 提交或克隆代码失败（hung up unexpectedly)
date: 2021-03-06T09:35:55+08:00
tags:
- git
draft: false
---

如果*git*提交 (`push`)或克隆 (`clone`)代码时，报如下的错误：

![git-clone-or-push-hung-up-unexpectedly](/images/tech-by-tag/git-clone-or-push-hung-up-unexpectedly.png)

可以尝试将`http.postBuffer`参数调大一点：

```bash
git config --global http.postBuffer 10M
```

通过`git config --help`可以看到`http.postBuffer`相关的解释：

> `http.postBuffer`
>     Maximum size in bytes of the buffer used by smart HTTP transports when POSTing data to the remote system. For requests larger than this buffer size, HTTP/1.1 and Transfer-Encoding: chunked is used to avoid creating a massive pack file locally. Default is 1 MiB, which is sufficient for most requests.

即，`http.postBuffer`参数是*HTTP*与远程系统交互的缓冲区大小，默认为1MiB（=1.048576MB）。

---

> 参考：
>
> - [The remote end hung up unexpectedly while git cloning](https://stackoverflow.com/questions/6842687/the-remote-end-hung-up-unexpectedly-while-git-cloning/6849424)

