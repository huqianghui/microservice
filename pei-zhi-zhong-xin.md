配置和代码的分离，是程序设计一个重要原则。

常见的实现信息配置的方法:

•    硬编码\(缺点:需要修改代码,风险大\)

•    放在xml等配置文件中,和应用一起打包\(缺点:需要重新打包和重启\)

•    文件系统中\(缺点:依赖操作系统等\)

•    环境变量\(缺点:有大量的配置需要人工设置到环境变量中,不便于管理,且依赖平台\)

•    云端存储\(缺点:与其他应用耦合\)Spring Cloud Config 就是云端存储配置信息的,它具有中心化,版本控制,支持动态更新,平台独立,语言独立等特性.

java里面里面可以通过配置简单property文件，或者环境变量已经JDNI等技术实现配置的分离。

在spring的应用中显得格外明显。通过xml的定义文件让bean的定义和创建变成配置项，在启动甚至在运行期的时候去创建，让代码的耦合度降低。

可以简单的面向接口变成，实现简单的修改配置文件来替换成不同的实现。

所以配置也是代码逻辑，只是被认为比较容易变化的业务逻辑。

一个系统的可变性，易变性，从配置项也能看出一个端倪。当然过度灵活过度可配置给系统的开发和维护都带来更多的复杂性。

随着系统的升级和人员变动带来的一些文档的不匹配以及信息传递的丢失，让系统的维护和升级进一步加大难度，容易构成一个恶性循环。

所以配置的默认配置以及可配置参数集中管理变得尤为重要。

上面说的还是一个服务的情况，当系统被拆分成多个微服务的时候，可能一个人需要运维负责多个服务的时候情况就变得更加复杂。所以配置中心也变成spring cloud标准组件的一部分而提供出来。

\_\*\*Disconf介绍

---

Disconf 可以为各种业务平台提供统一的配置管理服务。

基于Spring，通过极简的使用方式（注解式编程 或 XML代码无代码侵入模式）低侵入性或无侵入性、强兼容性。

支持配置（配置项+配置文件）的分布式化管理，平台提供自校验功能（进一步提高稳定性），可以定时校验应用系统的配置是否正确。

Disconf为应用方提供了三个工具，

disconf-client, 您可以在您的应用系统里加入此Jar包；

disconf-web, 它是一个Web平台，您可以此Web平台上管理您的配置。

disconf-tool,可选包。

重要功能特点

1）支持配置（配置项+配置文件）的分布式化管理

2）配置发布统一化

配置发布、更新统一化（云端存储、发布）:配置存储在云端系统，用户统一在平台上进行发布、更新配置。

配置更新自动化：用户在平台更新配置，使用该配置的系统会自动发现该情况，并应用新配置。特殊地，如果用户为此配置定义了回调函数类，则此函数类会被自动调用。

3）配置异构系统管理

异构包部署统一化：这里的异构系统是指一个系统部署多个实例时，由于配置不同，从而需要多个部署包（jar或war）的情况（下同）。使用 Disconf后，异构系统的部署只需要一个部署包，不同实例的配置会自动分配。特别地，在业界大量使用部署虚拟化（如JPAAS系统，SAE，BAE） 的情况下，同一个系统使用同一个部署包的情景会越来越多，Disconf可以很自然地与他天然契合。

异构主备自动切换：如果一个异构系统存在主备机，主机发生挂机时，备机可以自动获取主机配置从而变成主机。

异构主备机Context共享工具：异构系统下，主备机切换时可能需要共享Context。可以使用Context共享工具来共享主备的Context。

4）极简的使用方式（注解式编程 或 XML代码无代码侵入模式）：我们追求的是极简的、用户编程体验良好的编程方式。目前支持两种开发模式：基于XML配置或才基于注解，即可完成复杂的配置分布式化。

Disconf服务搭建环境依赖，

1.Mysql

2.Tomcat（apache-tomcat-7.0.50）

3.Nginx（nginx/1.5.3）

