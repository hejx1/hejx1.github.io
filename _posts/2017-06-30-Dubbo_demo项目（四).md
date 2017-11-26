---
layout: post
title: Dubbo demo项目 (四) 定义业务中心配置文件
date: 2017-07-03
tags: Dubbo
---

### 业务实现类

实现remoteapi中的IMessage接口

### 定义 Message 接口实现类

```
package cn.hejx.service.impl;

import cn.hejx.IMessage;
import org.springframework.stereotype.Component;

/**
 * Created by 追风少年
 *
 * @email doubihah@foxmail.com
 * @create 2017-06-30 16:07
 **/
@Component
public class MessageImpl implements IMessage{

    @Override
    public String echo(String msg) {
        return "ECHO:" + msg;
    }

}
```
此时 **MessageImpl** 程序类定义的 Bean 的名称为 **messageImpl** ,首字母小写

### 定义业务中心配置文件

1. 在一个真实的项目之中必须要去考虑各种可能出现的不同的项目环境，所以一般会建立一个profiles目录，在这个目录下我们建立若干个子目录，例如：dev、beta、product
- “src/main/profiles/dev” ：描述开发环境下的相关属性
- “src/main/profiles/beta” ：描述测试环境下的相关属性
- “src/main/profiles/product” ：描述生产环境下的相关属性

2. 由于本次项目使用了 logback 日志组件,所以还需要有一个 logback.xml 的配置文件存在,这个文件可以直接保存在 “src/main/profiles/{dev,beta,product}” 目录下
3. 在 “src/main/profiles/{dev,beta,product}/config” 目录,里面保存所有的 *.properties 文件
4. 定义一个 dubbo.properties 文件,主要的功能是进行 dubbo 相关的配置,而在实际开发之中还需要定义一个 database.properties 文件,进行数据库的连接项的信息配置

建立一个 “src/main/resources” 目录，在这个目录之中一定要有一个 META-INF 的目录，里面可以保存具体的配置文件，例如：如果现在需要定义的是所有的 Spring 配置文件，那么可以将其目录设置为“src/main/resources/META-INF/spring”

> 默认情况下 Dubbo 在进行配置文件加载的时候使用的就是 META-INF 作为加载配置文件的标记

将之前建立的 resources 目录和 profiles/xx 目录设置为资源根目录

修改 service/pom.xml 配置文件,重点在于构建 profile 编译的相关处理操作

1.增加一个环境属性,描述 profile 的路径

```
<properties>
    <profiles.dir>src/main/profiles</profiles.dir>
    <maven.jar.plugin.version>3.0.2</maven.jar.plugin.version>
</properties>
```
2.在 pom.xml 里面追加 profiles 和 resources 定义


```
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <profile.dir>${profiles.dir}/dev</profile.dir>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>beta</id>
        <properties>
            <profile.dir>${profiles.dir}/beta</profile.dir>
        </properties>
        <activation>
            <activeByDefault>false</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>product</id>
        <properties>
            <profile.dir>${profiles.dir}/product</profile.dir>
        </properties>
        <activation>
            <activeByDefault>false</activeByDefault>
        </activation>
    </profile>
</profiles>

<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
    <resource>
        <directory>${profile.dir}</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```
3.在 “src/main/resources/META-INF/spring” 目录下创建 spring-common.xml 配置文件


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置 Annotation 扫描包 -->
    <context:component-scan base-package="cn.hejx.service" />
    <!-- 定义所有要导入的属性文件的路径 -->
    <context:property-placeholder location="classpath:config/*.properties" />

