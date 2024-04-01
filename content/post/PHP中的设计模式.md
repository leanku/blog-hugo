---
title: "PHP中的设计模式"
date: 2024-03-15T13:46:01+08:00
draft: false
categories: ["PHP"]
tags: ["PHP","设计模式"]
keywords: ["PHP","设计模式"]
---

# PHP中的设计模式

## 介绍
    设计模式：提供了一种广泛的可重用的方式来解决我们日常编程中常常遇见的问题。设计模式并不一定就是一个类库或者第三方框架，它们更多的表现为一种思想并且广泛地应用在系统中。它们也表现为一种模式或者模板，可以在多个不同的场景下用于解决问题。设计模式可以用于加速开发，并且将很多大的想法或者设计以一种简单地方式实现。当然，虽然设计模式在开发中很有作用，但是千万要避免在不适当的场景误用它们。

## 分类
### 按照目的分，目前常见的设计模式主要有23种，根据使用目标的不同可以分为以下三大类：

#### 创建设计模式（Creational Patterns）(5种)：用于创建对象时的设计模式。更具体一点，初始化对象流程的设计模式。当程序日益复杂时，需要更加灵活地创建对象，同时减少创建时的依赖。而创建设计模式就是解决此问题的一类设计模式。
* 单例模式【Singleton】
* 工厂模式【Factory】
* 抽象工厂模式【AbstractFactory】
* 建造者模式【Builder】
* 原型模式【Prototype】

#### 结构设计模式（Structural Patterns）(7种)：用于继承和接口时的设计模式。结构设计模式用于新类的函数方法设计，减少不必要的类定义，减少代码的冗余。

* 适配器模式【Adapter】
* 桥接模式【Bridge】
* 合成模式【Composite】
* 装饰器模式【Decorator】
* 门面模式【Facade】
* 代理模式【Proxy】
* 享元模式【Flyweight】

#### 行为模式（Behavioral Patterns）(11种)：用于方法实现以及对应算法的设计模式，同时也是最复杂的设计模式。行为设计模式不仅仅用于定义类的函数行为，同时也用于不同类之间的协议、通信。

* 策略模式【Strategy】
* 模板方法模式【TemplateMethod】
* 观察者模式【Observer】
* 迭代器模式【Iterator】
* 责任链模式【ResponsibilityChain】
* 命令模式【Command】
* 备忘录模式【Memento】
* 状态模式【State】
* 访问者模式【Visitor】
* 中介者模式【Mediator】
* 解释器模式【Interpreter】

### 按照范围分为：类的设计模式，以及对象设计模式

* 类的设计模式(Class patterns)：用于类的具体实现的设计模式。包含了如何设计和定义类，以及父类和子类的设计模式。

* 对象设计模式(Object patterns): 用于对象的设计模式。与类的设计模式不同，对象设计模式主要用于运行期对象的状态改变、动态行为变更等。

## 设计模式原则
**设计模式六大原则**
* 开放封闭原则：一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。
* 里氏替换原则：所有引用基类的地方必须能透明地使用其子类的对象.
* 依赖倒置原则：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。
* 单一职责原则：不要存在多于一个导致类变更的原因。通俗的说，即一个类只负责一项职责。
* 接口隔离原则：客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上。
* 迪米特法则：一个对象应该对其他对象保持最少的了解。


## 设计模式实现 
### 1. Singleton(单例模式)
单例模式是最常见的模式之一，在Web应用的开发中，常常用于允许在运行时为某个特定的类创建仅有一个可访问的实例。
```php
<?php
final class Mysql
{
 
    /**
     *
     * @var self[该属性用来保存实例]
     */
    private static $instance;
 
    /**
     *
     * @var mixed
     */
    public $mix;
 
    /**
     * Return self instance[创建一个用来实例化对象的方法]
     *
     * @return self
     */
    public static function getInstance()
    {
        if (! (self::$instance instanceof self)) {
            self::$instance = new self();
        }
        return self::$instance;
    }
 
    /**
     * 构造函数为private,防止创建对象
     */
    private function __construct()
    {}
 
    /**
     * 防止对象被复制
     */
    private function __clone()
    {
        trigger_error('Clone is not allowed !');
    }
}
 
// @test
$firstMysql = Mysql::getInstance();
$secondMysql = Mysql::getInstance();
 
$firstMysql->mix = 'ityangs_one';
$secondMysql->mix = 'ityangs_two';
 
print_r($firstMysql->mix);
// 输出： ityangs_two
print_r($secondMysql->mix);
```

### 2. Factory(工厂模式)
工厂模式是另一种非常常用的模式，正如其名字所示：确实是对象实例的生产工厂。某些意义上，工厂模式提供了通用的方法有助于我们去获取对象，而不需要关心其具体的内在的实现。
```php
<?php
interface SystemFactory
{
    public function createSystem($type);
}

class MySystemFactory implements SystemFactory
{
    // 实现工厂方法
    public function createSystem($type)
    {
        switch ($type) {
            case 'Mac':
                return new MacSystem();
            case 'Win':
                return new WinSystem();
            case 'Linux':
                return new LinuxSystem();
        }
    }
}
 
class System{ /* ... */}
class WinSystem extends System{ /* ... */}
class MacSystem extends System{ /* ... */}
class LinuxSystem extends System{ /* ... */}
 
//创建我的系统工厂
$System_obj = new MySystemFactory();
//用我的系统工厂分别创建不同系统对象
var_dump($System_obj->createSystem('Mac'));//输出：object(MacSystem)#2 (0) { }
var_dump($System_obj->createSystem('Win'));//输出：object(WinSystem)#2 (0) { }
var_dump($System_obj->createSystem('Linux'));//输出：object(LinuxSystem)#2 (0) { }
```

