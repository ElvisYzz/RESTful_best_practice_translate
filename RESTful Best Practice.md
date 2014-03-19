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
任何API调用者都可以发送`GET`,`POST`,`PUT`,`DELETE`动词，他们都极大的明确了请求的要做什么。一个GET请求不能改变任何基本资源数据。测量和跟踪可能仍然会发生，这会更新数据，但是不是更新URI中被识别的资源数据。

### *明显的资源名* ###
使用明显的资源名或者路径（例如：/posts/23 而不是 /api?type=posts&id=23）会提高一个请求想做什么的清晰度。使用Query-string参数对使用过滤来说很好，但对资源命名来说不怎么样。
合适的命名提供了一个请求的上下文，提高了一个API的可理解性。资源可以通过他们URI的命名被视作层次关系，提供使用者一个友好容易理解的资源层次关系来在他们的应用程序中使用。

资源命名应当是名词——避免使用动词作为资源名。这回让事物变得清晰，使用HTTP动词来指明请求的动作。
### *XML和JSON* ###
最好使用JSON作为默认，但是除非同时支持JSON和XML开销巨大，同时支持他们。理想的是，让用户去通过改变拓展名.xml和.json。另外，为了支持Ajax风格的用户接口，一个wrapped response很有帮助。提供wrapped response，无论是默认或者是使用.wjson或者.wxml来指明客户端请求的是一个wrapped response。

JSON作为一个标准有很少的要求。这些要求也只是在句法上，而不是内容格式和布局上，换句话说，使用JSON响应是约定的，不是被描述在标准中的。更多关于JSON请看http://www.json.org/ 。

关于REST service中使用XML，XML标准和约束也仅仅是在使用句法正确的标签和文本。特别是，命名空间不是，也不应该在RESTful service的上下文中使用。XML返回的也像JSON一样，简单而易读，不需要schema和namespace等，只有links和data。如果最后变得比这个复杂，那么就是这节第一部分说的，XML的开销将是巨大的。无论怎样，依我们的经验，很少有用户还在用XML。

### *创建细粒度的资源* ###
当开始的时候，很容易模仿基本应用程序域或数据库架构创建API。最后你想要聚合这些services——为了减少通信量而是用了多个基本资源的services。但是比起从大的资源中创建单个资源或者细粒度的资源，更容易从单个资源创建更大的资源。从小的，容易定义的资源开始，为他们提供CRUD操作，之后你可以创建面向用例，通信量降低的资源。

### *考虑连通性* ###
REST的一个准则是无连接——通过超链接。尽管没有超链接service仍然有用，dan如果在响应中返回他们的话，API变得更加能自描述。至少，一个“self”引用告诉客户端一个数据是怎么或者可以怎样去检索的。另外，使用Location头来包含使用POST进行创建的资源的链接。对于在响应中返回的集合，如果使用了分页，至少包含“first”，“last”，“next”，“prev”这些链接是很有帮助的。

## 定义 ##

### *幂等性* ###
相反的声音，别搞错，这和某些领域的功能紊乱没有关系。来自Wikipedia：

     In computer science, the term idempotent is used more comprehensively to describe an operation that will produce the same results if executed once or multiple times. This may have a different meaning depending on the context in which it is applied. In the case of methods or subroutine calls with side effects, for instance, it means that the modified state remains the same after the first call.

从一个RESTful service的立场看，因为一个operation（或者service call），客户端可以重复调用相同的操作——操作很像在编程语言中的`setter`。换句话说，进行多次相同的请求产生的影响跟一次请求操作一样。主要到当幂等操作在服务端产生相同结果时，响应本身可能并不相同（例如，资源状态可能改变了）

`PUT`和`DELETE`方法是幂等的。但是，请看下面*HTTP Verbs，DELETE*章节中关于DELETE的警告。

`GET`，`HEAD`，`OPTIONS`和`TRACE`方法也是幂等的，因为他们是被定义成安全操作。请看下面关于安全性的章节。

### *安全性* ###
来自Wikipedia：

	  Some methods (for example, HEAD, GET, OPTIONS and TRACE) are defined as safe, which means they are intended only  for information retrieval and should not change the state of the server. In other words, they should not have side effects, beyond relatively harmless effects such as logging, caching, the serving of banner advertisements or incrementing a web counter. Making arbitrary GET requests without regard to the context of the application's state should therefore be considered safe.

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

更深层次的例子：
> `GET` http://www.example.com/customers/33245/orders/8769/lineitems/1

返回这一个order里的第一个line item。

现在你已经清楚层次结构概念是如何工作的，没有任何困难和快速的规则，只要确保使用的结构对使用者有意义。在每一种软件开发的技巧中，命名都是成功的关键。

看看被广泛使用的API来鉴赏一下，然后根据你队友的直觉来精炼一下你的API。一些API的示例如下：

- Twitter: https://dev.twitter.com/docs/api
- Facebook: http://developers.facebook.com/docs/reference/api/
- LinkedIn: https://developer.linkedin.com/apis

### *资源命名的反模式* ###
当我们讨论一些恰当的资源名时，有时候看到一些反模式的例子也是有益的。下面是一些比较差的RESTful资源URI的例子，当然我们不应该这样：

首先，通常service使用一单个的URI来给定service的接口，其中使用了query-string参数和/或HTTP动词来说明这些请求操作。例如更新一个customer的ID 12345，使用JSON的请求可能是这样：
> `GET` http://api.example.com/services?op=update_customer&id=12345&format=json

尽管这个'services'是一个名词，但是这个URL并不能自描述因为这个URI层级对所有请求都是一样的。另外，使用了GET即使我们正在进行一个更新操作。这对客户端来说是违反直觉也是令人痛苦（甚至危险）的。

另一个更新customer的例子，
> `GET` http://api.example.com/update_customer/12345

和它的邪恶兄弟：
> `GET` http://api.example.com/customers/12345/update

