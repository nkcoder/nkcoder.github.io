---
title: Go的程序结构
date: 2021-02-26T09:46:31+08:00
categories:
- Go程序设计
draft: false
---

### 命名 (Names)

Go中的函数、变量、常量、类型、语句标签（label）、包都遵循一个简单的命名规则：以字母开头或以下划线(`_`)开头，紧跟着任意数量的字母、数字或下划线，大小写敏感。

Go的**25**个关键字不能作为名称：

```
break			default				func		interface		select
case			defer					go			map					struct
chan			else					goto		package			switch
const			fallthrough		if			range				type
continue	for						import	return			var
```

还有一些预定义的常量、类型和函数，这些名称虽然不是保留关键字，可以使用，但是容易引起混淆：

```tex
Constants		true	false	iota	nil
Types				int	int8	int16	int32	int64	uint	uint8	uint16	uint32	uint64	uintptr
						float32	float64	complex128	complex64		bool	byte	rune	string	error
Functions		make	len	cap	new	append	copy	close	delete	complex	real	
						imag	panic	recover
```

函数里定义的变量仅在函数内可见；函数外定义的变量（或函数）对属于同一包（*package*）的所有源文件可见。如果名称（变量或函数）的首字母是大写的，则表示被导出（*exported*），在包（*package*）外可见，如`fmt`包的`Printf`函数。包名总是全小写的。

对名称的长度没有限制，但Go倾向于短名称，尤其是在较小的作用域中。一般来讲，作用域越大，名称应该越长、越有意义。

如果名称包含多个单词，使用驼峰方式，即将内部单词的首字母大写，而不是用下划线连接，如`parseRequestLine`。

### 声明 (Declarations)

主要有4种声明：`var`, `const`, `type`, `func`。

一个标准的Go源文件，首先是`packge`声明，然后是`import`声明，最后是变量、常量、类型和函数等包级别的声明。

### 变量

变量声明的通用形式为：

```go
var name type = expression
```

`type`和`= expression`是可选的，但不能同时省略。

- 如果`type`省略，通过表达式推断类型
- 如果`= expression`省略，则使用类型的零值（*zero value*）：数值类型为*0*，布尔类型为*false*，字符串类型为*""*，接口或引用类型（如*slice*, *pointer*, *map*, *channel*, *function*）为*nil*，数组或*struct*类型的所有字段使用对应类型的零值。

在Go里，没有*未初始化的变量*这一说法，所有变量都有合理的默认值，这样可以简化代码以及一些边界场景。

可以一次声明多个变量，如果忽略类型，可以定义多个不同类型的变量：

```go
var i, j, k int
var b, f, s = true, 2.3, "four"
```

---

**简短变量声明**的形式为：

```go
name := expression
```

因为这种方式的简洁性和灵活性，大多数局部变量的声明都会使用这种形式。在以下两种场景下使用*var*定义局部变量：

- 根据变量的初始值推断的类型与变量的类型不同，如：`var boiling float64 = 100`
- 变量的初始值缺失或不重要，如：`var names []string`

简短变量声明方式也可以同时声明多个变量：

```go
i, j := 0, 1
```

但是这种方式应该仅在有助于提高代码可读性时使用，比如`for`循环的初始化。

**注意**：`:=`是变量声明，而`=`是赋值，所以不要把多个变量的简短声明与元组赋值项混淆。

```go
i, j = 2, 3
```

与*var*声明一样，`:=`也可以直接用于函数调用返回多个值的场景：

```go
f, err := os.Open(fileName)
```

还有一点需要注意的是，`:=`必须至少声明一个新变量，即`:=`左侧的变量必须至少有一个是新声明的：

```go
in, err := os.Open(infile)
// ...
out, err :os.Open(outfile)	// 正确：out是新变量
```

```go
file, err := os.Open(infile)
// ...
file, err :os.Open(outfile)	// 编译错误：没有声明新变量
```

---

