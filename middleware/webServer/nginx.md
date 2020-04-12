# Nginx

## 架构

### 多进程模型

Nginx 在启动后，会有一个 master 进程和多个 worker 进程。

#### master 进程

master 进程主要用来管理 worker 进程，包含：

- 接收来自外界的信号，向各 worker 进程发送信号
- 监控 worker 进程的运行状态
- 当 worker 进程退出后(异常情况下)，会自动重新启动新的 worker 进程。

而基本的网络事件，则是放在 worker 进程中来处理了。

#### woker 进程

多个 worker 进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。一个请求，只可能在一个 worker 进程中处理，一个 worker 进程，不可能处理其它进程的请求。一般 worker 个数与 cpu 核数一致

#### 如何处理请求

每个 worker 进程都是从 master 进程 fork 过来，在 master 进程里面，先建立好需要 listen 的 socket（listenfd）之后，然后再 fork 出多个 worker 进程。所有 worker 进程的 listenfd 会在新连接到来时变得可读，为保证只有一个进程处理该连接，所有 worker 进程在注册 listenfd 读事件前抢 accept_mutex，抢到互斥锁的那个进程注册 listenfd 读事件，在读事件里调用 accept 接受该连接。当一个 worker 进程在 accept 这个连接之后，就开始读取请求，解析请求，处理请求，产生数据后，再返回给客户端，最后断开连接

#### 优点
  - 利用多核系统的并发处理能力

  - 负载均衡

  - 管理监控工作进程状态

### 事件模型

- 异步非阻塞

	- poll, select, epoll等

### 事件

- 网络事件

- 信号

- 定时器

	- 红黑树

### 内存池设计

- 避免内存碎片
- 减少向操作系统申请内存次数

## 基础数据结构

### ngx_str_t

- 带长度的字符串

### ngx_pool_t

- 管理使用资源并统一释放

### ngx_array_t

- 数组

### ngx_hash_t

- hash 表

### ngx_hash_wildcard_t

- 处理带有通配符的域名匹配，hash表

### ngxI_chain_t

- 链表

### ngx_list_t

- list

### ngx_queue_t

- 双向链表

### ngx_rbtree_t

- 红黑树

## 特点

### 处理静态文件

### 无缓存反向代理，简单的负载均衡

### FastCGI

### 模块化结构

### 支持SSL 和 TLSSNI

## 模块

### 分类

- phase handle

	- 处理请求

- output filter

	- 输出过滤

- upstream

	- 反向代理

- load balancer

	- 负载均衡

- event module

	- 事件处理

## 配置

### nginx 服务的基本配置

- 是否以守护进程方式运行Nginx

	- `daemon on|off;`
	  默认： `daemon on;`

### web 服务器基础配置

- 虚拟主机与请求分发

	- 监听端口

		- `listen address:port`
		  默认：`listen 80;`

	- 主机名称

		- `server_name name [...]`
		  默认: `server_name ""`

		- 匹配优先级
		  如果host与所有的server_name都不匹配，则会如下处理：  
		  1）优先选择listen配置项后加 default|default_server  的server块  
		  2）找到匹配listen端口第一个server块
- 1.首先选择所有字符串匹配的
			  如：`www.baidu.com`
	
- 2.其次选择通配符在前面的
			  如：`*.baidu.com`
	
- 3.再选择通配符在后面的
			  如：`www.baidu.*`
	
- 4.最后选择使用正则才能匹配的
			  如：`~^\.baidu\.com$`
	
- location
	
	- `location [=|~|~*|^~|@] /uri/ {...}`
	
	- 匹配规则
	
		- `=`
			  表示把URI作为字符串，以便与参数中的uri做完全匹配。  
			  如：  
			
			  ```
			  location = / {  
			  	#只有当用户请求是/ 时，才会使用该配置  
			  }
			  ```
			- `~`
			  表示匹配URI时，字母大小敏感
			- `~*`
			  表示匹配URI时，忽略字母大小写问题
	
  		- `^~`
	表示匹配URI时，只需要将前半部分与uri参数匹配即可。  
			  如： 
        ```
			  location ^~ images {  
			  	#以images开始的请求都会匹配上  
			  }
			```
			- `@`
		  表示仅用于nginx服务器内部请求间的重定向，不直接处理用户请求
	
		- uri 可用正则表达式
	