1. Redis （2.4.5）

5.Zookeeper

页面显示：

![](/assets/disconfig.png)Apollo配置中心介绍

Apollo（阿波罗）是携程框架部门研发的开源配置管理中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性。

Apollo的架构图如下：

![](/assets/apollo-overall-architecture.png)

Apollo支持4个维度管理Key-Value格式的配置：

application \(应用\)

environment \(环境\)

cluster \(集群\)

namespace \(命名空间\)

正是基于配置的特殊性，所以Apollo从设计之初就立志于成为一个有治理能力的配置管理平台。

目前提供了以下的特性：

1）统一管理不同环境、不同集群的配置

Apollo提供了一个统一界面集中式管理不同环境（environment）、不同集群（cluster）、不同命名空间（namespace）的配置。

同一份代码部署在不同的集群，可以有不同的配置，比如zk的地址等

通过命名空间（namespace）可以很方便的支持多个不同应用共享同一份配置，同时还允许应用对共享的配置进行覆盖

2）配置修改实时生效（热发布）

用户在Apollo修改完配置并发布后，客户端能实时（1秒）接收到最新的配置，并通知到应用程序

3）版本发布管理

所有的配置发布都有版本概念，从而可以方便地支持配置的回滚

4）灰度发布

支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例

5）权限管理、发布审核、操作审计

应用和配置的管理都有完善的权限管理机制，对配置的管理还分为了编辑和发布两个环节，从而减少人为的错误。

所有的操作都有审计日志，可以方便的追踪问题

6）客户端配置信息监控

可以在界面上方便地看到配置在被哪些实例使用

提供Java和.Net原生客户端

提供了Java和.Net的原生客户端，方便应用集成

支持Spring Placeholder, Annotation和Spring Boot的ConfigurationProperties，方便应用使用（需要Spring 3.1.1+）

同时提供了Http接口，非Java和.Net应用也可以方便的使用提供开放平台API

Apollo自身提供了比较完善的统一配置管理界面，支持多环境、多数据中心配置管理、权限、流程治理等特性。

不过Apollo出于通用性考虑，对配置的修改不会做过多限制，只要符合基本的格式就能够保存。

在我们的调研中发现，对于有些使用方，它们的配置可能会有比较复杂的格式，而且对输入的值也需要进行校验后方可保存，如检查数据库、用户名和密码是否匹配。

对于这类应用，Apollo支持应用方通过开放接口在Apollo进行配置的修改和发布，并且具备完善的授权和权限控制

7）部署简单

配置中心作为基础服务，可用性要求非常高，这就要求Apollo对外部依赖尽可能地少

目前唯一的外部依赖是MySQL，所以部署非常简单，只要安装好Java和MySQL就可以让Apollo跑起来

Apollo还提供了打包脚本，一键就可以生成所有需要的安装包，并且支持自定义运行时参数

apollo的配置界面如下：

![](/assets/apollo.png)

![](/assets/apoll-edit-item-1.png)

从上述分析不难看出，百度的disconfig在部署上还是相对复杂，而且与spring cloud的思路有点远。但是携程的apollo的从设计，

到部署都和spring cloud都有不错的表现。如果没有特别的需要的情况，应该选择这个作为基础来构建。

考虑到传统企业保守性，让学习曲线相对平和一点，与spring cloud结合更加紧密以及UI等因素最后还是决定基于spring cloud config来构建一个属于自己的配置中心。

Spring cloud config

Spring Cloud Config的原理如图所示,

![](/assets/spring config server.png)

真正的数据存在Git等repository中,Config Server去获取相应的信息,然后开发给Client Application,相互间的通信基于HTTP,TCP,UDP等协议.

Spring Boot使用约定的方式来构建Spring应用，例如：它约定了配置文件的位置、通用的端点管理和监控任务。Spring Cloud建立在它的上面，并且增加了一些分布式系统可能需要的组件。

在说明配置之前，要介绍一下springboot和spring cloud启动上下文

