springMVC是spring3时代一个很重要的组成模块。springMVC也贯彻了Spring的思想简单，配置少，而且一开始就整合了java5的annotation的特性，只需要加上一些注解就可以把一个普通的类，改造成一个能处理selvet请求的处理器。但是它的底层实现还是比较复杂，通过一个servlet统一处理， 然后通过它的九大组件来分别处理参数，异常，返回值等等。

上述我们在下面会一一介绍。而在介绍这些之前，能够横向比较一些其他的框架和知识。这样大家如果有相应的需求的时候能够做出更加适合自己的选择。

springMVC和structs1，structs2

做过java web的程序员，我想大部分人都用过structs1，structs2吧。说起java web开发，就要提及SSH，第一个s就是structs。但是随着时间的推移，它慢慢淡出了我们视野，后两者的发展都还是处于主流状态。虽然hibernate也有ibatis，mybatis类似的轻量级的框架来不断冲击，但是hibernate确实还是有着它独有的优势。

用过struct1和struct2的人，一定有个疑问明明者两者没什么联系，struct1和struct2本质上都不一样，感觉就是借用一个名字。有点类似java和javascript一样，强行绑定联系在一起。事实也确实如此，struct2是从Webwork2进化而来。经历几年的发展之后，struts和WebWork社区决定合二 为一，也就是今天的struts2.

Struts1官方已经停止更新，现在用的也比较少了。说几个特性大家对有一个大致的印象，新项目选择它的可能性不大。

1）Struts1要求Action类继承一个抽象基类。Struts1的一个普遍问题是使用抽象类编程而不是接口。主要是使用模版模式，处理一些共通的逻辑。那个时候java的接口也没有default 方法。

2）Struts1 Action是单例模式并且必须是线程安全的，因为仅有Action的一个实例来处理所有的请求。单例策略限制了Struts1 Action能作的事，并且要在开发时特别小心。Action资源必须是线程安全的或同步的。

3）Struts1 Action 依赖于ServletAPI ,因为当一个Action被调用时HttpServletRequest 和 HttpServletResponse 被传递给execute方法。这个用习惯了新一代框架的人，可能很不适应。写业务代码还需要知道这些，但是当时确实也就是这样的。

4）Struts1 使用ActionForm对象捕获输入。所有的 ActionForm必须继承一个基类。因为其他JavaBean不能用作ActionForm，开发者经常创建多余的类捕获输入。动态 Bean（DynaBeans）可以作为创建传统ActionForm的选择，但是开发者可能是在重新描述\(创建\)已经存在的JavaBean（仍然会 导致有冗余的javabean）。

5）Struts1 整合了JSTL，因此使用JSTL EL。这种EL有基本对象图遍历，但是对集合和索引属性的支持很弱。

6）Struts 1使用标准JSP机制把对象绑定到页面中来访问。

7） Struts 1 ActionForm 属性通常都是String类型。Struts1使用Commons-Beanutils进行类型转换。

```
 每个类一个转换器，对每一个实例来说是不可配置的。
```

8）Struts1支持每一个模块有单独的Request Processors（生命周期），但是模块中的所有Action必须共享相同的生命周期。

Structs1当时社区还是很多，使用率也很高。但是随着其他框架的兴起，这些特点变得格格不入。在与WebWork社区合作创建出了struct2.

struct2基本对上述问题都有很大的改进和提升。

1）Struts 2 Action类可以实现一个Action接口，也可实现其他接口，使可选和定制的服务成为可能。

Struts2提供一个ActionSupport基类去 实现常用的接口。Action接口不是必须的，

任何有execute标识的POJO对象都可以用作Struts2的Action对象。

2）Struts2 Action对象为每一个请求产生一个实例，因此没有线程安全问题。

实际上，servlet容器给每个请求产生许多可丢弃的对象，并且不会导致性能和垃圾回收问题。

3）Struts 2 Action不依赖于容器，允许Action脱离容器单独被测试。如果需要，Struts2 Action仍然可以访问初始的request和     response。但是，其他的元素减少或者消除了直接访问 HttpServetRequest 和 HttpServletResponse的必要性。

4）Struts 2直接使用Action属性作为输入属性，消除了对第二个输入对象的需求。输入属性可能是有自己\(子\)属性的rich对象类型。

```
Action属性能够通过 web页面上的taglibs访问。

Struts2也支持ActionForm模式。rich对象类型，包括业务对象，能够用作输入/输出对象。

这种 ModelDriven 特性简化了taglib对POJO输入对象的引用。
```

5）Struts2可以使用JSTL，但是也支持一个更强大和灵活的表达式语言－－"Object Graph Notation Language" \(OGNL\).

6）Struts 2 使用 "ValueStack"技术，使taglib能够访问值而不需要把你的页面（view）和对象绑定起来。

ValueStack策略允许通过一系列名称相同但类型不同的属性重用页面（view）。

7）Struts2 使用OGNL进行类型转换。提供基本和常用对象的转换器。

8）Struts2支持通过拦截器堆栈（Interceptor Stacks）为每一个Action创建不同的生命周期。堆栈能够根据需要和不同的Action一起使用。

SpringMVC和Struct2的比较

1）机制

spring mvc的入口是servlet，而struts2是filter。

这样就导致了二者的机制不同，这里就牵涉到servlet和filter的区别了。

2）性能

spring会稍微比struts快。spring mvc是基于方法的设计，而sturts是基于类，每次发一次请求都会实例一个action，每个action都会被注入属性，而spring基于方法， 粒度更细，但要小心把握像在servlet控制数据一样。spring3 mvc是方法级别的拦截，拦截到方法后根据参数上的注解，把request数据注入进去，在spring3 mvc中，一个方法对应一个request上下文。

而struts2框架是类级别的拦截，每次来了请求就创建一个Action，然后调用setter getter方法把request中的数据注入；

struts2实际上是通过setter getter方法与request打交道的；struts2中，一个Action对象对应一个request上下文。

3）参数传递

struts是在接受参数的时候，可以用属性来接受参数，这就说明参数是让多个方法共享的。springMVC更多的是方法的参数，通过其他的形式补充。

总结如下：

spring mvc用的是独立的AOP方式，有以自己的interceptor机制。这样导致struts的配置文件量还是比spring mvc大，虽然struts的配置能继承，所以我觉得论使用上来讲，spring mvc使用更加简洁，开发效率Spring MVC确实比struts2高。spring mvc是方法级别的拦截，一个方法对应一个request上下文，而方法同时又跟一个url对应，所以说从架构本身上spring3 mvc就容易实现restful url。

struts2是类级别的拦截，一个类对应一个request上下文；实现restful url要费劲，因为struts2 action的一个方法可以对应一个url；

而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。

spring3mvc的方法之间 基本上独立的，独享request response数据，请求数据通过参数获取，

处理结果通过ModelMap交回给框架方法之间不共享变量，而struts2搞的就比较乱，虽然方法之间 也是独立的，

