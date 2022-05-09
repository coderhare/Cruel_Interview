## 一、写在前面

Restful API设计指南-最佳实践

> 参考链接：https://blog.csdn.net/ananlele_/article/details/107147331



## 二、架构



API是一个接口，许多开发人员可通过该接口与数据进行交互。设计良好的API总是非常易于使用，并使开发人员的工作变得非常顺利。API是开发人员的GUI，如果感到困惑或冗长，则会开始寻找替代方案或停止使用它。

### 1）术语

以下是与REST API相关的最重要的术语

资源  是事物的对象或表示形式，它具有一些关联的数据，并且可以使用一组方法对其进行操作。例如，订单是资源，删除，添加，更新是对这些资源要执行的操作。
集合  是资源集，例如，订单是订单资源的集合。
URL（统一资源定位符）是可用来定位资源并对其执行某些操作的路径。

### 2）API端点

让我们为有一些员工这个表来写一些API ，/getAllEmployees是一个获取员工列表的API。关于员工的其他API 如下所示：

```bash
/addNewEmployee
/updateEmployee
/deleteEmployee
/deleteAllEmployees
/promoteEmployee
/promoteAllEmployees
```

诸如此类的大量其他API端点将用于不同的操作。所有这些将包含许多多余的动作。因此，当API数量增加时，所有这些API端点都将难以维护。

为什么难以维护？
该URL仅应包含资源（名词，如Employee）而不是动作或动词(addNew、deleteAll等)。而API路径/addNewEmployee包含操作addNew 以及资源名称Employee。

那正确的方法是什么？
/companies端点是一个很好的示例，其中不包含任何操作。但是问题是我们如何告诉服务器要在companies资源上执行是否添加，删除或更新的操作呢？

这就是HTTP方法（GET，POST，DELETE，PUT）（也称为动词）发挥作用的地方。

该资源在API端点中应始终为复数形式，如果我们要访问该资源的一个实例，则可以始终在URL中传递ID。

```
GET方法   路径/companies      获取所有公司的列表
GET方法   路径/companies/34   获取ID为34的公司信息
DELETE方法   路径/companies/34   删除ID为34的公司信息
```

在其他一些用例中，如果我们在某个资源下拥有资源，例如公司的员工，则API端点如下表示：

```
GET  /companies/3/employees             从ID为3的公司获取所有员工的名单
GET  /companies/3/employees/45        获取属于ID为3的公司员工45的详细信息
DELETE  /companies/3/employees/45   删除属于公司3的员工45
POST    /companies                              创建一个新公司并返回创建的新公司的详细信息
```

API现在不是更加精确和一致吗？

>  结论：路径应包含多种形式的资源，HTTP方法应定义对资源执行的操作的类型。

### 3）HTTP方法（动词）

HTTP定义了几套方法，这些方法指示对资源执行的操作的类型。

URL是一个句子，其中资源是名词，而HTTP方法是动词。

重要的HTTP方法如下：

GET方法从资源中请求数据
例如，/companies/3/employees  返回公司3中所有员工的列表。
POST方法请求服务器在数据库中创建资源，通常是在提交Web表单时使用
例如，/companies/3/employees  创建公司3的新雇员。
PUT方法请求服务器更新资源或创建资源（如果不存在）。
例如，/companies/3/employees/john 请求服务器更新或创建（如果不存在）公司3下的员工中的john资源
DELETE方法请求应从数据库中删除资源或其实例。
例如，/companies/3/employees/john/ 请求服务器从公司3下的员工集合中删除john资源。

### 4）HTTP响应状态码

当客户端通过API向服务器提出请求时，客户端应该知道反馈，无论是成功还是请求错误。HTTP状态代码是一堆标准化代码，在各种情况下都有各种说明。服务器应始终返回正确的状态代码。
以下是HTTP代码的重要分类：

2xx（成功类别）

这些状态代码表示服务器已接收并成功处理了请求的操作。

200  Ok  标准的HTTP响应，表示GET，PUT或POST成功。
201  已创建  每当创建新实例时，都应返回此状态代码。例如，使用POST方法创建新实例时，应始终返回201状态代码。
204  No Content  表示请求已成功处理，但未返回任何内容。
删除就是一个很好的例子。
API DELETE /companies/43/employees/2 删除雇员2，并且作为回报，我们明确要求系统删除，因此API的响应正文中不需要任何数据。如果有任何错误（例如如果employee 2数据库中不存在），则响应代码将不是2xx Success Category而是4xx Client Error category。
3xx（重定向类别）

304 Not Modified表示客户端已经在其缓存中包含响应。因此，无需再次传输相同的数据。
4xx（客户端错误类别）

这些状态代码表示客户端提出了错误的请求。

400  错误的请求  表示客户端的请求未处理，因为服务器无法理解客户端的要求。
401  Unauthorized  表示不允许客户端访问资源，应使用所需的凭据重新请求。
403  禁止  表示请求有效并且客户端已通过身份验证，但是由于任何原因不允许客户端访问页面或资源。例如，有时不允许授权的客户端访问服务器上的目录。
404  找不到  表示请求的资源现在不可用。
410  已消失  表示已被有意移动的请求资源不再可用。
5xx（服务器错误类别）

500 Internal Server Error  表示请求有效，但是服务器完全混乱，要求服务器提供某些意外条件。
503 服务不可用  表示服务器已关闭或无法接收和处理该请求。通常，如果服务器正在进行维护。

### 5）字段名称大小写约定

您可以遵循任何大小写约定，但请确保在整个应用程序中保持一致。如果请求正文或响应类型为JSON，则请遵循camelCase保持一致性。

### 6）搜索，排序，过滤和分页

所有这些操作仅是对一个数据集的查询。将没有新的API集来处理这些操作。我们需要使用GET方法的API附加查询参数。
让我们通过几个示例来了解如何实现这些操作。

排序  如果客户想要获取公司的排序列表，则GET /companies 应在查询中接受多个排序参数。
例如，GET /companies?sort=rank_asc将按升序对公司进行排序。
过滤  为了过滤数据集，我们可以通过查询参数传递各种选项。
例如，GET /companies?category=banking&location=india 将使用“banking”的公司类别以及位置在india的公司列表数据进行过滤。
搜索  在公司列表中搜索公司名称时，API端点应为GET /companies?search=Digital
分页  当数据集太大时，我们将数据集分成较小的块，这有助于提高性能并更容易处理响应。例如。GET /companies?page=23意味着在第23页上获取公司列表。
如果在GET方法中添加许多查询参数会使URI太长，则服务器可能会以414 URI Too longHTTP状态进行响应，在这种情况下，参数也可以在方法的请求主体中传递POST。

### 7）版本控制

当你的API受到外部的欢迎时，一些重大更改、升级API，也会导致破坏现有的产品或服务。`http://api.robinpm.com/v1/companies/34/employees`是一个很好的例子，该路径中包含API的版本号。如果有重大更新，我们可以将新的API命名为v2或v1.x.x
