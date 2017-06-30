
# **可信物联网云平台第三方api接口**

- [一、简介](#Introduction)
- [二、状态码](#StatusCode)
- [三、签名验证](#Sign)
- [四、接口](#API)
    - [通过lock_id获取锁详情](#LockGet)
    - [通过Mac获取锁详情](#LockByMacGet)
    - [网络远程开锁（需要网关）](#LockOpen)
    - [人脸识别远程开锁](#LockOpenFace)
    - [ 获取锁的状态信息（需要网关）](#LockState)
    - [获取临时开锁密码](#LockPwdGet)
    - [分配用户开锁密码（写入方式，需要网关）](#V2LockPwdGet)
    - [回收用户开锁密码（需要网关）](#V2LockPwdDel)
    - [通过密码远程开锁（需要网关）](#V2LockOpen)
    - [消息推送](#PushMessage)

## 一、<a name="Introduction">简介</a>
本文档是可信物联网云平台第三方api接口文档


## 二、<a name="StatusCode">状态码</a>
```
    StatusContinue              = 100
    StatusSwitchingProtocols    = 101

    StatusOK                    = 200
    StatusCreated               = 201
    StatusAccepted              = 202
    StatusNonAuthoritativeInfo  = 203
    StatusNoContent             = 204 // 没有内容
    StatusResetContent          = 205
    StatusPartialContent        = 206

    StatusMultipleChoices       = 300
    StatusMovedPermanently      = 301
    StatusFound                 = 302
    StatusSeeOther              = 303
    StatusNotModified           = 304
    StatusUseProxy              = 305
    StatusTemporaryRedirect     = 307

    StatusUserNotExists         = 310 // 用户不存在
    StatusLoginFail             = 311 // 登录失败，账号或密码错误
    StatusPermissionDeniedFail  = 312 // 登录失败，没有权限
    StatusLogoutFail            = 315 // 登出失败

    StatusRegisterFail          = 320 // 注册失败
    StatusUserNameUsed          = 321 // 用户名已使用
    StatusEmailUsed             = 322 // 邮箱已使用
    StatusTelUsed               = 323 // 手机号已使用
    StatusUpdateTokenFail       = 324 // 更新token失败
    StatusUserUpdateFail        = 325 // 更新失败
    StatusPwdIllegal            = 326 // 密码不合法
    StatusFaceExceed            = 327 // 人脸识别太频繁
    StatusSmsFail               = 328 // 短信发送失败

    StatusLockNotExists         = 350 // 锁不存在
    StatusLockIdExists          = 351 // lock_id已经存在
    StatusLockNoAlarm           = 352 // 没有警报
    StatusLockUserAuthed        = 353 // 用户已授权
    StatusLockUnAuthorized      = 354 // 没有权限
    StatusLockNotBind           = 355 // 没有绑定
    StatusLockBinded            = 356 // 已经绑定
    StatusExpired               = 357 // 已经过期
    StatusFaceErr               = 358 // 匹配错误
    StatusOpenLockFail          = 359 // 开锁失败,稍后再试
    StatusLockNotOnline         = 360 // 锁不在线
    StatusLockMemberNotEmpty    = 361 // 锁成员非空
    StatusLockNotBoundAp        = 362 // 锁没有绑定ap
    StatusAddSelfForbidden      = 363 // 不允许添加自己
    StatusSetOrDelLockPwdFail   = 364 // 设置或删除开锁密码失败,稍后再试
    StatusDeviceBusyFail        = 365 // 设备繁忙,请稍后再试

    StatusBadRequest                   = 400 // 请求错误, 检查请求参数
    StatusUnauthorized                 = 401 // 请先注册或登录
    StatusPaymentRequired              = 402
    StatusForbidden                    = 403
    StatusNotFound                     = 404
    StatusMethodNotAllowed             = 405
    StatusNotAcceptable                = 406
    StatusProxyAuthRequired            = 407
    StatusRequestTimeout               = 408
    StatusConflict                     = 409
    StatusGone                         = 410
    StatusLengthRequired               = 411
    StatusPreconditionFailed           = 412
    StatusRequestEntityTooLarge        = 413
    StatusRequestURITooLong            = 414
    StatusUnsupportedMediaType         = 415
    StatusRequestedRangeNotSatisfiable = 416
    StatusExpectationFailed            = 417
    StatusTeapot                       = 418
    StatusPreconditionRequired         = 428
    StatusTooManyRequests              = 429
    StatusRequestHeaderFieldsTooLarge  = 431
    StatusUnavailableForLegalReasons   = 451

    StatusInternalServerError           = 500
    StatusNotImplemented                = 501
    StatusBadGateway                    = 502
    StatusServiceUnavailable            = 503
    StatusGatewayTimeout                = 504
    StatusHTTPVersionNotSupported       = 505
    StatusNetworkAuthenticationRequired = 511
    
    StatusUnknownError                  = 1000  //未知错误
```
## 三、<a name="Sign">签名验证</a>
#### 1.联系接口对接人获取app_id 和app_secret
#### 2.签名步骤：
- (1) 请求json参数按字母升序排列+app_secret+时间变量(用+连接, 且不要有空格)，生成待签名字符串。
- (2) 再对字符串使用MD5生成sign。
- (3) 将sign加入原json作为实际发送的请求数据。

#### 3.签名举例：
- app_id: "app_test"
- app_secret: a0cc195a3034c6de34c9dfccf765117b
```json
{  "lock_id":1,
   "app_id":"app_test"
}
```
- (1)待签名字符串为：{"app_id":"app_test","lock_id":1}+a0cc195a3034c6de34c9dfccf765117b+413118
（时间变量=当前unix时间戳/3600，比如当前时间戳为1487226152，那么时间变量是413118）
- (2) MD5生成sign为：a707c749d0e02a7d8e7a4d670e8632c3
- (3) 请求时,实际发送的json为：
```json
{  "lock_id":1,
   "app_id":"app_test",
   "sign":"a707c749d0e02a7d8e7a4d670e8632c3"
}
```


## 四、<a name="API">接口</a>

###  <a name="LockGet">通过lock_id获取锁详情</a>

- method: post
- url: /third/lock/get
- request

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
lock_id|unit32|是| |锁id|
sign|string|是| |签名|
app_id|string|是| |app_id|

```json
{
    "lock_id": 1000007,
    "sign":"sign",
    "app_id":"app_id"
}
```

- response data:

```json
{
    "code": 200,
    "desc": "OK",
    "data": {
        "lock_id": 1000007,
        "mac": "72:00:03:16:7d:f0",
        "master": {
            "name": "xiao ming",
            "tel": "13591951888"
        },
        "name": "",
        "descr": "",
        "corp_id": 0,
        "device_ver": "ver",
        "firmware_ver": "",
        "is_init": true,
        "is_bind": true,
        "init_at": "2016-09-26T15:23:35.570062558+08:00",
        "bind_at": "2016-09-08T15:20:03.222925+08:00",
        "updated_at": "2016-09-08T15:20:03.222925+08:00",
        "state": 0,
        "is_del": false
    }
}

```

###  <a name="LockByMacGet"> 通过mac获取锁详情</a>

- method: post
- url: /third/lock/by_mac/get
- request

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
mac|string|是| |锁mac|
sign|string|是| |sign|
app_id|string|是| |app_id|

```json
{
    "mac": "72:00:03:16:7d:f0",
    "sign":"sign",
    "app_id":"app_id"
}
```

- response data:

```json
{
    "code": 200,
    "desc": "OK",
    "data": {
        "lock_id": 1000001,
        "mac": "72:00:03:16:7d:f0",
        "master": {
            "name": "xiao ming",
            "tel": "13591951888"
        },
        "name": "",
        "descr": "",
        "corp_id": 0,
        "device_ver": "ver",
        "firmware_ver": "",
        "is_init": true,
        "is_bind": true,
        "init_at": "2016-09-26T15:23:35.570062558+08:00",
        "bind_at": "2016-09-08T15:20:03.222925+08:00",
        "updated_at": "2016-09-08T15:20:03.222925+08:00",
        "state": 0,
        "is_del": false
    }
}

```

###  <a name="LockOpen">网络远程开锁（需要网关）</a>

- method: post
- url: /third/lock/open
- request

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
lock_id|uint32|是| |锁id|
attach|string|是| |保留，这里填“test”|
sign|string|是| |sign|
app_id|string|是| |app_id|

```json
{
    "lock_id": 1000001,
    "attach":"test",
    "sign":"sign",
    "app_id":"app_id"
}
```

- response data:

```json
{
    "code": 200,
    "desc": "OK",
    "data": null
}

```

###  <a name="LockOpenFace">人脸识别远程开锁</a>

- method: post
- url: /third/lock/open/face
- request:
!!! 特殊处理 !!!: img_code不加入签名

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
app_id|string|是| |app_id|
lock_id|uint32|是| |锁id|
name|string|是| |用户真实姓名|
id_num|string|是| |用户真实身份证号|
img_code|string|是| |图片 base64 编码，图片大小限制在100k以内|
sign|string|是| |sign|

```json
{
    "app_id":"app_id",
    "lock_id": 1000001,
    "name":"name",
    "id_num":"id_num",
    "img_code":"img base64 code",
    "sign":"sign"
}

```

- response data:

```json
{
    "code": 200,
    "desc": "OK",
    "data": null
}

```

###  <a name="LockState"> 获取锁的状态信息（需要网关）</a>

- method: post
- url: /third/lock/state/get
- request:

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
lock_id|unit32|是| |锁id|
sign|string|是| |sign|
app_id|string|是| |app_id|

```json
{
    "lock_id": 1000007,
    "sign":"sign",
    "app_id":"app_id"
}
```


- response data:

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
lock_id|unit32|是| |锁id|
online|bool|是|true|是否在线|
is_abnormal|bool|是|false|是否异常|
low_voltage|bool|是|false|是否低电压|

```json
{
    "code": 200,
    "desc": "OK",
    "data":{
        "lock_id": 1000001,
        "online": false,
        "is_abnormal": false,
        "low_voltage": false
    }
}
```

###  <a name="LockPwdGet"> 获取临时开锁密码</a>

- method: post
- url: /third/lock/pwd/get
- request:

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
app_id|string|是| |app_id|
lock_id|uint32|是| |锁id|
effective_at|uint32|是| |开始生效时间，unix时间戳，必须是从当天开始的时间|
expire|uint32|是| |密码有效时长，单位秒,目前固定写300秒|
sign|string|是| |sign|

```json
{
    "app_id": "app_kexin",
    "effective_at": 1478585100,
    "lock_id": 1000003,
    "expire": 300,
    "sign": "a6b5986b23d42b4f09868e40632ef002"
}
```

- response data:
```json
{
    "code": 200,
    "desc": "ok",
    "data": {
       "password":"145332"
    }
}
```

###  <a name="V2LockPwdGet">分配用户开锁密码（写入方式，需要网关）</a>

- method: post
- url: /third/v2/lock/pwd/get
- request:

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
app_id|string|是| |app_id|
lock_id|uint32|是| |锁id|
uid|uint32|是| |用户ID,给此用户分配一个密码|
effective_at|uint32|是| |开始生效时间，unix时间戳，必须是从当天开始的时间|
expire|uint32|是| |密码有效时长，单位秒|
sign|string|是| |sign|

```json
{
    "app_id": "app_kexin",
    "effective_at": 1478585100,
    "lock_id": 1000003,
    "uid" : 100001,
    "expire": 300,
    "sign": "a6b5986b23d42b4f09868e40632ef002"
}
```

- response data:
```json
{
    "code": 200,
    "desc": "ok",
    "data": {
       "password":"145332"
    }
}
```


###  <a name="V2LockPwdDel"> 回收用户开锁密码（需要网关）</a>

- method: post
- url: /third/v2/lock/pwd/del
- request:

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
app_id|string|是| |app_id|
lock_id|uint32|是| |锁id|
uid|uint32|是| |用户ID,回收此用户密码，当uid=0时，删除所有用户的密码|
sign|string|是| |sign|

```json
{
    "app_id": "app_kexin",
    "lock_id": 1000003,
    "uid" : 100001,
    "sign": "a6b5986b23d42b4f09868e40632ef002"
}
```

- response data:
```
{
    "code": 200,
    "desc": "OK",
    "data": null
}
```

###  <a name="V2LockOpen">通过密码远程开锁（需要网关）</a>

- method: post
- url: /third/v2/lock/open
- request:

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
app_id|string|是| |app_id|
lock_id|uint32|是| |锁id|
password|string|是| |密码|
sign|string|是| |sign|

```json
{
    "app_id": "app_kexin",
    "lock_id": 1000003,
    "password" : "234541",
    "sign": "a6b5986b23d42b4f09868e40632ef002"
}
```

- response data:
```
{
    "code": 200,
    "desc": "OK",
    "data": null
}
```


###  <a name="PushMessage">消息推送</a>

- method: post
- url: （由第三方提供）
- request:

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
type|int|是| |消息类型|
data|object|是| |消息内容，根据type而定|


- type = 1，网关上线或下线

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
ap_id|int|是| |网关Id|
online|bool|是| |true：上线 ， false：掉线|

```json
{
    "type": 1,
    "data": {
        "ap_id": 1000002,
        "online": true
    }
}
```

- type = 2，开锁日志

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
lock_id|int|是| |锁Id|
result|int|是| |1 ：开锁成功 ， 2：开锁失败|
uid|int|是| |用户ID|
mode|int|是| |开锁方式：1按键开锁，2，蓝牙开锁 3，网络开锁|
time|string|是| |开锁时间|
```json
{
    "type": 2,
    "data": {
        "lock_id": 1000001,
        "result": 1,
        "uid": 100000,
        "mode": 1,
        "time": "2017-03-2910: 34: 34"
    }
}
```

- type = 3，低电压警报

|字段名|类型|必填|示例|描述|
---|---|---|---|---|
lock_id|int|是| |锁Id|
remain|float|是| |剩余电量百分比|

```json
{
    "type": 3,
    "data": {
        "lock_id": 1000001,
        "remain": 30
    }
}
```

