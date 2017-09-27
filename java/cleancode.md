
# 第二章 有意义的命名

## 名副其实
* 如果名称需要注释来补充,那就不算是名副其实

## 避免误导
* 别用accountList来指定一组账号,除非他真的是List类型,用accountGroup,bunchOfAccounts替代,accounts也会好一些,即便容器真的时List最好也别在名称中写出容器类型
* XYZControllerHandinglingOfStrings,XYZControllerEfficientOfStrings这样的名称太相似了

## 做有意义的区分
* 不用使用a1,a2,a3这种没有区分意义的变量名称
* 废话都是冗余的
  * Variable一词永远不要出现在变量中
  * Table一词永远不要出现在表中
  * NameString会比Name更好吗
* getActiveAccount(),getActiveAccounts(),getActiveAccountInfo(),程序员不看注释或代码能知道该调用哪个函数
* 要区分名称,就应该以读者能鉴别不同之处的方式来区分
  * moneyAmount与money没区别
  * customerInfo与customer没区别
  * accountData与account没区别

## 使用读得出来的名称
* 当一个名称能够读出来,这个名称会更善于记忆和理解

## 使用可以搜索的名称
* 若变量或常量可能在代码中多处使用,则赋一个便于搜到的名称
* STUENT可能在系统中的多个地方出现,但使用MAX_CLASSES_PER_STUDENT就很容易在系统中搜索到
* 名称长度应与其作用域相对应

## 避免使用编码
* 类中的成员变量不必用m_前缀标明,应该把类和函数做的足够小,消除对成员前缀的需要
* IXxxService不需要用I来修饰接口(不想让调用方知道返回的是接口)

## 类名称
* 类名和对象名应该是名词或名词短语,如Customer,WikiPage,Account,避免使用Manager,Processor,Data类名不应当是动词

## 方法名
* 方法名应当是动词或动词短语,如postPayment,deletePage
* 重载构造器时使用静态工厂方法(参考Effective Java)

## 每个概念对应一个词
* 在一堆代码中有controller,manager,driver,它们之间到底有什么分别,会让人困惑

## 避免双关语
* 避免将同一个词用于不同的目的,同一术语用于不同的概念就是双关语
* 代码的作者应该尽力写出易于理解的代码

## 使用解决方案领域的名称
* 只有程序员才会读你的代码,尽量用那些计算机科学术语,算法名,模式名,数学术语吧
* 如果不能用程序员熟悉的术语来命名,就采用所涉及问题领域而来的名称,至少读代码的人还可以去请教领域专家

## 添加有意义的环境语
* 除非用于良好命名的类,函数,或者名称空间来放置名称,给读者提供语境,否则只能添加前缀

## 不要添加没用的语境
* 只要名称足够清楚,语境足够清晰,就不要添加语境

      class Address{
        String addrFirstName;
        String addrLastName;
      }

## 小结
* 取好名字最难的地方在于需要良好的描述技巧和共有的文化背景


# 第三章 函数

## 短小
* 函数的第一规则是要短小,第二规则是还要短小
* 函数不应该有100行那么长,20行封顶最佳
* if,else,while等语句,其中的代码块应该只有一行,该行应该是一个函数语句的调用

## 只做一件事
* 函数应该只做一件事,做好这件事,只做这一件事

## 每个函数一个抽象层级
* 确保函数只做一件事,函数中的语句都要在同一个抽象层级上
* 要让代码拥有自顶向下的阅读顺序

## 使用描述性的名称
* 长而具有描述性的名称,要比短而令人费解的名称要好;长而具有描述性的名称,要比描述性的注释要好

## 函数参数
* 最理想的参数数量是零个,其次是一个,再次是两个,应尽量避免三个参数
* 输出参数比输入参数还要难理解,习惯于信息通过参数输入函数,通过返回值从函数中输出

### 一元函数的普遍形式
* 操作该参数,将其转换为其它什么东西,再输出
      InputStream fileOpen("MyFile")
* 程序将函数看做一个事件,有输入参数而无输出参数,使用该参数修改系统状态

### 标识参数
* 向函数传入boolean参数会让方法签名立刻变得复杂起来,使函数不止做一件事情,如果参数为true将这样,如果参数为false将那样

### 参数对象
* 如果函数需要三个或三个以上的参数,就说明其中一些参数应该封装为类