但其所有Action变量是共享的，这不会影响程序运行，却给我们编码，读程序时带来麻烦。

springMVC 和 java restful web service相关框架

上面提及了Restful相关的概念，这个又是一个新技术时代的产物。上面简单介绍了structs框架，不难看出它也是一个完整的MVC的框架和springMVC是对等的。

虽然主要是Controller层，但是对model和view都有一定程度的控制和支持。但是Restful web service更加偏重服务也就是Controller，然后也一个简单

输入结果。对view层不做任何控制和要求，这个是时代对前后端分离的产物。

主要原因是现在有越来越多的公司希望能以简单而又贴合Web架构本身的方式公开Web API，因此REST变得越来越重要也就不足为奇了。使用Ajax进行通信的富浏览器端也在朝这个目标不断迈进。这个架构原则提升了万维网的可伸缩性，无论何种应用都能从该原则中受益无穷。

在java技术体系中就要依靠JSR来提供标准了。

JAX-RS（JSR 311）指的是Java API for RESTful Web Services，Roy Fielding也参与了JAX-RS的制订，他在自己的博士论文中定义了REST。

对于那些想要构建RESTful Web Services的开发者来说，JAX-RS给出了不同于JAX-WS（JSR-224）的另一种解决方案。

目前共有4种JAX-RS实现，所有这些实现都支持Spring，Jersey则是JAX-RS的参考实现。

Spring MVC与JAX-RS有何异同点？更进一步，如果有一个Spring MVC应用，使用了控制类继承（SimpleFormController等），Spring MVC对REST广泛的支持。  
本文将介绍Spring 3中的REST特性并与JAX-RS进行对比，希望能帮助你理顺这两种编程模型之间的异同点。

再次说明一下JAX-RS的目标是Web Services开发（这与HTML Web应用不同）而Spring MVC的目标则是Web应用开发。SpringMVC是一个MVC框架，包括MVC三者。而JAX-RS就是一个restful service，指对服务和放回的数据负责。对调用的service放，是用view展示，还是额外的加工，不做任何处理的。

Spring 3为Web应用与Web Services增加了广泛的REST支持，这里只讨论JAX-RS的上下文中讨论Spring MVC。

还要要说明的是要讨论的REST特性是Spring Framework的一部分，也是现有的Spring MVC编程模型的延续，

因此并没有所谓的“Spring REST framework”这种概念，有的只是Spring和Spring MVC。这意味着如果你有一个Spring应用的话，

你既可以使用Spring MVC创建HTML Web层，也可以创建RESTful Web Services层。

也是spring mvc为什么即使好像是上一代的技术，但是在restful service的时代，仍然有着自己的一席之地。

JAX-RS只是一个标准，JAX-RS提供一些标注将一个资源类，一个 POJO Java类，封装成Web资源。

@Path，标注资源类或方法的相对路径  @GET，@PUT，@POST，@DELETE，标注方法是用的HTTP请求的类型

@Produces，标注返回的MIME媒体类型

@Consumes，标注可接受请求的MIME媒体类型

@PathParam，@QueryParam，@HeaderParam，@CookieParam。@MatrixParam，@FormParam分别标注方法的参数来自于HTTP请求的不同位置，

例如@PathParam来自于URL的路径，@QueryParam来自于URL的查询参数，@HeaderParam来自于HTTP请求的头信息，@CookieParam来自HTTP请求的Cookie。

所以下面我们使用springMVC和其他实现了JAX—RS框架的来做一个横向比较。

在这个横向比较的过程也展示一些springMVC自己的基本用法。

1.Jersey与springMVC

Jersey是JAX-RS的参考实现，即Jersey不是基于Servlet API的，而是采用了JAX-RS API。

同时它扩展了JAX-RS 参考实现， 提供了更多的特性和工具， 可以进一步地简化 RESTful service 和 client 开发。

尽管相对年轻，它已经是一个产品级的 RESTful service 和 client 框架。大致特性如下：

1）Jersey采用了JAX——RS的Annotation机制，所有的HTTP相关的参数设置都采用标注实现。

因此，在编程的时候我们针对的仍然是POJO，体会不到分布式或J2EE编程的痛苦，只需要了解一些关键的Annotation即可。

2）Jersey是一个开发的平台，我们可以扩展自己的需求，比如在消息格式上，

虽然Jersey已经提供了Java基本数据类型、JSON、XML等类型，我们还是可以很容易地扩展自己的格式。

3）Jersey建立的服务可以很简单地部署到JDK6自带的轻量级Server上，过程极其简单 。

4）Jersey建立的服务可以非常容易地部署为Servlet，支持各种J2EE容器。

5）Jersey可以为我们编写的服务自动生成WADL

6）支持Spring。

除此之外Jersey有着优秀的文档和例子，性能表现也很好很容易开发，但是Jersey 2.0+使用了有些复杂的依赖注入实现。

一大堆第三方库只支持 Jersey 1.X， 在 Jersey 2.X 不可用。如果涉及到功能扩展的话要考虑一下这个问题了。

接下来一一对比它和springMVC的功能不同的地方。

我们会在Spring MVC和JAX-RS中都使用Spring实现依赖注入。Spring MVC DispatcherServlet和Jersey SpringServlet会把请求代理给

Spring管理的REST层组件（控制器或资源），后者会由业务或持久层组件包装起来。

Jersey和Spring MVC都使用Spring的ContextLoaderListener加载业务与持久层组件，

比如JpaAccountRepository：  
&lt;context-param&gt;

```
&lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;     

&lt;param-value&gt;classpath:META-INF/spring/module-config.xml&lt;/param-value&gt;
```

&lt;/context-param&gt;

&lt;listener&gt;

&lt;listener-class&gt;org.springframework.web.context.ContextLoaderListener&lt;/listener-class&gt;

&lt;/listener&gt;  
ContextLoaderListener可用于任何Web或REST框架环境中。

在Jersey中创建Spring管理的JAX-RS资源  
Jersey支持在REST层中使用Spring，三个简单的步骤就能搞定。

步骤一：

将构建依赖加到maven artifact com.sun.jersey.contribs:jersey-spring中。

步骤二：

将如下配置片段加到web.xml中以保证Spring能够创建JAX-RS根资源：

&lt;servlet&gt;  
    &lt;servlet-name&gt;Jersey Web Application&lt;/servlet-name&gt;  
    &lt;servlet-class&gt;  
        com.sun.jersey.spi.spring.container.servlet.SpringServlet  
    &lt;/servlet-class&gt;  
&lt;/servlet&gt;

&lt;servlet-mapping&gt;  
    &lt;servlet-name&gt;Jersey Web Application&lt;/servlet-name&gt;  
    &lt;url-pattern&gt;/resources/\*&lt;/url-pattern&gt;  
&lt;/servlet-mapping&gt;

步骤三：

