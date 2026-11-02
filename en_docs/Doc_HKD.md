# 1. Integration Process

> 1. Complete business onboarding and confirm the relevant rates.
>
> 2. Contact operations to create the merchant number, secret key, merchant appId, product code, and apiUrl.
>
> 3. After development is completed, both parties will perform integration testing to verify that requests, reporting, and other information are complete.

# 2. MD5 Signature Algorithm

> 1. Sort all parameters in ascending order by key-value pair (`key1=value1`). Parameters with empty values do not participate in the signature.
>
> 2. Concatenate them in the format `key1=value1&key2=value2`.
>
> 3. Append the merchant secret key: `key1=value1&key2=value2...&key=merchant_secret_key`.
>
> 4. `sign=md5(the string assembled in the previous step)`. The signature result is a 32-character lowercase string.
>
> 5. For the signing key, please check Merchant Backend -> Basic Information, or contact our customer service.

# 3. Notes

## 3.1 API Related
> 1. All APIs in this document use the standard HTTP protocol with POST requests. Both request and response `Content-type` are `application/json`, and the character encoding is unified as `UTF-8`.
>
> 2. The amount unit is <span style="color:red;">Hong Kong Cents (Cents)</span>.
>
> 3. The requesting IP must be whitelisted.
>
> 4. `user_ip` should collect the user's real IP whenever possible. If unavailable, leave it empty. Do not use a local IP such as `127.0.0.1`.

## 3.2 Callback Related
> 1. After successfully processing the callback, please return the text <span style="color:red;">`success`</span> without any additional characters. Otherwise, the system will continue to resend the order notification multiple times.
>
> 2. During asynchronous notification interaction, if the received response is not `success`, the notification will be considered failed, and retries will be sent periodically according to the following schedule: `1m`, `1m`, `4m`, `10m`, `10m`, `1h`, `2h`, `6h`, `15h`.
>
> 3. If `pay_notice_url` is empty, it will be considered that the merchant does not require a callback, and the system will not push notifications.

# 4. Pay-in Order API

(The order IP must be whitelisted by contacting us.)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayInOrder`

## 4.1 Pay-in Order Request Parameters

| Name             | Type   | Required  | Description                                                                                                      |
|----------------|------|-----|---------------------------------------------------------------------------------------------------------|
| trade_no       | int    | true  | Merchant number                                                                                                     |
| app_id         | int    | true  | Merchant appId                                                                                                |
| pay_code       | int    | true  | Product code, contact our operations team to obtain                                                                                           |
| pay_method     | string    | true  | Payment method, see the payment methods below                                                                                            |
| price          | int    | true  | Order amount, unit: Hong Kong Cents (Cents), integer. 1 Hong Kong Dollar (HKD) = 100 Hong Kong Cents                                                                 |
| order_no       | string | true  | Merchant order number                                                                                                   |
| success_url    | string | false | Redirect URL after successful payment                                                                                              |
| fail_url       | string | false | Redirect URL after failed payment                                                                                              |
| pay_notice_url | string | false | Payment success notification URL                                                                                              |
| user_id        | string | false | Merchant user ID                                                                                                  |
| user_ip        | string | false | Payer IP                                                                                                  |
|attach|string|false| Additional parameter JSON string for payer information |
| sign           | string | true  | Signature result, see the signature method at the top of this document                                                                                          |
|timestamp|string|false| Order timestamp, 10-digit timestamp in seconds                                                                                         |

- Pay-in `attach` Additional Parameter Field Description

| Name            | Type     | Required    | Description                                                                      |
|---------------|--------|-------|-------------------------------------------------------------------------|
| phone         | string    | false | Phone number (please provide the real phone number whenever possible; use a placeholder if unavailable)                                                     |
| email         | string    | false | Email address (please provide the real email address whenever possible; use a placeholder if unavailable)                                                       |
| product_name  | string    | false | Product name                                                                    |


- Pay-in Order Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"phone\":\"91234567\",\"email\":\"john.doe@example.com\",\"product_name\":\"Test Product\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "203.198.1.1",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}
```

## 4.2 Pay-in Order Response

