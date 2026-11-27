# 1. Integration Process

> 1. Discuss account opening and applicable fees with our business team.
>
> 2. Contact our operations team to set up the merchant ID, secret key, merchant App ID, product code, and apiUrl.
>
> 3. After development, both teams run integration tests to verify that requests, reports, and other data are complete.

# 2. MD5 Signature Algorithm

> 1. Sort all key-value pairs (key1=value1) in ascending order by key. Exclude parameters with empty values from the signature.
>
> 2. Join the pairs as key1=value1&key2=value2.
>
> 3. Append the merchant secret key to obtain key1=value1&key2=value2...&key=MERCHANT_SECRET_KEY.
>
> 3. Calculate sign=md5(the string from the previous step) to produce a 32-character lowercase signature.
>
> 4. Find the signing key under Basic Information in the merchant portal or contact customer support.

# 3. Important Notes

## 3.1 API Requirements

> 1. All endpoints in this document use HTTP POST. Requests and responses use application/json as the Content-Type and UTF-8 encoding.
>
> 2. Amounts are in <span style="color:red;"> minor units (NGN) </span>.
>
> 3. The IP address sending API requests must be on the allowlist.
>
> 4. Use the actual user IP address for user_ip whenever possible. Leave it blank if unavailable. Do not use a local address such as 127.0.0.1.

## 3.2 Callback Requirements

> 1. After successfully receiving and processing a callback, return only <span style="color:red;">success</span>. The system stops sending notifications for this order when it receives that exact text. Otherwise, it retries several times.
>
> 2. For asynchronous callbacks, any response other than success counts as a failure. The system retries at these intervals: 1m, 1m, 4m, 10m, 10m, 1h, 2h, 6h, 15h.
>
> 3. If pay_notice_url is empty, the system assumes the merchant does not require a callback and sends no notification.

# 4. Pay-in Order API

(Contact us to allowlist the IP address used to place orders)
Order URL: https://{api_domain}/api/v1/payApi/CreatePayInOrder

## 4.1 Pay-in Order Request Parameters

| Name             | Type     | Required    | Description                                                                                                                                                                            |
| -------------- | ------ | ----- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| trade_no       | int    | true  | Merchant ID                                                                                                                                                                           |
| app_id         | int    | true  | Merchant App ID                                                                                                                                                                      |
| pay_code       | int    | true  | Product code. Contact our operations team to obtain it.                                                                                                                                                                 |
| pay_method     | string | true  | Payment method                                                                                                                                                               |
| price          | int    | true  | Order amount as an integer in minor units                                                                                                                                                                 |
| order_no       | string | true  | Merchant order number                                                                                                                                                                         |
| success_url    | string | false | Redirect URL for successful payments                                                                                                                                                                    |
| fail_url       | string | false | Redirect URL for failed payments                                                                                                                                                                    |
| pay_notice_url | string | false | Notification URL for successful payments                                                                                                                                                                    |
| user_id        | string | true  | System user ID                                                                                                                                                                        |
| user_ip        | string | true  | Payer IP address                                                                                                                                                                        |
| attach         | string | true  | Additional payer information as a JSON string |
| sign           | string | true  | Signature. See the signing instructions at the beginning of this document.                                                                                                                                                                |
| timestamp      | string | false | Order timestamp (10 digits, in seconds)                                                                                                                                                               |

- Pay-in attach Fields

  | Name            | Type     | Required    | Description                                               |
    | ------------- | ------ | ----- |--------------------------------------------------|
  | name          | string | true | Payer name                                            |
  | phone         | string | true | Payer mobile phone number                                           |
  | account_no    | string | false  | Payer account number                                             |
  | account_type  | string | false  | Account type: PHONE or BANK (bank account)                            |
  | bank_code     | string | false | Bank code                                     |
  | bank_name     | string | false | Bank name                                             |

- Pay-in Order Request Example

  ```json
  {
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"name\":\"Zhang San\",\"email\":\"zhangsan@example.com\",\"phone\":\"09123456789\",\"account_no\":\"1234567890\",\"account_type\":\"PHONE\",\"bank_code\":\"123456\",\"bank_name\":\"Bank of China\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
  }
  ```

## 4.2 Pay-in Order Response

