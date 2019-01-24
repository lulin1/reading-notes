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

![15-other-2.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/15-other-2.jpg)

所以说,轮询最适合间隔时间长,新事件到达时间有规律,且传输数据量大的场景。这个组合可以抵消多余的 HTTP 开销,并将消息交付的延迟最小化。



### 15.7.2  通过 XHR 实现长轮询

定时轮询的一个大问题就是很可能造成大量没必要的空检查。记住这一点之后,让我们来对这个轮询过程做一改进(图 15-1):在没有更新的时候不再返回空响应,而是把连接保持到有更新的时候。

![15-1.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/15-1.jpg)


利用长时间保留的 HTTP 请求(“挂起的 GET”)来让服务器向浏览器推送数据的技术,经常被称作 Comet。不过,有时候也有人用其他名字称呼这种技术,比如“保留 AJAX”、“AJAX 推送”或“HTTP 推送”。

![15-other-3.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/15-other-3.jpg)



## 15.8   XHR 使用场景及性能

XMLHttpRequest 是我们从在浏览器中做网页转向开发 Web 应用的关键。首先,它让我们在浏览器中实现了异步通信,但同样重要的是,它还把这个过程变得非常简单。分派和控制 HTTP 请求只要几行 JavaScript 代码,而其他复杂的任务浏览器都包办了:

	• 浏览器格式化 HTTP 请求并解析响应;
	
	• 浏览器强制施加相关的安全(同源)策略;
	
	• 浏览器处理内容协商(如 gzip 压缩);
	
	• 浏览器处理请求和响应的缓存;
	
	• 浏览器处理认证、重定向......

想取得需要认证的资源,且传输期间需要压缩,取得后需要缓存以备后用?浏览器会替我们完成这一切,以及更多。作为开发者,我们只要关注自己应用的逻辑即可!

简言之,XHR 不适合流式数据处理。

类似地,也没有最好的方式通过 XHR 实时交付更新。

现代浏览器支持比它更简单也更高效的 API,比如 Server-Sent Events和 WebSocket。事实上,除非你有特别的理由要使用 XHR 轮询,否则应该使用这些新技术。



# chapter 16    服务器发送事件

Server-Sent Events(SSE)让服务器可以向客户端流式发送文本消息,比如服务器上生成的实时通知或更新。为达到这个目标,SSE 设计了两个组件:  浏览器中的
EventSource 和  新的“事件流”数据格式。其中, EventSource 可以让客户端以 DOM事件的形式接收到服务器推送的通知,而新数据格式则用于交付每一次更新。

	• 通过一个长连接低延迟交付;
	
	• 高效的浏览器消息解析,不会出现无限缓冲;
	
	• 自动跟踪最后看到的消息及自动重新连接;
	
	• 消息通知在客户端以 DOM 事件形式呈现。
	
	
	
## 16.1   EventSource API

EventSource 接口通过一个简单的浏览器 API 隐藏了所有的底层细节,包括建立连接和解析消息。要使用它,只需指定 SSE 事件流资源的 URL,并在该对象上注册相应 243的 JavaScript 事件监听器即可:

	var source = new EventSource("/path/to/stream-url"); ➊
	
	source.onopen = function () { ... }; ➋
	source.onerror = function () { ... }; ➌
	
	source.addEventListener("foo", function (event) { ➍
		processFoo(event.data);
	});
	
	source.onmessage = function (event) {➎
		log_message(event.id, event.data);
		
		if (event.id== "CLOSE") {
			source.close(); ➏
		}
	}
	
	➊ 打开到流终点的 SSE 连接
	
	➋ 可选的回调,建立连接时调用
	
	➌ 可选的回调,连接失败时调用
	
	➍ 监听 "foo" 事件,调用自定义代码
	
	➎ 监听所有事件,不明确指定事件类型
	
	➏ 如果服务器发送 "CLOSE" 消息 ID,关闭 SSE 连接
	
	
SSE 实现了节省内存的 XHR 流。与原始的 XHR 流在连接关闭前会缓冲接收到的所有响应不同,SSE 连接会丢弃已经处理过的消息,而不会在内存中累积。
	
	
EventSource 接口还能自动重新连接并跟踪最近接收的消息:如果连接断开了, EventSource 会自动重新连接到服务器,还可以向服务器发送上一次接收到的消息 ID,以便服务器重传丢失的消息并恢复流。



## 16.2   Event Stream 协议
	
SSE 事件流  是以 流式 HTTP 响应 形式交付的:客户端发起常规 HTTP 请求,服务器以自定义的“text/event-stream”内容类型响应,然后交付 UTF-8 编码的事件数据。
	
在接收端, EventSource 接口通过检查换行分隔符来解析到来的数据流,
	
不支持二进制传输是有意为之的。SSE 的设计目标是简单、高效,作为一种服务器向客户端传送文本数据的机制。如果你想传输二进制数据,WebSocket 才是更合适的选择。
	