### 3. AbstractFactory(抽象工厂模式)
有些情况下我们需要根据不同的选择逻辑提供不同的构造工厂，而对于多个工厂而言需要一个统一的抽象工厂：
```php
<?php
 
class System{}
class Soft{}
 
class MacSystem extends System{}
class MacSoft extends Soft{}
 
class WinSystem extends System{}
class WinSoft extends Soft{}
 
interface AbstractFactory {
    public function CreateSystem();
    public function CreateSoft();
}
 
class MacFactory implements AbstractFactory{
    public function CreateSystem(){ return new MacSystem(); }
    public function CreateSoft(){ return new MacSoft(); }
}
 
class WinFactory implements AbstractFactory{
    public function CreateSystem(){ return new WinSystem(); }
    public function CreateSoft(){ return new WinSoft(); }
}
 
//@test:创建工厂->用该工厂生产对应的对象
 
//创建MacFactory工厂
$MacFactory_obj = new MacFactory();
//用MacFactory工厂分别创建不同对象
var_dump($MacFactory_obj->CreateSystem());//输出：object(MacSystem)#2 (0) { }
var_dump($MacFactory_obj->CreateSoft());// 输出：object(MacSoft)#2 (0) { }
 
 
//创建WinFactory
$WinFactory_obj = new WinFactory();
//用WinFactory工厂分别创建不同对象
var_dump($WinFactory_obj->CreateSystem());//输出：object(WinSystem)#3 (0) { }
var_dump($WinFactory_obj->CreateSoft());//输出：object(WinSoft)#3 (0) { }
```

### 4. Builder(建造者模式)
建造者模式主要在于创建一些复杂的对象。将一个复杂对象的构造与它的表示分离，使同样的构建过程可以创建不同的表示的设计模式;
```php
<?php
/**
 * 
 * 产品本身
 */
class Product { 
    private $_parts; 
    public function __construct() { $this->_parts = array(); } 
    public function add($part) { return array_push($this->_parts, $part); }
}
 
/**
 * 建造者抽象类
 *
 */
abstract class Builder {
    public abstract function buildPart1();
    public abstract function buildPart2();
    public abstract function getResult();
}
 
/**
 * 
 * 具体建造者
 * 实现其具体方法
 */
class ConcreteBuilder extends Builder {  
    private $_product;
    public function __construct() { $this->_product = new Product(); }
    public function buildPart1() { $this->_product->add("Part1"); } 
    public function buildPart2() { $this->_product->add("Part2"); }
    public function getResult() { return $this->_product; }
}
 /**
  * 
  *导演者
  */
class Director { 
    public function __construct(Builder $builder) {
        $builder->buildPart1();//导演指挥具体建造者生产产品
        $builder->buildPart2();
    }
}
 
// client 
$buidler = new ConcreteBuilder();
$director = new Director($buidler);
$product = $buidler->getResult();
echo "<pre>";
var_dump($product);
echo "</pre>";
/*输出： object(Product)#2 (1) {
["_parts":"Product":private]=>
array(2) {
    [0]=>string(5) "Part1"
    [1]=>string(5) "Part2"
}
} */
?>
```

### 5. Prototype(原型模式)
有时候，部分对象需要被初始化多次。而特别是在如果初始化需要耗费大量时间与资源的时候进行预初始化并且存储下这些对象，就会用到原型模式：

``` php
<?php
/**
 * 
 * 原型接口
 *
 */
interface Prototype { public function copy(); }
 
/**
 * 具体实现
 *
 */
class ConcretePrototype implements Prototype{
    private  $_name;
    public function __construct($name) { $this->_name = $name; } 
    public function copy() { return clone $this;}
}
 
class Test {}
 
// client
$object1 = new ConcretePrototype(new Test());
var_dump($object1);//输出：object(ConcretePrototype)#1 (1) { ["_name":"ConcretePrototype":private]=> object(Test)#2 (0) { } } 
$object2 = $object1->copy();
var_dump($object2);//输出：object(ConcretePrototype)#3 (1) { ["_name":"ConcretePrototype":private]=> object(Test)#2 (0) { } }
?>
```

### 6. Adapter(适配器模式)
这种模式允许使用不同的接口重构某个类，可以允许使用不同的调用方式进行调用：

```php
<?php
/**
 * 第一种方式：对象适配器
 */
interface Target {
    public function sampleMethod1();
    public function sampleMethod2();
}
 
class Adaptee {
    public function sampleMethod1() {
        echo '++++++++';
    }
}
 
class Adapter implements Target {
    private $_adaptee;
 
    public function __construct(Adaptee $adaptee) {
        $this->_adaptee = $adaptee;
    }
 
    public function sampleMethod1() {
        $this->_adaptee->sampleMethod1(); 
    }
 
    public function sampleMethod2() {
        echo '————————'; 
    }
}
$adapter = new Adapter(new Adaptee());
$adapter->sampleMethod1();//输出：++++++++
$adapter->sampleMethod2();//输出：————————
 
 
 
/**
 * 第二种方式：类适配器
 */
interface Target2 {
    public function sampleMethod1();
    public function sampleMethod2();
}
 
class Adaptee2 { // 源角色
    public function sampleMethod1() {echo '++++++++';}
}
 
class Adapter2 extends Adaptee2 implements Target2 { // 适配后角色
    public function sampleMethod2() {echo '————————';} 
}
 
$adapter = new Adapter2();
$adapter->sampleMethod1();//输出：++++++++
$adapter->sampleMethod2();//输出：————————
?>
```

### 7. Bridge(桥接模式)
将抽象部分与它的实现部分分离，使他们都可以独立的变抽象与它的实现分离，即抽象类和它的派生类用来实现自己的对象

