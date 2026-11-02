# Uzbekistan (UZB) Payment API Documentation

> The names, account numbers, order numbers, and signatures in this document are examples of the required formats only. Use actual parameter values during integration and calculate signatures with your merchant key as described in Section 2.

# 1. Integration Process

> 1. Contact our business team to open an account and agree on the applicable fees.
>
> 2. Contact our operations team to obtain your merchant ID, merchant key, merchant appId, product code, and apiUrl.
>
> 3. Once development is complete, perform joint integration testing to verify that requests, reported data, and other information are complete.

# 2. MD5 Signature Algorithm

> 1. Sort all parameters by key in ascending order, using the format `key1=value1`. Exclude parameters with empty values from the signature.
>
> 2. Join the parameters as `key1=value1&key2=value2`.
>
> 3. Append the merchant key: `key1=value1&key2=value2...&key=merchant_key`.
>
> 4. Calculate `sign=md5(the string assembled in the previous step)`. The result must be a 32-character lowercase hexadecimal string.
>
> 5. Find your signature key under Basic Information in the merchant dashboard, or contact our customer service team.

# 3. Important Notes

## 3.1 API Requests

> 1. All APIs in this document use HTTP and the POST method. The Content-Type for both requests and responses is `application/json`, with UTF-8 encoding.
>
> 2. Amounts are in <span style="color:red;">tiyin</span>. 1 Uzbekistani som (UZS) = 100 tiyin. The `amount` parameter of the checkout interface in Section 13 uses som (UZS).
>
> 3. The IP address used to call the APIs must be added to the whitelist.
>
> 4. Provide the user's actual IP address in `user_ip` whenever possible. If it is unavailable, leave the field empty. Do not use a local IP address such as `127.0.0.1`.

## 3.2 Callbacks

> 1. After successfully receiving and processing a callback, return the exact text <span style="color:red;">success</span> with no other characters. The system will then stop sending notifications for the order. Otherwise, it will retry multiple times.
>
> 2. If the response to an asynchronous notification is not `success`, the notification is considered unsuccessful and will be retried at these intervals: 1m, 1m, 4m, 10m, 10m, 1h, 2h, 6h, 15h.
>
> 3. If `pay_notice_url` is empty, the system assumes that the merchant does not require callbacks and will not send notifications.

# 4. Create Pay-in Order API

Contact us to whitelist the IP address used to place orders.

Endpoint: https://{api_domain}/api/v1/payApi/CreatePayInOrder

## 4.1 Pay-in Order Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant appId |
| pay_code | int | true | Product code; obtain it from our operations team |
| pay_method | string | true | Payment method. Use the fixed value `UZS-payment` (Uzbekistan Payment); see the payment method dictionary |
| price | int | true | Order amount as an integer in tiyin. 1 som (UZS) = 100 tiyin |
| order_no | string | true | Merchant order number |
| success_url | string | false | Redirect URL after successful payment |
| fail_url | string | false | Redirect URL after failed payment |
| pay_notice_url | string | false | Notification URL for successful payment |
| user_id | string | true | Payer's ID on the merchant platform |
| user_ip | string | false | Payer's IP address |
| attach | string | false | Additional parameters as a JSON string, for example: `{"name":"Name","email":"Email address"}`. See the field descriptions below |
| sign | string | true | Signature; see Section 2 for the signature algorithm |
| timestamp | string | false | Order timestamp: a 10-digit timestamp in seconds |

- Pay-in `attach` Fields

| Name | Type | Required | Description |
|------|------|----------|-------------|
| name | string | false | Payer's name |
| email | string | false | Payer's email address |

