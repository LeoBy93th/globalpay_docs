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
> 4、sign=md5(上一步组装待签名字的符串) 签名结果 32 位小写
>
> 5、签名密钥,请在商户后台->基础信息 查看,或者询问我方客服

# 3、注意事项

## 3.1 接口相关
> 1、本文档中的所有接口，均采用 HTTP 标准通信协议，POST提交，请求和响应的 Content-type 均为 application/json，字符编码统一为 UTF-8。
>
> 2、金额单位为 <span style="color:red;">分（Centavo）</span>  。
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

| 名称             | 类型   | 必填  | 描述                          |
|----------------|------|-----|-----------------------------|
| trade_no       | int    | true  | 商户号                         |
| app_id         | int    | true  | 商户 appId                    |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                        |
| pay_method     | string    | true  | 支付方式   参照支付方式字典                      |
| price          | int    | true  | 下单金额，单位：<span style="color:red;">分（Centavo）</span>，整型。1 阿根廷比索 = 100 分（Centavo） |
| order_no       | string | true  | 商户订单号                              |
| success_url    | string | false | 支付成功跳转 url                         |
| fail_url       | string | false | 支付失败跳转 url                         |
| pay_notice_url | string | false | 支付成功通知 url  `建议填写`               |
| user_id        | string | false | 系统用户ID   `建议填写`                 |
| user_ip        | string | false | 付款人 IP    `建议填写`                |
|attach|string|true| 附加参数 json字符串   |
| sign           | string | true  | 签名结果,签名方法在文档顶部 |
|timestamp|string|false| 下单时间戳 10位时间戳单位S |

-  代收-attach 附加参数字段说明

| 名称           | 类型     | 必填    | 描述                                                    |
|--------------|--------|-------|-------------------------------------------------------|
| name       | string | false | 姓名   `建议填写`                                           |
| phone       | string | false | 手机号   `建议填写`                                          |
| email       | string | false | 邮箱   `建议填写`                                           |
| identify_type       | string    | true | 证件类型 ` DNI（个人身份证-8位数字) RUC(税卡-11位数)` |
| identify_num       | string    | true  | 证件号码 `当identify_type=DNI时，阿根廷身份id 8 位（确实没有的保证位数足够`    |

- attach 示例
```json
{"identify_type":"DNI", "identify_num":"123456789" }
```

- 代收-下单请求示例

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_method": "CVU",
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"identify_type\":\"DNI\", \"identify_num\":\"123456789\" }",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}

```

## 4.2  代收-下单响应

| 名称         | 类型   | 必填 | 描述                                                |
|------------|------|----|---------------------------------------------------|
| code         | int    | true | 200:下单成功 其他:下单失败                                  |
| msg          | string | true | 失败原因                                              |
| pay_url      | string | false | 付款链接                                              |
| qr_code      | string | false | 二维码base64字符串                                      |
| order_no     | string | true | 商户订单号                                             |
| dis_order_no | string | true | 平台订单号                                             |
| create_time  | int    | true | 创建时间                                              |
| pay_info  | string    | false | 付款信息 json字符串 例如：{"pay_raw":"支付原生信息，商户可以自行转换成二维码"} |
| sign         | string | true | 签名结果,签名方法在文档顶部                                    |

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
  "dis_order_no": "2025071130770572062498816india1oushe",
  "create_time": 1752825512,
  "pay_url": "https://api.sunpayinr.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
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
| order_price  | int    | true  | 订单金额,单位:<span style="color:red;">分（Centavo）</span>                                                                                              |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">用户真实付款金额 ,单位:分（Centavo）</span>                                                                                         |
| nti_time     | int    | false | 发起通知时间                                                                                                 |
| payer        | string | false | JSON 字符串,付款人信息{"name":"姓名","account":"账号","utr2":"银行流水号"}，除示例字段外，当前参数会整合商户传递的attach里付款人信息相关字段|
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 |
| create_time  | int    | true  | 创建时间                                                                                                   |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                                                                         |

-  代收回调-请求参数示例

```json
{
  "trade_no": 10003,
  "status": 3,
  "order_no": "p71584120256SlWlKkymb",
  "dis_order_no": "2025071130460153942908928india1sKQbX",
  "order_price": 10099,
  "real_price": 10000,
  "payer": "{\"name\":\"姓名\"}",
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

| 名称             | 类型   | 必填    | 描述                                                                |
|----------------|------|-------|-------------------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                               |
| order_no       | string | true  | 商户订单号                                                             |
| app_id         | int    | true  | 商户 appId                                                          |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                     |
| price          | int    | true  | 下单金额，单位：<span style="color:red;">分（Centavo）</span> ，整型            |
| account_no     | string | true  | 收款账号 `CVU/CBU/BANK 账号`                                            |
| account_type   | string | true  | 账号类型: CBU ,CVU, BANK                                              |
| account_name   | string | true  | 姓名                                                                |
| bank_code      | string | true  | 银行编码   参照银行编码字典                                                   |
| identify_type  | string | true  | 证件类型: DNI（个人身份证-8位数字) CUIL(社保卡-11位数字) RUC(税卡-11位数) PP(护照:最长15位数字) |
| identify_num   | string | true  | 证件号码                                                              |
| pay_notice_url | string | false | 代付成功通知 url                                                        |
| attach         | string | false | 附加参数   {"email":"邮箱","phone":"手机号"}                               |
| user_ip        | string | false  | 收款用户 IP    `建议填写，不能使用同一个ip避免出现限流`                                 |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                    |
|timestamp|string| false | 下单时间戳 10位时间戳单位S   `建议填写`                                          |

