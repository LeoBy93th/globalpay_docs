# 1. Integration Process

> 1. Business negotiation for account opening and communication regarding relevant rates.
>
> 2. Contact operations to create Merchant ID, Secret Key, Merchant AppId, Product Code, and apiUrl.
>
> 3. Upon completion of development, both parties conduct joint debugging and testing to verify the integrity of requests, reporting, and other information.

# 2. Md5 Signature Algorithm

> 1. Sort all parameters in ascending order as key-value pairs (key1=value1) (empty parameter values are not included in the signature).
>
> 2. Combine them in the format key1=value1&key2=value2.
>
> 3. Append the merchant secret key: key1=value1&key2=value2...&key=MerchantSecretKey.
>
> 4. sign=md5(the string assembled in the previous step). The signature result is a 32-character lowercase string.
>
> 5. The signature key can be found in the Merchant Backend -> Basic Information, or by inquiring with our customer service.

# 3. Precautions

## 3.1 Interface Related

> 1. All interfaces in this document use standard HTTP communication protocols, submitted via POST. Both request and response Content-type are application/json, and the character encoding is unified as UTF-8.
>
> 2. The currency unit is <span style="color:red;"> cents (KES) </span>.
>
> 3. The IP address for requesting the interface needs to be whitelisted.
>
> 4. Collect the real user IP for user_ip as much as possible; if truly unavailable, leave it blank. Do not use local IPs like 127.0.0.1.

## 3.2 Callback Related

> 1. If the callback is successfully received and processed, please return the text <span style="color:red;">success</span> without any other characters. The system will stop pushing this order information; otherwise, it will be resent multiple times.
>
> 2. During asynchronous notification interaction, if the received response is not `success`, it is considered a notification failure, and notifications will be re-initiated periodically based on a certain strategy. The notification intervals are: 1m, 1m, 4m, 10m, 10m, 1h, 2h, 6h, 15h.
>
> 3. If the pay_notice_url notification address is empty, it will be considered that the merchant does not need a callback, and the system will not push a notification.

# 4. Pay-in (Collection) Order Interface

(The order placement IP needs to be whitelisted by contacting us)
Order address: https://{api_domain}/api/v1/payApi/CreatePayInOrder

## 4.1 Pay-in - Order Request Parameters

| Name      | Type  | Required | Description                                                                                                                              |
| -------------- | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| trade_no    | int  | true   | Merchant ID.                                                                                                                             |
| app_id     | int  | true   | Merchant appId.                                                                                                                            |
| pay_code    | int  | true   | Product code, obtained from our operations.                                                                                                              |
| pay_method   | string | true   | Payment method: KES-Payment.                                                                                                                     |
| price     | int  | true   | Order amount, unit: cents, integer.                                                                                                                  |
| order_no    | string | true   | Merchant order number.                                                                                                                        |
| success_url  | string | false  | Redirect URL for successful payment.                                                                                                                 |
| fail_url    | string | false  | Redirect URL for failed payment.                                                                                                                   |
| pay_notice_url | string | false  | Notification URL for successful payment.                                                                                                               |
| user_id    | string | true   | System user ID.                                                                                                                            |
| user_ip    | string | true   | Payer IP address.                                                                                                                           |
| attach     | string | true   | Additional parameters in JSON string format: payer information. |
| sign      | string | true   | Signature result, see the top of the document for the signature method.                                                                                                |
| timestamp   | string | false  | Order timestamp (10-digit timestamp in seconds).                                                                                                           |

- Pay-in - attach Additional Parameter Field Description

| Name         | Type   | Required | Description                              |
| ------------ | ------ | -------- | ---------------------------------------- |
| email        | string | true     | Payer email.                             |
| phone        | string | true     | Payer phone number. An STK Push is sent to the payer's phone after the order is successfully placed. Must be a 10-digit number starting with 0. |

- Pay-in - Order Request Example

  ```json
  {
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "pay_method": "KES-Payment",
  "price": 10000,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"email\":\"john.kamau@example.com\",\"phone\":\"0712345678\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
  }
  ```

## 4.2 Pay-in - Order Response

