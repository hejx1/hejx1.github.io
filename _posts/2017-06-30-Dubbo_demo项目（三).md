---
layout: post
title: Dubbo demo项目 (三) 搭建业务中心项目
date: 2017-07-03
tags: Dubbo
---

# 业务中心

1. 业务中心是整个 Dubbo 的核心所在
2. 业务中心的搭建需要配置许多的开发包



### 搭建业务中心的开发环境

1. 首先建立一个 service 的项目，作为业务中心的存在：cn.hejx.service
2. 修改 parent/pom.xml 配置文件导入所需要 Dubbo 的开发类库

### 修改 parent/pom.xml 配置文件

主要是新增开发包的属性配置

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
    <modules>
        <module>remoteapi</module>
        <module>service</module>
    </modules>

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

        <!-- 定义所有使用的开发包的版本 -->

        <dubbo.version>2.5.3</dubbo.version>    <!-- 定义要使用的dubbo开发版本 -->
        <!-- 定义要使用的zookeeper开发版本,注意这个版本必须与服务器上的zookeeper版本一致 -->
        <zookeeper.version>3.4.9</zookeeper.version>
        <!-- ZkClient 在整个项目里面是Dubbo要使用的开发包，他描述的是Zookeeper客户端的操作处理 -->
        <zkclient.version>0.10</zkclient.version>

        <spring.version>4.3.5.RELEASE</spring.version> <!-- 定义要使用的Spring版本 -->
        <logback.version>1.2.3</logback.version> <!-- 日志处理包 -->
        <slf4j.version>1.7.25</slf4j.version> <!-- 定义日志的支持包 -->
        <!--<aspectj.version>1.8.10</aspectj.version> &lt;!&ndash; Aspectj 面向切面开发包 &ndash;&gt;-->
        <commons.beanutils.version>1.9.3</commons.beanutils.version> <!-- commons.beanutils开发包 -->
        <commons-lang3.version>3.5</commons-lang3.version> <!-- commons-lang3开发包 -->
        <netty.version>4.1.9.Final</netty.version> <!-- netty -->
        <javassist.version>3.21.0-GA</javassist.version> <!-- Javassist动态编译的开发包 -->
        <junit.version>4.12</junit.version> <!-- 定义要使用的junit版本 -->
        <remoteapi.version>1.0-SNAPSHOT</remoteapi.version>

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
### 修改 service/pom.xml 配置文件

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo-demo</artifactId>
        <groupId>cn.hejx</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <artifactId>service</artifactId>
    <packaging>jar</packaging>
    <name>service</name>
    <url>http://maven.apache.org</url>
    
    <properties>
        <maven.jar.plugin.version>3.0.2</maven.jar.plugin.version>
    </properties>

    <dependencies>
        <!-- 导入项目之中要使用的公共接口项目 -->
        <dependency>
            <groupId>cn.hejx</groupId>
            <artifactId>remoteapi</artifactId>
            <version>${remoteapi.version}</version>
        </dependency>

        <dependency> <!-- 导入Dubbo开发包 -->
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>${dubbo.version}</version>
            <exclusions>
                <exclusion>
                    <artifactId>spring</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        
        <dependency> <!-- 导入Zookeeper开发包 -->
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>${zookeeper.version}</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>${zkclient.version}</version>
        </dependency>

        <!-- 导入所有要使用到的日志组件 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>${logback.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>

        <!-- 导入 commons 组件包 -->
        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>${commons.beanutils.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>${commons-lang3.version}</version>
        </dependency>

        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>${netty.version}</version>
        </dependency>

        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>${javassist.version}</version>
        </dependency>

        <!-- 导入Spring相关配置包 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>



    </dependencies>

    <build>
        <finalName>service</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>${maven.jar.plugin.version}</version>
            </plugin>
        </plugins>
    </build>


</project>
```
### 小结

此时引入了所有开发类库需要的相关程序包，但是这些开发包由于彼此之间会存在有一些其他版本的依赖库的问题，所以在随后进行业务启动的时候还需要去排查冲突的开发包