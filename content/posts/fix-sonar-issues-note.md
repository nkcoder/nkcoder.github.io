---
title: 记录一次修复SonarCube issue的过程
date: 2020-02-19T23:28:25+08:00
tags:
- sonar
draft: false
---

本地配置了[Jacoco](https://www.eclemma.org/jacoco/)的覆盖率：行覆盖率为 0.8，但是排除了一些特殊的包和类。

接入[SonarCube](https://www.sonarqube.org)后，显示的指标惨不忍睹：

```text
Bugs: 44    Vulnerability: 54
Bad Smell: 531    Debt: 7 days
Coverage: 4.4%    Duplicated Blocks: 19
```

于是开始着手修复。

## Bugs

这个分类里只有两种问题，第一种问题是缺少构造函数。

```text
Add a constructor to the class.
```

被标记为缺少构造函数的 class 都是使用了 lombok 的`@Builder`注解，比如：

```java
@Getter
@Builder
public class Book implements Serializable {
  private static final long serialVersionUID = -812324454545L;

  private String id;
  private String remarks;

  // other fields
}
```

`@Builder`默认会创建一个全参的私有构造函数，所以其实不需要额外定义构造函数，估计 Sonar 无法处理这种情况，我们没有权限从 sonar 层面解决问题，只能去适应这个规则。为了解决这个问题，可以显式定义一个私有的全参构造函数，可以使用 lombok 注解，如：

```java
@Getter
@Builder
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Book implements Serializable {
  private static final long serialVersionUID = -812324454545L;

  private String id;
  private String remarks;

  // other fields
}
```

另外一种问题是：如果类继承了`Serializable`接口，其中`value-based`类型的字段应该标记为 `transient`：

```text
Make this value-base field transient so that it is not included in the serialization of the class.
```

`value-based`的类主要是`java.time`包下除`Clock`类之外的所有类。

这其实不能算是个 bug，根据具体情况讨论下：

- 如果使用 JPA，且该日期类型的字段确实需要存入数据库中，将字段标记为`transient`显然是不行的。解决办法，如果有权限，可以修改 Sonar 的配置规则，如果无法修改 Sonar 配置规则，不得已的方法就是在需要的地方忽略。关于这个问题的讨论可以参考[Github 上的这个 issue](https://github.com/jhipster/generator-jhipster/issues/4553)。

```java
@SuppressWarnings("squid:S3437")
```

- 如果该类不会被序列化（比如只是用于微服务接口调用的数据传输），则该错误提示就是代码优化的信号。比如我们当前的场景，很多类似的类都只用于通过 HTTP 调用外部系统。解决方法，一种方式是该类不继承`Serializable`接口，另一种方式是将日期类型的字段标记为`transient`，我这里采用的是第二种方式，可能移植性会更好一点：

```java
@Getter
@Builder
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Book implements Serializable {
  private static final long serialVersionUID = -812324454545L;

  private String id;

  transient private LocalDateTime createAt;

  // other fields
}
```

这样修改完后，`Bug`类型的问题数量就为 0 了。

## Vulnerability

漏洞都是一个类型的问题：可变类型(数组、列表)的数据应该复制一份，而不应该直接返回或直接使用

```text
Store a copy of "books"
```

如果一个对象拥有一个列表类型的数据，如果直接返回给调用者，因为是可变的，如果数据被修改，原对象中的数据也就被修改了，所以会有安全问题。

这个问题比较好处理，在数据设值(构造函数或 Setter 方法)或取值(如 Getter 方法)时将其封装为不可变类型即可，如：

```java
private List<String> books;

public void setBooks(List<String> books) {
  this.books = Collections.unmodifiableList(books);
}
```

> 注意：`Collections.unmodifiableList()`的参数不能为 null，所以需要根据自己的业务看是否需要提前判断或处理。

只有这一个类型的问题，修复后，数量降为 0。

## Bad Smell

代码坏味道里有很多个来源，数量最多的一个来源是，**请移除未被使用的私有字段**：

```text
Remove this unused "userId" private field
```

Sonar 的意思是这个 private 字段没有被使用，应该被移除掉。怎么会没有使用呢，在 IDE 中可以查看到很多有效的引用。
不过发现所有这些没被引用的字段，都是来自于被 lombok 的`@Getter`注解的类，难道是 Sonar 并没有完全识别并支持 lombok 的注解导致误报吗？
搜索了下 Sonar 相关的配置，然后去 Pipeline 上 check 了下流程配置，果然有所发现。

如果需要支持 lombok 等框架，需要将相关依赖配置在`sonar.java.libraries`参数下，如：

```text
sonar.java.libraries = build/libs/lombok-1.18.8.jar
```

现在很多应用都是使用 SpringBoot，最后 build 出来的结果是一个可执行的 jar 包，依赖都在该 jar 包中，所以没法直接引用 jar 包中的依赖。

一种简单的解决方法，可以在 build 工具，如 maven/gradle 中，定义一个 goal/task，将需要的依赖包拷贝到一个指定目录，然后就可以引用了。比如使用`maven-dependency-plugin`，配置如下：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-dependency-plugin</artifactId>
      <version>3.1.1</version>
      <executions>
        <execution>
          <id>copy</id>
          <phase>package</phase>
          <goals>
            <goal>copy</goal>
          </goals>
          <configuration>
            <artifactItems>
              <artifactItem>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <overWrite>false</overWrite>
                <outputDirectory>${project.build.directory}/libs</outputDirectory>
              </artifactItem>
            </artifactItems>
            <outputDirectory>${project.build.directory}/libs</outputDirectory>
            <overWriteReleases>false</overWriteReleases>
            <overWriteSnapshots>true</overWriteSnapshots>
            <overWriteIfNewer>true</overWriteIfNewer>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

引用如下：

```text
sonar.java.libraries = target/libs/lombok-1.18.8.jar
```

在 gradle 中配置如下：

```gradle
task copyDependencies(type: Copy) {
//    from configurations.default
    from configurations.default.filter {it.name.startsWith("lombok")}
    into "${project.buildDir}/libs"
}
```

引用如下：

```text
sonar.java.libraries = build/libs/lombok-1.18.8.jar
```

该配置生效后，坏味道降到了 59 个，差不多是原来的十分之一。

Bad Smell 的第二个来源是，**构造函数参数超过了允许的 7 个**：

```text
Constructor has 9 parameters, which is greater than 7 authorized.
```

如果对领域对象有控制权，构造函数的参数太多，那就需要去确认下是否可以对领域对象进行合理拆分。但是在类似我们当前的场景下，这些数据对象是根据第三方服务的接口定义的，所以拆分就不太合理，这个 bad smell 就不打算修复了。

Bad Smell 的第三个来源，也是最后一个来源，**重复代码块**：

```text
1 duplicated blocks of code must be removed.
```

一般情况下，如果出现代码重复，应该尽量提取公共代码，去除重复。

但是如果重复代码块出现在不同的 Domain，按照 DDD 的原则，子域的交集部分应该放在共享内核中(Shared Kernel)，修改共享内核需要小心并注意沟通。

这个项目里为了偷懒，也为了不至于将简单的项目复杂化，暂时忽略这个 bad smell 了。

修复加忽略后，bad smell 的数量为 50，Debt 为 2 天。

## Coverage

覆盖率只有 4.4%，实在不能忍。分析了下，主要原因是我们在 jacoco 中配置了根据 package 进行忽略的规则，对于这些忽略的 package 中的类，Sonar 会直接忽略相关的测试，认为这些类的覆盖率为 0。

Jacoco 中忽略的 package 主要有：`Constant`、`Exception`、`dto`, `util`, `config`以及`client`。这些 package 中的类都有对应的测试去覆盖，只是有些类的覆盖率不到配置的阀值 80%。

将`Constant`、`Exception`、`dto`, `util`, `config`等包都从排除中去掉，然后大家一起补上相关的单元测试，最后的单元测试覆盖率为 88.2%。

至于`client`包，根据小伙伴反馈，第三方提供的基于 socket 的示例代码不容易编写单元测试，可能需要先重构。这部分的测试暂时先忽略了。

经过努力修复之后，结果目前还可以接收，Sonar 上的统计为：

```text
Bugs: 0    Vulnerability: 0
Bad Smell: 50    Debt: 2 days
Coverage: 88.2%    Unit Tests: 184
```

## 参考

1. [Copying specific artifacts](https://maven.apache.org/plugins/maven-dependency-plugin/examples/copying-artifacts.html)
2. [Copying project dependencies](https://maven.apache.org/plugins/maven-dependency-plugin/examples/copying-project-dependencies.html)
3. [sonarqube + lombok = false positives](https://stackoverflow.com/questions/46362965/sonarqube-lombok-false-positives/46365394)
4. [Gradle equivalent to Maven's “copy-dependencies”?](https://stackoverflow.com/questions/26697999/gradle-equivalent-to-mavens-copy-dependencies)
5. [Extract specific JARs from dependencies](https://stackoverflow.com/questions/11297612/extract-specific-jars-from-dependencies)

