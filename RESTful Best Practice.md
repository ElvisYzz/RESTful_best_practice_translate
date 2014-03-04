# RESTful Service 最佳实践 #

## 基本介绍 ##
&emsp;&emsp;目前有许多关于创建REST风格web service的最佳实践方面的资源（见文档最后的资源章节），其中有很多资源是相互冲突的,这跟它们所写的时间有关。另外，为了实现service而去阅读和理解一些书也是不可行的。为了促进快速吸收和理解RESTful的概念，这篇指南旨在加速这个进程，让你避免阅读至少3到5本的关于这方面的书籍。

&emsp;&emsp;与其说是一组标准，REST更像是一些法则的集合，除了贯穿REST的6个约束以外，没有说明其他任何东西。 虽然有所谓的最佳实践和事实标准，但这些都是在不断发展中的。

&emsp;&emsp;本文提供了很多建议和菜谱式的围绕REST的常见问题的讨论，并提供了一些简短的背景信息来支持有效建立真实世界，生产就绪的，一致的REST风格的service。本文聚合了很多其他来源的信息，通过一些努力来从中获取经验。

&emsp;&emsp;目前对于REST是否真的比SOAP好依然有很大的争论，可能仍然有理由去支持SOAP service, 本文并不会花很多时间去说SOAP的好处, 相反，由于技术和工业界的推进，我们将假设使用REST是目前创建web service的最佳实践。

&emsp;&emsp;第一章讲诉了什么是REST，它的约束以及是什么让REST变得独特。第二章提供了一些简单的技巧来点出一些REST service的概念，之后的章节将会给web服务创建者提供更深入的支持和讨论来理解创建实际环境中的高品质REST服务的具体细节。

## REST是什么 ##
&emsp;&emsp;REST架构风格描述了6种约束。这些针对架构的约束，最早出自由Roy Fielding在他的[博士论文](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)（http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm）中定义的RESTful风格架构的基本概念中。

这6种约束分别是：

* 统一的接口
* 无状态
* 缓存
* 客户-服务器
* 分层系统
* 按需代码

下面是关于这些约束更详细的讨论：
### *统一接口* ###
统一接口约束定义了客户端和服务器的接口，它简化和解耦了架构，使得每一部分都能独自演进。这四项指导统一接口的原则是：

#### 基于资源 ####
独立的资源在请求中是通过URI来标识资源的，这些资源本身在概念上与返回给客户端的表现形式分离。举个例子，服务器并不会发送它的数据库，而是HTML，XML或者JSON等来表现数据库记录的东西被发送回来，例如，时候为芬兰语和使用UTF-8编码，取决于请求的细节和服务器的实现。

#### 通过表述来操作资源 ####
当客户端拥有了资源的一种表现形式，包括所有元数据，只要有权限，它就有足够的信息来修改或者删除服务端的资源。

#### 自描述的信息 ####
每一条信息都包含了足够的信息来描述如何处理这条信息。例如，调用哪一个分析器可能是被Internet Media Type（之前称为MIME类型）所定义的。回复信息里也会明确指定他的缓存能力。

#### 超媒体作为应用程序状态引擎(HATEOAS) ####
客户端传递状态是通过body的内容，查询字符串参数，请求头以及被请求的URI（资源名）。服务端传递状态是通过body内容，响应状态码，响应头。这在技术上称为超媒体（或者超文本中的超链接）。

除了上述内容，HATEOAS也意味着当需要的时候，链接会被包含在返回体（或者头）中来提供URI检索这个资源本身或者其相关资源。我们会在后面详细讨论。

统一接口是任何REST服务必须提供的，这是这种设计的基本原则。

### *无状态* ###
REST是__RE__presentational __S__tate __T__ransfer的首字母缩写，无状态是其关键。本质上，这意味着处理请求必要的状态被包含在了这个请求本身，不管是作为URI的一部分，查询字符串参数，体或者头中。URI唯一的标识这个资源，体包含了这个资源的状态（或者状态变化）。在服务端处理之后，适当的状态，或者有关的那块状态会通过头，状态码或者返回体来传回客户端。

我们大多数在业内工作一段时间的人都习惯于在一个提供了session的容器里编程，它维护了多个HTTP请求之间的状态。在REST中，客户端必须包含所有信息供服务端完成这个请求，需要时重发状态如果这个状态必须跨越多个请求。无状态允许更多的可拓展性，因为服务端没必要去维护，更新和传达这个会话状态。另外，负载平衡器也没必要无状态系统的会话亲和性。

