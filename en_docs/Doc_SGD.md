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
> 2. The amount unit is <span style="color:red;"> Cents </span>. `1 Singapore dollar (SGD) = 100 cents`.
>
> 3. The request IP must be whitelisted.
>
> 4. Please collect the user's real IP for `user_ip` whenever possible. If it is not available, leave it blank. Do not use local IPs such as `127.0.0.1`.

## 3.2 Callback Related

> 1. If the callback is processed successfully, please return the plain text <span style="color:red;">`success`</span> without any extra characters. The system will stop pushing this order notification; otherwise, the system will continue to push the order notification repeatedly.
>
> 2. During asynchronous notification processing, if the received response is not `success`, the notification will be considered failed and retried periodically. The retry schedule is: `1m`, `1m`, `4m`, `10m`, `10m`, `1h`, `2h`, `6h`, `15h`.
>
> 3. If `pay_notice_url` is empty, it will be regarded as no callback required by the merchant, and the system will not push notifications.

# 4. Pay-in Order Interface

(The order placement IP must be whitelisted by contacting us)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayInOrder`

## 4.1 Pay-in Order Request Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant `appId` |
| pay_code | int | true | Product code, please contact our operations team |
| pay_method | string | true | Payment method |
| price | int | true | Order amount, unit: cents, integer. `1 Singapore dollar (SGD) = 100 cents` |
| order_no | string | true | Merchant order number |
| success_url | string | false | Redirect URL after successful payment |
| fail_url | string | false | Redirect URL after failed payment |
| pay_notice_url | string | false | Payment success callback URL |
| user_id | string | false | Merchant user ID |
| user_ip | string | false | Payer IP |
| attach | string | true | Additional parameters in JSON string format, payer information: `{"name":"name","bank_code":"PayNow_SGD","phone":"3211234567"}` |
| sign | string | true | Signature result, see the signature method at the top of this document |
| timestamp | string | false | Order timestamp, 10-digit Unix timestamp in seconds |

- Pay-in `attach` Field Description

```json
{"name":"Tan Wei Ming","bank_code":"PayNow_SGD","phone":"91234567"}
```

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| name | string | true | Payer name |
| bank_code | string | true | Bank code, see the pay-in bank codes below |
| phone | string | false | Payer phone number |

- Pay-in Order Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"name\":\"Tan Wei Ming\",\"account_no\":\"S1234567D\",\"bank_code\":\"PayNow_SGD\",\"phone\":\"91234567\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}
```

## 4.2 Pay-in Order Response

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| code | int | true | `200`: order created successfully, others: order creation failed |
| msg | string | true | Failure reason |
| pay_url | string | false | Payment URL |
| qr_code | string | false | QR code string |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| create_time | int | true | Creation time |
| pay_info | string | false | Payment info JSON string, for example: payer/payee native information, card number, name, bank, etc. `{"pay_raw":"native payment info, which the merchant can convert into a QR code","phone":"phonepe deep link","google":"google deep link","paytm":"paytm deep link","mobik":"mobik deep link","bhim":"bhim deep link","upi":"upiLink"}` |
| sign | string | true | Signature result, see the signature method at the top of this document |

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
  "dis_order_no": "2025071130770572062498816sgd1oushe",
  "create_time": 1752825512,
  "pay_url": "https://api.sunpayinr.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5. Pay-in Callback Notification `post/json`

Push URL: the `pay_notice_url` submitted by the merchant when placing the order. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 5.1 Pay-in Callback Request Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| trade_no | int | true | Merchant ID |
| status | int | true | Order status, <span style="color:red;">`2` = success</span>, `3` = failed |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| order_price | int | true | Order amount, unit: cents |
| <span style="color:red;">real_price</span> | int | true | <span style="color:red;">Actual amount paid by the user, unit: cents</span> |
| nti_time | int | false | Notification time |
| payer | string | false | JSON string of payer information: `{"name":"name","account":"account","bank":"payer bank code","utr2":"bank reference number","email":"email","phone":"phone number","identify_type":"ID type","identify_num":"NRIC,FIN,PASSPORT"}`. In addition to the sample fields, this parameter includes payer-related information merged from the merchant-provided `attach` field |
| pay_info | string | false | Payment info JSON string, for example: payer/payee native information, card number, name, bank, etc. `25-10-28` |
| create_time | int | true | Creation time |
| sign | string | true | Signature result, see the signature method at the top of this document |

