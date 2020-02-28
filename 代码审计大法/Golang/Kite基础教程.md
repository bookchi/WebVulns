## Kite基础教程

[toc]

### Kite基本用法

Kite是Go开发用的一套RPC库。

废话不多少，先来看一套代码，对kite的用法有个基础了解。

**server0.go**

```go
package main
import "github.com/koding/kite"

func main(){
  // new一个kite
  k := kite.New("first", "1.0.0")
  k.Config.Port = 6000
  k.Run()
}
```

以上代码，先创建一个kite对象，然后设置kite服务运行所在的端口，最后run。

**client0.go**

```go
pakcage main
import (
    "fmt"
    "github.com/koding/kite"
)

func main(){
  // 新建一个kite
  k := kite.New("second", "1.0.0")
  // 返回一个client
  client := k.NewClient("http://localhost:6000/kite")
  // 连接kite server
  client.Dial()
  //调用方法
  response, _ := client.Tell("kite.ping")
  fmt.Println(response.MustString())
}
```

以上代码，先新建一个kite，然后通过`NewClient(url)`返回一个client(未连接)的指针，之后调用`Dial()`进行连接。

然后就可以调用远程的函数，通过`Tell()`函数调用方法，Tell的部分函数签名如下：

```go
// param: 方法名称， 方法所需要的参数
func (c *Client) Tell(method string, args ...interface{})
```

上面调用的是kite默认的一个方法，那如果我们想要调用自定义的方法呢？

---

下面来看调用一个自定义方法，还是先来看一套简短的代码。

**server1.go**

```go
package main
import "github.com/koding/kite"

func main(){
  k := kite.New("first", "1.0.0")
  k.Config.Port = 6000
  k.Config.DisableAuthentication = true
  k.HandleFunc("square", func(r *kite.Reqeust)(interface{}, error){
    a := r.Arg.One().MustFloat64()
    return a*a, nil
  })
  
  k.Run()
}
```

以上代码与`server0.go`很像，只是多了一段`k.HandleFunc()`的内容，就是这段代码实现了用户自定义函数的注册。我们具体来看一下该段代码。

`k.HandleFunc()`的函数签名如下：

```go
// param: 方法名， 方法
func (k *Kite) HandleFunc(method string, handler HandlerFunc) *Method
```

显然，它接受两个参数，第一个参数是我们自定义方法的名字，第二个函数是对应的自定义方法。这个方法要是`HandlerFunc`类型——`type HandlerFunc func(*Request) (result interface{}, err error)`。

这样就很明显了，只要我们实现一个满足`HandlerFunc`类型的函数，然后调用`k.HandleFunc()`进行注册即可。

**client1.go**

```go
package main
import (
    "fmt"
    "github.com/koding/kite"
)
func main() {
    k := kite.New("second", "1.0.0")
    client := k.NewClient("http://localhost:6000/kite")
    client.Dial()
    response, _ := client.Tell("square", 4)
    fmt.Println(response.MustFloat64())
}
```

客户端代码与`client0.go`没什么差别，这里不再赘述。

### 服务注册和发现Kontrol

Kite之间可以互相通信。通过Kontrol的服务发现机制，一个kite可以发现其他kites。也就是说，一个kite可以在Kontrol中注册自己，从而让其他kites找到它。

Kontrol本身也是一个Kite，它用于对服务进行注册和鉴权。Kontorl使用etcd作为后端存储。当然，也可用其他数据库。

**server2.go**

```go
package main
import (
    "net/url"
    "github.com/koding/kite"
)

func main(){
  k := k.New("first", "1.0.0")
  k.Config.Port = 6000
  k.HandleFunc("square", func(r *kite.Request)(interface{}, error){
    a := r.Args.One().MustFloat64()
    return a*a, nil
  })
  
  // kite注册到Kontrol
  k.Register(&url.URL{Schema: "http", Host: "localhost:6000/kite"})
  k.Run()
}
```

以上代码与sever1.go很类似，多了一行注册kite到Kontrol的代码,我们重点来看16行。使用的URL参数是其他kites连接该kite的地址，Kontrol会保存这个URL，方便其他kites获取。

`k.Register()`的主要功能是注册kite到Kontrol。注册之后，其他的kites就可以通过`GetKites()`或`WatchKites()`方法来找到这个kite。该函数签名如下：

```go
func (k *Kite) Register(kiteURL *url.URL) (*registerResult, error)

// url.URL结构体
type URL struct {
	Scheme     string
	Opaque     string    
	User       *Userinfo 
	Host       string    
	Path       string    
	RawPath    string    
	ForceQuery bool      
	RawQuery   string    
	Fragment   string    
}
```

**client.go**

```go
package main

import (
    "fmt"
    "github.com/koding/kite"
    "github.com/koding/kite/protocol"
)

func main(){
  k := kite.New("second", "1.0.0")
  //
  kites, _ := k.GetKites(&protocol.KontrolQuery{
    Username:			k.Config.Username,
    Environment:	k.Config.Environment,
    Name:					"first",
  })
  
  // 查找符合条件
  client := kites[0]
  client.Dial()
  response, _ := client.Tell("square", 4)
  fmt.Println(response.MustFloat64())
}
```

我们来分析以上代码。

首先通过`GetKites()`返回服务查询的Kites列表, 然后选择返回的第一个kite，与之建立连接，然后通过`Tell()`调用远程的方法`square`，并传入`square`所需的参数4，最后打印出执行结果。

> 该例是查找和这个kite同一个username和environment，且名为first的所有kites。如果该用户注册了10个名为first的kites，client都能返回。
>
> 消费者可以使用特定的LB算法选择其中一个，我们这里简单起见直接选了第一个

### Reference

- [kite教程](https://www.cnblogs.com/chenny7/p/6846925.html)





