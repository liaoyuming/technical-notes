### 100 Continue
表示目前为止一切正常, 客户端应该继续请求, 如果已完成请求则忽略.

为了让服务器检查请求的首部, 客户端必须在发送请求实体前, 在初始化请求中发送 Expect: 100-continue 首部并接收 100 Continue 响应状态码.

#### Expect
包含一个期望条件，表示服务器只有在满足此期望条件的情况下才能妥善地处理请求。

规范中只规定了一个期望条件，即 Expect: 100-continue, 对此服务器可以做出如下回应：

- 100 如果消息头中的期望条件可以得到满足，使得请求可以顺利进行的话，
- 417 (Expectation Failed) 如果服务器不能满足期望条件的话；也可以是其他任意表示客户端错误的状态码（4xx）。

### 200 OK
表明请求已经成功. 默认情况下状态码为200的响应可以被缓存。

### 201 Created
表示请求已经被成功处理，并且创建了新的资源。新的资源在应答返回之前已经被创建。

### 202 Accepted
表示服务器端已经收到请求消息，但是尚未进行处理。但是对于请求的处理确实无保证的，即稍后无法通过 HTTP 协议给客户端发送一个异步请求来告知其请求的处理结果。

### 204 No Content
表示目前请求成功，但客户端不需要更新其现有页面。

使用惯例是，在 PUT 请求中进行资源更新，但是不需要改变当前展示给用户的页面，那么返回 204 No Content。如果新创建了资源，那么返回 201 Created 。如果页面需要更新以反映更新后的资源，那么需要返回 200 。

### 301 Moved Permanently
永久重定向。说明请求的资源已经被移动到了由 Location 头部指定的 url 上，是固定的不会再改变。搜索引擎会根据该响应修正。

> 尽管标准要求浏览器在收到该响应并进行重定向时不应该修改 http method 和 body，但是有一些浏览器可能会有问题。所以最好是在应对 GET 或 HEAD 方法时使用 301，其他情况使用 308 来替代 301

### 302 Found
临时重定向。重定向状态码表明请求的资源被暂时的移动到了由 Location 头部指定的 URL 上。浏览器会重定向到这个URL，但是搜索引擎不会对该资源的链接进行更新。

> 即使规范要求浏览器在重定向时保证请求方法和请求主体不变，但并不是所有的用户代理都会遵循这一点，你依然可以看到有缺陷的软件的存在。所以推荐仅在响应 GET 或 HEAD 方法时采用 302 状态码，而在其他时候使用 307  来替代，因为在这些场景下方法变换是明确禁止的。

> 在确实需要将重定向请求的方法转换为 GET 的场景下，可以使用 303。例如在使用 PUT 方法进行文件上传操作时，需要返回确认信息（例如“你已经成功上传了xyz”）而不是上传的资源本身，就可以使用这个状态码。


### 303 See Other
通常作为 PUT 或 POST 操作的返回结果，它表示重定向链接指向的不是新上传的资源，而是另外一个页面，比如消息确认页面或上传进度页面。而请求重定向页面的方法要总是使用 GET。

### 304 Not Modified
说明无需再次传输请求的内容，也就是说可以使用缓存的内容。这通常是在一些安全的方法（safe），例如GET 或HEAD, 或在请求中附带了头部信息： If-None-Match 或If-Modified-Since。

如果返回 200，响应会带有头部 Cache-Control, Content-Location, Date, ETag, Expires，和 Vary.

#### Last-Modified 和 If-Modified-Since
1. 客户端请求一个文件（A）。 服务器返回文件A，并返回 Last-Modified。 
2. 客户端收到响应后，缓存文件A 和 Last-Modified。
3. 客户端再次请求文件A 时，发现该文件有 Last-Modified ，那么 header 离包含 If-Modified-Since，这个时间就是缓存文件的 Last-Modified。
4. 服务端收到请求，只需要判断这个时间和当前请求的文件的修改时间就可以确定是返回 304 还是 200 

If-Modified-Since 的主要缺点是只能精确到秒的级别，一旦在一秒内出现多次修改，是无法判断出已修改的状态。所以一般用在对时间不太敏感的静态资源。

