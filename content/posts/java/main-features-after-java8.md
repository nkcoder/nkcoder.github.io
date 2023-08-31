---
title: "Java refresh: main features after Java8"
date: 2023-08-31T21:55:36+10:00
draft: false
---

Just take some time to refresh my Java knowledge, especially the new features after Java 8. 

## 1. JEP 413: Code Snippets in Java API Documentation (@18)

To replace the use of `<pre>{@code ...}</pre>` blocks.

```java
/**
 * JEP 413: Code Snippets in Java API Documentation
 * The following code shows how to use {@code Optional.isPresent}:
 * {@snippet :
 * if (v.isPresent()) {
 *     System.out.println("v: " + v.get());
 * }
 * }
 */
```

## 2. JEP 400: UTF-8 by Default (@18)

Specify UTF-8 as the default charset of the standard Java APIs. With this change, APIs that depend upon the default charset will behave consistently across all implementations, operating systems, locales, and configurations.

If a charset argument is not passed, then standard Java APIs typically use the default charset. The JDK chooses the default charset at startup based upon the run-time environment: the operating system, the user's locale, and other factors.

Because the default charset is not the same everywhere, APIs that use the default charset pose many non-obvious hazards, even to experienced developers.

## 3. JEP 409: Sealed Classes (@17)

A sealed class or interface can be extended or implemented only by those classes and interfaces permitted to do so.

A sealed class imposes three constraints on its permitted subclasses:

- The sealed class and its permitted subclasses must belong to the same module, and, if declared in an unnamed module, to the same package.
- Every permitted subclass must directly extend the sealed class.
- Every permitted subclass must use a modifier:
  - final: prevent from being extended further
  - sealed: allow to be extended further
  - non-sealed: being open for extension by unknown subclasses

```java
public abstract sealed class Shape
        permits Circle, Rectangle, Square {
}

public final class Circle extends Shape {
}

public sealed class Rectangle extends Shape
    permits FilledRectangle, TransparentRectangle{
}

public final class Square extends Shape {
}

public final class FilledRectangle extends Rectangle {
}

public final class TransparentRectangle extends Rectangle {
}
```

## 4. JEP 395: Records (@16)

Record classes are a new kind of class in the Java language. Record classes help to model plain data aggregates with less ceremony than normal classes.

Rules for `Record` classes:

- A record class cannot explicitly extend any class
- A record class is implicitly final, and cannot be abstract.
- The fields derived from the record components are final.
- A record class cannot explicitly declare instance fields, and cannot contain instance initializers.

```java
record Point(int x, int y) {
    public int square() {
        return x * y;
    }
}

record Range(int lo, int hi) {
    Range {
        if (lo > hi)
            throw new IllegalArgumentException(String.format("(%d,%d)", lo, hi));
    }
}
```

## 5. JEP 394: Pattern Matching for instanceof (@16)

If obj is an instance of String, then it is cast to String and the value is assigned to the variable s.

```java
if (obj instanceof String s) {
    // Let pattern matching do the work!
    ...
}
```

A pattern variable is only in scope where the compiler can deduce that the pattern has definitely matched and the variable will have been assigned a value.

```java
if (a instanceof Point p) {
    // p is in scope
    ...
}
// p not in scope here
if (b instanceof Point p) {     // Sure!
        ...
}
```

The use of pattern matching in `instanceof` should significantly reduce the overall number of explicit casts in Java programs.

```java
public final boolean equals(Object o) {
    return (o instanceof CaseInsensitiveString cis) &&
        cis.s.equalsIgnoreCase(s);
}
```

## 6. JEP 378: Text Blocks (@15)

A text block is a multi-line string literal that avoids the need for most escape sequences, automatically formats the string in a predictable way, and gives the developer control over the format when desired.

