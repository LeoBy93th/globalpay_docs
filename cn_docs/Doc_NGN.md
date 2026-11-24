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
> 2、金额单位为<span style="color:red;"> 分（NGN） </span>。
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
| -------------- | ------ | ----- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                                                                                                                                           |
| app_id         | int    | true  | 商户 appId                                                                                                                                                                      |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                                                                                                                                 |
| pay_method     | string | true  | 支付方式                                                                                                                                                               |
| price          | int    | true  | 下单金额,单位:分 ,整数                                                                                                                                                                 |
| order_no       | string | true  | 商户订单号                                                                                                                                                                         |
| success_url    | string | false | 支付成功跳转 url                                                                                                                                                                    |
| fail_url       | string | false | 支付失败跳转 url                                                                                                                                                                    |
| pay_notice_url | string | false | 支付成功通知 url                                                                                                                                                                    |
| user_id        | string | true  | 系统用户ID                                                                                                                                                                        |
| user_ip        | string | true  | 付款人 IP                                                                                                                                                                        |
| attach         | string | true  | 附加参数 json字符串 付款人信息 |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                                                                                                                                |
| timestamp      | string | false | 下单时间戳 10位时间戳单位S                                                                                                                                                               |

- 代收-attach 附加参数字段说明

  | 名称            | 类型     | 必填    | 描述                                               |
    | ------------- | ------ | ----- |--------------------------------------------------|
  | name          | string | true | 付款人姓名                                            |
  | phone         | string | true | 付款人手机号                                           |
  | account_no    | string | false  | 付款账号                                             |
  | account_type  | string | false  | 账号类型:PHONE,BANK(银行帐号)                            |
  | bank_code     | string | false | 银行代码                                     |
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
  "attach": "{\"name\":\"张三\",\"email\":\"zhangsan@example.com\",\"phone\":\"09123456789\",\"account_no\":\"1234567890\",\"account_type\":\"PHONE\",\"bank_code\":\"123456\",\"bank_name\":\"中国银行\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
  }
  ```

## 4.2 代收-下单响应

| 名称           | 类型     | 必填    | 描述                                         |
| ------------ | ------ | ----- | ------------------------------------------ |
| code         | int    | true  | 200:下单成功 其他:下单失败                           |
| msg          | string | true  | 失败原因                                       |
| pay_url      | string | false | 付款链接                                       |
| qr_code      | string | false | 二维码字符串                                 |
| order_no     | string | true  | 商户订单号                                      |
| dis_order_no | string | true  | 平台订单号                                      |
| create_time  | int    | true  | 创建时间                                       |
| pay_info     | string | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 |
| sign         | string | true  | 签名结果,签名方法在文档顶部                             |


- 代收-pay_info 附加参数字段说明

  | 名称      | 类型     | 必填    | 描述                 |
    |---------|--------|-------|--------------------|
  | acc_no  | string | false | 收款账户               |
  | bank    | string | false | 银行信息               |  
  | memo    | string | false | 收款描述               |  
  | name    | string | false | 收款人                |  
  | pay_raw | string | false | 支付原生信息，商户可自行转化为二维码 |

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
  "pay_url": "https://api.sunpayinr.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5",
  "pay_info": "{\"acc_no\":\"96311*****2701\",\"bank\":\"NG000016\",\"memo\":\"B4VGR8dfkdalds\",\"name\":\"MR CHI****** OKAFOR\",\"pay_raw\":\"00020101021238580010A00000072701280006970418011496311435212701**************************\"}"
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
| payer        | string | false | JSON 字符串,付款人信息{"name":"姓名","account":"账号","bank":"付款的用户银行编码","utr2":"银行流水号","email":"邮箱","phone":"手机号","identify_type":"证件类型","identify_num":"CPF,CNPJ"}，除示例字段外，当前参数会整合商户传递的attach里付款人信息相关字段 |
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
  "payer": "{\"name\":\"姓名\",\"email\":\"邮箱\",\"phone\":\"手机号\",\"identify_type\":\"证件类型\",\"identify_num\":\"CPF,CNPJ\"}",
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

| 名称             | 类型     | 必填    | 描述                                                   |
| -------------- | ------ | ----- |------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                  |
| order_no       | string | true  | 商户订单号                                                |
| app_id         | int    | true  | 商户 appId                                             |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                        |
| price          | int    | true  | 下单金额,单位:分 ,整数 转元后不能有小数点                              |
| account_no     | string | true  | 收款账号                                                 |
| account_type   | string | true  | 账号类型:PHONE,BANK(银行帐号)                    |
| account_name   | string | true  | 姓名                                                   |
| bank_code      | string | true  | 收款银行代码 参照银行编码                                        |
| pay_notice_url | string | false | 代付成功通知 url                                           |
| attach         | string | false | 附加参数 {"email":"邮箱","phone":"手机号","bank_name":"银行名称"} |
| user_ip        | string | true  | 收款用户 IP                                              |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                       |
| timestamp      | string | false | 下单时间戳 10位时间戳单位S                                      |

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
  "account_no": "1234567890123",
  "account_name": "Chinedu Okafor",
  "bank_code": "NG000014"
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

| 名称           | 类型     | 必填    | 描述                                          |
| ------------ | ------ | ----- | ------------------------------------------- |
| trade_no     | int    | true  | 商户号                                         |
| order_no     | string | true  | 商户订单号                                       |
| dis_order_no | string | true  | 平台订单号                                       |
| order_price  | int    | true  | 订单金额,单位:分                                   |
| fee          | int    | false | 订单手续费 ,单位:分                                 |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">真实代付出款金额 (代付成功时才有) 如需使用real_price字段，请与工作人员联系配置</span>                                                                                       |
| status       | int    | true  | 订单状态, <span style="color:red;">2.代付成功</span>, 3.代付失败, 7.驳回 9.冲正             |
| pay_info     | string | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 25-10-28 |
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
| payer        | string | false | JSON 字符串,付款人信息{"account_name":"姓名","account_type":"账号类型:PHONE,BANK","account_no":"账号","bank_code":"银行编码"} |
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
  "payer": "{\"account_name\":\"Chinedu Okafor\",\"account_type\":\"BANK\",\"account_no\":\"1234567890123\",\"bank_code\":\"NG000014\"}",
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


# 11、支付方式 代收字段 pay_method


| pay_method | 国家   | 值          | 描述            |
|------------|--------|------------|---------------|
| pay_method     | 尼日利亚 | NET_BANKING   | NET BANKING         |


# 12、银行编码
| 字段名称  | 编码       | 银行名称                                               |
|:----------|:-----------|:-------------------------------------------------------|
| bank_code | NG_MFB     | 5TT MFB                                                |
| bank_code | NG_FCL     | 78 FINANCE COMPANY LIMITED                             |
| bank_code | NG_PSB     | 9 PSB                                                  |
| bank_code | NG_9JP     | 9jaPay                                                 |
| bank_code | NG050005   | AAA FINANCE                                            |
| bank_code | NG_AMEB    | AB MICROFINANCE BANK                                   |
| bank_code | NG070010   | ABBEY MORTGAGE BANK                                    |
| bank_code | NG090260   | ABOVE ONLY MICROFINANCE BANK                           |
| bank_code | NG_ASMB    | ABSU Microfinance Bank                                 |
| bank_code | NG_AUMB    | ABU MICROFINANCE BANK                                  |
| bank_code | NG_ACMB    | Abucoop Microfinance BANK                              |
| bank_code | NG_AMBL    | ABULESORO MICROFINANCE BANK LTD                        |
| bank_code | NG000014   | Access Bank                                            |
| bank_code | NG000005   | Access Bank (Diamond)                                  |
| bank_code | NG100052   | ACCESS Y'ello & Beta                                   |
| bank_code | NG_A       | ACCESSMONEY                                            |
| bank_code | NG_ACM     | ACCION MFB                                             |
| bank_code | NG_AMMM    | Ada MFB                                                |
| bank_code | NG_AMAB    | ADAMAWA MORTGAGE BANK                                  |
| bank_code | NG_ASM     | ADDOSSER MFB                                           |
| bank_code | NG_ACSMB   | ADEYEMI COLLEGE STAFF MICROFINANCE BANK                |
| bank_code | NG_AVM     | Advancly MFB                                           |
| bank_code | NG_ALFM    | ADVANS LA FAYETTE MFB                                  |
| bank_code | NG_ALMM    | Aella MFB                                              |
| bank_code | NG_AFMM    | Afekhafe MFB                                           |
| bank_code | NG_AMFB    | Afemai Microfinance Bank                               |
| bank_code | NG_ANP     | AFRIBANK NIGERIA PLC                                   |
| bank_code | NG_AMBP    | AG MORTGAGE BANK PLC                                   |
| bank_code | NG_AGMB    | AGOSASA MICROFINANCE BANK                              |
| bank_code | NG_AKM     | Akalabo MFB                                            |
| bank_code | NG_APMB    | Akpo Microfinance Bank                                 |
| bank_code | NG_ASUM    | AKSU MFB                                               |
| bank_code | NG_AKUM    | Aku MFB                                                |
| bank_code | NG_AMFBL   | AKUCHUKWU MICROFINANCE BANK LTD                        |
| bank_code | NG_ABM     | AL-BARKAH MFB                                          |
| bank_code | NG_AKMB    | ALEKUN MICROFINANCE BANK                               |
| bank_code | NG_ALM     | ALERT MFB                                              |
| bank_code | NG_AHM     | ALHAYAT MFB                                            |
| bank_code | NG_AWM     | ALLWORKERS MFB                                         |
| bank_code | NG_ALMB    | ALLY MICROFINANCE BANK                                 |
| bank_code | NG_AHMB    | ALPHA MORGAN BANK                                      |
| bank_code | NG_APKM    | ALPHAKAPITAL MFB                                       |
| bank_code | NG000037   | Alternative Bank Limited                               |
| bank_code | NG090489   | Alvana Microfinance BANK                               |
| bank_code | NG_AMF     | AMAANAH FINANCE                                        |
| bank_code | NG_AMMB    | Amac Microfinance BANK                                 |
| bank_code | NG090180   | AMJU MFB                                               |
| bank_code | NG_ALFB    | AMML MFB                                               |
| bank_code | NG_AOM     | AMOYE MFB                                              |
| bank_code | NG_AEM     | Ample MFB                                              |
| bank_code | NG_AF      | AMS FINANCE                                            |
| bank_code | NG_ACGM    | Anchorage MFB                                          |
| bank_code | NG_AIM     | Aniocha MFB                                            |
| bank_code | NG_AOMM    | ANIOMA MFB                                             |
| bank_code | NG_ARFB    | APEKS MICROFINANCE BANK                                |
| bank_code | NG_ATM     | Apex Trust MFB                                         |
| bank_code | NG_APRB    | APPLE MICROFINANCE BANK                                |
| bank_code | NG_ARMB    | Aramoko Microfinance Bank                              |
| bank_code | NG_ARCM    | ARCPAY Microfinance                                    |
| bank_code | NG_ARIM    | ARISE MFB                                              |
| bank_code | NG_ARMM    | ARM MFB                                                |
| bank_code | NG_AFCL    | ASCENSIA FINANCE COMPANY LIMITED                       |
| bank_code | NG_ASL     | Aso Savings and Loans                                  |
| bank_code | NG_ACBL    | ASPIRE MICROFINANCE BANK LTD                           |
| bank_code | NG_ARBL    | ASPIRE MORTGAGE BANK LIMITED                           |
| bank_code | NG_AMM     | Assets Matrix MFB                                      |
| bank_code | NG_ATMB    | ASSETS Microfinance BANK                               |
| bank_code | NG_ASTM    | ASTRAPOLARIS MFB                                       |
| bank_code | NG_ABMB    | ATBU Microfinance BANK                                 |
| bank_code | NG_ACCB    | AUCHI MICROFINANCE BANK                                |
| bank_code | NG_APM     | AUCHI POLY MFB                                         |
| bank_code | NG_AMMBL   | AVE MARIA MICROFINANCE BANK LTD                        |
| bank_code | NG_AVBM    | Avuenegbe MFB                                          |
| bank_code | NG_AVCM    | AVVIC MFB                                              |
| bank_code | NG090542   | Awacash Microfinance Bank                              |
| bank_code | NG_AM      | Awe MFB                                                |
| bank_code | NG090662   | Awesome MFB                                            |
| bank_code | NG_AMB     | Aztec Microfinance Bank                                |
| bank_code | NG_BCKM    | BABCOCK MFB                                            |
| bank_code | NG_BRMB    | BABURA MICROFINANCE BANK                               |
| bank_code | NG_BGM     | BAIGE MFB                                              |
| bank_code | NG_BCM     | BAINES CREDIT MFB                                      |
| bank_code | NG_BFBL    | BALERA MICROFINANCE BANK LTD                           |
| bank_code | NG_BFMB    | Balogun Fulani Microfinance BANK                       |
| bank_code | NG090326   | BALOGUN GAMBARI MFB                                    |
| bank_code | NG_BAM     | BAM MFB                                                |
| bank_code | NG_BCFB    | BANC CORP MICROFINANCE BANK                            |
| bank_code | NG_BXMB    | Banex Microfinance BANK                                |
| bank_code | NG_BOA     | BANK OF AGRICULTURE                                    |
| bank_code | NG_BMF     | BANK78 MFB                                             |
| bank_code | NG_BEM     | Bankeasy MFB                                           |
| bank_code | NG_BTM     | BANKIT MFB                                             |
| bank_code | NG_BTB     | Bankit Microfinance Bank                               |
| bank_code | NG_BYB     | Bankly Microfinance Bank                               |
| bank_code | NG_BMFB    | BAOBAB MICROFINANCE BANK                               |
| bank_code | NG_BWM     | BARNAWA MFB                                            |
| bank_code | NG_BYMB    | BAYERO MICROFINANCE BANK                               |
| bank_code | NG_BKM     | BC KASH MFB                                            |
| bank_code | NG_BMM     | Beamer MFB                                             |
| bank_code | NG_BBM     | BELLBANK MFB                                           |
| bank_code | NG_BEMB    | Benysta Microfinance BANK                              |
| bank_code | NG_BHMB    | Berachah Microfinance Bank                             |
| bank_code | NG_BSM     | Beststar MFB                                           |
| bank_code | NG_BLM     | BETHEL MFB                                             |
| bank_code | NG_BMEB    | BIPC MICROFINANCE BANK                                 |
| bank_code | NG_BPMB    | BISHOPGATE MICROFINANCE BANK                           |
| bank_code | NG_BNM     | BKD NEW MFB                                            |
| bank_code | NG_BOCM    | BLOC MFB                                               |
| bank_code | NG_BMSM    | BLOOMS MFB                                             |
| bank_code | NG_BIMB    | Blue Investments Microfinance Bank                     |
| bank_code | NG_BWMB    | Bluewhales Microfinance BANK                           |
| bank_code | NG_BOM     | Bobble MFB                                             |
| bank_code | NG_BTMB    | BOCTRUST MICROFINANCE BANK                             |
| bank_code | NG_BMFK    | BOI MF BANK                                            |
| bank_code | NG_BBMB    | Boji Boji Microfinance BANK                            |
| bank_code | NG_BKSM    | Bokkos MFB                                             |
| bank_code | NG_BDM     | BOLD MFB                                               |
| bank_code | NG_BGMB    | BONGHE MICROFINANCE BANK                               |
| bank_code | NG_BOTM    | Boost MFB                                              |
| bank_code | NG_BUM     | Borgu MFB                                              |
| bank_code | NG_BRM     | Borno Renaissance Microfinance bank                    |
| bank_code | NG_MMFB    | Boromu Microfinance bank                               |
| bank_code | NG_BLMB    | BORSTAL MICROFINANCE BANK                              |
| bank_code | NG_BSKM    | BOSAK MFB                                              |
| bank_code | NG_BWNM    | BOWEN MFB                                              |
| bank_code | NG_BMNM    | Bowman MFB                                             |
| bank_code | NG050006   | Branch International Financial Services                |
| bank_code | NG_BTTB    | Brent Mortgage Bank                                    |
| bank_code | NG_BNMB    | BRETHREN MICROFINANCE BANK                             |
| bank_code | NG_BDMB    | Bridgeway Microfinance BANK                            |
| bank_code | NG_BWYM    | BRIGHTWAY MFB                                          |
| bank_code | NG_BCMB    | Briyth Covenant Microfinance Bank                      |
| bank_code | NG_BMBL    | BROADVIEW MICROFINANCE BANK LTD                        |
| bank_code | NG_BMBD    | BUBAYERO Microfinance Bank                             |
| bank_code | NG_BMBI    | Build Microfinance Bank                                |
| bank_code | NG_BMDI    | BUNDI MFB                                              |
| bank_code | NG_BMK     | BUNKASA MFB                                            |
| bank_code | NG_BMKE    | Bunkure MFB                                            |
| bank_code | NG_BSMB    | Business Support Microfinance BANK                     |
| bank_code | NG_BMB     | BuyPower Microfinance Bank                             |
| bank_code | NG_CMBR    | CALABAR MICROFINANCE BANK                              |
| bank_code | NG_CMBN    | Canaan Microfinance Bank                               |
| bank_code | NG_CSMB    | CAPITALMETRIQ SWIFT MICROFINANCE BANK                  |
| bank_code | NG_CMBT    | CAPSTONE MICROFINANCE BANK                             |
| bank_code | NG_CRB     | CARBON                                                 |
| bank_code | NG_CF      | Cardinalstone Finance                                  |
| bank_code | NG_CMBK    | CARETAKER Microfinance BANK                            |
| bank_code | NG_CMS     | CASA MFB                                               |
| bank_code | NG_CMH     | CASHA MFB                                              |
| bank_code | NG_CMBG    | CASHBRIDGE MICROFINANCE BANK                           |
| bank_code | NG_CMBE    | CASHCONNECT MICROFINANCE BANK                          |
| bank_code | NG_CMR     | Cashrite MFB                                           |
| bank_code | NG_CMBD    | Catland Microfinance BANK                              |
| bank_code | NG_CTA     | CBN_TSA                                                |
| bank_code | NG_RMBL    | CEDAR MICROFINANCE BANK LTD                            |
| bank_code | NG_CMU     | CEDRUS MFB                                             |
| bank_code | NG_CMM     | CEMCS MFB                                              |
| bank_code | NG000028   | Central Bank of Nigeria                                |
| bank_code | NG_CFLT    | CEVIANT FINANCE LIMITED                                |
| bank_code | NG_CB      | Chanelle BANK                                          |
| bank_code | NG_CMBS    | CHARIS MICROFINANCE BANK                               |
| bank_code | NG_CMF     | Chase Microfinance bank                                |
| bank_code | NG_CMBH    | CHERISH MICROFINANCE BANK                              |
| bank_code | NG_CMBZ    | Chibueze Microfinance BANK                             |
| bank_code | NG_CMBM    | CHIKUM MICROFINANCE BANK                               |
| bank_code | NG_CMBY    | Chukwunenye Microfinance BANK                          |
| bank_code | NG000009   | Citibank Nigeria                                       |
| bank_code | NG_CTMN    | Citizen Trust Microfinance BANK Ltd                    |
| bank_code | NG_CCMB    | CITY CODE MORTGAGE BANK                                |
| bank_code | NG_CML     | Cloverleaf MFB                                         |
| bank_code | NG_CMBP    | CoalCamp Microfinance BANK                             |
| bank_code | NG_CMBF    | COASTLINE MICROFINANCE BANK                            |
| bank_code | NG_CMBL    | CONFIDENCE MICROFINANCE BANK LTD                       |
| bank_code | NG_CTMT    | CONSISTENT TRUST MICROFINANCE BANK LTD                 |
| bank_code | NG_CMSM    | CONSUMER MFB                                           |
| bank_code | NG_CMOL    | COOL MFB                                               |
| bank_code | NG_CPMB    | COOP Mortgage BANK                                     |
| bank_code | NG_CMFD    | COOPFUND MFB                                           |
| bank_code | NG_COMB    | Corestep MICROFINANCE BANK                             |
| bank_code | NG_CRN     | Coronation                                             |
| bank_code | NG_CFL     | COUNTY FINANCE LTD                                     |
| bank_code | NG_CMV     | COVENANT MFB                                           |
| bank_code | NG_CAM     | CREDIT AFRIQUE MFB                                     |
| bank_code | NG_CDL     | Credit Direct Limited                                  |
| bank_code | NG_CMLE    | Creditville MFB                                        |
| bank_code | NG_CMT     | Crescent Microfinance bank                             |
| bank_code | NG_CRMB    | CROSS RIVER MICROFINANCE BANK                          |
| bank_code | NG_CMB     | CRUTECH MICROFINANCE BANK                              |
| bank_code | NG_CFCL    | CRYSTAL FINANCE COMPANY LIMITED                        |
| bank_code | NG_CM      | CSD MFB                                                |
| bank_code | NG_DAMB    | DAL Microfinance Bank                                  |
| bank_code | NG_DMH     | DASH MFB                                               |
| bank_code | NG_DMT     | DAVENPORT MFB                                          |
| bank_code | NG_DMBI    | Davodani Microfinance BANK                             |
| bank_code | NG_DMBT    | DAYLIGHT MICROFINANCE BANK                             |
| bank_code | NG_DMG     | DAYSPRING MFB                                          |
| bank_code | NG_DTM     | Delta Trust Mortgage bank                              |
| bank_code | NG_DMY     | DESTINY MFB                                            |
| bank_code | NG_DMI     | Digitvant MFB                                          |
| bank_code | NG_DF      | Dignity Finance                                        |
| bank_code | NG_DMN     | DILLON MFB                                             |
| bank_code | NG_DMB     | DIOBU MICROFINANCE BANK                                |
| bank_code | NG_DMBL    | Doje Microfinance Bank Limited                         |
| bank_code | NG_DMA     | DOT MFB                                                |
| bank_code | NG_DMC     | DSC MFB                                                |
| bank_code | NG_DMD     | DUXBANK MFB                                            |
| bank_code | NG_DM      | DW MFB                                                 |
| bank_code | NG_EME     | E-BARCS MFB                                            |
| bank_code | NG_EB      | E-Finance                                              |
| bank_code | NG_EFM     | EAGLE FLIGHT MFB                                       |
| bank_code | NG_EMF     | EARNWELL MFB                                           |
| bank_code | NG_EL      | Eartholeum                                             |
| bank_code | NG_EMG     | Eastman MFB                                            |
| bank_code | NG_EMBH    | EBSU MICROFINANCE BANK                                 |
| bank_code | NG_EMI     | Ecobank Mobile                                         |
| bank_code | NG000010   | Ecobank Nigeria                                        |
| bank_code | NG_EXA     | Ecobank Xpress Account                                 |
| bank_code | NG_EMJ     | EDFIN MFB                                              |
| bank_code | NG_EMBT    | EGWAFIN MICROFINANCE BANK LTD                          |
| bank_code | NG_EMK     | EJINDU MFB                                             |
| bank_code | NG_EMBR    | EK-Reliable Microfinance BANK                          |
| bank_code | NG_EMBM    | EKIMOGUN MICROFINANCE BANK                             |
| bank_code | NG_EML     | EKONDO MFB                                             |
| bank_code | NG_EMN     | ELLINGTON MFB                                          |
| bank_code | NG_EMBL    | EMAAR MICROFINANCE BANK LTD                            |
| bank_code | NG_EMO     | EMERALDS MFB                                           |
| bank_code | NG_EMBP    | EMPIRETRUST MICROFINANCE BANK                          |
| bank_code | NG000033   | eNaira(e-Naira)                                        |
| bank_code | NG_EFQ     | Enco Finance                                           |
| bank_code | NG_EMBN    | Enrich Microfinance Bank                               |
| bank_code | NG000019   | ENTERPRISE BANK LIMITED                                |
| bank_code | NG_EMR     | Entity MFB                                             |
| bank_code | NG_EMBS    | EQUATOR MICROFINANCE BANK                              |
| bank_code | NG_EMT     | ESAN MFB                                               |
| bank_code | NG_EMBU    | ESO-E MICROFINANCE BANK                                |
| bank_code | NG_        | eTranzact                                              |
| bank_code | NG_EMV     | EVANGEL MFB                                            |
| bank_code | NG_EMBW    | EVERGREEN MICROFINANCE BANK                            |
| bank_code | NG_EF      | EVIB FINANCE                                           |
| bank_code | NG_EMBX    | EWT Microfinance Bank                                  |
| bank_code | NG_EM      | EXCEL MFB                                              |
| bank_code | NG_EMBY    | EXCELLENT MICROFINANCE BANK                            |
| bank_code | NG_EMB     | EYOWO MICROFINANCE BANK                                |
| bank_code | NG_FSCF    | FACTORING AND SUPPLY CHAIN FINANCE LIMITED             |
| bank_code | NG090308   | Fairmoney MFB                                          |
| bank_code | NG_FMBZ    | FAME MICROFINANCE BANK                                 |
| bank_code | NG_FC      | FAST CREDIT                                            |
| bank_code | NG_FMBA    | FAST Microfinance BANK                                 |
| bank_code | NG_FMLB    | FBN MOBILE                                             |
| bank_code | NG060002   | FBNQuest MERCHANT BANK                                 |
| bank_code | NG_FEA     | FCMB Easy Account                                      |
| bank_code | NG_FMC     | FCMB MFB                                               |
| bank_code | NG_FMT     | FCT MFB                                                |
| bank_code | NG_FPNMB   | Federal Polytechnic Nekede Microfinance BANK           |
| bank_code | NG_FN      | FEDERALPOLY NASARAWAMFB                                |
| bank_code | NG_FMH     | Fedeth MFB                                             |
| bank_code | NG_FET     | FETS                                                   |
| bank_code | NG050002   | FEWCHORE FINANCE COMPANY LIMITED                       |
| bank_code | NG_FMBS    | FFS MICROFINANCE BANK                                  |
| bank_code | NG_FMBH    | FHA MORTGAGE BANK LTD                                  |
| bank_code | NG000007   | Fidelity Bank                                          |
| bank_code | NG_FMA     | Fidelity Mobile                                        |
| bank_code | NG_FMFB    | FIDFUND MFB                                            |
| bank_code | NG_FMMS    | FIMS MFB                                               |
| bank_code | NG_FMBC    | FINATRUST MICROFINANCE BANK                            |
| bank_code | NG090400   | Finca Microfinance BANK                                |
| bank_code | NG_FMBD    | Firmus MICROFINANCE BANK                               |
| bank_code | NG_FAMB    | FIRST ALLY MICROFINANCE BANK                           |
| bank_code | NG000016   | First Bank Of Nigeria                                  |
| bank_code | NG000003   | First City Monument Bank                               |
| bank_code | NG_FGMB    | First Generation Mortgage Bank                         |
| bank_code | NG_FHM     | First Heritage MFB                                     |
| bank_code | NG_FMTF    | FIRST MARINA TRUST FINANCE                             |
| bank_code | NG_FMM     | FIRST MULTIPLE MFB                                     |
| bank_code | NG_FOM     | FIRST OPTION MFB                                       |
| bank_code | NG_FRMB    | FIRST ROYAL MICROFINANCE BANK                          |
| bank_code | NG_FMBL    | FIRSTMIDAS MICROFINANCE BANK LTD                       |
| bank_code | NG_FW      | FIRSTMONIE WALLET                                      |
| bank_code | NG_FMBE    | FIRSTTRUST MORTGAGE BANK                               |
| bank_code | NG_FMBF    | FIT MICROFINANCE BANK                                  |
| bank_code | NG_FMG     | Flexi MFB                                              |
| bank_code | NG_FMSH    | FOCUS MFB                                              |
| bank_code | NG_FMI     | Foresight Microfinance bank                            |
| bank_code | NG_FMJ     | FORTIS MFB                                             |
| bank_code | NG_F       | FortisMobile                                           |
| bank_code | NG_FMBK    | Fortress Microfinance BANK                             |
| bank_code | NG400001   | FSDH Merchant Bank                                     |
| bank_code | NG_FML     | FUD MFB                                                |
| bank_code | NG_FRM     | FULL RANGE MFB                                         |
| bank_code | NG_FFS     | FUNDQUEST FINANCIAL SERVICES                           |
| bank_code | NG_FMB     | Futminna Microfinance BANK                             |
| bank_code | NG_FM      | FUTO MFB                                               |
| bank_code | NG_GMBM    | Gabasawa MFB                                           |
| bank_code | NG_GF      | GADOL FINANCE                                          |
| bank_code | NG_GMN     | Garki MFB                                              |
| bank_code | NG_GMMO    | Garun Mallam MFB                                       |
| bank_code | NG_GMBP    | GASHUA MICROFINANCE BANK                               |
| bank_code | NG_GMBQ    | GATEWAY MORTGAGE BANK                                  |
| bank_code | NG_GMBW    | GBEDE Microfinance Bank                                |
| bank_code | NG_GSMX    | Giant Stride MFB                                       |
| bank_code | NG_GAMB    | GIDAUNIYAR ALHERI MICROFINANCE BANK                    |
| bank_code | NG_GMY     | Giginya MFB                                            |
| bank_code | NG_GMBZ    | GiGinya Microfinance BANK                              |
| bank_code | NG_GMA     | Girei MFB                                              |
| bank_code | NG_GMBB    | GIWA MICROFINANCE BANK                                 |
| bank_code | NG_GIM     | GLOBAL INITIATIVE MFB                                  |
| bank_code | NG_GTSL    | GLOBAL TRUST SAVINGS AND LOANS                         |
| bank_code | NG000027   | Globus Bank                                            |
| bank_code | NG_GMC     | GLORY MFB                                              |
| bank_code | NG_GMBD    | GMB Microfinance BANK                                  |
| bank_code | NG_GMBL    | GOLDMAN MICROFINANCE BANK LTD                          |
| bank_code | NG_GME     | GOMBE MFB                                              |
| bank_code | NG_G       | GoMoney                                                |
| bank_code | NG_GNMB    | Good Neighbours Microfinance BANK                      |
| bank_code | NG_GSM     | GOOD SHEPHARD MFB                                      |
| bank_code | NG_GMF     | Gosifechukwu MFB                                       |
| bank_code | NG_GMG     | GOWANS MFB                                             |
| bank_code | NG_GMBH    | Grant Microfinance BANK                                |
| bank_code | NG_GEMBL   | GREEN ENERGY MICROFINANCE BANK LTD                     |
| bank_code | NG_GMI     | Greenacres MFB                                         |
| bank_code | NG_GMJ     | GREENBANK MFB                                          |
| bank_code | NG_GMBK    | GREENVILLE MICROFINANCE BANK                           |
| bank_code | NG_GMBN    | GREENWICH MERCHANT BANK                                |
| bank_code | NG_GFL     | GRIFFIN FINANCE LIMITED                                |
| bank_code | NG_GMBR    | GROOMING MICROFINANCE BANK                             |
| bank_code | NG000013   | GTBank                                                 |
| bank_code | NG_GMML    | GTBank Mobile Money                                    |
| bank_code | NG_GMB     | GTI Microfinance BANK                                  |
| bank_code | NG_GM      | Gwong Microfinance bank                                |
| bank_code | NG_HMBM    | HACKMAN MICROFINANCE BANK                              |
| bank_code | NG_HMBN    | Haggai Mortgage Bank                                   |
| bank_code | NG_HMO     | HalaCredit MFB                                         |
| bank_code | NG_HMP     | HASAL MFB                                              |
| bank_code | NG_HTM     | HAYAT TRUST MFB                                        |
| bank_code | NG_HMQ     | Headway MFB                                            |
| bank_code | NG_HR      | Hedonmark                                              |
| bank_code | NG_HMB     | Highland Microfinance BANK                             |
| bank_code | NG_HM      | HomeBase Mortgage                                      |
| bank_code | NG_H       | HopePSB                                                |
| bank_code | NG_IMBS    | I-MONIE Microfinance Bank                              |
| bank_code | NG_IMT     | IBA MFB                                                |
| bank_code | NG_IMBU    | IBank Microfinance Bank                                |
| bank_code | NG_IMV     | IBBU MFB                                               |
| bank_code | NG_IMBW    | IBETO MICROFINANCE BANK                                |
| bank_code | NG_IMBX    | IBILE MICROFINANCE BANK                                |
| bank_code | NG_IMBY    | IBOLO MICORFINANCE BANK LTD                            |
| bank_code | NG_IMZ     | Ibom fadama Microfinance Bank                          |
| bank_code | NG_IMBA    | Ibom Mortgage Bank                                     |
| bank_code | NG_IMAB    | Ibu-Aje Microfinance                                   |
| bank_code | NG_IG      | IC GLOBALMicrofinance bank                             |
| bank_code | NG_IMAC    | IHIALA MFB                                             |
| bank_code | NG_IMD     | IJARE MFB                                              |
| bank_code | NG_IMBE    | IJEBU-IFE MICROFINANCE BANK LTD                        |
| bank_code | NG_IMF     | IKENNE MFB                                             |
| bank_code | NG_IMG     | IKERE MFB                                              |
| bank_code | NG_IMH     | IKIRE MFB                                              |
| bank_code | NG_IDM     | IKORODU DIVISION MFB                                   |
| bank_code | NG_IIM     | IKOYI ILE MFB                                          |
| bank_code | NG_IMBI    | Ikoyi-Osun Microfinance Bank                           |
| bank_code | NG_IPMBL   | ILARO POLY MICROFINANCE BANK LTD                       |
| bank_code | NG_IMBJ    | ILASAN MICROFINANCE BANK                               |
| bank_code | NG_IMBK    | ILE-OLUJI MICROFINANCE BANK                            |
| bank_code | NG_IMBM    | Ilora Microfinance BANK                                |
| bank_code | NG_IMBN    | ILORIN MICROFINANCE BANK                               |
| bank_code | NG_IML     | ILUTITUN-OSORO MFB                                     |
| bank_code | NG_IMBO    | IMO MICROFINANCE BANK                                  |
| bank_code | NG_IMBP    | Imowo Microfinance BANK                                |
| bank_code | NG_IHMB    | Imperial Homes Mortgage Bank                           |
| bank_code | NG_IMQ     | IMSU MFB                                               |
| bank_code | NG_IMR     | INDULGE MFB                                            |
| bank_code | NG_IS      | INEBA                                                  |
| bank_code | NG_IGMBL   | INEBA GOGO MICROFINANCE BANK LIMITED                   |
| bank_code | NG_IMU     | INFINITY MFB                                           |
| bank_code | NG_IMW     | Infinity trust Mortgage Bank                           |
| bank_code | NG_IMBT    | Insight Microfinance BANK                              |
| bank_code | NG_ILF     | Intellifin                                             |
| bank_code | NG_IMX     | Interland MFB                                          |
| bank_code | NG_IFIS    | Interswitch Financial Inclusion Services (IFIS)        |
| bank_code | NG_IMBV    | Iperu Microfinance BANK                                |
| bank_code | NG_IMBZ    | IRL MICROFINANCE BANK                                  |
| bank_code | NG_IMBB    | ISALEOYO MICROFINANCE BANK                             |
| bank_code | NG_IMBC    | Ishie Microfinance BANK                                |
| bank_code | NG_IUM     | ISI UZO MFB                                            |
| bank_code | NG_IMY     | ISLAND MFB                                             |
| bank_code | NG_IM      | Isuofia MFB                                            |
| bank_code | NG_IMBD    | IWADE MICROFINANCE BANK LTD                            |
| bank_code | NG_IMBF    | IWOAMA MICROFINANCE BANK                               |
| bank_code | NG_IMBL    | IYAMOYE MICROFINANCE BANK LTD                          |
| bank_code | NG_IOMBL   | IYERU OKIN MICROFINANCE BANK LTD                       |
| bank_code | NG_IMB     | Izon Microfinance BANK                                 |
| bank_code | NG000006   | Jaiz Bank                                              |
| bank_code | NG_JMB     | Jessefield Microfinance BANK                           |
| bank_code | NG_JSLL    | JIGAWA SAVINGS AND LOANS LIMITED                       |
| bank_code | NG_J       | JUBILEELIFE                                            |
| bank_code | NG_KMBA    | KABO MICROFINANCE BANK                                 |
| bank_code | NG_KMBB    | KADPOLY MICROFINANCE BANK                              |
| bank_code | NG_KMC     | KADUPE MFB                                             |
| bank_code | NG_KMD     | KAIZEN MFB                                             |
| bank_code | NG_KPM     | KANO POLY MFB                                          |
| bank_code | NG_KWM     | KANO WEST MFB                                          |
| bank_code | NG_KME     | Katagum MFB                                            |
| bank_code | NG_KMF     | KATSU MFB                                              |
| bank_code | NG_KMG     | KAYI MFB                                               |
| bank_code | NG_KMBH    | KAYVEE MICROFINANCE BANK                               |
| bank_code | NG_KMBI    | KC MICROFINANCE BANK                                   |
| bank_code | NG_KMJ     | KCMB MFB                                               |
| bank_code | NG_KSHSA   | KEBBI STATE HOME SAVINGS AND LOANS LIMITED             |
| bank_code | NG_KK      | Kegow                                                  |
| bank_code | NG_KL      | KEGOW(CHAMSMOBILE)                                     |
| bank_code | NG_KMBM    | KENECHUKWU MICROFINANCE BANK                           |
| bank_code | NG000002   | Keystone Bank                                          |
| bank_code | NG_KCMB    | Kingdom College Microfinance BANK                      |
| bank_code | NG_KMBN    | KKU Microfinance Bank                                  |
| bank_code | NG_KMBO    | KOBOWEB MICROFINANCE BANK                              |
| bank_code | NG_KMP     | KOINS MFB                                              |
| bank_code | NG_KMQ     | KOLISA MFB                                             |
| bank_code | NG_KMR     | KOLOMONI MFB                                           |
| bank_code | NG_KP      | KongaPay                                               |
| bank_code | NG_KM      | KONTAGORA MFB                                          |
| bank_code | NG_KMMB    | KREDI MONEY MICROFINANCE BANK                          |
| bank_code | NG090267   | Kuda MFB                                               |
| bank_code | NG_KMB     | KWASU MICROFINANCE BANK                                |
| bank_code | NG_LMBS    | Landgold Microfinance BANK                             |
| bank_code | NG090177   | LAPO MFB                                               |
| bank_code | NG_LMBT    | LAVENDER MICROFINANCE BANK                             |
| bank_code | NG_LMU     | LAWYERS MFB                                            |
| bank_code | NG_LMBV    | LBIC Mortgage Bank                                     |
| bank_code | NG_LMW     | Leadcity MFB                                           |
| bank_code | NG_LMBX    | LEGEND MICROFINANCE BANK                               |
| bank_code | NG_LMBY    | LETSHEGO MICROFINANCE BANK                             |
| bank_code | NG_LMBZ    | LEVERAGE MICROFINANCE BANK                             |
| bank_code | NG_LMA     | LEVITE MFB                                             |
| bank_code | NG_LMBL    | LIFEGATE MICROFINANCE BANK LTD                         |
| bank_code | NG_LMBB    | Light Microfinance BANK                                |
| bank_code | NG_LM      | LIQUIDCREST MFB                                        |
| bank_code | NG_LMBC    | Livingtrust Mortgage Bank                              |
| bank_code | NG_LMBD    | Lobrem Microfinance Bank                               |
| bank_code | NG_LMBE    | LOMA Microfinance Bank                                 |
| bank_code | NG000029   | LOTUS BANK                                             |
| bank_code | NG_LMB     | LOVONUS MICROFINANCE BANK                              |
| bank_code | NG_LFCL    | Lukefield Finance Company Limited                      |
| bank_code | NG_MF      | M36                                                    |
| bank_code | NG_MMG     | MAAL MFB                                               |
| bank_code | NG_MAM     | Mab Allianz MFB                                        |
| bank_code | NG_MMH     | Mabinas MFB                                            |
| bank_code | NG_MMI     | Macrod MFB                                             |
| bank_code | NG_MMBJ    | MADOBI MICROFINANCE BANK                               |
| bank_code | NG_MMK     | Maestro MFB                                            |
| bank_code | NG_MML     | MAHFUZ MFB                                             |
| bank_code | NG_MMBM    | MAINLAND MICROFINANCE BANK                             |
| bank_code | NG_MMN     | MAINSTREET MFB                                         |
| bank_code | NG_MMO     | Maintrust MFB                                          |
| bank_code | NG_MMQ     | MALACHY MFB                                            |
| bank_code | NG_MMBP    | MANNY MICROFINANCE BANK                                |
| bank_code | NG_MMBR    | Mautech Microfinance BANK                              |
| bank_code | NG_MMS     | MAXITRUST MFB                                          |
| bank_code | NG_MMT     | MAYDEN MFB                                             |
| bank_code | NG_MMU     | MAYFAIR MFB                                            |
| bank_code | NG_MMBV    | MAYFRESH MORTGAGE BANK                                 |
| bank_code | NG_MMW     | Medef MFB                                              |
| bank_code | NG_MMBX    | MEGA MICROFINANCE BANK                                 |
| bank_code | NG_MMBY    | MEGAPRAISE MICROFINANCE BANK                           |
| bank_code | NG_MMBZ    | Memphis Microfinance BANK                              |
| bank_code | NG_MMBA    | MERCURY MICROFINANCE BANK                              |
| bank_code | NG_MMBB    | Mgbidi Microfinance Bank                               |
| bank_code | NG_MMBC    | MIA MICROFINANCE BANK                                  |
| bank_code | NG_MMD     | MICROBIZ MFB                                           |
| bank_code | NG_MBME    | MICROFINANCE BANK MEMENT                               |
| bank_code | NG_MMBF    | MICROVIS MICROFINANCE BANK                             |
| bank_code | NG_MMDG    | Midland MFB                                            |
| bank_code | NG_MMBH    | MINJIBIR MICROFINANCE BANK                             |
| bank_code | NG_MMTI    | MINT MFB                                               |
| bank_code | NG_MMJ     | MINT-FINEX MFB                                         |
| bank_code | NG_MMBK    | Mkobo Microfinance Bank                                |
| bank_code | NG_ML      | Mkudi                                                  |
| bank_code | NG_MMM     | MODEL MFB                                              |
| bank_code | NG_MMBN    | MOLUSI MICROFINANCE BANK                               |
| bank_code | NG_MP      | MoMo PSB                                               |
| bank_code | NG_MMBU    | Monarch Microfinance BANK                              |
| bank_code | NG_MMP     | Money Master PSB                                       |
| bank_code | NG_M       | MoneyBox                                               |
| bank_code | NG_MMBW    | MONEYFIELD MICROFINANCE BANK                           |
| bank_code | NG_MMV     | MONEYTRONICS MFB                                       |
| bank_code | NG_MMX     | MONEYTRUST MFB                                         |
| bank_code | NG090405   | Moniepoint                                             |
| bank_code | NG_MMY     | Moremonee MFB                                          |
| bank_code | NG_MMZ     | MOUA MFB                                               |
| bank_code | NG_MM      | MOVASCO-OP MFB                                         |
| bank_code | NG_MMBD    | MOYOFADE MICROFINANCE BANK                             |
| bank_code | NG_MMB     | Mozfin Microfinance BANK                               |
| bank_code | NG_MAMB    | Mutual Alliance Mortgage Bank                          |
| bank_code | NG_MBM     | MUTUAL BENEFITS MFB                                    |
| bank_code | NG_MTMB    | MUTUAL TRUST MICROFINANCE BANK                         |
| bank_code | NG_NMA     | NAF MFB                                                |
| bank_code | NG_NMC     | NARGATA MFB                                            |
| bank_code | NG_NMD     | Nasarawa Microfinance bank                             |
| bank_code | NG_NMBB    | NAVY MICROFINANCE BANK                                 |
| bank_code | NG_NSCM    | NDDC STAFF COOP MFB                                    |
| bank_code | NG_NME     | NDIORAH MFB                                            |
| bank_code | NG_NMBF    | NEPTUNE MICROFINANCE BANK                              |
| bank_code | NG_NMG     | NET MFB                                                |
| bank_code | NG_NGPMB   | NEW GOLDEN PASTURES MICROFINANCE BANK                  |
| bank_code | NG_NPMB    | NEW PRUDENTIAL MORTGAGE BANK                           |
| bank_code | NG_NMBH    | NEWDAWN MICROFINANCE BANK                              |
| bank_code | NG_NFL     | NEWEDGE FINANCE LTD                                    |
| bank_code | NG030001   | NEXIM BANK                                             |
| bank_code | NG_NMBI    | NICE MICROFINANCE BANK                                 |
| bank_code | NG_NP      | Nigeria PrisonsMicrofinance bank                       |
| bank_code | NG090194   | NIRSAL MicroFinance Bank                               |
| bank_code | NG_NFJ     | NOLT FINANCE                                           |
| bank_code | NG_NMK     | Nomase MFB                                             |
| bank_code | NG_NML     | Nombank MFB                                            |
| bank_code | NG_NF      | NORTHQUEST FINANCE                                     |
| bank_code | NG_NMM     | NOUN MFB                                               |
| bank_code | NG_NB      | Nova Bank                                              |
| bank_code | NG_NMBN    | NOVEL MICROFINANCE BANK                                |
| bank_code | NG_NMO     | NOVUS MFB                                              |
| bank_code | NG_NG      | NOWNOW（Contec Global）                                  |
| bank_code | NG090001   | NPF MICROFINANCE BANK                                  |
| bank_code | NG_NMBP    | Npolu-UST Microfinance Bank                            |
| bank_code | NG_NMBQ    | Nsehe Microfinance Bank                                |
| bank_code | NG_NMBR    | Nsuk Microfinance BANK                                 |
| bank_code | NG_NMS     | NSUKKA MFB                                             |
| bank_code | NG_NMT     | NUGGETS MFB                                            |
| bank_code | NG_NMU     | Numo Microfinance bank                                 |
| bank_code | NG_NMV     | NUTURE MFB                                             |
| bank_code | NG_NMW     | Nwannegadi MFB                                         |
| bank_code | NG_OMBX    | OAKLAND MICROFINANCE BANK                              |
| bank_code | NG_OMBY    | OAU MICROFINANCE BANK LTD                              |
| bank_code | NG_OMBZ    | OBA MICROFINANCE BANK                                  |
| bank_code | NG_OMA     | OBELEDU MFB                                            |
| bank_code | NG_OMC     | OBOLLO MFB                                             |
| bank_code | NG_OMD     | OCHE MFB                                               |
| bank_code | NG_OMBB    | OCTOPUS MICROFINANCE BANK LTD                          |
| bank_code | NG_OME     | Odoakpu MFB                                            |
| bank_code | NG_OMBF    | Ogberuru Microfinance Bank                             |
| bank_code | NG_OMG     | Ogige MFB                                              |
| bank_code | NG_OMH     | OHAFIA MFB                                             |
| bank_code | NG_OMBI    | OHHA MICROFINANCE BANK                                 |
| bank_code | NG_OMJ     | Ojokoro Mfb                                            |
| bank_code | NG_OMBK    | OK MICROFINANCE BANK                                   |
| bank_code | NG_OOMBL   | OKE-ARO OREDEGBE MICROFINANCE BANK LTD                 |
| bank_code | NG_OML     | Okengwe MFB                                            |
| bank_code | NG_OMM     | OKPE MFB                                               |
| bank_code | NG_OMN     | OKPOGA MFB                                             |
| bank_code | NG_OMBO    | OKUKU MICROFINANCE BANK LTD                            |
| bank_code | NG_OMP     | OKWO-OHA MFB                                           |
| bank_code | NG_OOUMB   | OLABISI ONABANJO UNIVERSITY MICROFINANCE BANK          |
| bank_code | NG_OSM     | Old Shoreham MFB                                       |
| bank_code | NG_OMQ     | OLIVE MFB                                              |
| bank_code | NG_OOMB    | OLOFIN OWENA Microfinance BANK                         |
| bank_code | NG_OAM     | OLOMU APERAN MFB                                       |
| bank_code | NG_OMBR    | OLUCHUKWU Microfinance BANK                            |
| bank_code | NG_OMS     | Omak MFB                                               |
| bank_code | NG_OMT     | OMIYE MFB                                              |
| bank_code | NG100004   | OPay                                                   |
| bank_code | NG000036   | OPTIMUS BANK                                           |
| bank_code | NG_OMBU    | Oraukwu Microfinance BANK                              |
| bank_code | NG_OMV     | ORISUN MFB                                             |
| bank_code | NG_OMBW    | ORITABASORUN MICROFINANCE BANK                         |
| bank_code | NG_OMBA    | OSANTA MICROFINANCE BANK                               |
| bank_code | NG_OMHY    | Oscotech MFB                                           |
| bank_code | NG_OMX     | OSOMHE MFB                                             |
| bank_code | NG_OMBC    | OSPOLY MICROFINANCE BANK                               |
| bank_code | NG_OMBD    | OTECH MICROFINANCE BANK LTD                            |
| bank_code | NG_OMBL    | OTUO MICROFINANCE BANK LTD                             |
| bank_code | NG_OMF     | OURPASS MFB                                            |
| bank_code | NG_OM      | OURS MFB                                               |
| bank_code | NG_OMB     | Oyan Microfinance Bank                                 |
| bank_code | NG_PMBE    | P-UP MICROFINANCE BANK                                 |
| bank_code | NG_PMG     | PACT MFB                                               |
| bank_code | NG100002   | Paga                                                   |
| bank_code | NG_PFL     | PAGE FINANCIALS                                        |
| bank_code | NG_PMBM    | Palmcoast Microfinance Bank                            |
| bank_code | NG_PMN     | Palmera MFB                                            |
| bank_code | NG100033   | PalmPay                                                |
| bank_code | NG000030   | PARALLEX BANK                                          |
| bank_code | NG_PMBO    | PARKWAY MICROFINANCE BANK                              |
| bank_code | NG_PR      | Parkway-ReadyCash                                      |
| bank_code | NG_PAX     | PARRALEX                                               |
| bank_code | NG_PMP     | PATHFINDER MFB                                         |
| bank_code | NG_PG      | PATRICK GOLD                                           |
| bank_code | NG_PO      | PayAttitude Online                                     |
| bank_code | NG_PMBQ    | PAYREP MICROFINANCE BANK LIMITED                       |
| bank_code | NG_PMR     | PAYSTACK MFB                                           |
| bank_code | NG_PS      | Paystack-Titan                                         |
| bank_code | NG_PMBT    | Peace Microfinance BANK                                |
| bank_code | NG_PTMB    | PECAN TRUST MICROFINANCE BANK                          |
| bank_code | NG_PMBL    | PENIEL MICORFINANCE BANK LTD                           |
| bank_code | NG_PMBU    | PENNYWISE MICROFINANCE BANK                            |
| bank_code | NG_PMV     | PENTECOST MFB                                          |
| bank_code | NG_PMBW    | PETRA MICROFINANCE BANK                                |
| bank_code | NG_PMX     | PETTYSAVE MFB                                          |
| bank_code | NG_PMY     | PILLAR MFB                                             |
| bank_code | NG_PMZ     | Platinum MFB                                           |
| bank_code | NG_PMBA    | PLATINUM MORTGAGE BANK                                 |
| bank_code | NG_PMBB    | PLUG MICROFINANCE BANK                                 |
| bank_code | NG_PK      | POCKETAPP                                              |
| bank_code | NG_PF      | Poder Finance                                          |
| bank_code | NG_PMC     | POINTONE MFB                                           |
| bank_code | NG000008   | Polaris Bank                                           |
| bank_code | NG_PMBD    | Polyibadan Microfinance Bank                           |
| bank_code | NG_PME     | POLYUWANNA MFB                                         |
| bank_code | NG_PMBF    | Preeminent Microfinance BANK                           |
| bank_code | NG_PMH     | PREMIER MFB                                            |
| bank_code | NG000031   | Premium Trust bank                                     |
| bank_code | NG_PMBG    | PRESTIGE MICROFINANCE BANK                             |
| bank_code | NG_PMBI    | Prisco Microfinance BANK                               |
| bank_code | NG_PDMB    | Pristine Divitis Microfinance BANK                     |
| bank_code | NG_PMJ     | PRODIGY MFB                                            |
| bank_code | NG_PMK     | Projects Microfinance bank                             |
| bank_code | NG_PCMB    | Prospa Capital Microfinance Bank                       |
| bank_code | NG_PMB     | Prosperity Microfinance Bank                           |
| bank_code | NG000023   | Providus Bank                                          |
| bank_code | NG_PML     | Prudent MFB                                            |
| bank_code | NG_PMM     | PURPLEMONEY MFB                                        |
| bank_code | NG_PM      | PYRAMID MFB                                            |
| bank_code | NG_QMBL    | QUBE MICROFINANCE BANK LTD                             |
| bank_code | NG_QM      | QUICK MFB                                              |
| bank_code | NG_QMB     | QUICKFUND MICROFINANCE BANK                            |
| bank_code | NG_RMN     | Radalpha Microfinance bank                             |
| bank_code | NG_RMO     | RAHAMA MFB                                             |
| bank_code | NG000024   | Rand Merchant Bank                                     |
| bank_code | NG_RMP     | RAYYAN MFB                                             |
| bank_code | NG_RMBQ    | Refuge Mortgage Bank                                   |
| bank_code | NG_RMR     | REGENT MFB                                             |
| bank_code | NG_RMBS    | Rehoboth Microfinance BANK                             |
| bank_code | NG_RMT     | RELIANCE MFB                                           |
| bank_code | NG_RMBU    | RENMONEY MICROFINANCE BANK                             |
| bank_code | NG_RMV     | REPHIDIM MFB                                           |
| bank_code | NG_RFL     | RESIDENT FINTECH LTD                                   |
| bank_code | NG_RMW     | RETRUST MFB                                            |
| bank_code | NG_RMX     | REVELATION MFB                                         |
| bank_code | NG_R       | REXEL                                                  |
| bank_code | NG_RMY     | RICHWAY MFB                                            |
| bank_code | NG_RMBZ    | RIGO Microfinance BANK                                 |
| bank_code | NG_RGPMB   | Rima Growth Pathway Microfinance Bank                  |
| bank_code | NG_RMBA    | RIMA Microfinance BANK                                 |
| bank_code | NG_RGM     | Rimin Gado MFB                                         |
| bank_code | NG_RMB     | ROCKSHIELD MICROFINANCE BANK                           |
| bank_code | NG_RBM     | Royal Blue MFB                                         |
| bank_code | NG_REMC    | ROYAL EXCELLENT MICROFINANCE BANK                      |
| bank_code | NG_REMB    | ROYAL EXCHANGE MICROFINANCE BANK                       |
| bank_code | NG_RMD     | RUBIES MFB                                             |
| bank_code | NG_RM      | RUN MFB                                                |
| bank_code | NG_SMF     | SABI MFB                                               |
| bank_code | NG_SHM     | Safe Haven MFB                                         |
| bank_code | NG_SMBB    | SAFEGATE MICROFINANCE BANK                             |
| bank_code | NG_ST      | SAFETRUST                                              |
| bank_code | NG_SMBE    | SAGAMU MICROFINANCE BANK                               |
| bank_code | NG050003   | SageGrey Finance Limited                               |
| bank_code | NG_SF      | SCIART FINANCE                                         |
| bank_code | NG_SMBG    | SEAP Microfinance Bank                                 |
| bank_code | NG_SCMB    | SEED CAPITAL MICROFINANCE BANK                         |
| bank_code | NG_SMBH    | SEEDVEST MICROFINANCE BANK                             |
| bank_code | NG_SMBI    | Shalom Microfinance Bank                               |
| bank_code | NG_STMB    | Shepherd Trust Microfinance BANK                       |
| bank_code | NG_SMBJ    | SHIELD MICROFINANCE BANK LTD                           |
| bank_code | NG_SMBK    | SHONGOM MICROFINANCE BANK LTD                          |
| bank_code | NG000034   | Signature Bank                                         |
| bank_code | NG_SFL     | SIMPLE FINANCE LIMITED                                 |
| bank_code | NG_SS      | Simplify Synergy                                       |
| bank_code | NG_SML     | SINCERE MFB                                            |
| bank_code | NG_SMBM    | SLS MICROFINANCE BANK                                  |
| bank_code | NG_SP      | SmartCash PSB                                          |
| bank_code | NG_SMN     | SNOW MFB                                               |
| bank_code | NG_SMO     | SOFRITRUST Microfinance                                |
| bank_code | NG_SAM     | Solid Allianze MFB                                     |
| bank_code | NG_SMP     | Solidrock Microfinance bank                            |
| bank_code | NG_SMQ     | SOROMAN MFB                                            |
| bank_code | NG_SMBR    | Source Microfinance Bank                               |
| bank_code | NG_SPSP    | SP                                                     |
| bank_code | NG090325   | Sparkle                                                |
| bank_code | NG_SMBS    | SPECTRUM MICROFINANCE BANK                             |
| bank_code | NG_SSF     | SPRING SKY FINANCE                                     |
| bank_code | NG_SMBU    | SPRINGFIELD MICROFINANCE BANK                          |
| bank_code | NG_SMBL    | SPRINGVILLE MICROFINANCE BANK LIMITED                  |
| bank_code | NG_SI      | Stanbic IBTC @ease wallet                              |
| bank_code | NG000012   | Stanbic IBTC Bank                                      |
| bank_code | NG000021   | STANDARD CHARTERED BANK NIGERIA LIMITED                |
| bank_code | NG_SMV     | Standard MFB                                           |
| bank_code | NG_SMW     | STANFORD MFB                                           |
| bank_code | NG_SMBX    | Stateside Microfinance Bank                            |
| bank_code | NG_SMBY    | STB Mortgage BANK                                      |
| bank_code | NG_SMBZ    | STELLAS MICROFINANCE BANK                              |
| bank_code | NG000001   | Sterling Bank                                          |
| bank_code | NG_SMBA    | STOCKCORP MICROFINANCE BANK                            |
| bank_code | NG_SM      | SULSPAP MFB                                            |
| bank_code | NG_SB      | SUMMIT BANK                                            |
| bank_code | NG_SMBC    | Sunbeam Microfinance BANK                              |
| bank_code | NG_SMB     | SUNTOP MICROFINANCE BANK                               |
| bank_code | NG000022   | SUNTRUST BANK                                          |
| bank_code | NG_SMBT    | SUPPORT MICROFINANCE BANK                              |
| bank_code | NG_SFBL    | SUPREME MICROFINANCE BANK LTD                          |
| bank_code | NG_SAMR    | SURE ANCHOR MFB                                        |
| bank_code | NG_STM     | SWIFT TRUST MFB                                        |
| bank_code | NG_SME     | SYCAMORE MFB                                           |
| bank_code | NG_TP      | TagPay                                                 |
| bank_code | NG000026   | Taj Bank                                               |
| bank_code | NG_TJP     | TAJ_PINSPAY                                            |
| bank_code | NG_TMBL    | TANADI MICROFINANCE BANK LTD                           |
| bank_code | NG_TMB     | Tangale Microfinance Bank                              |
| bank_code | NG_TM      | TASUED MFB                                             |
| bank_code | NG_TB      | TATUM BANK                                             |
| bank_code | NG_TMY     | TeasyMobile                                            |
| bank_code | NG050007   | TEKLA FINANCE                                          |
| bank_code | NG_TFM     | TELLERONE FI MFB                                       |
| bank_code | NG_TMN     | TENN MFB                                               |
| bank_code | NG_TMBF    | TF MICROFINANCE BANK                                   |
| bank_code | NG_TBFL    | THE BROOK FINANCE LIMITED                              |
| bank_code | NG_TMV     | THRIVE MFB                                             |
| bank_code | NG000025   | TITAN TRUST BANK                                       |
| bank_code | NG_TME     | TOPRATE MFB                                            |
| bank_code | NG_TMS     | TREASURES MFB                                          |
| bank_code | NG_TMBK    | TRIDENT MICROFINANCE BANK                              |
| bank_code | NG_TFSL    | TRINITY FINANCIAL SERVICES LIMITED                     |
| bank_code | NG_TAM     | Triple A Microfinance bank                             |
| bank_code | NG_TFCL    | TRIVES FINANCE COMPANY LTD                             |
| bank_code | NG_TMT     | TRUST MFB                                              |
| bank_code | NG_TJMBL   | TRUSTBANC J6 MICROFINANCE BANK LIMITED                 |
| bank_code | NG_TD      | TRUSTBOND                                              |
| bank_code | NG_TMBD    | TRUSTFUND MICROFINANCE BANK                            |
| bank_code | NG_UCM     | U AND C MFB                                            |
| bank_code | NG_UM      | UBA MONI                                               |
| bank_code | NG_UMJ     | UBJ MFB                                                |
| bank_code | NG_UME     | UCEE MFB                                               |
| bank_code | NG_UMBL    | UDA Microfinance Bank Limited                          |
| bank_code | NG_UMRO    | UFORO MFB                                              |
| bank_code | NG_UMRB    | Uhuru Microfinance bank                                |
| bank_code | NG_UMP     | UKPOR MFB                                              |
| bank_code | NG_UMB     | Ultimate Microfinance Bank                             |
| bank_code | NG_UMBH    | Ummah Microfinance Bank                                |
| bank_code | NG_UPMB    | Umuchinemere Procredit Microfinance Bank               |
| bank_code | NG_UMBM    | UMUCHUKWU MICROFINANCE BANK                            |
| bank_code | NG_UMBI    | Umunnachi Microfinance Bank                            |
| bank_code | NG_UMR     | UMUNRI MFB                                             |
| bank_code | NG_UMY     | Umyu MFB                                               |
| bank_code | NG_UMF     | UNAAB MFB                                              |
| bank_code | NG_UMBU    | UNIBADAN MICROFINANCE BANK                             |
| bank_code | NG_UMNK    | UNIBEN MICROFINANCE BANK                               |
| bank_code | NG_UML     | UNICAL MFB                                             |
| bank_code | NG_UMBK    | Unifund Microfinance Bank                              |
| bank_code | NG_UMBG    | UNILAG MICROFINANCE BANK                               |
| bank_code | NG_UMBN    | UNILORIN MICROFINANCE BANK                             |
| bank_code | NG_UMBD    | Unimaid Microfinance BANK                              |
| bank_code | NG000018   | Union Bank Of Nigeria                                  |
| bank_code | NG000004   | UNITED BANK FOR AFRICA                                 |
| bank_code | NG000011   | UNITY BANK PLC                                         |
| bank_code | NG_UMBY    | UniUyo Microfinance BANK                               |
| bank_code | NG_UNL     | Unlimint Nigeria Limited                               |
| bank_code | NG_UMN     | UNN MFB                                                |
| bank_code | NG_UMI     | UNUBI MFB                                              |
| bank_code | NG_UMBR    | Ure Microfinance Bank                                  |
| bank_code | NG_UMK     | UTAKO MFB                                              |
| bank_code | NG_UMU     | UVUOMA MFB                                             |
| bank_code | NG_UMBZ    | Uzondu Microfinance Bank                               |
| bank_code | NG_VFL     | Vale Finance Limited                                   |
| bank_code | NG_VMB     | Verdant Microfinance Bank                              |
| bank_code | NG_VDM     | VFD MFB                                                |
| bank_code | NG_VYM     | Victory MFB                                            |
| bank_code | NG_VTM     | VIRTUE MFB                                             |
| bank_code | NG_VSMB    | VISA MICROFINANCE BANK                                 |
| bank_code | NG_V       | VTNetworks                                             |
| bank_code | NG_WIM     | WAILA MFB                                              |
| bank_code | NG_WTM     | WALLET MFB                                             |
| bank_code | NG_WYM     | WAYA MFB                                               |
| bank_code | NG_WMB     | WE MICROFINANCE BANK                                   |
| bank_code | NG000017   | Wema Bank                                              |
| bank_code | NG_WSM     | WESLEY MFB                                             |
| bank_code | NG_WLM     | WETLAND MFB                                            |
| bank_code | NG_WCM     | Wetson-Charis MFB                                      |
| bank_code | NG_WFL     | WHITECRUST FINANCE LTD                                 |
| bank_code | NG_WB      | WinView BANK                                           |
| bank_code | NG_WRM     | WRA MFB                                                |
| bank_code | NG_WDMB    | Wudil Microfinance Bank                                |
| bank_code | NG_XM      | XPRESS MTS                                             |
| bank_code | NG_X       | Xpresswallet                                           |
| bank_code | NG_XMB     | XSLNCE MICROFINANCE BANK                               |
| bank_code | NG_YMB     | YCT Microfinance BANK                                  |
| bank_code | NG_YSM     | YES MFB                                                |
| bank_code | NG_YBM     | Yobe MFB                                               |
| bank_code | NG_ZIM     | ZAIN MFB                                               |
| bank_code | NG_ZFL     | ZEDVANCE FINANCE LIMITED                               |
| bank_code | NG_ZFM     | Zefa MFB                                               |
| bank_code | NG000015   | Zenith Bank                                            |
| bank_code | NG_ZEW     | ZENITH EASY WALLET                                     |
| bank_code | NG_ZHM     | ZENITH Mobile                                          |
| bank_code | NG_ZMB     | Zikora Microfinance bank                               |
| bank_code | NG_ZOMB    | ZION MICROFINANCE BANK                                 |
| bank_code | NG_ZTM     | Zitra MFB                                              |

# 13、错误码

| 状态码  | 描述                |
|------|-------------------|
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

# 14、文档更新时间
```
2026-11-17 20:51:12
```
