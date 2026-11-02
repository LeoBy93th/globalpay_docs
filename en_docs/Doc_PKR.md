
---

# 1. Integration Process

> 1. Business negotiation and account opening, confirm relevant rates.
>
> 2. Contact operations to create the merchant number, secret key, merchant appId, product code, and apiUrl.
>
> 3. After development is completed, both parties perform joint debugging and testing to verify that requests, reporting, and other information are complete.

---

# 2. MD5 Signature Algorithm

> 1. Sort all parameters in ascending order by key in the format of key-value pairs (key1=value1). (Parameters with empty values are not included in the signature.)
>
> 2. Combine them in the format: key1=value1&key2=value2
>
> 3. Append the merchant secret key: key1=value1&key2=value2...&key=merchant_secret_key
>
> 4. sign = md5(the string assembled in the previous step). The signature result is a 32-character lowercase string.
>
> 5. The signature key can be viewed in Merchant Backend -> Basic Information, or you may contact our customer service.

---

# 3. Notes

## 3.1 API Related

> 1. All APIs in this document use the HTTP standard communication protocol. Requests are submitted via POST. Both request and response Content-Type are application/json, and the encoding is UTF-8.
>
> 2. The amount unit is <span style="color:red;"> centavos </span>.
>
> 3. The requesting IP must be whitelisted.
>
> 4. user_ip should collect the real user IP as much as possible. If it is not available, leave it empty. Do not use local IPs such as 127.0.0.1.

## 3.2 Callback Related

> 1. If the callback is received and processed successfully, please return the text <span style="color:red;">**success**</span> without any other characters. The system will stop pushing this order information. Otherwise, it will push repeatedly multiple times.
>
> 2. During asynchronous notification interaction, if the response received is not **success**, it will be considered a notification failure. The system will resend notifications periodically based on a strategy. The retry intervals are: 1m, 1m, 4m, 10m, 10m, 1h, 2h, 6h, 15h.
>
> 3. If pay_notice_url is empty, it will be considered that the merchant does not need callbacks, and the system will not push notifications.

---

# 4. Pay-in Order API

