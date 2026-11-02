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
> 2. The currency unit is <span style="color:red;"> Kopek </span>.
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
| pay_method   | string | true   | Payment method: MIR_Bank.                                                                                                                       |
| price     | int  | true   | Order amount, unit: Kopek, integer.                                                                                                                  |
| order_no    | string | true   | Merchant order number.                                                                                                                        |
| success_url  | string | false  | Redirect URL for successful payment.                                                                                                                 |
| fail_url    | string | false  | Redirect URL for failed payment.                                                                                                                   |
| pay_notice_url | string | false  | Notification URL for successful payment.                                                                                                               |
| user_id    | string | true   | System user ID.                                                                                                                            |
| user_ip    | string | true   | Payer IP address.                                                                                                                           |
| attach     | string | false   | Additional parameters in JSON string format: {"account_no":"","account_type":"","name":"Name","email":"Email","phone":"Phone","identify_type":"Identity Type","identify_num":"Identity Num"} (Recommended for Brazil, Pakistan). |
| sign      | string | true   | Signature result, see the top of the document for the signature method.                                                                                                |
| timestamp   | string | false  | Order timestamp (10-digit timestamp in seconds).                                                                                                           |

- Pay-in - attach additional parameter field description

| Name     | Type  | Required | Description                                                               |
| ------------- | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| name     | string | false  | Payer name. (Recommended, mandatory for Pakistan). For Brazil individuals, provide individual name; for Brazil companies, company name. |
| email     | string | false  | Payer email. (Recommended, mandatory for Pakistan).                                           |
| phone     | string | false  | Payer phone. (Mandatory for Pakistan, must be 11 digits starting with 03).                               |
| identify_type | string | false  | Identity type                                                     |
| identify_num | string | false  | Identity number                                   |
| account_no  | string | false   | Payment account number.                                                         |
| account_type | string | false   | Account type                              |
| bank_code   | string | false  | Bank code (Mandatory for Thailand).                                                   |
| bank_name   | string | false  | Bank name.                                                               |

- Pay-in - Order Request Example

```json
 {
 "trade_no": 10003,
 "order_no": "p7158412025RAprmNz7lR",
 "app_id": 10002,
 "pay_code": 0,
 "price": 10099,
 "pay_notice_url": "http://host/api/v1/mer/cbtest",
 "attach": "",
 "sign": "3d6dea05a7c08564911b9922e16455c2",
 "user_ip": "87.200.59.100",
 "success_url": "",
 "fail_url": "",
 "user_id": "2677343"
 }
```

## 4.2 Pay-in - Order Response

| Name     | Type  | Required | Description                                                  |
| ------------ | ------ | -------- | ------------------------------------------------------------------------------------------------------------- |
| code     | int  | true   | 200: Success; Others: Failure.                                        |
| msg     | string | true   | Failure reason.                                                |
| pay_url   | string | false  | Payment link.                                                 |
| qr_code   | string | false  | PIX QR code string.                                              |
| order_no   | string | true   | Merchant order number.                                            |
| dis_order_no | string | true   | Platform order number.                                            |
| create_time | int  | true   | Creation time.                                                |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc.      |
| sign     | string | true   | Signature result, see the top of the document for the signature method.                    |

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
 "pay_url": "https://api.sunpayinr.net/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
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
| order_price | int  | true   | Order amount, unit: Kopek.                                                                                                                      |
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid by the user, unit: Kopek.</span>                                                                                                             |
| nti_time   | int  | false  | Notification initiation time.                                                                                                                     |
| payer    | string | false  | JSON string, payer info: {"name":"Name", "account":"Account", "bank":"User Bank Code", "utr2":"Bank serial number", "email":"Email", "phone":"Phone", "identify_type":"Identity Type", "identify_num":"Identity Num"}. Also includes payer-related fields from `attach`. |
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

<span style="color:red;">A pay-out request is considered successful unless an explicit failure is returned, even if it times out. Do not submit the request again unless an explicit failure is returned.</span>

