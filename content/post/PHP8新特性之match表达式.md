---
title: "PHP8新特性之match表达式"
date: 2023-03-14T20:46:01+08:00
draft: false
categories: ["PHP8"]
tags: ["PHP8"]
keywords: ["PHP8","match"]
---

PHP8 alpha2发布了，最近引入了一个新的关键字：match, 这个关键字的作用跟switch有点类似。

虽然我一般对语法糖无感，但这个我觉得还是有点意思，match这个词也挺好看，那么它是干啥的呢？

在以前我们可能会经常使用switch做值转换类的工作，类似:

``` php
  switch ($input) {
      case "true":
          $result = 1;
      break;
      case "false":
          $result = 0;
      break;
      case "null":
          $result = NULL;
      break;
  }
```
(当然，有的同学会说，谁会这么写，用个数组转换不行么？ 拜托，这是举例啊，数组也只能数字键和整数啊，万一key是需要其他表达式呢，万一你要多个key对应一个值呢，对吧？)

那么如果使用match关键字呢，可以变成类似:
``` php
$result = match($input) {
          "true" => 1,
          "false" => 0,
          "null" => NULL,
};
```


相比switch， match会直接返回值，可以直接赋值给$result了。

并且，类似switch的多个case一个block一样，match的多个条件也可以写在一起，比如:

``` php
  $result = match($input) {
      "true", "on" => 1,
      "false", "off" => 0,
      "null", "empty", "NaN" => NULL,
};
```

需要注意的和switch不太一样的是，以前我们用switch可能会经常遇到这种诡异的问题:

``` php
  $input = "2 person";
  switch ($input) {
      case 2:
          echo "bad";
      break;
}
```

你会发现，bad竟然被输出了，这是因为switch使用了宽松比较(==)。match就不会有这个问题了, 它使用的是严格比较(===)，就是值和类型都要完全相等。

还有就是，当input并不能被match中的所有条件满足的时候，match会抛出一个UnhandledMatchError exception:

``` php
  $input = "false";
  $result = match($input) {
          "true" => 1,
  };
```

会得到:

1.  Fatal error: Uncaught UnhandledMatchError: Unhandled match value of type string

这样就不用担心万一match条件没写全导致了不可预知的错误。

另外还是要说明，match是关键字，也就是从PHP8开始它不能出现在namespace或者类名中，如果你的项目中有用match作为类名的:

1.  class Match {}

在PHP8开始将会得到语法错误了, 当然，方法名中还是可以用的。

详细的，可以参考RFC：[Match Expression](https://wiki.php.net/rfc/match_expression_v2)

[原文地址](https://www.laruence.com/2020/07/13/6033.html)