| Name           | Type     | Required    | Description                                         |
| ------------ | ------ | ----- | ------------------------------------------ |
| code         | int    | true  | 200 means the order was placed successfully. Any other code means it failed.                           |
| msg          | string | true  | Failure reason                                       |
| pay_url      | string | false | Payment link                                       |
| qr_code      | string | false | QR code string                                 |
| order_no     | string | true  | Merchant order number                                      |
| dis_order_no | string | true  | Platform order number                                      |
| create_time  | int    | true  | Creation time                                       |
| pay_info     | string | false | Payment information as a JSON string, including raw pay-in or pay-out data, card number, name, and bank. |
| sign         | string | true  | Signature. See the signing instructions at the beginning of this document.                             |


- Pay-in pay_info Fields

  | Name      | Type     | Required    | Description                 |
    |---------|--------|-------|--------------------|
  | acc_no  | string | false | Payee account               |
  | bank    | string | false | Bank information               |  
  | memo    | string | false | Payment description               |  
  | name    | string | false | Payee name                |  
  | pay_raw | string | false | Raw payment data that merchants can convert into a QR code |

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
  "dis_order_no": "2025071130770572062498816india1oushe",
  "create_time": 1752825512,
  "pay_url": "https://api.sunpayinr.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5",
  "pay_info": "{\"acc_no\":\"96311*****2701\",\"bank\":\"NG000016\",\"memo\":\"B4VGR8dfkdalds\",\"name\":\"MR CHI****** OKAFOR\",\"pay_raw\":\"00020101021238580010A00000072701280006970418011496311435212701**************************\"}"
}
```



# 5. Pay-in Callback Notification post/json

The callback goes to the pay_notice_url the merchant provides when placing the order. Add our callback IP address, call_back_server_ip, to your allowlist.

## 5.1 Pay-in Callback Request Parameters

| Name           | Type     | Required    | Description                                                                                                                                                                                           |
| ------------ | ------ | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| trade_no     | int    | true  | Merchant ID                                                                                                                                                                                          |
| status       | int    | true  | Order status: <span style="color:red;">2. Success</span>, 3. Failure                                                                                                                                                                             |
| order_no     | string | true  | Merchant order number                                                                                                                                                                                        |
| dis_order_no | string | true  | Platform order number                                                                                                                                                                                        |
| order_price  | int    | true  | Order amount in minor units                                                                                                                                                                                    |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">Actual amount paid by the user in minor units</span>                                                                                                                                                                               |
| nti_time     | int    | false | Time the notification was sent                                                                                                                                                                                       |
| payer        | string | false | Payer information as a JSON string: {"name":"Name","account":"Account","bank":"Payer bank code","utr2":"Bank reference number","email":"Email","phone":"Phone","identify_type":"ID type","identify_num":"CPF,CNPJ"}. This field also includes payer information supplied by the merchant in attach. |
| pay_info     | string | false | Payment information as a JSON string, including raw pay-in or pay-out data, card number, name, and bank. 25-10-28                                                                                                                                                  |
| create_time  | int    | true  | Creation time                                                                                                                                                                                         |
| sign         | string | true  | Signature. See the signing instructions at the beginning of this document.                                                                                                                                                                               |

- Pay-in Callback Request Example

```json

{
  "trade_no": 10003,
  "status": 3,
  "order_no": "p71584120256SlWlKkymb",
  "dis_order_no": "2025071130460153942908928india1sKQbX",
  "order_price": 10099,
  "real_price": 10000,
  "payer": "{\"name\":\"Name\",\"email\":\"Email\",\"phone\":\"Phone\",\"identify_type\":\"ID type\",\"identify_num\":\"CPF,CNPJ\"}",
  "nti_time": 1752826164,
  "create_time": 1752751502,
  "sign": "eba7f27e0f49581d8784294ef29f994d"
}
```

## 5.2 Pay-in Callback Response

After successfully receiving and processing the callback, return <span style="color:red;">success</span>. The system then stops sending notifications for the order. Otherwise, it retries several times.

# 6. Pay-out Order API

(Contact us to add your order request IP address to the allowlist)  
Order URL: https://{api_domain}/api/v1/payApi/CreatePayOutOrder

## 6.1 Pay-out Request Parameters

<span style="color:red;">Treat pay-out requests as successful unless the API explicitly returns a failure. This applies to timeouts. Resubmit only after an explicit failure.</span>

| Name             | Type     | Required    | Description                                                   |
| -------------- | ------ | ----- |------------------------------------------------------|
| trade_no       | int    | true  | Merchant ID                                                  |
| order_no       | string | true  | Merchant order number                                                |
| app_id         | int    | true  | Merchant App ID                                             |
| pay_code       | int    | true  | Product code. Contact our operations team to obtain it                                        |
| price          | int    | true  | Order amount as an integer in minor units. The amount in major currency units must also be an integer                              |
| account_no     | string | true  | Recipient account number                                                 |
| account_type   | string | true  | Account type (PHONE or BANK for a bank account)                    |
| account_name   | string | true  | Recipient name                                                   |
| bank_code      | string | true  | Recipient bank code. See Bank Codes                                        |
| pay_notice_url | string | false | Pay-out success callback URL                                           |
| attach         | string | false | Additional parameters {"email":"Email","phone":"Phone number","bank_name":"Bank name"} |
| user_ip        | string | true  | Recipient IP address                                              |
| sign           | string | true  | Signature. See the signing instructions at the beginning of this document                                       |
| timestamp      | string | false | Order timestamp as a 10-digit value in seconds                                      |

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
  "user_ip": "87.200.59.100",
  "account_type": "BANK",
  "account_no": "1234567890123",
  "account_name": "Chinedu Okafor",
  "bank_code": "NG000014"
}
```

