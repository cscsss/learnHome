# 前言
字符串的常用操作，记一下。如果有字符串的操作小技巧，好用的工具类，欢迎补全。大家随意点赞，随意看看
- String实现原理
- String常用方法
- 字符串的缓存池和String.intern
- 正则匹配和字符串
- StringBuffer和StringBuilder
- spring工具类StringUtils

**关注公众号，一起交流，微信搜一搜: 潜行前行**
---
## String实现原理
```java
public final class String implements Serializable,Comparable<String>, CharSequence {
    private final char value[];
    private int hash; // Default to 0
```
String有一个value的字符数组，用来保存字符串的。默认编码是utf-8

## String常用方法
```java
//从toffset开始，是否以prefix开头
public boolean startsWith(String prefix, int toffset)
//str出现的最后一次的下标
public int lastIndexOf(String str)
//返回下标在beginIndex和endIndex之间的字符串
public String substring(int beginIndex, int endIndex)
//连接str，并返回一个新的字符串
public String concat(String str)
//匹配regex正则表达式
public boolean matches(String regex) 
//替换字符
public String replace(char oldChar, char newChar)
//替换字符串
public String replace(CharSequence target, CharSequence replacement)
//获取charsetName编码的字节数组
public byte[] getBytes(String charsetName)
//获取
public int codePointAt(int index)
//分割
public String[] split(String regex, int limit)
//转成小写
public String toLowerCase()
//转成大写
public String toUpperCase() 
//截取前后空格
public String trim()
//elements使用delimiter作为连接符，合并成一个新字符串
public static String join(CharSequence delimiter, CharSequence... elements)   
public static String join(CharSequence delimiter, Iterable<? extends CharSequence> elements)
//args参数填充进format的占位符，并返回新字符串
public static String format(String format, Object... args)
//转为缓存池的字符串
public native String intern()
```

使用示例
```java
String data = "String字符串操作－必知必备";
System.out.println(data.startsWith("St",0));
System.out.println(data.startsWith("St",5));
----------
true
false
```
### 字符串的格式化输出
`public static String format(String format, Object... args)`
```java
System.out.println(String.format("%s-%s","wst","csc"));
--------
wst-csc
```

## 字符串的缓存池和String.intern

string.intern();intern先判断常量池是否存相同字符串,存在则返回该引用；否则在常量池中记录堆中首次出现该字符串的引用，并返回该引用。

如果是先执行 String s = "st" ;相当于执行了intern();先在常量池创建"st",并且将引用A存入常量池，返回给s。此时String("hello").intern()会返回常量池的引用A返回

使用示例
```java
public static void main(String[] args) throws Exception {
        String one = "st";
        String two = new String("st");
        String three = two.intern();
        System.out.println(two == one);
        System.out.println(three == one);
    }
---------------
false
true
```

## 字符串和正则匹配、Matcher
字符 | 说明
---|---
\ | 将下一字符标记为特殊字符、文本、反向引用或八进制转义符。例如，"n"匹配字符"n"。"\n"匹配换行符，"\\("匹配"("
^ | 匹配输入字符串开始的位置。如果设置了 RegExp 对象的 Multiline 属性，^ 还会与"\n"或"\r"之后的位置匹配
$ | 匹配输入字符串结尾的位置。如果设置了 RegExp 对象的 Multiline 属性，$ 还会与"\n"或"\r"之前的位置匹配
\* |  零次或多次匹配前面的字符或子表达式。例如，zo* 匹配"z"和"zoo"。* 等效于 {0,}
\+ | 一次或多次匹配前面的字符或子表达式。例如，"zo+"与"zo"和"zoo"匹配，但与"z"不匹配。+ 等效于 {1,}
? | 零次或一次匹配前面的字符或子表达式。例如，"do(es)?"匹配"do"或"does"中的"do"。? 等效于 {0,1}
{n}| n 是非负整数。正好匹配 n 次。例如，"o{2}"与"Bob"中的"o"不匹配，但与"food"中的两个"o"匹配
{n,} | n 是非负整数。至少匹配 n 次。例如，"o{2,}"不匹配"Bob"中的"o"，而匹配"foooood"中的所有 o。"o{1,}"等效于"o+"。"o{0,}"等效于"o*"
{n,m} | m 和 n 是非负整数，其中 n <= m。匹配至少 n 次，至多 m 次。例如，"o{1,3}"匹配"fooooood"中的头三个 o。'o{0,1}' 等效于 'o?'。注意：您不能将空格插入逗号和数字之间。
? | 当此字符紧随任何其他限定符（*、+、?、{n}、{n,}、{n,m}）之后时，匹配模式是"非贪心的"。"非贪心的"模式匹配搜索到的、尽可能短的字符串，而默认的"贪心的"模式匹配搜索到的、尽可能长的字符串。例如，在字符串"oooo"中，"o+?"只匹配单个"o"，而"o+"匹配所有"o"。
.  | 匹配除"\r\n"之外的任何单个字符。若要匹配包括"\r\n"在内的任意字符，请使用诸如"[\s\S]"之类的模式。
\n | 换行符匹配。等效于 \x0a 和 \cJ
\r |匹配一个回车符。等效于 \x0d 和 \cM
\s | 匹配任何空白字符，包括空格、制表符、换页符等
\d | 数字字符匹配。等效于 [0-9]
[a-z] | 字符范围。匹配指定范围内的任何字符。例如，"[a-z]"匹配"a"到"z"范围内的任何小写字母

使用示例

```java
public static void main(String[] args) throws Exception {
        String content = "I am st ! ";
        String pattern = ".*st.*";
        boolean isMatch = Pattern.matches(pattern, content);
        System.out.println("字符串中是否包含了 'st' 子字符串? " + isMatch);
    }
---------------
字符串中是否包含了 'st' 子字符串? true
```

## StringBuffer和StringBuilder

StringBuffer、StringBuilder和String一样，也用来代表字符串。String类是不可变类，每次都要重新申请内存，不过StringBuffer和StringBuilder是事先申请缓存内存，在拼接变长且较长的字符串时，性能比String好

StringBuffer是线程安全的，每一次加减操作都是加上锁的；StringBuilder则是StringBuffer无锁版本。如果对线程安全不了解，建议使用StringBuffer，不需考虑加锁会带来性能的损耗，因为jvm会自动视情况优化为无锁操作

## spring工具类StringUtils
```java
public static boolean isEmpty(Object str)
//是否含有空格符
public static boolean containsWhitespace(CharSequence str) 
//截取两边空格符
public static String trimWhitespace(String str)
//替换所有空格符
public static String trimAllWhitespace(String str)
//获取文件名
public static String getFilename(String path)
```

欢迎指正文中错误
---
##  参考文章
-	[关于String.intern()](https://blog.csdn.net/u014333083/article/details/80354900)
-	[String的Intern方法](https://www.cnblogs.com/xuyunqi/p/7928435.html)
-	[Java 正则表达式](https://www.runoob.com/java/java-regular-expressions.html)
