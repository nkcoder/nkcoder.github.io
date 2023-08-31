---
title: Go的基本数据类型
date: 2021-04-18T22:36:16+08:00
categories:
- Go程序设计
draft: false
---

### 整数类型

**整数区分有符号和无符号**，且有四种大小：

```go
int8, 	int16, 	int32,	int64		// 有符号
uint8, 	uint16,	uint32,	uint64	// 无符号
```

其它类型说明：

- `int`和`uint`，它们的大小相同，32位或64位，与平台有关，不同的编译器即使在相同的硬件上得到的结果可能也不相同

- `rune`类型是`int32`的同义词，表示该值是一个Unicode码点(*code point*)
- `byte`是`uint8`的同义词，表示该值是原始数据(*raw data*)
- `uintptr`也是一个整型类型，但大小不确定，主要用于低级编程（如调用`C`语言库）。

> `int`, `uint`, `uintptr`与带大小的类型如`int32`, `uint64`等都是不同的类型，即使底层的类型相同，同时使用时需要做类型转换。

```go
r1, _ := utf8.DecodeRuneInString("世")
fmt.Printf("r1 = %c\n", r1)				// 世
```

---

**比较运算符**：

```go
== 	!=	<		<= 	>		>=
```

所有的基本类型，如布尔、数值、字符串都是可比较的，即同一类型的两个值，可以使用上述比较运算符直接比较。

```go
fmt.Printf("%t\n", 10 < 5)					// false
fmt.Printf("%t\n", 10 < 20.1)				// true
fmt.Printf("%t\n", "abc" >= "ab")		// true
```

---

**位运算符：**

```go
&		位AND，位的交集
|		位OR，位的并集
^		位XOR，用作二元运算符时表示异或，用作单目运算符时表示按位取反
&^	位清空(AND NOT), z=x &^ y, 如果y的位为1，则z对应的位为0，否则与x对应的位相同
<<	位左移
>>	位右移
```

```go
// bit operation
const b1 uint8 = 1 << 1 | 1 << 5	// set 1 and 5 bit
const b2 uint8 = 1 << 1 | 1 << 2	// set 1 and 2 bit

fmt.Printf("%08b\n", b1)		// 00100010
fmt.Printf("%08b\n", b2)		// 00000110

fmt.Printf("%08b\n", b1 & b2)		// 00000010
fmt.Printf("%08b\n", b1 | b2)		// 00100110
fmt.Printf("%08b\n", b1 ^ b2)		// 00100100
fmt.Printf("%08b\n", b1 &^ b2)	// 00100100
```

> - %08b：将结果以8位展示，不足8位的用0补齐。
> - 对于无符号数，左移右移都是补0，对于有符号数，左移补0，右移补符号位。所以整数在做位运算时，要注意使用无符号。

在Go语言中，对于不能为负值的整数，一般使用有符号数来表示，比如数组的长度，`len`函数返回的是`int`，如果返回的是`uint`，在与0比较时永远为true：

```go
data := []string{"hello", "world"}
for i := len(data) - 1; i >= 0; i-- {
  println(data[i])
}
```

---

**类型转换**

将一个值从一个类型转换成另一个类型，需要进行显示的类型转换。二元操作符的算术和逻辑运算（移位操作除外），要求两个操作数的类型完全一样。

`T(x)`表示将x转换成类型T。不同宽度类型的整数之间转换，或者整数与浮点数之间转换，可能会修改变量的值或丢失精度，比如浮点数转换成整数，小数部分会被丢弃。

```go
var n1 int16 = 10
var n2 int32 = 20
var sum int = int(n1) + int(n2)
fmt.Println(sum)		// 30

f1 := 3.1415
i1 := int(f1)
fmt.Println(i1)		// 3
```

### 浮点数

浮点数只有两种类型：`float32`和`float64`，其中`float32`的精度为6位，`float64`的精度为15位。因为`float32`类型的浮点数运算很容易出错，所以，在大多数情况下，应该优先使用`float64`类型进行浮点数运算。

`+Inf`和`-Inf`分别表示正无穷、负无穷，即除以0的结果，`NaN`表示不是一个数值类型，通常为*0/0*或对*Sqrt(-1)*的结果，任何涉及`NaN`的比较结果都是false：

```go
var z float64
fmt.Println(z, -z, 1/z, -1/z, z/z)	// 0 -0 +Inf -Inf NaN
fmt.Println(math.IsInf(-1/z, -1), math.IsInf(1/z, 1))	// true true
nan := math.NaN()
fmt.Println(nan == nan, nan < nan, nan > nan)		// false false	false
```

### 复数（complex）

复数由实数和虚数构成，有两种类型的复数，`complex64`和`complex128`，对应的组成部分（实数和虚数）的类型为`float32`和`float64`。

`complex`函数可以创建一个复数，`real`和`imag`函数获取对应的实数和虚数部分，`math/cmplx`包提供了对复数操作的函数：

```go
var x complex64 = complex(1, 2)
var y complex64 = complex(3, 4)
var z = x * y

fmt.Println(x, y, z)						// (1+2i) (3+4i) (-5+10i)
fmt.Println(real(z), imag(z))		// -5 10
fmt.Printf("%T %T\n", real(z), imag(z))		// float32 float32

var r = cmplx.Sqrt(-1)
fmt.Println(r)					// (0+1i)
```

### 布尔

