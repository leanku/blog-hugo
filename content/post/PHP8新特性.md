---
title: "PHP 8 新特性"
date: 2023-05-11T13:46:01+08:00
draft: false
categories: ["PHP8"]
tags: ["PHP8"]
keywords: ["PHP8"]
---

# PHP 8 新特性

## 命名参数
新增命名参数的功能:

从 PHP 8.0.0 开始，函数参数列表可以包含一个尾部的逗号，这个逗号将被忽略。这在参数列表较长或包含较长的变量名的情况下特别有用，这样可以方便地垂直列出参数。
```php
<?php
function takes_many_args(
    $first_arg,
    $second_arg,
    $a_very_long_argument_name,
    $arg_with_default = 5,
    $again = 'a default string', // 在 8.0.0 之前，这个尾部的逗号是不允许的。
)
{
    // ...
}
?>
```

## 注解（Attributes）
新增[注解](https://www.php.net/manual/zh/language.attributes.php)的功能。
[此篇]()单独介绍


## 构造器属性提升（Constructor Property Promotion）
新增[构造器属性提升](https://www.php.net/manual/zh/language.oop5.decon.php#language.oop5.decon.constructor.promotion)功能 在构造函数中声明类的属性）。

构造器的参数也可以相应提升为类的属性。 构造器的参数赋值给类属性的行为很普遍，否则无法操作。 而构造器提升的功能则为这种场景提供了便利。 
```php
<?php
class Point {
    protected int $x;
    protected int $y;

    public function __construct(int $x, int $y = 0) {
        $this->x = $x;
        $this->y = $y;
    }
}

// 两个参数都传入
$p1 = new Point(4, 5);
// 仅传入必填的参数。 $y 会默认取值 0。
$p2 = new Point(4);
// 使用命名参数（PHP 8.0 起）:
$p3 = new Point(y: 5, x: 4);

// 上面的例子可以用以下方式重写：
class Point {
    public function __construct(protected int $x, protected int $y = 0) {
    }
}
```

## 联合类型
新增 联合类型。

联合类型是两个或者多个类型的集合，表示可以使用其中任何一个类型。
```php
public function foo(Foo|Bar $input): int|float;

// 联合类型中不包含 void，因为 void 表示的含义是 “根本没有返回值”。  另外，可以使用 |null 或者现有的 ? 表示法来表示包含 nullable 的联合体 
public function foo(Foo|null $foo): void;

public function bar(?Bar $bar): void;
```

## Match 表达式
新增 [match](https://www.php.net/manual/zh/control-structures.match.php) 表达式。
: match 表达式基于值的一致性进行分支计算。 match表达式和 switch 语句类似， 都有一个表达式主体，可以和多个可选项进行比较。 与 switch 不同点是，它会像三元表达式一样求值。 与 switch 另一个不同点，它的比较是严格比较（ ===）而不是松散比较（==）。 Match 表达式从 PHP 8.0.0 起可用。
```php
<?php
$return_value = match (subject_expression) {
    single_conditional_expression => return_expression,
    conditional_expression1, conditional_expression2 => return_expression,
};
?>
```

## Nullsafe 运算符
新增[Nullsafe](https://www.php.net/manual/zh/language.oop5.basic.php#language.oop5.basic.nullsafe) 运算符(?->)。

自 PHP 8.0.0 起，类属性和方法可以通过 "nullsafe" 操作符访问： ?->。 除了一处不同，nullsafe 操作符和以上原来的属性、方法访问是一致的： 对象引用解析（dereference）为 null 时不抛出异常，而是返回 null。 并且如果是链式调用中的一部分，剩余链条会直接跳过。
```php
<?php

// 自 PHP 8.0.0 起可用
$result = $repository?->getUser(5)?->name;

// 上边那行代码等价于以下代码
if (is_null($repository)) {
    $result = null;
} else {
    $user = $repository->getUser(5);
    if (is_null($user)) {
        $result = null;
    } else {
        $result = $user->name;
    }
}
// 仅当 null 被认为是属性或方法返回的有效和预期的可能值时，才推荐使用 nullsafe 操作符。如果业务中需要明确指示错误，抛出异常会是更好的处理方式。
?>
```

## 其他新特性
* 新增 [WeakMap](https://www.php.net/manual/zh/class.weakmap.php) 类。

* 新增 [ValueError](https://www.php.net/manual/zh/class.valueerror.php) 类。

* 现在，只要类型兼容，任意数量的函数参数都可以用一个可变参数替换。 例如允许编写下面的代码：
  ```php
  <?php
  class A {
      public function method(int $many, string $parameters, $here) {}
  }
  class B extends A {
      public function method(...$everything) {}
  }
  ?>
  ```

* static ("后期静态绑定"中) 可以作为返回类型：
  ```php
  <?php
  class Test {
      public function create(): static {
            return new static();
      }
  }
  ?>
  ```

* 现在可以通过 $object::class 获取类名，返回的结果和 get_class($object) 一致。

* new、instanceof 可用于任何表达式， 用法为 new (expression)(...$args) 和 $obj instanceof (expression)。

* 添加对一些变量语法一致性的修复，例如现在能够编写 Foo::BAR::$baz。

* 添加 Stringable interface， 当一个类定义 __toString() 方法后会自动实现该接口。

* Trait 可以定义私有抽象方法（abstract private method）。 类必须实现 trait 定义的该方法。

* 可作为表达式使用 throw。 使得可以编写以下用法：

  ```php
  <?php
  $fn = fn() => throw new Exception('Exception in arrow function');
  $user = $session->user ?? throw new Exception('Must have user');
  ```

* 参数列表中的末尾逗号为可选。

  ```php
  <?php
  function functionWithLongSignature(
      Type1 $parameter1,
      Type2 $parameter2, // <-- 这个逗号也被允许了
  ) {
  }
  ```
* 现在允许 catch (Exception) 一个 exception 而无需捕获到变量中。

* 支持 mixed 类型。

* 父类中声明的私有方法不在对子类中的方法执行任何继承规则（final private 构造函数除外）。下列示例说明删除了那些限制：

  ```php
  <?php
  class ParentClass {
      private function method1() {}
      private function method2() {}
      private static function method3() {}
      // 抛出警告，因为“final”不再有效：
      private final function method4() {}
  }
  class ChildClass extends ParentClass {
      // 现在允许以下所有内容，即使修饰符与父类中的私有方法不同。
      public abstract function method1() {}
      public static function method2() {}
      public function method3() {}
      public function method4() {}
  }
  ?>
  ```

* 新增 get_resource_id()，返回值跟 (int) $resource 相同。其在更清晰的 API 下提供了相同的功能。

* 新增 InternalIterator。
  
## 日期和时间
* 新增 [DateTime::createFromInterface()](https://www.php.net/manual/zh/datetime.createfrominterface.php) 和 [DateTimeImmutable::createFromInterface()](https://www.php.net/manual/zh/datetimeimmutable.createfrominterface.php)。

* 新增 DateTime 格式化标识符 p，跟 P 相同，但对于 UTC 返回 Z 而不是 +00:00。

## Filter

新增 FILTER_VALIDATE_BOOL，是 FILTER_VALIDATE_BOOLEAN 的别名。首选新名称，因为其使用规范类型名称。


## 标准库
* 新增 str_contains()、str_starts_with() 和 str_ends_with()，分别检查 haystack 是否包含 needle 或以 needle 开头/结尾。
* 新增 fdiv() 在 IEEE 754 语义下执行浮点除法。认为除以零已经明确定义，将返回 Inf、-Inf 或 NaN。
* 新增 get_debug_type() 返回对错误消息有用的类型。与 gettype() 不同的是，它使用规范的类型名称，为对象返回类名，为资源表示资源类型。
* printf() 和相关函数现在支持 %h 和 %H 格式说明符。它们与 %g 和 %G 相同，但始终使用 "." 作为小数点分隔符，而不是通过 LC_NUMERIC 区域确定。
* 现在，printf() 和相关函数支持使用 "*" 作为宽度或精度，此时宽度/精度将作为参数传递给 printf。这也允许在 %g、%G、%h 和 %H 中使用精度 -1。例如，以下代码可用于重现 PHP 的默认浮点数格式化：
  ```php
  <?php
  printf("%.*H", (int) ini_get("precision"), $float);
  printf("%.*H", (int) ini_get("serialize_precision"), $float);
  ?>
  ```
* 现在，proc_open() 支持伪终端（PTY）描述符。以下将 stdin、stdout 和 stderr 附加到同一个 PTY：
  ```php
  <?php
  $proc = proc_open($command, [['pty'], ['pty'], ['pty']], $pipes);
  ?>
  ```
* proc_open() 现在支持套接字对描述符。以下将独立的套接字对附加到stdin、stdout 和 stderr：
  ```php
  <?php
  $proc = proc_open($command, [['socket'], ['socket'], ['socket']], $pipes);
  ?>
  // 与管道不同，套接字在 Windows 上不会出现阻塞 I/O 问题。然而，并非所有程序都能正确地与 stdio 套接字配合工作。
  ```
  * 排序函数现在已稳定，这意味着相等的元素比较将保留其原始顺序。
  * array_diff() 和 array_intersect() 及其变体现在可以接受单个数组作为参数。这意味着现在可以使用下列用法：
  ```php
  <?php
  // 如果 $excludes 为空也可以：
  array_diff($array, ...$excludes);
  // 如果 $arrays 仅包含单个数组也可以：
  array_intersect(...$arrays);
  ?>
  ```
  * ob_implicit_flush() 的 flag 参数已经从接受 int 变更为接受 bool。


参考文章： [PHP: 新特性 - Manual ](https://www.php.net/manual/zh/migration80.new-features.php)