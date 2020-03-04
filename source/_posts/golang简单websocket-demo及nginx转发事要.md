---
title: golang简单websocket demo及nginx转发事要
date: 2020-03-04 19:57:25
tags:
---

## 序

最近一个项目上用到消息推送功能，boss说写个简单的ws让前端自行处理消息就好（前端：MMP）。
看到网上用到比较多的框架是[gorilla/websocket](https://github.com/gorilla/websocket)，但是要验证第三方包的稳定性还是需要一定时间，还是用官方的websocket包吧（又不是不能用.jpg）。
思路其实很简单，一端接收到消息之后直接推入一个channel。另启一个协程去接收这个channel的消息，然后广播到所有ws连接。

## Code

```go
package main

import (
    "context"
    "log"
    "net/http"
    "sync"

    "golang.org/x/net/websocket"
)

var (
    // WsSrv -
    WsSrv *WsServer
)

const (
    // MessageChanSize -
    MessageChanSize = 1024
)

// WsServer -
type WsServer struct {
    Clients     map[string]*WsClient
    MessageChan chan string
    Codec       websocket.Codec
    sync.RWMutex
}

// ClientConn -
type WsClient struct {
    websocket *websocket.Conn
}

// InitWsServer -
func InitWsServer() {
    WsSrv = &WsServer{
        Clients:     map[string]*WsClient{},
        MessageChan: make(chan string, MessageChanSize),
        Codec:       websocket.Message,
    }
}

// WsReceiver -
func WsReceiver(ws *websocket.Conn) {
    var (
        msg string
        err error
    )
    defer func() {
        if err = ws.Close(); err != nil {
            log.Printf("[WsReceiver] ws(%v) closed err: %v\n", ws.RemoteAddr(), err)
        }
    }()

    userID := ws.Request().URL.Query().Get("user_id")
    if userID == "" {
        return
    }

    defer func() {
        WsSrv.Lock()
        defer WsSrv.Unlock()
        delete(WsSrv.Clients, userID)
    }()

    WsSrv.Lock()
    WsSrv.Clients[userID] = &WsClient{websocket: ws}
    WsSrv.Unlock()

    for {
        if err = WsSrv.Codec.Receive(ws, &msg); err != nil {
            log.Printf("[WsReceiver] receive from remote result err: %v\n", err)
            return
        }
        WsSrv.MessageChan <- msg
    }
}

// WsSender -
func WsSender(ctx context.Context) {
    var err error
    for {
        select {
        case msg := <-WsSrv.MessageChan:
            for userID, conn := range WsSrv.Clients {
                if err = websocket.Message.Send(conn.websocket, msg); err != nil {
                    log.Printf("[WsSender] send msg(%v) to user_id(%v) result err: %v\n", msg, userID, err)
                }
            }
        case <-ctx.Done():
            return
        }
    }
}

func main() {
    var err error

    InitWsServer()
    go WsSender(context.Background())

    mux := http.NewServeMux()
    mux.Handle("/im", websocket.Handler(WsReceiver))
    mux.HandleFunc("/ping", func(writer http.ResponseWriter, request *http.Request) {
        _, _ = writer.Write([]byte("pong"))
    })

    if err = http.ListenAndServe(":23333", mux); err != nil {
        log.Fatalln(err)
    }
}
```

## nginx转发事要

在http请求升级为ws请求的过程中会带着两个重要的header，一个是`Upgrade: websocket`，另一个是`Connection: upgrade`。只有带上这两个header才能在http/1.1下成功将http请求升级为ws请求，但是如果在服务器中有nginx反向代理的情况下往往会把这个头丢掉，所以要在配置文件上加上来，否则会出现著名的[http状态码400](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)错误

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    "" close;
}

upstream srv {
    server 127.0.0.1:23333;
}

server {
    listen 80;

    location ~ ^/im {
        proxy_pass http://srv;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
    }
}
```

## 鸣谢

> [golang websocket的例子](https://blog.csdn.net/yxw2014/article/details/24317993)
> [Using NGINX as a WebSocket Proxy](https://www.nginx.com/blog/websocket-nginx/)
> [websocket 在线测试工具](http://www.easyswoole.com/wstool.html)
> [wscat](https://www.npmjs.com/package/wscat)
