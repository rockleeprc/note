## 函数

### 偏函数

> 1. 对符合某个特定条件，而不是所有条件进行逻辑操作时，使用偏函数
>
> 2. 将包裹在大括号内的一组case语句封装为函数，我们称之为偏函数
>
> 3. 偏函数在Scala中是一个特质PartialFunction

* 对List中的数值元素进行提取，使用偏函数实现，也是使用filter+map、模式匹配（模式匹配会出现()问题）

  ```scala
  val list = List(1, 2, 3, "abc")
  /*
      new PartialFunction[Any, Int]：Any函数接收类型，Int函数返回类型
       */
  val parialFun = new PartialFunction[Any, Int] {
      /**
          * 当返回true时调用apply()
          *
          * @param x
          * @return
          */
      override def isDefinedAt(x: Any): Boolean = {
          x.isInstanceOf[Int]
      }
  
      /**
          * isDefinedAt()为true是才会被调用
          *
          * @param v1
          * @return
          */
      override def apply(v1: Any): Int = {
          v1.asInstanceOf[Int] + 1
      }
  }
  //隐式转换，调用偏函数
  val res = list.collect(parialFun)
  println(res)
  
  //filter+map实现
  val res1 = list.filter(x => x.isInstanceOf[Int]).map(x => x.asInstanceOf[Int]).map(x => x + 1)
  //模式匹配实现
  val res1 = list.map(x => x match {
      case e: Int => e + 1
      case _ =>
  })
  ```

* 偏函数的两种简略写法

  ```scala
  val list = List(1, 2, 3, "abc")
  //第一种，定义偏函数+模式匹配
  def pFun: PartialFunction[Any, Int] = {
      case e: Int => e + 1
  }
  val res = list.collect(pFun)
  //第二种，不用定义偏函数，直接模式匹配
  val res =list.collect{case i:Int=>i+1}
  ```

### 匿名函数

> 1. 没有名字的函数
> 2. 函数不需要定义返回值，通过类型推导获得
> 3. 函数不使用=，使用=>
> 4. 使用val接收函数

* 定义一个匿名函数，函数体是`(x: Int, y: Int) => x * y`

  ```scala
  val plus = (x: Int, y: Int) => x * y
  println(plus(1, 10))
  ```

### 高阶函数

> 1. 能够接收函数作为参数的函数

* 定义高阶函数，f函数需要两个Int类型参数，返回Int类型的值

  ```scala
  //参数中的函数没有变量，只有类型
  def highFunction(f:(Int,Int)=>Int,x:Int,y:Int):Int={
      f(x,y)
  }
  ```


* 定义一个满足`f:(Int,Int)=>Int`的函数

  ```scala
  def plus(x:Int,y:Int):Int={
      x+y
  }
  def mod(x:Int,y:Int):Int={
      x%y
  }
  ```

* 调用时函数plus不需要()

  ```scala
  highFunction(plus,1,20)
  ```

> 2. 返回一个函数

* 定义一个返回函数的函数，返回函数的参数类型是`Int=>Double`

  ```scala
  def minsxy(x:Int):Int=>Double={
      (y:Int)=>x-y
  }
  ```

* 调用方式一，完整写法，minsxy(10)相当于`(y:Int)=>10-y`

  ```scala
  val f = minsxy(10)//x=10
  println(f(5))//y=5
  ```

* 调用方式二，简化写法

  ```scala
  println(minsxy(10)(5))
  ```

### 参数类型推断

> 1. 参数类型已知时可以省略，比如集合的map()操作
> 2. 函数只有一个参数时，可以省略()
> 3. 如果变量只在=>右侧只出现一次，省略=>，使用_代替变量

* map()、reduce()简写

  ```scala
  val list = List(1, 2, 3)
  list.map((x: Int) => x * 2)
  list.map((x)=>x*2)
  list.map(x=>x*2)
  list.map(_*2)
  
  list.reduce((x:Int,y:Int)=>{x+y})//匿名函数
  list.reduce((x,y)=>x+y)//省略类型，不能省略()
  list.reduce(_+_)//省略类型和()，使用_代替
  ```

### 闭包

> 1. 一个函数与其相关的引用环境组合为一个整体（实体、对象）
>
> 2. 闭包的变量可以在函数中反复使用

* 闭包的函数，变量x是外部传进来的值，被关闭的minsxy()函数中，f匿名函数和变量x形成闭包

  ```scala
  def minsxy(x:Int):Int=>Double={
      (y:Int)=>x-y
  }
  ```

* 闭包的最佳实现案例，makeSuffix()和suffix变量一起组成闭包，suffix不用每次都传入到makeSuffix()中，fileName是每次都需要传入的变量

  ```scala
  def makeSuffix(suffix: String): String => String = {
      (fileName: String) => {
          if (fileName.endsWith(suffix)) {
              fileName
          } else {
              fileName + suffix
          }
      }
  }
  
  val f = makeSuffix(".jpg")
  println(f("abc"))
  println(f("hello.jpg"))
  println(makeSuffix(".jpg")("abc"))
  ```

### 柯里化

> 1. 接收多个参数的函数都可以转化为接收单个参数的函数，这个转化过程就叫柯里化
> 2. 柯里化的思想是把函数中的每一步操作进行拆分
> 3. 体会：常规函数->闭包函数->柯里化函数（柯里化后函数的简略写法，参考类型推断）

* 计算两个数的乘积

  ```scala
  def mul1(x:Int,y:Int):Int=x*y	//常会函数
  def mul2(x:Int):Int=>Int=(y:Int)=>x*y	//闭包函数
  def mul3(x:Int)(y:Int):Int=x*y	//柯里化函数
  ```

### 控制抽象函数

> 1. 参数是函数
> 2. 参数函数没有输入参数也没有输出参数

* 正常的没有参数的函数，在RunThread1()内启动给一个线程，执行f函数

  ```scala
  def RunThread1(f: () => Unit): Unit = {
      new Thread {
          override def run(): Unit = {
              f()
          }
      }.start()
  }
  //()
  RunThread1(() => {
      println("aaa")
      println("bbb")
  })
  //{}
  RunThread1 {
      () =>
      println("aaa")
      println("bbb")
  }
  ```

* 使用控制抽象函数实现，等同于breakable{}的实现方式

  ```scala
  def RunThread2(f: => Unit): Unit = {//f函数没有()
      new Thread {
          override def run(): Unit = {
              f//调用是也不用加()
          }
      }.start()
  }
  //调用，函数{代码内容}
  RunThread2 {
      println("aaa")
      println("bbb")
  }
  ```

* todo:控制抽象案例，实现类似while()

  ```scala
  
  ```

