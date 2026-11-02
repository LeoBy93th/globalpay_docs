
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

| Name           | Type   | Required | Description                                                                       |
| -------------- | ------ |----------| --------------------------------------------------------------------------------- |
| trade_no       | int    | true     | Merchant number                                                                   |
| app_id         | int    | true     | Merchant appId                                                                    |
| pay_code       | int    | true     | Product code, obtain from our operations team                                     |
| pay_method     | string | true     | Payment method, refer to the payment method dictionary                            |
| price          | int    | true     | Order amount, unit: centavos, integer. 1 peso = 100 centavos                      |
| order_no       | string | true     | Merchant order number                                                             |
| success_url    | string | false    | Redirect URL after successful payment                                             |
| fail_url       | string | false    | Redirect URL after failed payment                                                 |
| pay_notice_url | string | false    | Payment success notification URL                                                  |
| user_id        | string | true     | System user ID                                                                    |
| user_ip        | string | true     | Payer IP                                                                          |
| attach         | string | false     | Additional parameter JSON string: {"name":"Name","phone":"Phone","email":"Email"} |
| sign           | string | true     | Signature result, signature method described at the top of the document           |
| timestamp      | string | false    | Order timestamp, 10-digit timestamp in seconds                                    |

### Pay-in Order Request Example

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

## 4.2 Pay-in Order Response

| Name         | Type   | Required | Description                                                                                       |
| ------------ | ------ | -------- | ------------------------------------------------------------------------------------------------- |
| code         | int    | true     | 200: order created successfully, others: failed                                                   |
| msg          | string | true     | Failure reason                                                                                    |
| pay_url      | string | false    | Payment link                                                                                      |
| qr_code      | string | false    | PIX QR code string                                                                                |
| order_no     | string | true     | Merchant order number                                                                             |
| dis_order_no | string | true     | Platform order number                                                                             |
| create_time  | int    | true     | Create time                                                                                       |
| pay_info     | string | false    | Payment info JSON string, e.g. {"pay_raw":"Native payment info, merchant can convert to QR code"} |
| sign         | string | true     | Signature result, signature method described at the top of the document                           |

### Pay-in Order Response Example

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

---

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
| sign         | string | true     | Signature result, signature method described at the top of the document                                                                                                                                                                                                                                                                                   |

### Pay-in Callback Request Example

```json
{
  "trade_no": 10003,
  "status": 3,
  "order_no": "p71584120256SlWlKkymb",
  "dis_order_no": "2025071130460153942908928india1sKQbX",
  "order_price": 10099,
  "real_price": 10000,
  "payer": "{\"name\":\"Name\",\"email\":\"Email\",\"phone\":\"Phone\",\"identify_type\":\"ID Type\",\"identify_num\":\"CPF,CNPJ\"}",
  "nti_time": 1752826164,
  "create_time": 1752751502,
  "sign": "eba7f27e0f49581d8784294ef29f994d"
}
```

## 5.2 Pay-in Callback Response Description

If the callback is received and processed successfully, please return <span style="color:red;">**success**</span>. The system will stop pushing this order information, otherwise it will push repeatedly multiple times.

---

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
| account_type   | string | true     | Account type: BANK                                                                    |
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
  "account_no": "012345678901",
  "account_name": "Juan Dela Cruz",
  "bank_code": "PH_BDO"
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

---

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
  "app_id": 165,
  "dis_order_no": "p7158277185f96603047656571",
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
| payer        | string | false    | JSON string, payer info {"account_name":"Name","account_type":"Account type: BANK","account_no":"Account","bank_code":"Bank code"} |
| pay_info     | string | false    | Payment info JSON string, e.g. collection/payment raw info, card number, name, bank, etc. 25-10-28                                                                             |
| sign         | string | true     | Signature result, signature method described at the top of the document                                                                                                        |
| utr2         | string | false    | Bank order number                                                                                                                                                              |

### Query Response Example

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
  "payer": "{\"account_name\":\"Juan Dela Cruz\",\"account_type\":\"BANK\",\"account_no\":\"012345678901\",\"bank_code\":\"PH_BDO\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

---

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
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
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
  "balance_frozen": 1000,
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

---

# 10. Payment Voucher Query API

(Request IP must be whitelisted by contacting us)
URL: `https://{api_domain}/api/v1/payApi/QueryCertificate`

## 10.1 Payment Voucher Request Parameters

| Name         | Type   | Required | Description                                                             |
| ------------ | ------ | -------- | ----------------------------------------------------------------------- |
| trade_no     | int    | true     | Merchant number                                                         |
| app_id       | int    | true     | Merchant appid                                                          |
| order_no     | string | false    | Merchant order number, choose one between order_no and dis_order_no     |
| dis_order_no | string | false    | Platform order number, choose one between order_no and dis_order_no     |
| sign         | string | true     | Signature result, signature method described at the top of the document |