除了自动解析事件数据,SSE 还内置支持断线重连,以及恢复客户端因断线而丢失的消息。默认情况下,如果连接中断,浏览器会自动重新连接。SSE 规范建议的间隔时间是 2~3 s,这也是大多数浏览器采用的默认值。不过,服务器也可以设置一个自定义的间隔时间,只要在推送任何消息时向客户端发送一个 retry 命令即可。
	
类似地,服务器还可以给每条消息关联任意 ID 字符串。浏览器会自动记录最后一次 收 到 的 消 息 ID, 并 在 发 送 重 连 请 求 时 自 动 在 HTTP 首 部 追 加“Last-Event-ID”值。
	
客户端应用不必为重新连接和记录上一次事件 ID 编写任何代码。这些都由浏览器自动完成,然后就是服务器负责恢复了。



## 16.3 SSE使用场景及性能
	
服务器可以在消息刚刚生成就将其推送到客户端(低延迟),使用长连接的事件流协议,而且可以 gzip 压缩(低开销),浏览器负责解析消息,也没有无限缓冲。再加上超级简单的 EventSourceAPI 能自动重新连接和把消息通知作为 DOM 事件,使得 SSE 成为处理实时数据不可或缺的得力工具!



SSE 主要有两个局限。

一,只能从服务器向客户端发送数据,不能满足需要请求流的场景(比如向服务器流式上传大文件);

二,事件流协议设计为只能传输 UTF-8 数据,即使可以传输二进制流,效率也不高。
	
	实时推送就像轮询一样,可能会极大影响电池的待机时间。首先,可以考虑批量处理消息,尽量少唤醒无线电模块。其次,避免不必要的长连接,
	SSE 连接在无线电空闲时不会断开。



# chapter 17  WebSocket

WebSocket 可以实现客户端与服务器间双向、基于消息的文本或二进制数据传输。



## 17.1   WebSocket API

	var ws = new WebSocket('wss://example.com/socket'); ➊
	ws.onerror = function (error) { ... } ➋
	ws.onclose = function () { ... } ➌
	ws.onopen = function () { ➍
		ws.send("Connection established. Hello server!"); ➎
	}
	ws.onmessage = function(msg) { ➏
		if(msg.data instanceof Blob) { ➐
			processBlob(msg.data);
		} else {
			processText(msg.data);
		}
	}
	
	➊ 打开新的安全 WebSocket 连接(wss)
	
	➋ 可选的回调,在连接出错时调用
	
	➌ 可选的回调,在连接终止时调用
	
	➍ 可选的回调,在 WebSocket 连接建立时调用
	
	➎ 客户端先向服务器发送一条消息
	
	➏ 回调函数,服务器每发回一条消息就调用一次
	
	➐ 根据接收到的消息,决定调用二进制还是文本处理逻辑
	
	
	
### 17.1.1   WS 与 WSS

