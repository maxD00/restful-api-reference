# REST API 文档
## 概述

文档用于说明REST API的基本规范.

## 1. __rap2模拟数据与server端响应数据的差异__

### 1.1 响应数据为json数组的处理

rap2模拟响应数据为一个json对象,其collection属性值为所需json数组,结构如下:

    {
      "collections": [
        {
          "path": "一级菜单",
          "is_leaf_node": false,
          "code": "01",
          "parent_code": null,
          "name": "一级菜单",
          "target_url": null
        },
        {
          "path": "一级菜单/二级菜单",
          "is_leaf_node": false,
          "code": "0101",
          "parent_code": "01",
          "name": "二级菜单",
          "target_url": "https://localhot/action"
        }
      ]
    }

server端响应数据直接为json数组,结构如下:

    [
      {
        "path": "一级菜单",
        "is_leaf_node": false,
        "code": "01",
        "parent_code": null,
        "name": "一级菜单",
        "target_url": null
      },
      {
        "path": "一级菜单/二级菜单",
        "is_leaf_node": false,
        "code": "0101",
        "parent_code": "01",
        "name": "二级菜单",
        "target_url": "https://localhot/action"
      }
    ]

### 1.2 响应码

暂时rap2对于请求成功的响应码为200,没有对请求失败的响应进行模拟.

server端按照REST API文档[响应码规范](#status),对响应码进行处理.

## 2. 认证

客户端完成门户SSO过程,请求时携带cookie进行认证.

## 3. HTTP方法

|方法|描述|
|---|---|
|`GET`|获取资源.|
|`POST`|新增资源.|
|`PUT`|更新资源,对于没有传递数据的资源属性,不会进行更新.|
|`DELETE`|删除资源.|

## 4. <span id="status">响应码</span>

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

## 5. 日期与时间格式

+ 日期格式yyyy-MM-dd
+ 时间采用ISO 8601标准,例如:2012-12-20T12:00:00+08:00

## 6.分页功能

`GET`方法获取资源时,可使用分页方式获取资源数组.传递`?page`参数设置页数,页数从1开始,传递`?per_page`设置每页数量.例如:
```bash
curl 'https://localhost/companies?page=1&per_page=100'
```

**注意**:不是每个`GET`方法api都具有分页功能,接口说明会明确标注.
















