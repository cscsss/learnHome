# 前言
我们常时不会见到shell脚本，但是需要阅读开发linux脚本时，这又是一项必不可少的技能。本文在于提供基础的shell编程语法和简单的实例，帮助同学快速开发，可当做shell手册使用也非常善
- shell脚本的执行和模块加载
- shell中的变量
- 位置参数和预定义变量
- 表达式的运算命令
- 表达式判断符号及相关操作符
- 跳转条件语句
- shell数组和字典
- shell自定义函数
- shell程序中常用命令

关注公众号，一起交流，微信搜一搜: 潜行前行
---

## shell脚本的执行和模块加载
- shell脚本的执行
    * `bash scriptName` 或 `sh scriptName`，当脚本文件本身没有执行权限时常使用的方法
    * `path/scriptName` 或 `./scriptName`，在当前路径下执行脚本，需要将脚本文件的权限改为可执行
    * `source scriptName` 或 `. scriptName`，通常是使用source或 "." 读入或加载指定的Shell脚本文件，然后依次执行指定的Shell脚本中的语句
- shell规范
    * 一个规范的Shell脚本在第一行会指出由哪个程序（解释器）来执行脚本中的内容，这一行内容Linux bash的编程一般为：#！/bin/bash  或 #!  /bin/sh，bash与sh的区别 , sh为bash的软连接

## shell 中的变量
- 变量的赋值方法为： 先写变量名称，紧接着是 "=" ，最后是值，中间无任何空格
### 赋值时使用带引号的作用
- 双引号：允许通过 $ 符号引用其他变量值
- 单引号：禁止引用其他变量值，$ 视为普通字符
- 反引号：命令替换，提取命令执行后的输出结果
- 示例
```c++
--- 双引号 ----
server11:~ # A=hello
hello
server11:~ # B="$A lwl"
hello lwl
--- 单引号 ---
server11:~ # B='$A lwl'
$A lwl
---- 反引号 -----
server11:~ # B=`du -sh /etc`
21M   /etc   #  du -sh /etc 的执行结果
```
## 位置参数和预定义变量
- 预定义变量
符号 | 描述
--- | ---
$# | 位置参数的数量
$* | 所有位置参数的内容
$? | 命令执行后返回的状态，0表示没有错误，非0表示有错误
$$ | 当前进程的进程号
$! | 后台运行的最后一个进程号
$0 | 当前执行的进程名
${n} | 第 {n} 个位置参数
- shell取第一个位置参数替换程序文件中的 \$1，第二个替换 \$2 , 依次类推。\$0 是一个特殊变量，它的内容是当前这个shell程序的文件名

```c++
server11:~ # cat test.sh 
#! /bin/bash
echo $1
server11:~ # bash test.sh lwl
lwl
```
## 表达式的运算命令
- Shell 中常见的算术运算命令
运算符 | 描述
--- | ---
(( expresions ))|用于整数运算的运算命令
let expresions |类似(())，常用于赋值表达式 
expr expresions|可用于 整数运算，可执行命令
$[ expresions ] |用于整数运算

- 对于(())运算命令,其返回值需要使用\$修饰，如： i=\$(($A+3))
- 示例
```c++
server11:~ # A=10
server11:~ # B=$(($A+5))
server11:~ # echo $B
15
```
## 表达式判断符号及相关操作符
- 表达式判断符号
判断符号 | 描述
--- | ---
test condition | 利用test命令进行条件判断
[ condition ]   | 利用中扣号命令进行条件判断
[[ condition ]] | 利用双中扣号命令进行条件判断
(( condition )) | 不能用于字符串比较

- 文件判断操作符
文件判断操作符 | 描述
--- | ---
-d (directory) |	文件存在且为目录则为真
-f (file) |	文件存在且为文件则为真
-e (exists)  |	文件存在则为真
-s (size)  |	文件存在且大小不为0则为真
-r (read)  |	文件存在且可读则为真
-w (write) |	文件存在且可写则为真
-x (executable) |	文件存在且可执行则为真
-L (link) |	文件存在且为链接文件则为真
f1 -nt f2 ，nt的全拼为newer than |	文件f1比文件f2新则为真
f1 -ot f2 ，ot的全拼为older than |	文件f1比文件f2旧则为真

- 字符串判断操作符（ == 和 !=  两端要有空格, (()) 不能用于字符判断）
常用字符串判断操作符 | 描述
--- | ---
-n   |	若字符串长度不为0，则为真
-z   |	若字符串长度为0，则为真
"str1"  ==  "str2"  | 若str1等于str2，则为真
"st1"  !=  "str2"   | 若str1不等于str2，则为真

- 整数比较操作符
在[]以及test中使用的比较符号 | 在(())和[[]]中使用的比较符号 | 描述
--- | --- |----
-eq	| == 或 = |  相等（equal）
-ne	| !=	  | 不相等（not equal）
-gt	| >	  | 大于（greater than）
-ge	| >=	  | 大于等于（greater equal）
-lt	| <	  | 小于（less than）
-le	| <=	  | 小于等于（less equal）

