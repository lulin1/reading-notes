# chapter 9 HTTP简史

## 9.1 HTTP 0.9: 只有一行的协议

简单总结一下 HTTP 0.9 的功能:

	• 客户端 / 服务器、请求 / 响应协议;
  
	• ASCII 协议,运行于 TCP/IP 链接之上;
  
	• 设计用来传输超文本文档(HTML);
  
	• 服务器与客户端之间的连接在每次请求之后都会关闭。


## 9.2 HTTP 1.0: 迅速发展及参考性 RFC

该协议的关键变化:

	• 请求可以由于多行首部字段构成;
  
	• 响应对象前面添加了一个响应状态行;
  
	• 响应对象也有自己的由换行符分隔的首部字段;
  
	• 响应对象不局限于超文本;
  
	• 服务器与客户端之间的连接在每次请求之后都会关闭。
	
HTTP 1.0 的优化策略非常简单,就一句话:升级到 HTTP 1.1。完了!


## 9.3 HTTP 1.1: 互联网标准

HTTP 1.1 标准厘清了之前版本中很多有歧义的地方,而且还加入了很多重要的性能优化: 持久连接、分块编码传输、字节范围请求、增强的缓存机制、传输编码及请求管道。

![9-other-1.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/9-other-1.jpg)

![9-other-2.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/9-other-2.jpg)

最明显的差别是这里发送了两次对象请求,一次请求 HTML 页面,一次请求图片,这两次请求都是通过一个连接完成的。这个连接是持久的,因而可以重用 TCP 连接对同一主机发送多次请求,从而实现更快的用户体验。

为终止持久连接,客户端的第二次请求通过 Connection 首部,向服务器明确发送了关闭令牌。类似地,服务器也可以在响应完成后,通知客户端自己想要关闭当前TCP 连接。从技术角度讲,不发送这个令牌,任何一端也可以终止 TCP 连接。但为
确保更好地重用连接,客户端和服务器都应该尽可能提供这个信息。

	HTTP 1.1 改变了 HTTP 协议的语义,默认使用持久连接。换句话说,除非明确告知(通过 Connection: close 首部),
	否则服务器默认会保持连接打开。
  
  
	不过,这个功能也反向移植到了 HTTP 1.0,可以通过 Connection: Keep-Alive 首部来启用。
	实际上,如果你使用的是 HTTP 1.1,从技术上说不需要 Connection: Keep-Alive 首部,但很多客户端还是选择加上它。


## 9.4 HTTP 2.0: 改进传输性能

HTTP 2.0 的主要目标是改进传输性能,实现低延迟和高吞吐量。
