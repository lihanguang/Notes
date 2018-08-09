### TCP连接三次握手
* 发送端先发送一个带有SYN标志数据包
* 接收端接收到之后向发送端发送带有SYN/ACK标志的数据包
* 发送端在发送一个带有ACK标志的数据包
* 发送端可以开始传输数据

### 统一资源标识符URI
* http://user:pass@www.example.com:80/dir/index.html?uid=111
* 协议方案名：http
* 登录信息：user:pass
* 服务器地址：www.example.com
* 服务器端口号：80
* 文件路径：dir/index.html
* 查询字符串：?uid=111

### HTTP的请求方式
* GET：请求获取一个资源
* POST：传输一段文本
* PUT：上传一个文件
* DELELTE：删除一个文件
* HEAD：获取相应头
* OPTIONS：询问支持的方法
* TRACE：追踪该路径
* CONNECT：要求用隧道协议连接代理

### HTTP持久连接
* 最初始的版本是么没有持久连接的，每一次HTTP通信都要断开一次TCP连接，HTTP/1.1之后可以使用keep-alive或HTTP connection reuse来持久连接。

### 压缩传输内容编码
* 可以使用压缩算法来对传输内容进行压缩，以减少传输内容的大小，但这样会增加CPU的负担，常用的压缩算法有
    * gzip(GUN zip)
    * compress (UNIX系统的标准压缩)
    * deflate(zlib)
    * identity(不进行编码)
    
### 分块传输
* multipart/form-data在Web表单文件上传时使用，使用boundary来进行分块
* multipart/byteranges请求某个范围的数据，使用方法
```
Range:bytes=500-1000 //请求500到1000这段数据
Range:bytes=500-  //请求500到后面所有数据
Range:bytes=-3000, 5000-7000 //请求0-3000和5000-7000这两段数据
```

### HTTP通用请求字段

##### Cache-Control
* no-cache：客户端不会接收缓存过的响应，并不是说不缓存，只是要求缓存服务器向源服务器发送客户端的请求
* no-store：请求和响应包含机密信息，不能保存到磁盘当中
* max-age：最大的缓存时间，超过这个时间就不再使用max-age=0时表示请求最新资源
* min-fresh：期望缓存在这段时间内是有效的
* max-stale：表示即使缓存过期也会接收，若不指定时间则无论多久都会接收，若指定时间则即使过期只要在这段时间内也会接收。
* only-if-cached：表示只接收缓存资源
* no-transform：代理不可更改媒体类型，防止图片压缩之类的

### HTTP响应字段

##### Cache-Control
* no-cache：缓存服务器不能对资源进行缓存
* no-store：请求和响应包含机密信息，不能保存到磁盘当中
* max-age：响应中使用该字段时缓存服务器在这个段时间内不会去确认资源的有效性
* s-maxage：公共缓存服务器响应的最大Age值
* must-revalidate：表示使用缓存的时候需要向源服务器进行验证
* proxy-revalidate：要求中间缓存服务器对缓存的响应有效性再进行确认。
* public：可向任意方提供响应的缓存
* private：仅向特定用户返回响应
* no-transform：代理不可更改媒体类型，防止图片压缩之类的