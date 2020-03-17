# Java基基础-3


- [Java基基础-3](#Java基基础-3)
  - [1 继承](#1-继承)
  - [1.1 继承格式](#11-继承格式)
  - [1.2 QuickStart](#12-quickstart)
  - [1.3 继承关键字](#13-继承关键字)
  - [1.4 构造函数](#14-构造函数)
- [2 Override/Overload](#2-overrideoverload)
  - [2.1 重写](#21-重写)
  - [2.2 重载](#22-重载)
- [3 多态](#3-多态)
  - [3.1 QuickStart](#31-quickstart)
  - [3.2 实现方式](#32-实现方式)
- [4 抽象类](#4-抽象类)
- [5 封装](#5-封装)
- [6 接口](#6-接口)
   - [6.1 接口声明与实现](#61-接口声明与实现)
   - [6.2 接口的继承](#62-接口的继承)
- [7 包package](#7-包package)
  - [7.1 创建和导入包](#71-创建和导入包)
- [8 Reference](#8-Reference)

## 1 继承

>  继承很简单啊，就是为了代码复用。一个类去继承另一个类。

继承是java面向对象编程技术的一块基石，因为它允许创建分等级层次的类。

### 1.1 继承格式

使用 extends 关键字

```java
class father{
  
}

class child extends father{
  
}
```

java支持的继承：

<img src="https://www.runoob.com/wp-content/uploads/2013/12/types_of_inheritance-1.png" width=60%>

**继承的作用：**

- 子类继承了父类非private的属性、方法

- 子类可以拥有自己的属性和方法，即子类可以对父类进行扩展
- 子类可以对父类的方法进行重写

### 1.2 QuickStart

来看一个例子:

```java
// 企鹅
public class Penguin { 
    private String name; 
    private int id; 
    public Penguin(String myName, int  myid) { 
        name = myName; 
        id = myid; 
    } 
    public void eat(){ 
        System.out.println(name+"正在吃"); 
    }
    public void sleep(){
        System.out.println(name+"正在睡");
    }
    public void introduction() { 
        System.out.println("大家好！我是"         + id + "号" + name + "."); 
    } 
}

// 老鼠
public class Mouse { 
    private String name; 
    private int id; 
    public Mouse(String myName, int  myid) { 
        name = myName; 
        id = myid; 
    } 
    public void eat(){ 
        System.out.println(name+"正在吃"); 
    }
    public void sleep(){
        System.out.println(name+"正在睡");
    }
    public void introduction() { 
        System.out.println("大家好！我是"         + id + "号" + name + "."); 
    } 
}
```

不难发现，代码太冗余了。

这时，就是使用继承的好机会！

**Animal.class**

```java
public class Animal { 
    private String name;  
    private int id; 
    public Animal(String myName, int myid) { 
        name = myName; 
        id = myid;
    } 
    public void eat(){ 
        System.out.println(name+"正在吃"); 
    }
    public void sleep(){
        System.out.println(name+"正在睡");
    }
    public void introduction() { 
        System.out.println("大家好！我是"         + id + "号" + name + "."); 
    } 
}
```

这个Animal类就可以作为一个父类，然后企鹅类和老鼠类继承这个类之后，就具有父类当中的属性和方法，子类就不会存在重复的代码，维护性也提高，代码也更加简洁。

**Penguin.class && Mouse.class**

```java
// penguin.class
public class Penguin extends Animal{
  public Penguin(String myName, int myid){
    // 调用父类的构造方法
    super(myName, myid);
  }
}

// mouse.class
public class Mouse extends Animal { 
    public Mouse(String myName, int myid) { 
        super(myName, myid); 
    } 
}
```

### 1.3 继承关键字

- **extends&implements**

  `extends`和`implements`两个关键字都可以实现继承，所有的类都是继承于 java.lang.Object。

  当一个类没有继承的两个关键字，则默认继承object（这个类在 **java.lang** 包中，所以不需要 **import**）

  extends前面已经说过了，我们下面来看implements关键字。

  `implements`可以变相的使java具有多继承的特性，使用范围为**类继承接口**的情况，可以同时继承多个接口

  ```java
  public interface A{
    public void eat();
    public void sleep();
  }
  
  public interface B{
    public void show();
  }
  
  public class C implements A,B{
    
  }
  ```

- **super&&this**

  super：通过它访问父类成员

  this：指向自己的引用

  ```java
  class Animal{
    void eat(){
      System.out.println("animal : eat");
    }
  }
  
  class Dog extends Animal{
    void eat() {
      System.out.println("dog : eat");
    }
    
    void eatTest(){
      this.eat();
      super.eat();
    }
  }
  
  public class Test {
    public static void main(String[] args) {
      Animal a = new Animal();
      a.eat();
      Dog d = new Dog();
      d.eatTest();
    }
  }
  ```

- **final**

  final：修饰类，把类定义为不能继承的，即最终类；修饰方法，该方法不能被子类重写。

  ```java
  // 修饰类
  final class Dog{
    
  }
  
  // 修饰方法
  public final static void num(){
    
  }
  ```

### 1.4 构造函数

子类不会继承父类的构造函数。它只是显/隐式调用。

如果父类的构造函数有参数，则必须在子类的构造函数中通过`super`来调用父类的构造函数。

如果父类构造器没有参数，则无需显式调用，系统会自动调用父类的无参数构造函数。

```java
class SuperClass {
  private int n;
  SuperClass(){
    System.out.println("SuperClass()");
  }
  SuperClass(int n) {
    System.out.println("SuperClass(int n)");
    this.n = n;
  }
}
// SubClass 类继承
class SubClass extends SuperClass{
  private int n;
  
  SubClass(){ // 自动调用父类的无参数构造器
    System.out.println("SubClass");
  }  
  
  public SubClass(int n){ 
    super(300);  // 调用父类中带有参数的构造器
    System.out.println("SubClass(int n):"+n);
    this.n = n;
  }
}
// SubClass2 类继承
class SubClass2 extends SuperClass{
  private int n;
  
  SubClass2(){
    super(300);  // 调用父类中带有参数的构造器
    System.out.println("SubClass2");
  }  
  
  public SubClass2(int n){ // 自动调用父类的无参数构造器
    System.out.println("SubClass2(int n):"+n);
    this.n = n;
  }
}
public class TestSuperSub{
  public static void main (String args[]){
    System.out.println("------SubClass 类继承------");
    SubClass sc1 = new SubClass();
    SubClass sc2 = new SubClass(100); 
    System.out.println("------SubClass2 类继承------");
    SubClass2 sc3 = new SubClass2();
    SubClass2 sc4 = new SubClass2(200); 
  }
}
```



## 2 Override/Overload

### 2.1 重写

子类对父类允许访问的方法进行重写。

>  注意：重写方法不能抛出新的检查异常或者比被重写方法申明更加宽泛的异常

```java
class Animal{
   public void move(){
      System.out.println("动物可以移动");
   }
}
 
class Dog extends Animal{
   public void move(){
      System.out.println("狗可以跑和走");
   }
}
 
public class TestDog{
   public static void main(String args[]){
      Animal a = new Animal(); // Animal 对象
      Animal b = new Dog(); // Dog 对象
 
      a.move();// 执行 Animal 类的方法
 
      b.move();//执行 Dog 类的方法
   }
}
```

尽管 b 属于 Animal 类型，但是它运行的是 Dog 类的 move方法。

这是由于在编译阶段，只是检查参数的引用类型。

然而在运行时，Java 虚拟机(JVM)指定对象的类型并且运行该对象的方法。

因此在上面的例子中，之所以能编译成功，是因为 Animal 类中存在 move 方法，然而运行时，运行的是特定对象的方法。

如果Dog再来一个bark()方法并调用，编译就会出错。

### 2.2 重载

重载是同名方法，但参数或返回值不同，就是重载。

```java
public class Overloading {
    public int test(){
        System.out.println("test1");
        return 1;
    }
 
    public void test(int a){
        System.out.println("test2");
    }   
 
    //以下两个参数类型顺序不同
    public String test(int a,String s){
        System.out.println("test3");
        return "returntest3";
    }   
 
    public String test(String s,int a){
        System.out.println("test4");
        return "returntest4";
    }   
 
    public static void main(String[] args){
        Overloading o = new Overloading();
        System.out.println(o.test());
        o.test(1);
        System.out.println(o.test(1,"test3"));
        System.out.println(o.test("test4",1));
    }
}
```

## 3 多态 

### 3.1 QuickStart

有点像Go的interface{}结构体的鸭子类型——只要某个类型实现了某个interface{}中的函数，那么这个类型就实现了这个接口，也就是这个接口的类型。

而Java的多态，体现在基类的指针也可以调用子类的方法。向上转型和强制类型转换。

```java
public class Test {
    public static void main(String[] args) {
        show(new Cat());
        show(new Dog());

        Animal a = new Cat();
        a.eat();
        Cat c = (Cat)a;
        c.work();
    }

    public static void show(Animal a){
        a.eat();
        if(a instanceof Cat){
            Cat c = (Cat)a;
            c.work();
        }else if(a instanceof Dog){
            Dog c = (Dog)a;
            c.work();
        }
    }
}

abstract class Animal{
    abstract void eat();
}

class Cat extends Animal{
    public void eat(){
        System.out.println("eat fish");
    }
    public void work(){
        System.out.println("catch mouse");
    }
}

class Dog extends Animal{
    public void eat(){
        System.out.println("eat bone");
    }
    public void work(){
        System.out.println("housekeeping");
    }
}
```

以上代码，Dog和Cat继承自Animal。在`show(Animal a)`函数中，接收的参数类型也可以是Dog和Cat。调用继承方法时，可以直接通过animal指针进行调用。给人的感觉就是，Dog和Cat也是Animal类型。

### 3.2 实现方式

多态的实现方式有三种：

- 重写
- 接口
- 抽象类和抽象方法

## 4 抽象类

在面向对象的概念中，所有的对象都是通过类来描述的。

但并非所有的类都用来描绘对象，如果一个类没有足够的信息来描绘一个对象，那这个类就是抽象类。

抽象类不能被实例化喂对象，但可以被继承，然后实例化子类。

Java中，抽象类表示的是一种继承关系。

抽象类就是在在前面加一个`abstract`关键字：`public abstract class Employee`

**抽象方法**

设计这么一个类：有一个特别的method，它的具体实现只能由子类来实现，那么就可以将至声明为抽象方法。

```java
public abstract class Employee{
  private String name;
  private String address;
  private int number;
  
  public abstract double computPay()''
}
```

抽象方法的要求：

- 抽象类中才能包含抽象方法
- 子类必须重写父类的抽象方法，或者子类本身也是抽象类
- 构造函数、static方法，不能被声明为抽象方法

## 5 封装

封装：将抽象性函数接口的实现细节包装、隐藏起来。

封装可以被认为是一个保护屏障，防止该类的代码和数据被外部类定义的代码随机访问。

要访问的话，必须通过严格的接口控制。

封装的最主要功能在于我们能修改自己的实现代码，而不用修改调用我们程序的代码。

**Java封装的步骤**

1. 修改属性的可见性
2. 为每个属性提供公共访问method

**示例**

```java
/* EncapTest.java */
public class EncapTest{
 
   // 设为私有
   private String name;
   private String idNum;
   private int age;
 
   // 公共方法
   public int getAge(){
      return age;
   }
 
   public String getName(){
      return name;
   }
 
   public String getIdNum(){
      return idNum;
   }
 
   public void setAge( int newAge){
      age = newAge;
   }
 
   public void setName(String newName){
      name = newName;
   }
 
   public void setIdNum( String newId){
      idNum = newId;
   }
}

/* RunEncap.java */
public class RunEncap{
   public static void main(String args[]){
      EncapTest encap = new EncapTest();
      encap.setName("James");
      encap.setAge(20);
      encap.setIdNum("12343ms");
 
      System.out.print("Name : " + encap.getName()+ 
                             " Age : "+ encap.getAge());
    }
}
```

## 6 接口

接口在Java中是一个抽象类型，是**抽象方法的集合**。类描述对象的属性和方法。接口则包含类要实现的方法。

一般情况下，继承接口的类要实现接口中的所有method，除非这个类是abstract。

接口无法被实例化，但是可以被子类实现。

**接口特性**：

- 每一个method都是隐式抽象的，被隐式的指定为`public abstract`（只能是这个，否则报错
- 接口中可以有变量，但被隐式指定为`public static final`（只能是`public`,`private`会报错
- 接口中的方法只能由子类实现

### 6.1 接口声明与实现

接口的声明如下：

```java
public interface NameOfInterface{
  // final，static字段
  // 抽象方法
}
```

来看一段示例代码：

```java
// MammaInt.java
public class MammaInt implements Animal1 {
    public void eat(){
        System.out.println("Mammal eats");
    }
    public void travel(){
        System.out.println("Mammal travels");
    }
    public int noOfLegs(){
        return 0;
    }

    public static void main(String[] args) {
        MammaInt m = new MammaInt();
        m.eat();
        m.travel();
    }
}

interface Animal1{
    void eat();
    void travel();
}
```

### 6.2 接口的继承

一个接口能继承另一个接口，和类之间的继承方式比较相似。接口的继承使用extends关键字，子接口继承父接口的方法。

```java
public class MammaInt extends Animal implements Animal1{}
```

**接口的多继承**

Java中，类的多继承是不合法的，但是接口可以多继承。

```java
public interface Hockey extends Sports, Event
```

**标记接口**

最常用的继承接口是一个没有任何方法的接口。它只是用来表明实现了它的类，属于某个特定类型。

```java
package java.util;
public interface EventListener
{}
```

标记接口主要用于：

- 建立一个公共的父接口

  正如`EventListener`接喽，这是由其他几十个接口扩展的Java API。可以使用一个标记接口来当作一组接口的父接口。

  例如：当一个接口继承了EventListener接口，Java虚拟机(JVM)就知道该接口将要被用于一个事件的代理方案。

- 类属于这个接口类型

  > 这个地方就很像Go的interface{}了

## 7 包package

为了更好的组织类，Java提供了包机制——区别类名的命名空间。

**包的作用**：

- 把功能相似的类or接口组织在一个包中
- 避免命名冲突
- 限定访问权限；拥有包访问权限的类才可以访问包中的类

包的语法格式：

```java
// package pkg1.pkg2;
package net.java.util;
public class Something{
  ...
}
```

那么以上代码的路径为`net/java/util/Something.java`。

一个包可以定义为一组互相联系的类型，比如Java中的一些包：

- java.lang：打包基础的类
- java.io：输入输出功能的类和函数

### 7.1 创建和导入包

创建包的时候，你需要为这个包取一个合适的名字。

如果某个源文件要属于这个包，则要将包的声明放在源文件开头。

包声明需放在第一函，每个原文件只属于一个包；若未使用包声明，则其中的实体都被放在一个`unnamed package`中。

**包的注意事项**：首字母一般小写。

来看一个例子：先直接在IDE里面创建一个package，名为animal。

```java
/* Animal.java */
package animals;
 
interface Animal {
   public void eat();
   public void travel();
}

package animals;
 
/* MammalInt.java */
public class MammalInt implements Animal{
 
   public void eat(){
      System.out.println("Mammal eats");
   }
 
   public void travel(){
      System.out.println("Mammal travels");
   } 
 
   public int noOfLegs(){
      return 0;
   }
 
   public static void main(String args[]){
      MammalInt m = new MammalInt();
      m.eat();
      m.travel();
   }
}
```

**import关键字**

为了能够使用某一个包的成员，我们需要使用`import`关键字来导入这个包。

```java
import package1[.package2…].(classname|*);
```

来看例子：

下面的 payroll 包已经包含了 Employee 类，接下来向 payroll 包中添加一个 Boss 类。Boss 类引用 Employee 类的时候可以不用使用 payroll 前缀，Boss 类的实例如下。

```java
package payroll;
 
public class Boss
{
   public void payEmployee(Employee e)
   {
      e.mailCheck();
   }
}
```

Boss不在payroll包中：

```java
// 使用全名
payroll.Employee;
// improt *
import payroll.*;
// import 类
import payroll.Employee;
```

**package的目录结构**

包名成为类名的一部分；包名必须与目录结构吻合。

```java
// com/runoob/test/Runoob.java
package com.runoob.test;
public class Runoob {
      
}
class Google {
      
}
```
## 8 Reference
- [菜鸟教程_Java教程](https://www.runoob.com/java/java-tutorial.html)
