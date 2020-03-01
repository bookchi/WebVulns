
## gRPC基础教程
**...未完...**

- [gRPC基础教程](#grpc基础教程)
  - [1 gRPC基本概念](#1-grpc基本概念)
  - [2 简单示例](#2-简单示例)
    - [2.1 定义服务](#21-定义服务)
    - [2.2 编译proto文件](#22-编译proto文件)
    - [2.3 编写服务端](#23-编写服务端)
    - [2.4 编写客户端](#24-编写客户端)
  - [3 Reference](#3-reference)

> 需要了解：
>
> - protobuf的一些概念
> - 编写server和client

gRPC和kite都是Golang的RPC框架。在xxx红已经讲解了kite框架的一些概念和用法，这篇文章便来介绍gRPC。

### 1 gRPC基本概念
> 这部分比较抽象，先大致看懂即可，也可以跳到简单示例部分，回过头再来看这些

gRPC是有Google公司开发的一款RPC框架，它底层使用protobuf来进行接口定义。我们下面简单的了解一下gRPC。

在gRPC里，client应用调用server上应用的方法，就像调用本地对象一样。这让我们更容易的创建分布式应用。

**gRPC的理念**：首先定义一个rpc服务，指定能够被远程调用的方法。然后在服务器端实现这个接口，并运行一个gRPC服务器来处理client调用。在client端有一个stub，有像服务端的方法。

**使用protocol buffers**：gRPC默认使用*protocol buffers*，它是Google开源的一套序列化机制。将会使用proto files创建gRPC服务，使用protocol buffers消息类型来定义方法的参数和返回类型。

gRPC的client和server端可以在多种环境中运行交互，不过我们这里是Golang的教程。

<img src="http://www.grpc.io/img/grpc_concept_diagram_00.png">

---

### 2 简单示例

上面说了一大堆，却都比较抽象和难以理解，看不懂的话没关系。下面来写一个简单的例子，就能明白gPRC的大致流程和上面的概念。

先来介绍一下这个Hello World示例，大概功能如下：

- 通过protocol buffers模式，定义一个带有Hello World方法的RPC服务
- 使用Golang来创建实现上接口的server端
- 使用Golang写一个client来访问server

#### 2.1 定义服务

**helloworld.proto**

```protobuf
syntax = "proto3"

option java_package = "io.grpc.examples";

package helloworld;

// 定义一个服务Greeter
service Greeter{
	// Greeter服务中包含一个SayHello接口
	rpc SayHello(HelloRequest) returns (HelloReply) {}
}

// 定义HelloReqeust
message HelloRequest{
	string name = 1;
}

// 定义HelloReply
message HelloReply{
	string msg = 1;
}
```

以上代码的主要功能是通过`.proto`文件定义一个RPC服务。该例中是定义一个`Greeter`服务，这个服务需要实现`SayHello()`接口。接口的参数和返回值也都在以上代码中进行了定义，很容易看明白，不再赘述。

#### 2.2 编译proto文件

写完proto代码后，使用protocol buffer编译器`protoc`编译helloworld.proto,会生成我们所需的代码`helloworld.pb.go`。至于如何编译，可以去这里查看：

- [protoc安装教程](https://www.grpc.io/docs/quickstart/go/)
- [下载地址](https://github.com/google/protobuf/releases)

```shell
# 在helloworld.proto文件所在目录下，执行如下protoc命令
$ protoc --go_out=plugins=grpc:. route_guide.proto
```

#### 2.3 编写服务端

**server.go**

```go
package main

type server struct {
   pb.UnimplementedGreeterServer
}

const (
  port = "：50051"
)

func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
   log.Printf("Received: %v", in.GetName())
   return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func main() {
  lis, err ：= net.Listen("tcp", port)
  if err != nil {
      log.Fatalf("failed to listen： %v", err)
  }
  s ：= grpc.NewServer()
  pb.RegisterGreeterServer(s, &server{})
  s.Serve(lis)
}
```



#### 2.4 编写客户端

**client.go**

```go
package main

const (
  address     = "localhost：50051"
  defaultName = "world"
)

func main() {
  // Set up a connection to the server.
  conn, err ：= grpc.Dial(address)
  if err != nil {
      log.Fatalf("did not connect： %v", err)
  }
  defer conn.Close()
  c ：= pb.NewGreeterClient(conn)
	
  r, err ：= c.SayHello(context.Background(), &pb.HelloRequest{Name： name})
	if err != nil {
      log.Fatalf("could not greet： %v", err)
	}
	log.Printf("Greeting： %s", r.Message)
}
```



### 3 Reference 
- [gRPC官方文档](https://www.grpc.io/docs/)
- [gRPC官方文档中文版](http://grpc.mydoc.io/?v=10467&t=58008)
- [protobuf官方文档](https://developers.google.com/protocol-buffers/docs/proto3)
- [protobuf中文博客教程](https://colobu.com/2017/03/16/Protobuf3-language-guide/)
