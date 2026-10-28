# 1. Integration Process

> 1. Business negotiation and account opening, discuss relevant rates.
> 2. Contact operations to create merchant ID, secret key, merchant appId, product code, and apiUrl.
> 3. After development is complete, conduct joint debugging and testing to verify request, notification and other information integrity.

# 2. MD5 Signature Algorithm

> 1. Sort all parameters as key-value pairs (key1=value1) in ascending order (empty parameter values are not included in the signature).
> 2. Concatenate as `key1=value1&key2=value2`.
> 3. Append the merchant secret key: `key1=value1&key2=value2...&key=MerchantSecretKey`.
> 4. `sign = MD5(above concatenated string)` – signature result is 32-bit lowercase.
> 5. The signature secret key can be found in Merchant Backend -> Basic Information, or ask our customer service.

# 3. Important Notes

## 3.1 Interface Related
> 1. All interfaces in this document use HTTP standard protocol, POST submission. Request and response `Content-Type` are both `application/json`, character encoding is UTF-8.
> 2. The amount unit is <span style="color:red;">cent （centavo） 1 peso = 100 centavos</span>.
> 3. The IP address calling the interface needs to be whitelisted.
> 4. `user_ip` – try to collect the user's real IP; if not available, leave it blank. Do not use local IPs like `127.0.0.1`.

## 3.2 Callback Related
> 1. If the callback is successfully processed, return the plain text <span style="color:red;">`success`</span> without any other characters. The system will not push this order again; otherwise, it will retry multiple times.
> 2. During async notification interaction, if the response is not `success`, the notification is considered failed and will be retried periodically. Retry intervals: 1m, 1m, 4m, 10m, 10m, 1h, 2h, 6h, 15h.
> 3. If `pay_notice_url` is empty, it means the merchant does not require a callback, and the system will not push notifications.

# 4. Collection Order Creation Interface

> (Order creation IP needs to be whitelisted by us)  
> Order URL: `https://{api_domain}/api/v1/payApi/CreatePayInOrder`

## 4.1 Collection – Order Request Parameters

| Name            | Type   | Required | Description                                                                                                         |
|-----------------|--------|----------|---------------------------------------------------------------------------------------------------------------------|
| trade_no        | int    | true     | Merchant number                                                                                                     |
| app_id          | int    | true     | Merchant appId                                                                                                      |
| pay_code        | int    | true     | Product code, contact our operations to obtain                                                                     |
| price           | int    | true     | Order amount, unit: cent, integer                                                                                  |
| pay_method        | string | true     | Payment method, see the payment methods below         |
| order_no        | string | true     | Merchant order number                                                                                               |
| success_url     | string | false    | Payment success redirect URL                                                                                        |
| fail_url        | string | false    | Payment failure redirect URL                                                                                        |
| pay_notice_url  | string | false    | Payment success notification URL                                                                                    |
| user_id         | string | false    | User ID                                                                                                             |
| user_ip         | string | false    | Payer IP                                                                                                            |
| identify_type   | string | false    | ID type: `RUT` (Tax ID), `PP` (Passport)                                                                            |
| identify_num    | string | false    | ID number: RUT (9 digits: 8 digits + check digit), PP (9 to 16 characters)                                          |
| attach          | string | true     | Additional parameters JSON string                                                                                  |
| sign            | string | true     | Signature result, see signature method at top of document                                                           |

- **Collection – attach Additional Parameter Field Description**

| Name            | Type   | Required | Description                                                    |
|-----------------|--------|----------|----------------------------------------------------------------|
| name            | string | true     | Payer name                                                     |
| identify_type   | string | true     | ID type: `RUT` (Tax ID), `PP` (Passport)                       |
| identify_num    | string | true     | ID number: RUT (9 digits: 8 digits + check digit), PP (9 to 16 characters) |
| phone           | string | false    | Phone number (provide real phone number if possible, can be filled if not available) |
| email           | string | false    | Email (provide real email if possible, can be filled if not available) |

- **Collection – Order Request Example**

