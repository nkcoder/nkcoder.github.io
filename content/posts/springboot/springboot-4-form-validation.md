---
title: SpringBoot系列四：表单校验
date: 2020-02-26T11:34:37+08:00
categories:
- SpringBoot
draft: true
---

在提交表单时，不仅前端需要对字段进行校验和提示，后端也需要做同样的工作。Spring 支持 Java 的 Bean Validation API（即 JSR-303），而且 Spring Boot 默认提供了 Validation API 及对应的 Hibernate 实现，只需要在 bean 的需要校验的字段上添加对应的校验注解即可。如下示例：

```java
@Getter
@RequiredArgsConstructor
public class Employee implements Serializable {
  private static final long serialVersionUID = -8224860450904540019L;

  @NotEmpty(message = "name is required")
  @UTF8Size(max = 16, message = "name should be short than 128")
  private final String name;

  @NotBlank(message = "city is required")
  @Size(max = 128, message = "city should be short than 128")
  private final String city;

  @CreditCardNumber(message = "invalid credit card number")
  private final String ccNumber;

  @Pattern(regexp = "^(0[1-9]|1[0-2])([\\\\/])([1-9][0-9])$", message = "required format MM/YY")
  private final String ccExpiration;

  @Digits(integer = 3, fraction = 0, message = "invalid CVV")
  private final String ccCVV;
}
```

经常使用的注解及简要说明如下：

| 注解              | 说明                                                                                                 |
| :---------------- | :--------------------------------------------------------------------------------------------------- |
| @NotEmpty         | **字符串、集合、Map、数组**等不能为 null 或空                                                        |
| @NotBlank         | **字符串**不能为 null，且至少包含一个非空字符                                                        |
| @NotNull          | **任意类型** 不能为 null                                                                             |
| @Size             | **字符串、集合、Map、数组**等元素的个数必须在指定的 min 和 max 范围内                                |
| @Email            | **字符串**是有效的邮箱                                                                               |
| @Digits           | **字符串、整数、浮点数**是一个数字，参数 integer 表示整数位的数字个数，fraction 表示小数位的数字个数 |
| @CreditCardNumber | **字符串**是有效的信用卡数字，不校验信用卡本身的有效性                                               |
| @AssertTrue       | **布尔类型**必须是 true，null 值被认为是有效的                                                       |
| @Max              | **整数、浮点数**必须小于等于指定的最大值                                                             |
| @Min              | **整数、浮点数**必须大于等于指定的最小值                                                             |
| @Range            | **字符串、数字**必须在指定的 min 和 max 范围内                                                       |
| @Pattern          | **字符串**必须匹配指定的正则表达式                                                                   |

> 除了`@NotEmpty`和`@NotBlank`将 null 值认为是非法的之外，其它注解如`@Size`, `@Max`, `@Min`等都将 null 认为是有效的，如果不允许 null 值，则需要额外添加`@NotNull`注解。

如果现有的注解不能满足需要，也可以自定义注解及校验规则。比如`@Size`并不能支持中文字符，我们可以自定义实现如下：

```java
@Documented
@Constraint(validatedBy = Utf8SizeValidator.class)
@Target({METHOD, FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface UTF8Size {
  String message() default validator;

  int min() default 0;
  int max() default Integer.MAX_VALUE;

  Class<?>[] groups() default {};

  Class<? extends Payload>[] payload() default {};

}
```

```java
public class Utf8SizeValidator implements ConstraintValidator<UTF8Size, String> {

  private int maxCharSize;

  @Override
  public void initialize(UTF8Size constraintAnnotation) {
    this.maxCharSize = constraintAnnotation.max();
  }

  @Override
  public boolean isValid(String value, ConstraintValidatorContext context) {
    if (Objects.isNull(value)) {
      return true;
    }
    return value.getBytes(Charset.forName("GB18030")).length <= maxCharSize;
  }
}
```

然后在需要校验字符（中英文）的字段上使用`@UTF8Size`即可。

接下来在 Bean 使用的地方，需要使用 Bean Validation API 的`@Valid`注解，或者 Spring Context 提供的`@Validated`注解启用对 Bean 的校验。主要区别是，`@Validated`是`@Valid`的变体，支持分组校验（validation groups）。

```java
  @PostMapping("")
  public String addEmploy(@RequestBody  @Valid Employee employee) {
    log.info("employee to create: {}", employee);

    String employeeId = UUID.randomUUID().toString();
    return employeeId;
  }
```

如果有字段校验失败，则会抛出`MethodArgumentNotValidException`，我们可以通过`@ExceptionHandler`将异常进行统一处理，如：

```java
@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {
  @ExceptionHandler(MethodArgumentNotValidException.class)
  @ResponseBody
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  public ResponseContent handleMethodArgumentNotValidException(
      MethodArgumentNotValidException exception) {
    log.error("method argument not valid: {}", exception.getMessage());
    String errorMessage = "field error";

    BindingResult bindingResult = exception.getBindingResult();
    if (bindingResult.hasErrors()) {
      errorMessage = bindingResult.getFieldErrors().stream()
          .map(FieldError::getDefaultMessage)
          .collect(Collectors.joining(" | "));
    }

    return ResponseContent.builder()
        .code("INVALID_ARGUMENT")
        .message(errorMessage)
        .build();
  }
}
```

> 完整的示例代码见项目：[github-springboot-form-validation](https://github.com/nkcoder/springboot-samples/tree/master/springboot-form-validation).

### 参考

- [Spring in Action, Fifth Edition](https://www.manning.com/books/spring-in-action-fifth-edition)
- [Difference between @Valid and @Validated in Spring](https://stackoverflow.com/questions/36173332/difference-between-valid-and-validated-in-spring)