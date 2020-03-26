现在人们写信比较少了，偶尔会寄明信片。在现在非常火爆的游戏《旅行青蛙》中，那只青蛙就会总是寄明信片回来。寄送明信片这件事中涉及到了三个角色：1）邮寄人员 2）派送人员（邮局系统）3）接受者。就这件事情映射设计模式或者架构模式，就是请求行为和实施行为的分离。在模式设计中归属与行为模式的，命令模式就是一个具体的例子。而在软件架构层面的话，选择一个合适的消息中间件这样的问题就会引刃而解了。

# 消息中间件的主要适用场景

多一个中间件会给系统运维带来一定的麻烦，需要考虑本本升级，资源分配，监控等等。整个软件整体的技术架构也相对而言会变得复杂一些，需要开发人员熟悉相应的API，以及一些错误处理。即使这样，消息中间不管在传统企业系统中，还是互联网的系统中都扮演着重要的角色。这样也反过来说明它存在的意义重大。

哪些场景可以考虑使用消息中间件来帮助我们处理一些问题，而不是我们自己来实施呢？

1） 这些任务可以拆分异步执行的一个个步骤。

虽然异步表示消息中间件的最主要特征，目前有很多的中间件也可以实现RPC，来实时同步调用，但是异步仍然是一个重要特点。

RPC这个特性只是在少数情况下，我们希望消息中间件给我们一个及时的回答。大部分情况还是上面举例的邮局的例子，我们发送出明信片之后，就交给邮局了。

程序是对现实生活的一个简单抽象，特别是面对对象的编程更是如此。现实生活中人们就很多使用异步和多任务来处理事情。我们那个年代的中学课本里面，有一篇陈景润的课文，告诉我们统筹学：“何为统筹法？举个简单的例子，一个忙碌的早上，一个人要做完烧水、刷牙洗脸、听广播、做早饭等几件事，怎么安排先后顺序最高效？这就是统筹法要解决的问题。统筹法是工程管理类的内容，并不算是数学。“  其实也就是一个多任务的处理方式。

编写和维护异步代码，包括并发安全和资源的共享，回收对一个初级程序员来说也不是一个简单的事情。如果使用消息中间件的代码默认就是异步之行的。

2）生产者和消费者的步调不一致

上面我们就提到生产者和消费者的解藕。但是为什么解藕呢，其中一个很重要的原因就是，消费者的功率不一样。有时候生产者比较高效，消费者跟不，有时候生产的高峰期过了之后消费者过剩，我们需要对消费者的资源回收，也就是减少消费者。

通过消息中间件的话，可以比较便利的对消费者或者生产者实施扩容或者资源回收。

包括当消费者短时间的下线的时候，可以消息中间件的存储对消息一定量的保留，当消费者再次上线的时候可以继续消费，从而不影响整个系统的使用。

比如在系统中，一些重要日志（访问日志，一些审计用的日志）的消费和使用就可以通过消息来实现中转，让写日志的服务压力小一些。还有那种非实时的收缴费用系统，可以把产生费用的系统和使用这些费用信息的财物系统等解藕，让财物系统有更加灵活的开发和部署方式。

3） 生产者可能增加不同的来源，或者可能需要增加额外的消费者。

在系统设计之初，我们可以就要考虑到一些系统的角色可能是要增加的。比如说对一些日志的消费，可能随着业务的发展需要增加不同的方式来分析和展示，这个是需要增加新的消费者。同时当我们增加新的集成渠道等等的的时候，我们需要把这些新的渠道的日志也纳入到我们统一到日志管理系统中来，也就是要增加新的生产者。这种情况下也非常适用用消息中间件。这个其实也引申出一个问题，可以供大家思考。也就是消息队列是由生产者决定的，还是消费者决定的？答案是视情况而定，取决于谁主导。当约定好一个消息之后，我们要看哪一方对这个消息有更强的主导权同时变化的可能性更小。先举一个简单的例子，一个生产系统当有新的产品发布的时候，通过消息的方式来告知。这种情况下很明显就是生产者来决定消息格式和队列，当新增加对新产品感兴趣的渠道时候，只要遵循这个消息队列的规约就可以轻松订阅和消费了。

而拿那个日志的例子来说明的话，消费者决定消息队列的可能性更高，因为新增加渠道也要往这个队列发布消息。

# 消息中间件的选择维度

选取消息中间件要从几个角度来考量。