| Name     | Type  | Required | Description                                            |
| ------------ | ------ | -------- | -------------------------------------------------------------------------------------------------- |
| code     | int  | true   | 200: Success; Others: Failure.                                   |
| msg     | string | true   | Failure reason.                                          |
| pay_url   | string | false  | Payment link.                                           |
| qr_code   | string | false  | QR code string.                                        |
| order_no   | string | true   | Merchant order number.                                       |
| dis_order_no | string | true   | Platform order number.                                       |
| create_time | int  | true   | Creation time.                                           |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc. |
| sign     | string | true   | Signature result, see the top of the document for the signature method.              |

- Pay-in - Order Response Example

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
  "dis_order_no": "2025071130770572062498816kenya1oushe",
  "create_time": 1752825512,
  "pay_url": "https://checkout.example.com/kes/order-example",
  "pay_info": "{\"acc_no\":\"07*****678\",\"bank\":\"M-PESA\",\"memo\":\"KES payment example\",\"name\":\"John K***\",\"pay_raw\":\"\"}"
}
```


- Pay-in - pay_info Additional Parameter Field Description

| Parameter | Type | Required | Description         |
|-----------|------|----------|---------------------|
| acc_no | string | false | Recipient Account   |
| bank | string | false | Bank or wallet information |
| memo | string | false | Payment Description |
| name | string | false | Payee Name          |
| pay_raw | string | false | Original payment information, which the merchant can convert into a QR code. |



# 5. Pay-in Callback Notification (post/json)

Push address: The `pay_notice_url` provided by the merchant during order placement. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 5.1 Pay-in Callback - Request Parameters

| Name     | Type  | Required | Description                                                                                                                              |
| ------------ | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| trade_no   | int  | true   | Merchant ID.                                                                                                                             |
| status    | int  | true   | Order status: <span style="color:red;">2. Success</span>, 3. Failure.                                                                                                                 |
| order_no   | string | true   | Merchant order number.                                                                                                                        |
| dis_order_no | string | true   | Platform order number.                                                                                                                        |
| order_price | int  | true   | Order amount, unit: cents.                                                                                                                      |
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid by the user, unit: cents.</span>                                                                                                             |
| nti_time   | int  | false  | Notification initiation time.                                                                                                                     |
| payer    | string | false  | JSON string, payer info: {"name":"Name", "account":"Account", "bank":"Payer bank or wallet code", "utr2":"Bank or wallet transaction reference", "email":"Email", "phone":"Phone", "identify_type":"Identity Type", "identify_num":"Identity number"}. Also includes payer-related fields from `attach`. |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc. 25-10-28 |
| create_time | int  | true   | Creation time.                                                                                                                            |
| sign     | string | true   | Signature result, see the top of the document for the signature method.                                                                                                |

- Pay-in Callback - Request Parameter Example

```json

