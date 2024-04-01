---
title: "PHP 8 新特性之枚举"
date: 2023-06-11T13:46:01+08:00
draft: false
categories: ["PHP8"]
tags: ["PHP8"]
keywords: ["PHP8"]
---

# PHP 8 新特性之枚举

## 枚举概览
枚举，或称 “Enum”，能够让开发者自定义类型为一系列可能的离散值中的一个。 在定义领域模型中很有用，它能够“隔离无效状态”（making invalid states unrepresentable）。

枚举以各种不同功能的形式出现在诸多语言中。 在 PHP 中， 枚举是一种特殊类型的对象。Enum 本身是一个类（Class）， 它的各种条目（case）是这个类的单例对象，意味着也是个有效对象 —— 包括类型的检测，能用对象的地方，也可以用它。

最常见的枚举例子是内置的 boolean 类型， 该枚举类型有两个有效值 true 和 false。 Enum 使开发者能够任意定义出用户自己的、足够健壮的枚举。

<br>

Enum 类似 class，它和 class、interface、trait 共享同样的命名空间。 也能用同样的方式自动加载。 一个 Enum 定义了一种新的类型，它有固定、数量有限、可能的合法值。
示例 #1 用注解实现接口的可选方法
```php
<?php

// 通过传入参数：待搜索的注解类名，可返回指定的注解类， 而不需要再到反射类中迭代循环获取所有注解。
```

示例
```php
<?php

<?php
// 声明新的枚举类型 Suit，仅有四个有效的值： Suit::Hearts、Suit::Diamonds、 Suit::Clubs、Suit::Spades。
enum Suit
{
    case Hearts;
    case Diamonds;
    case Clubs;
    case Spades;
}

//  变量可以赋值为以上有效值里的其中一个。 函数可以检测枚举类型，这种情况下只能传入类型的值。
function pick_a_card(Suit $suit)
{
    /* ... */
}

$val = Suit::Diamonds;

// OK
pick_a_card($val);

// OK
pick_a_card(Suit::Clubs);
?>

```

所有的case都有一个只读的属性 name，它大小写敏感，是case自身的名称
``` 
var_dump(Suit::Diamonds->name);
```

enum_exists() 函数判断枚举是否存在

### 枚举值
可以直接分配值和获取值  
如果决定分配枚举值，则所有情况都具有值，不能混用  
```php
<?php
enum Test: String
{
    case One = 'O';
    case Two = 'T';
    case Three = 'R';
}
var_dump(Test::One);
var_dump(Test::One->value);

//根据值去找枚举
var_dump(Test::from('O'));
var_dump(Test::tryFrom('O')); //找不到返回null

```
注意：只有int 和 string 才被支持

静态方法cases()，返回数组，获取枚举中所有值，
``` 
var_dump(Suit::cases());
```

枚举中可以定义方法，和普通类一样。

也能实现 interface。 如果 Enum 实现了 interface，则其中的条目也能接受 interface 的类型检测。

<?php

参考文章： [PHP: 枚举概览 - Manual ](https://www.php.net/manual/zh/language.enumerations.overview.php)