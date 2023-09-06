---
title: "VAVR: brings functional programming type to Java"
date: 2023-09-07T07:26:15+10:00
draft: false
---

Java, while powerful, often lacks the functional programming constructs found in languages like Haskell and Scala. Enter [VAVR](hat.openai.com), a library designed to bridge this gap, granting Java developers access to an enriched functional toolkit.

## What is VAVR?

VAVR, formerly known as Javaslang, is a functional library for Java 8+ that enriches the language with immutable data types and functional control structures. It aims to reduce the lines of code, enhance code quality, and improve the overall developer experience.

## Why Choose VAVR?

With its enhanced functional constructs like immutable collections, functional control structures, and pattern matching, VAVR aims to bring Java closer to true functional languages, increasing code safety and reducing boilerplate.

## Functional programming concepts

**Side effects**: common side effects are changing objects or variables in place, printing to the console, writing to a log file or to a database. Side-effects are considered harmful if they affect the semantics of our program in an undesirable way.

**Referential transparent**: a function, or more generally an expression, is called referentially transparent if a call can be replaced by its value without affecting the behavior of the program. Simply spoken, given the same input the output is always the same.

**Pure function**: a function is called pure if all expressions involved are referentially transparent. Easy to reason about, test and debug. The key to a better Java is to use immutable values paired with referentially transparent functions.

**Partial function**: A partial function from X to Y is a function f: X′ → Y, for some subset X′ of X. It generalizes the concept of a function f: X → Y by not forcing f to map every element of X to an element of Y. That means a partial function works properly only for some input values. If the function is called with a disallowed input value, it will typically throw an exception.

## Key Components of VAVR

## Tuple

Tuples are immutable and can hold multiple objects of different types. They're great for returning multiple values from a function without the need to create a custom class.

```java
// create tuples, you have Tuple1...Tuple8
Tuple2<String, Integer> java17 = Tuple.of("Java", 17);

// map each element with a mapper
Tuple2<String, String> java17 = Tuple.of("Java", 17).map(
        s -> s + "!",
        String::valueOf
);

// map all elements with one mapper
Tuple2<String, String> java17Again = Tuple.of("Java", 17).map(
        (s, i) -> Tuple.of(s + "!", String.valueOf(i))
);

// transform the tuple to a new type
String java17 = Tuple.of("Java", 17).apply(
        (s, i) -> s + i + "!"
);
```

## Functions

Functional programming is all about values and transformation of values using functions. VAVR provides first-class functions, enabling higher-order functions, currying, memoization, and more.

Functional interfaces are from `Function0`, `Function1` to `Function 8`, and functional interfaces that throw a checked exception are from `CheckedFunction0` to `CheckedFunction8`.

```java
// create functions
Function2<Integer, Integer, Integer> sum = Function2.of(Integer::sum);
Function3<Integer, Integer, Integer, Integer> sum3 = (x, y, z) -> x + y + z;

// composition: the application of one function to the result of another to produce a third function
// `andThen` or `compose`
Function1<Integer, Integer> plusOne = x -> x + 1;
Function1<Integer, Integer> multiply2 = x -> x * 2;
Function1<Integer, Integer> andThen = plusOne.andThen(multiply2);
Function1<Integer, Integer> compose = multiply2.compose(plusOne);

// lift: lift a partial function into a total function that returns an Option result.
Function2<Integer, Integer, Integer> divide = (x, y) -> x / y;
Function2<Integer, Integer, Option<Integer>> safeDivide = Function2.lift(divide);

// partial application: allows you to derive a new function from an existing one by fixing some values.
Function5<Integer, Integer, Integer, Integer, Integer, Integer> sum = (a, b, c, d, e) -> a + b + c + d + e;
Function2<Integer, Integer, Integer> sum6 = sum.apply(1, 2, 3);

// curring: partially apply a function by fixing a value for one of the parameters, resulting in a `Function1` function that returns a `Function1`.
Function3<Integer, Integer, Integer, Integer> sum = (a, b, c) -> a + b + c;
Function1<Integer, Function1<Integer, Integer>> add5 = sum.curried().apply(5);
```

## Values

This includes powerful constructs like Option, Try, Either, and Lazy:

- Option handles potential absence of a value better than null references.
- Try is a monadic container for a computation which may result in an exception.
- Either represents a value of two possible types.
- Lazy represents a value computed lazily.

```java
Option<String> some = Option.of("Hello");
Option<String> none = Option.of(null);

static Either<String, Integer> divide(Integer x, Integer y) {
    if (y == 0) {
        return Either.left("Error: divide by zero");
    } else {
        return Either.right(x / y);
    }
}

Try<Integer> byZeroWithRecover = Try.of(() -> divide(8, 0))
                .recover(x -> Integer.MAX_VALUE);

public static Validation<Seq<ValidationError>, Account> validateAccount(String name, Integer age, Double amount) {
    return Validation.combine(validateName(name), validateAge(age), validateAmount(amount)).ap(Account::new);
}

Number sum = List.of(1, 2, 3, 4, 5)
        .map(x -> x * 2)
        .distinct()
        .filter(x -> x > 5)
        .sum();
```

## Collections

VAVR's collections are immutable, ensuring thread-safety. It provides alternatives to Java's collections, like List, Set, Map, all being immutable.

```java
Number sum = List.of(1, 2, 3, 4, 5)
        .map(x -> x * 2)
        .distinct()
        .filter(x -> x > 5)
        .sum();
assertEquals(24, sum.intValue());

Stream<String> nums = Stream.from(1).map(String::valueOf);
assertFalse(nums.hasDefiniteSize());
```

## Property Checking and Pattern Matching

VAVR integrates property-based testing, allowing for a range of inputs to be tested against a property. Additionally, it supports pattern matching, akin to Scala, which enhances readability.

```java
Arbitrary<Integer> nums = Arbitrary.ofAll(Gen.choose(5, 10));
Property.def("x >= 5")
        .forAll(nums)
        .suchThat(x -> x >= 5)
        .check()
        .assertIsSatisfied();

List<Integer> nums = List.of(2, 4, 6, 8);
String result = Match(nums).of(
        Case($(isNull()), "Null"),
        Case($(forAll(x -> x % 2 == 0)), "All even"),
        Case($(), "Has Odd")
);
```

## Conclusion

VAVR is a stepping stone towards adopting functional programming in Java. With its array of features, VAVR makes Java programming more concise, expressive, and safe. For those looking to imbue their Java code with a functional flair, VAVR is a library worth exploring.


### References

- The [Vavr User Guide](https://docs.vavr.io/) is a great document of VAVR.
- The above examples can be found in [Github](https://github.com/nkcoder/fp-with-vavr)