负载均衡是我们处理高并发、缓解网络压力和进行服务端扩容的重要手段之一，但是一般情况下我们所说的负载均衡通常都是指服务端负载均衡，服务端负载均衡又分为两种，一种是硬件负载均衡，还有一种是软件负载均衡。

负载均衡\(Load Balance\)同时也是集群技术（Cluster）的一种应用。负载均衡可以将工作任务分摊到多个处理单元，从而提高并发处理能力.

服务器集群\(Cluster\)使得多个服务器节点能够协同工作，根据目的的不同，服务器集群可以分为：

a\)高性能集群：将单个重负载的请求分散到多个节点进行处理，最后再将处理结果进行汇总。一般职责和角色不同。

b\)高可用集群：提高冗余单元，避免单点故障。只有master宕机的时候，对应的节点来接管相应的工作。

c\)负载均衡集群：将大量的并发请求分担到多个处理节点。由于单个处理节点的故障不影响整个服务，负载均衡集群同时也实现了高可用性。处理同样的事情，而且大家都一起共同处理。

一般提到的负载均衡\(Load Balance\)，是指实现负载均衡集群。负载均衡实现了横向扩展（Scale Out），避免纵向的升级（Scale Up）换代。

服务器负载均衡基本原理

任何的负载均衡技术都要想办法建立某种一对多的映射机制：一个请求的入口映射到多个处理请求的节点，从而实现分而治之（Divide and Conquer）。

这种映射机制使得多个物理存在对外体现为一个虚拟的整体，对服务的请求者屏蔽了内部的结构。

采用不同的机制建立映射关系，可以形成不同的负载均衡技术，常见的包括：

a）DNS轮询

DNS轮询是最简单的负载均衡方式。以域名作为访问入口，通过配置多条DNS A记录使得请求可以分配到不同的服务器。

DNS轮询没有快速的健康检查机制，而且只支持WRR的调度策略导致负载很难“均衡”，通常用于要求不高的场景。并且DNS轮询方式直接将服务器的真实地址暴露给用户，不利于服务器安全。

b）CDN

CDN（Content Delivery Network，内容分发网络）。通过发布机制将内容同步到大量的缓存节点，并在DNS服务器上进行扩展，找到里用户最近的缓存节点作为服务提供节点。

因为很难自建大量的缓存节点，所以通常使用CDN运营商的服务。目前国内的服务商很少，而且按流量计费，价格也比较昂贵。

c）IP负载均衡

IP负载均衡是基于特定的TCP/IP技术实现的负载均衡。比如NAT、DR、Turning等。是最经常使用的方式。

IP负载均衡可以使用硬件设备，也可以使用软件实现。硬件设备的主要产品是F5-BIG-IP-GTM（简称F5\)，软件产品主要有LVS、HAProxy、NginX。其中LVS、HAProxy可以工作在4-7层，NginX工作在7层。

硬件负载均衡设备可以将核心部分做成芯片，性能和稳定性更好，而且商用产品的可管理性、文档和服务都比较好。唯一的问题就是价格。

软件负载均衡通常是开源软件。自由度较高，但学习成本和管理成本会比较大。

IP硬件负载均衡

F5的全称是F5-BIG-IP-GTM，是最流行的硬件负载均衡设备，其并发能力达到百万级。F5的主要特性包括：

a\)多链路的负载均衡和冗余

可以接入多条ISP链路，在链路之间实现负载均衡和高可用。

b\)防火墙负载均衡

F5具有异构防火墙的负载均衡与故障自动排除能力。

c\)服务器负载均衡

这是F5最主要的功能，F5可以配置针对所有的对外提供服务的服务器配置Virtual Server实现负载均衡、健康检查、回话保持等。

d\)高可用

F5设备自身的冗余设计能够保证99.999%的正常运行时间，双机F5的故障切换时间为毫秒级。

使用F5可以配置整个集群的链路冗余和服务器冗余，提高可靠的健康检查机制，以保证高可用。

e\)安全性

与防火墙类似，F5采用缺省拒绝策略，可以为任何站点增加额外的安全保护，防御普通网络攻击，包括DDoS、IP欺骗、SYN攻击、teartop和land攻击、ICMP攻击等。

f\)易于管理

F5提供HTTPS、SSH、Telnet、SNMP等多种管理方式，包含详尽的实时报告和历史纪录报告。同时还提供二次开发包\(i-Control\)。

