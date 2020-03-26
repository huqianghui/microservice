应用服务微服务化之后，系统的复杂度增加了，同时可能暴露给外面的端口，API等相应的可能也会更加细力度化给系统的安全也会带来一定程度的影响。想要的代码监测和系统安全检测也要跟上。

微服务系统安全根据运行的位置和角色来区分大致可以分成三个：

a）浏览器的客户端安全

b）web容器安全

c） 应用程序安全

下面来一一介绍相关的概念和防御方法。

首先来介绍浏览器的客户端安全的相关知识

在浏览器的安全开发和运维中，遇到最多的一个问题就是跨域请求和跨域攻击。在说这个问题之前，我们首先要弄清楚几个概念。

1）同源策略

同源策略是浏览器最核心最基本的安全功能。它限制了来自不同源的“document”或脚本，对当前“document”读取或者设置某些属性。

所谓同源是指host（域名或者ip地址）、子域名、端口和协议相同。不同源的客户端脚本\(javascript、ActionScript\)在没明确授权的情况下，不能读写对方的资源。

但需要注意的是，对于当前页面来说，页面存放js文件的域并不重要，重要的是加载js的页面所在域是什么。

例如：在a.com的页面下通过&lt;script src='\[\[\[[http://b.com/b.js'&gt;&lt;/script&gt;获取了b.com域中的资源，但是由于b.js是运行在a.com的页面中，因此对于当前页面来讲，b.js的域就是a.com。\]\(http://b.com/b.js'&gt;\]\(http://b.com/b.js'&gt;\]\(http://b.com/b.js'&gt;](http://b.com/b.js'></script>获取了b.com域中的资源，但是由于b.js是运行在a.com的页面中，因此对于当前页面来讲，b.js的域就是a.com。]%28http://b.com/b.js'>]%28http://b.com/b.js'>]%28http://b.com/b.js'>)&lt;/script&gt;获取了b.com域中的资源，但是由于b.js是运行在a.com的页面中，因此对于当前页面来讲，b.js的域就是a.com。\]%28[http://b.com/b.js'&gt;\]\(http://b.com/b.js'&gt;\)&lt;/script&gt;获取了b.com域中的资源，但是由于b.js是运行在a.com的页面中，因此对于当前页面来讲，b.js的域就是a.com。\]\(\[http://b.com/b.js'&gt;\)&lt;/script&gt;获取了b.com域中的资源，但是由于b.js是运行在a.com的页面中，因此对于当前页面来讲，b.js的域就是a.com。\]\(http://b.com/b.js'&gt;\)](http://b.com/b.js'>]%28http://b.com/b.js'>%29</script>获取了b.com域中的资源，但是由于b.js是运行在a.com的页面中，因此对于当前页面来讲，b.js的域就是a.com。]%28[http://b.com/b.js'>%29</script>获取了b.com域中的资源，但是由于b.js是运行在a.com的页面中，因此对于当前页面来讲，b.js的域就是a.com。]%28http://b.com/b.js'>%29)&lt;/script&gt;获取了b.com域中的资源，但是由于b.js是运行在a.com的页面中，因此对于当前页面来讲，b.js的域就是a.com。\)\)

在浏览器中，&lt;script&gt;、&lt;img&gt;、&lt;iframe&gt;、&lt;link&gt;等标签都可以跨域加载资源，而不受同源策略的限制，这些带src属性的标签每次加载时，实际上时由浏览器发起了一次get请求，通过src属性加载的资源，浏览器限制了javascript的权限，使其不能读、写返回的内容。

由于这个漏洞能够跨域读取页面内容，因此绕过了同源策略，成为一个跨域漏洞。

2）跨域资源共享\(Cross-Origin Resource Sharing, CORS\)

受浏览器的同源策略限制，JavaSript只能请求本域内的资源。

跨域资源共享\(Cross-Origin Resource Sharing, CORS\)是为解决Ajax技术难实现跨域问题而提出的一个规范，这个规范试着从根本上解决安全的跨域资源共享问题。

在此之前，解决此类问题的途径往往是服务器代理、JSONP等，治标不治本。目前基本所有浏览器都已经支持该规范。

CORS约定服务器端和浏览器在HTTP协议之上，通过一些额外HTTP头部信息，进行跨域资源共享的协商。

服务器端和浏览器都必需遵循规范中的要求。

CORS把HTTP请求分成两类，不同类别按不同的策略进行跨域资源共享协商。

1. 简单跨域请求。

当HTTP请求出现以下两种情况时，浏览器认为是简单跨域请求：

1\). 请求方法是GET、HEAD或者POST，并且当请求方法是POST时，Content-Type必须是application/x-www-form-urlencoded, multipart/form-data或着text/plain中的一个值。

2\). 请求中没有自定义HTTP头部。

对于简单跨域请求，浏览器要做的就是在HTTP请求中添加Origin Header，将JavaScript脚本所在域填充进去，向其他域的服务器请求资源。

服务器端收到一个简单跨域请求后，根据资源权限配置，在响应头中添加Access-Control-Allow-Origin Header。

浏览器收到响应后，查看Access-Control-Allow-Origin Header，如果当前域已经得到授权，则将结果返回给JavaScript。否则浏览器忽略此次响应。

1. 带预检\(Preflighted\)的跨域请求。

当HTTP请求出现以下两种情况时，浏览器认为是带预检\(Preflighted\)的跨域请求：

1\). 除GET、HEAD和POST\(only with application/x-www-form-urlencoded, multipart/form-data, text/plain Content-Type\)以外的其他HTTP方法。

2\). 请求中出现自定义HTTP头部。

带预检\(Preflighted\)的跨域请求需要浏览器在发送真实HTTP请求之前先发送一个OPTIONS的预检请求，检测服务器端是否支持真实请求进行跨域资源访问，

真实请求的信息在OPTIONS请求中通过Access-Control-Request-Method Header和Access-Control-Request-Headers Header描述，此外与简单跨域请求一样，浏览器也会添加Origin Header。

服务器端接到预检请求后，根据资源权限配置，在响应头中放入Access-Control-Allow-Origin Header、Access-Control-Allow-Methods和Access-Control-Allow-Headers Header，分别表示允许跨域资源请求的域、请求方法和请求头。

此外，服务器端还可以加入Access-Control-Max-Age Header，允许浏览器在指定时间内，无需再发送预检请求进行协商，直接用本次协商结果即可。