{
  "trade_no": 10003,
  "status": 3,
  "order_no": "p71584120256SlWlKkymb",
  "dis_order_no": "2025071130460153942908928kenya1sKQbX",
  "order_price": 10000,
  "real_price": 10000,
  "payer": "{\"name\":\"John Kamau\",\"email\":\"john.kamau@example.com\",\"phone\":\"0712345678\",\"account\":\"0712345678\",\"bank\":\"M-PESA\"}",
  "nti_time": 1752826164,
  "create_time": 1752751502,
  "sign": "eba7f27e0f49581d8784294ef29f994d"
}
```

## 5.2 Pay-in Callback - Response Description

If the callback is successfully received and processed, please return <span style="color:red;">`success`</span>. The system will stop pushing this order information; otherwise, it will be resent multiple times.

# 6. Pay-out (Disbursement) Order Interface

(The order placement IP needs to be whitelisted by contacting us)
Order address: https://{api_domain}/api/v1/payApi/CreatePayOutOrder

## 6.1 Pay-out - Request Parameters

<span style="color:red;">A pay-out request is considered successful unless an explicit failure is returned, even if it times out. Do not submit the request again unless an explicit failure is returned.</span>

| Name           | Type   | Required | Description                                                                                     |
| -------------- | ------ | -------- | ----------------------------------------------------------------------------------------------- |
| trade_no       | int    | true     | Merchant number                                                                                 |
| order_no       | string | true     | Merchant order number                                                                           |
| app_id         | int    | true     | Merchant appId                                                                                  |
| pay_code       | int    | true     | Product code, contact our operations team to obtain                                             |
| price          | int    | true     | Order amount, unit: cents, integer. After conversion to Kenyan shillings, decimals are not allowed |
| account_no     | string | true     | Receiving account. Must be a 10-digit number starting with 0.                                    |
| account_type   | string | true     | Account type: PHONE                                                                            |
| account_name   | string | true     | Recipient name                                                                                  |
| bank_code      | string | true     | Receiving bank code, refer to bank code list                                                    |
| pay_notice_url | string | false    | Payout success callback URL                                                                     |
| attach         | string | true     | Additional parameters {"email":"Email"}                                                        |
| user_ip        | string | true     | Recipient user IP                                                                               |
| sign           | string | true     | Signature result, signature method described at the top of this document                        |
| timestamp      | string | false    | Order timestamp, 10-digit Unix timestamp in seconds                                             |

- Pay-out - attach Additional Parameter Field Description

| Name         | Type   | Required | Description                              |
| ------------ | ------ | -------- | ---------------------------------------- |
| email        | string | true     | Recipient email.                         |

- Pay-out - Request Example

```json

{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10000,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"email\":\"john.kamau@example.com\",\"phone\":\"0712345678\",\"bank_name\":\"M-PESA\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "87.200.59.100",
  "account_type": "PHONE",
  "account_no": "0712345678",
  "account_name": "John Kamau",
  "bank_code": "M-PESA"
}
```

## 6.2 Pay-out - Order Response

| Name     | Type  | Required | Description                                  |
| ------------ | ------ | -------- | ----------------------------------------------------------------------------- |
| code     | int  | true   | 200: Success; Others: Failure.                        |
| msg     | string | true   | Failure reason.                                |
| dis_order_no | string | true   | Platform order number.                            |
| order_no   | string | true   | Merchant order number.                            |
| status    | int  | true   | Order status: 2. Success, 3. Failure, 7. Rejected, 9. Reversal, 10. Processing. |
| create_time | int  | true   | Creation time.                                |
| sign     | string | true   | Signature result, see the top of the document for the signature method.    |

- Pay-out - Order Response Example

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
  "dis_order_no": "2025071130776296733810688kenya1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7. Pay-out Callback Notification

Push address: The `pay_notice_url` provided by the merchant during order placement. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 7.1 Pay-out Callback Request Parameters

| Name     | Type  | Required | Description                                            |
| ------------ | ------ | -------- | -------------------------------------------------------------------------------------------------- |
| trade_no   | int  | true   | Merchant ID.                                            |
| order_no   | string | true   | Merchant order number.                                       |
| dis_order_no | string | true   | Platform order number.                                       |
| order_price | int  | true   | Order amount, unit: cents.                                     |
| fee     | int  | false  | Order fee, unit: cents.                                      |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status    | int  | true   | Order status: <span style="color:red;">2. Success</span>, 3. Failure, 7. Rejected, 9. Reversal.                  |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc. 25-10-28 |
| remark    | string | false  | Failure reason.                                          |
| create_time | int  | true   | Creation time.                                           |
| sign     | string | true   | Signature result, see the top of the document for the signature method.              |
| nti_time   | int  | true   | Notification initiation time.                                   |

- Pay-out Callback Request Example

```json

