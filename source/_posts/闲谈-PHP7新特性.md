---
title: 闲谈 PHP7新特性
date: 2017-05-27 21:09:12
tags:
     - PHP
     - PHP7
---
+  PHP7已经发布有段时间了，其最大的特点是快。其性能高于 HHVM， 是 PHP5.6的两倍。本人就整理一些PHP7.0.x的新特性，也当做自己学习和复习PHP7。
 <!-- more -->
### 0、可变参数
####  PHP7允许传入可变个数的同类型参数
```php
<?php
 
function  sum(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sum(4, 5, 6, 7));
```
#### 输出

```php
int(22)
```

### 1、标量类型声明
#### 标量类型声明 有两种模式: 强制 (默认) 和 严格模式。 现在可以使用下列类型参数（无论用强制模式还是严格模式）： 字符串(string), 整数 (int), 浮点数 (float), 以及布尔值 (bool)。它们扩充了PHP5中引入的其他类型：类名，接口，数组和 回调类型。

```php
<?php
// 强制模式
function  sum(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sum(4, '5', 6.1));
```

#### 输出
```php
int(15)
```
#### 要使用严格模式需要在文件顶部添加 declare(strict_types=1);在严格模式中，只有一个与类型声明完全相符的变量才会被接受，否则将会抛出一个TypeError。  严格类型仅用于标量类型声明，也正是因为如此，这需要PHP 7.0.0 或更新版本，因为标量类型声明也是在那个版本中添加的。
例子
```php
<?php
declare(strict_types=1);

function sum(int $m, int $n) {
    return $m + $n;
}

var_dump(sum(0, 1));
var_dump(sum(0.5, 1.5));
?>
```
#### 输出
```php
int(2)

Fatal error: Uncaught TypeError: Argument 1 passed to sum() must be of the type integer, float given,  
Stack trace:
#0 -(9): sum(0.5, 1.5)
 
```

### 3、函数返回类型声明
#### PHP 7 增加了对返回类型声明的支持。函数返回值的类型必须与声明的类型一致， 否则会抛出错误。
```php
<?php

function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
```
#### 输出
```php
Array
(
    [0] => 6
    [1] => 15
    [2] => 24
)

#### 由于近期有学习 Swift，这种写法和 Swift 类似：
在 Swift 中
```php
func test(a: String) -> String { 
    let b = a + "b"
    return b
} 

```

### 3、null合并运算符
#### PHP7中添加了null合并运算符 (??) 这个语法糖。如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。

```php
<?php
// Fetches the value of $_GET['user'] and returns 'nobody'
// if it does not exist.
$username = $_GET['user'] ?? 'nobody';
// This is equivalent to:
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';

// Coalesces can be chained: this will return the first
// defined value out of $_GET['user'], $_POST['user'], and
// 'nobody'.
$username = $_GET['user'] ?? $_POST['user'] ?? 'nobody';
?>
```

### 4、太空船操作符（组合比较符） 
#### 太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。
```php
<?php
// 整数
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// 浮点数
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
?>
```
### 5、通过 define() 定义常量数组
#### PHP7中Array 类型的常量现在可以通过 define() 来定义。在 PHP5.6 中仅能通过 const 定义。
```php
<?php
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);

echo ANIMALS[1]; // 输出 "cat"
?>
```
### 5、匿名类
#### PHP7支持通过new class 来实例化一个匿名类，这可以用来替代一些“用后即焚”的完整类定义。
```php
<?php
interface Logger {
    public function log(string $msg);
}

class Application {
    private $logger;

    public function getLogger(): Logger {
         return $this->logger;
    }

    public function setLogger(Logger $logger) {
         $this->logger = $logger;
    }
}

$app = new Application;
$app->setLogger(new class implements Logger {
    public function log(string $msg) {
        echo $msg;
    }
});

var_dump($app->getLogger());
?>
```

### 6、use 批量声明
#### PHP 7 中 use 可以在一句话中声明多个类或函数或 const 了：

```php
<php 
use some/namespace/{ClassA, ClassB, ClassC as C}; 
use function some/namespace/{fn_a, fn_b, fn_c}; 
use const some/namespace/{ConstA, ConstB, ConstC}; 
```

#### 但还是要写出每个类或函数或 const 的名称（并没有像 python 一样的 from some import * 的方法）。需要留意的问题是：如果你使用的是基于 composer 和 PSR-4 的框架，这种写法是否能成功的加载类文件？其实是可以的，composer 注册的自动加载方法是在类被调用的时候根据类的命名空间去查找位置，这种写法对其没有影响。

### 7、其他特性
#### 其他的特性包括但不限于

1、Int64支持，统一不同平台下的整型长度，字符串和文件上传都支持大于2GB。
2、统一变量语法（Uniform variable syntax）。
3、Unicode字符格式支持（\u{xxxxx}）
4、移除了一些老的不在支持的SAPI（服务器端应用编程端口）和扩展

#### 有兴趣直接访问官网 [http://php.net/manual/zh/migration70.new-features.php](http://php.net/manual/zh/migration70.new-features.php)





