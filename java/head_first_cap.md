> https://robertgreiner.com/cap-theorem-revisited/


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