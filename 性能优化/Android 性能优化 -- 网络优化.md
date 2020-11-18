
- 避免频繁网络请求
- 使用线程池
- 图片必须缓存，最好一句机型做图片适配
- 所有http请求必须添加http timeout
- http 开启gzip压缩
- 选择合适的数据格式传输，比如json、[protocol](https://blog.csdn.net/CSqingchen/article/details/50669725)
- 依据http头信息中cache-contrl及expires确定是否缓存请求结果
- 确定请求的connection是否keep-alive
- 减少请求次数，适当做请求合并
- 减少重定向次数
- 不是用域名，用IP直连
- 服务器分布式部署
- 连接复用，请求合并
- CDN缓存静态资源
- 对post请求的body做gzip压缩，如日志文件上传
- 精简数据格式
- 增量更新、文件下载支持断点续传
- 预读取、分优先级、延迟部分请求、多连接

<img src="https://pic.chenjim.com/20200610171413.png?x-oss-process=style/blog" width = "600" />



https 通信流程：
<img src="https://pic.chenjim.com/20200611100618.png?x-oss-process=style/blog"  />


