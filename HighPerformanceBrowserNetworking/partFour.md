# chapter 14   浏览器网络概述

![14-1.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/14-1.jpg)


## 14.1  连接管理与优化

套接字是以池的形式进行管理的(图 14-2 )， 即按照来源,每个池都有自己的连接限制和安全约束。挂起的请求是排好队的、有优先次序的,然后再适时把它们绑定到池中个别的套接字上。除非服务器有意关闭连接,否则同一个套接字可以自动用于多个请求!

![14-2.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/14-2.jpg)


  • 来源
  
   由应用协议、域名和端口三个要件构成,比如 (http, www.example.com, 80) 与 (https, www.example.com, 443) 就是两个不同的来源。
    
  • 套接字池
  
   属于同一个来源的一组套接字。实践中,所有主流浏览器的最大池规模都是 6 个套接字。
    
    
## 14.2 网络安全与沙箱

将个别套接字的管理任务委托给浏览器还有另一个重要的用意:可以让浏览器运用沙箱机制,对不受信任的应用代码采取一致的安全与策略限制。比如,浏览器不允许直接访问原始网络套接字 API,因为这样给恶意应用向任意主机发起任意请求
(端口扫描、连接邮件服务器或发送未知消息)提供可乘之机。

  • 连接限制
    浏览器管理所有打开的套接字池并强制施加连接数限制,保护客户端和服务器的资源不会被耗尽。
    
  • 请求格式化与响应处理
    浏览器格式化所有外发请求以保证格式一致和符合协议的语义,从而保护服务器。类似地,响应解码也会自动完成,以保护用户。
    
  • TLS 协商
    浏览器执行 TLS 握手和必要的证书检查。任何证书有问题(比如服务器正在使用自已签发的证书),用户都会收到通知。
    
  • 同源策略
    浏览器会限制应用只能向哪个来源发送请求。
    
    
  
## 14.3  资源与客户端状态缓存

最后,浏览器还有一个经常被人忽视的重要功能,  那就是提供会话认证和 cookie 管理。浏览器为每个来源维护着独立的 cookie 容器,为读写新 cookie、会话和认证数据提供必要的应用及服务器 API,还会为我们自动追加和处理 HTTP 首部,让一切都自动化。

举一个简单但直观的例子,它能说明把会话状态管理委托给浏览器的好处:认证的会话可以在多个标签页或浏览器口间共享,反之亦然;如果用户在某个标签页中退出,那么其他所有打开窗口中的会话都将失效。



## 14.4  应用 API 与协议

不存在哪个协议或API最好的问题。每个稍微复杂点的应用都会基于不同的 需求用到各种传输机制,包括读写浏览器缓存、协议开销、消息延迟、可靠性、数据 传 输 类 型, 等 等。某 些 协 议 的 交 付 延 迟 可 能 短 一 些( 比 如 Server-Sent Events、WebSocket),但却不能满足其他条件,比如利用浏览器缓存,或者在所有场景下支
持高效的二进制传输(表 14-1)。

![table-14-1.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/table-14-1.jpg)


我们在这个表中有意忽略了 WebRTC,因为那是一种端到端的交付模型,与 XHR、SSE 和 WebSocket 协议有着根本的不同。

理解了每种协议的长处和短处,根据应用的需求恰当运用它们,就可以摆脱贫乏的用户体验,打造出高性能应用。

:bookmark: Web Real-Time Communication(Web 实时通信,WebRTC)由一组标准、协议和JavaScript API 组成,用于实现浏览器之间(端到端)的音频、视频及数据共享。WebRTC 使得实时通信变成一种标准功能,任何 Web 应用都无需借助第三方插件和
专有软件,而是通过简单的 JavaScript API 即可利用。



# chapter 15   XMLHttpRequest


## 15.1   XHR 简史

尽管名字里有 XML 的 X,XHR 也不是专门针对 XML 开发的。这只是因为 Internet Explorer 5 当初发布它的时候,把它放到 MSXML 库里,这才“继承”了这个 X:

![15-other-1.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/15-other-1.jpg)



## 15.2  跨源资源共享 (CORS)

XHR 是一个浏览器层面的 API,  向我们隐藏了大量底层处理,包括缓存、重定向、内容协商、认证,等等。这样做有两个目的。第一,XHR 的 API 因此非常简单,开发人员可以专注业务逻辑。 其次,浏览器可以采用沙箱机制,对应用代码强制施加一套安全限制。

CORS 请求也使用相同的 XHR API,区别仅在于请求资源用的 URL 与当前脚本并不同源。

针对 CORS 请求的选择同意认证机制由底层处理:请求发出后,浏览器自动追加受保护的 Origin HTTP 首部,包含着发出请求的来源。相应地,远程服务器可以检查Origin首部,决定是否接受该请求,如果接受就返回 Access-Control-Allow-Origin 响应首部:

    => 请求
    GET /resource.js HTTP/1.1
    Host: thirdparty.com
    Origin: http://example.com ➊
    ...
	  <= 响应
    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: http://example.com ➋
    …
    ➊ Origin 首部由浏览器自动设置
    ➋ 选择同意首部由服务器设置


