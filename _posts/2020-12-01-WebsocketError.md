---
layout: article
title: "WebSocket 远程端点处于[TEXT_FULL_WRITING]状态"
key: 8
tags: 
        - websocket
        - 报错
toc: true
---
### 报错信息
![error info]({{site.url}}/assets/images/websocket_error/websocket_TEXT_FULL_WRITING.png)

### 报错原因
> 当几个线程试图通过相同的会话（套接字）发送一些消息时，会抛出异常

### 解决方法
> 代码同步，使用`getBasicRemote()`同步方法，使用`getAsyncRemote()`异步方法可能还是会有这个报错
```java
synchronized (toSession) {
    log.info("服务端给客户端[{}]发送消息成功{}", toSession.getId(), message);
    toSession.getBasicRemote().sendText(message);
}
```