</beans>
```
4.编辑 dubbo.properties 配置文件,追加相关的信息

```
# 定义当前业务中心操作业务的名称
dubbo.application.name=hejx-service
# 定义dubbo的注册中心的地址项,本次利用ZooKeeper进行的dubbo数据保存
dubbo.registry.address=zookeeper://192.168.2.233:2181
# 定义dubbo在哪个端口上要进行服务的发布,10000以上随便写
dubbo.protocol.port=19372
# 要进行的是远程调用,那么就一定会有一个超时时间的问题
dubbo.provider.timeout=3000
# 定义远程服务的操作版本
dubbo.interface.version=dev
```

5.在 “src/main/resources/META-INF/spring” 目录下创建 spring-dubbo.xml 配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
         http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 定义当前应用的名称,主要是用于注册中心的信息保存,这个名称可以任意填写 -->
    <dubbo:application name="${dubbo.application.name}" />
    <!-- 定义dubbo注册中心的地址 -->
    <dubbo:registry protocol="zookeeper" address="${dubbo.registry.address}" />

    <!-- 定义dubbo所在的服务,执行时暴露给客户端的端口 -->
    <dubbo:protocol name="dubbo" port="${dubbo.protocol.port}" />

    <!-- 定义远程服务提供者操作的超时时间 -->
    <dubbo:provider timeout="${dubbo.provider.timeout}" />

    <!-- 定义dubbo的远程服务的接口,这个接口定义的时候需要考虑到一个版本问题 -->
    <dubbo:service interface="cn.hejx.remoteapi.IMessage" ref="messageImpl" version="${dubbo.interface.version}" />

</beans>
```
6.logback.xml 配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 配置IP地址 -->
    <!--<conversionRule conversionWord="ip" converterClass="com.xyk.util.log4j.IpConvert" />-->
    <!-- Console 输出格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%date{yyyy-MM-dd HH:mm:ss} %boldGreen | %highlight(%-5level) | %boldYellow(%thread) | %boldGreen(%logger) | %msg%n"/>

    <!-- 文件输出格式 -->
    <property name="FILE_LOG_PATTERN"
              value="===%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger Line:%-3L - %msg%n"/>

    <!-- Console 输出设置 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>
                ${CONSOLE_LOG_PATTERN}
            </pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
    </appender>

    <!--
       说明：
       1、日志级别及文件
           日志记录采用分级记录，级别与日志文件名相对应，不同级别的日志信息记录到不同的日志文件中
           例如：error级别记录到log_error_xxx.log或log_error.log（该文件为当前记录的日志文件），而log_error_xxx.log为归档日志，
           日志文件按日期记录，同一天内，若日志文件大小等于或大于2M，则按0、1、2...顺序分别命名
           例如log-level-2013-12-21.0.log
           其它级别的日志也是如此。
       2、文件路径
           若开发、测试用，在Eclipse中运行项目，则到Eclipse的安装路径查找logs文件夹，以相对路径../logs。
           若部署到Tomcat下，则在Tomcat下的logs文件中
       3、Appender
           FILEERROR对应error级别，文件名以log-error-xxx.log形式命名
           FILEWARN对应warn级别，文件名以log-warn-xxx.log形式命名
           FILEINFO对应info级别，文件名以log-info-xxx.log形式命名
           FILEDEBUG对应debug级别，文件名以log-debug-xxx.log形式命名
           stdout将日志信息输出到控制上，为方便开发测试使用
    -->
    <contextName>dubboDemo</contextName>
    <property name="LOG_PATH" value="log/" />
    <!--设置系统日志目录-->
    <property name="APPDIR" value="service" />

    <!-- 日志记录器，日期滚动记录 -->
    <appender name="FILEERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${LOG_PATH}/${APPDIR}/log_error.log</file>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 归档的日志文件的路径，例如今天是2013-12-21日志，当前写的日志文件路径为file节点指定，可以将此文件与file指定文件路径设置为不同路径，从而将当前日志文件或归档日志文件置不同的目录。
            而2013-12-21的日志文件在由fileNamePattern指定。%d{yyyy-MM-dd}指定日期格式，%i指定索引 -->
            <fileNamePattern>${LOG_PATH}/${APPDIR}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 除按日志记录之外，还配置了日志文件不能超过2M，若超过2M，日志文件会以索引0开始，
            命名日志文件，例如log-error-2013-12-21.0.log -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>2MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- 追加方式记录日志 -->
        <append>true</append>
        <!-- 日志文件的格式 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>utf-8</charset>
        </encoder>
        <!-- 此日志文件只记录error级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>error</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 日志记录器，日期滚动记录 -->
    <appender name="FILEWARN" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${LOG_PATH}/${APPDIR}/log_warn.log</file>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 归档的日志文件的路径，例如今天是2013-12-21日志，当前写的日志文件路径为file节点指定，可以将此文件与file指定文件路径设置为不同路径，从而将当前日志文件或归档日志文件置不同的目录。
            而2013-12-21的日志文件在由fileNamePattern指定。%d{yyyy-MM-dd}指定日期格式，%i指定索引 -->
            <fileNamePattern>${LOG_PATH}/${APPDIR}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 除按日志记录之外，还配置了日志文件不能超过2M，若超过2M，日志文件会以索引0开始，
            命名日志文件，例如log-error-2013-12-21.0.log -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>2MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- 追加方式记录日志 -->
        <append>true</append>
        <!-- 日志文件的格式 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>utf-8</charset>
        </encoder>
        <!-- 此日志文件只记录warn级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 日志记录器，日期滚动记录 -->
    <appender name="FILEINFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${LOG_PATH}/${APPDIR}/log_info.log</file>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 归档的日志文件的路径，例如今天是2013-12-21日志，当前写的日志文件路径为file节点指定，可以将此文件与file指定文件路径设置为不同路径，从而将当前日志文件或归档日志文件置不同的目录。
            而2013-12-21的日志文件在由fileNamePattern指定。%d{yyyy-MM-dd}指定日期格式，%i指定索引 -->
            <fileNamePattern>${LOG_PATH}/${APPDIR}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 除按日志记录之外，还配置了日志文件不能超过2M，若超过2M，日志文件会以索引0开始，
            命名日志文件，例如log-error-2013-12-21.0.log -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>2MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- 追加方式记录日志 -->
        <append>true</append>
        <!-- 日志文件的格式 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>utf-8</charset>
        </encoder>
        <!-- 此日志文件只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <logger name="org.springframework" level="WARN" />
    <logger name="org.hibernate" level="WARN" />

    <!-- 生产环境下，将此级别配置为适合的级别，以免日志文件太多或影响程序性能 -->
    <!--这里改level 生产环境改成ERROR 开发环境为INFO-->
    <root level="INFO">
        <appender-ref ref="FILEERROR" />
        <appender-ref ref="FILEWARN" />
        <appender-ref ref="FILEINFO" />

        <!-- 生产环境将请stdout,testfile去掉 -->
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

### 项目结构图

这个图片有错误:profiles文件夹应该是在src/main文件夹下

![image](http://note.youdao.com/yws/api/personal/file/WEBeff68358c8befe040ebcefaf64b72ad6?method=download&shareKey=4761642d359602519303ca916478ae57)

### 小结

此时一个 Dubbo 业务中心就已经成功的实现了,之后就是调试和运行测试的的过程了