---
layout: post
title: RESTful API设计指南
category: 技术
tags: RESTful
---

## 请求

### API版本号

API应该包含版本信息，主要有两种方式：

一种是在URL中放置版本号

```
example.com/api/v1/*
```

另外一种是将版本号放在`HTTP Header`中

```
Accept: application/vnd.example.com.v1+json
```

> 其中 `vnd` 表示 `Standards Tree` 标准树类型，有三个不同的树: `x`，`prs` 和 `vnd`。你使用的标准树需要取决于你开发的项目
>
> - 未注册的树（`x`）主要表示本地和私有环境
>
> - 私有树（`prs`）主要表示没有商业发布的项目
>
> - 供应商树（`vnd`）主要表示公开发布的项目
>
> 后面几个参数依次为应用名称（一般为应用域名）、版本号、期望的返回格式。

**推荐采用第一种方式，即在URL中嵌入版本号的方式，这种做法版本号直观、易于调试。**

**另外，API必须保持向后兼容，引入新版本API的同时确保旧版本API仍然可用。**

### URL格式

- URL的命名必须全部小写
- URL中资源（resource）的命名必须是名词，并且必须是复数形式，例如`employees`

- 多个单词必须使用`-`连接，例如`bank-accounts`

表示资源集合的URL

```
/employees
```

表示具体某个资源的URL

```
/employees/1
```

### 查询字符串

- 查询字符串的参数命名必须使用小驼峰命名法，例如`firstName`，更符合Java和JavaScript的通用规范。

```
/employees?firtName=tom
```

### 非资源请求

有时API调用并不涉及资源（如翻译、计算或搜索），API响应不会返回任何资源，而是执行一个操作并将结果返回给客户端，有时候强行使用名词表示反而不直观。这种情况下允许有例外，可以在URL中使用动词而不是名词，来清楚的区分资源请求和非资源请求，例如：

```
GET /translate?from=de_DE&to=en_US&text=Hallo
GET /calculate?para2=23&para2=432
GET /search?query=Paul
```

### 过滤

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。
下面是一些常见的参数：

```
?limit=10：指定返回记录的数量
?offset=10：指定返回记录的开始位置。
?page=2&perPage=100：指定第几页，以及每页的记录数。
?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
?animalTypeId=1：指定筛选条件
```

### HTTP方法

- GET：获取某个资源或资源集合。GET操作应该是幂等的，且无副作用，不会改变任何数据。
- POST：创建一个新的资源。非幂等的。
- PUT：完整替换某个已有的资源。PUT操作虽然有副作用，但其应该是幂等的。
- DELETE：删除某个资源。DELETE操作有副作用，但也是幂等的。

> 幂等的定义：简单说来就是一个操作符合幂等性，那么相同的数据和参数下，执行一次或多次产生的效果（副作用）是一样的。

2个URL乘以4个HTTP方法可以组成一组功能：

|               | POST（创建）   | GET（读取）                          | PUT（更新）        | DELETE（删除） |
| ------------- | -------------- | ------------------------------------ | ------------------ | -------------- |
| /employees    | 创建一个新员工 | 列出所有员工（ID和名称，不要太详细） | 批量更新员工信息   | 删除所有员工   |
| /employees/56 | （错误）       | 获取56号员工的信息                   | 更新56号员工的信息 | 删除56号员工   |

### HTTP Header

- Accept：告诉服务器需要返回什么类型的实体。采用`application/json`

  > 如果客户端要求返回"application/xml"，服务器端只能返回"application/json"，那么应该返回HTTP状态码 406 Not Acceptable。

- Content-Type：请求的实体数据类型，采用`application/json`

## 响应

- 采用JSON
- JSON的属性命名必须使用小驼峰命名法，更符合Java和JavaScript的通用规范。

### 无返回数据实体

- 实体内容包含2个基本属性：
  - `code`：返回码
  - `message`：返回消息

```
{
    "code": 0,
    "message": "ok"
}
```

### 有返回数据实体

- 在无返回数据实体的基础上，新增1个属性：
  - `data`：返回数据

```
{
    "code": 0,
    "message": "ok",
    "data": null
}
```

```
{
    "code": 0,
    "message": "ok",
    "data": {
        "id": 2,
        "firstName": "harry",
        "lastName": "potter",
        "age": 22
    }
}
```

### 分页数据实体

- 在有返回数据实体的基础上，新增4个基本属性：
  - `page`：第几页
  - `perPage`：每页个数
  - `pageCount`：总页数
  - `total`：总个数

