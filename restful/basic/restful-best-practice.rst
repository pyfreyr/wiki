.. restful-best-practice:

=========================
RESTful API 最佳实践
=========================



基本要求
===============

1. 当标准合理的时候遵守标准。
2. API 应该对程序员友好，并且在浏览器地址栏容易输入。
3. API 应该简单，直观，容易使用的同时优雅。
4. API 应该具有足够的灵活性来支持上层ui。
5. API 设计权衡上述几个原则。

你的 API 越容易使用，那么就会有越多的人去用它。

术语
===============

- 资源：一个对象的单独实例，如一只动物
- 集合：一群同种对象，如动物
- 端点：endpoint，也称为“路径”，表示 API 的具体网址
- 幂等：无边际效应，多次操作得到相同的结果
- URL 段：在 URL 里面以斜杠分隔的内容


协议
===========

API 与用户的通信协议，总是使用 HTTPS 协议，确保交互数据的传输安全。

域名
===========

应该尽量将 API 部署在专用域名之下，如果你的应用很庞大或者你预期它将会变的很庞大，那么将 API 放到子域下通常是一个好选择。这种做法可以保持某些规模化上的灵活性。。

::

    https://api.example.com/

如果确定 API 很简单，不会有进一步扩展，可以考虑放在主域名下。

::

    https://example.org/api/

版本
=========

常见的三种方式：

- 在 URI 中放版本信息：``GET /v1/users/1``
- Accept Header：``Accept: application/json+v1``
- 自定义 Header：``X-Api-Version: 1``

用第一种，虽然没有那么优雅，但最明显最方便：

::

    https://api.example.com/v{n}/

版本号分为整形和浮点型：

- 整形的版本号：大功能版本发布形式；具有当前版本状态下的所有 API 接口 ,例如：v1，v2
- 浮点型：小版本号，只具备补充 API 的功能，其他api都默认调用对应大版本号的 API，例如：v1.1，v2.2

随着系统发展，总有一些 API 失效或者迁移，对失效的 API，返回 404 Not Found 或 410 Gone；对迁移的 API，返回 301 重定向。



URI
======

URI 表示资源，资源一般对应服务器端领域模型中的实体类。

URI 命名规范
-----------------

- 不用大写；
- 用中杠 ``-`` 不用下划线 ``_``，因为浏览器中超链接的默认效果是带下划线；
- 参数列表要 encode；
- 使用名词，URI 是指向资源的，而不是描述行为的，应使用名词而非动词来描述语义。
- URI 中的名词表示资源集合，使用复数形式。

不推荐的 API：

::

    /api/getArticle/1/
    /api/updateArticle/1/
    /api/deleteArticle/1/

推荐的 API：

::

    /api/articles/

资源的获取、更新和删除分别通过 GET, PUT 和 DELETE 方法请求 API 即可：

::

    GET /api/articles/    //文章列表
    POST /api/articles/    // 文章添加
    PUT /api/articles/    // 文章修改
    DELETE /api/articles/    // 文章删除


资源表示
-------------

URI 表示资源的两种方式：资源集合、单个资源。

资源集合：

::

    /zoos    //所有动物园
    /zoos/1/animals    //id为1的动物园中的所有动物

单个资源：

::

    /zoos/1    //id为1的动物园

资源子集
-------------

如果要获取一个资源子集，采用 nested routing 是一个优雅的方式：

::

    /api/authors/gevin/articles/

另一种方式是基于过滤，这两种方式都符合规范，但语义不同：如果语义上将资源子集看作一个独立的资源集合，则使用 nested routing 感觉更恰当，如果资源子集的获取是出于过滤的目的，则使用 filter 更恰当。

可以使用过滤处理那些获取资源集合的请求。所以只要出现 GET 的请求，就应该通过 URL 来过滤信息：

::

    ?type=1&age=16    // 允许一定的冗余，如 /zoos/1 与 /zoos?id=1
    ?sortby=age&order=asc
    ?page=2&per_page=100
    ?limit=10&offset=3

减少路径嵌套
----------------

在一些有父路径/子路径嵌套关系的资源数据模块中，路径可能有非常深的嵌套关系，例如：

::

    /orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}

