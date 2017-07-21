# 关于Http协议
>主讲人：刘龙龙

## 简介
**HTTP**`（HyperText Transfer Protocol）`
协议是基于TCP的应用层协议，它不关心数据传输的细节，主要是用来规定客户端和服务端的数据传输格式，最初是用来向客户端传输HTML页面的内容。默认端口是80，HTTPS的端口号为443。

## 发展过程

[![image](http://img2.ph.126.net/JE5cjxoqg8AFu2EouDh7ZA==/6631874807237304287.png)](http://note.youdao.com/share/?id=67ef7466dcd0ced369abf79e2aa317e3&type=note#/)

## 在TCP/IP协议栈中的位置
HTTP协议通常承载于TCP协议之上，有时也承载于TLS或SSL协议层之上，这时就成了HTTPS。如下图：

![image](http://www.blogjava.net/images/blogjava_net/amigoxie/40799/o_http%e5%8d%8f%e8%ae%ae%e5%ad%a6%e4%b9%a0-11.jpg)

## HTTP 请求组成：
- 请求行:请求方法 URI 协议/版本
- 请求头
- 请求正文

## HTTP响应格式:
- 状态行
- 消息报头
- 响应正文

## HTTP请求方法
请求方法是客户端用来告知服务器其动作意图的方法。就像下达命令一样。在HTTP1.1版本中支持GET、POST等近10种方法。需要注意的是方法名区分大小写，需要用大写字母

- 1、GET：获取资源

  GET方法用来请求访问已被URI识别的资源。也就是指定了服务器处理请求之后响应的内容。

- 2、POST：传输实体主体

   POST方法用来传输实体主体。POST与GET的区别之一就是目的不同，二者之间的区别会在文章的最后详细说明。虽然GET方法也可以传输，但是一般不用，因为GET的目的是获取，POST的目的是传输。

- 3、PUT：传输文件

  PUT方法用来传输文件。类似FTP协议，文件内容包含在请求报文的实体中，然后请求保存到URL指定的服务器位置。

- 4、HEAD：获得报文首部

  HEAD方法类似GET方法，但是不同的是HEAD方法不要求返回数据。用于确认URI的有效性及资源更新时间等。

- 5、DELETE：删除文件

  DELETE方法用来删除文件，是与PUT相反的方法。DELETE是要求返回URL指定的资源。

- 6、OPTIONS：询问支持的方法

  因为并不是所有的服务器都支持规定的方法，为了安全有些服务器可能会禁止掉一些方法例如DELETE、PUT等。那么OPTIONS就是用来询问服务器支持的方法。

- 7、TRACE：追踪路径

  TRACE方法是让Web服务器将之前的请求通信环回给客户端的方法。这个方法并不常用。

- 8、CONNECT：要求用隧道协议连接代理

  CONNECT方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行TCP通信。主要使用SSL/TLS协议对通信内容加密后传输。
  
## Windows下使用[telnet](http://blog.chinaunix.net/uid-26167002-id-3054040.html)进行http测试
1. Windows7中开启telnet服务
   
2. 打开Telnet服务
 [传送门](http://blog.csdn.net/hsd2012/article/details/51075811)


## HTTP的请求响应模型
HTTP协议永远都是客户端发起请求，服务器回送响应。如下图：

![image](http://www.blogjava.net/images/blogjava_net/amigoxie/40799/o_http%E5%8D%8F%E8%AE%AE%E5%AD%A6%E4%B9%A0-12.jpg?_=6558756)

这样就限制了使用HTTP协议，无法实现在客户端没有发起请求的时候，服务器将消息推送给客户端。

HTTP协议是一个无状态的协议，同一个客户端的这次请求和上次请求是没有对应关系。

## Cookie和Session
Cookie和Session都为了用来保存状态信息，都是保存客户端状态的机制，它们都是为了解决HTTP无状态的问题而所做的努力。

Session可以用Cookie来实现，也可以用URL回写的机制来实现。用Cookie来实现的Session可以认为是对Cookie更高级的应用。

### 两者比较

Cookie和Session有以下明显的不同点：

- Cookie将状态保存在客户端，Session将状态保存在服务器端；

- Cookies是服务器在本地机器上存储的小段文本并随每一个请求发送至同一个服务器。Cookie最早在RFC2109中实现，后续RFC2965做了增强。网络服务器用HTTP头向客户端发送cookies，在客户终端，浏览器解析这些cookies并将它们保存为一个本地文件，它会自动将同一服务器的任何请求缚上这些cookies。Session并没有在HTTP的协议中定义；

- Session是针对每一个用户的，变量的值保存在服务器上，用一个sessionID来区分是哪个用户session变量,这个值是通过用户的浏览器在访问的时候返回给服务器，当客户禁用cookie时，这个值也可能设置为由get来返回给服务器；

- 就安全性来说：当你访问一个使用session 的站点，同时在自己机子上建立一个cookie，建议在服务器端的SESSION机制更安全些.因为它不会任意读取客户存储的信息。

### Session机制

Session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。

当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了一个session标识 - 称为 session id，如果已包含一个session id则说明以前已经为此客户端创建过session，服务器就按照session id把这个 session检索出来使用（如果检索不到，可能会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个 session id将被在本次响应中返回给客户端保存。

#### Session的实现方式

1. 使用Cookie来实现

    服务器给每个Session分配一个唯一的JSESSIONID，并通过Cookie发送给客户端。当客户端发起新的请求的时候，将在Cookie头中携带这个JSESSIONID。这样服务器能够找到这个客户端对应的Session。

流程如下图所示：
![image](http://www.blogjava.net/images/blogjava_net/amigoxie/40799/o_http%E5%8D%8F%E8%AE%AE%E5%AD%A6%E4%B9%A0%E5%92%8C%E6%80%BB%E7%BB%93%E7%B3%BB%E5%88%97-3-1.jpg?_=6558756)

2. 使用URL回显来实现

   URL回写是指服务器在发送给浏览器页面的所有链接中都携带JSESSIONID的参数，这样客户端点击任何一个链接都会把JSESSIONID带会服务器。如果直接在浏览器输入服务端资源的url来请求该资源，那么Session是匹配不到的。Tomcat对Session的实现，是一开始同时使用Cookie和URL回写机制，如果发现客户端支持Cookie，就继续使用Cookie，停止使用URL回写。如果发现Cookie被禁用，就一直使用URL回写。jsp开发处理到Session的时候，对页面中的链接记得使用response.encodeURL() 。


## Web缓存的实现原理

### 什么是Web缓存

WEB缓存(cache)位于Web服务器和客户端之间。

缓存会根据请求保存输出内容的副本，例如html页面，图片，文件，当下一个请求来到的时候：如果是相同的URL，缓存直接使用副本响应访问请求，而不是向源服务器再次发送请求。

HTTP协议定义了相关的消息头来使WEB缓存尽可能好的工作。

### 缓存的优点

- 减少相应延迟：因为请求从缓存服务器（离客户端更近）而不是源服务器被相应，这个过程耗时更少，让web服务器看上去相应更快。

- 减少网络带宽消耗：当副本被重用时会减低客户端的带宽消耗；客户可以节省带宽费用，控制带宽的需求的增长并更易于管理。

### 与缓存相关的HTTP扩展消息头

- Expires：指示响应内容过期的时间，格林威治时间GMT

- Cache-Control：更细致的控制缓存的内容

- Last-Modified：响应中资源最后一次修改的时间

- ETag：响应中资源的校验值，在服务器上某个时段是唯一标识的。

- Date：服务器的时间

- If-Modified-Since：客户端存取的该资源最后一次修改的时间，同Last-Modified。

- If-None-Match：客户端存取的该资源的检验值，同ETag。

### 客户端缓存生效的常见流程

服务器收到请求时，会在200OK中回送该资源的Last-Modified和ETag头，客户端将该资源保存在cache中，并记录这两个属性。当客户端需要发送相同的请求时，会在请求中携带If-Modified-Since和If-None-Match两个头。两个头的值分别是响应中Last-Modified和ETag头的值。服务器通过这两个头判断本地资源未发生变化，客户端不需要重新下载，返回304响应。常见流程如下图所示：
![image](http://www.blogjava.net/images/blogjava_net/amigoxie/40799/o_http%E5%8D%8F%E8%AE%AE%E5%AD%A6%E4%B9%A0%E5%92%8C%E6%80%BB%E7%BB%93%E7%B3%BB%E5%88%97-3-3.jpg?_=6558756)


###  Web缓存机制

HTTP/1.1中缓存的目的是为了在很多情况下减少发送请求，同时在许多情况下可以不需要发送完整响应。前者减少了网络回路的数量；HTTP利用一个“过期（expiration）”机制来为此目的。后者减少了网络应用的带宽；HTTP用“验证（validation）”机制来为此目的。

HTTP定义了3种缓存机制：

1.  `Freshness`：允许一个回应消息可以在源服务器不被重新检查，并且可以由服务器和客户端来控制。例如，Expires回应头给了一个文档不可用的时间。Cache-Control中的max-age标识指明了缓存的最长时间；

2. `Validation`：用来检查以一个缓存的回应是否仍然可用。例如，如果一个回应有一个Last-Modified回应头，缓存能够使用If-Modified-Since来判断是否已改变，以便判断根据情况发送请求；

3. `Invalidation`： 在另一个请求通过缓存的时候，常常有一个副作用。例如，如果一个URL关联到一个缓存回应，但是其后跟着POST、PUT和DELETE的请求的话，缓存就会过期。

# 关于HTTPS

### 简介
HTTPS（全称：Hypertext Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL。

https所用的端口号是443。

### https的实现原理

有两种基本的加解密算法类型：

1）对称加密：密钥只有一个，加密解密为同一个密码，且加解密速度快，典型的对称加密算法有DES、AES等；

2）非对称加密：密钥成对出现（且根据公钥无法推知私钥，根据私钥也无法推知公钥），加密解密使用不同密钥（公钥加密需要私钥解密，私钥加密需要公钥解密），相对对称加密速度较慢，典型的非对称加密算法有RSA、DSA等。

### https的通信过程：

   ![image](http://www.blogjava.net/images/blogjava_net/amigoxie/40799/o_http%E5%8D%8F%E8%AE%AE%E5%AD%A6%E4%B9%A0%E5%92%8C%E6%80%BB%E7%BB%93%E7%B3%BB%E5%88%97-3-5.jpg?_=6558756)

### https通信的优点：

1）客户端产生的密钥只有客户端和服务器端能得到；

2）加密的数据只有客户端和服务器端才能得到明文；

3）客户端到服务端的通信是安全的。


# 关于Android网络通信的方式
Android SDK中的一些与网络有关的API包名


  包    |  描述
---|---
java.net   | 提供与联网有关的类，包括流和数据包（datagram）sockets、Internet 协议和常见 HTTP 处理。该包是一个多功能网络资源。有经验的 Java 开发人员可以立即使用这个熟悉的包创建应用程序。
java.io |虽然没有提供显式的联网功能，但是仍然非常重要。该包中的类由其他 Java 包中提供的 socket 和连接使用。它们还用于与本地文件（在与网络进行交互时会经常出现）的交互
java.nio | 包含表示特定数据类型的缓冲区的类。适合用于两个基于 Java 语言的端点之间的通信。
org.apache.*  |  表示许多为 HTTP 通信提供精确控制和功能的包。可以将 Apache 视为流行的开源 Web 服务器。
android.net | 除核心 java.net.* 类以外，包含额外的网络访问 socket。该包包括 URI 类，后者频繁用于 Android 应用程序开发，而不仅仅是传统的联网方面。        1
android.net.http  | 包含处理 SSL 证书的类。
android.net.wifi  | 包含在 Android 平台上管理有关 WiFi（802.11 无线 Ethernet）所有方面的类。
android.telephony.gsm |  包含用于管理和发送 SMS（文本）消息的类。一段时间后，可能会引入额外的包来来为非 GSM 网络提供类似的功能，比如 CDMA 或 android.telephony.cdma 等网络。    
android.net.sip | 包含Andriod平台上管理有关SIP协议如建立和回应Voip的类  
Android.net.nfc |  包含所有用来管理近场通信相关的功能类    

## Android中几种网络编程的方式：
1.   针对TCP/IP的Socket、ServerSocket
2.   针对UDP的DatagramSocket、DatagramPackage。这里需要注意的是，考虑到Android设备通常是手持终端,IP都是随着上网进行分配的。不是固定的。因此开发也是有一点与普通互联网应用有所差异的。
3.   针对直接URL的HttpURLConnection
4.   Google集成了ApacheHTTP客户端，可使用HTTP进行网络编程。针对HTTP，Google集成了Appache Http core和httpclient 4版本，因此特别注意Android不支持httpclient 3.x系列，而且目前并不支持Multipart（MIME）,需要自行添加httpmime.jar 
5.   使用Web Service。Android可以通过开源包如jackson去支持Xmlrpc和Jsonrpc,另外也可以用Ksoap2去实现Webservice 
6.   直接使用WebView视图组件显示网页。基于WebView 进行开发，Google已经提供了一个基于chrome-lite的Web浏览器，直接就可以进行上网浏览网页。


## 相关学习书籍

[![link](https://img1.doubanio.com/lpic/s11329547.jpg)](https://www.amazon.cn/HTTP%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97-%E5%90%89%E5%B0%94%E5%88%A9/dp/B008XFDQ14/ref=sr_1_1?ie=UTF8&qid=1496911630&sr=8-1&keywords=HTTP)