你会看到很多这样的例子。开发者确实是在试图创建一个RESTful API，也取得了一些进展。事实上我们并不需要在URI中指出我们的‘update’，因为我们可以使用HTTP动词来指明。需要澄清的是下面这种是多余的：
> `PUT` http://api.example.com/customers/12345/update

`PUT`和‘update’同时存在，这会迷惑使用者。

### *复数化* ###
让我们看看复数化和单数化之间的争论，本质上，可以归结为下面这个问题：
层次结构中URI节点应该使用单数还是复数命名？例如：检索一个customer的表述，URI是否应该像这样：
> `GET` http://www.example.com/customer/33245

还是：
> `GET` http://www.example.com/customers/33245

两者都有道理，但是通常被接受的而是使用复数来使API URI在所有HTTP动词之间保持一致。原因是基于这样一个概念，customers是一个集合，ID指向了这个集合customers的一个。

使用这条规则，一个使用复数的多节点的URI例子像这样：
> `GET` http://www.example.com/**customers**/33245/**orders**/8769/**lineitems**/1

这意味着你只需要两个base URI，一个用来创建资源，另一个用来查询，更新和删除资源。创建资源的例子：
> `POST` http://www.example.com/customers

查询，更新，删除的例子：
> `GET`|`PUT`|`DELETE` http://www.example.com/customers/{id}

对一个资源的操作可能有多种URI，但是满足CRUD操作最小集合就只要这两种简单的URI。

你肯定想问是否有种情况是用复数不合适的，确实有。当没有集合概念的时候。换句话说，接受使用单数是当资源只有一个的时候——这是一个singleton资源。例如，有一个单个的包含一切的configuration资源，你可能使用一个单数名字来表述它：
> `GET`|`PUT`|`DELETE` http://www.example.com/configuration

注意没有ID和POST动词。如果每一个customer也只有一个configuration， URL可能长这样：
> `GET`|`PUT`|`DELETE` http://www.example.com/customers/12345/configuration

再说一次，没有ID和POST。尽管我确定这两种情况下POST可能会被认为是有效的。那……好吧。

## 返回表述 ##
如之前提到的，一个service支持多种表述形式，包括JSON和XML是令人满意的。作为默认的表述，JSON是被推荐的。但是service应当允许客户端定义多种表述形式。

当客户端请求一种表述形式，是使用Accept头还是query-string呢。最好的是，service同时支持这两种。但是，业内现在达成协议使用一个format说明符，就像一个文件拓展名。所以，推荐service使用'.json','.xml'和被包装的'.wjson','.wxml'。

使用这种技术，是在URI里说明表述方式，例如，`GET` http://www.example.com/customers.xml会用xml返回，`GET` http://www.example.com/customers.json会用json返回。这也使得service被大多数基础客户端(例如'curl')易于使用。

service也应该返回一种默认的形式（可能是JSON）
> `GET` http://www.example.com/customers/12345
> `GET` http://www.example.com/customers/12345.json

应当都使用JSON来返回。
> `GET` http://www.example.com/customers/12345.xml

返回xml形式，如果支持的话。如果xml并不支持，应当返回一个`404`。

使用HTTP Accept头被认为是最优雅的实现，但是为了同时支持wrapped和unwrapped响应，你必须实现你自己的自定义类型，因为没有一个表述的类型来表现他们。这也会增加客户端和服务端的复杂度。查看 [Section 14.1 of RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.1)了解Accept的详细内容。支持文件拓展名形式是简单，直接使用最少的字母也是最容易支持的方式——不需要使用HTTP header。

总的来说，当谈论REST service，XML都是不太相关的。即使支持了XML也很少有人使用。特别地，命名空间不是，也不应该用来RESTful service的上下文中。这只会让事情变得复杂。所以用XML返回时，应该要更多的像JSON一样，简单易读，没有schema和namespace的约束——不标准但是可被
理解。

### *通过链接的资源可发现性（HATEOAS）* ###
一个指导性原则（通过统一接口约束）是通过超文本传递应用程序状态。这也通常是指***超媒体作为应用程序状态引擎(HATEOAS)***，见*什么是REST*章节。

根据Roy Fielding的[博客](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertextdriven)，REST接口最重要的部分就是超文本。进一步他指出在一个初始URI没有先验知识和之外的信息时，一个API也应该可用和可理解。也就是，API应该可导航，通过它的links来知道数据的各种构件。只返回数据的表述是令人泄气的。

这个时间经常不被业界的遵循，反映出HATEOAS更多的用在成熟度模式上。看看很多service，约定都是返回更多的数据和更少的（或者没有）links。这与Fielding的REST约束相违背。Fielding认为：“任何信息携带的地址都是可被定位的单元……查询结果使用带总结信息的links去表述，而不是一组事物的表述。”

另一方面，简单的返回links的集合是造成网络通信量的主要原因。在真实世界中，依靠需求或者用例，API的通信量可以通过平衡响应中有多少总结数据被包含在了相关超文本链接里来很好的控制。

同时，完全的使用HATEOAS会增加实现的复杂度，以及给客户端强加了很多负担，降低了客户端和服务端开发者的产量。因此，很有必要去平衡超链接的实现程度和可用的开发资源。

当为了最小限度影响开发和降低客户端和服务端的耦合度，一个超链接实践的最小集合在service可用性，可导航性和可理解性方面能提供主要作用。这些最小化的建议是通过`POST`来创建资源，通过`GET`来返回资源集合，更多的建议是分页，如下面的描述。

#### 最少链接的建议 ####
在创建用例中，新创建资源的URI(link)应当在响应头的'Location'中返回，响应体为空——或者只包含新创建资源的ID。

返回的表述的集合中每一个表述都应该至少在它自己的links集合中带有一个‘self’的链接属性。其他links可能被表述称分开的links集合来促进分页，使用'first','previous','next','last'链接。