g\)其他

F5还提供了SSL加速、软件升级、IP地址过滤、带宽控制等辅助功能。

IP软件负载均衡

a\)LVS

LVS\(Linux Virtual Server, Linux虚拟服务器），是章文嵩博士开发的开放软件，目前已经集成到Linux内核中。

基于不同的网络技术，LVS支持多种负载均衡机制。包括：VS/NAT（基于网络地址转换技术）、VS/TUN（基于IP隧道技术）和VS/DR（基于直接路由技术）。

此外，为了适应不同的需要，淘宝开发了VS/FULLNAT，从本质上来说也是基于网络地址转换技术。最近还有一个基于VS/FULLNAT的DNAT模块。

不管使用哪种机制，LVS都不直接处理请求，而是将请求转发到后面真正的服务器\(Real Server\)。不同的机制，决定了响应包如何返回到客户端。

b\)VS/NAT

NAT（Network Address Translation，网络地址转换）也叫做网络掩蔽或者IP掩蔽，是将IP 数据包头中的IP 地址转换为另一个IP 地址的过程。

NAT能够将私有（保留）地址转化为合法IP地址，通常用于一个公共IP地址和多个内部私有IP地址直接的映射，广泛应用于各种类型Internet接入方式和各种类型的网络中。

通过使用NAT将目的地址转换到多个服务器的方式，可以实现负载均衡，同时能够隐藏并保护内部服务器，避免来自网络外部的攻击。商用负载均衡设备如Cisco的LocalDirector、F5的Big/IP和Alteon的ACEDirector都是基于NAT方法。

VS/NAT\(Virtual Server via Network Address Translation\)是基于NAT技术实现负载均衡的方法。

VS/TUN

IP Tunneling\(IP隧道\)技术，又称为IP封装技术\(IP encapsulation\)，是一种在网络之间传递数据的方式。可以将一个IP报文封装到另一个IP报文（可能是不同的协议）中，并转发到另一个IP地址。

IP隧道主要用于移动主机和虚拟私有网络（Virtual Private Network），在其中隧道都是静态建立的，隧道一端有一个IP地址，另一端也有唯一的IP地址。

VS/TUN（Virtual Server via IP Tunneling）是基于隧道技术实现负载均衡的方法。

客户端负载均衡

客户端负载均衡和服务端负载均衡最大的区别在于服务清单所存储的位置。在客户端负载均衡中，所有的客户端节点都有一份自己要访问的服务端清单，这些清单统统都是从服务注册中心获取的。

Ribbon是一个基于HTTP和TCP的客户端负载均衡器，当我们将Ribbon和Eureka一起使用时，Ribbon会从注册中心去获取服务端列表，然后进行轮询访问以到达负载均衡的作用，客户端负载均衡中也需要心跳机制去维护服务端清单的有效性，当然这个过程需要配合服务注册中心一起完成。

在Spring Cloud中我们如果想要使用客户端负载均衡，方法很简单，开启@LoadBalanced注解即可，这样客户端在发起请求的时候会先自行选择一个服务端，向该服务端发起请求，从而实现负载均衡。

Ribbon客户端路由算法：

负载均衡有好几种实现策略，常见的有：

a）随机 \(Random\)

b\)轮询 \(RoundRobin\)

c\)一致性哈希 \(ConsistentHash\)

d\)哈希 \(Hash\)

e\)加权（Weighted）

以ribbon的实现为基础，看看其中的一些算法是如何实现的。

ribbon是一个为客户端提供负载均衡功能的服务，它内部提供了一个叫做ILoadBalance的接口代表负载均衡器的操作，比如有添加服务器操作、选择服务器操作、获取所有的服务器列表、获取可用的服务器列表等等。

还提供了一个叫做IRule的接口代表负载均衡策略：

public interface IRule{

```
public Server choose\(Object key\);

public void setLoadBalancer\(ILoadBalancer lb\);

public ILoadBalancer getLoadBalancer\(\);
```

}

IRule接口的实现类有以下几种：

![](/assets/loadbalance01.png)

其中RandomRule表示随机策略、RoundRobin表示轮询策略、WeightedResponseTimeRule表示加权策略、BestAvailableRule表示请求数最少策略等等。

随机策略很简单，就是从服务器中随机选择一个服务器，RandomRule的实现代码如下：