那状态跟一个资源之间的区别是什么呢？状态，或者说应用程序状态，是服务器关心的东西，它是用来定义资源表述——例如存储在数据库的数据。考虑到应用程序状态是数据，可能会因为不同的客户端以及每次请求的不同而变化。另一方面，资源状态对请求它的每一个客户端是是不变的。

出现后退按钮问题和它在某一时刻擅职离守web应用是因为它期望你按照特定的顺序去执行？那是因为它违反了无状态准则。有些情况不适用无状态准则，比如三条腿的OAuth，API调用率限制等等。但是，尽所有努力去保证应用程序状态不跨越多个请求。

### *缓存* ###
在万维网上，客户端可以缓存响应，所以响应必须显式或者隐式定义其自身为可缓存，或者不缓存来防止客户端重用过期或者不合适的数据来作进一步的请求。有效管理缓存可以部分或者全部消除一些客户-服务器的交互，从而提高可拓展性和性能。

### *客户-服务器* ###
统一接口分离了客户端和服务器，这种关注点的分离意味着，举个例子，客户端不用关注数据的存储，这样数据保持在每个服务器内部，这样使得客户端代码的可移植性提高了。服务器不用关心用户接口和用户状态，从而服务端可以更简单以及更可拓展。服务端和客户端也可能被替换和独立开发，只要接口没有改变。

### *分层系统* ###
客户端通常不能分辨出是直接连到了服务端，还是一个中介。中介服务器可能会通过负载平衡和使用共享缓存来提高系统可拓展性。每一层也可以实施安全措施。

### *按需代码（可选）* ###
服务端可以通过转移逻辑代码到客户端来临时拓展或者定制客户端的功能。这样的例子可能包括编译好的组件包括Java Applets和客户端的脚本比如JavaScript。

遵守这些约束，也就是符合REST架构风格，将可以允许任何分布式超媒体系统拥有理想的涌现性，比如性能，可拓展性，简单性，可修改性，可见性，可移植性和可靠性。

注意：唯一可选的约束是*按需代码*。如果一个service违反了任一其他约束，严格地说它就不能称为RESTful。


## REST 技巧 ##
不管在技术上是否是RESTful（根据上面提到的6个约束），这里是一些推荐的REST-like概念，可以有助于创建更好更可用的service：

### *使用HTTP动词* ###
### *明显的资源名* ###
### *XML和JSON* ###
### *创建细粒度的资源* ###
### *考虑连通性* ###

## 定义 ##

### *幂等性* ###
相反的声音，别搞错，这和某些领域的功能紊乱没有关系。来自Wikipedia：

     In computer science, the term idempotent is used more comprehensively to describe an operation that will produce the same 
     results if executed once or multiple times. This may have a different meaning depending on the context in which it is 
	 applied. In the case of methods or subroutine calls with side effects, for instance, it means that the modified state 
	 remains the same after the first call.

从一个RESTful service的立场看，因为一个operation（或者service call），客户端可以重复调用相同的操作——操作很像在编程语言中的`setter`。换句话说，进行多次相同的请求产生的影响跟一次请求操作一样。主要到当幂等操作在服务端产生相同结果时，响应本身可能并不相同（例如，资源状态可能改变了）

`PUT`和`DELETE`方法是幂等的。但是，请看下面*HTTP Verbs，DELETE*章节中关于DELETE的警告。

`GET`，`HEAD`，`OPTIONS`和`TRACE`方法也是幂等的，因为他们是被定义成安全操作。请看下面关于安全性的章节。

### *安全性* ###
来自Wikipedia：

	 Some methods (for example, HEAD, GET, OPTIONS and TRACE) are defined as safe, which means they are intended only for 
	 information retrieval and should not change the state of the server. In other words, they should not have side effects, 
	 beyond relatively harmless effects such as logging, caching, the serving of banner advertisements or incrementing a web counter. 
     Making arbitrary GET requests without regard to the context of the application's state should therefore be considered safe.

简而言之，安全性意味着调用一个方法不会产生副作用。因此，客户端可以重复进行安全请求而不会对服务端造成副作用。这意味着service必须遵守`GET`，`HEAD`，`OPTIONS`，`TRACE`操作的定义。否则，除了让service使用者困惑以外，还会对web缓存，搜索引擎以及其他机器人程序造成问题——对服务端造成意外的变化。

根据定义，安全操作是幂等的，因为他们产生相同的结果。

安全方法也被实现成只读操作，但是，安全性并不意味着服务端必须每次返回相同的响应。

## HTTP Verbs ##
HTTP动词构成了“统一接口”的主要部分，为我们action和基于名词的资源，主要的或者最常使用的HTTP动词（或者方法，当被正确调用）是`POST`，`GET`，`PUT`和`DELETE`。他们分别对应创建，查询，更新和删除（CRUD）。当然也哟一些其他操作，但是很少被用到。在这些用的比较少的操作中，`OPTIONS`和`HEAD`相对来说稍微对一点。