- Pay-in Order Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "pay_method": "UZS-payment",
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"name\":\"Aziz Karimov\",\"email\":\"aziz.karimov@example.com\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}
```

## 4.2 Pay-in Order Response

| Name | Type | Required | Description |
|------|------|----------|-------------|
| code | int | true | 200: order created successfully; other values: order creation failed |
| msg | string | true | Failure reason |
| pay_url | string | false | Payment URL |
| qr_code | string | false | QR code string |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| create_time | int | true | Creation time |
| pay_info | string | false | Payment information as a JSON string, for example: `{"pay_raw":"Raw payment data that the merchant can convert into a QR code"}` |
| sign | string | true | Signature; see Section 2 for the signature algorithm |

- Pay-in Order Response Examples

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
  "dis_order_no": "2025071130770572062498816uzb1oushe",
  "create_time": 1752825512,
  "pay_url": "https://{api_domain}/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5. Pay-in Callback Notification (POST/JSON)

Notifications are sent to the `pay_notice_url` provided by the merchant when placing the order. The callback IP address is `call_back_server_ip`; add our IP address to your callback whitelist.

## 5.1 Pay-in Callback Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| trade_no | int | true | Merchant ID |
| status | int | true | Order status: <span style="color:red;">2. Success</span>, 3. Failure |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| order_price | int | true | Order amount in tiyin |
| <span style="color:red;">real_price</span> | int | true | <span style="color:red;">Amount actually paid by the user, in tiyin</span> |
| nti_time | int | false | Notification initiation time |
| payer | string | false | Payer information as a JSON string: `{"name":"Name","account":"Account number","bank":"UZSBANK","utr2":"Bank transaction reference","email":"Email address","phone":"Mobile number","identify_type":"ID type","identify_num":"ID number"}`. In addition to these example fields, this parameter includes payer-related fields supplied by the merchant in `attach` |
| pay_info | string | false | Payment information as a JSON string, such as raw pay-in or payout data, card number, name, and bank |
| create_time | int | true | Creation time |
| sign | string | true | Signature; see Section 2 for the signature algorithm |

- Pay-in Callback Request Example

```json
{
  "trade_no": 10003,
  "status": 3,
  "order_no": "p71584120256SlWlKkymb",
  "dis_order_no": "2025071130460153942908928uzb1sKQbX",
  "order_price": 10099,
  "real_price": 10000,
  "payer": "{\"name\":\"Aziz Karimov\",\"email\":\"aziz.karimov@example.com\",\"phone\":\"Mobile number\",\"identify_type\":\"ID type\",\"identify_num\":\"ID number\"}",
  "nti_time": 1752826164,
  "create_time": 1752751502,
  "sign": "eba7f27e0f49581d8784294ef29f994d"
}
```

## 5.2 Pay-in Callback Response

After successfully receiving and processing the callback, return <span style="color:red;">success</span>. The system will then stop sending notifications for the order. Otherwise, it will retry multiple times.

# 6. Create Payout Order API

Contact us to whitelist the IP address used to place orders.

Endpoint: https://{api_domain}/api/v1/payApi/CreatePayOutOrder

## 6.1 Payout Order Request Parameters

<span style="color:red;">A pay-out request is considered successful unless an explicit failure is returned, even if it times out. Do not submit the request again unless an explicit failure is returned.</span>

| Name | Type | Required | Description |
|------|------|----------|-------------|
| trade_no | int | true | Merchant ID |
| order_no | string | true | Merchant order number |
| app_id | int | true | Merchant appId |
| pay_code | int | true | Product code; obtain it from our operations team |
| price | int | true | Order amount as an integer in tiyin. 1 som (UZS) = 100 tiyin |
| account_no | string | true | Recipient's Uzbekistan bank account number. Provide the actual, complete account number |
| account_type | string | true | Account type: `BANK` |
| account_name | string | true | Account holder's name |
| bank_code | string | true | Use the fixed value `UZSBANK` (Uzbekistan Bank); see the bank code dictionary |
| pay_notice_url | string | false | Notification URL for successful payout |
| attach | string | false | Additional parameters as a JSON string. See the field descriptions and example below |
| user_ip | string | false | Recipient's IP address |
| sign | string | true | Signature; see Section 2 for the signature algorithm |
| timestamp | string | false | Order timestamp: a 10-digit timestamp in seconds |

- Payout `attach` Fields

```json
{"email":"Email address","phone":"Phone number"}
```

| Name | Type | Required | Description |
|------|------|----------|-------------|
| email | string | false | Email address |
| phone | string | false | Phone number |

- Payout Order Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"email\":\"aziz.karimov@example.com\",\"phone\":\"+998901234567\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "",
  "account_type": "BANK",
  "account_no": "Recipient bank account number",
  "account_name": "Aziz Karimov",
  "bank_code": "UZSBANK"
}
```

