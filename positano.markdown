# Positano API Reference

* Author: DingLin (dinglin1991@yeah.net)

## 概述

* 基于 HTTP
* RESTful
* 采用标准 HTTP Code 返回错误
* 返回格式为 JSON

### 分页

默认返回30条记录，可以通过 `per_page` 参数指定最多每次返回100条记录，但是有些 API 因为一些技术原因，会忽略 `per_page` 参数。页码用 `?page` 参数指定，从1开始。
在`参数`中不再明确列出 `page` 和 `per_page` 参数，返回结果中包含 `current_page/per_page/count`，就表示支持分页功能。

cURL 请求范例：

```
curl https://api.soyep.com/v1/friendships?page=2&per_page=100
```

<!---
### 请求速率

1小时支持5000次请求，你能通过 HTTP Header 了解到请求速率的限制：

cURL 请求范例：

```
curl -i https://api.soyep.cim/v1/friendships
```

返回范例：

| Header 字段 | 描述 |
|---|---|
| X-RateLimit-Limit | 1小时内最大请求数 |
| X-RateLimit-Remaining | 在时间窗口内，剩余的请求数 |
| X-RateLimit-Reset | 请求窗口重置的时间，UTC epoch seconds 表示 |

```
HTTP/1.1 200 OK
Date: Thu, 30 Oct 2014 16:27:06 GMT
Status: 200 OK
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4999
X-RateLimit-Reset: 1372700873
```

超过请求速率后，你会收到一个 429 Too Many Requests 的错误。

```
HTTP/1.1 403 Too Many Requests
Date: Tue, 20 Aug 2013 14:50:41 GMT
Status: 429 Too Many Requests
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1377013266

{
    error: "来自 xxx.xxx.xxx.xxx 的 API 请求已经超过限制"
}
```
--->

### Header

所有 API 请求都需要包含如下 headers:

| header | 说明 |
|--------|--------|
| Accept-Language | 当前客户端所使用的语言，目前只支持 `en-US` 和 `zh-CN`，无效的设置将默认为 `en-US` |

### Response

HTTP Code 大于等于 `200` 且小于 `300` 表示请求成功，反之则请求失败，失败会有错误信息，错误信息在 `error` 中。

### 字段模板

为了方便 API 文档的阅读，将会用模板重用一些重复的字段结构。

#### ID

所有返回和提交的 ID 都是加密后的字符串("516055075accc1e4067dd5ff6b2682cd")，在 API 中用到 ID 时，都将以 `<id>` 替代。


#### 用户基本信息字段结构

在 API 返回基本的用户信息时，将会以 `<mini_user>` 替代如下结构：

```
"id":<id>,
"username":"asdaasd",
"nickname":"user9",
"avatar":{
  "url":"http://catch-avatars.qiniudn.com/sJAUYG6nc84glXkq.jpg",
  // 有可能没有 thumb_url，因为是 background 方式生成缩略图的
  "thumb_url":"http://catch-avatars.qiniudn.com/thumb_sJAUYG6nc84glXkq.jpg"
},
"latitude":0.0,
"longitude":0.0,
"introduction":"",
"badge":null,
"created_at":1441347808,
"updated_at":1441693888,
"last_sign_in_at":1441347780
```

#### 用户信息字段结构

在 API 返回用户信息时，将会以 `<user>` 替代如下结构：

```
"id":<id>,
"username":"tumayun",
"nickname":"tumayun",
"avatar":{
  "url":"http://catch-avatars.qiniudn.com/sJAUYG6nc84glXkq.jpg",
  // 有可能没有 thumb_url，因为是 background 方式生成缩略图的
  "thumb_url":"http://catch-avatars.qiniudn.com/thumb_sJAUYG6nc84glXkq.jpg"
},
"latitude":28.3213,
"longitude":117.001,
"introduction":"",
"badge":"apple",
"last_sign_in_at":1433930183, // UNIX 时间戳
"created_at":1433930183, // UNIX 时间戳
"updated_at":1433930183, // UNIX 时间戳
"providers":{ // 第三方平台绑定情况
  "github":false,
  "dribbble":false,
  "instagram":false
},
"master_skills":[
  {
    <skill_with_category>
  },
  ...
],
"learning_skills":[
  {
    <skill_with_category>
  },
  ...
]
```

