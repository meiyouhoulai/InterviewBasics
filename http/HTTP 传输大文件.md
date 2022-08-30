## HTTP 传输大文件

> 2019/9/23

#### 数据压缩

Accept-Encoding 字段标记的是客户端支持的压缩格式，例如上面说的 gzip、deflate 等，同样也可以用 “,” 列出多个，服务器可以选择其中一种来压缩数据，实际使用的压缩格式放在响应头字段 Content-Encoding 里。

```html
Accept-Encoding: gzip, deflate, br
Content-Encoding: gzip
```

如果请求报文中没有 Accept-Encoding 字段，就表示客户端不支持压缩数据；

如果响应报文中没有 Content-Encoding 字段，就表示响应数据没有被压缩。

注意：gzip 等压缩算法通常只对文本文件有很好的压缩率，而图片、音频、视频等多媒体数据本身就已经高度压缩，再用 gzip 也不会变小。

#### 分块传输

在响应报文里面用 **Transfer-Encoding: chunked** 来表示，意思是报文里面的 body 不是一次性发过来的，而是分成了许多块（chuck），以**流式**的方式一块一块发送数据，客户端无法知道全部数据的大小，客户端完全接收后，再将数据拼接起来。

Transfer-Encoding: chunked 和 Content-Length 是互斥的，也就是说响应报文里面两个字段不能同时出现，一个响应报文要么长度已知，要么长度未知（chucked）。

分块传输的编码规则：

1. 每个块包含两部分，长度和数据块
2. 长度是一个 16 进制的数字，最后以 CRLF 结尾
3. 数据紧跟在长度后面，最后也用 CRLF 结尾，但是不包括 CRLF
4. 最后用一个长度为 0 的快表示结束

![](../resource/image/25e7b09cf8cb4eaebba42b4598192410.png)

浏览器在收到分块传输的数据之后会自动按照规则去掉分块编码，将各个 chunked data 重新组装出内容。

#### 范围请求

范围请求的作用是获取一个大文件中的片段数据，看视频时候滑动快进和多线程断点续传也是这个原理。

服务器需要在响应头中使用 **Accept-Ranges: bytes** 明确告诉客户端自己是支持范围请求的，如果不支持的话可以发送 **Accept-Ranges: none** 或者干脆不返回 **Accept-Ranges** 字段

**Ranges:bytes=x-y** 是 HTTP 范围请求的专用字段，其中 x 和 y 是以字节为单位的数据范围（从 0 开始）

Ranges 的格式也非常灵活，假设文件有 100 个字节：

- “0-” 表示从文件的起点到终点，即整个文件，相当于 0-99
- “10-” 表示从文件的第 10 个字节到文件末尾，相当于 10-99
- ”-1“ 表示文件最后一个字节，相当于 99-99
- ”-10“ 表示文件倒数 10 个字节 相当于 90-99

服务器收到 Range 后，需要做四件事：

1. 检查范围是否合法，比如文件只有 200 个字节，但请求是 ”200-300“，服务器会返回 416 表示请求范围有误
2. 如果范围正确，服务器可以根据 Range 计算偏移量，返回状态码 206，表示 body 只是原数据的一部分。
3. 服务器需要添加一个响应字段 **Content-Range:bytes x-y/length**，其中 length 表示总长度。
4. 使用 TCP 将片段数据发送给客户端。

范围请求的要点：

1. 先发个 HEAD 请求，看服务器是否支持范围请求，同时获取文件大小
2. 开 N 个线程，每个线程使用 Range 字段划分出各自负责的下载片段，去下载数据
3. 下载过程中记录下载的字节数，下载中断后根据记录重新用 Range 请求剩下的数据

#### 多段数据

多段数据就是在一个请求中，同时请求多个 Range。

范围请求一次只能请求一个片段，其实它还支持 Range 头里面使用多个 x-y，一次性获取多个片段的数据。

这是时候需要一种特殊的 MIME 类型 **multipart/byteranges**，表示报文的 body 是由多段数据组成，并且还需要一个 **boundary=xxx** 给出段之间的分隔标记。

多段数据由 4 部分组成：

1. 分隔符
2. 响应头
3. 空行
4. 分段的数据

![](../resource/image/fffa3a65e367c496428f3c0c4dac8a37.png)

每一个分段必须以 “- -boundary” 开始（前面加两个 “-” ），之后要用 “Content-Type” 和 和 “Content-Range” 标记这段数据的类型和所在范围，然后就像普通的响应头一样以回车换行结束，再加上分段数据，最后用一个“- -boundary- -”（前后各有两个“-”）表示所有的分段结束。

请求：

```html
GET /16-2 HTTP/1.1
Host: www.chrono.com
Range: bytes=0-9, 20-29
```

响应：

```html
HTTP/1.1 206 Partial Content
Content-Type: multipart/byteranges; boundary=00000000001
Content-Length: 189
Connection: keep-alive
Accept-Ranges: bytes


--00000000001
Content-Type: text/plain
Content-Range: bytes 0-9/96

// this is
--00000000001
Content-Type: text/plain
Content-Range: bytes 20-29/96

ext json d
--00000000001--
```

--00000000001 就是多段分隔符，使用它客户端能够很容易的区分出多段的 Range 数据。

**这个通常用于分段下载数据，一次可以批量下载，不用发多个请求。**

#### 问题

1. 分块传输数据的时候，如果数据中有 CRLF 是否会影响数据的分块处理呢？

   不会影响，因为有分块长度 length 这个字段

2. 如果对一个被 gzip 的文件执行范围请求，比如 “Range: bytes=10-19”，那么这个范围是应用于原文件还是压缩后的文件呢？

   Range 范围请求是针对原文件，也就是说在范围请求后，服务器先给出分段数据，然后再压缩传输，客户端收到压缩数据后再解压。