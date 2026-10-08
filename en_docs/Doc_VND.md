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
> 2. The currency unit is <span style="color:red;"> Xu </span>.
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
| pay_method   | string | true   | Payment method: VND-Payment.                                                                                                                     |
| price     | int  | true   | Order amount, unit: Xu, integer.                                                                                                                  |
| order_no    | string | true   | Merchant order number.                                                                                                                        |
| success_url  | string | false  | Redirect URL for successful payment.                                                                                                                 |
| fail_url    | string | false  | Redirect URL for failed payment.                                                                                                                   |
| pay_notice_url | string | false  | Notification URL for successful payment.                                                                                                               |
| user_id    | string | true   | System user ID.                                                                                                                            |
| user_ip    | string | true   | Payer IP address.                                                                                                                           |
| attach     | string | true   | Additional parameters in JSON string format: {"account_no":"","account_type":"","name":"Name","email":"Email","phone":"Phone","identify_type":"Identity Type","identify_num":"CPF,CNPJ,IFSC,BANK_CODE"} (Recommended for Brazil, Pakistan). |
| sign      | string | true   | Signature result, see the top of the document for the signature method.                                                                                                |
| timestamp   | string | false  | Order timestamp (10-digit timestamp in seconds).                                                                                                           |

- Pay-in - attach additional parameter field description

| Name         | Type   | Required | Description                              |
| ------------ | ------ | -------- | ---------------------------------------- |
| name         | string | false    | Payer name                               |
| email        | string | false    | Payer email                              |
| phone        | string | false    | Payer phone number                       |
| account_no   | string | true     | Payment account                          |
| account_type | string | true     | Account type: PHONE, BANK (bank account) |
| bank_code    | string | false    | Bank code                                |
| bank_name    | string | false    | Bank name                                |

* Collection - Request Example

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


- Collection - pay_info Additional Parameters

| Parameter | Type | Required | Description         |
|-----------|------|----------|---------------------|
| acc_no | string | false | Recipient Account   |
| bank | string | false | Bank Information    |
| memo | string | false | Payment Description |
| name | string | false | Payee Name          |
| pay_raw | string | false | Payment Original Information |


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
 "pay_url": "https://api.sunpayinr.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5", 
 "pay_info": "{\"acc_no\":\"96311*****2701\",\"bank\":\"BIDV\",\"memo\":\"B4VGR8dfkdalds\",\"name\":\"POT TRAN****** VAN DU\",\"pay_raw\":\"00020101021238580010A00000072701280006970418011496311435212701**************************\"}"
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
| order_price | int  | true   | Order amount, unit: Xu.                                                                                                                      |
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid by the user, unit: Xu.</span>                                                                                                             |
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
| price          | int    | true     | Order amount, unit: cents, integer. After conversion to currency unit, decimals are not allowed |
| account_no     | string | true     | Receiving account                                                                               |
| account_type   | string | true     | Account type: PHONE, BANK (bank account)                                                        |
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
 "account_type": "BANK",
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
| order_price | int  | true   | Order amount, unit: Xu.                                     |
| fee     | int  | false  | Order fee, unit: Xu.                                      |
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
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid, unit: Xu.</span>                                                                                       |
| status    | int  | true   | Order status: 1. Unpaid, <span style="color:red;">2. Success</span>, 3. Failure, 7. Rejected, 9. Reversal, 10. Processing.                                                          |
| success_time | int  | true   | Success timestamp.                                                                                              |
| order_no   | string | true   | Merchant order number.                                                                                            |
| dis_order_no | string | true   | Platform order number.                                                                                            |
| remark    | string | true   | Reason for pay-out failure.                                                                                         |
| fee     | int  | false  | Order fee, unit: Xu.                                                                                           |
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
| balance    | int  | true   | Balance, unit: Xu.                          |
| balance_frozen | int  | false  | Frozen balance, unit: Xu.                      |
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


# 11. Bank Codes

