# REST API 文档
## 概述

文档用于说明REST API的基本规范.

## 1. HTTP方法

|方法|描述|
|---|---|
|`GET`|获取资源.|
|`POST`|新增资源.|
|`PUT`|更新资源,对于没有传递数据的资源属性,不会进行更新.|
|`DELETE`|删除资源.|

## 2. URL

URL设计遵守 RFC 3986 的规范，并且不使用大写字母，使用下划线`_`进行单词分割，表达资源时采用其英文复数形式。

为避免URL多长、层级过深导致的可读性下降，存在子资源时，减少通过父资源层层定位子资源的方式，将子资源作为根资源`root`使用。
以公司`company`、部门`department`、雇员`employee`为例，公司下面存在若个部门，部门便为公司的子资源，部门下存在若干雇员，雇员为部门的子资源，相关的接口设计如下：

```
### 公司URLS ###
DELETE   /companies/{companyld}                    删除某一公司
GET     /companies/{companyld}                     获取某一公司信息
POST    /companies                                 新增公司
PUT     /companies/{companyld}                     更新某一公司信息
### 部门URLS ###
DELETE  /departments/{departmentld}                删除某一部门
GET     /departments/{departmentld}                获取某一部门信息
GET     /companies/{companyld}/departments         获取某一公司下所有部门
POST    /companies/{companyld}/departments         在某一公司下新增部门
PUT     /departments/{departmentld}                更新某一部门信息
### 雇员URLS ###
DELETE  /employees/{employeeld}                    删除某一雇员
GET     /employees/{employeeld}                    获取某一雇员信息
GET     /departments/{departmentld}/employees      获取某一部门下所有雇员
POST    /departments/{departmentld}/employees      在某一部门下新增雇员
PUT     /employees/{employeeld}                    更新某一雇员信息
```

## 3. 媒体类型

接口资源可使用多种媒体类型。媒体类型通过请求头的Accept属性控制，属性值格式为：

```
application/vnd.cngc[.版本].param[+json]
```

其中，param参数指定媒体类型，如text、html、excel等等

例如需要调用某个接口的v2版本，并且返回的资源信息的excel文件，则请求头Accept属性设置为：

```
Accept: application/vnd.cngc.v2.excel
```

又如需要获取富文本的评论信息，其媒体类型为html，并且数据以Json格式返回，响应数据包含在body_html属性中，则请求头Accept属性设置为：

```
Accept: application/vnd.cngc.v2.html+json
```

若**不指定**Accept属性，则为调用此接口的**最新版本**，并且以Json格式返回响应体数据。

## 4. 响应码

|响应码|描述|
|---|---|
|`200`|请求成功,并获取到响应内容.|
|`201`|资源创建成功.|
|`202`|服务端已经接收到请求,但是尚未处理完毕,也不保证处理成功,用于异步请求|
|`204`|请求成功,没有响应内容.|
|`301`|资源永久重定向到`Location`指向的地址,对此资源的操作,都需使用新地址.|
|`302`|资源临时重定向到`Location`指向的地址,此次对资源的操作使用新地址,后续操作仍使用原地址.|
|`400`|请求时发送的数据,不符合json格式.|
|`401`|未携带有效的身份认证凭证.|
|`403`|认证的用户不具有此接口的访问权限.|
|`404`|操作的资源不存在.|
|`422`|请求体中的某些属性值不符合要求.|
|`500, 501, 502, etc`|服务端内部错误.|

*错误代码补充说明*

客户端请求错误`400`, `401`, `403`, `404`, `etc`响应数据含有message属性,用于描述错误的基本信息,如:

    {
      "message": "您无权访问此接口!"
    }

客户端请求错误`422`的响应数据包含具体的无效数据属性`attribute`,如:

    {
      "message": "提交的数据属性值不符合要求!",
      "errors": [
        {
          "code": "missing_attribute",
          "attribute": "identity",
          "message": "身份证不能为空!"
        }
      ]
    }

响应数据中,通过code属性进行属性值的错误类别说明:

|错误code|描述|
|-------|----|
|`missing_attribute`|属性值的必填校验错误.|
|`already_exists`|属性值的唯一性校验错误.|
|`invalid_format`|属性值不符合规定的格式要求.|
|`missing_resource`|属性值需要关联某一资源,但资源不存在.如使用了不存在的数据字典项.|
|`custom`|自定义类型属性值错误,通过`message`属性获取错误描述|

`500`系列的服务端错误不返回响应数据.

## 5. 编码

请求与响应统一采用utf-8编码。

## 6. 日期与时间格式

+ 日期格式yyyy-MM-dd
+ 时间采用ISO 8601标准,例如:2012-12-20T12:00:00+08:00

## 7. 分页功能

`GET`方法获取资源时,可使用分页方式获取资源数组.传递`?page`参数设置页数,页数从1开始,传递`?per_page`设置每页数量.例如:

```bash
curl 'https://localhost/companies?page=1&per_page=100'
```

*注意*:不是每个`GET`方法api都具有分页功能,支持的接口会明确标注.

## 8. 多层级结构数据的响应

多层级结构数据,也即树形结构数据,大数据量下对服务端造成较大压力,同时过多的数据也不便前端处理与显示.利用数据的层级结构,单独加载某一节点下的直接子节点,实现数据的懒加载或异步加载,能够提高用户的操作体验,降低计算与网络资源的消耗.

参数`?recursive`控制是否对数据进行递归查询,`?recursive=true`时返回所有后代资源,`?recursive=false`时仅返回直接子节点资源.

*注意*:不是所有多层级结构数据都支持`?recursive`参数,支持的接口会明确标注.

## 9. 排序功能

返回集合的接口,可是使用参数`?sort`设定排序,使用逗号`,`分割多个排序字段,字段默认正序排序.在字段前增加前缀`-`设置字段为倒序.

*注意*:不是所有接口都支持`?sort`参数,支持的接口会明确标注.