### 动词与关键字
* 对于一元函数,函数和参数应该形成一种非常良好的"动词(名词)"对形式,如,write(name)

## 无副作用
* 通过函数名称的描述,函数承诺只做一件事,而不会产生函数名称描述外的副作用

## 分隔指令与询问
* 函数要么做什么事,要么回答什么事,但二者不可兼得
      # 拆解为两个函数更好
      public boolean set(String attribute,String value)

## 使用异常代替返回码
* 从指令式函数返回错误码轻微违反了指令于询问分隔的规则
      if(deletePage(page)==Error.OK)

### 抽离try/catch代码块
* try/catch搞乱了代码的结构,把错误处理与正常流程混为一谈

      public void delete(Page page){
        try{
          deletePage(page);
          registry.deleteReference(page.name);
          configKeys.deleteKey(page.name.makeKey());
        }catch(Exception e){
          logger.log(e.getMessage());
        }
      }

* 函数应该只做一件事,错误处理就是一件事

      # 只做与错误处理有关
      public void delete(Page page){
        try{
          deletePageAllReferences(page);
        }catch(Exception e){
          logError(e);
        }
      }
      # 只做于删除一个page有关
      private void deletePageAllReferences(Page page) throws Exception{
        deletePage(page);
        registry.deleteReference(page.name);
        configKeys.deleteKey(page.name.makeKey());
      }
      # 只与记录错误日志有关
      private void LogError(Exception e){
        logger.log(e.getMessage());
      }

## 如何写出这样的函数
* 打磨代码,分解函数,修改名称,消除重复

## 小结
* 程序员把系统当做故事来讲,而不是程序来写,真正的目标在于讲述系统的故事,编写的函数必须干净利落的拼装在一起,形成一种精确而清晰的语言,帮助你讲故事


# 第四章 注释
* 别给糟糕的代码加注释——重新写新的吧
* 若编程语言足够有表达力，或这程序员善于用语言来表达意图，就不那么需要注释
* 如果发现自己需要写注释，想想看是否有办法翻盘，用代码来表达意图
* 注释存在的时间越久，就离其所描述的代码越远，越来越变得全然错误，因为程序员不能坚持维护注释
* 真是的程序意图只在代码中，尽管有时也需要注释，也应该多花写心思尽量减少注释量

## 注释不能美化糟糕的代码
* 带有少量注释的整洁而有表达立的代码，要比带有大量注释的零碎而复杂的代码像样的多
* 多少注释也无法解救糟糕的代码

# 第六章 对象和数据结构

## 数据、对象的反对称性
* 过程式代码（使用数据结构的代码）便于在不改动既有数据结构的前提下添加新函数，面向对象代码便于在不改动既有函数的前提下添加新类

## 得墨忒耳律
* 得墨忒耳律：模块不应了解它所操作对象的内部情形
* 对象隐藏数据，暴露操作，对戏不应该通过getXXX暴露其内部结构
* 得墨忒耳律认为，类C的f()应该调用以下对象的方法
	* C中定义的方法
	* 由f()创建的对象
	* 作为参数传递给f的对象
	* 由C的实体变量持有的对象
* 方法不应该调用由任何函数返回的对象的方法


# 第七章 错误处理
* 错误处理很重要，如果它搞乱了代码逻辑，就是错误的做法

## 使用异常而非返回码
* tryToShutDown()中的方法调用不检查返回值的合法性，通过异常处理参数不合法的问题，代码整洁

		public void sendShutDown(){
			try{
				tryToShutDown();
			}catch(Exception e){
				logger.log(e);
			}
		}

## 先写try-catch-finally语句
* 尝试编写强行抛出异常的测试，在往代码中添加行为，使之满足测试要求，先构造try代码块事物范围

## 使用不可控异常
* 在方法中抛出可控异常，而catch语句在三个层级之上，需要在catch语句和抛出异常处之间的每个方法签名中声明该异常
* 可控异常的代价是违反开闭原则，封装被打破，异常抛出的路径中每个函数都需要了解下一层的异常细节

## 别返回null值
* 返回null值，是在给自己增加工作量，需要对函数返回值做！=null判断，影响代码结构
* 函数返回null不如抛出异常或返回特定对象
* 如果第三方API中有函数返回null，考虑用新方法包装

## 别传递null值
* 大多数编程语言中，没有良好的方法能对付由调用者意外传入的null值，恰当的做法就是禁止传入null值