```php
<?php
/**
 * 
 *实现化角色, 给出实现化角色的接口，但不给出具体的实现。
 */
abstract class Implementor { 
    abstract public function operationImp();
}
 
class ConcreteImplementorA extends Implementor { // 具体化角色A
    public function operationImp() {echo "A";}
}
 
class ConcreteImplementorB extends Implementor { // 具体化角色B
    public function operationImp() {echo "B";}
}
 
 
/**
 * 
 * 抽象化角色，抽象化给出的定义，并保存一个对实现化对象的引用
 */
abstract class Abstraction { 
    protected $imp; // 对实现化对象的引用
    public function operation() {
        $this->imp->operationImp();
    }
}
class RefinedAbstraction extends Abstraction { // 修正抽象化角色, 扩展抽象化角色，改变和修正父类对抽象化的定义。
    public function __construct(Implementor $imp) {
        $this->imp = $imp;
    }
    public function operation() { $this->imp->operationImp(); }
}
 
// client
$abstraction = new RefinedAbstraction(new ConcreteImplementorA());
$abstraction->operation();//输出:A
$abstraction = new RefinedAbstraction(new ConcreteImplementorB());
$abstraction->operation();//输出:B
?>
```

8. Composite(合成模式)
组合模式（Composite Pattern）有时候又叫做部分-整体模式，用于将对象组合成树形结构以表示“部分-整体”的层次关系。组合模式使得用户对单个对象和组合对象的使用具有一致性。

常见使用场景：如树形菜单、文件夹菜单、部门组织架构图等。

```php
<?php
/**
 * 
 *安全式合成模式
 */
interface Component {
    public function getComposite(); //返回自己的实例
    public function operation();
}
 
class Composite implements Component { // 树枝组件角色
    private $_composites;
    public function __construct() { $this->_composites = array(); }
    public function getComposite() { return $this; }
     public function operation() {
         foreach ($this->_composites as $composite) {
            $composite->operation();
        }
     }
 
    public function add(Component $component) {  //聚集管理方法 添加一个子对象
        $this->_composites[] = $component;
    }
 
    public function remove(Component $component) { // 聚集管理方法 删除一个子对象
        foreach ($this->_composites as $key => $row) {
            if ($component == $row) { unset($this->_composites[$key]); return TRUE; }
        } 
        return FALSE;
    }
 
    public function getChild() { // 聚集管理方法 返回所有的子对象
       return $this->_composites;
    }
 
}
 
class Leaf implements Component {
    private $_name; 
    public function __construct($name) { $this->_name = $name; }
    public function operation() {}
    public function getComposite() {return null;}
}
 
// client
$leaf1 = new Leaf('first');
$leaf2 = new Leaf('second');
 
$composite = new Composite();
$composite->add($leaf1);
$composite->add($leaf2);
$composite->operation();
 
$composite->remove($leaf2);
$composite->operation();
 
 
 
 
 
 
/**
 * 
 *透明式合成模式
 */
interface Component { // 抽象组件角色
    public function getComposite(); // 返回自己的实例
    public function operation(); // 示例方法
    public function add(Component $component); // 聚集管理方法,添加一个子对象
    public function remove(Component $component); // 聚集管理方法 删除一个子对象
    public function getChild(); // 聚集管理方法 返回所有的子对象
}
 
class Composite implements Component { // 树枝组件角色
    private $_composites;
    public function __construct() { $this->_composites = array(); } 
    public function getComposite() { return $this; }
    public function operation() { // 示例方法，调用各个子对象的operation方法
        foreach ($this->_composites as $composite) {
            $composite->operation();
        }
    }
    public function add(Component $component) { // 聚集管理方法 添加一个子对象
        $this->_composites[] = $component;
    }
    public function remove(Component $component) { // 聚集管理方法 删除一个子对象
        foreach ($this->_composites as $key => $row) {
            if ($component == $row) { unset($this->_composites[$key]); return TRUE; }
        } 
        return FALSE;
    }
    public function getChild() { // 聚集管理方法 返回所有的子对象
       return $this->_composites;
    }
 
}
 
class Leaf implements Component {
    private $_name;
    public function __construct($name) {$this->_name = $name;}
    public function operation() {echo $this->_name."<br>";}
    public function getComposite() { return null; }
    public function add(Component $component) { return FALSE; }
    public function remove(Component $component) { return FALSE; }
    public function getChild() { return null; }
}
 
// client 
$leaf1 = new Leaf('first');
$leaf2 = new Leaf('second');
 
$composite = new Composite();
$composite->add($leaf1);
$composite->add($leaf2);
$composite->operation();
 
$composite->remove($leaf2);
$composite->operation();
 
?>
```

### 9. Decorator(装饰器模式)
装饰器模式允许我们根据运行时不同的情景动态地为某个对象调用前后添加不同的行

```php
<?php
interface Component {
    public function operation();
}
 
abstract class Decorator implements Component{ // 装饰角色 
    protected  $_component;
    public function __construct(Component $component) {
        $this->_component = $component;
    }
    public function operation() {
        $this->_component->operation();
    }
}
 
class ConcreteDecoratorA extends Decorator { // 具体装饰类A
    public function __construct(Component $component) {
        parent::__construct($component);
    } 
    public function operation() {
        parent::operation();    //  调用装饰类的操作
        $this->addedOperationA();   //  新增加的操作
    }
    public function addedOperationA() {echo 'A加点酱油;';}
}
 
class ConcreteDecoratorB extends Decorator { // 具体装饰类B
    public function __construct(Component $component) {
        parent::__construct($component);
    } 
    public function operation() {
        parent::operation();
        $this->addedOperationB();
    }
    public function addedOperationB() {echo "B加点辣椒;";}
}
 
class ConcreteComponent implements Component{ //具体组件类
    public function operation() {} 
}
 
// clients
$component = new ConcreteComponent();
$decoratorA = new ConcreteDecoratorA($component);
$decoratorB = new ConcreteDecoratorB($decoratorA);
 
$decoratorA->operation();//输出：A加点酱油;
echo '<br>--------<br>';
$decoratorB->operation();//输出：A加点酱油;B加点辣椒;
?>
```

