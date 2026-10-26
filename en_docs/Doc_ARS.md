# 1. Integration Process

> 1. Conduct business negotiations to open an account and discuss applicable rates.
>
> 2. Contact operations to create the merchant ID, secret key, merchant appId, product code, and apiUrl.
>
> 3. Once development is complete, both parties perform integration testing to verify that requests, callbacks, and other information are complete.

# 2. Md5 Signature Algorithm

> 1. Sort all parameters in ascending order as key-value pairs (key1=value1); parameters with empty values are excluded from signing.
>
> 2. Concatenate them in the format: key1=value1&key2=value2
>
> 3. Append the merchant secret key: key1=value1&key2=value2...&key=MerchantSecretKey
>
> 4. sign = md5(string assembled in the previous step) — the signature result is 32 lowercase characters.
>
> 5. The signing key can be found in the Merchant Dashboard → Basic Information, or by contacting our customer support.

# 3. Precautions

## 3.1 API-Related

> 1. All interfaces in this document use the standard HTTP protocol with POST submission. Both request and response Content-Type are `application/json`, and the character encoding is UTF-8.
>
> 2. All monetary amounts are in <span style="color:red;">Centavo</span>.
>
> 3. The IP address used to call the API must be whitelisted.
>
> 4. For `user_ip`, please collect the user's real IP address. If unavailable, leave it empty — do not use local IPs such as 127.0.0.1.

## 3.2 Callback-Related

> 1. Upon successfully receiving and processing a callback, return the plain text <span style="color:red;">`success`</span> with no other characters. The system will then stop pushing notifications for that order; otherwise, it will retry multiple times.
>
> 2. During asynchronous notification interactions, if the response received is not `success`, the notification is considered failed, and the system will re-send notifications at scheduled intervals: 1m, 1m, 4m, 10m, 10m, 1h, 2h, 6h, 15h.
>
> 3. If `pay_notice_url` is empty, it will be treated as the merchant not requiring a callback, and the system will not push any notifications.

# 4. Payment Collection (Pay-In) Order API

(The ordering IP must be whitelisted by us — please contact us.)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayInOrder`

## 4.1 Pay-In — Order Request Parameters

| Name             | Type   | Required | Description                                                                 |
|------------------|--------|----------|-----------------------------------------------------------------------------|
| trade_no         | int    | true     | Merchant ID                                                                 |
| app_id           | int    | true     | Merchant appId                                                              |
| pay_code         | int    | true     | Product code — contact our operations team to obtain                        |
| pay_method       | string | true     | Payment method — refer to the payment method dictionary                     |
| price            | int    | true     | Order amount in <span style="color:red;">Centavo</span> (integer). 1 ARS = 100 Centavo |
| order_no         | string | true     | Merchant order number                                                       |
| success_url      | string | false    | Redirect URL on payment success                                             |
| fail_url         | string | false    | Redirect URL on payment failure                                             |
| pay_notice_url   | string | false    | Payment success notification URL `recommended`                              |
| user_id          | string | false    | System user ID `recommended`                                                |
| user_ip          | string | false    | Payer's IP address `recommended`                                            |
| attach           | string | true     | Additional parameters as a JSON string                                      |
| sign             | string | true     | Signature result — see signing method at the top of this document           |
| timestamp        | string | false    | Order timestamp — 10-digit Unix timestamp in seconds                        |

- Pay-In — `attach` Additional Parameter Field Description

| Name            | Type   | Required | Description                                                                                      |
|-----------------|--------|----------|--------------------------------------------------------------------------------------------------|
| name            | string | false    | Full name `recommended`                                                                          |
| phone           | string | false    | Phone number `recommended`                                                                       |
| email           | string | false    | Email address `recommended`                                                                      |
| identify_type   | string | true     | ID document type: `DNI` (National ID — 8 digits) `RUC` (Tax card — 11 digits)                   |
| identify_num    | string | true     | ID number. When `identify_type=DNI`, use the Argentine national ID (8 digits — ensure sufficient digit length) |

##### attach Example
```json
{"identify_type":"DNI", "identify_num":"123456789" }
```

- Pay-In — Order Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_method": "CVU",
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"identify_type\":\"DNI\", \"identify_num\":\"123456789\" }",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}
```