浏览器根据OPTIONS请求返回的结果来决定是否继续发送真实的请求进行跨域资源访问。这个过程对真实请求的调用者来说是透明的。

XMLHttpRequest支持通过withCredentials属性实现在跨域请求携带身份信息\(Credential，例如Cookie或者HTTP认证信息\)。

浏览器将携带Cookie Header的请求发送到服务器端后，如果服务器没有响应Access-Control-Allow-Credentials Header，那么浏览器会忽略掉这次响应。

这里讨论的HTTP请求是指由Ajax XMLHttpRequest对象发起的，所有的CORS HTTP请求头都可由浏览器填充，无需在XMLHttpRequest对象中设置。

以下是CORS协议规定的HTTP头，用来进行浏览器发起跨域资源请求时进行协商：

a. Origin。HTTP请求头，任何涉及CORS的请求都必需携带。

b. Access-Control-Request-Method。HTTP请求头，在带预检\(Preflighted\)的跨域请求中用来表示真实请求的方法。

c. Access-Control-Request-Headers。HTTP请求头，在带预检\(Preflighted\)的跨域请求中用来表示真实请求的自定义Header列表。

d. Access-Control-Allow-Origin。HTTP响应头，指定服务器端允许进行跨域资源访问的来源域。可以用通配符\*表示允许任何域的JavaScript访问资源，但是在响应一个携带身份信息\(Credential\)的HTTP请求时，Access-Control-Allow-Origin必需指定具体的域，不能用通配符。

e. Access-Control-Allow-Methods。HTTP响应头，指定服务器允许进行跨域资源访问的请求方法列表，一般用在响应预检请求上。

f. Access-Control-Allow-Headers。HTTP响应头，指定服务器允许进行跨域资源访问的请求头列表，一般用在响应预检请求上。

g. Access-Control-Max-Age。HTTP响应头，用在响应预检请求上，表示本次预检响应的有效时间。在此时间内，浏览器都可以根据此次协商结果决定是否有必要直接发送真实请求，而无需再次发送预检请求。

h. Access-Control-Allow-Credentials。HTTP响应头，凡是浏览器请求中携带了身份信息，而响应头中没有返回Access-Control-Allow-Credentials: true的，浏览器都会忽略此次响应。

所以只要是带自定义header的跨域请求，在发送真实请求前都会先发送OPTIONS请求，浏览器根据OPTIONS请求返回的结果来决定是否继续发送真实的请求进行跨域资源访问。所以复杂请求肯定会两次请求服务端。

js 端的ajax请求：

$.ajax\({

```
   url: "http://test.com",  

   dataType: 'json',  

   type: 'GET',  

   beforeSend: function \(xhr\) {  

       xhr.setRequestHeader\("Test", "testheadervalue"\);  

   },  

   async: false,  

   cache: false,  

   //contentType: 'application/x-www-form-urlencoded',  

   success: function \(sResponse\) {  

   }
```

}\);

服务端的action

//允许跨域访问

HttpServletResponse response = \(HttpServletResponse\) resp;

response.setHeader\("Access-Control-Allow-Origin", "\*"\);

response.setHeader\("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE, PUT"\);

response.setHeader\("Access-Control-Allow-Headers", "Content-Type, Content-Range, Content-Disposition, Content-Description, Cache-Control, No-Cache, Pragma, Accept, X-Requested-With, Authorization, FWDTOKEN, x-ebao-target-ip-address"\);

response.setHeader\("Access-Control-Allow-Credentials", "true"\);

3）跨站脚本攻击（XSS）

为了解决跨域的问题，上面做出了解释。同时也带来了一定的风险。

xss（cross site script），是指通过“html注入”篡改了网页，插入了恶意脚本，从而在用户浏览网页时，控制用户浏览器的一种攻击。由于css层叠样式表cascading style sheets相同缩写，所以就改成xss。

xss根据不同效果分为以下几种类型：

反射型xss（非持久型xss）：

简单地把用户输入的数据“反射”给浏览器。将用户输入的数据通过后端直接输出，跳转等。

解决方案：

a.对参数进行过滤：

如果要输出内容，将其中的“&lt;”,”&gt;”,” ‘ “,”&”,” “ “等转义输出，避免浏览器在输出时解析js代码，造成污染。

b.验证get参数，判断是否满足要求

对于不满足的做校验失败。

存储型xss（持久型xss）：

存储型xss会把用户输入数据“存储”在服务器端，所以这类攻击具有很强的稳定性。

解决方案：

a.对输出数据进行过滤，转义”&lt;“,”&gt;”,” ’ “, ” “,”&”等标签。

b. 设置cookie的值为http only。

基于DOM的xss：

这类xss效果上是反射型xss，但是形成原因是通过修改页面的DOM节点来达到攻击目的的。通过将内容，添加到DOM 上，造成恶意代码嵌入。

解决方案：

针对以上几种XSS攻击都有效

a. 设置cookie标记为http only

b. 只允许用户输入我们期望的数据。 例如：　年龄的textbox中，只允许用户输入数字。 而数字之外的字符都过滤掉。

c. 对数据进行Html Encode 处理

d. 过滤或移除特殊的Html标签， 例如: &lt;script&gt;, &lt;iframe&gt; ,  &lt; for &lt;, &gt; for &gt;, " for

f. 过滤JavaScript 事件的标签。例如 "onclick=", "onfocus" 等等。

通过上述分析可以看出都是通过一些特殊字符标签来实现攻击。

现在前后端分离之后，后端只提供数据，所以不应该有任何html，css，js相关的代码。现在流行的UI端框架比如ReactJs和vueJs等中，控件的输出都做了编码，让内容转码，不执行当前文本内容。

所以这类攻击在新一代的UI技术中都得到了有效的处理。

4）CSRF 跨站点请求伪造

CSRF是Cross Site Request Forgery的缩写，乍一看和XSS差不多的样子，但是其原理正好相反，XSS是利用合法用户获取其信息，而CSRF是伪造成合法用户发起请求。

用户登录后会把登录信息存放在服务器，客户端有一个用户标识存在cookie中，只要用户不关闭浏览器或者退出登录，在其有效期内服务器就会把这个浏览器发送的请求当作当前客户，

