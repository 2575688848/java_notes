### dubbo支持的协议

Dubbo支持dubbo、rmi、hessian、http、webservice、thrift、redis等多种协议，但是Dubbo官网是推荐我们使用Dubbo协议的。

I/O 框架使用的是 netty

### dubbo协议

连接个数：单连接
连接方式：长连接
传输协议：TCP
传输方式：NIO异步传输
序列化方式：Hessian二进制序列化

1、dubbo默认采用dubbo协议，dubbo协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况
2、他不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低。



### rmi协议

连接个数：多连接
连接方式：短连接
传输协议：TCP
传输方式：同步传输
序列化方式：Java标准二进制序列化
适用范围：传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。
适用场景：常规远程服务方法调用，与原生RMI服务互操作



### hessian协议

基于Hessian的远程调用协议。
连接个数：多连接
连接方式：短连接
传输协议：HTTP
传输方式：同步传输
序列化方式：json
适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。
适用场景：需同时给应用程序和浏览器JS使用的服务。

1、Hessian协议用于集成Hessian的服务，Hessian底层采用Http通讯，采用Servlet暴露服务，Dubbo缺省内嵌Jetty作为服务器实现。
2、Hessian是Caucho开源的一个RPC框架：[http://hessian.caucho.com](http://hessian.caucho.com/)，其通讯效率高于WebService和Java自带的序列化。



### http协议

连接个数：多连接
连接方式：短连接
传输协议：HTTP
传输方式：同步传输
序列化：表单序列化
适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。
适用场景：需同时给应用程序和浏览器JS使用的服务。



### thrift协议

当前 dubbo 支持的 thrift 协议是对 thrift 原生协议的扩展，在原生协议的基础上添加了一些额外的头信息，比如service name，magic number等。使用dubbo thrift协议同样需要使用thrift的idl compiler编译生成相应的java代码，后续版本中会在这方面做一些增强。