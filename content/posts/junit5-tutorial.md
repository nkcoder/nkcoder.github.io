---
title: Junit5 用法介绍
date: 2020-02-19T23:23:29+08:00
tags:
- junit5
draft: false
---

## Junit 5 简介

Junit 5 由三个子项目构成：

    JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

- JUnit Platform：在 JVM 上启动测试框架的基础
- JUnit Jupiter：提供在 JUnit 5 环境下编写测试和扩展的编程模型和扩展模型
- JUnit Vintage：兼容 JUnit 3 和 JUnit 4 环境编写的测试

Junit 5 需要 Java 8 以上版本。

如果在新环境下，仅支持 JUnit 5，gradle 依赖如下：

    testImplementation('org.junit.jupiter:junit-jupiter:5.4.2')

如果需要支持 JUnit 3 或 JUnit 4，需要加入 junit-vintage 的依赖，gradle 依赖如下：

    testImplementation('org.junit.jupiter:junit-jupiter:5.4.2')
    testImplementation('org.junit.vintage:junit-vintage-engine:5.4.1')
    testImplementation('junit:junit:4.12')

## 常用注解汇总

| 注解                   | 说明                                                                                                                                            |
| :--------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| @Test                  | 表明这是一个测试方法，类比于 JUnit 4 的`@Test`但是不支持任何参数                                                                                |
| @ParameterizedTest     | 带参数的测试                                                                                                                                    |
| @RepeatedTest          | 使测试重复执行                                                                                                                                  |
| @TestFactory           | 该方法是一个支持动态测试的测试工厂                                                                                                              |
| @TestTemplate          | 该方法是一个测试模板，支持在不同的测试场景下的多次调用                                                                                          |
| @TestMethodOrder       | 配置测试方法的执行顺序                                                                                                                          |
| @TestInstance          | 配置测试实例的生命周期                                                                                                                          |
| @DisplayName           | 自定义显示名称                                                                                                                                  |
| @DisplayNameGeneration | 自定义显示名称生成器                                                                                                                            |
| @BeforeEach            | 该方法会在当前类的`每一个`测试方法`之前`执行，包括：`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`, 与 JUnit 4 的`@Before` 类似 |
| @AfterEach             | 该方法会在当前类的`每一个`测试方法`之后`执行，包括：`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`, 与 JUnit 4 的`@After` 类似  |
| @BeforeAll             | 该方法在当前类的`所有`测试方法`之前`执行，类似于 JUnit 4 的`@BeforeClass`                                                                       |
| @AfterAll              | 该方法在当前类的`所有`测试方法`之后`执行，类似于 JUnit 4 的`@AfterClass`                                                                        |
| @Tag                   | 用于测试的过滤，类似于 JUnit 4 的 Category 和 TestNG 的 Group                                                                                   |
| @Disabled              | 禁用当前测试，类比于 JUnit 4 的`@Ignore`                                                                                                        |
| @TempDir               | 通过字段注入或参数注入提供一个临时目录                                                                                                          |

## 自定义注解

比如我们要给一个类的所有测试方法加上一个`@Tag("fast")`，我们可以定义一个新的注解，然后用`@FastTest`替换`@Test`:

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Test
@Tag("fast")
public @interface FastTest {

}
```

## @DisplayName

可以用在 class 和 method 上，测试报告和 IDE 上会使用`@DisplayName`配置的名称，支持特殊字符和 emoji。

```java
@DisplayName("display name demo")
public class DisplayNameDemo {

  @Test
  @DisplayName("simple name test")
  void testSimpleName() {
  }

  @Test
  @DisplayName("╯°□°）╯")
  void testSpecialCharacters() {
  }

  @Test
  @DisplayName("😱")
  void testWithDisplayNameContainingEmoji() {
  }

}
```

## @DisplayNameGeneration

只能用在 class 上，如果与`@DisplayName` 一起使用，`@DisplayNameGeneration` 不生效。

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
public class DisplayNameGeneratorDemo {

  @Test
  public void test_replace_underscore_with_space() {

  }

}
```