| Field Name | Code       | Bank Name                                              |
|:-----------|:-----------|:-------------------------------------------------------|
| bank_code  | PVCB       | Vietnam Public Joint Stock Commercial Bank             |
| bank_code  | VPB        | Vietnam Prosperity Bank                                |
| bank_code  | VCB        | Bank for Foreign Trade of Vietnam                      |
| bank_code  | CTG        | Vietnam Bank for Industry and Trade                    |
| bank_code  | BIDV       | Bank for Investment & Development of Vietnam           |
| bank_code  | EIB        | Vietnam Export Import Commercial Joint Stock Bank      |
| bank_code  | TCB        | Vietnam Technological And Commercial Joint Stock Bank  |
| bank_code  | TPB        | Tien Phong Commercial Joint Stock Bank                 |
| bank_code  | LPB        | Viet Nam Post Joint Stock Commercial Bank              |
| bank_code  | PGB        | Petrolimex Group Commercial Joint Stock Bank           |
| bank_code  | SEAB       | Southeast Asia Commercial Joint Stock Bank             |
| bank_code  | SGB        | Saigon Bank For Industry And Trade                     |
| bank_code  | VIB        | Vietnam International Commercial Joint Stock Bank      |
| bank_code  | MBV        | Modern Bank of Vietnam                                 |
| bank_code  | HDB        | HoChiMinh City Development Joint Stock Commercial Bank |
| bank_code  | GPB        | Global Petro Bank                                      |
| bank_code  | BVB        | Baoviet Joint Stock Commercial Bank                    |
| bank_code  | VRB        | Vietnam - Russia Joint Venture Bank                    |
| bank_code  | VNDSHINHAN | SHINHAN Bank                                           |
| bank_code  | WOORI      | WOORI BANK                                             |
| bank_code  | MSB        | Vietnam Maritime Commercial Joint Stock Bank           |
| bank_code  | STB        | Saigon Thuong Tin Commercial Joint Stock Bank          |
| bank_code  | NASB       | North Asia Commercial Joint Stock Bank                 |
| bank_code  | VAB        | Vietnam Asia Commercial Joint Stock Bank               |
| bank_code  | ABB        | An Binh Commercial Joint Stock Bank                    |
| bank_code  | ACB        | Asia Commercial Bank                                   |
| bank_code  | AGRIBANK   | Vietnam Bank For Agriculture and Rural Development     |
| bank_code  | MBB        | Military Commercial Joint Stock Bank                   |
| bank_code  | OCB        | Orient Commercial Joint Stock Bank                     |
| bank_code  | SHB        | Saigon – Hanoi Commercial Joint Stock Bank             |
| bank_code  | NAB        | Nam A Commercial Joint Stock Bank                      |
| bank_code  | SCB        | Saigon Commercial Bank                                 |
| bank_code  | KLB        | Kien Long Commercial Joint Stock Bank                  |
| bank_code  | VTTB       | Viet Nam Thuong Tin Commercial Joint Stock Bank        |
| bank_code  | VCB_CAP    | Viet Capital Bank                                      |
| bank_code  | PBVN       | Public Bank Vietnam Limited                            |
| bank_code  | IVB        | Indovina Bank LTD                                      |
| bank_code  | UOB        | United Overseas Bank                                   |
| bank_code  | VIKKI      | Vikki Digital Bank Limited                             |
| bank_code  | NCB        | National Citizen Bank                                  |
| bank_code  | VIETCREDIT | Công Ty Tài Chính Cổ Phần Tín Việt                     |
| bank_code  | CIMB       | TNHH MTV CIMB                                          |
| bank_code  | VNDDBS     | DBS Bank (Ho Chi Minh City)                            |
| bank_code  | UMEE       | UMEE by Kienlongbank                                   |
| bank_code  | NONGHYUP   | NongHyup Bank Hanoi Branch                             |
| bank_code  | LIO        | Liobank by OCB                                         |
| bank_code  | VNDHSBC    | HSBC Vietnam                                           |
| bank_code  | KB         | KB Kookmin Bank Ha Noi Branch                          |
| bank_code  | TIMO       | Timo by Viet Capital Bank                              |
| bank_code  | VNPT       | VNPT Fintech                                           |
| bank_code  | MIRAE      | Mirae Asset Finance Company (Vietnam)                  |
| bank_code  | CAKE       | Cake by VP Bank                                        |
| bank_code  | VIETTELPAY | Viettel Digital Services Corporation                   |
| bank_code  | UBANK      | Ubank by VPBank                                        |
| bank_code  | VBSP       | Vietnam bank for social policies                       |
| bank_code  | VNDKBANK   | Kasikornbank Bank                                      |
| bank_code  | HLB        | Hong Leong Bank Vietnam Limited                        |
| bank_code  | DAB        | DongA Bank                                             |
| bank_code  | COOP       | Vietnam Cooperative Bank                               |
| bank_code  | BAB        | Ngân hàng TMCP Bắc Á (BAC A BANK)                      |
| bank_code  | VCBNEO     | Ngân hàng số Vietcombank                               |

# 12. Error Codes

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

# 13. Pay-in Checkout Interface

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
# 14. Document Update Time
```
2026-08-24 13:05:52
```
