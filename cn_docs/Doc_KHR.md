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
> 2、金额单位为<span style="color:red;"> 分 1 KHR=100 或者 1KHRUSD =100</span>。
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
| pay_method     | string | true  | 支付方式,见下方支付方式                                                                                                                                                               |
| price          | int    | true  | 下单金额,单位:分 ,整数                                                                                                                                                                 |
| order_no       | string | true  | 商户订单号                                                                                                                                                                         |
| success_url    | string | false | 支付成功跳转 url                                                                                                                                                                    |
| fail_url       | string | false | 支付失败跳转 url                                                                                                                                                                    |
| pay_notice_url | string | false | 支付成功通知 url                                                                                                                                                                    |
| user_id        | string | true  | 系统用户ID                                                                                                                                                                        |
| user_ip        | string | true  | 付款人 IP                                                                                                                                                                        |
| attach         | string | false  | 附加参数 json字符串 付款人信息 |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                                                                                                                                |
| timestamp      | string | false | 下单时间戳 10位时间戳单位S                                                                                                                                                               |

- 代收-attach 附加参数字段说明

  | 名称            | 类型     | 必填    | 描述                                               |
    | ------------- | ------ | ----- |--------------------------------------------------|
  | name          | string | false | 付款人姓名                                            |
  | email         | string | false | 付款人邮箱                                            |

