# 1、接入流程

> 1、商务洽谈开户，沟通相关费率。
>
> 2、联系运营创建商户号、密钥、商户appId、产品编码、apiUrl。
> 
> 3、当开发完成，双方进行联调测试，验证请求、上报等信息完整。

# 2、Md5 签名算法

> 1、所有参数按键对数组(key1=value1)进行升序排序(参数值为空时不参与签名)
> 
> 2、按照 key1=value1&key2=value2 进行组合
> 
> 3、加上商户秘钥 key1=value1&key2=value2...&key=商户秘钥
> 
> 3、sign=md5(上一步组装待签名字的符串) 签名结果 32 位小写
> 
> 4、签名密钥,请在商户后台->基础信息 查看,或者询问我方客服

# 3、注意事项

## 3.1 接口相关
> 1、本文档中的所有接口，均采用 HTTP 标准通信协议，POST提交，请求和响应的 Content-type 均为 application/json，字符编码统一为 UTF-8。
> 
> 2、金额单位为<span style="color:red;"> 萨当（Satang） </span>。
> 
> 3、请求接口的 IP 需要加白。
> 
> 4、user_ip 尽量收集用户真实ip，确实没有就留空，不要使用127.0.0.1这种本地ip

## 3.2 回调相关
> 1、回调接收处理成功，请返回文本<span style="color:red;">success</span>不能含有其他任何字符，系统将不再推送此订单信息，否则还会重复推送多次
> 
> 2、在进行异步通知交互时，如果收到的应答不是 success ，会认为通知失败，会通过一定的策略定期重新发起通知。通知的间隔频率为：1m、1m、4m、10m、10m、1h、2h、6h、15h。
> 
> 3、pay_notice_url通知地址如果为空，会视为商家不需要回调，系统不会推送通知。

# 4、代收下单接口

(下单 ip 需要联系我方加白)
下单地址 https://{api_domain}/api/v1/payApi/CreatePayInOrder

## 4.1 代收-下单请求参数

| 名称             | 类型   | 必填  | 描述                                                                                              |
|----------------|------|-----|-------------------------------------------------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                                                             |
| app_id         | int    | true  | 商户 appId                                                                                        |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                                                   |
| pay_method     | string    | true  | 支付方式                                                                                            |
| price          | int    | true  | 下单金额，单位：萨当（Satang），整型。1 泰铢（THB）= 100 萨当                                                         |
| order_no       | string | true  | 商户订单号                                                                                           |
| success_url    | string | false | 支付成功跳转 url                                                                                      |
| fail_url       | string | false | 支付失败跳转 url                                                                                      |
| pay_notice_url | string | false | 支付成功通知 url                                                                                      |
| user_id        | string | false | 商家用户ID                                                                                          |
| user_ip        | string | false | 付款人 IP                                                                                          |
|attach|string|true| 附加参数 json字符串 付款人信息 |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                                                  |
|timestamp|string|false| 下单时间戳 10位时间戳单位S                                                                                 |

-  代收-attach 附加参数字段说明

| 名称            | 类型     | 必填    | 描述                                                                      |
|---------------|--------|-------|-------------------------------------------------------------------------|
| bank_code         | string    | true | 代收银行代码 |
| name         | string    | true | 付款人姓名 |
| account_no  | string    | true | 付款人银行账号 |