{
  "trade_no": 10003,
  "status": 3,
  "order_price": 10000,
"order_no": "p71584120257igU8n8FII",
  "dis_order_no": "2025071130700746140950528kenya15rLVI",
  "nti_time": 1752808888,
  "create_time": 1752808865,
  "sign": "00000000000000000000000000000000"
}
```

## 7.2 Pay-out Callback Response Description

If the callback is successfully received and processed, please return <span style="color:red;">`success`</span>. The system will stop pushing this order information; otherwise, it will be resent multiple times.

# 8. Query Order Interface (Common for Pay-in and Pay-out)

(The request IP needs to be whitelisted by contacting us)
Query address: https://{api_domain}/api/v1/payApi/QueryOrder

## 8.1 Query Request Parameters

| Name     | Type  | Required | Description                               |
| ------------ | ------ | -------- | ----------------------------------------------------------------------- |
| order_type  | string | true   | pay_out: Disbursement, pay_in: Collection.               |
| trade_no   | int  | true   | Merchant ID.                              |
| app_id    | int  | true   | Merchant appId.                             |
| dis_order_no | string | false   | Platform order number.                         |
| order_no | string | false  | Merchant  order number.              |
| sign     | string | true   | Signature result, see the top of the document for the signature method. |

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

| Name     | Type  | Required | Description                                                                                                 |
| ------------ | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| code     | int  | true   | 200: Query successful; Others: Failure.                                                                                   |
| msg     | string | true   | Query failure reason.                                                                                            |
| trade_no   | int  | true   | Merchant ID.                                                                                                 |
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid, unit: cents.</span>                                                                                       |
| status    | int  | true   | Order status: 1. Unpaid, <span style="color:red;">2. Success</span>, 3. Failure, 7. Rejected, 9. Reversal, 10. Processing.                                                          |
| success_time | int  | true   | Success timestamp.                                                                                              |
| order_no   | string | true   | Merchant order number.                                                                                            |
| dis_order_no | string | true   | Platform order number.                                                                                            |
| remark    | string | true   | Reason for pay-out failure.                                                                                         |
| fee     | int  | false  | Order fee, unit: cents.                                                                                           |
| create_time | int  | true   | Creation time.                                                                                                |
| payer    | string | false  | JSON string, payer info: {"account_name":"Name", "account_type":"Account Type: PHONE, BANK", "account_no":"Account", "bank_code":"Bank or wallet code"}.       |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc. 25-10-28 |
| sign     | string | true   | Signature result, see the top of the document for the signature method.                                                                   |
| utr2     | string | false  | Bank order number.                                                                                              |

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
  "payer": "{\"account_name\":\"John Kamau\",\"account_type\":\"PHONE\",\"account_no\":\"0712345678\",\"bank_code\":\"M-PESA\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Pay-out Balance Query Interface

(The request IP needs to be whitelisted by contacting us)
Address: https://{api_domain}/api/v1/payApi/QueryBalance

## 9.1 Balance Request Parameters

| Name   | Type  | Required | Description                               |
| -------- | ------ | -------- | ----------------------------------------------------------------------- |
| trade_no | int  | true   | Merchant ID.                              |
| app_id  | int  | true   | Merchant appid.                             |
| sign   | string | true   | Signature result, see the top of the document for the signature method. |

- Balance Request Example

```json

{
  "trade_no": 165,
  "app_id": 281,
  "sign": "00000000000000000000000000000000"
}
```

## 9.2 Balance Response

| Name      | Type  | Required | Description                               |
| -------------- | ------ | -------- | ----------------------------------------------------------------------- |
| code      | int  | true   | 200: Query successful; Others: Failure.                 |
| msg      | string | true   | Failure reason.                             |
| balance    | int  | true   | Balance, unit: cents.                          |
| balance_frozen | int  | false  | Frozen balance, unit: cents.                      |
| sign      | string | true   | Signature result, see the top of the document for the signature method. |

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

(The request IP needs to be whitelisted by contacting us)
Address: https://{api_domain}/api/v1/payApi/QueryCertificate

## 10.1 Payment Voucher Request Parameters

| Name     | Type  | Required | Description                                |
| ------------ | ------ | -------- | ------------------------------------------------------------------------- |
| trade_no   | int  | true   | Merchant ID.                               |
| app_id    | int  | true   | Merchant appid.                              |
| order_no   | string | false  | Merchant order number (choose either this or dis_order_no).        |
| dis_order_no | string | false  | Platform order number (choose either this or order_no).          |
| sign     | string | true   | Signature result, see the top of the document for the signature method. |

- Payment Voucher Request Example

```json

