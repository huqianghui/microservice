a）CAP理论的使用

在分布式系统中设计到状态的时候，都要考虑CAP相关的东西。在微服务的应用中，我们都说是无状态的。只把状态保存在数据库，缓存中。通过关系型数据库的自带事务机制和事务隔离级别来实现CAP的选择，而在缓存的设计中通过消息中间件来实现事务最终一致性。除此之外的非关系型数据库mongoDB，ES以及服务注册管理中的状态等都要依据CAP的了解和指导来做一些选择和配置。

接着Eureka的选择说明一下CAP相关知识。

CAP原则又称CAP定理，指的是在一个分布式系统中， Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可得兼。

分布式系统的CAP理论：理论首先把分布式系统中的三个特性进行了如下归纳：

* 一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
* 可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
* 分区容忍性（P）：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

CAP理论就是说在分布式存储系统中，最多只能实现上面的两点。而由于当前的网络硬件肯定会出现延迟丢包等问题，所以分区容忍性是我们必须需要实现的。所以我们只能在一致性和可用性之间进行权衡。

### BASE理论

BASE是Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）三个短语的简写，BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的结论，是基于CAP定理逐步演化而来的，其核心思想是即使无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。接下来我们着重对BASE中的三要素进行详细讲解。

#### 基本可用

基本可用是指分布式系统在出现不可预知故障的时候，允许损失部分可用性——但请注意，这绝不等价于系统不可用，以下两个就是“基本可用”的典型例子。

* 响应时间上的损失：正常情况下，一个在线搜索引擎需要0.5秒内返回给用户相应的查询结果，但由于出现异常（比如系统部分机房发生断电或断网故障），查询结果的响应时间增加到了1~2秒。
* 功能上的损失：正常情况下，在一个电子商务网站上进行购物，消费者几乎能够顺利地完成每一笔订单，但是在一些节日大促购物高峰的时候，由于消费者的购物行为激增，为了保护购物系统的稳定性，部分消费者可能会被引导到一个降级页面。

弱状态也称为软状态，和硬状态相对，是指允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同节点的数据副本之间进行数据听不的过程存在延时。

#### 最终一致性

最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性

亚马逊首席技术官Werner Vogels在于2008年发表的一篇文章中对最终一致性进行了非常详细的介绍。他认为最终一致性时一种特殊的弱一致性：系统能够保证在没有其他新的更新操作的情况下，数据最终一定能够达到一致的状态，因此所有客户端对系统的数据访问都能够获取到最新的值。同时，在没有发生故障的前提下，数据达到一致状态的时间延迟，取决于网络延迟，系统负载和数据复制方案设计等因素。

在实际工程实践中，最终一致性存在以下五类主要变种。

因果一致性：

```
    因果一致性是指，如果进程A在更新完某个数据项后通知了进程B，那么进程B之后对该数据项的访问都应该能够获取到进程A更新后的最新值，并且如果进程B要对该数据项进行更新操作的话，务必基于进程A更新后的最新值，即不能发生丢失更新情况。与此同时，与进程A无因果关系的进程C的数据访问则没有这样的限制。
```

读己之所写：

```
    读己之所写是指，进程A更新一个数据项之后，它自己总是能够访问到更新过的最新值，而不会看到旧值。也就是说，对于单个数据获取者而言，其读取到的数据一定不会比自己上次写入的值旧。因此，读己之所写也可以看作是一种特殊的因果一致性。
```

会话一致性：

```
    会话一致性将对系统数据的访问过程框定在了一个会话当中：系统能保证在同一个有效的会话中实现“读己之所写”的一致性，也就是说，执行更新操作之后，客户端能够在同一个会话中始终读取到该数据项的最新值。
```

单调读一致性：

```
    单调读一致性是指如果一个进程从系统中读取出一个数据项的某个值后，那么系统对于该进程后续的任何数据访问都不应该返回更旧的值。
```

单调写一致性：

```
     单调写一致性是指，一个系统需要能够保证来自同一个进程的写操作被顺序地执行。
```

