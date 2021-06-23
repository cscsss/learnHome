# 前言
sql 是程序开发员绕不开的一项技能，而mysql是当前最流行的数据库，熟知其内置的一些函数，可以使我们平时的开发工作更加顺畅和方便
- 时间日期函数
- 字符串函数
- 数学计算相关函数
- 条件判断函数
- 加密和压缩函数
- 聚合函数
- 格式或类型转化函数
- 加锁函数和解锁函数

关注公众号，一起交流，微信搜一搜: 潜行前行
---
## 1 时间日期函数
| 函数       | 功能描述| 
| ---                | --- | 
| CURDATE(),CURRENT_DATE()　　|返回当前日期，精确到年月日| 
|CURTIME(),CURRENT_TIM　　　|　返回当前时间，时分秒 |
|NOW(),CURRENT_TIMESTAMP(),LOCALTIME(),SYSDATE(),LOCALTIMESTAMP()　　|返回当前日期和时间：年月日时分秒 |
|UNIX_TIMESTAMP()　　　|以UNIX时间戳的形式返回当前时间 |
|UNIX_TIMESTAMP(timeStr)　| 将时间timeStr字符串以UNIX时间戳的形式返回 |
|FROM_UNIXTIME(timestamp)　|将UNIX时间戳timestamp转换为普通格式的时间 |
|UTC_DATE()	　|返回UTC日期 |
|UTC_TIME()    　| 返回UTC时间  |
|MONTH(dateTime)　　| 返回日期d中的月份值，1～12　SELECT MONTH('2021-06-05 11:11:11')->06|
|MONTHNAME(dateTime) 　 |返回日期 dateTime 当中的月份名称，如Janyary|
|DAYNAME(dateTime) 　|返回日期 dateTime 是星期几，如Monday,Tuesday|
|DAYOFWEEK(dateTime)	　|日期 dateTime 今天是星期几，1:星期日，2：星期一...7:星期六 |
|WEEKDAY(dateTime)　　|日期 dateTime 今天是星期几， 0表示星期一，1表示星期二 |
|WEEK(dateTime)，WEEKOFYEAR(dateTime)　|计算日期 dateTime 是本年的第几个星期，范围是0->53 |
|DAYOFYEAR(dateTime)　　|计算日期 dateTime 是本年的第几天|
|DAYOFMONTH(dateTime)　　|计算日期 dateTime 是本月的第几天 |
|QUARTER(dateTime)　　　|返回日期 dateTime 是第几季节，返回1->4 | 
|HOUR(t)	　　　　|返回t中的小时值 SELECT HOUR('5:13:14') ->5 |
|MINUTE(t) 　　　|返回t中的分钟值 SELECT MINUTE('5:13:14') ->13 |
|SECOND(t) 　　　　|返回t中的秒钟值 SELECT SECOND('5:13:14') ->14 |
|EXTRACT(type FROM dateTime) 　| 从日期 dateTime 中获取指定的值，type指定返回的值 SELECT EXTRACT(MINUTE FROM '2021-06-05 11:13:14') ->13  |
TIME_TO_SEC(time)	　|将时间 time 转换为秒：SELECT TIME_TO_SEC('01:12:00')　->4320
SEC_TO_TIME(second)	|　将以秒为单位的时间second转换为时分秒的格式　SELECT SEC_TO_TIME(4320)->01:12:00
TO_DAYS(dateTime)   |计算日期 dateTime 距离0000年1月1日的天数
DATEDIFF(date1,date2)	|计算日期 date1->date2 之间相隔的天数 SELECT DATEDIFF('2001-01-01','2001-02-02')->-32
ADDDATE(d,n)	| 计算其实日期d加上n天的日期
ADDDATE(d，INTERVAL expr type) | 计算起始日期d加上一个时间段后的日期; SELECT ADDDATE('2021-06-11 11:13:14', INTERVAL 5 MINUTE)->2021-06-11 11:18:14 (TYPE的取值与上面那个列出来的函数类似)
SUBDATE(d,n) | 日期d减去n天后的日期
SUBDATE(d,INTERVAL expr type) |	日期d减去一个时间段后的日期
ADDTIME(t,n)  | 时间t加上n秒的时间
SUBTIME(t,n)  | 时间t减去n秒的时间
DATE_FORMAT(d,f)  | 按表达式f的要求显示日期d
TIME_FORMAT(t,f)  | 按表达式f的要求显示时间t

