# Java基础-1

- [Java基础-1](#java基础-1)
  - [1 quickstart](#1-quickstart)
  - [2 基本语法](#2-基本语法)
  - [3 java基本类型](#3-java基本类型)
  - [4 变量类型](#4-变量类型)
  - [5 修饰符](#5-修饰符)
  - [6 运算符](#6-运算符)
  - [7 循环](#7-循环)
  - [8 条件语句](#8-条件语句)
- [Java基础-2](#java基础-2)
  - [1 Number&amp;Math类](#1-numbermath类)
  - [2 Character类](#2-character类)
  - [2 String类](#2-string类)
  - [3 StringBuidler和StringBuffer类](#3-stringbuidler和stringbuffer类)
  - [4 数组](#4-数组)
  - [5 Stream、File和IO](#5-streamfile和io)
  - [6 Scanner类](#6-scanner类)
  - [7 异常处理](#7-异常处理)

## 1 quickstart

先来一段简单的程序（文件名和类名一致

```java
public class HelloWorld {
  public static void main(String[] args){
    System.out.println("Hello World!")
  }
}
```

然后通过命令行运行：

```shell
$ javac HelloWorld.java 
$ java HelloWorld      
Hello World!
```

Javac：将java源文件编译为class字节码。若没有错误，就会出现`HelloWorld.class`文件。

java后面跟的就是类名`HelloWorld`

## 2 基本语法

一个java程序可以认为是一系列对象的集合，而这些对象通过调用彼此的方法来协同工作。涉及到了类、对象、方法和实例变量等概念。

**java的一些基本注意点：**

- 大小写敏感

- 类名：首字母大写，驼峰式命名

- 方法名：首字母小写，驼峰式命名

- 源文件名：和类名相同（如果文件名和类名不相同则会导致编译错误)

- 主方法入口：所有的程序由`public static void main(String[] args)`开始执行

  > 不遵守这些会如何

**标识符规范：**age、$salary、_value、__1_value

- 以字母、$、or_开头

- 首字符之后的字母，可以是字母、$、_、数字

- 关键字不可作为标识符

  > $、_开头的标识符，有什么特殊含义吗

**修饰符：**

> go只有导出和未导出的概念；没有修饰符，只通过首字母大小写区分

- 访问控制修饰符：default, public, protected, private
- 非访问控制修饰符：final, abstract, static, synchronized

**变量：**

- 局部变量

- 类变量(静态变量)

- 成员变量(非静态变量)

  > 没有全局变量吗？

**继承：**

java也有继承的概念。

**接口：**

java重的接口，可以理解为对象间相互通信的协议。接口再继承中是很重要的角色。

接口只定义要用到的方法， 但是方法的具体实现取决于派生类。

**Java源程序与编译型程序运行区别：**

<img src="https://www.runoob.com/wp-content/uploads/2013/12/ZSSDMld.png" width=60%>

## 3 java基本类型

java的两大数据类型：

- 内置数据类型
- 引用数据类型

**内置数据类型**：

java提供了八种基本类型。六种数字类型，一种字符类型，一种布尔类型。

- 数字类型：byte, short, int, long, float, double
- 字符类型：char，String
- 布尔类型：boolean

除此之外，java中还存在另一种基本类型void，它也有对应的包装类`java.lang.Void`，不过我们无法对它们直接进行操作。

**引用类型**：

- 对象、数组
- 引用类型默认值为null

**Java常量**：

java中使用`final`关键字来修饰常量：`finale double PI = 3.14`

常量名一般是大写，便于识别。

byte、int、long、和short都可以用十进制、16进制以及8进制的方式来表示。

```java
int decimal = 100;
int octal = 0144;
int hexa =  0x64;
```

**自动类型转换**

整型、常量、字符类型数据可以混合运算。**运算中，不同类型的数据先转化为同一类型，然后进行运算。**

转换从低级到高级。

```java
// 低  ------------------------------------>  高

byte,short,char—> int —> long—> float —> double -> String
```

转换规则如下：

- boolen类型不可进行类型转换
- 不可把object类型转换为不相关类的对象
- 高->低转换时，必须使用强制类型转换
- 转换过程中可能会溢出or损失精度
- 浮点数->整数，是直接丢弃小数位

> java会不会也有弱类型问题

**隐含强制类型转换**

- \1. 整数的默认类型是 int。
- \2. 浮点型不存在这种情况，因为在定义 float 类型时必须在数字后面跟上 F 或者 f。

## 4 变量类型

Java支持的变量类型有：

- 类变量：独立于方法之外的，有static修饰
- 实例变量：独立于方法之外，无static修饰
- 局部变量：method中的变量

```java
public class Variable{
    static int allClicks=0;    // 类变量
 
    String str="hello world";  // 实例变量
 
    public void method(){
 
        int i =0;  // 局部变量
 
    }
}
```

**局部变量**：

- 声明在method或语句块中
- 局部变量在method或语句快执行是被创建，之后被销毁
- 访问修饰符不可用于局部变量
- 在栈上分配
- 无默认值，如无初始化，就是内存中的垃圾值

**实例变量**：

- 声明在类中，method之外
- 随对象的创建/销毁而创建/销毁
- 访问修饰符可访问

**类变量**：

- method之外，static修饰
- 无论一个类创建了多少个对象，类只拥有类变量的一份拷贝
- 静态变量储存在静态存储区。经常被声明为常量，很少单独使用static声明变量
- 静态变量可以通过：*ClassName.VariableName*的方式访问

## 5 修饰符

修饰符有两类：

- 访问修饰符
- 非访问修饰符

修饰符用来定义类、方法或者变量，通常放在语句的最前端。

**访问修饰符**：

用来控制类、变量、方法哥构造方法的可见性。有四种权限：

- default(什么都不写，默认)：在统一package内可见；可修饰：类、接口、变量】方法
- private：仅在一个类中可见；不可修饰：类（外部类）
- public：对所有类可见； 可修饰：类、接口、变量、方法
- protected：同一package内的类和所有子类可见；不可修饰：类（外部类）

**非访问修饰符**：

- static
- final
- abstract：用来创建抽象类和抽象方法
- synchronized和volatile：主要用于多线程

## 6 运算符

算数运算符：`+, -, *, /, %, ++, --`

关系运算符：`==, !=, >, <, >=, >=`

位运算符：`&, |, ^, ~, <<, >>, >>>`

逻辑运算符：`&&, ||, !`

赋值运算符：`=, +=, -=, *=, /=, %=, <<=, >>=, &=, ^=, |=`

条件运算符：`?:`

`instanceof`运算符：用来检查对象是否是一个特定的类型。

```java
String name = "James";
boolen resutl = name instanceof String; // 由于 name 是 String 类型，所以返回true
```

## 7 循环

Java的循环有for循环，while循环和do...while循环。

**while循环**

```java
while(bool表达式){
  
}
```

**do..while**循环

```java
// 即使不满足条件，也至少执行一次
do{
  
}while(bool表达式)
```

**for循环**

```java
// 很C
for (int x = 10; x < 20; x++){
  
}
```

**增强for循环**

`for(声明语句 : 表达式)` 

表达式：数组

```java
String[] names = {"James", "Larry", "Tom", "Lacy"}
for(String name : names){
  
}

int [] numbers = {10, 20, 30, 40, 50};
for(int x : numbers ){
  
}
```

而对于Go语言，类似的有：

```go
for k, v := range pkgs{
  
}
```

**break、continue关键字**

主要用在循环语句或switch语句中，用来调出整个语句块。

用于循环语句中，后面的语句不执行。作用是让程序立刻跳转到下一次循环的迭代。

## 8 条件语句

if-else

```java
if(x < 20){
  // ...
}else{
  // ...
}
```

If-else if-else

```java
if(x = 10){
  // ...
}else if(x == 20){
  // ...
}else{
  // ...
}
```

switch语句

```java
switch(expression){
  case value:
    // ...
    break; // or not
  case value:
    // ...
    break;
  default:
    // ...
}
```

不同于java，go的switch语句的expression支持初始化，case可以是expression，默认无需break。
# Java基础-2

## 1 Number&Math类

> 对这些类只是简单介绍一下概念，具体的方法还要去看官方文档or直接在IDE里看

一般情况下，要使用数字是，我们都会用内置的数据类型，比如：byte, int, long, double等。

然而在实际开发过程中，我们经常会需要使用对象，而非内置数据类型，因而有了包装类。

> 虽然不知道为什么开发过程中会需要使用对象而非内置数据类型

所有的包装类**（Integer、Long、Byte、Double、Float、Short）**都是抽象类 Number 的子类。

<img src="https://www.runoob.com/wp-content/uploads/2013/12/number1.png" width=60%>

这种由编译器特别支持的包装称为**装箱**，对象拆箱成为**拆箱**。

```java
public static void main(String[] args) {
  			Integer x = 5;	// 装箱
        x += 10;				// 拆箱
        System.out.println(x);
    }
```

**Math类**

Java 的 Math 包含了用于执行基本数学运算的属性和方法，如初等指数、对数、平方根和三角函数。

Math 的方法都被定义为 static 形式，通过 Math 类可以在主函数中直接调用。

```java
public class Test {  
    public static void main (String []args)  
    {  
        System.out.println("90 度的正弦值：" + Math.sin(Math.PI/2));  
        System.out.println("0度的余弦值：" + Math.cos(0));  
        System.out.println("60度的正切值：" + Math.tan(Math.PI/3));  
        System.out.println("1的反正切值： " + Math.atan(1));  
        System.out.println("π/2的角度值：" + Math.toDegrees(Math.PI/2));  
        System.out.println(Math.PI);  
    }  
}
```

## 2 Character类

Character 类用于对单个字符进行操作。

Character 类在对象中包装一个基本类型 **char** 的值。

Character类提供了一系列方法来操纵字符。

使用Character的构造方法创建一个Character类对象，例如：`Character ch = new Character('a');`

在某些情况下，Java编译器会自动创建一个Character对象：

```java
// 原始字符 'a' 装箱到 Character 对象 ch 中
Character ch = 'a';

// 原始字符 'x' 用 test 方法装箱
// 返回拆箱的值到 'c
char c = test('x')
```

## 2 String类

字符串操作的重要性无需多说，直接来看操作。

**创建String字符串**

```java
// 直接声明
String greeting = "hello";

// 通过构造函数
public class StringDemo{
  public static void main(String[] args){
    char[] helloArray = {'h', 'e', 'l', 'l', 'o'};
    String hellString = new String(helloArray);
  }
}
```

**注意:**String 类是不可改变的，所以一旦创建了 String 对象，那它的值就无法改变。

**一系列操作**

```java
String str = "helo test";
// 字符串长度
int len = str.length();

// 连接字符串
str1.concat(str2);

// 创建格式化字符串
// 仅仅打印
System.out.printf("浮点型变量的值为 " +
                  "%f, 整型变量的值为 " +
                  " %d, 字符串变量的值为 " +
                  "is %s", floatVar, intVar, stringVar);
// 创建一个格式化字符串----很类似go的fmt.Spriontf()
String fs;
fs = String.format("浮点型变量的值为 " +
                   "%f, 整型变量的值为 " +
                   " %d, 字符串变量的值为 " +
                   " %s", floatVar, intVar, stringVar);
```

## 3 StringBuidler和StringBuffer类

当要对字符串进行修改时，需要使用StringBuffer和StringBuidler类。

和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象。

StringBuilder与StringBuffer的不同：sbuilder的方法线程不安全，但速度快。sbuffer的方法线程安全

来看一个实例代码：

```java
public class Test{
  public static void main(String args[]){
    StringBuffer sBuffer = new StringBuffer("菜鸟教程官网：");
    sBuffer.append("www");
    sBuffer.append(".runoob");
    sBuffer.append(".com");
    System.out.println(sBuffer);  
  }
}
```

## 4 数组

数组对于每一门编程语言来说都是重要的数据结构之一，当然不同语言对数组的实现及处理也不尽相同。

Java的数组大小固定，元素类型相同。

**声明数组**

```java
int[] list;

// or c/c++ style
int list[];

// or
double[] myList = new double[size];
```

**处理数组**

for循环，for-each循环

```java
// for
public class TestArray {
   public static void main(String[] args) {
      double[] myList = {1.9, 2.9, 3.4, 3.5};
 
      // 打印所有数组元素
      for (int i = 0; i < myList.length; i++) {
         System.out.println(myList[i] + " ");
      }
   }
}

// for-each
public class TestArray {
   public static void main(String[] args) {
      double[] myList = {1.9, 2.9, 3.4, 3.5};
 
      // 打印所有数组元素
      for(double value : myList) {
         System.out.println(value + " ");
      }
   }
}
```

**Arrays类**

java.util.Arrays 类能方便地操作数组，它提供的所有方法都是静态的。

大概具有一下功能：

- 给数组赋值：通过fill方法
- 排序：sort
- 比较：equals
- 查找：比如binarySearch进行二分查找

## 5 Stream、File和IO

Java.io 包几乎包含了输入、输出需要的类。而流类代表了输入源和输出目标。

Java.io 包中的流支持很多种格式，比如：基本类型、对象、本地化字符集等等。

一个流可以理解为一个数据的序列。输入流表示从一个源读取数据，输出流表示向一个目标写数据。

## 6 Scanner类

通过 Scanner 类来获取用户的输入。

```java
Scanner s = new Scanner(System.in);
```

例子：

```java
import java.util.Scanner; 
 
public class ScannerDemo {
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        // 从键盘接收数据
 
        // next方式接收字符串
        System.out.println("next方式接收：");
        // 判断是否还有输入
        if (scan.hasNext()) {
            String str1 = scan.next();
            System.out.println("输入的数据为：" + str1);
        }
        scan.close();
    }
}
```

## 7 异常处理

捕获异常：(这个就有点类似于python的异常处理机制)

```java
try
{
   // 程序代码
}catch(ExceptionName e1)
{
   //Catch 块
}

// finally关键字
try{
  // 程序代码
}catch(异常类型1 异常的变量名1){
  // 程序代码
}catch(异常类型2 异常的变量名2){
  // 程序代码
}finally{
  // 程序代码
}
```

