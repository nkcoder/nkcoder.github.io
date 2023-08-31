---
title: "Scala: Build-in Control Structures"
date: 2021-08-01T08:20:47+08:00
categories:
- Scala
draft: false
---
## If expressions

```scala
val fileName = if (!args.isEmpty) args(0) else "default.txt"
```

- Scala's `if` is an expression that results in a value.
- The first advantage is that we use a *val* instead of *var.* **Using a *val* is the functional style and it tells readers of the code that the variable will never change**, saving them from scanning all code in the variable's scope to see if it ever changes.
- **The second advantage to using a *val* instead of *var* is it better supports equational reasoning.** The introduced variable is equal to the expression that computes it, assuming that expression has no side effects. Thus any time you want to write the variable name, you cloud instead write the expression.

> Look for opportunities to use *vals*. They can make your code both easier to read and easier to refactor.

## While loops

```scala
def gcdLoop(x: Long, y: Long): Long = {
    var a = x
    var b = y

    while (a != 0) {
      val temp = a
      a = a % b
      b = temp
    }

    b
  }
```

- Scala also has a *do-while* loop. **They are loops, not expressions, because they don't result in an interesting value.**
- The return value is `Unit` and is written `()`. Reassignment to *vars* also results in `Unit` value.
- Because the *while* loop results in no value, it is often left out of pure functional languages. Scala includes the while loop nonetheless because sometimes an imperative solution can be more readable.
- **In general, we recommend you challenge *while* loops in your code in the same way you challenge *vars*.** If there isn't a good justification for a particular *while* or *do-while* loop, try to find a way to do the same thing without it.

## For expressions

```scala
val filesHere = new File(".").listFiles
for (file <- filesHere) println(file)

for (i <- 1 to 4) println(s"iteration: $i")
for (i <- 1 until 4) println(s"iteration: $i"
```

- **The `for` expression syntax works for any kind of collections**: Array, List, Range etc., any collection that has a `foreach` method with appropriate signatures.

```scala
for (file <- filesHere if file.getName().endsWith(".scala")) println(file)
for (
  file <- filesHere
  if file.isFile
  if file.getName().endsWith(".scala")
) println(file)
```

- **You can add a filter by adding an `if` clause inside the `for`'s parentheses.**
- You can add more filters, just keep adding `if` clauses

```scala
for (
	file <- filesHere if file.getName.endsWith(".scala");
	line <- fileLines(file) if line.trim().matches("intro")
) println(s"file: $file, line: $line")

for {
	file <- filesHere if file.getName.endsWith(".scala")
	line <- fileLines(file) if line.trim.matches("intro")
} println(s"file: $file, line: ${line.trim}")
```

- **If you add multiple `<-` clauses, you will get nested *loops*.**
- **If you prefer, you can use curly braces instead of parentheses to surround the generators and filters.** One advantage to use curly braces is that you can leave off the semicolons since the Scala compile will not infer semicolons while inside parentheses.

```scala
for {
	file <- filesHere if file.getName.endsWith(".scala")
	line <- fileLines(file) 
	trimmed = line.trim()
	if trimmed.matches("intro")
} println(s"file: $file, line: $trimmed"
```

- **You can bind the result to a new variable using an equals sign(`=`)**. The bound variable is introduced and used just like a *val*, only with the *val* keyword left out.

```scala
for {
	file <- filesHere if file.getName().endsWith(".scala")
	line <- fileLines(file)
	trimmed = line.trim
	if trimmed.matches(".*for*.")
} yield trimmed.length()
```

- **The `for` expression can generate a value for each iteration.** Each time the body of the *for* expression executes, it produces one value. When the *for* expression completes, all yielded values are contained in a single collection.
- The type of the resulting collection is based on the kind of collections processed in the iteration clauses.
- **The syntax is: `for clauses yield body`**. **The *yield* goes before the entire body.** Even if the body is a block surrounded by curly braces, put the yield before the first curly brace, not before the last expression of the block.

## Exception handling

```scala
def half(n: Int): Int = {
  if (n % 2 == 0) n / 2 else throw new RuntimeException("n must be even.")
}

val h = try {
  half(11)
} catch {
  case ex: RuntimeException => 0
  case ex: ArithmeticException => -1
} finally {
	//
}
```

- **In Scala, *throw* is an expression that has a result type `Nothing`**, which is at the bottom of Scala's type hierarchy, which means it's a subtype of any other types.
- **Unlike Java, Scala doesn't  require you to catch checked exceptions or declare them in a *throws* clause.** You can declare a *throws* clause if you wish with the *@throws* annotation, but it is no required.
- In Scala, *try-catch-finally* results in a value.

> This is my Scala notes as a Developer with heavy Java experience.《Programming in Scala》(Fourth Edition)(2.13): Martin Odersky | Lex Spoon | Bill Venner