## 6.2 Pay-out Order Response

| Name           | Type     | Required   | Description                                    |
| ------------ | ------ | ---- | ------------------------------------- |
| code         | int    | true | Order creation status (200 for success, any other value for failure)                      |
| msg          | string | true | Failure reason                                  |
| dis_order_no | string | true | Platform order number                                 |
| order_no     | string | true | Merchant order number                                 |
| status       | int    | true | Order status (2. Pay-out succeeded, 3. Pay-out failed, 7. Rejected, 9. Reversed, 10. Processing) |
| create_time  | int    | true | Creation time                                  |
| sign         | string | true | Signature. See the signing instructions at the beginning of this document                        |

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
  "dis_order_no": "2025071130776296733810688india1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

## 6.3 Pay-out Reverse Query Protocol (Optional)

Merchants that support pay-out reverse queries must implement an order query endpoint that follows this protocol for the platform API to call.

### 6.3.1 Request and Response

The platform API sends JSON using `POST`. The merchant must return HTTP 200 with a flat JSON object. The merchant must verify the request signature and timestamp before querying its order database. The endpoint is read-only and must never create an order or initiate a pay-out. The platform may query the same order more than once. The merchant must return the nonce from the current request and calculate a new signature for each response.

The request omits the expected amount. Each field listed for a request or response is required for that direction.

| Field | Type | Request | Response |
| --- | --- | --- | --- |
| `trade_no` | int64 | Merchant ID | Return unchanged |
| `app_id` | int64 | Merchant App ID | Return unchanged |
| `order_no` | string | Merchant order number `MerOrderNo` | Return the order number found. If the order does not exist, return the requested order number |
| `timestamp` | string | Unix timestamp in seconds. The merchant must verify that the time difference is at most 300 seconds | Omitted |
| `nonce` | string | 16 cryptographically secure random bytes encoded as 32 lowercase hexadecimal characters | Return unchanged |
| `price` | int64 | Omitted | Original order amount recorded by the merchant, in minor units (the amount in major currency units × 100). Return `0` if the order does not exist |
| `status` | string | Omitted | `EXISTS` if the order exists or `NOT_FOUND` if it does not. Independent of payment status |
| `sign` | string | Request signature | New signature calculated from the response fields |

The examples below use the public test key `demo_merchant_key`. The fixed timestamp and nonce are for signature verification examples. Live requests must use a current timestamp and a fresh, cryptographically secure random nonce.

Request:

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

Response:

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

### 6.3.2 Signing and Validation

This reverse query endpoint follows the signing and validation rules in this section. Exclude `sign` and include all other protocol fields, including `price=0`. Convert numeric values to decimal integers and preserve string values unchanged. Sort by field name in ascending order, concatenate as `key=value&key=value` without URL encoding, then append `&key=MERCHANT_SECRET_KEY`. Calculate MD5 over the UTF-8 bytes of the resulting string and return 32 lowercase hexadecimal characters.

The platform API recalculates the signature from the response fields and compares it with the response signature in constant time. The original order signature and the current request signature are outside this comparison. The signing method must keep the signing string that contains the secret key out of its output.

The platform API validates the structure, signature, and business fields in that order. It proceeds to prechecks only after all validations pass.

- Reject JSON with missing fields, `null` values, duplicate keys, unknown fields, multiple top-level values, or incorrect types. Numeric values must fit within the int64 range. `price` accepts only a JSON integer. Reject strings, decimals, and exponent notation.
- The merchant ID, App ID, and nonce in the response must match those in the current request. The merchant order number must match exactly, and the existence status must be `status=EXISTS`.
- The amount must match the original order amount stored by the platform. The platform stores `Amount` for the primary order in major currency units. Multiply it by 100 using decimal arithmetic and confirm that the result is a positive integer within the int64 range before converting it to int64. Use the original order amount for comparison and avoid truncating floating-point values or substituting the actual paid amount, credited amount, or fees. Verify the primary order amount even when the original request's `price` is available.

