```toml
title = "gRPC使用小结"
slug = "gRPC使用小结"
desc = "gRPC使用小结"
date = "2018-08-12 23:12:12"
update_date = "2018-08-12 23:12:12"
author = "doozz"
thumb = ""
tags = ["go", "grpc", "cobra"]
```


#### Cobra
[Cobra](https://github.com/spf13/cobra)提供简单的接口来创建强大现代的CLI接口,所以本文的gRPC服务采用cobra包来开发。

首先安装cobra

```
go get -u github.com/spf13/cobra/cobra
```

快速创建一个项目：cobra init [name]

```
D:\gopath\bin>cobra.exe init cli
Your Cobra application is ready at
D:\gopath\src\cli

Give it a try by going there and running `go run main.go`.
Add commands to it by running `cobra add [cmdname]`.
```

此时会在src文件夹中生成一个cli文件夹
```
▾ cli/
    ▾ cmd/
        root.go
     LICENSE
     main.go
```

使用 cobra add 可以添加自己的命令，接下来我们执行 cobra add start 。此时会在cmd目录下生成start.go文件
```
cobra add start
start created at D:\gopath\src\cli\cmd\start.go
```


接下来我们跑一下main.go 并加上我们刚生成的命令 start
```
go run main.go start
Using config file: C:\Users\Administrator\.cli.yaml
start called
```
可以看到这个配置的目录不在我们的项目文件下，于是我们可以在项目文件夹下创建一个cli.yaml配置文件
````
cli.yaml:
test:
  say: hello word
````

并且修改start.go
````
var startCmd = &cobra.Command{
	Use:   "start",
	Short: "A brief description of your command",
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("start called")
		fmt.Println(viper.GetString("test.say"))
	},
}

func init() {
	rootCmd.AddCommand(startCmd)
}
````

然后重新执行go run main.go start，只是这次我们要指定config位置
````
D:\gopath\src\cli>go run main.go start --config ./cli.yaml
Using config file: ./cli.yaml
start called
hello word
````

以上就是基本的cobra操作。接下来我们来学习grpc

#### gRPC

[gRPC](https://github.com/grpc/grpc.git) 一开始由 google 开发，是一款语言中立、平台中立、开源的远程过程调用(RPC)系统。首先安装gRPC,跟protobuf
````
go get google.golang.org/grpc

go get -u github.com/golang/protobuf/{proto,protoc-gen-go}

export PATH=$PATH:$GOPATH/bin
````

再windows上安装 protoc,[下载地址](https://github.com/google/protobuf/releases)。下载完成后解压其中的protoc.exe。进入src\github.com\golang\protobuf\protoc-gen-go文件夹执行
go build。会在 bin目录生成 protoc-gen-go.exe。有了这两个文件就可以编译.proto文件了

在原来的目录新加docs文件夹，然后编写我们的第一个proto文件 heardbeat.proto
````
syntax = "proto3";
package heardbeat;

message pingReq {
    string ping = 1;
}

message pongRep {
    string pong = 1; 
    string message = 2;
}

//定义一个服务
service HeardbeatService {
    rpc GetShortUrl(pingReq) returns (pongRep);
}
````

编译proto文件
````
D:\gopath\bin>protoc --go_out=plugins=grpc:. ./protoc/heardbeat.proto
````

在原来的目录新加heardbeat文件夹,把heardbeat.pb.go文件放入
修改start.go
````
var startCmd = &cobra.Command{
	Use:   "start",
	Short: "A brief description of your command",
	Run: func(cmd *cobra.Command, args []string) {
		var err error
		var grpcServer *grpc.Server
		defer func() {
			grpcServer.Stop()
		}()

		log.Println("heardbeat grpc server starting...")

		lis, err := net.Listen("tcp", fmt.Sprintf("%s:%d", viper.GetString("rpc.host"), viper.GetInt("rpc.port")))
		if err != nil {
			log.Fatalf("heardbeat grpc initialize failed. %s\n", err.Error())
		}

		grpcServer = grpc.NewServer()
		RegisterHeardbeatSrvServer(grpcServer, new(Server))
		grpcServer.Serve(lis)
	},
}

func init() {
	rootCmd.AddCommand(startCmd)
}
````

在heardbeat目录新加service.go文件
````
package heardbeat

import (
	context "golang.org/x/net/context"
)

type Server struct{}

func (s *Server) Check(ctx context.Context, req *PingReq) (*PongRep, error) {
	resp := &PongRep{}
	if req.Ping == "ping" {
		resp.Pong = "PONG"
		resp.Message = "OK"
	} else {
		resp.Pong = ""
		resp.Message = "error"
	}

	return resp, nil

}
````

然后启动grpc服务
````
D:\gopath\src\cli>go run main.go start --config ./cli.yaml
Using config file: ./cli.yaml
heardbeat grpc server starting...
````

接下来我们来编写客户端代码

目录结构
```
▾ cli-client/
    ▾ heardbeat/
        heardbeat.pd.go
     main.go
```

````go
package main

import (
	"fmt"
	"log"

	"cli-client/heardbeat"

	"golang.org/x/net/context"
	"google.golang.org/grpc"
)

func main() {
	// Set up a connection to the server.
	conn, err := grpc.Dial("127.0.0.1:9411", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := heardbeat.NewHeardbeatSrvClient(conn)

	// Contact the server and print out its response.
	r, err := c.Check(context.Background(), &heardbeat.PingReq{Ping: "ping"})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	fmt.Println(r)
}

````

然后运行 
````
D:\gopath\src\cli-client>go run main.go
pong:"PONG" message:"OK"
````

以上。