Spring Cloud会创建一个Bootstrap Context，作为Spring应用的Application Context的父上下文。初始化的时候，Bootstrap Context负责从外部源加载配置属性并解析配置。

这两个上下文共享一个从外部获取的Environment。Bootstrap属性有高优先级，默认情况下，它们不会被本地配置覆盖。

Bootstrap context和Application Context有着不同的约定，所以新增了一个bootstrap.properties或者yml文件，而不是使用application.properties。

保证Bootstrap Context和Application Context配置的分离。下面是一个例子： \*\*bootstrap.properties\*\*

spring.application.name=foo

spring.cloud.config.uri=${SPRING\_CONFIG\_URI:[http://localhost:8888}](http://localhost:8888})

可以通过设置spring.cloud.bootstrap.enabled=false来禁用bootstrap。

应用上下文层次结构

如果你通过SpringApplication或者SpringApplicationBuilder创建一个Application Context,那么会为spring应用的Application Context创建父上下文Bootstrap Context。

在Spring里有个特性，子上下文会继承父类的property sources和profiles，所以main application context 相对于没有使用Spring Cloud Config，会新增额外的property sources。

如果在Bootstrap Context扫描到PropertySourceLocator并且有属性，则会添加到CompositePropertySource。

Spirng Cloud Config就是通过这种方式来添加的属性的，详细看源码ConfigServicePropertySourceLocator。

下面也也有一个例子自定义的例子。

“applicationConfig: \[classpath:bootstrap.yml\]” ，

（如果有spring.profiles.active=production则例如 applicationConfig: \[classpath:/bootstrap.yml\]\#production）:

如果你使用bootstrap.yml来配置Bootstrap Context，他比application.yml优先级要低。

它将添加到子上下文，作为Spring Boot应用程序的一部分。下文有介绍。

由于优先级规则，Bootstrap Context不包含从bootstrap.yml来的数据，但是可以用它作为默认设置。

这样可以很容易的扩展任何你建立的上下文层次，可以使用它提供的接口，或者使用SpringApplicationBuilder包含的方法（parent\(\)，child\(\)，sibling\(\)）。

Bootstrap Context将是最高级别的父类。扩展的每一个Context都有有自己的bootstrap property source（有可能是空的）。

扩展的每一个Context都有不同spring.application.name。同一层层次的父子上下文原则上也有一有不同的名称，因此，也会有不同的Config Server配置。

子上下文的属性在相同名字的情况下将覆盖父上下文的属性。

注意SpringApplicationBuilder允许共享Environment到所有层次，但是不是默认的。因此，同级的兄弟上下文不在和父类共享一些东西的时候不一定有相同的profiles或者property sources。

bootstrap.yml是由spring.cloud.bootstrap.name（默认:”bootstrap”）或者spring.cloud.bootstrap.location（默认空）。这些属性行为与spring.config.\*类似，通过它的Environment来配置引导ApplicationContext。如果有一个激活的profile（来源于spring.profiles.active或者Environment的Api构建），例如bootstrap-development.properties 就是配置了profile为development的配置文件.

property sources被bootstrap context 添加到应用通常通过远程的方式，比如”Config Server”。默认情况下，本地的配置文件不能覆盖远程配置，但是可以通过启动命令行参数来覆盖远程配置。如果需要本地文件覆盖远程文件，需要在远程配置文件里设置授权

spring.cloud.config.allowOverride=true（这个配置不能在本地被设置）。一旦设置了这个权限，你可以配置更加细粒度的配置来配置覆盖的方式，

比如：

* spring.cloud.config.overrideNone=true 覆盖任何本地属性

* spring.cloud.config.overrideSystemProperties=false 仅仅系统属性和环境变量

源文件见PropertySourceBootstrapProperties

自定义启动配置

bootstrap context是依赖/META-INF/spring.factories文件里面的org.springframework.cloud.bootstrap.BootstrapConfiguration条目下面，通过逗号分隔的Spring  @Configuration类来建立的配置。任何main application context需要的自动注入的Bean可以在这里通过这种方式来获取。这也是ApplicationContextInitializer建立@Bean的方式。可以通过@Order来更改初始化序列，默认是”last”。

\# spring.factories

\# AutoConfiguration

org.springframework.boot.autoconfigure.EnableAutoConfiguration=\

org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration,\

org.springframework.cloud.autoconfigure.RefreshAutoConfiguration,\

org.springframework.cloud.autoconfigure.RefreshEndpointAutoConfiguration,\

org.springframework.cloud.autoconfigure.LifecycleMvcEndpointAutoConfiguration

\# Application Listeners

org.springframework.context.ApplicationListener=\

org.springframework.cloud.bootstrap.BootstrapApplicationListener,\

org.springframework.cloud.context.restart.RestartListener

\# Bootstrap components

org.springframework.cloud.bootstrap.BootstrapConfiguration=\

org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration,\

org.springframework.cloud.bootstrap.encrypt.EncryptionBootstrapConfiguration,\

org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration,\

org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration

bootstrap context通过spring.factories配置的类初始化的所有的Bean都会在SpingApplicatin启动前加入到它的上下文里去。

自定义引导配置来源：Bootstrap Property Sources

默认的property source添加额外的配置是通过配置服务（Config Server），你也可以自定义添加property source通过实现PropertySourceLocator接口来添加。

你可以使用它加配置属性从不同的服务、数据库、或者其他。

下面是一个自定义的例子:

复制代码

@Configuration

public class CustomPropertySourceLocator implements PropertySourceLocator {

```
@Override

public PropertySource&lt;?&gt; locate\(Environment environment\) {

    return new MapPropertySource\("customProperty",

            Collections.&lt;String, Object&gt;singletonMap\("property.from.sample.custom.source", "worked as intended"\)\);

}
```

}

Environment被ApplicationContext建立，并传入property sources（可能不同个profile有不同的属性），所以可以从Environment寻找找一些特别的属性。比如spring.application.name，它是默认的Config Server property source。

如果你建立了一个jar包，里面添加了一个META-INF/spring.factories文件：

org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator

那么，”customProperty“的PropertySource将会被包含到应用。

所以spring config center变在spring启动的时候，通过相应的维度获取到数据设置启动环境中去。

Spring cloud config

创建并运行一个Spring Cloud Config Server  
1.创建一个名为my-config-server的应用,并添加spring-cloud-starter-parent,spring-cloud-config-server依赖

2.在Application主类上添加@EnableConfigServer注解。

创建并运行一个Spring Cloud Config Client

1.创建一个名为my-config-client的应用,并添加spring-cloud-starter-parent,spring-cloud-starter-config,spring-boot-starter-web依赖

2.创建bootstrap.yml在resource下,并设置spring.application.name,spring.cloud.config.uri,server.port信息,具体如下

spring:

application:

```
  name: mmb-config-client
```

cloud:

```
 config:


         uri:  http://localhost:8001
```

---

server:  
  port: 8002

注意这里是bootstrap.yml而不是appliction.yml,因为bootstrap.yml会在应用启动之前读取,而spring.cloud.config.uri会影响应用启动，

这样spring cloud client变可以从config server中获取数据了。

spring cloud config

整体的架构

![](/assets/ebaoConfigCenter.png)

1）维度拆分

在实际使用中，会分成很多不同的环境和版本。比如开发阶段有开发环境，测试环境，配置环境等等，所以环境是影响参数配置的最大的一个因素，可以当做系统级别的因子。然后是应用，也就是各个微服务需要自己配置参数，当然也支持继承关系，让一些共同的参数放到一个微服务中配置，比如叫做Application，把一些共同的配置，归纳到一起。比如Redis，Es，数据库的一些方言等等。

版本作为最后一个维度，当系统发布不同版本的时候，比如灰度发布的时候，API升级实现蓝绿部署的时候，就需要同一个应用不同的版本需要不同的参数。

所以总结如下：

配置中心基于环境、应用、版本三个维度管理。

•    环境\(profile\):表示获取指定环境下配置，例如开发环境、测试环境、生产环境 默认值default，实际开发中可以是 dev、test、demo、production等；

•    应用\(application\):表示应用名称,在client中通过spring.config.name配置；

•    版本\(label\): 默认值snapshot。

2）数据安全和权限

设计到参数范围，设计相关的角色等等、

配置参数的范围  
出于责任单一，数据的安全控制，业务的参数配置还是通过代码，或者配置表等来实现。比如续保的批处理中用到，离满期的时间间隔等。  
使用配置中心的人，只有发布人员和运维人员使用，这样权限安全相对单一和简单。  
配置中心的配置参数主要有三类：

a）面向系统发布人员的系统配置参数。

b）面向系统运维人员的监控参数配置。

c）系统集成的参数，比如DMS，EMAIL Server等。

配置参数的分组  
根据使用配置中心角色和系统参数的业务意义对参数进行分组，便于参数的管理和使用。例如DB信息配置主要是系统发布人员使用，这样的参数主要就是通过  
TS.DataBase.UserName=\*\*\*\*\*\*  
配置系统安全  
为了系统的稳定性，对参数项的修改等操作要做相应的控制。

具体的控制如下：

a\)    在生产环境的系统配置项和集成配置型不能增，改，删操作。

b\)    对运维数据的增，改，删操作提供日志功能。

