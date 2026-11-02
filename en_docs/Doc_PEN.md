# 1. Integration Process

> 1. Conduct business negotiation for account opening and discuss the relevant rates.
>
> 2. Contact operations to create the merchant ID, secret key, merchant `appId`, product code, and `apiUrl`.
>
> 3. After development is completed, both parties conduct joint testing to verify that requests, reporting, and other information are complete.

# 2. MD5 Signature Algorithm

> 1. Sort all parameters in ascending order as key-value pairs (`key1=value1`). Parameters with empty values do not participate in the signature.
>
> 2. Concatenate them in the format `key1=value1&key2=value2`.
>
> 3. Append the merchant secret key: `key1=value1&key2=value2...&key=MerchantSecretKey`.
>
> 4. `sign=md5(the string assembled in the previous step)`. The signature result is a 32-character lowercase string.
>
> 5. The signature key can be found in the merchant backend under Basic Information, or by contacting our customer service.

# 3. Notes

## 3.1 Interface Related
> 1. All interfaces in this document use the standard HTTP communication protocol with POST requests. The Content-type for both request and response is `application/json`, and the character encoding is UTF-8.
>
> 2. The amount unit is <span style="color:red;"> centimos </span>.
>
> 3. The request IP must be whitelisted.
>
> 4. Please collect the user's real IP address for `user_ip` as much as possible. If unavailable, leave it blank. Do not use local IP addresses such as `127.0.0.1`.

## 3.2 Callback Related
> 1. If the callback is received and processed successfully, please return the plain text <span style="color:red;">`success`</span> without any extra characters. Otherwise, the system will continue to retry the notification.
>
> 2. During asynchronous notification processing, if the response is not `success`, the notification is considered failed and will be retried according to the following schedule: `1m`, `1m`, `4m`, `10m`, `10m`, `1h`, `2h`, `6h`, `15h`.
>
> 3. If `pay_notice_url` is empty, it is treated as no callback required, and the system will not push notifications.

# 4. Pay-in Order Interface

(The request IP for order creation must be whitelisted by contacting us)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayInOrder`

## 4.1 Pay-in Order Request Parameters

| Name | Type | Required | Description |
|----------------|------|-----|---------------------------------------------------------------------------------------------------------|
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant `appId` |
| pay_code | int | true | Product code, please contact our operations team |
| pay_method | string | true | Payment method, see the payment methods below |
| price | int | true | Order amount, unit: centimos, integer. `1 Peruvian Sol = 100 centimos` |
| order_no | string | true | Merchant order number |
| success_url | string | false | Redirect URL after successful payment |
| fail_url | string | false | Redirect URL after failed payment |
| pay_notice_url | string | false | Payment success callback URL |
| user_id | string | false | Merchant user ID |
| user_ip | string | false | Payer IP |
| attach | string | true | Additional parameters in JSON string format: payer information `{"name":"Name","identify_type":"ID type","identify_num":"ID number","phone":"Phone","email":"Email"}` |
| sign | string | true | Signature result, see the signature method at the top of the document |
| timestamp | string | false | Order timestamp, 10-digit Unix timestamp in seconds |

- Pay-in `attach` field description

| Name | Type | Required | Description |
|---------------|--------|-------|-------------------------------------------------------------------------|
| name | string | true | Payer name |
| identify_type | string | true | Supported ID types `(DNI,CE,RUC,PAS)` `"DNI":"National ID","CE":"Foreigner Resident Card","RUC":"Tax ID","PAS":"Passport"` |
| identify_num | string | true | ID number |
| phone | string | false | Mobile number, preferably a real mobile number; can be filled if unavailable |
| email | string | false | Email address, preferably a real email address; can be filled if unavailable |
| product_name | string | false | Product name |

- Pay-in order request example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"name\":\"Zhang San\",\"identify_type\":\"DNI\",\"identify_num\":\"44030419900101001X\",\"phone\":\"13800000000\",\"email\":\"zhangsan@example.com\",\"product_name\":\"Test Product\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}
```

## 4.2 Pay-in Order Response

| Name | Type | Required | Description |
|------------|------|----|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code | int | true | `200`: order created successfully, others: order creation failed |
| msg | string | true | Failure reason |
| pay_url | string | false | Payment URL |
| qr_code | string | false | QR code string |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| create_time | int | true | Creation time |
| pay_info | string | false | Payment information JSON string, for example: `{"pay_raw":"Native payment info","redirect_url":"Payment redirect URL"}` |
| sign | string | true | Signature result, see the signature method at the top of the document |

- Pay-in order response example

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
  "dis_order_no": "2025071130770572062498816peru1oushe",
  "create_time": 1752825512,
  "pay_url": "https://api.sunpaypen.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5. Pay-in Callback Notification

