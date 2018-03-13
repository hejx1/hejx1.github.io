---
layout: post
title: websocket 应用示例
date: 2018-03-12
tags: websocket
---



### websocket 应用

这篇文章主要来介绍一下在java项目中，特别是java web项目中websocket的应用。

场景：我做了一个商城系统，跟大多数商城系统，分为客户端和后台，客户端供客户浏览，下单，购买，后台主要管理商品，处理订单，发货等。我现在要实现的功能是，当客户端有客户下单，并且支付完成以后，主动推送消息给后台，让后台的人知道，好去处理发货等事宜。

首先，我们要知道websocket是一个连接，这个连接是客户端（页面）与服务端之间的连接，所以我们要分两部分来完成这个连接，服务端代码和客户端代码。

1.首先，在pom.xml引入如下jar包

```
<!-- websocket -->
<dependency>
    <groupId>org.java-websocket</groupId>
    <artifactId>Java-WebSocket</artifactId>
    <version>1.3.0</version>
</dependency>
```

2.然后我们要知道的是，websocket是客户端和服务端之间建立了一个连接，建立完连接以后，会生成一个websocket对象，我们可以用这个对象来执行发送，接收等操作。但是这只是一个存在于客户端与服务器之间的链接，换句话说，系统只能识别到这个websocket连接是对应于哪个页面（浏览器），而这个页面在系统中是对应哪个用户（数据库中的用户，或者根本就没有对应任何用户，即未登录，只是一个游客），我们是无法从这个websocket对象中获取的。所以我们需要创建一个Map对象，用于将websocket对象和实际的user对象进行关联，这样为我们后续向特定的用户推送消息做铺垫。

为此，我们创建一个WsPool，即websocket连接池的类，该类用于管理现实中的用户和websocket对象之间的关联。代码如下。

```
package com.xdx.websocket;

import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

import org.java_websocket.WebSocket;

public class WsPool {
    private static final Map<WebSocket, String> wsUserMap = new HashMap<WebSocket, String>();

    /**
     * 通过websocket连接获取其对应的用户
     * 
     * @param conn
     * @return
     */
    public static String getUserByWs(WebSocket conn) {
        return wsUserMap.get(conn);
    }

    /**
     * 根据userName获取WebSocket,这是一个list,此处取第一个
     * 因为有可能多个websocket对应一个userName（但一般是只有一个，因为在close方法中，我们将失效的websocket连接去除了）
     * 
     * @param user
     */
    public static WebSocket getWsByUser(String userName) {
        Set<WebSocket> keySet = wsUserMap.keySet();
        synchronized (keySet) {
            for (WebSocket conn : keySet) {
                String cuser = wsUserMap.get(conn);
                if (cuser.equals(userName)) {
                    return conn;
                }
            }
        }
        return null;
    }

    /**
     * 向连接池中添加连接
     * 
     * @param inbound
     */
    public static void addUser(String userName, WebSocket conn) {
        wsUserMap.put(conn, userName); // 添加连接
    }

    /**
     * 获取所有连接池中的用户，因为set是不允许重复的，所以可以得到无重复的user数组
     * 
     * @return
     */
    public static Collection<String> getOnlineUser() {
        List<String> setUsers = new ArrayList<String>();
        Collection<String> setUser = wsUserMap.values();
        for (String u : setUser) {
            setUsers.add(u);
        }
        return setUsers;
    }

    /**
     * 移除连接池中的连接
     * 
     * @param inbound
     */
    public static boolean removeUser(WebSocket conn) {
        if (wsUserMap.containsKey(conn)) {
            wsUserMap.remove(conn); // 移除连接
            return true;
        } else {
            return false;
        }
    }

    /**
     * 向特定的用户发送数据
     * 
     * @param user
     * @param message
     */
    public static void sendMessageToUser(WebSocket conn, String message) {
        if (null != conn && null != wsUserMap.get(conn)) {
            conn.send(message);
        }
    }

    /**
     * 向所有的用户发送消息
     * 
     * @param message
     */
    public static void sendMessageToAll(String message) {
        Set<WebSocket> keySet = wsUserMap.keySet();
        synchronized (keySet) {
            for (WebSocket conn : keySet) {
                String user = wsUserMap.get(conn);
                if (user != null) {
                    conn.send(message);
                }
            }
        }
    }

}
```



