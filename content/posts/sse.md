---
title: "SSE 服务端推送技术"
date: 2024-10-22T16:33:18+08:00
draft: false
tags:
- sse
- go
categories:
- 开发
---

<!--more-->



目前基于 AI 对话相关的接口都使用了 SSE 的技术。

让我们看看SSE是什么，以及怎么实现。

SSE(Server-Sent Events) 是一种用于实现服务器主动向客户端推送数据的技术。 

### **实现原理**

SSE 基于 HTTP 协议，利用了其长连接特性, 就是服务器向客户端声明，要发送的是流信息（streaming）, 不是一次性的数据包. 

客户端根据 Response 头信息, 不关闭连接，一直等着服务器发过来的新的数据流。这样就实现了服务端的推送功能。


**服务端数据格式**

header 头:

```
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```

body 格式：

```
: 这是注释 \n\n
id: 消息id\n
event: 事件名字\n
data: 内容换行\n
data: 内容..\n\n
```

> 每段消息内容以 `\n\n` 分割

客户端使用 `EventSource` 对象读取消息

### **实现**

知道了原理，实现起来就很简单了，服务端声明头信息为 streaming， 持续向客户端发送数据即可。


**服务端**

```go
package main

import (
  "fmt"
  "net/http"
  "time"
)

func main() {
  http.HandleFunc("/events", func(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    w.Header().Set("Access-Control-Allow-Origin", "*")

    for {
      // Simulate sending events every second
      fmt.Fprintf(w, "data: %s\n\n", time.Now().Format(time.Stamp))
      w.(http.Flusher).Flush()
      time.Sleep(1 * time.Second)
    }
  })

  http.ListenAndServe(":8080", nil)
}
```

**客户端**

```js
<script>
  const eventSource = new EventSource('http://localhost:8080/events')

  eventSource.onmessage = function (event) {
    console.log('Message received: ', event.data)
  }

  eventSource.onerror = function (event) {
    console.error('An error occurred:', event)
  }
</script>
```



