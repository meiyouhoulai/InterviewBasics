## HTTP 的 Body

> 2019/9/23

#### MIME Type

- text：文本格式的数据，text/html 超文本文档、text/plain 纯文本、text/css 样式表
- image：图像文件，image/git、image/jpeg、image/png
- audio/video：音频和视频数据，audio/mepg、video/map4
- application：数据格式不固定，可能是文本也可能是二进制，常见的有 application/json、application/pdf 等，如果实在不知道数据是什么，就会是 application/octet-stream 即不透明的二进制数据。

客户端使用 Accept 字段表示对内容的支持能力，服务端需要在这些选项中挑一个最合适的返回给客户，防止服务器发过来的数据不认识。

Content-Type 用来表示请求体的数据类型。

```html
Accept: text/html,application/xml,image/webp,image/png
Content-Type:application/json;charset=UTF-8
```

#### Encoding Type

HTTP 在传输数据的时候，有时候会压缩数据，Encoding Type 指定了压缩数据的编码格式

- gzip：GNU zip 压缩格式，也是最流行的压缩格式
- deflate：zlib（deflate）压缩格式，流行程度仅次与 gzip
- br：一种专门为 HTTP 优化的新的压缩算法（Brotli）

Accept-Encoding 字段标记的是客户端支持的压缩格式，例如上面说的 gzip、deflate 等，同样也可以用 “,” 列出多个，服务器可以选择其中一种来压缩数据，实际使用的压缩格式放在响应头字段 Content-Encoding 里。

```html
Accept-Encoding: gzip, deflate, br
Content-Encoding: gzip
```

如果请求报文中没有 Accept-Encoding 字段，就表示客户端不支持压缩数据；

如果响应报文中没有 Content-Encoding 字段，就表示响应数据没有被压缩。