### 10. Facade(门面模式)
门面模式 （Facade）又称外观模式，用于为子系统中的一组接口提供一个一致的界面。门面模式定义了一个高层接口，这个接口使得子系统更加容易使用：引入门面角色之后，用户只需要直接与门面角色交互，用户与子系统之间的复杂关系由门面角色来实现，从而降低了系统的耦

```php
<?php
class Camera {
    public function turnOn() {}
    public function turnOff() {}
    public function rotate($degrees) {}
}
 
class Light {
    public function turnOn() {}
    public function turnOff() {}
    public function changeBulb() {}
}
 
class Sensor {
    public function activate() {}
    public function deactivate() {}
    public function trigger() {}
}
 
class Alarm {
    public function activate() {}
    public function deactivate() {}
    public function ring() {}
    public function stopRing() {}
}
 
class SecurityFacade {
    private $_camera1, $_camera2;
    private $_light1, $_light2, $_light3;
    private $_sensor;
    private $_alarm;
 
    public function __construct() {
        $this->_camera1 = new Camera();
        $this->_camera2 = new Camera();
 
        $this->_light1 = new Light();
        $this->_light2 = new Light();
        $this->_light3 = new Light();
 
        $this->_sensor = new Sensor();
        $this->_alarm = new Alarm();
    }
 
    public function activate() {
        $this->_camera1->turnOn();
        $this->_camera2->turnOn();
 
        $this->_light1->turnOn();
        $this->_light2->turnOn();
        $this->_light3->turnOn();
 
        $this->_sensor->activate();
        $this->_alarm->activate();
    }
 
    public  function deactivate() {
        $this->_camera1->turnOff();
        $this->_camera2->turnOff();
 
        $this->_light1->turnOff();
        $this->_light2->turnOff();
        $this->_light3->turnOff();
 
        $this->_sensor->deactivate();
        $this->_alarm->deactivate();
    }
}
 
 
//client 
$security = new SecurityFacade();
$security->activate();
?> 
```

### 11. Proxy(代理模式)
代理模式（Proxy）为其他对象提供一种代理以控制对这个对象的访问。使用代理模式创建代理对象，让代理对象控制目标对象的访问（目标对象可以是远程的对象、创建开销大的对象或需要安全控制的对象），并且可以在不改变目标对象的情况下添加一些额外的功能。

在某些情况下，一个客户不想或者不能直接引用另一个对象，而代理对象可以在客户端和目标对象之间起到中介的作用，并且可以通过代理对象去掉客户不能看到的内容和服务或者添加客户需要的额外服务。

经典例子就是网络代理，你想访问 Facebook 或者 Twitter ，如何绕过 GFW？找个代理

```php
<?
abstract class Subject { // 抽象主题角色
    abstract public function action();
}
 
class RealSubject extends Subject { // 真实主题角色
    public function __construct() {}
    public function action() {}
}
 
class ProxySubject extends Subject { // 代理主题角色
    private $_real_subject = NULL;
    public function __construct() {}
 
    public function action() {
        $this->_beforeAction();
        if (is_null($this->_real_subject)) {
            $this->_real_subject = new RealSubject();
        }
        $this->_real_subject->action();
        $this->_afterAction();
    }
 
    private function _beforeAction() {
        echo '在action前,我想干点啥....';
    }
 
    private function _afterAction() {
         echo '在action后,我还想干点啥....';
    }
}
 
// client
$subject = new ProxySubject();
$subject->action();//输出：在action前,我想干点啥....在action后,我还想干点啥....
?> 
```

### 12. Flyweight(享元模式)
运用共享技术有效的支持大量细粒度的对象

享元模式变化的是对象的存储开销

享元模式中主要角色：

抽象享元（Flyweight）角色：此角色是所有的具体享元类的超类，为这些类规定出需要实现的公共接口。那些需要外运状态的操作可以通过调用商业以参数形式传入

具体享元（ConcreteFlyweight）角色：实现Flyweight接口，并为内部状态（如果有的话）拉回存储空间。ConcreteFlyweight对象必须是可共享的。它所存储的状态必须是内部的

不共享的具体享元（UnsharedConcreteFlyweight）角色：并非所有的Flyweight子类都需要被共享。Flyweigth使共享成为可能，但它并不强制共享

享元工厂(FlyweightFactory)角色：负责创建和管理享元角色。本角色必须保证享元对象可能被系统适当地共享

客户端(Client)角色：本角色需要维护一个对所有享元对象的引用。本角色需要自行存储所有享元对象的外部状态

享元模式的优点：

Flyweight模式可以大幅度地降低内存中对象的数量

享元模式的缺点：

Flyweight模式使得系统更加复杂

Flyweight模式将享元对象的状态外部化，而读取外部状态使得运行时间稍微变长

享元模式适用场景：

当一下情况成立时使用Flyweight模式：

1 一个应用程序使用了大量的对象

2 完全由于使用大量的对象，造成很大的存储开销

3 对象的大多数状态都可变为外部状态

4 如果删除对象的外部状态，那么可以用相对较少的共享对象取代很多组对象

5 应用程序不依赖于对象标识

```php
<?php
abstract class Resources{
    public $resource=null;
 
    abstract public function operate();
}
 
class unShareFlyWeight extends Resources{
    public function __construct($resource_str) {
        $this->resource = $resource_str;
    }
 
    public function operate(){
        echo $this->resource."<br>";
    }
}
 
class shareFlyWeight extends Resources{
    private $resources = array();
 
    public function get_resource($resource_str){
        if(isset($this->resources[$resource_str])) {
            return $this->resources[$resource_str];
        }else {
            return $this->resources[$resource_str] = $resource_str;
        }
    }
 
    public function operate(){
        foreach ($this->resources as $key => $resources) {
            echo $key.":".$resources."<br>";
        }
    }
}
 
 
// client
$flyweight = new shareFlyWeight();
$flyweight->get_resource('a');
$flyweight->operate();
 
 
$flyweight->get_resource('b');
$flyweight->operate();
 
$flyweight->get_resource('c');
$flyweight->operate();
 
// 不共享的对象，单独调用
$uflyweight = new unShareFlyWeight('A');
$uflyweight->operate();
 
$uflyweight = new unShareFlyWeight('B');
$uflyweight->operate();
/* 输出：
 a:a
 a:a
 b:b
 a:a
 b:b
 c:c
 A
 B */
```

