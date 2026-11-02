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
> 2、金额单位为<span style="color:red;"> 港仙（Cents） </span>。
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

| 名称             | 类型   | 必填  | 描述                                                                                                      |
|----------------|------|-----|---------------------------------------------------------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                                                                     |
| app_id         | int    | true  | 商户 appId                                                                                                |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                                                           |
| pay_method     | string    | true  | 支付方式,见下方支付方式                                                                                            |
| price          | int    | true  | 下单金额，单位：港仙（Cents），整型。1 港元（HKD）= 100 港仙                                                                 |
| order_no       | string | true  | 商户订单号                                                                                                   |
| success_url    | string | false | 支付成功跳转 url                                                                                              |
| fail_url       | string | false | 支付失败跳转 url                                                                                              |
| pay_notice_url | string | false | 支付成功通知 url                                                                                              |
| user_id        | string | false | 商家用户ID                                                                                                  |
| user_ip        | string | false | 付款人 IP                                                                                                  |
|attach|string|false| 附加参数 json字符串 付款人信息 |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                                                          |
|timestamp|string|false| 下单时间戳 10位时间戳单位S                                                                                         |

- 代收-attach 附加参数字段说明

| 名称            | 类型     | 必填    | 描述                                                                      |
|---------------|--------|-------|-------------------------------------------------------------------------|
| phone         | string    | false | 手机号(尽量传真实手机号，没有可填充)                                                     |
| email         | string    | false | 邮箱(尽量传真实邮箱，没有可填充)                                                       |
| product_name  | string    | false | 商品名                                                                    |


- 代收-下单请求示例

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"phone\":\"91234567\",\"email\":\"john.doe@example.com\",\"product_name\":\"Test Product\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "203.198.1.1",
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
| qr_code      | string | false | 二维码字符串                                                                                                                                                                                          |
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
  "dis_order_no": "2025071130770572062498816hkg1oushe",
  "create_time": 1752825512,
  "pay_url": "https://{api_domain}/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5、代收回调通知   post/json

推送地址:商户下单传送的 pay_notice_url 回调 ip:call_back_server_ip ,请将我方 ip 加入回调白名单

## 5.1 代收回调-请求参数

| 名称         | 类型   | 必填  | 描述                                                                                                                                                                                       |
|------------|------|-----|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| trade_no     | int    | true  | 商户号                                                                                                                                                                                      |
| status       | int    | true  | 订单状态, <span style="color:red;">2.成功</span>, 3.失败                                                                                                                                                                         |
| order_no     | string | true  | 商户订单号                                                                                                                                                                                    |
| dis_order_no | string | true  | 平台订单号                                                                                                                                                                                    |
| order_price  | int    | true  | 订单金额,单位:港仙（Cents）                                                                                                                                                                        |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">用户真实付款金额 ,单位:港仙（Cents）</span>                                                                                                                                                                   |
| nti_time     | int    | false | 发起通知时间                                                                                                                                                                                   |
| payer        | string | false | JSON 字符串,付款人信息{"name":"姓名","account":"账号","bank":"付款的用户银行编码","utr2":"银行流水号","email":"邮箱","phone":"手机号","identify_type":"证件类型","identify_num":"证件号码"}，除示例字段外，当前参数会整合商户传递的attach里付款人信息相关字段 |
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等                                                                                                                                             |
| create_time  | int    | true  | 创建时间                                                                                                                                                                                     |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                                                                                                                                                           |

-  代收回调-请求参数示例

