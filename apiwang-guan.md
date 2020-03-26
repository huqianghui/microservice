当微服务系统变得庞大的时候，希望一些共同的逻辑能剥离出来做成一个单独的服务，而不是嵌入到各个微服务当中。网关的引入的最初原因也是这样的，就是每个服务当从外部调用的时候需要安全认证，权限控制，甚至包括数据安全控制，路由规则等等。网关最终发展能为微服务系统对外提供的一扇门，如果想反问任何里面的服务都需要通过这扇门。这样其实又可以延伸出很多有用的功能：包括API统计，计费，微服务的熔断包括限流等等。接下来会一一介绍相关的内容，请做好了，我们要出发了。

spring cloud提供了Zuul组件提供相应的功能，先通过简单的使用建立一个初步的了解。

准备工作：

在使用Zuul之前，我们先构建一个服务注册中心、以及两个简单的服务，比如：我构建了一个service-A，一个service-B。然后启动eureka-server和这两个服务。

通过访问eureka-server，我们可以看到service-A和service-B已经注册到了服务中心。

开始使用Zuul

引入依赖spring-cloud-starter-zuul、spring-cloud-starter-eureka，

如果不是通过指定serviceId的方式，eureka依赖不需要，但是为了对服务集群细节的透明性，还是用serviceId来避免直接引用url的方式吧。

&lt;dependency&gt;

```
&lt;groupId&gt;org.springframework.cloud&lt;/groupId&gt;

&lt;artifactId&gt;spring-cloud-starter-zuul&lt;/artifactId&gt;
```

&lt;/dependency&gt;

&lt;dependency&gt;

```
&lt;groupId&gt;org.springframework.cloud&lt;/groupId&gt;

&lt;artifactId&gt;spring-cloud-starter-eureka&lt;/artifactId&gt;
```

&lt;/dependency&gt;

应用主类使用@EnableZuulProxy注解开启Zuul

@EnableZuulProxy

@SpringCloudApplication

public class Application {

```
public static void main\(String\[\] args\) {

    new SpringApplicationBuilder\(Application.class\).web\(true\).run\(args\);

}
```

}

这里用了@SpringCloudApplication注解，之前没有提过，通过源码我们看到，它整合了@SpringBootApplication、@EnableDiscoveryClient、@EnableCircuitBreaker，主要目的还是简化配置。

application.properties中配置Zuul应用的基础信息，如：应用名、服务端口等。

spring.application.name=api-gateway

server.port=8080

Zuul配置

完成上面的工作后，Zuul已经可以运行了，但是如何让它为我们的微服务集群服务，还需要我们另行配置，下面详细的介绍一些常用配置内容。

服务路由

通过服务路由的功能，我们在对外提供服务的时候，只需要通过暴露Zuul中配置的调用地址就可以让调用方统一的来访问我们的服务，而不需要了解具体提供服务的主机信息了。

在Zuul中提供了两种映射方式：

通过url直接映射，我们可以如下配置：

\# routes to url

zuul.routes.api-a-url.path=/api-a-url/\*\*