c\)    对于新环境的参数配置，要通过脚本的方式来实现配置中心的初始值的设定。减少人员的修改错误，同时这部分不数据日志范围内。

参数配置的数据安全  
为了保护参数的安全性，支持内容的加密。同时为了保持使用的便利性，对消费方保持透明，在客户端中对加密内容进行了解密。

Config Server为我们提供了加密与解密的端点，分别是/encrypt 与/decrypt  
支持对称加密和非对称加密两种方式。

参数的存储  
支持git ，svn，文件系统的的存储模式。  
现在代码是通过SVN管理，默认使用SVN来实现参数的存储。

客户端模块设计  
配置中心客户端有两个功能：

a）实现从配置中心获取参数，来实现对默认参数等的替换。

b）    监听配置中心的数据变更，体现对配置数据的热替换。

c）    微服务系统的所有配置的查询

3）模型设计

public class ConfigPropertyUpdate{

// 对应上面三个维度

```
private String label;

private String application;

private String profile;
```

// 值健对

```
private String key;

private String value;
```

// 可能输入的值：比如boolean的true/false

```
private String options;
```

// 描述

```
private String description;
```

// 是否支持热部署

```
private String refreshable;
```

// 修改的用户角色

```
private String userRoles;
```

}

4）数据热部署

因为spring cloud 提供的刷新整个spring context，这样导致系统有一段时间不可用。需要重新装载这个系统，所以代价太高。

