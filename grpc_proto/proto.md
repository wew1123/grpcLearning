# 使用protobuf生成grpc代码

## 1. 编写proto文件
定义rpc服务:
1. 生成代码的包名
2. 服务名
3. 定义函数
4. 定义函数的参数和返回值类型
```
syntax = "proto3" ;// 指定proto版本
package hello_grpc  ;   // 指定默认包名

// 指定golang包名
option go_package = "/hello_grpc";

//定义rpc服务
service HelloService {
// 定义函数
rpc SayHello (HelloRequest) returns (HelloResponse) {}
}

// HelloRequest 请求内容
message HelloRequest {
string name = 1;
string message = 2;
}

// HelloResponse 响应内容
message HelloResponse{
string name = 1;
string message = 2;
}

```

## 2. 生成grpc 代码
使用当前目录所有proto文件生成grpc代码
使用以下命令分别生成普通代码和 gRPC 代码：
```
protoc --go_out=. --go-grpc_out=. *.proto
## --go_out：生成普通 Protobuf 代码（消息结构）。
## --go-grpc_out：生成 gRPC 服务代码（客户端/服务端接口）。
*/

```
指定路径
```
protoc --go_out=./gen --go-grpc_out=./gen *.proto
```
指定heelo.proto 文件生成grpc代码
```
protoc --go_out=. --go-grpc_out=. hello.proto
## --go_out=. 生成hello.pb.go 在当前目录 定义消息结构体
## --go-grpc_out=. 生成gello_groc.pb grpc代码在当前目录 定义函数
```

### 可以写一个bat脚本
```set.bat
protoc --go_out=. --go-grpc_out=. hello.proto
```

## 3.实现proto文件中所定义的方法
hello.proto文件相当于一个接口，其中的函数需要在服务端自己实现

去server文件夹下新建grpc_server.go文件 并在其中实现sayHello方法
```
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/grpclog"
	"grpcLearning/grpc_proto/hello_grpc"
	"net"
)

// HelloServer1 得有一个结构体，需要实现这个服务的全部方法,叫什么名字不重要
type HelloServer1 struct {
}

func (HelloServer1) SayHello(ctx context.Context, request *hello_grpc.HelloRequest) (pd *hello_grpc.HelloResponse, err error) {
	fmt.Println("入参：", request.Name, request.Message)
	pd = new(hello_grpc.HelloResponse)
	pd.Name = "你好"
	pd.Message = "ok"
	return
}

func main() {
	// 监听端口
	listen, err := net.Listen("tcp", ":8080")
	if err != nil {
		grpclog.Fatalf("Failed to listen: %v", err)
	}

	// 创建一个gRPC服务器实例。
	s := grpc.NewServer()
	server := HelloServer1{}
	// 将server结构体注册为gRPC服务。
	hello_grpc.RegisterHelloServiceServer(s, &server)
	fmt.Println("grpc server running :8080")
	// 开始处理客户端请求。
	err = s.Serve(listen)
}

```