### 13. Strategy(策略模式)
策略模式主要为了让客户类能够更好地使用某些算法而不需要知道其具体的实现。
```php
<?php
interface Strategy { // 抽象策略角色，以接口实现
    public function do_method(); // 算法接口
}
 
class ConcreteStrategyA implements Strategy { // 具体策略角色A 
    public function do_method() {
        echo 'do method A';
    }
}
 
class ConcreteStrategyB implements Strategy { // 具体策略角色B 
    public function do_method() {
        echo 'do method B';
    }
}
 
class ConcreteStrategyC implements Strategy { // 具体策略角色C
    public function do_method() {
        echo 'do method C';
    }
}
 
 
class Question{ // 环境角色
    private $_strategy;
 
    public function __construct(Strategy $strategy) {
        $this->_strategy = $strategy;
    } 
    public function handle_question() {
        $this->_strategy->do_method();
    }
}
 
// client
$strategyA = new ConcreteStrategyA();
$question = new Question($strategyA);
$question->handle_question();//输出do method A
 
$strategyB = new ConcreteStrategyB();
$question = new Question($strategyB);
$question->handle_question();//输出do method B
 
$strategyC = new ConcreteStrategyC();
$question = new Question($strategyC);
$question->handle_question();//输出do method C
?>
```

### 14. TemplateMethod(模板方法模式)
模板模式准备一个抽象类，将部分逻辑以具体方法以及具体构造形式实现，然后声明一些抽象方法来迫使子类实现剩余的逻辑。不同的子类可以以不同的方式实现这些抽象方法，从而对剩余的逻辑有不同的实现。先制定一个顶级逻辑框架，而将逻辑的细节留给具体的子类去实现。

```php
<?php
abstract class AbstractClass { // 抽象模板角色
    public function templateMethod() { // 模板方法 调用基本方法组装顶层逻辑
        $this->primitiveOperation1();
        $this->primitiveOperation2();
    }
    abstract protected function primitiveOperation1(); // 基本方法
    abstract protected function primitiveOperation2();
}
 
class ConcreteClass extends AbstractClass { // 具体模板角色
    protected function primitiveOperation1() {}
    protected function primitiveOperation2(){}
 
}
 
$class = new ConcreteClass();
$class->templateMethod();
?>
```

### 15. Observer(观察者模式)
某个对象可以被设置为是可观察的，只要通过某种方式允许其他对象注册为观察者。每当被观察的对象改变时，会发送信息给观察者。
```php
<?php
 
    interface IObserver{
        function onSendMsg( $sender, $args );
        function getName();
    }
 
    interface IObservable{
        function addObserver( $observer );
    }
 
    class UserList implements IObservable{
        private $_observers = array();
 
        public function sendMsg( $name ){
            foreach( $this->_observers as $obs ){
                $obs->onSendMsg( $this, $name );
            }
        }
 
        public function addObserver( $observer ){
            $this->_observers[]= $observer;
        }
 
        public function removeObserver($observer_name) {
            foreach($this->_observers as $index => $observer) {
                if ($observer->getName() === $observer_name) {
                    array_splice($this->_observers, $index, 1);
                    return;
                }
            }
        }
    }
 
    class UserListLogger implements IObserver{
        public function onSendMsg( $sender, $args ){
            echo( "'$args' send to UserListLogger\n" );
        }
 
        public function getName(){
            return 'UserListLogger';
        }
    }
 
    class OtherObserver implements IObserver{
        public function onSendMsg( $sender, $args ){
            echo( "'$args' send to OtherObserver\n" );
        }
 
        public function getName(){
            return 'OtherObserver';
        }
    }
 
 
    $ul = new UserList();//被观察者
    $ul->addObserver( new UserListLogger() );//增加观察者
    $ul->addObserver( new OtherObserver() );//增加观察者
    $ul->sendMsg( "Jack" );//发送消息到观察者
 
    $ul->removeObserver('UserListLogger');//移除观察者
    $ul->sendMsg("hello");//发送消息到观察者
 
 /*    输出：'Jack' send to UserListLogger 'Jack' send to OtherObserver 'hello' send to OtherObserver */
?>
```

### 16. Iterator(迭代器模式)
迭代器模式 （Iterator），又叫做游标（Cursor）模式。提供一种方法访问一个容器（Container）对象中各个元素，而又不需暴露该对象的内部细节。

当你需要访问一个聚合对象，而且不管这些对象是什么都需要遍历的时候，就应该考虑使用迭代器模式。另外，当需要对聚集有多种方式遍历时，可以考虑去使用迭代器模式。迭代器模式为遍历不同的聚集结构提供如开始、下一个、是否结束、当前哪一项等统一的接口。

php标准库（SPL）中提供了迭代器接口 Iterator，要实现迭代器模式，实现该接口即可。