| Name      | Type  | Required | Description                                        |
| -------------- | ------ | -------- | ----------------------------------------------------------------------------------------- |
| trade_no    | int  | true   | Merchant ID.                                       |
| order_no    | string | true   | Merchant order number.                                  |
| app_id     | int  | true   | Merchant appId.                                      |
| pay_code    | int  | true   | Product code, obtained from our operations.                        |
| price     | int  | true   | Order amount, unit: Kopek, integer. Cannot have decimal points after converting to Ruble. |
| account_no   | string | true   | Receiving account number.                                 |
| account_type  | string | true   | Account type: BANK. |
| account_name  | string | true   | Name.                                           |
| bank_code   | string | true   | Receiving bank code, refer to bank codes.                         |
| identify_type | string | true   | Identity type: Brazil (CPF, CNPJ), Pakistan (CNIC), India (IFSC, BANK_CODE).       |
| identify_num  | string | true   | Identity number.                                     |
| pay_notice_url | string | false  | Notification URL for successful pay-out.                         |
| attach     | string | false  | Additional parameters: {"email":"Email", "phone":"Phone", "bank_name":"Bank Name"}.    |
| user_ip    | string | true   | Receiving user IP address.                                |
| sign      | string | true   | Signature result, see the top of the document for the signature method.          |
| timestamp   | string | false  | Order timestamp (10-digit timestamp in seconds).                     |

- Pay-out - Request Parameter Example

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
 "account_no": "40817810099910004312",
 "account_name": "Ivan Ivanov",
 "bank_code": "SBER",
 "identify_type": "BANK_CODE",
 "identify_num": "044525225"
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
| order_price | int  | true   | Order amount, unit: Kopek.                                     |
| fee     | int  | false  | Order fee, unit: Kopek.                                      |
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
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid, unit: Kopek.</span>                                                                                       |
| status    | int  | true   | Order status: 1. Unpaid, <span style="color:red;">2. Success</span>, 3. Failure, 7. Rejected, 9. Reversal, 10. Processing.                                                          |
| success_time | int  | true   | Success timestamp.                                                                                              |
| order_no   | string | true   | Merchant order number.                                                                                            |
| dis_order_no | string | true   | Platform order number.                                                                                            |
| remark    | string | true   | Reason for pay-out failure.                                                                                         |
| fee     | int  | false  | Order fee, unit: Kopek.                                                                                           |
| create_time | int  | true   | Creation time.                                                                                                |
| payer    | string | false  | JSON string, payer info: {"account_name":"Name", "account_type":"Account Type: BANK", "account_no":"Account", "bank_code":"Bank Code"}.       |
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
 "payer": "{\"account_name\":\"Ivan Ivanov\",\"account_type\":\"BANK\",\"account_no\":\"40817810099910004312\",\"bank_code\":\"SBER\"}",
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
| balance    | int  | true   | Balance, unit: Kopek.                          |
| balance_frozen | int  | false  | Frozen balance, unit: Kopek.                      |
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

