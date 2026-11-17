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
> 2、金额单位为<span style="color:red;"> 分 </span>。
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

| 名称             | 类型   | 必填  | 描述                                                              |
|----------------|------|-----|-----------------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                             |
| app_id         | int    | true  | 商户 appId                                                        |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                   |
| pay_method     | string    | true  | 支付方式   参照支付方式字典                                                 |
| price          | int    | true  | 下单金额,单位:分 ,整数  转元后不能有小数点                                        |
| order_no       | string | true  | 商户订单号                                                           |
| success_url    | string | false | 支付成功跳转 url                                                      |
| fail_url       | string | false | 支付失败跳转 url                                                      |
| pay_notice_url | string | false | 支付成功通知 url                                                      |
| user_id        | string | true | 系统用户ID                                                          |
| user_ip        | string | true | 付款人 IP                                                          |
|attach|string|false| 附加参数 json字符串 付款人信息（建议填写）{”name”:”姓名”,"phone":"电话","email":"邮箱"} |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                  |
|timestamp|string|false| 下单时间戳 10位时间戳单位S                                                 |

-  代收-attach 附加参数字段说明

| 名称    | 类型     | 必填  | 描述      |
|-------|--------|-----|---------|
| name  | string | false  | 付款人姓名   |
| phone | string    | false  | 付款人电话号码 |
| email | string    | false  | 付款人邮箱号码 |


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

## 4.2  代收-下单响应

| 名称         | 类型   | 必填 | 描述                                                |
|------------|------|----|---------------------------------------------------|
| code         | int    | true | 200:下单成功 其他:下单失败                                  |
| msg          | string | true | 失败原因                                              |
| pay_url      | string | false | 付款链接                                              |
| qr_code      | string | false | pix 二维码字符串                                        |
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
| order_price  | int    | true  | 订单金额,单位:分                                                                                              |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">用户真实付款金额 ,单位:分</span>                                                                                         |
| nti_time     | int    | false | 发起通知时间                                                                                                 |
| payer        | string | false | JSON 字符串,付款人信息{"name":"姓名","account":"账号","bank":"付款的用户银行编码","utr2":"银行流水号","email":"邮箱","phone":"手机号","identify_type":"证件类型","identify_num":"RFC,CURP"}，除示例字段外，当前参数会整合商户传递的attach里付款人信息相关字段|
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等  25-10-28 |
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
  "payer": "{\"name\":\"Juan Perez\",\"email\":\"juan.perez@example.mx\",\"phone\":\"5512345678\",\"identify_type\":\"RFC\",\"identify_num\":\"XAXX010101000\"}",
  "nti_time": 1752826164,
  "create_time": 1752751502,
  "sign": "eba7f27e0f49581d8784294ef29f994d"
}
```
## 5.2 代收回调-响应说明
回调接收处理成功，请返回 <span style="color:red;">success</span>，系统将不再推送此订单信息，否则还会重复推送多次

## 5.3 代收重复扫码回调说明

适用场景：

- 用户重复使用历史二维码进行付款
- 商户侧未重新发起新的代收下单
- 商户后续可能再次收到该笔商户订单号对应的成功回调

在该场景下，商户会收到一笔新的代收成功回调。

回调规则：

- `order_no` 仍返回商户原始订单号
- `dis_order_no` 返回本次回调对应的平台订单号
- `status` 按本次支付结果返回，成功时为 `2`
- `order_price` 为原订单金额
- `real_price` 为本次实际付款金额

商户处理建议：

- 请将 `dis_order_no` 视为平台侧唯一订单号
- 若同一个 `order_no` 收到多次成功回调，请按 `dis_order_no` 做幂等和入账区分
- 查询订单时，建议优先使用 `dis_order_no` 查询对应支付结果

示例说明：

```text
第一次下单：
  商户订单号 order_no     = M001
  平台订单号 dis_order_no = SN20260607000001

