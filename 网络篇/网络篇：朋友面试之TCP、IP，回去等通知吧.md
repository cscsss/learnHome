# 前言
最近和一同学聊天，他想换工作，然后去面了一家大厂。当时，他在简历上写着精通TCP/IP，本着对TCP协议稍有了解，面试官也不会深问的想法，就写了精通二字。没想到，大意了

**关注公众号，一起交流，微信搜一搜: 潜行前行**
---

# 开场
朋友约的是十点半的面试，提前了十分钟到，然后安静地坐在沙发等待，顺便回忆下之前看的资料。快到十点半时，一个高瘦，穿着格子衫的男子推开门而进，说了句“你好，我们来开始面试吧！”，朋友不失礼貌地笑着回了句“行”

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d611b13e780c4f3783dd172b1ab0bd1a~tplv-k3u1fbpfcp-watermark.image)

### 面试官：看你简历说精通TCP和IP，那我们来讨论下网络模型和TCP、IP协议，讲下你的理解先
-	朋友（怎么一上来就问TCP，不按套路出牌啊，不该问问java基础吗？不过常规题，我还行）
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bf03ce2211f4f9899347ec6625856f3~tplv-k3u1fbpfcp-watermark.image)
-	朋友：网络模型一般分七层：应用层、表示层、会话层、传输层、网络层、数据链路层、物理层。应用层的协议包括HTTP、FTP、SMTP，而TCP属于传输层，IP协议则属于网络层
-	朋友：TCP/IP网络模型层次由上到下，层层包装，每一层都对应不同的协议解析，我来画个图
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7549ee30346a48428613ae5e6bed6f62~tplv-k3u1fbpfcp-watermark.image)

### 面试官：看你画的图，TCP有自己的首部结构，这都有哪些字段，最好说说它们的作用
-	朋友（什么鬼！当我百度词典，这怎么记得住？等等，昨天晚上好像看过，有印象）
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/597cfd2106ce441180e4dc071f09fdb9~tplv-k3u1fbpfcp-watermark.image)
-	朋友：继续画个图，直观点
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45234eba59be46658951e4168dfd5d0e~tplv-k3u1fbpfcp-watermark.image)
-	朋友：TCP首部结构先是16位的源端口号和目标端口号、接着是32位的序列号和确认号。再下面就是4bit的头部长度和6个bit的保留位及6bit的标志位
-	朋友：16位的属性则有窗口大小（控制发送窗口），检验和（校验数据段是否未被修改）及紧急指针。最后是选项，其长度由头部长度决定
-	朋友：详细说下序列号，它是TCP报文段的一数字编号，为保证TCP可靠连接，每一个发送的数据段都要加上序列号。建立连接时，两端都会随机生成一个初始序列号。而确认号而是和序列号配合使用的，应答某次请求时，则返回一个确认号，它的值等于对方请求序列号加1
-	朋友：而6个标志位分别是，URG：这是条紧急信息，ACK:应答消息，PSH:缓冲区尚未填满，RST:重置连接，SYN:建立连接消息标志，FIN：连接关闭通知信息
-	朋友：窗口大小是接收端用来控制发送端的滑动窗口大小

### 面试官：那TCP和UDP有什么区别
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3c26c6d705b45d1942c13c28f108e58~tplv-k3u1fbpfcp-watermark.image)
-	朋友（松了一口气）
-	朋友：1）连接方面:TCP面向连接。UDP是无连接的，发送数据之前不需要建立连接
-	朋友：2）安全方面:TCP提供可靠的服务，保证传送的数据，无差错，不丢失，不重复，且按序到达。UDP则是尽最大努力交付，不保证可靠交付
-	朋友：3）传输效率：TCP传输效率相对较低，UDP传输效率高

### 面试官：刚才你说TCP是可靠的连接，它是怎么实现的
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2addf197af4c4c99a85f47780fbc8498~tplv-k3u1fbpfcp-watermark.image)
-	朋友：TCP的连接是基于三次握手，而断开则是四次挥手
-	朋友：为了保障数据不丢失及错误（可靠性），它有报文校验、ACK应答、超时重传(发送方)、失序数据重传（接收方）、丢弃重复数据、流量控制（滑动窗口）和拥塞控制等机制