public Server choose\(ILoadBalancer lb, Object key\) {

```
if \(lb == null\) {

    return null;

}

Server server = null;



while \(server == null\) {

    if \(Thread.interrupted\(\)\) {

        return null;

    }

    List&lt;Server&gt; upList = lb.getReachableServers\(\);

    List&lt;Server&gt; allList = lb.getAllServers\(\);

    int serverCount = allList.size\(\);

    if \(serverCount == 0\) {

        return null;

    }

    int index = rand.nextInt\(serverCount\); // 使用jdk内部的Random类随机获取索引值index

    server = upList.get\(index\); // 得到服务器实例



    if \(server == null\) {

        Thread.yield\(\);

        continue;

    }



    if \(server.isAlive\(\)\) {

        return \(server\);

    }



    server = null;

    Thread.yield\(\);

}

return server;
```

}

RoundRobin轮询策略表示每次都取下一个服务器，比如一共有5台服务器，第1次取第1台，第2次取第2台，第3次取第3台，以此类推：

public Server choose\(ILoadBalancer lb, Object key\) {

```
if \(lb == null\) {

    log.warn\("no load balancer"\);

    return null;

}



Server server = null;

int count = 0;

while \(server == null && count++ &lt; 10\) { // retry 10 次

    List&lt;Server&gt; reachableServers = lb.getReachableServers\(\);

    List&lt;Server&gt; allServers = lb.getAllServers\(\);

    int upCount = reachableServers.size\(\);

    int serverCount = allServers.size\(\);



    if \(\(upCount == 0\) \|\| \(serverCount == 0\)\) {

        log.warn\("No up servers available from load balancer: " + lb\);

        return null;

    }



    int nextServerIndex = incrementAndGetModulo\(serverCount\); // incrementAndGetModulo方法内部使用nextServerCyclicCounter这个AtomicInteger属性原子递增对serverCount取模得到索引值

    server = allServers.get\(nextServerIndex\); // 得到服务器实例



    if \(server == null\) {

        Thread.yield\(\);

        continue;

    }



    if \(server.isAlive\(\) && \(server.isReadyToServe\(\)\)\) {

        return \(server\);

    }



    server = null;

}



if \(count &gt;= 10\) {

    log.warn\("No available alive servers after 10 tries from load balancer: "

            + lb\);

}

return server;
```

}

BestAvailableRule策略用来选取最少并发量请求的服务器：

public Server choose\(Object key\) {

```
if \(loadBalancerStats == null\) {

    return super.choose\(key\);

}

List&lt;Server&gt; serverList = getLoadBalancer\(\).getAllServers\(\); // 获取所有的服务器列表

int minimalConcurrentConnections = Integer.MAX\_VALUE;

long currentTime = System.currentTimeMillis\(\);

Server chosen = null;

for \(Server server: serverList\) { // 遍历每个服务器

    ServerStats serverStats = loadBalancerStats.getSingleServerStat\(server\); // 获取各个服务器的状态

    if \(!serverStats.isCircuitBreakerTripped\(currentTime\)\) { // 没有触发断路器的话继续执行

        int concurrentConnections = serverStats.getActiveRequestsCount\(currentTime\); // 获取当前服务器的请求个数

        if \(concurrentConnections &lt; minimalConcurrentConnections\) { // 比较各个服务器之间的请求数，然后选取请求数最少的服务器并放到chosen变量中

            minimalConcurrentConnections = concurrentConnections;

            chosen = server;

        }

    }

}

if \(chosen == null\) { // 如果没有选上，调用父类ClientConfigEnabledRoundRobinRule的choose方法，也就是使用RoundRobinRule轮询的方式进行负载均衡        

    return super.choose\(key\);

} else {

    return chosen;

}
```

}

Ribbon的LoadBalance：

上面说了rule，也就是ribbon在选择的server的一些算法。但是真正的主角LoadBalance还没讲到。

Ribbon和feignClient一样，当有一个服务请求发起的时候，如果对于的severId的context不存在的话，

会利用springFactory构建一个对应的子的context，构成spring context的层级关系。

最底层的是servletContext。它为web 容器提供的全局上下文环境，为后面的 Spring IOC 提供宿主环境。

![](/assets/springServletContext.png)

然后上面一层是ApplicationContext

