---
title: Go语言入门
date: 2021-02-23T20:28:08+08:00
categories:
- Go程序设计
draft: false
---

> 跟着《The Go Programming Language》学习Go语言。

在学习一门新语言时，人们可能倾向于以自己熟悉的其它语言的方式来写代码。在学习Go语言时，请摒弃这种偏见。本书的示例代码就是好的、规范的Go语言代码，当写自己的Go代码时，请以此作为参考。

---

### Hello World

`helloworld.go`

```go
package main

import "fmt"

func main() {
	fmt.Println("hello, 世界")
}
```

直接运行：

```bash
$ go run helloworld.go
```

编译再运行：

```go
$ go build helloworld.go
$ ./helloworld
```

- Go是一门编译型语言，`run`命令执行编译、链接和运行，`build`命令执行编译和链接，并生成可执行程序。
- Go也是以包（package）的方式组织代码。每个源文件都以`package`声明开头，表示该源文件所属的包， `import`用于导入程序依赖的包。`main`是一个特殊的包，表示定义的是可独立运行的程序，而不是库（library）。你必须只导入（import）程序需要的包，如果导入了不必要的包，或者没有导入需要的包，程序将无法编译。
- 函数定义由`func`关键字、函数名称、参数列表、返回值构成。`main`是一个特殊的函数，表示程序的执行入口。
- 在语句或声明的最后，不需要分号表示结束，除非多条语句或声明出现在一行上。

### 命令行参数

`os`模块的`Args`变量的值是一个字符串切片（slice，可以暂时理解为可动态扩容的序列），表示命令行参数，第一个参数`Args[0]`表示程序的名称，`Args[1:]`表示程序执行所需的所有参数。

`echo1.go`:

```go
// echo1 prints it's command-line arguments
package main

import (
	"fmt"
	"os"
)

func main() {
	var s, sep string
	for i := 1; i < len(os.Args); i++ {
		s += sep + os.Args[i]
		sep = " "
	}

	fmt.Println(s)
}
```

运行：

```bash
$ go run echo1.go hello world
```

- `//`表示注释，注释的内容会被编译器忽略

- `var`定义变量，`string`是类型。变量定义时可以初始化，如果没初始化，默认值为类型的零值（zero value），如数值类型的零值为0，字符类型的零值为空字符串""

- `:=`表示简短变量声明，即声明变量并基于默认值推断合适的变量类型

- `++`表示自增，还有`--`表示自减，仅支持后缀操作符，不支持前缀操作符，如`--i`是不合法的

- `for`循环是go唯一的循环语句

  ```go
  for initialization; condition; post {
  	
  }
  ```

  `for`语句中不需要小括号`()`，但大括号`{}`是必须的，且左大括号`{`必须与`post语句`处于同一行。`for`语句的三部分都是可选的。

---

`echo2.go`:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	s, sep := "", ""
	for _, arg := range os.Args[1:] {
		s += sep + arg
		sep = " "
	}

	fmt.Println(s)
}
```

- `for`语句只有*condtion*部分，每次循环迭代，range会产生一对值，第一个值表示当前迭代的索引（index），第二个值表示当前迭代的值。如果不会用到第一个值，可以使用**空标志符**（_，即下划线），空标志符用于语法需要一个变量名，而程序逻辑不需要的场景。

- 变量声明可以有多种形式：

  ```go
  s := ""
  var s string
  var s = ""
  var s string = ""
  ```

  第一种形式最简洁，但只能用于函数中，不能用于包级别变量声明；第二种形式依赖于默认值进行初始化，第三种形式很少使用，除非定义多个变量，第四种形式是冗余的。**实际中，一般使用第一种和第二种形式，一个显式表明初始值很重要，一个隐式表明初始值不重要。**

---

`echo3.go`:

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	fmt.Println(strings.Join(os.Args[1:], " "))
}
```

- `Join`是`strings`包中的方法，用于字符串连接

### 统计重复行

`dup.go`:

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	counts := make(map[string]int)
	files := os.Args[1:]
	if len(files) == 0 {
		countLines(os.Stdin, counts)
	} else {
		for _, arg := range files {
			f, err := os.Open(arg)
			if err != nil {
				fmt.Fprintf(os.Stderr, "dup: %v\n", err)
				continue
			}

			countLines(f, counts)
			f.Close()
		}
	}

	for line, n := range counts {
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)
		}
	}
}