**指针（pointer）**：就是变量的地址，每个变量都有地址。可以通过指针间接地更新变量的值。

```go
x := 1
p := &x
*p = 2
```

`&x`表示获取变量*x*的地址，创建一个*指针*指向该整型变量，*指针*的类型是`*int`，我们称为`p指向x`，或者`p包含x的地址`；`*p`表示*p*指向的变量，当`*p`出现在`=`的右边，表示获取指针*p*指向的变量的值，当`*p`出现在`=`的左边时，表示更新*p*指向的变量。

指针的零值（*zero value*）是`nil`。指针是可以比较的，当且仅当两个指针指向同一个变量或两个指针都为`nil`，它们才相等。如果*p*指向某个变量，则`p != nil`是*true*。

```go
var i, j int
fmt.Println(&i == &i, &i == &j, &i == nil) // true false false
```

将函数中局部变量的地址返回完全是安全的，指针仍然指向之前的变量：

```go
func f() *int {
	v := 3
	return &v
}

p1 := f()
p2 := f()
fmt.Printf("p1 == p2: %t, p1 = %d, p2 = %d\n", p1 == p2, *p1, *p2) // p1 == p2: false, p1 = 3, p2 = 3
```

> 在Go里局部变量可能分配在栈上，也可能分配在堆上，具体参考后面关*局部变量的生命期*一节。本例中，因为*v*在函数外被引用，所以*v*是分配在堆上的。每次调用`f()`返回的地址都不同。

将指针作为函数的参数传递时，因为可以通过指针间接修改所指向变量的值，所以要注意可能产生副作用：

```go
func incr(p *int) {
	*p++
}
p3 := 1
incr(&p3)
fmt.Printf("p3 = %d\n", p3) // p3 = 
```

指针还是`flag`包的关键。`flag`包用于设置和解析命令行参数的：

`echo4.go`：

```go
// n and sep and pointers
var n = flag.Bool("n", false, "commit trailing newline")
var sep = flag.String("s", " ", "seperator")

func main() {
	flag.Parse()
	result := strings.Join(flag.Args(), *sep)
	fmt.Print(result)

	if !*n {
		fmt.Println()
	}
}
```

> `flag.Bool()`创建一个*bool*类型的`flag`参数，`flag.Parse()`解析参数，`flag.Args()`获取非`flag`参数。

```bash
➜  ch2-program-structure ./echo4 --help
Usage of ./echo4:
  -n	commit trailing newline
  -s string
    	seperator (default " ")
➜  ch2-program-structure ./echo4 a b c
a b c
➜  ch2-program-structure ./echo4 -s / a b c
a/b/c
➜  ch2-program-structure ./echo4 -n a b c
a b c%
```

---

**new函数**：`new`是预定义函数，是创建变量的另一种方式。`new(T)`创建一个类型为`T`的匿名变量，默认值为`T`的零值，返回变量的地址，即指针。

```go
func main() {
	p1 := new(int)
	fmt.Println(*p1) // 0

	*p1 = 1
	fmt.Println(*p1) // 1
}
```

用`new`函数创建的变量与用var创建的变量然后返回地址没什么本质区别，唯一的区别就是`new`函数创建的是匿名变量，所以`new`函数更像是一种*语法糖*。以下两种方式是等价的：

```go
func newIntByNew() *int {
	return new(int)
}

func newIntByVar() *int {
	var dummy int
	return &dummy
}
```

另外，`new`属于预定义函数，不是关键字，所以名称可以被覆盖定义（注意不要引起混淆）：

```go
func delta(old int, new int) int {
	return new - old
}
delta(5, 10) // 5
```

---

**变量的生命期**：即变量在程序执行过程中存在的时间。*包级别*的变量的生命期为程序的整个执行期，而*局部变量*的生命期则是不固定的：变量会一直存活直到**不可达 (unreachable)**，此时变量的存储空间可能会被垃圾回收。

因为变量的生命期取决于其是否可达，所以如果在函数外引用函数内的局部变量，则该局部变量可能会一直存在，即使函数已经结束。

