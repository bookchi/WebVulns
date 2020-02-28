## gRPC学习
> 需要了解：
>  - protobuf的一些概念
>  - 编写server和client
### 快速开始
#### gRPC是什么
在gRPC里，client应用调用server上应用的方法，就像调用本地对象一样。这让我们更容易的创建分布式应用。
gRPC理念：定义一个服务，指定能够被远程调用的方法(包括参数和返回类型)。在server端实现这个接口，并运行一个gRPC服务器来处理client调用。在client端拥有一个stub，有像服务端的方法。
感觉很像web诶

gRPC的client和server端可以在多种环境中运行交互。

使用protocol buffers
gRPC 默认使用 protocol buffers，它是Google开源的一套序列化机制。将会使用proto files创建gRPC服务，用protocol buffers消息类型来定义方法的参数和返回类型。
#### 简单示例
下面来看一个Hello World示例，大概功能：
- 通过一个protocol buffers模式，定义一个带有Hello World方法的RPC服务
- 用golang来创建实现了这个接口的server端
- 用golang来访问server

1. 定义服务

首先，使用protobuf定义一个RPC服务。
```protobuf
syntax = "proto3";

option java_package = "io.grpc.examples";

package helloworld;

// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```
2. 生成gRPC代码

然后，使用protocol buffer 编译器 protoc 编译helloworld.proto，生成所需代码。

`protoc -I ../protos ../protos/helloworld.proto --go_out=plugins=grpc：helloworld\`

这生成了 helloworld.pb.go ，包含了我们生成的客户端和服务端类，此外还有用于填充、序列化、提取 HelloRequest 和 HelloResponse 消息类型的类。

3. 编写server端

服务器：实现RPC方法，使RPC在网络上可用（绑定端口的赶脚）
首先，实现RPC方法——实现 Greeter 服务所需要的行为。创建了server结构，实现了SayHello方法。也就实现了Greeter。
```go
// server is used to implement helloworld.GreeterServer.
type server struct {
   pb.UnimplementedGreeterServer
}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
   log.Printf("Received: %v", in.GetName())
   return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}
```
然后，与网络绑定。
```go
const (
  port = "：50051"
)
...
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
4. 编写cient端

客户端的代码很简单，直接首先连接Greeter服务器，然后创建stub。再调用方法即可。
有点像Socket通信
- 建立连接，创建stub
在 gRPC Go 是使用一个特殊的 Dial() 方法来创建频道。
```go
const (
  address     = "localhost：50051"
  defaultName = "world"
)func main() {
  // Set up a connection to the server.
  conn, err ：= grpc.Dial(address)
  if err != nil {
      log.Fatalf("did not connect： %v", err)
  }
  defer conn.Close()
  c ：= pb.NewGreeterClient(conn)
...
}
```
- 调用RPC
现在我们可以联系服务并获得一个 greeting ：
1. 我们创建并填充一个 HelloRequest 发送给服务。
2. 我们用请求调用stub的 SayHello()，如果 RPC 成功，会得到一个填充的 HelloReply ，从其中我们可以获得 greeting。
```go
r, err ：= c.SayHello(context.Background(), &pb.HelloRequest{Name： name})
if err != nil {
      log.Fatalf("could not greet： %v", err)
}
log.Printf("Greeting： %s", r.Message)
}
```
### Reference
- copy自[gRPC官方文档中文版](http://grpc.mydoc.io/?t=60133)