CORS 还会提前采取一系列安全措施,以确保服务器支持 CORS:

	• CORS 请求会省略 cookie 和 HTTP 认证等用户凭据;
  
	• 客户端被限制只能发送“简单的跨源请求”,包括只能使用特定的方法(GET、POST 和 HEAD),以及只能访问可以通过 XHR 发送并读取的 HTTP 首部。



要启用 cookie 和 HTTP 认证,  客户端必须在发送请求时通过 XHR 对象发送额外的属性( withCredentials ),而服务器也必须以适当的首部(Access-Control-Allow-Credentials)响应,表示它允许应用发送用户的隐私数据。   类似地,如果客户端需要写或者读自定义的 HTTP 首部,或者想要使用“不简单的方法”发送请求,那么它必须首先要获得第三方服务器的许可,即向第三方服务器发送一个预备(preflight)请求:


    => 预备请求
    OPTIONS /resource.js HTTP/1.1 ➊
    Host: thirdparty.com
    Origin: http://example.com
    Access-Control-Request-Method: POST
    Access-Control-Request-Headers: My-Custom-Header
    ...
    <= 预备响应
    HTTP/1.1 200 OK ➋
    Access-Control-Allow-Origin: http://example.com
    Access-Control-Allow-Methods: GET, POST, PUT
    Access-Control-Allow-Headers: My-Custom-Header
    ...
    (正式的 HTTP 请求) ➌

    ➊ 验证许可的预备 OPTIONS 请求
    ➋ 第三方源的成功预备响应
    ➌ 实际的 CORS 请求

W3C 官方的 CORS 规范规定了何时何地必须使用预备请求:   “简单的”请求可以跳过它,但很多条件下这个请求都是必需的,因此也会为验证许可而增加仅有一次往返的网络延迟。好在,只要完成预备请求,客户端就会将结果缓存起来,后续请求就不必重复验证了。



## 15.3  通过 XHR 下载数据

浏览器可以自动为各种原生数据类型提供编码和解码服务,因此应用在直接将这些数据传给 XHR 时就已经编码 / 解码好了

浏览器可以自动解码的数据类型如下

	• ArrayBuffer
	  固定长度的二进制数据缓冲区。
  
	• Blob
	  二进制大对象或不可变数据。
  
	• Document
	  解析后得到的 HTML 或 XML 文档。
  
	• JSON
	  表示简单数据结构的 JavaScript 对象。
  
	• Text
	  简单的文本字符串。


浏 览 器 可 以 依 靠 HTTP 的 content-type 首 部 来 推 断 适 当 的 数 据 类 型( 比 如 把 application/json 响应解析为 JSON 对象),应用也可以在发起 XHR 请求时显式重写数据类型:

	var xhr = new XMLHttpRequest();
	xhr.open('GET', '/images/photo.webp');
	xhr.responseType = 'blob'; ➊
	
	➊ 将返回数据类型设置为 Blob



## 15.4  通过 XHR 上传数据

事实上,上传不同类型数据的代码都一样,只不过最后在调用 XHR 请求对象的 send() 方法时,要传入相应的数据对象。

XHR 对 象 的 send() 方 法 可 以 接 受 DOMString 、 Document 、 FormData 、 Blob 、 File 及 ArrayBuffer 对象,并自动完成相应的编码,设置适当的 HTTP 内容类型 ( content-type ),然后再分派请求。需要发送二进制 Blob 或上传用户提交的文件?简单,取得对该对象的引用,传给 XHR。事实上,多写几行代码,还可以把大文件切成几小块:



## 15.5  监控下载和上传进度

XHR 对象提供了一个方便的 API,用于监控进度事件(表 15-1),这些事件代表请求的当前状态。

![table-15-1.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/table-15-1.jpg)

每个 XHR 请求开始时都会触发 loadstart 事件,而结束时都会触发 loadend 事件。在这两事件之间,还可能触发一或多个其他事件,表示传输状态。因此,要监控进
度,可以在 XHR 对象上注册一系列 JavaScript 事件监听器:



## 15.6  通过 XHR 实现流式数据传输

在 Streams API 规范正式推出之前,XHR 并不适合用来实现流式数据处理。


虽然 XHR 满足不了我们的要求,我们还有其他办法,而且是专门为流式数据处理设计的:

Server-Sent Events 提供方便的流 API,用于从服务器向客户端发送文本数据,而 WebSocket 则提供了高效、双向的流机制,而且同时支持二进制和文本数据。



## 15.7  实时通知与交付

### 15.7.1  通过 XHR 实现轮询

从服务器取得更新的一个最简单的办法,就是客户端在后台定时发起 XHR 请求,也就是轮询(polling)。

如果服务器有新数据,返回新数据,否则返回空响应。

轮询实现起来简单,但也经常效率很低。其中关键在于选择轮询间隔:长轮询间隔意味着延迟交付,而短轮询间隔会导致客户端与服务器间不必要的流量和协议开销。