如果这时候用户被欺骗，使用浏览器打开了某些恶意网址，里面就会包含一些不是用户希望发送的请求，服务器也会把这些请求当作是当前客户发送的请求，这时候用户的个人信息、资金安全、如果用户权限高整个站点都可能会受到危害。

CSRF原理很简单，当用户登录以站点时用浏览器打开一恶意网址，就有可能遭受攻击。

必须同时满足两个条件才行。比如我们使用QQ，看看QQ zone，突然蹦出个包含中奖或者问卷调查链接的聊天窗口（或者是。。。），这个腾讯做了防范，但是我们收到封邮件包含此内容，很多用户会选择去点击。

解决方案：

a. 使用post，不使用get修改信息

b. 验证码，所有表单的提交需要验证码，但是貌似用起来很麻烦，所以一些关键的操作可以

c. 在表单中预先植入一些加密信息，验证请求是此表单发送

现在oath2，cas等协议中，都建议把认证信息放到header头部，而不是拼接在url后面，能起到一定的保护作用。

CSRF归根结底还是token认证安全问题，所以通过认证refer url，或者让token加入更多的信息也能保证一定的安全。

5）ClickJacking点击劫持

是一种视觉上的欺骗手段，通过伪造一个透明的iframe。当用户点击当前网站上功能型按钮，实际上点击的是iframe上的按钮

解决方案：

a.防御ClickJacking一般通过禁止跨域的iframe来防范。

设置X-Frame-Options，防止页面被frame

apache:Header always append X-Frame-Options SAMEORIGIN

nginx:add\_header X-Frame-Options SAMEORIGIN;

b.设置代码使其不能被frame，例如

\(function \(\) {

```
if \(window != window.top\) {

    window.top.location.replace\(window.location\); //或者干别的事情

}
```

}\)\(\);

6\) html5的安全问题

新的一代UI技术，大都是基于html5技术上进行扩展的。所以html5解决了一些旧的安全问题比如为iframe元素增加了sandbox属性防止不信任的Web页面执行某些操作，例如访问父页面的DOM、执行脚本、访问本地存储或者本地数据库等等。

但是html5同时也引入了一些一些的标签和隐患。下面来做一个简单的介绍。

a.包括上面提到的CORS攻击,这里就不在重复说明。

b.Web Storage攻击

HTML5支持WebStorage，开发者可以为应用创建本地存储，存储一些有用的信息。

例如LocalStorage可以长期存储，而且存放空间很大，一般是5M，极大的解决了之前只能用Cookie来存储数据的容量小、存取不便、容易被清除的问题。这个功能为客户端提供了极大的灵活性。

LocalStorage的API都是通过Javascript提供的，这样攻击者可以通过XSS攻击窃取信息，例如用户token或者资料。攻击者可以用下面的脚本遍历本地存储。

