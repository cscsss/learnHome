
# 前言
关系型数据库的表在设计时，我们常常需要考虑哪些业务字段应该放哪张表，字段是否应该拆分，表与表之间该怎么关联。那有没有相应的规范或者原则来指导我们去设计表呢？数据库设计三范式；三范式主要是解决表之间的关联关系，和字段冗余问题

关注公众号，一起交流，微信搜一搜: 潜行前行
---

## 第一范式
-	列都是不可再分，第一范式的目标是确保每列的原子性，每列都是不可再分的最小数据单元
-	身高体重是两个属性，违反第一范式，不能划分为同一个列
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d525382958340de95f5e5c1750ecb2f~tplv-k3u1fbpfcp-watermark.image)
-	符合第一范式的设计
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd0d0a8e0dd84abba3244a2616a536db~tplv-k3u1fbpfcp-watermark.image)

## 第二范式
-	首先满足第一范式，并且表中非主键列不存在对主键不依赖或者部分依赖，确保每个列都和主键相关。一般因为是存在多个主键，或者存在复合主键，因此需要拆表
-	存在复合主键（学号，学科），而学科学分却只依赖分部主键-学科，不符合第二范式
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/118de63ad8424fe196d4f0d31be60ed9~tplv-k3u1fbpfcp-watermark.image)
-	第二范式的正确示范
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dedaecffb32b44adaa51307a80099fcd~tplv-k3u1fbpfcp-watermark.image)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d0c578f9edf4ce98e48683ed65c98b2~tplv-k3u1fbpfcp-watermark.image)
## 第三范式
-	满足第二范式，并且表中的列不存在对非主键列的传递依赖，每列都和主键列直接相关，而不是间接相关
-	在成绩表里，爱好是依赖学生的，学生又是依赖主键ID，存在传递依赖应该提取出学生的个人信息为表。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0199884b866645a285fa770b9f390f36~tplv-k3u1fbpfcp-watermark.image)
-	符合第三范式的规范
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01dccc0f73584e4a8dfeda6cf69e5b28~tplv-k3u1fbpfcp-watermark.image)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fbca0a7bdf3448fb23bef3a7ed0253f~tplv-k3u1fbpfcp-watermark.image)

欢迎指正文中错误
---
# 参考文章
-	[mysql 数据库的设计三范式](https://www.cnblogs.com/xiaozengzeng/p/10720226.html)
