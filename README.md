## 一、功能

cocolian-thrift-protobuf是一个RPC容器和客户端连接池的实现。目前由于这两个功能耦合度较高，都放在一个项目中实现。

RPC容器，支持：
1. 服务注册： 这个版本是注册到zookeeper上，之后会提供对consul的支持。
2. Spring MVC支持，将thrift service实现映射到Spring Controller上，通过controller的name来locate服务。 

客户端提供RPC连接池，支持：
1. 服务发现
2. 负载均衡：在客户端实现。

## 二、技术栈

使用Apache Thrift 作为容器， Google Protocol Buffer 作为输入输出。相对于dubbo、 纯Apache Thrift等RPC容器，优势在于：

1.  高性能，Apache Thrift是已知RPC容器中性能最好的。
2.  传输效率高， Google Protocol Buffer 的压缩率相对Apache Thrift 的strut 结构 可以节省20% 空间。
3.  可扩展性好，得益于Protobuf优越的兼容性设计，对接口参数进行调整时，对老接口仍然可以保持很好的兼容。

这个引擎是对Apache Thrift 的极简轻量级封装，可靠，易于使用。 和Spring 良好集成，易于开发。

## 三、项目结构

     .
     ├── src/main
     │    ├── gen                       # 由rpc_service.thrift自动编译出来的代码
     │    │    └── ..  
     │    ├── org/cocolian/rpc
     │    │    ├── register            # 服务注册
     │    │    ├── server              # 服务器端的实现框架
     │    │    └── sharder             # 客户端的rpc连接池   
     │    └── resources
     │          ├── META-INF
     │          │     └── srping.factories  
     │          └── rpc_service.thrif
     ├── CHANGELOG.md
     ├── cocolian-thrift-protobuf.iml
     ├── pom.xm
     └── README.md

## 四、关联模块

 这两个模块是用来测试这个模块的服务端和客户端  
jigsaw-rpc-example-client  
jigsaw-rpc-example-server  

## 五、Docker支持

这个模块输出的docker是其他rpc模块的base image，注意：
1. 这个image是基于centos来构建的。 
2. 预装oracle server jre。 

注意：
**这个版本支持jdk1.8.0_152版本，必须先下载这个版本，并解压缩后，放到src/main/docker下，目录为 src/main/docker/jdk1.8.0_152**

TODO： 
1. 进一步优化base image， 删除不必要的模块。 
2. 增加监控和日志收集组件。