### 面试官：具体说一说三次握手和四次挥手机制
-	朋友（又是常规题，晒晒水啦）
-	朋友：TCP是可靠的双向通道，所以需要三次握手和四次挥手，我来画个图
-	三次握手
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9c32572c5fe422b8c1efd687bb12c81~tplv-k3u1fbpfcp-watermark.image)
-	四次挥手
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7aa5e9955dc14c50ac78e3f5ab87b545~tplv-k3u1fbpfcp-watermark.image)
-	朋友：提前抢答下，关闭连接时需要四次挥手，比建立时多一次，是因为被动关闭端或许还有数据没被送出去，不能像握手时一样，第二次握手既是发起握手也是响应握手

### 面试官：如果没有三次握手会有什么问题呢
-	朋友：如果只有两次握手，client发连接请求后不会再ACK服务端的SYN
-	朋友：此时若客户端因为自身原因判断建立连接失败，可能会重复建立TCP连接，而服务端却会认为那些被client丢弃的TCP还是有效，会白白浪费资源

### 面试官：TIME_WAIT和CLOSE_WAIT的区别在哪
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/876e87c804574bf699a3b416779b2db0~tplv-k3u1fbpfcp-watermark.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7aa5e9955dc14c50ac78e3f5ab87b545~tplv-k3u1fbpfcp-watermark.image)
-	朋友：CLOSE_WAIT是被动关闭形成的；当对方close socket而发送FIN报文过来时，回应ACK之后进入CLOSE_WAIT状态。随后检查是否存在未传输数据，如果没有则发起第三次挥手，发送FIN报文给对方，进入LAST_ACK状态并等待对方ACK报文到来
-	朋友：TIME_WAIT是主动关闭连接方式形成的；处于FIN_WAIT_2状态时，收到对方FIN报文后进入TIME_WAIT状态；之后再等待两个MSL(Maximum Segment Lifetime:报文最大生存时间)

### 面试官：TIME_WAIT的作用呢，还有为啥状态时间要保持两个MSL
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76b2b438760b4c1a8908f4533592b697~tplv-k3u1fbpfcp-watermark.image)
-	朋友(这问得太深了吧，老哥。还好昨天偷偷补课了)
-	朋友：1）TIME_WAIT的作用是为了保证最后一次挥手的ACK报文能送达给对方，如果ACK丢失，对方会超时重传FIN，主动关闭端会再次响应ACK过去；如果没有TIME_WAIT状态，直接关闭，对方重传的FIN报文则被响应一个RST报文，此RST会被动关闭端被解析成错误
-	朋友：2）存在两个连接，第一个连接正常关闭，第二相同的连接紧接着建立；如果第一个连接的迷路报文到来，则会干扰第二连接，等待两个MSL则可以让上次连接的报文数据消逝在网络后

### 面试官：刚才你还有提到拥塞控制，TCP协议用什么方式去解决拥塞的
-	朋友：**第一方式是慢启动和拥塞避免**
-	朋友：1）慢启动，TCP发送端会维护一个拥塞窗口（congestionwindow）,简称为cwnd。拥塞窗口初始为1个报文段，每经过一次RTT（数据完全发送完到确认的时间），窗口大小翻倍（指数增长，只是前期慢）
-	朋友：2）拥塞避免，它思路是让拥塞窗口cwnd缓慢增大，发送方的cwnd达到阀值ssthresh(初始值由系统决定的)之后，每经过一个RTT就把拥塞窗口加一，而不是加倍（收到两个或四个确认，都是cwnd+1），cwnd呈线性增加（加法增大）
-	朋友：（画个图好解析）
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f13dfaa95cb4bd895dd54fe77715a7e~tplv-k3u1fbpfcp-watermark.image)
-	朋友：如果遇到网络拥塞，拥塞窗口阀值ssthresh减半，cwnd设置为1，重新进入慢启动阶段