| Name         | Type   | Required | Description                                                                                                                                                                                              |
|------------|------|----|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code         | int    | true | `200`: Order created successfully; others: order creation failed                                                                                                                                                                                |
| msg          | string | true | Failure reason                                                                                                                                                                                            |
| pay_url      | string | false | Payment link                                                                                                                                                                                            |
| qr_code      | string | false | QR code string                                                                                                                                                                                          |
| order_no     | string | true | Merchant order number                                                                                                                                                                                           |
| dis_order_no | string | true | Platform order number                                                                                                                                                                                           |
| create_time  | int    | true | Creation time                                                                                                                                                                                            |
| pay_info  | string    | false | Payment information JSON string, for example: collection/payment raw information, card number, name, bank, etc. `{"pay_raw":"raw payment info","redirect_url":"payment redirect link"}` |
| sign         | string | true | Signature result, see the signature method at the top of this document                                                                                                                                                                                  |

- Pay-in Order Response Example

Failed:

```json
{
  "code": 1005,
  "msg": "Merchant not found",
  "sign": ""
}
```

Successful:

```json
{
  "code": 200,
  "msg": "",
  "sign": "b449b4b6907204a683ec6c50bff92b01",
  "order_no": "p7158412025J2dZjXLmz0",
  "dis_order_no": "2025071130770572062498816hkg1oushe",
  "create_time": 1752825512,
  "pay_url": "https://{api_domain}/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5. Pay-in Callback Notification `post/json`

Push URL: the `pay_notice_url` submitted by the merchant when creating the order. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 5.1 Pay-in Callback Request Parameters

| Name         | Type   | Required  | Description                                                                                                                                                                                       |
|------------|------|-----|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| trade_no     | int    | true  | Merchant number                                                                                                                                                                                      |
| status       | int    | true  | Order status, <span style="color:red;">`2`: success</span>, `3`: failure                                                                                                                                                                         |
| order_no     | string | true  | Merchant order number                                                                                                                                                                                    |
| dis_order_no | string | true  | Platform order number                                                                                                                                                                                    |
| order_price  | int    | true  | Order amount, unit: Hong Kong Cents (Cents)                                                                                                                                                                        |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">Actual amount paid by the user, unit: Hong Kong Cents (Cents)</span>                                                                                                                                                                   |
| nti_time     | int    | false | Notification initiation time                                                                                                                                                                                   |
| payer        | string | false | JSON string containing payer information `{"name":"name","account":"account","bank":"payer bank code","utr2":"bank serial number","email":"email","phone":"phone number","identify_type":"ID type","identify_num":"ID number"}`. In addition to the example fields, this parameter currently merges payer-related fields passed by the merchant in `attach`. |
| pay_info  | string    | false | Payment information JSON string, for example: collection/payment raw information, card number, name, bank, etc. |
| create_time  | int    | true  | Creation time                                                                                                                                                                                     |
| sign         | string | true  | Signature result, see the signature method at the top of this document                                                                                                                                                                           |

- Pay-in Callback Request Example

```json
{
  "trade_no": 10238,
  "status": 2,
  "order_no": "RCG0126042435820040804784",
  "dis_order_no": "Meg1352644s0800hkganVMR",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1776680622,
  "payer": "{\"account_no\":\"1234567890\",\"account_type\":\"FPS\",\"identify_num\":\"A123456(7)\",\"identify_type\":\"HKID\",\"req_api_ip\":\"203.198.1.1\",\"utr2\":\"9901108391\"}",
  "create_time": 1776680593,
  "sign": "7b23565a3dc790b6e55f29f0f0cf5f1a"
}
```
## 5.2 Pay-in Callback Response Description
After successfully processing the callback, please return <span style="color:red;">`success`</span>. The system will then stop pushing this order notification; otherwise, it will continue to retry.

# 6. Pay-out Order API

(The order IP must be whitelisted by contacting us.)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayOutOrder`

## 6.1 Pay-out Request Parameters

<span style="color:red;">A pay-out request is considered successful unless an explicit failure is returned, even if it times out. Do not submit the request again unless an explicit failure is returned.</span>