zuul.routes.api-a-url.url=[http://localhost:8090/](http://localhost:8090/)

该配置，定义了，所有到Zuul的中规则为：/api-a-url/\*\*的访问都映射到[http://localhost:8090/上，也就是说当我们访问http://localhost:8080/api-a-url/add?a=1&b=2的时候，Zuul会将该请求路由到：http://localhost:8090/add?a=1&b=2上。](http://localhost:8090/上，也就是说当我们访问http://localhost:8080/api-a-url/add?a=1&b=2的时候，Zuul会将该请求路由到：http://localhost:8090/add?a=1&b=2上。)

其中，配置属性zuul.routes.api-a-url.path中的api-a-url部分为路由的名字，可以任意定义，但是一组映射关系的path和url要相同，下面讲serviceId时候也是如此。

通过url映射的方式对于Zuul来说，并不是特别友好，Zuul需要知道我们所有为服务的地址，才能完成所有的映射配置。而实际上，我们在实现微服务架构时，服务名与服务实例地址的关系在eureka server中已经存在了，所以只需要将Zuul注册到eureka server上去发现其他服务，我们就可以实现对serviceId的映射。例如，我们可以如下配置：

zuul.routes.api-a.path=/api-a/\*\*

zuul.routes.api-a.serviceId=service-A

zuul.routes.api-b.path=/api-b/\*\*

zuul.routes.api-b.serviceId=service-B

eureka.client.serviceUrl.defaultZone=[http://localhost:8010/eureka/](http://localhost:8010/eureka/)

zuul.routes.api-a.path=/api-a/\*\*

zuul.routes.api-a.serviceId=service-A

zuul.routes.api-b.path=/api-b/\*\*

zuul.routes.api-b.serviceId=service-B

eureka.client.serviceUrl.defaultZone=[http://localhost:8010/eureka/](http://localhost:8010/eureka/)

推荐使用serviceId的映射方式，除了对Zuul维护上更加友好之外，serviceId映射方式还支持了断路器，对于服务故障的情况下，可以有效的防止故障蔓延到服务网关上而影响整个系统的对外服务，下面会讲到。

服务过滤

在完成了服务路由之后，我们对外开放服务还需要一些安全措施来保护客户端只能访问它应该访问到的资源。所以我们需要利用Zuul的过滤器来实现我们对外服务的安全控制。

在服务网关中定义过滤器只需要继承ZuulFilter抽象类实现其定义的四个抽象函数就可对请求进行拦截与过滤。

比如下面的例子，定义了一个Zuul过滤器，实现了在请求被路由之前检查请求中是否有accessToken参数，若有就进行路由，若没有就拒绝访问，返回401 Unauthorized错误。

public class HeaderInfoFilter extends ZuulFilter {

```
private final static Logger logger = LoggerFactory.getLogger\(\);

@Override

public boolean shouldFilter\(\) {

    return true;

}



@Override

public Object run\(\) {

    RequestContext ctx = RequestContext.getCurrentContext\(\);

    logger.debug\("request's url:{}", ctx.getRequest\(\).getRequestURL\(\)\);

    logger.debug\("request's method:{}", ctx.getRequest\(\).getMethod\(\)\);

    if \(RequestMethod.OPTIONS.toString\(\).equals\(ctx.getRequest\(\).getMethod\(\)\)\) {

        return null;

    }



    if \(SecurityContextHolder.getContext\(\).getAuthentication\(\) == null\) {

       logger.info\(" url:{} without authentication. IP address is {}", ctx.getRequest\(\).getRequestURL\(\), getRemoteHost\(ctx.getRequest\(\)\)\);

        ctx.setSendZuulResponse\(false\);

        ctx.setResponseStatusCode\(401\);



    } else {

        Object principal = SecurityContextHolder.getContext\(\).getAuthentication\(\).getPrincipal\(\);

        LocalUser localUser = \(LocalUser\) principal;

        ThreadContext.set\(FoundationAuthConstant.IS\_CROSS\_REQUEST, BooleanConstant.TRUE\);

        String header = ctx.getRequest\(\).getHeader\("Authorization"\);

        logger.debug\("Authorization:{}", header\);

        logger.debug\("userName:{}", localUser.getUsername\(\)\);

        ctx.addZuulRequestHeader\("Authorization", header\);

        ctx.addZuulRequestHeader\(USER\_NAME\_HEADER\_KEY, localUser.getUsername\(\)\);

        ……..

    }

    return null;

}

@Override

public String filterType\(\) {

     return FilterConstants.PRE\_TYPE;

}

@Override

public int filterOrder\(\) {

    return 0;

}
```

}

Zuul工作原理：

zuul 本质也是一个springmvc项目，配置了一个zuulcontroller，这个controller又使用了ZuulServlet，这个servlet又用到了RibbonRoutingFilter过滤器。

Zuul通过一系列的ZuulFilter来实现逻辑控制，通过过滤器类型与请求生命周期以及Order实现一个执行链的组装。

过滤器类型与请求生命周期  
Zuul大部分功能都是通过过滤器来实现的。Zuul中定义了四种标准过滤器类型，这些过滤器类型对应于请求的典型生命周期。  
\(1\) PRE：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。  
\(2\) ROUTING：这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用Apache HttpClient或Netfilx Ribbon请求微服务。  
\(3\) POST：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。  
\(4\) ERROR：在其他阶段发生错误时执行该过滤器。  
除了默认的过滤器类型，Zuul还允许我们创建自定义的过滤器类型。例如，我们可以定制一种STATIC类型的过滤器，直接在Zuul中生成响应，而不将请求转发到后端的微服务。  
Zuul请求的生命周期如图8-5所示，该图详细描述了各种类型的过滤器的执行顺序。

![](/assets/ZuulFilter.png)

SpringCloud中默认的实现：

Spring Cloud默认为Zuul编写并启用了一些过滤器，这些过滤器有什么作用呢？我们不妨按照@EnableZuulServer、@EnableZuulProxy两个注解进行展开，相信大家对这两个注解都不陌生（至少都见过吧）。如果觉得陌生也没有关系，可将@EnableZuulProxy简单理解为@EnableZuulServer的增强版。事实上，当Zuul与Eureka、Ribbon等组件配合使用时，@EnableZuulProxy加入了@EnableCircuitBreaker和@EnableDiscoveryClient。

![](/assets/ZuulProxy.png)

RequestContext：用于在过滤器之间传递消息。它的数据保存在每个请求的ThreadLocal中。它用于存储请求路由到哪里、错误、HttpServletRequest、HttpServletResponse都存储在RequestContext中。RequestContext扩展了ConcurrentHashMap，所以，任何数据都可以存储在上下文中。

@EnableZuulServer装配的ZuulFilter：

pre类型过滤器

\(1\) ServletDetectionFilter：该过滤器用于检查请求是否通过Spring Dispatcher。检查后，通过isDispatcherServletRequest设置布尔值。

\(2\) FormBodyWrapperFilter：解析表单数据，并为请求重新编码。

\(3\) DebugFilter：顾名思义，调试用的过滤器，可以通过zuul.debug.request=true ，或在请求时，加上debug=true的参数，例如$ZUUL\_HOST:ZUUL\_PORT/path?debug=true 开启该过滤器。这样，该过滤器就会把RequestContext.setDebugRouting\(\) 、RequestContext.setDebugRequest\(\) 设为true。

route类型过滤器

SendForwardFilter：该过滤器使用Servlet RequestDispatcher转发请求，转发位置存储在RequestContext.getCurrentContext\(\).get\("forward.to"\) 中。

可以将路由设置成：  
zuul:  
  routes:  
    abc:  
      path: /abc/\*\*

url: forward:/abc  
然后访问$ZUUL\_HOST:ZUUL\_PORT/abc ，观察该过滤器的执行过程。

post类型过滤器

SendResponseFilter：将Zuul所代理的微服务的的响应写入当前响应。

error类型过滤器

SendErrorFilter：如果RequestContext.getThrowable\(\) 不为null，那么默认就会转发到/error，也可以设置error.path属性修改默认的转发路径。

@EnableZuulProxy过滤器

如果使用注解@EnableZuulProxy，那么除上述过滤器之外，Spring Cloud还会安装以下过滤器：

pre类型过滤器

PreDecorationFilter：该过滤器根据提供的RouteLocator确定路由到的地址，以及怎样去路由。该路由器也可为后端请求设置各种代理相关的header。

route类型过滤器

\(1\) RibbonRoutingFilter：该过滤器使用Ribbon，Hystrix和可插拔的HTTP客户端发送请求。serviceId在RequestContext.getCurrentContext\(\).get\("serviceId"\) 中。该过滤器可使用不同的HTTP客户端，例如  
Apache HttpClient：默认的HTTP客户端  
Squareup OkHttpClient v3：如需使用该客户端，需保证com.squareup.okhttp3的依赖在classpath中，并设置ribbon.okhttp.enabled = true 。  
Netflix Ribbon HTTP client：设置ribbon.restclient.enabled = true 即可启用该HTTP客户端。需要注意的是，该客户端有一定限制，例如不支持PATCH方法，另外，它有内置的重试机制。

\(2\)    SimpleHostRoutingFilter：该过滤器通过Apache HttpClient向指定的URL发送请求。URL在RequestContext.getRouteHost\(\) 中。

编写Zuul过滤器  
理解过滤器类型和请求生命周期后，我们来编写一个Zuul过滤器。编写Zuul的过滤器非常简单，我们只需继承抽象类ZuulFilter，然后实现几个抽象方法就可以了。  
可以参照上面的HeaderInfoFilter来实现自己的业务逻辑。

禁用Zuul过滤器

Spring Cloud默认为Zuul编写并启用了一些过滤器，例如DebugFilter、FormBodyWrapperFilter、PreDecorationFilter等。这些过滤器都存放在spring-cloud-netflix-core这个Jar包的org.springframework.cloud.netflix.zuul.filters包中。  
一些场景下，我们想要禁用掉部分过滤器，此时该怎么办呢？  
答案非常简单，只需设置zuul.&lt;SimpleClassName&gt;.&lt;filterType&gt;.disable=true ，即可禁用SimpleClassName所对应的过滤器。以过滤器org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter为例，只需设置zuul.SendResponseFilter.post.disable=true ，即可禁用该过滤器。

@EnableZuulProxy不仅仅是在@EnableZuulServer基础上，加入上述配置，同时根据上面的那个集成图，它同时也引入了一些额外的配置—-ZuulProxyConfiguration。

来看源码：

@Import\({ RibbonCommandFactoryConfiguration.RestClientRibbonConfiguration.class,  
RibbonCommandFactoryConfiguration.OkHttpRibbonConfiguration.class,  
RibbonCommandFactoryConfiguration.HttpClientRibbonConfiguration.class }\)  
public class ZuulProxyConfiguration extends ZuulConfiguration {

ZuulProxyConfiguration 配置了http客户端，默认是apache httpclient，在RibbonRoutingFilter使用。

Apache HttpClient 库的功能强大，使用率也很高，基本上是 Java 平台中事实上的标准 HTTP 客户端。

还有一个是OkHttpClient. 通过设置ribbon.okhttp.enabled=true，引入OkHttpClient的jar到classpath中来开启。

OkHttpClient是由 Square 公司开发的 OkHttp，是一个专注于性能和易用性的 HTTP 客户端。  
OkHttp 库的设计和实现的首要目标是高效。这也是选择 OkHttp 的重要理由之一。OkHttp 提供了对最新的 HTTP 协议版本 HTTP/2 和 SPDY 的支持，这使得对同一个主机发出的所有请求都可以共享相同的套接字连接。如果  
 HTTP/2 和 SPDY 不可用，OkHttp 会使用连接池来复用连接以提高效率。OkHttp 提供了对 GZIP 的默认支持来降低传输内容的大小。OkHttp 也提供了对 HTTP 响应的缓存机制，可以避免不必要的网络请求。当网络出现问题时，OkHttp 会自动重试一个主机的多个 IP 地址。

第三个Netflix Ribbon HTTP client. 通过设置参数ribbon.restclient.enabled=true开启.这个client不支持PATCH的method，同时也植了retry功能。

网关的熔断，降级与重试

在上面提到@EnableZuulProxy中以及包含了@EnableCircuitBreaker，所以只要保证熔断相关的jar在classpath，网关层也可以使用熔断机制。

上面提到了网关层用来做转发http的三个实现：apache的httpClient，okHttp，RibbonHttpClient。在hystrix中也分别派生出了三个HystrixCommand，

类图如下：

![](/assets/RestClient.png)

![](/assets/RibbonClient.png)

![](/assets/okClient.png)

具体的实现原理在熔断章节有具体的说明，这里就不重复了。更多的实现可以去看相应的源码。

服务降级

同理当后端服务出现异常的时候，我们不希望将异常抛出给最外层，期望服务可以自动进行一降级。Zuul给我们提供了这样的支持。当某个服务出现异常时，直接返回我们预设的信息。

我们通过自定义的fallback方法，并且将其指定给某个route来实现该route访问出问题的熔断处理。主要继承ZuulFallbackProvider接口来实现，ZuulFallbackProvider默认有两个方法，一个用来指明熔断拦截哪个服务，一个定制返回内容。

重载getRoute方法的返回值就是要监听的挂掉的微服务名字，这里只能是serviceId，不能是url。  
可以看到已经进入到熔断后的自定义处理了，目的已经达成。

在Edgware.RC1版本中Spring cloud zuul针对于降级进行了升级，升级的内容主要是当降级出现时，怎样在降级方法中获取具体的异常信息。

增加了一个接口FallbackProvider，这个接口继承了现有的ZuulFallbackProvider接口

组件多样化引出来的复杂性

网关层的超时设置就显得有点复杂了。一个请求过来的时候我们需要通过zuul过滤和转发，同时通过ribbon组件来选择服务器，还有通过hystrix来实现熔断的。因此要自定义ribbon超时的话，需要这个超时时间是小于hystrix的，不然就提前被hystrix超时了，无法起到效果。通过hystrxi的超时要小于zuul的整体转发超时，不然也起不到作用。

常见的限流算法有：令牌桶、漏桶。计数器也可以进行粗暴限流实现。

漏桶算法思路很简单，水（请求）先进入到漏桶里，漏桶以一定的速度出水，当水流入速度过大会直接溢出，可以看出漏桶算法能强行限制数据的传输速率  
图1 漏桶算法示意图

![](/assets/bucketToken.png)

对于很多应用场景来说，除了要求能够限制数据的平均传输速率外，还要求允许某种程度的突发传输。这时候漏桶算法可能就不合适了，令牌桶算法更为适合。如图2所示，令牌桶算法的原理是系统会以一个恒定的速度往桶里放入令牌，而如果请求需要被处理，则需要先从桶里获取一个令牌，当桶里没有令牌可取时，则拒绝服务。

![](/assets/bucketToken2.png)

图2 令牌桶算法示意图

我们可以使用 Guava 的 RateLimiter 来实现基于令牌桶的流量控制。RateLimiter 令牌桶算法的单桶实现,RateLimiter 对简单的令牌桶算法做了一些工程上的优化，具体的实现是 SmoothBursty。需要注意的是，RateLimiter 的另一个实现 SmoothWarmingUp，就不是令牌桶了，而是漏桶算法。  
SmoothBursty 有一个可以放 N 个时间窗口产生的令牌的桶，系统空闲的时候令牌就一直攒着，最好情况下可以扛 N 倍于限流值的高峰而不影响后续请求,就像三峡大坝一样能扛千年一遇的洪水.

SpringCloud本身也整合了一个rateLimit的组件，通过增加依赖和简单的配置实现限流功能：

&lt;dependency&gt;

```
        &lt;groupId&gt;com.marcosbarbero.cloud&lt;/groupId&gt;

        &lt;artifactId&gt;spring-cloud-zuul-ratelimit&lt;/artifactId&gt;

        &lt;version&gt;1.5.0.release&lt;/version&gt;
```

&lt;/dependency&gt;

在application.yaml中增加相应的配置项就可以使用了。

zuul:

```
ratelimit:



    key-prefix: your-prefix  \#对应用来标识请求的key的前缀



    enabled: true



    repository: REDIS  \#对应存储类型（用来存储统计信息）



    behind-proxy: true  \#代理之后



    default-policy: \#可选 - 针对所有的路由配置的策略，除非特别配置了policies



         limit: 10 \#可选 - 每个刷新时间窗口对应的请求数量限制



         quota: 1000 \#可选-  每个刷新时间窗口对应的请求时间限制（秒）



          refresh-interval: 60 \# 刷新时间窗口的时间，默认值 \(秒\)



           type: \#可选 限流方式



                - user



                - origin



                - url



      policies:



            myServiceId: \#特定的路由



                  limit: 10 \#可选- 每个刷新时间窗口对应的请求数量限制



                  quota: 1000 \#可选-  每个刷新时间窗口对应的请求时间限制（秒）



                  refresh-interval: 60 \# 刷新时间窗口的时间，默认值 \(秒\)



                  type: \#可选 限流方式



                      - user



                      - origin



                      - url
```

如果是application.properties

zuul.ratelimit.key-prefix=api-gateway

zuul.ratelimit.repository=REDIS

zuul.ratelimit.enabled=true

zuul.ratelimit.behind-proxy=true

zuul.ratelimit.default-policy.limit=10

zuul.ratelimit.default-policy.quota=1000

zuul.ratelimit.default-policy.refresh-interval=60

zuul.ratelimit.default-policy.type=user,origin,url  
z

uul.ratelimit.default-policy-list.limit=10

zuul.ratelimit.default-policy-list.quota=1000

zuul.ratelimit.default-policy-list.refresh-interval=60

zuul.ratelimit.default-policy-list.type=user,origin,url

zuul.ratelimit.policies.product.limit=10

zuul.ratelimit.policies.product.quota=1000

zuul.ratelimit.policies.product.refresh-interval=60

zuul.ratelimit.policies.product.type=user,origin,url

zuul.ratelimit.policy-list.product.limit=10

zuul.ratelimit.policy-list.product.quota=1000

zuul.ratelimit.policy-list.product.refresh-interval=60

zuul.ratelimit.policy-list.product.type=user,origin,url

具体的可以查看RateLimitProperties类看更多的参数配置。

通过查看远吗RateLimitPreFilter，可以看到如果生效的情况下，会把相应的限流信息写入到http header里面去，方便浏览器调用后的查看.

@Override

```
public Object run\(\) {

    final RequestContext ctx = RequestContext.getCurrentContext\(\);

    final HttpServletResponse response = ctx.getResponse\(\);

    final HttpServletRequest request = ctx.getRequest\(\);

    final Route route = route\(\);



    policy\(route\).forEach\(policy -&gt; {

        final String key = rateLimitKeyGenerator.key\(request, route, policy\);

        final Rate rate = rateLimiter.consume\(policy, key, null\);

        final String httpHeaderKey = key.replaceAll\("\[^A-Za-z0-9-.\]", "\_"\).replaceAll\("\_\_", "\_"\);



        final Long limit = policy.getLimit\(\);

        final Long remaining = rate.getRemaining\(\);

        if \(limit != null\) {

            response.setHeader\(LIMIT\_HEADER + httpHeaderKey, String.valueOf\(limit\)\);

            response.setHeader\(REMAINING\_HEADER + httpHeaderKey, String.valueOf\(Math.max\(remaining, 0\)\)\);

        }



        final Long quota = policy.getQuota\(\);

        final Long remainingQuota = rate.getRemainingQuota\(\);

        if \(quota != null\) {

            RequestContextHolder.getRequestAttributes\(\)

                .setAttribute\(REQUEST\_START\_TIME, System.currentTimeMillis\(\), SCOPE\_REQUEST\);

            response.setHeader\(QUOTA\_HEADER + httpHeaderKey, String.valueOf\(quota\)\);

            response.setHeader\(REMAINING\_QUOTA\_HEADER + httpHeaderKey,

                String.valueOf\(MILLISECONDS.toSeconds\(Math.max\(remainingQuota, 0\)\)\)\);

        }



        response.setHeader\(RESET\_HEADER + httpHeaderKey, String.valueOf\(rate.getReset\(\)\)\);



        if \(\(limit != null && remaining &lt; 0\) \|\| \(quota != null && remainingQuota &lt; 0\)\) {

            HttpStatus tooManyRequests = HttpStatus.TOO\_MANY\_REQUESTS;

            ctx.setResponseStatusCode\(tooManyRequests.value\(\)\);

            ctx.put\("rateLimitExceeded", "true"\);

            ctx.setSendZuulResponse\(false\);

            ZuulException zuulException =

                new ZuulException\(tooManyRequests.toString\(\), tooManyRequests.value\(\), null\);

            throw new ZuulRuntimeException\(zuulException\);

        }

    }\);



    return null;

}
```

网关的API统计

网关的路由，熔断，限流明白大概意思之后，很多时候需要更多的数据帮助我们决策。比如哪些API反应特别慢，可能是什么原因导致的。还有哪些用户在什么时候，访问了哪些API等等。只有这些统计数据的帮助才能帮助我们做更加有效的规划，调整等，做到有的放矢。而不是仅仅靠一些牛人的经验来做决策。

API访问数据的来源问题

首先确认就是作为统计API数据用的日志，应该和业务日志区分开。两者的日志格式，作用以及产生的文件大小等等都不一样。

首先能想到的方案就是利用网关容器的access log作为数据源，但是通过笔者尝试当需要增加字段的时候实现起来还是比较麻烦。

而且spring boot已经提供了一个很好的默认实现，那就是TraceRepository也是spring boot的actuate的特色功能之一。

所以可以在这个基础上进行重载，实现自己的数据客制化逻辑。这样解决了数据的产生。

代码实现如下：

@Service

public class MessageQueueTraceServiceImpl implements TraceRepository {

```
private static final Logger LOGGER = LoggerFactory.getLogger\(\);



@Autowired

AppMessageSender appMessageSender;



@Override

public List&lt;Trace&gt; findAll\(\) {

    return null;

}



@Override

public void add\(Map&lt;String, Object&gt; traceInfo\) {

    LOGGER.debug\("EbaoMessageQueueTraceServiceImpl.add..."\);

    traceInfo.remove\("body"\);

    TraceAppMessage traceAppMessage = new TraceAppMessage\(traceInfo\);

    appMessageSender.sendMessage\(”Access\_Log“, traceAppMessage\);

}
```

}

API访问数据存储问题

在非关系数据的章节会介绍到一些非关系数据库。通过对数据的特点和数据量分析来看有Elastic Search或者Solr这种搜索引擎来存储数据是比较合理的。

它提供了很多集合操作，而且在速度和可靠性上都有保证。

同时这种API的统计的数据的时效性，没有那么高，所以系统给配置的服务器可能没有达不到预期，这个时候我们可以通过增加一个消息中间层来缓解服务器的写压力。

综上所述对API的统计方案，就是通过实现TraceRepository对数据实现客制化，然后发动到对应的消息队列。然后通过消息消费者从队列中获取数据，写入到搜索引擎中。







_**网关API 权限（和URP的结合）//TODO**_

_**网关API 映射和API 编排    //TODO**_