1） 消息中间件的区别于其它的消息主要特点，可以所谓的软件基因。

这个可以从它的产生以及一些发展轨迹中得到。它产生的最初想法，后面的主要使用的行业和使用场景中获取到很多有效的信息。因为技术选型不可能一个个都进行深入的了解，先通过这些条件进行筛选。这些基因中的特点是否解决了这些使用的痛点。

2）具体的性能测试数据

在相同的配置情况下，安全性和稳定性相同的情况下，哪个表现更好无形中也给使用者降低的硬件成本。而且很有可能在信息量达到一定量的峰值的时候就是避免的不必要的扩容。

3）开源社区的热度以及在现有的技术体系下对开发人员的要求

社区热度比较高的中间件在易用性方面都有着不错的表现。比如spring integration模块已经对其进行了分装，让开发人员不必需要一开始的时候就需要知道过多的细节。让上手的学习曲线比较平稳的过度。

4）配置和运维的复杂度和安全性的要求

系统开发测试上线以后，更长的时间是运维人员在监控和维护。所以中间件的稳定性，易维护性刚开始的时候也要纳入考虑范围。不然后期之后，再改造的成本变的更大。

# 消息中间件中主要技术协议

1）JMS

通常而言提到JMS（java[** **](http://lib.csdn.net/base/java)MessageService）实际上是指JMS API。JMS是是市面上繁多的消息中间件之后，由Sun公司早期提出的消息标准，旨在为java应用提供统一的消息操作，包括create、send、receive等从而降低开发开发人员的难度以及在技术切换的时候提供一定程度的便利。JMS已经成为Java Enterprise Edition的一部分。从使用角度看，JMS和JDBC担任差不多的角色，用户都是根据相应的接口可以和实现了JMS的服务进行通信，进行相关的操作。JMS的实现消息中间件中使用比较广泛的有activeMQ等。

JMS通常包含如下一些角色：

| Elements | Notes |
| :--- | :--- |
| JMS provider | 实现了JMS接口的消息中间件，如ActiveMQ |
| JMS client | 生产或者消费消息的应用（使用基础jar包，配置和开发的程序） |
| JMS producer/publisher | JMS消息生产者 |
| JMS consumer/subscriber | JMS消息消费者 |
| JMS message | 消息，在各个JMS client传输的对象； |
| JMS queue | Provider存放等待被消费的消息的地方（就是通过这个中间站，来实现生产和消费的解藕） |
| JMS topic | 一种提供多个订阅者消费消息的一种机制；在MQ中常常被提到，topic模式。 |

JMS同样提供了两种消息模型，peer-2-peer\(点对点\)以及publish-subscribe（发布订阅）模型。

当采用点对点模型时，消息将发送到一个队列，该队列的消息只能被一个消费者消费。但是可能会有多个接受者在一个队列中侦听，但是每个队列中的消息只能被队列中的一个接受者消费。消息存在先后顺序，一个队列会按照消息服务器将消息放入队列中的顺序，要在多个JMS客户端之间均衡消息处理的负载，把它们传送给消费者当消息已被消费时，就会从队列头部将它们删除。点对点模型的另一优点就是，它所提供的QueueBrowser允许JMS客户端对队列进行快照（Snapshot）,以查看正在等待被消费的消息。

同时点对点消息传送模型有两种类型：

**异步即发即弃（fire-and-forget）处理**和**异步请求/应答处理**

使用即发即弃处理时，消息生产者向某个队列发送一条消息，而且它并不会期望接受到一个响应（至少不是立刻接收到响应）。这类处理可用于触发一个事件，或者用于向接受者发出请求来执行一个并不需要响应的特定活动。

使用异步请求/应答处理时，消息生产者向队里发送一条消息，然后阻塞等待（blocking wait）应答队列，该应答队列正在等待来自接受者的响应。请求/应答处理实现了生产者和消费者之间的高度去耦，允许消息生产者和消费者组件采用不同的语言或平台。

当采用发布订阅模型时，消息可以被多个消费者消费，每个消费者都能消费这个消息，也不存在同步等待的模式了。

2） AMPQ协议

AMQP（advanced message queuing protocol）在2003年时被提出，最早用于解决金融领不同平台之间的消息传递交互问题。顾名思义，AMQP是一种协议，更准确的说是一种binary wire-level protocol（链接协议）。这是其和JMS的本质差别，AMQP不从API层进行限定，而是直接定义网络交换的数据格式。这使得实现了AMQP的provider天然性就是跨平台的。意味着我们可以使用Java的AMQP provider，同时使用一个C\#的producer加一个rubby的consumer。从这一点看，AQMP可以用http来进行类比，不关心实现的语言，只要大家都按照相应的数据格式去发送报文请求，不同语言的client均可以和不同语言的server链接。这个以后的技术架构扩展带来更大的灵活性。

在AMQP中，消息路由（messagerouting）和JMS存在一些差别，在AMQP中增加了Exchange和binding的角色。producer将消息发送给Exchange，binding决定Exchange的消息应该发送到那个queue，而consumer直接从queue中消费消息。queue和exchange的bind有consumer来决定。AMQP的routing scheme图示过程如下：

![](/assets/import.png)

目前AMQP逐渐成为消息队列的一个标准协议，当前比较流行的rabbitmq、stormmq都使用了AMQP实现。

最后将JMS和AMQP的各项对比如下：

|  | JMS | AMQP |
| :--- | :--- | :--- |
| 定义 | Java api | Wire-protocol |
| 跨语言 | 否 | 是 |
| 跨平台 | 否 | 是 |
| Model | 提供两种消息模型：（1）Peer-2-Peer（2）Pub/sub | 提供了五种消息模型：（1）direct exchange（2）fanout exchange（3）topic change（4）headers exchange（5）system exchange本质来讲，后四种和JMS的pub/sub模型没有太大差别，仅是在路由机制上做了更详细的划分； |
| 支持消息类型 | 多种消息类型：TextMessage，MapMessage，BytesMessage，StreamMessage，ObjectMessage，Message （只有消息头和属性） | byte\[\]当实际应用时，有复杂的消息，可以将消息序列化后发送。 |
| 综合评价 | JMS 定义了JAVA API层面的标准；在java体系中，多个client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差； | AMQP定义了wire-level层的协议标准；天然具有跨平台、跨语言特性。 |

3） 其它自定义传输协议的消息中间件

主要是基于自身对性能或者安全有更高的要求。下面以kafka的协议举例：

Kafka的Producer、Broker和Consumer之间采用的是一套自行设计的基于TCP层的协议。Kafka的这套协议完全是为了Kafka自身的业务需求而定制的，而非要实现一套类似于Protocol Buffer的通用协议。本文将介绍这套协议的相关内容。

基本数据类型

1. 定长数据类型：int8,int16,int32和int64，对应到Java中就是byte, short, int和long。
2. 变长数据类型：bytes和string。变长的数据类型由两部分组成，分别是一个有符号整数N\(表示内容的长度\)和N个字节的内容。其中，N为-1表示内容为null。bytes的长度由int32表示，string的长度由int16表示。
3. 数组：数组由两部分组成，分别是一个由int32类型的数字表示的数组长度N和N个元素。

Request和Response的基本结构

Kafka中两个角色之间通讯的基本单位是Request/Response，Request和Response的基本结构如下：

```
RequestOrResponse => MessageSize (RequestMessage | ResponseMessage)
```

其中各字段的含义为：

| 名称 | 类型 | 描述 |
| :--- | :--- | :--- |
| MessageSize | int32 | 表示RequestMessage或者ResponseMessage的长度 |
| RequestMessage/ResponseMessage | - | 表示Request或者Response的内容，在下面将会介绍其具体格式。 |

这个结构定义了通讯双方交换数据的基本结构。通讯的过程可以简单地表示为：客户端打开与服务器端的Socket，然后往Socket写入一个int32的数字表示这次发送的Request有多少字节，然后继续往Socket中写入对应字节数的数据。服务器端先读出一个int32的整数从而获取这次Request的大小，然后读取对应字节数的数据从而得到Request的具体内容。服务器端处理了请求后，也用同样的方式来发送响应。

# 具体中间件大比拼

上面介绍了消息中间件主流的规范和协议。接下来就一个个具体消息中间件来介绍。

主要用上文提到的比较有代表性的ActiveMQ，RabbitMQ，kafka来做比较。

|  | ActiveMq | RabbitMq | Kafka |
| :--- | :--- | :--- | :--- |
| **producer容错，是否会丢数据** | 有ack模型 | 有ack模型，也有事务模型，保证至少不会丢数据。ack模型可能会有重复消息，事务模型则保证完全一致 | 批量形式下，可能会丢数据。 非批量形式下， 1. 使用同步模式，可能会有重复数据。 2. 异步模式，则可能会丢数据。 |
| 消息持久化 | 支持 | 支持 |  |
| **consumer容错，是否会丢数据** | 有ack模型，数据不会丢，但可能会重复处理数据。 | 有ack模型，数据不会丢，但可能会重复处理数据。 | 批量形式下，可能会丢数据。非批量形式下，可能会重复处理数据。（ZK写offset是异步的） |
| **架构模型** | 基于JMS规范，基于STOP传输协议 | 基于AMQP模型，比较成熟，但更新超慢。RabbitMQ的broker由Exchange,Binding,queue组成，其中exchange和binding组成了消息的路由键；客户端Producer通过连接channel和server进行通信，Consumer从queue获取消息进行消费（长连接，queue有消息会推送到consumer端，consumer循环从输入流读取数据）。rabbitMQ以broker为中心；有消息的确认机制 | producer，broker，consumer，以consumer为中心，消息的消费信息保存的客户端consumer上，consumer根据消费的点，从broker上批量pull数据；无消息确认机制。 |
| **吞吐量** | activeMQ性能表现要差于rabbitMQ和kafka | rabbitMQ在吞吐量方面稍逊于kafka，他们的出发点不一样，rabbitMQ支持对消息的可靠的传递，支持事务，不支持批量的操作；基于存储的可靠性的要求存储可以采用内存或者硬盘。 | kafka具有高的吞吐量，内部采用消息的批量处理，zero-copy机制，数据的存储和获取是本地磁盘顺序批量操作，具有O\(1\)的复杂度，消息处理的效率很高 |
| **可用性** |  | rabbitMQ支持miror的queue，主queue失效，miror queue接管 | kafka的broker支持主备模式 |
| **集群负载均衡** |  | rabbitMQ的负载均衡需要单独的loadbalancer进行支持 | kafka采用zookeeper对集群中的broker、consumer进行管理，可以注册topic到zookeeper上；通过zookeeper的协调机制，producer保存对应topic的broker信息，可以随机或者轮询发送到broker上；并且producer可以基于语义指定分片，消息发送到broker的某分片上 |

总结下来它有很长的历史，而且被广泛的使用。它还是跨平台的，给那些非微软平台的产品提供了一个天然的集成接入点。

rabbitMQ是为了满足金融行业用Erlang写成的消息中间件的优秀的特性。它支持开放的高级消息队列协议\(AMQP，Advanced Message Queuing Protocol\)，从根本上避免了生产厂商的封闭，使用任何语言的各种客户都可以从中受益。这种协议提供了相当复杂的消息传输模式，所以基本上不需要 MassTransit或NServiceBus的配合。它还具有“企业级”的适应性和稳定性。这些东西对我的客户来说十分的有吸引力。在事务支持和性能上都有不俗的表现。

Kafka设计的初衷就是处理日志的，可以看做是一个日志系统，针对性很强，所以它并没有具备一个成熟MQ应该具备的特性。Kafka的性能（吞吐量、tps）比RabbitMq要强，但是事务性支持上有所欠缺。

# 基于RabbitMQ开发

通过上述分析，在不同的情况下根据自己的判断，可以选择出一款适合自己团队和业务场景的消息中间件。

笔者在系统重构的时候，考虑到集群系统的维护和性能要求，把系统中的activeMQ替换成了RabbitMQ。

接下来就主要按照RabbitMQ来分享一些自己使用心得。

## 1）安装和维护

RabbitMQ的安装非常简单。但是因为rabbitMQ是为了利用erlang的编写的，在安装软件之前先要安装一下Erlang的语言安装包。Erlang不需要有多了解，只要把它和java编程语言一样，也是一个提供运行虚拟机就好。

一般本地开发比较多的还是windows环境或者mac，一般不需要安装集群或者负载均衡，所以也把windows下的单点安装和linux下的集群安装分别简单介绍一下。

### windows下的单机版安装

a）根据自己操作系统信息选择合适的Erlang语言安装包和rabbit broker的版本安装文件来下载。我自己安装的是如下两个文件。

```
     otp\_win64\_20.0.exe （erlang语言安装包）

     rabbitmq-server-3.6.10.exe（rabbit broker安装包）
```

b）安装

```
  根据提示，首先安装第一个erlang语言安装包，然后接着安装**rabbit broker**安装包。
```

建议安装在启动盘，因为设计到一些文件的权限的读写。而且安装了rabbit broker

安装包之后，默认会建立一个windows的服务。这样操作系统重启之后，服务也能正常运行，不需要手动的再次去启动erlang的虚拟机和消息服务。给开发时候，省去了很多麻烦。

c）插件管理

```
  为了使用rabbit MQ的插件，这里主要是为了使用UI插件。
```

到安装目录的bin目录下运行如下脚本

./rabbitmq-plugins enable rabbitmq\_management

d）登陆web控制台

```
 通过上面的三步，就可以通过web控制台的方式来管理交换机，消息队列等资源。同时也
```

可以在页面进行消息的发送。账户管理，权限分配以及监控消息的消费速度等都可以集中管理。

默认的信息如下：

\[[http://localhost:15672/\#/exchanges\]\(](http://localhost:15672/#/exchanges]%28)"\)

用户名/密码：guest/guest

### Linux的rabbitMQ的集群安装

1）根据操作系统信息选择erlang语言安装包和rabbitMQ下载

以下是我们选择的版本：

erlang-19.0.4-1.el7.centos.x86\_64.rpm

rabbitmq-server-3.6.11-1.el7.noarch.rpm

2）安装

把这两个文件放到一个目录下，通过rpm –ivh \*.rpm命令依次安装

3）环境准备以及规划

rabbitMQ里面的交换机，队列，以及路由信息包括消息都是可以持久化到磁盘中的。

所以在安装之前就要选择一些节点的工作模式是Disk或者RAM。

Disk模式的是可以存储的，但是速度较慢。Ram不存储处理速度较快。

同时为了备份，一般要有多个Disk的节点这样一个节点崩溃之后，依然可以保存信息。

所以我们准备三台这样的服务器信息：

| RabbitMQ节点 | IP地址 | 工作模式 | 操作系统 |
| :--- | :--- | :--- | :--- |
| rabbitmqcluster | 172.25.17.228 | DISK | CentOS 7.2 - 64位 |
| rabbitmqcluster01 | 172.25.17.229 | DISK | CentOS 7.2 - 64位 |
| rabbitmqcluster02 | 172.25.17.230 | RAM | CentOS 7.2 - 64位 |

4）cluster安装

a）局域网配置

分别在三个节点的/etc/hosts下设置相同的配置信息

172.25.17.228 rabbitmqcluster

172.25.17.229 rabbitmqcluster01

172.25.17.230 rabbitmqcluster02

同时需要修改三台服务器主机名hostnamectl set-hostname rabbitmqcluster

b）设置不同节点间同一认证的**Erlang Cookie**

采用从主节点copy的方式保持Cookie的一致性

/var/lib/rabbitmq/.erlang.cookie

注意：这个是隐藏文件无法scp，只能手动vi，需要注意该文件权限400

而且先要把第一台服务器rabbit服务起来，不然是不生成/.erlang.cookie文件的，copy完毕需要重启

c）使用**-detached**运行各节点

rabbitmqctl stop

rabbitmq-server –detached

d）查看各节点的状态

\[root@rabbitmqcluster\]\#rabbitmqctl cluster\_status

\[root@rabbitmqcluster01\]\#rabbitmqctl cluster\_status

\[root@rabbitmqcluster02\]\#rabbitmqctl cluster\_status

e）创建并部署集群

以rabbitmqcluster节点为例：

\[root@ rabbitmqcluster01\]\#rabbitmqctl stop\_app

\[root@ rabbitmqcluster01\]\#rabbitmqctl join\_cluster rabbit@ rabbitmqcluster

\[root@ rabbitmqcluster01\]\#rabbitmqctl start\_app

f）查看集群状态

\[root@ rabbitmqcluster\]\#rabbitmqctl cluster\_status

g）更改节点类型（内存型或磁盘型）

rabbitmqctl stop\_app  
 rabbitmqctl change\_cluster\_node\_type disc  
或  
 rabbitmqctl change\_cluster\_node\_type ram  
 rabbitmqctl start\_app

h）启动**web**管理

rabbitmq-plugins enable rabbitmq\_management

rabbitmqctl add\_user admin admin

rabbitmqctl set\_user\_tags admin administrator

i） RabbitMQ负载均衡配置

RabbitMQ的分布式通信框架主要包括两部分内容，

一是要确保可用性和性能，

另一个就是当节点发生故障时知道如何重连到集群的应用程序。

负载均衡就是解决处理节点的选择问题，对应用程序对集群具体信息透明化同时支持高可用。

i1）安装HAProxy

选择开源的HAProxy为RabbitMQ集群做负载均衡器，在linux中安装HAProxy。

安装peel：

rpm -ivh \[[http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-5.noarch.rpm//\]\(](http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-5.noarch.rpm//]%28)"\)

安装HAProxy：

yum -y install haproxy

i2）配置HAProxy

cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak

vim /etc/haproxy/haproxy.cfg

添加如下信息：

//前段IP供product和consumer来进行选择。5672端口已经使用，这里选择5670端口

listen rabbitmq\_local\_cluster 127.0.0.1:5670

//负载均衡选项

mode tcp

//轮询算法将负载发给后台服务器

balance roundrobin

//负载均衡中的集群节点配置，这里选择的rabbit节点

server rabbit 127.0.0.1:5672 check inter 5000 rise 2 fall 3

listen private\_monitoring :8100

mode http

option httplog

stats enable

stats uri  /stats

stats refresh 60s

i3）启动HAProxy通过查看信息

service haproxy restart

访问地址：[http://172.25.17.228:8100/stats](http://172.25.17.228:8100/stats)

### web控制台的简单使用介绍

![](/assets/rabbitMQWebConsole.png)

tabs：

overview：

a）整理展示消息的入队与出队的速度。也就是消息的生产和消费速度。

b）节点信息。也包括交换机，队列，渠道，消费者的统计数。

c）节点的端口信息以及一些配置信息上下文信息。

d）这些节点数据的导入导出

Connection：

所有的连接信息。客户端的IP和端口，以及连接的节点信息以及渠道数量。同时包括了一些连接用户和速度等。

Channel：

所有的连接渠道信息。渠道名称，具体渠道的消息速度信息。

客户端和消息中间件建立连接之后，要建立一个专属渠道才能正常的收发信息。

引入渠道的概念，主要是为了能过减少TCP级别连接的生成和销毁。通过能够更加细粒度地控制网络连接资源。

Exchange：

交换机信息。发送信息首先到达的地方。同时也有消息的进出速度。

Queues：

队列信息。队列的名称以及一些属性：是否是持久化的，是否是自动删除的等，还有队列层级的速度。

点击一个队列名称，就可以在页面往队列中发送消息来测试。

Admin：

用户和权限的管理。

## 2）参数设置

Spring的RabbitMQTemplate给使用带来了极大的简化。通过一些简单的配置信息，就可以利用rabbitMQTemplate来为我们服务。这些配置信息大致可以归纳成三种。

这些配置可以在

org.springframework.boot.autoconfigure.amqp.RabbitProperties中一一查看。

1）在web控制台的那些内容在应用程序都可以做相应的配置。

消息中间件的主机信息:

spring.rabbitmq.addresses

spring.rabbitmq.password

spring.rabbitmq.username

创建交换机，队列的配置信息:

@Bean  
 @ConditionalOnProperty\(value = "need.app.queue", havingValue = "true"\)  
 public Queue appQueue\(\) {  
 return new Queue\(applicationName, true\);  
 }

@Bean  
 @ConditionalOnProperty\(value = "need.app.queue", havingValue = "true"\)  
 DirectExchange exchange\(\) {  
 return new DirectExchange\(applicationName, true =&gt;durable, false =&gt; autoDelete\);  
 }

2）spring integration对rabbitMQ提供了一些封装之后对属性配置

比如：确认回调方法和返回回调函数等。

@Bean\("ebaoRabbitTemplate"\)  
 public RabbitTemplate rabbitTemplate\(ConnectionFactory connectionFactory\) {  
 RabbitTemplate rabbitTemplate = new RabbitTemplate\(\);  
 \(\(CachingConnectionFactory\) connectionFactory\).setPublisherReturns\(true\);  
 \(\(CachingConnectionFactory\) connectionFactory\).setPublisherConfirms\(true\);  
 rabbitTemplate.setConnectionFactory\(connectionFactory\);  
_** rabbitTemplate.setMandatory\(true\);  
 rabbitTemplate.setChannelTransacted\(false\);**_  
 rabbitTemplate.setMessageConverter\(new EbaoJackson2JsonMessageConverter\(\)\);

rabbitTemplate.setConfirmCallback\(new RabbitTemplate.ConfirmCallback\(\) {  
 @Override  
 _**public void confirm\(CorrelationData correlationData, boolean ack,  
 String cause\) **_{  
LOGGER.debug\("correlationData:{}", correlationData\);  
LOGGER.debug\("ack:{}", ack\);  
LOGGER.debug\("cause:{}", cause\);  
 if \(correlationData == null \|\| correlationData.getId\(\) == null\) {  
 return;  
 }

//TODO 消息发送失败的处理  
}

rabbitTemplate.setReturnCallback\(new RabbitTemplate.ReturnCallback\(\) {  
_** @Override  
 public void returnedMessage\(Message message, int replyCode, String replyText,  
 String exchange, String routingKey\) **_{  
LOGGER.debug\("message:{}", message\);  
LOGGER.debug\("replyCode:{}", replyCode\);  
LOGGER.debug\("replyText:{}", replyText\);  
LOGGER.debug\("exchange:{}", exchange\);  
LOGGER.debug\("routingKey:{}", routingKey\);  
 }  
 }\);  
 return rabbitTemplate;  
 }

3）spring integration 自身为了优化或者便利提供的一些配置。

channel的checkTimeOut等.

spring.rabbitmq.channel.check-timeout

## 3）队列的规划

在前面简单分析过队列是应该生产者还是消费者决定的一些分析。rabbitMQ为了解决这个问题提出了它的解决之道，就是在保持消息队列的简单性，提出exchange交换机。

简单介绍一下RabbitMQ的交换机和队列已经路由。

exchange在和queue进行binding时会设置**routingkey，**有**如下**4个类型：

direct，topic，fanout，header。

1）direct exchange

在direct类型的exchange中，只有这两个routingkey完全相同，exchange才会选择对应的binging进行消息路由。

这样效率相对较高。

2）topic exchange

和direct类似，但是routingkey可以有通配符：'\*','\#'.

其中'\*'表示匹配一个单词，'\#'则表示匹配没有或者多个单词。

这样可以实现多对一的发送，当增加一个消息生产者，只要用满足表达式的routerKey去发送，就可以到达队列。

3）fanout exchange

路由规则很简单直接将消息路由到所有绑定的队列中，无须对消息的routingkey进行匹配操作。

这样可以简单的实现，一个消息发送到多个队列的情况。

也就是由生产者来决定队列的。

在广播的消息一般就是用fanout的exchange，这样新增加一个消费者就可以直接消费。

在分布式缓存的时候，就是使用广播的消息来实现ehcache的同步问题。

包括配置中心的变更等，也是通过广播的方式来实现的。

4）header exchange

路由的规则是根据header里面的参数来判断。

Dictionary&lt;string,object&gt; aHeader =newDictionary&lt;string,object&gt;\(\);

aHeader.Add\("format",“json””;

aHeader.Add\("type",“text”\);

aHeader.Add\("x-match","all"\);

channel.QueueBind\(queue:“test”,

exchange:"agreements",

routingKey:string.Empty,

arguments: aHeader\);

其中的x-match为特殊的header，可以为all则表示要匹配所有的header，如果为any则表示只要匹配其中的一个header即可。

在发布消息的时候就需要传入header值：

varproperties =channel.CreateBasicProperties\(\);

properties.Persistent=true;

Dictionary&lt;string,object&gt; mHeader1 =newDictionary&lt;string,object&gt;\(\);

mHeader1.Add\("format",“json””;

mHeader1.Add\("type",“text”\);

properties.Headers= mHeader1;

介绍完了exchange，我们来回到队列的规划问题上来。在之前我们的系统中，随着集成点的增加队列来处理的越来越受到大家青睐。

通过上述的exchange来解决大部分问题，但是还是有一些情况导致大家新增加很多队列。而且队列没有同一的管理，最后系统想替换

消息中间件或者系统要重构的时候就带来很大的麻烦。

所以在使用合理的exchange的情况下，对队列做一个规划还是很有必要的。笔者在项目中，就是通过建立和微服务同名的队列提供出来

给生产者。然后通过message来选择合适的消费者handler来处理。这样队列也作为一个有效的资源被管理起来。

## 4）消息的确认模式与事务

有很多框架通过消息来做分布式事物的替代方案，这也说明了消息中间件的稳定性和可靠性。

但是既然是分布式事物，自然有一些特殊情况是会不一致的，这个就要根据CAP理论进行取舍。这里是介绍rabbitMQ的机制，保证数据的最终一致性。

1）消息的一致性

RabbitMQ支持本地事务，但是不支持JTA事务或XA事务。

RabbitTransactionManager：

\* This local strategy is an alternative to executing Rabbit operations within, and synchronized with, external

\* transactions. This strategy is &lt;i&gt;not&lt;/i&gt; able to provide XA transactions, for example in order to share transactions

\* between messaging and database access.

RabbitMQ中与事务机制有关的方法有三个：txSelect\(\), txCommit\(\)以及txRollback\(\), txSelect用于将当前channel设置成transaction模式，txCommit用于提交事务，txRollback用于回滚事务，在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定到达了broker了，如果在txCommit执行之前broker异常崩溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了。

  
AMQP事物局限和一些相关说明：

a）AMQP只是对消息发送，和消息确认有事务。对于exchange，queue，route等的创建，删除，修改等都是没有事务的。

b）消息的确认事务和消息的消费事务不是同一个，是分开管理。所以消息消费方可以在后面的事务中reject消息，即使消息确认的消息已经发送。

c）AMQP只能对单个消息队列的消息有事务保护作用，也就是如果route对应的有多个，或者广播的时候，都没有事务。

d）AMQP对消息的发送和确认都是异步的。

e）RabbitMQ是通过commit-ok的时候，才确认消息能客户端消费，或者持续化到磁盘。同时，只是表示消息被接受到了，并不是被消费掉了。服务器端可能失败复活确认信息，所以客户端可能二次消费到相同的信息。



同时官方也明确说明了如果使用事物的话，消息的处理速度会降低很多。官方给出来的性能描述是：

It takes a bit more than 4 minutes to publish 10000 messages.

异步的方法的效率是事务方法效率的100倍，官方的描述是：

It takes a bit more than 2 seconds to publish 10000 messages.

发送者发送消息出来，在数据一致性的要求下，我们通常认为必须达到以下条件

1. broker持久化消息

2. publisher知道消息已经成功持久化

首先，我们可以采用事务来解决此问题。每个消息都必须经历以上两个步骤，就算一次事务成功。

事务是同步的。因此，如果采用事务，发送性能必然很差，因为发送完成后要等到broker

持久化之后才能释放这个channel，进行下一次的消息放送。

采用异步的方式来解决此问题。publisher发送消息后，不进行等待，而是异步监听是否成功。这种方式又分为两种模式，

一种是return，

另一种是confirm.

调用回调有如下几种情况：

* broker 发现当前消息无法被路由到指定的 queues 中（如果设置了  
  mandatory 属性，则 broker 会先发送basic.return）

* 非持久属性的消息到达了其所应该到达的所有 queue 中（和镜像 queue 中）

* 持久消息到达了其所应该到达的所有 queue 中（和镜像 queue 中），并被持久化到了磁盘（被 fsync）

* 持久消息从其所在的所有 queue 中被 consume 了

![](/assets/messageCallBackProcess.png)

总结一下  
**confirm** 主要是用来判断消息是否有正确到达交换机，如果有，那么就 **ack**就返回 

**true**如果没有，则是 **false**

**return**则表示如果你的消息已经正确到达交换机但是后续处理出错了（比如没有达到队列），那么就会回调**return**

，并且把信息送回给你（前提是需要设置了 **Mandatory**，不设置那么就丢弃）；如果消息没有到达交换机，那么不会调用 **return。**



AMQP协议mandatory和immediate标志位的作用：

mandatory和immediate是AMQP协议中basic.pulish方法中的两个标志位，它们都有当消息传递过程中不可达目的地时将消息返回给生产者的功能。具体区别在于：

1. mandatory标志位

当mandatory标志位设置为true时，如果exchange根据自身类型和消息routeKey无法找到一个符合条件的queue，那么会调用basic.return方法将消息返还给生产者；当mandatory设为false时，出现上述情形broker会直接将消息扔掉。

1. immediate标志位

当immediate标志位设置为true时，如果exchange在将消息route到queue\(s\)时发现对应的queue上没有消费者，那么这条消息不会放入队列中。当与消息routeKey关联的所有queue\(一个或多个\)都没有消费者时，该消息会通过basic.return方法返还给生产者。

这时，对于publisher来说，只能重发消息解决问题。而在这里面，我们会发生 重复消息的问题。如果业务类型要求数据一致性非常高，可以采用低效率的事务型解决方案。

