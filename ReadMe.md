# grpc 学习

## 原生rpc
### 服务端
```
package main

import (
	"fmt"
	"net"
	"net/http"
	"net/rpc"
)

type Server struct {
}
type Req struct {
	Num1 int
	Num2 int
}
type Res struct {
	Num int
}

func (s Server) Add(req Req, res *Res) error {
	res.Num = req.Num1 + req.Num2
	fmt.Printf("请求来了！！23")
	return nil
}

func main() {
	// 注册rpc服务
	rpc.Register(new(Server))
	rpc.HandleHTTP()
	listen, err := net.Listen("tcp", ":8080")
	if err != nil {
		fmt.Println(err)
		return
	}
	http.Serve(listen, nil)
}
```
服务端通过Server结构体注册一个grpc服务
为其绑定了add方法
使用rpc.Register(new(Server))注册
listen,err = net.Listen("tcp","8080") 
http.Serve(listen,nil) 开启监听

### 客户端
客户端使用
client,err = rpc.DialHttp("tcp",":8080") 获取调用句柄
client.Call("Server.Add",req,&res) 传参 并获取返回值
```
package main

import (
	"fmt"
	"net/rpc"
)

type Req struct {
	Num1 int
	Num2 int
}
type Res struct {
	Num int
}

func main() {
	req := Req{1, 2}
	client, err := rpc.DialHTTP("tcp", ":8080")
	if err != nil {
		fmt.Println(err)
		return
	}
	var res Res
	client.Call("Server.Add", req, &res)
	fmt.Println(res)
}

```
### 原生rpc的缺点
1. 编写相对复杂，需要自己关注实现过程
2. 没有代码提示，容易写错

