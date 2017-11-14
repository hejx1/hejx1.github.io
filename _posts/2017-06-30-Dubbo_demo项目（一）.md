---
layout: post
title: Dubbo demo项目 (一) 定义父项规则
date: 2017-06-30
tags: Dubbo
---

# 定义父项规则

在本次的开发之中，如果想要实现一个Dubbo项目的编写，需要有三个项目：远程接口、业务实现、客户端，为了保证整个项目的整体风格一致，以及所有开发包的版本的统一管理，需要定义一个父项目，在这个项目里面主要的目的是定义一些基础的插件，以及版本的属性信息，同时其他的项目里面都要求继承这个项目。

## 所有项目的搭建使用的是Maven的方式管理

> 1. 创建一个parent项目，这个项目作为整个的公共的项目约束（使用maven创建项目,使用quickstart默认风格）
> 2. 修改项目的pom.xml配置文件

### 配置项目打包形式
```
<!-- 该项目应该作为一个父项目的pom.xml的配置文件,所以必须将 packaging 设置为pom -->
<packaging>pom</packaging>
```
### 配置jdk版本

maven是个项目管理工具，如果我们不告诉它我们的代码要使用什么样的jdk版本编译的话，它就会用maven-compiler-plugin默认的jdk版本来进行处理，这样就容易出现版本不匹配的问题，以至于可能导致编译不通过的问题。例如代码中要是使用上了jdk1.7的新特性，但是maven在编译的时候使用的是jdk1.6的版本，那这一段代码是完全不可能编译成.class文件的。为了处理这一种情况的出现，在构建maven项目的时候，我习惯性第一步就是配置maven-compiler-plugin插件

```
<jdk.version>1.8</jdk.version> <!-- 定义要使用的jdk的开发版本 -->
<!-- 定义程序编译的开发版本，这样整体的项目会从JDK1.5变为JDK1.8 -->
<maven.compiler.plugin.version>3.6.0</maven.compiler.plugin.version>
```
### 定义源代码生成的插件版本信息

```
<!-- 定义源代码生成的插件版本信息 -->
<maven.source.plugin.version>3.0.1</maven.source.plugin.version>
```
### 定义编译时的配置项

```
<build><!-- 定义编译时的配置项 -->
<finalName>parent</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>${maven.source.plugin.version}</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <goals>
                        <goal>jar</goal> <!-- 排除所有的jar文件 -->
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>${maven.compiler.plugin.version}</version>
            <configuration>
                <source>${jdk.version}</source> <!-- 定义源代码的开发版本 -->
                <target>${jdk.version}</target> <!-- 定义生成class文件的编译版本 -->
                <encoding>${project.build.sourceEncoding}</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```
### 最后附上完整的pom.xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- 此配置文件作为父项pom.xml -->

    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.hejx</groupId>
    <artifactId>dubbo-demo</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <jdk.version>1.8</jdk.version> <!-- 定义要使用的jdk的开发版本 -->
        <!-- maven是个项目管理工具，如果我们不告诉它我们的代码要使用什么样的jdk版本编译的话，
        它就会用maven-compiler-plugin默认的jdk版本来进行处理，这样就容易出现版本不匹配的问题，
        以至于可能导致编译不通过的问题。例如代码中要是使用上了jdk1.7的新特性，但是maven在编译的时候使用的是jdk1.6的版本，
        那这一段代码是完全不可能编译成.class文件的。为了处理这一种情况的出现，
        在构建maven项目的时候，我习惯性第一步就是配置maven-compiler-plugin插件。 -->
        <maven.compiler.plugin.version>3.6.0</maven.compiler.plugin.version>
        <!-- 源代码打包插件 -->
        <maven.source.plugin.version>3.0.1</maven.source.plugin.version>
    </properties>

    <build><!-- 定义编译时的配置项 -->
        <finalName>parent</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>${maven.source.plugin.version}</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <goals>
                            <goal>jar</goal> <!-- 排除所有的jar文件 -->
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven.compiler.plugin.version}</version>
                <configuration>
                    <source>${jdk.version}</source> <!-- 定义源代码的开发版本 -->
                    <target>${jdk.version}</target> <!-- 定义生成class文件的编译版本 -->
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