使用Spring和JAX-RS注解声明根JAX-RS资源类：  
@Path\("/accounts/"\)  
@Component  
@Scope\("prototype"\)  
public class AccountResource {

```
@Context
UriInfo uriInfo;

@Autowired
private AccountRepository accountRepository;



@GET
@Path\("{username}"\)
public Account getAccount\(@PathParam\("username"\) String username\) {

}
```

如下是对这些注解的说明：

@Component将AccountResource声明为Spring bean。

@Scope声明了一个prototype Spring bean，这样每次使用时都会实例化（比如每次请求时）。

@Autowired指定了一个AccountRepository引用，Spring会提供该引用。

@Path是个JAX-RS注解，它将AccountResource声明为“根”JAX-RS资源。

@Context也是一个JAX-RS注解，要求注入特定于请求的UriInfo对象。

JAX-RS有“根”资源（标记为@Path）和子资源的概念。在上面的示例中，AccountResource就是个根资源，

它会处理以“/accounts/”开头的路径。AccountResource中的方法如getAccount\(\)只需声明针对类型级别的相对路径即可。  
访问路径“/accounts/{username}”（其中的username是路径参数，可以是某个账户的用户名）的请求将由getAccount\(\)方法处理。

创建Spring MVC @Controller类  
对于Spring MVC来说，我们需要创建DispatcherServlet，同时将contextConfigLocation参数指定为Spring MVC配置：

&lt;servlet&gt;  
    &lt;servlet-name&gt;Spring MVC Dispatcher Servlet&lt;/servlet-name&gt;

&lt;servlet-class&gt;org.springframework.web.servlet.DispatcherServlet&lt;/servlet-class&gt;

&lt;init-param&gt;  
        &lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;

&lt;param-value&gt;  
            /WEB-INF/spring/\*.xml  
        &lt;/param-value&gt;

&lt;/init-param&gt;

&lt;/servlet&gt;

要想在Spring MVC（@MVC）中使用基于注解的编程模型还需要少量的配置。

下面的component-scan元素会告诉Spring去哪里寻找@Controller注解类。

&lt;context:component-scan base-package="org.springframework.samples.stocks" /&gt;  
接下来，我们声明了AccountController，如下代码所示：

@Controller  
@RequestMapping\("/accounts"\)

public class AccountController {

@Autowired

private AccountRepository accountRepository;

}

@RequestMapping注解会将该控制器映射到所有以“/accounts”开头的请求上。

AccountController中的方法如getAccount\(\)只需声明针对“/accounts”的相对地址即可。

@RequestMapping\(value = "/{username}", method = GET\)  
public Account getAccount\(@PathVariable String username\) {

}  
Spring MVC则没有根资源与子资源的概念，这样每个控制器都是由Spring而非应用来管理的。类名上写@RequestMapping\("/accounts”\)相当于根资源，但是没有特别提出来作为另外一个概念也算是一种程度的简化。

处理请求数据  
HTTP请求中包含着应用需要提取和处理的数据，如HTTP头、cookie、查询字符串参数、表单参数以及请求体（XML、JSON等）中所包含的大量数据。

在RESTful应用中，URL本身也可以带有重要的信息，如通过路径参数指定需要访问哪个资源、通过文件扩展名（.html, .pdf）指定需要何种内容类型等。

HttpServletRequest提供了处理这一切的所有底层访问机制，但直接使用HttpServletRequest实在是太乏味了。  
请求参数、Cookies和HTTP头  
Spring MVC和JAX-RS拥有能够抽取这种HTTP请求值的注解：  
@GET

@Path  
public void foo\(@QueryParam\("q"\) String q,

```
    @FormParam\("f"\) String f, 

    @CookieParam\("c"\) String c,


         @HeaderParam\("h"\) String h, 

    @MatrixParam\("m"\) m\) {
// JAX-RS
```

}

@RequestMapping\(method=GET\)  
public void foo\(

```
@RequestParam\("q"\) String q, 

@CookieValue\("c"\) String c, 

@RequestHeader\("h"\) String h\) {
// Spring MVC
```

}  
上面的注解非常像，区别在于JAX-RS支持矩阵参数（matrix parameters）的抽取，拥有单独的注解来处理查询字符串和表单参数。

矩阵参数并不常见，他们类似于查询字符串参数，但却使用了特殊的路径片段（比如GET /images;name=foo;type=gif）。稍后将介绍表单参数。  
假如使用了前请求范围声明资源，那么JAX-RS可以在属性和setters方法上使用上述注解。  
Spring MVC有个特性能让我们少敲几个字符，如果注解名与Java参数名相同，那么就可以省略掉上面的注解名了。

比如说，名为“q”的请求参数要求方法参数也得为“q”：  
public void foo\(

```
@RequestParam String q, 

@CookieValue c, 

@RequestHeader h\) {
```

}  
这对于那些在参数中使用了注解而导致方法签名变长的情况来说实在是太方便了。

请记住，这个特性要求代码使用调试符号进行编译。

类型转换与HTTP请求值的格式化

HTTP请求值（头、cookies和参数）是不变的字符串并且需要解析。  
JAX-RS通过寻找valueOf\(\)方法或是在客户化的目标类型中接收字符串的构造方法来解析请求数据。

JAX-RS支持如下类型的注解方法参数，包括路径变量、请求参数、HTTP头值和cookies：  
原生类型。  
拥有接收单个字符串参数的构造方法的类型。  
拥有一个接收单个字符串参数的名为valueOf的静态方法的类型。  
List&lt;T&gt;、Set&lt;T&gt;或是SortedSet&lt;T&gt;，其中的T满足上面2个或3个要求。  
Spring 3支持上面所有要求。除此之外，Spring 3提供了一种全新的类型转换与格式化机制，并且可以使用注解实现。

表单数据  
如前所述，JAX-RS处理查询字符串参数和表单参数的方式是不同的。虽然Spring MVC只有一个@RequestParam，

但它还提供了一种Spring MVC用户很熟悉的数据绑定机制来处理表单输入。

比如说，如果一个表单提交了3个数据，那么一种可能的处理方式就是声明一个带有3个参数的方法：  
@RequestMapping\(method=POST\)  
public void foo\(@RequestParam String name,

```
    @RequestParam creditCardNumber, 

    @RequestParam expirationDate\) {
Credit card = new CreditCard\(\);
card.setName\(name\);
card.setCreditCardNumber\(creditCardNumber\);
card.setExpirationDate\(expirationDate\);
```

}  
然而，随着表单数据量的增加，这种处理方式就会变得不切实际。

借助于数据绑定，Spring MVC可以创建、组装并传递包含有嵌套数据（账单地址、邮件地址等）、任意结构的表单对象。

@RequestMapping\(method=POST\)

public void foo\(CreditCard creditCard\) {

// POST /creditcard/1

//        name=Bond  
    //        creditCardNumber=1234123412341234  
    //

expiration=12-12-2012  
}

要想与Web浏览器协同工作，表单处理是个重要环节。另一方面，Web Services客户端一般会在请求体中提交XML或JSON格式的数据。

处理请求体中的数据  
无论是Spring MVC还是JAX-RS都能够自动处理请求体中的数据：

@POST

public Response createAccount\(Account account\) {

// JAX\_RS

}

@RequestMapping\(method=POST\)

public void createAccount\(@RequestBody Account account\) {

// Spring MVC  
}

JAX-RS中的请求体数据  
在JAX-RS中，类型MessageBodyReader的实体供应者负责转换请求体数据。JAX-RS的实现需要拥有一个JAXB MessageBodyReader，这可以使用具有注解@Provider的客户化MessageBodyReader实现。

Spring MVC中的请求体数据  
在Spring MVC中，如果想通过请求体数据初始化方法参数，那可以将@RequestBody注解加到该方法参数前，这与之前介绍的表单参数初始化正好相反。

在Spring MVC中，HttpMessageConverter类负责转换请求体数据，Spring MVC提供了一个开箱即用的Spring OXM HttpMessageConverter。

它支持JAXB、Castor、JiBX、XMLBeans和XStream，此外还有一个用于处理JSON的Jackson HttpMessageConverter。  
HttpMessageConverter会注册到AnnotationMethodHandlerAdapter上，后者会将到来的请求映射到Spring MVC @Controllers上。

下面是其配置：

&lt;bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" &gt;

&lt;property name="messageConverters" ref="marshallingConverter"/&gt;

&lt;/bean&gt;

&lt;bean id="marshallingConverter" class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter"&gt;  
    &lt;constructor-arg ref="jaxb2Marshaller"/&gt;

&lt;property name="supportedMediaTypes" value="application/vnd.stocks+xml"/&gt;  
&lt;/bean&gt;

&lt;oxm:jaxb2-marshaller id="jaxb2Marshaller"/&gt;

Spring 3新增的mvc客户化命名空间将上述配置自动化了，只需增加如下配置片段即可：  
 &lt;mvc:annotation-driven /&gt;  
如果JAXB位于类路径上，它会注册一个用于读写XML的转换器；如果Jackson位于类路径上，它会注册一个用于读写JSON的转换器。

准备响应  
典型的响应需要准备响应代码、设定HTTP响应头、将数据放到响应体当中，还需要处理异常。

使用JAX-RS设定响应体数据  
在JAX-RS中，要想将数据加到响应体中，只需要从资源方法中返回对象即可：

@GET

@Path\("{username}"\)

public Account getAccount\(@PathParam\("username"\) String username\) {  
    return accountRepository.findAccountByUsername\(username\);  
}

JAX-RS会寻找类型MessageBodyWriter的实体供应者，它能将对象转换为所需的内容类型。

JAX-RS实现需要具备一个JAXB MessageBodyWriter，这可以使用具有注解@Provider的客户化MessageBodyWriter实现。

使用Spring MVC设定响应体数据  
在Spring MVC中，响应是通过一个视图解析过程来实现的，这样就可以从一系列视图技术中选择了。但在与Web Services客户端交互时，

更加合理的方式则是舍弃视图解析过程，转而使用方法所返回的对象：

@RequestMapping\(

value="/{username}", method=GET\)

public @ResponseBody Account

getAccount\(@PathVariable String username\) {

return accountRepository.

findAccountByUsername\(username\);  
}

如果对控制器方法或其返回类型应用注解@ResponseBody，那么就会使用HttpMessageConverter处理返回值，然后用该返回值设定响应体。

用于请求体参数的HttpMessageConverter集合也用于响应体，因此无需再做任何配置。

状态代码与响应头  
JAX-RS使用一个链式API来构建响应：  
@PUT

@Path\("{username}"\)

public Response updateAccount\(Account account\) {  
    // ...  
    return Response.noContent\(\).build\(\);    // 204 \(No Content\)  
}  
这可以与UriBuilder联合使用来为Location响应头创建实体链接：

@POST  
public Response createAccount\(Account account\) {  
    // ...  
    URI accountLocation = uriInfo.getAbsolutePathBuilder\(\).path\(account.getUsername\(\)\).build\(\);  
    return Response.created\(accountLocation\).build\(\);  
}  
上面代码中所用的uriInfo要么被注入到根资源（使用了@Context）中，要么是从父资源传递给子资源。它可以附加到当前请求的路径之后。  
Spring MVC提供了一个注解来设定响应代码：

@RequestMapping\(method=PUT\)  
@ResponseStatus\(HttpStatus.NO\_CONTENT\)  
public void updateAccount\(@RequestBody Account account\) {  
    // ...  
}

可以直接使用HttpServletResponse对象设定Location头：

@RequestMapping\(method=POST\)  
@ResponseStatus\(CREATED\)  
public void createAccount\(

```
    @RequestBody Account account, 

    HttpServletRequest request,
    HttpServletResponse response\) {
// ...
String requestUrl = request.getRequestURL\(\).toString\(\);
URI uri = new UriTemplate\("{requestUrl}/{username}"\).expand\(requestUrl, account.getUsername\(\)\);
response.setHeader\("Location", uri.toASCIIString\(\)\);
```

}

异常处理  
JAX-RS允许资源方法抛出WebApplicationException类型的异常，该异常会包含一个响应。

下面的示例代码将一个JPA NoResultException转换为特定于Jersey的NotFoundException，这会导致一个404的错误：  
@GET  
@Path\("{username}"\)  
public Account getAccount\(@PathParam\("username"\) String username\) {  
    try {  
        return accountRepository.findAccountByUsername\(username\);  
    } catch \(NoResultException e\) {  
        throw new NotFoundException\(\);  
    }  
}  
WebApplicationException实例会封装必要的逻辑来生成特定的响应，但每个独立的资源类方法中都需要捕获异常。  
Spring MVC支持定义控制器级别的方法来处理异常：

@Controller  
@RequestMapping\("/accounts"\)  
public class AccountController {

```
@ResponseStatus\(NOT\_FOUND\)
@ExceptionHandler\({NoResultException.class}\)
public void handle\(\) {
    // ...
}
```

}  
如果任何控制器方法抛出了JPA的NoResultException异常，上面的处理器方法就会得到调用并处理该异常，然后返回一个404错误。

这样，每个控制器就都能处理异常了，好象来自同一个地方一样。

总结一下，从上述分析可以看出各有各的有限。比如springMVC参数上更加简单，而比如在客户化reader和writer上面jersey更加灵活。

后面讲解springMVC的时候，也会展示利用springMVC怎么实现这样的功能。

除了这两个框架之外还简单介绍其他的几个框架，供大家参考。

Restlet

Restlet支持JAX-RS \(就像 Jersey\)，完全抛弃了Servlet API，作为替代，自己实现了一套API。能够支持复杂的REST架构设计。

Restlet 帮助Java程序员建立大规模的快速的符合 RESTful 架构模式的web api。  
它提供了强大的路由和 filtering 系统。统一的client/server Java API.

满足所有主要的平台 \(Java SE/EE, Google AppEngine, OSGi, GWT, Android\) 以及提供了无数的扩展以满足程序员的需求。  
据我说知，它是第一个 java RESTful web 框架。

它的组要特点有：

1）自身没有包括与Spring的集成，可以使用第三方代码与Spring集成，集成难度较大。

2）文档不丰富，学习起来较困难，学习曲线算是比较陡峭了。关闭的社区，尽管 StackOverflow 上还是开放的。

3）灵活性较高，支持更多的REST概念，支持透明的内容协商，适合于开发更强大的REST组件，属于企业级的框架的强大框架。

4）零配置文件，全部配置通过代码完成。这个理念和spring boot一致了。

5）模块化

6）智能的url绑定， 全功能的 URI 路由

很多公司都在用它，包括我们公司平台的第一版也是使用这个的。基于上述集成困难，而且关闭的设计，最后还是决定放弃。后来使用SpringMVC

做了全面替换。

Play Framework

没有实现JAX-RS标准，但是也有自己的特色。

使用它很容易地创建，构建和发布 web 应用程序，支持 Java & Scala。它使用Akka, 基于一个轻量级的无状态的架构。

它应该应用于大规模地低CPU和内存消耗的应用。

主要特点如下：

1）基于 Netty, 支持非阻塞的 I/O. 并行处理远程调用的时候很优秀。

2）快速的项目构建和启动

3）SBT构建工具号称 Maven 杀手, 但是从没有优秀到替换它。难以学习和配置。

4）非 servlet，支持Async同时也支持REST, JSON/XML, Web Sockets, non-blocking I/O等。

5）不向后兼容; Play 2.X 重写了。

从上述的不同年代，不同角度分析了springMVC的竞争对手，也着重讲了Java Restful的一些框架。

最后总结下来，还是springMVC最时候我们公司的技术要求。因为我们选择了SpringBoot和Spring

cloud，这样和springMVC不需要任何整合和集成工作。而且springMVC即使在rest方面也有不错的

表现。这个社区也很活跃，出现问题很快就能找到答案，或者获取帮助。

当然它也有它的一些不足的地方，正如上面我们提到的没有办法比较方便的controller里面制定消息转化器。

jersey是提供了相应的功能的，而且项目中和别的老系统集成的时候，这个功能的确也是必要的。

为了能快速的做一些客制化功能，同时不破坏它已有的设计，对它进行深一步的了解是很有必要的。

下面我们就开始springMVC的学习之旅。

springMVC的处理流程图

![](/assets/屏幕快照 2018-04-07 下午1.33.33.png)

⑴ 用户发送请求至前端控制器DispatcherServlet

⑵ DispatcherServlet收到请求调用HandlerMapping处理器映射器。

⑶ 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器\(如果有则生成\)一并返回给DispatcherServlet。

⑷ DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

⑸ 执行处理器\(Controller，也叫后端控制器\)。

⑹ Controller执行完成返回ModelAndView

⑺ HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

⑻ DispatcherServlet将ModelAndView传给ViewReslover视图解析器

⑼ ViewReslover解析后返回具体View

⑽ DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

⑾ DispatcherServlet响应用户。

从上面可以看出，DispatcherServlet有接收请求，响应结果，转发等作用。有了DispatcherServlet之后，可以减少组件之间的耦合度。

同时从图中可以看出，SpringMVC本质上是一个Servlet,这个 Servlet 继承自 HttpServlet。FrameworkServlet负责初始化SpringMVC的容器，

并将Spring容器设置为父容器。这几点从DispatcherServlet也能看出。

![](/assets/DispatcherServlet.png)DispatcherServlet的initStrategies方法会初始化9大组件，但是这里将实现一些SpringMVC的最基本的组件而不是全部。

初始化阶段完成如下几件事情：

1\)加载配置文件

2\)扫描用户配置包下面所有的类