### Payment Voucher Request Example

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

| Name     | Type   | Required | Description                                                             |
| -------- | ------ | -------- | ----------------------------------------------------------------------- |
| code     | int    | true     | 200: response success, others: failure                                  |
| msg      | string | true     | Failure reason                                                          |
| img_link | string | false    | Voucher link                                                            |
| img_base | string | false    | Base64 code generated for voucher                                       |
| sign     | string | true     | Signature result, signature method described at the top of the document |

### Payment Voucher Response Example

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

Payment voucher available:

```json
{
  "code": 200,
  "msg": "",
  "sign": "3969f17cd1a551769f85967d0a05b7b6",
  "img_link": "http://dsggfgdsf.djdj?ddd=snn",
  "img_base": "data:image/png;base64,hfhshdhfhfh"
}
```

---

# 11. Payment Method (Pay-in Field: pay_method)

| Field      | Country     | Value | Description |
| ---------- | ----------- | ----- | ----------- |
| pay_method      | Philippines | PH_BAYAD      | Bayad             |
| pay_method    | Philippines | PH_GRABPAY    | GrabPay           |
| pay_method      | Philippines | PH_COINS      | Coins.PH          |
| pay_method       | Philippines | PH_OMNI       | Omnipay, Inc.     |
| pay_method      | Philippines | PH_GCASH      | GCash             |
| pay_method    | Philippines | PH_PAYMAYA    | PayMaya           |
| pay_method       | Philippines | PH_QRPH    | QR            |

---

# 12. Bank Codes

| Field Name | Code       | Bank Name                                        |
| ---------- | ---------- | ------------------------------------------------ |
| bank_code | PH_TC | TayoCash |
| bank_code | PH_SP | ShopeePay |
| bank_code | PH_SB | Seabank |
| bank_code | PH_QB | Queenbank |
| bank_code | PH_JC | JuanCash |
| bank_code | PH_SCB | Standard Chartered Bank |
| bank_code | PH_BAYAD | Bayad |
| bank_code | PH_EWB | East West Banking Corporation |
| bank_code | PH_GRABPAY | GrabPay |
| bank_code | PH_CSB | Citystate Savings Bank |
| bank_code | PH_ISLA | ISLA Bank |
| bank_code | PH_UBPEON | UnionBank EON |
| bank_code | PH_PAR | Partner Rural Bank (Cotabato), Inc. |
| bank_code | PH_PRB | Producers Bank |
| bank_code | PH_QCB | Queen City Development Bank, Inc. |
| bank_code | PH_ROB | Robinsons Bank |
| bank_code | PH_AUB | Asia United Bank (AUB) |
| bank_code | PH_BMB | Bangko Mabuhay (A Rural Bank), Inc. |
| bank_code | PH_BPI | Bank of the Philippine Islands (BPI) |
| bank_code | PH_COINS | Coins.PH |
| bank_code | PH_DBP | Development Bank of the Philippines |
| bank_code | PH_ING | ING Bank N.V. |
| bank_code | PH_MSB | Malayan Bank Savings and Mortgage Bank, Inc. |
| bank_code | PH_PBC | Philippine Bank of Communications (PBCOM) |
| bank_code | PH_PNB | Philippine National Bank (PNB) |
| bank_code | PH_PSB | Philippine Savings Bank (PSBank) |
| bank_code | PH_QCRB | Quezon Capital Rural Bank, Inc |
| bank_code | PH_SEC | Security Bank Corporation |
| bank_code | PH_SPY | Starpay Corporation |
| bank_code | PH_YUANSB | Yuanta Savings Bank |
| bank_code | PH_SSB | Sun Savings Bank |
| bank_code | PH_UBP | Union Bank of the Philippines (UBP) |
| bank_code | PH_UCBSB | UCBP Savings bank |
| bank_code | PH_ABP | AllBank Inc. |
| bank_code | PH_BOC | Bank of Commerce |
| bank_code | PH_BRB | Binangonan Rural Bank (BRBDigital) |
| bank_code | PH_CRD | CARD Bank |
| bank_code | PH_MET | Metropolitan Bank and Trust Company (Metrobank) |
| bank_code | PH_OMNI | Omnipay, Inc. |
| bank_code | PH_PNBSB | PNB Savings Bank |
| bank_code | PH_PVB | Philippine Veterans Bank |
| bank_code | PH_RBG | RURAL BANK OF GUINOBATAN, INC. |
| bank_code | PH_WDB | Wealth Development Bank Corporation |
| bank_code | PH_CBC | China Bank |
| bank_code | PH_CMG | CAMALIG BANK |
| bank_code | PH_DBI | Dungganon Bank (A Microfinance Rural Bank), Inc. |
| bank_code | PH_EQB | Equicom Savings Bank, Inc. |
| bank_code | PH_EWR | EastWest Rural Bank or Komo |
| bank_code | PH_GCASH | GCash |
| bank_code | PH_LBP | Land Bank of The Philippines |
| bank_code | PH_PAYMAYA | PayMaya |
| bank_code | PH_PBB | Philippine Business Bank, Inc., A Savings Bank |
| bank_code | PH_PTC | Philippine Trust Company |
| bank_code | PH_RCBC | Rizal Commercial Banking Corporation (RCBC) |
| bank_code | PH_SBA | Sterling Bank of Asia |
| bank_code | PH_UCPB | United Coconut Planters Bank (UCPB) |
| bank_code | PH_JPM | JP Morgan Chase Bank, N.A. |
| bank_code | PH_BDO | BDO Unibank |
| bank_code | PH_BPIDB | BPI Direct BanKO, Inc., A Savings Bank |
| bank_code | PH_BPISB | BPI / BPI Family Savings Bank |
| bank_code | PH_RSBI | Rcbc Savings Bank Inc. |
| bank_code | PH_CBS | China Bank Savings |
| bank_code | PH_CEBRUR | Cebuana Lhuillier Rural Bank, Inc. |
| bank_code | PH_CTBC | CTBC Bank (Philippines) Corp. |
| bank_code | PH_DCP | DCPAY Philippines |
| bank_code | PH_MPI | Maybank Philippines |
| bank_code | PH_ONB | BDO Network Bank |
| bank_code | PH_ASENSO | Asenso |
| bank_code | PH_DCDB | Dumaguete City Development Bank |
| bank_code | PH_LSB | Legazpi Savings Bank |
| bank_code | PH_MCC | Mindanao Consolidated CoopBank |
| bank_code | PH_NET | Netbank |
| bank_code | PH_CIMB | Commerce International Merchant Bank |
| bank_code | PH_GTB | Gotyme Bank |
| bank_code | PH_USSC | USSC Money Services |
| bank_code | PH_MYB | Maya Bank, Inc. |
| bank_code | PH_METRO | Metrobank |
| bank_code | PH_PPSF | PalawanPay |
| bank_code | ABCRB | Agribusiness Banking Corporation-A Rural Bank |