spring bean中默认都是单例的，热部署相对来说使用比较少，所以为了一个小的功能来系统等待，没有必要。但是在熔断，限流等功能中

参数的热替换又是必不可少的，面对一些突发情况必须需要使用特替换来完成相应的限制。

所以参数特替换，需要一定程度的客户化。

1）变化告知

配置中心服务器通过消息的data bus也就是对应board cast把相应的变化告知各个应用实例。

应用实例接受到变化之后，配置的客户端保留着更新的参数到环境变量中。

这样配置中心的服务器端和客户端的事情做完了。

2）应用变化

各个热替换参数，可能是用变化存储或者缓存等等。

这样，应用需要监听相应的数据变化，当方式变化的时候，清除数据缓存或者对象。

同时在使用的时候，根据懒加载或者加载的方式实现新实例的创建。

举一个实际的例子：

在熔断设置中，HystrixPropertiesFactory里面就缓存了对应的数据

/\*\*

```
 \* Clears all the defaults in the static property cache. This makes it possible for property defaults to not persist for

 \* an entire JVM lifetime.  May be invoked directly, and also gets invoked by &lt;code&gt;Hystrix.reset\(\)&lt;/code&gt;

 \*/

public static void reset\(\) {

    commandProperties.clear\(\);

    threadPoolProperties.clear\(\);

    collapserProperties.clear\(\);

}
```