# 7. Pay-out Callback Notification

The platform sends the callback to the pay_notice_url supplied by the merchant when placing the order. Add the platform's callback IP address (call_back_server_ip) to your callback allowlist.

## 7.1 Pay-out Callback Request Parameters

| Name           | Type     | Required    | Description                                          |
| ------------ | ------ | ----- | ------------------------------------------- |
| trade_no     | int    | true  | Merchant ID                                         |
| order_no     | string | true  | Merchant order number                                       |
| dis_order_no | string | true  | Platform order number                                       |
| order_price  | int    | true  | Order amount in minor units                                   |
| fee          | int    | false | Order fee in minor units                                 |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual pay-out amount, present only for successful pay-outs. Contact our team to configure the real_price field</span>                                                                                       |
| status       | int    | true  | Order status (<span style="color:red;">2. Pay-out succeeded</span>, 3. Pay-out failed, 7. Rejected, 9. Reversed)             |
| pay_info     | string | false | Payment information as a JSON string, such as raw pay-in or pay-out details, card number, name, or bank (25-10-28) |
| remark       | string | false | Failure reason                                        |
| create_time  | int    | true  | Creation time                                        |
| sign         | string | true  | Signature. See the signing instructions at the beginning of this document                              |
| nti_time     | int    | true  | Time the notification was sent                                      |

- Pay-out Callback Request Example

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

## 7.2 Pay-out Callback Response

Return <span style="color:red;">success</span> after receiving and processing the callback to stop repeat notifications for the order. The platform retries the callback if it receives a different response.

# 8. Order Query API (Pay-in and Pay-out)

(Contact us to allowlist the request IP address)  
Query URL: https://{api_domain}/api/v1/payApi/QueryOrder

## 8.1 Query Request Parameters

| Name           | Type     | Required   | Description                   |
| ------------ | ------ | ---- | -------------------- |
| order_type   | string | true | pay_out (pay-out), pay_in (pay-in) |
| trade_no     | int    | true | Merchant ID                  |
| app_id       | int    | true | Merchant App ID             |
| dis_order_no | string | false | Platform order number                |
| order_no | string | false  | Merchant order number                |
| sign         | string | true | Signature. See the signing instructions at the beginning of this document.       |

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

| Name           | Type     | Required    | Description                                                                                                                                                |
| ------------ | ------ | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| code         | int    | true  | 200 indicates a successful query. Other codes indicate failure.                                                                                                                                    |
| msg          | string | true  | Reason the query failed                                                                                                                                            |
| trade_no     | int    | true  | Merchant ID                                                                                                                                               |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">Actual amount paid in minor units</span>                                                                                                                                      |
| status       | int    | true  | Order status. 1. Unpaid, <span style="color:red;">2. Success</span>, 3. Failed, 7. Rejected, 9. Reversed, 10. Processing                                                                                                        |
| success_time | int    | true  | Timestamp when the order succeeded                                                                                                                                             |
| order_no     | string | true  | Merchant order number                                                                                                                                             |
| dis_order_no | string | true  | Platform order number                                                                                                                                             |
| remark       | string | true  | Reason the pay-out failed                                                                                                                                            |
| fee          | int    | false | Order fee in minor units                                                                                                                                       |
| create_time  | int    | true  | Creation time                                                                                                                                              |
| payer        | string | false | Payer information as a JSON string {"account_name":"Name","account_type":"Account type: PHONE,BANK","account_no":"Account number","bank_code":"Bank code"} |
| pay_info     | string | false | Payment information as a JSON string with details such as raw pay-in or pay-out data, card number, name, and bank (25-10-28)                                                                                                       |
| sign         | string | true  | Signature. See the signing instructions at the beginning of this document.                                                                                                                                    |
| utr2         | string | false | Bank order number                                                                                                                                             |

- Query Response Examples

Failure:

```json

{
  "code": 1017,
  "msg": "Order record does not exist"
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
  "payer": "{\"account_name\":\"Chinedu Okafor\",\"account_type\":\"BANK\",\"account_no\":\"1234567890123\",\"bank_code\":\"NG000014\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Pay-out Balance Query API

(Contact us to allowlist the request IP address)  
URL: https://{api_domain}/api/v1/payApi/QueryBalance

## 9.1 Balance Query Request Parameters

| Name       | Type     | Required   | Description             |
| -------- | ------ | ---- | -------------- |
| trade_no | int    | true | Merchant ID            |
| app_id   | int    | true | Merchant App ID       |
| sign     | string | true | Signature. See the signing instructions at the beginning of this document. |

- Balance Query Request Example

```json

