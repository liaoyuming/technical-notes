### 概述
WebSocket 是一种网络通信协议，只要经过一次握手就可以跟 WebSocket 服务器建立全双工通信的连接。属于应用层协议，基于TCP传输协议，并复用了 HTTP 的握手通道。

WebSocket 是类似 Socket 的 TCP 长连接通讯模式。一旦 WebSocket 连接建立后，后续数据都以帧序列的形式传输。

WebSocket 协议定义了一个 ws:// 和 wss:// 前缀，分别表示 WebSocket 和 WebSocket Secure 连接。两种方案都使用 HTTP 升级机制来升级到 WebSocket 协议。

协议主要有两个部分：握手和数据传输。

### 握手

例：
客户端的握手如下：
```
    GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
    Origin: http://example.com
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13
```
服务器端的握手如下：
```
    HTTP/1.1 101 Switching Protocols
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
    Sec-WebSocket-Protocol: chat
```

#### 101 Switching Protocols
表示协议切换

#### Upgrade 和 Connection
Upgrade 报头字段被用于由客户端请服务器切换到的一种列出的协议。这里指明升级到 websocket 协议。

Connection 这里表示要升级协议

#### Sec-WebSocket-Key 和 Sec-WebSocket-Accept
Sec-WebSocket-Key 是由一个随机的 16byte 值经过 bast64 编码组成。向服务器提供所需信息，以确认客户端有权请求升级到 WebSocket。

服务器获取 Sec-WebSocket-Key的值后，追加 258EAFA5-E914-47DA-95CA-C5AB0DC85B11，应用 SHA-1 散列函数，得到以 20 字节的值，并使用 base64 对其进行编码，通过 Sec-WebSocket-Accept 返回。伪代码如下：
```
base64_encode(sha1(Sec-WebSocket-Key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"))
```

##### 为啥追加 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 字符串呢
RFC 6455 - The WebSocket Protocol 如下解释
> the Globally Unique Identifier (GUID) "258EAFA5-E914-47DA-
   95CA-C5AB0DC85B11" in string form, which is unlikely to be used by
   network endpoints that do not understand the WebSocket Protocol.

#### Sec-WebSocket-Version
指定客户端希望使用的 WebSocket 协议版本，以便服务器确认是否支持该版本。目前 WebSocket 协议的最新版本是版本13。

#### Sec-WebSocket-Protocol
按优先顺序的一个或多个的 WebSocket 协议。服务器将支持的第一个，在 Sec-WebSocket-Protocol 响应头中返回。非必要字段。

#### Sec-WebSocket-Extensions
指定一个或多个协议级 WebSocket 扩展，请求服务器使用。非必要字段。

### 数据帧

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-------+-+-------------+-------------------------------+
     |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
     |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
     |N|V|V|V|       |S|             |   (if payload len==126/127)   |
     | |1|2|3|       |K|             |                               |
     +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
     |     Extended payload length continued, if payload len == 127  |
     + - - - - - - - - - - - - - - - +-------------------------------+
     |                               |Masking-key, if MASK set to 1  |
     +-------------------------------+-------------------------------+
     | Masking-key (continued)       |          Payload Data         |
     +-------------------------------- - - - - - - - - - - - - - - - +
     :                     Payload Data continued ...                :
     + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
     |                     Payload Data continued ...                |
     +---------------------------------------------------------------+