#### ETag 和 If-None-Match
1. 客户端请求一个文件（A）。 服务器返回文件A，并在给A加上一个 ETag。 
2. 客户端收到响应后，并将文件连同 ETag 一起缓存。         
3. 客户再次请求文件A，会发送 If-None-Match，内容是缓存该文件A的 Etag 值
4. 服务器检查该 ETag，和计算出来的 Etag 匹配，来判断文件是否未被修改。如果未修改就直接返回 304 和一个空的响应体。否则返回 200 和 文件。

当与  If-Modified-Since  一同使用的时候，If-None-Match 优先级更高（假如服务器支持的话）

### 307 Temporary Redirect
临时重定向。类似 302，区别在于能够确保请求方法和消息主体不会发生改变。

### 308  Permanent Redirect
永久重定向。类似 301，区别在于能够确保请求方法和消息主体不会发生改变。


### 400 Bad Request
表示由于语法无效，服务器无法理解该请求。客户端不应该在未经修改的情况下重复此请求。

### 401 Unauthorized
说明由于缺乏目标资源要求的身份验证凭证，发送的请求未得到满足。

这个状态码会与 WWW-Authenticate 首部一起发送，其中包含有如何进行验证的信息。

#### WWW-Authenticate 和 Authorization
WWW-Authenticate 定义了应该用来访问资源的认证方法。语法：
``` 
WWW-Authenticate: <type> realm=<realm> 
```
- <type> : Authentication type. A common type is "Basic".
- realm=<realm> : A description of the protected area. If no realm is specified, clients often display a formatted hostname instead.

例：
```
WWW-Authenticate: Basic

WWW-Authenticate: Basic realm="Access to the staging site", charset="UTF-8"
```

Authorization 请求消息头含有服务器用于验证用户代理身份的凭证，通常会在服务器返回401 Unauthorized 状态码以及WWW-Authenticate 消息头之后在后续请求中发送此消息头。语法：
```
Authorization: <type> <credentials>
```
- <type> : Authentication type. A common type is "Basic".
- <credentials> :
    1. The username and the password are combined with a colon (aladdin:opensesame).
    2. The resulting string is base64 encoded (YWxhZGRpbjpvcGVuc2VzYW1l).


### 403 Forbidden
指的是服务器端有能力处理该请求，但是拒绝授权访问。进入该状态后，不能再继续进行验证。该访问是永久禁止的，并且与应用逻辑密切相关（例如不正确的密码）


### 404 Not Found
说明服务器端无法找到所请求的资源。返回该响应的链接通常称为坏链（broken link）或死链（dead link），它们会导向链接出错处理

404 不能说明请求的资源是临时还是永久丢失。如果服务器知道该资源是永久丢失，那么应该返回 410 (Gone) 而不是 404 。

### 405 Method Not Allowed
表明服务器禁止了使用当前 HTTP 方法的请求。需要注意的是，GET 与 HEAD 两个方法不得被禁止，当然也不得返回状态码 405。

### 406 Not Acceptable
表示服务器端不支持 Accept、Accept-Charset、Accept-Encoding、 Accept-Language header 所要求的。

#### Accept 和 Content-Type 
Accept 用来告知客户端可以处理的内容类型，这种内容类型用 [MIME](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types) 类型来表示。服务器从中选择一项进行应用，并使用 Content-Type 应答头通知客户端。
```
Accept: <MIME_type>/<MIME_subtype>
Accept: <MIME_type>/*
Accept: */*

// Multiple types, weighted with the quality value syntax:
Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8
```
- <MIME_type>/<MIME_subtype> : A single, precise MIME type, like text/html.
- <MIME_type>/* :  A MIME type, but without any subtype. image/* will match image/png, image/svg, image/gif and any other image types.
- */* : Any MIME type
- ;q= (q-factor weighting) 

#### Accept-Encoding 和 Content-Encoding
Accept-Encoding 会将客户端能够理解的内容编码方式——通常是某种压缩算法——进行通知。服务端会从中选择一个使用，并在响应报文首部 Content-Encoding 中通知客户端。
```
Accept-Encoding: deflate, gzip;q=1.0, *;q=0.5

Content-Encoding: gzip
```
### 409 Conflict
表示请求与服务器端目标资源的当前状态相冲突。