-  代付-attach 附加参数字段说明

| 名称           | 类型     | 必填    | 描述                            |
|--------------|--------|-------|-------------------------------|
| phone       | string | false | 手机号   `建议填写`                         |
| email       | string | false | 邮箱   `建议填写`                         |

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
  "account_type": "CBU",
  "account_no": "2850590940090418135201",
  "account_name": "Juan Perez",
  "bank_code": "CBU",
  "identify_type": "DNI",
  "identify_num": "12345678"
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
  "dis_order_no": "2025071130776296733810688india1Dhr7H",
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
| order_price        | int    | true  | 订单金额                                  |
| fee          | int    | false | 订单手续费                                   |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">真实代付出款金额 (代付成功时才有) 如需使用real_price字段，请与工作人员联系配置</span>                                                                                       |
| status       | int    | true  | 订单状态, <span style="color:red;">2.代付成功</span>, 3.代付失败, 7.驳回 9.冲正               |
| pay_info  | string    | false | 付款信息 |
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
  "dis_order_no": "Meg2352644o2nmjo0800indiaYZ2A",
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

| 名称         | 类型   | 必填   | 描述                   |
|------------|------|------|----------------------|
| order_type | string | true | pay_out:代付,pay_in:代收 |
| trade_no   | int    | true | 商户号                  |
| app_id         | int    | true | 商户 appId             |
| order_no   | string    | false | 商户订单号  与dis_order_no二选一    |
| dis_order_no   | string    | false | 平台订单号  与order_no二选一    |
| sign       | string | true | 签名结果,签名方法在文档顶部       |

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

| 名称         | 类型   | 必填  | 描述                                                                  |
|------------|------|-----|---------------------------------------------------------------------|
| code         | int    | true  | 200:查询成功 其他:失败                                                      |
| msg          | string | true  | 查询失败原因                                                              |
| trade_no     | int    | true  | 商户号                                                                 |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">真实付款金额</span>                                                     |
| status       | int    | true  | 订单状态, 1.未支付, <span style="color:red;">2.成功</span>, 3.失败 , 7.驳回 9.冲正  10:处理中                         |
| success_time | int    | true  | 成功时间戳                                                               |
| order_no     | string | true  | 商户订单号                                                               |
| dis_order_no | string | true  | 平台订单号                                                       |
| remark       | string | true  | 代付失败原因                                                      |
| fee          | int    | false | 订单手续费                                              |
| create_time  | int    | true  | 创建时间                                                        |
| payer        | string | false | JSON 字符串,付款人信息{"account_name":"姓名","account_type":"账号类型:CBU,CVU,BANK","account_no":"账号","identify_type":"证件类型:DNI,CUIL,RUC,PP","identify_num":"证件号码","bank_code":"银行编码","utr2":"银行流水号"}|
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等|
| sign         | string | true  | 签名结果,签名方法在文档顶部           |
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
  "payer": "{\"account_name\":\"Juan Perez\",\"account_type\":\"CBU\",\"account_no\":\"2850590940090418135201\",\"identify_type\":\"DNI\",\"identify_num\":\"12345678\",\"bank_code\":\"CBU\",\"utr2\":\"9901108391\"}",
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
| balance | int    | true | 余额              |
| balance_frozen | int    | false | 冻结余额                |
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
  "dis_order_no": "35011C02gljuf6k0800india1lVY",
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