| Field Name | Code | Bank Name |
| :--- | :--- | :--- |
| bank_code | RUB_BANK | RUBBANK |
| bank_code | SBER | Сбер Be patient |
| bank_code | TINKOFF | Тинькофф Tinkoff |
| bank_code | АБ_РОССИЯ | АБ РОССИЯ |
| bank_code | АГРОПРОМКРЕДИТ | АГРОПРОМКРЕДИТ |
| bank_code | АКБ_Держава | АКБ Держава |
| bank_code | АКБ_ЕВРОФИНАНС_МОСНАРБАНК | АКБ ЕВРОФИНАНС МОСНАРБАНК |
| bank_code | АКБ_Приморье | АКБ Приморье |
| bank_code | АКБ_СЛАВИЯ | АКБ СЛАВИЯ |
| bank_code | АКБ_Тендер_Банк | АКБ Тендер Банк |
| bank_code | АКБ_ЦентроКредит | АКБ ЦентроКредит |
| bank_code | АКИБАНК | АКИБАНК |
| bank_code | Абсолют_Банк | Абсолют Банк |
| bank_code | Авангард | Авангард |
| bank_code | Авто_Финанс_Банк | Авто Финанс Банк |
| bank_code | Автоградбанк | Автоградбанк |
| bank_code | Автоторгбанк | Автоторгбанк |
| bank_code | Азиатско-Тихоокеанский_Банк | Азиатско-Тихоокеанский Банк |
| bank_code | Ак_Барс_Банк | Ак Барс Банк |
| bank_code | Алеф-Банк | Алеф-Банк |
| bank_code | Алмазэргиэнбанк | Алмазэргиэнбанк |
| bank_code | Алтайкапиталбанк | Алтайкапиталбанк |
| bank_code | Альфа-Банк | Альфа-Банк |
| bank_code | Америкэн_Экспресс_Банк | Америкэн Экспресс Банк |
| bank_code | Аресбанк | Аресбанк |
| bank_code | БАНК_МОСКВА-СИТИ | БАНК МОСКВА-СИТИ |
| bank_code | БАНК_ОРЕНБУРГ | БАНК ОРЕНБУРГ |
| bank_code | БАНК_УРАЛСИБ | БАНК УРАЛСИБ |
| bank_code | БЖФ_Банк | БЖФ Банк |
| bank_code | БКС_Банк | БКС Банк |
| bank_code | Банк_131 | Банк 131 |
| bank_code | Банк_Аверс | Банк Аверс |
| bank_code | Банк_Агророс | Банк Агророс |
| bank_code | Банк_Акцепт | Банк Акцепт |
| bank_code | Банк_Александровский | Банк Александровский |
| bank_code | Банк_ББР | Банк ББР |
| bank_code | Банк_БКФ | Банк БКФ |
| bank_code | Банк_ВБРР | Банк ВБРР |
| bank_code | Банк_Венец | Банк Венец |
| bank_code | Банк_Вологжанин | Банк Вологжанин |
| bank_code | Банк_ДОМ.РФ | Банк ДОМ.РФ |
| bank_code | Банк_Екатеринбург | Банк Екатеринбург |
| bank_code | Банк_Заречье | Банк Заречье |
| bank_code | Банк_Зенит | Банк Зенит |
| bank_code | Банк_ИПБ | Банк ИПБ |
| bank_code | Банк_ИТУРУП | Банк ИТУРУП |
| bank_code | Банк_Интеза | Банк Интеза |
| bank_code | Банк_Йошкар-Ола | Банк Йошкар-Ола |
| bank_code | Банк_Казани | Банк Казани |
| bank_code | Банк_Кремлевский | Банк Кремлевский |
| bank_code | Банк_МБА-МОСКВА | Банк МБА-МОСКВА |
| bank_code | Банк_Мир_Привилегий | Банк Мир Привилегий |
| bank_code | Банк_Объединенный_капитал | Банк Объединенный капитал |
| bank_code | Банк_Оранжевый | Банк Оранжевый |
| bank_code | Банк_ПСКБ | Банк ПСКБ |
| bank_code | Банк_Пермь | Банк Пермь |
| bank_code | Банк_Развитие-Столица | Банк Развитие-Столица |
| bank_code | Банк_Раунд | Банк Раунд |
| bank_code | Банк_Русский_Стандарт | Банк Русский Стандарт |
| bank_code | Банк_СИАБ | Банк СИАБ |
| bank_code | Банк_Саратов | Банк Саратов |
| bank_code | Банк_Синара | Банк Синара |
| bank_code | Банк_Снежинский | Банк Снежинский |
| bank_code | Банк_Финсервис | Банк Финсервис |
| bank_code | Банк_ЧБРР | Банк ЧБРР |
| bank_code | Белгородсоцбанк | Белгородсоцбанк |
| bank_code | Бланк_банк | Бланк банк |
| bank_code | Братский_АНКБ | Братский АНКБ |
| bank_code | БыстроБанк | БыстроБанк |
| bank_code | ВЛАДБИЗНЕСБАНК | ВЛАДБИЗНЕСБАНК |
| bank_code | ВНЕШФИНБАНК | ВНЕШФИНБАНК |
| bank_code | ВТБ | ВТБ |
| bank_code | ВУЗ_банк | ВУЗ банк |
| bank_code | Вайлдберриз_Банк | Вайлдберриз Банк |
| bank_code | ГЕНБАНК | ГЕНБАНК |
| bank_code | ГОРБАНК | ГОРБАНК |
| bank_code | ГУТА-БАНК | ГУТА-БАНК |
| bank_code | Газпромбанк | Газпромбанк |
| bank_code | Газтрансбанк | Газтрансбанк |
| bank_code | Газэнергобанк | Газэнергобанк |
| bank_code | Гарант-Инвест_банк | Гарант-Инвест банк |
| bank_code | Дальневосточный_банк | Дальневосточный банк |
| bank_code | Датабанк | Датабанк |
| bank_code | Джей_энд_Ти_Банк | Джей энд Ти Банк |
| bank_code | Дойче_Банк | Дойче Банк |
| bank_code | Долинск | Долинск |
| bank_code | Драйв_Клик_Банк | Драйв Клик Банк |
| bank_code | Енисейский_Объединенный_банк | Енисейский Объединенный банк |
| bank_code | Земский_банк | Земский банк |
| bank_code | Золотая_Корона_(РНКО_Платежный_центр) | Золотая Корона (РНКО Платежный центр) |
| bank_code | ИК_Банк | ИК Банк |
| bank_code | ИНЭКО | ИНЭКО |
| bank_code | ИШБАНК | ИШБАНК |
| bank_code | Инбанк | Инбанк |
| bank_code | Ингосстрах_Банк | Ингосстрах Банк |
| bank_code | КБ_ИС_Банк | КБ ИС Банк |
| bank_code | КБ_Новый_Век | КБ Новый Век |
| bank_code | КБ_Пойдем! | КБ Пойдем! |
| bank_code | КБ_РостФинанс | КБ РостФинанс |
| bank_code | КБ_СОЛИДАРНОСТЬ | КБ СОЛИДАРНОСТЬ |
| bank_code | КБ_Стройлесбанк | КБ Стройлесбанк |
| bank_code | КБ_Хлынов | КБ Хлынов |
| bank_code | КБ_ЮНИСТРИМ | КБ ЮНИСТРИМ |
| bank_code | КИВИ_Банк | КИВИ Банк |
| bank_code | КОШЕЛЕВ-БАНК | КОШЕЛЕВ-БАНК |
| bank_code | КЭБ_ЭйчЭнБи_Банк | КЭБ ЭйчЭнБи Банк |
| bank_code | Кредит_Европа_Банк_(Россия) | Кредит Европа Банк (Россия) |
| bank_code | Кредит_Урал_Банк | Кредит Урал Банк |
| bank_code | Крокус-Банк | Крокус-Банк |
| bank_code | Кубань_Кредит | Кубань Кредит |
| bank_code | Кубаньторгбанк | Кубаньторгбанк |
| bank_code | Кузнецкбизнесбанк | Кузнецкбизнесбанк |
| bank_code | ЛОКО-Банк | ЛОКО-Банк |
| bank_code | Ланта-Банк | Ланта-Банк |
| bank_code | Левобережный | Левобережный |
| bank_code | МЕЖДУНАРОДНЫЙ_ФИНАНСОВЫЙ_КЛУБ | МЕЖДУНАРОДНЫЙ ФИНАНСОВЫЙ КЛУБ |
| bank_code | МЕТКОМБАНК | МЕТКОМБАНК |
| bank_code | МКБ | МКБ |
| bank_code | МОНЕТА | МОНЕТА |
| bank_code | МОРСКОЙ_БАНК | МОРСКОЙ БАНК |
| bank_code | МОСКОМБАНК | МОСКОМБАНК |
| bank_code | МОСОБЛБАНК | МОСОБЛБАНК |
| bank_code | МС_Банк_Рус | МС Банк Рус |
| bank_code | МСП_Банк | МСП Банк |
| bank_code | МТС-Банк | МТС-Банк |
| bank_code | Металлинвестбанк | Металлинвестбанк |
| bank_code | Мир_Бизнес_Банк | Мир Бизнес Банк |
| bank_code | Модульбанк | Модульбанк |
| bank_code | Москоммерцбанк | Москоммерцбанк |
| bank_code | НБД-Банк | НБД-Банк |
| bank_code | НДБанк | НДБанк |
| bank_code | НИКО-БАНК | НИКО-БАНК |
| bank_code | НК_Банк | НК Банк |
| bank_code | НКО_МОБИ.Деньги | НКО МОБИ.Деньги |
| bank_code | НКО_Мобильная_карта | НКО Мобильная карта |
| bank_code | НКО_НРД | НКО НРД |
| bank_code | НКО_Перспектива_(Все_платежи) | НКО Перспектива (Все платежи) |
| bank_code | НКО_ЭЛЕКСНЕТ | НКО ЭЛЕКСНЕТ |
| bank_code | НКО_Элексир | НКО Элексир |
| bank_code | НКО_ЮМани | НКО ЮМани |
| bank_code | НОВИКОМБАНК | НОВИКОМБАНК |
| bank_code | НОКССБАНК | НОКССБАНК |
| bank_code | НРБанк | НРБанк |
| bank_code | НС_Банк | НС Банк |
| bank_code | Нацинвестпромбанк | Нацинвестпромбанк |
| bank_code | Национальный_стандарт | Национальный стандарт |
| bank_code | Новобанк | Новобанк |
| bank_code | Норвик_Банк | Норвик Банк |
| bank_code | ОТП_Банк | ОТП Банк |
| bank_code | Озон_Банк_(Ozon) | Озон Банк (Ozon) |
| bank_code | Открытие | Открытие |
| bank_code | Первый_Дортрансбанк | Первый Дортрансбанк |
| bank_code | Первый_Инвестиционный_Банк | Первый Инвестиционный Банк |
| bank_code | Почта_Банк | Почта Банк |
| bank_code | Примсоцбанк | Примсоцбанк |
| bank_code | Прио-Внешторгбанк | Прио-Внешторгбанк |
| bank_code | ПроБанк | ПроБанк |
| bank_code | ПромТрансБанк | ПромТрансБанк |
| bank_code | Промсвязьбанк | Промсвязьбанк |
| bank_code | Промсельхозбанк | Промсельхозбанк |
| bank_code | РЕСО_Кредит | РЕСО Кредит |
| bank_code | РНКБ_Банк | РНКБ Банк |
| bank_code | РНКО_Деньги.Мэйл.Ру | РНКО Деньги.Мэйл.Ру |
| bank_code | РУСНАРБАНК | РУСНАРБАНК |
| bank_code | Райффайзен_Банк | Райффайзен Банк |
| bank_code | Реалист_Банк | Реалист Банк |
| bank_code | Ренессанс_Кредит | Ренессанс Кредит |
| bank_code | РосДорБанк | РосДорБанк |
| bank_code | Росбанк | Росбанк |
| bank_code | Россельхозбанк | Россельхозбанк |
| bank_code | Роял_Кредит_Банк | Роял Кредит Банк |
| bank_code | Русьуниверсалбанк | Русьуниверсалбанк |
| bank_code | СДМ-Банк | СДМ-Банк |
| bank_code | СИНКО-БАНК | СИНКО-БАНК |
| bank_code | СМП_Банк | СМП Банк |
| bank_code | СНГБ | СНГБ |
| bank_code | СОЦИУМ-БАНК | СОЦИУМ-БАНК |
| bank_code | Санкт-Петербург | Санкт-Петербург |
| bank_code | Сбербанк | Сбербанк |
| bank_code | Свой_Банк | Свой Банк |
| bank_code | Севергазбанк | Севергазбанк |
| bank_code | Северный_Народный_Банк | Северный Народный Банк |
| bank_code | Сибсоцбанк | Сибсоцбанк |
| bank_code | Ситибанк | Ситибанк |
| bank_code | Совкомбанк | Совкомбанк |
| bank_code | Солид_Банк | Солид Банк |
| bank_code | Ставропольпромстройбанк | Ставропольпромстройбанк |
| bank_code | ТАТСОЦБАНК | ТАТСОЦБАНК |
| bank_code | Таврический_Банк | Таврический Банк |
| bank_code | Тимер_Банк | Тимер Банк |
| bank_code | Тинькофф | Тинькофф |
| bank_code | Тойота_Банк | Тойота Банк |
| bank_code | Тольяттихимбанк | Тольяттихимбанк |
| bank_code | Томскпромстройбанк | Томскпромстройбанк |
| bank_code | Точка_'ФК_Открытие' | Точка 'ФК Открытие' |
| bank_code | Точка_Банк | Точка Банк |
| bank_code | Точка_КИВИ | Точка КИВИ |
| bank_code | Транскапиталбанк | Транскапиталбанк |
| bank_code | Трансстройбанк | Трансстройбанк |
| bank_code | УБРиР | УБРиР |
| bank_code | УРАЛПРОМБАНК | УРАЛПРОМБАНК |
| bank_code | Углеметбанк | Углеметбанк |
| bank_code | Урал_ФД | Урал ФД |
| bank_code | Уралфинанс | Уралфинанс |
| bank_code | ФИНАМ | ФИНАМ |
| bank_code | ФОРА-БАНК | ФОРА-БАНК |
| bank_code | ФФИН_Банк_(Цифра_банк) | ФФИН Банк (Цифра банк) |
| bank_code | Форштадт | Форштадт |
| bank_code | Хайс | Хайс |
| bank_code | Хакасский_муниципальный_банк | Хакасский муниципальный банк |
| bank_code | Хоум_Кредит_Банк_(Хоум_Банк) | Хоум Кредит Банк (Хоум Банк) |
| bank_code | Центр-инвест | Центр-инвест |
| bank_code | ЧЕЛИНДБАНК | ЧЕЛИНДБАНК |
| bank_code | ЧЕЛЯБИНВЕСТБАНК | ЧЕЛЯБИНВЕСТБАНК |
| bank_code | Экономбанк | Экономбанк |
| bank_code | Экспобанк | Экспобанк |
| bank_code | Элита | Элита |
| bank_code | Элплат | Элплат |
| bank_code | Энергобанк | Энергобанк |
| bank_code | Энерготрансбанк | Энерготрансбанк |
| bank_code | Эс-Би-Ай_Банк | Эс-Би-Ай Банк |
| bank_code | ЮГ-Инвестбанк | ЮГ-Инвестбанк |
| bank_code | Юникредит_Банк | Юникредит Банк |
| bank_code | Яндекс_Банк | Яндекс Банк |