- Pay-in Callback Request Example

```json
{
  "trade_no": 10238,
  "status": 2,
  "order_no": "RCG0126042435820040804784",
  "dis_order_no": "Meg1352644s0800sgdVMR",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1776680622,
  "payer": "{\"account_no\":\"S1234567D@gmail.com\",\"account_type\":\"EMAIL\",\"identify_num\":\"7171\",\"identify_type\":\"BIC\",\"req_api_ip\":\"\",\"utr2\":\"SGD9901108391\"}",
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

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| trade_no | int | true | Merchant ID |
| order_no | string | true | Merchant order number |
| app_id | int | true | Merchant `appId` |
| pay_code | int | true | Product code, please contact our operations team |
| price | int | true | Order amount, unit: cents, integer. `1 Singapore dollar (SGD) = 100 cents` |
| account_no | string | true | Recipient account number |
| account_type | string | true | Account type: `PHONE` (e-wallet phone number), `BANK` (bank account) |
| account_name | string | true | Name |
| bank_code | string | true | Recipient bank/wallet code, see the pay-out bank codes below |
| pay_notice_url | string | false | Pay-out success callback URL |
| attach | string | true | Additional parameters: `{"email":"email","phone":"phone number","pay_type":"payment method"}` |
| user_ip | string | false | Recipient IP |
| sign | string | true | Signature result, see the signature method at the top of this document |
| timestamp | string | false | Order timestamp, 10-digit Unix timestamp in seconds |

- Pay-out `attach` Field Description

```json
{"email":"limmeiling@gmail.com","phone":"98765432","pay_type":"SGD_PAYNOW"}
```

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| pay_type | string | true | Payment method, see the pay-out payment methods below |
| phone | string | false | Recipient phone number |
| email | string | false | Email address |

- Pay-out Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"email\":\"limmeiling@gmail.com\",\"phone\":\"98765432\",\"pay_type\":\"SGD_PAYNOW\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "87.200.59.100",
  "account_type": "PHONE",
  "account_no": "98765432",
  "account_name": "Lim Mei Ling",
  "bank_code": "PayNow_SGD"
}
```

## 6.2 Pay-out Order Response

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| code | int | true | `200`: order created successfully, others: order creation failed |
| msg | string | true | Failure reason |
| dis_order_no | string | true | Platform order number |
| order_no | string | true | Merchant order number |
| status | int | true | Order status: `2` = pay-out successful, `3` = pay-out failed, `7` = rejected, `9` = reversed, `10` = processing |
| create_time | int | true | Creation time |
| sign | string | true | Signature result, see the signature method at the top of this document |

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
  "dis_order_no": "2025071130776296733810688sgd1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7. Pay-out Callback Notification

Push URL: the `pay_notice_url` submitted by the merchant when placing the order. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 7.1 Pay-out Callback Request Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| trade_no | int | true | Merchant ID |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| order_price | int | true | Order amount, unit: cents |
| fee | int | false | Order handling fee, unit: cents |
| <span style="color:red;">real_price</span> | int | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span> |
| status | int | true | Order status, <span style="color:red;">`2` = pay-out successful</span>, `3` = pay-out failed, `7` = rejected, `9` = reversed |
| pay_info | string | false | Payment info JSON string, for example: payer/payee native information, card number, name, bank, `utr2`, etc. |
| remark | string | false | Failure reason |
| create_time | int | true | Creation time |
| sign | string | true | Signature result, see the signature method at the top of this document |
| nti_time | int | true | Notification time |

- Pay-out Callback Request Example

```json
{
  "trade_no": 10000,
  "status": 2,
  "order_no": "20060354339090013",
  "dis_order_no": "Meg2352644o2nmjo0800sgdYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": "{\"utr2\":\"SGD611011445289\"}"
}
```

## 7.2 Pay-out Callback Response Description

If the callback is processed successfully, please return <span style="color:red;">`success`</span>. The system will stop pushing this order notification; otherwise, it will continue to retry.

