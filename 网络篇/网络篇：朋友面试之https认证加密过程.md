# 前言
上次朋友关于TCP/IP面试的后续，主要是https的相关面试要点，请看下文

关注公众号，一起交流，微信搜一搜: 潜行前行
---

### 面试官：HTTPS它的认证加密过程是怎样，它怎么保证内容不会被篡改
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/873d7168bda54d028780be999cad1556~tplv-k3u1fbpfcp-watermark.image)
-	朋友：1，https是基于tcp协议的，客户端先会和服务端发起链接建立
-	朋友：2，接着服务端会把它的证书返回给客户端，证书里面包括公钥S.pub、颁发机构和有效期等信息
-	朋友：3，拿到的证书可以通过浏览器内置的根证书（内含C.pub）验证其合法性
-	朋友：4，客户端生成随机的对称加密秘钥Z，通过服务端的公钥S.pub加密发给服务端
-	朋友：5，客户端和服务端通过对称秘钥Z加密数据来进行http通信

### 面试官：那根证书怎么保证签发的证书是有安全有效的
-	朋友：1-服务器会预先生成非对称加密密钥，私钥S.pri自己保留；而公钥S.pub则发给CA机构进行签名认证
-	朋友：2-CA也会预先生成一非对称加密密钥，其私钥C.pri用来对服务器的公钥S.pub进行签名生成CA证书
-	朋友：3-CA机构会把签名生成的CA证书返回给服务器，也就是刚才服务端给客户端那个证书
-	朋友：4-因为CA(证书颁发机构)比较权威，所以很多浏览器会内置包含它公钥(C.pub)的证书，称之为根证书。然后可以使用根证书来验证其颁发证书的合法性了
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d7903a82d6242f183079ffb07dd7070~tplv-k3u1fbpfcp-watermark.image)
### 面试官：如果发生无限套娃的情况，根证书都被篡改怎么办？
-	朋友：无解，这需要CA根证书是准确无误，不手动强制修改本地根证书就没事，因为不经过原有根证书认证的证书是无法自动被加入根证书

### 面试官：你讲得有些快，画下图看看
-	朋友：https加密过程
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/816bf5ed7c1d4fd091e87209b0aceae1~tplv-k3u1fbpfcp-watermark.image)
-	朋友：服务端证书通过CA机构签名认证的过程如下
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12479beef2054354b8a1bbe48071bf69~tplv-k3u1fbpfcp-watermark.image)

### 面试官：前面你说到CA机构会用密钥对服务器的公钥进行签名，签名和加密，你是怎么理解
-	朋友：使用非对称加密算法时，签名是用来表示使用私钥的加密过程
-	朋友：如果用公钥对数据加密，是加密
-	朋友：反之用私钥对数据加密，则称之为签名
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/895a88a71d4545dd947a3b21699f20e9~tplv-k3u1fbpfcp-watermark.image)

### 面试官：那CA证书是什么？
-	朋友：CA证书是为了确保服务端的公钥是准确无误，没被修改过的
-	朋友：证书通常包含这些内容(1) 服务端的公钥；(2) 证书发行者(CA)对证书的数字签名；(3) 证书所用的签名算法；(4) 证书发布机构、有效期、所有者的信息等其他信息

### 面试官：你讲到HTTPS用到了加密算法，加密算法都有哪些类型，说说看
-	朋友：加密算法分三大类：单向加密，对称加密算法和非对称加密算法

### 面试官：对称加密和非对称加密有什么区别吗
-	朋友：使用对称加密时，加密和解密用的都是同一个密钥；而非对称加密，则是两个密钥，公钥加密则需要私钥解密，私钥加密则需要公钥解密。不能私钥加密，私钥解密

### 面试官：MD5、SHA、Base64和RSA属于什么类型的算法，对称还是非对称？
-	朋友：MD5、SHA，称为摘要算法，可以归类为单向加密算法，其计算出的摘要信息，是不可逆向恢复成原来的数据
-	朋友：RSA属于非对称加密算法
-	朋友：而Base64并不算是加密算法，它更多时候是被称为一种数据编码方式
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/baaad6f183fa4ca59eb7e7d3099e2a4b~tplv-k3u1fbpfcp-watermark.image)

### 面试官：用过哪些HTTP客户端工具类？
-	朋友：apache的CloseableHttpClient、jdk9的httpClient和spring clould体系里的ribbon、feign

### 面试官：有没有遇到过使用https证书问题，如果有，是什么问题呢？
-	朋友：当然，有一次使用apache-httpClient加载自定义证书时（没经过CA认证），测试服抛出证书无法信任，然而本地运行却没问题
-	朋友：原因是证书在本地生成的，那时已经默认加入到根证书下了，而测试服jre的根证书目录（/lib/security/cacerts）是没有这个证书，放在项目resource下的证书也无法没生效

### 面试官：哦，那你是怎样解决的
-	朋友：三种解决方案。1-重写TrustManager，无条件信任证书；２-把证书加到jre的根证书目录；３-通过CA认证


### 面试官：网络数据抓包了解不
-	朋友：在linux系统可以使用tcpdump命令对tcp请求数据抓包，抓到的数据输出到一个文件；然后可以在window使用wireshark软件加载tcp数据文件，它可以提供界面分析


### 面试官：讲得还不错，那下面我们换个话题，聊聊mysql事务....
-	朋友：可以啊，我也稍微了解过一些事务的知识...
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8913319861a5466b9f8c680049a81319~tplv-k3u1fbpfcp-watermark.image)

欢迎指正文中错误（故事纯属虚构，如有雷同纯属巧合）
---
#  参考文章
-	[图解HTTPS建立过程](https://www.cnblogs.com/softidea/p/6958394.html)
-	[摘要、签名与数字证书都是什么？](https://www.jianshu.com/p/b2774634041a)
-	[数字签名是什么？](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)





