# 1. 开发环境配置

## 1.1. 环境配置及发布

1. 导入统一认证framework

直接将统一认证`TYRZSDK.framework`拖到项目中

![iOS-1](image\支付宝-ios-01.png)

2. 在Xcode中找到`TARGETS-->Build Setting-->Linking-->Other Linker Flags`在这选项中需要添加`-ObjC`

![iOS-4](image\iOS-4.jpg)

3. TARGETS-->Build Setting-->搜索框中搜索"BitCode"选项,并且将该选项的属性设置为NO

![iOS-5](D:\$$ 统一认证\文档\1. 融合SDK开发文档\image\iOS-5.jpg)

## 1.2. Hello 统一认证

本节内容主要面向新接入统一认证的开发者，介绍快速集成统一认证的基本服务的方法。

### 1.2.1. 统一认证登录集成步骤

**第一步：**

在需要用到登录的地方调用登录接口即可，以下是登录示例

```objective-c
- (void)showImplicitLogin {
    self.waitBGV.hidden = NO;
    [self.waitAV startAnimating];
     __weak typeof(self) weakSelf = self;
    [TYRZLogin loginImplicitly:^(id sender) {
        dispatch_async(dispatch_get_main_queue(), ^{
            weakSelf.waitBGV.hidden = YES;
            [weakSelf.waitAV stopAnimating];
            NSString *resultCode = sender[@"resultCode"];
            self.token = sender[@"token"];
            NSMutableDictionary *result = [NSMutableDictionary dictionaryWithDictionary:sender];
            if ([resultCode isEqualToString:SUCCESSCODE]) {
                result[@"result"] = @"获取token成功";
            } else {
                result[@"result"] = @"获取token失败";
            }
            [self showInfo:result];
        });
    }];
}
```

<div STYLE="page-break-after: always;"></div>

# 2.SDK方法描述

## 2.1. 获取token

### 2.1.1. 方法说明

**功能**

开发者通过隐式登录方法，无授权弹窗，可获取到token和openID，应用服务端凭token向SDK服务端请求校验是否本机号码。

**原型**

TYRZLogin -> loginImplicitly

```objective-c
+ (void)loginImplicitlyWithAppId:(NSString *)appid
                          appkey:(NSString *)appkey
                          capaId:(NSString *)capaid
                      capaIdTime:(NSString *)capaidTime
                           scene:(NSString *)scene
                        complete:(void (^)(id sender))complete
```

### 2.1.2. 参数说明

**请求参数**

| 参数         | 类型            | 说明              | 是否必填 |
| ---------- | ------------- | --------------- | ---- |
| appid      | NSString      | 开放平台申请得到的appid  | 是    |
| appkey     | NSString      | 开放平台申请得到的appkey | 是    |
| capaId     | NSString.     | 授权列表            | 是    |
| capaidTime | NSString.     | 当前时间            | 否    |
| complete   | UAFinishBlock | 登录回调            | 是    |
| scene      | NSString      | 扩展参数，默认为空       | 否    |


**响应参数**

| 参数          | 类型         | 说明                                       | 是否必填  |
| ----------- | ---------- | ---------------------------------------- | ----- |
| resultCode  | NSUinteger | 返回相应的结果码                                 | 是     |
| token       | NSString   | 登录时需要的token                              | 成功时必填 |
| openid      | NSString   | 成功时返回：用户身份唯一标识                           | 成功时必填 |
| authType    | NSString   | 认证类型：0:其他；</br>1:WiFi下网关鉴权；</br>2:网关鉴权；</br>3:短信上行鉴权；</br>7:短信验证码登录 | 成功时必填 |
| authTypeDes | NSString   | 认证类型描述，详见authType描述                      | 成功时必填 |
| desc        | NSString   | 调用描述                                     | 否     |

### 2.1.3. 示例

**请求示例代码**