它继承自BeanFactory接口，除了包含BeanFactory的所有功能之外，在国际化支持、资源访问（如URL和文件）、事件传播等方面进行了良好的支持，被推荐为Java EE应用之首选，可应用在Java APP与Java Web中。

由web.xml中提供的 contextLoaderListener，在web容器启动时触发容器初始化事件，其中的 contextInitialized 方法被调用时创建的根上下文（即：WebApplicationContext.ROOT\_WEB\_APPLICATION\_CONTEXT\_ATTRIBUTE,）。

采用 XmlWebApplicationContext 实现方法，作为 Spring IOC 容器，对应的 Bean 定义的配置由 web.xml 中的 context-param 标签指定。会被存储在 ServletContext 中。

而对于一个简单的Spring boot应用，它的spring context是只会有一个。

非web spring boot应用，context是AnnotationConfigApplicationContext

web spring boot应用，context是AnnotationConfigEmbeddedWebApplicationContext

AnnotationConfigEmbeddedWebApplicationContext是spring boot里自己实现的一个context，主要功能是启动embedded servlet container，比如tomcat/jetty。

然后ribbon会在这个context基础上建立一个context，实现不同服务客户端的context的隔离。

通过Ribbon的springClientFactory维护一个服务名称的context，

private Map&lt;String, AnnotationConfigApplicationContext&gt; contexts = new ConcurrentHashMap&lt;&gt;\(\);

比如说A服务里面，需要对服务B，C分别实现负载均衡，那么在A服务中，就会建立两个ribbon context，分别为LB\_B,LB\_C的context。因为这个context和Loadbalance是有状态的。

因为rule里面rule里面会设置load balance，所以rule也是有状态的，需要在不同的context中创建不同的spring bean。否则就会loadBalance就会串掉，会出现路由到不对的主机或端口。

springFactory的继承关系如下：

![](/assets/SpringClientFactory.png)

rule里面会设置load balance，所以rule也是有状态的，需要在不同的context中创建不同的spring bean。否则就会loadBalance就会串掉，会出现路由到不对的主机或端口。在定义自己的rule的spring bean的@scope要定义为protype,每次获取的时候创建一个新的实例。

Ribbon的一些默认配置在org.springframework.cloud.netflix.ribbon.RibbonClientConfiguration里面的配置以及调用RibbonAutoConfiguration里面来实现的。

其中loadbalance是最重要的角色。

@Bean

@ConditionalOnMissingBean

public ILoadBalancer ribbonLoadBalancer\(

IClientConfig config,

ServerList&lt;Server&gt; serverList, ServerListFilter&lt;Server&gt; serverListFilter,

IRule rule,

IPing ping,

ServerListUpdater serverListUpdater\) {

if \(this.propertiesFactory.isSet\(ILoadBalancer.class, name\)\) {

// 给loadBalance设置Name

return this.propertiesFactory.get\(ILoadBalancer.class, config, name\);

}

return new ZoneAwareLoadBalancer&lt;&gt;\(config, rule, ping, serverList,  
serverListFilter, serverListUpdater\);

}

@Bean

```
@ConditionalOnMissingBean

public IRule ribbonRule\(IClientConfig config\) {

    if \(this.propertiesFactory.isSet\(IRule.class, name\)\) {

        return this.propertiesFactory.get\(IRule.class, config, name\);

    }

    ZoneAvoidanceRule rule = new ZoneAvoidanceRule\(\);

    rule.initWithNiwsConfig\(config\);

    return rule;

}
```

我们可以看出默认使用的就是ZoneAwareLoadBalancer和ZoneAvoidanceRule。



LoadBalanceClient

定义了三个主要的接口来说明作为一个客户端负载均衡器要做哪些事情：

通过serviced获取对应不同context里面的loadbalanceClient，第二参数是可以在request的前后做一些其他的切面事情，计量等。

1）&lt;T&gt; T execute\(String serviceId, LoadBalancerRequest&lt;T&gt; request\) throws IOException;

在具体的load balance里面，使用具体的serviceInstance来执行请求

2）&lt;T&gt; T execute\(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest&lt;T&gt; request\) throws IOException;

用具体的host和端口来组成完整的url

3）URI reconstructURI\(ServiceInstance instance, URI original\);

根据服务名得到具体的服务实例

4）ServiceInstance choose\(String serviceId\);