if\(localStorage.length\){

for\(I in localStorage\) {

console.log\(i\);

console.log\(localStorage.getItem\(i\)\);

```
    }
```

}

对于WebStorage攻击的防御措施是：

数据放在合适的作用域里

例如用户sessionID就不要用LocalStorage存储，而需要放在sessionStorage里。而用户数据不要储存在全局变量里，而应该放在临时变量或者局部变量里。

不要存储敏感的信息

因为我们总也无法知道页面上是否会存在一些安全性的问题，一定不要将重要的数据存储在WebStorage里。

c.WebSQL攻击

自从HTML5引入本地数据库和WebSQL之后，前端开发对于数据库的安全也必须要有所了解和警惕。

和后端一样，通过参数化来防止SQL注入。

d.Web Worker攻击

由于Javascript是单线程执行的，在执行过程中浏览器不能执行其它Javascript脚本，UI渲染线程也会被挂起，从而导致浏览器进入僵死状态。

使用WebWorker可以将计算过程放入一个新线程里去执行将避免这种情况的出现。

这样我们可以同时执行多个JS任务而不会阻塞浏览器，非常适合异步交互和大规模计算，这在以前是很难做到的。

e.劫持攻击

这个上面讲过了，不在重复。

f.API攻击

HTML5里有许多协议、模式和API，可能成为攻击者的攻击途径。

比如窃取文件，HTML5另外一些API从安全角度来看很有意思，它们还能混合起来使用。

例如文件API允许浏览器访问本地文件，攻击者可以将它与点击劫持和其他攻击混合起来获得用户本地的文件。

比如骗取你从本地拖放文件到页面里。

g.新标签攻击

HTML5去掉了很多过时的标签，例如&lt;center&gt;和&lt;frameset&gt;，同时又引入了许多有趣的新标签，例如&lt;video&gt;和&lt;audio&gt;标签可以允许动态的加载音频和视频。

HTML5引入的新标签包括&lt;Audio&gt;、&lt;Video&gt;、&lt;Canvas&gt;、&lt;Article&gt;、&lt;Footer&gt;等等，而这些标签又有一些有趣的属性，

例如poster、autofocus、onerror、formaction、oninput，这些属性都可以用来执行javascript。这会导致XSS和CSRF跨域请求伪造。

我们对此攻击的防御方式是，对前端或者后端的过滤器进行优化，添加过滤规则或者黑名单。

h.Web Socket攻击

HTML5的最好的功能之一WebSocket允许浏览器打开到特定IP目标端口的Socket连接，它提供了基于TCP Socket的全双工双向通信，可以实现消息推送机制，大大减少了服务器和浏览器之间的不必要的通信量。

例如可以用它来实现QQ的消息弹窗或者微博的新消息通知，让我们可以更好的实现Web应用。

HTML5限制了Web Socket可以使用的端口，但是，它可能会成为攻击者的载体。想象你打开一个页面，这个页面打开Socket连接并且执行一个内部IP地址的端口扫描。

如果端口扫描发现了内部网络上发现了一个开启的80端口，一个隧道就可能通过你的浏览器建立。这样做会实际上最终绕过防火墙，并且允许访问内部内容。

Web Socket会带来的威胁包括：

◆成为后门

◆端口扫描

◆僵尸网络（一到多的连接）

◆构造基于WebSocket的嗅探器

JS-Recon是一个基于HTML5的JavaScript网络探测工具，它可以使用WebSocket执行网络及端口扫描。

通过我们的应对策略只能加强连接管理，严格控制连接数等。

后台应用安全问题

后台业务设计到权限和数据安全，在处理所有安全问题之前首先要解决两个问题：

1）Who am I?\(Autentication\)

认证。确实当前访问者是谁。

2）What can I do?\(Authorization\)

授权。Autentication与Authorization的不同。

我能做什么，包括对资源API，和数据的权限等。

我们首先来说明认证这个事情。对于微服务的拆分成一个个子系统，分别做认证的话，容易造成逻辑的大量冗余和性能的损耗。所以解决方法就是在网关层做统一的登录处理。

其他的内部服务器之前的调用不重要做认证过程。同时和其他已有的系统做最好也能保持减少用户输入用户名和密码的操作，就需要使用到单点登录。

单点登录（Single sign on）SSO

http是无状态协议之上建立简单状态：

web应用采用browser/server架构，http作为通信协议。http是无状态协议，浏览器的每一次请求，服务器会独立处理，不与之前或之后的请求产生关联。

任何用户都能通过浏览器访问服务器资源，如果想保护服务器的某些资源，必须限制浏览器请求；要限制浏览器请求，必须鉴别浏览器请求，响应合法请求，忽略非法请求。

要鉴别浏览器请求，必须清楚浏览器请求状态。既然http协议无状态，那就让服务器和浏览器共同维护一个状态吧！这就是会话机制。

会话机制：

通过不同的认证协议，让浏览器第一次请求服务器，服务器创建一个会话，并将会话的id作为响应的一部分发送给浏览器，浏览器存储会话id，并在后续第二次和第三次请求中带上会话id，

服务器取得请求中的会话id就知道是不是同一个用户了，让后续请求与第一次请求产生了关联。服务器在内存中保存在会话对象，浏览器通过请求参数或者cookie实现id的存储。

浏览器将会话id作为每一个请求的参数，服务器接收请求自然能解析参数获得会话id，并借此判断是否来自同一会话，很明显，这种方式不安全，容易泄露或者窃取。

第二种让浏览器自己来维护这个会话id，每次发送http请求时浏览器自动发送会话id，cookie机制正好用来做这件事。

cookie是浏览器用来存储少量数据的一种机制，数据以”key/value“形式存储，浏览器发送http请求时自动附带cookie信息。通过浏览器存储还可以通过sessionStorage，localStorage来实现存储。

三者之间的区别如下：

a.cookie的内容主要包括：名字、值、过期时间、路径和域。路径与域一起构成cookie的作用范围。若不设置时间，则表示这个cookie的生命期为浏览器会话期间，关闭浏览器窗口，cookie就会消失。这种生命期为浏览器会话期的cookie被称为会话cookie。

会话cookie一般不存储在硬盘而是保存在内存里，当然这个行为并不是规范规定的。若设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再打开浏览器这些cookie仍然有效直到超过设定的过期时间。对于保存在内存里的cookie，不同的浏览器有不同的处理方式session机制。

cookie也是不可或缺的，cookie的作用是与服务器进行交互，作为http规范的一部分而存在的。

b.web Storage仅仅是为了在本地“存储”数据而生 sessionStorage、localStorage、cookie都是在浏览器端存储的数据。和cookie相似，区别是它是为了更大容量存储设计的，cookie的大小是受限的，并且每次请求一个新的页面的时候cookie都会被发送过去，这样无形中浪费了带宽，另外cookie还需要指定作用域，不可跨域调用。

c.sessionStorage对数据作用域增加了一层控制。sessionStorage是在同源的同窗口中，始终存在的数据，也就是说只要这个浏览器窗口没有关闭，即使刷新页面或进入同源另一个页面，数据仍然存在，关闭窗口后，sessionStorage就会被销毁，同时“独立”打开的不同窗口，即使是同一页面，sessionStorage对象也是不同的

web storage拥有setItem,getItem,removeItem,clear等方法，不像cookie需要前端开发者自己封装setCookie，getCookie。

所以通过如上分析我们可以得知在sessionId，如果放在cookie中，浏览器发送请求的时候会发送给服务器，如果放在web storage里面需要客户端代码来设置了。

不同web服务器也有自己的session机制，比如tomcat。tomcat实现了cookie，访问tomcat服务器时，浏览器中可以看到一个名为“JSESSIONID”的cookie，这就是tomcat会话机制维护的会话id。

cookie携带会话id在浏览器与服务器之间维护会话状态。但cookie是有限制的，这个限制就是cookie的域（通常对应网站的域名），浏览器发送http请求时会自动携带与该域匹配的cookie，而不是所有cookie。

要解决这个问题需要所有子系统的域名统一在一个顶级域名下，其次要使用相同web服务器，因为如同上面tomcat举例，它使用的key就是Jsession但是不同的服务器可能key不相同就没有办法使用了。

所以需要一种SSO来解决这个问题。

SSO是指在多系统应用群中登录一个系统，便可在其他所有系统中得到授权而无需再次登录，包括单点登录与单点注销两部分。

相比于单系统登录，sso需要一个独立的认证中心，只有认证中心能接受用户的用户名密码等安全信息，其他系统不提供登录入口，只接受认证中心的间接授权。间接授权通过令牌实现，sso认证中心验证用户的用户名密码没问题，创建授权令牌，

在接下来的跳转过程中，授权令牌作为参数发送给各个子系统，子系统拿到令牌，即得到了授权，可以借此创建局部会话，局部会话登录方式与单系统的登录方式相同。这个过程，也就是单点登录的原理，用下图说明

![](/assets/SSO.png)

注销的过程如下：

单点登录自然也要单点注销，在一个子系统中注销，所有子系统的会话都将被销毁，用下面的图来说明

sso认证中心一直监听全局会话的状态，一旦全局会话销毁，监听器将通知所有注册系统执行注销操作

![](/assets/sso2.png)

下面对上图简要说明

a\)用户向系统1发起注销请求

b\)系统1根据用户与系统1建立的会话id拿到令牌，向sso认证中心发起注销请求

c\)sso认证中心校验令牌有效，销毁全局会话，同时取出所有用此令牌注册的系统地址

d\)sso认证中心向所有注册系统发起注销请求

e\)各注册系统接收sso认证中心的注销请求，销毁局部会话

f\)sso认证中心引导用户至登录页面

部署图与实现

单点登录涉及sso认证中心与众子系统，子系统与sso认证中心需要通信以交换令牌、校验令牌及发起注销请求，因而子系统必须集成sso的客户端，sso认证中心则是sso服务端，整个单点登录过程实质是sso客户端与服务端通信的过程，用下图描述