- 当匹配到多个，走第一个location
	
- 文件路径的定义

  - 以root方式设置资源路径

    - `root path`
      默认：`root html;`  
      例：  
      
      ```
      location /download/ {  
       	root optwebhtml;  
      }  
      ```
      如果一个请求的URI是 /download/index/test.html，那么将返回服务器上optwebhtml/download/index/test.html 文件的内容

  - 以alias方式设置资源路径

    - `alias path`
      例：  
      
      ```
      location conf {  
      	alias usr/local/nginx/conf;  
      }  
      ```
      如果请求的URI是/conf/nginx.conf，用户将实际访问usr/local/nginx/conf/nginx.conf  
      
      alias 可以添加正则匹配表达式  
      例： 
      ```
      location ~ ^/test/(\w+)\.(\w+)$ {  
      	alias usr/local/nginx/$2/$1.$2;  
      }  
      ```
      当用户请求的URI是 /test/nginx.conf时，nginx会返回usr/local/nginx/conf/nginx.conf文件中的内容

  - 访问首页

  	- `index file...`
  	  默认：`index index.html;  `
  	  
  	  例：  
  	  ```
  	  server {  
  	      listen      80;  
  	      server_name example.org www.example.org;      
  	        
  	      location / {  
  	          root    /data/www;  
  	          index   index.html index.php;  
  	      }  
  	        
  	      location ~ \.php$ {  
  	          root    /data/www/test;  
  	      }  
  	  }  
  	  ```
  	  
  	  上面的例子中，如果你使用example.org或www.example.org直接发起请求，那么首先会访问到“/”的location，结合root与index指令，会先判断/data/www/index.html是否存在，如果不，则接着查看  
  	  /data/www/index.php ，如果存在，则使用/index.php发起内部重定向，就像从客户端再一次发起请求一样，Nginx会再一次搜索location，毫无疑问匹配到第二个~ \.php$，从而访问到/data/www/test/index.php。  
  	  
  	  https://blog.csdn.net/qq_32331073/article/details/81945134

  - 根据http返回码重定向页面

  	- `error_page code[code...][=|=answer-code]uri|@named_location`

  - try_files

  	- `try_files path1[path2]uri`
  	  https://www.hi-linux.com/posts/53878.html

- 内存及磁盘资源的分配

- 网络连接的配置

- MIME类型的设置

  - MIME type与文件扩展的映射

  	- `type{...};`
  	  例：  
  	  
  	  ```
  	  types {  
  	  	text/html	html;  
  	  	text/html	conf;  
  	  	image/gif	gif;  
  	  	image/jpeg	jpg;  
  	  }
  	  ```
  - 默认MIME type

  	- `default_type MIME-type;`
  	  默认： `default_type text/plain;`

- 对客户端请求的限制

  - 按HTTP方法名限制用户请求

  	-  `limit_except method...{...}`
  	  Nginx通过limit_except后面指定的方法名来限制用户请求。  
  	  方法名可取值包括：GET、HEAD、POST、PUT、DELETE、MKCOL、COPY、MOVE、OPTIONS、PROPFIND、PROPPATCH、LOCK、UNLOCK或者PATCH。  
  	  例：  
  	  
  	  ```
  	  limit_except GET {  
  	  	allow 192.168.1.0/32;  
  	  	deny	all;  
  	  }  
  	  ```
  	  注意，允许GET方法就意味着也允许HEAD方法。因此，上面这段代码表示的是禁止GET方法和HEAD方法，但其他HTTP方法是允许的。

- 文件操作的优化

