## 基础知识

* java8中的特性
    1 函数
    2 stream api、并行
    3 default接口
    4 optional
    5 模式匹配 

* 函数式编程中的两个思想
    1 在程序运行期间将方法、Lambda作为值传递
    2 在没有共享变量时，函数可以并行执行

* 行为参数化
    1 处理平凡变更需求的一种软件开发模式
    2 准备好一段代码，将这段代码块传递给一个方法（推迟代码块的执行），该方法的行为就基于代码块被参数化
    3 让方法接收多种行为作为参数，并在方法内部使用，使方法完成不同的功能
    
* Lambda特点
    1 简洁：不需要像匿名类一样写模版代码
    2 匿名：没有名称（写的少，想的多）
    3 函数：有参数列表、返回值、函数主体、可抛异常，不属于特定类
    4 传递：可以作为参数传递，也可以存储在变量中

* Lambda使用局部变量
    1 可以访问实例变量、静态变量、局部变量必须为final或事实final
    2 实例变量存储在堆上，局部变量存储在栈上，堆上的变量本来就是线程间共享的，访问局部变量时在访问它的副本（java8 p53）

* 函数式接口：只定义一个抽象方法的接口（p46）
    1 Predicate<T>  T->boolean
    2 Consumer<T>   T->void
    3 Function<T,R> T->R
    4 Supplier<T>   ()->T
    5 BinaryOperator<T> (T,T)->T

* 方法引用
    1 如果一个Lambda代表的只是“直接调用这个方法”，那最好还是用名称来调用它，而不是去描述如何调用它
    2 静态方法引用
    3 实例方法引用
    4 实例变量方法引用
    5 构造函数引用

* 复合Lambda
    1 比较复合：reversed()、thenComparing()
    2 谓词复合：and()、or、negate()
    3 函数复合：andThen()、compose()

## 函数式数据处理

* stream特点
    1 以声明的方式处理数据集合，遍历数据集的高级迭代器
    2 声明性、可复合、可并行
    3 集合的目的在于存储和访问，stream的目的在于计算（filter、map、sorted）
    4 只能被遍历一次

* 筛选
    1 filter
    2 distinct
    3 limit
    4 skip
* 映射
    1 map
    2 flatMap
* 查找匹配（短路操作）
    1 anyMatch：流中是否有一个元素能匹配给定的谓词
    2 allMath：流中所有元素是否匹配谓词
    3 noneMatch：流中没有任何元素匹配谓词
* 查找（短路操作）
    1 findAny：返回流中任意元素
    2 findFirst：返回流中第一个元素
    3 对于一些有序集合findAny与findFirst只在并行时有差别
* 规约
    1 reduce 
        有初始值返回T，没有初始值返回Optional<T>
        参数是BinaryOperator<T>，可以累加、求最小值、最大值
        任何传入两个参数，返回一个参数的逻辑reduce都能处理
        map-reduce结合使用

* 原始类型stream
    1 IntStream
    2 DoubleStream
    3 LongStream
    通过mapToInt将对象流转为原始流，通过boxed将原始流转为对象流

* 创建stream
    1 值创建流 Stream.of()、Stream.empty()
    2 数组创建流 Arrays.stream(array)
    3 由文件创建流 File类相关api
    4 函数生成流 Stream.iterate()迭代、Stream.generate()

* Collectors静态工厂方法（p128）
    1 汇总 
        counting()
        summingInt(对象映射为int所需的函数)
        summingLong(对象映射为long所需的函数)
        summingDouble()
    2 平均值 
        averagingInt(函数)
        averagintLong(函数)
        averagintDouble()
    3 最大值/最小值 maxBy()、minBy()
    4 对于汇总、平均值、最大/小值 
        summarizingInt()
        summarizingLong()
        summarizingDouble()
    5 连接字符串
        joining() 将每个对象的toString()连接到一起
        joining(分隔符)
    6 底层调用
        reducing() 多个重载
        sum() = reducing(0,对象转int函数,Integer::sum)
    7 分组 groupingBy()
    8 分区 partitionBy()