```php
<?php
class sample implements Iterator {
    private $_items ;
 
    public function __construct(&$data) {
        $this->_items = $data;
    }
    public function current() {
        return current($this->_items);
    }
 
    public function next() {
        next($this->_items);   
    }
 
    public function key() {
        return key($this->_items);
    }
 
    public function rewind() {
        reset($this->_items);
    }
 
    public function valid() {                                                                              
        return ($this->current() !== FALSE);
    }
}
 
// client
$data = array(1, 2, 3, 4, 5);
$sa = new sample($data);
foreach ($sa AS $key => $row) {
    echo $key, ' ', $row, '<br />';
}
/* 输出：
0 1
1 2
2 3
3 4
4 5 */
//Yii FrameWork Demo
class CMapIterator implements Iterator {
/**
* @var array the data to be iterated through
*/
    private $_d;
/**
* @var array list of keys in the map
*/
    private $_keys;
/**
* @var mixed current key
*/
    private $_key;
 
/**
* Constructor.
* @param array the data to be iterated through
*/
    public function __construct(&$data) {
        $this->_d=&$data;
        $this->_keys=array_keys($data);
    }
 
/**
* Rewinds internal array pointer.
* This method is required by the interface Iterator.
*/
    public function rewind() {                                                                                 
        $this->_key=reset($this->_keys);
    }
 
/**
* Returns the key of the current array element.
* This method is required by the interface Iterator.
* @return mixed the key of the current array element
*/
    public function key() {
        return $this->_key;
    }
 
/**
* Returns the current array element.
* This method is required by the interface Iterator.
* @return mixed the current array element
*/
    public function current() {
        return $this->_d[$this->_key];
    }
 
/**
* Moves the internal pointer to the next array element.
* This method is required by the interface Iterator.
*/
    public function next() {
        $this->_key=next($this->_keys);
    }
 
/**
* Returns whether there is an element at current position.
* This method is required by the interface Iterator.
* @return boolean
*/
    public function valid() {
        return $this->_key!==false;
    }
}
 
$data = array('s1' => 11, 's2' => 22, 's3' => 33);
$it = new CMapIterator($data);
foreach ($it as $row) {
    echo $row, '<br />';
}
 
/* 输出：
11
22
33 */
?>
```

### 17. ResponsibilityChain(责任链模式)
这种模式有另一种称呼：控制链模式。它主要由一系列对于某些命令的处理器构成，每个查询会在处理器构成的责任链中传递，在每个交汇点由处理器判断是否需要对它们进行响应与处理。每次的处理程序会在有处理器处理这些请求时暂停。
```php
<?php
 
abstract class Responsibility { // 抽象责任角色
    protected $next; // 下一个责任角色
 
    public function setNext(Responsibility $l) {
        $this->next = $l;
        return $this;
    }
    abstract public function operate(); // 操作方法
}
 
class ResponsibilityA extends Responsibility {
    public function __construct() {}
    public function operate(){
        if (false == is_null($this->next)) {
            $this->next->operate();
            echo 'Res_A start'."<br>";
        }
    }
}
 
class ResponsibilityB extends Responsibility {
    public function __construct() {}
    public function operate(){
        if (false == is_null($this->next)) {
            $this->next->operate();
            echo 'Res_B start';
        }
    }
}
 
$res_a = new ResponsibilityA();
$res_b = new ResponsibilityB();
$res_a->setNext($res_b);
$res_a->operate();//输出：Res_A start
?>
```

### 18. Command(命令模式)
命令模式：在软件系统中，“行为请求者”与“行为实现者”通常呈现一种“紧耦合”。但在某些场合，比如要对行为进行“记录、撤销/重做、事务”等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将“行为请求者”与“行为实现者”解耦？将一组行为抽象为对象，实现二者之间的松耦合。这就是命令模式。
角色分析：
抽象命令：定义命令的接口，声明执行的方法。
具体命令：命令接口实现对象，是“虚”的实现；通常会持有接收者，并调用接收者的功能来完成命令要执行的操作。
命令接收者：接收者，真正执行命令的对象。任何类都可能成为一个接收者，只要它能够实现命令要求实现的相应功能。
控制者：要求命令对象执行请求，通常会持有命令对象，可以持有很多的命令对象。这个是客户端真正触发命令并要求命令执行相应操作的地方，也就是说相当于使用命令对象的入口。
```php
      <?php
interface Command { // 命令角色
    public function execute(); // 执行方法
}
 
class ConcreteCommand implements Command { // 具体命令方法 
    private $_receiver; 
    public function __construct(Receiver $receiver) {
        $this->_receiver = $receiver;
    }
    public function execute() {
        $this->_receiver->action();
    }
}
 
class Receiver { // 接收者角色
    private $_name;
    public function __construct($name) {
        $this->_name = $name;
    }
    public function action() {
        echo 'receive some cmd:'.$this->_name;
    }
}
 
class Invoker { // 请求者角色
    private $_command; 
    public function __construct(Command $command) {
        $this->_command = $command;
    }
    public function action() {
        $this->_command->execute();
    }
}
 
$receiver = new Receiver('hello world');
$command = new ConcreteCommand($receiver);
$invoker = new Invoker($command);
$invoker->action();//输出：receive some cmd:hello world
?>
```


### 19. 备忘录模式
备忘录模式又叫做快照模式（Snapshot）或 Token 模式，备忘录模式的用意是在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样就可以在合适的时候将该对象恢复到原先保存的状态。

我们在编程的时候，经常需要保存对象的中间状态，当需要的时候，可以恢复到这个状态。比如，我们使用Eclipse进行编程时，假如编写失误（例如不小心误删除了几行代码），我们希望返回删除前的状态，便可以使用Ctrl+Z来进行返回。这时我们便可以使用备忘录模式来实现。

备忘录模式所涉及的角色有三个：备忘录(Memento)角色、发起人(Originator)角色、负责人(Caretaker)角色。

这三个角色的职责分别是：