## 4.2 Pay-In — Order Response

| Name          | Type   | Required | Description                                                                                         |
|---------------|--------|----------|-----------------------------------------------------------------------------------------------------|
| code     | int  | true   | 200: Success; Others: Failure.                                        |
| msg           | string | true     | Failure reason                                                                                      |
| pay_url       | string | false    | Payment link                                                                                        |
| qr_code       | string | false    | QR code Base64 string                                                                               |
| order_no      | string | true     | Merchant order number                                                                               |
| dis_order_no  | string | true     | Platform order number                                                                               |
| create_time   | int    | true     | Creation timestamp                                                                                  |
| pay_info      | string | false    | Payment information as a JSON string, e.g.: `{"pay_raw":"Raw payment data — merchant can convert to QR code"}` |
| sign          | string | true     | Signature result — see signing method at the top of this document                                  |

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

# 5. Pay-In Callback Notification — POST/JSON

Push URL: The `pay_notice_url` provided when placing the order.
Callback IP: `call_back_server_ip` — please add our IP to your callback whitelist.

## 5.1 Pay-In Callback — Request Parameters

| Name          | Type   | Required | Description                                                                                                              |
|---------------|--------|----------|--------------------------------------------------------------------------------------------------------------------------|
| trade_no      | int    | true     | Merchant ID                                                                                                              |
| status        | int    | true     | Order status: <span style="color:red;">2 = Success</span>, 3 = Failed                                                                                    |
| order_no      | string | true     | Merchant order number                                                                                                    |
| dis_order_no  | string | true     | Platform order number                                                                                                    |
| order_price   | int    | true     | Order amount in <span style="color:red;">Centavo</span>                                                                    |
| <span style="color:red;">real_price</span>    | int    | true     | <span style="color:red;">Actual amount paid by user in Centavo</span>                                                      |
| nti_time      | int    | false    | Notification dispatch time                                                                                               |
| payer         | string | false    | JSON string with payer information: `{"name":"Full Name","account":"Account","utr2":"Bank Transaction No."}`. In addition to the example fields, this parameter will also include payer-related fields from the merchant's `attach`. |
| pay_info      | string | false    | Payment information as a JSON string, e.g.: raw payment data, card number, name, bank, etc.                              |
| create_time   | int    | true     | Order creation time                                                                                                      |
| sign          | string | true     | Signature result — see signing method at the top of this document                                                        |

- Pay-In Callback — Request Example

```json
{
  "trade_no": 10003,
  "status": 3,
  "order_no": "p71584120256SlWlKkymb",
  "dis_order_no": "2025071130460153942908928india1sKQbX",
  "order_price": 10099,
  "real_price": 10000,
  "payer": "{\"name\":\"Full Name\"}",
  "nti_time": 1752826164,
  "create_time": 1752751502,
  "sign": "eba7f27e0f49581d8784294ef29f994d"
}
```

## 5.2 Pay-In Callback — Response Description

Upon successfully receiving and processing the callback, return <span style="color:red;">`success`</span>. The system will then stop pushing notifications for this order; otherwise, it will retry multiple times.

# 6. Payout (Pay-Out) Order API