冲突最有可能发生在对 PUT 请求的响应中。例如，当上传文件的版本比服务器上已存在的要旧，从而导致版本冲突的时候，那么就有可能收到状态码为 409 的响应。

### 410 Gone
说明请求的内容在服务器上不存在了，同时是永久性的丢失。如果不清楚是否为永久或临时的丢失，应该使用404。

### 413 Payload Too Large
表示请求主体的大小超过了服务器愿意或有能力处理的限度，服务器可能会（may）关闭连接以防止客户端继续发送该请求。

如果“超出限度”是暂时性的，服务器应该返回  Retry-After 首部字段，说明这是暂时性的，以及客户端可以在什么时间后重试。

### 412 Precondition Failed
表示客户端错误，意味着对于目标资源的访问请求被拒绝。这通常发生在采用除 GET 和 HEAD 之外的方法进行条件请求时，由首部字段 If-Unmodified-Since 或 If-None-Match 规定的先决条件不成立的情况下。

### 414  URI Too Long
表示客户端所请求的 URI 超过了服务器允许的范围。

### 431 Request Header Fields Too Large
表示由于请求中的首部字段的值过大，服务器拒绝接受客户端的请求。客户端可以在缩减首部字段的体积后再次发送请求。

该响应码可以用于首部总体体积过大的情况，也可以用于单个首部体积过大的情况。

### 500 Internal Server Error
表示所请求的服务器遇到意外的情况并阻止其执行请求。
 
举例：代码语法错误；php代码运行内存超了内存限制 memory_limit；nginx config 配置错误；


### 501 Not Implemented 
表示request header 里的 method 或 Content-* 时不被服务器支持，无法被处理。另，服务器必须支持的方法（即不会返回这个状态码的方法）只有 GET 和 HEAD。501 响应默认是可缓存的。

### 502 Bad Gateway
表示作为网关或代理角色的服务器，从上游服务器（如tomcat、php-fpm）中接收到的响应是无效的。

举例： php-fpm 挂掉了

### 503 Service Unavailable
表示服务器尚未处于可以接受请求的状态。通常造成这种情况的原因是由于服务器停机维护或者已超载。该种响应应该用于临时状况下，与之同时，在可行的情况下，应该在 Retry-After 首部字段中包含服务恢复的预期时间。

举例：服务器停机维护时，主动用503响应请求；nginx 设置限速之类的，超过限速，会返回503。

#### Retry-After
表示用户代理需要等待多长时间之后才能继续发送请求。这个首部主要应用于以下几种场景：
- When sent with a 503 response, this indicates how long the service is expected to be unavailable.
- When sent with a 429 response, this indicates how long to wait before making a new request.
- When sent with a redirect response, such as 301, this indicates the minimum time that the user agent is asked to wait before issuing the redirected request.

语法：
```
Retry-After: <http-date>
Retry-After: <delay-seconds>
```
- <http-date> : A date after which to retry.
- <delay-seconds> : A non-negative decimal integer indicating the seconds to delay after the response is received.

例：
```
Retry-After: Wed, 21 Oct 2015 07:28:00 GMT
Retry-After: 120
```

### 504 Gateway Timeout
表示网关或者代理的服务器无法在规定的时间内获得想要的响应。

举例：代码执行时间超时，或死循环了。


## 问题
#### 500 和 503 分别表示什么，以及哪些情况下会用到。
#### 401 和 403 的区别
1. 401 Unauthorized 用于丢失或错误的身份验证，响应头包含 www-Authenticate 来描述如何验证，通常由 Web 服务器 返回，而不是应用程序，具有暂时性，服务器会要求重试
2. 403 Forbidden 是指已经验过身份验证，但是在某项请求资源操作上没有权限，具有永久性，与应用程序逻辑相关。
3. 参考 [403 Forbidden vs 401 Unauthorized HTTP responses](https://stackoverflow.com/questions/3297048/403-forbidden-vs-401-unauthorized-http-responses)




---
参考自
- [Etag 和 If-None-Match](https://www.cnblogs.com/xuzhudong/p/8339853.html)
- [HTTP 响应代码](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)
- [HTTP Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)