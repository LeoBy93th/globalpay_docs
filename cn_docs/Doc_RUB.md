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
> 2、金额单位为<span style="color:red;"> 分（Kopek） </span>。
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

| 名称             | 类型     | 必填    | 描述                                                                                                                                                                            |
| -------------- | ------ | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| trade_no       | int    | true  | 商户号                                                                                                                                                                           |
| app_id         | int    | true  | 商户 appId                                                                                                                                                                      |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                                                                                                                                 |
| pay_method     | string | true  | 支付方式  MIR_Bank                                                                                                                                                                |
| price          | int    | true  | 下单金额,单位:分 ,整数                                                                                                                                                                 |
| order_no       | string | true  | 商户订单号                                                                                                                                                                         |
| success_url    | string | false | 支付成功跳转 url                                                                                                                                                                    |
| fail_url       | string | false | 支付失败跳转 url                                                                                                                                                                    |
| pay_notice_url | string | false | 支付成功通知 url                                                                                                                                                                    |
| user_id        | string | true  | 系统用户ID                                                                                                                                                                        |
| user_ip        | string | true  | 付款人 IP                                                                                                                                                                        |
| attach         | string | false  | 附加参数 json字符串 付款人信息(建议填写){"account_no":"","account_type":"","name":"姓名","email":"邮箱","phone":"手机号","identify_type":"证件类型","identify_num":"证件号"} |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                                                                                                                                |
| timestamp      | string | false | 下单时间戳 10位时间戳单位S                                                                                                                                                               |

- 代收-attach 附加参数字段说明

| 名称            | 类型     | 必填    | 描述                                               |
| ------------- | ------ | ----- | ------------------------------------------------ |
| name          | string | false | 付款人姓名     |
| email         | string | false | 付款人邮箱                             |
| phone         | string | false | 付款人手机号  （真实有效）          |
| identify_type | string | false | 证件类型                               |
| identify_num  | string | false | 证件号码      |
| account_no    | string | false  | 付款账号                                             |
| account_type  | string | false  | 账号类型:  |
| bank_code     | string | false | 银行代码                                      |
| bank_name     | string | false | 银行名称                                             |

- 代收-下单请求示例

```json
  {
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
  }
```

## 4.2 代收-下单响应

| 名称           | 类型     | 必填    | 描述                                          |
| ------------ | ------ | ----- | ------------------------------------------- |
| code         | int    | true  | 200:下单成功 其他:下单失败                            |
| msg          | string | true  | 失败原因                                        |
| pay_url      | string | false | 付款链接                                        |
| qr_code      | string | false | pix 二维码字符串                                  |
| order_no     | string | true  | 商户订单号                                       |
| dis_order_no | string | true  | 平台订单号                                       |
| create_time  | int    | true  | 创建时间                                        |
| pay_info     | string | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 |
| sign         | string | true  | 签名结果,签名方法在文档顶部                              |

- 代收-下单响应示例

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
  "dis_order_no": "2025071130770572062498816india1oushe",
  "create_time": 1752825512,
  "pay_url": "https://api.sunpayinr.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5、代收回调通知 post/json

推送地址:商户下单传送的 pay_notice_url 回调 ip:call_back_server_ip ,请将我方 ip 加入回调白名单

## 5.1 代收回调-请求参数

| 名称           | 类型     | 必填    | 描述                                                                                                                                                                                           |
| ------------ | ------ | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| trade_no     | int    | true  | 商户号                                                                                                                                                                                          |
| status       | int    | true  | 订单状态, <span style="color:red;">2.成功</span>, 3.失败                                                                                                                                                                             |
| order_no     | string | true  | 商户订单号                                                                                                                                                                                        |
| dis_order_no | string | true  | 平台订单号                                                                                                                                                                                        |
| order_price  | int    | true  | 订单金额,单位:分                                                                                                                                                                                    |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">用户真实付款金额 ,单位:分</span>                                                                                                                                                                               |
| nti_time     | int    | false | 发起通知时间                                                                                                                                                                                       |
| payer        | string | false | JSON 字符串,付款人信息{"name":"姓名","account":"账号","bank":"付款的用户银行编码","utr2":"银行流水号","email":"邮箱","phone":"手机号","identify_type":"证件类型","identify_num":"证件号码"}，除示例字段外，当前参数会整合商户传递的attach里付款人信息相关字段 |
| pay_info     | string | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 25-10-28                                                                                                                                                  |
| create_time  | int    | true  | 创建时间                                                                                                                                                                                         |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                                                                                                                                                               |