```
{
    "code": 0,
    "message": "ok",
    "page": 1,
    "perPage": 3,
    "pageCount": 4,
    "total": 12,
    "data": [
        {
            "id": 1,
            "firstName": "Tom",
            "lastName": "Cruise",
            "age": 28
        },
        {
            "id": 2,
            "firstName": "Harry",
            "lastName": "Potter",
            "age": 22
        },
        {
            "id": 3,
            "firstName": "Mike",
            "lastName": "Johnson",
            "age": 20
        }
    ]
}
```

### 错误处理

错误响应格式：

```
{
    "code": -1,
    "message": "error"
}
```

### HTTP状态码

如果使用SpringMVC，则默认已定义的HTTP状态码有：

| 状态码 | 用途                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | GET、POST、PUT、DELETE请求成功后均返回200状态码              |
| 400    | 请求有误时返回，一般是由于请求数据和格式不符合服务器要求     |
| 404    | 请求了一个不存在的资源，一般为URL有误时返回                  |
| 406    | 请求的实体类型不对，例如客户端Accept要求返回"application/xml"，服务器端只能返回"application/json"，那么就会返回406状态码 |
| 500    | 请求发生错误，一般是服务器执行请求异常时返回                 |

简单起见，暂不修改或扩展SpringMVC默认定义的HTTP状态码及使用场景

### 另附：常用的HTTP状态码

2开头，成功状态码。

| 状态码 | 原因短语               | 含义                                                     |
| ------ | ---------------------- | -------------------------------------------------------- |
| 200    | OK                     | 服务器已成功处理请求                                     |
| 201    | Created（已创建）      | 对那些要服务器创建对象的请求来说，资源已创建完毕         |
| 202    | Accepted（已接受）     | 请求已接受，但服务器尚未处理                             |
| 204    | No Content（没有内容） | 响应报文包含一些首部和一个状态行，但不包含实体的主体内容 |

3开头，重定向状态码。

| 状态码 | 原因短语                      | 含义                                                         |
| ------ | ----------------------------- | ------------------------------------------------------------ |
| 301    | Moved Permanently（永久移除） | 请求的URL已移走。响应中应该包含一个Location URL，说明资源现在所处的位置 |
| 302    | Found（已找到）               | 与状态码301类似，但这里的移除是临时的。客户端应该用Location首部给出的URL对资源进行临时定位 |

4开头，客户端错误状态码。

| 状态码 | 原因短语                   | 含义                                                         |
| ------ | -------------------------- | ------------------------------------------------------------ |
| 400    | Bad request（坏请求）      | 告诉客户端它发送了一条异常请求                               |
| 401    | Unauthorized（未授权）     | 与适当的首部一起返回，在客户端获得资源访问权之前，请它进行身份认证 |
| 403    | Forbidden（禁止）          | 服务器拒绝了请求                                             |
| 404    | Not Found（未找到）        | 服务器无法找到所请求的URL                                    |
| 406    | Not Acceptable（无法接受） | 客户端可以指定一些参数来说明希望接受哪些类型的实体。服务器没有资源与客户端可接受的URL相匹配时可使用此代码 |
| 410    | Gone（消失了）             | 除了服务器曾持有这些资源之外，与状态码404类似                |

5开头，服务器错误状态码。

| 状态码 | 原因短语                                | 含义                                                     |
| ------ | --------------------------------------- | -------------------------------------------------------- |
| 500    | Internal Server Error（内部服务器错误） | 服务器遇到了一个错误，使其无法为请求提供服务             |
| 503    | Service Unavailable（未提供此服务）     | 服务器目前无法为请求提供服务，但过一段时间就可以恢复服务 |

### HTTP Header

- Content-Type：响应的实体数据类型，采用`application/json`

## 认证

- 采用JWT
- 为方便起见，accessToken通过查询字符串传递

```
/employees?firtName=tom?accessToken=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImlhdCI6MTUzNjA0ODQ0NX0.1JEEnjcbQ3kzJ5YZ1Px1ljBaYcpvooRN9Gn9Sf7wB8o
```

> 有安全要求应该使用HTTPS，并且accessToken通过HTTP Header传递
>
> 如需要经常对接第三方系统，则应当采用OAuth2.0

## 示例

### 列出所有员工信息

请求：

```
GET /api/v1/employees HTTP/1.1
Host: localhost:8901
Accept: application/json
```

响应：

```
HTTP/1.1 200
Content-Type: application/json:charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 29 Aug 2018 02:06:19 GMT

{
    "code": 0,
    "msg": "ok",
    "data": [
        {
            "id": 1,
            "firstName": "Tom",
            "lastName": "Cruise",
            "age": 28
        },
        {
            "id": 2,
            "firstName": "Harry",
            "lastName": "Potter",
            "age": 22
        }
    ]
}
```

