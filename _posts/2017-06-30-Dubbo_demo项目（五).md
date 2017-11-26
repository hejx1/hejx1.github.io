---
layout: post
title: Dubbo demo项目 (五) 启动Dubbo服务
date: 2017-07-04
tags: Dubbo
---

# 尝试启动

现在已经基本上搭建完成了 Dubbo 业务中心,但是如果想要正常的使用 Dubbo 还需要对整体的代码进行调试

### 定义一个 Dubbo 服务的启动程序类：ProviderStartup

```
package cn.hejx.service.impl.demo;

import com.alibaba.dubbo.container.Main;

/**
 * Created by 追风少年
 * service 启动入口
 * @email doubihah@foxmail.com
 * @create 2017-07-03 10:07
 **/
public class ProviderStartup {

    public static void main(String[] args) {
        Main.main(args);    // 启动dubbo服务
    }

}
```
- 能够这样启动的主要的原因在于项目之中已经按照 Dubbo 的默认要求编写了 META-INF
- “Main.main(args)” 在读取的时候会自动的找到指定目录,进行配置文件的加载

### 排除错误

```
log4j:WARN No appenders could be found for logger (com.alibaba.dubbo.common.logger.LoggerFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
org.springframework.beans.factory.BeanDefinitionStoreException: Unexpected exception parsing XML document from file [E:\IdeaProjects\dubbodemo\service\target\classes\META-INF\spring\spring-common.xml]; nested exception is java.lang.IllegalStateException: Context namespace element 'component-scan' and its parser class [org.springframework.context.annotation.ComponentScanBeanDefinitionParser] are only available on JDK 1.5 and higher
	at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(XmlBeanDefinitionReader.java:420)
	at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(XmlBeanDefinitionReader.java:342)
	at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(XmlBeanDefinitionReader.java:310)
	at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:143)
	at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:178)
	at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:149)
	at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:212)
	at org.springframework.context.support.AbstractXmlApplicationContext.loadBeanDefinitions(AbstractXmlApplicationContext.java:113)
	at org.springframework.context.support.AbstractXmlApplicationContext.loadBeanDefinitions(AbstractXmlApplicationContext.java:80)
	at org.springframework.context.support.AbstractRefreshableApplicationContext.refreshBeanFactory(AbstractRefreshableApplicationContext.java:123)
	at org.springframework.context.support.AbstractApplicationContext.obtainFreshBeanFactory(AbstractApplicationContext.java:422)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:352)
	at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:139)
	at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:93)
	at com.alibaba.dubbo.container.spring.SpringContainer.start(SpringContainer.java:50)
	at com.alibaba.dubbo.container.Main.main(Main.java:80)
	at cn.hejx.service.impl.demo.ProviderStartup.main(ProviderStartup.java:14)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)
Caused by: java.lang.IllegalStateException: Context namespace element 'component-scan' and its parser class [org.springframework.context.annotation.ComponentScanBeanDefinitionParser] are only available on JDK 1.5 and higher
	at org.springframework.context.config.ContextNamespaceHandler$1.parse(ContextNamespaceHandler.java:65)
	at org.springframework.beans.factory.xml.NamespaceHandlerSupport.parse(NamespaceHandlerSupport.java:69)
	at org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseCustomElement(BeanDefinitionParserDelegate.java:1297)
	at org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseCustomElement(BeanDefinitionParserDelegate.java:1287)
	at org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.parseBeanDefinitions(DefaultBeanDefinitionDocumentReader.java:135)
	at org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.registerBeanDefinitions(DefaultBeanDefinitionDocumentReader.java:92)
	at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.registerBeanDefinitions(XmlBeanDefinitionReader.java:507)
	at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(XmlBeanDefinitionReader.java:398)
	... 21 more
```
### 问题处理

```
[org.springframework.context.annotation.ComponentScanBeanDefinitionParser] are only available on JDK 1.5 and higher
```

控制台显示ComponentScanBeanDefinitionParser应该运行在 jdk1.5 或者更高级别,现在我们对整个项目的依赖包进行排查

由于我使用的是 IntelliJ IDEA 进行开发

1. 该工具有个Maven Projects窗口，一般在右侧能够找到，如果没有可以从菜单栏打开：View>Tool Windows>Maven Projects
2. 选择要分析的maven module(idea的module相当于eclipse的project),右击show dependencies,会出来该module的全部依赖关系图，非常清晰细致
3. 在图里选中一个artifact,则所有依赖该artifact的地方都会一起连带出来突出显示，如果有不同版本的也会标记出来。这样该artifact在该工程里是如何被直接或间接引入的进来也就明朗了
4. 如果有冲突的版本，可以右击该版本的节点然后Exclude，对应的pom.xml就已经成功修改了。(IntelliJ IDEA对于文件的修改都是实时保存的，无须Ctrl+S)

