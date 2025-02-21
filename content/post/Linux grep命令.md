---
title: "Linux grep命令"
date: 2025-02-17T22:33:45+08:00
draft: false
categories: ["Linux"]
tags: ["Linux"]
keywords: ["Linux","grep"]
---



## Linux grep命令

> 全拼:Global search REgular expression and Print out the line.
> 
> 作用:文本搜索工具，根据用户指定的“模式（过滤条件)”对目标文本逐行进行匹配检查，打印匹配到的行.
> 
> 模式:由正则表达式的元字符及文本字符所编写出的过滤条件﹔
>

**grep的语法格式：**

grep -option（参数） ‘word’（关键词） file（文本文件）；

**grep常用参数：**
–color=auto 或者 –color：表示对匹配到的文本着色显示

-i：在搜索的时候忽略大小写

-n：显示结果所在行号

-c：统计匹配到的行数，注意，是匹配到的总行数，不是匹配到的次数

-o：只显示符合条件的字符串，但是不整行显示，每个符合条件的字符串单独显示一行

-v：输出不带关键字的行（反向查询，反向匹配）

-w：匹配整个单词，如果是字符串中包含这个单词，则不作匹配

-Ax：在输出的时候包含结果所在行之后的指定行数，这里指之后的x行，A：after

-Bx：在输出的时候包含结果所在行之前的指定行数，这里指之前的x行，B：before

-Cx：在输出的时候包含结果所在行之前和之后的指定行数，这里指之前和之后的x行，C：context

-e：实现多个选项的匹配，逻辑or关系

-q：静默模式，不输出任何信息，当我们只关心有没有匹配到，却不关心匹配到什么内容时，我们可以使用此命令，然后，使用”echo $?”查看是否匹配到，0表示匹配到，1表示没有匹配到。

-P：表示使用兼容perl的正则引擎。

-E：使用扩展正则表达式，而不是基本正则表达式，在使用”-E”选项时，相当于使用egrep。


##  示例

``` shell
# 从test.txt文本文件中搜索包含”L”字符的行
grep "L" test.txt 

# 输出以 I 开头的行(不区分大小写)
grep "^i" test.txt -i -n -o

# 可以使用”--color”选项，高亮显示行中的关键字
grep "L" test.txt -i -n --color

# -B”选项，显示符合条件的行之前的行，"B"有before之意
grep "L" test.txt -i -n --color -B2

# "-A"有After之意，"-A"代表显示符合条件的行的同时，还要显示之后的行，"-A3"表示同时显示符合条件的行之后的3行。
grep "L" test.txt -i -n --color -A2

# "-C"选项表示在显示符合条件的行的同时，也会显示其前后的行
grep "L" test.txt -i -n --color -C2

# 利用grep判断文本中是否存在某个字符串在写脚本时，你可能只是想要利用grep判断文本中是否存在某个字符串，你只关心有没有匹配到，而不关心匹配到的内容，你只关心有，或者没有，这时，我们可以使用grep的静默模式，示例如下
grep "L" test.txt -i -n --color -C2
grep "L" test.txt -i -n --color -C2 -q
echo $?

# 使用”-c”选项即可只统计符合条件的总行数，而不会打印出行。 显示出文章中有多少行有a或b或c
grep "^i" test.txt -i -n -o -c

# 输出以.结尾的行
grep "\.$" test.txt -n -o

# 不不打印以.结尾的行
grep "\.$" test.txt -n -v

# ^$(代表空行的意思)组合符.  如：找出文件的空行, 以及行号
grep "^$" test.txt -n

# "."点表示任意一个字符, 有且只有一个, 不包含空行
grep "." test.txt -n

# "*"表示找出前一个字符0次或一次以上. 如：找出文件中i出现0次或多次的行和行号
grep "i*" test.txt -n

# ".*"表示所有内容, 包括空行
grep ".*" test.txt -n -o

# ^.*t符 (含义: 以任意内容开头, 直到t结束)
grep "^.*t" test.txt -n -o

# 中括号表达式,[abc]表示匹配中括号中任意一个字符  如：匹配abc字符中的任意一个,得到它的行数和行号
grep "[abc]" test.txt -n -o

# [^abc]或[^a-c]这样的命令, "^"符号在中括号中第一位表示排除, 就是排除字符a,b,c
grep "[^abc]" test.txt -n

# +号表示匹配前一个字符1一次或多次,必须使用grep -E扩展正则
grep -E "a+" test.txt -n -o

# ?符 匹配前一个字符0次或1次 如 找出文件中包含ji或者joi的行
grep -E "jo?i" test.txt -n -o

# 竖线|在正则中是或者的意思 ，如 找出文件中包含j或者i的行, 不区分大小写(-i)
grep -E "j|i" test.txt -n -i

# ()小括号 将一个或多个字符捆绑在一起, 当作一个整体进行处理
grep -E "(ji)|(joi)" test.txt -n -i -o

# {n,m}匹配次数
# {n,m}:匹配前一个字符至少n次, 最多m次
# {n,}: 匹配前一个字符至少n次, 没有上限
# {,m}: 匹配前一个字符最多m次,可以没有
# 重复前一个字符各种次数, 可以通过-o参数显示明确的匹配过程
grep -E "a{1,3}" test.txt -o


```