``` java
/**
 隐式登录
 */
- (IBAction)loginimplicit:(id)sender {
    
    [TYRZLogin loginImplicitlyWithAppId:APPID
                                 appkey:APPKEY
                                 capaId:@"200"
                               capaIdTime:@""
                                   scene:@"scene"
                               complete:^(id sender) {
                                   NSString *resultCode = sender[@"resultCode"];
                                   self.token = sender[@"token"];
                                   NSMutableDictionary *result = [NSMutableDictionary dictionaryWithDictionary:sender];
                                   if ([resultCode isEqualToString:CLIENTSUCCESSCODECLIENT]) {
                                       result[@"result"] = @"获取token成功";
                                   } else {
                                       result[@"result"] = @"获取token失败";
                                   }
                                   [self showInfo:result];
                               }];
}
```

**响应示例代码**

```
{
    authType = 2;
    resultCode = 103000;
    openId = "lcIvEt9u1RRBwr8richVcbEUzzrm6IaFXNPWPejB2b3O_vVIkAwI";
    authToeDes = @"网关鉴权"
    result = @"获取token成功"
    token = 84840100013202003A516B55354D6B4A434D4467304D5456434F4441334D30553040687474703A2F2F3132302E3139372E3233352E32373A383038302F72732F403031030004031A1A9B040012383030313230313730383137313031343230FF00200970BFEA09AEF18EEDCF32A4F960412E9AA5DE9A21DF7DC669E4D27E3519A1A4
}
```
## 2.2. 获取SDK内部异常的信息

### 2.2.1. 方法说明

**功能**

监听SDK内部异常的信息

**原型**

TYRZLogin -> TYRZLoginDelegate

```objective-c

-(void)TYRZDeBugWithMessage:(NSDictionary *)message

```
### 2.2.2. 使用说明

```
TYRZLogin.debugDelegate = self;
- (void)TYRZDeBugWithMessage:(NSDictionary *)message {
    
    NSLog(@"debug message: %@", message);
    
}
```
### 2.2.3. 返回字典结构

```
@{@"message":message}
```
**参数说明**

| 参数      | 类型                |
| ------- | ----------------- |
| message | NSString或者NSError |


### 2.3.1. 取消隐式登录操作

### 2.3.2. 方法说明

**功能**

取消隐式登录操作

**原型**

TYRZLogin -> cancel

```objective-c

+ (void)cancel

```
### 2.3.3. 使用说明

```
[TYRZLogin cancel];

```

<div STYLE="page-break-after: always;"></div>

# 3. 平台接口说明

## 3.1. 获取用户信息接口

### 3.1.1. 接口说明

客户端SDK携带用户授权成功后的token来调用SDK服务端的相应访问用户资源的接口。

### 3.1.2. 接口描述

**请求地址：**https://www.cmpassport.com/unisdk/rsapi/tokenValidate

**协议：**HTTPS

**请求方法：**POST+JSON

### 3.1.3.参数说明

**请求参数**

| 参数名称          | 约束      | 层级   | 参数类型   | 说明                                       |
| ------------- | ------- | ---- | ------ | ---------------------------------------- |
| header        | 必选      | 1    |        |                                          |
| version       | 必选      | 2    | string | 填1.0                                     |
| msgid         | 必选      | 2    | string | 标识请求的随机数即可(1-36位)                        |
| systemtime    | 必选      | 2    | string | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| strictcheck   | 必选      | 2    | string | 暂时填写"0"                                  |
| sourceid      | 可选      | 2    | string | 业务集成统一认证的标识                              |
| ssotosourceid | 可选      | 2    | string | 单点登录时使用，填写被登录业务的sourceid                 |
| appid         | 必选      | 2    | string | 业务在统一认证申请的应用                             |
| apptype       | 必选      | 2    | string | 1:BOSS</br>2:web</br>3:wap</br>4:pc客户端</br>5:手机客户端 |
| expandparams  | 扩展参数    | 2    | Map    | map(key,value)                           |
| sign          | 当有密钥时必填 | 2    | String | 业务端RSA私钥签名（appid+token）, 服务端使用开发者提供的公钥进行RSA公钥解密开发者 |
| body          | 必选      | 1    |        |                                          |
| token         | 必选      | 2    | string | 需要解析的凭证值。                                |

**请求示例**

