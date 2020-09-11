 

## effective go-2

- [effective go-2](#effective-go-2)
	- [基本特性](#基本特性)

### 基本特性

- 初始化

  - 常量

    即不变量，在编译时创建，也可能是函数中定义的局部变量；之能是数字、字符、字符串或bool。

    由于编译限制，定义的表达式必须是可被编译器求值的常量表达式，例如1<<3，但math.Sin(30)就不可。(*因为函数调用只在运行时发生*)

    枚举与iota是go的一个idiom。

    ```go
    type ByteSize float64
    
    const (
        // 通过赋予空白标识符来忽略第一个值
        _           = iota // ignore first value by assigning to blank identifier
        KB ByteSize = 1 << (10 * iota)
        MB
        GB
        TB
        PB
        EB
        ZB
        YB
    )
    ```

  - 变量

    与常量类似，但初始值可以使运行时计算的expr。

    ```go
    var (
    	home   = os.Getenv("HOME")
    	user   = os.Getenv("USER")
    	gopath = os.Getenv("GOPATH")
    )
    ```

  - init函数

    源文件可通过，无参数 init 函数来设置一些必要的状态

- 方法

  指针接收器和值接收器，前者可以改变接收器对应变量的值。

  值方法可通过指针和值调用， 而指针方法只能通过指针来调用。

- 接口和其他类型

  接口是一组方法的集合。某个类型实现了这个接口，就相当于是这个接口类型。

  - 类型转换：例如`type Sequence []int`，如果`sequence`想用`[]int`的方法，只需`[]int(Sequence)`转换类型即可。

  - 接口转换：`type switches`是类型转换的一种形式，用于将接口转换为某种类型。它接受一个接口，在选择 （switch）中根据其判断选择对应的情况（case），并转换为这种类型。
  - 类型断言：预期接口是哪种类型。

  ```go
  // type switches
  var value interface{} // 调用者提供的值。
  switch str := value.(type) {
  case string:
  	return str
  case Stringer:
  	return str.String()
  }
  
  // 类型断言
  str, ok := value.(string)
  ```

- 空白符

  - 多重赋值

    ```go
    // 只想要报错
    if _, err := os.Stat(path); os.IsNotExist(err) {
    	fmt.Printf("%s does not exist\n", path)
    }
    ```

    

  - 未使用的imports和变量

    如下代码，有变量和imports未使用，这样编译器会报错。

    ```go
    package main
    
    import (
        "fmt"
        "io"
        "log"
        "os"
    )
    
    func main() {
        fd, err := os.Open("test.go")
        if err != nil {
            log.Fatal(err)
        }
        // TODO: use fd.
    }
    ```

    可通过`_`来修改：空白标识符来引用已导入包中的符号;未使用的变量 fd 赋予空白标识符。

    ```go
    package main
    
    import (
        "fmt"
        "io"
        "log"
        "os"
    )
    
    var _ = fmt.Printf // For debugging; delete when done. // 用于调试，结束时删除。
    var _ io.Reader    // For debugging; delete when done. // 用于调试，结束时删除。
    
    func main() {
        fd, err := os.Open("test.go")
        if err != nil {
            log.Fatal(err)
        }
        // TODO: use fd.
        _ = fd
    }
    ```

    > 但是这么做是图什么？我不导入不就好了吗？

  - 为side effect

    有时只是为了某个包的init函数，此时可以：

    ```go
    import _ "net/http/pprof"
    ```

  - 接口检查

    判断某个类型是否实现了某个接口

    ```go
    if _, ok := val.(json.Marshaler); ok {
    	fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
    }
    ```

    有一种情况需要注意，类型`json.RawMessage`需要custom的输出，它应当实现`json.Marshaler`，但没有静态转换，编译器不会去自动验证。若该类型忽略转换失败来满足该接口，json编码器仍能工作但不是custom。为确保正确实现：

    ```go
    var _ json.Marshaler = (*RawMessage)(nil)
    ```

     若` json.RawMessage`未实现该接口，此包将无法通过编译。

- embedding

  接口嵌套和结构体嵌套。

  **只有接口能被嵌入到接口中。**

  ```go
  // 接口内嵌
  type Reader interface {
  	Read(p []byte) (n int, err error)
  }
  
  type Writer interface {
  	Write(p []byte) (n int, err error)
  }
  
  // 开始接口内嵌--注意没有字段名
  // ReadWriter 接口结合了 Reader 和 Writer 接口。
  type ReadWriter interface {
  	Reader
  	Writer
  }
  ```

  结构体嵌套的意义更加深远，比如`bufio`包中的`Reader`和`Writer`结构体类型。

  嵌套与子类(继承)不同的是，a嵌套了类型b，b的方法会成为a的方法，但调用时接收器是b；子类的接收器则是a。

  ```go
  // ReadWriter stores pointers to a Reader and a Writer.
  // It implements io.ReadWriter.
  // 这样就实现了bufio.Reader、bufio.Writer、io.Writer、io.Reader接口
  type ReadWriter struct {
  	*Reader  // *bufio.Reader
  	*Writer  // *bufio.Writer
  }
  
  // 单纯的包含这两种类型的字段而非嵌套
  // 仍需去实现reader、writer接口中的方法
  type ReadWriter struct {
  	reader *Reader
  	writer *Writer
  }
  ```

  还有常规的内嵌一个字段和常规命名字段。

  ```go
  type Job struct {
  	Command string
  	*log.Logger
  }
  
  
  // 初始化
  func NewJob(command string, logger *log.Logger) *Job {
  	return &Job{command, logger}
  }
  // 直接复合字面量
  job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
  
  // 直接调用logger的方法
  job.Log("starting now...")
  
  job.Logger.Logf("%q: %s", job.Command, fmt.Sprintf(format, args...))
  ```

  > 所以到底是什么意思？

- 并发

  略。。。

- 错误

  error与go的多值返回，可以提供更详细的错误信息。

  错误类型：

  ```go
  type error interface {
  	Error() string
  }
  ```

  通过更丰富的底层模型，可轻松实现这个接口，看见错误并提供上下文，来看标准库`os.Open`的例子。

  错误字符串应尽量指明来源，例如产生该错误的包名前缀(如`image`)。

  ```go
  // PathError 记录一个错误以及产生该错误的路径和操作。
  // 结构体记录了更多的信息，然后去实现error接口，提供更丰富的报错
  type PathError struct {
  	Op string    // "open"、"unlink" 等等。
  	Path string  // 相关联的文件。
  	Err error    // 由系统调用返回。
  }
  
  func (e *PathError) Error() string {
  	return e.Op + " " + e.Path + ": " + e.Err.Error()
  }
  
  // output
  // 包含了出错的文件名、操作和触发的操作系统错误
  // 这可比单纯的no such file or directory有用多了
  open /etc/passwx: no such file or directory
  ```

- panic

  当程序发生致命错误时，会调用panic，并结束掉程序，分为显示调用和隐式调用。

  隐式调用：数组越界、空指针，就会隐式调用panic并终结程序。

  显示调用：在代码中显示的panic("xxxxx")。*(参数一般为字符串)*

  ```go
  if user == "" {
  		panic("no value for $USER")
  	}
  ```

- recover

  发生panic时，恢复一些东西(挽回)。

  > *当 panic 被调用后（包括不明确的运行时错误，例如切片检索越界或类型断言失败）， 程序将立刻终止当前函数的执行，并开始回溯 goroutine 的栈，运行任何被推迟的函数。 若回溯到达 goroutine 栈的顶端，程序就会终止。不过我们可以用内建的 recover 函数来重新取回 goroutine 的控制权限并使其恢复正常执行。*

  例如：

  ```go
  package main
  
  import "fmt"
  
  func a(){
      fmt.Println("aaaaaaa")
  }
  
  func b(){
      panic("awsl")
      fmt.Println("bbbbbbb")
  }
  
  func c(){
      fmt.Println("ccccccc")
  }
  
  func main(){
      a()
      b()
      c()
  }
  
  // prints，发现c受到b，没有执行
  aaaaaaa
  panicxxxxxxxx
  ```

  但如果加上recover，**必须在`defer`func中**：

  ```go
  package main
  
  import "fmt"
  
  func a(){
      fmt.Println("aaaaaaa")
  }
  
  func b(x int){
      defer func(){
          if err := recover(); err != nil{
          	fmt.Println(err)
      	}
      }()
      
      var a [10]int
      a[x] = 11
      
  }
  
  func c(){
      fmt.Println("ccccccc")
  }
  
  func main(){
      a()
      b(20)
      c()
  }
  
  // prints
  aaaaaaa
  runtime error: xxxxx
  ccccccc
  ```

### 收官

最后来一个web server的编写。

```go
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET"><input maxLength=1024 size=70
name=s value="" title="Text to QR Encode"><input type=submit
value="Show QR" name=qr>
</form>
</body>
</html>
`
```

 Go语言强大到能让很多事情以短小精悍的方式解决。

> 还有init那部分有问题
>
> 嵌套有一点小问题

   