- 逻辑操作符
在[]和test中使用的逻辑操作符 |	在[[]]和(())中使用的操作符 |	描述
--- | --- |----
-a |	&&  |	and ，与，两端都为真，才为真
-o |   \|\| |	or ，或， 两端有一个为真，就为真
!  |	!   |	not ，非， 两端相反，则结果为真

- 表达式判断符号 test 、[] 、[[]] 、 (()) 的区别
表达式判断符号	|  test | 	[] | 	[[]] |  	(())
--- | --- |---- |--- |---
边界是否需要空格 |  需要      |	需要 |	需要 |	不需要
逻辑操作符      | ! 、-a、 -o |	! 、-a、 -o |	! 、&& 、 \|\| |	! 、&& 、 \|\|
整数比较操作符	 | -eq 、 -gt 、-lt、-ge 、-le | -eq 、 -gt 、-lt、-ge 、-le | -eq 、 -gt 、-lt、-ge 、-le 或  = 、>  、< 、 >= 、 <= |	= 、>  、< 、 >= 、 <=
字符串比较操作符 | = 、 == 、!= |	= 、 == 、!= |	= 、 == 、!=	| 不支持
文件操作 |	-d、-f、-e、-r、-s、-w、-x、-L、-nt、-ot	| -d、-f、-e、-r、-s、-w、-x、-L、-nt、-ot | -d、-f、-e、-r、-s、-w、-x、-L、-nt、-ot | 不支持
是否支持通配符匹配 | 不支持 | 不支持 |	支持 |不支持

- 示例
```c++
server11:~ # [ 1 == 1 -a 2 == 3 ] && echo true  # 2不等于3，因此true没打印出来
server11:~ # [ 1 == 1 -a 2 == 2 ] && echo true
true
```

## 跳转条件语句
- if 条件关键字
```C++
if  条件判断;then 
     逻辑代码
else
     逻辑代码
fi
-------------------------
server11:~ # choose="lwl"
server11:~ # if [ choose == "lwl" ];then
> echo true
> else
> echo false
> fi;
false
```

- case 条件语句
```c++
case  变量  in
        one)
            逻辑代码
;;
        two)
             逻辑代码
;;
         *) 
             逻辑代码
esac
```
- for 循环语句
```c++
for  条件;do
   逻辑代码
done
-------------
server11:~ # for (( i=0;i<2;i++ ));do
> echo $i
> done;
0
1
```
- while 循环语句
```c++
while  条件
do
   逻辑代码
done
-------------------------
server11:~ # i=0
server11:~ # while [[ $i -lt 2 ]];do
> let i++
> echo $i
> done;
1
2
```
- break、continue、exit 循环控制语句
    * Break跳出for while 循环
    * Continue跳过当次循环，执行下一次循环
    * exit直接退出程序
    
## shell数组和字典
- 数组
```c++
#方式一
server11:~ # array=(A B "C" D)
#方式二
server11:~ # array[0]='csc'
server11:~ # array[1]='lwl'
server11:~ # echo  ${#array[*]}  ## 获取数组长度 数组的长度${#array[@]} 和 ${#array[*]}
2
server11:~ # for((i=0;i<${#array[@]};i++)); do echo ${array[$i]}; done;
csc   ## 下标遍历
lwl
server11:~ # for i in ${array[@]}; do echo $i; done;
csc   ## 直接遍历value
lwl
```
- 字典
```c++
server11:~ # declare -A dic   ## 声明 dic
server11:~ # dic=([csc]="hello" [lwl]="world")
server11:~ # echo ${!dic[*]}
lwl csc   ## 打印所有key
server11:~ # echo ${dic[*]}
world hello   ## 打印所有value
server11:~ # echo ${dic["lwl"]}
world     ## 打印指定key的value
server11:~ # for key in $(echo ${!dic[*]})  ##遍历 dic
> do
>         echo "$key : ${dic[$key]}"
> done;
lwl : world
csc : hello
```

## shell自定义函数
```c++
function funcName(){
  echo "data: $1"
}
-----函数调用----
$ funcName 10
运行结果： data: 10
```
- 引用自定义函数文件时，使用`source func.sh`加载文件
- 函数传递参数（可以使用类似于Java中的args，args[1]代表Shell中的$1）

## shell程序中常用命令
- shell程序可使用在shell交互界面的一切命令

常用的命令方法 | 描述
--- | ---
git | 代码版本控制
awk、sed | 文本处理命令
mvn | java服务常用的打包命令
java | jvm 启动相关
docker | 服务docker镜像打包
kubectl | k8s相关部署命令


欢迎指正文中错误
---

#  参考文章
- [Shell 教程](https://www.runoob.com/linux/linux-shell.html)
- [Linux中编写Shell脚本](https://blog.csdn.net/qq_36119192/article/details/82964713)
- [shell编程](https://blog.csdn.net/qq_23929673/article/details/103798909)
