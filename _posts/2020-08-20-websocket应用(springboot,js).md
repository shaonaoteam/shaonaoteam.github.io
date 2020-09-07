---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 学习记录
---
>pom.xml

```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.68</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>cn.zjh.spring.websocket.WebSocketApplication</mainClass>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>

```
>websocket.java

```
package cn.zjh.spring.websocket.server;

import cn.zjh.spring.websocket.pojo.Position;
import com.alibaba.fastjson.JSON;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;

@ServerEndpoint("/websocket/{username}")
@Component
@Slf4j
@SuppressWarnings("unchecked")
public class WebSocketServer {
    private static int onlineCount = 0;
    private static Map<String, WebSocketServer> clients = new ConcurrentHashMap();
    private Session session;
    private String username;
    private static Integer threadStatus = 0;
    public WebSocketServer() {

    }

    @OnOpen
    public void onOpen(@PathParam("username") String username, Session session) throws IOException {

        this.username = username;
        this.session = session;
        addOnlineCount();
        clients.put(username, this);
        //连接逻辑处理
        StringBuffer sb = new StringBuffer();
        clients.forEach((key, value) -> sb.append("\tusername:").append(key));
        log.info("【{}】已建立连接,当前连接ws连接池:{}",username,sb);
        log.info("开始推送坐标信息\n------------------------------------");
        if (threadStatus == 0) {
            log.info("线程初始化");
            TestThread testThread = new TestThread();
            Thread thread = new Thread(testThread);
            thread.start();
            log.info("初始化完毕");
        }

    }

    @OnClose
    public void onClose() throws IOException {
        clients.remove(this.username);
        subOnlineCount();
    }

    @OnMessage
    public void onMessage(String message, Session session) throws IOException {
        synchronized(session) {
            this.session.getAsyncRemote().sendText(message);
        }
    }

    @OnError
    public void onError(Session session, Throwable error) {
        error.printStackTrace();
    }

    public void sendMessageTo(String message, String To) throws IOException {
        Iterator var3 = clients.values().iterator();

        while(var3.hasNext()) {
            WebSocketServer item = (WebSocketServer)var3.next();
            if (item.username.equals(To)) {
                item.session.getAsyncRemote().sendText(message);
            }
        }

    }

    public void sendMessageAll(String message) throws IOException {
        Iterator var2 = clients.values().iterator();

        while(var2.hasNext()) {
            WebSocketServer item = (WebSocketServer)var2.next();
            item.session.getAsyncRemote().sendText(message);
        }

    }

    public static synchronized int getOnlineCount() {
        return onlineCount;
    }

    public static synchronized void addOnlineCount() {
        ++onlineCount;
    }

    public static synchronized void subOnlineCount() {
        --onlineCount;
    }

    public static synchronized Map<String, WebSocketServer> getClients() {
        return clients;
    }




    /***
     * 邱刚的临时内部类
     * {id：1,course:180.5,altitude:2200,longitude:123.45,latitude:27.22}
     */

    private class TestThread implements Runnable {

        /***
         * 线程
         */
        @Override
        public void run() {
            log.info("线程启动");
            threadStatus=1;
            int count=0;
            for (int i = 0;  ; i++) {
                if(clients.size()!=0){

                    for (Map.Entry<String, WebSocketServer> item : clients.entrySet()) {

                        log.info("当前线程还存在{}个webSocket客户端对象",clients.size());
                        try {
                            Thread.sleep(3000/clients.size());
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        try {
                            List<Position> list = new ArrayList<>();
                            list.add(Position.builder().id(1).course(353.5).altitude(2200.0).longitude(106.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(2200.0).longitude(108.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(9000.0).longitude(110.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(2200.0).longitude(112.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(2200.0).longitude(114.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(2200.0).longitude(116.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(2200.0).longitude(118.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(7800.0).longitude(120.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(4500.0).longitude(122.1).latitude(27.123).build());
                            list.add(Position.builder().id(1).course(353.5).altitude(8000.0).longitude(124.1).latitude(27.123).build());
                            List<Position> list2 = new ArrayList<>();
                            System.out.println(count);
                            list2.add(list.get(count%10));
                            ObjectMapper objectMapper = new ObjectMapper();
                            sendMessageTo(  objectMapper.writeValueAsString(list2),item.getKey());
                            count ++;
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }

                }else{
                    log.info("告知线程休眠");
                    Thread.interrupted();
                    break;
                }

            }
            log.info("线程结束\n---------------------------");
            threadStatus=0;
        }
    }


    }


```

>html

```
<script>
var ws = new WebSocket("ws://localhost:13000/websocket/zjh"); 
//申请一个WebSocket对象，参数是服务端地址，同http协议使用http://开头一样，WebSocket协议的url使用ws://开头，另外安全的WebSocket协议使用wss://开头
ws.onopen = function(){
　　//当WebSocket创建成功时，触发onopen事件
  
}
ws.onmessage = function(e){
　　//当客户端收到服务端发来的消息时，触发onmessage事件，参数e.data包含server传递过来的数据
　　console.log("服务器返回",e.data);
}
ws.onclose = function(e){
　　//当客户端收到服务端发送的关闭连接请求时，触发onclose事件
　　console.log("close");
}
ws.onerror = function(e){
　　//如果出现连接、处理、接收、发送数据失败的时候触发onerror事件
　　console.log(error);
}
</script>

```