布尔类型为`bool`，取值只能为`true`或`false`，布尔变量或表达式可以使用`&&`和`||`进行组合，并且支持**短路**。

布尔变量不会隐式转换成0或1:

```go
func main() {
	const b bool = true
	if (b) {
		fmt.Println(b)				// true
	}

	fmt.Println(btoi(b))		// 1
	fmt.Println(itob(1))		// true
}

func btoi(b bool) int {
	if b {
		return 1
	}
	return 0
}

func itob(i int) bool {
	return i != 0
}
```

### 字符串

字符串类型为`string`，表示一个不可变的字节序列。在`string`上的操作都是基于字节（`byte`）的，比如:

- `len()`函数返回字节数
- `s[i:j]`操作返回第i（包含）个字节到第j（不包含）个字节间的子串
- 比较操作（如`==`或`<`），逐个字节的比较

`string`的第i个字节并不一定等于第i个字符，因为非`ASCII`字符码点（`rune`）需要2个或多个字节。

```go
s := "hello, world"
fmt.Println(len(s))										// 12
fmt.Println(s[0], s[7], s[0] == 'h', s[7] == 'w')		// 104 119 true true

fmt.Println(s[7:])			// world
fmt.Println(s[:5])			// hello
fmt.Println(s[:])				// hello, world

s2 := "hello"
fmt.Println(s > s2)		// true
fmt.Println(s2 == s[:5])		// true
```

---

原始（`raw`）字符串以反引号表示，即**``**，其中的所有字符都不会被转义，因此可以跨行。

```go
const GoRunHelp = `
	usage: go run [build flags] [-exec xprog] package [arguments...]
	Run 'go help run' for details.
	`
fmt.Println(GoRunHelp)
```

---

Unicode码点（*code point*），在Go中叫`rune`。在`UTF-8`编码中，对于`ASCII`字符，一个字节表示一个`rune`，但其它语言中的有些字符，可能2～3个字节表示一个`rune`。

在`string`变量上使用`range`循环时，会自动进行`UTF-8`解码（decoding）：

```go
s3 := "hello, 世界"
fmt.Println(len(s3))												// 13
fmt.Println(utf8.RuneCountInString(s3))			// 9
for i := 0; i < len(s); {
r, size := utf8.DecodeRuneInString(s3[i:])
fmt.Printf("%d\t%c\n", i, r)
i += size
}	
for i, r := range(s3) {
fmt.Printf("%d\t%c\n", i, r)
}
/*
0	h
1	e
2	l
3	l
4	o
5	,
6
7	世
10	界
*/
```

将整数转换成字符串时，会将整数解析成`UTF-8`表示的`rune`值：

```go
fmt.Println(string(65))				// "A"
fmt.Println(fmt.Sprintf("%d", 65))		// "65"
```

---

在对字符串进行操作时，以下4个包很重要：

- `strings`：字符串的搜索、替换、连接、包含等函数
- `bytes`：对`[]byte`类型的操作
- `strconv`：`bool`、`int`、`float`与`string`之间的互相转换
- `unicode`：对`rune`的操作，包括`IsDigit`, `IsLetter`, `IsUpper`, `IsLower`等函数

`string`类型是不可变的，所以不能进行修改，但是`[]byte`是可以任意修改的。同时`bytes`包提供了`Buffer`类型用于对[]byte的高效操作。

为了避免不必要的类型转换以及内存分配，`bytes`包提供了与`strings`包中功能相同的工具函数，只是函数参数是`[]byte`，而不是`string`，如`contains`,` count()`等。

```go
func intsToString(values []int) string {
  var buf bytes.Buffer
  buf.WriteRune('[')
  for i, v := range values {
    if i > 0 {
      buf.WriteRune(',')
    }
    fmt.Fprintf(&buf, "%d", v)
  }
  buf.WriteRune(']')
  return buf.String()
}
fmt.Println(intsToString([]int{1, 2, 3, 4}))		// [1,2,3,4]
```

---

将整数转换成字符串有两种方式，使用`fmt.Sprintf`函数，或者使用`strconv.Itoa`函数（即*integer to ASCII*）。将字符串转成整数，可以使用`strconv.Atoi`函数或者`strconv.ParseInt`函数：

```go
x := 456
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x))		// 456 456
fmt.Println(strconv.FormatInt(int64(x), 2))		// 111001000
fmt.Println(fmt.Sprintf("x = %b", x))					// x = 111001000

m := "789"
n, _ := strconv.Atoi(m)
p, _ := strconv.ParseInt(m, 10, 64)
fmt.Println(n, p)			// 789 789
```

### 常量

`const`关键字定义常量，即值在编译时即已知。

当定义一组常量时，后定义的常量的表达式可以省略，表示与前一个常量的类型和值相同（所以第一个常量的定义不可省略）：

```go
const (
  a = 1
  b
  c = 3
  d
)
fmt.Println(a, b, c, d)				// 1 1 3 3
```

在`const`声明中，常量生成器`iota`的起始值为0，后续每一项的值递增：

```go
type Weekday int
const (
  Sunday Weekday = iota
  Monday
  Tuesday
  Wednesday
  Thursday
  Friday
  Saturday
)
fmt.Println(Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday)		// 0 1 2 3 4 5 6
```



> 参考：《The Go Programming Language》(Alan Donovan, Brian Kernighan) (英文版)