- 代收回调-请求参数示例

```json
{
  "trade_no": 10003,
  "status": 3,
  "order_no": "p71584120256SlWlKkymb",
  "dis_order_no": "2025071130460153942908928india1sKQbX",
  "order_price": 10099,
  "real_price": 10000,
  "payer": "{\"name\":\"姓名\",\"email\":\"邮箱\",\"phone\":\"手机号\",\"identify_type\":\"证件类型\",\"identify_num\":\"证件号码\"}",
  "nti_time": 1752826164,
  "create_time": 1752751502,
  "sign": "eba7f27e0f49581d8784294ef29f994d"
}
```

## 5.2 代收回调-响应说明

回调接收处理成功，请返回 <span style="color:red;">success</span>，系统将不再推送此订单信息，否则还会重复推送多次

# 6、代付下单接口

(下单 ip 需要联系我方加白)  
下单地址 https://{api_domain}/api/v1/payApi/CreatePayOutOrder

## 6.1 代付-请求参数

<span style="color:red;">代付未返回明确失败的情况下均认为请求成功，包括超时，即未返回明确失败请不要重复请求。</span>

| 名称             | 类型     | 必填    | 描述                                                         |
| -------------- | ------ | ----- | ---------------------------------------------------------- |
| trade_no       | int    | true  | 商户号                                                        |
| order_no       | string | true  | 商户订单号                                                      |
| app_id         | int    | true  | 商户 appId                                                   |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                              |
| price          | int    | true  | 下单金额,单位:分 ,整数 转元后不能有小数点                                    |
| account_no     | string | true  | 收款账号                                                       |
| account_type   | string | true  | 账号类型:BANK |
| account_name   | string | true  | 姓名                                                         |
| bank_code      | string | true  | 收款银行代码 参照银行编码                                              |
| identify_type  | string | true  | 证件类型            |
| identify_num   | string | true  | 证件号码                                                       |
| pay_notice_url | string | false | 代付成功通知 url                                                 |
| attach         | string | false | 附加参数 {"email":"邮箱","phone":"手机号","bank_name":"银行名称"}       |
| user_ip        | string | true  | 收款用户 IP                                                    |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                             |
| timestamp      | string | false | 下单时间戳 10位时间戳单位S                                            |