| Name             | Type     | Required    | Description                                                                     |
|----------------|--------|-------|------------------------------------------------------------------------|
| trade_no       | int    | true  | Merchant number                                                                    |
| order_no       | string | true  | Merchant order number                                                                  |
| app_id         | int    | true  | Merchant appId                                                               |
| pay_code       | int    | true  | Product code, contact our operations team to obtain                                                          |
| price          | int    | true  | Order amount, unit: Hong Kong Cents (Cents), integer. 1 Hong Kong Dollar (HKD) = 100 Hong Kong Cents                                 |
| account_no     | string | true  | Beneficiary account                                                                   |
| account_type   | string | false | Account type                                                                   |
| account_name   | string | true  | Name                                                                     |
| bank_code      | string | true  | Bank code                                                                   |
| identify_type  | string | false | ID type                                                                   |
| identify_num   | string | false | ID number                                                                   |
| pay_notice_url | string | false | Pay-out success notification URL                                                             |
| attach         | string | false | Additional parameter JSON string                                                           |
| user_ip        | string | false | Beneficiary IP                                                                |
| sign           | string | true  | Signature result, see the signature method at the top of this document                                                         |
| timestamp      | string | false | Order timestamp, 10-digit timestamp in seconds                                                       |

- Pay-out `attach` Additional Parameter Field Description

| Name       | Type     | Required   | Description             |
|----------|--------|------|----------------|
| fps_id |string| false | FPS identifier |
| bank_name | string | false | Bank name |
| branch_code | string | false | Branch code |