推荐在根路径下指定资源来限制路径的嵌套深度。使用嵌套指定范围的资源，例如在下面的情况下：

::

    /orgs/{org_id}
    /orgs/{org_id}/apps

    /apps/{app_id}
    /apps/{app_id}/dynos

    /dynos/{dyno_id}


Request
=============

HTTP 方法
-------------

GET，从服务器查询资源（一个具体的资源或者一个资源列表。）：

::

    GET /zoos
    GET /zoos/1
    GET /zoos/1/employees

POST，在服务器新建单个资源（一般向“资源集合”型 URI 发起）：

::

    POST /animals    //新增动物
    POST /zoos/1/employees    //为id为1的动物园雇佣员工

PUT，以整体的方式更新服务器上的一个资源（ 一般向“单个资源”型 URI 发起）：

::

    PUT /animals/1
    PUT /zoos/1

PATCH，只更新服务器上一个资源的一个属性（ 一般向“单个资源”型 URI 发起）。

DELETE，从服务器删除资源：

::

    DELETE /zoos/1/employees/2
    DELETE /zoos/1/animals    //删除id为1的动物园内的所有动物

HEAD，获取一个资源的元数据，如数据的哈希值或最后的更新时间。

OPTION，获取客户端能对资源做什么操作的信息。

安全性和幂等性
------------------

- 安全性：不会改变资源状态，可以理解为只读的；
- 幂等性：执行 1 次和执行 N 次，对资源状态改变的效果是等价的。

GET 和 HEAD 方法必须始终是安全的。

===========  ========    =========
HTTP 方法      安全性      幂等性
===========  ========    =========
GET             √           √
POST            ×           ×
PUT             ×           √
DELETE          ×           √
===========  ========    =========

安全性和幂等性均不保证反复请求能拿到相同的 Response。以 DELETE 为例，第一次 DELETE 返回 200 表示删除成功，第二次返回 404 提示资源不存在，这是允许的。



消息体
----------

只用以下常见的 3 种 body format：

Content-Type: ``application/json``

::

    POST /v1/animal HTTP/1.1
    Host: api.example.org
    Accept: application/json
    Content-Type: application/json
    Content-Length: 24

    {
      "name": "Gir",
      "animalType": "12"
    }

Content-Type: ``application/x-www-form-urlencoded``

::

    POST /login HTTP/1.1
    Host: example.com
    Content-Length: 31
    Accept: text/html
    Content-Type: application/x-www-form-urlencoded

    username=root&password=Zion0101

Content-Type: ``multipart/form-data``


Response
==============

不要包装
-------------

Response 的 body 直接就是数据，不要做多余的无意义包装。

::

    {
        "login": "octocat",
        "id": 1,
        "node_id": "MDQ6VXNlcjE=",
        "avatar_url": "https://github.com/images/error/octocat_happy.gif",
        ...
    }

错误示例：

::

    {
        "success":true,
        "data":{"id":1,"name":"xiaotuan"},
    }

各 HTTP 方法处理后数据格式
------------------------------

============    =================
HTTP 方法        Response 格式
============    =================
GET              单个对象、集合
POST             新增成功的对象
PUT/PATCH        更新成功的对象
DELETE           空
============    =================

HTTP 状态码
==================

HTTP 状态码分类
-------------------

======   =======================================================
分类      分类描述
======   =======================================================
1xx      信息，服务器收到请求，需要请求者继续执行操作
2xx      成功，操作被成功接收并处理
3xx      重定向，需要进一步的操作以完成请求
4xx      客户端错误，请求包含语法错误或无法完成请求
5xx      服务器错误，服务器在处理请求的过程中发生了错误
======   =======================================================

一些说明：