{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Query Response

| Name             | Type     | Required    | Description             |
| -------------- | ------ | ----- | -------------- |
| code           | int    | true  | 200 indicates a successful query. Other codes indicate failure. |
| msg            | string | true  | Reason for failure           |
| balance        | int    | true  | Balance in minor units        |
| balance_frozen | int    | false | Frozen balance in minor units      |
| sign           | string | true  | Signature. See the signing instructions at the beginning of this document. |

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

# 10. Payment Voucher Query API

(Contact us to allowlist the request IP address)  
URL: https://{api_domain}/api/v1/payApi/QueryCertificate

## 10.1 Payment Voucher Query Request Parameters

| Name           | Type     | Required    | Description                     |
| ------------ | ------ | ----- | ---------------------- |
| trade_no     | int    | true  | Merchant ID                    |
| app_id       | int    | true  | Merchant App ID               |
| order_no     | string | false | Merchant order number. Provide either order_no or dis_order_no. |
| dis_order_no | string | false | Platform order number. Provide either dis_order_no or order_no.     |
| sign         | string | true  | Signature. See the signing instructions at the beginning of this document.         |

- Payment Voucher Query Request Example

```json

{
  "trade_no": 10003,
  "app_id": 10003,
  "order_no": "",
  "dis_order_no": "35011C02gljuf6k0800india1lVY",
  "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```


## 10.2 Payment Voucher Query Response Parameters

| Name       | Type     | Required    | Description             |
| -------- | ------ | ----- | -------------- |
| code     | int    | true  | 200 indicates success. Other codes indicate failure. |
| msg      | string | true  | Reason for failure           |
| img_link | string | false | Link to the payment voucher           |
| img_base | string | false | Base64-encoded payment voucher       |
| sign     | string | true  | Signature. See the signing instructions at the beginning of this document. |

- Payment Voucher Query Response Examples  
  No payment voucher

```json

{
  "code":200,
  "msg":"No payment voucher available at the moment",
  "sign": "",
  "img_link": "",
  "img_base": ""
}
```



With a payment voucher

```json

{
  "code": 200,
  "msg": "",
  "sign": "3969f17cd1a551769f85967d0a05b7b6",
  "img_link": "http://dsggfgdsf.djdj?ddd=snn",
  "img_base": "data:image/png;base64,hfhshdhfhfh"
}
```


# 11. Payment Methods for the Pay-in Field pay_method


| pay_method | Country   | Value          | Description            |
|------------|--------|------------|---------------|
| pay_method     | Nigeria | NET_BANKING   | NET BANKING         |


# 12. Bank Codes
| Field Name  | Code       | Bank Name                                               |
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

# 13. Error Codes

| Status Code  | Description                |
|------|-------------------|
| 200 | Success  |
| 1000 | Internal error |
| 1001 | The IP address is not on the merchant's IP allowlist |
| 1002 | Invalid parameters |
| 1003 | Invalid signature |
| 1004 | This endpoint is currently unavailable for the merchant (Contact operations to check whether the merchant or app is missing \| disabled \| or has no payment product configured) |
| 1005 | Merchant does not exist |
| 1006 | The user's IP address is on the blocklist |
| 1007 | The user is on the blocklist |
| 1008 | Merchant app does not exist |
| 1009 | Payment product does not exist |
| 1010 | Payment channel does not exist |
| 1011 | Payment channel is still in development and is temporarily unavailable |
| 1012 | Payment channel error. Try again later. |
| 1013 | Order volume is too high. Try again later. |
| 1014 | Duplicate order number |
| 1015 | Insufficient app balance |
| 1016 | The user has placed orders too frequently. Try again later. |
| 1017 | Order record does not exist |
| 1018 | Order amount is not supported |
| 1019 | Pay-in is not enabled for the app's country |
| 1020 | Pay-out is not enabled for the app's country |
| 1021 | Failure  |
| 1036 | Endpoint is not yet available |
| 1037 | Currency is not supported |
| 1038 | Error reporting the pay-in UTR |
| 9999 | Other error |
| 3000 | The system is undergoing maintenance. Order placement is suspended. Try again later. |

# 14. Document Update Time
```
2026-11-17 20:51:12
```