Push URL: the `pay_notice_url` submitted by the merchant when creating the order. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 5.1 Pay-in Callback Request Parameters

| Name | Type | Required | Description |
|------------|------|-----|--------------------------------------------------------------------------------------------------------|
| trade_no | int | true | Merchant ID |
| status | int | true | Order status: <span style="color:red;">`2` success</span>, `3` failed |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| order_price | int | true | Order amount, unit: centimos |
| <span style="color:red;">real_price</span> | int | true | <span style="color:red;">Actual amount paid by the user, unit: centimos</span> |
| nti_time | int | false | Notification time |
| payer | string | false | JSON string of payer information: `{"name":"Name","account":"Account","bank":"Payer bank code","utr2":"Bank reference number","email":"Email","phone":"Phone","identify_type":"ID type","identify_num":"ID number"}`. In addition to the example fields, payer-related fields passed in `attach` may also be merged into this parameter |
| pay_info | string | false | Payment information JSON string, for example: native payment info, card number, name, bank, etc. |
| create_time | int | true | Creation time |
| sign | string | true | Signature result, see the signature method at the top of the document |

- Pay-in callback request example

```json
{
  "trade_no": 10238,
  "status": 2,
  "order_no": "RCG0126042435820040804784",
  "dis_order_no": "Meg1352644s0800perunVMR",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1776680622,
  "payer": "{\"account_no\":\"1234567890\",\"account_type\":\"BANK\",\"identify_num\":\"\",\"identify_type\":\"\",\"req_api_ip\":\"\",\"utr2\":\"9901108391\"}",
  "create_time": 1776680593,
  "sign": "7b23565a3dc790b6e55f29f0f0cf5f1a"
}
```

## 5.2 Pay-in Callback Response
If the callback is received and processed successfully, please return <span style="color:red;">`success`</span>. The system will stop retrying this order notification. Otherwise, it will continue retrying.

# 6. Pay-out Order Interface

(The request IP for order creation must be whitelisted by contacting us)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayOutOrder`

## 6.1 Pay-out Request Parameters

<span style="color:red;">A pay-out request is considered successful unless an explicit failure is returned, even if it times out. Do not submit the request again unless an explicit failure is returned.</span>

| Name | Type | Required | Description                                                                                                      |
|----------------|--------|-------|------------------------------------------------------------------------------------------------------------------|
| trade_no | int | true | Merchant ID                                                                                                      |
| order_no | string | true | Merchant order number                                                                                            |
| app_id | int | true | Merchant `appId`                                                                                                 |
| pay_code | int | true | Product code, please contact our operations team                                                                 |
| price | int | true | Order amount, unit: centimos, integer. `1 Peruvian Sol = 100 centimos`                                           |
| account_no | string | true | Recipient account number                                                                                         |
| account_type | string | true | Account type `(CA,SA)` `"CA":"Checking account","SA":"Savings account"`                                          |
| account_name | string | true | Recipient name                                                                                                   |
| bank_code | string | true | Bank code                                                                                                        |
| identify_type | string | true | ID types `(DNI,CE,RUC,PAS)` `"DNI":"National ID","CE":"Foreigner Resident Card","RUC":"Tax ID","PAS":"Passport"` |
| identify_num | string | true | ID number                                                                                                        |
| pay_notice_url | string | false | Pay-out success callback URL                                                                                     |
| attach | string | true | Additional parameters JSON string                                                                                |
| user_ip | string | false | Recipient IP                                                                                                     |
| sign | string | true | Signature result, see the signature method at the top of the document                                            |
| timestamp | string | false | Order timestamp, 10-digit Unix timestamp in seconds                                                              |

- Pay-out `attach` field description

```json
{"email":"Email","phone":"Phone","bank_code":"Bank code","pay_type":"Payment method"}
```

| Name     | Type | Required | Description |
|----------|--------|------|---------------|
| phone    | string | true | Recipient mobile number |
| email    | string | true | Email address |
| pay_type | string | true | Payment method, see the pay-out payment methods below |
| cci      | string | true | `CCI` account |

- Pay-out request example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"email\":\"Email\",\"phone\":\"Phone\",\"cci\":\"CCI number\",\"pay_type\":\"Payment method\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "87.200.59.100",
  "account_type": "SA",
  "account_no": "12345678901234",
  "account_name": "Juan Perez",
  "bank_code": "BCP",
  "identify_type": "DNI",
  "identify_num": "12345678"
}
```

## 6.2 Pay-out Order Response

| Name | Type | Required | Description |
|------------|------|----|---------------------------------------------|
| code | int | true | `200`: order created successfully, others: order creation failed |
| msg | string | true | Failure reason |
| dis_order_no | string | true | Platform order number |
| order_no | string | true | Merchant order number |
| status | int | true | Order status: `2` pay-out success, `3` pay-out failed, `7` rejected, `9` reversed, `10` processing |
| create_time | int | true | Creation time |
| sign | string | true | Signature result, see the signature method at the top of the document |