![](/assets/SSO_deploy.png)

sso认证中心与sso客户端通信方式有多种，这里以简单好用的httpClient为例，web service、rpc、restful api都可以

只是简要介绍下基于java的实现过程，不提供完整源码，明白了原理，我相信你们可以自己实现。sso采用客户端/服务端架构，我们先看sso-client与sso-server要实现的功能（下面：sso认证中心=sso-server）

只是简要介绍下基于java的实现过程，不提供完整源码，明白了原理，我相信你们可以自己实现。sso采用客户端/服务端架构，我们先看sso-client与sso-server要实现的功能（下面：sso认证中心=sso-server）

sso-client:

a\)拦截子系统未登录用户请求，跳转至sso认证中心

b\)接收并存储sso认证中心发送的令牌

c\)与sso-server通信，校验令牌的有效性

d\)建立局部会话

e\)拦截用户注销请求，向sso认证中心发送注销请求

f\)接收sso认证中心发出的注销请求，销毁局部会话

sso-server:

a\)验证用户的登录信息

b\)创建全局会话

c\)创建授权令牌

d\)与sso-client通信发送令牌

e\)校验sso-client令牌有效性

f\)接收sso-client注销请求，注销所有会话

对比OATH2.0协议：

![](/assets/auth2.png)

（A）用户打开客户端以后，客户端要求用户给予授权。

（B）用户同意给予客户端授权。

（C）客户端使用上一步获得的授权，向认证服务器申请令牌。

（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。

（E）客户端使用令牌，向资源服务器申请获取资源。

（F）资源服务器确认令牌无误，同意向客户端开放资源。

CAS的单点登录时保障客户端的用户资源的安全

oauth2则是保障服务端的用户资源的安全

CAS客户端要获取的最终信息是，这个用户到底有没有权限访问我（CAS客户端）的资源。

oauth2获取的最终信息是，我（oauth2服务提供方）的用户的资源到底能不能让你（oauth2的客户端）访问

CAS的单点登录，资源都在客户端这边，不在CAS的服务器那一方。

用户在给CAS服务端提供了用户名密码后，作为CAS客户端并不知道这件事。

随便给客户端个ST，那么客户端是不能确定这个ST是用户伪造还是真的有效，所以要拿着这个ST去服务端再问一下，这个用户给我的是有效的ST还是无效的ST，是有效的我才能让这个用户访问。

oauth2认证，资源都在oauth2服务提供者那一方，客户端是想索取用户的资源。

所以在最安全的模式下，用户授权之后，服务端并不能直接返回token，通过重定向送给客户端，因为这个token有可能被黑客截获，如果黑客截获了这个token，那用户的资源也就暴露在这个黑客之下了。

于是聪明的服务端发送了一个认证code给客户端（通过重定向），客户端在后台用这个code，以及另一串客户端和服务端预先商量好的密码，才能获取到token和刷新token，这个过程是非常安全的。

如果黑客截获了code，他没有那串预先商量好的密码，他也是无法获取token的。这样oauth2就能保证请求资源这件事，是用户同意的，客户端也是被认可的，可以放心的把资源发给这个客户端了。

所以cas登录和oauth2在流程上的最大区别就是，通过ST或者code去认证的时候，需不需要预先商量好的密码。

当然也也可以做成简化版本，直接code当做认证用，这样只要一次通信即可。

##### 密码安全与加密解密

##### 解决了认证之后，要说说密码和加密相关的事情。

最近twitter和github中纷纷暴露，密码被明文现实在log中的事件。说明即是在这么大的IT公司任然有着低级的安全错误，更加说明安全之路还很长，要引起大家的足够重视。

密码的分类：

a）单因素密码

b\) 多因素密码

手机绑定的动态口令，数字证书等等

设计到私密敏感信息的获取和更新的时候，一般需要多因素的密码。比如邮件，手机短信验证等等。

设计比较多的，比如密码更改，支付等重要环节。现在随着大数据的兴起，拥有大量数据的公司开始也用大数据建立的人物画像来推测是否是本人，比如异地登录消费等等来就会提醒账户异常。

通过人工电话沟通进一步确定等。

密码相关介绍

根据密钥类型不同可以将现代密码技术分为两类：对称加密算法（私钥密码体系）和非对称加密算法（公钥密码体系）。

对称加密算法、非对称加密算法和不可逆加密算法可以分别应用于数据加密、数据安全传和输身份认证。

a\) 对称加密算法中，数据加密和解密采用的都是同一个密钥，因而其安全性依赖于所持有密钥的安全性。

对称加密算法的主要优点是加密和解密速度快，加密强度高，且算法公开.

缺点是实现密钥的秘密分发困难，在大量用户的情况下密钥管理复杂，而且无法完成身份认证等功能，不便于应用在网络开放的环境中。

对称加密算法的特点是算法公开、计算量小、加密速度快、加密效率高。

目前最著名的对称加密算法有数据加密标准DES,但传统的DES由于只有56位的密钥，因此已经不适应当今分布式开放网络对数据加密安全性的要求。

欧洲数据加密标准IDEA等，目前加密强度最高的对称加密算法是高级加密标准AES，AES提供128位密钥，128位AES的加密强度是56位DES加密强度的1021倍还多。

对称加密算法过程是将数据发信方将明文（原始数据）和加密密钥一起经过特殊加密算法处理后，使其变成复杂的加密密文发送出去。

收信方收到密文后，若想解读原文，则需要使用加密用过的密钥及相同算法的逆算法对密文进行解密，才能使其恢复成可读明文。

在对称加密算法中，使用的密钥只有一个，发收信双方都使用这个密钥对数据进行加密和解密，这就要求解密方事先必须知道加密密钥。

不足之处是，交易双方都使用同样钥匙，安全性得不到保证。假设两个用户需要使用对称加密方法加密然后交换数据，则用户最少需要2个密钥并交换使用，

如果企业内用户有n个，则整个企业共需要n×\(n-1\) 个密钥。

b\) 非对称加密算法

非对称加密算法使用两把完全不同但又是完全匹配的一对钥匙（即一把公开密钥或加密密钥和专用密钥或解密密钥）—公钥和私钥。

在使用不对称加密算法加密文件时，只有使用匹配的一对公钥和私钥，才能完成对明文的加密和解密过程。加密明文时采用公钥加密，