### *GET* ###
HTTP `GET`方法被用来检索（或查询）资源的表述。在“Happy”（或者无错）路径上，GET返回XML或者JSON的表述和HTTP状态码`200`（OK）。错误情况下，经常返回`404`（NOT FOUND）或者`400`（BAD REQUEST）。
例如：
> `GET` http://www.example.com/customers/12345  
> `GET` http://www.example.com/customers/12345/orders  
> `GET` http://www.example.com/buckets/sample

根据HTTP规范，`GET`（和`HEAD`）请求只能被用来查询数据，不能改变数据。 因此，当这样使用时，他们是安全的。也就是他们可以被调用而不会有修改或者弄脏数据和风险。另外，`GET`（和`HEAD`）是幂等的，这意味着多次调用跟一次操作结果相同。

不要通过`GET`暴露不安全的操作——它永远不应该修改服务端的任何资源。

### *PUT* ###
`PUT`是被用来更新的最常用操作，往一个特定的URI进行`PUT`，在请求体中包含更新需要的数据。

但是，`PUT`也可以用来进行创建资源的操作，只要资源`ID`有客户端来选择而不是服务端，换句话说，只要`PUT`中包含不存在的资源ID。重复一下，请求体包含资源的表述。很多人认为这是令人费解和困惑的。因此，如果有的话，使用这种方法来创建的话应该谨慎使用。

或者，使用`POST`来创建资源然后在请求体中提供客户端定义的ID——如果是一个不包含资源ID的URI。

例如：

> `PUT` http://www.example.com/customers/12345  
> `PUT` http://www.example.com/customers/12345/orders/98765  
> `PUT` http://www.example.com/buckets/secret_stuff

一旦更新成功，`PUT`会返回`200`（或者`204`如果在体中没有任何内容）。如果使用PUT进行创建，返回HTTP状态码`201`。响应体是可选的——如果消耗太多带宽。在创建时通过Location header返回一个链接是不必要的，因为客户端已经设置了资源ID，见“*返回值*”章节。

`PUT`不是一个安全操作，因为它修改（或者创建）了服务端状态，但它是幂等的。换句话说，如果你通过`PUT`创建或者更新了一个资源然后再进行一次相同操作，资源依然在那儿，依然有着相同的状态。

举个例子，如果调用`PUT`让某资源计数器自增，这种调用就不是幂等的了。但是，保持`PUT`幂等性是被推荐的。强烈推荐使用POST进行非幂等请求。

### *POST* ###
`POST`是被用来创建的最常用操作，特别地，用来创建下级资源，也就是从属于某一其他（例如父）资源。也就是说，当创建一个新资源的时候，`POST`在上级资源时，service需要关心相关的新资源，给ID（新资源URI）赋值等等。

例如：
> `POST` http://www.example.com/customers  
> `POST` http://www.example.com/customers/12345/orders

创建成功，返回`201`，返回一个指向新创建资源的Location头。

`POST`既不安全也不幂等。因此推荐用于非幂等的请求。调用两次相同的`POST`请求很可能导师两个资源包含相同的信息。

### *PUT vs POST for Creation* ###
简言之，赞成使用`POST`进行资源创建。否则，当客户端来决定新资源的URI（通过其资源名或者ID）时使用`PUT`,如果客户端知道结果URI（或者ID），使用`PUT`.否则当由服务端来决定新创建的资源的URI时，使用`POST`。换言之，当客户端在创建之前不知道（或不应该应知道）结果URI时，使用`POST`来创建资源。

### *DELETE* ###
`DELETE`非常容易理解，它被用来通过URI来删除一个资源。
例如：
> DELETE http://www.example.com/customers/12345  
> DELETE http://www.example.com/customers/12345/orders  
> DELETE http://www.example.com/buckets/sample

删除成功时，返回`200`和响应体，可能是被删除资源的表述（经常需要太多带宽），或者一个包装过的响应（见下面*返回值*）。又或者是返回状态`204`（NO CONTENT）。换句话说，一个没有体的`204`状态，或者`JSEND`风格的响应和`200`状态，这两种是推荐的响应。

HTTP规范定义`DELETE`是幂等的。如果你删除一个资源，它就被移除了。重复在这个资源上调用`DELETE`结果相同，该资源没有了。如果调用`DELETE`会导致（资源内）计数器自减，这样的调用就不是幂等了。正如前面提到的，使用统计和度量可能会被更新，但是只要没有资源数据被改变仍然认为service幂等。对非幂等的请求使用`POST`是被推荐的。