- Pay-out order response example

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
  "dis_order_no": "2025071130776296733810688peru1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7. Pay-out Callback Notification

Push URL: the `pay_notice_url` submitted by the merchant when creating the order. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 7.1 Pay-out Callback Request Parameters

| Name | Type | Required | Description |
|------------|------|-----|-----------------------------------------------|
| trade_no | int | true | Merchant ID |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| order_price | int | true | Order amount, unit: centimos |
| fee | int | false | Order fee, unit: centimos |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status | int | true | Order status: <span style="color:red;">`2` pay-out success</span>, `3` pay-out failed, `7` rejected, `9` reversed |
| pay_info | string | false | Payment information JSON string, for example: native payment info, card number, name, bank, `utr2`, etc. |
| remark | string | false | Failure reason |
| create_time | int | true | Creation time |
| sign | string | true | Signature result, see the signature method at the top of the document |
| nti_time | int | true | Notification time |

- Pay-out callback request example

```json
{
  "trade_no": 10000,
  "status": 2,
  "order_no": "20060354339090013",
  "dis_order_no": "Meg2352644o2nmjo0800peruYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": "{\"utr2\":\"611011445289\"}"
}
```

## 7.2 Pay-out Callback Response
If the callback is received and processed successfully, please return <span style="color:red;">`success`</span>. The system will stop retrying this order notification. Otherwise, it will continue retrying.

# 8. Query Order Interface (Shared by Pay-in and Pay-out)

(The request IP must be whitelisted by contacting us)
Query URL: `https://{api_domain}/api/v1/payApi/QueryOrder`

## 8.1 Query Request Parameters

| Name | Type | Required | Description |
|------------|------|----|---------------------------|
| order_type | string | true | `pay_out`: pay-out, `pay_in`: pay-in |
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant `appId` |
| dis_order_no | string | false | Platform order number |
| order_no | string | false | Merchant order number |
| sign | string | true | Signature result, see the signature method at the top of the document |

- Query request example

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
|------------|------|-----|---------------------------------------------------------------------------------------------------------------------------------------------------|
| code | int | true | `200`: query successful, others: failed |
| msg | string | true | Query failure reason |
| trade_no | int | true | Merchant ID |
| <span style="color:red;">real_price</span> | int | true | <span style="color:red;">Actual amount paid, unit: centimos</span> |
| status | int | true | Order status: `1` unpaid, <span style="color:red;">`2` success</span>, `3` failed, `7` rejected, `9` reversed, `10` processing |
| success_time | int | true | Success timestamp |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| remark | string | true | Pay-out failure reason |
| fee | int | false | Order fee, unit: centimos |
| create_time | int | true | Creation time |
| payer | string | false | JSON string of payer information: `{"account_name":"Name","account_type":"Account type: CA, SA","account_no":"Account","identify_type":"ID type: DNI, CE, RUC, PAS","identify_num":"ID number","bank_code":"Bank code"}` |
| pay_info | string | false | Payment information JSON string, for example: native payment info, card number, name, bank, etc. |
| sign | string | true | Signature result, see the signature method at the top of the document |
| utr2 | string | false | Bank order number |

- Query response example

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
  "payer": "{\"account_name\":\"Juan Perez\",\"account_type\":\"SA\",\"account_no\":\"12345678901234\",\"identify_type\":\"DNI\",\"identify_num\":\"12345678\",\"bank_code\":\"BCP\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Pay-out Balance Query Interface

(The request IP must be whitelisted by contacting us)
URL: `https://{api_domain}/api/v1/payApi/QueryBalance`

## 9.1 Balance Request Parameters

| Name | Type | Required | Description |
|--------|------|----|----------------|
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant `appId` |
| sign | string | true | Signature result, see the signature method at the top of the document |

- Balance request example

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Response

| Name | Type | Required | Description |
|-------|------|----|---------------------------|
| code | int | true | `200`: query successful, others: failed |
| msg | string | true | Failure reason |
| balance | int | true | Balance, unit: centimos |
| balance_frozen | int | false | Frozen balance, unit: centimos |
| sign | string | true | Signature result, see the signature method at the top of the document |

- Balance response example

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
|--------|------|----|----------------|
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant `appId` |
| order_no | string | false | Merchant order number, choose either this or `dis_order_no` |
| dis_order_no | string | false | Platform order number, choose either this or `order_no` |
| sign | string | true | Signature result, see the signature method at the top of the document |

- Payment voucher request example

```json
{
  "trade_no": 10003,
  "app_id": 10003,
  "order_no": "",
  "dis_order_no": "35011C02gljuf6k0800peru1lVY",
  "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```

## 10.2 Payment Voucher Response Parameters

