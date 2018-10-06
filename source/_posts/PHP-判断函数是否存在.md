---
title: PHP 判断函数是否存在
date: 2018-10-06 18:59:10
tags: 
    - PHP
---

PHP 判断函数是否存在有三个函数
 + function_exists 检查给定的函数是否被定义
 + method_exists 检查类的方法是否存在
 + is_callable 检测参数是否为合法的可调用结构
 
 虽然这三个函数的名字和定义都有差别，但是用起来非常的相似，在部分情况下他们的使用方法完全一致，所以容易混淆。
 
 下面了解一下每种方法的定义和行为。

## function_exists

function_exists 比较简单，只接收一个 string 类型的参数作为要查找的方法名。

它会去系统函数表和用户自定义函数表查找，如果找到相应名字的就会返回 true。**不过这里要注意，PHP 的编译选项可以禁用函数，但是这些函数名是存在于系统函数表的，所以可能会导致 function_exists 会返回 true 但无法调用的情况**。

一定要注意 PHP 的关键字并不是函数，例如 `echo` ，function_exists('echo') 降会返回 bool(false)。

## method_exists

method_exists 使用的对象是类，它用来检查类里的方法是否存在。所以它有两个参数 `mixed $object`，`string $method_name`，$object 是所检查的类，$method_name 是检查的方法名。

$object 可以是 object 类型，或者 string 类型，但是要注意，**如果使用了 string 类型参数，并且此类当前无定义，则会触发 PHP 的自动加载机制**。

method_exists 并不会在意类中方法的访问修饰符，所以即使在类外部无法调用的 private 方法也会返回 bool(true)。因此 method_exists 也不代表当前可以直接调用此方法。另一个检查类中属性的方法 `property_​exists` 则会检查访问修饰符。

method_exists 也不会处理魔术方法 __call()。

## is_callable

is_callable 应该是这三个函数中最复杂的一个。

它有三个参数，`mixed $var`，`bool $syntax_only`，`string &$callable_name`。

第一个参数 $var 是待检查的变量，类型可以是 string 和 array，如果是 array 则代表传入的是类和方法名，分别是数组坐标的0和1，数组里的两个参数和 `method_exists` 方法的两个参数含义一样，**注意这里也可能会触发自动加载**。

第二个参数 $syntax_only 是指的是否只检查是方法或者函数，而不检查是否能调用(这样考虑作用域、访问修饰符等)。默认为 false。

第三个参数 &$callable_name 要注意它是一个引用，is_callable 会给它赋一个这次检查变量的 `callable name`(不知道如何解释)，**注意，即使 is_callable 返回的是 false 这个变量也会被赋值**。它最重要的作用就是给方法定位，返回的值不能直接用来调用，都是静态方法的格式，但是只用来说明归属，如果是函数，则直接是函数名。请看如下代码
``` PHP
<?php
class B
{
    public function bb()
    {
        echo 222;
    }
}

class A extends B
{
    public function aa() {
        echo 111;
    }
}

$a = new A();
$b = new B();

var_dump(is_callable([$a,'bb'], false, $aName));
var_dump(is_callable([$b,'bb'], false, $bName));

var_dump($aname);
var_dump($bname);

/* return value
bool(true)
bool(true)
string(5) "A::bb"
string(5) "B::bb"
*/
```
其中 A 和 B 都是 instanceof B。可以看出差别来。不过此参数一般不使用。

is_callable 与 method_exists 不同，它是会参考类中的魔术方法 __call() 的。

## 使用建议

1. 如果检查是为了接下来正常调用的话，建议只使用 is_callable，函数如其名，它就是用来判断是否可以正常调用。参数使用默认的 $syntax_only = false (这代表检查作用域和访问修饰符等)。第三个参数一般不使用。

2. method_exists 的第一个参数和 is_callable 第一个参数是数组时的第一个变量（也就是要检查的类/对象），都尽量不要使用字符串，应该使用对象，因为类不存在的话会触发自动加载，耗时增加。

3. function_exists 和 method_exists 都会比 is_callable 快，但是缺点很明显，再明确清楚这些缺点不会带来问题的时候再使用这两个方法。

4. is_callable 第二个参数为 true 的时候和 function_exists 和 method_exists 基本一样，只差一个 __call()，但是函数名称的语义就没有用了，一定要确定和你一起开发的人也会这么使用。