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
## service版本控制 ##
## 日期/时间处理 ##
## service安全 ##
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

### ETag header###


## HTTP状态码（Top 10） ##
下面是RESTful service或者API最常使用的HTTP状态码和它们的一个普遍接受的用法的简要总结。其他HTTP状态码偶尔被使用，但是特殊后者比较高级。大多数service套件只支持下面这些或甚至它的一个子集。

**200 (OK)** – General success status code. Most common code to indicate success.
**201 (CREATED)** – Successful creation occurred (via either POST or PUT). Set the Location header to contain a link to the newly-created resource. Response body content may or may not be present.
**204 (NO CONTENT)** – Status when wrapped responses are not used and nothing is in the body (e.g. DELETE).
**304 (NOT MODIFIED)** – Used in response to conditional GET calls to reduce band-width usage. If used, must set the Date, Content-Location, Etag headers to what they would have been on a regular GET call. There must be no response body.
**400 (BAD REQUEST)** – General error when fulfilling the request would cause an invalid state. Domain validation errors, missing data, etc. are some examples.
**401 (UNAUTHORIZED)** – Error code for a missing or invalid authentication token.
**403 (FORBIDDEN)** – Error code for user not authorized to perform the operation, doesn't have rights to access the resource, or the resource is unavailable for some reason (e.g. time constraints, etc.).
**404 (NOT FOUND)** – Used when the requested resource is not found, whether it doesn't exist or if there was a 401 or 403 that, for security reasons, the service wants to mask.
**409 (CONFLICT)** – Whenever a resource conflict would be caused by fulfilling the request. Duplicate entries, deleting root objects when cascade-delete not supported are a couple of examples.
**500 (INTERNAL SERVER ERROR)** – The general catch-all error when the server-side throws an exception.

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