```json
{
  "trade_no": 165,
  "order_no": "p71584121t1693047656571",
  "app_id": 51,
  "pay_code": 91145,
  "price": 10000,
  "pay_method": "NET_BANKING_CLP",
  "success_url": "",
  "fail_url": "",
  "pay_notice_url": "http://www.test.com",
  "pay_remind_url": "",
  "attach": "",
  "user_ip": "87.200.59.188",
  "user_id": "2677323",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
## 4.2 Collection – Order Response
| Name         | Type   | Required | Description                                        |
| ------------ | ------ | -------- | -------------------------------------------------- |
| code         | int    | true     | 200: order created successfully, others: failure   |
| msg          | string | true     | Failure reason                                     |
| pay_url      | string | true     | Payment URL                                        |
| qr_code      | string | true     | QR code string                                     |
| order_no     | string | true     | Merchant order number                              |
| dis_order_no | string | true     | Platform order number                              |
| create_time  | int    | true     | Creation time                                      |
| sign         | string | true     | Signature result, see signature method at top      |

- **Response Example**
Failure:

```json
{
  "code": 10009,
  "msg": "ip not in merchant whitelist, ip:158.28.87.267",
  "sign": "db3406277185f9660b3b928d6adc7bc4",
  "pay_url": "",
  "qr_code": "",
  "order_no": "",
  "dis_order_no": "",
  "create_time": 0
}
```
Success:

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

# 5. Pay-in Callback Notification (post/json)
Push URL: pay_notice_url provided by merchant in order creation.
Callback IP: call_back_server_ip. Please add our IP to the callback whitelist.

## 5.1 Collection Callback – Request Parameters
| Name            | Type   | Required | Description  |
|-----------------|--------|----------|---------------------------------------------------------------------------------------------------------------------| 
| trade_no   | int  | true   | Merchant ID.                                                                                                                             |
| status    | int  | true   | Order status: <span style="color:red;">2. Success</span>, 3. Failure.                                                                                                                 |
| order_no   | string | true   | Merchant order number.                                                                                                                        |
| dis_order_no | string | true   | Platform order number.                                                                                                                        |
| order_price | int  | true   | Order amount, unit: Centavos.                                                                                                                      |
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid by the user, unit: Centavos.</span>                                                                                                             |
| nti_time   | int  | false  | Notification initiation time.                                                                                                                     |
| payer    | string | false  | JSON string, payer info: {"name":"Name","email":"Email","phone":"Phone","identify_type":"ID Type","identify_num":"RUT ID"}. |
| pay_info   | string | false  | Payment information JSON string. e.g., original pay-in/pay-out info, card number, name, bank, etc.                                                                                  |
| create_time | int  | true   | Creation time.                                                                                                                            |
| sign     | string | true   | Signature result, see the top of the document for the signature method. |

```json
{
  "trade_no": 165,
  "status": 2,
  "order_no": "p71584121t1693047656571",
  "dis_order_no": "lufei169246816001692",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1693057443,
  "payer": "{\"name\":\"Name\",\"email\":\"Email\",\"phone\":\"Phone\",\"identify_type\":\"ID Type\",\"identify_num\":\"RUT ID\"}",
  "create_time": 1695317066,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 5.2 Collection Callback – Response Instructions
If the callback is successfully processed, return <span style="color:red;">success</span>. The system will not push this order again; otherwise, it will retry multiple times.

# 6. Payout Order Creation Interface
(Order creation IP needs to be whitelisted by us)
Order URL: https://{api_domain}/api/v1/payApi/CreatePayOutOrder

## 6.1 Payout – Request Parameters
| Name      | Type  | Required | Description                                                                         |
|----------------|--------|----------|-------------------------------------------------------------------------------------|
| trade_no    | int  | true     | Merchant ID.                                                                        |
| order_no    | string | true     | Merchant order number.                                                              |
| app_id     | int  | true     | Merchant appId.                                                                     |
| pay_code    | int  | true     | Product code, obtained from our operations.                                         |
| price     | int  | true     | Order amount, unit: Centavos, integer                                                |
| account_no   | string | true     | Receiving account number.                                                           |
| account_type  | string | true     | Account type: CHECKING, SAVINGS, RUT, VISTA                                                        |
| account_name  | string | true     | Name.                                                                               |
| bank_code   | string | true     | Bank code, see end of document                                                      |
| identify_type | string | true     | ID type: `RUT` (Tax ID), `PP` (Passport)                                            |
| identify_num  | string | true     | ID number                                                                           |
| pay_notice_url | string | false    | Payout success notification URL                                                     |
| attach     | string | true     | Additional parameters: {"email":"Email","phone":"Phone","bank_name":"Bank Name"} |
| sign      | string | true     | Signature result, see the top of the document for the signature method.             |                            

- **Payout – attach Additional Parameter Field Description**

```json
{"email":"Email","phone":"Phone","bank_name":"Bank Name"}
```

| Name        | Type   | Required | Description      |
|-------------|--------|----------|------------------|
| phone       | string | false    | Recipient phone number |
| email       | string | false    | Email address    |
| bank_name   | string | true     | Bank name        |


- Pay-out - Request Parameter Example

```json
{
  "trade_no": 165,
  "order_no": "p71584121t1693047656571",
  "app_id": 51,
  "pay_code": 91145,
  "price": 10000,
  "account_type": "CHECKING",
  "account_no": "1234567890",
  "account_name": "Juan Perez",
  "bank_code": "Chile",
  "identify_type": "RUT",
  "identify_num": "123456785",
  "pay_notice_url": "http://www.test.com",
  "attach": "",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
## 6.2 Payout – Order Response
| Name     | Type  | Required | Description                                  |
|--------------|--------|----------|-------------------------------------------------------------------------------|
| code     | int  | true   | 200: Success; Others: Failure.                        |
| msg     | string | true   | Failure reason.                                |
| dis_order_no | string | true   | Platform order number.                            |
| order_no   | string | true   | Merchant order number.                            |
| status    | int  | true   | Order status: 2. Payout success, 3. Payout failure, 7. Rejected, 8. Reversal, others: Processing. |
| create_time | int  | true   | Creation time.                                |
| sign     | string | true   | Signature result, see the top of the document for the signature method.    |

- Pay-out - Order Response Example

Failure:

```json
{
  "code": 10009,
  "msg": "Order creation failed"
}
```

Success:

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
# 7. Payout Callback Notification

Push address: The `pay_notice_url` provided by the merchant during order placement. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.
## 7.1 Pay-out Callback Request Parameters

| Name         | Type   | Required | Description                                                         |
| ------------ | ------ | -------- | ------------------------------------------------------------------- |
| trade_no     | int    | true     | Merchant ID                                                         |
| order_no     | string | true     | Merchant order number                                               |
| dis_order_no | string | true     | Platform order number                                               |
| order_price  | int    | true     | Order amount, unit: Centavos                                        |
| fee          | int    | false    | Order fee, unit: Centavos                                           |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status       | int    | true     | Order status: <span style="color:red;">2. Payout success</span>, 3. Payout failure, 7. Rejected, 9. Reversal |
| pay_info     | string | false    | Payment information                                                 |
| remark       | string | false    | Failure reason                                                      |
| create_time  | int    | true     | Creation time                                                       |
| sign         | string | true     | Signature result, see the top of the document for the signature method |
| nti_time     | int    | true     | Notification initiation time                                        |

- Pay-out Callback Request Example

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
## 7.2 Pay-out Callback Response Description
If the callback is successfully received and processed, please return <span style="color:red;">`success`</span>. The system will stop pushing this order information; otherwise, it will be resent multiple times.

# 8. Query Order Interface (Common for Pay-in and Pay-out)
(Request IP needs to be whitelisted by us)
Query URL: https://{api_domain}/api/v1/payApi/QueryOrder

## 8.1 Query Request Parameters
| Name       | Type   | Required | Description                                                        |
| ---------- | ------ | -------- | ------------------------------------------------------------------ |
| order_type | string | true     | pay_out: Payout, pay_in: Collection                                |
| trade_no   | int    | true     | Merchant ID                                                        |
| order_no   | string | true     | Merchant order number                                              |
| sign       | string | true     | Signature result, see the top of the document for the signature method |

- Query Request Example

```json
{
  "order_type": "pay_in",
  "trade_no": 165,
  "order_no": "p71584121t1693047656571",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
## 8.2 Query Response

| Name     | Type  | Required | Description                                                                                                 |
|--------------|--------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code     | int  | true   | 200: Query successful; Others: Failure.                                                                                   |
| msg     | string | true   | Query failure reason.                                                                                            |
| trade_no   | int  | true   | Merchant ID.                                                                                                 |
| <span style="color:red;">real_price</span>  | int  | true   | <span style="color:red;">Actual amount paid, unit: Centavos.</span>                                                                                       |
| status    | int  | true   |  Order status: `1` unpaid, <span style="color:red;">`2` success</span>, `3` failed, `7` rejected, `9` reversed, `10` processing                                                       |
| success_time | int  | true   | Success timestamp.                                                                                              |
| order_no   | string | true   | Merchant order number.                                                                                            |
| dis_order_no | string | true   | Platform order number.                                                                                            |
| remark    | string | true   | Reason for pay-out failure.                                                                                         |
| fee     | int  | false  | Order fee, unit: Centavos.                                                                                           |
| create_time | int  | true   | Creation time.                                                                                                |
| payer    | string | false  | JSON string, payer info: {"account_name":"Name","account_type":"Account Type: CHECKING, SAVINGS, RUT, VISTA","account_no":"Account","identify_type":"ID Type: RUT, PP","identify_num":"ID Number","bank_code":"Bank Code"}.       |
| sign     | string | true   | Signature result, see the top of the document for the signature method.                                                                   |
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
 "payer": "{\"account_name\":\"Juan Perez\",\"account_type\":\"CHECKING\",\"account_no\":\"1234567890\",\"identify_type\":\"RUT\",\"identify_num\":\"123456785\",\"bank_code\":\"Chile\"}",
 "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```


# 9. Pay-out Balance Query Interface
The Request IP needs to be whitelisted by contacting us
Address: https://{api_domain}/api/v1/payApi/QueryBalance

## 9.1 Balance Request Parameters

| Name   | Type  | Required | Description                               |
|----------|--------|----------|-------------------------------------------------------------------------|
| trade_no | int  | true   | Merchant ID.                              |
| app_id  | int  | true   | Merchant appid.                             |
| sign   | string | true   | Signature result, see the top of the document for the signature method. |

- Balance Request Example

```json
{
  "trade_no": 165,
  "app_id": 28,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
## 9.2 Balance Response

| Name      | Type  | Required | Description                               |
|----------------|--------|----------|-------------------------------------------------------------------------|
| code      | int  | true   | 200: Query successful; Others: Failure.                 |
| msg      | string | true   | Failure reason.                             |
| balance    | int  | true   | Balance, unit: Centavos.                          |
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
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
# 10. Error Codes
| Status Code | Description                                                              |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------|
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

# 11. Pay-in Payment Methods

| Field Name | Value | Description |
|-----------|-------|------|
| pay_method | WALLET | WALLET |
| pay_method | CASH | CASH |
| pay_method | TC | TC |
| pay_method | TD | TD |
| pay_method | NET_BANKING_CLP | NET BANKING |

# 12. Bank Codes
| Field Name | Code       | Bank Name |
| :--- | :--- | :--- |
| bank_code | BBVA_CLP | Banco BBVA |
| bank_code | Bice | Banco Bice |
| bank_code | BBPC | Banco BTG Pactual Chile |
| bank_code | Consorcio | Banco Consorcio |
| bank_code | Chile | Banco de Chile |
| bank_code | Estado | Banco Estado |
| bank_code | Falabella_CLP | Banco Falabella |
| bank_code | Itau_CLP | Banco Itau |
| bank_code | Paris | Banco Paris |
| bank_code | Penta | Banco Penta |
| bank_code | Ripley_CLP | Banco Ripley |
| bank_code | Security | Banco Security |
| bank_code | BCI | BCI |
| bank_code | CajaLH | Caja Los Heroes |
| bank_code | Coopeuch | Coopeuch |
| bank_code | HSBCBC | HSBC Bank Chile |
| bank_code | Los_Andes | Los Andes |
| bank_code | Mercado_Pago | Mercado Pago |
| bank_code | Rabobank_Chile | Rabobank Chile |
| bank_code | Santander_CLP | Santander |
| bank_code | Scotiabank_CLP | Scotiabank |
| bank_code | Tenpo | Tenpo |
| bank_code | 100%Banco | 100% Banco |
| bank_code | Bancamiga | Bancamiga |
| bank_code | Activo | Banco Activo |
| bank_code | Bicentenario | Banco Bicentenario |
| bank_code | Caroní | Banco Caroní |
| bank_code | Venezuela | Banco de Venezuela |
| bank_code | del_Caribe | Banco del Caribe |
| bank_code | del_Tesoro | Banco del Tesoro |
| bank_code | Exterior | Banco Exterior |
| bank_code | Fondo_Común | Banco Fondo Común |
| bank_code | Mercantil | Banco Mercantil |
| bank_code | Nacional_de_Crédito | Banco Nacional de Crédito |
| bank_code | Plaza | Banco Plaza |
| bank_code | Provincial | Banco Provincial |
| bank_code | Venezolano_de_Crédito | Banco Venezolano de Crédito |
| bank_code | Banesco | Banesco |
| bank_code | Banplus | Banplus |
| bank_code | CLP_BANK | CLP_BANK |