- ``1xx`` 范围的状态码是保留给底层 HTTP 功能使用的，并且估计在你的职业生涯里面也用不着手动发送这样一个状态码出来。
- ``2xx`` 范围的状态码是保留给成功消息使用的，你尽可能的确保服务器总发送这些状态码给用户。
- ``3xx`` 范围的状态码是保留给重定向用的。大多数的API不会太常使用这类状态码，但是在新的超媒体样式的 API 中会使用更多一些。
- ``4xx`` 范围的状态码是保留给客户端错误用的。例如，客户端提供了一些错误的数据或请求了不存在的内容。这些请求应该是幂等的，不会改变任何服务器的状态。
- ``5xx`` 范围的状态码是保留给服务器端错误用的。这些错误常常是从底层的函数抛出来的，并且开发人员也通常没法处理。发送这类状态码的目的是确保客户端能得到一些响应。收到 5xx 响应后，客户端没办法知道服务器端的状态，所以这类状态码是要尽可能的避免。

HTTP 状态码列表
---------------------

=========    ================================  ======================================================================================================================================================================
状态码        英文名称                           中文描述
=========    ================================  ======================================================================================================================================================================
100          Continue                          继续。客户端应继续其请求
101          Switching Protocols               切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议
200          OK                                请求成功。一般用于GET与POST请求
201          Created                           已创建。成功请求并创建了新的资源
202          Accepted                          已接受。已经接受请求，但未处理完成
203          Non-Authoritative Information     非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本
204          No Content                        无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档
205          Reset Content                     重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域
206          Partial Content                   部分内容。服务器成功处理了部分GET请求
300          Multiple Choices                  多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择
301          Moved Permanently                 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替
302          Found                             临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI
303          See Other                         查看其它地址。与301类似。使用GET和POST请求查看
304          Not Modified                      未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源
305          Use Proxy                         使用代理。所请求的资源必须通过代理访问
306          Unused                            已经被废弃的HTTP状态码
307          Temporary Redirect                临时重定向。与302类似。使用GET请求重定向
400          Bad Request                       客户端请求的语法错误，服务器无法理解
401          Unauthorized                      请求要求用户的身份认证
402          Payment Required                  保留，将来使用
403          Forbidden                         服务器理解请求客户端的请求，但是拒绝执行此请求
404          Not Found                         服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面
405          Method Not Allowed                客户端请求中的方法被禁止
406          Not Acceptable                    服务器无法根据客户端请求的内容特性完成请求
407          Proxy Authentication Required     请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权
408          Request Time-out                  服务器等待客户端发送的请求时间过长，超时
409          Conflict                          服务器完成客户端的PUT请求是可能返回此代码，服务器处理请求时发生了冲突
410          Gone                              客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置
411          Length Required                   服务器无法处理客户端发送的不带Content-Length的请求信息
412          Precondition Failed               客户端请求信息的先决条件错误
413          Request Entity Too Large          由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息
414          Request-URI Too Large             请求的URI过长（URI通常为网址），服务器无法处理
415          Unsupported Media Type            服务器无法处理请求附带的媒体格式
416          Requested range not satisfiable   客户端请求的范围无效
417          Expectation Failed                服务器无法满足Expect的请求头信息
500          Internal Server Error             服务器内部错误，无法完成请求
501          Not Implemented                   服务器不支持请求的功能，无法完成请求
502          Bad Gateway                       充当网关或代理的服务器，从远端服务器接收到了一个无效的请求
503          Service Unavailable               由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中
504          Gateway Time-out                  充当网关或代理的服务器，未及时从远端服务器获取请求
505          HTTP Version not supported        服务器不支持请求的HTTP协议的版本，无法完成处理
=========    ================================  ======================================================================================================================================================================

错误处理
================

- 不要发生了错误但给 ``2xx`` 响应，客户端可能会缓存成功的 HTTP 请求；
- 正确设置 HTTTP 状态码，不要自定义；
- Response body 提供：

    - 错误的代码（日志/问题追查）
    - 错误的描述文本（展示给用户）。

示例：

::

    {
       "error": {
          "message": "An active access token must be used to query information about the current user.",
          "type": "OAuthException",
          "code": 2500,
          "fbtrace_id": "DzkTMkgIA7V"
       }
    }


序列化和反序列化
====================

RESTful API 以规范统一的格式作为数据的载体，常用的格式为 JSON 或 XML。确保序列化和反序列化方法的实现，是开发 RESTful API 最重要的一步准备工作。

服务器返回的数据格式，应该尽量使用 JSON，避免使用 XML。

JSON 格式约定
--------------------

- 时间用长整形(毫秒数)，客户端自己按需解析
- 不传 null 字段

