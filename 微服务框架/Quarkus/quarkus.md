### 简介

Red Hat开源的Java框架Quarkus，定位为GraalVM和OpenJDK HotSpot量身定制的一个Kurbernetes Native Java框架。虽然开源时间较短，但是生态方面也已经达到可用的状态，自身包含扩展框架，已经支持像Netty、Undertow、Hibernate、JWT等框架，足以用于开发企业级应用，用户也可以基于扩展框架自行扩展。

Quarkus定位是一个Native Java的框架，可以将一个 java 项目构建成Native 本地应用。



### 开发环境

开发环境 除了一般的Java开发环境外，你还需要额外安装Graalvm，用于构建Native应用。 Graalvm安装参考：[Install GraalVM](https://www.graalvm.org/docs/getting-started/)



### 构建一个Quarkus应用

[参考文章](https://my.oschina.net/centychen/blog/3049808)

1、在 pom.xml 中增加构建profile配置，如下：

```java
 <profiles>
        <profile>
            <id>native</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>io.quarkus</groupId>
                        <artifactId>quarkus-maven-plugin</artifactId>
                        <version>${quarkus.version}</version>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>native-image</goal>
                                </goals>
                                <configuration>
                                    <enableHttpUrlHandler>true</enableHttpUrlHandler>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
```



使用`mvn package -Pnative`命令构建Native Image，构建完成后，target目录下会存在一个名字为`[project name]-runner`的文件，这个就是应用的可执行文件，你可以拷贝到其它目录运行，运行如下：

```java
./quarkus-simple-example-1.0-SNAPSHOT-runner 
2019-05-15 12:02:31,199 INFO  [io.quarkus] (main) Quarkus 0.14.0 started in 0.012s. Listening on: http://[::]:8080
2019-05-15 12:02:31,201 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]

```

搭建一个 Restful 服务并构建成 Native Image。完成这一步之后，你还可以将 Native Image 构建成 Docker 镜像并使用 Kubernetes 进行部署。