以上就是最终一致性的五类常见的变种，在时间系统实践中，可以将其中的若干个变种互相结合起来，以构建一个具有最终一致性的分布式系统。事实上，可以将其中的若干个变种相互结合起来，以构建一个具有最终一致性特性的分布式系统。事实上，最终一致性并不是只有那些大型分布式系统才设计的特性，许多现代的关系型数据库都采用了最终一致性模型。在现代关系型数据库中，大多都会采用同步和异步方式来实现主备数据复制技术。在同步方式中，数据的复制国耻鞥通常是更新事务的一部分，因此在事务完成后，主备数据库的数据就会达到一致。而在异步方式中，备库的更新往往存在延时，这取决于事务日志在主备数据库之间传输的时间长短，如果传输时间过长或者甚至在日志传输过程中出现异常导致无法及时将事务应用到备库上，那么狠显然，从备库中读取的的数据将是旧的，因此就出现了不一致的情况。当然，无论是采用多次重试还是认为数据订正，关系型数据库还是能搞保证最终数据达到一致——这就是系统提供最终一致性保证的经典案例。

总的来说，BASE理论面向的是大型高可用可扩展的分布式系统，和传统事务的ACID特性使相反的，它完全不同于ACID的强一致性模型，而是提出通过牺牲强一致性来获得可用性，并允许数据在一段时间内是不一致的，但最终达到一致状态。但同时，在实际的分布式场景中，不同业务单元和组件对数据一致性的要求是不同的，因此在具体的分布式系统架构设计过程中，ACID特性与BASE理论往往又会结合在一起使用。

可以参照关系型数据库中的事物隔离级别来对比。

数据库隔离级别是关系型数据库保障事务生命周期过程中几个层次的数据安全策略。

隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大；而隔离级别越低，事务并发性越强，但同时会出现与业务情况相背的数据情况。如果系统压力偏大而业务数据能支持较强的容错情况，我们可以选择相对较低的事务隔离级别，在个别应用场景可以由应用程序采用悲观锁或乐观锁来控制低级别事务隔离级别引出的数据安全情况；而数据一致性需求较强的应用场景，如果系统压力并不大且在一个可控的压力范围之内，我们可以选择较高的事务隔离级别支持，甚至可以选择最高的串行执行的隔离级别Serializable。

事务完整性

事务完整性可以用ACID特性来衡量事务的质量，违反事务完整性的问题有3类：脏读、不可重复读和幻像读。

ACID：原子性\(atomicity\)，一致性\(consistency\)，隔离性\(isolation\)，持续性（durability）

原子性：事务是原子的，要不全部完成，要不全部失败。

一致性：保证数据库的一致性，事务执行之前是一致的，执行之后也是一致的。

隔离性：每个事务必须和其他事务产生的结果隔离开来。

持续性：事务的持续性指不管系统发生了什么故障，事务处理的结果都是永久的，数据库产品必须保证，即使存放数据的驱动器融化了，也能将数据库恢复到驱动器融化之前，即最后一个事务提交的瞬间状态。

事务缺陷：脏读、不可重复读、幻像读

1）脏读

读取了其它事务未提交的数据。

2）不可重复读

对于单行数据来说。事务A两次读取的数据不一致，因为看到的是事务B提交前后的数据，自然不一致。

3）幻像读

对于多行数据来说。事务A两次读取的行数不一致，因为在其过程中正巧事务B修改并提交了满足事务A查询条件的记录。

几种隔离级别

以JDBC中Connection的事务隔离级别定义来说明：

1）Read Uncommitted（TRANSACTION\_READ\_UNCOMMITTED）

事务A可以读取到事务B未提交的数据，如果事务B发生异常回滚，事务A读取到的将是脏数据。此事务隔离级别的应用场景中将出现脏读、不可重复读和幻像读这几种情况。

2）Read Committed（TRANSACTION\_READ\_COMMITTED）

事务A可以读取到事务B已经提交的数据，不可读取到事务B未提交的数据，避免了脏读，但没有避免不可重复读和幻像读。

3）Repeatable Read（TRANSACTION\_REPEATABLE\_READ）

可以回避脏读、不可重复读的情况，但无法避免幻像读。

4）Serializable（TRANSACTION\_SERIALIZABLE）

防止所有的事务缺陷，适用于绝对的事务完整性的要求比性能更为重要的情况，银行帐务系统、高度竞争性的销售数据库，等等，可以选择使用Serializable。

b）Eureka ，consul ，zookeeper的比较

c）三个主要的方法

d）zone的使用

e）eureka client的使用
