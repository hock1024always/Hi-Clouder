# ```GRPC```是什么

## 定义

```RPC```(远程过程调用)，是指项目在使用微服务架构的时候，用来屏蔽分布式计算机调用的具体细节，使得调用远程函数就像调用本地函数一样方便。```GRPC```就是一个高性能的，开源的```RPC```框架。

官网：[gRPC](https://grpc.io/)

中文文档：[gRPC 官方文档中文版](https://doc.oschina.net/grpc)

## 用途

```GRPC```是采用“服务定义”的思想，通过某种**与语言无关**的方式来描写服务，当定义好这些服务之后，```GRPC```会屏蔽这些细节，客户端直接**调用某个接口即可**。因此，定义编码和解码方式，并且做到与语言无关就很重要了。样例如下：

![image-20241024071752939](C:\Users\BuBu\AppData\Roaming\Typora\typora-user-images\image-20241024071752939.png)

## 工具

```GRPC```使用```Protocol Buffss```来实现数据的序列化,在发送和接受的过程中，```Protocol Buffss```完成对应的编码和解码工作。

优势：

1. 序列化后体积相比```Json```和```XML```很小，适合网络传输
2. 支持跨平台多语言
3. 消息格式升级和兼容性还不错
4. 序列化反序列化速度很快

下载地址：[ProtoBuf](https://github.com/protocolbuffers/protobuf/releases)

## 环境配置

在项目终端使用下面三个指令：

```
go get google.golang.org/grpc

go install google.golang.org/protobuf/cmd/protoc-gen-go@latest

go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

注意在系统变量中要配置正确的```GOPATH```和```Path```

检查环境：

```
protoc-gen-go --version
protoc-gen-go-grpc --version
```

# 使用```ProtoBuf```的流程方法

一个简单的方法就是生成一个这样的文件```hello.proto```

```protobuf
//约定使用proto3版本语法
syntax = "proto3";

//规定最后生成的.go文件在什么位置
//pb相当于是一个两端的通讯规范 业务逻辑需要再他生成的.go文件中实现
option go_package = ".;service";//文件位置；包名

//定义一个服务 这个服务中需要有一个方法 方法可以接受客户端参数 再返回服务端响应
service HelloService {//定义服务名
  //发送一个HelloRequest请求 得到一个HelloResponse响应
  rpc SayHello (HelloRequest) returns (HelloResponse){}
}

//定义消息的参数
message HelloRequest {//message可以理解为一个声明结构体的
  string name = 1; //参数和序号 序号是变量在message中的位置
  int64 age = 2;
}

message HelloResponse {
  string message = 1;
}
```

之后使用下面的两个指令分别生成对应的.go文件（使用时记得切换到```proto```文件的目录下）

```go
//protoc --go_out=. hello.proto //生成.go文件
//protoc --go-grpc_out=. hello.proto //生成grpc相关的文件
```

之后在```hello_grpc.pb.go```文件中找到对应的方法写业务逻辑即可

# 语法

## message

message关键字生成一个消息体，在这个消息体会在对应的.go文件中生成一个结构体

消息体的每一个字段都必须标识消息号

### 字段规则

required：在```proto3```中被删去

optional: 在```proto3```中每个字段都默认是这个 可选字段

repeated：声明切片类型时使用这个关键字

### 嵌套消息

```protobuf
//定义消息的参数
message HelloRequest {//message可以理解为一个声明结构体的
   message Person {
     string name = 1; //参数和序号 序号是变量在message中的位置
     int64 age = 2;
     repeated string hobbies = 3;
   }
   repeated Person info = 1;
}
```

外部调用

```protobuf
message PersonMessage{
	PersonInfo.Person info = 1;
}
```

### 服务定义

如果想将消息类型应用到```RPC```系统中，可以在```.proto```文件中设置一个```RPC```服务接口，样例如下

```protobuf
service HelloService {//定义服务名
  //发送一个HelloRequest请求 得到一个HelloResponse响应
  rpc SayHello (HelloRequest) returns (HelloResponse){}
}
```

本质上，使用proto生成的两个文件都是约束性的接口，客户端和服务端都是通过调用接口来实现方法的使用（服务端实现，客户端调用）

