---
title: "The difference between call-by-value and call-by-name"
date: 2022-06-19T13:00:20+08:00
categories:
- Scala
draft: false
---

We have two simple functions:
```scala
def calledByValue(x: Long): Unit = {
	println("call by value: " + x)
	println("call by value: " + x)
}

def calledByName(x: => Long): Unit = {
	println("call by name: " + x)
	println("call by name: " + x)
}
```

Let's call the two functions with the same parameter:
```scala
calledByValue(System.nanoTime())
calledByName(System.nanoTime())
```

The output is:
```
call by value: 329019404609381
call by value: 329019404609381
call by name: 329019470999309
call by name: 329019472001451
```

We can see that the outputs in the first function are the same, but the outputs in the second function are different. 

When we call the first function (call by value), the expression `System.nanoTime()` is evaluated first and then passed to the body of the function, so it's similar to:
```scala
def calledByValue(329019404609381): Unit = {
	println("call by value: " + 329019404609381)
	println("call by value: " + 329019404609381)
}
```

But when we call the second function (call by name), the express `System.nanoTime()` is passed to the body of the function and the execution is delayed to where it is called, so it's similar to:
```scala
def calledByName(System.nanoTime()): Unit = {
	println("call by name: " + System.nanoTime())
	println("call by name: " + System.nanoTime())
}
```

So the difference makes sense now.

> Learnt from: [Scala & Functional Programming Essentials | Rock the JVM](https://www.udemy.com/course/rock-the-jvm-scala-for-beginners/)