# 12. Error Codes

| Status Code | Description                                                              |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 200     | Success                                                                |
| 1000    | Internal Error                                                             |
| 1001    | IP not in merchant IP whitelist.                                                    |
| 1002    | Parameter Error                                                            |
| 1003    | Signature Error                                                            |
| 1004    | Interface currently unavailable for the merchant (Contact operations to verify: Merchant or App (Not exist\|Closed\|Product not configured)) |
| 1005    | Merchant does not exist.                                                        |
| 1006    | Current user IP is in the blacklist.                                                  |
| 1007    | Current user is in the blacklist.                                                   |
| 1008    | Merchant App does not exist.                                                      |
| 1009    | Payment product does not exist.                                                    |
| 1010    | Payment channel does not exist.                                                    |
| 1011    | Payment channel development not completed, temporarily unavailable.                                  |
| 1012    | Payment channel exception, please try again later.                                           |
| 1013    | High order volume, please try again later.                                               |
| 1014    | Duplicate order number.                                                        |
| 1015    | Insufficient app balance.                                                       |
| 1016    | Frequent order placement by the same user, please try again later.                                   |
| 1017    | Order record does not exist.                                                      |
| 1018    | Current amount not supported.                                                     |
| 1019    | Pay-in not enabled for the app's country.                                               |
| 1020    | Pay-out not enabled for the app's country.                                               |
| 1021    | Failure                                                                |
| 1036    | Interface not yet open.                                                        |
| 1037    | Currency not supported.                                                        |
| 1038    | Pay-in utr reporting error.                                                      |
| 9999    | Other errors.                                                             |
| 3000    | System maintenance, order placement suspended, please try again later.                                 |

# 13. Pay-in Checkout Interface

Address: https://{api_domain}/api/v1/cashApi/CashIn.html
Request Method: GET

### Parameters:

| Name    | Type  | Required | Description            |
|------------|--------|----------|------------------------------------|
| app_id   | string | true   | Merchant app_id.          |
| order_no  | string | true   | Merchant order number.       |
| amount   | string | true   | Merchant Amount (Unit: Russian Ruble) |
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
