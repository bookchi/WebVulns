## PHP弱类型

[toc]

问题在于：

- 变量是弱类型，`==`比较不同类型变量时，存在类型转换，而转换的结果可能和我们想象的并不一致

- 内置函数对传入参数的松散处理。

### 比较时的自动转换

PHP在使用`==`比较不通类型变量时，会存在变量转换。

（关键是这里哪里涉及到比较了？？？

```php
$a = null, $b = false;	// true
$a = '', $b = null;			// false
```

```php
// intval()会将前面的，数字转换成数字
intval('abcd') == 0;
intval('8as891') == 8;

// hex转换
"0x1e240"=="123456";     # true
"0x1e240"== 123456;      # true
"0x1e240"=="1e240";      # false

// hash比较 0e\d+--科学计数法
"0e132456789" == "0e7124511451155"; # true		因为都是0啊
"0e123456abc" == "0e1dddada";       # false
"0e1abc" == "0" ;                   # true
  
```

常见的类型转换：int <-> string。

int->string

```php
$var = 5;

$str = (string)$var;
$str = strval($var);
```

String->int

```php
var_dump(intval('2'));			// 2
var_dump(intval('3abcd'));	// 3
var_dump(intval('abcd'));		// 0
```

intval()转换的时候，会从字符串开始直到遇到一个非数字的字符，然后返回这之间的数字；如果没有数字，就返回0.

```php
if(intval($a)>1000) {
    mysql_query("select * from news where id=".$a)
}
// $a的值有可能是1002 union…..
```

### 内置函数的松散性

就是函数期待接受某个类型的值，但是你偏不按要求传递。

#### md5()

```php
string md5 ( string $str [, bool $raw_output = false ] )
```

函数需要一个string类型作为参数，但是如果你传递一个array时，它就无法正确的求出array的md5值，就会导致任意2个array的md5值都相等。也可以使得求出的md5 值为0e\d+ 形式

```php
$array1[] = array(
    "foo" => "bar",
    "bar" => "foo",
);
$array2 = array("foo", "bar", "hello", "world");
var_dump(md5($array1)==var_dump($array2));       //true

md5('240610708') == md5('QNKCDZO')      # true
```

#### strcmp()

```php
int strcmp ( string $str1 , string $str2 ),
```

函数需要两个string作为参数，如果str1小于str2,返回-1，相等返回0，否则返回1。它比较的本质是将两个变量转换为ascii，然后进行减法运算，然后根据运算结果来决定返回值。

但当传入非字符串类型的数据的时，函数将发生错误**并返回0**。

```php
$array=array(1,2,3);
var_dump(strcmp($array,'123')); //null,在某种意义上null也就是相当false。也就是相等--0
```

#### switch

如果是数字类型的case判断，switch会将其中的参数转换为int类型。

```php
$i = "2abc";
switch($i){
    case 0:
    case 1:
    case 2:
        echo "i is less than 3 but not negative";
        break;
    case 3:
        echo "i is 3";
}
```

程序会输出`i is less than 3 but not negative`

#### In_array()

```php
bool in_array ( mixed $needle , array $haystack [, bool $strict = FALSE ] )
```

如果strict参数没有提供，那么`in_array`就会使用松散比较来判断 $needle 是否在 $haystack 中。

当 strict 的值为 true 时，`in_array()` 会比较 ​needle 中的类型和 ​haystack 中的类型是否相同。

```php
$array = [0, 1, 2, '3'];
in_array('abc', $array);
in_array('1bc', $array);
```

两个都是true，因为不比较类型，并且会进行自动的类型转换(相当于==)，'abc'->0, '1bc'->1，所以都返回`true`

### 实战

#### 题目1

```php
<?php
show_source(__FILE__);
$flag = "xxxx";
if(isset($_GET['time'])){ 
        if(!is_numeric($_GET['time'])){ 
                echo 'The time must be number.'; 
        }else if($_GET['time'] < 60 * 60 * 24 * 30 * 2){ 
                        echo 'This time is too short.'; 
        }else if($_GET['time'] > 60 * 60 * 24 * 30 * 3){ 
                        echo 'This time is too long.'; 
        }else{ 
                sleep((int)$_GET['time']); 
                echo $flag; 
        } 
                echo '<hr>'; 
}
?>
```

这个逻辑很明显，首先判断输入是不是数字，然后如果不在(60 * 60 * 24 * 30 * 2, 60 * 60 * 24 * 30 * 3)之间的，就sleep输入的时间，然后显示flag。

问题就在于，这样要等待很长时间，这显然是不现实的。

我们输入的GET参数肯定是作为字符串保存了，`is_numeric()`支持普通数字型字符串、科学记数法型字符串、部分支持十六进制0x型字符串，但强制类型转换int，却不能正确的转换十六进制型字符串、科学计数法型字符串（部分）。

所以上面我们只要输入科学计数法形式或者16进制形式的字符串就可以绕过漫长的sleep了。`0x5b8d80`/`6e6`

比如：

```php
<?php
echo (int)"0x8888"; //输出为0
echo (int)"8e22"; //输出为8
?>
```

> **划重点**：
>
> `is_numeric()`支持：一般string、科学技术发string、hex string
>
> `(int)`支持：普通字符串



这题涉及到了int强制类型转换。有个web题有个小点大概如下：

```php
if (intval($password) < 2333 && intval($password + 1) > 2333){
    echo $flag;
}
```

通过以上分析，很容易可知，可以用科学计数法表示password，绕过限制。

### Reference

- [关于php弱类型的小总结](https://saucer-man.com/information_security/262.html)（有一些实际代码
- [PHP弱类型简介](https://www.hacking8.com/MiscSecNotes/php/type-question.html)
- [php弱类型安全](https://www.zzzsdust.com/articles/22/)