使用友好的 JSON 输出
-----------------------

用户在第一次看到你的 API 使用情况，很可能是在命令行下使用 curl 进行的。友好的输出会让他们非常容易的理解你的 API：

::

    {
      "beta": false,
      "email": "alice@heroku.com",
      "id": "01234567-89ab-cdef-0123-456789abcdef",
      "last_login": "2012-01-01T12:00:00Z",
      "created_at": "2012-01-01T12:00:00Z",
      "updated_at": "2012-01-01T12:00:00Z"
    }

而不是这样：

::

    {"beta":false,"email":"alice@heroku.com","id":"01234567-89ab-cdef-0123-456789abcdef","last_login":"2012-01-01T12:00:00Z", "created_at":"2012-01-01T12:00:00Z","updated_at":"2012-01-01T12:00:00Z"}


数据校验
=============

数据校验，是开发健壮 RESTful API 中另一个重要的一环。服务器在做数据处理之前，先做数据校验。如果客户端发送的数据不正确或不合理，服务器端直接向客户端返回 400 错误及相应的数据错误信息即可。常见的数据校验包括：

- 数据类型校验，如字段类型如果是 int，那么给字段赋字符串的值则报错；
- 数据格式校验，如邮箱或密码，其赋值必须满足相应的正则表达式，才是正确的输入数据；
- 数据逻辑校验，如数据包含出生日期和年龄两个字段，如果这两个字段的数据不一致，则数据校验失败。

数据校验虽然是 RESTful API 编写中的一个可选项，但它对 API 的安全、服务器的开销和交互的友好性而言，都具有重要意义。






认证和权限
===============

常用的认证机制是 Basic Auth 和 OAuth，RESTful API 开发中，除非 API 非常简单，且没有潜在的安全性问题，否则，认证机制是必须实现的，并应用到 API 中去。

权限机制是对 API 请求更近一步的限制，只有通过认证的用户符合权限要求，才能访问API。常用的权限机制主要包含全局型的和对象型的：

- 全局型的权限机制，主要指通过为用户赋予权限，或者为用户赋予角色或划分到用户组，然后为角色或用户组赋予权限的方式来实现权限控制；
- 对象型的权限机制，主要指权限控制的颗粒度在 object 上，用户对某个具体对象的访问、修改、删除或其行为，要单独在该对象上为用户赋予相关权限来实现权限控制。


提供人类可读的文档
=======================

提供人类可读的文档让客户端开发人员可以理解你的 API。

除此之在详细信息的结尾，提供一个关于如下信息的API摘要:

- 验证授权，包含获取及使用验证 tokens；
- API 稳定性及版本控制，包含如何选择所需要的版本；
- 一般的请求和响应头信息；
- 错误信息序列格式；
- 不同语言客户端使用 API 的例子，以减少用户尝试使用 API 的工作量。。



API 范例
==============

Github
------------

https://api.github.com/

https://developer.github.com/v3/

::

    GET https://api.github.com/users/{user}
    GET https://api.github.com/user/orgs
    GET https://api.github.com/users/{user}/repos{?type,page,per_page,sort}
    GET https://api.github.com/search/users?q={query}{&page,per_page,sort,order}
    GET https://api.github.com/rate_limit



Twitter
--------------

https://developer.twitter.com/en/docs/api-reference-index

::

    GET account/settings
    GET favorites/list
    GET followers/ids
    GET followers/list
    GET friends/ids
    GET friends/list
    GET geo/search
    GET users/search
    GET users/show

Facebook
-------------

https://developers.facebook.com/docs/apis-and-sdks

::

    GET https://graph.facebook.com/820882001277849
    GET https://graph.facebook.com/820882001277849/feed
    GET https://graph.facebook.com/820882001277849?fields=about,fan_count,website
    GET https://graph.facebook.com/820882001277849/photos


参考资料
===============

1. https://blog.csdn.net/yue7603835/article/details/52536497
2. https://blog.igevin.info/posts/restful-api-get-started-to-write/
3. https://github.com/cocoajin/http-api-design-ZH_CN/blob/master/README.md
4. http://www.cnblogs.com/moonz-wu/p/4211626.html