所以当数据发生变化的时候，可以通过焦勇reset方法来实现清理。

这样的话，可以让command等，重新创建的时候去拿最新的数据如下：

通过重载FeignHystrixSetterFactory的

@Override

```
public HystrixCommand.Setter create\(Target&lt;?&gt; target, Method method\) {

    String serverName = target.type\(\).getAnnotation\(FeignClient.class\).name\(\);

    String path = target.type\(\).getAnnotation\(FeignClient.class\).path\(\);

    String url = getUriByMethod\(method\);

    LOGGER.debug\("class :{}", target.getClass\(\)\);

    LOGGER.debug\("method :{}", method.getName\(\)\);

    LOGGER.debug\("serverName :{}", serverName\);

    Assert.isNotEmpty\(path + url, "The url is null."\);

    String absUrlWithoutApplicationName = \(path + url\).startsWith\("/"\) ? \(path + url\) : \("/" + path + url\);

    String absUrl = serverName + absUrlWithoutApplicationName;

    LOGGER.debug\("url :{}", absUrl\);

    HystrixCommandProperties.Setter hystrixCommandPropertiesSetter;

    if \(hystrixCommandPropertiesSetterMap.get\(absUrl\) != null\) {

        hystrixCommandPropertiesSetter = hystrixCommandPropertiesSetterMap.get\(absUrl\);

    } else {

        hystrixCommandPropertiesSetter = buildHystrixCommandPropertiesSetter\(absUrlWithoutApplicationName, serverName\);

        hystrixCommandPropertiesSetterMap.putIfAbsent\(absUrl, hystrixCommandPropertiesSetter\);

    }

    HystrixThreadPoolProperties.Setter hystrixThreadPoolPropertiesSetter;

    if\(hystrixThreadpoolPropertiesSetterMap.get\(serverName\) !=null\){

        hystrixThreadPoolPropertiesSetter = hystrixThreadpoolPropertiesSetterMap.get\(serverName\);

    }else{

        hystrixThreadPoolPropertiesSetter = buildHystrixThreadpoolPropertiesSetter\(serverName\);

        hystrixThreadpoolPropertiesSetterMap.putIfAbsent\(serverName,hystrixThreadPoolPropertiesSetter\);

    }



    return HystrixCommand.Setter

            .withGroupKey\(HystrixCommandGroupKey.Factory.asKey\(serverName\)\)

            .andCommandKey\(HystrixCommandKey.Factory.asKey\(absUrl\)\)

            .andThreadPoolKey\(HystrixThreadPoolKey.Factory.asKey\(serverName\)\)

            .andCommandPropertiesDefaults\(hystrixCommandPropertiesSetter\)

            .andThreadPoolPropertiesDefaults\(hystrixThreadPoolPropertiesSetter\);



}
```

private HystrixCommandProperties.Setter buildHystrixCommandPropertiesSetter\(String commandKey, String applicationName\) {

```
    Assert.isNotNull\(commandKey, "The commandKey is null."\);

    Assert.isNotNull\(applicationName, "The applicationName is null."\);

    LOGGER.debug\("commandKey:{}", commandKey\);

    LOGGER.debug\("applicationName:{}", applicationName\);

    Map&lt;String, String&gt; customizationParameterMap = HystrixCommandHotspotParameterProcessor.getAllHotspotParameter\(commandKey, applicationName\);

    if \(MapUtils.isNotEmpty\(customizationParameterMap\)\) {

        return getHystrixCommandPropertiesSetterByCustomizationParameterMap\(customizationParameterMap\);

    }

    return HystrixCommandProperties.defaultSetter\(\);



}
```