- EXTRACT方法的type可取值为：MICROSECOND、SECOND、MINUTE、HOUR、DAY、WEEK、MONTH、QUARTER、YEAR、SECOND_MICROSECOND、MINUTE_MICROSECOND、MINUTE_SECOND、HOUR_MICROSECOND、HOUR_SECOND、HOUR_MINUTE、DAY_MICROSECOND、DAY_SECOND、DAY_MINUTE、DAY_HOUR、YEAR_MONTH

## 2 字符串函数
函数               | 功能描述
---                | ---
CHAR_LENGTH(s)|返回字符串s的字符数　
LENGTH(s)|返回字符串s的长度　
CONCAT(s1,s2,...)|将字符串s1,s2等多个字符串合并为一个字符串　
CONCAT_WS(x,s1,s2,...)|同CONCAT(s1,s2,...)函数，但是使用x作为连接符
INSERT(s1,x,len,s2)|用字符串s2替换s1的x位置开始，长度为len的字符串
UPPER(s),UCAASE(S)	|将字符串s的所有字母变成大写字母
LOWER(s),LCASE(s)	|将字符串s的所有字母变成小写字母
LEFT(s,n)	|返回字符串s的前n个字符
RIGHT(s,n)	|返回字符串s的后n个字符
LPAD(s1,len,s2)	|使用字符串s2来填充s1的开始处，使字符串长度达到len
RPAD(s1,len,s2)　|使用字符串s2来填充s1的结尾处，使字符串的长度达到len
LTRIM(s)	|去掉字符串s开始处的空格
RTRIM(s)	|去掉字符串s结尾处的空格
TRIM(s)	|去掉字符串s开始和结尾处的空格
TRIM(s1 FROM s)|去掉字符串s中开始处和结尾处的字符串s1　SELECT TRIM('@' FROM '@@abc@@') -> abc
REPEAT(s,n)	 |将字符串s重复n次
SPACE(n) |	返回n个空格
REPLACE(s,s1,s2) | 用字符串s2替代字符串s中的字符串s1
STRCMP(s1,s2)	|　比较字符串s1和s2
SUBSTRING(s,n,len)|获取从字符串s中的第n个位置开始长度为len的字符串
MID(s,n,len)	|　同SUBSTRING(s,n,len)
LOCATE(s1,s),POSITION(s1 IN s)	|　从字符串s中获取s1的开始位置
INSTR(s,s1）　|　从字符串s中获取s1的开始位置
REVERSE(s)　|　将字符串s的顺序反过来
FIELD(s,s1,s2...)　|　返回第一个与字符串s匹配的字符串位置；SELECT FIELD('c','a','b','c') ->  3
SUBSTRING_INDEX	　|返回从字符串str的第count个出现的分隔符delim之后的子串

## 3 数学计算相关函数】
函数               | 功能描述
---                | ---
ABS(x)             | 返回x的绝对值　　
CEIL(x),CEILING(x) | 向上取整
FLOOR(x)　|向下取整
RAND()　|返回0～1范围的随机数
RAND(x) | 
SIGN(x) | 判断数值是正负或者零
PI()　| 返回圆周率(3.141593）
TRUNCATE(x,y) | 返回数值x保留到小数点后y位的值（截断）
ROUND(x)　|　返回离x最近的整数，四舍五入
ROUND(x,y)　|　保留x小数点后y位的值，但截断时要进行四舍五入
POW(x,y).POWER(x,y)	|返回x的y次方　　
SQRT(x)	|返回x的平方根　　
EXP(x)	|返回e的x次方　SELECT EXP(3) -- 20.085536923188
MOD(x,y)|返回x除以y以后的余数
LOG(x)	|返回自然对数(以e为底的对数)　　
LOG10(x)|返回以10为底的对数　　
RADIANS(x)|将角度转换为弧度　　
DEGREES(x)|将弧度转换为角度　　
SIN(x)|求正弦值(参数是弧度)　
ASIN(x)|	求反正弦值(参数是弧度)
COS(x)	|求余弦值(参数是弧度)
ACOS(x)|	求反余弦值(参数是弧度)
TAN(x)	|求正切值(参数是弧度)
ATAN(x)|ATAN2(x)	求反正切值(参数是弧度)
COT(x)	|求余切值(参数是弧度)

## 4 条件判断函数
函数               | 功能描述
---                | ---
IF(expr1,expr2,expr3)|如果expr1不为0或者NULL,则返回expr2的值，否则返回expr3的值
IFNULL(expr1,expr2)|如果expr1不为NULL,返回expr1,否则返回expr2
NULLIF(expr1,expr2)|如果expr1=expr2则返回NULL, 否则返回expr2
CASE value WHEN [compare_value] THEN result [WHEN [compare_value] THEN result ...] [ELSE result] END　|当compare_value=value时返回result
CASE WHEN [condition] THEN result [WHEN [condition] THEN result ...] [ELSE result] END　|当condition　为TRUE时返回result

## 5 加密和压缩函数
函数               | 功能描述
---                | ---
MD5　| MD5(str)函数可以对字符串str进行散列，可以用于一些普通的不需要解密的数据加密
SHA1(str), SHA(str)　| 计算160位校验和，返回40位16进制数构成的字符串，当str为NULL时返回NULL
SHA2(str, hash_length) | 计算SHA-2系列的哈希方法(SHA-224, SHA-256, SHA-384, and SHA-512)
ENCRYPT(str[,salt])| 用unix crypt()来加密str，salt至少要有两位字符
ENCODE(str,pswd) | ENCODE函数可以使用加密密码pswd来加密字符串str
DECODE(crypt_str,pswd)　　| 对应ENCODE函数
MAX([DISTINCT] expr) | 最小值
MIN([DISTINCT] expr) | 最大值
SUM([DISTINCT] expr) | 累计总和

## 6 聚合函数
函数               | 功能描述
---                | ---
AVG([DISTINCT] expr) |  返回expr的平均值，distinct选项用于忽略重复值
COUNT([DISTINCT] expr) | 返回select中expr的非0值个数，返回值为bigint类型
GROUP_CONCAT | 连接组内的非空值，若无非空值，则返回NULL

## 7 格式或类型转化函数
函数                     | 功能描述
---                     | ---
CONV(N,from_base,to_base) |改变数字N的进制，返回值为该进制下的数字构成的字符串
INET_ATON(expr)  |  ip字符串转数字
INET_NTOA(expr)  |  数字转ip字符串
CAST(expr AS type)  | 转换数据类型
CONVERT(expr,type)|  type可以为BINARY[(N)]，CHAR[(N)]，DATE，DATETIME， DECIMAL[(M[,D])]，DECIMAL[(M[,D])]，TIME，UNSIGNED [INTEGER]等等
CONVERT(expr USING cs) | 将字符串s的字符集变成cs,cs如utf8等等

## ８ 加锁函数和解锁函数
函数                     | 功能描述
---                     | ---
GET_LOCK(name,time)   | 函数定义一个名称为nam、持续时间长度为time秒的锁。如果锁定成功，则返回1；如果尝试超时，则返回0；如果遇到错误，返回NULL。
RELEASE_LOCK(name)    |函数解除名称为name的锁。如果解锁成功，则返回1；如果尝试超时，返回0了如果解锁失败，返回NULL；
IS_FREE_LOCK(name)   |函数判断是否已使用名为name的锁定。如果使用，返回0，否则，返回1

欢迎指正文中错误
---

# 参考文章
- [MySQL 常用内置函数与所有内置函数](https://blog.csdn.net/hellokandy/article/details/82964077)
- [mysql内置函数大全](https://blog.csdn.net/qq646040754/article/details/82721801)\
- [MySQL DATE_FORMAT() 函数](https://www.w3school.com.cn/sql/func_date_format.asp)
