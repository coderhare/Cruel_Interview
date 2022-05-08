## 一、写在前面

API设计指南-一个接口文档模板的最佳实践

> 参考链接：https://www.aisoutu.com/a/79070

## 二、API设计的最佳实践

## 1. 基础说明 

### 1.1 背景

[说明文档用途，下面举个例子：]

编写本文的目的是为了将系统功能进行模块化、服务化，将用户的操作以服务的方式提供。系统与系统之间遵循服务规范，将系统与系统之间的交互转为定制化服务交互，以实现系统与系统之间的集成。

### 1.2 基本约束

#### 1.2.1 基本设计原则

请参考上一篇文章《API设计指南-RestAPI设计最佳实践。

#### 1.2.2 字符集

- 所有接口字符集采用UTF-8。

#### 1.2.3 返回类型约束

- 所有接口返回必须是严格定义的JSON类型。

#### 1.2.4 接口版本约束

- 不允许发布无版本号的接口。
- 接口版本首先解决的是一组接口的版本问题。

### 1.3 请求公共约束

请求的基本模板：

```
curl -X[HTTP METHOD] -H "Content-Type: application/json" -H "[token-info]:""https://api-[env-name].[groupname].domain.io/[client-group]/[service-group]/[version]/[endpoint]" -d '{  "head": [meta-parameters],  "body": [content]}'
```

### 1.4 URL整体规划

#### 1.4.1 域名规范

- https://api-[env-name].[groupname].domain.io/[client-group]/[service-group]/[version]/[endpoint]

#### 1.4.2 域名规则

- 开发环境：https://api-dev.payment.domain.io/
- 测试环境：https://api-test.payment.domain.io/
- 预演环境：https://api-staging.payment.domain.io/
- 线上测试环境：https://api-onlinetest.payment.domain.io/
- 生产环境：https://api.payment.domain.io/
- 其中线上测试环境是上线过程中备用，比如线上一共3台生产环境服务器，将其中一台从生产环境切掉，更新程序并且将域名指向它，测试完之后再将生产环境流量切换过来。

### 1.5 基本数据类型约定

此约定是系统整体容错的一部分，但是无论接口使用者还是生产者，都不应该因为此容错而减少自己模块本来需要的容错工作。

![image-20220508203421992](/Users/bytedance/Library/Application Support/typora-user-images/image-20220508203421992.png)



### 1.6 公共输入参数规范

![image-20220508203412187](/Users/bytedance/Library/Application Support/typora-user-images/image-20220508203412187.png)

### 1.7 公共返回对象约定

```json
{
    "responseCode": [responseCode],
    "responseInfo":
    {
        "userMessage": [userMessage],
        "internalMessage": [internalMessage],
        "guideline": [guideline link]
    },
    "link":
    {
        "document": " https://[domain]/docs#zoos",
        "href": [uri - info],
        "title": [doc - title],
        "type": "application/[vnd.yourformat]+json"
    },
    "responseData": [responseData]
}
```

### 1.8 公共错误编码及说明

![image-20220508203335343](/Users/bytedance/Library/Application Support/typora-user-images/image-20220508203335343.png)



### 1.9 公共数据字典

![image-20220508203349963](/Users/bytedance/Library/Application Support/typora-user-images/image-20220508203349963.png)

## 2. 订单服务

### 2.1 查询订单列表

#### 2.1.1 接口规范

#### 2.1.2 输入参数示范

```
curl -XGET -H "Content-Type: application/json" -H "Access-Token:abcd1234" "https://api-dev.haitao.domain.io/mobile/data-platform/v1/orders/base-orders" -d '{  "head": [meta-parameters],  "body": {    "pageSize":10,    "pageNo":1  }}'
```

#### 2.1.3 返回参数示范

```
{  "responseCode": [responseCode],  "responseInfo": {    "userMessage": [userMessage],    "internalMessage": [internalMessage],    "guideline": [guideline link]  },  "link": {    "document":" https://[domain]/docs#zoos",    "href":[uri-info],    "title":[doc-title],    "type":"application/[vnd.yourformat]+json"  },  "responseData": {    "pageCount": 12,    "pageNo": 2,    "data": {    }  }}
```