但是，关于`DELETE`幂等有一个注意点。在一个资源上第二次调用`DELETE`通常会返回`404`（NOT FOUND）因为该资源已经被移除所以不能再被发现。这将导致`DELETE`操作不再幂等。但是如果资源从数据库移除而不是被简单的标记为已删除，这是一个适当的妥协。

下面这张表总结了主要的HTTP方法结合资源URI时推荐的返回值：

| HTTP Verb  | /customers | /customers/{id} |
|------------|:-----------|:----------------|
| GET        |200(OK), customer列表使用分页，排序和过滤来操作大列表|200(OK)，单个Customer。当ID没找到或者无效时返回404(NOT FOUND)|
| PUT        |404(NOT FOUND)，除非你希望对整个集合的每一个资源都进行更新/替换|200(OK)或者204(NO CONTENT)，当ID没找到或者无效时返回404(NOT FOUND)|
| POST       |201(CREATED),"Location"头包含一个带有新ID的链接指向/customers/{id}|404(NOT FOUND)|
| DELETE     |404(NOT FOUND)，除非你想删除整个集合——通常这不是你希望的|200(OK)。当ID没找到或者无效时返回404(NOT FOUND)|

## 资源命名 ##
除了恰当的使用HTTP动词，资源的命名可能是在创建易理解，易使用的 Web service API 时争论最多，也是最重要的概念。当资源命名得当，API就会很直观，也易于使用。没做好的话，同样的API也会显得笨拙，难以使用和理解。下面是一些小贴士帮助你创建新的API。

本质上看，RESTful API 是一组URI的集合，使用HTTP调用这些URI，然后用JSON和/或XML来表述这些资源，通常还会包含相关的链接。RESTful可定位能力是通过URI来包含的。每一个资源都有其地址和URI——服务端能提供的所有有意义的信息都是作为资源来暴露出去，*统一接口*约束部分是通过URI和HTTP动词相结合并按标准和约定使用来实现的。

当决定在系统中包含哪些资源，就用名词来命名他们，换言之，一个RESTful URI指向的资源应该是一个东西，而不是指向一个动作。名词具有属性而动词没有，这也是另一个显著的因素。

关于资源的一些示例：

* Users of the system.
* Courses in which a student is enrolled.
* A user's timeline of posts.
* The users that follow another user.
* An article about horseback riding.

service中每一个资源至少有一个URI去标识它，URI有意义并且能充分描述这个资源那是最好的。URI应当是可预见的，使用分层结构去提高其可理解性，这样也就提高了可用性，可预见意味着他们是一致的，分层意味着数据是由结构的——通过他们之间的关系。这不是REST的规则或约束，而是它能增强API的可用性。

RESTful API是为消费者写的，他的名字和URI结构应该要能向使用者传达意义。通常很难去知道数据的边界，但是理解了你的数据，你就最可能抓住重点然后给你的客户端返回合理的表述。为你的客户考虑，而不是你的数据。

我们来描述这样一个订单系统，包括customers，orders，line items，products等，考虑一下描述这些资源的URI：
### *资源URI示例* ###
插入（创建）一个新的customer：
> `POST` http://www.example.com/customers

查询ID为33245的Customer
> `GET`  http://www.example.com/customers/33245

同样也可以对上面的URI进行更新(UPDATE)和删除(DELETE)操作。

创建一个新的product：
> `POST` http://www.example.com/products

查询，更新，删除product 66432
> `GET`|`PUT`|`DELETE` http://www.example.com/products/66432

下面就有趣了，如何为一个customer创建一个新的order
一种方案可能是：
> `POST` http://www.example.com/orders

这样是会创建一个新的order，但是可能是跟customer没有关系的。  
因为我们是想为customer创建一个order（注意关系），这个URI可能并不直观。有人提出像下面那种URI会更清晰：
> `POST` http://www.example.com/customers/33245/orders

这会给ID为33245的customer创建一个order

然后
> `GET` http://www.example.com/customers/33245/orders

将会返回customer33245所创建或者拥有的orders。注意：我们可能会选择不支持`DELETE`和`PUT`操作因为它操作的是一个集合。

现在继续我们分层的概念，看下面的URI：
> `POST` http://www.example.com/customers/33245/orders/8769/lineitems

这会给order8769增加一个lineitem。对！对这个URI进行GET将会返回这个order的所有lineitem。但是，如果lineitem在customer上下文中没有意义或者也在customer之外有意义，我们会进行这样的POST：
> `POST` www.example.com/orders/8769/lineitems