```
{
    "header": {
        "strictcheck": "0",
        "version": "1.0",
        "msgid": "4d1953f426a14b2ba9edd07b269cea70",
        "systemtime": "20171109145619809",
        "appid": "300005687666",
        "apptype": "5",
        "sourceid": "800120170818101447",
        "sign": "751E95E65384F2563E9285E94248AF63E972B2326F648CB947D8308543BCF4902D82A9CA7174880A388D6649BD66B2848A4B814143FA61584506CD7E9FB6B7A8AB5AF40F944CC930703ADC3356DE13B93FB429CFC393D8B73FDC02711B0B60848A735D1F90810FBEB8475FEEFBE6D8E774ABE9CB77E857A1CF49AFBA097B0413"
    },
    "body": {
        "token": "STsid0000001510210576710ivYn20UXdDa8Nhw0LpL1szQi5KJhgf5r"
    }
}
```

**响应参数**

| 参数名称                | 约束   | 层级   | 参数类型   | 说明                                       |
| ------------------- | ---- | ---- | ------ | ---------------------------------------- |
| header              | 必选   | 1    |        |                                          |
| version             | 必选   | 2    | string | 1.0                                      |
| inresponseto        | 必选   | 2    | string | 对应的请求消息中的msgid                           |
| systemtime          | 必选   | 2    | string | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| resultCode          | 必选   | 2    | string | 返回码，返回码请参考SDK返回码说明                       |
| body                | 必选   | 1    |        |                                          |
| userid              | 必选   | 2    | string | 系统中用户的唯一标识                               |
| pcid                | 必选   | 2    | string | 伪码id（显示和隐试登录都有）                          |
| usessionid          | 可选   | 2    | string | 暂忽略                                      |
| passid              | 可选   | 2    | string | 用户统一账号的系统标识                              |
| andid               | 可选   | 2    | string | 用户的“和ID”                                 |
| msisdn              | 可选   | 2    | string | 表示手机号码(当appid有开发者提供的密钥时，手机号用RSA公钥加密返回。使用私钥可以) |
| email               | 可选   | 2    | string | 表示邮箱地址                                   |
| loginidtype         | 可选   | 2    | string | 登录使用的用户标识：</br>0：手机号码</br>1：邮箱           |
| msisdntype          | 可选   | 2    | string | 手机号码的归属运营商：</br>0：中国移动</br>1：中国电信</br>2：中国联通</br>99：未知的异网手机号码 |
| province            | 可选   | 2    | string | 用户所属省份(暂无)                               |
| authtype            | 可选   | 2    | string | 认证类型：</br>0:其他；</br>1:WiFi下网关鉴权；</br>2:网关鉴权；</br>3:短信上行鉴权；</br>7:短信验证码登录 |
| authtime            | 可选   | 2    | string | 统一认证平台认证用户的时间                            |
| lastactivetime      | 可选   | 2    | string | 暂无                                       |
| relateToAndPassport | 可选   | 2    | string | 是否已经关联到统一账号，暂无用处                         |
| fromsourceid        | 可选   | 2    | string | 来源sourceid（即签发token sourceid）            |
| tosourceid          | 可选   | 2    | string | 目的sourceid（即被登录业务sourceid）               |

**响应示例**

```
{
    "body": {
        "msisdntype": "0",
        "lastactivetime": "2017-11-09 14:56:16",
        "authtype": "WAPGW",
        "fromsourceid": "800120170714100001",
        "loginidtype": "0",
        "authtime": "2017-11-09 14:56:16",
        "msisdn": "48DB711B1BA151E20DBD2F726404E44F352561A30A58C74368A8A8C64851AA9E652AA55E8924FA6BACA660DFBE0F3E9EEA5FF7369CED03817445397DA0E0D590E0A6871655464C0F336192F00147718FD4C05511E3C0F400F65DCC18FD80D368F3F429C67FD4D6B6F38673AF8D3388891099847203897CD4025D5E54BFC3B570"
    },
    "header": {
        "inresponseto": "4d1953f426a14b2ba9edd07b269cea70",
        "resultcode": "103000",
        "systemtime": "20171109145620587",
        "version": "1.0"
    }
}
```



