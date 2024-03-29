---
title: "一些PHP基础的简单整理"
date: 2024-02-18T20:46:01+08:00
draft: false
categories: ["PHP"]
tags: ["PHP"]
keywords: ["PHP"]
---


# 一些PHP基础的简单整理

## 常量
1. 关于常量定义
```php
<?php
$a = 1;
define("AAA".$a,"AAA");
echo constant("AAA".$a);
```
define() 在运行时定义一个常量。  
constant() 返回一个常量的值。当你不知道常量名，却需要获取常量的值时，constant() 就很有用了。也就是说，常量名储存在一个变量里，或者由函数返回时。

const不能在条件语句中定义常量。
const用于类成员变量的定义，define()不可用于类成员变量的定义，可用于全局变量。  
const可在类中使用，define不能。

2. 常量和变量的区别
   * 常量前面没有且不能有$符号
   * 常量可以不用理会变量的作用域，在任何地方定义和使用
   * 常量一旦定义就不能重新定义或取消定义
   * 常量的值只能是标量（字符串、整数、浮点数、布尔值），支持数组  
  
    获取所有常量  
    get_defined_constants()  
    get_defined_constants(true)  
    get_defined_constants(true)['user']  

3. 魔术常量：它的值随着它在代码中的位置改变而变化
   * __LINE__  文件中的当前行号
   * __FILE__  文件的完整路径和文件名，包含根目录
   * __DIR__  文件所在目录
   * __FUNCTION__  该函数被定义时的名字，区分大小写
   * __CLASS__  该类被定义时的名字，区分大小写
   * __NAMESPANCE__  命名空间
   * __METHOD__  命名空间类名方法名
   * __TRAIT__  当前使用的trait的名字

## 包含文件
1. include和require的区别
   include和require除了错误处理的方式不同，其他方面都是相同的。  
   require生成一个致命错误（E_COMPILE_ERROR），在错误发生后脚本停止运行  
   include生成一个警告（E_WARNING）,在错误发生后脚本继续运行

## 面向对象(OO)
1. 概念
   * 面向对象是一种编程思想和方法，它将程序中的数据和操作数据的方法封装在一起，形成“对象”，并通过对象之间的交互和消息传递来完成程序的功能。 
   * 面向对象编程（OOP）强调数据的封装、继承、多态和动态绑定等特性，使得程序具有更好的可扩展性、可维护性和可重用性。
2. OOP特性
   *  封装：指将对象的属性和方法封装在一起，使得外部无法直接访问和修改对象的内部状态。通过使用访问控制修饰符（public、private、protected）来限制属性和方法的访问权限，从而实现封装
   *  继承：指可以创建一个新的类该类继承（extends）了父类的属性和方法，并且可以添加自己的属性和方法。通过继承，可以避免重复编写相似的代码，并且可以实现代码的复用。
   *  多态：指可以使用一个父类类型的变量来引用不同子类类型的对象，从而实现对不同对象统一操作。多态可以使得代码更加灵活，具有更好的可扩展性和可维护性。在PHP中多态可通过实现接口（interface）和使用抽象类（abstract class）来实现。
3. 类的访问控制
    * public（公有）：公有的类成员可以在任何地方被调用。
    * protected（受保护）：受保护的类成员可以被其自身以及其子类和父类访问。
    * private（私有）：私有的类成员只能被其定义所在的类访问 
4. 构造方法 __construct
    * 构造方法是一种特殊的方法，在创建一个新对象时，它会被自动调用
    * 他可以用来 **初始化** 对象的属性或执行其他必要的操作
    * 没有返回值
5. 析构函数 __destruct
    * 析构函数是一种特殊的方法，它在对象被销毁时自动调用
    * 它可以用来执行一些清理操作，如释放资源或关闭连接
    * 当对象不再被引用或脚本执行结束时，析构函数会被自动调用
6. final 关键字
    * 防止类被继承
    * 防止类的方法被重写
    * 如果一个类前加final，那么这个类不能被继承
    * 如果一个方法前加final，name这个方法不能被重写
7. PHP类的多态
    * 多态性允许不同类的对象对相同的消息作出不同的响应
    * 多态性通过方法重写(覆盖)和方法的重载来实现
    * 方法重写是指子类重写父类的方法，以改变方法的实现细节
    * 方法重载是指在同一个类中根据参数个数或类型不同来实现不同功能
    * 需要注意的是，多态性只适用于继承关系的类。子类必须重写父类的方法才能实现多态性