## 注册登录

### 手机号和验证码认证

**发送验证码到指定手机号。**

对于1个手机号，1小时内只发送20次，24小时内只发送50次。

```
POST /v1/sms_verification_codes
```

| 参数 | 描述 |
|--------|--------|
| mobile | 手机号 |
| phone_code | 国家代码 |
| method | 发送方式，`sms` 短信方式，`call` 语音方式 |

cURL 请求范例：

```
curl -X POST -H "Content-Type: application/json" -d '{"phone_code":"86","mobile":"12345678","method":"sms"}' http://api.soyep.com/v1/sms_verification_codes
```

返回范例：

```
{}
```

如果发送次数过多，则返回 429 Too Many Requests 的错误。

```
HTTP/1.1 403 Too Many Requests
Date: Tue, 20 Aug 2013 14:50:41 GMT
Status: 429 Too Many Requests

{
  error: "发给手机号 xxxxxxxxxxx 的验证码数量已经超过限制"
}
```

**发送手机号 (mobile) 和验证码 (verify_code)，可获取相应的 access_token。**

```
POST /v1/auth/token_by_mobile
```

| 参数 | 描述 |
|--------|--------|
| mobile | 手机号 |
| phone_code | 国家代码 |
| verify_code | 验证码 |
| expiring | access_token 过期时间。单位为秒，设置为0表示永不过期，不设置默认7天过期 |
| client | 用于推送, official=0, company=1, local=2, 默认是official |

cURL 请求范例：

```
curl -X POST   -H "Content-Type: application/json" -d '{"phone_code":"86",
"mobile":"12345678", "verify_code": "23397"}'
http://api.soyep.com/v1/auth/token_by_mobile
```

返回范例：

```
{
  "access_token":"Bs2H_1hVWLseG7rDy5hz1422602902.8925965",
  "user": {
    "id":<id>,
    "username":"ruanwz",
    "nickname":"ruanwz",
    "avatar":{
      "url":"http://catch-avatars.qiniudn.com/sJAUYG6nc84glXkq.jpg",
      // 有可能没有 thumb_url，因为是 background 方式生成缩略图的
      "thumb_url":"http://catch-avatars.qiniudn.com/thumb_sJAUYG6nc84glXkq.jpg"
    },
    "mobile":"12345678",
    "phone_code":"86"
  }
}
```

cURL 请求范例（1小时过期）：

```

curl -X POST -H "Content-Type: application/json" -d '{"phone_code":"86",
"mobile":"12345678", "verify_code": "23397", "expiring": 3600}'
http://api.soyep.com/v1/auth/token_by_mobile
```

如果手机号和验证码错误，或验证码已经过期，则返回 HTTP 401.
{
  error: "手机号或验证码错误"
}

### 发送用户名,手机号码，发起注册,等待接收手机验证码

```
POST   /v1/registration/create
```

| 参数 | 描述 |
|--------|--------|
| mobile | 手机号 |
| nickname | 用户昵称 |
| phone_code | 国家码 |
| longitude | longitude |
| latitude | latitude |

cURL 请求范例：

```
curl -X POST -H "Content-Type: application/json" -d '{"phone_code":"86",
"mobile":"15626044835", "nickname": "testnick", "latitude": 123.123, "longitude": 23.23}'
http://api.soyep.com/v1/registration/create
```

返回范例：

```
{
  "username":"testnick",
  "nickname":"testnick",
  "mobile":"15626044835",
  "phone_code":"86"
  "state":"blocked"
  "sent_sms": true, // true 表示验证码发送正常，false 表示验证码发送失败
}
```

### 验证手机验证码完成注册

```
PUT   /v1/registration/update
```
| 参数 | 描述 |
|--------|--------|
| mobile | 手机号 |
| phone_code | 国家码 |
| token | 手机短信收到的验证码 |
| client |可选，用于推送, official=0, company=1, local=2 ，默认为0|
| expiring | 可选，access_token 过期时间。单位为秒，设置为0表示永不过期，不设置默认一年过期 |

