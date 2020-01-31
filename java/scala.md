
## scala基本概念
### scala第一印象 
1. 多范式的编程语言，支持面向对象和函数式编程，Java不是一门完全面向对象的语言，比如说Java中有`byte、short、int、long、float、double、boolean`这些基本数据类型；有static类型，脱离对象实例可直接使用；有`null`这么个玩意，就是个值，不是对象，但是scala是完全面向对象，使用的过程用好好体会一下“什么是面向对象”

2. `.scala`文件会被编译成`.class`文件运行在JVM上，可以调用Java类库，与Java语言无缝衔接

3. 与java一样，scala也是静态类型语言，在编译期就需要知道声明的类型（JS动态类型语言，运行时判断类型）

3. 作为一门语言来看，scala代码还非常简洁的，艺术圈有一种设计风格叫`极简主义`，但是成年人的世界要明白任何极简主义都会代来其它方面的复杂度提升，正所谓`天下没有免费的午餐`，scala代码的简洁意味着思想复杂度的提升，对于习惯面向对象开发思想的程序员，函数式编程绝对是巨大的挑战（那点简洁全集中在语法糖上了）

4. 学习过程中多用java语言做类比，接受度会大大提升，重新建立一种知识体系和在现有知识体系中添砖加瓦，哪个效率更高，掰掰手指头算一下

### Hello Scala
学习任何一门编程语言，第一个程序都是`Hello World`，看看怎么使用scala打印`Hello World`
```java
object Hello {
  def main(args: Array[String]): Unit = {
    println("hello scala")
  }
}
```
解释一下这段程序：
1. JVM是从`main()`开始执行的，先关注一下`main()`，scala中定义方法使用`def`关键字
2. 在`def`前没有访问权限修饰符，scala中默认的权限是public，不需要显示定义（java中默认的访问权限是package）
3. `main()`中定义的参数`args: Array[String]`，scala定义参数的方式为`变量名:变量类型`，与java定义参数和声明变量方式相反，scala的作者认为变量在使用过程中更加关注对变量的逻辑操作，给变量启一个好名称更加的重要
4. scala中没有`void`这种太不面向对象的关键字，使用`Unit`代表返回值为`void`
5. 方法声明后使用`:`连接方法返回值，返回值后使用`=`连接方法体
6. `println()`就是打印输出了
7. 所有语句不需要使用`;`结束

这段scala hello world程序比起java hello world还是少打了几个单词的，但是复杂度绝对高于java

### 窥探细节
都JVM上的语言，使用java反编译这段“Hello World”程序，窥探一下细节，有助于了解scala的一些思想，通过`.class`反编译后的代码如下，程序只定义了一个`Hello.scala`文件，怎么反编译后生成了两个java文件
```java
public final class Hello {
  public static void main(String[] paramArrayOfString) {
    // 通过Hello$中MODULE$静态变量调用成员main()
    Hello$.MODULE$.main(paramArrayOfString);
  }
}

public final class Hello$ {
  public static Hello$ MODULE$;
  /*
  成员方法
  */
  public void main(String[] args) {
    scala.Predef$.MODULE$.println("hello scala");
  }
  
  private Hello$() {
    // 将当前类赋值给MODULE$
    MODULE$ = this; 
  }
}
```
scala是完全面向对象的语言，JVM调用main()后不能直接调用程序代码，要先生成一个`Hello$`对象，通过Hello$中的成员方法`main()`实现打印“hello scala”（在java中JVM通过Hello.main()直接调用程序逻辑，整个过程可以没有对象实例产生），在编译过后会生成两个类`Hello.class、Hello$.class`用于模拟静态调用过程

### 伴生对象
反编译后对于这种以`$`结尾的对象scala中称之为`伴生对象`，`Hello$`就是`Hello`的伴生对象，伴生对象中的方法都可以通过类名直接访问（模拟java中的静态方法），在语法层面就是使用`object`声明的类
```scala
object Hello {
  def main(args: Array[String]): Unit = {
    // 通过伴生对象直接访问方法
    Hello.method
  }
  // 定义方法，先不用关注语法细节
  def method():Unit={
    println("通过伴生对象直接方法")
  }
}
```