(Order IP must be whitelisted by contacting us)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayInOrder`


## 4.1 Pay-in Order Request Parameters
| Name           | Type   | Required | Description                                                                                                    |
| -------------- | ------ |----------|----------------------------------------------------------------------------------------------------------------|
| trade_no       | int    | true     | Merchant number                                                                                                |
| app_id         | int    | true     | erchant appId                                                                                                  |
| pay_code       | int    | true     | Product code, obtain from our operations team                                                                  |
| pay_method     | string    | true     | Payment method, refer to the payment method dictionary                                                         |
| price          | int    | true     | Order amount, unit: centavos, integer. 1 peso = 100 centavos                                                   |
| order_no       | string | true     | Merchant order number                                                                                          |
| success_url    | string | false    | Redirect URL after successful payment                                                                          |
| fail_url       | string | false    | Redirect URL after failed payment                                                                              |
| pay_notice_url | string | false    | Payment success notification URL                                                                               |
| user_id        | string | true     | System user ID                                                                                                 |
| user_ip        | string | false    | Payer IP                                                                                                       |
| attach         | string | true     | Additional parameter JSON string |
| sign           | string | true     | Signature result, signature method described at the top of the document                                                                                                 |
|timestamp|string| false    | Order timestamp, 10-digit timestamp in seconds                                                                                                   |



- Pay-in attach additional parameter field description

| Name           | Type   | Required | Description                                                  |
|---------------|--------|----------|--------------------------------------------------------------|
| name          | string | true     | Payer name                                                   |
| phone         | string | true     | Phone number, 11 digits starting with 03 or 10 digits starting with 3 |
| email         | string | false    | Email                                                        |
| identify_type | string | false    | CNIC                                                         |
| identify_num  | string | false    | Identification number                                        |

### Pay-in Order Request Example

```json
{
  "trade_no": 165,
  "order_no": "p71584121t1693047656571",
  "app_id": 51,
  "pay_code": 91145,
  "price": 10000,
  "success_url": "",
  "fail_url": "",
  "pay_notice_url": "http://www.test.com",
  "pay_remind_url": "",
  "attach": "{\"phone\":\"123456789\"} ",
  "user_ip": "87.200.59.188",
  "user_id": "2677323",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 4.2 Pay-in Order Response

| Name         | Type   | Required | Description                        |
| ------------ | ------ | ---- | --------------------------- |
| code         | int    | true | 200: order created successfully, others: failed    |
| msg          | string | true | Failure reason                     |
| pay_url      | string | true | Payment link                      |
| qr_code      | string | true | PIX QR code string                           |
| order_no     | string | true | Merchant order number                  |
| dis_order_no | string | true | Platform order number                  |
| create_time  | int    | true | Create time                    |
| pay_info  | string    | false | Payment info JSON string, e.g. {"pay_raw":"Native payment info, merchant can convert to QR code"} |
| sign         | string | true | Signature result, signature method described at the top of the document |

### Pay-in Order Response Example
Failure:

```json
{
  "code": 10009,
  "msg": "IP is not in the merchant's IP whitelist, ip:158.28.87.267",
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

# 5. Pay-in Callback Notification (POST/JSON) 

Push URL: the pay_notice_url provided by the merchant when placing the order
Callback IP: `call_back_server_ip`, please add our IP into the callback whitelist

## 5.1 Pay-in Callback Request Parameters

| Name         | Type   | Required | Description                                                                                                                                                                                                                                                                                                                                               |
| ------------ | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| trade_no     | int    | true     | Merchant number                                                                                                                                                                                                                                                                                                                                           |
| status       | int    | true     | Order status: <span style="color:red;">2 = success</span>, 3 = failure                                                                                                                                                                                                                                                                                                                    |
| order_no     | string | true     | Merchant order number                                                                                                                                                                                                                                                                                                                                     |
| dis_order_no | string | true     | Platform order number                                                                                                                                                                                                                                                                                                                                     |
| order_price  | int    | true     | Order amount, unit: centavos                                                                                                                                                                                                                                                                                                                              |
| <span style="color:red;">real_price</span>   | int    | true     | <span style="color:red;">Actual payment amount paid by user, unit: centavos</span>                                                                                                                                                                                                                                                                                                        |
| nti_time     | int    | false    | Notification initiation time                                                                                                                                                                                                                                                                                                                              |
| payer        | string | false    | JSON string of payer information {"name":"Name","account":"Account","bank":"User bank code","utr2":"Bank transaction number","email":"Email","phone":"Phone number","identify_type":"ID type","identify_num":"CPF,CNPJ"}. In addition to the example fields, this parameter will integrate payer-related fields from the attach provided by the merchant. |
| pay_info     | string | false    | Payment info JSON string, e.g. collection/payment raw info, card number, name, bank, etc.                                                                                                                                                                                                                                                                 |
| create_time  | int    | true     | Create time                                                                                                                                                                                                                                                                                                                                               |
| sign         | string | true     | Signature result, signature method described at the top of the document                                        |

### Pay-in Callback Request Example

```json
{
  "trade_no": 165,
  "status": 2,
  "order_no": "p71584121t1693047656571",
  "dis_order_no": "lufei169246816001692",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1693057443,
  "payer": "{\"name\":\"Name\",\"email\":\"Email\",\"phone\":\"Phone\"}",
  "create_time": 1695317066,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 5.2 Pay-in Callback Response Description

If the callback is received and processed successfully, please return <span style="color:red;">**success**</span>. The system will stop pushing this order information, otherwise it will push repeatedly multiple times.


# 6. Pay-out Order API

(Order IP must be whitelisted by contacting us)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayOutOrder`

## 6.1 Pay-out Request Parameters

<span style="color:red;">A pay-out request is considered successful unless an explicit failure is returned, even if it times out. Do not submit the request again unless an explicit failure is returned.</span>

| Name           | Type   | Required | Description                                                                           |
| -------------- | ------ | -------- | ------------------------------------------------------------------------------------- |
| trade_no       | int    | true     | Merchant number                                                                       |
| order_no       | string | true     | Merchant order number                                                                 |
| app_id         | int    | true     | Merchant appId                                                                        |
| pay_code       | int    | true     | Product code, obtain from our operations team                                         |
| price          | int    | true     | Order amount, unit: centavos, integer. 1 peso = 100 centavos                          |
| account_no     | string | true     | Beneficiary account number                                                            |
| account_type   | string | true     | Account type: BANK,JAZZCASH,EASYPAISA,WALLET |
| account_name   | string | true     | Name                                                                                  |
| bank_code      | string | true     | Bank code                                                                             |
| pay_notice_url | string | false    | Pay-out success notification URL                                                      |
| attach         | string | false    | Additional parameter {"email":"Email","phone":"Phone number","bank_name":"Bank name"} |
| user_ip        | string | true     | Beneficiary user IP                                                                   |
| sign           | string | true     | Signature result, signature method described at the top of the document               |
| timestamp      | string | false    | Order timestamp, 10-digit timestamp in seconds                                        |

### Pay-out Request Example

```json
{
  "trade_no": 165,
  "order_no": "p71584121t1693047656571",
  "app_id": 51,
  "pay_code": 91145,
  "price": 10000,
  "account_type": "BANK",
  "account_no": "012345678910111213",
  "account_name": "TEST",
  "identify_num":"1234567890123",
  "bank_code": "BKKB",
  "pay_notice_url": "http://www.test.com",
  "attach": "",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 6.2 Pay-out Order Response

| Name         | Type   | Required | Description                                                                         |
| ------------ | ------ | -------- | ----------------------------------------------------------------------------------- |
| code         | int    | true     | 200: order created successfully, others: failed                                     |
| msg          | string | true     | Failure reason                                                                      |
| dis_order_no | string | true     | Platform order number                                                               |
| order_no     | string | true     | Merchant order number                                                               |
| status       | int    | true     | Order status: 2 = success, 3 = failure, 7 = rejected, 9 = reversed, 10 = processing |
| create_time  | int    | true     | Create time                                                                         |
| sign         | string | true     | Signature result, signature method described at the top of the document             |


### Pay-out Order Response Example

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

Push URL: the pay_notice_url provided by the merchant when placing the order
Callback IP: `call_back_server_ip`, please add our IP into the callback whitelist

## 7.1 Pay-out Callback Request Parameters
| Name         | Type   | Required | Description                                                             |
| ------------ | ------ | -------- | ----------------------------------------------------------------------- |
| trade_no     | int    | true     | Merchant number                                                         |
| order_no     | string | true     | Merchant order number                                                   |
| dis_order_no | string | true     | Platform order number                                                   |
| order_price  | int    | true     | Order amount, unit: centavos                                            |
| fee          | int    | false    | Transaction fee, unit: centavos                                         |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status       | int    | true     | Order status: <span style="color:red;">2 = success</span>, 3 = failure, 7 = rejected, 9 = reversed      |
| pay_info     | string | false    | Payment info                                                            |
| remark       | string | false    | Failure reason                                                          |
| create_time  | int    | true     | Create time                                                             |
| sign         | string | true     | Signature result, signature method described at the top of the document |
| nti_time     | int    | true     | Notification initiation time                                            |

### Pay-out Callback Request Example

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

If the callback is received and processed successfully, please return <span style="color:red;">**success**</span>. The system will stop pushing this order information, otherwise it will push repeatedly multiple times.

---

# 8. Order Query API (Shared by Pay-in and Pay-out)

(Request IP must be whitelisted by contacting us)
Query URL: `https://{api_domain}/api/v1/payApi/QueryOrder`

## 8.1 Query Request Parameters

| Name         | Type   | Required | Description                                                             |
| ------------ | ------ | -------- | ----------------------------------------------------------------------- |
| order_type   | string | true     | pay_out: pay-out, pay_in: pay-in                                        |
| trade_no     | int    | true     | Merchant number                                                         |
| app_id       | int    | true     | Merchant appId                                                          |
| dis_order_no | string | false   | Platform order number.                         |
| order_no | string | false  | Merchant  order number.              |
| sign         | string | true     | Signature result, signature method described at the top of the document |

### Query Request Example

```json
{
  "order_type": "pay_in",
  "trade_no": 165,
  "order_no": "p71584121t1693047656571",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

## 8.2 Query Response

| Name         | Type   | Required | Description                                                                                                                                                                    |
| ------------ | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| code         | int    | true     | 200: query success, others: failure                                                                                                                                            |
| msg          | string | true     | Query failure reason                                                                                                                                                           |
| trade_no     | int    | true     | Merchant number                                                                                                                                                                |
| <span style="color:red;">real_price</span>   | int    | true     | <span style="color:red;">Actual payment amount, unit: centavos</span>                                                                                                                                          |
| status       | int    | true     | Order status: 1 = unpaid, <span style="color:red;">2 = success</span>, 3 = failure, 7 = rejected, 9 = reversed, 10 = processing                                                                                |
| success_time | int    | true     | Success timestamp                                                                                                                                                              |
| order_no     | string | true     | Merchant order number                                                                                                                                                          |
| dis_order_no | string | true     | Platform order number                                                                                                                                                          |
| remark       | string | true     | Pay-out failure reason                                                                                                                                                         |
| fee          | int    | false    | Transaction fee, unit: centavos                                                                                                                                                |
| create_time  | int    | true     | Create time                                                                                                                                                                    |
| payer        | string | false    | JSON string, payer info {"account_name":"Name","account_type":"Account type: BANK,JAZZCASH,EASYPAISA,WALLET","account_no":"Account","bank_code":"Bank code","identify_num":"ID number"} |
| pay_info     | string | false    | Payment info JSON string, e.g. collection/payment raw info, card number, name, bank, etc. 25-10-28                                                                             |
| sign         | string | true     | Signature result, signature method described at the top of the document                                                                                                        |
| utr2         | string | false    | Bank order number                                                                                                                                                              |

### Query Response Example

Failure:

```json
{
  "code": 10009,
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
  "payer": "{\"account_name\":\"TEST\",\"account_type\":\"BANK\",\"account_no\":\"012345678910111213\",\"bank_code\":\"BKKB\",\"identify_num\":\"1234567890123\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Pay-out Balance Query API

(Request IP must be whitelisted by contacting us)
URL: `https://{api_domain}/api/v1/payApi/QueryBalance`

## 9.1 Balance Request Parameters

| Name     | Type   | Required | Description                                                             |
| -------- | ------ | -------- | ----------------------------------------------------------------------- |
| trade_no | int    | true     | Merchant number                                                         |
| app_id   | int    | true     | Merchant appid                                                          |
| sign     | string | true     | Signature result, signature method described at the top of the document |

### Balance Request Example

```json
{
  "trade_no": 165,
  "app_id": 28,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```
## 9.2 Balance Response

| Name           | Type   | Required | Description                                                             |
| -------------- | ------ | -------- | ----------------------------------------------------------------------- |
| code           | int    | true     | 200: query success, others: failure                                     |
| msg            | string | true     | Failure reason                                                          |
| balance        | int    | true     | Balance, unit: centavos                                                 |
| balance_frozen | int    | false    | Frozen balance, unit: centavos                                          |
| sign           | string | true     | Signature result, signature method described at the top of the document |

### Balance Response Example

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

# 10、Payment Methods - Collection Field pay_method
| Field      | Country | Value     | Description |
|------------|---------|-----------|-------------|
| pay_method | Pakistan     | easypaisa | Easypaisa   |
| pay_method | Pakistan    | jazzcash  | Jazzcash    |
| pay_method | Pakistan   | QR-PKR    | QR          |

# 11、Bank Codes bank_code

| Field | Country | Value | Description |
|-----------|--------|-------------|------------------------------|
| bank_code | Pakistan | PKRABL | Allied Bank Limited |
| bank_code | Pakistan | PKRADVANS | Advans Pakistan Microfinance Bank Limited |
| bank_code | Pakistan | PKRAIBPL | Albaraka Islamic Bank Pakistan Limited |
| bank_code | Pakistan | PKRALFALAH | Bank Alfalah Limited |
| bank_code | Pakistan | PKRAPNA MFB | Apna Microfinance Bank Limited |
| bank_code | Pakistan | PKRASKARI | Askari Bank Limited |
| bank_code | Pakistan | PKRBAHL | Bank Al Habib Limited |
| bank_code | Pakistan | PKRBIPL | Bank Islami Pakistan Limited |
| bank_code | Pakistan | PKRBOK | The Bank of Khyber |
| bank_code | Pakistan | PKRBOP | The Bank of Punjab |
| bank_code | Pakistan | PKRCITI | Citibank N.A. Pakistan |
| bank_code | Pakistan | PKRDIBPL | Dubai Islamic Bank Pakistan Limited |
| bank_code | Pakistan | PKREASYPAISA | Easypaisa |
| bank_code | Pakistan | PKRFAYSAL | Faysal Bank Limited |
| bank_code | Pakistan | PKRFINCA | FINCA Microfinance Bank Limited |
| bank_code | Pakistan | PKRFINJA | Finja Private Limited |
| bank_code | Pakistan | PKRFWBL | First Women Bank Limited |
| bank_code | Pakistan | PKRHBL | Habib Bank Limited |
| bank_code | Pakistan | PKRHBL MFB | HBL Microfinance Bank Limited |
| bank_code | Pakistan | PKRHMBL | Habib Metropolitan Bank Limited |
| bank_code | Pakistan | PKRICBC | Industrial and Commercial Bank of China Limited Pakistan |
| bank_code | Pakistan | PKRJAZZCASH | JazzCash |
| bank_code | Pakistan | PKRJSBL | JS Bank Limited |
| bank_code | Pakistan | PKRKHUSHHALI | Khushhali Microfinance Bank Limited |
| bank_code | Pakistan | PKRKONNECT | Konnect by Bank Alfalah |
| bank_code | Pakistan | PKRMCB | MCB Bank Limited |
| bank_code | Pakistan | PKRMCBAH | MCB Arif Habib Limited |
| bank_code | Pakistan | PKRMCB ISLAMIC | MCB Islamic Bank Limited |
| bank_code | Pakistan | PKRMEEZAN | Meezan Bank Limited |
| bank_code | Pakistan | PKRNAYAPAY | NayaPay |
| bank_code | Pakistan | PKRNBP | National Bank of Pakistan |
| bank_code | Pakistan | PKRNBP FUNDS | NBP Funds Management Limited |
| bank_code | Pakistan | PKRNRSP | NRSP Microfinance Bank Limited |
| bank_code | Pakistan | PKRSADAPAY | SadaPay |
| bank_code | Pakistan | PKRSAMBA | Samba Bank Limited |
| bank_code | Pakistan | PKRSCB | Standard Chartered Bank Pakistan Limited |
| bank_code | Pakistan | PKRSILK | Silk Bank Limited |
| bank_code | Pakistan | PKRSINDH | Sindh Bank Limited |
| bank_code | Pakistan | PKRSONERI | Soneri Bank Limited |
| bank_code | Pakistan | PKRSUMMIT | Summit Bank Limited |
| bank_code | Pakistan | PKRTMB | Telenor Microfinance Bank Limited |
| bank_code | Pakistan | PKRUBL | United Bank Limited |
| bank_code | Pakistan | PKRUMFB | U Microfinance Bank Limited |
| bank_code | Pakistan | PKRUPAISA | Upaisa |


# 12. Error Codes

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

# 13. Pay-in Checkout Interface

Address: https://{api_domain}/api/v1/cashApi/CashIn.html
Request Method: GET

### Parameters:

| Name    | Type  | Required | Description                        |
|------------|--------|----------|------------------------------------|
| app_id   | string | true   | Merchant app_id.                   |
| order_no  | string | true   | Merchant order number.             |
| amount   | string | true   | Merchant Amount (Unit: Lira)       |
| notice_url | string | false  | Asynchronous notification address. |
|pay_code|int|true|Product Code|

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{MerchantOrderNumber}}&amount={{MerchantAmount}}&notice_url={{AsynchronousNotificationAddress}}&pay_code={{ProductCode}}
```

# 14. Document Update Time
```
2026-08-24 13:05:52
```