父类ServiceInstanceChooser的方法





在ribbon里面具体的实现类RibbonLoadBalancerClient。在这个里面可以清楚看到上述几个方法的实现，代码逻辑也很简单清晰。

这里要说的是方法

protected Server getServer\(ILoadBalancer loadBalancer\) {

		if \(loadBalancer == null\) {

			return null;

		}

		return loadBalancer.chooseServer\("default"\); // TODO: better handling of key

}

代码里面chooseServer的参数写死了default，而正确的做法应该是取对应的zone。

幸好这个方法是protected，所以可以通过override进行重载，让ribbon对zone有更好的支持。



这里就设计到ribbon和eureka的结合部分了。

zone和region都是AWS的概念，这里不做过多的解释，在eureka的章节会有更加详细的介绍，这里只是把region这个参数配置在，

通过override对应方法和配置，实现对zone的更加完备的支持。

属性配置：

eureka.instance.metadata-map.zone=myZone



默认使用配置，如果没有可以使用available的zone，最后通过default zone最为保底选择，这样的一个机制就完备了。

可以新建一个比如TestRibbonLoadBalancerClient来集成RibbonLoadBalancerClient。



private String getZone\(\){

       String configZone =  eurekaInstanceConfig.getMetadataMap\(\).get\(ZONE\);

        if\(StringUtils.isNotEmpty\(configZone\)\){

            return configZone;

        }

        String\[\] availZones = eurekaClient.getEurekaClientConfig\(\).getAvailabilityZones\(eurekaClient.getEurekaClientConfig\(\).getRegion\(\)\);

        if \(availZones == null \|\| availZones.length == 0\) {

            return DEFAULT\_ZONE;

        }else{

            return availZones\[0\];

        }

    }



这样就可以override对应的方法了：

@override

protected Server getServer\(ILoadBalancer loadBalancer\) {

        if \(loadBalancer == null\) {

            return null;

        }

 	String zone = getZone\(\);

            if\(StringUtils.isNotEmpty\(zone\)\){

                logger.debug\("choose server of {} zone.",zone\);

                server = loadBalancer.chooseServer\(zone\);

                logger.debug\("server:{}",server\);

                return server;

            }

        logger.debug\("choose default zone server."\);

        return loadBalancer.chooseServer\(DEFAULT\_ZONE\);

}





按照spring boot的一致风格在对应的RibbonAutoConfiguration中有相应的配置。

所以如果有自己的客制化的配置的话，也需要覆盖初始配置；

@Configuration

@ConditionalOnClass\({ IClient.class, RestTemplate.class, AsyncRestTemplate.class, Ribbon.class}\)

@RibbonClients

@AutoConfigureAfter\(name = "org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration"\)

@AutoConfigureBefore\({LoadBalancerAutoConfiguration.class, AsyncLoadBalancerAutoConfiguration.class}\)

@EnableConfigurationProperties\(RibbonEagerLoadProperties.class\)

public class TestRibbonAutoConfiguration  extends RibbonAutoConfiguration {



    @Bean

    public LoadBalancerClient loadBalancerClient\(\) {

        return new TestRibbonLoadBalancerClient\(springClientFactory\(\)\);

    }

}



Ribbon的serverList 和serverStatics，ZoneStats

在ribbon的路由中，如果一些服务出现异常或者负载太高，需要特别对待，比如从服务列表中剔除，或者分配中减少任务是通过

ServerListUpdater和LoadBalancerStats相关的逻辑实现的。

上面提到的ZoneAwareLoadBalancer也具有这个能力，动态的去更新服务列表，把一些掉线的服务剔除等。

而实现这个方法的类是DynamicServerListLoadBalancer里面的成员

protected volatile ServerListUpdater serverListUpdater;

才类构造的时候会传入一个serverList，通过也会传入serverListUpdater。

根据更新策略获取最新的服务列表，来更新自己的候选名单。ribbon有自带的实现类PollingServerListUpdater，

但是如果和eureka整合之后，对应的EurekaNotificationServerListUpdater来完成相应工作。

这里面会开启一个线程池，请求去不断从eureka server中获取最新的服务信息来更新列表。





LoadBalancerStats和ZoneStats

保留一个本地map来记录调用信息，给rule选择的时候提供依据。具体实现可以看具体类里面的逻辑，这里不一一展开了。

