- 对客户端请求的特殊处理

	- 对If-Modified-Since头部的处理策略

		-  `if_modified_since[off|exact|before]`
		  默认：` if_modified_since exact;`

		- 参数

			- off
			  表示忽略用户请求中的If-Modified-Since头部。这时，如果获取一个文件，那么会正常地返回文件内容。HTTP响应码通常是200

			- exact
			  将If-Modified-Since头部包含的时间与将要返回的文件上次修改的时间做精确比较  
			  如果没有匹配上，则返回200和文件的实际内容，如果匹配上，则表示浏览器缓存的文件内容已经是最新的了，没有必要再返回文件从而浪费时间与带宽了  
			  这时会返回304 Not  
			  Modified，浏览器收到后会直接读取自己的本地缓存。

			- before
			  是比exact更宽松的比较。只要文件的上次修改时间等于或者早于用户请求中的If-Modified-Since头部的时间，就会向客户端返回304 Not Modified。

	- 文件未找到时是否记录到error日志

		- `log_not_found on|off;`
		  默认： `log_not_found on;  `
		  
		  此配置项表示当处理用户请求且需要访问文件时，如果没有找到文件，是否将错误日志记录到error.log文件中。这仅用于定位问题。

	- DNS解析地址

		- `resolver address...;`
		  设置DNS名字解析服务器的地址  
		  例：  
		  `resolver 127.0.0.1 192.0.2.1;`

- 提供的变量

	- $arg_PARAMETER
	  http 请求中的某个参数值  
	  如：  
	  /index.html?size=100  
	  可用 $arg_size 取得100这个值

	- $args
	  http 请求中的完整参数  
	  如：  
	  /index.html?w=120&h=120  
	  则 $args 表示字符串 w=120&h=120

	- $uri / $document_uri
	  表示当前请求的URI，不带任何参数

	- $request_uri
	  表示客户端发来的原始请求URI，带完整参数。   
	  另，$uri 和 $document_uri 有可能是内部重定向后的URI，而$request_uri永远是客户端的原始URI

	- $request_method

	- $remote_addr

	- $remote_port

	- $host

### 负载均衡的基本配置

- upstream

  - `upstream name{...}`

  - 配置块： http
    upstream块定义了一个上游服务器的集群，便于反向代理中的proxy_pass使用。  
    
    如：  
    ```
    upstream backend {  
    	server backend1.example.com;  
    	server backend2.example.com;  
    	server backend3.example.com;  
    }  
    server {  
    	location / {  
    		proxy_pass	http://backend;  
    	}  
    }
    ```
- server_name

	- `server name[parameters];`

	- 配置块： upstream

	- 参数

		- weight=number

		- max_fails=number

		- fail_timeout=time

		- down

		- backup

- ip_hash

	- ip_hash
	  首先根据客户端的IP地址计算出一个key，将key按照upstream集群里的上游服务器数量进行取模，然后以取模后的结果把请求转发到相应的上游服务器中。这样就确保了同一个客户端的请求只会转发到指定的上游服务器中。  
	  
	  ip_hash与weight（权重）配置不可同时使用。如果upstream集群中有一台上游服务器暂  
	  
	  时不可用，不能直接删除该配置，而是要down参数标识，确保转发策略的一贯性。  
	  
	  例：  
	  ```
	  upstream backend {  
	  	ip_hash;  
	  	server	backend1.example.com;  
	  	server	backend2.example.com;  
	  	server	backend3.example.com	down;  
	  	server	backend4.example.com;  
	  }
		```
	- 配置块： upstream

### 反向代理基本配置

- proxy_pass

	- `proxy_pass URL;`
	  此配置项将当前请求反向代理到URL参数指定的服务器上，URL可以是主机名或IP地址加端口的形式  
	  例：  
	  `proxy_pass http://localhost:8000/uri/ `
	  
	  还可以如上节负载均衡中所示，直接使用upstream块  
	  ```
	  upstream backend {  
	  	…  
	  }  
	  server {  
	  	location / {  
	  		proxy_pass	http://backend;  
	  	}  
	  }
		```
	- 配置块： location、if

- proxy_method

	-  `proxy_method method;`
	  此配置项表示转发时的协议方法名。  
	  例：  
	  `proxy_method POST;  `
	  那么客户端发来的GET请求在转发时方法名也会改为POST。

	- 配置块： http、server、location

---

参考资料

- 《深入理解Nginx模块开发与架构解析》陶辉著