### 变量
1. 没有基本数据类型，所有类型都是对象`Byte、Short、Int、Long、Float、Double、Char、Boolean`，没有包装类的概念，但可以直接使用java的包装类
2. 真的是一切皆对象
```
# scala
10.toDouble // 10在scala中是个对象
# java
Integer i = 10; // 10在java中是字面量，i才是真正的对象，java语言其实没那么“一切皆对象”
```
3. java中的变量分为“基本类型”、“引用类型”，scala中分为“值类型（AnyVal）”、“引用类型（AnyRef）”，`AnyVal、AnyRef`都是对象，且无法相互转换
4. 类型提升
```
# scala
var c:Char = 'A'
c = c+1 // 所有“+”都是Int操作，Byte、Short、Char没有“+”、“-”指令
var i:Int  = c+1 // 类习惯提升为Int

# java
int byte=10; // 重载只与类型、个数、顺序有关
public void print(byte i){}
public void print(short i){} // 如果没有print(byte i)，将会调用print(short i)基本类型考虑数据精度
public void print(char i){} // char没有负数
public void print(int i){}  // 没有byte、short，会调用int
```
4. `Unit`表死无值，类似与java中的void，用于不返回任何结果的方法，只有一个“()”实例；`Null`只有一个null实例对象；`Nothing`任何类型的子类型，在异常时没有正常返回值时使用
5. scala中的变量必须显示初始化，无法先声明变量，再初始化变量
```
# scala
var name:String
name = "tom" // scala无法通过编译
# java
String name;
name = "tome"; // java可以编译通过
```
2. `var`声明的变量值可以修改，`val`声明的变量值不能修改，反编译class文件后局部变量`var、val`是没有差别的，只在编译层面上有区别，全局变量`val`会被声明为`final`类型
```scala
var name = "tom"
name = "jack" // 编译通过

val name = "tom"
name = "jack" // 编译无法通过
```
3. 变量的类型如果能通过编译器推断出来，可以省略不写，类型第一次确定后不可以修改
```scala
val name = "tom" // 带""一定是字符串
name = true // name已经是String类型，不能修改为Boolean类型
val result = 1 + 1 // 默认推断为Int类型
```

### 运算符
1. scala中没有`++、--`运算符，使用`+=、-=`代替
2. scala中所有表达式都有值
```
# scala
var i:Int = 10;
println(i=100) // 正常编译，返回()
# java
int i = 10
System.out.println(i=100); // 编译无法通过
```
3. scala中的所有的运算符都是方法调用

### for循环
1. `for(i <- 0 to 5)`范围1-5，`i`不用声明类型，编译器会自动推导出`1 to 5`的类型
2. `for(i <- 0 until 5)`范围1-4，不包含5 
3. `to、until`不是关键字，一切皆对象`0.until(5)、0.to(5)` 
4. `for(i <- Range(0,10,2))`Range作为一个类，可以设置步长`def apply(start: Int, end: Int, step: Int): Range = new Range(start, end, step)`
5. `for(i <- 0 to 10 if i%2==0)`循环后添加一个boolean表达式`if i%2==0`，替代continue
6. `for(i <- 0 to 10; j=10;y=2)`循环中可以引入变量，如果不想使用";"，可是使用`for{}`每个表达式独立成行
7. `val unit:Unit = for(i <- 0 to 10)`默认返回值类型是Unit，`val res:Vector = for(i <- 0 to 10) yield i`使用yield将每次循环的结果以集合的形式返回
8. `for (i <- 1 to 3;from = 4-1;j<-from until 10)`for中可以出现多个表达式，相互间用;分割


### while/do...while循环
1. while永远返回Unit，其它与java中的while一毛一样

### 循环中断
1. scala中没有break关键词，break作为流程控制的关键字在java中出现，但是break是脱离对象使用的，scala中提供以对象的方式实现break功能
2. Breaks是scala中的一个中断对象，中断采用抛出异常的方式实现
```scala
    Breaks.breakable{ // 类似try{}
      for(i <- 1 to 10){
        if(i==5) {
          Breaks.break() // 抛异常，中断循环
        }
        println(s"i=$i")
      }
    }
```

## 面向对象 VS 函数式编程
* 面向对象：将问题拆解为一个一个小问题，形成对象，分别解决（继承、实现、多态、包含、引用、重载）
* 函数式编程：将问题拆解为步骤，每个步骤一个功能（函数），关注点在功能的入参、出参，不关注功能的具体实现
* method：声明和调用时必定从属于某个类或某个类的实例
* function：随意声明随意使用（声明时关注入参数、返回值；调用时关注是否有实例对象）
* 函数没有重载，因为不依赖对象实例，同一个作用域中函数名不能相同
* 函数使用`def`定义，`def i: String = "A"`、`val j: String = "A"`

### 函数返回值类型推断
```
def function():Unit={
  return "A" // 不会有任何返回，方法声明Unit
}

def function():String={
  "A" // 最后一行作为方法的返回值
}
def function()="A"
def function="A" // 调用是不能加()

def funciton{"A"} // 没有= 表示方法没有返回值，最后一行不会返回

()->"A" // 匿名函数，直接调用
()=>println("B") // 匿名函数，只声明不调用，需要赋值给变量或作为函数参数进行传递
```