### 面试官：那拥塞控制还有其他什么方式呢
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a464e76b1b1b4672b1478a503400653d~tplv-k3u1fbpfcp-watermark.image)
-	朋友：**快重传和快恢复**
-	朋友：1）快重传是当接收方收到了一个失序的报文，则立马报告给发送方，赶紧重传
-	朋友：假如接收方M1收到了，M2没有收到，之后的M3、M4、M5又发送了，此时接收方一共连续给发送方反馈了3个M1确认报文。那么快重传规定，发送方只要连续收到3个重复确认，立即重传对方发来的M2（重复确认报文的后一个报文）
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4194fcd3dd94ee58dab36cc9cbae56c~tplv-k3u1fbpfcp-watermark.image)
-	朋友：2）快恢复
-	朋友：当发送方连续收到三个重复确认，ssthresh减半；由于发送方可能认为网络现在没有拥塞，因此与慢启动不同，把cwnd值设置为ssthresh减半之后的值，然后执行拥塞避免算法，cwnd线性增大
-	朋友：（再来一图）
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a72eb095730847d6b28da75c56abd059~tplv-k3u1fbpfcp-watermark.image)

### 面试官：知道滑动窗口不，客户端和服务端控制滑动窗口的过程是怎样的
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bca9594e489742318620915187498aec~tplv-k3u1fbpfcp-watermark.image)
-	朋友：接收端将自己可以接收的缓冲区大小放入TCP首部中的“窗口大小”字段，通过ACK报文来通知发送端，滑动窗口是接收端用来控制发送端发送数据的大小，从而达到流量控制
-	朋友：其实发送方的窗口上限，是取值拥塞窗口和滑动窗口两者的最小值

### 面试官：那你知道滑动窗口和拥塞窗口有什么区别不
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b81cea9252445af999353e790e6d540~tplv-k3u1fbpfcp-watermark.image)
-	朋友：相同点都是控制丢包现象，实现机制都是让发送方发得慢一点
-	朋友：不同点在于控制的对象不同
-	朋友：1）流量控制的对象是接收方，怕发送方发的太快，使得接收方来不及处理
-	朋友：2）拥塞控制的对象是网络，怕发送方发的太快，造成网络拥塞，使得网络来不及处理

### 面试官：TCP的粘包和拆包问题，你怎么看 
-	朋友：程序需要发送的数据大小和TCP报文段能发送MSS（Maximum Segment Size，最大报文长度）是不一样的
-	朋友：大于MSS时，而需要把程序数据拆分为多个TCP报文段，称之为拆包；小于时，则会考虑合并多个程序数据为一个TCP报文段，则是粘包；其中MSS = TCP报文段长度-TCP首部长度
-	朋友：在IP协议层或者链路层、物理层，都存在拆包、粘包现象

### 面试官：那解决粘包和拆包的方法都有哪些？
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b73034f325cc40769a4e8194722aeb56~tplv-k3u1fbpfcp-watermark.image)
-	朋友：1）在数据尾部增加特殊字符进行分割
-	朋友：2）将数据定为固定大小
-	朋友：3）将数据分为两部分，一部分是头部，一部分是内容体；其中头部结构大小固定，且有一个字段声明内容体的大小

### 面试官：SYN Flood了解吗
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41d3070ec2bd4beda04db131938ea7c6~tplv-k3u1fbpfcp-watermark.image)
-	朋友：SYN Flood 伪造 SYN 报文向服务器发起连接，服务器在收到报文后用 SYN_ACK 应答，此应答发出去后，不会收到 ACK 报文，造成一个半连接
-	朋友：若攻击者发送大量这样的报文，会在被攻击主机上出现大量的半连接，耗尽其资源，使正常的用户无法访问，直到半连接超时

### 面试官：对TCP的掌握挺不错的，下面问下HTTP的知识。你知道一次HTTP请求，程序一般经历了哪几个步骤？
-	朋友：1）解析域名 -> 2）发起TCP三次握手，建立连接 -> 3）基于TCP发起HTTP请求 -> 4）服务器响应HTTP请求，并返回数据 -> 5）客户端解析返回数据
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/068eb96d41c448489ac07f62ad5f052a~tplv-k3u1fbpfcp-watermark.image)

### 面试官：HTTP有哪几种响应状态码，列举几个你熟悉的
-	朋友：大概有以下几种
	*	200：表示成功正常请求
	*	400：语义有误，一般是请求格式不对
    *	401：需求用户验证权限，一般是证书token没通过认证
    *	403：拒绝提供服务
    *	404：资源不存在
    *	500：服务器错误
    *	503：服务器临时维护，过载；可恢复