- 代收-下单请求示例

  ```json
  {
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"name\":\"张三\",\"email\":\"zhangsan@example.com\"}",
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
  "pay_url": "https://api.test.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5",
  "pay_info": ""
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

| 名称             | 类型     | 必填    | 描述                                                   |
| -------------- | ------ | ----- |------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                  |
| order_no       | string | true  | 商户订单号                                                |
| app_id         | int    | true  | 商户 appId                                             |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                        |
| price          | int    | true  | 下单金额,单位:分 ,整数                             |
| account_no     | string | true  | 收款账号                                |
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
  "account_no": "1234567890123",
  "account_name": "Nguyen Van A",
  "bank_code": "KHR_ABA"
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
  "payer": "{\"account_name\":\"Nguyen Van A\",\"account_type\":\"BANK\",\"account_no\":\"1234567890123\",\"bank_code\":\"VCB\"}",
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
# 11、代收银行编码 代收字段 pay_method

| 字段         | 值               | 描述   |
|------------|------------------|------|
| pay_method | KHR_QR     | 扫码转账 |
| pay_method | KHR_BANK     | 银行卡转账 |

# 12、银行编码
| 字段名称 | 编码         | 银行名称 |
| :--- |:-----------| :--- |
| bank_code | KHR_ABA | ABA Bank |
| bank_code | KHR_AMRET | Amret Plc. |
| bank_code | KHR_SATHAPANA | Sathapana Bank Plc |
| bank_code | KHR_AMK | AMK Microfinance Plc. |
| bank_code | KHR_WING | Wing Bank (Cambodia) Plc |
| bank_code | KHR_RHB | RHB Bank (Cambodia) Plc. |
| bank_code | KHR_BRED | BRED Bank (Cambodia) Plc |
| bank_code | KHR_BOC | Bank of China (Hong Kong) Limited |
| bank_code | KHR_UPAY | U-Pay Digital Plc |
| bank_code | KHR_BONGLOY | BongLoy |
| bank_code | KHR_ARDB | Agricultural and Rural Development Bank |
| bank_code | KHR_UCB | Union Commercial Bank Plc. |
| bank_code | KHR_CCU | CCU Commercial Bank PLC. |
| bank_code | KHR_PHILLIP | Phillip Bank Plc |
| bank_code | KHR_VATTANAC | Vattanac Bank |
| bank_code | KHR_TRUEMONEY | TrueMoney Cambodia |
| bank_code | KHR_ASIA_WEI_LUY | Asia Wei Luy |
| bank_code | KHR_FTB | Foreign Trade Bank of Cambodia |
| bank_code | KHR_PPCB | Phnom Penh Commercial Bank |
| bank_code | KHR_MOHANOKOR | MOHANOKOR MFI Plc. |
| bank_code | KHR_DGB | DGB Bank |
| bank_code | KHR_DARA_SAKOR_PAY | Dara Sakor Pay PLC |
| bank_code | KHR_ALPHA | Alpha Commercial Bank PLC |
| bank_code | KHR_KESS | Kess Innovation Plc. |
| bank_code | KHR_AEON | Aeon Specialized Bank (Cambodia) PLC. |
| bank_code | KHR_KB_PRASAC | KB PRASAC Bank Plc |
| bank_code | KHR_PRINCE | PRINCE BANK PLC |
| bank_code | KHR_ACLEDA | ACLEDA Bank Plc. |
| bank_code | KHR_CAMBODIAN_PUBLIC_BANK | Cambodian Public Bank Plc |
| bank_code | KHR_EMONEY | eMoney |
| bank_code | KHR_CAMBODIA_POST_BANK | Cambodia Post Bank Plc |
| bank_code | KHR_HATTHA | Hattha Bank Plc |
| bank_code | KHR_MAYBANK | Maybank Cambodia PLC |
| bank_code | KHR_CAMBODIA_ASIA_BANK | Cambodia Asia Bank |
| bank_code | KHR_CHIP_MONG | Chip Mong Commercial Bank Plc. |
| bank_code | KHR_LY_HOUR_PAY_PRO | LY HOUR PAY PRO PLC |
| bank_code | KHR_CANADIA | Canadia Bank Plc |
| bank_code | KHR_SPEEDPAY | Speedpay PLC |
| bank_code | KHR_IBANK | IBANK (CAMBODIA) PLC. |
| bank_code | KHR_COOL_CASH | Cool Cash Plc |
| bank_code | KHR_CHIEF | Chief (Cambodia) Commercial Bank Plc. |
| bank_code | KHR_CATHAY_UNITED | Cathay United Bank (Cambodia) |
| bank_code | KHR_JTRUST_ROYAL | J Trust Royal Bank Plc. |
| bank_code | KHR_PANDA | Panda Commercial Bank PLC. |
| bank_code | KHR_IBK | IBK Bank Cambodia |
| bank_code | KHR_HONG_LEONG | Hong Leong Bank (Cambodia) Plc |
| bank_code | KHR_LOLC | LOLC (Cambodia) Plc. |
| bank_code | KHR_WOORI | Woori Bank (Cambodia) Plc. |
| bank_code | KHR_BIDC | BIDC Bank |
| bank_code | KHR_SBI | SBI BANK (CAMBODIA) PLC. |
| bank_code | KHR_ORIENTAL | Oriental Bank |
| bank_code | KHR_APD | APD Bank |
| bank_code | KHR_ICBC | ICBC |
| bank_code | KHR_SACOMBANK | Sacombank Cambodia |
| bank_code | KHR_FIRST_COMMERCIAL | First Commercial Bank |
| bank_code | KHR_HENG_FENG | Heng Feng (Cambodia) Bank |
| bank_code | KHR_LANTON_PAY | Lanton Pay |
| bank_code | KHR_MB_CAMBODIA | MBCambodia |
| bank_code | KHR_BRIDGE | BRIDGE Bank |
| bank_code | KHR_BOOYOUNG_KHMER | Booyoung Khmer Bank |
| bank_code | KHR_SHINHAN | Shinhan Bank Cambodia Plc |
| bank_code | KHR_CIMB | CIMB |
| bank_code | KHR_SBI_LY_HOUR | SBI LY HOUR Bank Plc. |
| bank_code | KHR_PEAK_WEALTH | PEAK WEALTH BANK PLC |
| bank_code | KHR_PI_PAY | Pi Pay Plc. |
| bank_code | KHR_BIC | B.I.C (Cambodia) Bank Plc. |


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

# 14、代收收银台接口

地址: https://{api_domain}/api/v1/cashApi/CashIn.html
请求方式: GET

### 参数:

| 名称           | 类型     | 必填    | 描述                        |
|--------------|--------|-------|---------------------------|
| app_id       | string | true  | 商户app_id                    |
| order_no       | string | true  | 商户订单号                     |
| amount       | string | true  | 商户金额单位越南盾                     |
| notice_url       | string | false | 异步通知地址                     |
| pay_code       | int    | true | 产品编码                     |

#### 示例

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{商户订单号}}&amount={{商户金额}}&notice_url={{异步通知地址}}&pay_code={{产品编码}}
```

# 15、文档更新时间
```
2026-10-02 19:25:00
```
