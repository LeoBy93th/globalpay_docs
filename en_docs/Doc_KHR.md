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
> 4. sign=md5(the string assembled in the previous step). The signature result is a 32-bit lowercase string.
>
> 5. The signature key can be found in the Merchant Backend -> Basic Information, or by inquiring with our customer service.

# 3. Precautions

## 3.1 Interface Related

> 1. All interfaces in this document use standard HTTP communication protocols, submitted via POST. Both request and response Content-type are application/json, and the character encoding is unified as UTF-8.
>
> 2. The currency unit is <span style="color:red;"> cent  1 KHR=100 or 1 KHRUSD=100 </span>.
>
> 3. The IP address for requesting the interface needs to be whitelisted.
>
> 4. Collect the real user IP for user_ip as much as possible; if truly unavailable, leave it blank. Do not use local IPs like 127.0.0.1.

## 3.2 Callback Related

> 1. The callback reception was successful. Please return the text "<span style="color:red;">success</span>". This text must not contain any other characters. Otherwise, the system will no longer push this order information; otherwise, it will push it multiple times.
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
| pay_method   | string | true   | Payment method.                                                                                                                     |
| price     | int  | true   | Order amount, unit: cent, integer.                                                                                                                  |
| order_no    | string | true   | Merchant order number.                                                                                                                        |
| success_url  | string | false  | Redirect URL for successful payment.                                                                                                                 |
| fail_url    | string | false  | Redirect URL for failed payment.                                                                                                                   |
| pay_notice_url | string | false  | Notification URL for successful payment.                                                                                                               |
| user_id    | string | true   | System user ID.                                                                                                                            |
| user_ip    | string | true   | Payer IP address.                                                                                                                           |
| attach     | string | false   | Additional parameters in JSON string format: {"name":"Name","email":"Email" |
| sign      | string | true   | Signature result, see the top of the document for the signature method.                                                                                                |
| timestamp   | string | false  | Order timestamp (10-digit timestamp in seconds).                                                                                                           |

- Pay-in - attach additional parameter field description

| Name         | Type   | Required | Description                              |
| ------------ | ------ | -------- | ---------------------------------------- |
| name         | string | false    | Payer name                               |
| email        | string | false    | Payer email                              |

* Collection - Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"name\":\"Zhang San\",\"email\":\"zhangsan@example.com\"}",
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
 "dis_order_no": "2025071130770572062498816india1oushe",
 "create_time": 1752825512,
 "pay_url": "https://api.test.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5", 
 "pay_info": ""
}
```




# 5. Pay-in Callback Notification (post/json)

Push address: The `pay_notice_url` provided by the merchant during order placement. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 5.1 Pay-in Callback - Request Parameters

| Name     | Type  | Required | Description                                                                                                                              |
| ------------ | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| trade_no   | int  | true   | Merchant ID.                                                                                                                             |
| status    | int  | true   | Order status: <span style="color:red;">2. Success</span>, 3. Failure.                                                                                                                 |
| order_no   | string | true   | Merchant order number.                                                                                                                        |
| dis_order_no | string | true   | Platform order number.                                                                                                                        |
| order_price | int  | true   | Order amount, unit: cent.                                                                                                                      |
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid by the user, unit: cent.</span>                                                                                                             |
| nti_time   | int  | false  | Notification initiation time.                                                                                                                     |
| payer    | string | false  | JSON string, payer info: {"name":"Name", "account":"Account", "bank":"User Bank Code", "utr2":"Bank serial number", "email":"Email", "phone":"Phone", "identify_type":"Identity Type", "identify_num":"CPF, CNPJ"}. Also includes payer-related fields from `attach`. |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc.                                                                                  |
| create_time | int  | true   | Creation time.                                                                                                                            |
| sign     | string | true   | Signature result, see the top of the document for the signature method.                                                                                                |

- Pay-in Callback - Request Parameter Example

```json

