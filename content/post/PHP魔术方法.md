---
title: "PHP魔术方法"
date: 2024-03-10T13:46:01+08:00
draft: false
categories: ["PHP"]
tags: ["PHP"]
keywords: ["PHP","PHP魔术方法"]
---

# PHP魔术方法
魔术方法是一种特殊的方法，当对对象执行某些操作时会覆盖 PHP 的默认操作。

PHP中把以两个下划线__开头的方法称为魔术方法(Magic methods)，这些方法在PHP中充当了举足轻重的作用。

1.  __construct()，类的构造函数。  
    PHP 允许开发者在一个类中定义一个方法作为构造函数。具有构造函数的类会在每次创建新对象时先调用此方法，所以非常适合在使用对象之前做一些初始化工作。
2.  __destruct()，类的析构函数。 析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。
3.  __call()，在对象中调用一个不可访问方法时调用。
4.  __callStatic()，用静态方式中调用一个不可访问方法时调用
5.  __get()，获得一个类的成员变量时调用。 读取不可访问（protected 或 private）或不存在的属性的值时，__get() 会被调用
6.  __set()，设置一个类的成员变量时调用。 在给不可访问（protected 或 private）或不存在的属性赋值时，__set() 会被调用。
7.  __isset()，当对不可访问属性调用isset()或empty()时调用。 当对不可访问（protected 或 private）或不存在的属性调用 isset() 或 empty() 时，__isset() 会被调用。
8.  __unset()，当对不可访问属性调用unset()时被调用。 当对不可访问（protected 或 private）或不存在的属性调用 unset() 时，__unset() 会被调用。
9.  __sleep()，执行serialize()时，先会调用这个函数。 __sleep() 方法常用于提交未提交的数据，或类似的清理操作。
10. __wakeup()，执行unserialize()时，先会调用这个函数。 __wakeup() 经常用在反序列化操作中，例如重新建立数据库连接，或执行其它初始化操作。
11. __toString()，类被当成字符串时的回应方法
12. __invoke()，调用函数的方式调用一个对象时的回应方法
13. __set_state()，调用var_export()导出类时，此静态方法会被调用。
14. __clone()，当对象复制完成时调用
15. __autoload()，尝试加载未定义的类
16. __debugInfo()，打印所需调试信息

参考文章： [PHP: 魔术方法 - Manual](https://www.php.net/__wakeup)