### 参数列表
* 参数匹配规则是从左到右
```
def function(params:String*){} // 可变参数

def function(date:String,format:String="yyyy-MM-dd"){} // 默认参数，不传format将使用声明的默认值

function(date="1999",format="yyyy") // 带名参数
```

## 函数的一等公民特性
### 值函数
* 函数可以赋值给变量，使用这个变量就等同于调用这个函数
```
// 将匿名函数赋值给变量double
val double: Int => Int = (i: Int) => i * 2
println(double(2))

// 将min函数赋值给fun变量
def min(x: Int, y: Int): Int = if (x > y) y else x
val fun: (Int, Int) => Int = min 
println(fun(1,10))
```
### 带函数参数的函数
* 使用函数作为参数传递
```
// i、j两个值如何操作由fun()决定
def function(i: Int, j: Int, fun: (Int, Int) => Int): Int = {
  fun(i, j)
}
// 调用
println(function(2, 2, (i, j) => {
  i + j
}))
println(function(2, 2, (i, j) => {
  i - j
}))
println(function(2, 2, (i, j) => {
  i * j
}))
println(function(2, 2, (i, j) => {
  i / j
}))
```
### 函数的返回值为函数
* 函数可以作为返回值
```
def function(i: Int): Int => Int = {
  (j:Int) => i * j // 匿名函数
}
function(2)(2)
```
* 函数柯里化（闭包实现柯里化）
```
// 返回值是函数
def fun1(i:Int):Int=>Int={ 
  def fun2(j:Int):Int={
    i*j
  }
  fun2 _
}
println(fun1(2)(3))
// 柯里化
def fun(i: Int)(j: Int): Int = {
  i * j
}
println(fun(2)(3))
```
* 闭包：将外部变量引入到函数的内部，改变外部变量的生命周期
```
def fun1(i:Int):Int=>Int={
  def fun2(j:Int):Int={
    i*j // 改变i的声明周期，扩大i的作用域到fun2中
  }
  fun2 _
}
```
### 声明函数位置相对随意
```
def fun1(): Unit = {
  def fun2(): Unit = {
    def fun3(): Unit = {

    }
  }
}
```
### 函数调用的简化写法
```scala
def fun(f: Int => Unit): Unit = {
  f(10)
}

fun((i: Int) => println(i)) // 完整
fun(i => println(i))  // 省略参数类型
fun(println(_)) // 省略参数
fun(println)  // 极简
```


## 面向对象

### package
* package可以多次声明
```
package org.foo
package bar.example
// 完整的包路径是org.foo.bar.example
```
* package内只能声明类，不能声明变量、方法，包也是对象，在包里声明变量和方法使用package object
```
package object foo{ // 在foo包下
  val field:String = "A"
  def function(i:Int):Unit={}
}
```
* 所有语法都支持嵌套
```
package foo1 {
  package object foo2 { // 包对象
    val book = "scala"

    def fun(): Unit = {
      println("包对象")
    }
  }

  class BarA { // 包中定义类
    var name: String = _
  }

  package foo2 {

    class BarA {
      var age: Int = _
    }

    object PackageExample {
      def main(args: Array[String]): Unit = {
        val bar1 = new BarA //就近原则，使用foo2包下的BarA
        bar1.age = 3
        println(bar1.age)
        
        val bar2 = new foo1.BarA //明确使用foo1包下的BarA
        bar2.name = "bar"
        println(bar2.name)

        fun() //调用包对象中的内容
        println(book)
      }
    }
  }
}
```


  

### import
* java中的import叫导入类，scala叫导入包
* 默认自动引入的包`java.lang.*、scala.*、Predf.*`
* 导入包中所有的类（java中使用*）
```
import java.util._ 
```
* 导入一个包中多个类
```
import java.util.{Date,ArrayList}
```
* 绝对路径查找（自己定义的包路径于jdk的包路径重名时，指定使用jdk包中的类）
```
import _root_.java.util._
```
* 给导入对象启别名
```
import java.util.{HashMap=>JavaHashMap} // 当前类中所有的HashMap使用JavaHashMap代替
```