```json
{
  "trade_no": 10238,
  "status": 2,
  "order_no": "RCG0126042435820040804784",
  "dis_order_no": "Meg1352644s0800hkganVMR",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1776680622,
  "payer": "{\"account_no\":\"1234567890\",\"account_type\":\"FPS\",\"identify_num\":\"A123456(7)\",\"identify_type\":\"HKID\",\"req_api_ip\":\"203.198.1.1\",\"utr2\":\"9901108391\"}",
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

<span style="color:red;">代付未返回明确失败的情况下均认为请求成功，包括超时，即未返回明确失败请不要重复请求。</span>

| 名称             | 类型     | 必填    | 描述                                                                     |
|----------------|--------|-------|------------------------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                                    |
| order_no       | string | true  | 商户订单号                                                                  |
| app_id         | int    | true  | 商户 appId                                                               |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                          |
| price          | int    | true  | 下单金额，单位：港仙（Cents），整型。1 港元（HKD）= 100 港仙                                 |
| account_no     | string | true  | 收款账号                                                                   |
| account_type   | string | false | 账号类型                                                                   |
| account_name   | string | true  | 姓名                                                                     |
| bank_code      | string | true  | 银行编码                                                                   |
| identify_type  | string | false | 证件类型                                                                   |
| identify_num   | string | false | 证件号码                                                                   |
| pay_notice_url | string | false | 代付成功通知 url                                                             |
| attach         | string | false | 附加参数 json字符串                                                           |
| user_ip        | string | false | 收款用户 IP                                                                |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                         |
| timestamp      | string | false | 下单时间戳 10位时间戳单位S秒                                                       |

-  代付-attach 附加参数字段说明

| 名称       | 类型     | 必填   | 描述             |
|----------|--------|------|----------------|
| fps_id |string| false | FPS 标识 |
| bank_name | string | false | 银行名称 |
| branch_code | string | false | 分行编码 |


- 代付-请求参数示例

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"fps_id\":\"john.doe@fps\",\"bank_name\":\"HSBC Hong Kong\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "203.198.1.1",
  "account_type": "FPS",
  "account_no": "1234567890",
  "account_name": "John Doe",
  "bank_code": "HK_HSBC",
  "identify_type": "HKID",
  "identify_num": "A123456(7)"
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
  "dis_order_no": "2025071130776296733810688hkg1Dhr7H",
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
| order_price        | int    | true  | 订单金额,单位:港仙（Cents）                                     |
| fee          | int    | false | 订单手续费 ,单位:港仙（Cents）                         |
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
  "dis_order_no": "Meg2352644o2nmjo0800hkgYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": "{\"utr2\":\"611011445289\"}"
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
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">真实付款金额 ,单位:港仙（Cents）</span>                                                                                                                              |
| status       | int    | true  | 订单状态, 1.未支付, <span style="color:red;">2.成功</span>, 3.失败 , 7.驳回 9.冲正  10:处理中                                                                                                       |
| success_time | int    | true  | 成功时间戳                                                                                                                                             |
| order_no     | string | true  | 商户订单号                                                                                                                                             |
| dis_order_no | string | true  | 平台订单号                                                                                                                                             |
| remark       | string | true  | 代付失败原因                                                                                                                                            |
| fee          | int    | false | 订单手续费 ,单位:港仙（Cents）                                                                                                                               |
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
  "payer": "{\"account_name\":\"John Doe\",\"account_type\":\"FPS\",\"account_no\":\"1234567890\",\"identify_type\":\"HKID\",\"identify_num\":\"A123456(7)\"}",
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
| balance | int    | true | 余额,单位:港仙（Cents）       |
| balance_frozen | int    | false | 冻结余额,单位:港仙（Cents） |
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
  "dis_order_no": "35011C02gljuf6k0800hkg1lVY",
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


# 11、支付方式 代收字段 pay_method


| pay_method | 地区   | 值             | 描述           |
|------------|--------|---------------|----------------|
| pay_method        | 香港   | FPS        | FPS Payment            |
| pay_method        | 香港   | BANK_HK        | Bank Transfer            |


# 12、银行编码

| 字段名称 | 编码 | 银行名称 |
| :--- | :--- | :--- |
| bank_code | BETPAY | BetPay |
| bank_code | ZSK | 转数快 |
| bank_code | ZDBXG | 渣打銀行(香港)有限公司 |
| bank_code | XGSHHF | 香港上海匯豐銀行有限公司 |
| bank_code | DFHL | 東方匯理銀行 |
| bank_code | HQB | 花旗銀行 |
| bank_code | MGDT | 摩根大通銀行 |
| bank_code | GMXMS | 國民西敏寺資本市場銀行有限公司 |
| bank_code | ZGJSB | 中國建設銀行(亞洲)股份有限公司 |
| bank_code | ZGB | 中國銀行(香港)有限公司 |
| bank_code | DYB | 東亞銀行有限公司 |
| bank_code | XZB | 星展銀行(香港)有限公司 |
| bank_code | ZXB | 中信銀行國際有限公司 |
| bank_code | ZSYL | 招商永隆銀行有限公司 |
| bank_code | HUAQB | 華僑銀行 |
| bank_code | HSB | 恒生銀行有限公司 |
| bank_code | SHY | 上海商業銀行有限公司 |
| bank_code | JTB | 交通銀行股份有限公司 |
| bank_code | DZB | 大眾銀行(香港)有限公司 |
| bank_code | HQYH | 華僑永亨銀行有限公司 |
| bank_code | DAYB | 大有銀行有限公司 |
| bank_code | JIYB | 集友銀行有限公司 |
| bank_code | DXB | 大新銀行有限公司 |
| bank_code | CXB | 創興銀行有限公司 |
| bank_code | NYSY | 南洋商業銀行有限公司 |
| bank_code | UCOB | UCOBANK |
| bank_code | KEBHANABANK | KEBHANABANK |
| bank_code | SLUFJ | 三菱UFJ銀行 |
| bank_code | PANGB | 盤谷銀行 |
| bank_code | YDHY | 印度海外銀行 |
| bank_code | DYZB | 德意志銀行 |
| bank_code | MGB | 美國銀行 |
| bank_code | FZBL | 法國巴黎銀行 |
| bank_code | YDB | 印度銀行 |
| bank_code | BJGM | 巴基斯坦國民銀行 |
| bank_code | DSB | 大生銀行有限公司 |
| bank_code | MLB | 馬來亞銀行 |
| bank_code | SJZY | 三井住友銀行 |
| bank_code | YNGB | 印尼國家銀行 |
| bank_code | JYB | 金融銀行有限公司 |
| bank_code | DAHB | 大華銀行有限公司 |
| bank_code | ZGGSB | 中國工商銀行(亞洲)有限公司 |
| bank_code | BARCLAYSBANKPLC | BARCLAYSBANKPLC. |
| bank_code | JNDFY | 加拿大豐業銀行 |
| bank_code | JNDHJ | 加拿大皇家銀行 |
| bank_code | FGXY | 法國興業銀行 |
| bank_code | YDGB | 印度國家銀行 |
| bank_code | DLDDM | 多倫多道明銀行 |
| bank_code | MDKB | 滿地可銀行 |
| bank_code | JNDDG | 加拿大帝國商業銀行 |
| bank_code | DGSY | 德國商業銀行 |
| bank_code | RSB | 瑞士銀行 |
| bank_code | MGHF | 美國滙豐銀行 |
| bank_code | RUISB | 瑞穗銀行 |
| bank_code | DGZY | 德國中央合作銀行 |
| bank_code | YOULB | 友利銀行 |
| bank_code | PHILIPPINENATIONALBANK | PHILIPPINENATIONALBANK |
| bank_code | FUBB | 富邦銀行(香港)有限公司 |
| bank_code | SLUFJXT | 三菱UFJ信託銀行 |
| bank_code | NYMLB | 紐約梅隆銀行有限公司 |
| bank_code | INGBANKN | INGBANKN.V. |
| bank_code | XBYDY | 西班牙對外銀行 |
| bank_code | ADLYG | 澳大利亞國民銀行 |
| bank_code | XTPY | 西太平洋銀行 |
| bank_code | AOXINB | 澳新銀行集團有限公司 |
| bank_code | AZLB | 澳洲聯邦銀行 |
| bank_code | YDLLH | 義大利聯合聖保羅銀行股份有限公司 |
| bank_code | YUXB | 裕信(德國)銀行股份有限公司 |
| bank_code | RDSY | 瑞典商業銀行 |
| bank_code | QIANYB | 千葉銀行 |
| bank_code | BLSLH | 比利時聯合銀行 |
| bank_code | FUGUO | 富國銀行香港分行 |
| bank_code | HELAN | 荷蘭合作銀行 |
| bank_code | XINZHAN | 星展銀行香港分行 |
| bank_code | JINW | 靜岡銀行 |
| bank_code | BASER | 八十二銀行 |
| bank_code | HUANSY | 華南商業銀行股份有限公司 |
| bank_code | ZIHB | 滋賀銀行 |
| bank_code | YIWAN | 臺灣銀行股份有限公司 |
| bank_code | THECHUGOKUBANKLIMITED | THECHUGOKUBANKLIMITED |
| bank_code | DYSY | 第一商業銀行股份有限公司 |
| bank_code | ZHSY | 彰化商業銀行股份有限公司 |
| bank_code | FGWM | 法國外貿銀行 |
| bank_code | ZGGS | 中國工商銀行股份有限公司 |
| bank_code | MGDF | 美國道富銀行 |
| bank_code | ZGJS | 中國建設銀行股份有限公司 |
| bank_code | ZGNY | 中國農業銀行股份有限公司 |
| bank_code | ERSTEGROUPBANKAG | ERSTEGROUPBANKAG |
| bank_code | ZGXT | 中國信託商業銀行股份有限公司 |
| bank_code | YWZXQY | 臺灣中小企業銀行股份有限公司 |
| bank_code | CREDITSUISSEAG | CREDITSUISSEAG |
| bank_code | GTSHSY | 國泰世華商業銀行股份有限公司 |
| bank_code | RSYF | 瑞士盈豐銀行股份有限公司 |
| bank_code | ZSYB | 招商銀行股份有限公司 |
| bank_code | TBFBSY | 台北富邦商業銀行股份有限公司 |
| bank_code | YFSYB | 永豐商業銀行股份有限公司 |
| bank_code | ZFGJSY | 兆豐國際商業銀行 |
| bank_code | YSSYB | 玉山商業銀行股份有限公司 |
| bank_code | TXGJSY | 台新國際商業銀行股份有限公司 |
| bank_code | FLYB | 豐隆銀行有限公司 |
| bank_code | ZDB | 渣打銀行 |
| bank_code | HQXGB | 花旗銀行(香港)有限公司 |
| bank_code | ICICIBANKLIMITED | ICICIBANKLIMITED |
| bank_code | MELLIBANKPLC | MELLIBANKPLC |
| bank_code | HUAMB | 華美銀行 |
| bank_code | BALD | 巴魯達銀行 |
| bank_code | YDGJSY | 遠東國際商業銀行股份有限公司 |
| bank_code | CANARABANK | CANARABANK |
| bank_code | GUOTB | 國泰銀行 |
| bank_code | TWTDB | 台灣土地銀行股份有限公司 |
| bank_code | HZJKSY | 合作金庫商業銀行股份有限公司 |
| bank_code | PUNJABNATIONALBANK | PUNJABNATIONALBANK |
| bank_code | XBYSTD | 西班牙桑坦德銀行有限公司 |
| bank_code | UNIONBANKOFINDIA | UNIONBANKOFINDIA |
| bank_code | SHSYCX | 上海商業儲蓄銀行股份有限公司 |
| bank_code | INDUSTRIALBANKOFKOREA | INDUSTRIALBANKOFKOREA |
| bank_code | XJPB | 新加坡銀行有限公司 |
| bank_code | SHINHANBANK | SHINHANBANK |
| bank_code | WDSY | 王道商業銀行股份有限公司 |
| bank_code | BNPPARIBASSECURITIESSERVICES | BNPPARIBASSECURITIESSERVICES |
| bank_code | GJKFB | 國家開發銀行 |
| bank_code | FIRSTABUDHABIBANKPJSC | FIRSTABUDHABIBANKPJSC |
| bank_code | SAFRASARASINLTD | BANKJ.SAFRASARASINLTD. |
| bank_code | ABNAMROBANKN | ABNAMROBANKN.V. |
| bank_code | HDFCBANKLIMITED | HDFCBANKLIMITED |
| bank_code | UNIONBANCAIREPRIVEE | UNIONBANCAIREPRIVEE,UBPSA |
| bank_code | SKANDINAVISKAENSKILDABANKENAB | SKANDINAVISKAENSKILDABANKENAB |
| bank_code | BANKJULIUSBAER | BANKJULIUSBAER&CO.LTD. |
| bank_code | CREDITINDUSTRIELETCOMMERCIAL | CREDITINDUSTRIELETCOMMERCIAL |
| bank_code | YWXGSY | 臺灣新光商業銀行股份有限公司 |
| bank_code | ZGYHXG | 中國銀行香港分行 |
| bank_code | SWITZERLAND | CAINDOSUEZ(SWITZERLAND)SA |
| bank_code | ICBCSTANDARDBANKPLC | ICBCSTANDARDBANKPLC |
| bank_code | LGTHJXG | LGT皇家銀行(香港) |
| bank_code | MGLB | 麥格理銀行有限公司 |
| bank_code | SHPDFZ | 上海浦東發展銀行股份有限公司 |
| bank_code | ZGMSB | 中國民生銀行股份有限公司 |
| bank_code | PICTET&CIE | PICTET&CIE(EUROPE)S.A. |
| bank_code | GFB | 廣發銀行股份有限公司 |
| bank_code | BOHB | 渤海銀行股份有限公司 |
| bank_code | ZGGD | 中國光大銀行股份有限公司 |
| bank_code | SJZYXT | 三井住友信託銀行 |
| bank_code | SHXGB | 上海銀行(香港)有限公司 |
| bank_code | CIMBBANKBERHAD | CIMBBANKBERHAD |
| bank_code | XINYEB | 興業銀行股份有限公司 |
| bank_code | YDSY | 元大商業銀行股份有限公司 |
| bank_code | MASHREQBANK | MASHREQBANK-PUBLICSHAREHOLDINGCOMPANY |
| bank_code | KOOKMINBANK | KOOKMINBANK |
| bank_code | JTXG | 交通銀行(香港)有限公司 |
| bank_code | ZSYHB | 浙商銀行股份有限公司 |
| bank_code | MGSDL | 摩根士丹利銀行亞洲有限公司 |
| bank_code | PINANB | 平安銀行股份有限公司 |
| bank_code | HUAXIA | 華夏銀行股份有限公司 |
| bank_code | ZANB | 眾安銀行有限公司 |
| bank_code | LIVIVBLIMITED | LIVIVBLIMITED |
| bank_code | MOXBANKLIMITED | MOXBANKLIMITED |
| bank_code | WELABBANKLIMITED | WELABBANKLIMITED |
| bank_code | FRB | 富融銀行有限公司 |
| bank_code | PAYZHT | 平安壹賬通銀行(香港)有限公司 |
| bank_code | MYXG | 螞蟻銀行(香港)有限公司 |
| bank_code | QATARNATIONALBANK | QATARNATIONALBANK(Q.P.S.C.) |
| bank_code | TXB | 天星銀行有限公司 |
| bank_code | HONGKONGSECURITIESCLEARINGCOMPANYLIMITED | HONGKONGSECURITIESCLEARINGCOMPANYLIMITED |
| bank_code | CLSBANKINTERNATIONAL | CLSBANKINTERNATIONAL |
| bank_code | BANQUE | BANQUE PICTET & CIE SA |
| bank_code | DGB | 東莞銀行股份有限公司 |
| bank_code | NXB | 農協銀行 |
| bank_code | WECHAT | WeChat Pay Hong Kong Limited |
| bank_code | ALIPAY | Alipay Financial Services(HK) Limited |
| bank_code | BDTK | 八達通卡有限公司 |
| bank_code | LHB | 388 LIVI BANK LIMITED |

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

| 名称           | 类型     | 必填    | 描述                        |
|--------------|--------|-------|---------------------------|
| app_id       | string | true  | 商户app_id                    |
| order_no       | string | true  | 商户订单号                     |
| amount       | string | true  | 商户金额单位港元（HKD）                     |
| notice_url       | string | false | 异步通知地址                     |
| pay_code       | int    | true | 产品编码                     |

#### 示例

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{商户订单号}}&amount={{商户金额}}&notice_url={{异步通知地址}}&pay_code={{产品编码}}
```
---
# 15、文档更新时间
```
2026-08-21 00:00:00
```
