# 1. Integration Process

> 1. Complete business discussions for account opening and confirm the relevant rates.
>
> 2. Contact the operations team to create the merchant ID, secret key, merchant `appId`, product code, and `apiUrl`.
>
> 3. After development is completed, both parties will conduct integration testing to verify that requests, reporting, and other information are complete.

# 2. MD5 Signature Algorithm

> 1. Sort all parameters in ascending order by key-value pair (`key1=value1`). Parameters with empty values do not participate in the signature.
>
> 2. Concatenate them in the format `key1=value1&key2=value2`.
>
> 3. Append the merchant secret key: `key1=value1&key2=value2...&key=MerchantSecretKey`.
>
> 3. `sign=md5(the string assembled in the previous step)`. The signature result is a 32-character lowercase string.
>
> 4. The signature key can be viewed in the merchant backend under `Basic Information`, or obtained by contacting our customer service.

# 3. Notes

## 3.1 Interface Related

> 1. All interfaces in this document use the standard HTTP protocol with `POST` requests. The `Content-type` of both requests and responses is `application/json`, and the character encoding is unified as `UTF-8`.
>
> 2. The amount unit is <span style="color:red;"> Sen </span>.
>
> 3. The request IP must be whitelisted.
>
> 4. Please collect the user's real IP for `user_ip` whenever possible. If it is not available, leave it blank. Do not use local IPs such as `127.0.0.1`.

## 3.2 Callback Related

> 1. If the callback is processed successfully, please return the plain text <span style="color:red;">`success`</span> without any extra characters. Otherwise, the system will continue to push the order notification repeatedly.
>
> 2. During asynchronous notification processing, if the received response is not `success`, the notification will be considered failed and retried periodically. The retry schedule is: `1m`, `1m`, `4m`, `10m`, `10m`, `1h`, `2h`, `6h`, `15h`.
>
> 3. If `pay_notice_url` is empty, it will be regarded as no callback required by the merchant, and the system will not push notifications.

# 4. Pay-in Order Interface

(The order placement IP must be whitelisted by contacting us)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayInOrder`

## 4.1 Pay-in Order Request Parameters

| Name           | Type   | Required | Description |
| -------------- | ------ | -------- | ----------- |
| trade_no       | int    | true     | Merchant ID |
| app_id         | int    | true     | Merchant `appId` |
| pay_code       | int    | true     | Product code, please contact our operations team |
| pay_method     | string | true     | Payment method, see the payment methods below |
| price          | int    | true     | Order amount, unit: Sen, integer. `1 RM = 100 Sen` |
| order_no       | string | true     | Merchant order number |
| success_url    | string | false    | Redirect URL after successful payment |
| fail_url       | string | false    | Redirect URL after failed payment |
| pay_notice_url | string | false    | Payment success callback URL |
| user_id        | string | false    | Merchant user ID |
| user_ip        | string | false    | Payer IP |
| attach         | string | false    | Additional parameters in JSON string format, payer information |
| sign           | string | true     | Signature result, see the signature method at the top of this document |
| timestamp      | string | false    | Order timestamp, 10-digit Unix timestamp in seconds |

- Pay-in `attach` Field Description

| Name      | Type   | Required | Description |
| --------- | ------ | -------- | ----------- |
| bank_code | string | false    | Bank code, required for bank card payments |
| name      | string | false    | Payer name, required for bank card payments |

- Pay-in Order Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"bank_code\":\"MAY\",\"name\":\"bob kite\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "203.0.113.15",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}
```

## 4.2 Pay-in Order Response

| Name         | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| code         | int    | true     | `200`: order created successfully, others: order creation failed |
| msg          | string | true     | Failure reason |
| pay_url      | string | false    | Payment URL |
| qr_code      | string | false    | QR code string |
| order_no     | string | true     | Merchant order number |
| dis_order_no | string | true     | Platform order number |
| create_time  | int    | true     | Creation time |
| pay_info     | string | false    | Payment info JSON string, for example: payer/payee native information, card number, name, bank, etc. `{"pay_raw":"native payment info","redirect_url":"payment redirect URL"}` |
| sign         | string | true     | Signature result, see the signature method at the top of this document |

