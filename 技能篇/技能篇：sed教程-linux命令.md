# 前言
sed 全名为 stream editor，是用于文本处理的流编辑器，支持正则表达式。 sed处理文本时是一次处理一行内容

关注公众号，一起交流，微信搜一搜: 潜行前行
---

# sed语法
- sed命令处理的内容是模式空间中的内容，而非直接处理文件内容。如果加上参数 i 则可直接修改文件内容
- 示例：`sed -i 's/原字符串/新字符串/' /home/test.txt`

## sed [-nefr参数] [function] [filePath]
选项与参数|描述 
---|---
-n | 使用 silent 模式。在一般 sed 的用法中，输入的数据都会被输出到屏幕上。但如果加上 -n 参数后，则不会显示，如果有跟着 p 标志，被 sed 特殊处理的那一行会被列出来 
-e | 直接在命令行界面上进行 sed 的动作编辑，执行多条子命令
-f | 将 sed 的动作写在一个文件内， -f filename 执行脚本文件的 sed 动作
-r | sed 的动作支持的是延伸型正则表达式的语法
-i | 直接修改读取的文件内容

- 选项-n，加上-n选项后被设定为安静模式，也就是不会输出默认打印信息，除非子命令中特别指定打印 p 选项，则只会把匹配修改的行进行打印
```c
---- 两行都打印出来 ----
server11:~/test # echo -e 'hello \n world' | sed 's/hello/csc/'
csc 
 world
---- 一行也没打印 -----
server11:~/test # echo -e 'hello \n world' | sed -n 's/hello/csc/'
---- 打印了匹配行 -----
server11:~/test # echo -e 'hello \n world' | sed -n 's/hello/csc/p'
csc 
```
- 选项-e，多条子命令连续进行操作
```c
echo -e 'hello world' | sed -e 's/hello/csc/' -e 's/world/lwl/'
结果：csc lwl
```
- 选项-i，直接修改读取的文件内容
```c
server11:~/test # cat file.txt 
hello world
server11:~/test # sed 's/hello/lwl/' file.txt 
lwl world
server11:~/test # cat file.txt              
hello world
---- 加上参数 i 可以直接修改文件内容----
server11:~/test # sed -i 's/hello/lwl/' file.txt 
lwl world
server11:~/test # cat file.txt              
lwl world
```
- 选项-f，执行文件脚本
```c
sed.script脚本内容：
s/hello/csc/
s/world/lwl/
------
echo "hello world" | sed -f sed.script
结果：csc lwl
```
- 选项-r，默认只能支持基本正则表达式，如果需要支持扩展正则表达式，需要加上 -r
```c
echo "hello world" | sed -r 's/(hello)|(world)/csc/g'
csc csc
```

## function表达式： [n1[,n2]] function or /{pattern}/function
- n1, n2 ：可选项，一般代表“选择进行动作的行数”，举例来说，如果**function**是需要在 10 到 20 行之间进行的，则表示为 `10,20 [function]`
- 如果需用正则表达式匹配字符串，则可用 `/{pattern}/` 匹配
```c
test.txt 内容 
111
222
333
444
----- 删除非第2第3行之间的所有行 ----------
server11:~ # sed -i '2,3!d' test.txt 
server11:~ # cat test.txt
222
333
------ 正则表达式匹配 ------------
server11:~ # echo 'clswcl.txt' | sed -nr '/.*/p'
clswcl.txt  //  /{pattern}/ = /.*/
```
## function 有以下这些选项
function | 描述 
---|---
a |新增：a 的后面可以接字串，而这些字串会在新的一行出现（目前的下一行）
i |插入：i 的后面可以接字串，而这些字串会在新的一行出现（目前的上一行）
c |取代：c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行
d |删除：因为是删除啊，所以 d 后面通常不接任何东西
p |打印：亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行
s |取代：可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正则表达式！ 例如：1,20 s/old/new/g 

- function：-a，行后插入新行
```c
sed -i '/特定字符串/a 新行字符串' fileName
```
- function：-i，行前插入新行
```c
sed -i '/特定字符串/i 新行字符串' fileName
```
- function：-c，修改指定内容行
```c
sed -i '/特定字符串/c csc lwl' fileName 
```
- function：-d，删除特定字符串
```c
sed -i '/特定字符串/d' fileName 
```

## sed s子命令: s/{pattern}/{replacement}/{flags}
- 如果{pattern}包含正则表达式，则需要加上 -r 
- 如果{pattern}存在分组，{replacement}中的"\n"代表第n个分组，"&"代表整个匹配的字符串。详情看示例
- flags的参数如下

flags | 描述 
---|---
n|可以是1-512，表示第n次出现的情况进行替换
g|全局更改
p|打印模式空间的内容
w file |写入到一个文件file中
- 示例
```c
server11:~ # echo -e 'hello 1112 world' | sed -r 's/([a-z]+)( [0-9]+ )([a-z]+)/&/'  
hello 1112 world
server11:~ # echo -e 'hello 1112 world' | sed -r 's/([a-z]+)( [0-9]+ )([a-z]+)/\3\2\1/' 
world 1112 hello
```

---
#  参考文章
- [sed -i命令详解及入门攻略](https://blog.csdn.net/qq_33468857/article/details/84324609)