3.接下来我们编写websocket的主程序类，该类用于管理websocket的生命周期。该类继承自WebSocketServer ，这是一个实现了runnable接口的类，他的构造函数需要传入一个端口，所以我们需要为websocket服务指定一个端口，该类有四个要重载的方法，onOpen()方法在连接创建成功以后调用，onClose在连接关闭以后调用，onError方法在连接发生错误的时候调用（一般连接出错以后触发了onError，也会紧接着触发onClose方法）。onMessage方法在收到客户端发来消息的时候触发。我们可以在这个方法中处理客户端所传递过来的消息。

```
package com.xdx.websocket;

import java.net.InetSocketAddress;

import org.java_websocket.WebSocket;
import org.java_websocket.handshake.ClientHandshake;
import org.java_websocket.server.WebSocketServer;

public class WsServer extends WebSocketServer {
    public WsServer(int port) {
        super(new InetSocketAddress(port));
    }

    public WsServer(InetSocketAddress address) {
        super(address);
    }

    @Override
    public void onOpen(WebSocket conn, ClientHandshake handshake) {
        // ws连接的时候触发的代码，onOpen中我们不做任何操作

    }

    @Override
    public void onClose(WebSocket conn, int code, String reason, boolean remote) {
        //断开连接时候触发代码
        userLeave(conn);
        System.out.println(reason);
    }

    @Override
    public void onMessage(WebSocket conn, String message) {
        System.out.println(message);
        if(null != message &&message.startsWith("online")){
            String userName=message.replaceFirst("online", message);//用户名
            userJoin(conn,userName);//用户加入
        }else if(null != message && message.startsWith("offline")){
            userLeave(conn);
        }

    }

    @Override
    public void onError(WebSocket conn, Exception ex) {
        //错误时候触发的代码
        System.out.println("on error");
        ex.printStackTrace();
    }
    /**
     * 去除掉失效的websocket链接
     * @param conn
     */
    private void userLeave(WebSocket conn){
        WsPool.removeUser(conn);
    }
    /**
     * 将websocket加入用户池
     * @param conn
     * @param userName
     */
    private void userJoin(WebSocket conn,String userName){
        WsPool.addUser(userName, conn);
    }

}
```



上述onMessage()方法中，我们接收到客户端传过来的一个message（消息），而这个客户端对应的websocket连接也被当成一个参数一起传递过来，我们通过message中携带的信息来判定这条信息对应是什么操作，如果是以online开头，则说明它是一条上线的是信息，我们就把该websocket和其对应的userName存入ws连接池中，如果是以offline开头，则说明websocket断开了，我们也没有必要维护这个websocket对应的map键值对，把它去除掉就好了。

4.如何在服务端开启这个socket呢。我们上面有说到WsServer的父类WebSocketServer 实现了一个runnable方法，由此可见我们需要在一个线程中运行这个WsServer，事实上，WebSocketServer 有个start()方法，其源码如下。

```
public void start() {
    if( selectorthread != null )
    	throw new IllegalStateException( getClass().getName() + " can only be started once." );
    new Thread( this ).start();;
}
```

很显然，它开了一个线程。所以我们可以用下面这样的方法来开启一个websocket线程。（该方法只是针对普通的java项目，如果是web项目需要在项目启动的时候运行websocket线程，后面第7点会讲）

```
public static void main(String args[]){
    WebSocketImpl.DEBUG = false;
    int port = 8887; // 端口
    WsServer s = new WsServer(port);
    s.start();
}
```

我们运行这个main方法，就开启了websocket的服务端。