- Pay-out Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"fps_id\":\"john.doe@fps\",\"bank_name\":\"HSBC Hong Kong\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "203.198.1.1",
  "account_type": "FPS",
  "account_no": "1234567890",
  "account_name": "John Doe",
  "bank_code": "HK_HSBC",
  "identify_type": "HKID",
  "identify_num": "A123456(7)"
}
```

## 6.2 Pay-out Order Response

| Name         | Type   | Required | Description                                          |
|------------|------|----|---------------------------------------------|
| code         | int    | true | `200`: Order created successfully; others: order creation failed                            |
| msg          | string | true | Failure reason                                        |
| dis_order_no | string | true | Platform order number                                       |
| order_no     | string | true | Merchant order number                                       |
| status       | int    | true | Order status: `2` pay-out success, `3` pay-out failure, `7` rejected, `9` reversed, `10` processing |
| create_time  | int    | true | Creation time                                        |
| sign         | string | true | Signature result, see the signature method at the top of this document                              |

- Pay-out Order Response Example

Failed:

```json
{
  "code": 1005,
  "msg": "Merchant not found",
  "sign": ""
}
```

Successful:

```json
{
  "code": 200,
  "msg": "",
  "sign": "d3ec1fa0f45bc44218d5fb63bb1beb61",
  "order_no": "p7158412025MsJydJqT7b",
  "dis_order_no": "2025071130776296733810688hkg1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7. Pay-out Callback Notification

Push URL: the `pay_notice_url` submitted by the merchant when creating the order. Callback IP: `call_back_server_ip`. Please add our IP to your callback whitelist.

## 7.1 Pay-out Callback Request Parameters

| Name         | Type   | Required  | Description                                            |
|------------|------|-----|-----------------------------------------------|
| trade_no     | int    | true  | Merchant number                                           |
| order_no     | string | true  | Merchant order number                                         |
| dis_order_no | string | true  | Platform order number                                         |
| order_price        | int    | true  | Order amount, unit: Hong Kong Cents (Cents)                                     |
| fee          | int    | false | Order fee, unit: Hong Kong Cents (Cents)                         |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status       | int    | true  | Order status, <span style="color:red;">`2` pay-out success</span>, `3` pay-out failure, `7` rejected, `9` reversed               |
| pay_info  | string    | false | Payment information JSON string, for example: collection/payment raw information, card number, name, bank, `utr2`, etc. |
| remark       | string | false | Failure reason                                          |
| create_time  | int    | true  | Creation time                                          |
| sign         | string | true  | Signature result, see the signature method at the top of this document                                |
| nti_time     | int    | true | Notification initiation time                                        |

- Pay-out Callback Request Example

```json
{
  "trade_no": 10000,
  "status": 2,
  "order_no": "20060354339090013",
  "dis_order_no": "Meg2352644o2nmjo0800hkgYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": "{\"utr2\":\"611011445289\"}"
}
```
## 7.2 Pay-out Callback Response Description
After successfully processing the callback, please return <span style="color:red;">`success`</span>. The system will then stop pushing this order notification; otherwise, it will continue to retry.

# 8. Order Query API (Shared by Pay-in and Pay-out)

(The requesting IP must be whitelisted by contacting us.)
Query URL: `https://{api_domain}/api/v1/payApi/QueryOrder`

## 8.1 Query Request Parameters

| Name         | Type   | Required | Description                        |
|------------|------|----|---------------------------|
| order_type | string | true | `pay_out`: pay-out, `pay_in`: pay-in    |
| trade_no   | int    | true | Merchant number                      |
| app_id         | int    | true  | Merchant appId                    |
| dis_order_no | string | false | Platform order number                |
| order_no | string | false  | Merchant order number                |
| sign       | string | true | Signature result, see the signature method at the top of this document |

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

| Name         | Type   | Required  | Description                                                                                                                                                |
|------------|------|-----|---------------------------------------------------------------------------------------------------------------------------------------------------|
| code         | int    | true  | `200`: Query successful; others: failed                                                                                                                                    |
| msg          | string | true  | Reason for query failure                                                                                                                                            |
| trade_no     | int    | true  | Merchant number                                                                                                                                               |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">Actual paid amount, unit: Hong Kong Cents (Cents)</span>                                                                                                                              |
| status       | int    | true  | Order status: `1` unpaid, <span style="color:red;">`2` success</span>, `3` failure, `7` rejected, `9` reversed, `10` processing                                                                                                       |
| success_time | int    | true  | Success timestamp                                                                                                                                             |
| order_no     | string | true  | Merchant order number                                                                                                                                             |
| dis_order_no | string | true  | Platform order number                                                                                                                                             |
| remark       | string | true  | Pay-out failure reason                                                                                                                                            |
| fee          | int    | false | Order fee, unit: Hong Kong Cents (Cents)                                                                                                                               |
| create_time  | int    | true  | Creation time                                                                                                                                              |
| payer        | string | false | JSON string containing payer information `{"account_name":"name","account_type":"account type","account_no":"account","identify_type":"ID type","identify_num":"ID number"}` |
| pay_info  | string    | false | Payment information JSON string, for example: collection/payment raw information, card number, name, bank, etc. |
| sign         | string | true  | Signature result, see the signature method at the top of this document                                                                                                                                    |
|utr2|string|false|Bank order number|

- Query Response Example

Failed:

```json
{
  "code": 1017,
  "msg": "Order does not exist"
}
```

Successful:

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
  "payer": "{\"account_name\":\"John Doe\",\"account_type\":\"FPS\",\"account_no\":\"1234567890\",\"identify_type\":\"HKID\",\"identify_num\":\"A123456(7)\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Pay-out Balance Query API

(The requesting IP must be whitelisted by contacting us.)
URL: `https://{api_domain}/api/v1/payApi/QueryBalance`

## 9.1 Balance Request Parameters

| Name     | Type   | Required | Description             |
|--------|------|----|----------------|
| trade_no | int    | true | Merchant number            |
| app_id   | int    | true | Merchant appid       |
| sign     | string | true | Signature result, see the signature method at the top of this document |

- Balance Request Example

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Response

| Name    | Type   | Required | Description                        |
|-------|------|----|---------------------------|
| code    | int    | true | `200`: Query successful; others: failed      |
| msg     | string | true | Failure reason                    |
| balance | int    | true | Balance, unit: Hong Kong Cents (Cents)       |
| balance_frozen | int    | false | Frozen balance, unit: Hong Kong Cents (Cents) |
| sign    | string | true | Signature result, see the signature method at the top of this document |

- Balance Response Example

Failed:

```json
{
  "code": 10001,
  "msg": "Merchant does not exist"
}
```

Successful:

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

(The requesting IP must be whitelisted by contacting us.)
URL: `https://{api_domain}/api/v1/payApi/QueryCertificate`

## 10.1 Payment Voucher Request Parameters

| Name     | Type   | Required | Description             |
|--------|------|----|----------------|
| trade_no | int    | true | Merchant number            |
| app_id   | int    | true | Merchant appid       |
| order_no   | string    | false | Merchant order number, choose one of `order_no` or `dis_order_no`    |
| dis_order_no   | string    | false | Platform order number, choose one of `order_no` or `dis_order_no`    |
| sign     | string | true | Signature result, see the signature method at the top of this document |

- Payment Voucher Request Example

```json
{
  "trade_no": 10003,
  "app_id": 10003,
  "order_no": "",
  "dis_order_no": "35011C02gljuf6k0800hkg1lVY",
  "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```

## 10.2 Payment Voucher Response Parameters

| Name    | Type   | Required | Description                        |
|-------|------|----|---------------------------|
| code    | int    | true | `200`: Response successful; others: failed      |
| msg     | string | true | Failure reason                    |
| img_link | string    | false | Voucher link       |
| img_base | string    | false | Base64 content of the generated voucher                 |
| sign    | string | true | Signature result, see the signature method at the top of this document |

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


| pay_method | Region   | Value             | Description           |
|------------|--------|---------------|----------------|
| pay_method        | Hong Kong   | FPS        | FPS Payment            |
| pay_method        | Hong Kong   | BANK_HK        | Bank Transfer            |


# 12. Bank Codes

| Field Name | Code | Bank Name |
| :--- | :--- | :--- |
| bank_code | BETPAY | BetPay |
| bank_code | ZSK | Faster Payment System (FPS) |
| bank_code | ZDBXG | Standard Chartered Bank (Hong Kong) Limited |
| bank_code | XGSHHF | The Hongkong and Shanghai Banking Corporation Limited |
| bank_code | DFHL | Credit Agricole CIB |
| bank_code | HQB | Citibank |
| bank_code | MGDT | JPMorgan Chase Bank |
| bank_code | GMXMS | NatWest Markets Plc |
| bank_code | ZGJSB | China Construction Bank (Asia) Corporation Limited |
| bank_code | ZGB | Bank of China (Hong Kong) Limited |
| bank_code | DYB | The Bank of East Asia, Limited |
| bank_code | XZB | DBS Bank (Hong Kong) Limited |
| bank_code | ZXB | China CITIC Bank International Limited |
| bank_code | ZSYL | CMB Wing Lung Bank Limited |
| bank_code | HUAQB | OCBC Bank |
| bank_code | HSB | Hang Seng Bank Limited |
| bank_code | SHY | Shanghai Commercial Bank Limited |
| bank_code | JTB | Bank of Communications Co., Ltd. |
| bank_code | DZB | Public Bank (Hong Kong) Limited |
| bank_code | HQYH | OCBC Wing Hang Bank Limited |
| bank_code | DAYB | Tai Yau Bank Limited |
| bank_code | JIYB | Chiyu Banking Corporation Limited |
| bank_code | DXB | Dah Sing Bank Limited |
| bank_code | CXB | Chong Hing Bank Limited |
| bank_code | NYSY | Nanyang Commercial Bank Limited |
| bank_code | UCOB | UCOBANK |
| bank_code | KEBHANABANK | KEBHANABANK |
| bank_code | SLUFJ | MUFG Bank, Ltd. |
| bank_code | PANGB | Bangkok Bank |
| bank_code | YDHY | Indian Overseas Bank |
| bank_code | DYZB | Deutsche Bank |
| bank_code | MGB | Bank of America |
| bank_code | FZBL | BNP Paribas |
| bank_code | YDB | Bank of India |
| bank_code | BJGM | National Bank of Pakistan |
| bank_code | DSB | Tai Sang Bank Limited |
| bank_code | MLB | Malayan Banking Berhad |
| bank_code | SJZY | Sumitomo Mitsui Banking Corporation |
| bank_code | YNGB | Bank Negara Indonesia |
| bank_code | JYB | Finance Bank Limited |
| bank_code | DAHB | United Overseas Bank Limited |
| bank_code | ZGGSB | Industrial and Commercial Bank of China (Asia) Limited |
| bank_code | BARCLAYSBANKPLC | BARCLAYSBANKPLC. |
| bank_code | JNDFY | The Bank of Nova Scotia |
| bank_code | JNDHJ | Royal Bank of Canada |
| bank_code | FGXY | Societe Generale |
| bank_code | YDGB | State Bank of India |
| bank_code | DLDDM | The Toronto-Dominion Bank |
| bank_code | MDKB | Bank of Montreal |
| bank_code | JNDDG | Canadian Imperial Bank of Commerce |
| bank_code | DGSY | Commerzbank |
| bank_code | RSB | UBS |
| bank_code | MGHF | HSBC Bank USA |
| bank_code | RUISB | Mizuho Bank |
| bank_code | DGZY | DZ Bank |
| bank_code | YOULB | Woori Bank |
| bank_code | PHILIPPINENATIONALBANK | PHILIPPINENATIONALBANK |
| bank_code | FUBB | Fubon Bank (Hong Kong) Limited |
| bank_code | SLUFJXT | Mitsubishi UFJ Trust and Banking Corporation |
| bank_code | NYMLB | The Bank of New York Mellon |
| bank_code | INGBANKN | INGBANKN.V. |
| bank_code | XBYDY | BBVA |
| bank_code | ADLYG | National Australia Bank |
| bank_code | XTPY | Westpac Banking Corporation |
| bank_code | AOXINB | Australia and New Zealand Banking Group Limited |
| bank_code | AZLB | Commonwealth Bank of Australia |
| bank_code | YDLLH | Intesa Sanpaolo S.p.A. |
| bank_code | YUXB | UniCredit Bank AG |
| bank_code | RDSY | Svenska Handelsbanken |
| bank_code | QIANYB | The Chiba Bank, Ltd. |
| bank_code | BLSLH | KBC Bank N.V. |
| bank_code | FUGUO | Wells Fargo Bank Hong Kong Branch |
| bank_code | HELAN | Rabobank |
| bank_code | XINZHAN | DBS Bank Hong Kong Branch |
| bank_code | JINW | The Shizuoka Bank, Ltd. |
| bank_code | BASER | The Hachijuni Bank, Ltd. |
| bank_code | HUANSY | Hua Nan Commercial Bank, Ltd. |
| bank_code | ZIHB | The Shiga Bank, Ltd. |
| bank_code | YIWAN | Bank of Taiwan Co., Ltd. |
| bank_code | THECHUGOKUBANKLIMITED | THECHUGOKUBANKLIMITED |
| bank_code | DYSY | First Commercial Bank, Ltd. |
| bank_code | ZHSY | Chang Hwa Commercial Bank, Ltd. |
| bank_code | FGWM | Natixis |
| bank_code | ZGGS | Industrial and Commercial Bank of China Limited |
| bank_code | MGDF | State Street Bank and Trust Company |
| bank_code | ZGJS | China Construction Bank Corporation |
| bank_code | ZGNY | Agricultural Bank of China Limited |
| bank_code | ERSTEGROUPBANKAG | ERSTEGROUPBANKAG |
| bank_code | ZGXT | CTBC Bank Co., Ltd. |
| bank_code | YWZXQY | Taiwan Business Bank, Ltd. |
| bank_code | CREDITSUISSEAG | CREDITSUISSEAG |
| bank_code | GTSHSY | Cathay United Bank Co., Ltd. |
| bank_code | RSYF | EFG Bank AG |
| bank_code | ZSYB | China Merchants Bank Co., Ltd. |
| bank_code | TBFBSY | Taipei Fubon Commercial Bank Co., Ltd. |
| bank_code | YFSYB | Bank SinoPac Co., Ltd. |
| bank_code | ZFGJSY | Mega International Commercial Bank Co., Ltd. |
| bank_code | YSSYB | E.SUN Commercial Bank, Ltd. |
| bank_code | TXGJSY | Taishin International Bank Co., Ltd. |
| bank_code | FLYB | Hong Leong Bank Berhad |
| bank_code | ZDB | Standard Chartered Bank |
| bank_code | HQXGB | Citibank (Hong Kong) Limited |
| bank_code | ICICIBANKLIMITED | ICICIBANKLIMITED |
| bank_code | MELLIBANKPLC | MELLIBANKPLC |
| bank_code | HUAMB | East West Bank |
| bank_code | BALD | Bank of Baroda |
| bank_code | YDGJSY | Far Eastern International Bank Co., Ltd. |
| bank_code | CANARABANK | CANARABANK |
| bank_code | GUOTB | Cathay Bank |
| bank_code | TWTDB | Land Bank of Taiwan Co., Ltd. |
| bank_code | HZJKSY | Taiwan Cooperative Bank, Ltd. |
| bank_code | PUNJABNATIONALBANK | PUNJABNATIONALBANK |
| bank_code | XBYSTD | Banco Santander, S.A. |
| bank_code | UNIONBANKOFINDIA | UNIONBANKOFINDIA |
| bank_code | SHSYCX | Shanghai Commercial & Savings Bank, Ltd. |
| bank_code | INDUSTRIALBANKOFKOREA | INDUSTRIALBANKOFKOREA |
| bank_code | XJPB | Bank of Singapore Limited |
| bank_code | SHINHANBANK | SHINHANBANK |
| bank_code | WDSY | O-Bank Co., Ltd. |
| bank_code | BNPPARIBASSECURITIESSERVICES | BNPPARIBASSECURITIESSERVICES |
| bank_code | GJKFB | China Development Bank |
| bank_code | FIRSTABUDHABIBANKPJSC | FIRSTABUDHABIBANKPJSC |
| bank_code | SAFRASARASINLTD | BANKJ.SAFRASARASINLTD. |
| bank_code | ABNAMROBANKN | ABNAMROBANKN.V. |
| bank_code | HDFCBANKLIMITED | HDFCBANKLIMITED |
| bank_code | UNIONBANCAIREPRIVEE | UNIONBANCAIREPRIVEE,UBPSA |
| bank_code | SKANDINAVISKAENSKILDABANKENAB | SKANDINAVISKAENSKILDABANKENAB |
| bank_code | BANKJULIUSBAER | BANKJULIUSBAER&CO.LTD. |
| bank_code | CREDITINDUSTRIELETCOMMERCIAL | CREDITINDUSTRIELETCOMMERCIAL |
| bank_code | YWXGSY | Shin Kong Commercial Bank Co., Ltd. |
| bank_code | ZGYHXG | Bank of China Hong Kong Branch |
| bank_code | SWITZERLAND | CAINDOSUEZ(SWITZERLAND)SA |
| bank_code | ICBCSTANDARDBANKPLC | ICBCSTANDARDBANKPLC |
| bank_code | LGTHJXG | LGT Bank (Hong Kong) |
| bank_code | MGLB | Macquarie Bank Limited |
| bank_code | SHPDFZ | Shanghai Pudong Development Bank Co., Ltd. |
| bank_code | ZGMSB | China Minsheng Banking Corp., Ltd. |
| bank_code | PICTET&CIE | PICTET&CIE(EUROPE)S.A. |
| bank_code | GFB | China Guangfa Bank Co., Ltd. |
| bank_code | BOHB | Bank of Bohai Co., Ltd. |
| bank_code | ZGGD | China Everbright Bank Co., Ltd. |
| bank_code | SJZYXT | Sumitomo Mitsui Trust Bank, Limited |
| bank_code | SHXGB | Bank of Shanghai (Hong Kong) Limited |
| bank_code | CIMBBANKBERHAD | CIMBBANKBERHAD |
| bank_code | XINYEB | Industrial Bank Co., Ltd. |
| bank_code | YDSY | Yuanta Commercial Bank Co., Ltd. |
| bank_code | MASHREQBANK | MASHREQBANK-PUBLICSHAREHOLDINGCOMPANY |
| bank_code | KOOKMINBANK | KOOKMINBANK |
| bank_code | JTXG | Bank of Communications (Hong Kong) Limited |
| bank_code | ZSYHB | China Zheshang Bank Co., Ltd. |
| bank_code | MGSDL | Morgan Stanley Bank Asia Limited |
| bank_code | PINANB | Ping An Bank Co., Ltd. |
| bank_code | HUAXIA | Hua Xia Bank Co., Ltd. |
| bank_code | ZANB | ZA Bank Limited |
| bank_code | LIVIVBLIMITED | LIVIVBLIMITED |
| bank_code | MOXBANKLIMITED | MOXBANKLIMITED |
| bank_code | WELABBANKLIMITED | WELABBANKLIMITED |
| bank_code | FRB | Fusion Bank Limited |
| bank_code | PAYZHT | PAO Bank Limited |
| bank_code | MYXG | Ant Bank (Hong Kong) Limited |
| bank_code | QATARNATIONALBANK | QATARNATIONALBANK(Q.P.S.C.) |
| bank_code | TXB | Airstar Bank Limited |
| bank_code | HONGKONGSECURITIESCLEARINGCOMPANYLIMITED | HONGKONGSECURITIESCLEARINGCOMPANYLIMITED |
| bank_code | CLSBANKINTERNATIONAL | CLSBANKINTERNATIONAL |
| bank_code | BANQUE | BANQUE PICTET & CIE SA |
| bank_code | DGB | Bank of Dongguan Co., Ltd. |
| bank_code | NXB | NongHyup Bank |
| bank_code | WECHAT | WeChat Pay Hong Kong Limited |
| bank_code | ALIPAY | Alipay Financial Services(HK) Limited |
| bank_code | BDTK | Octopus Cards Limited |
| bank_code | LHB | 388 LIVI BANK LIMITED |

# 13. Error Codes

| Status Code  | Description                |
|------|-------------------|
| 200  | Success                |
| 1000 | Internal error              |
| 1001 | IP is not in the merchant IP whitelist       |
| 1002 | Parameter error              |
| 1003 | Signature error              |
| 1004 | This API is not enabled for the current merchant (contact operations to verify: merchant or App does not exist / is disabled / payment product is not configured)   |
| 1005 | Merchant does not exist             |
| 1006 | Current user IP is blacklisted       |
| 1007 | Current user is blacklisted         |
| 1008 | Merchant App does not exist          |
| 1009 | Payment product does not exist           |
| 1010 | Payment channel does not exist           |
| 1011 | Payment channel development is not completed yet and is temporarily unavailable  |
| 1012 | Payment channel exception, please try again later      |
| 1013 | The current order volume is too high, please try again later      |
| 1014 | Duplicate order number             |
| 1015 | Insufficient app balance           |
| 1016 | The same user is placing orders too frequently, please try again later    |
| 1017 | Order record does not exist           |
| 1018 | Current amount is not supported           |
| 1019 | Payment collection is not enabled in the current app's country |
| 1020 | Pay-out is not enabled in the current app's country |
| 1021 | Failure                |
|1036|API not open|
|1037|Currency not supported|
|1038|Abnormal UTR returned in pay-in callback|
| 9999 | Other error              |
|3000|System under upgrade and maintenance, order placement is temporarily suspended, please try again later|

# 14. Pay-in Cashier API

URL: `https://{api_domain}/api/v1/cashApi/CashIn.html`
Request method: `GET`

### Parameters:

| Name           | Type     | Required    | Description                        |
|--------------|--------|-------|---------------------------|
| app_id       | string | true  | Merchant app_id                    |
| order_no       | string | true  | Merchant order number                     |
| amount       | string | true  | Merchant amount, unit: Hong Kong Dollar (HKD)                     |
| notice_url       | string | false | Asynchronous notification URL                     |
| pay_code       | int    | true | Product code                     |

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{merchant_order_no}}&amount={{merchant_amount}}&notice_url={{async_notification_url}}&pay_code={{product_code}}
```
---
# 15. Document Update Time
```
2026-08-24 13:05:52
```
