## CAP基本概念


> https://robertgreiner.com/cap-theorem-revisited/

引用Robert Greiner的一篇博文，先给CAP下个定义，以及什么样的场景适用于CAP理论v

> The CAP Theorem states that, in a distributed system (a collection of interconnected nodes that share data.), you can only have two out of the following three guarantees across a write/read pair: Consistency, Availability, and Partition Tolerance - one of them must be sacrificed.

CAP定理指出，在分布式系统中（共享数据的互连节点的集合），在写/读操作中，只能有以下三个保证中的两个：一致性、可用性和分区容忍度，必须牺牲其中一个。

* 节点间互联
* 节点间数据共享 

> Consistency - A read is guaranteed to return the most recent write for a given client.

读操作保证返回给定客户端的最新写操作。 

* 从客户端读角度描述集群内数据的一致性

> Availability - A non-failing node will return a reasonable response within a reasonable amount of time (no error or timeout).

非故障节点将在合理的时间内（无错误或超时）返回合理的响应。

* 合理但可能不正确
* 没有错误
* 没有超时

> Partition Tolerance - The system will continue to function when network partitions occur.

当发生网络分区时，系统将继续工作。

* 集群提供服务的能力

![CAP](https://robertgreiner.com/content/images/2019/09/CAP-overview.png)

## CAP该如何选择

### CA
* 放弃P，但网络无法保证100%可靠，分区必然会出现，所以P是必选
* 当分区发生时，为了保证C，系统要禁止写入，无法提供服务，又违背了A

### CP
* 放弃A，保证数据一致性
![CP](https://robertgreiner.com/content/images/2019/09/CAP-CP.png)

### AP
* 放弃C，保证高可用
![AP](https://robertgreiner.com/content/images/2019/09/CAP-AP.png)

## CAP细节

### 如何做取舍
以一个用户管理系统为例，用户管理系统包含用户账号数据（用户ID，密码），用户信息数据（性别，爱好，昵称），通常在分布式环境下，用户账号数据会选择CP，用户信息数据使用AP，一个分布式系统环境下不可能只有一种数据，对不同的数据做不同的取舍，技术方案才有最大的落地可能，不然也能停留在PPT上

### CAP忽略网络延时
CAP理论是忽略到网络延时的，实际场景中A节点数据复制到B节点一定会有网络延时，在复制过程中，A、B节点数据并不一致
一些与钱相关的数据是无法在分布式环境下做到完美一致的，连CP都做不到，必须是CA，单点写入，针对单点做备份，分布式场景下只是这一小部分数据无法做到分布式，并不影响整个系统是分布式的

### 正常运行下不存在CP、AP选择
如果P没有发生，就不存在选择CP、AP的可能，系统就是CA的，架构落地方案要考虑P发生时选择CP还是、AP，也要考虑P没有发生时如何保证CA

### 二选一后放弃并不意味着什么都不做
CAP中做二选一后，被牺牲的那个只是在P发生过程中无法保证C或A，并不意味着P恢复后，什么都不做，要早P发生后考虑后续的数据恢复，确保系统数据是CA的

### ACID
* Atomicity：一个事物中的所有操作全部成功或者全部失败，不存在中间状态，一部分成功，一部分失败，事物执行过程发生错误，要会滚到事物开始之前的状态
* Consistency：事物开始前和事物结束后，数据库的完整性没有被破坏
* Isolation：数据库允许多个并发事物同时对数据进行读/写/修改的能力，隔离性可以防止多个事物并发操作数据是发生脏读、不可重复度、换读，隔离界别有如下
    * Read uncommitted
    * Read committed
    * Repeatable read
    * Serializable
* Durability：事物结束后，对数据的修改是永久的，即使数据库发生故障也不会丢失

### BASE理论
* Basically Available：分布式系统出现故障时，允许部分可用性，保证核心功能可用
* Soft State：允许系统存在中间状态，中间状态不会影响系统整体可用性，中间状态指CAP中C的状态
* Eventual Consistency：系统中所有数据副本经过一定时间后，最终能达到一致状态
BASE是对CAP中AP方案的一种补充，牺牲一致性，但不是放弃一致性