编译器会选择将局部变量分配在*栈(stack)*上或*堆(heap)*上，但不是根据`var`或`new`来决定的：

```go
var global *int

func main() {
	global = f()

	g()

	fmt.Println(*global)
}

func f() *int {
	var x int
	x = 1
	return &x
}

func g() {
	y := new(int)
	*y = 1
}
```

> 变量`x`是分配在*堆*上的，因为函数`f`返回了`x`的地址，所以`x`在函数`f`外仍然是可达的，这种情况称为`x`从`f`中逃逸。
>
> 在函数`g`中，`y`并没有在函数外被引用，所以`y`是分配在栈上的，当函数结束后，`y`可以被回收。

虽然我们不需要显式地分配和释放内存，但是为了写出高效的代码，我们仍然需要关注变量的生命期。比如，在长生命期的变量（尤其是全局变量）中，通过不必要的指针引用短生命期的局部变量，将阻止编译器对短生命期对象的垃圾回收。

### 赋值 (Assignment)

**元组 (tuple) 赋值**：同时对多个变量赋值。`=`右边的表达式先解析，然后才赋值给`=`左边的变量，当变量同时出现在`=`两边时尤其有用。

```go
func main() {
	x, y := 1, 2
	x, y = y, x

	fmt.Printf("x = %d, y = %d\n", x, y) // x = 2, y = 1
}
```

如果表达式比较复杂，尽量避免元组赋值，单个赋值的可读性更强一些。

很多函数调用的返回值有多个，如果赋值给变量，则`=`左侧需要数量相同的变量，如果某个变量不会用到，则可以使用`_`来忽略：

```go
	m1 := make(map[string]int)
	m1["x"] = 1
	m1["y"] = 2

	v, ok := m1["x"]
	if !ok {
		fmt.Fprintf(os.Stderr, "not exist in map")
	}

	fmt.Printf("v = %d\n", v)	// v = 1
```

---

**类型声明 (type declarations)**：以一个已经存在的类型作为底层类型，定义一个类型名，主要用于分离与该底层类型不同或者不兼容的使用。

类型声明大多数出现在包级别，因此在整个包内可见，如果被导出（首字母大写），则对其它包也是可见的。

```go
type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC     Celsius = 0
	BoilingC      Celsius = 100
)

func CToF(c Celsius) Fahrenheit {
	return Fahrenheit(c*9/5 + 32)
}

func FToC(f Fahrenheit) Celsius {
	return Celsius((f - 32) * 5 / 9)
}
```

> - `Celsius`和`Fahrenheit`的底层类型都是`float64`，但它们不是同样的类型，所以不能直接比较，或混合进行四则运算
> - `Fahrenheit()`和`Celsius()`是类型转换，不是函数调用，不会改变值，而是改变值的含义。

对于任意类型`T`，都存在一个对应的转换操作`T(x)`，将变量`x`的值转换成类型`T`。

- 这种转换仅当它们的底层类型相同，或当它们都是指针，指向的变量的底层类型相同时才可以。
- 这种转换只会改变类型，不会改变值

数值类型之间的转换也是允许的，但是可能会改变值，如将浮点类型转换为整型会丢弃小数部分：

```go
x := 10.555
fmt.Println(int(x)) // 10
```

### 包与文件 (Package & Files)

**导入 (Import)**：根据约定，包名为导入路径的最后一部分，如：`import "a/b/c"`，则包名为`c`。然后可以通过包名+包内导出的变量/函数名去引用，如`c.Convert()`。

导入一个包但是不使用会导致编译错误。可以借助`goimports`工具或IDE等自动添加、移除包。

---

**包初始化 (package initialization)**：即初始化包级别的变量，根据变量声明的顺序依次初始化，如果变量之间有依赖关系，则根据依赖进行初始化：

```go
var a = b + c
var b = f() + 1
var c = 1

func f() int {
	return c + 1
}

func main() {
	fmt.Println(a) // 4
}
```

