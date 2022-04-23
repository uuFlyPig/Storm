# Storm基础知识

## 一、什么是Storm

```
	Storm是Twitter开源的分布式实时大数据处理框架，被业界称为实时版Hadoop。随着越来越多的场景对Hadoop的MapReduce高延迟无法容忍，比如网站统计、推荐系统、预警系统、金融系统(高频交易、股票)等等，大数据实时处理解决方案（流计算）的应用日趋广泛，目前已是分布式技术领域最新爆发点，而Storm更是流计算技术中的佼佼者和主流。
	是一个分布式的，可靠的，容错的数据流处理系统。它会把工作任务委托给不同类型的组件，每个组件负责处理一项简单特定的任务。

```

## 二、Storm的特性

```
1.适用场景广泛： storm可以实时处理消息和更新DB，对一个数据量进行持续的查询并返回客户端（持续计算），对一个耗资源的查询作实时并行化的处理(分布式方法调用，即DRPC），storm的这些基础API可以满足大量的场景。

2. 可伸缩性高:  Storm的可伸缩性可以让storm每秒可以处理的消息量达到很高。扩展一个实时计算任务，你所需要做的就是加机器并且提高这个计算任务的并行度 。Storm使用ZooKeeper来协调集群内的各种配置使得Storm的集群可以很容易的扩展。

3. 保证无数据丢失： 实时系统必须保证所有的数据被成功的处理。 那些会丢失数据的系统的适用场景非常窄， 而storm保证每一条消息都会被处理， 这一点和S4相比有巨大的反差。

4. 异常健壮（强壮的鲁棒性）： storm集群非常容易管理，轮流重启节点不影响应用。

5. 容错性好：在消息处理过程中出现异常， storm会进行重试

6. 语言无关性： Storm的topology和消息处理组件(Bolt)可以用任何语言来定义， 这一点使得任何人都可以使用storm.

```

## 三、storm的基本架构

### 1.Storm的核心技术和基本组成

```
(1)Topology(拓扑) ：是一个有向图的计算，是Storm中运行的一个实时应用程序，因为各个组件间的消息流动而形成逻辑上的拓扑结构。类似于MapReduce的作业(Job)，其主要区别是Job最终会完成，而拓扑一直在运行直到它被杀死。

(2)Stream(流)：是Storm的核心抽象，一个流是一个无界Tuple序列，源源不断传递的元组就组成了流，在分布式环境中并行地进行创建和处理。

(3)龙卷(Spout)：是拓扑的流的来源，是一个拓扑中产生源数据流的组件。通常情况下，Spout会从外部数据源中读取数据，然后转换为拓扑内部的源数据。Spout的主要方法是nextTuple()，此方法会发出一个新Tuple到拓扑，如果没有新的元组发出，则简单的返回。nextTuple()方法不阻止任何Spout的实现，因为Storm在同一线程调用所有的Spout方法。

(4)闪电(Bolt)：Bolt是流的处理节点，从一个拓扑接收数据，然后执行进行处理的组件。Bolt可以完成过滤、 业务处理、连接运算、连接与访问数据库等任何操作，其是一个被动的角色，接口中有一个execute()方法，在接收到信息后会调用此方法，用户可以其中执行自己希望的操作。可以在Bolt中启动新的线程进行异步处理。

(5)流分组(Stream Grouping): 流分组用于在Bolt的任务中定义流应该如何分区。Storm中有8个内置的流分组接口：随机分组(Shuffle grouping)、字段分组(Fields grouping)、部分关键字分组(Partial key grouping)、全部分组(ALL grouping)、全局分组(Global grouping)、无分组(None grouping)、直接分组(Direct grouping)、本地或者随机分组(Local or shuffle grouping)。

(6)任务(Task)：每个Spout或者Bolt在集群执行许多任务。每个任务对应一个线程的执行，流分组定义如何从不一个任务集到另一个任务集发送Tuple。可通过TopologyBuilder类的setSpout()和setBolt()方法来设置每个Spout或者Bolt的并行度。

(7)Worker(工作进程)：是Spout/Bolt中运行具体处理逻辑的进程，拓扑跨一个或多个Worker进程执行。每个Worker进程是一个物理的JVM和拓扑执行所有任务的一个子集。Storm会尝试再所有Worker上均匀地发布任务。
```