#### 链接格式 ####
关于整体上链接格式标准，推荐的是注重Atom，AtomPub或者Xlink风格。JSON-LD也是，但是并没有被广泛接受。业内广泛接受的是使用带有‘ref’元素和‘href’元素的Atom link风格，这些元素包含了资源的所有URI，并且没有验证和query-string参数。‘ref’元素可以包含标准值‘alternate’，‘related’，‘self’，‘enclosure’，‘via’，以及‘first’，‘last’，‘previous’，‘next’等用来分页。当他们有道理是使用他们，当需要时你增加你自己的。

我们用一些例子来看，
> `POST` http://api.example.com/users

下面是一个相应头的示例：

>HTTP/1.1 201 CREATED  
>Status: 201  
>Connection: close  
>Content-Type: application/json; charset=utf-8  
>Location: http://api.example.com/users/12346  

响应体是空的或者包含一个wrapped response（请看下面*wrapped response*）
下面是一个JSON响应的例子
```
{
  "data":[
	{
      "user_id":"42",
      "name":"Bob",
      "links":[
        {
          "rel":"self",
          "href":"http://api.example.com/users/42"
        }
      ]
    },
    {
      "user_id":"22",
      "name":"Frank",
      "links":[
        {
          "rel":"self",
          "href":"http://api.example.com/users/22"
        }
      ]
    },
    {
      "user_id":"125",
      "name":"Sally",
      "links":[
        {
          "rel":"self",
          "href":"http://api.example.com/users/125"
        }
      ]
    }
  ]
}
```
注意到links数组包含了一个指向‘self’的引用，这个数据也可以去包含其他关系，比如children，parent等。

最后一个例子是一个用GET请求一个集合，使用了分页（每页3个），这是第三页的JSON response：

```
{
   "data":[
      {
         "user_id":"42",
         "name":"Bob",
         "links":[
            {
               "rel":"self",
               "href":"http://api.example.com/users/42"
            }
         ]
      },
      {
         "user_id":"22",
         "name":"Frank",
         "links":[
            {
               "rel":"self",
               "href":"http://api.example.com/users/22"
            }
         ]
      },
      {
         "user_id":"125",
         "name":"Sally",
         "links":[
            {
               "rel":"self",
               "href":"http://api.example.com/users/125"
            }
         ]
      }
   ],
   "links":[
      {
         "rel":"first",
         "href":"http://api.example.com/users?offset=0&limit=3"
      },
      {
         "rel":"last",
         "href":"http://api.example.com/users?offset=55&limit=3"
      },
      {
         "rel":"previous",
         "href":"http://api.example.com/users?offset=3&limit=3"
      },
      {
         "rel":"next",
         "href":"http://api.example.com/users?offset=9&limit=3"
      }
   ]
}
```

### *Wrapped Response* ###
service有机会同时返回HTTP状态码和响应体，在很多javascript框架中，HTTP状态码是不会返回给终端开发者的，通常防止客户端靠状态码来决定行为。另外，在HTTP说明中有无数的响应码，通常只有很少几个是客户端真正关心的，‘success’，‘error’和‘failure’是最频繁的。因此，用包含响应信息的表述来封装这个response。

一个来自OmniTI Labs的提议，所谓的`JSEND` response。更多的信息在http://labs.omniti.com/labs/jsend 。另外一个选择是由Douglas Crockford提出，可以在这里查看http://www.json.org/JSONRequest.html 。

事实上这两者都没有充分覆盖所有情况。基本上，现在最佳实践是包装常规的(non-JSONP)的响应，带有下面这些属性：
- **code** – contains the HTTP response status code as an integer.
- **status** – contains the text: “success”, “fail”, or “error”. Where “fail” is for HTTP status
response values from 500-599, “error” is for statuses 400-499, and “success” is for everything
else (e.g. 1XX, 2XX and 3XX responses).
- **message** – only used for “fail” and “error” statuses to contain the error message. For
internationalization (i18n) purposes, this could contain a message number or code, either alone
or contained within delimiters.
- **data** – that contains the response body. In the case of “error” or “fail” statuses, this contains the
cause, or exception name.

一个warpped风格的成功响应像这样：
```
{
   "code":200,
   "status":"success",
   "data":{
      "lacksTOS":false,
      "invalidCredentials":false,
      "authToken":"4ee683baa2a3332c3c86026d"
   }
}
```
一个错误响应像这样：
```
{
   "code":401,
   "status":"error",
   "message":"token is invalid",
   "data":"UnauthorizedException"
}
```

这两种wrapped响应在XML中相当于：
```
<response>
	<code>200</code>
	<status>success</status>
	<data class="AuthenticationResult">
		<lacksTOS>false</lacksTOS>
		<invalidCredentials>false</invalidCredentials>
		<authToken>1.0|idm|idm|4ee683baa2a3332c3c86026d</authToken>
	</data>
</response>
```
和
```
<response>
	<code>401</code>
	<status>error</status>
	<message>token is invalid</message>
	<data class="string">UnauthorizedException</data>
</response>
```

### *解决跨域问题* ###
我们都听说过在浏览器的同源策略或同源性要求。换句话说，浏览器只能对它当前的站点进行访问。例如，如果当前站点是www.example.com，这个网站不能对www.example2.com发起请求。很明显，这个影响网站访问service的方式。

目前，有两种被广泛接受的方式去支持跨域请求：JSONP和Cross-Origin Resource Sharing（CORS）。JSONP或者“JSON with Padding” 是一种使用模式提供了一种方法去请求一个不同域的service的数据。它的工作原理是返回任意javascript代码，而不是JSON。这些响应由javascript解释器去处理，不是被一个JSON解析器解析。另一方面，CORS是一个web浏览器的技术规范，它定义了一个web服务器允许它的资源被一个不同域的网页访问的方式。它被看做是一个替代JSONP更现代的方式，被所有现代浏览器所支持。所以，JSONP并不被推荐。无论何时不管那里有可能，选择CORS吧。

#### 支持CORS ####
在一个服务端实现CORS和在一个响应中发送一个附加的HTTP头一样简单。例如：
> Access-Control-Allow-Origin: *

