---
layout: post
title: Dubbo demo项目 (二) 定义远程服务接口
date: 2017-06-30
tags: Dubbo
---

# 定义远程服务接口

远程服务接口实际上指的就是业务接口，但是对于整个项目里面由于需要考虑到开发版本的问题，所以针对于我们的业务接口也需要继承 **parent** 项目中的相关约定

### remoteapi

为了方便进行管理，暂时将远程的业务接口项目定义为 **remoteapi**

1. 建立一个Maven的快速启动项目，项目的整体包名：cn.hejx.remoteapi
2. 修改 pom.xml 配置文件，让其去继承 parent/pom.xml 配置文件
3. 随后就需要进行远程接口的搭建，这次要搭建的远程接口的核心意义在于进行一个echo程序的实现

### pom.xml

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent> <!-- 定义要实现的父项目的名称 -->
        <artifactId>dubbo-demo</artifactId>
        <groupId>cn.hejx</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>remoteapi</artifactId>
    <packaging>jar</packaging>
    <name>remoteapi</name>
    <url>http://maven.apache.org</url>
</project>
```
### IMessage接口

```
package cn.hejx;

/**
 * Created by 追风少年
 *
 * @email doubihah@foxmail.com
 * @create 2017-06-30 14:46
 **/
public interface IMessage {

    /**
     * 实现信息的回应处理,回应的信息是在原始信息前追加一些相应的提示返回
     * @param msg 接收的消息
     * @return 处理后的 Echo 数据
     */
    public String echo(String msg);

}
```
### 提醒

以后在业务层的实现项目里面以及客户端去调用远程业务项目里面必须去引用 **remoteapi** 的项目