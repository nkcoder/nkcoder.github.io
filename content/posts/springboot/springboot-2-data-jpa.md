---
title: SpringBoot系列二：Spring Boot Data JPA
date: 2020-02-21T09:52:25+08:00
categories:
- SpringBoot
draft: false
---

Spring Boot Data JPA的依赖为：

```gradle
dependencies {
  implementation("org.springframework.boot:spring-boot-starter-data-jpa")
}
```

## 关于JPA的实现

默认的实现是Hibernate，也可以使用其它实现，如`eclipselink`，依赖如下：

```gradle
dependencies {
  implementation("org.springframework.boot:spring-boot-starter-data-jpa") {
    exclude group: "org.hibernate", module: "hibernate-entitymanager"
  }
  implementation "org.eclipse.persistence:eclipselink:2.7.4"
}
```

## 关于实体类（Entity）

数据库的表（table）和JPA的实体类（entity）一一对应，并且实体类需要添加一些注解：

```java
@Getter
@Entity
@Table(name = "player")
@NoArgsConstructor(access = AccessLevel.PRIVATE, force = true)
@RequiredArgsConstructor
public class PlayerEntity {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Integer id;

  private final String name;

  // other fields
}
```

注解说明如下：

- `@Entity`：声明这个类为JPA的实体，并且需要一个`@Id`注解的字段作为主键
- `@Table(name = "player")`: 表示该entity对应的表，`name`属性是表名，默认是entity的名称
- `@NoArgsConstructor(access = AccessLevel.PRIVATE, force = true)`：JPA要求entity有一个无参构造函数，但实际用不上，所以这里创建一个private的无参构造函数，`force`表示将final字段也初始化
- `@RequiredArgsConstructor`：创建一个所有final字段的构造函数。`@Data`默认会创建一个包含所有final字段的构造函数，但如果有无参构造函数，则不会创建了。

## 关于Repository

业务的Repository是一个接口，并且继承JPA的`Repository`或其子类：
```java
@Repository
public interface PlayerRepository extends CrudRepository<Player, Long> {}
```

- `CrudRepository`和`JpaRepository`是`Repository`的两个子类，前者提供基础的CRUD方法，后者在此基础上增加了分页和排序的方法
- `@Repository`用于 Spring 自动扫描并注册为 Bean，同时将 JPA 的异常转换为`DataAccessException`，因为异常转换是通过 proxy 实现的，所以 Repository 类不能声明为 final 的
- 在运行时，Spring Data JPA 会自动生成该接口的一个实现

### Repository 中自定义方法的签名解析规则

如果 Repository 及其子类提供的操作无法满足要求，可以根据命名规范增加新的方法。

Spring Data 会分析方法签名，生成要执行的查询语句，具体的命名规范为：

    动词 + 可选的实体类型 + by + 断言 + 可选的排序

比如：

```java
Player readByIdAndBornAtBetween(Long id, LocalDateTime from, LocalDateTime to);
```

第一个是动词，

- 如果是查询语句，可以是`get`, `read`, `find`
- 保存就是`save`，删除`delete`，还有`exists`等

第二个是实体类型，是可选的，因为`Repository`的第一个参数已经表明了实体的类型。

然后是`by`，后面的断言是实体的字段与条件的组合，组合之间可以使用`and`或`or`连接。

Spring Data JPA 支持以下条件：

    IsAfter, After, IsGreaterThan, GreaterThan
    IsGreaterThanEqual, GreaterThanEqual
    IsBefore, Before, IsLessThan, LessThan
    IsLessThanEqual, LessThanEqual
    IsBetween, Between
    IsNull, Null
    IsNotNull, NotNull
    IsIn, In
    IsNotIn, NotIn
    IsStartingWith, StartingWith, StartsWith
    IsEndingWith, EndingWith, EndsWith IsContaining, Containing, Contains IsLike, Like
    IsNotLike, NotLike
    IsTrue, True
    IsFalse, False
    Is, Equals
    IsNot, Not
    IgnoringCase, IgnoresCase

### 使用@Query 查询

如果通过 Spring Data JPA 解析规则创建的方法不能满足要求或者太麻烦，还可以使用`@Query`通过 SQL 语句自定义查询操作：