# 8. Order Query Interface (Shared by Pay-in and Pay-out)

(The request IP must be whitelisted by contacting us)
Query URL: `https://{api_domain}/api/v1/payApi/QueryOrder`

## 8.1 Query Request Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| order_type | string | true | `pay_out`: pay-out, `pay_in`: pay-in |
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant `appId` |
| dis_order_no | string | false | Platform order number |
| order_no | string | false | Merchant order number |
| sign | string | true | Signature result, see the signature method at the top of this document |

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

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| code | int | true | `200`: query successful, others: failed |
| msg | string | true | Query failure reason |
| trade_no | int | true | Merchant ID |
| <span style="color:red;">real_price</span> | int | true | <span style="color:red;">Actual payment amount, unit: cents</span> |
| status | int | true | Order status, `1` = unpaid, <span style="color:red;">`2` = success</span>, `3` = failed, `7` = rejected, `9` = reversed, `10` = processing |
| success_time | int | true | Success timestamp |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| remark | string | true | Pay-out failure reason |
| fee | int | false | Order handling fee, unit: cents |
| create_time | int | true | Creation time |
| payer | string | false | JSON string of payer information: `{"account_name":"name","account_type":"account type:PHONE,BANK","account_no":"account","bank_code":"bank or wallet code"}` |
| pay_info | string | false | Payment info JSON string, for example: payer/payee native information, card number, name, bank, etc. `25-10-28` |
| sign | string | true | Signature result, see the signature method at the top of this document |
| utr2 | string | false | Bank order number |

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
  "payer": "{\"account_name\":\"Lim Mei Ling\",\"account_type\":\"PHONE\",\"account_no\":\"98765432\",\"bank_code\":\"PayNow_SGD\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Pay-out Balance Query Interface

(The request IP must be whitelisted by contacting us)
URL: `https://{api_domain}/api/v1/payApi/QueryBalance`

## 9.1 Balance Request Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant `appId` |
| sign | string | true | Signature result, see the signature method at the top of this document |

- Balance Request Example

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Response

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| code | int | true | `200`: query successful, others: failed |
| msg | string | true | Failure reason |
| balance | int | true | Balance, unit: cents |
| balance_frozen | int | false | Frozen balance, unit: cents |
| sign | string | true | Signature result, see the signature method at the top of this document |

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

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant `appId` |
| order_no | string | false | Merchant order number, provide either this or `dis_order_no` |
| dis_order_no | string | false | Platform order number, provide either this or `order_no` |
| sign | string | true | Signature result, see the signature method at the top of this document |

- Payment Voucher Request Example

```json
{
  "trade_no": 10003,
  "app_id": 10003,
  "order_no": "",
  "dis_order_no": "35011C02gljuf6k0800sgd1lVY",
  "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```

## 10.2 Payment Voucher Response Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| code | int | true | `200`: response successful, others: failed |
| msg | string | true | Failure reason |
| img_link | string | false | Voucher link |
| img_base | string | false | Base64 content of the generated voucher |
| sign | string | true | Signature result, see the signature method at the top of this document |

- Payment Voucher Response Example

No payment voucher available

