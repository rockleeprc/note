
Any
    AnyVal
        Unit等于void，用于函数返回值，唯一实例()
        StringOps等于String
    AnyRef
        Null唯一实例null，所有AnyRef的子类，AnyVal不能赋值null
Nothing 
    函数没有明确的返回值，不知道是AnyVal或AnyRef
    任何类型的子类
    函数抛出异常时，用于异常处理
    def ex(i:Int):Int{  // 返回值声明Int，Nothing是Int的子类
        if(i=0){
            throws Exception
        }
        return i 
    }

String
    ==：比较内容
    eq：比较地址

没有++、--运算法，使用+=、-=代替

def m(): Unit = {
    return "A"; // Unit可以有返回值，只是无意义
}
通用的父类Any
没有三元运算符，使用if else替代

for(i <- 1 to 10) // 1-10 to是方法调用，隐士转换调用RichInt
for(i <- Range(1,10)) // 1-9
for(i <- 1 until 10) // 1-9
for(i <- Array(1,2,3)) // 可以遍历集合类型，List、Set
for(i <- 1 to 10 if != 2) // 循环守卫，替代循环中的continue
for(i <- 1 to 10 by 2) // 设置循环步长 (1 to 10).by(2)，by不能为0
for(i <- 1 to 10 reverse) // 反转遍历
for(i <- 1 to 3;j <- 1 to 3) //  遍历二维数组
for(i <-1 to 3;j = 4-i) // 引入变量j
var res = for(i <- 1 to 3) yield i // 声明返回值，返回一个集合

没有continue、break
Breaks.breakable(
    for (i <- 1 to 3) {
        if (i == 2) Breaks.break(); // 循环退出
    }
)

def fun(str:String*) // 可变参数必须放在参数列表的最后
def fun(str:String="默认值") // 有默认值的参数可以不传
fun(name="liyan",age=18) // 传参数时可以直接指定参数名称
(name:String)=>{println(name)} // 只关心函数逻辑，省略def、函数名称，匿名函数，lambda表达式，一搬用在将函数作为参数传递，可以很简化

函数高阶用法
    val f = fun _ // 函数作为值传递
    def func(a:Int, b:Int, op:(Int,Int)=>Int):Unit={} // 函数作为参数传递

    def fun(): Int => Int = { // 函数作为函数的返回值
        def func(a: Int): Int = {
            a
        }
        func // 返回函数
    }
    // 使用
    fun()(222) 
    或
    val f = fun()
    f(222)

闭包：函数内部访问到函数外部的局部变量，函数所在的环境称为闭包
柯里化：把一个参数列表的多个参数，变成多个参数列表 
    (a,b,c) (a)(b)(c)
函数惰性执行使用lazy关键字

包对象：在包下定义同名对象，对象内的成员作为包下所有class、object的共享变量，可以直接访问
package object com{
    // 共享属性、方法
}

默认导包：
scala.Predef._
scala._
java.lang._ 

// 默认class是public权限
class Student{
    private var name:String="tom" // 私有权限，类内部、伴生对象访问
    @BeanProperty // 生成setter/getter 符合javabean规范
    var age:Int = 20 // 默认访问权限public，scala没有public关键字
    var sex:String = _ // _声明默认null值
    private[package] // 包访问权限
}

class Student(  // 定义参数列表
            name:String // 局部变量，类内可以访问
            var age:Int // 类内部属性
                ){ // 主构造器
    
    println("xxx") // 代码都是主构造器代码

    def this(参数列表){ // 辅助构造器，必须直接或间接调用主构造器
        this() // 调用主构造器
    } 
}

extends 可以指定父类使用哪个构造器执行，父类辅助构造器最终都会调用主构造器

java
    方法调用：动态绑定
    属性：静态绑定
scala 
    属性、方法都是动态绑定
    override 重写属性、方法

抽象类：abstract
抽象属性：没有初始化的属性
抽象方法：没有实现的方法
  abstract class A { // 抽象类
    val name: String

    def method(): Unit //
  }
  class B extends A {
    override val name: String = "A" // 只能override val的属性，var属性直接修改不需要override

    override def method(): Unit = {

    }
  }

abstract  class A(val age:Int,val name:String) // abstract可以有构造参数列表，trait不能有参数
val p = new Student with Person // trait动态混入
abstract、trait存在同名的属性、方法时，子类必须override
混入时super调用从右往左，只有没有with时才会调用extends类，也可以指定调用父类方法supler[指定父类].method()，优先使用trait
注意棱形继承带来的super调用问题

自身类型：相当于注入，却没有继承关系
class/abstract/trait A{}
trait B{
  _:A => // 注入A，B拥有A的所有属性和方法
}

obj.isInstanceOf[类型] 判断类型
obj.asInstanceOf[类型] 类型强转
classOf[类型] 获取类名

枚举 Enumeration
    object Color extends Enumeration{
        val red = Value(1,"red")
        val green = Value(2,"green")
    }
应用 App
定义别名 type
    type myString = String; // 定义别名

Iterable(trait)
    Seq
    Set
    Map

不可变集合推荐符号操作，可变集合推荐方法操作