只有当一个数据是会**公共消费**的时候，访问源才应该设为“\*”。在大多数情况下，Access-Control-Allow-Origin头部应该说明**哪个域**应该可以开始一个CORS请求。只有需要被跨域访问的URL才应该设置CORS头。

> Access-Control-Allow-Origin: http://example.com:8080 http://foo.example.com

只允许被信任的域可以使用：
> Access-Control-Allow-Credentials: true

只有当需要的时候使用这个header，因为如果用户登录了应用它会发送cookies/sessions。

配置这些头可以通过web服务器，代理或者从这个service本身发送。
在service内实现它并不被推荐因为它很灵活。相反，使用第二种形式，在你的Web服务器上配置用空格分隔的可用域列表。更新关于CORS的信息可查看http://enable-cors.org/ 。

#### 支持JSONP ####
JSONP使用GET请求对所有service调用来绕过浏览器的限制。本质上，请求者添加一个query-string参数（例如jsonp=“jsonp_callback”），jsonp参数的值是一个响应返回时将被调用的javascript的函数名。

这些功能的严重局限性被JSONP所允许，因为GET请求并不包含一个请求体，因此信息必须通过query-string参数进行传递。为了支持PUT，POST和DELETE操作，有效的HTTP方法必须通过query-string参数进行传递，例如_method=POST.所以这种方式并不被推荐，也会给service带来安全风险。

JSONP在一些不支持CORS的遗留浏览器中工作，但是如果要去支持JSONP，这影响一个service的创建方式。或者，JSONP可以通过代理来实现。总之，为了支持CORS，JSONP不再被重视，任何有可能时都去支持CORS。

为了在服务端支持JSONP，当JSONP query-string参数被传进来时，响应必须像下面一样操作：
1. 响应体必须被封装成JSONP参数所给的javascript函数的参数(例如：jsonp_callback("<JSON response body>"))。
2. 总是返回HTTP状态200(OK)和实际状态作为JSON响应的一部分。

另外，通常需要包含头部作为响应体的一部分。这能允许JSONP回调方法决定响应处理因为在响应头和状态中这是无法得到的信息。


一个error响应像下面这样：
（注意HTTP响应状态总是为`200`）
> jsonp_callback(“{'code':'404', 'status':'error','headers':[],'message':'resource XYZ not found','data':'NotFoundException'}”)

一个successful创建的响应像下面这样（状态依然为`200`）
> jsonp_callback(“{'code':'201', 'status':'successful','headers':[{'Location':'http://www.example.com/customers/12345' }],'data':'12345'}”)
(注意原文有错误**'status':'error'**)


## 查询，过滤，分页 ##
对于大的数据集，限制返回的数据量从带宽的观点来看是非常重要的。但它从一个UI处理的角度来看同样重要，因为一个用户界面往往只能显示一个巨大的数据集的一小部分。在数据集无限增长的情况下，限制默认情况下返回的数据量是有帮助的。例如，在Twitter的返回一个人的tweets（通过他们的主页时间轴）的情况下，返回了20个项目，除非在请求另有说明，即使这样最多也只返回200个。

除了限制返回的数据量，我们还需要考虑如何为“页”，或通过大型数据集滚动，如果超过了第一子集需要检索。这被称为数据的分页创造“页面” ，然后返回一个更大的列表的已知部分，并通过大型数据集能够页“前进”和“后退” 。另外，我们可能想要指定响应中要包括的资源的字段或属性，从而限制了返回的数据量，我们最终想要查询具体的值和/或为返回的数据排序。

有两种主要的方法来限制​​查询结果并进行分页。首先，索引方案要么是面向页面或以项目为导向。换句话说，传入的请求将指定从哪页开始返回数据，指定每页的数量，或直接指定第一个和最后一个项目数（范围）返回。换句话说，有两个选项，“给我5页​​假设每页显示20项”或“给我100到120之间的内容”。

服务提供商正在分裂这个应该如何工作。然而，一些用户界面工具，如Dojo JSON Datastore，选择以模仿HTTP规范使用的字节范围。这是非常有益的，如果你的服务支持开箱即用，因此没有必要在UI工具包和后端服务之间进行翻译。

下方的建议支持Dojo model进行分页，这是使用Range头指定被请求项的范围，也支持利用查询字符串参数。通过支持两种方式，通过先进的用户界面工具包，像Dojo，或者通过简单的，直接的链接和锚标签使用服务将会变得很灵活。它不应该增加开发难度，以支持这两个选项。但是，如果你的service不直接支持UI功能，考虑消除对Range头选项的支持。

需要注意的是查询，过滤和分页不适用于所有服务，这是很重要的一点。此行为是资源特定的，不应该在默认情况下对所有资源支持。服务和资源的文档应该提到哪些端点支持这些更复杂的功能。 

### *限制结果* ###
“给我的项目3至55”这种请求方式与HTTP定义中使用Range头的方式是一致的，但是”从第二项给我最多20项“的方式更便于人类阅读和理解，所以我们使用这种方式来支持query-string参数。
如上所述，推荐是同时使用Range头和query-string参数，offset和limit。注意的是，同时使用的话，query-stirng参数会覆盖Range头。
第一个问题是，为什么我们要支持两种方式，他们功能相似，而且请求中数据可能永远不会匹配。这是第二个问题。关键在于，我们希望让query-stirng里的东西非常清晰，容易理解，人类可读，容易构造和解析。但是，Range头更加机械化，HTTP说明文档中将用法写的很清楚了。
简而言之，使用Range头的内容必须得解析，增加了复杂度，加上客户端也必须进行一个操作来构造这一请求。使用limit和offset参数更容易理解和创建。
#### 通过Range头限制 ####
使用Range头的方式如下：
> Range: items=0-24

注意到它是从0开始的，这和HTTP说明中如何使用Range头进行请求时一致的。换句话说，数据集中第一项是从0开始的，上述请求将返回前25项，假设数据集中一共有至少25项内容。
服务端检测Range头来知道返回哪些项，一旦一个Range被确定下来，将使用一个正则表达式去解析（例如：“items=(\\d+)-(\\d+)”）。