- Pay-in Order Response Example

Failure:

```json
{
  "code": 1005,
  "msg": "Merchant not found",
  "sign": ""
}
```

Success:

```json
{
  "code": 200,
  "msg": "",
  "sign": "b449b4b6907204a683ec6c50bff92b01",
  "order_no": "p7158412025J2dZjXLmz0",
  "dis_order_no": "2025071130770572062498816myr1oushe",
  "create_time": 1752825512,
  "pay_url": "https://{api_domain}/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5. Pay-in Callback Notification `post/json`

Push URL: the `pay_notice_url` submitted by the merchant when placing the order. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 5.1 Pay-in Callback Request Parameters

| Name         | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| trade_no     | int    | true     | Merchant ID |
| status       | int    | true     | Order status, <span style="color:red;">`2` = success</span>, `3` = failed |
| order_no     | string | true     | Merchant order number |
| dis_order_no | string | true     | Platform order number |
| order_price  | int    | true     | Order amount, unit: Sen, integer. `1 RM = 100 Sen` |
| <span style="color:red;">real_price</span>   | int    | true     | <span style="color:red;">Actual amount paid by the user, unit: Sen, integer. `1 RM = 100 Sen`</span> |
| nti_time     | int    | false    | Notification time |
| payer        | string | false    | JSON string of payer information: `{"name":"name","account":"account","bank":"payer bank code","utr2":"bank reference number","email":"email","phone":"phone number","identify_type":"ID type","identify_num":"ID number"}`. In addition to the sample fields, this parameter may also include payer-related information merged from the merchant-provided `attach` field |
| pay_info     | string | false    | Payment info JSON string, for example: payer/payee native information, card number, name, bank, etc. `25-10-28` |
| create_time  | int    | true     | Creation time |
| sign         | string | true     | Signature result, see the signature method at the top of this document |

- Pay-in Callback Request Example

```json
{
  "trade_no": 10238,
  "status": 2,
  "order_no": "RCG0126042435820040804784",
  "dis_order_no": "Meg1352644s0800myrnVMR",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1776680622,
  "payer": "{\"account_no\":\"1234567890\",\"account_type\":\"BANK\",\"identify_num\":\"900101-14-5678\",\"identify_type\":\"NRIC\",\"req_api_ip\":\"203.0.113.15\",\"utr2\":\"9901108391\"}",
  "create_time": 1776680593,
  "sign": "7b23565a3dc790b6e55f29f0f0cf5f1a"
}
```

## 5.2 Pay-in Callback Response Description

If the callback is processed successfully, please return <span style="color:red;">`success`</span>. The system will stop pushing this order notification; otherwise, it will continue to retry.

# 6. Pay-out Order Interface

(The order placement IP must be whitelisted by contacting us)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayOutOrder`

## 6.1 Pay-out Request Parameters

<span style="color:red;">A pay-out request is considered successful unless an explicit failure is returned, even if it times out. Do not submit the request again unless an explicit failure is returned.</span>

| Name           | Type   | Required | Description |
| -------------- | ------ | -------- | ----------- |
| trade_no       | int    | true     | Merchant ID |
| order_no       | string | true     | Merchant order number |
| app_id         | int    | true     | Merchant `appId` |
| pay_code       | int    | true     | Product code, please contact our operations team |
| price          | int    | true     | Order amount, unit: Sen, integer. `1 RM = 100 Sen` |
| account_no     | string | true     | Recipient account number |
| account_name   | string | true     | Recipient name |
| bank_code      | string | true     | Bank code |
| pay_notice_url | string | false    | Pay-out success callback URL |
| attach         | string | false    | Additional parameters in JSON string format |
| user_ip        | string | false    | Recipient IP |
| sign           | string | true     | Signature result, see the signature method at the top of this document |
| timestamp      | string | false    | Order timestamp, 10-digit Unix timestamp in seconds |