## Assertions

断言在`org.junit.jupiter.api.Assertions`包下，可以静态导入。

`assertEquals`和`assertTrue`的消息放在第三个参数，而不是第一个。

```java
@Test
void standardAssertions() {
  assertEquals(2, 1 + 1);
  assertEquals(4, 2 * 2, "The optional failure message is now the last parameter");
  assertTrue('a' < 'b', () -> "Assertion messages can be lazily evaluated -- "
      + "to avoid constructing complex messages unnecessarily.");
}
```

`assertAll`表示分组断言，每一个分组都会执行，任何一个分组执行失败都会导致该测试失败。

```java
@Test
void groupedAssertions() {
  // In a grouped assertion all assertions are executed, and all
  // failures will be reported together.
  assertAll("person",
      () -> assertEquals("Jane", "Jane"),
      () -> assertEquals("Doe", "Doe")
  );
}
```

但是同一个分组中（block）的断言之间有前后依赖关系，即如果前面的断言失败了，后面的断言不会执行。

```java
@Test
void dependentAssertions() {
  // Within a code block, if an assertion fails the
  // subsequent code in the same block will be skipped.
  assertAll("properties",
      () -> {
        String firstName = "Jane";
        assertNotNull(firstName);

        // Executed only if the previous assertion is valid.
        assertAll("first name",
            () -> assertTrue(firstName.startsWith("J")),
            () -> assertTrue(firstName.endsWith("e"))
        );
      },
      () -> {
        // Grouped assertion, so processed independently
        // of results of first name assertions.
        String lastName = "Doe";
        assertNotNull(lastName);

        // Executed only if the previous assertion is valid.
        assertAll("last name",
            () -> assertTrue(lastName.startsWith("D")),
            () -> assertTrue(lastName.endsWith("e"))
        );
      }
  );
}
```

`assertThrows`测试抛出的异常。

```java
@Test
void exceptionTesting() {
  Exception exception = assertThrows(ArithmeticException.class, () -> divide(1, 0));
  assertEquals("/ by zero", exception.getMessage());
}

private int divide(int a, int b) {
  return a / b;
}
```

`assertTimeout`测试超时，`assertTimeoutPreemptively`表示抢占超时，即如果超时了，方法还没有返回，则立即中断执行。

> 需要注意的是：`assertTimeoutPreemptively`中的参数方法 Executable 是在独立的线程中执行的，而不是在调用线程中执行，因此如果 executable 中的代码依赖 ThreadLocal，可能会导致意料之外的结果。

```java
@Test
void timeoutNotExceeded() {
  // The following assertion succeeds.
  assertTimeout(ofMinutes(2), () -> {
    // Perform task that takes less than 2 minutes.
  });
}

@Test
void timeoutNotExceededWithResult() {
  // The following assertion succeeds, and returns the supplied object.
  String actualResult = assertTimeout(ofMinutes(2), () -> "a result");
  assertEquals("a result", actualResult);
}

@Test
void timeoutExceeded() {
  // The following assertion fails with an error message similar to:
  // execution exceeded timeout of 10 ms by 91 ms
  assertTimeout(ofMillis(10), () -> {
    // Simulate task that takes more than 10 ms.
    Thread.sleep(100);
  });
}

@Test
void timeoutExceededWithPreemptiveTermination() {
  // The following assertion fails with an error message similar to:
  // execution timed out after 10 ms
  assertTimeoutPreemptively(ofMillis(10), () -> {
    // Simulate task that takes more than 10 ms.
    Thread.sleep(100);
  });
}
```

第三方测试库如`Hamcrest`可以继续使用，但 JUnit 5 没有提供方法可以接受`Matcher`作为参数。

```java
@Test
void assertWithHamcrestMatcher() {
    assertThat(calculator.subtract(4, 1), is(equalTo(3)));
}
```

## @Disabled

可以用在 class 和 method 上，强烈建议提供原因描述。