cURL 请求范例：
```
 curl -X PUT -H "Content-Type: application/json" -d '{"phone_code":"86","mobile":"15626044835", "token": 70215}' http://api.soyep.com/v1/registration/update

```
返回范例：
```
{
  "access_token":"DAo4Cs4dkaE-7ADS-63Q1422604371.509334",
  "user":{
    "id":<id>,
    "username":"testnick",
    "nickname":"testnick",
    "avatar":{
      "url":"http://catch-avatars.qiniudn.com/sJAUYG6nc84glXkq.jpg",
      // 有可能没有 thumb_url，因为是 background 方式生成缩略图的
      "thumb_url":"http://catch-avatars.qiniudn.com/thumb_sJAUYG6nc84glXkq.jpg"
    },
    "mobile":"15626044835",
    "phone_code":"86",
    "state":"active"
  }
}
```
### 用 Access Token 调用其他 API

现在你可以通过 **access_token** 来调用其他 API 了，比如：

```
GET /v1/xxx
```

cURL 请求范例：

```
curl https://api.soyep.com/v1/xxx  -H 'Authorization: Token token="p7DvqB4MoT5ux-B1xg"'
```

如果 access_token 不存在或过期了，则返回 HTTP 401

HTTP Token: Access denied.

## User 个人信息 API

### 可能认识的好友

```
GET /v1/user/may_know_friends
```

#### 参数

无

#### 示例

```
curl https://api.soyep.com/v1/user/may_know_friends -H Authorization: Token token="kuH3PbRifgSATCanYwxd1418031570.162303"'
```

#### 响应

```
{
  "friends":[
    {
      <mini_user>,
      "common_friend_names":[
        "friend2",
        "friend3",
        "friend4",
        "friend5"
      ]
    },
    .
    .
    .
  ]
}
```

### 获取个人信息

```
GET /v1/user
```

#### 参数

无

#### 示例

```
curl https://api.soyep.com/v1/user -H 'Authorization: Token oken="kuH3PbRifgSATCanYwxd1418031570.162303"'
```

#### 响应

```
{
  <user>,
  "push_content":true,
  "phone_code":"86",
  "mobile":"15158161111",
  "pusher_id":"439ee7d09180529d3442bd25",
  "state":"active",
  "mute_started_at_string":"23:30", // 防打扰开始时间, UTC 时间，需要转换成客户端当前时区后再显示
  "mute_ended_at_string":"07:30" // 防打扰结束时间, UTC 时间，需要转换成客户端当前时区后再显示
}
```

### 更新个人信息

`mute_started_at_string` 和 `mute_ended_at_string` 都有值时，勿扰功能开启，都为空时，勿扰功能关闭。

```
PATCH /v1/user
```

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| nickname | String | 否 | 昵称 |
| username | String | 否 | 用户名，必须唯一（忽略大小写），只能是由字母、数字、下划线组成，长度为4到16个字符 |
| latitude | Float | 否 | 纬度 |
| longitude | Float | 否 | 经度 |
| push_content | Boolean | 否 | 标识推送时是推送消息内容还是推送通知，true 推送消息内容，false 推送通知 |
| introduction | Text | 否 | 个人介绍 |
| badge | String | 否 | 徽章，有：android apple ball bubble camera game heart music palette pet plane star steve tech wine |
| mute_started_at_string | String | 否 | 防打扰开始时间，如：23:30，UTC 时间 |
| mute_ended_at_string | String | 否 | 防打扰结束时间，如：07:30，UTC 时间 |

#### 示例

```
curl -X PATCH https://api.soyep.com/v1/user -F badge=apple -F username=tumayun -F latitude=26.331920 -F longitude=168.3097112 -F nickname=Tumayun -F push_content=false -F mute_started_at_string=23:30 -F mute_ended_at_string=07:30 -H 'Authorization: Token oken="E9PnSDQMRZvjzL84yBi21418033718.2053812"'
```

#### 响应

```
{
  <user>,
  "push_content":true,
  "phone_code":"86",
  "mobile":"15158161111",
  "pusher_id":"439ee7d09180529d3442bd25",
  "state":"active",
  "mute_started_at_string":"23:30", // 防打扰开始时间，UTC 时间
  "mute_ended_at_string":"07:30" // 防打扰结束时间，UTC 时间
}
```

### 设置头像

```
PATCH /v1/user/set_avatar
```

### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| avatar | File | 是 | 头像，目前只支持 jpg |

### 示例