不可变数组Array操作
    val array1: Array[Int] = new Array[Int](10) // 初始化
    val array2: Array[Int] = Array(1, 2, 3) // 初始化

    println(array2(0)) // 索引
    array2(0) = 11 // 修改、赋值
    array2.update(1, 22) // 修改
    // 添加元素
    val newArray1: Array[Int] = array2.:+(33) // 右侧添加，:+、+: 控制添加位置
    val newArray2: Array[Int] = 33 +: array2 // 左侧添加
    val newArray3: Array[Int] = 44 +: 33 +: array2 // 连续添加

    for (elem <- array2) {} // 迭代器遍历
    array2.foreach(println(_)) // 遍历
    for (i <- 0 until array2.length) {} // 索引遍历
    for (i <- array2.indices) {} // 索引遍历
    println(array2.mkString(",")) // 直接打印

可变数组ArrayBuffor操作
    val array1: ArrayBuffer[Int] = ArrayBuffer(1, 2, 3)
    val array2: ArrayBuffer[Int] = new ArrayBuffer[Int]

    // array2(0) // 未添加内容，索引越界

    // 符号添加
    val array3: ArrayBuffer[Int] = array2 :+ 12 // 可变集合不推荐
    array2 += 1 // 右侧添加
    99 +=: array2 // 左侧添加
    array2 += 2 += 3
    // 方法添加，推荐
    array2.append(4)
    array2.prepend(44)
    array2.insert(1, 999)
    // 删除
    array2 -= 99 // 元素删除
    array2.remove(2) // 索引删除
    array2.remove(2, 2) // 索引删除
    array2.update(2, 22) // 索引修改
to操作可以将Array、ArrayBuffer相互转换

不可变List操作
    val list1: List[Int] = List[Int](1, 2, 3)
    println(list1(0)) // 获取元素，无法赋值，只能获取

    // 添加元素
    val list2: List[Int] = list1 :+ 1
    val list3: List[Int] = 1 +: list1
    val list4: List[Int] = list1.::(51) // 左侧添加
    val list5: List[Int] = Nil.::(13) //
    val list6: List[Int] = 11 :: 22 :: 33 :: Nil // :: 更多应用于创建List
    val list7: List[Int] = list2 ::: list3 // 合并两个list
    val list8: List[Int] = list2 ++ list3 // 合并两个list

可变ListBuffer操作

    // 创建
    val list1: ListBuffer[Int] = new ListBuffer[Int]
    val list2: ListBuffer[Int] = ListBuffer(1, 2, 2)

    // 添加
    list2.prepend(33)
    list2.append(3)
    list2.insert(1, 22)
    99 +=: list2
    // 删除
    list2.remove(2) // 索引删除
    list2 -= 2 // 元素删除
    //  修改
    list2(0) = 12
    list2.update(0, 11) // 索引更新

    // 合并list
    val list3: ListBuffer[Int] = list2 ++ list1
    val list4: list1.type = list2 ++=: list1   

不可变Set
    // 默认不导包是不可变
    val set1: Set[Int] = Set(1, 2, 3, 3, 3)
    // 添加
    val set2: Set[Int] = set1 + 20
    // 删除元素
    val set3: Set[Int] = set1 - 3
    // 合并
    val set4: Set[Int] = set2 ++ set3

可变Set
    // 导包
    val set1: mutable.Set[Int] = mutable.Set(1, 2, 3)

    val set2: mutable.Set[Int] = set1 + 77 // 不修改原集合数据
    // 添加
    set1 += 11
    set1.add(22)
    // 删除
    set1.remove(1)
    set1 -= 11
    // 合并
    val set3: mutable.Set[Int] = set1 ++ set2
    set1 ++=set2

不可变Map
    val m1: Map[Int, String] = Map(1 -> "b", 2 -> "c")
    // 遍历
    m1.foreach((m: (Int, String)) => println(m))
    val keys: Iterable[Int] = m1.keys
    val values: Iterable[String] = m1.values

    // 获取
    val result: String = m1.getOrElse(3, "nothing") // 获取Some包装的值
可变Map
   val m1: mutable.Map[Int, String] = mutable.Map(1 -> "b", 2 -> "c")
    // 遍历
    m1.foreach((m: (Int, String)) => println(m))
    val keys: Iterable[Int] = m1.keys
    val values: Iterable[String] = m1.values
    // 获取
    val result: String = m1.getOrElse(3, "nothing") // 获取Some包装的值
    // 添加
    m1.put(3, "a")
    m1 += ((4, "d"))
    // 删除，key删除
    m1.remove(1)
    m1 -= 2
    // 修改
    m1.update(5, "f") // 与put等效
    // 合并
    m1 ++= m1
    val m2: mutable.Map[Int, String] = m1 ++ m1

元组：将多个无关的元素封装为一个整体，最大只能有22个元素
    val tuple: (String, Boolean, Int, Char) = ("A", true, 12, 'C')
    // 访问元素
    tuple._1
    tuple._2
    tuple.productElementName(1) // 索引访问
    // 遍历
    for (e <- tuple.productIterator) {
      println(e)
    }


p112