- 代收-下单请求示例

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"bank_code\":\"THB_ICBC\",\"name\":\"Somchai Som\",\"account_no\":\"XXXXXXXXXXXXXXXXX\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "203.144.160.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}
```

## 4.2  代收-下单响应

| 名称         | 类型   | 必填 | 描述                                                                                                                                                                                              |
|------------|------|----|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code         | int    | true | 200:下单成功 其他:下单失败                                                                                                                                                                                |
| msg          | string | true | 失败原因                                                                                                                                                                                            |
| pay_url      | string | false | 付款链接                                                                                                                                                                                            |
| qr_code      | string | false | 二维码字符串                                                                                                                                                                                      |
| order_no     | string | true | 商户订单号                                                                                                                                                                                           |
| dis_order_no | string | true | 平台订单号                                                                                                                                                                                           |
| create_time  | int    | true | 创建时间                                                                                                                                                                                            |
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 {"pay_raw":"支付原生信息","redirect_url":"支付跳转链接"} |
| sign         | string | true | 签名结果,签名方法在文档顶部                                                                                                                                                                                  |

-  代收-下单响应示例

失败:

```json
{
  "code": 1005,
  "msg": "Merchant not found",
  "sign": ""
}
```

成功:

```json
{
  "code": 200,
  "msg": "",
  "sign": "b449b4b6907204a683ec6c50bff92b01",
  "order_no": "p7158412025J2dZjXLmz0",
  "dis_order_no": "2025071130770572062498816tha1oushe",
  "create_time": 1752825512,
  "pay_url": "https://{api_domain}/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5、代收回调通知   post/json

推送地址:商户下单传送的 pay_notice_url 回调 ip:call_back_server_ip ,请将我方 ip 加入回调白名单

## 5.1 代收回调-请求参数

| 名称         | 类型   | 必填  | 描述                                                                                                     |
|------------|------|-----|--------------------------------------------------------------------------------------------------------|
| trade_no     | int    | true  | 商户号                                                                                                    |
| status       | int    | true  | 订单状态, <span style="color:red;">2.成功</span>, 3.失败                                                                                       |
| order_no     | string | true  | 商户订单号                                                                                                  |
| dis_order_no | string | true  | 平台订单号                                                                                                  |
| order_price  | int    | true  | 订单金额,单位:萨当（Satang）                                                                                              |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">用户真实付款金额 ,单位:萨当（Satang）</span>                                                                                         |
| nti_time     | int    | false | 发起通知时间                                                                                                 |
| payer        | string | false | JSON 字符串,付款人信息{"name":"姓名","account":"账号","bank":"付款的用户银行编码","utr2":"银行流水号","email":"邮箱","phone":"手机号","identify_type":"证件类型","identify_num":"证件号码"}，除示例字段外，当前参数会整合商户传递的attach里付款人信息相关字段|
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 |
| create_time  | int    | true  | 创建时间                                                                                                   |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                                                                         |

-  代收回调-请求参数示例

```json
{
  "trade_no": 10238,
  "status": 2,
  "order_no": "RCG0126042435820040804784",
  "dis_order_no": "Meg1352644s0800thanVMR",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1776680622,
  "payer": "{\"account_no\":\"1234567890\",\"account_type\":\"BANK\",\"identify_num\":\"\",\"identify_type\":\"\",\"req_api_ip\":\"\",\"utr2\":\"9901108391\"}",
  "create_time": 1776680593,
  "sign": "7b23565a3dc790b6e55f29f0f0cf5f1a"
}
```
## 5.2 代收回调-响应说明
回调接收处理成功，请返回 <span style="color:red;">success</span>，系统将不再推送此订单信息，否则还会重复推送多次

# 6、代付下单接口

(下单 ip 需要联系我方加白)
下单地址 https://{api_domain}/api/v1/payApi/CreatePayOutOrder

## 6.1 代付-请求参数

| 名称             | 类型     | 必填    | 描述                                                                     |
|----------------|--------|-------|------------------------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                                    |
| order_no       | string | true  | 商户订单号                                                                  |
| app_id         | int    | true  | 商户 appId                                                               |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                          |
| price          | int    | true  | 下单金额，单位：萨当（Satang），整型。1 泰铢（THB）= 100 萨当                                 |
| account_no     | string | true  | 收款账号                                                                   |
| account_type   | string | false  | 账号类型                                   |
| account_name   | string | true  | 姓名                                                                     |
| bank_code      | string | true  | 银行编码                                                                   |
| identify_type  | string | false  | 证件类型  |
| identify_num   | string | false  | 证件号码                                                                   |
| pay_notice_url | string | false | 代付成功通知 url                                                             |
| attach         | string | false  | 附加参数 json字符串                                                           |
| user_ip        | string | false | 收款用户 IP                                                                |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                         |
| timestamp      | string | false | 下单时间戳 10位时间戳单位S秒                                                       |

-  代付-attach 附加参数字段说明

| 名称       | 类型     | 必填   | 描述             |
|----------|--------|------|----------------|
| phone      |string| false | 收款人手机号          |
| email| string| false | 收款人邮箱 |



- 代付-请求参数示例

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"phone\":\"0812345678\",\"email\":\"somchai@example.com\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "203.144.160.100",
  "account_type": "BANK",
  "account_no": "1234567890",
  "account_name": "Somchai Jaidee",
  "bank_code": "THB_BANK"
}
```

## 6.2 代付-下单响应

| 名称         | 类型   | 必填 | 描述                                          |
|------------|------|----|---------------------------------------------|
| code         | int    | true | 200:下单成功 其他:下单失败                            |
| msg          | string | true | 失败原因                                        |
| dis_order_no | string | true | 平台订单号                                       |
| order_no     | string | true | 商户订单号                                       |
| status       | int    | true | 订单状态 2.代付成功, 3.代付失败, 7.驳回 9.冲正 10:处理中 |
| create_time  | int    | true | 创建时间                                        |
| sign         | string | true | 签名结果,签名方法在文档顶部                              |

- 代付-下单响应示例

失败:

```json
{
  "code": 1005,
  "msg": "Merchant not found",
  "sign": ""
}
```

成功:

```json
{
  "code": 200,
  "msg": "",
  "sign": "d3ec1fa0f45bc44218d5fb63bb1beb61",
  "order_no": "p7158412025MsJydJqT7b",
  "dis_order_no": "2025071130776296733810688tha1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7、代付回调通知

推送地址:商户下单传送的 pay_notice_url 回调 ip:call_back_server_ip ,请将我方 ip 加入回调白名单

## 7.1 代付回调请求参数

| 名称         | 类型   | 必填  | 描述                                            |
|------------|------|-----|-----------------------------------------------|
| trade_no     | int    | true  | 商户号                                           |
| order_no     | string | true  | 商户订单号                                         |
| dis_order_no | string | true  | 平台订单号                                         |
| order_price        | int    | true  | 订单金额,单位:萨当（Satang）                                     |
| fee          | int    | false | 订单手续费 ,单位:萨当（Satang）                         |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">真实代付出款金额 (代付成功时才有) 如需使用real_price字段，请与工作人员联系配置</span>                                                                                       |
| status       | int    | true  | 订单状态, <span style="color:red;">2.代付成功</span>, 3.代付失败, 7.驳回 9.冲正               |
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行、utr2等 |
| remark       | string | false | 失败原因                                          |
| create_time  | int    | true  | 创建时间                                          |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                |
| nti_time     | int    | true | 发起通知时间                                        |

- 代付回调请求示例

```json
{
  "trade_no": 10000,
  "status": 2,
  "order_no": "20060354339090013",
  "dis_order_no": "Meg2352644o2nmjo0800thaYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": ""
}
```
## 7.2 代付回调响应说明
回调接收处理成功，请返回 <span style="color:red;">success</span>，系统将不再推送此订单信息，否则还会重复推送多次

# 8、查询订单接口 (代收 代付共用)

(请求 ip 需要联系我方加白)
查询地址: https://{api_domain}/api/v1/payApi/QueryOrder

## 8.1 查询请求参数

| 名称         | 类型   | 必填 | 描述                        |
|------------|------|----|---------------------------|
| order_type | string | true | pay_out:代付,pay_in:代收    |
| trade_no   | int    | true | 商户号                      |
| app_id         | int    | true  | 商户 appId                    |
| dis_order_no | string | false | 平台订单号                |
| order_no | string | false  | 商户订单号                |
| sign       | string | true | 签名结果,签名方法在文档顶部 |

- 查询请求示例

```json
{
  "order_type": "pay_in",
  "trade_no": 165,
  "app_id": 165,
  "dis_order_no": "p7158277185f96603047656571",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 8.2 查询响应

| 名称         | 类型   | 必填  | 描述                                                                                                                                                |
|------------|------|-----|---------------------------------------------------------------------------------------------------------------------------------------------------|
| code         | int    | true  | 200:查询成功 其他:失败                                                                                                                                    |
| msg          | string | true  | 查询失败原因                                                                                                                                            |
| trade_no     | int    | true  | 商户号                                                                                                                                               |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">真实付款金额 ,单位:萨当（Satang）</span>                                                                                                                              |
| status       | int    | true  | 订单状态, 1.未支付, <span style="color:red;">2.成功</span>, 3.失败 , 7.驳回 9.冲正  10:处理中                                                                                                       |
| success_time | int    | true  | 成功时间戳                                                                                                                                             |
| order_no     | string | true  | 商户订单号                                                                                                                                             |
| dis_order_no | string | true  | 平台订单号                                                                                                                                             |
| remark       | string | true  | 代付失败原因                                                                                                                                            |
| fee          | int    | false | 订单手续费 ,单位:萨当（Satang）                                                                                                                               |
| create_time  | int    | true  | 创建时间                                                                                                                                              |
| payer        | string | false | JSON 字符串,付款人信息{"account_name":"姓名","account_type":"账号类型","account_no":"账号","identify_type":"证件类型","identify_num":"证件号码"} |
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                                                                                                                    |
|utr2|string|false|银行订单号|

- 查询响应示例

失败:

```json
{
  "code": 1017,
  "msg": "订单不存在"
}
```

成功:

```json
{
  "code": 200,
  "msg": "success",
  "trade_no": 123,
  "real_price": 10000,
  "status": 2,
  "success_time": 1693057443,
  "order_no": "47210116924681604173",
  "dis_order_no": "lufei169246816001692",
  "remark": "",
  "fee": 10,
  "create_time": 1695317066,
  "payer": "{\"name\":\"Somchai Jaidee\",\"email\":\"somchai@example.com\",\"phone\":\"0812345678\",\"identify_type\":\"ID\",\"identify_num\":\"1234567890123\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9、代付余额查询接口

(请求 ip 需要联系我方加白)
地址: https://{api_domain}/api/v1/payApi/QueryBalance

## 9.1 余额请求参数

| 名称     | 类型   | 必填 | 描述             |
|--------|------|----|----------------|
| trade_no | int    | true | 商户号            |
| app_id   | int    | true | 商户 appid       |
| sign     | string | true | 签名结果,签名方法在文档顶部 |

-  余额请求示例

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 余额响应

| 名称    | 类型   | 必填 | 描述                        |
|-------|------|----|---------------------------|
| code    | int    | true | 200:查询成功 其他:失败      |
| msg     | string | true | 失败原因                    |
| balance | int    | true | 余额,单位:萨当（Satang）       |
| balance_frozen | int    | false | 冻结余额,单位:萨当（Satang） |
| sign    | string | true | 签名结果,签名方法在文档顶部 |

-  余额响应示例

失败:

```json
{
  "code": 10001,
  "msg": "商户不存在"
}
```

成功:

```json
{
  "code": 200,
  "msg": "success",
  "balance": 10000,
  "balance_frozen": 1000,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
# 10、付款凭证查询接口

(请求 ip 需要联系我方加白)
地址: https://{api_domain}/api/v1/payApi/QueryCertificate

## 10.1 付款凭证请求参数

| 名称     | 类型   | 必填 | 描述             |
|--------|------|----|----------------|
| trade_no | int    | true | 商户号            |
| app_id   | int    | true | 商户 appid       |
| order_no   | string    | false | 商户订单号  与dis_order_no二选一    |
| dis_order_no   | string    | false | 平台订单号  与order_no二选一    |
| sign     | string | true | 签名结果,签名方法在文档顶部 |

- 付款凭证请求示例

```json
{
  "trade_no": 10003,
  "app_id": 10003,
  "order_no": "",
  "dis_order_no": "35011C02gljuf6k0800tha1lVY",
  "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```

## 10.2 付款凭证响应参数

| 名称    | 类型   | 必填 | 描述                        |
|-------|------|----|---------------------------|
| code    | int    | true | 200:响应成功 其他:失败      |
| msg     | string | true | 失败原因                    |
| img_link | string    | false |凭证链接       |
| img_base | string    | false |凭证生成的64码                 |
| sign    | string | true | 签名结果,签名方法在文档顶部 |

-  付款凭证响应示例
   没有付款凭证
```json
{
  "code":200,
  "msg":"No payment voucher available at the moment",
  "sign": "",
  "img_link": "",
  "img_base": ""
}
```
有付款凭证
```json
{
  "code": 200,
  "msg": "",
  "sign": "3969f17cd1a551769f85967d0a05b7b6",
  "img_link": "http://dsggfgdsf.djdj?ddd=snn",
  "img_base": "data:image/png;base64,hfhshdhfhfh"
}
```

# 11、错误码

| 状态码  | 描述                |
|------|-------------------|
| 200  | 成功                |
| 1000 | 内部错误              |
| 1001 | ip不在商户ip白名单       |
| 1002 | 参数错误              |
| 1003 | 签名错误              |
| 1004 | 当前商户暂未开放该接口  （联系运营核实:商户或App(不存在|已关闭|支付产品未配置)）   |
| 1005 | 商户不存在             |
| 1006 | 当前用户ip在黑名单中       |
| 1007 | 当前用户在黑名单中         |
| 1008 | 商户App不存在          |
| 1009 | 支付产品不存在           |
| 1010 | 支付渠道不存在           |
| 1011 | 支付通道暂未开发完成，暂时不开放  |
| 1012 | 支付通道异常，请稍后再试      |
| 1013 | 当前单量过大，请稍后再试      |
| 1014 | 订单号重复             |
| 1015 | app余额不足           |
| 1016 | 同一用户频繁下单，请稍后再试    |
| 1017 | 订单记录不存在           |
| 1018 | 当前金额不支持           |
| 1019 | 当前app所在国家暂未开通支付下单 |
| 1020 | 当前app所在国家暂未开通代付下单 |
| 1021 | 失败                |
|1036|接口暂未开放|
|1037|不支持该货币|
|1038|代收回传utr异常|
| 9999 | 其他错误              |
|3000|系统升级维护中，暂停下单，请稍后尝试|

# 12、代收收银台接口

地址: https://{api_domain}/api/v1/cashApi/CashIn.html
请求方式: GET

### 参数:

| 名称           | 类型     | 必填    | 描述                        |
|--------------|--------|-------|---------------------------|
| app_id       | string | true  | 商户app_id                    |
| order_no       | string | true  | 商户订单号                     |
| amount       | string | true  | 商户金额单位泰铢（THB）                     |
| notice_url       | string | false | 异步通知地址                     |
| pay_code       | int    | true | 产品编码                     |

#### 示例

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{商户订单号}}&amount={{商户金额}}&notice_url={{异步通知地址}}&pay_code={{产品编码}}
```

# 13、代收支付方式 代收字段 pay_method

| 字段         | 值               | 描述   |
|------------|------------------|------|
| pay_method | THB          | 泰国支付 |


# 14、代收银行编码 代收字段 attach.bank_code

| 字段名称 | 编码 | 银行名称 |
|:---------|:-----|:---------|
| attach.bank_code | THB_BBL | Bangkok Bank Public Company Limited |
| attach.bank_code | THB_KBANK | Kasikornbank Public Company Limited |
| attach.bank_code | THB_KTB | Krung Thai Bank Public Company Limited |
| attach.bank_code | THB_JPMC | JPMorgan Chase Bank, Bangkok Branch |
| attach.bank_code | THB_OCBC | Oversea-Chinese Banking Corporation Ltd. |
| attach.bank_code | THB_BTMU | The Bank of Tokyo-Mitsubishi UFJ, Ltd |
| attach.bank_code | THB_TTB | TMB Bank Public Company Limited |
| attach.bank_code | THB_SCB | Siam Commercial Bank Public Company Limited |
| attach.bank_code | THB_CITI | Citibank N.A. |
| attach.bank_code | THB_SMBC | Sumitomo Mitsui Banking Corporation |
| attach.bank_code | THB_SCBT | Standard Chartered Bank (Thai) Public Company Limited |
| attach.bank_code | THB_CIMB | CIMB Thai Bank Public Company Limited |
| attach.bank_code | THB_RHB | RHB Bank Berhad |
| attach.bank_code | THB_UOBT | United Overseas Bank (Thai) PCL |
| attach.bank_code | THB_BAY | Bank of Ayudhya Public Company Limited |
| attach.bank_code | THB_MEGA_ICBC | Mega International Commercial Bank Public Company Limited |
| attach.bank_code | THB_AMERICA | Bank of America National Association |
| attach.bank_code | THB_CALYON | Calyon |
| attach.bank_code | THB_IOB | Indian Overseas Bank, Bangkok Branch |
| attach.bank_code | THB_GOV | Government Saving Bank |
| attach.bank_code | THB_HSBC | Hong Kong & Shanghai Corporation Limited |
| attach.bank_code | THB_DB | Deutsche Bank Aktiengesellschaft |
| attach.bank_code | THB_GHB | Government Housing Bank |
| attach.bank_code | THB_AGRI | Bank for Agriculture and Agricultural Cooperatives |
| attach.bank_code | THB_EXIM | Export-Import Bank of Thailand |
| attach.bank_code | THB_MHCB | Mizuho Corporate Bank Limited |
| attach.bank_code | THB_BNPP | BNPN Paribas, Bangkok |
| attach.bank_code | THB_BOC | Bank of China Limited, Bangkok Branch |
| attach.bank_code | THB_ISBT | Islamic Bank of Thailand |
| attach.bank_code | THB_TISCO | Tisco Bank Public Company Limited |
| attach.bank_code | THB_KK | Kiatnakin Bank Public Company Limited |
| attach.bank_code | THB_ICBC | Industrial and Commercial Bank of China (THAI) Public Company Limited |
| attach.bank_code | THB_TCRB | The Thai Credit Retail Bank Public Company Limited |
| attach.bank_code | THB_LHBANK | Land and Houses Bank Public Company Limited |
| attach.bank_code | THB_ANZ | ANZ Bank (Thai) Public Company Limited |
| attach.bank_code | THB_SMTB | Sumitomo Mitsui Trust Bank (THAI) PCL. |
| attach.bank_code | THB_SMEB | Small and Medium Enterprise Development Bank of Thailand |



# 15、代付银行编码 代付字段 bank_code

| 字段名称 | 编码 | 银行名称 |
|:---------|:-----|:---------|
| bank_code | THB_BBL | Bangkok Bank Public Company Limited |
| bank_code | THB_KBANK | Kasikornbank Public Company Limited |
| bank_code | THB_KTB | Krung Thai Bank Public Company Limited |
| bank_code | THB_JPMC | JPMorgan Chase Bank, Bangkok Branch |
| bank_code | THB_OCBC | Oversea-Chinese Banking Corporation Ltd. |
| bank_code | THB_BTMU | The Bank of Tokyo-Mitsubishi UFJ, Ltd |
| bank_code | THB_TTB | TMB Bank Public Company Limited |
| bank_code | THB_SCB | Siam Commercial Bank Public Company Limited |
| bank_code | THB_CITI | Citibank N.A. |
| bank_code | THB_SMBC | Sumitomo Mitsui Banking Corporation |
| bank_code | THB_SCBT | Standard Chartered Bank (Thai) Public Company Limited |
| bank_code | THB_CIMB | CIMB Thai Bank Public Company Limited |
| bank_code | THB_RHB | RHB Bank Berhad |
| bank_code | THB_UOBT | United Overseas Bank (Thai) PCL |
| bank_code | THB_BAY | Bank of Ayudhya Public Company Limited |
| bank_code | THB_MEGA_ICBC | Mega International Commercial Bank Public Company Limited |
| bank_code | THB_AMERICA | Bank of America National Association |
| bank_code | THB_CALYON | Calyon |
| bank_code | THB_IOB | Indian Overseas Bank, Bangkok Branch |
| bank_code | THB_GOV | Government Saving Bank |
| bank_code | THB_HSBC | Hong Kong & Shanghai Corporation Limited |
| bank_code | THB_DB | Deutsche Bank Aktiengesellschaft |
| bank_code | THB_GHB | Government Housing Bank |
| bank_code | THB_AGRI | Bank for Agriculture and Agricultural Cooperatives |
| bank_code | THB_EXIM | Export-Import Bank of Thailand |
| bank_code | THB_MHCB | Mizuho Corporate Bank Limited |
| bank_code | THB_BNPP | BNPN Paribas, Bangkok |
| bank_code | THB_BOC | Bank of China Limited, Bangkok Branch |
| bank_code | THB_ISBT | Islamic Bank of Thailand |
| bank_code | THB_TISCO | Tisco Bank Public Company Limited |
| bank_code | THB_KK | Kiatnakin Bank Public Company Limited |
| bank_code | THB_ICBC | Industrial and Commercial Bank of China (THAI) Public Company Limited |
| bank_code | THB_TCRB | The Thai Credit Retail Bank Public Company Limited |
| bank_code | THB_LHBANK | Land and Houses Bank Public Company Limited |
| bank_code | THB_ANZ | ANZ Bank (Thai) Public Company Limited |
| bank_code | THB_SMTB | Sumitomo Mitsui Trust Bank (THAI) PCL. |
| bank_code | THB_SMEB | Small and Medium Enterprise Development Bank of Thailand |


# 16、文档更新时间
```
2026-06-11 00:30:00
```