解密密文时使用私钥才能完成，而且发信方（加密者）知道收信方的公钥，只有收信方（解密者）才是唯一知道自己私钥的人。

非对称加密算法的基本原理是，如果发信方想发送只有收信方才能解读的加密信息，发信方必须首先知道收信方的公钥，然后利用收信方的公钥来加密原文；收信方收到加密密文后，使用自己的私钥才能解密密文。

广泛应用的非对称加密算法有RSA算法和美国国家标准局提出的DSA。

非对称加密算法的保密性比较好，它消除了最终用户交换密钥的需要，但加密和解密花费时间长、速度慢，它不适合于对文件加密而只适用于对少量数据进行加密。

c）不可逆加密算法

不可逆加密算法的特征是加密过程中不需要使用密钥，输入明文后由系统直接经过加密算法处理成密文，这种加密后的数据是无法被解密的，只有重新输入明文，并再次经过同样不可逆的加密算法处理，得到相同的加密密文并被系统重新识别后，才能真正解密。

显然，在这类加密过程中，加密是自己，解密还得是自己，而所谓解密，实际上就是重新加一次密，所应用的“密码”也就是输入的明文。不可逆加密算法不存在密钥保管和分发问题，非常适合在分布式网络系统上使用，但因加密计算复杂，工作量相当繁重，

通常只在数据量有限的情形下使用，如广泛应用在计算机系统中的口令加密，利用的就是不可逆加密算法。近年来，随着计算机系统性能的不断提高，不可逆加密的应用领域正在逐渐增大。

在计算机网络中应用较多不可逆加密算法的有RSA公司发明的MD5算法和由美国国家标准局建议的不可逆加密标准SHS\(Secure Hash Standard:安全杂乱信息标准\)等。

虽然通过不可逆加密算法加密之后，知道密文之后没有办法通过算法来直接解密获取明文，但是如果密码简单的话，通过彩虹表可以得知明文。

所谓彩虹表就是预先建立一个可逆向的散列链并将其存储在表中，在破解时先查表得到可能包含结果的散列链，然后在内存中重新计算并得到最终结果。折中方式综合了计算暴力破解和查找表破解的优点，并将计算时间和存储空间降低到可以接受的范围。

为了让彩虹表失效，一般对应的做法通过对密码加盐，让密码更加复杂。

盐（Salt）是什么？就是一个随机生成的字符串。我们将盐与原始密码连接（concat）在一起（放在前面或后面都可以），然后将concat后的字符串加密。采用这种方式加密密码，查表法就不灵了（因为盐是随机生成的）。

由于加了 Salt，即便数据库泄露了，但是由于密码都是加了 Salt 之后的散列，坏人们的数据字典已经无法直接匹配，明文密码被破解出来的概率也大大降低。

是不是加了 Salt 之后就绝对安全了呢？淡然没有！坏人们还是可以他们数据字典中的密码，加上我们泄露数据库中的 Salt，然后散列，然后再匹配。

但是由于我们的 Salt 是随机产生的，假如我们的用户数据表中有 30w 条数据，数据字典中有 600w 条数据，坏人们如果想要完全覆盖的坏，

他们加上 Salt 后再散列的数据字典数据量就应该是 300000\* 6000000 = 1800000000000，一万八千亿啊，干坏事的成本太高了吧。但是如果只是想破解某个用户的密码的话，只需为这 600w 条数据加上 Salt，然后散列匹配。

可见 Salt 虽然大大提高了安全系数，但也并非绝对安全。

实际项目中，Salt 不一定要加在最前面或最后面，也可以插在中间嘛，也可以分开插入，也可以倒序，程序设计时可以灵活调整，都可以使破解的难度指数级增长。

但是同时要把盐不要被泄露。

##### sessionId 或者token安全

通过xss窃取到token，不建议放到url中，容易被refer出去，然后导致token泄露。

session fixation问题，登录前后要重新换sessionId或者token

同时要考虑是否允许一个用户保持多个session的相关事宜，可以通过检查userAgent和IP地址等，强制退出等。

介绍完认证之后的事情之后，要说有一下授权的事情。

##### 访问控制：权限问题

##### What can I do?\(Authorization\)

Autentication与Authorization的不同。Autentication解决是who an I的问题。通过上面的cas或oath2等协议确认身份。

而Authorization是解决当前这个登录者能做什么的问题。

在解决能做什么的权限问题的时候，首先要解决系统设计者把什么设计为系统的资源，划分资源的维度分别是什么。因为只有规划好资源的范围和维度，

才有可能在这些范围和维度的基础之上，建立和映射权限。

在微服务化的主要特征之一就是把restful化，也就是把一个个URL作为资源的一个单位。同时根据知否能修改资源，也能分成只读用户和可更新资源的用户，

这样的通过http的method可以进行划分。其中比较复杂的基于数据的划分。就是资源url，和method相同的情况下，只能访问部分资源。比如和登录用户的角色，所属机构等相关联。所以资源划分归纳为如下三种：

1\) 基于URL

2）基于方法

3）基于数据

在现实当中，复杂系统中一把会是这三种方式的组合，不仅仅选择一种。

在上述通过资源的维度基础之上，权限的访问控制也可以分成垂直权限控制和水平权限控制：

垂直访问权限控制：基于角色的访问权限控制

spring security 就有基于角色的访问权限控制。可以通过在spring security的配置文件或者代码中，对url和method做权限控制，

实现很简单，对于简单系统来说可能已经足够了。复杂的系统可能要用到如下的水平权限控制。

水平访问权限控制：

角色还不够，设计到自拥有的数据也就是基于数据的访问权限控制。

考虑使用用户组group的概念或者是一个完全的规则引擎来实现垂直数据校验。

因为和业务耦合度比较高，所以没有通用的解决方案

或参考云里面的租户概念，增加系统数据维度



web server配置安全与web 框架安全



运行环境包括：web server，脚本语言解释器，中间件等。由于开发人员或者运维人员的一些知识或者经验不足，没有引起足够的重视，或者简单的引入了一些第三方的插件等功能，

也是系统安全的一个很重要的威胁。如果要分析的话，要一个个来举例说明，这里主要解释一下apache server 和ngnix的一些安全配置和安全缺陷介绍。主要引入几个原则作为参考。



1）webserver安全自身的安全。

通过阅读官方文档和社区，即是更新安全插件等来保证这些web server的安全。

2）提供的功能模块的安全。

