## HTTP 优化

> 2019/10/16

1. 购买现成的硬件，比如换上更强的 CPU、更快的网卡、更大的带宽
2. 购买外部的软件或者服务，比如 CDN
3. 选择高性能的 Web 服务器，例如 Nginx/OpenResty
4. 🍎启用 HTTP 长连接：Connection:keep-alive
5. 🍎使用 TCP Fast Open，在握手的时候就传输数据
6. 🍎启用 HTTP 压缩：Content-Encoding: gzip
7. 🍎对于图片，去除图片里的拍摄时间、地点、机型等元数据，适当降低分辨率、缩小尺寸；尽量选择高压缩的格式，有损格式应该用 JEPG，无损格式使用 Webp
8. 🍎减少 Header 的数量，不必要的字段尽量不发
9. 🍎减少 Cookie 记录的数据量，使用 domain 和 path 限定 Cookie 的作用，尽量减少 Cookie 的传输
10. DNS 解析也会花费不少时间，如果网站有多个域名，从域名解析到 IP 地址也是一个不小的成本，尽量收缩域名，限制在两三个左右，必要的时候可以使用 IP 直连。
11. 除非不要，尽量不要使用重定向。
12. 使用缓存：HTTP 自身的缓存 Cache-Control、服务器缓存、客户端缓存