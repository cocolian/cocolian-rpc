## 一、功能

cocolian-rpc是一个RPC容器和客户端连接池的实现。
- cocolian-rpc-server: 使用Apache Thrift 作为容器，Protobuf Message作为输入输出参数的RPC服务器；
- cocolian-rpc-client: 客户端的实现，提供一个连接池。 
- cocolian-rpc-service: 服务器端和客户端共用的一些服务，比如zookeeper注册参数对象、一些共用的thrift对象等。 
- cocolian-rpc-docker：运行rpc服务器的docker 镜像。 

## 二、cocolian-rpc-server 

使用Apache Thrift 作为RPC Server， Protobuf Message作为输入输出参数。 Apache Thrift作为Server，我们以这个Servce作为基础服务：

```java
service BaseService {
  /**
    * 通用的execute服务。
   **/
    binary execute(1:binary request)
    throws(   1:NotFoundException notFoundException,
              2:SystemException systemException,
              3:UserException userException
    );
  
}
```

在上述定义中， 当客户端向服务器端请求BaseService::execute操作时，Thrift 有如下处理：

- BaseService不会被编码到调用中， 也就是服务名称是FooService， BarService， 对这个调用是没有影响的。 
- execute是在Thrift 编码的头中定义的，所以只要把这个头读出来，就知道调用的是什么方法 。 
- Protobuf Message 可以很容易的转换成binary字节，通过Thrift来传输到服务器端。 处理完成后， 将结果也是表示成ProtobufMessage，传输到客户端。 

而Protobuf Message提供如下比Thrift 的struct更有优势的特性：
1. 压缩率高， 性能更好。 
2. 可扩展性更好。 只要保持序号和类型的一致性，可添加optional的参数。 特别是对enum类型的兼容性支持。 
3. option提供类似java annotation的支持。 


基于上述考虑，我们使用Apache Thrift + Protobuf Message来构建RPC Server。 在服务器端实现，是和Spring Framework集成，核心类： 

- TProtobufProcessor：重写了TProcessor的实现， 从Thrift序列化的字节流中解析出调用的方法名。这个方法名被映射到Spring Bean容器中的Component名字，通过这个Bean名字来调用对应的Component的方法，完成服务调用。 比如 
- Controller： 这个接口即用来支持TProtobufProcessor调用的Component的基类，提供process接口，供其调用。 
- BaseController： Controller实现的公共基类，封装一些通用的异常处理。 
- RpcServerConfiguration： 用来支持将RPC Server注册到Zookeeper上以及启动Server的配置。 支持自动配置。 


## 三、cocolian-rpc-client 

和rpc-server相对应的客户端实现， 主要是提供一个Transport连接池，这是基于Apache Commons Pools实现的RPC 连接池。 
     ├── TransportManager 连接池接口  
           └── PooledTransport： 基于Apache Commons Pools 的连接池声明   
                 └── AbstractTransportPool 实现基本的连接池接口   
                          └──RefreshableTransportPool 支持按照一定的策略来更新链接的连接池   
                                    └──BasicTransportPool 最基本的round-robin轮询方式的可更新的链接池，即按照zk上注册的顺序，依次选择服务器。    

## 四、cocolian-rpc-docker

提供一个支持rpc server的docker基础镜像。 注意：
1. 这个image是基于centos来构建的。 
2. 预装oracle server jre。 

注意：
**这个版本支持jdk1.8.0_152版本，必须先下载这个版本，并解压缩后，放到src/main/docker下，目录为 src/main/docker/jdk1.8.0_152**

TODO： 
1. 进一步优化base image， 删除不必要的模块。 
2. 增加监控和日志收集组件。
3. 
