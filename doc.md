# 浅淡 HTTP 基础

## 一、HTTP 版本历史

* ### ***1.1*** HTTP/0.9

    HTTP 是基于 TCP/IP 协议的应用层协议。它不涉及数据包（packet）传输，主要规定了客户端和服务器之间的通信格式，默认使用80端口。

    ` GET /index.html `
    
    上面命令表示，TCP 连接（connection）建立后，客户端向服务器请求（request）网页 ` index.html `。
    
    协议规定，服务器只能回应HTML格式的字符串，不能回应别的格式。

    ```
    <html>
        <body>Hello World</body>
    </html>
    ```
    服务器发送完毕，就关闭TCP连接。
    
    ---
* ### ***1.2*** HTTP/1.0

    1. 简介

    
        1996年5月，HTTP/1.0 版本发布，内容大大增加。

        首先，任何格式的内容都可以发送。这使得互联网不仅可以传输文字，还能传输图像、视频、二进制文件。这为互联网的大发展奠定了基础。

        其次，除了GET命令，还引入了POST命令和HEAD命令，丰富了浏览器与服务器的互动手段。

        再次，HTTP请求和回应的格式也变了。除了数据部分，每次通信都必须包括头信息（HTTP header），用来描述一些元数据。

        其他的新增功能还包括状态码（status code）、多字符集支持、多部分发送（multi-part type）、权限（authorization）、缓存（cache）、内容编码（content encoding）等

    2. 请求格式

        下面是一个1.0版的HTTP请求的例子。

        ```
        GET / HTTP/1.0
        User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5)
        Accept: */*
         ```

        可以看到，这个格式与0.9版有很大变化。

        第一行是请求命令，必须在尾部添加协议版本`（HTTP/1.0）`。后面就是多行头信息，描述客户端的情况。

    3. 响应格式
            
            HTTP/1.0 200 OK 
            Content-Type: text/plain
            Content-Length: 137582
            Expires: Thu, 05 Dec 1997 16:00:00 GMT
            Last-Modified: Wed, 5 August 1996 15:55:28 GMT
            Server: Apache 0.84

            <html>
            <body>Hello World</body>
            </html

        响应的格式是" 头信息 + 一个空行`（\r\n）` + 数据"。其中，第一行是"协议版本 + 状态码（status code） + 状态描述"。

    ---
* ### ***1.3*** HTTP/1.1

    1997年1月，HTTP/1.1 版本发布，只比 1.0 版本晚了半年。它进一步完善了 HTTP 协议，一直用到了20年后的今天，直到现在还是最流行的版本。

    1. 持久连接

        1.1 版的最大变化，就是引入了持久连接`（persistent connection）`，即TCP连接默认不关闭，可以被多个请求复用，不用声明 `Connection: keep-alive`。

        客户端和服务器发现对方一段时间没有活动，就可以主动关闭连接。不过，规范的做法是，客户端在最后一个请求时，发送 `Connection: close `，明确要求服务器关闭TCP连接

        ```
        Connection: close
        ```
        
        目前，对于同一个域名，大多数浏览器允许同时建立6个持久连接。
    
    2. 新增其他功能

        1.1版还新增了许多动词方法：`PUT`、`PATCH、HEAD`、 `OPTIONS`、`DELETE`。

        另外，客户端请求的头信息新增了 `Host` 字段，用来指定服务器的域名。

        ```
        Host: www.example.com
        ```

        有了 `Host` 字段，就可以将请求发往同一台服务器上的不同网站，为虚拟主机的兴起打下了基础。
* ### ***1.4*** HTTP/2.0 
    2015年，HTTP/2 发布。它不叫 HTTP/2.0，是因为标准委员会不打算再发布子版本了，下一个新版本将是 HTTP/3。 
    
    由于文章长度原因，这里就不做过多描述感兴趣可以看 [HTTP/2 Specification](https://http2.github.io/http2-spec/)（英文）

## 二、HTTP 协议详解

**`HTTP` 请求由 请求行，请求头（Requset Header），响应头（Response Header），请求正文三部分构成**

* ### ***2.1*** URL
    ```
    http://host[":"port][abs_path]
    ```
    * 以上表示要通过 `HTTP` 协议来定位网络资源。
    
    * `host` 表示合法的 `Internet` 主机域名或者IP地址。
    
    * `port` 指定一个端口号，为空则使用缺省端口 `80`，`abs_path` 指定请求资源的URI。
* ### ***2.2*** 请求行

    ```
    GET /example.html HTTP/1.1 (CRLF)
    ```

    HTTP 协议常见的方法 `Method`

    * `GET` 请求获取Request-URI所标识的资源
    * `POST` 在Request-URI所标识的资源后附加新的数据
    * `HEAD` 请求获取由Request-URI所标识的资源的响应消息报头
    * `PUT` 请求服务器存储一个资源，并用Request-URI作为其标识
    * `DELETE` 请求服务器删除Request-URI所标识的资源

* ### ***2.2*** 请求头  Requset


    报头由一系列的键值对组成，允许客户端向服务器端发送一些附加信息或者客户端自身的信息，常见格式

    * `Accept` 请求报头域用于指定客户端接受哪些类型的信息，下面是常见字段的值。

        ```
        * text/plain
        * text/html
        * text/css
        * image/jpeg
        * image/png
        * image/svg+xml
        * audio/mp4
        * video/mp4
        * application/javascript
        * application/pdf
        * application/zip
        * application/atom+xml
        ```
    * `Accept-Encoding`  向服务器申明客户端（浏览器）接收的编码方法，通常为压缩方法
        ```
        Accept-Encoding: gzip, deflate, br
        ```
    * `Accept-Language` 向服务器申明客户端（浏览器）接收的语言
        ```
        Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
        ```
    * `Cache-control` 控制浏览器的缓存
        * 常见的值有：`private`、`no-cache`、`max-age`、`alidate`，默认为 `private`
    * `Refer` 告诉服务器该页面 来源于 哪个页面的链接
        * 值：[https://www.baidu.com]
    * `User-agent` 告诉客户端将它的操作系统、浏览器和其它属性告诉服务器，常见的统计工具 就是从这个字段读取所需的信息，常见值如下：
        ```
        Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36
        ```
