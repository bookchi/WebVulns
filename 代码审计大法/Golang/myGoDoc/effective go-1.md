## effective go-1

- [effective go-1](#effective-go-1)
	- [基本规范](#基本规范)
	- [基本结构](#基本结构)

主要是go语言的一些规范。

### 基本规范

- 格式化

  这是一个充满争议的问题，go不想面对这个争议，所以把问题交给了程序。

  直接用gofmt(以包为处理对象)命令对写好的代码格式化即可。

- 注释

  Go支持`/**/`和`//`注释。

  一般包前，会有一段块注释；exported func之前会有行注释，并且函数的注释以函数名开头；group声明，注释会比较笼统。

  例如：

  ```go
  /*
  Package regexp implements a simple library for regular expressions.
  
  The syntax of the regular expressions accepted is:
  
  	regexp:
  		concatenation { '|' concatenation }
  	concatenation:
  		{ closure }
  	closure:
  		term [ '*' | '+' | '?' ]
  	term:
  		'^'
  		'$'
  		'.'
  		character
  		'[' [ '^' ] character-ranges ']'
  		'(' regexp ')'
  */
  package regexp
  
  
  // Compile parses a regular expression and returns, if successful, a Regexp
  // object that can be used to match against text.
  func Compile(str string) (regexp *Regexp, err error) {
  
  
  // 表达式解析失败后返回错误代码。
  var (
  	ErrInternal      = errors.New("regexp: internal error")
  	ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
  	ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
  	...
  )
  ```

- 命名

  - 包名

    以小写字母开头，不应包含下划线or驼峰；包名和最近的目录同名；包名和函数名共同表达出函数的用途。长命名并不会使代码更具可读性，建议用注释。

  - getters

    Go并不对getter和setter提供自动支持，但可以自己设置，并且通常很值得这样做。

    假设有一个`owner`字段，getter：`Owner`；setter：`SetOwner()`

  - 接口名

    只有一个method的接口，应该加上er后缀，例如：`Reader`、`Writer`、 `Formatter`、`CloseNotifier `等；Read、Write、Close、Flush、 String 等都具有典型的签名和意义，未免冲突，不要用这些名称作为方法名。

  - 驼峰命名

    Go使用驼峰命名，而非下划线。

- 分号

  Go无需在源代码的行末加上分号，词法分析器会根据规则自动插入。

### 基本结构

- 控制结构

  if语句，无需括号，可包含初始化语句； 用 if 和 switch来设置局部变量十分常见。

  典型示例：

  ```go
  if x > 0 {
  	return y
  }
  
  // 初始化与err
  if err := file.Chmod(0664); err != nil {
  	log.Print(err)
  	return err
  }
  ```

- redeclaration和reassignment

  已被声明的变量v要出现在:=中(重新赋值，这个特性纯粹是为了实用)，需要满足：

  - := 与v同域（否则就是创建新的err
  - 类型相同
  - :=中至少有一个变量是新声明的

- for循环

  Go没有do/while，只有for；有三种形式：

  ```go
  // while true{}
  for{}
  // while cond{}
  for cond{}
  // 类似C的for
  for init; cond; post{}
  ```

  `for-range`语句也很常用，对于遍历map、slice、channel等很实用。

  ```go
  // key, value 都要
  for key, value := range array {
  	sum += value
  }
  // 只要key
  for key := range array {
  	sum += value
  }
  // 只要value
  for _, value := range array {
  	sum += value
  }
  ```

- switch

  比C的更通用，case的expr无需必为常量或整数；switch后没有expr将会匹配true。

  将if-else-if-else链写成一个switch，更符合go的style。

  ```go
  func unhex(c byte)byte{
   switch {
  	case '0' <= c && c <= '9':
  		return c - '0'
  	case 'a' <= c && c <= 'f':
  		return c - 'a' + 10
  	case 'A' <= c && c <= 'F':
  		return c - 'A' + 10
  	}
  	return 0
  }
  
  // case：逗号列举条件
  switch c {
  	case ' ', '?', '&', '=', '#', '+', '%':
  		return true
  	}
  	return false
  ```

  也可设置标签，然后break到那里。

- type switch

  用于判断接口变量的动态类型。

  ```go
  switch t := t.(type) {
  default:
  	fmt.Printf("unexpected type %T", t)       // %T prints whatever type t has
  case bool:
  	fmt.Printf("boolean %t\n", t)             // t has type bool
  case int:
  	fmt.Printf("integer %d\n", t)             // t has type int
  case *bool:
  	fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
  case *int:
  	fmt.Printf("pointer to integer %d\n", *t) // t has type *int
  }
  ```

- 函数

  可有多个返回值；返回结果可命名；defer；

  defer用于预设函数调用(也就是推迟执行)，比如可用于关闭文件。

  ```go
  	f, err := os.Open(filename)
  	if err != nil {
  		return "", err
  	}
  	defer f.Close()  // f.Close 会在我们结束后运行。
  ```

  defer能确保你不会忘记关闭文件；打开离关闭很近，结构更清晰。这也是go的idiom。

  defer函数的参数，在推迟执行时被求值，而非真的去执行这个调用函数时。

  ```go
  for i := 0; i < 5; i++ {
  	defer fmt.Printf("%d ", i)
  }
  // LIFO: 4 3 2 1
  
  func trace(s string) string {
  	fmt.Println("entering:", s)
  	return s
  }
  
  func un(s string) {
  	fmt.Println("leaving:", s)
  }
  
  func a() {
  	defer un(trace("a"))
  	fmt.Println("in a")
  }
  
  func b() {
  	defer un(trace("b"))
  	fmt.Println("in b")
  	a()
  }
  
  func main() {
  	b()
  }
  
  // prints: 
  entering: b
  in b
  entering: a
  in a
  leaving: a
  leaving: b
  ```

  panic与recover也常用到defer。

- data

  - new与make分配、构造函数与**复合字面量**

    `new(T)`不会初始化内存，而是置零，并返回`*T`(指向新分配的类型为T的零值)

    `make(T, args)`，只用于创建slice、map和channel，返回一个已初始化(而非置零)的`T`。

    *因为这三种类型为引用数据类型(例如slice是包含了数组ptr，len和cap的结构体)，这三项需要初始化而非简单置零；若未能初始化，切片为nil。*

    示例如下：

    ```go
    var p *[]int = new([]int) // *p = nil，没啥用
    var v []int = make([]int, 100)			// v，一个slice
    
    // 没必要
    var p *[]int = new([]int)
    *p = make([]int, 100, 100)
    
    // 习惯用法
    v := make([]int, 100)
    ```

    有时零值不够，需要初始化的构造函数。要知道，获取一个复合字面量的地址时，都将为一个新的实例分配内存。

    ```go
    func NewFile(fd int, name string) *File{
      // 这么一个复合字面量
      return &File{fd, name, nil, 0}
    }
    ```

  - arrary与slice

    既然有切片了，数组有什么用？数组是切片的底层结构，而且在详细规划内存布局时，数组非常有用的,有时能避免过多的内存分配。

    Go中的数组：

    - 是值；数组赋值是会复制所有的元素
    - 值传递；数组大小也是类型的一部分，[10]int和[20]int类型不同。

    Go 中的大部分数组编程都是通过切片来完成。

    一个切片赋值给另一个切片，会引用相同的底层数组。

    必须返回切片，切片自身（其运行时数据结构包含指针、长度和容量）是通过值传递的。(*如果追加数据过大，切片底层数组会重新分配内存，指针会改变，所以要返回*)

    ```go
    func (file *File) Read(buf []byte) (n int, err error){
    	buf[0:32]
    }
    ```

    

  - 二维slice

    遍历二维slice时，有两种方式：1.独立分配每一个切片；2。只分配一个数组，然后指向它。后者适用于切片长度固定的情况。

    ```go
    picture := make([][]uint8, YSize) // 每 y 个单元一行。
    // 遍历行，为每一行都分配切片
    for i := range picture {
    	picture[i] = make([]uint8, XSize)
    
    // 2.只分配一个数组 
    picture := make([][]uint8, YSize) // 每 y 个单元一行。
    // 分配一个大的切片来保存所有像素
    pixels := make([]uint8, XSize*YSize) // 拥有类型 []uint8，尽管图片是 [][]uint8.
    // 遍历行，从剩余像素切片的前面切出每行来。
    for i := range picture {
    	picture[i], pixels = pixels[:XSize], pixels[XSize:]
    }
    ```

  - map

    maps用来关联不同的值。key可为任何支持相等性的类型，如数字、string、指针、接口(动态类型支持相等性)、结构体和数组，但**不可为slice**(相等性未定义)。

    一般使用复合字面量语法构建，也可像数组那样赋值。

    ```go
    var timeZone = map[string]int{
    	"UTC":  0*60*60,
    	"EST": -5*60*60,
    	"CST": -6*60*60,
    	"MST": -7*60*60,
    	"PST": -8*60*60,
    }
    
    offset := timeZone["EST"]
    ```

    其他的，判断key是否存在（也是go的idiom），删除某项之类的操作。

    ```go
    if seconds, ok := timeZone[tz]; ok {
    		return seconds
    }
    _, ok := timeZone[tz]
    // 删除某项，参数为key；即便key不存在也是安全的
    delete(timeZone, "PDT")
    ```

  - print

    。。。这个不想写

  - append

    显然只针对slice

    ```go
    func append(slice []T, elements ...T) []T
    // 添加多个elems
    x := []int{1,2,3}
    x = append(x, 4, 5, 6)
    
    // 追加切片
    x := []int{1,2,3}
    y := []int{4,5,6}
    x = append(x, y...)
    ```

   > md累了写不动了...后续的写在2里吧