```
curl -X PATCH https://api.soyep.com/v1/user/set_avatar -F avatar=@/Users/tumayun/workspcae/park_server/spec/fixtures/image.jpg -H 'Authorization: Token token="test-token"'
```

### 响应

```
{"avatar":{"url":"https://s3.cn-north-1.amazonaws.com.cn/ruanwz-test-public/a2692db13f2c2879f7ae118a46b62bd9/image.jpg"}}
```

### 更新手机号流程

1. 发送当前手机号验证码 (POST /v1/sms_verification_codes)
2. 校验当前手机号验证码  (PATCH /v1/user/check_verify_code)
3. 发送新手机号验证码 (POST /v1/user/send_update_mobile_code)
4. 校验新手机号验证码，通关验证后更新手机号为新手机好 (PATCH /v1/user/update_mobile)

### 验证更新手机号请求的验证码

```
PATCH /v1/user/check_verify_code
```

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| token | String | 是 | 短信验证码 |

#### 示例

```
curl -X PATCH https://api.soyep.com/v1/user/check_verify_code -F token=1234 -H 'Authorization: Token oken="E9PnSDQMRZvjzL84yBi21418033718.2053812"'
```

#### 响应

```
{}
```

### 发送新手机号验证码

```
POST /v1/user/send_update_mobile_code
```

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| phone_code | String | 是 | 国家码 |
| mobile | String | 是 | 手机号 |
| method | String | 是 | 发送方式，`sms` 短信方式，`call` 语音方式 |

#### 示例

```
curl -X POST -H 'Authorization: Token oken="E9PnSDQMRZvjzL84yBi21418033718.2053812"' -H "Content-Type: application/json" -d '{"phone_code":"86","mobile":"12345678","method":"sms"}' https://api.soyep.com/v1/user/send_update_mobile_code
```

#### 响应

```
{}
```

### 更新手机号