### 面试官：不错，再考考你，session和cookie有什么区别
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05b05d43d01d4be591239296eb7027e6~tplv-k3u1fbpfcp-watermark.image)
-	朋友：1）存储位置不同，cookie是保存在客户端的数据；session的数据存放在服务器上
-	朋友：2）存储容量不同，单个cookie保存的数据小，一个站点最多保存20个Cookie；对于session来说并没有上限
-	朋友：3）存储方式不同，cookie中只能保管ASCII字符串；session中能够存储任何类型的数据
-	朋友：4）隐私策略不同，cookie对客户端是可见的；session存储在服务器上，对客户端是透明对
-	朋友：5）有效期上不同，cookie可以长期有效存在；session依赖于名为JSESSIONID的cookie，过期时间默认为-1，只需关闭窗口该session就会失效
-	朋友：6）跨域支持上不同，cookie支持跨域名访问；session不支持跨域名访问

### 面试官：不错，那你了解什么是HTTP分块传送吗
-	朋友：分块传送是HTTP的一种传输机制，允许服务端发送给客户端的数据分成多个部分，该协议在HTTP/1.1提供

### 面试官：HTTP分块传送有什么好处
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26a1b18946124df7be26af6d82c8474f~tplv-k3u1fbpfcp-watermark.image)
-	朋友：HTTP分块传输编码允许服务器为动态生成的内容维持HTTP持久连接
-	朋友：分块传输编码允许服务器在最后发送消息头字段。对于那些头字段值在内容被生成之前无法知道的情形非常重要，例如消息的内容要使用散列进行签名
-	朋友：HTTP服务器有时使用压缩 （gzip或deflate）以缩短传输花费的时间。分块传输编码可以用来分隔压缩对象的多个部分。在这种情况下，块不是分别压缩的，而是整个负载进行压缩。分块编码有利于一边进行压缩一边发送数据
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5c12faeda5b4aefbaab1f514a4cdbf7~tplv-k3u1fbpfcp-watermark.image)
### 面试官：HTTP的长连接你怎么理解
-	朋友：长连接是指客户端和服务建立TCP连接后，它们之间的连接会持续存在，不会因为一次HTTP请求后关闭，后续的请求也是用这个连接
-	朋友：长连接可以省去TCP的建立和关闭操作，对于频繁请求的客户端适合使用长连接，但是注意恶意的长连接导致服务受损（建议内部服务之间使用）

### 面试官：HTTP是安全的吗？怎么做到安全的HTTP协议传输
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf0bb1fc77284126b4b54cd59e69aa5e~tplv-k3u1fbpfcp-watermark.image)
-	朋友：并非安全，HTTP传输的数据都是明文的，容易被第三方截取；要做安全传输数据，可以使用HTTP的升级版HTTPS协议

### 面试官：HTTPS和HTTP的区别，你是怎么理解的
-	朋友：1）http协议的连接是无状态的，明文传输 
-	朋友：2）HTTPS则是由SSL/TLS＋HTTP协议构建的有加密传输、身份认证的网络协议

### 面试官：SSL/TLS是什么，HTTPS的安全性是怎样实现的？

-	朋友：SSL(Secure Socket Layer 安全套接层)是基于HTTPS下的一个协议加密层，保障数据私密性。TLS(Transport Layer Security)则是升级版的SSL
-	朋友：https在http基础加了一层安全认证及加密层TLS或者SSL，它首先会通过安全层进行ca证书认证，正确获取服务端的公钥
-	朋友：接着客户端会通过公钥和服务端确认一种加密算法，后面的数据则可以使用该加密算法对数据进行加密

### 面试官：你能详细说下TLS/SSL的认证过程不...（此时面试官放在桌面的手机震动了起来，他下意识看了看手机，停顿下)
###	朋友面试暂时告一段落（下回继续）
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf182f96c94745f2be25d34c66df618a~tplv-k3u1fbpfcp-watermark.image)

欢迎指正文中错误
---
#  参考文章
-	[腾讯面试HTTP与TCP/IP20连问，你能答出多少](https://blog.csdn.net/weixin_48182198/article/details/107611341)
-	[什么是TCP/IP协议？](https://blog.csdn.net/bjweimengshu/article/details/79214572)
-	[太厉害了，终于有人能把TCP/IP协议讲的明明白白了！](https://developer.51cto.com/art/201906/597961.htm)
-	[TCP的滑动窗口与拥塞窗口](https://blog.csdn.net/ligupeng7929/article/details/79597423)