---
title: SpringBoot系列一：Spring Boot Devtools
date: 2020-02-20T08:17:39+08:00
categories:
- SpringBoot
draft: false
---

`spring-boot-devtools`提供了一些辅助开发的功能，比如：

- 自动禁用依赖库或模板的缓存，启动 web 模块的 debug 日志
- 自动重启：保存文件（Eclipse）或者 Build（Intellij Idea）时自动重启应用
- LiveReload 自动触发浏览器的刷新（资源发生变化时）
- 全局配置：提供全局配置文件，对所有包含 devtools 的应用生效

在实际开发中，自动重启功能比较有用，所以这里主要介绍自动重启的配置与使用。

## 添加依赖

maven 配置：

```maven
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <optional>true</optional>
</dependency>
```

gradle 配置：

```gradle
configurations {
  developmentOnly
  runtimeClasspath {
    extendsFrom developmentOnly
  }
}
dependencies {
  developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

- 在依赖配置中，如 maven 中添加`optional`标记，在 gradle 中使用自定义的 developmentOnly，表示该依赖不会被传递。
- 在生产环境下，`devtool`会自动禁用，通过`java -jar`启动应用被认为是生产环境。

## 自动重启

如果 `classpath` 上的文件发生了变化，应用会自动重启。

自动重启的触发：

- 在 Eclipse 中，被修改的文件保存时会触发重启
- 在 Intellij Idea 中，使用`Build -> Build Project (CMD+F9)`手动触发

> 在 Intellij Idea 中也可以配置在文件保存时触发重启，但需要使以下两项配置生效：
`1. Preference -> compiler -> 选中 Build project automatically`
`2. Help -> Find action(Shift + CMD + A) -> 选中 compiler.automake.allow.when.app.running`

自动重启原理：Spring Boot 会使用两个 `classloader`，不会发生变化的类（主要是第三方依赖库）会被加载到 `base classloader` 中，开发过程中使用的类会被加载到 `restart classloader`，应用重启时，只有 `restart classloader` 会重启，而 `base classloader` 不变，所以重启速度会更快一些。另一方面，如果依赖发生了变化，则需要手动重启。

### 扩展自动重启监视的目录

```
spring.devtools.restart.exclude=static/**,public/**
spring.devtools.restart.additional-paths=mypackage/**
```

### 禁用自动重启：

可以有两种方式：

在 application.properties 中配置：

```
spring.devtools.restart.enabled=false
```

或者在调用`SpringApplication.run()`之前调用：

```
System.setProperty("spring.devtools.restart.enabled", "false");
```

> 完整的示例代码见: [github-springboot-devtools](https://github.com/nkcoder/springboot-samples/tree/master/springboot-devtools).

## 参考

- [Spring in Action, Fifth Edition](https://www.manning.com/books/spring-in-action-fifth-edition)
- [Spring Boot Developer Tools: How to enable automatic restart in IntelliJ IDEA](https://dev.to/suin/spring-boot-developer-tools-how-to-enable-automatic-restart-in-intellij-idea-1c6i)
- [Developer Tools](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html)