<div STYLE="page-break-after: always;"></div>

# 4. 平台返回码说明

## 4.1. 平台返回码

| 错误编号   | 返回码描述                |
| ------ | -------------------- |
| 103101 | 签名错误                 |
| 103103 | 用户不存在                |
| 103104 | 用户不支持该种登录方式          |
| 103105 | 密码错误                 |
| 103106 | 用户名错误                |
| 103107 | 已存在相同的随机数            |
| 103108 | 短信验证码错误              |
| 103109 | 短信验证码超时              |
| 103111 | WAP网关IP不合法           |
| 103112 | 请求错误 reqError        |
| 103113 | Token内容错误            |
| 103114 | token验证 KS过期         |
| 103115 | token验证 KS不存在        |
| 103116 | token验证 sqn错误        |
| 103117 | mac异常 macError       |
| 103118 | sourceid不存在          |
| 103119 | appid不存在appidNOExist |
| 103120 | clientauth不存在        |
| 103121 | passid不存在            |
| 103122 | btid不存在              |
| 103123 | redisinfo不存在         |
| 103124 | ksnaf校验不一致           |
| 103125 | 手机格式错误               |
| 103126 | 手机号不存在               |
| 103127 | 证书验证，版本过期            |
| 103128 | gba webservice接口调用失败 |
| 103129 | 获取短信验证码的msgtype异常    |
| 103130 | 新密码不能与当前密码相同         |
| 103131 | 密码过于简单               |
| 103132 | 用户注册失败               |
| 103133 | sourceid不合法          |
| 103134 | wap方式手机号为空           |
| 103135 | 昵称非法                 |
| 103136 | 邮箱非法                 |
| 103138 | appid已存在             |
| 103139 | sourceid已存在          |
| 103200 | 不需要更新ks              |
| 103204 | 缓存随机数不存在             |
| 103205 | 服务器内部异常              |
| 103207 | 发送短信失败               |
| 103212 | 校验密码失败               |
| 103213 | 旧密码错误                |
| 103214 | 访问缓存或数据库错误           |
| 103226 | sqn过小或过大             |
| 103265 | 用户已存在                |
| 103901 | 短信验证码下发次数已达上限        |
| 103902 | 凭证校验失败               |
| 104001 | APPID和APPKEY已存在      |
| 105001 | 联通网关取号失败             |
| 105002 | 移动网关取号失败             |
| 105003 | 电信网关取号失败             |
| 105004 | 短信上行ip检测不合法          |
| 105005 | 短信上行发送信息为空           |
| 105006 | 手机号码为空               |
| 105007 | 手机号码格式错误             |
| 105008 | 短信内容为空               |
| 105009 | 解析失败                 |

</br>

## 4.2. SDK返回码说明

| 错误编号   | 返回码描述                                  |
| ------ | -------------------------------------- |
| 103000 | 成功                                     |
| 102101 | 无网络                                    |
| 102102 | 网络异常                                   |
| 102103 | 未打开数据网络                                |
| 102109 | 网络错误，1. 欠费卡；2. 网络环境不佳                  |
| 102203 | 输入参数缺失（缺少appid、appkey、capaId其中任何一个时返回） |
| 102208 | 参数错误                                   |
| 102210 | 不支持短信发送                                |
| 102301 | 用户取消登录                                 |
| 102302 | 没有执行参数初始化                              |
| 102507 | 请求超时                                   |
| 200002 | 没有sim卡                                 |
| 200009 | Bundle ID与服务器填写的不一致                    |

<div STYLE="page-break-after: always;"></div>

# 附录1 认证方法标识

| authType | authTypeDes        |
| -------- | ------------------ |
| 0        | 其他（超时无法确认认证类型，返回0） |
| 1        | WIFI下网关鉴权          |
| 2        | 网关鉴权               |
| 3        | 短信上行鉴权             |
| 4        | WIFI下网关鉴权复用中间件登录   |
| 5        | 网关鉴权复用中间件登录        |
| 6        | 短信上行鉴权复用中间件登录      |
| 7        | 短信验证码登录            |
