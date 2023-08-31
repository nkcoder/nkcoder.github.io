---
title: HTTPie用法介绍
date: 2020-02-19T23:26:04+08:00
tags:
- httpie
draft: false
---

`HTTPie`是基于命令行的 HTTP 客户端，功能与`CURL`类似，但更简单、易用。

但是如果对用法不熟悉，经常容易混淆，比如查询参数用`==`还是`=`，请求体的数据用`=`还是`:=`等，这里记录日常开发中的常见用法作为备忘。

以下示例大多来自[HTTPie 官方文档](https://httpie.org/doc)，如果有不清楚的地方，请移步参考。

## 语法

```text
http [flags][method] URL [ITEM [ITEM]]
```

## 默认请求方法

如果`[METHOD]`被忽略，没有请求数据，则默认为`GET`，如果有请求数据，则默认为`POST`：

```shell
$ http localhost:8080/books
$ http localhost:8080/books id:=1 name='Spring in action'
```

## localhost 可忽略

默认协议为 http，所以`http://`可以忽略，如果 URL 是`localhost`，也可以忽略，如：

```shell
$ http :8080/books
```

## HTTP headers

格式为：header-name:header-value

```shell
$ http get :8080/books/1 Authorization:my-auth-token
```

请求数据默认会被序列化为 json，同时，HTTPie 默认设置以下 header：

```text
Content-Type: application/json
Accept: application/json, */*
```

## URL 请求参数

当然可以将请求参数追加在 URL 后面，但是使用`param==value`的方式可以不用关心特殊字符的转义：

```shell
$ http get :8080/books name=="my book" author=="kate"
```

```text
GET /?name=my+book&author=Kate
```

## 请求体数据

字符串类型的数据用`=`，非字符串类型或 Raw Json 的数据用`:=`，引用文件数据使用`@data-file`:

```shell
$ http post :8080/books id:=1 name="Akka in Action" soldOut:=false
```

```json
{
  "id": 1,
  "name": "Akka in Action",
  "soldOut": false
}
```

```shell
$ http PUT api.example.com/person/1 \
  name=John \
  age:=29 married:=false hobbies:='["http", "pies"]' \ # Raw JSON
  description=@about-john.txt \ # Embed text file
  bookmarks:=@bookmarks.json # Embed JSON file
```

```text
PUT /person/1 HTTP/1.1
Accept: application/json, _/_
Content-Type: application/json
Host: api.example.com

{
  "age": 29,
  "hobbies": [
    "http",
    "pies"
  ],
  "description": "John is a nice guy who likes pies.",
  "married": false,
  "name": "John",
  "bookmarks": {
    "HTTPie": "http://httpie.org",
  }
}
```

如果字段较多，数据较复杂，最好是直接加载数据文件：

```shell
$ http PUT api.example.com/person/1 < person.json
```

## 下载文件

重定向方式：

```shell
$ http example.org/file > file
```

`wget`方式：

```shell
$ http --download example.org/file
```

## form 提交

普通 form：

```shell
$ http --form POST api.example.org/person/1 name='John Smith'
```

```text
POST /person/1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded; charset=utf-8

name=John+Smith
```

文件上传的 form：

```shell
$ http -f POST example.com/jobs name='John Smith' cv@~/Documents/cv.pdf
```

等价与以下 HTML：

```xml
<form enctype="multipart/form-data" method="post" action="http://example.com/jobs">
  <input type="text" name="name" />
  <input type="file" name="cv" />
</form>
```

## 常用 flag

```text
--headers, -h  仅输出headerxinxi
--stream, -S  将response按行stream输出，而不是等待所有response然后输出
--output FILE, -o FILE  输出到文件，而不是控制台
--verbose, -v  输出详细的request和response信息
--json, -j  默认，命令行的数据会被序列化为JSON对象
--form, -f  命令行的数据会被序列化为FORM的字段，并将Content-Type设置为application/x-www-form-urlencoded
```

## 参考

1. [HTTPie doc](https://httpie.org/doc)