---

# 13. Error Codes

| Status Code | Description                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200         | Success                                                                                                                                                       |
| 1000        | Internal error                                                                                                                                                |
| 1001        | IP is not in the merchant IP whitelist                                                                                                                        |
| 1002        | Parameter error                                                                                                                                               |
| 1003        | Signature error                                                                                                                                               |
| 1004        | This API is not enabled for the current merchant (contact operations to confirm: merchant or App does not exist / is closed / payment product not configured) |
| 1005        | Merchant does not exist                                                                                                                                       |
| 1006        | Current user IP is blacklisted                                                                                                                                |
| 1007        | Current user is blacklisted                                                                                                                                   |
| 1008        | Merchant App does not exist                                                                                                                                   |
| 1009        | Payment product does not exist                                                                                                                                |
| 1010        | Payment channel does not exist                                                                                                                                |
| 1011        | Payment channel is not yet developed and is temporarily unavailable                                                                                           |
| 1012        | Payment channel exception, please try again later                                                                                                             |
| 1013        | Too many orders currently, please try again later                                                                                                             |
| 1014        | Duplicate order number                                                                                                                                        |
| 1015        | Insufficient app balance                                                                                                                                      |
| 1016        | The same user is placing orders too frequently, please try again later                                                                                        |
| 1017        | Order record does not exist                                                                                                                                   |
| 1018        | Current amount is not supported                                                                                                                               |
| 1019        | Payment order is not available in the country of the current app                                                                                              |
| 1020        | Pay-out order is not available in the country of the current app                                                                                              |
| 1021        | Failed                                                                                                                                                        |
| 1036        | API is not open yet                                                                                                                                           |
| 1037        | Currency not supported                                                                                                                                        |
| 1038        | Pay-in callback utr exception                                                                                                                                 |
| 9999        | Other error                                                                                                                                                   |
| 3000        | System upgrade/maintenance in progress, ordering is suspended, please try again later                                                                         |

---

# 14. Pay-in Cashier API

Address: https://{api_domain}/api/v1/cashApi/CashIn.html
Request Method: GET

### Parameters:

| Name       | Type   | Required | Description                   |
| ---------- | ------ | -------- | ----------------------------- |
| app_id     | string | true     | Merchant app_id               |
| order_no   | string | true     | Merchant order number         |
| amount     | string | true     | Merchant amount (unit: Philippine Peso)  |
| notice_url | string | false    | Asynchronous notification URL |
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