发起人：记录当前时刻的内部状态，负责定义哪些属于备份范围的状态，负责创建和恢复备忘录数据。
备忘录：负责存储发起人对象的内部状态，在需要的时候提供发起人需要的内部状态。
管理角色：对备忘录进行管理，保存和提供备忘录。
```php
<?php
 
class Originator { // 发起人(Originator)角色
    private $_state;
    public function __construct() {
        $this->_state = '';
    }
    public function createMemento() { // 创建备忘录
        return new Memento($this->_state);
    }
    public function restoreMemento(Memento $memento) { // 将发起人恢复到备忘录对象记录的状态上
        $this->_state = $memento->getState();
    }
    public function setState($state) { $this->_state = $state; } 
    public function getState() { return $this->_state; }
    public function showState() {
        echo $this->_state;echo "<br>";
    }
 
}
 
class Memento { // 备忘录(Memento)角色 
    private $_state;
    public function __construct($state) {
        $this->setState($state);
    }
    public function getState() { return $this->_state; } 
    public function setState($state) { $this->_state = $state;}
}
 
class Caretaker { // 负责人(Caretaker)角色 
    private $_memento;
    public function getMemento() { return $this->_memento; } 
    public function setMemento(Memento $memento) { $this->_memento = $memento; }
}
 
// client
/* 创建目标对象 */
$org = new Originator();
$org->setState('open');
$org->showState();
 
/* 创建备忘 */
$memento = $org->createMemento();
 
/* 通过Caretaker保存此备忘 */
$caretaker = new Caretaker();
$caretaker->setMemento($memento);
 
/* 改变目标对象的状态 */
$org->setState('close');
$org->showState();
 
$org->restoreMemento($memento);
$org->showState();
 
/* 改变目标对象的状态 */
$org->setState('close');
$org->showState();
 
/* 还原操作 */
$org->restoreMemento($caretaker->getMemento());
$org->showState();
/* 输出：
open
close
open
close
open */
?>
```

### 20. State(状态模式)
状态模式当一个对象的内在状态改变时允许改变其行为，这个对象看起来像是改变了其类。状态模式主要解决的是当控制一个对象状态的条件表达式过于复杂时的情况。把状态的判断逻辑转移到表示不同状态的一系列类中，可以把复杂的判断逻辑简化。

角色：
上下文环境（Work）：它定义了客户程序需要的接口并维护一个具体状态角色的实例，将与状态相关的操作委托给当前的具体对象来处理。
抽象状态（State）：定义一个接口以封装使用上下文环境的的一个特定状态相关的行为。
具体状态（AmState）：实现抽象状态定义的接口。

```php
         <?php
interface State { // 抽象状态角色
    public function handle(Context $context); // 方法示例
}
 
class ConcreteStateA implements State { // 具体状态角色A
    private static $_instance = null;
    private function __construct() {}
    public static function getInstance() { // 静态工厂方法，返还此类的唯一实例
        if (is_null(self::$_instance)) {
            self::$_instance = new ConcreteStateA();
        }
        return self::$_instance;
    }
 
    public function handle(Context $context) {
        echo 'concrete_a'."<br>";
        $context->setState(ConcreteStateB::getInstance());
    }
 
}
 
class ConcreteStateB implements State { // 具体状态角色B
    private static $_instance = null;
    private function __construct() {}
    public static function getInstance() {
        if (is_null(self::$_instance)) {
            self::$_instance = new ConcreteStateB();
        }
        return self::$_instance;
    }
 
    public function handle(Context $context) {
        echo 'concrete_b'."<br>";
        $context->setState(ConcreteStateA::getInstance());
    }
}
 
class Context { // 环境角色 
    private $_state;
    public function __construct() { // 默认为stateA
        $this->_state = ConcreteStateA::getInstance();
    }
    public function setState(State $state) {
        $this->_state = $state;
    }
    public function request() {
        $this->_state->handle($this);
    }
}
 
// client
$context = new Context();
$context->request();
$context->request();
$context->request();
$context->request();
/* 输出：
concrete_a
concrete_b
concrete_a
concrete_b */
?>
```

### 21. Visitor(访问者模式)
访问者模式是一种行为型模式，访问者表示一个作用于某对象结构中各元素的操作。它可以在不修改各元素类的前提下定义作用于这些元素的新操作，即动态的增加具体访问者角色。

访问者模式利用了双重分派。先将访问者传入元素对象的Accept方法中，然后元素对象再将自己传入访问者，之后访问者执行元素的相应方法。

主要角色

抽象访问者角色(Visitor)：为该对象结构(ObjectStructure)中的每一个具体元素提供一个访问操作接口。该操作接口的名字和参数标识了 要访问的具体元素角色。这样访问者就可以通过该元素角色的特定接口直接访问它。
具体访问者角色(ConcreteVisitor)：实现抽象访问者角色接口中针对各个具体元素角色声明的操作。
抽象节点（Node）角色：该接口定义一个accept操作接受具体的访问者。
具体节点（Node）角色：实现抽象节点角色中的accept操作。
对象结构角色(ObjectStructure)：这是使用访问者模式必备的角色。它要具备以下特征：能枚举它的元素；可以提供一个高层的接口以允许该访问者访问它的元素；可以是一个复合（组合模式）或是一个集合，如一个列表或一个无序集合(在PHP中我们使用数组代替，因为PHP中的数组本来就是一个可以放置任何类型数据的集合)
适用性

访问者模式多用在聚集类型多样的情况下。在普通的形式下必须判断每个元素是属于什么类型然后进行相应的操作，从而诞生出冗长的条件转移语句。而访问者模式则可以比较好的解决这个问题。对每个元素统一调用element−>accept(vistor)即可。
访问者模式多用于被访问的类结构比较稳定的情况下，即不会随便添加子类。访问者模式允许被访问结构添加新的方法。