# 11、支付方式字典 代收字段 pay_method

| 字段名称 | 值         | 描述                    |
|------|-----------|-----------------------|
| pay_method     | CVU       | 阿根廷银行转账       `该支付方式需要用户付完款在收银台填支付银行单号，不然会掉单`        |
| pay_method     | QR_ARS | 扫码 `暂不支持，以后扫码开通后通知商户` |

# 12、银行编码字典 代付字段 bank_code

| 字段名称 | 值 | 描述 |
|------|-----------|-----------------------|
| bank_code     | CBU | CBU |
| bank_code     | CVU                                                       | CVU |
| bank_code     | ALIAS                                                     | ALIAS |
| bank_code     | ABACO_CIA_FINANCIERA                                      | ABACO CIA. FINANCIERA S.A. |
| bank_code     | ALHEC_TOURSC                                              | ALHEC TOURS S.A.C.B.YT. |
| bank_code     | AMERICAN_EXPRESS                                          | AMERICAN EXPRESS BANK LTD. |
| bank_code     | ASTROPAY                                                  | ASTROPAY |
| bank_code     | BANKERS_TRUST_ARGENTINA_SOCIEDAD_DE_BOLSA                 | BANKERS TRUST ARGENTINA S.A. SOCIEDAD DE BOLSA |
| bank_code     | BANKERS_TRUST_RIO_DE_LA_PLATA_SACF                        | BANKERS TRUST RIO DE LA PLATA SACF |
| bank_code     | BBVA_FRANCES                                              | BBVA BANCO FRANCES S.A. |
| bank_code     | BNP_PARIBAS_SUCCURSALE_DE_BUENOS_AIRES                    | BNP PARIBAS SUCCURSALE DE BUENOS AIRES |
| bank_code     | BOLSA_DE_COMERCIO_ROSARIO                                 | BOLSA DE COMERCIO ROSARIO |
| bank_code     | BOSTON_INVERSORA_DE_VALORES                               | BOSTON INVERSORA DE VALORES S.A. |
| bank_code     | BRADESCO_ARGENTINA                                        | BANCO BRADESCO ARGENTINA S/A |
| bank_code     | CAJA_DE_VALORES                                           | CAJA DE VALORES S.A. |
| bank_code     | CAMBIO_EXCURSIONES_TURISMO_COLUMBUS                       | CAMBIO EXCURSIONES TURISMO COLUMBUS S.A. |
| bank_code     | CAMBIO_GARCIA_NAVARRO_RAMAGLIO_Y_CIA                      | CAMBIO GARCIA NAVARRO RAMAGLIO Y CIA. S.A. |
| bank_code     | CAPITAL_MARKETS                                           | CAPITAL MARKETS |
| bank_code     | CASA_LERCHUNDI_SACIF                                      | CASA LERCHUNDI SACIF |
| bank_code     | CENTRAL_DE_LA_REPUBLICA_ARGENTINA                         | BANCO CENTRAL DE LA REPUBLICA ARGENTINA |
| bank_code     | CITIBANK_ARS                                                  | CITIBANK N.A. |
| bank_code     | CMF                                                       | BANCO CMF S.A. |
| bank_code     | COLUMBIA                                                  | BANCO COLUMBIA S.A. |
| bank_code     | COMAFI                                                    | BANCO COMAFI S.A. |
| bank_code     | COOPERATIVO_DEL_ESTE_ARGENTINO                            | BANCO COOPERATIVO DEL ESTE ARGENTINO LTDO. |
| bank_code     | CORPORACION_DE_SERVICIOS_FINANCIEROS                      | CORPORACION DE SERVICIOS FINANCIEROS S.A. |
| bank_code     | CREDICOOP_COOPERATIVO_LIMITADO                            | BANCO CREDICOOP COOPERATIVO LIMITADO |
| bank_code     | DEL_CHUBUT                                                | BANCO DEL CHUBUT S.A. |
| bank_code     | DEL_TUCUMAN                                               | BANCO DEL TUCUMAN S.A. |
| bank_code     | DEUTSCHE_ARS                                                  | DEUTSCHE BANK SA |
| bank_code     | DE_FORMOSA                                                | BANCO DE FORMOSA S.A. |
| bank_code     | DE_GALICIA_Y_BUENOS_AIRES                                 | BANCO DE GALICIA Y BUENOS AIRES |
| bank_code     | DE_INVERSION_Y_COMERCIO_EXTERIOR                          | BANCO DE INVERSION Y COMERCIO EXTERIOR S.A. |
| bank_code     | DE_LA_CIUDAD_DE_BUENOS_AIRES                              | BANCO DE LA CIUDAD DE BUENOS AIRES |
| bank_code     | DE_LA_NACION_ARGENTINA                                    | BANCO DE LA NACION ARGENTINA |
| bank_code     | DE_LA_PAMPA                                               | BANCO DE LA PAMPA S.E.M. |
| bank_code     | DE_LA_PROVINCIA_DEL_NEUQUEN                               | BANCO DE LA PROVINCIA DEL NEUQUEN S.A. |
| bank_code     | DE_LA_PROVINCIA_DE_BUENOS_AIRES                           | BANCO DE LA PROVINCIA DE BUENOS AIRES |
| bank_code     | DE_LA_PROVINCIA_DE_CORDOBA                                | BANCO DE LA PROVINCIA DE CORDOBA |
| bank_code     | DE_LA_REPUBLICA_ORIENTAL_DEL_URUGUAY                      | BANCO DE LA REPUBLICA ORIENTAL DEL URUGUAY |
| bank_code     | DE_SANTA_CRUZ                                             | BANCO DE SANTA CRUZ S A |
| bank_code     | DE_SAN_JUAN                                               | BANCO DE SAN JUAN SA |
| bank_code     | DE_SERVICIOS_Y_TRANSACCIONES                              | BANCO DE SERVICIOS Y TRANSACCIONES SA |
| bank_code     | DE_VALORES                                                | BANCO DE VALORES |
| bank_code     | FINANCIERA_BUENOS_AIRESL                                  | FINANCIERA BUENOS AIRES S.A.L. Y C. |
| bank_code     | FINANSUR                                                  | BANCO FINANSUR S.A. |
| bank_code     | FITCH_ARGENTINA_CALIFICADORA_DE_RIESGO                    | FITCH ARGENTINA CALIFICADORA DE RIESGO S.A. |
| bank_code     | FOREXCAMBIO                                               | FOREXCAMBIO SA |
| bank_code     | GARCIA_NAVARROY                                           | GARCIA NAVARROY CIA. S.A. |
| bank_code     | HIPOTECARIO                                               | BANCO HIPOTECARIO S.A. |
| bank_code     | HSBC_BANK_ARGENTINA                                       | HSBC BANK ARGENTINA SA |
| bank_code     | INDUSTRIAL                                                | BANCO INDUSTRIAL S.A. |
| bank_code     | INDUSTRIAL_AND_COMMERCIAL_BANK_OF_CHINA                   | INDUSTRIAL AND COMMERCIAL BANK OF CHINA (ARGENTINA)S.A. |
| bank_code     | INTERAMERICANO_DE_DESARROLLO                              | BANCO INTERAMERICANO DE DESARROLLO |
| bank_code     | INTERFINANZAS                                             | BANCO INTERFINANZAS S.A. |
| bank_code     | INVERSIONARIA_ARGENTINA                                   | INVERSIONARIA ARGENTINA S.A. |
| bank_code     | INVERSIONES_UNIDAS                                        | INVERSIONES UNIDAS S.A. |
| bank_code     | INVERSORA_COMPANIA_FINANCIERA                             | INVERSORA S.A. COMPANIA FINANCIERA |
| bank_code     | INVERSUR                                                  | INVERSUR S.R.L. |
| bank_code     | ITAU_ARGENTINA                                            | BANCO ITAU ARGENTINA S.A. |
| bank_code     | JPMORGAN_CHASE                                            | JPMORGAN CHASE BANK N.A. |
| bank_code     | MACRO                                                     | BANCO MACRO S.A. |
| bank_code     | MARIVA                                                    | BANCO MARIVA S.A. |
| bank_code     | MBA_DE_INVERSIONES                                        | MBA BANCO DE INVERSIONES SA |
| bank_code     | MERCADADOPAGO                                             | MERCADADOPAGO |
| bank_code     | MERCADO_DE_VALORES_DE_BUENOS_AIRES_MERVAL                 | MERCADO DE VALORES DE BUENOS AIRES S.A. MERVAL |
| bank_code     | MERCADO_DE_VALORES_DE_BUENOS_AIRES_SA                     | MERCADO DE VALORES DE BUENOS AIRES SA |
| bank_code     | MERCHANT_BANK_ARGENTINA                                   | MERCHANT BANK ARGENTINA |
| bank_code     | MERCURIO                                                  | BANCO MERCURIO S.A. |
| bank_code     | MERIDIAN                                                  | BANCO MERIDIAN S.A. |
| bank_code     | METROPOLIS_COMPANIA_FINANCIERA                            | METROPOLIS COMPANIA FINANCIERA S.A. |
| bank_code     | MONTEMAR_COMPANIA_FINANCIERA                              | MONTEMAR COMPANIA FINANCIERA S.A. |
| bank_code     | MOODYS_LATIN_AMERICA_CALIFICADORA_DE_RIESGO               | MOODYS LATIN AMERICA CALIFICADORA DE RIESGO S.A. |
| bank_code     | MULTIFINANZAS_COMPANIA_FINANCIERA                         | MULTIFINANZAS COMPANIA FINANCIERA S.A. |
| bank_code     | MUNICIPAL_DE_ROSARIO                                      | BANCO MUNICIPAL DE ROSARIO |
| bank_code     | NUEVO_DEL_CHACO                                           | NUEVO BANCO DEL CHACO S.A. |
| bank_code     | NUEVO_DE_ENTRE_RIOS                                       | NUEVO BANCO DE ENTRE RIOS S.A. |
| bank_code     | NUEVO_DE_SANTA_FE                                         | NUEVO BANCO DE SANTA FE S.A. |
| bank_code     | PATAGONIA                                                 | BANCO PATAGONIA S.A. |
| bank_code     | PIANO                                                     | BANCO PIANO S.A. |
| bank_code     | PROVINCIA_DE_TIERRA_DEL_FUEGO                             | BANCO PROVINCIA DE TIERRA DEL FUEGO |
| bank_code     | RABELLO_Y_CIA_SOCIEDAD_DE_BOLSA                           | RABELLO Y CIA S.A. SOCIEDAD DE BOLSA |
| bank_code     | RAYMOND_JAMES_ARGENTINA_SOCIEDAD_DE_BOLSA_SOCIEDADANONIMA | RAYMOND JAMES ARGENTINA SOCIEDAD DE BOLSA SOCIEDADANONIMA |
| bank_code     | ROELA                               | BANCO ROELA S.A. |
| bank_code     | ROYAL_BANK_OF_SCOTLAND_NV           | ROYAL BANK OF SCOTLAND NV (SUC ARGENTINA) |
| bank_code     | SAENZ                               | BANCO SAENZ S.A. |
| bank_code     | SAMSUNG_ELECTRONICS_ARGENTINA       | SAMSUNG ELECTRONICS ARGENTINA S.A. |
| bank_code     | SANTANDER_RIO                       | BANCO SANTANDER RIO S.A. |
| bank_code     | SBS_SOCIEDAD_DE_BOLSA               | SBS SOCIEDAD DE BOLSA S.A. |
| bank_code     | SUPERVIELLE                         | BANCO SUPERVIELLE |
| bank_code     | T.P.C.G._CAPITAL                    | T.P.C.G. CAPITAL S.A. |
| bank_code     | TAVELLI_Y_CIA.S.A.SOCIEDAD_DE_BOLSA | TAVELLI Y CIA.S.A.SOCIEDAD DE BOLSA |
| bank_code     | TOKYO_MITSUBISHI_UFJ_LTD_THE | BANK OF TOKYO MITSUBISHI UFJ LTD. THE |
| bank_code     | TRANSCAMBIO                 | TRANSCAMBIO S.A. |
| bank_code     | TUTELAR_COMPANIA_FINANCIERA | TUTELAR COMPANIA FINANCIERA S.A. |
| bank_code     | VANEXVA_BURSATIL_SA | VANEXVA BURSATIL SA |

# 13、错误码

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

# 14、代收收银台接口

地址: https://{api_domain}/api/v1/cashApi/CashIn.html
请求方式: GET

### 参数:

| 名称           | 类型     | 必填    | 描述           |
|--------------|--------|-------|--------------|
| app_id       | string | true  | 商户app_id     |
| order_no       | string | true  | 商户订单号        |
| amount       | string | true  | 商户金额单位 `阿根廷比索` |
| notice_url       | string | false | 异步通知地址       |
| pay_code       | int    | true | 产品编码         |

#### 示例

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{商户订单号}}&amount={{商户金额}}&notice_url={{异步通知地址}}&pay_code={{产品编码}}
```

# 15、文档更新时间
```
2026-06-10 20:25:00
```