// NOTE: ignoring potential errors from input.Err()
func countLines(f *os.File, counts map[string]int) {
	input := bufio.NewScanner(f)
	for input.Scan() {
		counts[input.Text()]++
	}
}
```

- `make(map[string]int)`表示创建一个map，key是string类型，value是int类型
- `os.Open`打开文件，返回指向文件的指针
- `input.Scan()`读取文件中的行，直到文件结束或发生错误，这里忽略了可能的错误

### 网络请求

`fetch.go`:

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"strings"
)

func main() {
	for _, url := range os.Args[1:] {
		const prefix = "http://"
		if !strings.HasPrefix(url, prefix) {
			url = prefix + url
		}
		resp, err := http.Get(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
			os.Exit(1)
		}

		_, err = io.Copy(os.Stdout, resp.Body)
		statusCode := resp.StatusCode
		resp.Body.Close()
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: reading %s: %v\n", url, err)
			os.Exit(1)
		}

		fmt.Println("status code: ", statusCode)
	}
}
```

- 这个程序的功能与`curl`命令类似，`http.Get`函数发起http请求，`io.Copy`函数复制数据，`body.Close`关闭流防止资源泄漏

---

`fetchall.go`:

```go
package main

import (
	"fmt"
	"io"
	"io/ioutil"
	"net/http"
	"os"
	"time"
)

func main() {
	start := time.Now()
	ch := make(chan string)

	for _, url := range os.Args[1:] {
		go fetch(url, ch)
	}

	for range os.Args[1:] {
		fmt.Println(<-ch)
	}

	fmt.Printf("%.2fs elapsed\n", time.Since(start).Seconds())

}

func fetch(url string, ch chan<- string) {
	start := time.Now()
	resp, err := http.Get(url)
	if err != nil {
		ch <- fmt.Sprint(err)
		return
	}

	nbytes, err := io.Copy(ioutil.Discard, resp.Body)
	resp.Body.Close()
	if err != nil {
		ch <- fmt.Sprintf("while reading %s: %v", url, err)
		return
	}

	secs := time.Since(start).Seconds()
	ch <- fmt.Sprintf("%.2fs %d7d %s", secs, nbytes, url)
}
```

- `goroutine`是并行的函数执行，`channel`是`goroutine`之间的通信机制，即一个`goroutine`将某种类型的数据传递给另一个`goroutine`。`main`方法运行在一个`goroutine`中，`go`语句会创建一个额外的`goroutine`。
- `main`方法通过`make`方法创建一个字符串类型的`channel`

---

`server.go`:

```go
package main

import (
   "fmt"
   "log"
   "net/http"
   "sync"
)

var mu sync.Mutex
var count int

func main() {
   http.HandleFunc("/", handler)
   http.HandleFunc("/count", counter)
   log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func handler(w http.ResponseWriter, r *http.Request) {
   mu.Lock()
   count++
   mu.Unlock()
   fmt.Fprintf(w, "URL.Path = %q\n", r.URL.Path)
}

func counter(w http.ResponseWriter, r *http.Request) {
   mu.Lock()
   fmt.Fprintf(w, "count %d\n", count)
   mu.Unlock()
}
```

运行：

```bash
$ go run server.go &
[1] 4890
$ go run fetch.go http://localhost:8000/
URL.Path = "/"
status code:  200
$ go run fetch.go http://localhost:8000/
^[[AURL.Path = "/"
status code:  200
$ go run fetch.go http://localhost:8000/
URL.Path = "/"
status code:  200
$ go run fetch.go http://localhost:8000/count
count 3
```

- 以/开头的请求会被handler方法处理，以`/count`开头的请求会被counter方法处理，每一个请求都是在独立的`goroutine`中被处理的。
- 因为`count`是共享变量，如果多个请求同时更新，会出现*竞争条件*，所以需要使用`sync.Mutex`进行同步。

---

> 参考：《The Go Programming Language》(Alan Donovan, Brian Kernighan) (英文版)