自身相对比较成熟，但是提供的功能模块，很多有个人贡献的，社区里面贡献的module，提供一些扩展功能容易出现问题。

要给最小的权限。



Apache服务器的主要安全缺陷

   Apache服务器应用最为广泛，设计上非常安全的程序。但是同其它应用程序一样，Apache也存在安全缺陷。毕竟它是完全源代码，Apache服务器的安全缺陷主要是使用HTTP协议进行的拒绝服务攻击\(denial of service\)、

缓冲区溢出攻击以及被攻击者获得root权限三缺陷和最新的恶意的攻击者进行“拒绝服务”\(DoS\)攻击。合理的网络配置能够保护Apache服务器免遭多种攻击。我们来介绍一下主要的安全缺陷：

（1）使用HTTP协议进行的拒绝服务攻??\(denial of service\)的安全缺陷

    这种方法攻击者会通过某些手段使服务器拒绝对HTTP应答。这样会使Apache对系统资源\(CPU时间和内存\)需求的剧增，最终造成Apache系统变慢甚至完全瘫痪。

（2）缓冲区溢出的安全缺陷

   该方法攻击者利用程序编写的一些缺陷，使程序偏离正常的流程。程序使用静态分配的内存保存请求数据，攻击者就可以发送一个超长请求使缓冲区溢出。比如一些Perl编写的处理用户请求的网关脚本。一旦缓冲区溢出，攻击者可以执行其恶意指令或者使系统宕机。

（3）被攻击者获得root权限的安全缺陷

   该安全缺陷主要是因为Apache服务器一般以root权限运行\(父进程\)，攻击者会通过它获得root权限，进而控制整个Apache系统。

（4）恶意的攻击者进行“拒绝服务”\(DoS\)攻击的安全缺陷

    它主要是存在于Apache的chunk encoding中，这是一个HTTP协议定义的用于接受web用户所提交数据的功能。 利用黑客程序可以对于运行在FreeBSD 4.5, OpenBSD 3.0 / 3.1, NetBSD 1.5.2平台上的Apache服务器均可进行有效的攻击.

所有说使用最高和最新安全版本对于加强Apache Web服务器的安全是至关重要的。请广大Apache服务器管理员去http://www.apache.org/dist/httpd/下载补丁程序以确保其WEB服务器安全！



Apache服务器正确维护和配置

  Apache服务器的开发者非常注重安全性，由于Apache服务器其庞大的项目，难免会存在安全隐患。正确维护和配置Apache WEB服务器就很重要了。我们应注意的一些问题：

　　（1）Apache服务器配置文件

　　Apache Web服务器主要有三个配置文件，位于/usr/local/apache/conf目录下。这三个文件是：

　　httpd.con-----&gt;主配置文件

　　srm.conf------&gt;填加资源文件

　　access.conf---&gt;设置文件的访问权限

注：具体配置可以参考：http://httpd.apache.org/docs/mod/core.html

　　（2）Apache服务器的日志文件

　　我们可以使用日志格式指令来控制日志文件的信息。使用LogFormat "%a %l"指令，可以把发出HTTP请求浏览器的IP地址和主机名记录到日志文件。出于安全的考虑，在日志中我们应知道至少应该那些验证失败的WEB用户，在http.conf文件中加入LogFormat "%401u"指令可以实现这个目的。这个指令还有其它的许多参数，用户可以参考Apache的文档。另外，Apache的错误日志文件对于系统管理员来说也是非常重要的，错误日志文件中包括服务器的启动、停止以及CGI执行失败等信息。更多请参看Apache日志系列1-5。

　　（3）Apache服务器的目录安全认证

　　在Apache Server中是允许使用 .htaccess做目录安全保护的，欲读取这保护的目录需要先键入正确用户帐号与密码。这样可做为专门管理网页存放的目录或做为会员区等。 

　　在保护的目录放置一个档案，档名为.htaccss   

   （4）Apache服务器访问控制

　　我们就要看三个配置文件中的第三个文件了，即access.conf文件，它包含一些指令控制允许什么用户访问Apache目录。应该把deny from all设为初始化指令，再使用allow from指令打开访问权限。

　　&lt;directory /usr/local/http/docs/private&gt;

　　&lt;limit&gt;

　　order deny,allow

　　deny from all

　　allow from safechina.net

　　&lt;/limit&gt;

　　&lt;/directory&gt;

　　设置允许来自某个域、IP地址或者IP段的访问。

　　（5）Apache服务器的密码保护问题

　　我们再使用.htaccess文件把某个目录的访问权限赋予某个用户。系统管理员需要在httpd.conf或者srm.conf文件中使用AccessFileName指令打开目录的访问控制。





Nginx安全配置

Nginx 有 2 个模块用于控制访问“数量”和“速度”，简单的说，控制你最多同时有 多少个访问，并且控制你每秒钟最多访问多少次， 你的同时并发访问不能太多，也不能太快，不然就“杀无赦”。

HttpLimitZoneModule 限制同时并发访问的数量

HttpLimitReqModule 限制访问数据，每秒内最多几个请求

（1）普通配置 

  服务器直接面向普通用户，例如 普通用户IE浏览器 ——-&gt; 你的服务器。

普通用户IE浏览器 ——-&gt; 你的服务器 由于普通用户直接访问你的服务器，所以你可以直接得到用户的 IP 地址， 而我们的限制就是基于对 来源IP 地址的访问限制。

Nginx里面设置一个限制。

（2）复杂配置，多层代理的配置

很多时候，我网站不是简单的 普通用户IE浏览器 ——-&gt; 你的服务器 的结构， 考虑到网络访问速度问题，我们中间可能会有各种 网络加速（CDN）。以本网站 www.bzfshop.net 为例，考虑到网站的安全性和访问加速，架构是：

普通用户浏览器 —–&gt; 360网站卫士加速（CDN，360防 CC,DOS攻击） ——&gt; 阿里云加速服务器（我们自己建的CDN，阿里云盾） —-&gt; 源服务器（PHP 程序部署在这里，iptables, nginx 安全配置）

可以看到，我们的网站中间经历了好几层的透明加速和安全过滤， 这种情况下，我们就不能用上面的“普通配置”。因为上面基于 源IP的限制 结果就是，我们把 360网站卫士 或者 阿里云盾 给限制了，因为这里“源IP”地址不再是 普通用户的IP，

