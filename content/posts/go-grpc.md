---
title: "Go使用gRPC进行通信"
description: RPC是远程过程调用的简称，是分布式系统中不同节点间流行的通信方式。
date: 2022-05-26T14:17:33+08:00
draft: false
slug: go-grpc
image:
categories:
  - Go
tags:
  - Go
  - gRPC
---

# 安装`gRPC`和`Protoc`

## 安装`protobuf`

```bash
go get -u google.golang.org/protobuf
go get -u google.golang.org/protobuf/proto
go get -u google.golang.org/protobuf/protoc-gen-go
```



## 安装`Protoc`

```shell
# 下载二进制文件并添加至环境变量
https://github.com/protocolbuffers/protobuf/releases
```

安装`Protoc`插件`protoc-gen-go`

```shell
# go install 会自动编译项目并添加至环境变量中
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

```shell
#protoc-gen-go 文档地址
https://developers.google.com/protocol-buffers/docs/reference/go-generated
```

# 创建`proto`文件并定义服务

## 新建 `task.proto`文件

```shell
touch task.proto
```

## 编写`task.proto`

```protobuf
// 指定proto版本
syntax = "proto3";
// 指定包名
package task;
// 指定输出 go 语言的源码到哪个目录和 包名
// 主要 目录和包名用 ; 隔开
// 将在当前目录生成 task.pb.go
// 也可以只填写 "./",会生成的包名会变成 "----"
option go_package = "./;task";

// 指定RPC的服务名
service TaskService {
    // 调用 AddTaskCompletion 方法
    rpc AddTaskCompletion(request) returns (response);
}

// RPC TaskService服务,AddTaskCompletion函数的请求参数,即消息
message request {
    uint32 id = 1;//任务id
    string module = 2;//所属模块
    int32  value = 3;//此次完成值
    string guid = 4;//用户id
}
// RPC TaskService服务,TaskService函数的返回值,即消息
message response{

}
```

## 使用`Protoc`来生成Go代码

```bash
protoc --go_out=. --go-grpc_out=. <要进行生成代码的文件>.proto
# example
protoc --go_out=. --go-grpc_out=. .\task.proto
```

这样生成会生成两个`.go`文件，一个是对应消息`task.pb.go`，一个对应服务接口`task_grpc.pb.go`。

在`task_grpc.pb.go`中，在我们定义的服务接口中，多增加了一个私有的接口方法：
`mustEmbedUnimplementedTaskServiceServer()`

# 使用`Go`监听`gRPC`服务端及客户端

## 监听服务端

并有生成的一个`UnimplementedTaskServiceServer`结构体来实现了所有的服务接口。因此，在我们自己实现的服务类中，需要继承这个结构体，如：

```go
// 用于实现grpc服务 TaskServiceServer 接口
type TaskServiceImpl struct {
    // 需要继承结构体 UnimplementedServiceServer 或mustEmbedUnimplementedTaskServiceServer
	task.mustEmbedUnimplementedTaskServiceServer()
}

func main() {
	// 创建Grpc服务
	// 创建tcp连接
	listener, err := net.Listen("tcp", ":8082")
	if err != nil {
		fmt.Println(err)
		return
	}
	// 创建grpc服务
	grpcServer := grpc.NewServer()
	// 此函数在task.pb.go中,自动生成
	task.RegisterTaskServiceServer(grpcServer, &TaskServiceImpl{})
	// 在grpc服务上注册反射服务
	reflection.Register(grpcServer)
	// 启动grpc服务
	err = grpcServer.Serve(listener)
	if err != nil {
		fmt.Println(err)
		return
	}

}

func (s *TaskServiceImpl) AddTaskCompletion(ctx context.Context, in *task.Request) (*task.Response, error) {
	fmt.Println("收到一个Grpc 请求, 请求参数为", in.Guid)
	r := &task.Response{
	}
	return r, nil
}

```

然后在`TaskService`上实现我们的服务接口。


## 客户端

```go
	conn, err := grpc.Dial("127.0.0.1:8082", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	// 创建grpc客户端
	client := task.NewTaskServiceClient(conn)
	// 创建请求
	req := &task.Request{
		Id:     1,
		Module: "test",
		Value:  3,
		Guid:   "test",
	}
	// 调用rpc TaskService AddTaskCompletion函数
	response, err := client.AddTaskCompletion(context.Background(), req)
	if err != nil {
		log.Println(err)
		return
	}
	log.Println(response)
```

[本文参考](https://www.cnblogs.com/whuanle/p/14588031.html)