{
  "trade_no": 10003,
  "app_id": 10003,
  "order_no": "",
  "dis_order_no": "35011C02gljuf6k0800kenya1lVY",
  "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```


## 10.2 Payment Voucher Response Parameters

| Name   | Type  | Required | Description                               |
| -------- | ------ | -------- | ----------------------------------------------------------------------- |
| code   | int  | true   | 200: Response successful; Others: Failure.               |
| msg   | string | true   | Failure reason.                             |
| img_link | string | false  | Voucher link.                              |
| img_base | string | false  | Base64 code generated for the voucher.                 |
| sign   | string | true   | Signature result, see the top of the document for the signature method. |

- Payment Voucher Response Example
  No payment voucher:

```json

{
  "code":200,
  "msg":"No payment voucher available at the moment",
  "sign": "",
  "img_link": "",
  "img_base": ""
}
```



With payment voucher:

```json

{
  "code": 200,
  "msg": "",
  "sign": "3969f17cd1a551769f85967d0a05b7b6",
  "img_link": "https://example.com/payment-voucher.png",
  "img_base": "data:image/png;base64,hfhshdhfhfh"
}
```


# 11. Payment Methods — Pay-in Field: `pay_method`

| Field Name | Value       | Description            |
| ---------- | ----------- | ---------------------- |
| pay_method | KES-Payment | Kenya pay-in |

# 12. Bank Codes

| Field Name | Code  | Bank Name |
| :--------- | :---- | :-------- |
| bank_code  | M-PESA | MPESA |


# 13. Error Codes

| Status Code | Description                                                              |
|------|----------------------------------------------------------------------------------------------------------------------------------------|
| 200 | Success                                                                |
| 1000 | Internal Error                                                             |
| 1001 | IP not in merchant IP whitelist.                                                    |
| 1002 | Parameter Error                                                            |
| 1003 | Signature Error                                                            |
| 1004 | Interface currently unavailable for the merchant (Contact operations to verify: Merchant or App (Not exist\|Closed\|Product not configured)) |
| 1005 | Merchant does not exist.                                                        |
| 1006 | Current user IP is in the blacklist.                                                  |
| 1007 | Current user is in the blacklist.                                                   |
| 1008 | Merchant App does not exist.                                                      |
| 1009 | Payment product does not exist.                                                    |
| 1010 | Payment channel does not exist.                                                    |
| 1011 | Payment channel development not completed, temporarily unavailable.                                  |
| 1012 | Payment channel exception, please try again later.                                           |
| 1013 | High order volume, please try again later.                                               |
| 1014 | Duplicate order number.                                                        |
| 1015 | Insufficient app balance.                                                       |
| 1016 | Frequent order placement by the same user, please try again later.                                   |
| 1017 | Order record does not exist.                                                      |
| 1018 | Current amount not supported.                                                     |
| 1019 | Pay-in not enabled for the app's country.                                               |
| 1020 | Pay-out not enabled for the app's country.                                               |
| 1021 | Failure                                                                |
| 1036 | Interface not yet open.                                                        |
| 1037 | Currency not supported.                                                        |
| 1038 | Pay-in utr reporting error.                                                      |
| 9999 | Other errors.                                                             |
| 3000 | System maintenance, order placement suspended, please try again later.                                 |

# 14. Pay-in Checkout Interface

Address: https://{api_domain}/api/v1/cashApi/CashIn.html
Request Method: GET

### Parameters:

| Name    | Type  | Required | Description            |
| ---------- | ------ | -------- | ---------------------------------- |
| app_id   | string | true   | Merchant app_id.          |
| order_no  | string | true   | Merchant order number.       |
| amount   | string | true   | Merchant amount (unit: Kenyan shillings) |
| notice_url | string | false  | Asynchronous notification address. |
|pay_code|int|true|Product Code|

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{MerchantOrderNumber}}&amount={{MerchantAmount}}&notice_url={{AsynchronousNotificationAddress}}&pay_code={{ProductCode}}
```

# 15. Document Update Time
```
2026-09-18 19:32:49
```