(The ordering IP must be whitelisted by us — please contact us.)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayOutOrder`

## 6.1 Pay-Out — Request Parameters

| Name             | Type   | Required | Description                                                                                                                                          |
|------------------|--------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| trade_no         | int    | true     | Merchant ID                                                                                                                                          |
| order_no         | string | true     | Merchant order number                                                                                                                                |
| app_id           | int    | true     | Merchant appId                                                                                                                                       |
| pay_code         | int    | true     | Product code — contact our operations team to obtain                                                                                                 |
| price            | int    | true     | Order amount in <span style="color:red;">Centavo</span> (integer)                                                                                    |
| account_no       | string | true     | Recipient account number `CVU/CBU/BANK account`                                                                                                      |
| account_type     | string | true     | Account type: CBU, CVU, BANK                                                                                                                         |
| account_name     | string | true     | Account holder name                                                                                                                                  |
| bank_code        | string | true     | Bank code — refer to the bank code dictionary                                                                                                        |
| identify_type    | string | true     | ID document type: `DNI` (National ID — 8 digits) `CUIL` (Social security — 11 digits) `RUC` (Tax card — 11 digits) `PP` (Passport — up to 15 digits) |
| identify_num     | string | true     | ID number                                                                                                                                            |
| pay_notice_url   | string | false    | Payout success notification URL                                                                                                                      |
| attach           | string | false    | Additional parameters: `{"email":"email address","phone":"phone number"}`                                                                            |
| user_ip          | string | false    | Recipient user IP — `recommended; do not use the same IP for multiple requests to avoid rate limiting`                                               |
| sign             | string | true     | Signature result — see signing method at the top of this document                                                                                    |
| timestamp        | string | false    | Order timestamp — 10-digit Unix timestamp in seconds `recommended`                                                                                   |

- Pay-Out — `attach` Additional Parameter Field Description

| Name    | Type   | Required | Description                      |
|---------|--------|----------|----------------------------------|
| phone   | string | false    | Phone number `recommended`       |
| email   | string | false    | Email address `recommended`      |

- Pay-Out — Request Example

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
  "account_type": "CBU",
  "account_no": "2850590940090418135201",
  "account_name": "Juan Perez",
  "bank_code": "CBU",
  "identify_type": "DNI",
  "identify_num": "12345678"
}
```

## 6.2 Pay-Out — Order Response

| Name          | Type   | Required | Description                                                                          |
|---------------|--------|----------|--------------------------------------------------------------------------------------|
| code          | int    | true     | 200: Order successful; other values: Order failed                                    |
| msg           | string | true     | Failure reason                                                                       |
| dis_order_no  | string | true     | Platform order number                                                                |
| order_no      | string | true     | Merchant order number                                                                |
| status        | int    | true     | Order status: 2 = Payout successful, 3 = Payout failed, 7 = Rejected, 9 = Reversed, 10 = Processing |
| create_time   | int    | true     | Creation timestamp                                                                   |
| sign          | string | true     | Signature result — see signing method at the top of this document                   |

- Pay-Out — Order Response Examples

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

# 7. Pay-Out Callback Notification

Push URL: The `pay_notice_url` provided when placing the order.
Callback IP: `call_back_server_ip` — please add our IP to your callback whitelist.

## 7.1 Pay-Out Callback — Request Parameters

| Name          | Type   | Required | Description                                                              |
|---------------|--------|----------|--------------------------------------------------------------------------|
| trade_no      | int    | true     | Merchant ID                                                              |
| order_no      | string | true     | Merchant order number                                                    |
| dis_order_no  | string | true     | Platform order number                                                    |
| order_price   | int    | true     | Order amount                                                             |
| fee           | int    | false    | Transaction fee                                                          |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status        | int    | true     | Order status: <span style="color:red;">2 = Payout successful</span>, 3 = Payout failed, 7 = Rejected, 9 = Reversed |
| pay_info      | string | false    | Payment information                                                      |
| remark        | string | false    | Failure reason                                                           |
| create_time   | int    | true     | Order creation time                                                      |
| sign          | string | true     | Signature result — see signing method at the top of this document        |
| nti_time      | int    | true     | Notification dispatch time                                               |

- Pay-Out Callback — Request Example

```json
{
  "trade_no": 10000,
  "status": 2,
  "order_no": "20060354339090013",
  "dis_order_no": "Meg2352644o2nmjo0800indiaYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": ""
}
```

## 7.2 Pay-Out Callback — Response Description