{
 "trade_no": 10003,
 "status": 3,
 "order_no": "p71584120256SlWlKkymb",
 "dis_order_no": "2025071130460153942908928india1sKQbX",
 "order_price": 10099,
 "real_price": 10000,
 "payer": "{\"name\":\"Name\",\"email\":\"Email\",\"phone\":\"Phone\",\"identify_type\":\"Identity Type\",\"identify_num\":\"CPF,CNPJ\"}",
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

| Name           | Type   | Required | Description                                                                                     |
| -------------- | ------ | -------- | ----------------------------------------------------------------------------------------------- |
| trade_no       | int    | true     | Merchant number                                                                                 |
| order_no       | string | true     | Merchant order number                                                                           |
| app_id         | int    | true     | Merchant appId                                                                                  |
| pay_code       | int    | true     | Product code, contact our operations team to obtain                                             |
| price          | int    | true     | Order amount, unit: cents, integer. |
| account_no     | string | true     | Receiving account                                                                               |
| account_name   | string | true     | Recipient name                                                                                  |
| bank_code      | string | true     | Receiving bank code, refer to bank code list                                                    |
| pay_notice_url | string | false    | Payout success callback URL                                                                     |
| attach         | string | false    | Additional parameters {"email":"Email","phone":"Phone number","bank_name":"Bank name"}          |
| user_ip        | string | true     | Recipient user IP                                                                               |
| sign           | string | true     | Signature result, signature method described at the top of this document                        |
| timestamp      | string | false    | Order timestamp, 10-digit Unix timestamp in seconds                                             |

* Payout - Request Example

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
 "bank_code": "VCB"
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
 "dis_order_no": "2025071130776296733810688india1Dhr7H",
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
| order_price | int  | true   | Order amount, unit: cent.                                     |
| fee     | int  | false  | Order fee, unit: cent.                                      |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status    | int  | true   | Order status: <span style="color:red;">2. Success</span>, 3. Failure, 7. Rejected, 9. Reversal.                  |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc. |
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
 "dis_order_no": "2025071130700746140950528india15rLVI",
 "nti_time": 1752808888,
 "create_time": 1752808865,
 "sign": "746140950528indi"
}
```

## 7.2 Pay-out Callback Response Description

If the callback is successfully received and processed, please return <span style="color:red;">`success`</span>. The system will stop pushing this order information; otherwise, it will be resent multiple times.

# 8. Query Order Interface (Common for Pay-in and Pay-out)

(The (Request IP needs to be whitelisted by contacting us)
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
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid, unit: cent.</span>                                                                                       |
| status    | int  | true   | Order status: 1. Unpaid, <span style="color:red;">2. Success</span>, 3. Failure, 7. Rejected, 9. Reversal, 10. Processing.                                                          |
| success_time | int  | true   | Success timestamp.                                                                                              |
| order_no   | string | true   | Merchant order number.                                                                                            |
| dis_order_no | string | true   | Platform order number.                                                                                            |
| remark    | string | true   | Reason for pay-out failure.                                                                                         |
| fee     | int  | false  | Order fee, unit: cent.                                                                                           |
| create_time | int  | true   | Creation time.                                                                                                |
| payer    | string | false  | JSON string, payer info: {"account_name":"Name", "account_type":"Account Type: PHONE, BANK", "account_no":"Account", "bank_code":"Bank Code"}.       |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc.                                                      |
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
 "payer": "{\"account_name\":\"Nguyen Van A\",\"account_type\":\"BANK\",\"account_no\":\"1234567890123\",\"bank_code\":\"VCB\"}",
 "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Pay-out Balance Query Interface

(The (Request IP needs to be whitelisted by contacting us)
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
 "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Response

| Name      | Type  | Required | Description                               |
| -------------- | ------ | -------- | ----------------------------------------------------------------------- |
| code      | int  | true   | 200: Query successful; Others: Failure.                 |
| msg      | string | true   | Failure reason.                             |
| balance    | int  | true   | Balance, unit: cent.                          |
| balance_frozen | int  | false  | Frozen balance, unit: cent.                      |
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

(The (Request IP needs to be whitelisted by contacting us)
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
 "dis_order_no": "35011C02gljuf6k0800india1lVY",
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
 "img_link": "http://dsggfgdsf.djdj?ddd=snn",
 "img_base": "data:image/png;base64,hfhshdhfhfh"
}
```

# 11、代收银行编码 代收字段 pay_method

| 字段         | 值               | 描述   |
|------------|------------------|------|
| pay_method | KHR_QR     | QR transfer |
| pay_method | KHR_BANK     | BANK transfer |

# 12. Bank Codes

| Field Name | Code       | Bank Name                                              |
|:-----------|:-----------|:-------------------------------------------------------|
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
| amount   | string | true   | Merchant Amount (Unit: Dong) |
| notice_url | string | false  | Asynchronous notification address. |
|pay_code|int|true|Product Code|

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{MerchantOrderNumber}}&amount={{MerchantAmount}}&notice_url={{AsynchronousNotificationAddress}}&pay_code={{ProductCode}}
```
---
# 15. Document Update Time
```
2026-08-24 13:05:52
```