#### 通过Query-String参数限制 ####
Query-String使用了limit和offset参数，offset是起始项(相当于Range头中items的第一个数)，limit是最多返回的项数。例如上面的例子使用query-string参数：
> GET http://api.example.com/resources?offset=0&limit=25

服务端可以定义默认的limit值，为没有说明limit参数的情况服务。但是请在文档中说明这些不可见内容。
注意query-string会覆盖Range的内容。
#### 基于Range的响应 ####
对一个基于Range的响应，不管是用Range头还是用query-stirng参数，服务端应当返回一个Content-Range头来指明哪些项被返回了，以及一共有多少项。
> Content-Range: items 0-24/66

注意到总量（66）并不是从0开始的，因此，请求最后几项时，Range-Content会被设置成这样：
>Content-Range: items 40-65/66

根据HTTP说明，如果响应时总数未知或者计算总数代价太高，（这里是66）可以使用\*替代。
>Content-Range: items 40-65/*

但是要注意Dojo或者一些UI工具并不支持这种表达。

### *分页* ###
上面用来限制结果的方式对分页同样起作用，通过允许指定感兴趣的数据项。使用上面的例子，一共有66项，每页25项，检索第二页内容：
>Range: items=25-49

或者通过query-string参数：
> GET ...?offset=25&limit=25

无论哪一种方式，服务端会返回：
> Content-Range: 25-49/66

通常这会工作得很好，但是偶尔有些情况数字不是直接翻译数据集中的某一行。同时，对一个非常活跃的数据集，新的项会被添加到列表的顶端，明显的”分页问题“就会出现，数据可能会重复。
按时间顺序的数据集就像Tweeter。尽管你仍然可以用item number进行分页，但有时更有效以及更能理解的方式是使用before和after这两个query-string参数。一起使用Range头也是可选的（或者query-string的limit和offset参数）。
例如，检索在某一个时间附近的remarks：

	GET http://www.example.com/remarks/home_timeline?after=<timestamp>
	Range: items=0-19
	GET http://www.example.com/remarks/home_timeline?before=<timestamp>
	Range: items=0-19
同样的，使用query-string：

	GET http://www.example.com/remarks/home_timeline?after=<timestamp>&offset=0&limit=20
	GET http://www.example.com/remarks/home_timeline?before=<timestamp>&offset=0&limit=20

即使客户端并没有指定Range，服务端也会有默认的数据集或者一个最大数量为一个默认值的一个数据集。例如上面那个例子，即使没有说明需要20项，服务端可能也只会返回20项，同样都会返回一个Content-Range头：
>Content-Range: 0-19/4125
或者Content-Range: 0-19/*

### *过滤和排序* ###
另一个需要考虑的是对返回内容的过滤和排序。这两者可以和上面的分页和限制结果一起使用,过滤和排序是复杂的操作，不需要对每个资源都进行这样的操作。在文档中说明哪些资源支持过滤和排序。

#### 过滤 ####
过滤就是指定一些标准从而达到减少返回的数据量。过滤可以变得很复杂，只要服务端设置了一组复杂的比较操作和复杂的匹配标准，但通常使用一些简单的比较，如start-with和contains是被接受的。
在讨论过滤这个query-string参数之前，我们必须理解单个参数和多个参数的区别。这会变得不容易降低命名冲突，我们已经试用了limit，offset，sort（见下一节）参数，如果再要支持jsonp，format，可能还有after和before，这些也还只是在本文中提到的参数。使用更多的参数，我们会更可能碰到命名冲突。使用filter参数来进行最小化。
另外，服务端也可容易判断是否是有带filter的请求，只需要简单的检查下filter参数是否存在。同时，当你查询需求的复杂度提高了，单个参数的方式提供了更多的灵活度，来创建你自己的全功能的查询语法。（见下面的OData注释或者http://www.odata.org）
通过包含一组通用，被接受的分隔符，等式比较可以很直接的实现。设置filter参数的值可以通过使用这些分隔符来创建一系列的键值对，可以很容易被服务端解析和使用。约定的分隔符“|”来隔开一个filter的词组，“::”来隔开键和值。用一个例子说明：
> GET http://www.example.com/users?filter="name::todd|city::denver|title::grand poobah"

"::"隔开了属性名和值，从而允许在值中间使用空格，使得服务端容易解析。

注意到键值对中的属性名如果匹配了服务端的属性名，会在响应的payload中返回。
简单而有效。大小写敏感当然值得讨论，但是在通常情况下，大小写无关时过滤能最好的工作。你也可以使用“*”作为通配符在值中使用。
对于那些使用等号或者通配符不能满足要求的查询，引入操作符是很有必要的。在这种情况下，操作符本身应当作为值得一部分被服务端解析，而不是作为属性名。当需要更复杂的查询语言风格的功能时，可以考虑从Open Data Protocol (OData) Filter System Query
Option的说明文档（见 http://www.odata.org/documentation/uriconventions#
FilterSystemQueryOption）引入查询概念。

#### 排序 ####
排序是对从service返回的payload中的内容决定应当以怎样的顺序排列。换句话说，就是对响应payload中多项内容的排序。
推荐的方式是使用sort作为query-string参数包含一组被分隔开的属性名。默认行为是对每一个属性进行按升序排列。对带有“-”前缀的属性名使用降序排列。使用“|”来分割属性名，跟上面说的filter参数中一样。例如：
> GET http://www.example.com/users?sort=last_name|first_name|-hire_date

再次注意，所有匹配的属性会在响应payload中被返回。另外，由于其复杂度，只对需要的资源提供排序。如果需要的话，小的资源集合可以在客户端进行排序。

## service版本控制 ##
直线地说，版本控制是很难的，艰巨的，困难的，充满了心痛，甚至痛苦和极度的悲伤——我们只能说这增添了很多API的复杂性，并可能对访问它的客户端也增加了复杂度。因此，在你的API设计时应当深思熟虑，并努力达到不需要版本化的表述。

不青睐使用版本控制，而不是使用版本控制来支撑不好的API设计。你会在早晨恨自己，如果你需要对你的API进行版本控制，更不用说频繁进行。精益的理念是使用JSON用法来表述，客户端可以容忍出现在一个不破坏响应的新的属性。但即使这样,在某些情况下还是充满了危险，如改变一个包含了内容或验证规则的现有属性的含义。

不可避免的会有一段时间一个API需要改变它的返回或预期表示,这将导致consumer不能工作,这必须避免。版本控制API的方式避免打破你的客户和消费者。

### *通过内容协商支持版本控制* ###
历史版本控制是通过URI本身中的版本号来实现的,客户把他们想要的版本直接写在他们请求的URI中。事实上,许多“大男孩”,比如Twitter,Yammer,Facebook、谷歌等经常在uri中使用版本号。即使API管理工具如WSO2在暴露的url中也要求版本号。

这种技术在面对REST约束是很有用,因为它不接受内置的HTTP规范的头系统，也不支持一个新的URI应该被添加只有当一个新的资源或概念被引入,而不是表示变化了。另一种反对观点是,资源uri不随时间变化。一个资源就是一个资源。

URI应该简单地识别资源——而不是其“形状”。必须使用另一个概念，指定格式的响应(表示)。这个“概念”是一对HTTP标头:Accept和Content-Type。Accept标头允许客户指定响应的媒体类型(或类型)。content-type头被客户端和服务器都使用，分别表示请求或响应体的格式。

例如, 用JSON格式检索一个User
**\# Request**

	GET http://api.example.com/users/12345
	Accept: application/json; version=1

**\# Response**

	HTTP/1.1 200 OK
	Content-Type: application/json; version=1
	{“id”:”12345”, “name”:”Joe DiMaggio”}

现在，同样使用JSON格式访问同一个资源，但适用version2
**\# Request**

	GET http://api.example.com/users/12345
	Accept: application/json; version=2

**\# Response**

	HTTP/1.1 200 OK
	Content-Type: application/json; version=2
	{“id”:”12345”, “firstName”:”Joe”, “lastName”:”DiMaggio”}

注意URI是相同的,Accept标头用于指定所需的响应格式(和版本)。如果客户端想要用xml格式返回，Accept头设置为application/xml，如果需要，加一个version。

因为Accept头可以设置为允许多个媒体类型,在应对请求时,服务器将设置content-type头为最佳匹配客户端的请求的值。更多信息请看    http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html

例如：
**\# Request**

	GET http://api.example.com/users/12345
	Accept: application/json; version=1, application/xml; version=1

上面的请求可能会是JSON也可能为XML，取决于服务端。但无论server选择哪个，都会在Content-Type中设置出来。
例如：如果server使用了XML来返回，

**# Response**

	HTTP/1.1 200 OK
	Content-Type: application/xml; version=1

	<user>
	<id>12345</id>
	<name>Joe DiMaggio</name>
	</user>

下面是一个使用JSON格式进行创建的例子：

**# Request**

	POST http://api.example.com/users
	Content-Type: application/json; version=1

	{“name”:”Marco Polo”}

或者使用version2：
**# Request**
	POST http://api.example.com/users
	Content-Type: application/json; version=2

	{“firstName”:”Marco”, “lastName”:”Polo”}

#### 当没有指定version时返回哪个版本 ####
提供version在每个请求中都是可选项，因为HTTP内容协商遵循“最佳匹配”的方法。当一个consumer没有指定version，这个API应当返回这个表述的最老支持的版本。
例如：
**# Request**

	GET http://api.example.com/users/12345
	Accept: application/json
**# Response**

	HTTP/1.1 200 OK
	Content-Type: application/json; version=1

	{“id”:”12345”, “name”:”Joe DiMaggio”}

同样的，当POST数据时，没有指定version，上面相同的规则会被使用。这里是一个在多版本端点使用JSON创建user的例子（期望是version1）：
**# Request**

	POST http://api.example.com/users
	Content-Type: application/json
	{“name”:”Marco Polo”}

**# Response**

	HTTP/1.1 201 OK
	Content-Type: application/json; version=1
	Location: http://api.example.com/users/12345

	{“id”:”12345”, “name”:”Marco Polo”}

#### 未支持版本的请求 ####
当请求一个不受支持的版本号，包括弃用的API资源版本,API应该返回一个错误响应406(不可接受)。此外,API应该返回一个响应体Content-Type:application/json，包含了支持的content-type的一个JSON数组。
**# Request**

	GET http :// api . example . com / users/12345
	Content-Type: application/json; version=999
**# Response**

	HTTP/1.1 406 NOT ACCEPTABLE
	Content-Type: application/json

	[“application/json; version=1”, “application/json; version=2”, “application/xml; version=1”,
	“application/xml; version=2”]

### *什么时候应该创建一个新版本* ###
在API开发中有很多可能会打破合同给你的客户带来负面影响。如果你对改变后后果不确定，为稳妥起见,最好考虑版本。当你试图决定一个新版本是否合适或者一个对现有表述的修改是否足够或者可接受，有这样几个因素需要考虑。
#### 会打破合同的修改 ####
- 改变了property名字（“name”改成了“firstName”）
- 移除property
- 改变property的数据类型
- 验证规则改变
- 在Atom风格链接中，修改了“rel”的值
- 一个请求的资源被引入了一个已存在的工作流
- 资源概念/含义改变，例如：
  - 一个content-type为text/html的资源曾经意味着表述是一个links的集合，新的text/html表述意味着用户输入的“web browser form”
  - “.../users/{id}/exams/{id}”曾经表示在那个时间学生提交了考试，新的意思变成了这个考试的结束时间
- 从一个现有资源添加新的字段从而不支持该资源。组合两个资源，成为一个资源，然后不再支持这两个本来的资源。
  - 有两个资源，“.../users/{id}/dropboxBaskets/{id}/messages/{id}”和“.../users/{id}/dropboxBaskets/{id}/messages/{id}/readStatus”，新的需求是将readStatus资源的属性放到个人的message资源中，这会导致在个人message资源中移除一个指向readStatus的链接。

虽然这个列表没有完全包含，但是他告诉了你哪些类型的change会对你的客户端造成大的破坏，以及需要一个新的资源或者一个新的版本。

#### 没有破坏的改变 ####
- 在JSON响应中添加新的属性
- 新的/额外的链接指向其他资源
- 新的content-type支持的格式
- 新的content-language支持的格式
- 大小写无关因为API producer和consumer都应该处理不同的大小写

### *版本控制在哪一级别应该出现* ###
推荐的是在单个资源的级别使用版本控制，一些对API的修改例如修改了工作流可能就需要对多个资源进行版本控制来防止破坏客户端。

### *使用Content-Location来增强响应* ###
可选，见 RDF说明

### *带Content-Type的links* ###
Atom形式的链接支持一个‘type’属性。提供足够的信息，这样客户端可以对特定的version和content type构造必要的请求。

### *找出那些版本是被支持的* ###
#### 应该一次支持几个版本 ####
因为维护多个版本变得繁琐,复杂,容易出错，代价也高，你应该对任何资源支持不超过2个版本。
#### 弃用 ####
弃用表示一个资源依然可用，但是将会不可用，在未来将不存在。
#### 如何通知客户端哪些资源将弃用 ####
在新的版本被引入之后，可能很多客户端还是在使用那些将要被弃用的资源，他们需要一些手段去恢复和监视他们的应用程序对这些弃用资源的使用。当一个将弃用的资源被请求时，API应当返回一个正常的响应，里面包含一个自定义的“Deprecated”头，如下：  
\#Request

	GET http://api.example.com/users/12345
	Accept: application/json
	Content-Type: application/json; version=1
\# Response

	HTTP/1.1 200 OK
	Content-Type: application/json; version=1
	Deprecated: true

	{“id”:”12345”, “name”:”Joe DiMaggio”}

## 日期/时间处理 ##
日期和时间戳是一个真正头痛的东西，如果不进行适当一致的处理。 
时区问题可以很容易地突然出现，并因为在JSON的payload中日期仅仅是字符串，解析是一个真正的问题，如果格式不可知，一致或指定。 
在内部，服务应该用UTC或GMT时间存储，处理，缓存等这样的时间戳。这能缓解日期和时间戳记时区的问题。

### *Body中日期/时间序列化*  ###
关于这一切，一个简单的方法是总使用相同的格式，在字符串中包括时间部分（与时区信息）。ISO 8601的时间点格式是一个很好的解决方案，使用充分增强的格式，包括小时，分钟，秒和秒的小数部分（例如YYYY-MM-dd'T'HH：MM：SS.SSS'Z'）。建议ISO 8601用于表示REST服务Body中所有数据（请求和响应）。

### *Header中日期/时间序列化* ###
尽管上面推荐的在请求或者响应的JSON或者XML格式的体中工作的很好，HTTP说明中对HTTP header使用了一种不同的格式。在RFC 822中说明，并在RFC 1123中被更新的格式包括了多种data，time和date-time的格式。但是推荐是一直使用一个时间戳格式，在请求头中像这个样子：
> Sun, 06 Nov 1994 08:49:37 GMT

不幸的是，它并没有包括毫秒以及秒的小数部分。Java SimpleDateFormat说明符字符串是：Sun, 06 Nov 1994 08:49:37 GMT

## service安全 ##
认证是确认一个来自某人（或某些系统）给定的请求的行为，请求者确实如他们描述的那样，而授权是确认请求者有权限进行所请求的操作。
本质上，流程如下：

	1. 客户端发出请求，包括身份验证令牌的X Authorization头或将令牌包含在请求的查询字符串参数中。
	2. Services验证授权令牌的存在，验证它（它是有效的，没有过期）并解析或加载基于令牌的内容的验证主体。
	3. Services将调用授权服务提供身份验证用户，被请求资源和操作所需的权限。
	4. 如果获得授权，services继续正常处理。


＃ 3以上可能是昂贵的，但假设一个可缓存的访问控制列表（ACL） ，可以设想到创建一个授权客户端来缓存最近期的ACL来在远程调用之前进行本地验证。

### *认证* ###
目前最好的做法是使用OAuth认证。 OAuth2是极力推荐的，但仍在草稿状态。 OAuth1绝对是一个可以接受的选择。三方模式的OAuth也是一个选择。了解更多关于OAuth的规范在 http://oauth.net/documentation/spec/ 。 
OpenID是一个额外的选项。然而，我们建议OpenID被用来作为一个额外的身份验证选项，主要利用OAuth。了解更多关于OpenID规范在 http://openid.net/developers/specs/ 。

### *传输安全* ###
所有的认证都应该使用SSL。 OAuth2要求授权服务器和访问令牌凭证使用TLS。 
HTTP和HTTPS之间切换带来安全弱点，最佳做法是默认对所有通信使用TLS。

### *授权* ###
授权服务并非真的跟对任何应用程序授权有什么不同。它是基于这样一个问题，“这个主体对给定资源是否有请求的权限？”鉴于数据的简单的三连胜（主体，资源和权限），这也很容易构造一个支持该概念的授权服务。主体是人或系统，它被授予对资源访问的权限。使用这些通用的概念，就可以对每个主体有一个可缓存的访问控制列表（ACL）。

### *应用安全* ###
开发一个安全的web应用所遵守的原则同样适用RESTful services。

- 服务端验证所有输入，接受“已知”的良好输入拒绝坏的输入。
- 防止SQL和NoSQL注入。
- 使用众所周知的库如Microsoft的Anti-XSS或者OWASP的AntiSammy来输出编码后的数据。
- 限制消息大小。
- service应该只显示通用的错误消息。
- 考虑业务逻攻击。例如一个攻击者是否可以跳过一个多步骤的订购流程或者不输入信用卡信息来订购一个产品。
- 日志可疑活动。

RESTful 安全考虑：
- 验证JSON和XML，防止不良数据
- 动词应当被限制为允许的方法，例如，一个`GET`请求不应该可以删除一个Entity。`GET`用来查询而`DELETE`用来删除entity。
- 注意竞态条件。

API网关可以被用来监视，节流和控制对API的访问。下面是可以被网关或者RESTful service管理的。
- 监视API的使用，了解哪个活动是好的，哪个超出正常使用模式。
- 对API使用节流，这样一个恶意用户就不能攻下一个API（DOS攻击），也有能力阻止一个恶意IP地址。
- 使用加密的安全密码库来存储API密钥。

## 缓存和拓展性 ##
缓存允许系统各层次消除远程调用检索被请求的数据，从而提高了可拓展性。service通过在响应中设置header来提高缓存能力。不幸的是，HTTP1.0和HTTP1.1中缓存相关的header是不一样的，所以service应该两个都支持。下面是支持GET请求的最少需要的header，和一些合适的值的描述。

|HTTP Header |Description |Example|
|------------|:-----------|:------|
|Date |响应返回的日期和时间(使用RFC1123格式)。|Date: Sun, 06 Nov 1994 08:49:37 GMT
|Cache-Control| 响应能被缓存的最大秒数（max age）。但是如果响应不支持缓存，那么值为no-cache。| Cache-Control: 360  Cache-Control: no-cache|
|Expires| 如果给出了max age,包含了响应过期的时间戳（RFC1123格式),也就是Date的值(e.g. now)加上max age。如果缓存不支持，将不会有这个header| Expires: Sun, 06 Nov 1994 08:49:37 GMT|
|Pragma | 当 Cache-Control 值为 'no-cache' 这个header也被设为 'no-cache'. 否则，不出现。| Pragma: no-cache|
|Last-Modified| 资源本身最后被修改的时间(RFC1123格式)。| Last-Modified: Sun, 06 Nov 1994 08:49:37 GMT|

为简化起见，这里是一个简答的GET请求，支持缓存一天的header集：
*Cache-Control: 86400
Date: Wed, 29 Feb 2012 23:01:10 GMT
Last-Modified: Mon, 28 Feb 2011 13:10:14 GMT
Expires: Thu, 01 Mar 2012 23:01:10 GMT*
下面是一个相似的响应，但是不允许缓存。
*Cache-Control: no-cache
Pragma: no-cache*

### ETag头###
ETag头对验证被缓存内容的新鲜度很有用，也对进行有条件的查询和更新（分别是`GET`和`PUT`）有帮助，它的值是一个任意字符串来体现版本，但是也应该对每一种格式有不同的值——对JSON响应的ETag值跟XML响应的同一资源的值不同。ETag值可以简单到是一个底层域对象的hash值（Java中为Object.hashcode()），当然其中也包含对format的hash。推荐给每个GET操作返回一个ETag头。另外，确保用双引号来包住ETag值。如：
*ETag: "686897696a7c876b7e"*


## HTTP状态码（Top 10） ##
下面是RESTful service或者API最常使用的HTTP状态码和它们的一个普遍接受的用法的简要总结。其他HTTP状态码偶尔被使用，但是特殊后者比较高级。大多数service套件只支持下面这些或甚至它的一个子集。

**200 (OK)** – 一般请求成功状态码。表示成功最常用状态码。
**201 (CREATED)** – 成功创建(通过POST或PUT). 设置Location头包含指向新创建资源的链接。响应体可能没有。
**204 (NO CONTENT)** – 当wrapped responses没有使用，body中也没有任何东西(如DELETE).
**304 (NOT MODIFIED)** – 在响应中使用减少带宽使用量。如果使用，必须设置Date，Content-Location，Etag头，没有响应体。
**400 (BAD REQUEST)** – 当执行请求时出现一个无效状态时的一般错误。例如域验证错误，缺失数据等。 
**401 (UNAUTHORIZED)** – 缺失或者无效的验证令牌时的错误码。
**403 (FORBIDDEN)** – 用户未经授权进行操作，没有权利访问资源或者资源不可用（例如时间约束）时的错误码。
**404 (NOT FOUND)** – 资源没找到，无论他是不存在还是401或者403因为安全原因，service想进行掩饰。
**409 (CONFLICT)** – 执行请求时资源发生冲突。礼服重复条目，当级联删除不支持时删除根资源。
**500 (INTERNAL SERVER ERROR)** – 当服务端抛出异常时通常所有捕获的错误。

## 其他资源 ##
**Books**
- REST API Design Rulebook, Mark Masse, 2011, O’Reilly Media, Inc.
- RESTful Web Services, Leonard Richardson and Sam Ruby, 2008, O’Reilly Media, Inc.
- RESTful Web Services Cookbook, Subbu Allamaraju, 2010, O’Reilly Media, Inc.
- REST in Practice: Hypermedia and Systems Architecture, Jim Webber, et al., 2010, O’Reilly Media, Inc.
- APIs: A Strategy Guide, Daniel Jacobson; Greg Brail; Dan Woods, 2011, O'Reilly Media, Inc.

**Websites**
http://www.restapitutorial.com
http://www.toddfredrich.com
http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
http://www.json.org/
https://github.com/tfredrich/DateAdapterJ
http://openid.net/developers/specs/
http://oauth.net/documentation/spec/
http://www.json.org/JSONRequest.html
http://labs.omniti.com/labs/jsend
http://enable-cors.org/
http://www.odata.org/documentation/uri-conventions#FilterSystemQueryOption
http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven
https://developer.linkedin.com/apis
http://developers.facebook.com/docs/reference/api/
https://dev.twitter.com/docs/api
http://momentjs.com/
http://www.datejs.com/

