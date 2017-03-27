<img src="/docs/aleph.png" align="left" height="210px" hspace="5px"/>

Aleph是在网络上通过[Manifold](https://github.com/ztellman/manifold)来传输数据
比使用`java.io.InputStream`,[core.async](https://github.com/clojure/core.async)通道,
Clojure序列,后者[其他字节表示组件](https://github.com/ztellman/byte-streams)
使用[Netty](https://github.com/netty/netty)来高性能、高可用的传输数据

```clj
[aleph "0.4.3"]
```

### HTTP

Aleph遵循[Ring](https://github.com/ring-clojure)规范，可以替换所有Ring系列的服务器
并且也支持基于[Manifold deferred](https://github.com/ztellman/manifold)事件返回，
这个特性虽然不如Ring的中间件机制那样修改返回结果，但是能够很简单的用它来重新实现相同的机制，
详细看[let-flow](https://github.com/ztellman/manifold/blob/master/docs/deferred.md#let-flow)

```clj
(require '[aleph.http :as http])

(defn handler [req]
  {:status 200
   :headers {"content-type" "text/plain"}
   :body "hello!"})

(http/start-server handler {:port 8080})
```

返回体可以是一个Manifold流,流是按照块来发送每次消息，使用[Server-sent_events](http://en.wikipedia.org/wiki/Server-sent_events)来更深度的控制
返回或者其他目的

HTTP客户端请求，Aleph模型遵循[clj-http](https://github.com/dakrone/clj-http)，
除了每次请求立即返回一个Manifold的deferred对象

```clj
(require
  '[manifold.deferred :as d]
  '[byte-streams :as bs])

(-> @(http/get "https://google.com/")
  :body
  bs/to-string
  prn)

(d/chain (http/get "https://google.com")
  :body
  bs/to-string
  prn)
```

当Aleph完全模仿clj-http的api接口和能力时，它目前还不支持多部请求，cookie保存，代理服务
[查看更多](http://aleph.io/examples/literate.html#aleph.examples.http)

### WebSockets

对于HTTP请求，可以使用`(aleph.http/websocket-connection)`来添加一个`Upgrade`请求头，
它返回一个延迟的 **双工数据流**，使用单个通道来回发送消息
服务端可以使用`take!`来接收消息，使用`put!`来发送消息
一个echo服务器如下

```clj
(require '[manifold.stream :as s])

(defn echo-handler [req]
  (let [s @(http/websocket-connection req)]
    (s/connect s s))))
```

这将会获取所有来自客户端的消息，把它们放入双工socket，推送回客户端
WebSocket文本消息被当成字符串，二进制消息被当成字节数组

WebSocket客户端可以通过调用`(aleph.http/websocket-client url)`，
它返回一个可以被用来读取和发送数据的双工流。

[阅读范例代码](http://aleph.io/examples/literate.html#aleph.examples.websocket)

### TCP

TCP服务与HTTP类似，除了每个链接处理器将会有两个参数：一个双工流一个包含客户端对象信息的map
流被当做字节数组，可以使用[byte-arrays](https://github.com/ztellman/byte-streams)来转换
成其他格式
流可以被用来接收任何二进制的数据

一个应答TCP服务

```clj
(require '[aleph.tcp :as tcp])

(defn echo-handler [s info]
  (s/connect s s))

(tcp/start-server echo-handler {:port 10001})
```

TCP客户端可以通过调用`(aleph.tcp/client {:host "example.com", :port 10001})`来创建，
返回一个双工数据流

[阅读范例代码](http://aleph.io/examples/literate.html#aleph.examples.tcp)

### UDP

UDP可以使用`(aleph.udp/socket {:port 10001 :broadcast? false})`

```clj
{:host "example.com"
 :port 10001
 :message ...}
```

`:message`是一个字节数组,如果`:port`被指定，socket仅能够发送消息

[阅读范例代码](http://aleph.io/examples/literate.html).

### license

Copyright © 2017 Zachary Tellman

Distributed under the MIT License