Upon successfully receiving and processing the callback, return <span style="color:red;">`success`</span>. The system will then stop pushing notifications for this order; otherwise, it will retry multiple times.

# 8. Order Query API (Shared by Pay-In and Pay-Out)

(The request IP must be whitelisted by us — please contact us.)
Query URL: `https://{api_domain}/api/v1/payApi/QueryOrder`

## 8.1 Query — Request Parameters

| Name          | Type   | Required | Description                                                       |
|---------------|--------|----------|-------------------------------------------------------------------|
| order_type    | string | true     | `pay_out`: Payout, `pay_in`: Pay-in                               |
| trade_no      | int    | true     | Merchant ID                                                       |
| app_id        | int    | true     | Merchant appId                                                    |
| order_no      | string | false    | Merchant order number — use either this or `dis_order_no`         |
| dis_order_no  | string | false    | Platform order number — use either this or `order_no`             |
| sign          | string | true     | Signature result — see signing method at the top of this document |

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

| Name          | Type   | Required | Description                                                                          |
|---------------|--------|----------|--------------------------------------------------------------------------------------|
| code          | int    | true     | 200: Query successful; other values: Failed                                          |
| msg           | string | true     | Query failure reason                                                                 |
| trade_no      | int    | true     | Merchant ID                                                                          |
| <span style="color:red;">real_price</span>    | int    | true     | <span style="color:red;">Actual payment amount</span>                                                                |
| status        | int    | true     | Order status: 1 = Unpaid, <span style="color:red;">2 = Success</span>, 3 = Failed, 7 = Rejected, 9 = Reversed, 10 = Processing |
| success_time  | int    | true     | Success timestamp                                                                    |
| order_no      | string | true     | Merchant order number                                                                |
| dis_order_no  | string | true     | Platform order number                                                                |
| remark        | string | true     | Payout failure reason                                                                |
| fee           | int    | false    | Transaction fee                                                                      |
| create_time   | int    | true     | Order creation time                                                                  |
| payer         | string | false    | JSON string with payer information: `{"account_name":"Full Name","account_type":"Account Type: CBU, CVU, BANK","account_no":"Account","identify_type":"ID Type: DNI, CUIL, RUC, PP","identify_num":"ID Number","bank_code":"Bank Code","utr2":"Bank Transaction No."}` |
| pay_info      | string | false    | Payment information as a JSON string, e.g.: raw payment data, card number, name, bank, etc. |
| sign          | string | true     | Signature result — see signing method at the top of this document                   |
| utr2          | string | false    | Bank order number                                                                    |

- Query Response Examples

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
  "payer": "{\"account_name\":\"Juan Perez\",\"account_type\":\"CBU\",\"account_no\":\"2850590940090418135201\",\"identify_type\":\"DNI\",\"identify_num\":\"12345678\",\"bank_code\":\"CBU\",\"utr2\":\"9901108391\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Payout Balance Query API

(The request IP must be whitelisted by us — please contact us.)
URL: `https://{api_domain}/api/v1/payApi/QueryBalance`

## 9.1 Balance Query — Request Parameters

| Name      | Type   | Required | Description                                                       |
|-----------|--------|----------|-------------------------------------------------------------------|
| trade_no  | int    | true     | Merchant ID                                                       |
| app_id    | int    | true     | Merchant appId                                                    |
| sign      | string | true     | Signature result — see signing method at the top of this document |

- Balance Request Example

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Query Response

| Name             | Type   | Required | Description                                                       |
|------------------|--------|----------|-------------------------------------------------------------------|
| code             | int    | true     | 200: Query successful; other values: Failed                       |
| msg              | string | true     | Failure reason                                                    |
| balance          | int    | true     | Available balance                                                 |
| balance_frozen   | int    | false    | Frozen balance                                                    |
| sign             | string | true     | Signature result — see signing method at the top of this document |

- Balance Response Examples

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

(The request IP must be whitelisted by us — please contact us.)
URL: `https://{api_domain}/api/v1/payApi/QueryCertificate`

