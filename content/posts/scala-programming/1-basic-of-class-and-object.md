---
title: 'Scala: Basics of class and object'
date: 2021-07-18T08:22:29+08:00
categories: 
- Scala
draft: false
---

## Classes, fields, and methods

Once you define a class, you can create objects with the keyword `new`. 

`public` is Scala's default access level. 

```scala
class Accumulator {
  var sum = 0
}

val acc = new Accumulator
println(acc.sum)			// 0

acc.sum = 10
println(acc.sum)	// 10
```

One important way to pursue robustness of an object is to ensure that the object's state—the values of its instance variables—remains valid during its entire lifetime. The first step is to prevent outsiders from accessing the fields directly by making the fields `private`:

```scala
class Accumulator {
  private var sum = 0

  def add(a: Int): Int = a + sum
  def subtract(b: Int): Unit = sum -= b
}
```

- One important characteristics of method parameters in Scala is that they are **vals**, not **vars**.
- In the absence of any explicit **return** statement, a Scala method returns the last value computed by the method. The recommend style for methods is in fact to avoid having explicit and especially multiple **return** statements. Instead, think of each method as an expression that yields one value, which is returned.
- You can leave off the curly braces `{}` If a method computes only a single result expression. If the result expression is short, it can even be placed on the same line as the `def` itself.
- It is often better to explicitly provide the result types of public methods  even the compiler would infer it for you. Because readers of the code will also need to mentally infer the result types by studying the bodies of the methods.
- Methods with a result type `Unit` are executed for their side effects. A **side effect** is generally defined as mutating state somewhere external to the method or performing an I/O action.

---

In a Scala program, a semicolon(`;`) a the end of a statement is usually optional except for the following two cases:

- A semicolon is required if you write multiple statements on a single line:

    ```scala
    val s = "hello"; println(s)
    ```

- If you want to enter a statement that spans multiple lines, most of the time Scala will separate the statements in the correct place. Occasionally the split may against your wishes. You can wrap it in parentheses or put the operator at the end of a line:

    ```scala
    x 
    + y		// parsed as two statements x and +y

    (x
    + y)	// this is OK

    x +
    y			// this is also OK
    ```

## Singleton objects

Classes in Scala **cannot** have static members. Instead, Scala has singleton objects, which start with the keyword `object`.

When a singleton object shares the same name with a class, it is called that class's **companion object**. The class and its companion object **must** be defined in the same source file. A class and its companion object can access each other's **private** members.

```scala
class Accumulator {
	private var sum = 0

	import Accumulator.value
	def add(a: Int): Int = a + sum + value
}

object Accumulator {
	private val value = 0
  def getValue: Int = value
}

println(Accumulator.getValue)
```

Defining a singleton object doesn't define a type. However, singleton objects can extend superclass and mix in traits.

Singleton objects cannot take parameters because you can't instantiate a single object with the `new` keyword, you have no way to pass parameters to it. In particular, a singleton object is instantiated the first time some code accesses it.

A singleton object that does not share the same name with a companion class is called a **standalone object**. Standalone objects can be used for collecting related utility methods together or defining an entry point to a Scala application.

## A Scala application

Any standalone object with a main method of the proper signature can be used as the entry point into an application.

```scala
object MainDemo {
	def main(args: Array[String]): Unit = {
		println("Hello, world")
	}
}
```

```bash
$ scalac MainDemo.scala
$ scala MainDemo
```

Scala implicitly imports members of packages `java.lang` and `scala`, as well as the members of `Predef` into every Scala source file. `println` is defined in `Predef` package. 

In Scala, you can name `.scala` file anything you want, no matter what Scala classes or code you put in them. In general, in the case of non-script, it is recommended style to name files after the classes they contain, so that programmers can more easily locate classes by looking at file names. 

> This is my Scala notes as a Developer with heavy Java experience.《Programming in Scala》(Fourth Edition)(2.13): Martin Odersky | Lex Spoon | Bill Venners