WebSocket 资 源 URL 采 用 了 自 定 义 模 式:ws 表 示 纯 文 本 通 信( 如 ws://example.com/socket),wss 表示使用加密信道通信(TCP+TLS)。

WebSocket 的连接协议也可以用于浏览器之外的场景,可以通过非 HTTP协商机制交换数据。
	
	使用自定义的 URL 模式虽然让非 HTTP 协商成为可能,但实践中还没有既定标准可以作为建立 WebSocket 会话的替代握手机制。


### 17.1.2  接收文本和二进制数据

WebSocket 通信只涉及消息,应用代码无需担心缓冲、解析、重建接收到的数据。比如,服务器发来了一个 1 MB 的净荷,应用的 onmessage 回调只会在客户端接收到全部数据时才会被调用。

WebSocket 协议不作格式假设,对应用的净荷也没有限制: 文本或者二进制数据都没问题。从内部看,协议只关注消息的两个信息:净荷长度和类型(前者是一个可变长度字段),据以区别 UTF-8 数据和二进制数据。

Blob 对象一般代表一个不可变的文件对象或原始数据。如果你不需要修改它或者不需要把它切分成更小的块,那这种格式是理想的(比如,可以把一个完整的 Blob 对
象传给 img 标签,参见 15.3 节“通过 XHR 下载数据”)。

而如果你还需要再处理接收到的二进制数据,那么选择 ArrayBuffer 应该更合适。



### 17.1.3  发送文本和二进制数据

建 立 了 WebSocket 连 接 后, 客 户 端 就 可 以 随 时 发 送 或 接 收 UTF-8 或 二 进 制 消 息。WebSocket 提供的是一条双向通信的信道,也就是说,在同一个 TCP 连接上,可以双向传输数据:

这里的 send() 方法是异步的:提供的数据会在客户端排队,而函数则立即返回。特别是在传输大文件的时候,千万别因为返回快,就错误地以为数据已经发送出去了!要监控在浏览器中排队的数据量,可以查询套接字的 bufferedAmount 属性:



### 17.1.4  子协议协商

WebSocket 协议对每条消息的格式事先不作任何假设:仅用一位标记消息是文本还是二进制,以便客户端和服务器有效地解码数据,而除此之外的消息内容就是未知的。

此外,与 HTTP 或 XHR 请求不同——它们是通过每次请求和响应的 HTTP 首部来沟通元数据,WebSocket 并没有等价的机制。因此,如果需要沟通关于消息的元数
据,客户端和服务器必须达成沟通这一数据的子协议。

好在,WebSocket 为此提供了一个简单便捷的子协议协商 API。客户端可以在初次连接握手时,告诉服务器自己支持哪种协议:

	var ws = new WebSocket('wss://example.com/socket',
					['appProtocol', 'appProtocol-v2']); ➊
	ws.onopen = function () {
		if (ws.protocol == 'appProtocol-v2') { ➋
			...
		} else {
			...
		}
	}
	
	➊ 在 WebSocket 握手期间发送子协议数组
	
	➋ 检查服务器选择了哪个子协议

	子协议名由应用自己定义,且在初次 HTTP 握手期间发送给服务器。除此之外,指定的子协议对核心 WebSocket API 不会有任何影响。
	
	
	
## 17.2   WebSocket 协议
	
HyBi Working Group 制定的 WebSocket 通信协议(RFC 6455)包含两个高层组件:开放性 HTTP 握手用于协商连接参数, 二进制消息分帧机制用于支持低开销的基于消息的文本和二进制数据传输。

	WebSocket 协议尝试在既有 HTTP 基础设施中实现双向 HTTP 通信,因此也使用 HTTP 的 80 和 443 端口......不过,这个设计不限于通过 HTTP 实现WebSocket 通信,未来的实现可以在某个专用端口上使用更简单的握手,而不必重新定义么一个协议。
												
												——WebSocket Protocol
												RFC 6455
												
												
WebSocket 协议是一个独立完善的协议,可以在浏览器之外实现。不过,它的主要应用目标还是实现浏览器应用的双向通信。

17.2.1  二进制分帧层

客户端和服务器 WebSocket 应用通过基于消息的 API 通信:发送端提供任意 UTF-8或二进制的净荷,接收端在整个消息可用时收到通知。为此,WebSocket 使用了自定义的二进制分帧格式(图 17-1),把每个应用消息切分成一或多个帧,发送到目的地之后再组装起来,等到接收到完整的消息后再通知接收端。

![17-1.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/17-1.jpg)


	• 帧
	最小的通信单位,包含可变长度的帧首部和净荷部分,净荷可能包含完整或部分应用消息。
	
	• 消息
	一系列帧,与应用消息对等。


所有 WebSocket 通信都是通过交换帧实现的,而帧将净荷视为不透明的应用数据块。


WebSocket 很容易发生队首阻塞的情况:消息可能会被分成一或多个帧,但不同消息的帧不能交错发送,因为没有与 HTTP 2.0 分帧机制中“流 ID”对等的字段

(参见 12.3.2 节“流、消息和帧”)。



### 17.2.2  协议扩展

WebSocket 协议扩展有哪些例子?负责制定 WebSocket 规范的 HyBi Working Group就进行了两项扩展。

	• 多路复用扩展 ( A Multiplexing Extension for WebSockets )
	  这个扩展可以将 WebSocket 的逻辑连接独立出来,实现共享底层的 TCP 连接。
	  
	• 压缩扩展 ( Compression Extensions for WebSocket )
	  给 WebSocket 协议增加了压缩功能。
	
	
如前所述,每个 WebSocket 连接都需要一个专门的 TCP 连接,这样效率很低。多路复用扩展解决了这个问题。它使用“信道 ID”扩展每个 WebSocket 帧,从而实现
多个虚拟的 WebSocket 信道共享一个 TCP 连接。

类似地,基本的 WebSocket 规范没有压缩数据的机制或建议,每个帧中的净荷就是应用提供的净荷。虽然这对优化的二进制数据结构不是问题,但除非应用实现自己的压缩和解压缩逻辑,否则很多情况下都会造成传输载荷过大的问题。实际上,压缩扩展就相当于 HTTP 的传输编码协商。

要使用扩展,客户端必须在第一次的 Upgrade 握手中通知服务器,服务器必须选择并确认要在商定连接中使用的扩展。



### 17.2.3   HTTP 升级协商

利用 HTTP 完成握手有几个好处。首先,让 WebSockets 与现有 HTTP 基础设施兼容:WebSocket 服务器可以运行在 80 和 443 端口上,这通常是对客户端唯一开放的端口。其次,让我们可以重用并扩展 HTTP 的 Upgrade 流,为其添加自定义的 WebSocket 首部,以完成协商。

	• Sec-WebSocket-Version
	  客户端发送,表示它想使用的 WebSocket 协议版本(“13”表示 RFC 6455)。如果服务器不支持这个版本,必须回应自己支持的版本。
	
	• Sec-WebSocket-Key
	  客户端发送,自动生成的一个键,作为一个对服务器的“挑战”,以验证服务器支持请求的协议版本。
	
	• Sec-WebSocket-Accept
	  服务器响应,包含 Sec-WebSocket-Key 的签名值,证明它支持请求的协议版本。
	
	• Sec-WebSocket-Protocol
	  用于协商应用子协议:客户端发送支持的协议列表,服务器必须只回应一个协议名。
	
	• Sec-WebSocket-Extensions
	  用于协商本次连接要使用的 WebSocket 扩展:客户端发送支持的扩展,服务器通过返回相同的首部确认自己支持一或多个扩展。
	
	
有了这些协商字段,就可以在客户端和服务器之间进行 HTTP Upgrade 并协商新的WebSocket 连接了：

	GET /socket HTTP/1.1
	Host: thirdparty.com
	Origin: http://example.com
	Connection: Upgrade
	Upgrade: websocket ➊
	Sec-WebSocket-Version: 13 ➋
	Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ== ➌
	Sec-WebSocket-Protocol: appProtocol, appProtocol-v2 ➍
	Sec-WebSocket-Extensions: x-webkit-deflate-message, x-custom-extension ➎
	
	➊ 请求升级到 WebSocket 协议
	
	➋ 客户端使用的 WebSocket 协议版本
	
	➌ 自动生成的键,以验证服务器对协议的支持
	
	➍ 可选的应用指定的子协议列表
	
	➎ 可选的客户端支持的协议扩展列表


与浏览器中客户端发起的任何连接一样,WebSocket 请求也必须遵守同源策略:  浏览器会自动在升级握手请求中追加 Origin 首部,远程服务器可能使用 CORS 判断接受或拒绝跨源请求 [ 参见 15.2 节“跨源资源共享(CORS)”]。要完成握手,服务器必须返回一个成功的“Switching Protocols”(切换协议)响应,并确认选择了客
户端发送的哪个选项:


	HTTP/1.1 101 Switching Protocols ➊
	Upgrade: websocket
	Connection: Upgrade
	Access-Control-Allow-Origin: http://example.com ➋
	Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo= ➌
	Sec-WebSocket-Protocol: appProtocol-v2 ➍
	Sec-WebSocket-Extensions: x-custom-extension ➎
	
	➊ 101 响应码确认升级到 WebSocket 协议
	
	➋ CORS 首部表示选择同意跨源连接
	
	➌ 签名的键值验证协议支持
	
	➍ 服务器选择的应用子协议
	
	➎ 服务器选择的 WebSocket 扩展


所有兼容 RFC 6455 的 WebSocket 服务器都使用相同的算法计算客户端挑战的答案:将 Sec-WebSocket-Key 的内容与标准定义的唯一 GUID 字符串拼接起来,计算出 SHA1 散列值,结果是一个 base-64 编码的字符串,把这个字符串发给客户端即可。
	
最低限度,成功的 WebSocket 握手必须是客户端发送协议版本和自动生成的挑战值,服务器返回 101 HTTP 响应码(Switching Protocols)和散列形式的挑战答案, 确认选择的协议版本。

最 后, 前 述 握 手 完 成 后, 如 果 握 手 成 功, 该 连 接 就 可 以 用 作 双 向 通 信 信 道 交 换 WebSocket 消息。从此以后,客户端与服务器之间不会再发生 HTTP 通信,一切由WebSocket 协议接管。



## 17.3   WebSocket 使用场景及性能

就 跟 以 前 关 于 性 能 的 讨 论 一 样, 虽 然 WebSocket 协 议 的 实 现 复 杂 性 对 应用隐藏了,但何时以及如何使用 WebSocket,毋庸置疑会对性能产生巨大影响。 WebSocket 不能取代 XHR 或 SSE,而要获得最佳性能,我们必须善于利用它的长处!



### 17.3.1  请求和响应流

WebSocket 是唯一一个能通过同一个 TCP 连接实现双向通信的机制(图 17-2),客户端和服务器随时可以交换数据。因此,WebSocket 在两个方向上都能保证文本和
二进制应用数据的低延迟交付。


• XHR 是专门为“事务型”请求 / 响应通信而优化的:客户端向服务器发送完整的、格式良好的 HTTP 请求,服务器返回完整的响应。这里不支持请求流,在Streams API 可用之前,没有可靠的跨浏览器响应流 API。
	
• SSE 可以实现服务器到客户端的高效、低延迟的文本数据流:客户端发起 SSE 连接,服务器使用事件源协议将更新流式发送给客户端。客户端在初次握手后,不能向服务器发送任何数据。

![17-2.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/17-2.jpg)



### 17.3.2  消息开销

建立了 WebSocket 连接后,客户端和服务器通过 WebSocket 协议交换数据:应用消息会被拆分为一或多个帧,每个帧会添加 2~14 字节的开销。而且,由于分帧是按
照自定义的二进制格式完成的,UTF-8 和二进制应用数据可以有效地通过相同的机制编码。

	• SSE 会给每个消息添加 5 字节,但仅限于 UTF-8 内容,参见 16.2 节“Event Stream协议”。
	
	• HTTP 1.x 请求(XHR 及其他常规请求)会携带 500~800 字节的 HTTP 元数据,加上 cookie,参见 11.5 节“度量和控制协议开销”。
	
	• HTTP 2.0 压缩 HTTP 元数据,这样可以显著减少开销,参见 12.3.8 节“首部压缩”。事实上,如果请求都不修改首部,那么开销可以低至 8 字节!


记住,这里的开销数不包括 IP、TCP 和 TLS 分帧的开销,后者一共会给每个消息增加 60~100 字节,无论使用的是什么应用协议,参见 4.7.4 节“TLS 记录大小”。
	
	

### 17.3.3  数据效率及压缩

通过常规的 HTTP 协商,每个 XHR 请求都可以协商最优的传输编码格式(如对文本数据采用 gzip 压缩)。类似地,SSE 局限于 UTF-8 文本数据,因此事件流数据可
以在整个会话期间使用 gzip 压缩。

而使用 WebSocket 时,情况要复杂一些:WebSocket 可以传输文本和二进制数据,因此压缩整个会话行不通。二进制的净荷也可能已经压缩过了!为此,WebSocket
必须实现自己的压缩机制,并针对每个消息选择应用。



### 17.3.4  自定义应用协议

自定义应用协议的灵活性也有缺点,应用可能必须实现自已的逻辑来填充某些功能空白,比如缓存、状态管理、元数据交付,等等。

使用常规 HTTP 有很多明显的优势。问自己一个简单的问题:客户端会不会因缓存接收到的数据而受益?或者中间设备如果缓存数据,是否可以优化对该数据的交付?

举个例子,WebSocket 支持二进制传输,因此应用可以流式传输任意图片而没有开销!然而,由于图片是采用自定义协议交付的,它不会被保存到浏览器或任何中间设备(如 CDN)的缓存中。结果,就可能给客户端造成不必要的下载,给来源服务器带来相当高的流量。同样的道理也适用于视频、文本等数据格式。

因此,要根据应用选择合适的传输机制!一个简单但有效的策略,就是使用WebSocket 交付无需缓存的数据,如实时更新和应用“控制”消息,后者再触发XHR 请求通过 HTTP 协议取得其他资源。



### 17.3.5  部署 WebSocket 基础设施

HTTP 是专为短时突发性传输设计的。于是,很多服务器、代理和其他中间设备的 HTTP 连接空闲超时设置都很激进。而这显然是我们在持久的 WebSocket 会话中所不愿意看到的。

为解决这个问题,要考虑三个方面:

	• 位于各自网络中的路由器、负载均衡器和代理;
	
	• 外部网络中透明、确定的代理服务器(如 ISP 和运营商的代理);
	
	• 客户网络中的路由器、防火墙和代理。


这里可以借助 TLS !通过建立一条端到端的加密信道,可以让WebSocket 通信绕过所有中间代理。


长时连接和空闲会话会占用所有中间设备及服务器的内存和套接字资源。实际上,短超时经常被视为安全、资源管理及运维的预防措施。无论部署 WebSocket、SSE,还是 HTTP 2.0,都有赖于长时会话,都会对运维提出新的挑战。
	


## 17.4  性能检查表

部署高性能的 WebSocket 服务要求细致地调优和考量,无论在客户端还是在服务器上。可以参考下列要点。

	• 使用安全 WebSocket(基于 TLS 的 WSS)实现可靠的部署。
	
	• 密切关注腻子脚本的性能(如果使用腻子脚本)。
	
	• 利用子协议协商确定应用协议。
	
	• 优化二进制净荷以最小化传输数据。
	
	• 考虑压缩 UTF-8 内容以最小化传输数据。
	
	• 设置正确的二进制类型以接收二进制净荷。
	
	• 监控客户端缓冲数据的量。
	
	• 切分应用消息以避免队首阻塞。
	
	• 合用的情况下利用其他传输机制。
	
	
最后但同样重要的是,为移动应用而优化!实时推送对于手持设备而言,反倒可能造成负面影响,因为手持设备的电池始终很宝贵。这并不代表不能在移动应用中使用 WebSocket 。相反, WebSocket 其实是一个高效的传输机制,但一定要确保注意以下问题:

	• 8.1 节“节约用电”;
	
	• 8.2 节“消除周期性及无效的数据传输”;
	
	• 8.2 节中的“内格尔及有效的服务器推送”;
	
	• 以及 8.2 节之后的“消除不必要的长连接”。
	


# chapter 18  WebRTC

Web Real-Time Communication(Web 实时通信,WebRTC)由一组标准、协议和JavaScript API 组成,用于实现浏览器之间(端到端)的音频、视频及数据共享。
WebRTC 使得实时通信变成一种标准功能,任何 Web 应用都无需借助第三方插件和专有软件,而是通过简单的 JavaScript API 即可利用。

要实现涵盖音频和视频的电话会议等完善、高品质的 RTC 应用,以及端到端的数据交换,需要浏览器具备很多新功能:音频和视频处理能力、支持新应用 API、支持
好几种新网络协议。好在,浏览器把大多数复杂性抽象成了三个主要 API:

	• MediaStream :获取音频和视频流;
	
	• RTCPeerConnection :音频和视频数据通信;
	
	• RTCDataChannel :任意应用数据通信。
	
WebRTC 的 架 构 和 协 议 也 决 定 了 其 性 能 特 点: 连 接 准 备 延 迟、 协 议开销,还有交付语义,只是其中一部分。事实上,与其他浏览器通信机制不同,WebRTC 通过 UDP 传输数据。不过,UDP 只是个起点,为了实现浏览器间实时通信,还有很多超出原始 UDP 之外的技术。



## 18.1  标准和 WebRTC 的发展

WebRTC 脱开我们熟悉的 C/S 通信模型,重新设计了浏览器的网络层,同时引入了全新的媒体机制,而这对于有效处理音频和视频是必需的。

正因为如此,WebRTC 架构由十余个标准组成,涵盖了应用和浏览器 API,以及很多必要的协议和数据格式:

换句话说, WebRTC 不仅要赋予浏览器实时通信能力,而且会把浏览器的全部功能带到通信行业。WebRTC 可远远不止是浏览器的一个新 API。



## 18.2  音频和视频引擎

不过,好消息是 WebRTC 会让浏览器具备功能完备的音频和视频引擎(图 18-1),由它们替我们完成处理信号等琐碎的工作。

![18-1.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/18-1.jpg)



所有这一切都由浏览器负责,而且更重要的是,浏览器会动态调整其处理流程,以适应不断变化的音频和视频流及网络条件。经过浏览器这一系列处理之后,Web 应用接收到了优化的媒体流,然后可以将其输出到显示器和扬声器,发送给另外一端,或者使用 HTML5 的媒体 API 进行后期处理!

通过 getUserMedia 获取音频和视频 W3C 的 Media Capture and Streams 规 范 规 定 了 一 套 新 JavaScript API, 应 用 可以 通 过 这 套 API 从 平 台 取 得 音 频 和 视 频 流, 并 对 它 们 进 行 操 作 和 处 理。 其 中,MediaStream 对象(图 18-2)是实现这个功能的主要接口。

![18-2.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/18-2.jpg)


	• MediaStream 对象包含一或多个 Track( MediaStreamTrack )。
	
	• MediaStream 中的多个 Track 相互之间是同步的。
	
	• 输入源可以是物理设备,如麦克风、摄像头、用户硬盘或另一端远程服务器中的文件。
	
	• MediaStream 的输出可以被发送到一或多个目的地:本地的视频或音频元素、后期处理的 JavaScript 代理,或者远程另一端。
	
MediaStream 对象表示一个实时的媒体流,以便应用代码从中取得数据,操作个别的Track 和控制输出。所有的音频和视频处理,比如降噪、均衡、影像增强等都由音频和视频引擎自动完成。

在浏览器中请求视频流时,可以通过 getUserMedia() API 指定一系列强制和可选的约束条件,以匹配应用的需求：

	<video autoplay></video> ➊
	<script>
		var constraints = {
			audio: true, ➋
			video: { ➌
				mandatory: { ➍
					width: { min: 320 },
					height: { min: 180 }
				},
				optional: [ ➎
					{ width: { max: 1280 }},
					{ frameRate: 30 },
					{ facingMode: "user" }
				]
			}
		}
		navigator.getUserMedia(constraints, gotStream, logError);➏
		
		function gotStream(stream) { ➐
			var video = document.querySelector('video');
			video.src = window.URL.createObjectURL(stream);
		}
		function logError(error) { ... }
	</script>
	
	➊ HTML 的 video 元素,用于输出
	
	➋ 指定请求音频 Track
	
	➌ 指定请求视频 Track
	
	➍ 对视频 Track 的强制约束条件
	
	➎ 对视频 Track 的可选约束条件
	
	➏ 从浏览器中请求音频和视频流
	
	➐ 处理 MediaStream 对象的回调函数
	
	
getUserMedia() API 负责获准访问用户的麦克风和摄像机,并获取符合指定要求的流。
	
getUserMedia() 就是从底层平台取得音频和视频流的简单 API。得到的媒体都经过了 WebRTC 音频和视频引擎的自动优化、编码、解码,然后可以输出到各种目的地。这样,我们完成了实时电话会议应用的一半,剩下的就是把数据传输给另一端了!
	

## 18.3  实时网络传输

正因为及时性的要求高于可靠性,所以 UDP 协议才更适合用于传输实时数据。TCP适合传输可靠的、有序的数据流:如果中间有丢包,TCP 就会缓冲后续所有包,等
待重传,然后再严格按照次序交给应用。相对来说,UDP 则提供下列“不服务”:

	• 不保证消息交付
	  不确认,不重传,无超时。
	
	• 不保证交付顺序
	  不设置包序号,不重排,不会发生队首阻塞。
	
	• 不跟踪连接状态
	  不必建立连接或重启状态机。
	
	• 不需要拥塞控制
	  不内置客户端或网络反馈机制。
	
	
WebRTC 就使用 UDP 作为传输层协议:低延迟和及时性才是关键。因此,只要打开音频、视频,应用就会发送 UDP 包,然后就搞定了?还没那么简单。我们还是需要一些机制,穿透层层 NAT 和防火墙,为每个流协商参数,对用户数据进行加密,实现拥塞和流量控制,等等。

UDP 是浏览器实时通信的基础,但要完全达到 WebRTC 的要求,浏览器还需要位于其上的大量协议和服务的支持(图 18-3)。

![18-3.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/18-3.jpg)

	• ICE,即 Interactive Connectivity Establishment(RFC 5245)
		 STUN,即 Session Traversal Utilities for NAT(RFC 5389)
		 TURN,即 Traversal Using Relays around NAT(RFC 5766)
	• SDP,即 Session Description Protocol(RFC 4566)
	• DTLS,即 Datagram Transport Layer Security(RFC 6347)
	• SCTP,即 Stream Control Transport Protocol(RFC 4960)
	• SRTP,即 Secure Real-Time Transport Protocol(RFC 3711)
	
	
ICE、STUN 和 TURN 是通过 UDP 建立并维护端到端连接所必需的。DTLS 用于保障传输数据的安全,毕竟加密是 WebRTC 强制的功能。最后,SCTP 和 SRTP 属于
应用层协议,用于在 UDP 之上提供不同流的多路复用、拥塞和流量控制,以及部分可靠的交付和其他服务。

RTCPeerConnection API 简介尽管用于建立和维护端到端连接涉及的协议很多,但浏览器中的应用 API 相对简单。其中, RTCPeerConnection 接口(图 18-4)就负责维护每一个端到端连接的完整生命周期:

![18-4.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/18-4.jpg)


简单地说, RTCPeerConnection 把所有连接设置、管理和状态都封装在了一个接口中。

## 18.4  建立端到端的连接

相对而言,WebRTC 两端则很可能分别位于自己的私有网络中,而且中间还隔着一或多层 NAT。结果,任何一方也不能直接访问对方。为发起会话,首先必须找到两端的候选 IP 和端口( candidate ),穿越 NAT,然后检查连接,以期找到可用路径。而即便到了这一步,也不能保证成功。

因此,要想成功地建立端到端的连接,必须首先解决另外几个问题:

	(1) 必须通知另一端我们想打开一个端到端的连接,以便它知道开始监听到来的分组;

	(2) 必须找出两端之间建立连接所需的路由线路,并在两端传播这个信息;

	(3) 必须交换有关媒介和数据流的必要信息,比如协议、编码,等等。


好消息是,WebRTC 为我们解决了其中一个问题:内置的 ICE 协议会执行必要的路由和连接检查。然而,发送通知(信号)和协商会话仍然要由应用负责。



### 18.4.1  发信号和协商会话

![18-5.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/18-5.jpg)


### 18.4.2  会话描述协议 (SDP)

SDP 是一个基于文本的简单协议(RFC 4568),用于描述会话属性。在前面的例子中,我们通过它描述的是采集的音频流。好消息是,WebRTC 应用不必直接处理SDP。JSEP( JavaScript Session Establishment Protocol , JavaScript 会话建立协议)定义了对 RTCPeerConnection 对象几个方法的简单调用,就把 SDP 所有的内部工作全都隐藏了起来


### 18.4.3  交互连接建立 (ICE)

如果其中一端或者干脆两端分别位于明显不同的私有网络中又会怎么样呢?当然还是重复前述工作流,找到并把每一端的 IP 地址嵌入SDP,但端到端的连接很明显无法连接成功!我们需要的是一条连接两端的公共路由线路。好在,WebRTC 框架可以代替我们处理大部分复杂工作:

	• 每个 RTCPeerConnection 连接对象都包含一个“ICE 代理”;
	
	• ICE 代理负责收集 IP 地址和端口( candidate );
	
	• ICE 代理负责执行两端的连接检查;
	
	• ICE 代理负责发送连接持久化信息。



### 18.4.4  增量提供 (Trickle ICE)

ICE 收集过程决非瞬间就能完成的:取得本地 IP 地址很快,但查询 STUN 服务器需要经过到外部服务器的往返,然后还有另一次端到端的 STUN 连接检查。Trickle
ICE 是对 ICE 协议的扩展,用意在于实现端与端之间的增量收集和连接检查



### 18.4.5  跟踪 ICE 收集和连接状态


### 18.4.6  完整的示例



## 18.5  交付媒体和应用数据

没有流量控制、拥塞控制、错误校验,以及一些预测带宽和延迟的机制,很容易把网络搞得一团糟,而这又会进一步降低两端及周边节点的性能。另外,UDP 以明
文传输数据,而 WebRTC 要求对所有通信加密!为解决这个问题,WebRTC 又在UDP 之上增加了几层协议:


	• 数据报传输层安全(DTLS,Datagram Transport Layer Security),用于加密传输应用数据时针对要传输的媒体数据协商密钥;
	
	• 安全实时传输(SRTP,Secure Real-Time Transport),用于传输音频和视频流;
	
	• 流控制传输协议(SCTP,Stream Control Transport Protocol)用于传输应用数据。



### 18.5.1  通过 DTLS 实现安全通信

WebRTC 规范要求所有传输的数据(音频、视频和自定义应用数据)都必须加密。当然啦,如果不是因为有赖于 TCP 的有序交付,TLS 是最合适的。既然是 UDP 连
接,所以 WebRTC 就使用 DTLS,它能提供与 TLS 相同的安全保护。

DTLS 在设计上有意与 TLS 保持一致,事实上,DTLS 本质上就是 TLS,只是为了兼容 UDP 的数据报传输而做了一些微小的修改。特别地,DTLS 解决了下列问题:

	(1) TLS 要求可靠的有序的适合分段的握手记录以协商信道;
	
	(2) 如果在混合多个分组的基础上对记录分段,就不能保证 TLS 的完整性校验;
	
	(3) 如果记录的顺序不对,也不能保证 TLS 的完整性校验。


DTLS 对 TLS 记录协议的扩展,就是为每条握手记录明确添加了分段偏移字段和序号。这样就满足了有序交付的条件,也能让大记录可以被分段成多个分组并在另一
端再进行组装。DTLS 握手记录严格按照 TLS 协议规定的顺序传输,顺序不对就报错。最后,DTLS 还要处理丢包问题:两端都使用计时器,如果预定时间内没有收
到应答,就重传握手记录。

记录序号、偏移值和重传计时器让 DTLS 在 UDP 之上实现了握手(图 18-12)。为保证过程完整,两端都要生成自已签名的证书,然后按照常规的 TLS 握手协议走。

	![18-12.jpg](https://github.com/lulin1/reading-notes/blob/master/HighPerformanceBrowserNetworking/pics/18-12.jpg)
	
	完整的 DTLS 握手需要两次往返,这一点必须牢记。换句话说,建立端到端的连接会产生额外延迟。
	
	
WebRTC 客户端自动为每一端生成自已签名的证书。因此,也就没有证书链需要验证。DTLS 保证了 加密和完整性,但把身份验证工作留给了应用;参见 4.1 节“加
密、身份验证与完整性”。最后,在满足握手要求的基础上,DTLS 为处理常规记录可能出现的分段和乱序问题,又增加了两条重要的规则:

	• DTLS 记录必须刚好放到一个网络分组中;
	
	• 必须有一个分组密码用于加密记录数据。



## 18.6   DataChannel

DataChannel 支持端到端的任意应用数据交换,就像 WebSocket 一样,但是端到端的,而且可以定义底层传输协议的交付属性。建立 RTCPeerConnection 连接之后,两端可以打开一或多个信道交换文本或二进制数据:


## 18.7   WebRTC 使用场景及性能


### 18.7.1  音频 、 视频和数据流

### 18.7.2  多方通信架构

### 18.7.3  基础设施及容量规划

### 18.7.4  数据效率及压缩


## 18.8  性能检查表

端到端的架构对应用设计提出了独特的挑战。直接、一对一的通信相对简单,而参与端越多,问题就越复杂,至少对性能来说如此。最后,我们给出有助于提高端到
端 WebRTC 应用性能的一些注意事项。

	• 发信服务
	  使用低延迟传输机制;
	  提供足够的容量;
	  建立连接后,考虑使用 DataChannel 发信。
	  防火墙和 NAT穿越
	  初始化 RTCPeerConnection 时提供 STUN 服务器;
	  尽可能使用增量 ICE,虽然发信次数多,但建立连接速度快;
	  提供 STUN 服务器,以备端到端连接失败后转发数据;
	  预计并保证 TURN 转发时容量足够用。
	  
	• 数据分发
	  对于大型多方通信,考虑使用超级节点或专用的中间设备;
	  中间设备在转发数据前,考虑先对其进行优化或压缩。
	  
	• 数据效率
	  对音频和视频流指定适当的媒体约束;
	  优化通过 DataChannel 发送的二进制净荷;
	  考虑压缩通过 DataChannel 发送的 UTF-8 数据;
	  监控 DataChannel 缓冲数据的量,同时注意适应网络条件变化。
	  
	• 交付及可靠性
	  使用乱序交付避免队首阻塞;
	  如果使用有序交付,把消息大小控制到最小,以降低队首阻塞的影响;
	  发送小消息(<1150 字节),以便将分段应用消息造成的丢包损失降至最低;
	  对部分可靠交付,设置适当的重传次数和超时间隔;“正确的”设置取决于消息大小、应用数据类型和端与端之间的延迟。
	
