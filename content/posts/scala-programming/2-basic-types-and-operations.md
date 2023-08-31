---
title: 'Scala: Basic Types and Operations'
date: 2021-07-18T17:06:23+08:00
categories:
- Scala
draft: false 
---

## Basic types

Basic types of Scala：

```scala
Byte, Short, Int, Long, Char, String, Float, Double, Boolean
```

```scala
scala> val aFloat = 1.2345F
val aFloat: Float = 1.2345

scala> val aDouble = 3E5
val aDouble: Double = 300000.0

scala> val anotherDouble = 1.2345D
val anotherDouble: Double = 1.2345
```

A string literal is a composed of characters surrounded by double quotes. 

A **raw** string starts and ends with three double quotation marks in a row(`"""`). The interior of a raw string may contain any characters including newlines, quotation marks and special characters.

```scala
scala> val aString = "hello, \n \\ world"
val aString: String =
hello,
 \ world

scala> val aRawString = """hello, \n \\ world"""
val aRawString: String = hello, \n \\ world
```

You can put a pipe character (`|`) at the front of each line and call `stripMargin` on the raw string to remove the leading spaces:

```scala
scala> val welcome = """|Welcome to BigSur.
     | Open `About This Mac` for more information.""".stripMargin
val welcome: String =
Welcome to BigSur.
Open `About This Mac` for more informatio
```

## String interpolation

**String interpolation** allows you to embed expressions within string literals.

For single-variable expressions, you can just place the variable name after the dollar sign. If the expression includes non-identifier characters, you must place it in curly braces.

```scala
scala> val name = "reader"
val name: String = reader

scala> s"Hello, $name!"
val res0: String = Hello, reader!

scala> s"The answer is ${6 * 7}"
val res2: String = The answer is 42
```

Scala provides two other string interpolators by default: `raw` and `f`. 

- The `raw` string interpolator behaves like s, except it does not recognize character literal escape sequences.
- The `f` string interpolator allows you to attach printf-style formatting instructions to embedded expressions. If you provide no formatting instructions for an embedded expression, the `f` string interpolator will default to %s, which behaves just like `s` string interpolator.

```scala
scala> raw"No\\\\\Escape."
val res3: String = No\\\\\Escape.

scala> f"PI is approximately ${math.Pi}%.8f"
val res5: String = PI is approximately 3.14159265
```

## Operators and methods

In Scala, operators are just a nice syntax for ordinary method calls.  You can use any method in  operator notation.

```scala
scala> val sum1 = 1 + 2
val sum1: Int = 3

scala> val sum2 = 1.+(2)
val sum2: Int = 3

scala> val s = "hello, world"
val s: String = hello, world

scala> val index1 = s indexOf 'o'
val index1: Int = 4

scala> val index2 = s indexOf ('o', 6)
val index2: Int = 8
```

- *infix* operator notion: takes two operands, invoke the object with the two parameters
- *prefix* operator notion: the only one operand is to the right of the operator, and it's a shorthand way of invoking the method: `unary_`.The only identifiers that can be used as prefix operators are: +, -, !, ~.
- *postfix* operator notion: are method calls that take no arguments when they are invoked without a dot or parentheses. In Scala, the convention is that you include parentheses if the method has side effects, such as `println()`, otherwise you can leave off them

```scala
scala> val x1 = -7
val x1: Int = -7

scala> val x2 = 7.unary_-
val x2: Int = -7
```

```scala
scala> val y1 = 7 toLong
val y1: Long = 7

scala> val y2 = 7.toLong
val y2: Long = 7
```

## Object equality

In Scala, you can use `==` or `!=` to compare two objects for equality. 

- The operations actually apply to all objects, not just basic types.
- You can compare two objects that have different types.
- You can even compare against **null**, or against things that might be **null**.

```scala
scala> 1 == 2
val res0: Boolean = false

scala> 1 != 2
val res1: Boolean = true

scala> List(1, 2, 3) == List(1, 2, 3)
val res2: Boolean = true

scala> List(1, 2, 3) == "hello"
val res3: Boolean = false

scala> null == List(1, 2, 3)
val res4: Boolean = false

scala> ("he" + "llo") == "hello"
val res5: Boolean = true
```

The equality comparison is accomplished with a very simple rule: First check the left side for **null**. if it is not **null**, call the `equals` method.

This kind of comparison will yield true on different objects, as long as their contents are the same and their `equals` method is written to be based on contents.

## Operator associativity

The associativity of an operator in Scala is determined by it's last character:

- any methods that ends in a `:` character is invoked on it's right operand, passing in the left operand
- methods that ends in any other character are invoked on their left operand, passing in the right operand.

So `a * b` yields `a.*(b)`, but `a ::: b` yields `b.:::(a)`. 

No matter what associativity an operator has, however, its operands are always evaluated left to right. So `a ::: b` is more precisely treated as: `{ val x = a; b.:::(x) }`

## Rich wrappers

For each basic types, there is a **rich wrapper** that provides several more methods, which are available via implicit conversions.

- `0 max 5` is invoked as `intwrapper(0) max 5`
- `-2.7 round` is invoked as `doublewrapper(-2.7) round`
- `4 to 6` is invoked as `intwrapper(4) to 6`
- `"bob" capitalize` is invoked as `augmentString("bob") capitalize`

> This is my Scala notes as a Developer with heavy Java experience.《Programming in Scala》(Fourth Edition)(2.13): Martin Odersky | Lex Spoon | Bill Venner