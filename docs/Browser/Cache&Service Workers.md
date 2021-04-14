缓存策略与 Service Workers

传统缓存策略

- 浏览器缓存（HTTP 缓存、浏览器内置缓存[cookie、local storage、session storage]）
- 代理服务器缓存（CDN 缓存）
- 服务器缓存
- 数据库缓存



浏览器请求资源的步骤

1. 判断请求是否命中缓存，命中->2，否则->3
2. 判断缓存是否过期，没有则直接返回，过期则->3，并带上缓存
3. 向服务器请求资源
4. 服务器判断缓存，未更新则304，没有缓存信息或资源已更新则返回200，并返回资源
5. 浏览器根据响应头决定要不要存储缓存（no-store时不存储）



HTTP 缓存

- Expires 头 （HTTP1.0）

  ```
  Expires: Tue, 01 May 2021 11:37:06 GMT
  ```

  表明这个资源在这个时间点过期

- Cache-Control （优先级高于 Expires）

  ```
  Cache-Control: s-maxage=300, public, max-age=60
  ```

  可缓存性

  - public：表明响应可以被任何对象缓存
  - private：表明响应只能被单个用户缓存，不能作为共享缓存（代理服务器不能缓存），私有缓存
  - no-cache：在发布缓存副本之前，强制要求缓存把请求提交给原始服务器进行验证（协商缓存）
  - no-store：缓存不存储有关客户端请求或服务器响应的任何内容，即不适用任何缓存

  到期

  - max-age：设置缓存存储的最大周期，单位 s
  - s-maxage：表示共享缓存的时间，单位是 s，私有缓存会忽略

  重新验证和重新加载

  - must-revalidate：一旦资源过期，在重新验证之前，缓存不能用该资源响应后续请求
  - proxy-revalidate：作用同上，但仅适用于共享缓存，会被私有缓存忽略

  其他

  - no-transform：不得对资源进行转换或转变
  - only-if-cached：表明客户端只接受已缓存的响应，并且不要向原始服务器检查是否有新的拷贝

  示例：

  禁止缓存

  ```
  Cache-Control: no-store
  ```

  缓存静态资源：对于应用程序中不会改变的静态资源可以积极添加缓存

  ```
  Cache-Control:public, max-age=31536000
  ```

  需要重新验证：表明客户端可以缓存，但每次使用前必须重新验证其有效性

  ```
  Cache-Control: no-cache
  Cache-Control: max-age=0
  ```

- Last-Modified/If-Modified-Since

  ```
  Last-Modified: Sat, 01 Jan 2000 00:00:00 GMT
  If-Modified-Since: Sat, 01 Jan 2000 00:00:00 GMT
  ```

  服务器在响应请求时除了返回 Cache-Control 头外，还会返回一个 Last-Modified 头，用于指定该资源的服务器更新时间。当资源在浏览器端过期时，浏览器会就会带上缓存信息去发起请求，也就是 If-Modified-Since，这个值通常是上一次 响应返回的 Last-Modified 的值。

- Etag/If-None-Math

  ```
  ETag: W/"541-U+f4/li90Tc9Mfu1asCpRz5TFuQ"
  If-None-Match: W/"1717b-2Z1o4jh4BYhel52WPc4NycnVIPM"
  ```

  虽然 Last-Modified/If-Modified-Since 可以提供对资源时间的控制，但是某些场景也不是那么适用，比如定期产生的资源，虽然时间不同，但是内容相同，这个时候我们会期望请求的时候根据资源的内容来决定是否更新缓存。为此 HTTP1.1 规定了 Etag/If-None-Math ，他的用法也是一个用于响应，一个用于请求。只是字段的值是服务器规定的一个标签，通常是资源内容、大小、时间的 Hash 值等。他的优先级高于 Last-Modified/If-Modified-Since。

chrome 浏览器有几种特殊的请求状态 

| 状态 | size 信息        | 说明                                                         |
| ---- | ---------------- | :----------------------------------------------------------- |
| 200  | 正常显示文件大小 | 从服务器下载的最新资源                                       |
| 200  | memory cache     | 不请求网络资源，资源在内存中，当kill进程后，也就是浏览器关闭以后，数据将不存在。 |
| 200  | disk cache       | 不请求网络资源，资源在磁盘中，当kill进程时，数据还是存在。   |
| 301  | disk cache       | 资源被重定向了，但是磁盘中仍然存在，不请求网络资源，资源在磁盘中，当kill进程时，数据还是存在。 |
| 304  | 报文大小         | 请求服务器发现资源没有更新，使用本地资源                     |

