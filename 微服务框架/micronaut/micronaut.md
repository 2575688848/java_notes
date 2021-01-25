### Java 的原生代码编译

为了将 Java 应用编译成本地可执行代码，我们首先要解决 JVM 和应用框架在运行时的动态性挑战。JVM 提供了灵活的类加载机制，Spring 的依赖注入(DI，Dependency-injection)可以实现运行时动态类加载和绑定。在 Spring 框架中，反射，Annotation 运行时处理器等技术也被广泛应用。这些动态性一方面提升了应用架构的灵活性和易用性，另一方面也降低了应用的启动速度，使得 AOT 原生编译和优化变得非常复杂。



### Micronaut 介绍

Micronaut 与 Spring 框架序不同，Micronaut 提供了编译时的依赖注入和AOP处理能力，并最小化反射和动态代理的使用。Micronaut 应用有着更快的启动速度和更低的内存占用。更加让我们更感兴趣的是 Micronaut 支持与 GraalVM 配合，可以将 Java 应用编译成为本地执行代码全速运行。