8. interface接口和抽象类
   1. interface接口
   接口是指一组方法的集合，不是类，不能被实例化
      *  可以指定某个类必须实现那些方法，但不需要定义这些方法的具体内容
      *  只能使用public
      *  通常用于定义一些规范，让代码更加有条理，不易出错
        ```php
        <?php
        // 定义接口
        interface A()
        {
            public function a();
            public static function b();

        }

        //实现接口
        class Test implements A {
            public function a(){}
            public static function b(){}
        }
        ```

    2. 抽象类  abstract
    和接口相似，使用它也是定义一种约束或规范，适合较大型的项目或库使用。
        * 抽象类是一种特殊的类，只能被继承，不能被实例化
        * 抽象类用于定义一组相关的方法，但这些方法的实现由继承它的子类来完成
        * 子类继承抽象类后，必须实现抽象类中的所有抽象方法
        * 抽象类只能包含抽象方法和普通方法
    ```php
    <?php
    // 定义
    abstract class A {
        abstract public function a();
        abstract protected function b();
        public function c(){

        };
    }

    //继承
    class B extends A {
        public function a(){};
        protected function b(){};
        public function c(){};
    }
    ```

    3. 抽象类和接口的区别
       * 抽象类可以包含非抽象方法的实现，而接口只能包含方法的声明，没有方法的实现。
       * 类只能继承一个抽象类，但可以实现多个接口
       * 抽象类可以有构造函数，而接口不能
       * 抽象类中的方法可以有public、protected、provate可以有public、protected、private访问修饰符，而接口类中的方法只能是public
       * 子类继承抽象类时，必须实现抽象类中所有的抽象方法，负责子类也必须声明为抽象类
       * 子类实现接口时，必须实现接口中的所有方法

9. 关键字 trait 实现复用   
    * 解决类的单一继承问题
    * 可同时使用多个trait，用都好隔开
    * 把常用的、通用的代码抽离出来，写成trait
    * 和类的继承非常像，但是trait里面不能有类常量，且trait不能被实例化
  
    ```
    <?php
        // 声明
        trait A{

        }
        trait B{
            
        }

        // 使用
        class C {
            use A,B;
        }
    ```





## 数组
1. 数组的合并  
    算术运算符+  
    “+”运算符把右边的数组元素附加到左边的数组后面，两个数组中都有的键名，则只用左边数组中的，右边的被忽略
  
    array_merge()  
    将一个或多个数组的单元合并起来，一个数组中的值附加在前一个数组的后面。返回作为结果的数组。  
    如果输入的数组中有相同的字符串键名，则该键名后面的值将覆盖前一个值。然而，如果数组包含数字键名，后面的值将 不会 覆盖原来的值，而是附加到后面。  
    如果输入的数组存在以数字作为索引的内容，则这项内容的键名会以连续方式重新索引。

    算术运算符+ 和array_merge()都会对数组进行合并操作，如果存在相同的键+会保留前面数组的数据，而merge会保留最新的数据也就是后面的数据。如

    ```php
    <?php
    $a = [
        'a'=>1,2,3,
    ];
    $b = [
        'a'=>3,4,5,
    ];
    print_r($a+$b);
    print_r(array_merge($a,$b));
    //Array
    //(
    //    [a] => 1
    //    [0] => 2
    //    [1] => 3
    //)
    //Array
    //(
    //    [a] => 3
    //    [0] => 2
    //    [1] => 3
    //    [2] => 4
    //    [3] => 5
    //)

    ```
2. 数组的比较
   
    $a == $b	相等	如果 $a 和 $b 具有相同的键／值对则为 true。  
    $a === $b	全等	如果 $a 和 $b 具有相同的键／值对并且顺序和类型都相同则为 true。  
    $a != $b	不等	如果 $a 不等于 $b 则为 true。  
    $a <> $b	不等	如果 $a 不等于 $b 则为 true。  
    $a !== $b	不全等	如果 $a 不全等于 $b 则为 true。
    $a <=> $b	太空船运算符（组合比较符）	当$a小于、等于、大于 $b时 分别返回一个小于、等于、大于0的 int 值。（即1，0，-1）  

## session
1. session和cookie的区别  
   cookie是一种服务器留在计算机上的小文件。PHP操作cookie的方法：setcookie()

   session 用于在服务器上存储关于用户会话的信息，并且对于程序中的所有页面都是可用的。  
   会话信息是临时的，在用户离开网站后会被删除。  
   它允许在同一个用户的多个请求之间保持数据的状态，便于传递数据  
   Session的工作机制是：为每个访客创建一个唯一的ID（UID），并基于这个UID来存储变量。UID存储在cookie中，或者通过URL进行传导  
   PHP启动session： session_start(),使用$_SESSION 来操作session

    关系：  
    * session和cookie都是用于在不同请求之间保持数据的状态
    * session使用cookie来跟踪和表示用户，通常在cookie中存储Session ID 来关联服务器端的session数据
  
    区别：
    * 存储位置：cookie的数据存储在客户端的浏览器中，而session数据存储在服务器端。
    * 容量限制：cookie的容量限制通常较小，一般为几kb。而session的容量限制较大，通常取决于服务器的配置。
    * 安全性：由于cookie存储在客户端，可能会被篡改或窃取，因此存储敏感信息的安全性较低。相比之下，session数据存储在服务器端，相对更加安全。
    * 生命周期：cookie可以设置过期时间，可以在浏览器关闭后仍然保持，具有较长的生命周期。而session通常在用户关闭浏览器或一段时间不活动后自动过期。
    * 存储方式：cookie以键值对形式存储数据，可以在客户端进行读取和修改。session数据存储在服务器端，客户端只保存了一个Session ID。