后续再次收到成功回调时：
  商户订单号 order_no     = M001
  新平台订单号 dis_order_no = SN20260607000002
```

重复扫码成功回调示例：

```json
{
  "trade_no": 10003,
  "status": 2,
  "order_no": "M001",
  "dis_order_no": "SN20260607000002",
  "order_price": 10099,
  "real_price": 10099,
  "payer": "{\"name\":\"姓名\",\"account\":\"账号\",\"bank\":\"银行编码\",\"utr2\":\"银行流水号\"}",
  "nti_time": 1752826164,
  "create_time": 1752751502,
  "sign": "eba7f27e0f49581d8784294ef29f994d"
}
```

# 6、代付下单接口

(下单 ip 需要联系我方加白)
下单地址 https://{api_domain}/api/v1/payApi/CreatePayOutOrder

## 6.1 代付-请求参数

<span style="color:red;">代付未返回明确失败的情况下均认为请求成功，包括超时，即未返回明确失败请不要重复请求。</span>

| 名称             | 类型   | 必填    | 描述                                                    |
|----------------|------|-------|-------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                   |
| order_no       | string | true  | 商户订单号                                                 |
| app_id         | int    | true  | 商户 appId                                              |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                         |
| price          | int    | true  | 下单金额，单位：分，整型。                                         |
| account_no     | string | true  | 收款账号                                                  |
| account_type   | string | true  | 账号类型:CLABE（clabe号）、BANK(银行账号)                         |
| account_name   | string | true  | 姓名                                                    |
| bank_code      | string | true  | 收款银行代码  参照银行编码                                          |
| identify_type  | string | false | 证件类型: 墨西哥（RFC,CURP）                                   |
| identify_num   | string | false | 证件号码                                                  |
| pay_notice_url | string | false | 代付成功通知 url                                            |
| attach         | string | true  | 附加参数   {"email":"邮箱","phone":"手机号","bank_name":"银行名称"} |
| user_ip        | string | true  | 收款用户 IP                                               |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                        |
|timestamp|string| false | 下单时间戳 10位时间戳单位S                                       |

- 代付-请求参数示例

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"email\":\"邮箱\",\"phone\":\"手机号\",\"bank_name\":\"银行名称\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "87.200.59.100",
  "account_type": "CLABE",
  "account_no": "032180000118359719",
  "account_name": "Juan Perez",
  "bank_code": "BANAMEX",
  "identify_type": "RFC",
  "identify_num": "XAXX010101000"
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

## 6.3 代付反查协议支持（可选）

代付反查为可选能力。商户如需支持，须按以下协议实现订单查询接口，供平台 API 调用。

### 6.3.1 请求与响应

平台 API 使用 `POST` 发送 JSON，商户必须返回 HTTP 200 和平铺的 JSON 对象。商户先校验请求签名和时间戳，再查询自己的订单库。接口只读，不创建订单、不触发打款，同一订单可以查询多次。每次响应都要返回当次 nonce，并重新签名。

请求不传预期金额。双方需要提供的字段如下，各方向列出的字段全部必填。

| 字段 | 类型 | 请求 | 响应 |
| --- | --- | --- | --- |
| `trade_no` | int64 | 商户号 | 原样返回 |
| `app_id` | int64 | 商户 App ID | 原样返回 |
| `order_no` | string | 商户订单号 `MerOrderNo` | 查询到的订单号；订单不存在时，返回所查询的单号 |
| `timestamp` | string | Unix 秒，商户校验时差不得超过 300 秒 | 不传 |
| `nonce` | string | 16 字节安全随机数的小写十六进制（32 位） | 原样返回 |
| `price` | int64 | 不传 | 商户记录的原下单金额，单位分（对应币种主单位金额 × 100）；订单不存在时为 `0` |
| `status` | string | 不传 | `EXISTS` 表示存在，`NOT_FOUND` 表示不存在；与支付状态无关 |
| `sign` | string | 请求签名 | 对响应字段重新签名 |

以下示例使用公开测试密钥 `demo_merchant_key`。固定时间戳和 nonce 仅用于签名校验演示；实际请求须使用当前时间戳和新生成的安全随机 nonce。

请求：

```json
{
  "trade_no": 10001,
  "app_id": 10002,
  "order_no": "M202611050001",
  "timestamp": "1793836800",
  "nonce": "00112233445566778899aabbccddeeff",
  "sign": "9c87ab59f72fc33050c687b19d0f49d6"
}
```

响应：

```json
{
  "trade_no": 10001,
  "app_id": 10002,
  "order_no": "M202611050001",
  "price": 10000,
  "status": "EXISTS",
  "nonce": "00112233445566778899aabbccddeeff",
  "sign": "8e045eddca1f5c2d22480ceb5681c4ae"
}
```

### 6.3.2 签名与校验

本反查接口以本节签名与校验规则为准。签名时排除 `sign`，其余协议字段全部参与，`price=0` 也不能省略。数值转为十进制整数，字符串保持原值。按字段名升序拼接为 `key=value&key=value`，不做 URL 编码，再追加 `&key=商户密钥`。对拼接结果的 UTF-8 字节计算 MD5，输出 32 位小写十六进制。

平台 API 用响应字段重算签名，采用恒定时间比较。比较对象是重算结果与响应中的签名，不与原下单签名或本次请求签名比较。签名方法不得输出包含密钥的签名原文。

平台 API 按结构、签名、业务字段的顺序校验，全部通过后才继续预检：

- JSON 缺字段、出现 `null`、重复键、未知字段、多个顶层值或类型错误，均判失败。数值必须在 int64 范围内，`price` 只接受 JSON 整数，不接受字符串、小数或指数形式。
- 响应中的商户、App 和 nonce 必须与本次请求一致，商户订单号必须精确相等，存在状态必须为 `status=EXISTS`。
- 金额必须与平台本地原下单金额一致。本地主单 `Amount` 的单位是元（对应币种主单位），使用十进制运算乘以 100，确认结果为正整数且未溢出后再转为 int64。不能用浮点截断，也不能拿实付、到账或手续费金额比较。即使读取了原请求的 `price`，仍须核对主单金额。

# 7、代付回调通知

推送地址:商户下单传送的 pay_notice_url 回调 ip:call_back_server_ip ,请将我方 ip 加入回调白名单

## 7.1 代付回调请求参数

| 名称         | 类型   | 必填  | 描述                                            |
|------------|------|-----|-----------------------------------------------|
| trade_no     | int    | true  | 商户号                                           |
| order_no     | string | true  | 商户订单号                                         |
| dis_order_no | string | true  | 平台订单号                                         |
| order_price        | int    | true  | 订单金额,单位:分                                     |
| fee          | int    | false | 订单手续费 ,单位:分                                   |
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
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">真实付款金额 ,单位:分</span>                                                                                                                                      |
| status       | int    | true  | 订单状态, 1.未支付, <span style="color:red;">2.成功</span>, 3.失败 , 7.驳回 9.冲正  10:处理中                                                                                                       |
| success_time | int    | true  | 成功时间戳                                                                                                                                             |
| order_no     | string | true  | 商户订单号                                                                                                                                             |
| dis_order_no | string | true  | 平台订单号                                                                                                                                             |
| remark       | string | true  | 代付失败原因                                                                                                                                            |
| fee          | int    | false | 订单手续费 ,单位:分                                                                                                                                       |
| create_time  | int    | true  | 创建时间                                                                                                                                              |
| payer        | string | false | JSON 字符串,付款人信息{"account_name":"姓名","account_type":"账号类型:CLABE,BANK","account_no":"账号","identify_type":"证件类型","identify_num":"RFC,CURP"} |
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等  25-10-28 |
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
  "payer": "{\"account_name\":\"Juan Perez\",\"account_type\":\"CLABE\",\"account_no\":\"032180000118359719\",\"identify_type\":\"RFC\",\"identify_num\":\"XAXX010101000\"}",
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
| balance | int    | true | 余额,单位:分                |
| balance_frozen | int    | false | 冻结余额,单位:分                |
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


# 11、支付方式 代收字段 pay_method

| 字段      | 国家   | 值     | 描述       |
|-----------|------|-------|----------|
| pay_method | 墨西哥   | CLABE | 墨西哥CLABE |
| pay_method | 墨西哥   | OXXO  | 墨西哥OXXO  |


# 12、银行编码

| 字段名称      | 编码              | 银行名称 |
|:----------|:----------------|:---------|
| bank_code | CB_INTERCAM     | CB INTERCAM |
| bank_code | MULTIVA_BANCO   | MULTIVA BANCO |
| bank_code | LIBERTAD        | LIBERTAD |
| bank_code | ARCUS           | ARCUS |
| bank_code | NAFIN           | NAFIN |
| bank_code | MASARI          | MASARI |
| bank_code | CI_BOLSA        | CI BOLSA |
| bank_code | BANORTE         | BANORTE |
| bank_code | BANOBRAS        | BANOBRAS |
| bank_code | BANCREA         | BANCREA |
| bank_code | AKALA           | AKALA |
| bank_code | BANCOMEXT       | BANCOMEXT |
| bank_code | VOLKSWAGEN      | VOLKSWAGEN |
| bank_code | VE_POR_MAS      | VE POR MAS |
| bank_code | MERCADO_PAGO_W  | Mercado Pago W |
| bank_code | INTERCAM_BANCO  | INTERCAM BANCO |
| bank_code | CAJA_TELEFONIST | CAJA TELEFONIST |
| bank_code | CAJA_POP_MEXICA | CAJA POP MEXICA |
| bank_code | BANJERCITO      | BANJERCITO |
| bank_code | ACTINVER        | ACTINVER |
| bank_code | PROFUTURO       | PROFUTURO |
| bank_code | HIPOTECARIA_FED | HIPOTECARIA FED |
| bank_code | CoDi_Valida     | CoDi Valida |
| bank_code | BANCOPPEL       | BANCOPPEL |
| bank_code | TESORED         | TESORED |
| bank_code | INDEVAL         | INDEVAL |
| bank_code | INBURSA         | INBURSA |
| bank_code | CRISTOBAL_COLON | CRISTOBAL COLON |
| bank_code | BAJIO           | BAJIO |
| bank_code | BANXICO         | BANK OF MEXICO |
| bank_code | KUSPIT          | KUSPIT |
| bank_code | CREDIT_SUISSE   | CREDIT SUISSE |
| bank_code | AZTECA          | AZTECA |
| bank_code | AFIRME          | AFIRME |
| bank_code | CIBANCO         | CIBANCO |
| bank_code | BARCLAYS        | BARCLAYS |
| bank_code | SPINOXXO        | Spin in OXXO |
| bank_code | VALUE           | VALUE |
| bank_code | VALMEX          | VALMEX |
| bank_code | REFORMA         | REFORMA |
| bank_code | PAGATODO        | PAGATODO |
| bank_code | MULTIVA_CBOLSA  | MULTIVA CBOLSA |
| bank_code | COMPARTAMOS     | COMPARTAMOS |
| bank_code | JP_MORGAN       | JP MORGAN |
| bank_code | EVERCORE        | EVERCORE |
| bank_code | CREDICAPITAL    | CREDICAPITAL |
| bank_code | BANSI           | BANSI |
| bank_code | BANREGIO        | BANREGIO |
| bank_code | BANKAOOL        | BANKAOOL |
| bank_code | BANK_OF_AMERICA | BANK OF AMERICA |
| bank_code | ABC_CAPITAL     | ABC CAPITAL |
| bank_code | TRANSFER        | TRANSFER |
| bank_code | STP             | STP |
| bank_code | FONDO_FIRA      | FONDO (FIRA) |
| bank_code | FOMPED          | FOMPED |
| bank_code | BANCO_S3        | BANCO S3 |
| bank_code | AUTOFIN         | AUTOFIN |
| bank_code | UNAGRA          | UNAGRA |
| bank_code | HSBC            | HSBC |
| bank_code | DONDE           | DONDE |
| bank_code | CONSUBANCO      | CONSUBANCO |
| bank_code | BMONEX          | BMONEX |
| bank_code | MONEXCB         | MONEXCB |
| bank_code | BBVA_MEXICO     | BBVA MEXICO |
| bank_code | BANSEFI         | BANSEFI |
| bank_code | BANCO_FINTERRA  | BANCO FINTERRA |
| bank_code | BANAMEX         | BANAMEX |
| bank_code | ASP_INTEGRA_OPC | ASP INTEGRA OPC |
| bank_code | SANTANDER       | SANTANDER |
| bank_code | INVEX           | INVEX |
| bank_code | MIFEL           | MIFEL |
| bank_code | INVERCAP        | INVERCAP |
| bank_code | GBM             | GBM |
| bank_code | VECTOR          | VECTOR |
| bank_code | SHINHAN         | SHINHAN |
| bank_code | SCOTIABANK      | SCOTIABANK |
| bank_code | SABADELL        | SABADELL |
| bank_code | MIZUHO_BANK     | MIZUHO BANK |
| bank_code | INMOBILIARIO    | INMOBILIARIO |
| bank_code | BBASE           | BBASE |
| bank_code | MUFG            | MUFG |
| bank_code | FINCOMUN        | FINCOMUN |
| bank_code | FINAMEX         | FINAMEX |
| bank_code | CITI_MEXICO     | CITI MEXICO |
| bank_code | NU_MEXICO       | Nu México Financiera |
| bank_code |  KLAR_MXN               |    Klar Technologies       | 
| bank_code |   FONDEADORA              |  Fondeadora    |
| bank_code |   HEY_BANCO              | HEY BANCO     |
| bank_code | ACCENDO_BANCO | ACCENDO BANCO |
| bank_code | AMERICAN_EXPRESS | AMERICAN EXPRES |
| bank_code | ICBC | ICBC |
| bank_code | BANK_OF_CHINA | BANK OF CHINA |
| bank_code | REVOLUT_BANK | REVOLUT BANK |
| bank_code | ESTRUCTURADORES | ESTRUCTURADORES |
| bank_code | HDI_SEGUROS | HDI SEGUROS |
| bank_code | ALTOR | ALTOR |
| bank_code | CB_INBURSA | C.B. INBURSA |
| bank_code | CREDICLUB | CREDICLUB |
| bank_code | OPORMEX | OPORMEX |
| bank_code | TE_CREEMOS | TE CREEMOS |
| bank_code | SC_PROM_Y_OP | SC PROM Y OP |
| bank_code | CAPITAL_ACTIVO | CAPITAL ACTIVO |
| bank_code | BURSAMETRICA | BURSAMETRICA |
| bank_code | NVIO | NVIO |
| bank_code | CASHI_CUENTA | CASHI CUENTA |
| bank_code | MEXPAGO | MexPago |
| bank_code | ALBO | albo |
| bank_code | CUENCA | Cuenca |
| bank_code | COOPDESARROLLO | COOPDESARROLLO |
| bank_code | DEP_Y_PAG_DIG | Dep y Pag Dig |
| bank_code | CLIP | Clip |
| bank_code | PEIBO | Peibo |
| bank_code | FINCO_PAY | FINCO PAY |
| bank_code | FINTOC | FINTOC |
| bank_code | CLS | CLS |
| bank_code | BANCOBRO | BanCobro |


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

# 14、文档更新时间
```
2026-11-17 20:51:12
```