## 10.1 Payment Voucher — Request Parameters

| Name          | Type   | Required | Description                                                        |
|---------------|--------|----------|--------------------------------------------------------------------|
| trade_no      | int    | true     | Merchant ID                                                        |
| app_id        | int    | true     | Merchant appId                                                     |
| order_no      | string | false    | Merchant order number — use either this or `dis_order_no`          |
| dis_order_no  | string | false    | Platform order number — use either this or `order_no`              |
| sign          | string | true     | Signature result — see signing method at the top of this document  |

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

## 10.2 Payment Voucher — Response Parameters

| Name       | Type   | Required | Description                                                       |
|------------|--------|----------|-------------------------------------------------------------------|
| code       | int    | true     | 200: Success; other values: Failed                                |
| msg        | string | true     | Failure reason                                                    |
| img_link   | string | false    | Voucher image URL                                                 |
| img_base   | string | false    | Voucher as a Base64-encoded image                                 |
| sign       | string | true     | Signature result — see signing method at the top of this document |

- Payment Voucher Response Examples

No voucher available:

```json
{
  "code": 200,
  "msg": "No payment voucher available at the moment",
  "sign": "",
  "img_link": "",
  "img_base": ""
}
```

Voucher available:

```json
{
  "code": 200,
  "msg": "",
  "sign": "3969f17cd1a551769f85967d0a05b7b6",
  "img_link": "http://dsggfgdsf.djdj?ddd=snn",
  "img_base": "data:image/png;base64,hfhshdhfhfh"
}
```

# 11. Payment Methods — Pay-In Field: `pay_method`

|Field Name| Value    | Description                                                                                                                                                             |
|----------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| pay_method| CVU      | Argentine bank transfer. **Note:** This method requires the user to enter the bank transaction reference number at the checkout page after completing payment; otherwise the order may be lost. |
| pay_method| QR_ARS   | QR code scan — not yet supported; merchants will be notified once this method becomes available.                                                                        |

# 12. Bank Code Dictionary — Pay-Out Field: `bank_code`

