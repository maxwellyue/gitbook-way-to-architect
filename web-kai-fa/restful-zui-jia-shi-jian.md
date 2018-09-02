# Restful最佳实践

RESTful 规范、易懂和优雅，一个结构清晰、易于理解的 API 完全可以省去许多无意义的沟通和文档。并且 RESTful 现在越来越流行，也有越来越多优秀的周边工具（例如文档工具 Swagger）。

## 协议

如果能全站 HTTPS 当然是最好的，不能的话也请尽量将登录、注册等涉及密码的接口使用 HTTPS。

## 给API加上版本

* 版本放到URL里（`/api/v1...`\)
* 使用 `Accept` HTTP header，来传递需要的版本

> Github 的格式：application/vnd.github\[.version\].param\[+json\]，
>
> 其中，version指定版本，param是想要的格式，txt，html等。

## 使用名词，而不是动词

我经常看到有人使用动词而不是名词来表示资源名称，例如下面这些：

* _/getProducts_
* _/listOrders_
* _/retreiveClientByOrder?orderId=1_

从结构整洁和一致角度考虑，你应该总是使用名词。而且，巧妙使用 HTTP 方法（GET，POST）可以把想要的操作从资源名称上去除。如下面的例子：

* _GET /products_  返回所有产品列表
* _POST /products_  添加产品到产品列表
* _GET /products/4_  提取Id为4的产品
* _PATCH/PUT /products/4_  更新Id为4的产品

**使用复数**：同一资源命名，混合使用单数和复数形式不是好主意。很快就会混淆，带来不一致。即使对 `show/delete/update` 操作，使用 `/artists` 而不是 `/artist` 也更好点。

**使用嵌套**：如果想获取全部的子集，使用嵌套路由来让风格简洁。例如想从所有唱片中选取特定的，使用 `GET /artists/8/albums`（备注：这里8就是所谓嵌套路由，指导选取哪个唱片）

**`HEAD` 和 `GET` ：**不能改变资源状态，必须是安全的，不要写出这样的url： `GET /deleteProduct?id=1`

**PUT 和 PATCH** ：都用于修改操作，区别是 PUT 需要提交整个对象，而 PATCH 只需要提交修改的信息。在实践中可以选择其中一种。

**POST 创建对象**：究竟该用表单提交更好些还是用 JSON 提交更好些。其实两者都可以，唯一的区别是 JSON 可以比较方便的表示更为复杂的结构（有嵌套对象）。另外无论使用哪种，请保持统一，不要两者混用。

## 使用合适的 HTTP 状态码

请求返回时，无论请求成功与否，总是使用正确的返回码。下面是一些可能用到的状态码。

#### 成功状态码（2XX系列） {#成功状态码（2XX系列）}

* `201 Created` 当成功创建资源时（INSERT）
* `202 Accepted` 当请求被接受，并放到后台执行时（异步任务）
* `204 No Content` 当请求成功，但是没有内容返回时（例如 DELETE 时）

#### 客户端错误（4xx系列） {#客户端错误（4xx系列）}

* `400 Bad Request` 当处理querystring或http body时出错（例如非法JSON）
* `401 Unauthorized` 认证失败
* `403 Forbidden` 认证成功时，操作或请求的资源不被允许
* `406 Not Acceptable` 请求格式不被接受（例如试图请求JSON数据，但服务器只提供XML）
* `410 Gone` 请求的资源被永久删除（备注：咋判断是不存在还是被永久删除了）
* `422 Unprocessable entity` 当创建对象时发生可用性错误

## 总是返回一致的错误内容

当发生错误时，总是返回一致的错误描述。错误结构总是相同，这样更容易解析错误信息。  
如下描述，清晰，简单，自说明：

```text
HTTP/1.1 401 Unauthorized
{
    "status": "Unauthorized",
    "message": "No access token provided.",
    "request_id": "594600f4-7eec-47ca-8012-02e7b89859ce"
}
```



## 内容来源

英文：[Some REST best practices](https://bourgeois.me/rest/)    

中文： [一些REST最佳实践](http://weibo.com/p/1001603873537160306692)

[我所认为的RESTful API最佳实践](http://www.scienjus.com/my-restful-api-best-practices/)

