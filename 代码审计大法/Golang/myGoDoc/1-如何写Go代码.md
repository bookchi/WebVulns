# 1-如何写Go代码

- [1-如何写Go代码](#1-如何写go代码)
         - [Code organization](#code-organization)
         - [Your first program](#your-first-program)
         - [Import packages from your code](#import-packages-from-your-code)
         - [Import packages from remote modules](#import-packages-from-remote-modules)
         - [Testing](#testing)
         
### Code organization



Go程序是通过package组织的。package是一个目录，其中包含了许多被一起编译的源文件。

一个源文件中定义的函数、类型、变量和常量对同一个包下的其他源文件来讲，是可见的。



一个repo包含了一个或多个module。module是一系列一起发布的packges。一个Go repo通常仅包含一个module，位于repo的根目录。`go.mod`文件声明了module path——module内所有package导入路径的前缀。



一个好习惯：一个module要有repo，而非单独一个module。

每个module的path不仅是导入的前缀，也是下载的地址。

一个import path是用来导入package的字符串，是module path+子目录。

###  Your first program

为了编译和执行一个简单的程序，首先选择一个module path(这里使用`example.com/user/hello`)并且创建`go.mod`

```shell
$ mkdir hello
$ cd hello
$ go mod init example.com/user/hello
go: creating new go.mod: module example.com/user/hello
$ cat ../go.mod
module example.com/user/hello

go 1.15

```

Go源文件首行必须声明package name，可执行命令必须使用`package main`。

Next，创建`hello.go`：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```

然后使用`go`工具来build和install。

```shell
$ go install example.com/user/hello
# or
$ go install .
# or
$ go install
```

这个命令会产生一个二进制文件，在GOBIN或GOPATH/bin目录下。

然后执行这个命令：

```shell
$ hello
Hello, world.
```



### Import packages from your code

在`hello/morestrings`编写`reverse.go`文件。

```go
package morestrings

func ReverseRunes(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

由于ReverseRunes是以大写字母开头，因此它是exported，可以在其他package中使用。

使用`go build`来测试这个package：

```shell
$ cd morestrings
$ go build
$
```

这并不会产生一个output file，而是将编译的package包存在本地的build cache中。

然后修改`hello.go`：

```go
package main

import (
	"fmt"

	"example.com/user/hello/morestrings"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
}
```

安装这个`hello`程序然后执行：

```shell
$ go install example.com/user/hello
$ hello
Hello, Go!
```



### Import packages from remote modules

一个import path可以描述如何使用版本控制系统(如Git)获取package源代码。Go使用这个特性来自动获取远程仓库的package。例如：

```go
package main

import (
	"fmt"

	"example.com/user/hello/morestrings"
	"github.com/google/go-cmp/cmp"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
	fmt.Println(cmp.Diff("Hello World", "Hello Go"))
}
```

当使用`go install`、`go build`、`go run`等命令时，`go`命令会自动下载远程的module并且在`go.mod`中记录版本信息。

```shell
$ go install example.com/user/hello
go: finding module for package github.com/google/go-cmp/cmp
go: downloading github.com/google/go-cmp v0.4.0
go: found github.com/google/go-cmp/cmp in github.com/google/go-cmp v0.4.0
$ hello
Hello, Go!
  string(
- 	"Hello World",
+ 	"Hello Go",
  )
$ cat go.mod
module example.com/user/hello

go 1.15

require github.com/google/go-cmp v0.5.2
$
```

module依赖会自动下载到`GOPATH/pkg/mod`中。

### Testing

Go有一个轻量级的测试框架，由`go test`和`testing`package构成。

需要写一个以`_test.go`结尾的文件，然后里面有一个函数以`Test`开头，函数签名为`func (t *testing.T)`。

test框架运行每一个这样的函数，如果函数调用了failure函数(如`t.Error`或`t.Fail`)，则认为测试失败。

写一个`/hello/morestrings/reverse_test.go`代码：

```go
package morestrings

import "testing"

func TestReverseRunes(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := ReverseRunes(c.in)
		if got != c.want {
			t.Errorf("ReverseRunes(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

然后使用`go test`来运行：

```shell
$ go test
PASS
ok  	example.com/user/hello/morestrings	1.093s
```

Run [go help test](https://golang.org/cmd/go/#hdr-Test_packages) and see the [testing package documentation](https://golang.org/pkg/testing/) for more detail.



涉及到了，go mod初始化，go install，go build， go test，还没有go run。