|Field Name| Value | Description |
|----------|---|---|
| bank_code | CBU | CBU |
| bank_code | CVU | CVU |
| bank_code | ALIAS | ALIAS |
| bank_code | ABACO_CIA_FINANCIERA | ABACO CIA. FINANCIERA S.A. |
| bank_code | ALHEC_TOURSC | ALHEC TOURS S.A.C.B.YT. |
| bank_code | AMERICAN_EXPRESS | AMERICAN EXPRESS BANK LTD. |
| bank_code | ASTROPAY | ASTROPAY |
| bank_code | BANKERS_TRUST_ARGENTINA_SOCIEDAD_DE_BOLSA | BANKERS TRUST ARGENTINA S.A. SOCIEDAD DE BOLSA |
| bank_code | BANKERS_TRUST_RIO_DE_LA_PLATA_SACF | BANKERS TRUST RIO DE LA PLATA SACF |
| bank_code | BBVA_FRANCES | BBVA BANCO FRANCES S.A. |
| bank_code | BNP_PARIBAS_SUCCURSALE_DE_BUENOS_AIRES | BNP PARIBAS SUCCURSALE DE BUENOS AIRES |
| bank_code | BOLSA_DE_COMERCIO_ROSARIO | BOLSA DE COMERCIO ROSARIO |
| bank_code | BOSTON_INVERSORA_DE_VALORES | BOSTON INVERSORA DE VALORES S.A. |
| bank_code | BRADESCO_ARGENTINA | BANCO BRADESCO ARGENTINA S/A |
| bank_code | CAJA_DE_VALORES | CAJA DE VALORES S.A. |
| bank_code | CAMBIO_EXCURSIONES_TURISMO_COLUMBUS | CAMBIO EXCURSIONES TURISMO COLUMBUS S.A. |
| bank_code | CAMBIO_GARCIA_NAVARRO_RAMAGLIO_Y_CIA | CAMBIO GARCIA NAVARRO RAMAGLIO Y CIA. S.A. |
| bank_code | CAPITAL_MARKETS | CAPITAL MARKETS |
| bank_code | CASA_LERCHUNDI_SACIF | CASA LERCHUNDI SACIF |
| bank_code | CENTRAL_DE_LA_REPUBLICA_ARGENTINA | BANCO CENTRAL DE LA REPUBLICA ARGENTINA |
| bank_code | CITIBANK_ARS | CITIBANK N.A. |
| bank_code | CMF | BANCO CMF S.A. |
| bank_code | COLUMBIA | BANCO COLUMBIA S.A. |
| bank_code | COMAFI | BANCO COMAFI S.A. |
| bank_code | COOPERATIVO_DEL_ESTE_ARGENTINO | BANCO COOPERATIVO DEL ESTE ARGENTINO LTDO. |
| bank_code | CORPORACION_DE_SERVICIOS_FINANCIEROS | CORPORACION DE SERVICIOS FINANCIEROS S.A. |
| bank_code | CREDICOOP_COOPERATIVO_LIMITADO | BANCO CREDICOOP COOPERATIVO LIMITADO |
| bank_code | DEL_CHUBUT | BANCO DEL CHUBUT S.A. |
| bank_code | DEL_TUCUMAN | BANCO DEL TUCUMAN S.A. |
| bank_code | DEUTSCHE_ARS | DEUTSCHE BANK SA |
| bank_code | DE_FORMOSA | BANCO DE FORMOSA S.A. |
| bank_code | DE_GALICIA_Y_BUENOS_AIRES | BANCO DE GALICIA Y BUENOS AIRES |
| bank_code | DE_INVERSION_Y_COMERCIO_EXTERIOR | BANCO DE INVERSION Y COMERCIO EXTERIOR S.A. |
| bank_code | DE_LA_CIUDAD_DE_BUENOS_AIRES | BANCO DE LA CIUDAD DE BUENOS AIRES |
| bank_code | DE_LA_NACION_ARGENTINA | BANCO DE LA NACION ARGENTINA |
| bank_code | DE_LA_PAMPA | BANCO DE LA PAMPA S.E.M. |
| bank_code | DE_LA_PROVINCIA_DEL_NEUQUEN | BANCO DE LA PROVINCIA DEL NEUQUEN S.A. |
| bank_code | DE_LA_PROVINCIA_DE_BUENOS_AIRES | BANCO DE LA PROVINCIA DE BUENOS AIRES |
| bank_code | DE_LA_PROVINCIA_DE_CORDOBA | BANCO DE LA PROVINCIA DE CORDOBA |
| bank_code | DE_LA_REPUBLICA_ORIENTAL_DEL_URUGUAY | BANCO DE LA REPUBLICA ORIENTAL DEL URUGUAY |
| bank_code | DE_SANTA_CRUZ | BANCO DE SANTA CRUZ S A |
| bank_code | DE_SAN_JUAN | BANCO DE SAN JUAN SA |
| bank_code | DE_SERVICIOS_Y_TRANSACCIONES | BANCO DE SERVICIOS Y TRANSACCIONES SA |
| bank_code | DE_VALORES | BANCO DE VALORES |
| bank_code | FINANCIERA_BUENOS_AIRESL | FINANCIERA BUENOS AIRES S.A.L. Y C. |
| bank_code | FINANSUR | BANCO FINANSUR S.A. |
| bank_code | FITCH_ARGENTINA_CALIFICADORA_DE_RIESGO | FITCH ARGENTINA CALIFICADORA DE RIESGO S.A. |
| bank_code | FOREXCAMBIO | FOREXCAMBIO SA |
| bank_code | GARCIA_NAVARROY | GARCIA NAVARROY CIA. S.A. |
| bank_code | HIPOTECARIO | BANCO HIPOTECARIO S.A. |
| bank_code | HSBC_BANK_ARGENTINA | HSBC BANK ARGENTINA SA |
| bank_code | INDUSTRIAL | BANCO INDUSTRIAL S.A. |
| bank_code | INDUSTRIAL_AND_COMMERCIAL_BANK_OF_CHINA | INDUSTRIAL AND COMMERCIAL BANK OF CHINA (ARGENTINA)S.A. |
| bank_code | INTERAMERICANO_DE_DESARROLLO | BANCO INTERAMERICANO DE DESARROLLO |
| bank_code | INTERFINANZAS | BANCO INTERFINANZAS S.A. |
| bank_code | INVERSIONARIA_ARGENTINA | INVERSIONARIA ARGENTINA S.A. |
| bank_code | INVERSIONES_UNIDAS | INVERSIONES UNIDAS S.A. |
| bank_code | INVERSORA_COMPANIA_FINANCIERA | INVERSORA S.A. COMPANIA FINANCIERA |
| bank_code | INVERSUR | INVERSUR S.R.L. |
| bank_code | ITAU_ARGENTINA | BANCO ITAU ARGENTINA S.A. |
| bank_code | JPMORGAN_CHASE | JPMORGAN CHASE BANK N.A. |
| bank_code | MACRO | BANCO MACRO S.A. |
| bank_code | MARIVA | BANCO MARIVA S.A. |
| bank_code | MBA_DE_INVERSIONES | MBA BANCO DE INVERSIONES SA |
| bank_code | MERCADADOPAGO | MERCADADOPAGO |
| bank_code | MERCADO_DE_VALORES_DE_BUENOS_AIRES_MERVAL | MERCADO DE VALORES DE BUENOS AIRES S.A. MERVAL |
| bank_code | MERCADO_DE_VALORES_DE_BUENOS_AIRES_SA | MERCADO DE VALORES DE BUENOS AIRES SA |
| bank_code | MERCHANT_BANK_ARGENTINA | MERCHANT BANK ARGENTINA |
| bank_code | MERCURIO | BANCO MERCURIO S.A. |
| bank_code | MERIDIAN | BANCO MERIDIAN S.A. |
| bank_code | METROPOLIS_COMPANIA_FINANCIERA | METROPOLIS COMPANIA FINANCIERA S.A. |
| bank_code | MONTEMAR_COMPANIA_FINANCIERA | MONTEMAR COMPANIA FINANCIERA S.A. |
| bank_code | MOODYS_LATIN_AMERICA_CALIFICADORA_DE_RIESGO | MOODYS LATIN AMERICA CALIFICADORA DE RIESGO S.A. |
| bank_code | MULTIFINANZAS_COMPANIA_FINANCIERA | MULTIFINANZAS COMPANIA FINANCIERA S.A. |
| bank_code | MUNICIPAL_DE_ROSARIO | BANCO MUNICIPAL DE ROSARIO |
| bank_code | NUEVO_DEL_CHACO | NUEVO BANCO DEL CHACO S.A. |
| bank_code | NUEVO_DE_ENTRE_RIOS | NUEVO BANCO DE ENTRE RIOS S.A. |
| bank_code | NUEVO_DE_SANTA_FE | NUEVO BANCO DE SANTA FE S.A. |
| bank_code | PATAGONIA | BANCO PATAGONIA S.A. |
| bank_code | PIANO | BANCO PIANO S.A. |
| bank_code | PROVINCIA_DE_TIERRA_DEL_FUEGO | BANCO PROVINCIA DE TIERRA DEL FUEGO |
| bank_code | RABELLO_Y_CIA_SOCIEDAD_DE_BOLSA | RABELLO Y CIA S.A. SOCIEDAD DE BOLSA |
| bank_code | RAYMOND_JAMES_ARGENTINA_SOCIEDAD_DE_BOLSA_SOCIEDADANONIMA | RAYMOND JAMES ARGENTINA SOCIEDAD DE BOLSA SOCIEDADANONIMA |
| bank_code | ROELA | BANCO ROELA S.A. |
| bank_code | ROYAL_BANK_OF_SCOTLAND_NV | ROYAL BANK OF SCOTLAND NV (SUC ARGENTINA) |
| bank_code | SAENZ | BANCO SAENZ S.A. |
| bank_code | SAMSUNG_ELECTRONICS_ARGENTINA | SAMSUNG ELECTRONICS ARGENTINA S.A. |
| bank_code | SANTANDER_RIO | BANCO SANTANDER RIO S.A. |
| bank_code | SBS_SOCIEDAD_DE_BOLSA | SBS SOCIEDAD DE BOLSA S.A. |
| bank_code | SUPERVIELLE | BANCO SUPERVIELLE |
| bank_code | T.P.C.G._CAPITAL | T.P.C.G. CAPITAL S.A. |
| bank_code | TAVELLI_Y_CIA.S.A.SOCIEDAD_DE_BOLSA | TAVELLI Y CIA.S.A.SOCIEDAD DE BOLSA |
| bank_code | TOKYO_MITSUBISHI_UFJ_LTD_THE | BANK OF TOKYO MITSUBISHI UFJ LTD. THE |
| bank_code | TRANSCAMBIO | TRANSCAMBIO S.A. |
| bank_code | TUTELAR_COMPANIA_FINANCIERA | TUTELAR COMPANIA FINANCIERA S.A. |
| bank_code | VANEXVA_BURSATIL_SA | VANEXVA BURSATIL SA |

