[SpringBoot集成WebSocket实现网页聊天](https://docs.qq.com/doc/DUVp0ZHFVVWtwY2FX?friendUin=%25252FJYpsJJ1jZeryahi3Yo8ag%25253D%25253D&ADUIN=3227489569&ADSESSION=1681737129&ADTAG=CLIENT.QQ.5971_.0&ADPUBNO=27303&jumpuin=3227489569)
# 基本配置

```xml
<!-- websocket -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

配置类：
```java
@Configuration  
public class WebSocketConfig {  
  
    /**  
     * 注入一个ServerEndpointExporter,该Bean会自动注册使用@ServerEndpoint注解申明的websocket endpoint  
     */    @Bean  
    public ServerEndpointExporter serverEndpointExporter() {  
        return new ServerEndpointExporter();  
    }  
}
```

可能需要手动开放url

# 生命周期
`@OnOpen`在一个用户进行连接的时候进行调用。`@OnClose`则在关闭时调用。

`@OnMessage`在收到客户端发送的消息时调用。

`@OnError`在出错时调用，第二个参数是`Throwable`。

```
@OnOpen
public void onOpen(Session , params...)
```

# 消息传输
session内的方法已经封装了发送。

而接收信息只要在`@OnMessage`里面像Controller一样接受参数就行了。