```java
@Query(value = "SELECT p FROM Player p where p.name = :name")
Player findByNameParam(@Param("name") String name);

@Query(value = "SELECT p FROM Player p where p.name = ?1")
Player findByName(String name);

@Query(value = "SELECT id, name, team, born_at FROM player where name = ?1", nativeQuery = true)
Player findByNameNativeQuery(String name);
```

如果是更新或删除等修改操作，还需要加上`@Modifying`，如：

```java
@Query(value = "UPDATE player SET team = ?2 WHERE name = ?1", nativeQuery = true)
@Modifying
void updateTeamByName(String name, String team);
```

### 分页与排序

在所有查询都可以额外接受一个参数，`Pageable`或`Sort`，用于分页和排序。它们不能同时出现，因为`Pageable`已经包含了`Sort`。

```java
List<Player> findByTeam(String team, Pageable pageable);

List<Player> findByTeam(String team, Sort sort);

Page<Player> findByTeam(String team, Pageable pageable);

@Query(value = "SELECT p FROM Player p where p.team = ?1")
List<Player> findByTeamAndPage(String name, Pageable pageable);

Page<T> findAll(Pageable pageable);
```

返回值可以是`List<T>`，`Iterable<T>`，以及`Page<T>`，根据实际需要选择其中`Page`的`getContent()`表示该页的内容，`getTotalElements()`表示总的数量。

### 使用 Specification 构造可扩展的查询

Spring Data Jpa 提供`Specification`用于构造灵活的、可扩展的查询。比如，用户可能使用不同的搜索条件组合成不同的查询，我们不能根据每一种可能出现的查询条件都去构造一个 query，`Specification`就是根据实际的条件构造查询，可以组合，因此更灵活，也更容易复用。

使用`Specification`，需要继承`JpaSpecificationExecutor`，它提供了根据 Specification 查询的方法：

```java
@Repository
public interface PlayerRepository extends JpaRepository<Player, Long>,
    JpaSpecificationExecutor<Player> {}
```

```java
List<T> findAll(@Nullable Specification<T> spec);

Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);

List<T> findAll(@Nullable Specification<T> spec, Sort sort);
```

首先我们定义`Specification`，可以直接构造，也可以继承来构造子类：

```java
public class PlaySpecification {

  public static Specification<Player> bornAtYearsAgo(int years) {
    return (Specification<Player>) (root, query, criteriaBuilder) -> {
      LocalDateTime yearsAgo = LocalDateTime.now().minusYears(years);
      return criteriaBuilder.greaterThan(root.get("bornAt"), yearsAgo);
    };
  }

  public static Specification<Player> teamEquals(String team) {
    return (Specification<Player>) (root, query, builder) ->
        builder.equal(root.get("team"), team);
  }

}
```

`Specification`类提供了`where`, `and`, `or`, `not`等条件组合：

```java
playerRepository.findAll(bornAtYearsAgo(1));
playerRepository.findAll(teamEquals("LA"));
playerRepository.findAll(where(bornAtYearsAgo(1)).and(teamEquals("LA")));
```

## 测试

仅测试JPA层，使用`@DataJpaTest`。

```java
@ExtendWith(SpringExtension.class)
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
public class UserRepositoryTest {

  @Autowired private UserRepository userRepository;

  @Test
  public void canFindByName() {
    // some test code
  }
}
```

- `@DataJpaTest`：只会启用JPA层的自动配置。会自动开启事务，每一个测试结束后数据会回滚，所以不会产生脏数据。默认会使用内存数据库，即替换已有的数据源配置，需要h2等内存数据库在classpath上。如果需要使用已有的数据源配置，使用`@AutoConfigureTestDatabase`配置。
- `@AutoConfigureTestDatabase`：`Replace.NONE`表示不替换，使用已有的数据源配置。

> 完整的项目代码见：[github-springboot-data-jpa](https://github.com/nkcoder/springboot-samples/tree/master/springboot-data-jpa).

## 参考

- [Spring in Action, Fifth Edition](https://www.manning.com/books/spring-in-action-fifth-edition)
- [Spring Data JPA - Reference Documentation](https://docs.spring.io/spring-data/jpa/docs/2.1.9.RELEASE/reference/html/#specifications)
- [REST Query Language with Spring Data JPA Specifications](https://www.baeldung.com/rest-api-search-language-spring-data-specifications)