```
PATCH /v1/user/update_mobile
```

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| phone_code | String | 是 | 手机号国家码，详见 [http://countrycode.org/](ttp://countrycode.org/) |
| mobile | String | 是 | 手机号 |
| token | String | 是 | 短信验证码 |

#### 示例

```
curl -X PATCH https://api.soyep.com/v1/user/update_mobile -F phone_ode=86 -F mobile=15158166372 -F token=131421 -H 'Authorization: Token oken="E9PnSDQMRZvjzL84yBi21418033718.2053812"'
```

#### 响应

```
{}
```

## Contact 通讯录

### 上传通讯录并返回已注册的通讯录好友

**覆盖式上传，上传后会删除之前的通讯录，返回已注册的通讯录好友，并且自动添加好友**

```
POST /v1/contacts/upload
```

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| contacts | JSON | 是 | contacts JSON format, 如 "[{\"name\":\"abc\",\"number\":\"15158166372\"},{\"name\":\"bac\",\"number\":\"15158166723\"}]"|

#### 示例

```
curl htts://api.soyep.com/v1/contacts/upload -F contacts="[{\"name\":\"abc\",\"number\":\"15158160004\"},{\"name\":\"bac\",\"number\":\"15158160005\"}]" -H 'Authorization: Token token="sVNxda9nywMLZkuzUqf31422601654.468095"'
```

#### 响应

```
{
  "registered_users":[
    {
      "contact_name":"bac",
      <mini_user>,
      distance: 1234.5
    },
    .
    .
    .
  ]
}
```

## Users 用户

### Type A Head (@ 用户)

一次最多匹配到 5 个用户，关键字越长，匹配越精确

```
GET /v1/users/typeahead
```

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| q | String | 是 | username 前缀 |

#### 示例

```
curl https://api.soyep.com/v1/users/typeahead?q=t -H 'Authorization: Token token="EtErCK18xN9pxakiCPp61418029033.582837"'
```

#### 响应

```
{
  "users": [
    {
      "id": "45533a143c5c912dc7fce6a9d4a1a1a5",
      "avatar": {
        "thumb_url": "https://s3.cn-north-1.amazonaws.com.cn/yep-avatars/thumb_19ac575a-c32e-481c-9398-a850b61c6561-1450874271.jpg"
      },
      "username": "tonyho",
      "nickname": "tonyho"
    },
    {
      "id": "a2692db13f2c2879f7ae118a46b62bd9",
      "avatar": {
        "thumb_url": "https://s3.cn-north-1.amazonaws.com.cn/yep-avatars/thumb_51cc3185-c36a-465b-93ee-1b0c2889cea6-1451304072.jpg"
      },
      "username": "tumayun",
      "nickname": "tumayun"
    },
    {
      "id": "4fe032e9ad285e75e31c8a06c584f1b8",
      "avatar": {
        "thumb_url": "https://s3.cn-north-1.amazonaws.com.cn/yep-avatars/thumb_e30d2a38-7057-49bf-9ee3-6da1a7ee3937-1450874344.jpg"
      },
      "username": "tifan",
      "nickname": "d zhang"
    }
  ]
}
```

### 搜索用户

```
GET v1/users/search
```

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| q | String | 是 | mobile|username|nickname，其中 username 和 nickname 是前缀匹配，mobile 则是全文匹配  |

#### 示例

```
curl https://api.soyep.com/v1/users/search\?q\=15158161111 -H 'Authorization: Token token="EtErCK18xN9pxakiCPp61418029033.582837"'
```

#### 响应

```
{
  "users":[
    {
      <mini_user>
      distance: 10.1 // 距离，单位 km
    }
  ],
  "current_page":1,
  "per_page":30,
  "count":1
}
```

### 校验手机号是否可用（无需登录）

```
GET v1/users/mobile_validate
```
** phone code 合法, mobile 合法，且具有唯一性    
在设置中修改手机号时，不能拿当前手机号去校验，否则会返回手机号已经被使用**

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| phone_code | String | 是 | 国家码 |
| mobile | String | 是 | 手机号 |

#### 示例

```
curl https://api.soyep.com/v1/users/mobile_validate\?phone_code\=86\&mobile\=15158166372
```

#### 响应

* 手机号可用

```
{  
  "available":true
}
```

* 手机号不可用

```
{  
  "available":false,
  "message":"手机号已经被使用"
}
```


### 获取指定用户信息（by id）

```
GET /v1/users/:id
```

#### 参数

| 名称 | 类型 | 是否必需 | 描述 |
|---|---|---|---|
| id | String | 是 | user id |

#### 示例

```
curl https://api.soyep.com/v1/users/90913b93738c8a627129e49db32eeec3 -H 'Authorization: Token token="test-token"'
```

#### 响应

```
{
  <user>
}
```


## 举报（reports）

### 举报用户

```
POST /v1/users/:id/reports
```

#### 参数

名称 | 类型 | 是否必需 | 描述
--- |--- |--- |--- |
id | String | 是 | 要举报的用户 ID
report_type | Integer | 是 | 0 表示色情低俗, 1 表示广告骚扰, 2 表示诈骗, 3 表示其他
reason | Text | 否 | 举报原因，当 report_type 为 3 时，reason 为必填

#### 示例

```
curl -X POST https://api.soyep.com/v1/users/bc93fe60a44cf376edeb98a9d68d85b9/reports -F report_type=1 -F reason=test -H 'Authorization: Token token="test-token"'
```

#### 响应

只返回 http code

### 举报话题

```
POST /v1/topics/:id/reports
```

#### 参数

名称 | 类型 | 是否必需 | 描述
--- |--- |--- |--- |
id | String | 是 | 要举报的话题 ID
report_type | Integer | 是 | 0 表示色情低俗, 1 表示广告骚扰, 2 表示诈骗, 3 表示其他
reason | Text | 否 | 举报原因，当 report_type 为 3 时，reason 为必填

#### 示例

```
curl -X POST https://api.soyep.com/v1/topics/bc93fe60a44cf376edeb98a9d68d85b9/reports -F report_type=1 -F reason=test -H 'Authorization: Token token="test-token"'
```

#### 响应

只返回 http code

## Oauth (绑定第三方平台账号)

### 流程

1. 客户端通过 WebView 发起请求`https://api.soyep.com/auth/:provider?_tkn=test-token`，其中`_tkn`为`access_token`值
2. 客户端等待接收绑定是否成功的通知

### 判断绑定结果

在一系列跳转结束后

1. URL 为 `yep://auth/success`，则说明绑定成功，其余 URL 都表示绑定失败
2. URL 为 `yep://auth/failure`，绑定失败，且如果 URL 携带 `error` 参数，则表示失败原因

