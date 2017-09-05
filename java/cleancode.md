


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