如果包下有多个源文件，`go`命令会根据文件名对这些文件进行排序，然后发送给编译器，编译器根据收到的文件的先后顺序进行初始化。

如果变量的初始化比较复杂，可以放在初始化函数`init`中。一个源文件里可以包含任意多个以下形式的`init`函数：

```go
func init() {
	// ...
}
```

这些`init`函数不能被调用或引用，但它们确实是正常的函数。一个文件里的`init`函数以声明的先后顺序在程序启动的时候被自动执行。

```go
func main() {}

func init() {
	println("this is init one")
}

func init() {
	println("this is init two")
}
```

包根据导入的顺序和依赖关系依次被初始化，`main`包最后被初始化。

### 作用域 (Scope)

***声明 (declaration)*** 将名称与标识符（如变量或函数）关联，声明的作用域指的是源码中可以通过该名称引用到该声明的范围。

不要将作用域 (*scope*) 与生命期 (*lifttime*)混淆。声明的作用域指的是程序的一段文本块，是一个编译时(*compile-time*)属性；而变量的生命期指的是在程序执行过程中变量可以被引用的时间段，是一个运行时(*run-time*)属性。

大括号`{}`构成一个**句法块(*syntactic block*)**，如函数体或循环体。在*句法块*中声明的名称在句法块外是不可见的。**语法块(*lexical blocks*)**是句法块的推而广之，一定范围的语句都可以构成语法块，如每个包、每个文件、每个`for`、`if`、`switch`语句等都有自己的语法块，整个源码构成的语法块称为**全局语法块(*universe block*)**。

一个声明的语法块决定了它的作用域大小。内嵌的类型、函数和常量，如`int`, `len`, `true`等都属于全局语法块，可以在整个程序中被引用。

程序可以包含同一个名称的多个声明，只要这些声明的语法块不同。编译器遇到名称引用的时候，从最内层的语法块一直寻找到全局语法块，如果没找到，则报错，如果在内部代码块和外部代码块都找到了声明，则使用先找到的内部语法块中的声明，这种情况称为：内部声明隐藏(*hide*)或屏蔽(*shadow*)了外部声明。

```go
func f() {}

var g = "g"

func main() {
	f := "f"
	fmt.Println(f) // "f"
	fmt.Println(g) // "g"
}
```

> 局部变量`f`屏蔽了包级别函数`f()`。

`for`循环会创建两个语法块：一个是循环体（即`{}`中的语句，显式语法块），另一个是循环初始化语句（隐式语法块）。初始化语句中声明的变量的作用域为循环条件、循环后置操作以及循环体，如：

```go
func toUpper1() {
	x := "hello!"
	for i := 0; i < len(x); i++ {
		x := x[i]
		if x != '!' {
			x := x + 'A' - 'a'
			fmt.Printf("%c", x)
		}
	}
	fmt.Println()
}
```

> - 循环的初始化语句中声明的变量`i`在循环体中、循环条件中以及循环后置语句中(`i++`)可见。
> - `if`语句构建的内部语句块中的变量`x`屏蔽了`for`循环体中声明的`x`，`for`循环体中声明的变量`x`又屏蔽了函数体中声明的`x`。

与`for`循环类似，`if`语句和`swith`语句也会同时创建两个语法块：条件和语句体。

- `if`条件中声明的变量的作用域为`if`语句体以及嵌套的`else`和`else if`
- `swith`条件中声明的变量的作用域为swith语句体

```go
func printValue() {
	if x := rand.Intn(50); x > 10 {
		fmt.Println(x)
	} else if y := rand.Intn(50); y > x {
		fmt.Println(y)
	}
	// fmt.Printf("x = %d, y = %d\n", x, y)	// compile error: x and y are not visible here
}
```

> 同名变量在不同作用域中导致屏蔽，会使代码的可读性变差，应该尽量避免使用。

---

> 参考：《The Go Programming Language》(Alan Donovan, Brian Kernighan) (英文版)