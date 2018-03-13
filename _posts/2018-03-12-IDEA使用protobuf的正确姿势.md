---
layout: post
title: IDEA中使用Protobuf的正确姿势
date: 2018-03-12
tags: [IDEA,protobuf]
---



### protobuf 简介

**protocolbuffer**(以下简称PB)是google 的一种数据交换的格式，它独立于语言，[独立](https://baike.baidu.com/item/%E7%8B%AC%E7%AB%8B/3259)于平台。google 提供了多种语言的实现：java、[c#](https://baike.baidu.com/item/c%23)、[c++](https://baike.baidu.com/item/c%2B%2B)、go 和 [python](https://baike.baidu.com/item/python)，每一种实现都包含了相应语言的[编译器](https://baike.baidu.com/item/%E7%BC%96%E8%AF%91%E5%99%A8)以及库文件。由于它是一种二进制的格式，比使用 [xml](https://baike.baidu.com/item/xml) 进行数据交换快许多。可以把它用于[分布式应用](https://baike.baidu.com/item/%E5%88%86%E5%B8%83%E5%BC%8F%E5%BA%94%E7%94%A8)之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。

### 1.安装 Protobuf Support 插件

依次点击IDEA中的“Preferences”-->"Plugins"-->"Browse repositories"，输入Protobuf 进行查找安装，也可以下载离线安装包，如图所示：

![](/images/posts/ideaProtobuf/1.png)

IDEA plugins地址：[IDEA plugins](https://plugins.jetbrains.com/idea) 

搜索 Protobuf Support 插件，进行下载，下载完成后点击 *Install plugin from disk* 进行安装，如图所示：

![](/images/posts/ideaProtobuf/2.png)

安装完后，重启Intellij IDEA，查看.proto文件，会发现已经支持语法高亮显示。

### 2.将 .proto 文件转成 Java 类

一般的做法是执行protoc命令,依次将.proto文件转成Java类

```
protoc.exe -I=d:/tmp --java_out=d:/tmp d:/tmp/monitor_data.proto
```

也可以写个批处理文件进行处理，具体如下:

```
@echo off

set Path=Path;%cd%
set PATH_DEST=H:\server\gamehall\protocol\src\main\java
set PATH_PO1=H:\server\Protocol\ClientProto
set PATH_PO2=H:\server\Protocol\ServerProto

cd %PATH_PO1%
pushd %PATH_PO1% 
for %%a in (*.proto) do (
    echo %%a
    protoc --java_out=%PATH_DEST% -I=. %%a
)

cd %PATH_PO2%
pushd %PATH_PO2% 
for %%a in (*.proto) do (
    echo %%a
    protoc --java_out=%PATH_DEST% -I=. %%a
)


pause
```

不过gRPC官方推荐了一种更优雅的使用姿势，可以通过maven轻松搞定



pom.xml 增加依赖

```
<properties>
    <protobuf.version>2.5.0</protobuf.version>
    <grpc.version>1.6.1</grpc.version>
</properties>

<dependencies>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-netty</artifactId>
        <version>${grpc.version}</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-protobuf</artifactId>
        <version>${grpc.version}</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-stub</artifactId>
        <version>${grpc.version}</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.google.protobuf</groupId>
        <artifactId>protobuf-java</artifactId>
        <version>${protobuf.version}</version>
    </dependency>
</dependencies>

<build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.5.0</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:${protobuf.version}:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}</pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```



<font style="color:red;">

注意事项：.proto文件需要放置在src/main/proto包下

</font>



最后使用maven进行编译,如图所示:



![](/images/posts/ideaProtobuf/3.png)

现在我们就可以愉快的使用protobuf了。