而是中间 网络加速服务器 的IP地址。我们需要限制的是 最前面的普通用户，而不是中间为我们做加速的 加速服务器。



面对的最直接的问题就是， 经过这么多层加速，我怎么得到“最前面普通用户的 IP 地址”呢？ 

当一个 CDN 或者透明代理服务器把用户的请求转到后面服务器的时候，这个 CDN 服务器会在 Http 的头中加入 一个记录

X-Forwarded-For : 用户IP, 代理服务器IP

如果中间经历了不止一个 代理服务器，像 www.bzfshop.net 中间建立多层代理之后，这个 记录会是这样

X-Forwarded-For : 用户IP, 代理服务器1-IP, 代理服务器2-IP, 代理服务器3-IP, ….

可以看到经过好多层代理之后， 用户的真实IP 在第一个位置， 后面会跟一串 中间代理服务器的IP地址，从这里取到用户真实的IP地址，针对这个 IP 地址做限制就可以了.

经过多层CDN之后取得原始用户的IP地址，nginx 配置 取得用户的原始地址来做相应的配置。

有很多的模块可以使用，大家用到的时候可以进一步探索。





##### web框架安全

Web 应用开发框架，统一设计，便于安全防控。

常用 Web 框架：MVC —— Model（数据处理）-View（与用户交互）-Controller（控制应用逻辑），

例如 Spring 不同层级对应的 Web 安全

View 视图层：XSS

Controller 控制层：CSRF、访问控制、认证、URL 跳转

Model 模型层：SQL 注入

模板引擎与 XSS 防御

XSS 防御中，输出编码的控制体现在 “HTML 标签”、“HTML 属性”、“script 标签”、“事件”、“CSS”、“URL” 中输出变量。

在不同的场景使用不同的编码方式是有效防御的重要原则。当前流行的 MVC 框架对这方面的支持比较薄弱。

Django 中使用 Django Templetes 作为模板引擎。

引擎本身提供一些编码方法：filters 中的 escape 是 HtmlEncode 的方法，例如 &lt;h1&gt;Hello, {{ name\|escape }}!&lt;/h1&gt;。默认情况下 Django 开启了 auto-escape，即所有变量都会经过 HtmlEncode 后输出。

其中编码了 5个字符（&lt;→&lt，&gt;→&gt，'→&\#39，"→&quot，&→&amp）。关闭 auto-escape 的方法：{{ parameter\|safe }}，或 {% autoescape off %}Hello {{ name }}{% endautoescape %}。可以自定义 filter，完善编码功能。

Velocity 模板引擎，默认没有开启 HtmlEncode，需要通过 Event Handler 进行 Html 编码。

“宏” 定义可以完善编码功能（XML 编码输出：\#SXML\(xml\)，JS 编码输出：\#SJS\(js\)）。

eventhandler.referenceinsertion.class = org.apache.velocty.app.event.implement.EscapeHtmlReference

eventhandler.escape.html.match = /msg.\*/



Web 框架防御 CSRF

security token 是防御 CSRF 攻击的有效方法。对于 Web 框架，可以自动地在所有涉及 POST 的代码中添加 token（所有 form 表单、Ajax POST 请求等）。

（1）在 Session 中绑定 token。如果不能保存到服务器端的 Session 中，则保存到 Cookie 里。

（2）在 form 表单中自动填入 token 字段，例如 &lt;input type=hidden name "anti\_csrf\_token" value="$token"/&gt;。

（3）在 Ajax 请求中自动添加 token。

（4）在服务器端对比 POST 提交参数的 token 与 Session 中绑定的 token 是否一致。

在 Rails 框架中实现该功能只需要在 Application Controller 中添加：

protect\_from\_forgery :secret =&gt; "12345678901234567890..."

它将根据 secret 和服务器端的随机因子自动生成 token，并自动添加到所有的 form 和由 Rails 生成的 Ajax 请求中。

在 Django 框架中，首先将 django.middleware.CsrfViewMiddleware 添加到 MIDDLEWARE\_CLASSES 中。然后在 form 表单的模板中添加 token：

&lt;form action="." method="post"&gt;{% csrf\_token %}

最后在 View 层的函数中使用 django.core.context\_processors.csrf

from django.core.context\_processors import csrf

from django.shortcuts import render\_to\_response

def my\_view\(request\):



　　c = {}



　　c.update\(csrf\(request\)\) 



　　...



　　return render\_to\_response\("a\_template.html", c\)

在 Ajax 请求中，一般插入一个包含了 token 的 HTTP 头（防止 token 泄密）。

HTTP Headers 管理

Web 框架中可以对 HTTP 头进行全局化的处理，便于实施一些基于 HTTP 头的安全方案。

例如防御 CRLF 攻击，返回号为 30X 的页面跳转 Location 管理，X-Frame-Options 属性配置，Cookie 的 HttpOnly 标签设置等。

数据持久层与 SQL 注入

使用 ORM（Object/Relation Mapping）框架防御 SQL 注入。例如 ibatis，基于 sqlmap，将 SQL 语句结构化地写入 XML 文件。其中 \#value\# 为安全的静态变量，而 value 为需要严格控制的动态变量。

在 Django 框架中，Database API 默认将所有输入进行了 SQL 转义。

框架设计的一些安全建议

凡是在 Web 框架中可能实现的安全方案，只要针对性能没有太大的损耗，都应该考虑实施。

在设计整体安全方案时，首先建立威胁模型，然后再判断哪些威胁可以在框架中得到解决。

在设计 Web 框架安全解决方案的时候，还要保存好安全检查日志，以便追溯分析。

框架要与时俱进，及时修复自身或依赖项存在的漏洞或缺陷。


安全与整个软件生命周期

设计阶段：

1）选择合适的web server和技术栈可以避免很多问题。

2）做好流量规划，可以进一步设置合理的安全参数等。

3）做好安全代码规范给开发指导。

4）设计合理的测试case，最后来回归测试安全问题

5）设计安全策略应对突发情况。

开发阶段：

1）提供安全函数

2）代码安全审计工具，比对代码是否符合规范，或者规范的进一步修订。

3）建立安全日志

测试阶段：

web安全扫描工具 skipfish等

上线运维：

保持这几条原则

1）find and fix： 及时发现及时处理 

2）Defend and Defer：安全监控和报警 Nagios结合安全日志，进一步分析。

3）Secure at the source：找出源头。