```php
<?php
interface Visitor { // 抽象访问者角色
    public function visitConcreteElementA(ConcreteElementA $elementA);
    public function visitConcreteElementB(concreteElementB $elementB);
}
 
interface Element { // 抽象节点角色
    public function accept(Visitor $visitor);
}
 
class ConcreteVisitor1 implements Visitor { // 具体的访问者1
    public function visitConcreteElementA(ConcreteElementA $elementA) {}
    public function visitConcreteElementB(ConcreteElementB $elementB) {}
}
 
class ConcreteVisitor2 implements Visitor { // 具体的访问者2
    public function visitConcreteElementA(ConcreteElementA $elementA) {}
    public function visitConcreteElementB(ConcreteElementB $elementB) {}
}
 
class ConcreteElementA implements Element { // 具体元素A
    private $_name;
    public function __construct($name) { $this->_name = $name; } 
    public function getName() { return $this->_name; }
    public function accept(Visitor $visitor) { // 接受访问者调用它针对该元素的新方法
        $visitor->visitConcreteElementA($this);
    }
}
 
class ConcreteElementB implements Element { // 具体元素B
    private $_name; 
    public function __construct($name) { $this->_name = $name;}
    public function getName() { return $this->_name; }
    public function accept(Visitor $visitor) { // 接受访问者调用它针对该元素的新方法
        $visitor->visitConcreteElementB($this);
    }
}
 
class ObjectStructure { // 对象结构 即元素的集合
    private $_collection; 
    public function __construct() { $this->_collection = array(); } 
    public function attach(Element $element) {
        return array_push($this->_collection, $element);
    }
    public function detach(Element $element) {
        $index = array_search($element, $this->_collection);
        if ($index !== FALSE) {
            unset($this->_collection[$index]);
        }
        return $index;
    }
    public function accept(Visitor $visitor) {
        foreach ($this->_collection as $element) {
            $element->accept($visitor);
        }
    }
}
 
// client
$elementA = new ConcreteElementA("ElementA");
$elementB = new ConcreteElementB("ElementB");
$elementA2 = new ConcreteElementB("ElementA2");
$visitor1 = new ConcreteVisitor1();
$visitor2 = new ConcreteVisitor2();
 
$os = new ObjectStructure();
$os->attach($elementA);
$os->attach($elementB);
$os->attach($elementA2);
$os->detach($elementA);
$os->accept($visitor1);
$os->accept($visitor2);
?>
```

### 22. Mediator(中介者模式)
中介者模式用于开发一个对象，这个对象能够在类似对象相互之间不直接相互的情况下传送或者调解对这些对象的集合的修改。 一般处理具有类似属性，需要保持同步的非耦合对象时，最佳的做法就是中介者模式。PHP中不是特别常用的设计模式。

```php
<?php
abstract class Mediator { // 中介者角色
    abstract public function send($message,$colleague); 
} 
 
abstract class Colleague { // 抽象对象
    private $_mediator = null; 
    public function __construct($mediator) { 
        $this->_mediator = $mediator; 
    } 
    public function send($message) { 
        $this->_mediator->send($message,$this); 
    } 
    abstract public function notify($message); 
} 
 
class ConcreteMediator extends Mediator { // 具体中介者角色
    private $_colleague1 = null; 
    private $_colleague2 = null; 
    public function send($message,$colleague) {
        //echo $colleague->notify($message);
        if($colleague == $this->_colleague1) { 
            $this->_colleague1->notify($message); 
        } else { 
            $this->_colleague2->notify($message); 
        } 
    }
    public function set($colleague1,$colleague2) { 
        $this->_colleague1 = $colleague1; 
        $this->_colleague2 = $colleague2; 
    } 
} 
 
class Colleague1 extends Colleague { // 具体对象角色
    public function notify($message) {
        echo 'colleague1：'.$message."<br>";
    } 
} 
 
class Colleague2 extends Colleague { // 具体对象角色
    public function notify($message) { 
        echo 'colleague2：'.$message."<br>";
    } 
} 
 
// client
$objMediator = new ConcreteMediator(); 
$objC1 = new Colleague1($objMediator); 
$objC2 = new Colleague2($objMediator); 
$objMediator->set($objC1,$objC2); 
$objC1->send("to c2 from c1"); //输出：colleague1：to c2 from c1
$objC2->send("to c1 from c2"); //输出：colleague2：to c1 from c2
?> 
```

### 23. Interpreter(解释器模式)
给定一个语言, 定义它的文法的一种表示，并定义一个解释器，该解释器使用该表示来解释语言中的句子。
角色：
环境角色(PlayContent)：定义解释规则的全局信息。
抽象解释器(Empress)：定义了部分解释具体实现，封装了一些由具体解释器实现的接口。
具体解释器(MusicNote)：实现抽象解释器的接口，进行具体的解释执行。

```php
<?php
class Expression { //抽象表示
    function interpreter($str) { 
        return $str; 
    } 
} 
 
class ExpressionNum extends Expression { //表示数字
    function interpreter($str) { 
        switch($str) { 
            case "0": return "零"; 
            case "1": return "一"; 
            case "2": return "二"; 
            case "3": return "三"; 
            case "4": return "四"; 
            case "5": return "五"; 
            case "6": return "六"; 
            case "7": return "七"; 
            case "8": return "八"; 
            case "9": return "九"; 
        } 
    } 
} 
 
class ExpressionCharater extends Expression { //表示字符
    function interpreter($str) { 
        return strtoupper($str); 
    } 
} 
 
class Interpreter { //解释器
    function execute($string) { 
        $expression = null; 
        for($i = 0;$i<strlen($string);$i++) { 
            $temp = $string[$i]; 
            switch(true) { 
                case is_numeric($temp): $expression = new ExpressionNum(); break; 
                default: $expression = new ExpressionCharater(); 
            } 
            echo $expression->interpreter($temp);
            echo "<br>"; 
        } 
    } 
} 
 
//client
$obj = new Interpreter(); 
$obj->execute("123s45abc"); 
/* 输出：
一
二
三
S
四
五
A
B
C */
?>
```

参考文章： [PHP 中最全的设计模式（23种）](https://blog.csdn.net/qq_29920751/article/details/87371445)