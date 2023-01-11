---
title: "PHP标准输入与输出"
date: 2023-01-11
draft: false
categories: ["PHP"]
tags: ["PHP"]
keywords: ["PHP","输入","输出"]
---

# PHP标准输入与输出

## 一、PHP STDIN、STDOUT、STDERR简介:

STDIN、STDOUT、STDERR命令输入输出流，用于向控制台(linux shell终端、windows cmd终端)输入、输出内容，它们默认是已经打开的，可以直接对他们进行读写操作，它们只能在CLI(command-line interface，命令行界面)模式中使用，在Http模式时，它们是未定义的。

而他们的打开副本php://stdin、php://stdout、php://stderr 也无法输出内容到http浏览器，经测试：写入php://stderr的内容将会输入到默认站点的错误日志中，其它两种无任何效果。

STDIN/STDOUT/STDERR简介：原始流流打开副本描述STDINphp://stdin标准输入(standard input)，只读，用于从控制台输入内容；

STDOUTphp://stdout标准输出(standard output)，只写，用于向控制台输出正常信息；

STDERRphp://stderr错误输出(standard error)，只写，用于向控制台输出错误信息；

官方推荐使用常量 STDIN、 STDOUT 和 STDERR 来代替它们手动打开的副本封装器php://stdin、 php://stdout 和 php://stderr。

## 二、PHP STDIN用法：

PHP语言中"STDIN"用于从控制台读取内容，遇到此常量或者通过fopen()函数打开php://stdin脚本将会等待用户输入内容，直到用户按下回车键提交。

新建文件demo.php
```php
<?php

echo "请输入内容:";
$input = fgets(STDIN);
echo sprintf("输入的内容为: %s\n", $input);

$handle = fopen('php://stdin', 'r');
echo "请输入: ";
$stdin = fread($handle, 10); //最多读取10个字符
echo sprintf("输入为: %s\n", $stdin);
fclose($handle);

//php demo.php 运行代码，键盘输入123，输出结果如下

//请输入内容:123
//输入的内容为: 123

//请输入: 456
//输入为: 456
```

<br>

## 三、PHP STDOUT用法：

PHP语言中STDOUT用于向控制台输出标准信息；向此常量、或者向fopen()函数打开的php://stdout写入的内容将直接输出到控制台的标准输出；标准输出的内容可以用过">"或者"1>"重定向到指定地方，比如文件。

```php
<?php

fwrite(STDOUT, "STDOUT写入的正常输出；\n");

$stdout = fopen("php://stdout", "w");

fwrite($stdout, "php://stdout写入的正常输出；\n");

fclose($stdout);

// php demo.php >stdout.txt
//stdout.txt文件类容如下
//STDOUT写入的正常输出；
//php://stdout写入的正常输出；
```


## 四、PHP STDERR用法：
PHP语言中"STDERR"用于向控制台输出错误信息；向常量、或者向fopen()函数打开的"php://stderr"写入的内容将直接输出到控制台的错误输出；错误输出的内容可以用过"2>"重定向到指定地方，比如文件；也可以使用"2>&1"将错误输出定向到标准输出，与标准输出合并。

```php
<?php
fwrite(STDERR, "STDERR写入的错误输出；\n");

$stderr = fopen("php://stderr", "w");

fwrite($stderr, "php://stderr写入的错误输出；\n");

fclose($stderr);

// php demo.php 输出如下
//STDERR写入的错误输出；
//php://stderr写入的错误输出；
```