### field
* 访问权限`private（当前类）、defualt（package权限，没有default关键字，private[包名]）、protected（子类）、public（默认权限，没有public关键字）`
* 类里声明的函数叫方法，和函数声明一样，只是在类中定义，必须通过对象实例调用
* 伴生对象中`apply([参数])`创建伴生类对象，不需要使用`new`关键字
* `_`默认初始化变量
* @BeanProperty生成与javabean同一的set/get方法
```
class Person{ // 伴生类 
  var name:String = _ // _ 默认初始值，public setter/getter
  private var age:Int = _ // private setter/getter 无法在外部访问
  val email:String = _ // pubulic getter 没有setter

  protected var address:String = _ // 子类访问
  private[p1] phone:Int = _ // 只能在p1包下访问属性
}
object Person{ // 伴生对象
  def apply(): Person = new Person() // 通过伴生对象创建伴生类 val p:Person = Person 不需要new

  def fun():Unit={ // 可以通过类名直接访问，类似静态方法
    val person:Person = new Person()
    person.age // 可以访问伴生类中的私有属性
  }
}
```

### 构造方法与apply
* scala中分主构造方法、辅助构造方法，辅助构造方法必须调用主构造方法
```
class Person(name:String,age:Int){ // 主构造方法
  this(name:String,age:Int,address:String){ // 辅助构造方法
    this(name,age) // 必须调用主构造方法
  }
}
```
* apply
```
class Pig(var name: String) {
  def info: Unit = println(name)
}

object Pig {
  def apply(name: String): Pig = new Pig(name)
}

val p = new Pig("new佩奇")
p.info
val pig = Pig("apply佩琦")//调用apply方法
pig.info
```
* 构造方法参数作用域
```
/**
  * 主构造器中的参数name是Cat1中的局部变量
  * @param name
  */
class Cat1(name:String){

}

/**
  * 主构造器中的参数使用val声明，会被当作类中的只读属性，有getter，类外部可以方法，但不能赋值
  * @param name
  */
class Cat2(val name:String){

}

/**
  * var声明的nane是读写属性
  * @param name
  */
class Cat3(var name:String){}
```

* 如果父类有主构造方法，子类要显示调用父类的主构造方法
```
class A(name: String) {

}
class B(name: String, age: Int) extends A(name) { // 显示调用父类的主构造方法
  def this(name: String, age: Int, address: String) {
    this(name, age) // 调用子类主构造方法
  }
}
```

* 抽象类与抽象方法
``` 
/**
 * 抽象类
 */
abstract class Tiger {
  /**
   * 1、生成两个抽象方法
   * public abstract String name(){}
   * public abstract void name_$eq(name:String){}
   * 2、子类重写父类的属性，等于实现了抽象方法
   * 3、抽象类不能使用private、final关键字
   */
  var name: String //抽象类中没有实例化的属性称为抽象属性
  /**
   * 普通字段
   */
  var age: Int = _
  /**
   * 1、抽象方法不需要abstract标记
   * 2、没有方法体的方法就是抽象方法
   */
  def fun(): String
  /**
   * 普通方法
   */
  def fun2(): Unit = {
    println("抽象类中的方法")
  }
}

class MaleTiger extends Tiger {
  /**
   * 重写父类的属性=实现抽象方法
   * scala操作属性=操作方法
   */
  override var name: String = _ 

  //  override var age: Int = _ // 编译通过，运行报错，不能override一个已经赋值的var变量，val是可以的
  override def fun(): String = {
    "实现抽象方法"
  }
}
```



### 内部类
```
class OuterClass{
  //给外部类定义别名
  outer=>
  var name:String="name"
  /*
  内部类
   */
  class InnerClass{
    def info():Unit={
      //访问外部类属性
      println(OuterClass.this.name)
      //通过别名访问
      println(outer.name)
    }
  }
}
// 实例化
val out1= new OuterClass
val out2 = OuterClass
val inner1 = new out1.InnerClass//内部类
```
* 静态内部类
```
object OuterClass{
  /*
  静态内部类，通过伴生对象实现
   */
  class StaticInnerClass{

  }
}
// 实例化
val inner2 = new OuterClass.StaticInnerClass//静态内部类
```


### trait
* scala中没有interface概念，有trait概念
* trait中可以有代码
```
trait A{
  pritln("A") // 代码
  var name:String = _ // 属性
  def fun1():Unit={} // 方法
  def fun2():Unit //抽象方法
}
```
* trait继承性与执行顺序
```
trait TA{} 
trait TB{}
class CA{}
class Person extends CA with TA with TB{}
```
* 多trait加载顺序从左到右，方法调用顺序从右向左
* java中的接口可以在scala中当作trait使用
* 动态混入

```
trait A{}
class Person{}
new Person with A // 动态混入

trait A{
  this.Exception=> // 只在有异常时使用Exception.getMessage，不需要extends Exception
  def method():Unit={
    this.getMessage
  }
}
```

### 类的Class信息
```
val clazz:Class[Person] = classOf[Person] // 获取Person类的Class实例对象

person.isInstanceOf[Person] // 判断person是否为Person对象实例
person.asInstanceOf[Object] // person强转为Object
```