- Pay-out `attach` Field Description

| Name  | Type   | Required | Description |
| ----- | ------ | -------- | ----------- |
| phone | string | false    | Phone number |

- Pay-out Request Example

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
  "user_ip": "203.0.113.15",
  "account_no": "1234567890",
  "account_name": "Lim Wei Ming",
  "bank_code": "MAY"
}
```

## 6.2 Pay-out Order Response

| Name         | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| code         | int    | true     | `200`: order created successfully, others: order creation failed |
| msg          | string | true     | Failure reason |
| dis_order_no | string | true     | Platform order number |
| order_no     | string | true     | Merchant order number |
| status       | int    | true     | Order status, `2` = pay-out success, `3` = pay-out failed, `7` = rejected, `9` = reversed, `10` = processing |
| create_time  | int    | true     | Creation time |
| sign         | string | true     | Signature result, see the signature method at the top of this document |

- Pay-out Order Response Example

Failure:

```json
{
  "code": 1005,
  "msg": "Merchant not found",
  "sign": ""
}
```

Success:

```json
{
  "code": 200,
  "msg": "",
  "sign": "d3ec1fa0f45bc44218d5fb63bb1beb61",
  "order_no": "p7158412025MsJydJqT7b",
  "dis_order_no": "2025071130776296733810688myr1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7. Pay-out Callback Notification

Push URL: the `pay_notice_url` submitted by the merchant when placing the order. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 7.1 Pay-out Callback Request Parameters

| Name         | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| trade_no     | int    | true     | Merchant ID |
| order_no     | string | true     | Merchant order number |
| dis_order_no | string | true     | Platform order number |
| order_price  | int    | true     | Order amount, unit: Sen, integer. `1 RM = 100 Sen` |
| fee          | int    | false    | Order fee, unit: Sen, integer. `1 RM = 100 Sen` |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status       | int    | true     | Order status, <span style="color:red;">`2` = pay-out success</span>, `3` = pay-out failed, `7` = rejected, `9` = reversed |
| pay_info     | string | false    | Payment info JSON string, for example: payer/payee native information, card number, name, bank, `utr2`, etc. |
| remark       | string | false    | Failure reason |
| create_time  | int    | true     | Creation time |
| sign         | string | true     | Signature result, see the signature method at the top of this document |
| nti_time     | int    | true     | Notification time |

- Pay-out Callback Request Example

```json
{
  "trade_no": 10000,
  "status": 2,
  "order_no": "20060354339090013",
  "dis_order_no": "Meg2352644o2nmjo0800myrYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": "{\"utr2\":\"611011445289\"}"
}
```

## 7.2 Pay-out Callback Response Description

If the callback is processed successfully, please return <span style="color:red;">`success`</span>. The system will stop pushing this order notification; otherwise, it will continue to retry.

# 8. Query Order Interface (Shared by Pay-in and Pay-out)

(The request IP must be whitelisted by contacting us)
Query URL: `https://{api_domain}/api/v1/payApi/QueryOrder`

## 8.1 Query Request Parameters

| Name         | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| order_type   | string | true     | `pay_out`: pay-out, `pay_in`: pay-in |
| trade_no     | int    | true     | Merchant ID |
| app_id       | int    | true     | Merchant `appId` |
| dis_order_no | string | false    | Platform order number |
| order_no     | string | false    | Merchant order number |
| sign         | string | true     | Signature result, see the signature method at the top of this document |

- Query Request Example

```json
{
  "order_type": "pay_in",
  "trade_no": 165,
  "app_id": 165,
  "dis_order_no": "p7158277185f96603047656571",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 8.2 Query Response

| Name         | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| code         | int    | true     | `200`: query successful, others: failed |
| msg          | string | true     | Query failure reason |
| trade_no     | int    | true     | Merchant ID |
| <span style="color:red;">real_price</span>   | int    | true     | <span style="color:red;">Actual amount paid, unit: Sen, integer. `1 RM = 100 Sen`</span> |
| status       | int    | true     | Order status, `1` = unpaid, <span style="color:red;">`2` = success</span>, `3` = failed, `7` = rejected, `9` = reversed, `10` = processing |
| success_time | int    | true     | Success timestamp |
| order_no     | string | true     | Merchant order number |
| dis_order_no | string | true     | Platform order number |
| remark       | string | true     | Pay-out failure reason |
| fee          | int    | false    | Order fee, unit: Sen, integer. `1 RM = 100 Sen` |
| create_time  | int    | true     | Creation time |
| payer        | string | false    | JSON string of payer information: `{"account_name":"name","account_type":"account type","account_no":"account","identify_type":"ID type","identify_num":"ID number"}` |
| pay_info     | string | false    | Payment info JSON string, for example: payer/payee native information, card number, name, bank, etc. `25-10-28` |
| sign         | string | true     | Signature result, see the signature method at the top of this document |
| utr2         | string | false    | Bank order number |

- Query Response Example

Failure:

```json
{
  "code": 1017,
  "msg": "Order does not exist"
}
```

Success:

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
  "payer": "{\"name\":\"Lim Wei Ming\",\"email\":\"lim.weiming@example.com\",\"phone\":\"0123456789\",\"identify_type\":\"NRIC\",\"identify_num\":\"900101-14-5678\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Pay-out Balance Query Interface

(The request IP must be whitelisted by contacting us)
URL: `https://{api_domain}/api/v1/payApi/QueryBalance`

## 9.1 Balance Request Parameters

| Name     | Type   | Required | Description |
| -------- | ------ | -------- | ----------- |
| trade_no | int    | true     | Merchant ID |
| app_id   | int    | true     | Merchant `appid` |
| sign     | string | true     | Signature result, see the signature method at the top of this document |

- Balance Request Example

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Response

| Name           | Type   | Required | Description |
| -------------- | ------ | -------- | ----------- |
| code           | int    | true     | `200`: query successful, others: failed |
| msg            | string | true     | Failure reason |
| balance        | int    | true     | Balance, unit: Sen, integer. `1 RM = 100 Sen` |
| balance_frozen | int    | false    | Frozen balance, unit: Sen, integer. `1 RM = 100 Sen` |
| sign           | string | true     | Signature result, see the signature method at the top of this document |

- Balance Response Example

Failure:

```json
{
  "code": 10001,
  "msg": "Merchant does not exist"
}
```

Success:

```json
{
  "code": 200,
  "msg": "success",
  "balance": 10000,
  "balance_frozen": 1000,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 10. Payment Voucher Query Interface

(The request IP must be whitelisted by contacting us)
URL: `https://{api_domain}/api/v1/payApi/QueryCertificate`

## 10.1 Payment Voucher Request Parameters

| Name         | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| trade_no     | int    | true     | Merchant ID |
| app_id       | int    | true     | Merchant `appid` |
| order_no     | string | false    | Merchant order number, choose one of `order_no` or `dis_order_no` |
| dis_order_no | string | false    | Platform order number, choose one of `dis_order_no` or `order_no` |
| sign         | string | true     | Signature result, see the signature method at the top of this document |

- Payment Voucher Request Example

```json
{
  "trade_no": 10003,
  "app_id": 10003,
  "order_no": "",
  "dis_order_no": "35011C02gljuf6k0800myr1lVY",
  "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```

## 10.2 Payment Voucher Response Parameters

| Name     | Type   | Required | Description |
| -------- | ------ | -------- | ----------- |
| code     | int    | true     | `200`: response successful, others: failed |
| msg      | string | true     | Failure reason |
| img_link | string | false    | Voucher link |
| img_base | string | false    | Base64 content of the generated voucher |
| sign     | string | true     | Signature result, see the signature method at the top of this document |

- Payment Voucher Response Example
  No payment voucher available

```json
{
  "code": 200,
  "msg": "No payment voucher available at the moment",
  "sign": "",
  "img_link": "",
  "img_base": ""
}
```

Payment voucher available

```json
{
  "code": 200,
  "msg": "",
  "sign": "3969f17cd1a551769f85967d0a05b7b6",
  "img_link": "http://dsggfgdsf.djdj?ddd=snn",
  "img_base": "data:image/png;base64,hfhshdhfhfh"
}
```

# 11. Error Codes

| Status Code | Description |
| ----------- | ----------- |
| 200         | Success |
| 1000        | Internal error |
| 1001        | IP is not in the merchant IP whitelist |
| 1002        | Parameter error |
| 1003        | Signature error |
| 1004        | This interface is not currently enabled for the merchant (please contact operations to verify whether the merchant or app does not exist, is disabled, or the payment product is not configured) |
| 1005        | Merchant does not exist |
| 1006        | Current user IP is in the blacklist |
| 1007        | Current user is in the blacklist |
| 1008        | Merchant app does not exist |
| 1009        | Payment product does not exist |
| 1010        | Payment channel does not exist |
| 1011        | The payment channel has not been fully developed and is temporarily unavailable |
| 1012        | Payment channel exception, please try again later |
| 1013        | Current order volume is too high, please try again later |
| 1014        | Duplicate order number |
| 1015        | Insufficient app balance |
| 1016        | The same user is placing orders too frequently, please try again later |
| 1017        | Order record does not exist |
| 1018        | The current amount is not supported |
| 1019        | Pay-in is not yet enabled for the current app country |
| 1020        | Pay-out is not yet enabled for the current app country |
| 1021        | Failed |
| 1036        | Interface is not yet open |
| 1037        | This currency is not supported |
| 1038        | Abnormal UTR returned in pay-in callback |
| 9999        | Other error |
| 3000        | System maintenance in progress, order placement is suspended, please try again later |

# 12. Pay-in Cashier Interface

URL: `https://{api_domain}/api/v1/cashApi/CashIn.html`
Request Method: `GET`

### Parameters:

| Name       | Type   | Required | Description |
| ---------- | ------ | -------- | ----------- |
| app_id     | string | true     | Merchant `app_id` |
| order_no   | string | true     | Merchant order number |
| amount     | string | true     | Merchant amount, unit: Ringgit (`RM`) |
| notice_url | string | false    | Asynchronous callback URL |
| pay_code   | int    | true     | Product code |

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{merchant_order_no}}&amount={{merchant_amount}}&notice_url={{callback_url}}&pay_code={{product_code}}
```

# 13. Pay-in Payment Method `pay_method`

| Field Name | Value    | Description |
| ---------- | -------- | ----------- |
| pay_method | MYR-Payment  | Malaysia pay-in |


# 14. Pay-in Bank Codes `attach.bank_code`

| Field Name | Code | Description |
|:-----------------|:----------| :------- |
| attach.bank_code | SUMITOMO  | SUMITOMO MITSUI CORPORATION Bank |
| attach.bank_code        | MBSB      | MBSB Bank |
| attach.bank_code        | FINEXUS   | FINEXUS CARDS Bank |
| attach.bank_code        | MUOB      | UOB |
| attach.bank_code        | BIMB      | Bank Islam Malaysia |
| attach.bank_code        | MJPMORGAN | J.P. MORGAN CHASE Bank |
| attach.bank_code        | MBOC      | OF CHINA Bank |
| attach.bank_code        | RAKYAT    | KERJASAMA RAKYAT MALAYSIA Bank |
| attach.bank_code        | AGRO      | AGRO |
| attach.bank_code        | MHSBC     | HSBC |
| attach.bank_code        | MCITI     | CITI Bank |
| attach.bank_code        | BI        | ISLAM Bank |
| attach.bank_code        | ICBC      | INDUSTRIAL AND COMMERCIAL OF CHINA (ICBC) Bank |
| attach.bank_code        | PBE       | Public Bank Berhad |
| attach.bank_code        | MSCB      | STANDARD CHARTERED Bank |
| attach.bank_code        | MCB       | MIZUHO CORPORATE Bank |
| attach.bank_code        | KFH       | KUWAIT FINANCE HOUSE Bank |
| attach.bank_code        | MBNP      | BNP PARIBAS MALAYSIA Bank |
| attach.bank_code        | MOCBC     | OCBC |
| attach.bank_code        | BIGPAY    | BIGPAY Bank |
| attach.bank_code        | BKK       | BANGKOK Bank |
| attach.bank_code        | BMMB      | Bank Muamalate |
| attach.bank_code        | AM        | AmBank |
| attach.bank_code        | GX        | GXbank |
| attach.bank_code        | MBOA      | OF AMERICA Bank |
| attach.bank_code        | BSN       | BSN |
| attach.bank_code        | BKRM      | Bank Rakyat |
| attach.bank_code        | ALLIANCE  | Alliance Bank Malaysia Berhad |
| attach.bank_code        | TNG       | TOUCH N GO EWALLET Bank |
| attach.bank_code        | MUFGB     | MUFG Bank |
| attach.bank_code        | MCCB      | CHINA CONSTRUCTION (CCB) Bank |
| attach.bank_code        | RAJHI     | AL-RAJHI Bank |
| attach.bank_code        | AFFIN     | Affin Bank |
| attach.bank_code        | MHLB      | Hong Leong Bank |
| attach.bank_code        | RHB       | RHB Bank |
| attach.bank_code        | MCIMB     | CIMB Bank |
| attach.bank_code        | MAY       | Maybank |
| attach.bank_code        | RYT       | RYT Bank |

# 15. Pay-out Bank Codes `bank_code`

| Field Name | Code      | Bank Name |
| :--------- | :-------- | :-------- |
| bank_code | SUMITOMO   | SUMITOMO MITSUI CORPORATION Bank |
| bank_code | MBSB       | MBSB Bank |
| bank_code | FINEXUS    | FINEXUS CARDS Bank |
| bank_code | MUOB       | UOB |
| bank_code | BIMB       | Bank Islam Malaysia |
| bank_code | MJPMORGAN  | J.P. MORGAN CHASE Bank |
| bank_code | MBOC       | OF CHINA Bank |
| bank_code | RAKYAT     | KERJASAMA RAKYAT MALAYSIA Bank |
| bank_code | AGRO       | AGRO |
| bank_code | MHSBC      | HSBC |
| bank_code | MCITI      | CITI Bank |
| bank_code | BI         | ISLAM Bank |
| bank_code | ICBC       | INDUSTRIAL AND COMMERCIAL OF CHINA (ICBC) Bank |
| bank_code | PBE        | Public Bank Berhad |
| bank_code | MSCB       | STANDARD CHARTERED Bank |
| bank_code | MCB        | MIZUHO CORPORATE Bank |
| bank_code | KFH        | KUWAIT FINANCE HOUSE Bank |
| bank_code | MBNP       | BNP PARIBAS MALAYSIA Bank |
| bank_code | MOCBC      | OCBC |
| bank_code | BIGPAY     | BIGPAY Bank |
| bank_code | BKK        | BANGKOK Bank |
| bank_code | BMMB       | Bank Muamalate |
| bank_code | AM         | AmBank |
| bank_code | GX         | GXbank |
| bank_code | MBOA       | OF AMERICA Bank |
| bank_code | BSN        | BSN |
| bank_code | BKRM       | Bank Rakyat |
| bank_code | ALLIANCE   | Alliance Bank Malaysia Berhad |
| bank_code | TNG        | TOUCH N GO EWALLET Bank |
| bank_code | MUFGB      | MUFG Bank |
| bank_code | MCCB       | CHINA CONSTRUCTION (CCB) Bank |
| bank_code | RAJHI      | AL-RAJHI Bank |
| bank_code | AFFIN      | Affin Bank |
| bank_code | MHLB       | Hong Leong Bank |
| bank_code | RHB        | RHB Bank |
| bank_code | MCIMB      | CIMB Bank |
| bank_code | MAY        | Maybank |
| bank_code | RYT       | RYT Bank |

# 16. Document Update Time

```
2026-08-24 13:05:52
```
