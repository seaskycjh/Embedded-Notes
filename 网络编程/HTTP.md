## HTTP

[TOC]

### 1 HTTP简介

HTTP协议（Hyper Text Transfer Protocol）即超文本传输协议，属于应用层的面向对象的协议，基于TCP/IP通信协议来传递数据。

**HTTP三点注意事项**：

- 支持客户/服务器模式。
- 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。
- 灵活：允许传输任意类型的数据对象。
- 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。
- 无状态：是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

- 媒体独立的：这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型。

### 2 HTTP消息结构

#### 2.1 客户端请求消息

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：请求行（request line）、请求头部（header）、空行和请求数据四个部分组成。

```http
GET /hello.txt HTTP/1.1
User-Agent: 
Host: www.example.com
Accept-Language: CN
```

#### 2.2 服务器响应消息

也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。

```http
HTTP/1.1 200 OK
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache
Content-Length: 51
Content-Type: text/plain
```

### 3 客户端请求消息

#### 3.1 请求行

格式如下：`Method Request-URI HTTP-Version CRLF`

**Method**：请求方法。

| 方法    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| GET     | 请求指定的页面信息，并返回实体主体。                         |
| HEAD    | 类似于 GET 请求，只不过返回的响应中没有具体的内容，用于获取报头 |
| POST    | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中，可能会导致新的资源的建立和/或已有资源的修改。 |
| PUT     | 从客户端向服务器传送的数据取代指定的文档的内容。             |
| DELETE  | 请求服务器删除指定的页面。                                   |
| CONNECT | HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。    |
| OPTIONS | 允许客户端查看服务器的性能。                                 |
| TRACE   | 回显服务器收到的请求，主要用于测试或诊断。                   |
| PATCH   | 是对 PUT 方法的补充，用来对已知资源进行局部更新 。           |

**Request-URI**：URL是一种特殊类型的URI，包含了用于查找某个资源的足够的信息。

```http
http://host[:port][abs_path]
http：通过HTTP协议来定位网络资源。
host：合法的Internet主机域名或者IP地址。
port：指定一个端口号，为空则使用缺省端口80。
abs_path：指定请求资源的URI。
```

**HTTP-Version**：表示服务器HTTP协议的版本。

**CRLF**：表示换行符，即\r\n。

#### 3.2 请求头部

请求头部由多个报头域组成，报头域的格式为：`Name: Value`。请求报头允许客户端向服务器端传递请求的附加信息以及客户端自身的信息。

| 报头域          | 说明                                                 |
| --------------- | ---------------------------------------------------- |
| Accept          | 用于指定客户端接受哪些类型的信息                     |
| Accept-Charset  | 用于指定客户端接受的字符集                           |
| Accept-Encoding | 用于指定可接受的内容编码                             |
| Accept-Language | 用于指定一种自然语言                                 |
| Authorization   | 用于证明客户端有权查看某个资源。                     |
| Host            | 用于指定被请求资源的Internet主机和端口号             |
| User-Agent      | 允许客户端将它的操作系统、浏览器和其它属性告诉服务器 |

### 4 服务器响应消息

#### 4.1 状态行

格式如下：`HTTP-Version Status-Code Reason-Phrase CRLF`

**HTTP-Version**：表示服务器HTTP协议的版本。

**Status-Code**：表示服务器发回的响应状态码，由三位数字组成，第一个数字定义了响应的类别。

| 分类 | 描述                                           |
| ---- | ---------------------------------------------- |
| 1**  | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2**  | 成功，操作被成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

**Reason-Phrase**：表示状态代码的文本描述。

**CRLF**：表示换行符，即\r\n。

#### 4.2 消息报头

消息报头也由多个报头域。

| 响应头           | 说明                                           |
| ---------------- | ---------------------------------------------- |
| Content-Encoding | 文档的编码（Encode）方法。                     |
| Content-Language | 描述了资源所用的自然语言。                     |
| Content-Length   | 表示内容长度。                                 |
| Content-Type     | 表示后面的文档属于什么MIME类型。               |
| Date             | 当前的GMT时间。                                |
| Last-Modified    | 用于指示资源的最后修改日期和时间               |
| Location         | 用于重定向接受者到一个新的位置                 |
| Refresh          | 表示浏览器应该在多少时间之后刷新文档，以秒计。 |
| Server           | 服务器名字。                                   |
| Set-Cookie       | 设置和页面关联的Cookie。                       |

**Content-Type**：

| 文件扩展名 | Content-Type                | 文件扩展名 | Content-Type                  |
| ---------- | --------------------------- | ---------- | ----------------------------- |
| .bmp       | application/x-bmp           | .exe       | application/x-msdownload      |
| .gif       | image/gif                   | .html      | text/html                     |
| .img       | application/x-img           | .jpeg      | image/jpeg                    |
| .jpg       | image/jpeg                  | .mp3       | audio/mp3                     |
| .mp4       | video/mpeg4                 | .pdf       | application/pdf               |
| .png       | image/png                   | .ppt       | application/vnd.ms-powerpoint |
| .rpm       | audio/x-pn-realaudio-plugin | .svg       | text/xml                      |
| .txt       | text/plain                  | .wav       | audio/wav                     |
| .wmv       | video/x-ms-wmv              | .xml       | text/xml                      |







