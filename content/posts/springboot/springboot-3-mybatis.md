---
title: SpringBoot系列三：Spring Boot MyBatis
date: 2020-02-22T14:34:41+08:00
categories:
- SpringBoot
draft: false
---

在SpringBoot集成MyBatis，需要的依赖为：

```gradle
implementation("org.mybatis.spring.boot:mybatis-spring-boot-starter:2.1.1")
```

该依赖会：
- 自动检测已有的`DataSource`
- 使用`DataSource`创建并注册一个`SqlSessionFactory`实例
- 从`SqlSessionFactory`中获取并注册一个`SqlSessionTemplate`的实例
- 自动扫描mapper，与`SqlSessionTemplate`连接并注入到Spring context中

## 基于注解

只需定义一个普通接口，并使用`@Mapper`注解，然后通过`@Insert`, `@Delete`, `@Update`, `@Select`等MyBatis提供的注解完成增删改查等操作：

```java
@Mapper
public interface PlayerMapper {

  @Insert("insert into player(name, team, join_at) values (#{name}, #{team}, #{joinAt})")
  @Options(useGeneratedKeys = true, keyProperty = "id")
  int insert(Player player);

  @Delete("delete from player where id = #{id}")
  int delete(@Param("id") Integer id);

  @Update("update player set name = #{name}, team = #{team}, join_at = #{joinAt} where id = #{id}")
  int update(Player player);

  @Select("select id, name, team, join_at from player where id = #{id}")
  Player findById(@Param("id") Integer id);

}
```

> 注意：`insert`的时候希望能返回自动生成的主键ID，需要使用`@Options(useGeneratedKeys = true, keyProperty = "id")`，其中，`keyProperty`表示数据库对应实体的ID属性。

然后在需要使用的地方注入`PlayerMapper`即可。

示例代码见：[springboot-mybatis-annotation](https://github.com/nkcoder/springboot-samples/tree/master/springboot-mybatis-annotation)

## 基于XML配置

既需要定义使用`@Mapper`注解的接口类，也需要基于XML的mapper文件。

```java
@Mapper
public interface PlayerMapper {

  int insert(Player player);

  int delete(@Param("id") Integer id);

  int update(Player player);

  Player findById(@Param("id") Integer id);

}
```

```xml
<mapper namespace="org.nkcoder.mybatis.xml.mapper.PlayerMapper">

  <sql id="columns">id, name, team, join_at</sql>

  <insert id="insert" useGeneratedKeys="true" keyProperty="id" parameterType="Player">
    insert into player (`name`, `team`, `join_at`)
    values (#{name}, #{team}, #{joinAt})
  </insert>

  <delete id="delete">
    delete
    from player
    where id = #{id}
  </delete>
</mapper>
```
> 其中，`namespace`表示对应的mapper接口，接口中的方法签名与XML文件语句块的id要相同。对于数据库自增ID，插入时如需在对应实体类返回ID，需添加`useGeneratedKeys="true" keyProperty="id"`。

另外，需要增加MyBatis的相关配置：

```yml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
    default-fetch-size: 100
    default-statement-timeout: 30
  mapper-locations: classpath:mybatis/mapper/*.xml
  type-aliases-package: org.nkcoder.mybatis.xml.entity;
```

- `map-underscore-to-camel-case`: 数据库字段到实体属性映射规则，即下划线到驼峰
- `mapper-locations`: 指定mapper的xml文件位置
- `type-aliases-package`: 类型别名的package，mapper的xml文件中，参数或返回类型不用使用全路径类名

示例代码见：[springboot-mybatis-xml](https://github.com/nkcoder/springboot-samples/tree/master/springboot-mybatis-xml)

## 测试

测试的依赖为：

```gradle
testImplementation('org.mybatis.spring.boot:mybatis-spring-boot-starter-test:2.1.1') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
}
runtime 'com.h2database:h2'
```

它提供了`@MybatisTest`注解，默认配置：

- `SqlSessionFactory`、`SqlSessionTemplate`以及Mapper接口
- 嵌入式的内存数据库
- 开启事务，测试结束回滚

```java
@ExtendWith(SpringExtension.class)
@MybatisTest
@AutoConfigureTestDatabase
public class PlayerMapperTest {

  @Autowired
  private PlayerMapper playerMapper;

  @Test
  public void shouldInsertPlayer() {
    Player player = new Player("dfa", "LA", LocalDate.now());
    int rowInserted = playerMapper.insert(player);

    assertThat(rowInserted).isEqualTo(1);
    assertThat(player.getId()).isGreaterThanOrEqualTo(1);
  }

  // other tests
}
```

> 完整的示例代码见项目：[github-springboot-mybatis-annotation](https://github.com/nkcoder/springboot-samples/tree/master/springboot-mybatis-annotation)以及[github-springboot-mybatis-xml](https://github.com/nkcoder/springboot-samples/tree/master/springboot-mybatis-xml).

### 参考

- [mybatis/spring-boot-starter](https://github.com/mybatis/spring-boot-starter/tree/master/mybatis-spring-boot-samples)
- [mybatis-spring-boot-test](http://mybatis.org/spring-boot-starter/mybatis-spring-boot-test-autoconfigure/)