## 6.2 Payout Order Response

| Name | Type | Required | Description |
|------|------|----------|-------------|
| code | int | true | 200: order created successfully; other values: order creation failed |
| msg | string | true | Failure reason |
| dis_order_no | string | true | Platform order number |
| order_no | string | true | Merchant order number |
| status | int | true | Order status: 2. Payout successful, 3. Payout failed, 7. Rejected, 9. Reversed, 10. Processing |
| create_time | int | true | Creation time |
| sign | string | true | Signature; see Section 2 for the signature algorithm |

- Payout Order Response Examples

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
  "dis_order_no": "2025071130776296733810688uzb1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7. Payout Callback Notification

Notifications are sent to the `pay_notice_url` provided by the merchant when placing the order. The callback IP address is `call_back_server_ip`; add our IP address to your callback whitelist.

## 7.1 Payout Callback Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| trade_no | int | true | Merchant ID |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| order_price | int | true | Order amount in tiyin |
| fee | int | false | Order fee in tiyin |
| <span style="color:red;">real_price</span> | int | false | <span style="color:red;">Actual payout amount in tiyin (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span> |
| status | int | true | Order status: <span style="color:red;">2. Payout successful</span>, 3. Payout failed, 7. Rejected, 9. Reversed |
| pay_info | string | false | Payment information |
| remark | string | false | Failure reason |
| create_time | int | true | Creation time |
| sign | string | true | Signature; see Section 2 for the signature algorithm |
| nti_time | int | true | Notification initiation time |

- Payout Callback Request Example

```json
{
  "trade_no": 10000,
  "status": 2,
  "order_no": "20060354339090013",
  "dis_order_no": "Meg2352644o2nmjo0800uzbYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": ""
}
```

## 7.2 Payout Callback Response

After successfully receiving and processing the callback, return <span style="color:red;">success</span>. The system will then stop sending notifications for the order. Otherwise, it will retry multiple times.

# 8. Query Order API (Pay-in and Payout)

Contact us to whitelist the IP address used to make requests.

Endpoint: https://{api_domain}/api/v1/payApi/QueryOrder

## 8.1 Order Query Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| order_type | string | true | `pay_out`: payout; `pay_in`: pay-in |
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant appId |
| dis_order_no | string | false | Platform order number |
| order_no | string | false | Merchant order number |
| sign | string | true | Signature; see Section 2 for the signature algorithm |

- Order Query Request Example

```json
{
  "order_type": "pay_in",
  "trade_no": 165,
  "app_id": 165,
  "dis_order_no": "p7158277185f96603047656571",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 8.2 Order Query Response

| Name | Type | Required | Description |
|------|------|----------|-------------|
| code | int | true | 200: query successful; other values: query failed |
| msg | string | true | Query failure reason |
| trade_no | int | true | Merchant ID |
| <span style="color:red;">real_price</span> | int | true | <span style="color:red;">Actual payment amount in tiyin</span> |
| status | int | true | Order status: 1. Unpaid, <span style="color:red;">2. Success</span>, 3. Failure, 7. Rejected, 9. Reversed, 10. Processing |
| success_time | int | true | Success timestamp |
| order_no | string | true | Merchant order number |
| dis_order_no | string | true | Platform order number |
| remark | string | true | Reason for payout failure |
| fee | int | false | Order fee in tiyin |
| create_time | int | true | Creation time |
| payer | string | false | Payer information as a JSON string: `{"account_name":"Name","account_type":"Account type: BANK","account_no":"Account number","bank_code":"UZSBANK"}` |
| pay_info | string | false | Payment information as a JSON string, such as raw pay-in or payout data, card number, name, and bank. 25-10-28 |
| sign | string | true | Signature; see Section 2 for the signature algorithm |
| utr2 | string | false | Bank order number |

- Order Query Response Examples

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
  "dis_order_no": "uzb169246816001692",
  "remark": "",
  "fee": 10,
  "create_time": 1695317066,
  "payer": "{\"account_name\":\"Aziz Karimov\",\"account_type\":\"BANK\",\"account_no\":\"Recipient bank account number\",\"bank_code\":\"UZSBANK\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Query Payout Balance API

Contact us to whitelist the IP address used to make requests.

Endpoint: https://{api_domain}/api/v1/payApi/QueryBalance

## 9.1 Balance Query Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| trade_no | int | true | Merchant ID |
| app_id | int | true | Merchant appId |
| sign | string | true | Signature; see Section 2 for the signature algorithm |

- Balance Query Request Example

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Query Response

| Name | Type | Required | Description |
|------|------|----------|-------------|
| code | int | true | 200: query successful; other values: query failed |
| msg | string | true | Failure reason |
| balance | int | true | Balance in tiyin |
| balance_frozen | int | false | Frozen balance in tiyin |
| sign | string | true | Signature; see Section 2 for the signature algorithm |

- Balance Query Response Examples

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

# 10. Payment Methods: Pay-in Field `pay_method`

| Field | Country | Value | Description |
|-------|---------|-------|-------------|
| pay_method | Uzbekistan | UZS-payment | Uzbekistan Payment |

# 11. Bank Codes: `bank_code`

| Field | Country | Value | Description |
|-------|---------|-------|-------------|
| bank_code | Uzbekistan | UZSBANK | Uzbekistan Bank |

# 12. Error Codes

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 1000 | Internal error |
| 1001 | IP address is not in the merchant IP whitelist |
| 1002 | Parameter error |
| 1003 | Signature error |
| 1004 | This API is currently unavailable for the merchant. Contact operations to check whether the merchant or app does not exist, is disabled, or has no payment product configured |
| 1005 | Merchant does not exist |
| 1006 | Current user's IP address is blacklisted |
| 1007 | Current user is blacklisted |
| 1008 | Merchant app does not exist |
| 1009 | Payment product does not exist |
| 1010 | Payment channel does not exist |
| 1011 | Payment channel development is incomplete; the channel is temporarily unavailable |
| 1012 | Payment channel error; please try again later |
| 1013 | Order volume is too high; please try again later |
| 1014 | Duplicate order number |
| 1015 | Insufficient app balance |
| 1016 | The same user is placing orders too frequently; please try again later |
| 1017 | Order record does not exist |
| 1018 | The specified amount is not supported |
| 1019 | Pay-in order creation is not enabled for the app's country |
| 1020 | Payout order creation is not enabled for the app's country |
| 1021 | Failure |
| 1036 | API is not yet available |
| 1037 | Currency is not supported |
| 1038 | Pay-in UTR reporting error |
| 9999 | Other error |
| 3000 | System upgrade or maintenance is in progress. Order creation is suspended; please try again later |

# 13. Pay-in Checkout Interface

Endpoint: https://{api_domain}/api/v1/cashApi/CashIn.html

Request method: GET

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| app_id | string | true | Merchant app_id |
| order_no | string | true | Merchant order number |
| amount | string | true | Merchant amount in Uzbekistani som (UZS) |
| notice_url | string | false | Asynchronous notification URL |
| pay_code | int | true | Product code |

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{MerchantOrderNumber}}&amount={{MerchantAmount}}&notice_url={{AsynchronousNotificationURL}}&pay_code={{ProductCode}}
```

# 14. Document Update Time

```
2026-09-24 16:38:03
```