### 列出所有员工信息（分页）

请求：

```
GET /api/v2/employees?page=1&perPage=5 HTTP/1.1
Host: localhost:8901
Accept: application/json
```

响应：

```
HTTP/1.1 200
Content-Type: application/json:charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 29 Aug 2018 02:06:19 GMT

{
    "code": 0,
    "msg": "ok",
    "page": 1,
    "perPage": 5,
    "pageCount": 3,
    "total": 12,
    "data": [
        {
            "id": 1,
            "firstName": "Tom",
            "lastName": "Cruise",
            "age": 28
        },
        {
            "id": 2,
            "firstName": "Harry",
            "lastName": "Potter",
            "age": 22
        },
        {
            "id": 3,
            "firstName": "Mike",
            "lastName": "Johnson",
            "age": 20
        },
        {
            "id": 4,
            "firstName": "Lebron",
            "lastName": "James",
            "age": 36
        },
        {
            "id": 5,
            "firstName": "Rajon",
            "lastName": "Rondo",
            "age": 31
        }
    ]
}
```

### 列出所有员工信息，但要求返回的实体类型不受支持

请求：

```
GET /api/v1/employees HTTP/1.1
Host: localhost:8901
Accept: application/xml
```

响应：

```
HTTP/1.1 406
Content-Length: 0
Date: Wed, 29 Aug 2018 02:06:19 GMT
```

### 根据员工ID查询员工信息（URL传参）

请求：

```
GET /api/v1/employees/2 HTTP/1.1
Host: localhost:8901
Accept: application/json
```

响应：

```
HTTP/1.1 200
Content-Type: application/json:charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 29 Aug 2018 02:06:19 GMT

{
    "code": 0,
    "msg": "ok",
    "data": {
        "id": 2,
        "firstName": "Harry",
        "lastName": "Potter",
        "age": 22
    }
}
```

### 根据员工名字查询员工信息（查询字符串传参）

请求：

```
GET /api/v1/employees/?firstName=tom HTTP/1.1
Host: localhost:8901
Accept: application/json
```

响应：

```
HTTP/1.1 200
Content-Type: application/json:charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 29 Aug 2018 02:06:19 GMT

{
    "code": 0,
    "msg": "ok",
    "data": [
        {
            "id": 1,
            "firstName": "Tom",
            "lastName": "Cruise",
            "age": 28
        }
    ]
}
```

### 创建新的员工信息

请求：

```
POST /api/v1/employees HTTP/1.1
Host: localhost:8901
Content-Type: application/json
Accept: application/json

{
  "id": 3,
  "firstName": "Mike",
  "lastName": "Johnson",
  "age": 20
}
```

响应：

```
HTTP/1.1 200
Content-Type: application/json:charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 29 Aug 2018 02:06:19 GMT

{
    "code": 0,
    "msg": "ok",
    "data": {
        “id”： 3
    }
}
```

### 替换更新指定的员工信息

请求：

```
PUT /api/v1/employees/3 HTTP/1.1
Host: localhost:8901
Content-Type: application/json
Accept: application/json

{
  "id": 3,
  "firstName": "Mike",
  "lastName": "Johnson",
  "age": 20
}
```

响应：

```
HTTP/1.1 200
Content-Type: application/json:charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 29 Aug 2018 02:06:19 GMT

{
    "code": 0,
    "msg": "ok"
}
```

### 根据员工ID删除员工信息

请求：

```
DELETE /api/v1/employees/?firstName=tom HTTP/1.1
Host: localhost:8901
Accept: application/json
```

响应：

```
HTTP/1.1 200
Content-Type: application/json:charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 29 Aug 2018 02:06:19 GMT

{
    "code": 0,
    "msg": "ok"
}
```

## 参考文章

[RESTful API设计规范](https://godruoyi.com/posts/resetful-api-design-specifications)

[RESTful API最佳实践](https://zhuanlan.zhihu.com/p/25647039)

[RESTful API设计最佳实践](https://www.zcfy.cc/article/restful-api-design-best-practices-in-a-nutshell-4388.html)

[RESTful API编写指南](https://blog.igevin.info/posts/restful-api-get-started-to-write/)

[撰写合格的REST API](https://mp.weixin.qq.com/s?__biz=MzA3NDM0ODQwMw==&mid=208060670&idx=1&sn=ce67b8896985e8448137052b338093e0&mpshare=1&scene=1&srcid=0217cuxzFnsWD3Kt3bjwjtTA%23rd)

[HTTP状态码详解](http://tool.oschina.net/commons?type=5)