```json
{
  "code":200,
  "msg":"No payment voucher available at the moment",
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

# 11. Payment Methods for Pay-in `pay_method`

| Field | Value | Description |
| --- | --- | --- |
| pay_method | BQR | QR code |
| pay_method | OB | Online banking |

# 12. Bank Codes for Pay-in `attach.bank_code`

| Field Name | Code | Description |
| :--- | :--- | :--- |
| bank_code | DASH_SGD | DASH |
| bank_code | LiquidPay_SGD | LiquidPay |
| bank_code | ABMB_SGD | Alliance Bank |
| bank_code | Wise_SGD | Wise |
| bank_code | TBL_SGD | Trust Bank |
| bank_code | RHB_SGD | RHB Bank |
| bank_code | MBBS_SGD | MayBank Singapore |
| bank_code | HSBCS_SGD | HSBC Bank Singapore |
| bank_code | HLB_SGD | HongLeong Bank |
| bank_code | CITIS_SGD | CitiBank Singapore |
| bank_code | BOC_SGD | Bank of China |
| bank_code | PayNow_SGD | PayNow |
| bank_code | POSB_SGD | The Development Bank of Singapore |
| bank_code | Maribank_SGD | Maribank |
| bank_code | GXS_SGD | GXS Bank |
| bank_code | SCBS_SGD | Standard Chartered Bank Singapore |
| bank_code | CIMB_SGD | CIMB Bank |
| bank_code | UOB_SGD | United Overseas Bank |
| bank_code | SDBS_SGD | DBS Bank Singapore |
| bank_code | OCBC_SGD | OCBC Bank |

# 13. Payment Methods for Pay-out `attach.pay_type`

| Field | Value | Description |
| --- | --- | --- |
| pay_type | WD | Payout-SGD |

# 14. Bank Codes for Pay-out `bank_code`

| Field Name | Code | Description |
| :--- | :--- | :--- |
| bank_code | DASH_SGD | DASH |
| bank_code | LiquidPay_SGD | LiquidPay |
| bank_code | ABMB_SGD | Alliance Bank |
| bank_code | Wise_SGD | Wise |
| bank_code | TBL_SGD | Trust Bank |
| bank_code | RHB_SGD | RHB Bank |
| bank_code | MBBS_SGD | MayBank Singapore |
| bank_code | HSBCS_SGD | HSBC Bank Singapore |
| bank_code | HLB_SGD | HongLeong Bank |
| bank_code | CITIS_SGD | CitiBank Singapore |
| bank_code | BOC_SGD | Bank of China |
| bank_code | PayNow_SGD | PayNow |
| bank_code | POSB_SGD | The Development Bank of Singapore |
| bank_code | Maribank_SGD | Maribank |
| bank_code | GXS_SGD | GXS Bank |
| bank_code | SCBS_SGD | Standard Chartered Bank Singapore |
| bank_code | CIMB_SGD | CIMB Bank |
| bank_code | UOB_SGD | United Overseas Bank |
| bank_code | SDBS_SGD | DBS Bank Singapore |
| bank_code | OCBC_SGD | OCBC Bank |

# 15. Error Codes

| Status Code | Description |
| --- | --- |
| 200 | Success |
| 1000 | Internal error |
| 1001 | IP is not in the merchant IP whitelist |
| 1002 | Parameter error |
| 1003 | Signature error |
| 1004 | This interface is not currently enabled for the merchant (contact operations to verify whether the merchant or App does not exist, has been disabled, or has no payment product configured) |
| 1005 | Merchant does not exist |
| 1006 | The current user IP is blacklisted |
| 1007 | The current user is blacklisted |
| 1008 | Merchant App does not exist |
| 1009 | Payment product does not exist |
| 1010 | Payment channel does not exist |
| 1011 | Payment channel development is not yet complete; the channel is temporarily unavailable |
| 1012 | Payment channel error, please try again later |
| 1013 | Order volume is currently too high, please try again later |
| 1014 | Duplicate order number |
| 1015 | Insufficient App balance |
| 1016 | The same user is placing orders too frequently, please try again later |
| 1017 | Order record does not exist |
| 1018 | The current amount is not supported |
| 1019 | Pay-in order placement is not yet enabled in the country of the current App |
| 1020 | Pay-out order placement is not yet enabled in the country of the current App |
| 1021 | Failed |
| 1036 | Interface not yet enabled |
| 1037 | Currency not supported |
| 1038 | Error in pay-in UTR submission |
| 9999 | Other error |
| 3000 | The system is undergoing an upgrade and maintenance. Order placement is suspended; please try again later |

# 16. Pay-in Cashier Interface

URL: `https://{api_domain}/api/v1/cashApi/CashIn.html`
Request Method: `GET`

### Parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| app_id | string | true | Merchant `app_id` |
| order_no | string | true | Merchant order number |
| amount | string | true | Merchant amount, unit: Singapore dollars (SGD) |
| notice_url | string | false | Asynchronous notification URL |
| pay_code | int | true | Product code |

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{MerchantOrderNumber}}&amount={{MerchantAmount}}&notice_url={{NotificationURL}}&pay_code={{ProductCode}}
```

# 17. Document Update Time

```
2026-09-20 10:00:00
```
