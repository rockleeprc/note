## 创建型模式

### 单例

### 工厂

#### 简单工厂

 * 最简单的工厂，增加实现类，就在 getInstance()中增加if判断
 * 缺点：违反OCP原则，修改原有代码

#### 工厂方法

 * 解决简单工厂创建新实现类时违反OCP原则
 * 单独创建CarFactory接口，增加实现类同时增加实现类工厂
  * 	如：增加Audi实现类同时增加AudiFactory实现类
 * 不修改原有代码，只创建新类
 * 缺点：类暴增

#### 抽象工厂

 * 抽象工厂不是解决创建单个对象问题，解决创建一种类型的对象
 * 多产品族会使用到这种方式 

### 建造者

* interface Builder
  * 建的接口，创建组件，不装配
* interface Director
  * 不创建组件，只装备组件，装配Builder创建的对象（持有Builder接口对象）
* 对象创建很复杂时，分离对象的创建和装配
  * 创建：
  *  装配：

###  原型

* new对象与clone对象区别

* 实现cloneable接口，注意对象的深拷贝、浅拷贝
* 实现Serializable接口，通过I/O流实现对象的clone，深拷贝
  * ByteArrayOutputStream：写进内存
  * FileInputStream：写进文件

## 结构型模式

### 适配器

### 代理

### 桥接

### 装饰

### 组合

### 外观

## 享元