```

- **FIN**  1bit。如果是1，代表这是消息中的最后一个数据块。
- **RSV1**, **RSV2**, **RSV3**  各占 1bit。一般情况下全为0。当客户端、服务端协商采用 WebSocket 扩展时，这三个标志位可以非0，且值的含义由扩展进行定义。如果出现非零的值，且并没有采用 WebSocket 扩展，连接出错。
- **Opcode**  4bit。定义了 Payload Data 的解释。当接收到一个不能识别的操作码时，接收方必须舍弃这个 WebSocket 连接。
    - %x0 表示这是一个延续帧。当 Opcode 为 0 时，表示本次数据传输采用了数据分片，当前收到的数据帧为其中一个数据分片。
    - %x1 表示这是一个文本帧
    - %x2 表示这是一个二进制帧
    - %x3-7 为将来控制外数据块预留
    - %x8 表示连接关闭
    - %x9 表示这是一个 ping 
    - %xA 表示这是一个 pong 
    - %xB-F 为将来控制数据块预留
- **Mask**  1bit。表示是否要对 Payload Data 进行掩码操作。如果设置为 1，需要在 Masking-key 中提供一个掩码钥匙，用来对负载数据进行解掩码。所有从客户端发往服务端的数据块该位都是 1。
- **Payload length** 7bit，7+16bit，或 7+64bit。Payload Data 的长度，单位是字节。前 7bit 解析为无符号整数，假设为 x
    - 如果 x 为 0-125， 则表示 x 为 Payload length
    - 如果 x 为 126，后面的2个字节解析的16位无符号整数作为 Payload length。
    - 如果 x 为 127，后面的8个字节解析的64位的无符号整数作为 Payload length。
- **Masking-key**  0或4byte（32bit）。所有从客户端发往服务端的帧都经过包含在帧中的一个 32bit 长的 Masking-key 进行掩码。这个值仅当掩码标志设置为1时存在。
- **Payload data** (x + y) byte.
    - Extension data: x byte. 扩展数据。如果没有使用扩展，则为0字节。所有的扩展都必须声明扩展数据的长度，或者可以计算出扩展数据的长度。此外，扩展如何使用必须在握手阶段就协商好。如果扩展数据存在，那么 Payload length 必须将扩展数据的长度包含在内。
    - Application data: y bytes. 应用数据

#### 掩码算法
掩码钥匙是客户端从 32bit值中随机选择的。当准备一个掩码的帧，客户端必须从允许的32位的值中选择一个作为新的 Masking-key。掩码钥匙必须是不可预知的，这对于防止恶意应用截取网络上的数据至关重要。

掩码不影响负载数据的长度。掩码数据与解码数据的过程是一样的。
```
    j = i MOD 4
    transformed-octet-i = original-octet-i XOR masking-key-octet-j
```
- i 为第 i 个字节
- j 为 i MOD 4
- original-octet-i 为转换前数据的第 i 个字节
- masking-key-octet-j 为 masking-key 的第 j 个字节
- transformed-octet-i 为转换后数据的第 i 个字节。值为 original-octet-i 异或 masking-key-octet-j


### 数据传输
WebSocket 数据传输是基于数据帧的传递，并根据 opcode 来区分操作的类型。

#### 分片
WebSocket的每条消息可能被切分成多个数据帧。当WebSocket的接收方收到一个数据帧时，会根据FIN的值来判断，是否已经收到消息的最后一个数据帧。

FIN=1表示当前数据帧为消息的最后一个数据帧，此时接收方已经收到完整的消息，可以对消息进行处理。FIN=0，则接收方还需要继续监听接收其余的数据帧。

例：
```
Client: FIN=1, opcode=0x1, msg="hello"
Server: (process complete message immediately) Hi.
Client: FIN=0, opcode=0x1, msg="and a"
Server: (listening, new message containing text started)
Client: FIN=0, opcode=0x0, msg="happy new"
Server: (listening, payload concatenated to previous message)
Client: FIN=1, opcode=0x0, msg="year!"
Server: (process complete message) Happy new year to you too!
```
- FIN=1，opcode=0x1 表示这是个文本帧，并且消息已经传输完。
- FIN=0, opcode=0x1 表示这是个文本帧，并且消息还没传输完，后面还有数据帧。
- FIN=0, opcode=0x0 表示这是个延续帧，并且还有消息未传输完。此帧接上一条数据帧。
- FIN=1, opcode=0x0 表示消息已经传输完，没有后续数据帧。此帧接上一条数据帧。服务端可以将关联的数据帧组装成完整的消息。


### 心跳
为了保持客户端、服务端的实时双向通信，需要确保客户端、服务端之间的TCP通道保持连接没有断开。这时候就需要 WebSockets 的心跳机制。

WebSockets 在经过握手之后的任意时刻里，无论客户端还是服务端都可以选择发送一个ping给另一方。 当ping消息收到的时候，接受的一方必须尽快回复一个pong消息。 

一个ping 或者 pong 都只是一个常规的帧， 只是这个帧是一个控制帧。Ping消息的opcode字段值为 0x9，pong消息的opcode值为  0xA 。当你获取到一个ping消息的时候，回复一个跟ping消息有相同载荷数据的pong消息 (对于ping和pong，最大载荷长度位125)。 你也有可能在没有发送ping消息的情况下，获取一个pong消息，当这种情况发生的时候忽略它

### 关闭连接
客户端或服务器端都可以通过发送一个带有指定控制序列的控制帧以开始关闭连接握手。

应用在发送完关闭数据包后就不能再发送任何其它的数据包。假如终端接收到了一个关闭数据包，还没有发送过关闭数据包，这时终端必须发送一个关闭数据包作为响应。

### 应用
使用 WebSocket 实现多人在线聊天功能。

这里介绍 Go 语言的 [gorilla/websocket](https://github.com/gorilla/websocket) 里面 chat example。

以下列出关键代码：

- 服务端端
```go
//监听端口
var addr = flag.String("addr", ":8080", "http service address")
http.HandleFunc("/ws", func(w http.ResponseWriter, r *http.Request) {
		serveWs(hub, w, r)
	})