* 自定义收集器
    1 Collector接口
    ```java
    /**
    * T 流中要收集的元素范型
    * A 累加器类型，用于在收集过程中累加数据的对象范型
    * R 收集操作得到的结果对象范型
    */
    public interface Collector<T, A, R> {
        /**
        * A function that creates and returns a new mutable result container.
        *
        * @return a function which returns a new, mutable result container
        * 
        * 创建空的累加器实例，对应范型A
        */
        Supplier<A> supplier();

        /**
        * A function that folds a value into a mutable result container.
        *
        * @return a function which folds a value into a mutable result container
        *
        * 执行规约的操作函数，函数执行时有两个参数
        * A 保存规约结果的累加器
        * T 流中元素 
        */
        BiConsumer<A, T> accumulator();

        /**
        * A function that accepts two partial results and merges them.  The
        * combiner function may fold state from one argument into the other and
        * return that, or may return a new result container.
        *
        * @return a function which combines two partial results into a combined
        * result
        * 
        * 并行执行时对流的各个子部分归于所得到的累加器进行合并
        */
        BinaryOperator<A> combiner();

        /**
        * Perform the final transformation from the intermediate accumulation type
        * {@code A} to the final result type {@code R}.
        *
        * <p>If the characteristic {@code IDENTITY_TRANSFORM} is
        * set, this function may be presumed to be an identity transform with an
        * unchecked cast from {@code A} to {@code R}.
        *
        * @return a function which transforms the intermediate result to the final
        * result
        *
        * 将累加器对象转换为collect最终的结果对象
        */
        Function<A, R> finisher();

        /**
        * Returns a {@code Set} of {@code Collector.Characteristics} indicating
        * the characteristics of this Collector.  This set should be immutable.
        *
        * @return an immutable set of collector characteristics
        */
        Set<Characteristics> characteristics();
    }
    ```
    2 collect() 三个参数的重载方法

* 应用并行流的建议（p148）


## 高效Java8编程
 
 * 重构
    1 用Lambda替代匿名内部类（匿名类重载时转换为Lambda会代理理解上的晦涩，可以通过显示的类型转换消除这种情况p166）
    2 方法引用重构Lambda表达式
    3 使用streamAPI重构数据处理
    4 采用函数式接口

* Lambda重构设计模式
    1 策略 对于boolean策略使用Predicate替代
    2 观察者 替换Observer（简单业务逻辑下）
    3 责任链
    4 工厂
    Lambda不是银弹，在针对设计模式重构时，慎重选择 

* 默认方法多继承问题
    1 类或父类中申明的方法优先级高于接口接口中的default方法
    2 子接口的方法比父接口的优先级高
    3 继承多个接口时必须显示覆盖或显示调用期望的方法

* 创建Optional
    1 Optional.empty() 创建空的Optional
    2 Optional.of(obj) 创建非空的Optional，如果obj为空抛npe
    3 Optional.offNullable(obj) obj可以为null
* 操作Optional
    * map
    * flatMap
    * orElse
    * orElseGet orElse的延迟版本，用于创建重量级对象
    * orElseThrow 与get类似，抛出指定的异常
    * get 不安全，不存在抛异常
    * isPresent
    * filter
    * 无法序列化

* LocalDate
    1 不可变对象
    2 提供简单的日期，不包含时间
    3 没有时区相关信息
    4 LocalDate.of(year,month,day) 创建对象
    5 LocalDate.now() 获取当前时间
    6 LocalDate.parse("") 解析字符串
    7 with(TemporalAdjuster) 复杂日期操作（TemporalAdjusters工厂方法p253）
* LocalTime
* LocalDateTime
    1 合并LocalDate与LocalTime
    2 不带时区信息
    3 atTime(LocalTime)、atDate(LocalDate)向LocalDateTime传递日期、时间对象
    4 toLocalDate()、toLocalTime()从LocalDateTime中提取LocalDate与LocalTime
* DateTimeFormatter
    1 替代DateFormat
    2 ofPatter(格式)
* Instant 
    1 设计的初衷时为了便于机器使用
    2 由秒、纳秒组成
    3 从1970-1-1开始
* Duration 以秒、纳秒计算时间差值
* Period 以年、月、日计算时间差值
* ZoneId 时区