3\)拿到扫描到的类，通过反射机制，实例化。并且放到ioc容器中\(Map的键值对  beanName-bean\) beanName默认是首字母小写

4\)初始化HandlerMapping，这里其实就是把url和method对应起来放在一个k-v的Map中,在运行阶段取出

代码DispatcherServlet中通过initStrategies方法：

protected void initStrategies\(ApplicationContext context\) {

```
    initMultipartResolver\(context\);

    initLocaleResolver\(context\);

    initThemeResolver\(context\);

    initHandlerMappings\(context\);

    initHandlerAdapters\(context\);

    initHandlerExceptionResolvers\(context\);

    initRequestToViewNameTranslator\(context\);

    initViewResolvers\(context\);

    initFlashMapManager\(context\);

}
```

然后运行期每一次请求将会调用doGet或doPost方法，所以统一运行阶段都放在doDispatch方法里处理，它会根据url请求去HandlerMapping中匹配到对应的Method，

然后利用反射机制调用Controller中的url对应的方法，并得到结果返回。

1）是否是文件上传

2）查找对应的Handler

3）获取请求传入的参数并处理参数

4）是否是异步处理

5）view或返回结果处理

6）异常处理

7）完成后的事件派发等等

HandlerMapping

在dispatcherServlet,doDispatch方法中有调用getHandler，代码如下：

其中List&lt; HandlerMapping&gt; handlerMappings是dispatcherServlet的内部变量。

那该方法的内容就是遍历handlerMappings,获得符合条件的HandlerMapping,调用其getHandler方法，返回获得的HandlerExecutionChain。

protected HandlerExecutionChain getHandler\(HttpServletRequest request\) throws Exception {

```
    for \(HandlerMapping hm : this.handlerMappings\) {

        if \(logger.isTraceEnabled\(\)\) {

            logger.trace\(

                    "Testing handler map \[" + hm + "\] in DispatcherServlet with name '" + getServletName\(\) + "'"\);

        }

        HandlerExecutionChain handler = hm.getHandler\(request\);

        if \(handler != null\) {

            return handler;

        }

    }

    return null;

}
```

HandlerMapping是一个接口，内部只有一个方法和诺干变量，它的作用是根据request找到对应的Handler。方法如下：

HandlerExecutionChain getHandler\(HttpSevletRequest request\) throws Exception

接下来看看一个该方法的实现，AbstractHandlerMapping,代码如下：

@Override

public final HandlerExecutionChain getHandler\(HttpServletRequest request\) throws Exception {

```
    Object handler = getHandlerInternal\(request\);

    if \(handler == null\) {

        handler = getDefaultHandler\(\);

    }

    if \(handler == null\) {

        return null;

    }

    // Bean name or resolved handler?

    if \(handler instanceof String\) {

        String handlerName = \(String\) handler;

        handler = getApplicationContext\(\).getBean\(handlerName\);

    }



    HandlerExecutionChain executionChain = getHandlerExecutionChain\(handler, request\);

    if \(CorsUtils.isCorsRequest\(request\)\) {

        CorsConfiguration globalConfig = this.corsConfigSource.getCorsConfiguration\(request\);

        CorsConfiguration handlerConfig = getCorsConfiguration\(handler, request\);

        CorsConfiguration config = \(globalConfig != null ? globalConfig.combine\(handlerConfig\) : handlerConfig\);

        executionChain = getCorsHandlerExecutionChain\(request, executionChain, config\);

    }

    return executionChain;

}
```

另外一个要讨论的就是顺序问题，不同hander负责映射的条件可能有重复的，这时候就需要定义不同的HandlerMapping执行的顺序，这里的顺序可以通过实现Order接口，通过Order属性定义。order越小越先使用。

如果发现有两个相同的handler则会抛出异常。所以当我们书写RequestMapping的时候value部分和method部分最好有一个规范，同时不要省略。因为SpringMVC的匹配功能很强大，有很多默认的

推算，所以当requestHandler相对比较少的时候，大家都可以相安无事。但是随着系统复杂，requestHandler的增加，一些错误就会显示出来。而且对Swagger等地方工具抽取出来的内容也会不完善。

所以建议大家能有统一的一个规范，形成一套完整的API信息。

在AbstractHandlerMethodMapping中也相应的实现。

/\*\*

```
 \* Look up the best-matching handler method for the current request.

 \* If multiple matches are found, the best match is selected.

 \* @param lookupPath mapping lookup path within the current servlet mapping

 \* @param request the current request

 \* @return the best-matching handler method, or {@code null} if no match

 \* @see \#handleMatch\(Object, String, HttpServletRequest\)

 \* @see \#handleNoMatch\(Set, String, HttpServletRequest\)

 \*/

protected HandlerMethod lookupHandlerMethod\(String lookupPath, HttpServletRequest request\) throws Exception {

    List&lt;Match&gt; matches = new ArrayList&lt;Match&gt;\(\);

    List&lt;T&gt; directPathMatches = this.mappingRegistry.getMappingsByUrl\(lookupPath\);

    if \(directPathMatches != null\) {

        addMatchingMappings\(directPathMatches, matches, request\);

    }

    if \(matches.isEmpty\(\)\) {

        // No choice but to go through all mappings...

        addMatchingMappings\(this.mappingRegistry.getMappings\(\).keySet\(\), matches, request\);

    }



    if \(!matches.isEmpty\(\)\) {

        Comparator&lt;Match&gt; comparator = new MatchComparator\(getMappingComparator\(request\)\);

        Collections.sort\(matches, comparator\);

        if \(logger.isTraceEnabled\(\)\) {

            logger.trace\("Found " + matches.size\(\) + " matching mapping\(s\) for \[" +

                    lookupPath + "\] : " + matches\);

        }

        Match bestMatch = matches.get\(0\);

        if \(matches.size\(\) &gt; 1\) {

            if \(CorsUtils.isPreFlightRequest\(request\)\) {

                return PREFLIGHT\_AMBIGUOUS\_MATCH;

            }

            Match secondBestMatch = matches.get\(1\);

            if \(comparator.compare\(bestMatch, secondBestMatch\) == 0\) {

                Method m1 = bestMatch.handlerMethod.getMethod\(\);

                Method m2 = secondBestMatch.handlerMethod.getMethod\(\);

                throw new IllegalStateException\("Ambiguous handler methods mapped for HTTP path '" +

                        request.getRequestURL\(\) + "': {" + m1 + ", " + m2 + "}"\);

            }

        }

        handleMatch\(bestMatch.mapping, lookupPath, request\);

        return bestMatch.handlerMethod;

    }

    else {

        return handleNoMatch\(this.mappingRegistry.getMappings\(\).keySet\(\), lookupPath, request\);

    }

}
```

HandlerAdapter

在dispatcherServlet通过如下方法获得HandlerAdapter，其中List&lt; HandlerAdapter&gt; handlerAdapters是dispatcherServlet的成员变量，

可以看到它的逻辑是遍历所有的Adapter，然后检查哪个可以处理当前的Handler，找到第一个可以处理Handler的Adapter后停止查找，返回。这里的顺序同样是通过Order属性设置的。

protected HandlerAdapter getHandlerAdapter\(Object handler\) throws ServletException {

```
    for \(HandlerAdapter ha : this.handlerAdapters\) {

        if \(logger.isTraceEnabled\(\)\) {

            logger.trace\("Testing handler adapter \[" + ha + "\]"\);

        }

        if \(ha.supports\(handler\)\) {

            return ha;

        }

    }

    throw new ServletException\("No adapter for handler \[" + handler +

            "\]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler"\);

}
```

HandlerAdapter的角色是使用工具\(handler\)的人。因为handler是Object类型，需要HandlerAdapter使用它来完成一定格式要求的任务。

它是一个接口，代码如下：

public interface HandlerAdapter {

```
    boolean supports\(Object handler\);   

    ModelAndView handle\(HttpServletRequest request, HttpServletResponse response, Object handler\) throws Exception;

   long getLastModified\(HttpServletRequest request, Object handler\);
```

public class SimpleControllerHandlerAdapter implements HandlerAdapter {

```
@Override

public boolean supports\(Object handler\) {

    return \(handler instanceof Controller\);

}



@Override

public ModelAndView handle\(HttpServletRequest request, HttpServletResponse response, Object handler\)

        throws Exception {



    return \(\(Controller\) handler\).handleRequest\(request, response\);

}



@Override

public long getLastModified\(HttpServletRequest request, Object handler\) {

    if \(handler instanceof LastModified\) {

        return \(\(LastModified\) handler\).getLastModified\(request\);

    }

    return -1L;

}
```

}

可以看到这个Adapter比较简单，它要求handler实现了Controller接口，方法的实现是通过处理器的handleRequest方法。

HandlerExceptionResolver

HandlerExceptionResolver是SpringMVC中专门负责处理异常的类。它负责：

根据异常设置ModelAndView。

之后交给render方法进行渲染。render只负责将Model渲染成页面。具体ModelAndView的来源render并不关心。

public interface HandlerExceptionResolver {

```
ModelAndView resolveException\(

        HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex\);
```

}

它的结构很简单，只有一个方法，只需要从异常解析出ModelAndView就可以了。具体实现可以维护一个异常为Key、View为value的Map，解析时直接从Map里获取View。

ViewResolver

ViewResolver用来将String类型的视图名（也叫逻辑视图）和Locale解析为View类型的视图，ViewResolver接口也很简单，代码如下

public interface ViewResolver {

```
View resolveViewName\(String viewName, Locale locale\) throws Exception;
```

}

这里可以看到参数是viewName和locale，不过一般我们只要根据视图名找到视图，然后渲染就可以，如果需要国际化支持也只要将显示的内容或者主题使用国际化支持。

View是用来渲染页面的，也就是将程序返回的参数填入模板中，生成html或其他格式的文件。

但是在我们的系统中，主要使用springMVC的restful service这一块功能，所以对view层是不做进一步的说明。

RequestToViewNameTranslator

ViewResolver根据ViewName查找View，但有的Handler处理完并没有设置View，也没有设置viewName，这时就需要从request中获取viewName。

也就是RequestToViewNameTranslator的任务。它是个接口，代码如下：

public interface RequestToViewNameTranslator {

```
String getViewName\(HttpServletRequest request\) throws Exception;
```

}

就像之前说的，他只有一个getViewName方法，能够从request获取到viewName就可以了。

RequestToViewNameTranslator在Spring MVC容器中只能配置一个，所以所有request到ViewName的转换规则都要在一个Translator里面实现。这里也不做过多的介绍。

LocaleResolver

ViewResolver有两个参数，viewname和Locale，viewname来自Handler或RequestToViewNameTranslator。locale变量就来自LocaleResolver。

LocaleResolver用于从request解析出Locale。

LocaleResolver是个接口，代码如下：

public interface LocaleResolver {

```
Locale resolveLocale\(HttpServletRequest request\);

void setLocale\(HttpServletRequest request, HttpServletResponse response, Locale locale\);
```

}

一共只有两个方法，第一个方法就是起到获取Locale的作用，在介绍doService方法时说过，容器会将localeResolver设置到request的attribute中。

代码如下：

request.setAttribute\(LOCALE\_RESOLVER\_ATTRIBUTE,this.localeResolver\);

切换Locale:

第二个方法可以将Locale设置到request中。

SpringMVC提供了统一修改request中Locale的机制，就是我们在分析doDispatch时见过的Interceptor。SpringMVC已经写好现成的了，配置一下就可以，也就是org.springframework.web.servlet.i18n.LocaleChangeInterceptor.

配置方法如：

&lt;mvc:interceptors&gt;

```
&lt;mvc:interceptor&gt;

    &lt;mvc:mapping path="/\*"/&gt;

    &lt;bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/&gt;

    //这里也可以自定义参数的名称，如：

    //&lt;bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" p:paramName="lang"/&gt;

&lt;/mvc:interceptor&gt;
```

&lt;/mvc:interceptors&gt;

这样通过request的locale参数就可以修改Locale了。

用到Locale的地方有两处：

1、ViewResolver解析视图的时候。

2、使用到国际化资源或者主题的时候。国际化资源或主题主要使用RequestContext的getMessage和getThemeMessage方法。

ThemeResolver

个接口，源码如下：

public interface ThemeResolver {

```
String resolveThemeName\(HttpServletRequest request\);

void setThemeName\(HttpServletRequest request, HttpServletResponse response, String themeName\);
```

}

SpringMVC中一套主题对应一个properties文件，里面存放着跟当前主题相关的所有资源，如图片、css样式表等。

SpringMVC跟主题有关的类主要有ThemeResolver、ThemeSource和Theme。

1. ThemeResolver的作用是从request解析出主题名。

2. ThemeSource的作用是根据主题名找到具体的主题。

3. Theme是ThemeSource找出的一个具体的主题。

因为也是和view层相关，所以也不做进一步说明。想了解的可以查看一下spring这一块相关的资料。

MultipartResolver

用于处理上传请求，处理方法时将普通的request包装成MultipartHttpServletRequest，后者可以直接调用getFile方法获取File，如果上传多个文件，

还可以调用getFileMap得到FileName-&gt;File结构的Map，这样就使得上传请求的处理变得非常简单。

然后，其实不包装直接用request也可以。所以SpringMVC中此组件没有提供默认值。MultipartResolver代码如下：

public interface MultipartResolver {

```
boolean isMultipart\(HttpServletRequest request\);

//判断是不是上传请求

MultipartHttpServletRequest resolveMultipart\(HttpServletRequest request\) throws MultipartException;

 //将request包装成MultipartHttpServletRequest

void cleanupMultipart\(MultipartHttpServletRequest request\);

//清除上传过程中产生的临时资源
```

}

FlashMapManager

FlashMap在之前的文章1中提到过了，主要用在redirect中传递参数。而FlashMapManager是用来管理FlashMap的，定义如下：

public interface FlashMapManager {

```
FlashMap retrieveAndUpdate\(HttpServletRequest request, HttpServletResponse response\);

 //恢复参数，将恢复过的和超时的参数从保存介质中删除；

void saveOutputFlashMap\(FlashMap flashMap, HttpServletRequest request, HttpServletResponse response\);

//用于将参数保存起来。
```

}

默认实现是org.springframework.web.servlet.support.SessionFlashMapManager。它将参数保存在session中，实现原理就是利用session作为中转站保存request中的参数，达到redirect传递参数的。



结合springboot开发一个能够根据controller能够客制化的messageConvert这么一个需求。

在实现这个需求之前，我们先介绍两个知识点，一个是ResponseBody。

在springMVC中有两种类型的handler返回ResponseBody，而不是一个ModelView。

1）返回值类型是HttpEntity类型

2）处理方法前增加了注解@ResponseBody

SpringMVC4.1中还新增加了ResponseBodyAdvise接口，实现接口的类可以修改作为ReponseBody返回值。

ResponseBodyAdvise实现类，主要修改json类型的返回值。ResponseBodyAdvise的实现类会被ReturnValueHandler调用，

处理Handler的返回值。

另外一个springBoot保持了一贯的制动化配置功能，如果发现classpath里面com.fasterxml.jackson.core.JsonGenerator

类时候，会设置jackson2Present为true，系统的自动调用相关的messageCovert进行转化数据并作为返回结果，而不是去匹配一个试图。所以SpringMVC对Restful service的支持也是很简单的。

同时SpringWeb中提供了很多默认的HttpMessageConverter，基本满足了大部分需求。

通过MappingJackson2HttpMessageConverter的类图来一个大概类图：

![](/assets/MappingJackson2HttpMessageConverter.png)

结合springboot开发一个能够根据controller能够客制化的messageConvert这么一个需求。

在实现这个需求之前，我们先介绍两个知识点，一个是ResponseBody。

在springMVC中有两种类型的handler返回ResponseBody，而不是一个ModelView。

1）返回值类型是HttpEntity类型

2）处理方法前增加了注解@ResponseBody

SpringMVC4.1中还新增加了ResponseBodyAdvise接口，实现接口的类可以修改作为ReponseBody返回值。

ResponseBodyAdvise实现类，主要修改json类型的返回值。ResponseBodyAdvise的实现类会被ReturnValueHandler调用，

处理Handler的返回值。



另外一个springBoot保持了一贯的制动化配置功能，如果发现classpath里面com.fasterxml.jackson.core.JsonGenerator

类时候，会设置jackson2Present为true，系统的自动调用相关的messageCovert进行转化数据并作为返回结果，而不是去匹配一个试图。

所以SpringMVC对Restful service的支持也是很简单的。



同时SpringWeb中提供了很多默认的HttpMessageConverter，基本满足了大部分需求。

通过MappingJackson2HttpMessageConverter的类图来一个大概类图：



里面很重要的几个方法就是

判断目标的class和媒体类型，这个messageConvert是否能反序列化成功。

boolean canRead\(Class&lt;?&gt; clazz, MediaType mediaType\);



判断这个返回值的类型和这个媒体类型，当前messageCovert能否序列化成功

boolean canWrite\(Class&lt;?&gt; clazz, MediaType mediaType\);



获取当前messageConvert支持的媒体类型

List&lt;MediaType&gt; getSupportedMediaTypes\(\);



去反序列化inputMessage到目标的class类型

T read\(Class&lt;? extends T&gt; clazz, HttpInputMessage inputMessage\)

			throws IOException, HttpMessageNotReadableException;



把当前class类型的outputMessage反序列化到到当前媒体类型的，

void write\(T t, MediaType contentType, HttpOutputMessage outputMessage\)

			throws IOException, HttpMessageNotWritableException;





上面所说的序列化和反序列化并非java里面的那个序列化，正确理解为转化为制定的媒体格式类型，比如json或xml等。

虽然根据上面的方法，我们知道可以根据媒体类型，或目标class完成一些客户化的messageConvert。

但是真实需求里面，有些情况不是根据这两个参数维度来实现的话，就需要HandlerMethodReturnValueHandler层做设置了。

从下图的类图中，可以看出可以在writeWithMessageConverters方法中：

![](/assets/EbaoRequestResponseBodyMethodProcessor.png) public &lt;T&gt; void writeWithMessageConverters\(T value, MethodParameter returnType,

                                               ServletServerHttpRequest inputMessage, ServletServerHttpResponse outputMessage\)

            throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException 



通过MethodParameter returnType这里方法returnType.getMethod\(\).getDeclaringClass\(\)获取controller的class信息，可以通过annotation等来增加自己

客户化逻辑实现。我在实际中就是通过增加一个annotation，根据里面指定的messageConvert来进行转化。

Class messgeConverterClass = returnType.getMethod\(\).getDeclaringClass\(\).getAnnotation\(MessageCovertCustomization.class\) == null ? null 

	: returnType.getMethod\(\).getDeclaringClass\(\).getAnnotation\(MessageCovertCustomization.class\).value\(\);

for \(HttpMessageConverter&lt;?&gt; messageConverter : this.messageConverters\) {

                if \(isNativeMessageConvertIgnoreUrl\(url\) && messageConverter.getClass\(\).getAnnotation\(EbaoCustomizationMessageConvert.class\) != null\) {

                    continue;

                }

                if \(messageConverter.getClass\(\).equals\(messgeConverterClass\)\) {

		\\….

	}



由于在springMVC中，returnValueHandler是使用的责任链模式，通过HandlerMethodReturnValueHandlerComposite来实现的。

可以通过Order让客户化的messageConvert在前面，但是有些情况可能仅仅通过order还不能满足，需要一定的客户化逻辑，比如业务代码

需要使用客户化的messageConvert，但是系统自带那些endpoint可能需要系统自带的messageConvert等等，可能还需要对

HandlerMethodReturnValueHandlerComposite做客制化了。

事实上我们也是这么做的。

![](/assets/EbaoHandlerMethodReturnValueHandlerComposite.png)

由于在springMVC中，returnValueHandler是使用的责任链模式，通过HandlerMethodReturnValueHandlerComposite来实现的。

可以通过Order让客户化的messageConvert在前面，但是有些情况可能仅仅通过order还不能满足，需要一定的客户化逻辑，比如业务代码需要使用客户化的messageConvert，但是系统自带那些endpoint可能需要系统自带的messageConvert等等，可能还需要对

HandlerMethodReturnValueHandlerComposite做客制化了。

springMVC中有一些默认的处理能力具有的时候，自己添加的组件可能没有办法用到。

当需要用自己的时候，需要客户化那些adapter等等。事实上我们也是这么做的。

可以通过重载handleReturnValue

 @Override

    public void handleReturnValue\(Object returnValue, MethodParameter returnType,

                                  ModelAndViewContainer mavContainer, NativeWebRequest webRequest\) throws Exception {



        //TODO 客户化代码

        HandlerMethodReturnValueHandler handler = selectHandler\(returnValue, returnType, webRequest\);

        if \(handler == null\) {

            throw new IllegalArgumentException\("Unknown return value type: " + returnType.getParameterType\(\).getName\(\)\);

        }

        handler.handleReturnValue\(returnValue, returnType, mavContainer, webRequest\);

    }



可以可以客户化相应的逻辑选择对应的handler

 private HandlerMethodReturnValueHandler selectHandler\(Object value, MethodParameter returnType, NativeWebRequest webRequest\) {

        String ulr = \(\(javax.servlet.http.HttpServletRequest\) webRequest.getNativeRequest\(\)\).getRequestURI\(\);

        if \(isNativeMessageConvertIgnoreUrl\(ulr\) \|\| returnType.getMethod\(\).getDeclaringClass\(\).getAnnotation\(MessageCovertCustomization.class\) != null\) {

            for \(HandlerMethodReturnValueHandler handler : this.getHandlers\(\)\) {

                if \(handler instanceof EbaoRequestResponseBodyMethodProcessor\) {

                    return handler;

                }

            }

        } else {

            boolean isAsyncValue = isAsyncReturnValue\(value, returnType\);

            for \(HandlerMethodReturnValueHandler handler : this.getHandlers\(\)\) {

                if \(\(isAsyncValue && !\(handler instanceof AsyncHandlerMethodReturnValueHandler\)\) \|\| \(handler instanceof EbaoRequestResponseBodyMethodProcessor\)\) {

                    continue;

                }

                if \(handler.supportsReturnType\(returnType\)\) {

                    return handler;

                }

            }

            return null;

        }

        return null;

    }

最后我们可以通过集成WebMvcConfigurationSupport，来实现增加ReturnValueHandlerComposite

@Configuration

@EnableWebMvc

public class WebConfig extends WebMvcConfigurationSupport implements WebMvcConfigurer {



@Override

    public void addReturnValueHandlers\(List&lt;HandlerMethodReturnValueHandler&gt; returnValueHandlers\) {

        EbaoHandlerMethodReturnValueHandlerComposite ebaoHandlerMethodReturnValueHandlerComposite = new EbaoHandlerMethodReturnValueHandlerComposite\(\);

        EbaoRequestResponseBodyMethodProcessor ebaoRequestResponseBodyMethodProcessor = new EbaoRequestResponseBodyMethodProcessor\(getMessageConverters\(\)\);

        ebaoHandlerMethodReturnValueHandlerComposite.addHandler\(ebaoRequestResponseBodyMethodProcessor\);

        ebaoRequestResponseBodyMethodProcessor.setMessageConverters\(getMessageConverters\(\)\);

        ebaoHandlerMethodReturnValueHandlerComposite.addHandlers\(ebaoRequestResponseBodyMethodProcessor.getDefaultReturnValueHandlers\(\)\);

        returnValueHandlers.add\(0, ebaoHandlerMethodReturnValueHandlerComposite\);

    }

}



这样一条龙的客户化就完成了。可以自由的使用自己的messageConvert了。



如果还要进一步交接messageConvert的话，就要知道jackson序列化框架和JavaType的一些知识了。













2。序列化实现的比较（Jackson等）