### 2.其他基本概念

```
(1)主控节点与工作节点：Storm集群中有两类节点：主控节点(Master Node)和工作节点(Worker Node)其中，主控节点只有一个，而工作节点可以有多个。
(2)Nimbus进程与Supervisor进程：主控节点运行一个称为Nimbus的守护进程，类似于Hadoop的JobTracker。Nimbus负责在集群中分发代码，对节点分配任务，并监视主机故障。每个工作节点运行称为Supervisor的守护进程。Supervisor监听其主机上已经分配的主机的作业，启动和停止Nimbus已经分配的工作进程。Nimbus 和Supervisors 之间所有的协调工作是通过 一个Zookeeper 集群。Nimbus进程和 Supervisors 进程是无法直接连接，并且是无状态的;  所有的状态维持在Zookeeper中或保存在本地磁盘上。
(3)执行器(Executor)：在Storm0.8以后，Task不再与物理线程对应，同一个Spout/Bolt的Task可能会共享一个物理线程，该线程称为执行器。
(4)可靠性(Reliability)：Storm保证一个Spout元组将被拓扑完全可靠地处理。它跟踪每个Spout元组的元组树，检测树中的元组什么时候可以完成。每个拓扑都有“消息超时时间”，如果Storm在超时之前未能检测到Spout元组可以完成，那么会设元组为失败并在之后重新发射它。

```

### 3.序列化

```
(1)序列化：Storm使用Kryo序列化。kryo是一个灵活和快速的序列化库，产生很小的序列化。
```

### 4.容错机制

```
(1)Worker进程死亡：当一个Worker(工作进程)死亡，Supervisor会尝试重启它。如果它在启动时连续失败了一定次数，无法发送心跳信息到Nimbus，Nimbus将在另一台主机上重新分配Worker。
(2)节点死亡：如果节点死亡，分配给该节点主机的任务会暂停，Nimbus会把这些任务重新分配给其他节点主机。
(3)Nimbus或者Supervisor守护进程死亡:Nimbus和Supervisor守护进程被设计成快速失败的(每当遇到任何意外的情况,进程自动毁灭)和无状态的(所有状态都保存在ZooKeeper或者磁盘上)。如创建Storm集群中所述,Nimbus和Supervisor守护进程应该使用daemontools或者monit工具监控运行。所以,如果Nimbus 或 Supervisor守护进程死亡,它们重启并会像什么事情都没发生一样正常工作。
Nimbus 或者 Supervisor的死亡不影响Worker进程的工作。这与Hadoop不同,在Hadoop中,如果JobTracker进程死亡,所有正在运行的Job都会丢失。 
(4)Nimbus是否是“单点故障”：如果失去了Nimbus节点, Worker也会继续执行。另外,如果Worker死亡, Supervisor也会继续重启它们。但是,没有Nimbus, Worker不会在必要时(例如,失去一个Worker的主机)被安排到其他主机。所以Nimbus "在某种程度上”是单点故障(Single Point Of Failure, SPOF)。在实践中,这不是一个大问题,因为Nimbus守护进程死亡,不会发生灾难性的问题。Storm官方计划让Nimbus在未来实现高可用性 
(5)可靠性机制---保证消息处理：Storm保证每个来自Spout的消息将被完全地处理
	1.消息被完全处理的含义：拓扑从Kestrel队列读取句子，分割句子单词，来自Spout的一个元组会触发很多基于它创建的元组，会创建一个元组树，而当元组树创建完毕的时候，并且树的每个消息都已经被处理时，Storm认为来自Spout的元组是“完全处理”的，当一个元组的消息树在指定的超时范围内不能被完全处理，则元组被认为时失败的。
	2.如果一个消息被完全处理会调用ack方法，如果完全处理失败会调用fail方法。（一个元组将由Spout任务来确认成功或失败）
```