err := http.ListenAndServe(*addr, nil)
```

```go
// 建立连接
var upgrader = websocket.Upgrader{
	ReadBufferSize:  1024,
	WriteBufferSize: 1024,
}
conn, err := upgrader.Upgrade(w, r, nil)

// 注册 client ，用于发送消息
client := &Client{hub: hub, conn: conn, send: make(chan []byte, 256)}
client.hub.register <- client

// 监听 channal 并进行读写操作
go client.writePump()
go client.readPump()
```

```go
// 读操作
func (c *Client) readPump() {
	...
	// 设置等待 pong 最大时长
	c.conn.SetReadDeadline(time.Now().Add(pongWait))
	c.conn.SetPongHandler(func(string) error {
	    c.conn.SetReadDeadline(time.Now().Add(pongWait))
	    return nil 
	})
	for {
	    // 读取消息
		_, message, err := c.conn.ReadMessage()
	    ...
		c.hub.broadcast <- message
	}
}

// 写操作
func (c *Client) writePump() {
	ticker := time.NewTicker(pingPeriod)
	...
	for {
		select {
		case message, ok := <-c.send:
			c.conn.SetWriteDeadline(time.Now().Add(writeWait))
			if !ok {
				// 发送关闭消息  opcode 为 8
				c.conn.WriteMessage(websocket.CloseMessage, []byte{})
				return
			}
			w, err := c.conn.NextWriter(websocket.TextMessage)
			if err != nil {
				return
			}
			 // 发送 text 消息，opcode 设置 1
			w.Write(message)
            ...
            // 发送完成 FIN 为 1
			if err := w.Close(); err != nil {
				return
			}
		case <-ticker.C:
		    // 发送 ping 
			c.conn.SetWriteDeadline(time.Now().Add(writeWait))
			if err := c.conn.WriteMessage(websocket.PingMessage, nil); err != nil {
				return
			}
		}
	}
}
```


```go
// 实现 多人聊天 的关键代码
func (h *Hub) run() {
	for {
		select {
		// 注册 client
		case client := <-h.register:
			h.clients[client] = true
		// 断开 client
		case client := <-h.unregister:
			if _, ok := h.clients[client]; ok {
				delete(h.clients, client)
				close(client.send)
			}
		// 广播消息
		case message := <-h.broadcast:
			for client := range h.clients {
				select {
				case client.send <- message:
				default:
					close(client.send)
					delete(h.clients, client)
				}
			}
		}
	}
}
```
- 客户端（浏览器）
```
    conn = new WebSocket("ws://" + document.location.host + "/ws");
    conn.onclose = function (evt) {
        ...
    };
    conn.onmessage = function (evt) {
        var messages = evt.data.split('\n');
        for (var i = 0; i < messages.length; i++) {
            ...
        }
    };
```


---
参考自：
- [RFC 6455 - The WebSocket Protocol](https://tools.ietf.org/html/rfc6455)
- [Protocol upgrade mechanism](https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism)
- [Writing WebSocket servers](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_servers)
- [Websocket Chat Example](https://github.com/gorilla/websocket/tree/master/examples/chat)