# 13. Error Codes

| Status Code | Description                                                                                                     |
|-------------|-----------------------------------------------------------------------------------------------------------------|
| 200         | Success                                                                                                         |
| 1000        | Internal error                                                                                                  |
| 1001        | IP address is not on the merchant whitelist                                                                     |
| 1002        | Parameter error                                                                                                 |
| 1003        | Signature error                                                                                                 |
| 1004        | This interface is not enabled for the current merchant (contact operations to verify: merchant or App does not exist / is disabled / payment product not configured) |
| 1005        | Merchant does not exist                                                                                         |
| 1006        | Current user IP is blacklisted                                                                                  |
| 1007        | Current user is blacklisted                                                                                     |
| 1008        | Merchant App does not exist                                                                                     |
| 1009        | Payment product does not exist                                                                                  |
| 1010        | Payment channel does not exist                                                                                  |
| 1011        | Payment channel is under development and not yet available                                                      |
| 1012        | Payment channel error — please try again later                                                                  |
| 1013        | Current order volume is too high — please try again later                                                       |
| 1014        | Duplicate order number                                                                                          |
| 1015        | Insufficient App balance                                                                                        |
| 1016        | Same user is placing orders too frequently — please try again later                                             |
| 1017        | Order record does not exist                                                                                     |
| 1018        | The current amount is not supported                                                                             |
| 1019        | Pay-in orders are not yet enabled for the country of the current App                                            |
| 1020        | Payout orders are not yet enabled for the country of the current App                                            |
| 1021        | Failed                                                                                                          |
| 1036        | Interface not yet available                                                                                     |
| 1037        | Currency not supported                                                                                          |
| 1038        | Abnormal UTR callback for pay-in                                                                                |
| 9999        | Other errors                                                                                                    |
| 3000        | System is under maintenance — order placement suspended, please try again later                                 |

# 14. Pay-In Cashier Interface

URL: `https://{api_domain}/api/v1/cashApi/CashIn.html`
Request Method: GET

### Parameters:

| Name         | Type   | Required | Description                              |
|--------------|--------|----------|------------------------------------------|
| app_id       | string | true     | Merchant app_id                          |
| order_no     | string | true     | Merchant order number                    |
| amount       | string | true     | Order amount — unit: `Argentine Peso (ARS)` |
| notice_url   | string | false    | Asynchronous notification URL            |
| pay_code     | int    | true     | Product code                             |

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{merchant_order_no}}&amount={{order_amount}}&notice_url={{async_notification_url}}&pay_code={{product_code}}
```

# 15. Document Last Updated
```
2026-06-10 20:25:00
```
