## 函数

### 偏函数

> 1. 在对符合某个条件，而不是所有情况进行逻辑操作时，使用偏函数
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

> 1. 能够接受函数作为参数的函数

* 定义高阶函数，f函数需要两个Int类型参数，返回Int类型的值

  ```scala
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

* 定义一个返回函数的函数，返回的函数`Int=>Double`接收Int返回Double

  ```scala
  def minsxy(x:Int):Int=>Double={
      (y:Int)=>x-y
  }
  ```

* 调用方式一，完整写法，minsxy(10)相当于`(y:Int)=>10-y`

  ```scala
  val f = minsxy(10)
  println(f(5))
  ```

* 调用方式二，简化写法

  ```scala
  println(minsxy(10)(2))
  ```

### 参数类型推断

> 1. 参数类型已知时可以省略，比如集合的map()操作
>
> 2. 函数只有一个参数时，可以省略()
>
> 3. 如果变量只在=>右侧只出现一次，省略=>，使用_代替变量
>
>    

* 例子

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

  