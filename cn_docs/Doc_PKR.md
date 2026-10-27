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
> 2、金额单位为<span style="color:red;"> 分（库鲁什） </span>。
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
| 名称           | 类型   | 必填    | 描述                                                                                 |
| -------------- | ------ |-------|------------------------------------------------------------------------------------|
| trade_no       | int    | true  | 商户号                                                                                |
| app_id         | int    | true  | 商户 appId                                                                           |
| pay_code       | int    | true  | 产品编码,联系我方运营获取                                                                      |
| pay_method     | string    | true  | 支付方式   参照支付方式字典                                                                    |
| price          | int    | true  | 下单金额,单位:分 ,整数                                                                      |
| order_no       | string | true  | 商户订单号                                                                              |
| success_url    | string | false | 支付成功跳转 url                                                                         |
| fail_url       | string | false | 支付失败跳转 url                                                                         |
| pay_notice_url | string | false | 支付成功通知 url                                                                         |
| user_id        | string | true  | 付款人在商户平台的 ID                                                                       |
| user_ip        | string | false | 付款人 IP                                                                             |
| attach         | string | true  | 附加参数json字符串 |
| sign           | string | true  | 签名结果,签名方法在文档顶部                                                                     |
|timestamp|string| false | 下单时间戳 10位时间戳单位S                                                                    |


-  代收-attach 附加参数字段说明
| 名称            | 类型     | 必填    | 描述                       |
|---------------|--------|-------|--------------------------|
| name          | string | true  | 付款人姓名                    |
| phone         | string    | true  | 手机号  03开头的十一位数或者3开头的10位数 |
| email         | string    | false | 邮箱                       |
| identify_type | string | false | CNIC                     |
| identify_num  | string | false | 识别号码                     |




请求参数示例