```java
@Disabled("Disabled until bug #99 has been fixed")
public class DisableDemo {

  @Disabled("Disabled until bug #42 has been resolved")
  @Test
  void testWillBeSkipped() {
  }

}
```

## 条件执行

支持包括操作系统类型、Java 运行时版本、系统变量、环境变量、脚本（实验阶段）等的条件执行，具体见示例：

```java
@Test
@EnabledOnOs({LINUX, MAC})
void onLinuxOrMac() {
  // ...
}

@Test
@DisabledOnJre({JAVA_9, JAVA_10})
void onJava9Or10() {
  // ...
}

@Test
@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
void onlyOn64BitArchitectures() {
  // ...
}

@Test
@EnabledIfEnvironmentVariable(named = "ENV", matches = "staging-server")
void onlyOnStagingServer() {
  // ...
}

@Test // Static JavaScript expression.
@EnabledIf("2 * 3 == 6")
void willBeExecuted() {
  // ...
}

@RepeatedTest(10) // Dynamic JavaScript expression.
@DisabledIf("Math.random() < 0.314159")
void mightNotBeExecuted() {
  // ...
}
```

## 测试实例生命周期

为了保证测试方法之间相互隔离，默认情况下，对每个测试方法，都会创建一个测试类的实例，即`Lifecycle.PER_METHOD`。如果我们希望所有测试方法共享同一个类实例，在测试类上添加注解：`@TestInstance(Lifecycle.PER_CLASS)`，这种情况下，所有测试方法共享测试类的实例数据。

```java
@TestMethodOrder(OrderAnnotation.class)
@TestInstance(Lifecycle.PER_CLASS)
class OrderedTestsDemo {
  private int count = 1;

  @Test
  @Order(4)
  void nullValues() {
    assertEquals(4, ++count);
  }

  @Test
  @Order(2)
  void emptyValues() {
    assertEquals(2, ++count);
  }

  @Test
  @Order(3)
  void validValues() {
    assertEquals(3, ++count);
  }

}
```

## @RepeatedTest

将测试方法重复执行多次。

```java
@RepeatedTest(10)
void repeatedTest() {
// ...
}
```

## @ParameterizedTest

使用不同的参数，重复执行测试，每一个参数的执行结果都会单独给出。需要提供一个数据源，Junit 5 提供的有：`@ValueSource`, `@NullAndEmptySource`, `@NullSource`, `@EmptySource`, `@EnumSource`, `@MethodSource`, `@CsvSource`, `@CsvFileSource`, `@ArgumentsSource`。该特性目前处于试验阶段。

```java
@ParameterizedTest
@NullSource
@EmptySource
@ValueSource(strings = {"racecar", "radar", "able was I ere I saw elba",})
void palindromes(String candidate) {
  assertTrue(StringUtils.isNotBlank(candidate));
}
```

## @TempDir

默认提供一个临时目录用于测试。`@TempDir`可以用在字段上，也可以用在方法参数上，变量类型必须是`java.nio.file.Path` 或者 `java.io.File`，用在字段上，可以所有测试共享，用在测试方法参数上，由测试方法独立使用。

```java
@Test
  void writeItemsToFile(@TempDir Path tempDir) throws IOException {
    Path file = tempDir.resolve("test.txt");

    Files.write(file, singletonList("hello"));

    assertEquals(singletonList("hello"), Files.readAllLines(file));
  }


  @TempDir
  static Path sharedTempDir;

  @Test
  void writeItemsToFile() throws IOException {
    Path file = sharedTempDir.resolve("tmp.txt");

    Files.write(file, singletonList("hello"));

    assertEquals(singletonList("hello"), Files.readAllLines(file));
  }

  @Test
  void writeOtherItemsToFile() throws IOException {
    Path file = sharedTempDir.resolve("tmp.txt");

    Files.write(file, singletonList("world"));

    assertEquals(Lists.newArrayList("world"), Files.readAllLines(file));
  }
```

- 参考：[JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)