问题原因：由于 Dubbo 的依赖程序库本身有自己依赖的 spring 的 jar 包,所以需要对重复的包做一个排除操作

```
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
```
项目中已经配置了日志组件,但是这个日志并没有正常的显示

```
log4j:WARN No appenders could be found for logger (com.alibaba.dubbo.common.logger.LoggerFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
[2017-07-03 10:57:23] Dubbo service server started!

Process finished with exit code 1
```
经检查发现在 Zookeeper 里面进行下载的时候也会出现log4j,slf4j组件,需要继续追加排除

```
<dependency> <!-- 导入Zookeeper开发包 -->
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>${zookeeper.version}</version>
    <exclusions>
        <exclusion>
            <artifactId>slf4j-api</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
        <exclusion>
            <artifactId>log4j</artifactId>
            <groupId>log4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
```
### 调试完成,成功启动

```
2017-07-03 14:06:56  | INFO  | main | cn.hejx.service.impl.demo.ProviderStartup | info ..............
2017-07-03 14:06:56  | ERROR | main | cn.hejx.service.impl.demo.ProviderStartup | error msg
2017-07-03 14:06:56  | INFO  | main | com.alibaba.dubbo.common.logger.LoggerFactory | using logger: com.alibaba.dubbo.common.logger.slf4j.Slf4jLoggerAdapter
2017-07-03 14:06:56  | INFO  | main | com.alibaba.dubbo.container.Main |  [DUBBO] Use container type([spring]) to run dubbo serivce., dubbo version: 2.5.3, current host: 127.0.0.1
七月 03, 2017 2:06:56 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@1bce4f0a: startup date [Mon Jul 03 14:06:56 CST 2017]; root of context hierarchy
七月 03, 2017 2:06:57 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from file [E:\IdeaProjects\dubbodemo\service\target\classes\META-INF\spring\spring-common.xml]
七月 03, 2017 2:06:57 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from file [E:\IdeaProjects\dubbodemo\service\target\classes\META-INF\spring\spring-dubbo.xml]
七月 03, 2017 2:06:57 下午 org.springframework.context.support.PropertySourcesPlaceholderConfigurer loadProperties
信息: Loading properties file from file [E:\IdeaProjects\dubbodemo\service\target\classes\config\dubbo.properties]
2017-07-03 14:06:58  | INFO  | main | com.alibaba.dubbo.config.AbstractConfig |  [DUBBO] The service ready on spring started. service: cn.hejx.remoteapi.IMessage, dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:58  | INFO  | main | com.alibaba.dubbo.config.AbstractConfig |  [DUBBO] Export dubbo service cn.hejx.remoteapi.IMessage to local registry, dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:58  | INFO  | main | com.alibaba.dubbo.config.AbstractConfig |  [DUBBO] Export dubbo service cn.hejx.remoteapi.IMessage to url dubbo://192.168.2.100:19372/cn.hejx.remoteapi.IMessage?anyhost=true&application=hejx-service&default.timeout=3000&dubbo=2.5.3&interface=cn.hejx.remoteapi.IMessage&methods=echo&pid=8928&revision=dev&side=provider&timestamp=1499062018653&version=dev, dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:58  | INFO  | main | com.alibaba.dubbo.config.AbstractConfig |  [DUBBO] Register dubbo service cn.hejx.remoteapi.IMessage url dubbo://192.168.2.100:19372/cn.hejx.remoteapi.IMessage?anyhost=true&application=hejx-service&default.timeout=3000&dubbo=2.5.3&interface=cn.hejx.remoteapi.IMessage&methods=echo&pid=8928&revision=dev&side=provider&timestamp=1499062018653&version=dev to registry registry://192.168.2.233:2181/com.alibaba.dubbo.registry.RegistryService?application=hejx-service&dubbo=2.5.3&pid=8928&registry=zookeeper&timestamp=1499062018617, dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:59  | INFO  | main | com.alibaba.dubbo.remoting.transport.AbstractServer |  [DUBBO] Start NettyServer bind /0.0.0.0:19372, export /192.168.2.100:19372, dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:59  | INFO  | main | com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistry |  [DUBBO] Load registry store file C:\Users\Administrator\.dubbo\dubbo-registry-192.168.2.233.cache, data: {cn.hejx.remoteapi.IMessage:dev=empty://192.168.2.100:19372/cn.hejx.remoteapi.IMessage?anyhost=true&application=hejx-service&category=configurators&check=false&default.timeout=3000&dubbo=2.5.3&interface=cn.hejx.remoteapi.IMessage&methods=echo&pid=7128&revision=dev&side=provider&timestamp=1499062000651&version=dev}, dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:59  | INFO  | ZkClient-EventThread-16-192.168.2.233:2181 | org.I0Itec.zkclient.ZkEventThread | Starting ZkClient event thread.
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:zookeeper.version=3.4.9-1757313, built on 08/23/2016 06:50 GMT
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:host.name=PC-20170423KMDT
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:java.version=1.8.0_131
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:java.vendor=Oracle Corporation
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:java.home=C:\Program Files\Java\jdk1.8.0_131\jre
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:java.class.path=C:\Program Files\Java\jdk1.8.0_131\jre\lib\charsets.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\deploy.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\access-bridge-64.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\cldrdata.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\dnsns.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\jaccess.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\jfxrt.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\localedata.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\nashorn.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\sunec.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\sunjce_provider.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\sunmscapi.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\sunpkcs11.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\ext\zipfs.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\javaws.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\jce.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\jfr.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\jfxswt.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\jsse.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\management-agent.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\plugin.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\resources.jar;C:\Program Files\Java\jdk1.8.0_131\jre\lib\rt.jar;E:\IdeaProjects\dubbodemo\service\target\classes;E:\IdeaProjects\dubbodemo\remoteapi\target\classes;C:\Users\Administrator\.m2\repository\com\alibaba\dubbo\2.5.3\dubbo-2.5.3.jar;C:\Users\Administrator\.m2\repository\org\jboss\netty\netty\3.2.5.Final\netty-3.2.5.Final.jar;C:\Users\Administrator\.m2\repository\org\apache\zookeeper\zookeeper\3.4.9\zookeeper-3.4.9.jar;C:\Users\Administrator\.m2\repository\jline\jline\0.9.94\jline-0.9.94.jar;C:\Users\Administrator\.m2\repository\io\netty\netty\3.10.5.Final\netty-3.10.5.Final.jar;C:\Users\Administrator\.m2\repository\com\101tec\zkclient\0.10\zkclient-0.10.jar;C:\Users\Administrator\.m2\repository\ch\qos\logback\logback-core\1.2.3\logback-core-1.2.3.jar;C:\Users\Administrator\.m2\repository\ch\qos\logback\logback-classic\1.2.3\logback-classic-1.2.3.jar;C:\Users\Administrator\.m2\repository\org\slf4j\slf4j-api\1.7.25\slf4j-api-1.7.25.jar;C:\Users\Administrator\.m2\repository\commons-beanutils\commons-beanutils\1.9.3\commons-beanutils-1.9.3.jar;C:\Users\Administrator\.m2\repository\commons-logging\commons-logging\1.2\commons-logging-1.2.jar;C:\Users\Administrator\.m2\repository\commons-collections\commons-collections\3.2.2\commons-collections-3.2.2.jar;C:\Users\Administrator\.m2\repository\org\apache\commons\commons-lang3\3.5\commons-lang3-3.5.jar;C:\Users\Administrator\.m2\repository\io\netty\netty-all\4.1.9.Final\netty-all-4.1.9.Final.jar;C:\Users\Administrator\.m2\repository\org\javassist\javassist\3.21.0-GA\javassist-3.21.0-GA.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-core\4.3.5.RELEASE\spring-core-4.3.5.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-context\4.3.5.RELEASE\spring-context-4.3.5.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-expression\4.3.5.RELEASE\spring-expression-4.3.5.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-context-support\4.3.5.RELEASE\spring-context-support-4.3.5.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-beans\4.3.5.RELEASE\spring-beans-4.3.5.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-aop\4.3.5.RELEASE\spring-aop-4.3.5.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-aspects\4.3.5.RELEASE\spring-aspects-4.3.5.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\aspectj\aspectjweaver\1.8.9\aspectjweaver-1.8.9.jar;D:\Program Files (x86)\JetBrains\IntelliJ IDEA 2016.3.4\lib\idea_rt.jar
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:java.library.path=C:\Program Files\Java\jdk1.8.0_131\bin;C:\Windows\Sun\Java\bin;C:\Windows\system32;C:\Windows;C:\ProgramData\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Program Files\Java\jdk1.8.0_131\bin;C:\Program Files\Java\jdk1.8.0_131\jre\bin;C:\Program Files\MySQL\MySQL Utilities 1.6\;C:\Program Files\TortoiseSVN\bin;D:\apache-maven-3.3.9\bin;D:\Program Files (x86)\Git\bin;C:\Program Files\MySQL\MySQL Server 5.7\bin;;.
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:java.io.tmpdir=C:\Users\ADMINI~1\AppData\Local\Temp\
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:java.compiler=<NA>
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:os.name=Windows 7
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:os.arch=amd64
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:os.version=6.1
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:user.name=Administrator
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:user.home=C:\Users\Administrator
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Client environment:user.dir=E:\IdeaProjects\dubbodemo
2017-07-03 14:06:59  | INFO  | main | org.apache.zookeeper.ZooKeeper | Initiating client connection, connectString=192.168.2.233:2181 sessionTimeout=30000 watcher=org.I0Itec.zkclient.ZkClient@4bc222e
2017-07-03 14:06:59  | INFO  | main | org.I0Itec.zkclient.ZkClient | Waiting for keeper state SyncConnected
2017-07-03 14:06:59  | INFO  | main-SendThread(192.168.2.233:2181) | org.apache.zookeeper.ClientCnxn | Opening socket connection to server 192.168.2.233/192.168.2.233:2181. Will not attempt to authenticate using SASL (unknown error)
2017-07-03 14:06:59  | INFO  | main-SendThread(192.168.2.233:2181) | org.apache.zookeeper.ClientCnxn | Socket connection established to 192.168.2.233/192.168.2.233:2181, initiating session
2017-07-03 14:06:59  | INFO  | main-SendThread(192.168.2.233:2181) | org.apache.zookeeper.ClientCnxn | Session establishment complete on server 192.168.2.233/192.168.2.233:2181, sessionid = 0x15d065f7a61001e, negotiated timeout = 30000
2017-07-03 14:06:59  | INFO  | main-EventThread | org.I0Itec.zkclient.ZkClient | zookeeper state changed (SyncConnected)
2017-07-03 14:06:59  | INFO  | main | com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistry |  [DUBBO] Register: dubbo://192.168.2.100:19372/cn.hejx.remoteapi.IMessage?anyhost=true&application=hejx-service&default.timeout=3000&dubbo=2.5.3&interface=cn.hejx.remoteapi.IMessage&methods=echo&pid=8928&revision=dev&side=provider&timestamp=1499062018653&version=dev, dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:59  | INFO  | main | com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistry |  [DUBBO] Subscribe: provider://192.168.2.100:19372/cn.hejx.remoteapi.IMessage?anyhost=true&application=hejx-service&category=configurators&check=false&default.timeout=3000&dubbo=2.5.3&interface=cn.hejx.remoteapi.IMessage&methods=echo&pid=8928&revision=dev&side=provider&timestamp=1499062018653&version=dev, dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:59  | INFO  | main | com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistry |  [DUBBO] Notify urls for subscribe url provider://192.168.2.100:19372/cn.hejx.remoteapi.IMessage?anyhost=true&application=hejx-service&category=configurators&check=false&default.timeout=3000&dubbo=2.5.3&interface=cn.hejx.remoteapi.IMessage&methods=echo&pid=8928&revision=dev&side=provider&timestamp=1499062018653&version=dev, urls: [empty://192.168.2.100:19372/cn.hejx.remoteapi.IMessage?anyhost=true&application=hejx-service&category=configurators&check=false&default.timeout=3000&dubbo=2.5.3&interface=cn.hejx.remoteapi.IMessage&methods=echo&pid=8928&revision=dev&side=provider&timestamp=1499062018653&version=dev], dubbo version: 2.5.3, current host: 127.0.0.1
2017-07-03 14:06:59  | INFO  | main | com.alibaba.dubbo.container.Main |  [DUBBO] Dubbo SpringContainer started!, dubbo version: 2.5.3, current host: 127.0.0.1
[2017-07-03 14:06:59] Dubbo service server started!
```
### 查看zookeeper信息

```
zkCli.sh -server 192.168.2.233
[zk: 192.168.2.233(CONNECTED) 3] ls /
[dubbo, zookeeper]
[zk: 192.168.2.233(CONNECTED) 4] ls /dubbo
[cn.hejx.remoteapi.IMessage]
[zk: 192.168.2.233(CONNECTED) 5] 
```

当业务中心注册成功以后会自动的在 zookeeper 里面保存有相应的 dubbo 的远程服务接口信息

### 这里附上修改后的service/pom.xml配置文件

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
        <profiles.dir>src/main/profiles</profiles.dir>
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
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>log4j</artifactId>
                    <groupId>log4j</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>slf4j-api</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>${zkclient.version}</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-api</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- 导入所有要使用到的日志组件 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>${logback.version}</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
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

    <!-- 配置 profile 文件 -->
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

    <build>
        <finalName>service</finalName>
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