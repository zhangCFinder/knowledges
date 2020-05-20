
[TOC]

## 一、Broker注册
Broker是分布式部署并且相互之间相互独立，但是需要有一个注册系统能够将整个集群中的Broker管理起来，此时就使用到了Zookeeper。在Zookeeper上会有一个专门用来进行Broker服务器列表记录的节点：

/brokers/ids

每个Broker在启动时，都会到Zookeeper上进行注册，即到/brokers/ids下创建属于自己的临时节点，如/brokers/ids/[0...N]。

Kafka使用了全局唯一的数字来指代每个Broker服务器，不同的Broker必须使用不同的Broker ID进行注册，创建完节点后，**每个Broker就会将自己的IP地址和端口信息记录到该节点中去**。其中，**Broker创建的节点类型是临时节点**，一旦Broker宕机，则对应的临时节点也会被自动删除。

初始化时 ZooKeeper 中 /brokers/seqid 节点的状态如下：

```shell
[zk: xxx.xxx.xxx.xxx:2181/kafka(CONNECTED) 6] get /brokers/seqid
null
cZxid = 0x200001b2b
ctime = Mon Nov 13 17:39:54 CST 2018
mZxid = 0x200001b2b
mtime = Mon Nov 13 17:39:54 CST 2018
pZxid = 0x200001b2b
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 0
```
可以看到 dataVersion=0，这个就是前面所说的 version。在插入一个空字符串之后，dataVersion 就自增1，表示数据发生了变更，这样通过 ZooKeeper 的这个功能来实现集群层面的序号递增，整体上相当于一个发号器。

## 二、 Controller
在Kafka集群中会有一个或者多个broker，其中有一个broker会被选举为控制器（Kafka Controller），它负责管理整个集群中所有分区和副本的状态。
* 当某个分区的leader副本出现故障时，由控制器负责为该分区选举新的leader副本。

* 当检测到某个分区的ISR集合发生变化时，由控制器负责通知所有broker更新其元数据信息。

* 当使用kafka-topics.sh脚本为某个topic增加分区数量时，同样还是由控制器负责分区的重新分配。

Kafka中的控制器选举的工作依赖于Zookeeper，成功竞选为控制器的broker会在Zookeeper中创建/controller这个临时（EPHEMERAL）节点，此临时节点的内容参考如下：
```json
{"version":1,"brokerid":0,"timestamp":"1529210278988"}
```
其中version在目前版本中固定为1，brokerid表示称为控制器的broker的id编号，timestamp表示竞选称为控制器时的时间戳。

在任意时刻，集群中有且仅有一个控制器。

1. 每个broker启动的时候会去尝试去读取/controller节点的brokerid的值，如果读取到brokerid的值不为-1，则表示已经有其它broker节点成功竞选为控制器，所以当前broker就会放弃竞选；

2. 如果Zookeeper中不存在/controller这个节点，或者这个节点中的数据异常，那么就会尝试去创建/controller这个节点，当前broker去创建节点的时候，也有可能其他broker同时去尝试创建这个节点，只有创建成功的那个broker才会成为控制器，而创建失败的broker则表示竞选失败。

3. 每个broker都会在内存中保存当前控制器的brokerid值，这个值可以标识为activeControllerId。


Zookeeper中还有一个与控制器有关的/controller_epoch节点，这个节点是持久（PERSISTENT）节点，节点中存放的是一个整型的controller_epoch值。

controller_epoch用于记录控制器发生变更的次数，即记录当前的控制器是第几代控制器，我们也可以称之为“控制器的纪元”。

controller_epoch的初始值为1，即集群中第一个控制器的纪元为1，当控制器发生变更时，每选出一个新的控制器就将该字段值加1。每个和控制器交互的请求都会携带上controller_epoch这个字段，

* 如果请求的controller_epoch值小于内存中的controller_epoch值，则认为这个请求是向已经过期的控制器所发送的请求，那么这个请求会被认定为无效的请求。

* 如果请求的controller_epoch值大于内存中的controller_epoch值，那么则说明已经有新的控制器当选了。

由此可见，Kafka通过controller_epoch来保证控制器的唯一性，进而保证相关操作的一致性。


## 三、Topic注册

在Kafka中，同一个Topic的消息会被分成多个分区并将其分布在多个Broker上，这些分区信息及与Broker的对应关系也都是由Zookeeper在维护，由专门的节点来记录，如：
```shell
/borkers/topics
```

Kafka中每个Topic都会以/brokers/topics/[topic]的形式被记录，如/brokers/topics/login和/brokers/topics/search等。

Broker服务器启动后，会到对应Topic节点（/brokers/topics）上注册自己的Broker ID并写入针对该Topic的分区总数，


## 四、__consumers_offsets

具体内容见6.Kafka offset管理