![img](https://img-blog.csdn.net/20180905105815163?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEwODI0NTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/2018090510591926?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEwODI0NTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 5.不同的流分组方式

```
1.随机分组(Shuffle grouping)：随机地分发元组到Bolt上的任务，这样能保证每个任务得到相同数量的元组。随机分组执行原子操作，如数学运算。如果操作不能被随机分发的话，应考虑其他的方式，如单词统计案例中，需要计算单词，就不适合使用随机分组。

2.字段分组(Fields grouping)：根据指定字段对流进行分组。如：如果流是按照user-id字段进行分组，具有相同user-id的元组总是被分发到相同的任务，具有不同的user-id的元组可能被分发到不同的任务。字段分组是实现流连接和关联，以及大量其他的用例的基础。在实现上，字段分组使用取模散列来实现。

3.广播分组(All Grouping):是指流被发送到所有Bolt的任务中。使用这个分组方式要小心。

4.全局分组(Global grouping)：指全部流都发送到Bolt的同一任务中，再具体一点，是发送给ID最小的任务。

5.无分组(None grouping):如果不关心流是如何分组的，则可以使用这种分组方式。目前这种分组和随机分组是一样的效果，有一点不同的是Storm会把这个Bolt放到Bolt的订阅者的同一个线程中执行。

6.直接分组(Direct grouping)：是一种特殊的分组。这种方式的流分组意味着由怨怒在的生产者决定元组的消费的接受元组的任务。直接分组只能在已经声明为直接流的流中使用，并且元组必须使用emitDirect方法来发射。Bolt通过TopologyContext对象或者OutputCollector类的emit方法的返回值，可以得到其消费者的任务id列表(List<Integer>)。

7.本地或者随机分组(Local or shuffle grouping)：如果目标Bolt在同一工作进程存在一个或多个任务，元组会随机分配给这些任务。否则，该分组方式与随机分组方式是一样的。

8.部分关键字分组(Partial key grouping)：与字段分组相似，根据定义的字段来对数据流进行分组。不同的是，者中分组方式会考虑下游Bolt数据处理的均衡性问题，在输入数据源关键字不平衡时会有更好的性能。

9.自定义分组：自定义流分组的方式，通过实现CustomStreamGrouping接口来创建自定义的流分组。
```



## 四、Storm的编程

```
Topology：计算拓扑，Storm 的拓扑是对实时计算应用逻辑的封装，它的作用与 MapReduce 的任务（Job）很相似，区别在于 MapReduce 的一个 Job 在得到结果之后总会结束，而拓扑会一直在集群中运行，直到你手动去终止它。拓扑还可以理解成由一系列通过数据流（Stream Grouping）相互关联的 Spout 和 Bolt 组成的的拓扑结构。

Spout：数据源（Spout）是拓扑中数据流的来源。一般 Spout 会从一个外部的数据源读取元组然后将他们发送到拓扑中。

Bolt：拓扑中所有的数据处理均是由 Bolt 完成的。通过数据过滤（filtering）、函数处理（functions）、聚合（aggregations）、联结（joins）、数据库交互等功能，Bolt 几乎能够完成任何一种数据处理需求。一个 Bolt 可以实现简单的数据流转换，而更复杂的数据流变换通常需要使用多个 Bolt 并通过多个步骤完成。
```

### 本地模式下运行拓扑的代码

```java
以ExclamationTopology为例：
Config conf = new Config();
conf.setDebug(true);	//设置日志信息，当为true时，每一个组件发射的每一个消息都让Storm记录到日志中
conf.setNumWorkers(2);//设置工作进程。

LocalCluster cluster = new LocalCluster();
cluster.submitTopology("test", conf, builder.createTopology());
Utils.sleep(1000);
cluster.killTopology("test");
cluster.shutdown();
```

### 集群模式下运行拓扑的代码

```java
Config conf = new Config();
conf.setNumworkers(20);
conf.setMaxSpoutPending(5000);
StormSubmitter.submitTopology("mytopology", conf, topology);
```