The opening delimiter is a sequence of three double quote characters (""") followed by zero or more white spaces followed by a line terminator. The content begins at the first character after the line terminator of the opening delimiter.

The closing delimiter is a sequence of three double quote characters. The content ends at the last character before the first double quote of the closing delimiter.

```java
String html = """
        <html>
            <body>
                <p>Hello, world</p>
            </body>
        </html>
        """;
System.out.println(html);

String query = """
        SELECT "EMP_ID", "LAST_NAME" FROM "EMPLOYEE_TB"
        WHERE "CITY" = 'INDIANAPOLIS'
        ORDER B
        """;
System.out.println(query);
```

## 7. New String methods

- `formatted()`, `stripIndent`, `translateEscape()`: @15

```java
 // formatted(): equivalent to String.format()
String formatted = "1. %s 2. %s 3. %s ".formatted("one", "two", "three");
System.out.println(formatted);

// stripIndent(): remove incidental white spaces from the beginning and end of every line
String stripIndent = " Good day!   ".stripIndent();
String html = """
        <html>
            <body>
                <p>Hello, world</p>
            </body>
        </html>
            """.stripIndent();
System.out.println(stripIndent);
System.out.println(html);

String str = "This is tab \t, Next New Line \n,next backspace \b,next Single Quotes \' next,Double Quotes \" ";
System.out.println(str.translateEscapes());
```

- `indent()` and `transform()`: @12

```java
// indent(n): insert n spaces before each line after `lines()`
String multilineStr = "This is\na multiline\nstring.";
System.out.println(multilineStr.indent(5));

// transform: apply a single argument function to the string
String concat = "hello".transform(p -> p + " world");
Integer parsedInt = "45".transform(Integer::parseInt);
System.out.println(concat);
System.out.println(parsedInt);
```

## 8. JEP 361: Switch Expressions @14

In addition to traditional "case L :" labels in a switch block, we define a new simplified form, with "case L ->" labels. If a label is matched, then only the expression or statement to the right of the arrow is executed; there is no fall through.

```java
switch (n) {
    case 1 -> System.out.println("one");
    case 2 -> System.out.println("two");
    default -> System.out.println("many");
}

String value = switch (n) {
    case 1 -> "one";
    case 2 -> "two";
    default -> "many";
};
```

In the event that a full block is needed, we introduce a new yield statement to yield a value, which becomes the value of the enclosing switch expression.

```java
int j = switch (n) {
    case 0, 1 -> 0;
    case 2, 3, 4 -> 1;
    default -> {
        int result = new Random().nextInt();
        yield result;
    }
};
```

## 9. JEP 286: Local-Variable Type Inference @10

For local variable declarations with initializers, enhanced for-loop indexes, and index variables declared in traditional for loops, allow the reserved type name `var` to be accepted in place of manifest types:

It would not be available for method formals, constructor formals, method return types, fields, catch formals, or any other kind of variable declaration.

```java
var list = Arrays.asList("A", "B", "C");   // infers ArrayList<String>
var stream = list.stream();     // infers Stream<String>
```

## 10. JEP 269: Convenience Factory Methods for Collections @9

Provide static factory methods on the List, Set, and Map interfaces for creating unmodifiable instances of those collections.

```java
var set1 = Set.of(1, 2, 3, 3, 5, 2, 10);
var list1 = List.of("A", "Grade", "Morning", "A");

var set1 = Set.of(1, 2, 3, 3, 5, 2, 10);
var list1 = List.of("A", "Grade", "Morning", "A");
```

## 11. Interface private methods @9

From Java 9, private methods can be added to interfaces in Java.

```java
interface Animal {
    default void bark() {
        think();
        System.out.println("hello");
    }

    private void think() {
        System.out.println("thinking...");
    }
}
```

## 12. New Optional Methods @9

```java
// ifPresentOrElse()
Optional<String> nonEmpty = Optional.of("Hello World");
nonEmpty.ifPresentOrElse(System.out::println, () -> System.out.println("Empty"));

// or()
Optional<String> empty = Optional.empty();
nonEmpty.or(() -> Optional.of("Default")).ifPresent(System.out::println);
empty.or(() -> Optional.of("Default")).ifPresent(System.out::println);

// stream()
var fruits = List.of("Apple", "Orange", "Pear", "Banana");
fruits.stream().map(NewOptionalMethods::transform)
        .flatMap(Optional::stream)
        .forEach(System.out::println);

// isEmpty()
System.out.println(nonEmpty.isEmpty());
System.out.println(empty.isEmpty());

// orElseThrow()
nonEmpty.orElseThrow();
empty.orElseThrow();
```

---

> The playground source code of this blog can be found [here](https://github.com/nkcoder/main-features-after-java8).
