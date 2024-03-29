---
title: "Scala: Functional Objects"
date: 2021-07-31T18:49:25+08:00
categories:
- Scala
draft: false
---

## Constructing an object

```scala
class Rational(n: Int, d: Int) {
	println("Created " + n + "/" + d)
}
```

- If your class doesn't have a body, you don't need to specify empty curly braces.
- The identifiers `n` and `d` are called **class parameters,** the Scala compiler will create a **primary constructor** that takes these two parameters.
- The Scala compiler will compile any code you place in the class body, which isn't part of a field or a method definition, into the primary constructor.

## Adding fields

```scala
class Rational(n: Int, d: Int) {
  private val numer: Int = n
  private val denom: Int = d

  def add(value: Int): Int = n + d + value

  def anotherAdd(value: Int): Int = this.n + this.d + value

  def add(that: Rational): Rational = {
    new Rational(n * that.d + d * that.n, d * that.d) // value d is not a member of Rational
  }

	def anotherAdd(that: Rational): Rational = {
    new Rational(numer * that.denom + denom * that.numer, denom * that.denom)
	}
}
```

- Class parameters are visible in the scope of the class, which means they can be used in class body, in methods and constructors.
- Although `n` and `d` are in the scope of `add()`, but you can only access them on the object on which `add()` was invoked.
- `numer` and `denom` are fields, so you can access them on objects.
- To make a field or method private you simply place the **private** keyword in front of the definition.

## Auxiliary constructors

```scala
class Rational(n: Int, d: Int) {
  def this(n: Int) = this(n, 1)
}
```

- In Scala, constructors other than the primary constructor are called **auxiliary constructors**.
- Every auxiliary constructor must invoke another constructor of the same class as its first action. Every constructor invocation will end up eventually calling the primary constructor.

## Defining operators

```scala
class Rational(n: Int, d: Int) {
  val numer: Int = n
  val denom: Int = d

	def + (that: Rational): Rational = new Rational(numer * that.denom + denom * that.numer, denom * that.denom)

	def * (that: Rational): Rational = new Rational(numer * that.numer, denom * that.denom)
}
```

```scala
scala> val r = new Rational(1, 5)
val r: Rational = 1/5

scala> val t = new Rational(2, 5)
val t: Rational = 2/5

scala> r + t
val res1: Rational = 15/25

scala> r * t
val res2: Rational = 2/25

scala> r.+(t)
val res3: Rational = 15/25

scala> r + r * t
val res4: Rational = 35/125
```

- The operator syntax is equivalent to the method call.
- Given Scala's rules for operator precedence, the `*` method will bind more tightly than the `+` method, which means `x + x * y` will execute as `x + (x * y)`.

## Identifiers

- `$` is reserved for identifiers generated by the Scala compiler.** Identifiers in user programs should not contain `$` characters since it might lead to name clashes with identifiers generated by the Scala compiler.
- Although underscores(`_`) are legal in identifiers, they are not used that often in Scala programs. **Because underscores have many other non-identifier uses in Scala code.** As a result, it is best to avoid identifiers like `to_string`, `__init__` or `name_`.
- constant *versus* val:
    - In Scala, the word *constant* doesn't just mean `val`.
    - **Even though a `val` does remain constant after it's initialized, it is still a variable.** For example, method parameters are *vals*, but each time the method is called, those vals can hold different values.
    - A constant is more permanent. For example, `scala.math.PI` is defined to be the double value closet to the real value of π. This value is unlikely to change ever, thus **PI** is clearly a constant.
    - Constant use cases:
        - Give names to values that would otherwise be *magic numbers* in your code: literal values with no explanation.
        - Define constants for use in pattern matching
    - **In Scala, naming convention for constants is merely that the first character should be upper case.** For example: `Preset_offset_years`.
- The Scala compiler will internally mangle operator identifiers to turn them into legal Java identifiers with embedded *$* characters. For instance, the operator identifier `:->` would be represented internally as `$colon$minus$greater`. If you ever wanted to access this identifier from Java code, you'd need to use the internal representation.
- You can put any string that's accepted by the runtime as an identifier between back ticks. The result is always a Scala identifier. A typical use case is accessing the static `yield` method in Java's `Thread` class. You cannot write `Thread.yield()` because `yield` is a reserved work in Scala.  However you can still name the method in back ticks: 
    ```scala
    Thread.`yield()`
    ```

## Immutable object trade-offs

Immutable objects offer several advantages over mutable objects, and one potential disadvantage. 

- Immutable objects are often easier to reason about because **they don't have complex state spaces that change over time.**
- **You can pass immutable objects around quite freely**, whereas you may need to make defensive copies of mutable objects before passing them to other code.
- There is no way for two threads concurrently accessing an immutable to corrupt its state once it has been properly constructed.
- Immutable objects make safe hash table keys.
- The main disadvantage of immutable objects is that they sometimes require that a large object graph be copied, whereas an update could be done in its place.

> This is my Scala notes as a Developer with heavy Java experience.《Programming in Scala》(Fourth Edition)(2.13): Martin Odersky | Lex Spoon | Bill Venner