- 代付-请求参数示例

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "87.200.59.100",
  "account_type": "BANK",
  "account_no": "40817810099910004312",
  "account_name": "Ivan Ivanov",
  "bank_code": "SBER",
  "identify_type": "BANK_CODE",
  "identify_num": "044525225"
}
```

## 6.2 代付-下单响应

| 名称           | 类型     | 必填   | 描述                                    |
| ------------ | ------ | ---- | ------------------------------------- |
| code         | int    | true | 200:下单成功 其他:下单失败                      |
| msg          | string | true | 失败原因                                  |
| dis_order_no | string | true | 平台订单号                                 |
| order_no     | string | true | 商户订单号                                 |
| status       | int    | true | 订单状态 2.代付成功, 3.代付失败, 7.驳回 9.冲正 10:处理中 |
| create_time  | int    | true | 创建时间                                  |
| sign         | string | true | 签名结果,签名方法在文档顶部                        |

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
  "dis_order_no": "2025071130776296733810688india1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7、代付回调通知

推送地址:商户下单传送的 pay_notice_url 回调 ip:call_back_server_ip ,请将我方 ip 加入回调白名单

## 7.1 代付回调请求参数

| 名称           | 类型     | 必填    | 描述                                          |
| ------------ | ------ | ----- | ------------------------------------------- |
| trade_no     | int    | true  | 商户号                                         |
| order_no     | string | true  | 商户订单号                                       |
| dis_order_no | string | true  | 平台订单号                                       |
| order_price  | int    | true  | 订单金额,单位:分                                   |
| fee          | int    | false | 订单手续费 ,单位:分                                 |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">真实代付出款金额 (代付成功时才有) 如需使用real_price字段，请与工作人员联系配置</span>                                                                                       |
| status       | int    | true  | 订单状态, <span style="color:red;">2.代付成功</span>, 3.代付失败, 7.驳回 9.冲正             |
| pay_info     | string | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 |
| remark       | string | false | 失败原因                                        |
| create_time  | int    | true  | 创建时间                                        |
| sign         | string | true  | 签名结果,签名方法在文档顶部                              |
| nti_time     | int    | true  | 发起通知时间                                      |

- 代付回调请求示例

```json
{
  "trade_no": 10003,
  "status": 3,
  "order_price": 10000,
"order_no": "p71584120257igU8n8FII",
  "dis_order_no": "2025071130700746140950528india15rLVI",
  "nti_time": 1752808888,
  "create_time": 1752808865,
  "sign": "746140950528indi"
}
```

## 7.2 代付回调响应说明

回调接收处理成功，请返回 <span style="color:red;">success</span>，系统将不再推送此订单信息，否则还会重复推送多次

# 8、查询订单接口 (代收 代付共用)

(请求 ip 需要联系我方加白)
查询地址: https://{api_domain}/api/v1/payApi/QueryOrder

## 8.1 查询请求参数

| 名称           | 类型     | 必填   | 描述                   |
| ------------ | ------ | ---- | -------------------- |
| order_type   | string | true | pay_out:代付,pay_in:代收 |
| trade_no     | int    | true | 商户号                  |
| app_id       | int    | true | 商户 appId             |
| dis_order_no | string | false | 平台订单号                |
| order_no | string | false  | 商户订单号                |
| sign         | string | true | 签名结果,签名方法在文档顶部       |

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

| 名称           | 类型     | 必填    | 描述                                                                                                                                                |
| ------------ | ------ | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| code         | int    | true  | 200:查询成功 其他:失败                                                                                                                                    |
| msg          | string | true  | 查询失败原因                                                                                                                                            |
| trade_no     | int    | true  | 商户号                                                                                                                                               |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">真实付款金额 ,单位:分</span>                                                                                                                                      |
| status       | int    | true  | 订单状态, 1.未支付, <span style="color:red;">2.成功</span>, 3.失败 , 7.驳回 9.冲正 10:处理中                                                                                                        |
| success_time | int    | true  | 成功时间戳                                                                                                                                             |
| order_no     | string | true  | 商户订单号                                                                                                                                             |
| dis_order_no | string | true  | 平台订单号                                                                                                                                             |
| remark       | string | true  | 代付失败原因                                                                                                                                            |
| fee          | int    | false | 订单手续费 ,单位:分                                                                                                                                       |
| create_time  | int    | true  | 创建时间                                                                                                                                              |
| payer        | string | false | JSON 字符串,付款人信息{"account_name":"姓名","account_type":"账号类型:BANK","account_no":"账号","bank_code":"银行编码"} |
| pay_info     | string | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 25-10-28                                                                                                       |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                                                                                                                    |
| utr2         | string | false | 银行订单号                                                                                                                                             |

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
  "payer": "{\"account_name\":\"Ivan Ivanov\",\"account_type\":\"BANK\",\"account_no\":\"40817810099910004312\",\"bank_code\":\"SBER\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9、代付余额查询接口

(请求 ip 需要联系我方加白)
地址: https://{api_domain}/api/v1/payApi/QueryBalance

## 9.1 余额请求参数

| 名称       | 类型     | 必填   | 描述             |
| -------- | ------ | ---- | -------------- |
| trade_no | int    | true | 商户号            |
| app_id   | int    | true | 商户 appid       |
| sign     | string | true | 签名结果,签名方法在文档顶部 |

- 余额请求示例

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 余额响应

| 名称             | 类型     | 必填    | 描述             |
| -------------- | ------ | ----- | -------------- |
| code           | int    | true  | 200:查询成功 其他:失败 |
| msg            | string | true  | 失败原因           |
| balance        | int    | true  | 余额,单位:分        |
| balance_frozen | int    | false | 冻结余额,单位:分      |
| sign           | string | true  | 签名结果,签名方法在文档顶部 |

- 余额响应示例

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

| 名称           | 类型     | 必填    | 描述                     |
| ------------ | ------ | ----- | ---------------------- |
| trade_no     | int    | true  | 商户号                    |
| app_id       | int    | true  | 商户 appid               |
| order_no     | string | false | 商户订单号 与dis_order_no二选一 |
| dis_order_no | string | false | 平台订单号 与order_no二选一     |
| sign         | string | true  | 签名结果,签名方法在文档顶部         |

- 付款凭证请求示例

```json
{
    "trade_no": 10003,
    "app_id": 10003,
    "order_no": "",
    "dis_order_no": "35011C02gljuf6k0800india1lVY",
    "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```

## 10.2 付款凭证响应参数

| 名称       | 类型     | 必填    | 描述             |
| -------- | ------ | ----- | -------------- |
| code     | int    | true  | 200:响应成功 其他:失败 |
| msg      | string | true  | 失败原因           |
| img_link | string | false | 凭证链接           |
| img_base | string | false | 凭证生成的64码       |
| sign     | string | true  | 签名结果,签名方法在文档顶部 |

- 付款凭证响应示例  
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

# 11、银行编码


| 字段名称      | 编码                                    | 银行名称                                  |
|:--------- |:------------------------------------- |:------------------------------------- |
| bank_code | RUB_BANK                              | RUBBANK                               |
| bank_code | SBER                                  | Сбер Be patient                       |
| bank_code | TINKOFF                               | Тинькофф Tinkoff                      |
| bank_code | АБ_РОССИЯ                             | АБ РОССИЯ                             |
| bank_code | АГРОПРОМКРЕДИТ                        | АГРОПРОМКРЕДИТ                        |
| bank_code | АКБ_Держава                           | АКБ Держава                           |
| bank_code | АКБ_ЕВРОФИНАНС_МОСНАРБАНК             | АКБ ЕВРОФИНАНС МОСНАРБАНК             |
| bank_code | АКБ_Приморье                          | АКБ Приморье                          |
| bank_code | АКБ_СЛАВИЯ                            | АКБ СЛАВИЯ                            |
| bank_code | АКБ_Тендер_Банк                       | АКБ Тендер Банк                       |
| bank_code | АКБ_ЦентроКредит                      | АКБ ЦентроКредит                      |
| bank_code | АКИБАНК                               | АКИБАНК                               |
| bank_code | Абсолют_Банк                          | Абсолют Банк                          |
| bank_code | Авангард                              | Авангард                              |
| bank_code | Авто_Финанс_Банк                      | Авто Финанс Банк                      |
| bank_code | Автоградбанк                          | Автоградбанк                          |
| bank_code | Автоторгбанк                          | Автоторгбанк                          |
| bank_code | Азиатско-Тихоокеанский_Банк           | Азиатско-Тихоокеанский Банк           |
| bank_code | Ак_Барс_Банк                          | Ак Барс Банк                          |
| bank_code | Алеф-Банк                             | Алеф-Банк                             |
| bank_code | Алмазэргиэнбанк                       | Алмазэргиэнбанк                       |
| bank_code | Алтайкапиталбанк                      | Алтайкапиталбанк                      |
| bank_code | Альфа-Банк                            | Альфа-Банк                            |
| bank_code | Америкэн_Экспресс_Банк                | Америкэн Экспресс Банк                |
| bank_code | Аресбанк                              | Аресбанк                              |
| bank_code | БАНК_МОСКВА-СИТИ                      | БАНК МОСКВА-СИТИ                      |
| bank_code | БАНК_ОРЕНБУРГ                         | БАНК ОРЕНБУРГ                         |
| bank_code | БАНК_УРАЛСИБ                          | БАНК УРАЛСИБ                          |
| bank_code | БЖФ_Банк                              | БЖФ Банк                              |
| bank_code | БКС_Банк                              | БКС Банк                              |
| bank_code | Банк_131                              | Банк 131                              |
| bank_code | Банк_Аверс                            | Банк Аверс                            |
| bank_code | Банк_Агророс                          | Банк Агророс                          |
| bank_code | Банк_Акцепт                           | Банк Акцепт                           |
| bank_code | Банк_Александровский                  | Банк Александровский                  |
| bank_code | Банк_ББР                              | Банк ББР                              |
| bank_code | Банк_БКФ                              | Банк БКФ                              |
| bank_code | Банк_ВБРР                             | Банк ВБРР                             |
| bank_code | Банк_Венец                            | Банк Венец                            |
| bank_code | Банк_Вологжанин                       | Банк Вологжанин                       |
| bank_code | Банк_ДОМ.РФ                           | Банк ДОМ.РФ                           |
| bank_code | Банк_Екатеринбург                     | Банк Екатеринбург                     |
| bank_code | Банк_Заречье                          | Банк Заречье                          |
| bank_code | Банк_Зенит                            | Банк Зенит                            |
| bank_code | Банк_ИПБ                              | Банк ИПБ                              |
| bank_code | Банк_ИТУРУП                           | Банк ИТУРУП                           |
| bank_code | Банк_Интеза                           | Банк Интеза                           |
| bank_code | Банк_Йошкар-Ола                       | Банк Йошкар-Ола                       |
| bank_code | Банк_Казани                           | Банк Казани                           |
| bank_code | Банк_Кремлевский                      | Банк Кремлевский                      |
| bank_code | Банк_МБА-МОСКВА                       | Банк МБА-МОСКВА                       |
| bank_code | Банк_Мир_Привилегий                   | Банк Мир Привилегий                   |
| bank_code | Банк_Объединенный_капитал             | Банк Объединенный капитал             |
| bank_code | Банк_Оранжевый                        | Банк Оранжевый                        |
| bank_code | Банк_ПСКБ                             | Банк ПСКБ                             |
| bank_code | Банк_Пермь                            | Банк Пермь                            |
| bank_code | Банк_Развитие-Столица                 | Банк Развитие-Столица                 |
| bank_code | Банк_Раунд                            | Банк Раунд                            |
| bank_code | Банк_Русский_Стандарт                 | Банк Русский Стандарт                 |
| bank_code | Банк_СИАБ                             | Банк СИАБ                             |
| bank_code | Банк_Саратов                          | Банк Саратов                          |
| bank_code | Банк_Синара                           | Банк Синара                           |
| bank_code | Банк_Снежинский                       | Банк Снежинский                       |
| bank_code | Банк_Финсервис                        | Банк Финсервис                        |
| bank_code | Банк_ЧБРР                             | Банк ЧБРР                             |
| bank_code | Белгородсоцбанк                       | Белгородсоцбанк                       |
| bank_code | Бланк_банк                            | Бланк банк                            |
| bank_code | Братский_АНКБ                         | Братский АНКБ                         |
| bank_code | БыстроБанк                            | БыстроБанк                            |
| bank_code | ВЛАДБИЗНЕСБАНК                        | ВЛАДБИЗНЕСБАНК                        |
| bank_code | ВНЕШФИНБАНК                           | ВНЕШФИНБАНК                           |
| bank_code | ВТБ                                   | ВТБ                                   |
| bank_code | ВУЗ_банк                              | ВУЗ банк                              |
| bank_code | Вайлдберриз_Банк                      | Вайлдберриз Банк                      |
| bank_code | ГЕНБАНК                               | ГЕНБАНК                               |
| bank_code | ГОРБАНК                               | ГОРБАНК                               |
| bank_code | ГУТА-БАНК                             | ГУТА-БАНК                             |
| bank_code | Газпромбанк                           | Газпромбанк                           |
| bank_code | Газтрансбанк                          | Газтрансбанк                          |
| bank_code | Газэнергобанк                         | Газэнергобанк                         |
| bank_code | Гарант-Инвест_банк                    | Гарант-Инвест банк                    |
| bank_code | Дальневосточный_банк                  | Дальневосточный банк                  |
| bank_code | Датабанк                              | Датабанк                              |
| bank_code | Джей_энд_Ти_Банк                      | Джей энд Ти Банк                      |
| bank_code | Дойче_Банк                            | Дойче Банк                            |
| bank_code | Долинск                               | Долинск                               |
| bank_code | Драйв_Клик_Банк                       | Драйв Клик Банк                       |
| bank_code | Енисейский_Объединенный_банк          | Енисейский Объединенный банк          |
| bank_code | Земский_банк                          | Земский банк                          |
| bank_code | Золотая_Корона_(РНКО_Платежный_центр) | Золотая Корона (РНКО Платежный центр) |
| bank_code | ИК_Банк                               | ИК Банк                               |
| bank_code | ИНЭКО                                 | ИНЭКО                                 |
| bank_code | ИШБАНК                                | ИШБАНК                                |
| bank_code | Инбанк                                | Инбанк                                |
| bank_code | Ингосстрах_Банк                       | Ингосстрах Банк                       |
| bank_code | КБ_ИС_Банк                            | КБ ИС Банк                            |
| bank_code | КБ_Новый_Век                          | КБ Новый Век                          |
| bank_code | КБ_Пойдем!                            | КБ Пойдем!                            |
| bank_code | КБ_РостФинанс                         | КБ РостФинанс                         |
| bank_code | КБ_СОЛИДАРНОСТЬ                       | КБ СОЛИДАРНОСТЬ                       |
| bank_code | КБ_Стройлесбанк                       | КБ Стройлесбанк                       |
| bank_code | КБ_Хлынов                             | КБ Хлынов                             |
| bank_code | КБ_ЮНИСТРИМ                           | КБ ЮНИСТРИМ                           |
| bank_code | КИВИ_Банк                             | КИВИ Банк                             |
| bank_code | КОШЕЛЕВ-БАНК                          | КОШЕЛЕВ-БАНК                          |
| bank_code | КЭБ_ЭйчЭнБи_Банк                      | КЭБ ЭйчЭнБи Банк                      |
| bank_code | Кредит_Европа_Банк_(Россия)           | Кредит Европа Банк (Россия)           |
| bank_code | Кредит_Урал_Банк                      | Кредит Урал Банк                      |
| bank_code | Крокус-Банк                           | Крокус-Банк                           |
| bank_code | Кубань_Кредит                         | Кубань Кредит                         |
| bank_code | Кубаньторгбанк                        | Кубаньторгбанк                        |
| bank_code | Кузнецкбизнесбанк                     | Кузнецкбизнесбанк                     |
| bank_code | ЛОКО-Банк                             | ЛОКО-Банк                             |
| bank_code | Ланта-Банк                            | Ланта-Банк                            |
| bank_code | Левобережный                          | Левобережный                          |
| bank_code | МЕЖДУНАРОДНЫЙ_ФИНАНСОВЫЙ_КЛУБ         | МЕЖДУНАРОДНЫЙ ФИНАНСОВЫЙ КЛУБ         |
| bank_code | МЕТКОМБАНК                            | МЕТКОМБАНК                            |
| bank_code | МКБ                                   | МКБ                                   |
| bank_code | МОНЕТА                                | МОНЕТА                                |
| bank_code | МОРСКОЙ_БАНК                          | МОРСКОЙ БАНК                          |
| bank_code | МОСКОМБАНК                            | МОСКОМБАНК                            |
| bank_code | МОСОБЛБАНК                            | МОСОБЛБАНК                            |
| bank_code | МС_Банк_Рус                           | МС Банк Рус                           |
| bank_code | МСП_Банк                              | МСП Банк                              |
| bank_code | МТС-Банк                              | МТС-Банк                              |
| bank_code | Металлинвестбанк                      | Металлинвестбанк                      |
| bank_code | Мир_Бизнес_Банк                       | Мир Бизнес Банк                       |
| bank_code | Модульбанк                            | Модульбанк                            |
| bank_code | Москоммерцбанк                        | Москоммерцбанк                        |
| bank_code | НБД-Банк                              | НБД-Банк                              |
| bank_code | НДБанк                                | НДБанк                                |
| bank_code | НИКО-БАНК                             | НИКО-БАНК                             |
| bank_code | НК_Банк                               | НК Банк                               |
| bank_code | НКО_МОБИ.Деньги                       | НКО МОБИ.Деньги                       |
| bank_code | НКО_Мобильная_карта                   | НКО Мобильная карта                   |
| bank_code | НКО_НРД                               | НКО НРД                               |
| bank_code | НКО_Перспектива_(Все_платежи)         | НКО Перспектива (Все платежи)         |
| bank_code | НКО_ЭЛЕКСНЕТ                          | НКО ЭЛЕКСНЕТ                          |
| bank_code | НКО_Элексир                           | НКО Элексир                           |
| bank_code | НКО_ЮМани                             | НКО ЮМани                             |
| bank_code | НОВИКОМБАНК                           | НОВИКОМБАНК                           |
| bank_code | НОКССБАНК                             | НОКССБАНК                             |
| bank_code | НРБанк                                | НРБанк                                |
| bank_code | НС_Банк                               | НС Банк                               |
| bank_code | Нацинвестпромбанк                     | Нацинвестпромбанк                     |
| bank_code | Национальный_стандарт                 | Национальный стандарт                 |
| bank_code | Новобанк                              | Новобанк                              |
| bank_code | Норвик_Банк                           | Норвик Банк                           |
| bank_code | ОТП_Банк                              | ОТП Банк                              |
| bank_code | Озон_Банк_(Ozon)                      | Озон Банк (Ozon)                      |
| bank_code | Открытие                              | Открытие                              |
| bank_code | Первый_Дортрансбанк                   | Первый Дортрансбанк                   |
| bank_code | Первый_Инвестиционный_Банк            | Первый Инвестиционный Банк            |
| bank_code | Почта_Банк                            | Почта Банк                            |
| bank_code | Примсоцбанк                           | Примсоцбанк                           |
| bank_code | Прио-Внешторгбанк                     | Прио-Внешторгбанк                     |
| bank_code | ПроБанк                               | ПроБанк                               |
| bank_code | ПромТрансБанк                         | ПромТрансБанк                         |
| bank_code | Промсвязьбанк                         | Промсвязьбанк                         |
| bank_code | Промсельхозбанк                       | Промсельхозбанк                       |
| bank_code | РЕСО_Кредит                           | РЕСО Кредит                           |
| bank_code | РНКБ_Банк                             | РНКБ Банк                             |
| bank_code | РНКО_Деньги.Мэйл.Ру                   | РНКО Деньги.Мэйл.Ру                   |
| bank_code | РУСНАРБАНК                            | РУСНАРБАНК                            |
| bank_code | Райффайзен_Банк                       | Райффайзен Банк                       |
| bank_code | Реалист_Банк                          | Реалист Банк                          |
| bank_code | Ренессанс_Кредит                      | Ренессанс Кредит                      |
| bank_code | РосДорБанк                            | РосДорБанк                            |
| bank_code | Росбанк                               | Росбанк                               |
| bank_code | Россельхозбанк                        | Россельхозбанк                        |
| bank_code | Роял_Кредит_Банк                      | Роял Кредит Банк                      |
| bank_code | Русьуниверсалбанк                     | Русьуниверсалбанк                     |
| bank_code | СДМ-Банк                              | СДМ-Банк                              |
| bank_code | СИНКО-БАНК                            | СИНКО-БАНК                            |
| bank_code | СМП_Банк                              | СМП Банк                              |
| bank_code | СНГБ                                  | СНГБ                                  |
| bank_code | СОЦИУМ-БАНК                           | СОЦИУМ-БАНК                           |
| bank_code | Санкт-Петербург                       | Санкт-Петербург                       |
| bank_code | Сбербанк                              | Сбербанк                              |
| bank_code | Свой_Банк                             | Свой Банк                             |
| bank_code | Севергазбанк                          | Севергазбанк                          |
| bank_code | Северный_Народный_Банк                | Северный Народный Банк                |
| bank_code | Сибсоцбанк                            | Сибсоцбанк                            |
| bank_code | Ситибанк                              | Ситибанк                              |
| bank_code | Совкомбанк                            | Совкомбанк                            |
| bank_code | Солид_Банк                            | Солид Банк                            |
| bank_code | Ставропольпромстройбанк               | Ставропольпромстройбанк               |
| bank_code | ТАТСОЦБАНК                            | ТАТСОЦБАНК                            |
| bank_code | Таврический_Банк                      | Таврический Банк                      |
| bank_code | Тимер_Банк                            | Тимер Банк                            |
| bank_code | Тинькофф                              | Тинькофф                              |
| bank_code | Тойота_Банк                           | Тойота Банк                           |
| bank_code | Тольяттихимбанк                       | Тольяттихимбанк                       |
| bank_code | Томскпромстройбанк                    | Томскпромстройбанк                    |
| bank_code | Точка_'ФК_Открытие'                   | Точка 'ФК Открытие'                   |
| bank_code | Точка_Банк                            | Точка Банк                            |
| bank_code | Точка_КИВИ                            | Точка КИВИ                            |
| bank_code | Транскапиталбанк                      | Транскапиталбанк                      |
| bank_code | Трансстройбанк                        | Трансстройбанк                        |
| bank_code | УБРиР                                 | УБРиР                                 |
| bank_code | УРАЛПРОМБАНК                          | УРАЛПРОМБАНК                          |
| bank_code | Углеметбанк                           | Углеметбанк                           |
| bank_code | Урал_ФД                               | Урал ФД                               |
| bank_code | Уралфинанс                            | Уралфинанс                            |
| bank_code | ФИНАМ                                 | ФИНАМ                                 |
| bank_code | ФОРА-БАНК                             | ФОРА-БАНК                             |
| bank_code | ФФИН_Банк_(Цифра_банк)                | ФФИН Банк (Цифра банк)                |
| bank_code | Форштадт                              | Форштадт                              |
| bank_code | Хайс                                  | Хайс                                  |
| bank_code | Хакасский_муниципальный_банк          | Хакасский муниципальный банк          |
| bank_code | Хоум_Кредит_Банк_(Хоум_Банк)          | Хоум Кредит Банк (Хоум Банк)          |
| bank_code | Центр-инвест                          | Центр-инвест                          |
| bank_code | ЧЕЛИНДБАНК                            | ЧЕЛИНДБАНК                            |
| bank_code | ЧЕЛЯБИНВЕСТБАНК                       | ЧЕЛЯБИНВЕСТБАНК                       |
| bank_code | Экономбанк                            | Экономбанк                            |
| bank_code | Экспобанк                             | Экспобанк                             |
| bank_code | Элита                                 | Элита                                 |
| bank_code | Элплат                                | Элплат                                |
| bank_code | Энергобанк                            | Энергобанк                            |
| bank_code | Энерготрансбанк                       | Энерготрансбанк                       |
| bank_code | Эс-Би-Ай_Банк                         | Эс-Би-Ай Банк                         |
| bank_code | ЮГ-Инвестбанк                         | ЮГ-Инвестбанк                         |
| bank_code | Юникредит_Банк                        | Юникредит Банк                        |
| bank_code | Яндекс_Банк                           | Яндекс Банк                           |

# 12、错误码

| 状态码 | 描述  |
| --- | --- | 
| 200 | 成功  |
| 1000 | 内部错误 |
| 1001 | ip不在商户ip白名单 |
| 1002 | 参数错误 |
| 1003 | 签名错误 |
| 1004 | 当前商户暂未开放该接口 （联系运营核实:商户或App(不存在 | 已关闭 | 支付产品未配置)） |
| 1005 | 商户不存在 |
| 1006 | 当前用户ip在黑名单中 |
| 1007 | 当前用户在黑名单中 |
| 1008 | 商户App不存在 |
| 1009 | 支付产品不存在 |
| 1010 | 支付渠道不存在 |
| 1011 | 支付通道暂未开发完成，暂时不开放 |
| 1012 | 支付通道异常，请稍后再试 |
| 1013 | 当前单量过大，请稍后再试 |
| 1014 | 订单号重复 |
| 1015 | app余额不足 |
| 1016 | 同一用户频繁下单，请稍后再试 |
| 1017 | 订单记录不存在 |
| 1018 | 当前金额不支持 |
| 1019 | 当前app所在国家暂未开通支付下单 |
| 1020 | 当前app所在国家暂未开通代付下单 |
| 1021 | 失败  |
| 1036 | 接口暂未开放 |
| 1037 | 不支持该货币 |
| 1038 | 代收回传utr异常 |
| 9999 | 其他错误 |
| 3000 | 系统升级维护中，暂停下单，请稍后尝试 |

# 13、代收收银台接口

地址: https://{api_domain}/api/v1/cashApi/CashIn.html
请求方式: GET

### 参数:

| 名称           | 类型     | 必填    | 描述                        |
|--------------|--------|-------|---------------------------|
| app_id       | string | true  | 商户app_id                   |
| order_no       | string | true  | 商户订单号                     |
| amount       | string | true  | 商户金额单位卢布                    |
| notice_url       | string | false | 异步通知地址                    |
| pay_code       | int    | true | 产品编码                     |

#### 示例

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{商户订单号}}&amount={{商户金额}}&notice_url={{异步通知地址}}&pay_code={{产品编码}}
```

# 14、文档更新时间
```
2026-05-10 19:25:00
```
