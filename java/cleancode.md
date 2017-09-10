
# 第二章 有意义的命名

## 名副其实
* 如果名称需要注释来补充,那就不算时名副其实

## 避免误导
* 别用accountList来指定一组账号,除非他真的时List类型,用accountGroup,bunchOfAccounts替代,accounts也会好一些,即便容器真的时List最好也别在名称中写出容器类型
* XYZControllerHandinglingOfStrings,XYZControllerEfficientOfStrings这样的名称太相似了

## 做有意义的区分
* 不用使用a1,a2,a3这种没有区分意义的变量名称
* 废话都是冗余的,Variable一词永远不要出现在变量中,Table一词永远不要出现在表中,NameString会不Name更好吗
* getActiveAccount(),getActiveAccounts(),getActiveAccountInfo(),程序员不看注释或代码能直到该调用那个函数
* moneyAmount于money没区别,customerInfo与customer没区别,accountData与account没区别,要区分名称,就应该以读者能鉴别不同之处的方式来区分

## 使用读得出来的名称
* 当一个名称能够读出来,这个名称会更善于记忆

## 使用可以搜索的名称
* 若变量或常量可能在代码中多出使用,则赋一个便于搜到的名称
* STUENT可能在系统中的多个地方出现,但使用MAX_CLASSES_PER_STUDENT就很容易在系统中搜索到
* 名称长多应与其作用域相对应

## 避免使用编码
* 类中的成员变量不必用m_前缀标明,应该把类和函数做的足够小,消除对成员前缀的需要
* IXxxService不需要用I来修饰接口(不想让用知道给他们返回的时接口,这点觉得于面向接口编程有些违背)

## 类名称
* 类名和对象名应该时名词或名词短语,如Customer,WikiPage,Account,避免使用Manager,Processor,Data,类名不应当是动词

## 方法名
* 方法名应当时动词或动词短语,如postPayment,deletePage
* 重载构造器时使用静态工厂方法(参考Effective Java)

## 每个概念对应一个次
* 在一堆代码中有controller,manager,driver,它们之间到底有什么分别,会让人困惑

## 避免双关语
* 避免讲同一个次用于不同的目的,同一术语用于不同的概念就是双关语
* 代码的作者应该尽力写出易于理解的代码

## 使用解决方案领域的名称
* 只有程序员才会读你的代码,尽管用那些计算机科学术语,算法名,模式名,数学术语吧
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
* 如果函数需要两个,三个或三个以上的参数,就说明其中一些参数应该封装为类

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