| Name | Type | Required | Description |
|-------|------|----|---------------------------|
| code | int | true | `200`: response successful, others: failed |
| msg | string | true | Failure reason |
| img_link | string | false | Voucher link |
| img_base | string | false | Base64 content of the voucher image |
| sign | string | true | Signature result, see the signature method at the top of the document |

- Payment voucher response examples
No payment voucher:

```json
{
  "code": 200,
  "msg": "No payment voucher available at the moment",
  "sign": "",
  "img_link": "",
  "img_base": ""
}
```

Payment voucher available:

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
|------|-------------------|
| 200 | Success |
| 1000 | Internal error |
| 1001 | IP is not in the merchant IP whitelist |
| 1002 | Parameter error |
| 1003 | Signature error |
| 1004 | This interface is not currently enabled for the merchant. Contact operations to verify whether the merchant or app does not exist, has been disabled, or the payment product is not configured |
| 1005 | Merchant does not exist |
| 1006 | Current user IP is in the blacklist |
| 1007 | Current user is in the blacklist |
| 1008 | Merchant app does not exist |
| 1009 | Payment product does not exist |
| 1010 | Payment channel does not exist |
| 1011 | Payment channel is not yet fully implemented and is temporarily unavailable |
| 1012 | Payment channel exception, please try again later |
| 1013 | Current volume is too high, please try again later |
| 1014 | Duplicate order number |
| 1015 | Insufficient app balance |
| 1016 | The same user is placing orders too frequently, please try again later |
| 1017 | Order record does not exist |
| 1018 | Current amount is not supported |
| 1019 | Pay-in is not enabled for the current app country |
| 1020 | Pay-out is not enabled for the current app country |
| 1021 | Failed |
| 1036 | Interface is not yet open |
| 1037 | Currency is not supported |
| 1038 | Abnormal UTR reported in pay-in callback |
| 9999 | Other errors |
| 3000 | System maintenance in progress, order placement is suspended, please try again later |

# 12. Pay-in Cashier Interface

URL: `https://{api_domain}/api/v1/cashApi/CashIn.html`
Request method: `GET`

### Parameters

| Name | Type | Required | Description |
|--------------|--------|-------|---------------------------|
| app_id | string | true | Merchant `app_id` |
| order_no | string | true | Merchant order number |
| amount | string | true | Merchant amount, unit: Peruvian Sol |
| notice_url | string | false | Asynchronous callback URL |
| pay_code | int | true | Product code |

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{merchant_order_no}}&amount={{merchant_amount}}&notice_url={{callback_url}}&pay_code={{product_code}}
```

# 13. Pay-in Payment Methods

| Field Name | Value | Description |
|-----------|-------|------|
| pay_method | BANK_TRANSFER | Bank transfer |
| pay_method | QR | QR payment |
| pay_method | CHECKOUT | Checkout |
| pay_method | Peru | Cash |

# 14. Pay-out Payment Methods

| Field Name | Value | Description |
|------------|---------------|------|
| pay_type | BANK_TRANSFER | Bank transfer |
| pay_type | YAPE | Yape |
| pay_type | PLIN | Plin wallet |

# 15. Pay-out Bank Codes

| Field Name | Code | Bank Name |
|:-----------|:-----|:----------|
| bank_code | Citibank_Pen | Citibank |
| bank_code | SCOTIABANK_Pen | SCOTIABANK |
| bank_code | ICBC_Pen | ICBC PERU BANK |
| bank_code | Azteca_Pen | Banco Azteca |
| bank_code | Santander_Pen | Banco Santander |
| bank_code | Mi | Mi Banco |
| bank_code | Trujillo | Caja Trujillo |
| bank_code | Sullana | Caja Sullana |
| bank_code | Municipal_Ica | Caja Municipal Ica |
| bank_code | GNB | Banco GNB |
| bank_code | Continental | Banco Continental |
| bank_code | Ripley | Banco Ripley |
| bank_code | Interbank | Interbank |
| bank_code | Huancayo | Caja Huancayo |
| bank_code | BanBif | Banco Interamericano de Finanzas (BanBif) |
| bank_code | Falabella | Banco Falabella |
| bank_code | BBVA | BBVA |
| bank_code | BCP | BCP |
| bank_code | Metropolitana | Caja Metropolitana |
| bank_code | Pichincha | Banco Pichincha |
| bank_code | Financiero | Banco Financiero |
| bank_code | Tacna | Caja Tacna |
| bank_code | Maynas | Caja Maynas |
| bank_code | Cusco | Caja Cusco |
| bank_code | Nation | Banco de la Nacion |
| bank_code | Cencosud | Banco Cencosud |
| bank_code | Comercio | Banco de Comercio |
| bank_code | Credito | Banco de Credito |

# 16. Document Update Time
```
2026-05-22 17:03:00
```
