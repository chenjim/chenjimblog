
# Android性能优化--网络优化

>本文地址 <https://blog.csdn.net/CSqingchen/article/details/106671184>  
最新更新地址 <https://gitee.com/chenjim/chenjimblog>  

- 服务器合理部署  
  服务器多运营商多地部署，一般至少含三大运营商、南中北三地部署。  
  配合上面说到的动态 IP 列表，支持优先级，每次根据地域、网络类型等选择最优的服务器 IP 进行连接。  
  对于服务器端还可以调优服务器的 TCP 拥塞窗口大小、重传超时时间(RTO)、最大传输单元(MTU)等。  
- CDN 缓存静态资源  
  缓存常见的图片、JS、CSS 等静态资源。应用从最近的服务器下载，可优化下载效率
- 不用域名，用 IP 直连  
  省去 DNS 解析过程，DNS 全名 Domain Name System，解析意指根据域名得到其对应的 IP 地址。  
  首次域名解析一般需要几百毫秒，可通过直接向 IP 而非域名请求，节省掉这部分时间，同时可以预防域名劫持等带来的风险。  
  当然为了安全和扩展考虑，这个 IP 可能是一个动态更新的 IP 列表，并在 IP 不可用情况下通过域名访问。  
- 连接复用  
  可以节省连接建立时间，如开启 keep-alive。  
  Http 1.1 默认启动了 keep-alive。[OkHttpClient](https://github.com/square/okhttp) 默认开启了 keep-alive。 HttpClient 不建议使用。
- 增加连接超时  
  网络连接底层都是socket连接，增加连接超时，可以避免应用长时间等待返回结果
- 增加请求超时  
  跟连接超时类似，针对跟服务器已经连接，但是接口返回数据超时
- 减少重定向次数  
  额外的 HTTP 重定向可能会增加一次或两次额外的网络往返（如果需要再次查找 DNS 的话就是两次），这在 4G 网络上将会导致数百毫秒的额外延迟
- 开启gzip压缩  
  可以减少减少带宽，减少手机数据流量，加快传输速度，OkHttpClient 默认支持gzip自动解压
- 应尽可能减少首次呈现内容所需的网络往返次数  
  鉴于 TCP 评估连接状况的方式（即 [TCP 慢启动](http://en.wikipedia.org/wiki/Slow-start)），  
  新的 TCP 连接无法立即使用客户端和服务器之间的全部有效带宽。  
  因此，在通过新连接进行首次往返的过程中，服务器最多只能发送 10 个 TCP 数据包（约 14KB），  
  然后必须等待客户端确认已收到这些数据，才能增大拥塞窗口并继续发送更多数据。  
  另外还需注意的是，10 个数据包 ([IW10](http://tools.ietf.org/html/draft-hkchu-tcpm-initcwnd-01)) 这一限值源自 TCP 标准的最近一次更新：  
  您应确保自己的服务器已升级到最新版本，以便能够充分利用这次更新。  
  否则，这一限值可能会降低到 3-4 个数据包！  
  参考自 <https://developers.google.com/speed/docs/insights/mobile>  
- 避免频繁网络请求  
  会导致CPU无法休眠，增加手机功耗
- 使用连接池  
  OkHttpClient采用了连接池机制，实现了连接的复用，避免了每次都创建新的连接从而导致资源的浪费。
- 缓存请求结果  
  依据http头信息中cache-contrl及expires确定是否缓存请求结果，可以减少非必要的网络请求
- 选择合适的数据格式传输  
  比如 [json](https://github.com/google/gson)、[protocol](https://blog.csdn.net/CSqingchen/article/details/50669725) 等，可以减少数据量
- 使用图片缓存  
  图片都很占用带宽，在应用使用图片加载框架 [Glide](https://github.com/bumptech/glide)、[Picasso](https://github.com/square/picasso)、[Fresco](https://github.com/facebook/fresco) 等，可避免二次下载和解码  
- 对post请求的body做gzip压缩  
  可以直接在请求中开启压缩，也可以先压缩日志文件为zip后，再上传。30M的日志可以压缩到2-3M
- 请求合并   
  如果一个页面请求很多，可以考虑将多个请求合并为一个进行请求
- 精简数据格式  
  请求及返回数据中不需要的参数去掉，可以减少数据流量
- 增量更新  
  需要数据更新时，可考虑增量更新。如常见的服务端进行 bsdiff，客户端进行 bspatch。
- 优化大文件下载  
  支持断点续传，并缓存 Http Resonse 的 ETag 标识，下次请求时带上，从而确定是否数据改变过，未改变则直接返回 304。
- 预取  
  包括预连接、预取数据。
- 分优先级、延迟部分请求  
  将不重要的请求延迟，这样既可以削峰减少并发、又可以和后面类似的请求做合并。
- 多连接  
  对于较大文件，如大图片、文件下载可考虑多连接。 需要控制请求的最大并发量，毕竟移动端网络受限。
- 数据监控  
  优化需要通过数据对比才能看出效果，所以监控系统必不可少，通过前后端的数据监控确定调优效果。  

本文地址 <https://blog.csdn.net/CSqingchen/article/details/106671184>  
以上大部分参考自 <https://www.trinea.cn/android/mobile-performance-optimization>

---

https 通信流程  
<img src="https://pic2.zhimg.com/v2-2c5ead7e9e4544335d3db4e5d1d4e355_r.jpg"  />
原文<https://zhuanlan.zhihu.com/p/56663184>

---