```json
{
  "trade_no": 165,
  "order_no": "p71584121t1693047656571",
  "app_id": 51,
  "pay_code": 91145,
  "price": 10000,
  "success_url": "",
  "fail_url": "",
  "pay_notice_url": "http://www.test.com",
  "pay_remind_url": "",
  "attach": "{\"phone\":\"123456789\"} ",
  "user_ip": "87.200.59.188",
  "user_id": "2677323",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 4.2下单响应

| 名称         | 类型   | 必填 | 描述                        |
| ------------ | ------ | ---- | --------------------------- |
| code         | int    | true | 200:下单成功 其他:下单失败  |
| msg          | string | true | 失败原因                    |
| pay_url      | string | true | 付款链接                    |
| qr_code      | string | true |                             |
| order_no     | string | true | 商户订单号                  |
| dis_order_no | string | true | 平台订单号                  |
| create_time  | int    | true | 创建时间                    |
| pay_info  | string    | false | 付款信息 json字符串 例如：{"pay_raw":"支付原生信息，商户可以自行转换成二维码"} |
| sign         | string | true | 签名结果,签名方法在文档顶部 |

-  代收-下单响应示例 
失败:

```json
{
  "code": 10009,
  "msg": "ip不在商户ip白名单, ip:158.28.87.267",
  "sign": "db3406277185f9660b3b928d6adc7bc4",
  "pay_url": "",
  "qr_code": "",
  "order_no": "",
  "dis_order_no": "",
  "create_time": 0
}
```

成功:

```json
{
  "code": 200,
  "msg": "success",
  "pay_url": "https://fourpay-intest2.ncjimmy.com/payApi/PayApi/CallBack/3_97091602d4501dbffce038ca6419e176",
  "qr_code": "TNVPdaYdXnUxADTfLcKvkam4836B2vkKT5",
  "order_no": "47210116924681604173",
  "dis_order_no": "lufei169246816001692",
  "sign": "db3406277185f9660b3b928d6adc7bc4",
  "create_time": 1695317066
}
```

# 5、代收回调通知   post/json 

推送地址:商户下单传送的 pay_notice_url 回调 ip:call_back_server_ip ,请将我方 ip 加入回调白名单

## 5.1 代收回调-请求参数

| 名称         | 类型   | 必填  | 描述                                                                  |
| ------------ | ------ | ----- | --------------------------------------------------------------------- |
| trade_no     | int    | true  | 商户号                                                                |
| status       | int    | true  | 订单状态, <span style="color:red;">2.成功</span>, 3.失败                                              |
| order_no     | string | true  | 商户订单号                                                            |
| dis_order_no | string | true  | 平台订单号                                                            |
| order_price  | int    | true  | 订单金额,单位:分                                                      |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">用户真实付款金额 ,单位:分</span>                                             |
| nti_time     | int    | false | 发起通知时间                                                          |
| payer        | string | false | JSON 字符串,付款人信息{"name":"姓名","email":"邮箱","phone":"手机号"} |
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等 |
| create_time  | int    | true  | 创建时间                                                              |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                           |

-  代收回调-请求参数示例

```json
{
  "trade_no": 165,
  "status": 2,
  "order_no": "p71584121t1693047656571",
  "dis_order_no": "lufei169246816001692",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1693057443,
  "payer": "{\"name\":\"姓名\",\"email\":\"邮箱\",\"phone\":\"手机号\"}",
  "create_time": 1695317066,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 5.2 代收回调-响应说明
回调接收处理成功，请返回 <span style="color:red;">success</span>，系统将不再推送此订单信息，否则还会重复推送多次

# 6、代付下单接口

(下单 ip 需要联系我方加白)
下单地址 https://{api_domain}/api/v1/payApi/CreatePayOutOrder

## 6.1 代付-请求参数

| 名称           | 类型   | 必填  | 描述                  |
| -------------- | ------ | ----- |---------------------|
| trade_no       | int    | true  | 商户号                 |
| order_no       | string | true  | 商户订单号               |
| app_id         | int    | true  | 商户 appId            |
| pay_code       | int    | true  | 产品编码,联系我方运营获取       |
| price          | int    | true  | 下单金额,单位:分 ,整数       |
| account_no     | string | true  | 出款账号                |
| account_name   | string | true  | 姓名                  |
| account_type   | string | true  | 账号类型:BANK,JAZZCASH,EASYPAISA,WALLET |
| bank_code      | string | true  | 银行编码在末尾             |
| pay_notice_url | string | false | 代付成功通知 url          |
| attach         | string | false | 附加参数                |
| user_ip        | string | false  | 收款用户 IP             |
| sign           | string | true  | 签名结果,签名方法在文档顶部      |
|timestamp|string|false| 下单时间戳 10位时间戳单位S     |

###请求参数示例

```json
{
  "trade_no": 165,
  "order_no": "p71584121t1693047656571",
  "app_id": 51,
  "pay_code": 91145,
  "price": 10000,
  "account_type": "BANK",
  "account_no": "012345678910111213",
  "account_name": "TEST",
  "identify_num":"1234567890123",
  "bank_code": "BKKB",
  "pay_notice_url": "http://www.test.com",
  "attach": "",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 6.2 代付-下单响应

| 名称         | 类型   | 必填 | 描述                                    |
| ------------ | ------ | ---- |---------------------------------------|
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
  "code": 10009,
  "msg": "下单失败"
}
```

成功:

```json
{
  "code": 200,
  "msg": "success",
  "status": 1,
  "order_no": "47210116924681604173",
  "dis_order_no": "lufei169246816001692",
  "create_time": 1695317066,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 7、代付回调通知 

推送地址:商户下单传送的 pay_notice_url 回调 ip:call_back_server_ip ,请将我方 ip 加入回调白名单

## 7.1 代付回调请求参数
| 名称         | 类型   | 必填  | 描述                                     |
| ------------ | ------ | ----- | ---------------------------------------- |
| trade_no     | int    | true  | 商户号                                   |
| order_no     | string | true  | 商户订单号                               |
| dis_order_no | string | true  | 平台订单号                               |
| order_price        | int    | true  | 订单金额,单位:分                         |
| fee          | int    | false | 订单手续费 ,单位:分                      |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">真实代付出款金额 (代付成功时才有) 如需使用real_price字段，请与工作人员联系配置</span>                                                                                       |
| status       | int    | true  | 订单状态, <span style="color:red;">2.代付成功</span>, 3.代付失败, 7.驳回 9.冲正 |
| pay_info  | string    | false | 付款信息                          |
| remark       | string | false | 失败原因                                 |
| create_time  | int    | true  | 创建时间                                 |
| sign         | string | true  | 签名结果,签名方法在文档顶部              |
| nti_time     | int    | true | 发起通知时间                        |

- 代付回调请求示例

```json
{
  "trade_no": 165,
  "status": 2,
  "order_no": "p71584121t1693047656571",
  "dis_order_no": "lufei169246816001692",
  "order_price": 10000,
"fee": 10,
  "pay_info": "",
  "remark": "",
  "nti_time": 1693057443,
  "create_time": 1695317066,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 7.2 代付回调响应说明
回调接收处理成功，请返回 <span style="color:red;">success</span>，系统将不再推送此订单信息，否则还会重复推送多次


# 8、查询订单接口 (代收 代付共用) 

(请求 ip 需要联系我方加白)  
查询地址: https://{api_domain}/api/v1/payApi/QueryOrder

## 8.1 查询请求参数

| 名称       | 类型   | 必填 | 描述                        |
| ---------- | ------ | ---- | --------------------------- |
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
  "order_no": "p71584121t1693047656571",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 8.2 查询响应

| 名称         | 类型   | 必填  | 描述                                                                  |
| ------------ | ------ | ----- | --------------------------------------------------------------------- |
| code         | int    | true  | 200:查询成功 其他:失败                                                |
| msg          | string | true  | 查询失败原因                                                          |
| trade_no     | int    | true  | 商户号                                                                |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">真实付款金额 ,单位:分</span>                                                 |
| status       | int    | true  | 订单状态:1.未支付, <span style="color:red;">2.成功</span>, 3.失败 , 7.驳回 9.冲正  10:处理中                                    |
| success_time | int    | true  | 成功时间戳                                                            |
| order_no     | string | true  | 商户订单号                                                            |
| dis_order_no | string | true  | 平台订单号                                                            |
| remark       | string | true  | 代付失败原因                                                          |
| fee          | int    | false | 订单手续费 ,单位:分                                                   |
| create_time  | int    | true  | 创建时间                                                              |
| payer        | string | false | JSON 字符串,付款人信息{"account_name":"姓名","account_type":"账号类型:BANK,JAZZCASH,EASYPAISA,WALLET","account_no":"账号","bank_code":"银行编码","identify_num":"证件号码"} |
| pay_info  | string    | false | 付款信息 json字符串 例如：收、付款原生信息、卡号、名字、银行等  25-10-28 |
| sign         | string | true  | 签名结果,签名方法在文档顶部                                                                                                                                  |
|utr2|string|false|银行订单号|

- 查询响应示例

失败:

```json
{
  "code": 10009,
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
  "payer": "{\"account_name\":\"TEST\",\"account_type\":\"BANK\",\"account_no\":\"012345678910111213\",\"bank_code\":\"BKKB\",\"identify_num\":\"1234567890123\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9、代付余额查询接口  
(请求 ip 需要联系我方加白)  
地址: https://{api_domain}/api/v1/payApi/QueryBalance

## 9.1 余额请求参数
| 名称     | 类型   | 必填 | 描述                        |
| -------- | ------ | ---- | --------------------------- |
| trade_no | int    | true | 商户号                      |
| app_id   | int    | true | appid                       |
| sign     | string | true | 签名结果,签名方法在文档顶部 |

-  余额请求示例

```json
{
  "trade_no": 165,
  "app_id": 28,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
## 9.2 余额响应

| 名称    | 类型   | 必填 | 描述                        |
| ------- | ------ | ---- | --------------------------- |
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
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
# 10、支付方式 代收字段 pay_method

| 字段         | 国家   | 值                  | 描述     |
|------------|------|--------------------|--------|
| pay_method | 巴基斯坦 | easypaisa | Easypaisa巴基斯坦代收 |
| pay_method | 巴基斯坦 | jazzcash | Jazzcash巴基斯坦代收 |
| pay_method | 巴基斯坦     | QR-PKR                 | QR巴基斯坦扫码 |


# 11、银行编码 bank_code

| 字段      | 国家     | 值           | 描述                           |
|-----------|--------|-------------|------------------------------|
| bank_code | 巴基斯坦 | PKRABL | Allied Bank Limited |
| bank_code | 巴基斯坦 | PKRADVANS | Advans Pakistan Microfinance Bank Limited |
| bank_code | 巴基斯坦 | PKRAIBPL | Albaraka Islamic Bank Pakistan Limited |
| bank_code | 巴基斯坦 | PKRALFALAH | Bank Alfalah Limited |
| bank_code | 巴基斯坦 | PKRAPNA MFB | Apna Microfinance Bank Limited |
| bank_code | 巴基斯坦 | PKRASKARI | Askari Bank Limited |
| bank_code | 巴基斯坦 | PKRBAHL | Bank Al Habib Limited |
| bank_code | 巴基斯坦 | PKRBIPL | Bank Islami Pakistan Limited |
| bank_code | 巴基斯坦 | PKRBOK | The Bank of Khyber |
| bank_code | 巴基斯坦 | PKRBOP | The Bank of Punjab |
| bank_code | 巴基斯坦 | PKRCITI | Citibank N.A. Pakistan |
| bank_code | 巴基斯坦 | PKRDIBPL | Dubai Islamic Bank Pakistan Limited |
| bank_code | 巴基斯坦 | PKREASYPAISA | Easypaisa |
| bank_code | 巴基斯坦 | PKRFAYSAL | Faysal Bank Limited |
| bank_code | 巴基斯坦 | PKRFINCA | FINCA Microfinance Bank Limited |
| bank_code | 巴基斯坦 | PKRFINJA | Finja Private Limited |
| bank_code | 巴基斯坦 | PKRFWBL | First Women Bank Limited |
| bank_code | 巴基斯坦 | PKRHBL | Habib Bank Limited |
| bank_code | 巴基斯坦 | PKRHBL MFB | HBL Microfinance Bank Limited |
| bank_code | 巴基斯坦 | PKRHMBL | Habib Metropolitan Bank Limited |
| bank_code | 巴基斯坦 | PKRICBC | Industrial and Commercial Bank of China Limited Pakistan |
| bank_code | 巴基斯坦 | PKRJAZZCASH | JazzCash |
| bank_code | 巴基斯坦 | PKRJSBL | JS Bank Limited |
| bank_code | 巴基斯坦 | PKRKHUSHHALI | Khushhali Microfinance Bank Limited |
| bank_code | 巴基斯坦 | PKRKONNECT | Konnect by Bank Alfalah |
| bank_code | 巴基斯坦 | PKRMCB | MCB Bank Limited |
| bank_code | 巴基斯坦 | PKRMCBAH | MCB Arif Habib Limited |
| bank_code | 巴基斯坦 | PKRMCB ISLAMIC | MCB Islamic Bank Limited |
| bank_code | 巴基斯坦 | PKRMEEZAN | Meezan Bank Limited |
| bank_code | 巴基斯坦 | PKRNAYAPAY | NayaPay |
| bank_code | 巴基斯坦 | PKRNBP | National Bank of Pakistan |
| bank_code | 巴基斯坦 | PKRNBP FUNDS | NBP Funds Management Limited |
| bank_code | 巴基斯坦 | PKRNRSP | NRSP Microfinance Bank Limited |
| bank_code | 巴基斯坦 | PKRSADAPAY | SadaPay |
| bank_code | 巴基斯坦 | PKRSAMBA | Samba Bank Limited |
| bank_code | 巴基斯坦 | PKRSCB | Standard Chartered Bank Pakistan Limited |
| bank_code | 巴基斯坦 | PKRSILK | Silk Bank Limited |
| bank_code | 巴基斯坦 | PKRSINDH | Sindh Bank Limited |
| bank_code | 巴基斯坦 | PKRSONERI | Soneri Bank Limited |
| bank_code | 巴基斯坦 | PKRSUMMIT | Summit Bank Limited |
| bank_code | 巴基斯坦 | PKRTMB | Telenor Microfinance Bank Limited |
| bank_code | 巴基斯坦 | PKRUBL | United Bank Limited |
| bank_code | 巴基斯坦 | PKRUMFB | U Microfinance Bank Limited |
| bank_code | 巴基斯坦 | PKRUPAISA | Upaisa |



# 12、错误码

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

# 13、代收收银台接口

地址: https://{api_domain}/api/v1/cashApi/CashIn.html
请求方式: GET

### 参数:

| 名称           | 类型     | 必填    | 描述        |
|--------------|--------|-------|-----------|
| app_id       | string | true  | 商户app_id  |
| order_no       | string | true  | 商户订单号     |
| amount       | string | true  | 商户金额单位:里拉 |
| notice_url       | string | false | 异步通知地址    |
| pay_code       | int    | true | 产品编码      |

#### 示例

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{商户订单号}}&amount={{商户金额}}&notice_url={{异步通知地址}}&pay_code={{产品编码}}
```

# 14、文档更新时间
```
2026-05-10 19:25:00
```
