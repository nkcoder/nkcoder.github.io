---
title: SpringBoot系列五：用scala开发
date: 2020-03-15T14:33:17+08:00
draft: false
categories:
- SpringBoot
---

Scala是一门面向对象和函数式语言，主要基于JVM平台，因此可以与Java互操作。与Java相比，Scala表达力更强，代码更简洁。

SpringBoot是Java生态最流行的Web框架，其实我们完全可以使用Scala在SpringBoot框架中开发。

本示例用到的主要技术栈为：SpringBoot、Scala、JPA、MySQL。

## 添加Scala依赖

```gradle
implementation 'org.scala-lang:scala-library:2.13.1'
implementation 'com.typesafe.scala-logging:scala-logging_2.13:3.9.2'
implementation 'com.fasterxml.jackson.module:jackson-module-scala_2.13:2.10.2'

testImplementation 'org.scalatest:scalatest_2.13:3.1.0'
```

## 定义实体类

首先定义JPA需要的实体类：

```scala
import java.lang.Long
import java.time.LocalDateTime

import javax.persistence._
import org.nkcoder.scala.api.Article

import scala.annotation.meta.field
import scala.beans.BeanProperty

@Entity
@Table(name = "article")
class ArticleEntity(@BeanProperty var subject: String,
                    @BeanProperty var content: String,
                    @BeanProperty var updatedAt: LocalDateTime,
                    @BeanProperty var createdBy: Long) {

  @(Id @field)
  @BeanProperty
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  var id: Long = _

  // for jpa
  private def this() = this(null, null, null, null)

}
```

其中，

- `@BeanProperty`表示给字段生成符合JavaBean的getter/setter方法，比如对于*subject*字段，scala生成的getter/setter都是字段本身，如果它出现在`=`的左边则为set，右边则为get，而使用了`@BeanProperty`注解后，会添加`getSubject()`和`setSubject()`，主要是JPA序列会和反序列化需要。如果参数是Boolean类型，而你希望生成的*getter*为*isXXX*，则应该使用`@BooleanBeanProperty`。

- JPA需要一个空的构造函数，而Scala中带构造参数的类是没有无参构造函数的，所以需要单独定义，*private*即可。
- `@(Id @field)`：表示`@Id`注解仅应用在字段上，而不会应用在getter/setter等方法上。

## 定义Repository

这个与Java基本没区别，使用Scala的语法即可：

```scala
import org.springframework.data.jpa.repository.JpaRepository
import org.springframework.stereotype.Repository

@Repository
trait ArticleRepository extends JpaRepository[ArticleEntity, Long]
```

## Service与Controller

因为框架使用的还是SpringBoot和JPA，因此API的使用是一样的，这里仅给出部分示例代码供参考，项目完整代码在[SpringBoot-Scala](https://github.com/nkcoder/springboot-samples/tree/master/springboot-scala).

```scala
@Service
class ArticleServiceImpl @Autowired()(articleRepository: ArticleRepository)
    extends ArticleService {

  override def create(
    createArticleRequest: CreateOrUpdateArticleRequest
  ): Long = {
    import createArticleRequest._
    articleRepository
      .save(
        ArticleEntity
          .create(subject = subject, content = content, createdBy = createdBy)
      )
      .id
  }
}
```

```scala
@RestController
@RequestMapping(Array("/articles"))
class ArticleController @Autowired()(articleService: ArticleService) {

  @PostMapping(Array(""))
  @ResponseStatus(HttpStatus.CREATED)
  def create(
    @RequestBody createArticleRequest: CreateOrUpdateArticleRequest
  ): Long =
    articleService.create(createArticleRequest)

}
```

> 完整的示例代码见项目：[github-springboot-scala](https://github.com/nkcoder/springboot-samples/tree/master/springboot-scala)

## 参考

- [Spring Boot with Scala](https://www.javacodegeeks.com/2016/02/spring-boot-scala.html)
- [Usage of @field annotations
](https://stackoverflow.com/questions/37014280/usage-of-field-annotations)