到目前为止，我们还没有编写任何客户端的代码。我们如何测试已经开启了websocket服务端呢？网上有一个免费的测试工具。

测试地址如下：[测试websocket](http://www.blue-zero.com/WebSocket/)

点击进去，写上我们的websocket服务地址。点击连接。如图所示。

![](/images/posts/websocket/2-1.png)

如果连接成功，他就会显示“连接已建立，正在等待数据……”

我们再文本框中输入onlinexdx，然后点回车试试。

![](/images/posts/websocket/2-2.png)

这就是模拟客户端向服务端发送onlinexdx请求，按前面的介绍，它会触发服务端的onMessage方法，我们看一下服务端控制台。除了我们希望看到的onlinexdx这个message,还有一些@heart,果然触发了这个方法。

![](/images/posts/websocket/2-3.png)

@heart是这个免费页面发送过来的心跳检测包，目的是让websocket一直处于连接状态。

5.好了，接下来我们要来完成客户端部分的功能。我想要实现的是后台用户登录以后，进入到后台主页的时候，执行websocket连接工作（就类似于上一步在免费页面点击连接按钮），然后向服务端发送“online+userName”这条消息，用以触发服务端的onMessage方法，就可以将该userName加入到连接池了。我们将这些代码封装到js中。

```
var websocket = '';
var ajaxPageNum = 1;
var last_health;
var health_timeout = 10;
var tDates = [], tData = [];
var rightIndex;
if ($('body').attr('userName') != '' && $('body').attr('ws') == 'yes') {
    var userName = $('body').attr('userName');
    if (window.WebSocket) {
        websocket = new WebSocket(
                encodeURI('ws://' + document.domain + ':8887'));
        websocket.onopen = function() {
            console.log('已连接');
            websocket.send("online"+userName);
            heartbeat_timer = setInterval(function() {
                keepalive(websocket)
            }, 60000);
        };
        websocket.onerror = function() {
            console.log('连接发生错误');
        };
        websocket.onclose = function() {
            console.log('已经断开连接');
            initWs();
        };
        // 消息接收
        websocket.onmessage = function(message) {
            console.log(message)
            showNotice("新订单", "您有新的逸品订单，请及时处理！")
        };
    } else {
        alert("该浏览器不支持下单提醒。<br/>建议使用高版本的浏览器，<br/>如 IE10、火狐 、谷歌  、搜狗等");
    }

}
var initWs = function() {
    if (window.WebSocket) {
        websocket = new WebSocket(
                encodeURI('ws://' + document.domain + ':8887'));
        websocket.onopen = function() {
            console.log('已连接');
            websocket.send("online"+userName);
            heartbeat_timer = setInterval(function() {
                keepalive(websocket)
            }, 60000);
        };
        websocket.onerror = function() {
            console.log('连接发生错误');
        };
        websocket.onclose = function() {
            console.log('已经断开连接');
            initWs();
        };
        // 消息接收
        websocket.onmessage = function(message) {
            console.log(message)
            showNotice("新订单", "您有新的逸品订单，请及时处理！")
        };
    } else {
        alert("该浏览器不支持下单提醒。<br/>建议使用高版本的浏览器，<br/>如 IE10、火狐 、谷歌  、搜狗等");
    }
}
var vadioTimeOut;
function showNotice(title, content) {
    if (!title && !content) {
        title = "新订单";
        content = "您有新的订单,请及时处理！";
    }
    var iconUrl = "http://www.wonyen.com/favicon.ico";
    $("#myaudio")[0].play();// 消息播放语音
    var playTime = 1;
    var audio = document.createElement("myaudio");
    clearTimeout(vadioTimeOut);
    audio.addEventListener('ended', function() {
        vadioTimeOut = setTimeout(function() {
            playTime = playTime + 1;
            playTime < 3 ? audio.play() : clearTimeout(vadioTimeOut);
        }, 500);
    })
    if (Notification.permission == "granted") {
        var notification = new Notification(title, {
            body : content,
            icon : iconUrl
        });

        notification.onclick = function() {
            notification.close();
        };
    }

}

// 心跳包
function keepalive(ws) {
    var time = new Date();
    if (last_health != -1 && (time.getTime() - last_health > health_timeout)) {

        // ws.close();
    } else {
        if (ws.bufferedAmount == 0) {
            ws.send('~HC~');
        }
    }
}
```

页面的主要代码如下。首先是引入上述js.这个js必须放在页面最后，因为它需要加载完页面以获取body的attr。

```
<!-- websocket -->
<script src="./static/js/OtherJs/ws.js" type="text/javascript"></script>
```

其次是在页面的body处加入userName和ws属性。作为参数传递到js里面。还需要加入语音附件。

```
<body userName=${adminName} ws="yes">
<!-- 消息提示音 -->
<audio id="myaudio" src="./static/new_order.wav"></audio>
```

上述js代码清楚地展示了websocket在页面端的生命周期，需要注意的是，我们在onopen()方法中，先是向服务端发送online+userName进行上线处理，紧接着开始调用心跳包，避免websocket长时间闲置而失效。

onmessage方法中，我们处理收到服务端推送过来的消息，然后以语音和弹出窗的形式提醒客户端。当然，我们在服务端可以通过封装message这个参数，把它变成一个json对象，给这个json对象一个msgType属性，这样就可以根据msgType的不同来执行不同的前端代码，比如**msgType=newOrder**表示有新的订单，就执行新订单到来的代码，**msgType=newUser**表示有新的用户注册，就执行新用户注册的代码。这边我没做区分，因为我在服务端只发送用户购买订单的消息，所以所有的消息，我都执行showNotice("新订单", "您有新的逸品订单，请及时处理！")这个方法。

6.最后一步，我们来编写从服务端向客户端发送消息的方法。一旦有订单到来，我们就向所有的后台用户发送消息。我们可以模拟一下这个动作，写一个controller方法，调用WsPool的sendMessageToAll方法。

```
@ResponseBody
@RequestMapping("sendWs")
public String sendWs(String message) {
    WsPool.sendMessageToAll(message);
    return message;
}
```

7.对了，如果是web项目，我们还需要在项目启动的时候开启websocket服务端线程，可以把启动的动作放在一个filter中，然后在web.xml里面配置这个filter，使它在项目启动时候运行。

```
package com.xdx.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

import org.java_websocket.WebSocketImpl;

import com.xdx.websocket.WsServer;


public class StartFilter implements Filter {

    public void destroy() {

    }

    public void doFilter(ServletRequest arg0, ServletResponse arg1,
            FilterChain arg2) throws IOException, ServletException {

    }

    public void init(FilterConfig arg0) throws ServletException {
        this.startWebsocketInstantMsg();
    }

    /**
     * 启动即时聊天服务
     */
    public void startWebsocketInstantMsg() {
        WebSocketImpl.DEBUG = false;
        WsServer s;
        s = new WsServer(8887);
        s.start();
    }
}
```

在web.xml配置。

```
<!-- filter -->
<filter>
    <filter-name>startFilter</filter-name>
    <filter-class>com.xdx.filter.StartFilter</filter-class>
</filter>
```

至此，我们完成了所有的代码。

8.测试，先运行起项目。然后登陆以后，进入后台主页，可以看到，已经连接成功了。

![](/images/posts/websocket/2-4.png)

然后我们从服务端向后台发送一条消息，执行sendWs这个方法。在浏览器输入http://192.168.1.185:8080/warrior/sendWs?message=xxx

![](/images/posts/websocket/2-5.png)

会播放语音，并且弹出提示。证明成功了。

上述只是websocket的一个简单的应用，在此基础上，我们还可以做很多扩展的工作，比如做聊天室，股票实时价格显示等，只要掌握了原理就好做了。

原文链接：https://www.cnblogs.com/roy-blog/p/7211761.html