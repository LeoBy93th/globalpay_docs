# 1. Integration Process

> 1. Conduct business negotiations to open an account and discuss the relevant fee rates.
>
> 2. Contact operations to create the merchant number, secret key, merchant appId, product code, and apiUrl.
>
> 3. Once development is complete, both parties conduct joint debugging and testing to verify that request and reporting information is complete.

# 2. MD5 Signature Algorithm

> 1. Sort all parameters in ascending order as key-value pairs (key1=value1). Parameters with empty values do not participate in the signature.
>
> 2. Concatenate them in the format: key1=value1&key2=value2
>
> 3. Append the merchant secret key: key1=value1&key2=value2...&key=MerchantSecretKey
>
> 4. sign = md5(string assembled in the previous step). The signature result is a 32-character lowercase string.
>
> 5. The signing key can be found in the merchant backend under Basic Information, or by contacting our customer service.

# 3. Notes

## 3.1 API-Related

> 1. All interfaces in this document use the HTTP standard communication protocol with POST submissions. The Content-Type for both requests and responses is `application/json`, and the character encoding is UTF-8 throughout.
>
> 2. The currency unit is <span style="color:red;"> Indonesian Rupiah (Sen) </span>.
>
> 3. The IP address used to call the API must be whitelisted.
>
> 4. For `user_ip`, please collect the user's real IP address as much as possible. If unavailable, leave it blank. Do not use local IPs such as 127.0.0.1.

## 3.2 Callback-Related

> 1. If the callback is received and processed successfully, please return the text <span style="color:red;">`success`</span> with no other characters. The system will then stop pushing notifications for this order. Otherwise, it will continue to push multiple times.
>
> 2. During asynchronous notification interactions, if the response received is not `success`, the notification is considered failed. The system will periodically retry using the following interval schedule: 1m, 1m, 4m, 10m, 10m, 1h, 2h, 6h, 15h.
>
> 3. If `pay_notice_url` is empty, it will be treated as the merchant not requiring a callback, and the system will not push any notification.

# 4. Payment Collection (Pay-In) Order API

(The order IP must be whitelisted by us. Please contact us.)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayInOrder`

## 4.1 Pay-In – Order Request Parameters

| Name             | Type   | Required | Description                                                                                                                                     |
|----------------|------|-----|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| trade_no       | int    | true  | Merchant number                                                                                                                                       |
| app_id         | int    | true  | Merchant appId                                                                                                                                        |
| pay_code       | int    | true  | Product code. Contact our operations team to obtain.                                                                                                  |
| pay_method     | string    | true  | Payment method                                                                                                                                        |
| price          | int    | true  | Order amount, unit: Sen, integer type. `1 Indonesian Rupiah (IDR) = 100 Sen`                                                                                      |
| order_no       | string | true  | Merchant order number                                                                                                                                 |
| success_url    | string | false | Redirect URL on payment success                                                                                                                       |
| fail_url       | string | false | Redirect URL on payment failure                                                                                                                       |
| pay_notice_url | string | false | Payment success notification URL                                                                                                                      |
| user_id        | string | false | Merchant user ID                                                                                                                                      |
| user_ip        | string | false | Payer's IP address                                                                                                                                    |
| attach         | string | true  | Additional parameters as a JSON string with payer information: `{"account_no":"ID number","name":"Full name","bank_code":"Pay","phone":"3211234567"}` |
| sign           | string | true  | Signature result. See signature method at the top of this document.                                                                                   |
| timestamp      | string | false | Order timestamp. 10-digit Unix timestamp in seconds (S).                                                                                              |

- Pay-In – `attach` Additional Parameter Field Description

```json
{"name":"Leo Raj","account_no":"1234456789","bank_code":"Pay","phone":"3211234567"}
```

| Name         | Type   | Required | Description                                  |
|-------------|--------|------|----------------|
| name        | string | true | Payer's full name                            |
| phone       | string | true | Payer's mobile number                        |
| account_no  | string | true | ID document number                           |
| bank_code   | string | true | Bank code. See the Pay-In bank code list below. |

- Pay-In – Order Request Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025RAprmNz7lR",
  "app_id": 10002,
  "pay_code": 0,
  "price": 10099,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"name\":\"Zhang San\",\"account_no\":\"1234567890\",\"bank_code\":\"Pay\",\"phone\":\"3211234567\"}",
  "sign": "3d6dea05a7c08564911b9922e16455c2",
  "user_ip": "87.200.59.100",
  "success_url": "",
  "fail_url": "",
  "user_id": "2677343"
}
```

## 4.2 Pay-In – Order Response

| Name         | Type   | Required | Description                                                                                                                                                                                                                                                |
|------------|------|----|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code         | int    | true | 200: Order placed successfully. Other values: Order failed.                                                                                                                                       |
| msg          | string | true | Reason for failure                                                                                                                                                                                |
| pay_url      | string | false | Payment link                                                                                                                                                                                      |
| qr_code      | string | false | QR code string                                                                                                                                                                                    |
| order_no     | string | true | Merchant order number                                                                                                                                                                             |
| dis_order_no | string | true | Platform order number                                                                                                                                                                             |
| create_time  | int    | true | Creation time                                                                                                                                                                                     |
| pay_info     | string | false | Payment information JSON string, for example: native payment info, redirect URL, payer account, name, bank, etc. `{"pay_raw":"Native payment info","redirect_url":"Payment redirect URL"}` |
| sign         | string | true | Signature result. See signature method at the top of this document.                                                                                                                               |

- Pay-In – Order Response Example

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
  "dis_order_no": "2025071130770572062498816idr1oushe",
  "create_time": 1752825512,
  "pay_url": "https://{api_domain}/checkout/scanqr/943543da169d4757a40bfa49b3eb83b5"
}
```

# 5. Pay-In Callback Notification  (POST/JSON)

Push address: The `pay_notice_url` provided by the merchant at order placement. Callback IP: call_back_server_ip. Please add our IP to your callback whitelist.

## 5.1 Pay-In Callback – Request Parameters

| Name         | Type   | Required | Description                                                                                                                                                                                                                         |
|------------|------|-----|--------------------------------------------------------------------------------------------------------|
| trade_no     | int    | true  | Merchant number                                                                                                                                                                                                                     |
| status       | int    | true  | Order status: <span style="color:red;">2 = Success</span>, 3 = Failed                                                                                                                                                                                               |
| order_no     | string | true  | Merchant order number                                                                                                                                                                                                               |
| dis_order_no | string | true  | Platform order number                                                                                                                                                                                                               |
| order_price  | int    | true  | Order amount, unit: Sen                                                                                                                                                                                                             |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">Actual amount paid by the user, unit: Sen</span>                                                                                                                                                                                           |
| nti_time     | int    | false | Time the notification was initiated                                                                                                                                                                                                 |
| payer        | string | false | JSON string with payer information: `{"name":"Full name","account_no":"Account","account_type":"Account type","bank_code":"Payer bank or wallet code","utr2":"Bank reference number","email":"Email","phone":"Phone number"}`. In addition to the example fields, this parameter may also include payer-related fields passed by the merchant in `attach`. |
| pay_info     | string | false | Payment information JSON string, for example: native payment info, redirect URL, payer account, name, bank, etc.                                                                                                                     |
| create_time  | int    | true  | Creation time                                                                                                                                                                                                                       |
| sign         | string | true  | Signature result. See signature method at the top of this document.                                                                                                                                                                 |

- Pay-In Callback – Request Parameter Example

```json
{
  "trade_no": 10238,
  "status": 2,
  "order_no": "RCG0126042435820040804784",
  "dis_order_no": "Meg1352644s0800idrnVMR",
  "order_price": 10000,
  "real_price": 10000,
  "nti_time": 1776680622,
  "payer": "{\"account_no\":\"081234567890\",\"account_type\":\"PHONE\",\"bank_code\":\"DANA\",\"req_api_ip\":\"\",\"utr2\":\"9901108391\"}",
  "create_time": 1776680593,
  "sign": "7b23565a3dc790b6e55f29f0f0cf5f1a"
}
```

## 5.2 Pay-In Callback – Response Description

If the callback is received and processed successfully, please return <span style="color:red;">`success`</span>. The system will then stop pushing notifications for this order. Otherwise, it will continue to push multiple times.

# 6. Payout (Pay-Out) Order API

(The order IP must be whitelisted by us. Please contact us.)
Order URL: `https://{api_domain}/api/v1/payApi/CreatePayOutOrder`

## 6.1 Pay-Out – Request Parameters

<span style="color:red;">A pay-out request is considered successful unless an explicit failure is returned, even if it times out. Do not submit the request again unless an explicit failure is returned.</span>

| Name             | Type   | Required | Description                                                                 |
|----------------|------|-------|------------------------------------------------------|
| trade_no       | int    | true  | Merchant number                                                             |
| order_no       | string | true  | Merchant order number                                                       |
| app_id         | int    | true  | Merchant appId                                                              |
| pay_code       | int    | true  | Product code. Contact our operations team to obtain.                        |
| price          | int    | true  | Order amount, unit: Sen, integer type. `1 Indonesian Rupiah (IDR) = 100 Sen` |
| account_no     | string | true  | Payee account number                                                        |
| account_type   | string | true  | Account type: PHONE (e-wallet mobile number), BANK (bank account)           |
| account_name   | string | true  | Full name                                                                   |
| bank_code      | string | true  | Payee bank / wallet code. Refer to the Pay-Out bank code list below.        |
| pay_notice_url | string | false | Payout success notification URL                                             |
| attach         | string | true  | Additional parameters: `{"email":"email address","phone":"phone number","pay_type":"payment method"}` |
| user_ip        | string | false | Payee's IP address                                                          |
| sign           | string | true  | Signature result. See signature method at the top of this document.         |
| timestamp      | string | false | Order timestamp. 10-digit Unix timestamp in seconds (S).                    |

- Pay-Out – `attach` Additional Parameter Field Description

```json
{"email":"email address","phone":"phone number","pay_type":"payment method"}
```

| Name     | Type   | Required | Description                                           |
|----------|--------|------|----------------|
| phone    | string | true | Payee's mobile number                                 |
| email    | string | true | Email address                                         |
| pay_type | string | true | Payment method. See the Pay-Out payment method list below. |

- Pay-Out – Request Parameter Example

```json
{
  "trade_no": 10003,
  "order_no": "p7158412025MsJydJqT7b",
  "app_id": 10002,
  "pay_code": 1,
  "price": 10001,
  "pay_notice_url": "http://host/api/v1/mer/cbtest",
  "attach": "{\"email\":\"1392@qq.com\",\"phone\":\"3211234567\",\"pay_type\":\"IDRDANA\"}",
  "sign": "12f74d71fa929087af79b5083567c453",
  "user_ip": "87.200.59.100",
  "account_type": "PHONE",
  "account_no": "123456789",
  "account_name": "test",
  "bank_code": "DANA"
}
```

## 6.2 Pay-Out – Order Response

| Name         | Type   | Required | Description                                                                              |
|------------|------|----|---------------------------------------------|
| code         | int    | true | 200: Order placed successfully. Other values: Order failed.                              |
| msg          | string | true | Reason for failure                                                                       |
| dis_order_no | string | true | Platform order number                                                                    |
| order_no     | string | true | Merchant order number                                                                    |
| status       | int    | true | Order status: 2 = Payout successful, 3 = Payout failed, 7 = Rejected, 9 = Reversal, 10 = Processing |
| create_time  | int    | true | Creation time                                                                            |
| sign         | string | true | Signature result. See signature method at the top of this document.                      |

- Pay-Out – Order Response Example

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
  "dis_order_no": "2025071130776296733810688idr1Dhr7H",
  "create_time": 1752826877,
  "status": 10
}
```

# 7. Payout (Pay-Out) Callback Notification

Push address: The `pay_notice_url` provided by the merchant at order placement. Callback IP: call_back_server_ip. Please add our IP to your callback whitelist.

## 7.1 Pay-Out Callback – Request Parameters

| Name         | Type   | Required | Description                                                                                 |
|------------|------|-----|-----------------------------------------------|
| trade_no     | int    | true  | Merchant number                                                                             |
| order_no     | string | true  | Merchant order number                                                                       |
| dis_order_no | string | true  | Platform order number                                                                       |
| order_price  | int    | true  | Order amount, unit: Sen                                                                     |
| fee          | int    | false | Order handling fee, unit: Sen                                                               |
| <span style="color:red;">real_price</span>   | int    | false | <span style="color:red;">Actual payout amount (only available when the payout succeeds). To use the real_price field, please contact our staff to configure it.</span>                                                                                       |
| status       | int    | true  | Order status: <span style="color:red;">2 = Payout successful</span>, 3 = Payout failed, 7 = Rejected, 9 = Reversal         |
| pay_info     | string | false | Payment information JSON string. Examples: raw payment info, card number, name, bank, utr2, etc. |
| remark       | string | false | Reason for failure                                                                          |
| create_time  | int    | true  | Creation time                                                                               |
| sign         | string | true  | Signature result. See signature method at the top of this document.                         |
| nti_time     | int    | true  | Time the notification was initiated                                                         |

- Pay-Out Callback – Request Example

```json
{
  "trade_no": 10000,
  "status": 2,
  "order_no": "20060354339090013",
  "dis_order_no": "Meg2352644o2nmjo0800idrYZ2A",
  "order_price": 11000,
  "nti_time": 1776665229,
  "create_time": 1776665034,
  "sign": "d2f74c18dca3bd6bd79172a1a7c26d9a",
  "pay_info": "{\"utr2\":\"611011445289\"}"
}
```

## 7.2 Pay-Out Callback – Response Description

If the callback is received and processed successfully, please return <span style="color:red;">`success`</span>. The system will then stop pushing notifications for this order. Otherwise, it will continue to push multiple times.

# 8. Order Query API (Shared by Pay-In and Pay-Out)

(The request IP must be whitelisted by us. Please contact us.)
Query URL: `https://{api_domain}/api/v1/payApi/QueryOrder`

## 8.1 Query Request Parameters

| Name         | Type   | Required | Description                                            |
|------------|------|----|---------------------------|
| order_type   | string | true | `pay_out`: Payout, `pay_in`: Pay-In                    |
| trade_no     | int    | true | Merchant number                                        |
| app_id       | int    | true | Merchant appId                                         |
| dis_order_no | string | false | Platform order number                                 |
| order_no     | string | false | Merchant order number                                 |
| sign         | string | true | Signature result. See signature method at the top of this document. |

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

| Name         | Type   | Required | Description                                                                                                                                                    |
|------------|------|-----|---------------------------------------------------------------------------------------------------------------------------------------------------|
| code         | int    | true  | 200: Query successful. Other values: Failed.                                                                                                                   |
| msg          | string | true  | Reason for query failure                                                                                                                                       |
| trade_no     | int    | true  | Merchant number                                                                                                                                               |
| <span style="color:red;">real_price</span>   | int    | true  | <span style="color:red;">Actual amount paid, unit: Sen</span>                                                                                                                                 |
| status       | int    | true  | Order status: 1 = Unpaid, <span style="color:red;">2 = Success</span>, 3 = Failed, 7 = Rejected, 9 = Reversal, 10 = Processing                                                               |
| success_time | int    | true  | Success timestamp                                                                                                                                             |
| order_no     | string | true  | Merchant order number                                                                                                                                         |
| dis_order_no | string | true  | Platform order number                                                                                                                                         |
| remark       | string | true  | Reason for payout failure                                                                                                                                     |
| fee          | int    | false | Order handling fee, unit: Sen                                                                                                                                 |
| create_time  | int    | true  | Creation time                                                                                                                                                 |
| payer        | string | false | JSON string with payer information: `{"account_name":"Full name","account_type":"Account type: PHONE,BANK","account_no":"Account","bank_code":"Bank or wallet code"}` |
| pay_info     | string | false | Payment information JSON string, for example: native payment info, redirect URL, payer account, name, bank, etc.                                               |
| sign         | string | true  | Signature result. See signature method at the top of this document.                                                                                           |
| utr2         | string | false | Bank order number                                                                                                                                             |

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
  "payer": "{\"account_name\":\"Full name\",\"account_type\":\"PHONE\",\"account_no\":\"081234567890\",\"bank_code\":\"DANA\"}",
  "sign": "db3406277185f9660b3b928d6adc7bc4"
}
```

# 9. Payout Balance Query API

(The request IP must be whitelisted by us. Please contact us.)
URL: `https://{api_domain}/api/v1/payApi/QueryBalance`

## 9.1 Balance Request Parameters

| Name     | Type   | Required | Description                                            |
|--------|------|----|----------------|
| trade_no | int    | true | Merchant number                                        |
| app_id   | int    | true | Merchant appId                                         |
| sign     | string | true | Signature result. See signature method at the top of this document. |

- Balance Request Example

```json
{
  "trade_no": 165,
  "app_id": 281,
  "sign": "db3406277185f9660b3b928d6adc115"
}
```

## 9.2 Balance Response

| Name           | Type   | Required | Description                                            |
|-------|------|----|---------------------------|
| code           | int    | true | 200: Query successful. Other values: Failed.           |
| msg            | string | true | Reason for failure                                     |
| balance        | int    | true | Balance, unit: Sen                                     |
| balance_frozen | int    | false | Frozen balance, unit: Sen                              |
| sign           | string | true | Signature result. See signature method at the top of this document. |

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

# 10. Payment Voucher Query API

(The request IP must be whitelisted by us. Please contact us.)
URL: `https://{api_domain}/api/v1/payApi/QueryCertificate`

## 10.1 Payment Voucher Request Parameters

| Name         | Type   | Required | Description                                                              |
|--------|------|----|----------------|
| trade_no     | int    | true  | Merchant number                                                          |
| app_id       | int    | true  | Merchant appId                                                           |
| order_no     | string | false | Merchant order number. Use either this or `dis_order_no`.                |
| dis_order_no | string | false | Platform order number. Use either this or `order_no`.                    |
| sign         | string | true  | Signature result. See signature method at the top of this document.      |

- Payment Voucher Request Example

```json
{
  "trade_no": 10003,
  "app_id": 10003,
  "order_no": "",
  "dis_order_no": "35011C02gljuf6k0800idr1lVY",
  "sign": "3969f17cd1a551769f85967d0a05b7b6"
}
```

## 10.2 Payment Voucher Response Parameters

| Name      | Type   | Required | Description                                            |
|-------|------|----|---------------------------|
| code      | int    | true | 200: Response successful. Other values: Failed.        |
| msg       | string | true | Reason for failure                                     |
| img_link  | string | false | Voucher link                                           |
| img_base  | string | false | Voucher Base64-encoded image data                      |
| sign      | string | true | Signature result. See signature method at the top of this document. |

- Payment Voucher Response Example

  No payment voucher available:
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

# 11. Pay-In Payment Methods – Field `pay_method`

| Field      | Value      | Description  |
|-----------|-------|----------------|
| pay_method | QRIS_IDR | Qris           |
| pay_method | DANA_IDR | DANA           |
| pay_method |  VA_IDR   | VA   |
| pay_method |  IDROVO| OVO wallet |
| pay_method |   IDRDANA   | DANA wallet |
| pay_method |  IDRLINKAJA| LINKAJA wallet |
| pay_method |   IDRSHOPEEPAY   | SHOPEEPAY wallet |
| pay_method |  IDRGOPAY| GOPAY wallet|

# 12. Pay-In Bank Codes – Field `attach.bank_code`

| Field Name | Code                       | Bank Name |
|:---------|:---------------------------|:---------|
| bank_code | MEGA                       | Bank Mega |
| bank_code | BANK_SRI_PARTHA            | Bank Sri Partha |
| bank_code | PRIMA_MASTER               | Prima Master Bank |
| bank_code | OVO                        | OVO E-Wallet |
| bank_code | BTPN_SYARIAH               | Bank Purba Danarta |
| bank_code | CENTRATAMA                 | Centratama Nasional Bank |
| bank_code | PANIN                      | Bank Panin |
| bank_code | LINKAJA                    | LINKAJA E-Wallet |
| bank_code | BANK_AGRIS                 | Bank Agris |
| bank_code | BNC                        | Bank BNC |
| bank_code | NATIONALNOBU               | Bank Alfindo (Bank National Nobu) |
| bank_code | ARTHA                      | Bank Artha Graha Internasional |
| bank_code | JENIUS                     | JENIUS |
| bank_code | BNI                        | Bank BNI |
| bank_code | BANK_JATENG                | Bank Jateng |
| bank_code | BPD_KALTENG                | Bank Kalteng |
| bank_code | BANK_OF_INDIA              | Bank of India Indonesia |
| bank_code | BANK_AKITA                 | Bank Akita |
| bank_code | DANA                       | DANA E-Wallet |
| bank_code | BANK_ING                   | ING Indonesia Bank |
| bank_code | BNP_PARIBAS                | Bank BNP Paribas Indonesia |
| bank_code | BJB_SYR                    | Bank BJB Syariah |
| bank_code | OCBC                       | Bank Dipo International (Bank Sahabat Sampoerna) |
| bank_code | ARTOS                      | Bank Artos IND |
| bank_code | MITSUI                     | Bank Sumitomo Mitsui Indonesia |
| bank_code | DKI                        | Bank DKI |
| bank_code | DOMPETKU                   | Indosat Dompetku |
| bank_code | BAML                       | Bank of America, N.A |
| bank_code | IDRCAPITAL                 | Bank Capital Indonesia |
| bank_code | BANK_EKONOMI               | Bank Ekonomi |
| bank_code | BANK_SULTRA                | Bank Sultra |
| bank_code | CCB                        | Bank Windu Kentjana |
| bank_code | BCA_SYR                    | Bank BCA Syariah |
| bank_code | MULTI_ARTA_SENTOSA         | Bank Multi Arta Sentosa |
| bank_code | BANK_YUDHA                 | Bank Yudha Bhakti / BANK NEO COMMERCE |
| bank_code | BANK_INDOMONEX             | Bank Indomonex (Bank SBI Indonesia) |
| bank_code | BANK_KESEJAHTERAAN_EKONOMI | Bank Kesejahteraan Ekonomi (Bank Seabank Indonesia) |
| bank_code | BANK_DANAMON               | Korea Exchange Bank Danamon |
| bank_code | BANK_JATIM                 | Bank Jatim |
| bank_code | MALUKU                     | Bank Maluku Malut |
| bank_code | CHINATRUST                 | Bank CTBC (China Trust) Indonesia |
| bank_code | MANDIRI                    | Bank Mandiri |
| bank_code | BANK_LIPPO                 | Bank Lippo |
| bank_code | DAERAH_ISTIMEWA            | BPD DIY |
| bank_code | SINARMAS_UUS               | Bank Sinarmas |
| bank_code | MASPION                    | Bank Maspion Indonesia |
| bank_code | AGRONIAGA                  | Bank BRI Agro |
| bank_code | BANK_TOKYO                 | The Bank of Tokyo Mitsubishi UFJ LTD |
| bank_code | RESONA                     | Bank Resona Perdania |
| bank_code | MAYAPADA                   | Bank Mayapada |
| bank_code | BANK_HAGAKITA              | Bank Hagakita |
| bank_code | JAMBI                      | BPD Jambi |
| bank_code | FAMA                       | Bank Fama Internasional |
| bank_code | KALIMANTAN_BARAT           | Bank Kalimantan Barat |
| bank_code | MEGA_SYR                   | Bank Syariah Mega |
| bank_code | BANK_JABAR                 | Bank Jabar dan Banten (BJB) |
| bank_code | BRI                        | Bank BRI |
| bank_code | BANK_PERSY                 | Bank Persyarikatan Indonesia |
| bank_code | BANK_IFI                   | Bank IFI |
| bank_code | SUMSEL_DAN_BABEL_SULUT     | Bank Sulut Gorontalo |
| bank_code | BISNIS_INTERNASIONAL       | Bank Bisnis Internasional |
| bank_code | HARDA_INTERNASIONAL        | Bank Harda |
| bank_code | GOPAY                      | GOPAY E-Wallet |
| bank_code | DBS                        | Bank DBS Indonesia |
| bank_code | BANK_ANTAR                 | Bank Antardaerah |
| bank_code | BALI                       | BPD Bali |
| bank_code | EXIMBANK                   | Bank Ekspor Indonesia |
| bank_code | RIAU_DAN_KEPRI             | Bank Riau |
| bank_code | JASA_JAKARTA               | Bank Jasa Jakarta |
| bank_code | BANK_LIMAN                 | Liman International Bank |
| bank_code | CITIBANK                   | Citibank |
| bank_code | PERMATA                    | Permata Bank |
| bank_code | BANK_C_AGR                 | Bank Credit Agricole Indosuez |
| bank_code | NUSANTARA_PARAHYANGAN      | Bank Nusantara Parahyangan |
| bank_code | BANK_ANGLOMAS              | Anglomas Internasional Bank |
| bank_code | NISP                       | Bank OCBC NISP |
| bank_code | INDEX_SELINDO              | Bank Index Selindo |
| bank_code | MIZUHO                     | Bank Mizuho Indonesia |
| bank_code | LAMPUNG                    | Bank Lampung |
| bank_code | PAPUA                      | Bank Papua |
| bank_code | BANK_SWAGUNA               | Bank Swaguna |
| bank_code | ROYAL                      | Bank Royal Indonesia |
| bank_code | JPMORGAN                   | JP. Morgan Chase Bank, N.A |
| bank_code | BANK_ABN                   | Bank ABN Amro |
| bank_code | GANESHA                    | Bank Ganesha |
| bank_code | BANK_HIM                   | Bank Himpunan Saudara 1906 |
| bank_code | BANK_MERIN                 | Bank Merincorp |
| bank_code | DANAMON                    | Bank Danamon |
| bank_code | DEUTSCHE                   | Deutsche Bank AG. |
| bank_code | BUMI_ARTA                  | Bank Bumi Arta |
| bank_code | MESTIKA_DHARMA             | Bank Mestika Dharma |
| bank_code | QNB_INDONESIA              | Bank QNB Kesawan (Bank QNB Indonesia) |
| bank_code | BANK_HARFA                 | Bank Harfa |
| bank_code | BANK_BUANA                 | Bank UOB Indonesia |
| bank_code | STANDARD_CHARTERED         | Standard Chartered Bank |
| bank_code | BANK_KEPPEL                | Bank Keppel Tatlee Buana |
| bank_code | SULAWESI                   | Bank Sulawesi Tengah |
| bank_code | BANK_HARM                  | Bank Harmoni International |
| bank_code | BANK_COMP                  | The Bangkok Bank Comp. LTD |
| bank_code | BANK_HAGA                  | Bank Haga |
| bank_code | MANDIRI_TASPEN             | Bank Mandiri Taspen Pos |
| bank_code | BANK_WOOR                  | Bank Woori Indonesia |
| bank_code | BANK_NTT                   | Bank NTT |
| bank_code | BANK_BINTANG               | Bank Bintang Manunggal |
| bank_code | BANK_BUMIPUTERA            | Bank MNC / Bank Bumiputera |
| bank_code | BANK_INA                   | Bank Ina Perdana |
| bank_code | BANK_MAYBANK               | Bank Maybank Indocorp |
| bank_code | BTN                        | Bank Tabungan Negara (BTN) |
| bank_code | BCA                        | Bank BCA |
| bank_code | SHOPEEPAY                  | SHOPEEPAY E-Wallet |
| bank_code | BANK_ANZ                   | Bank ANZ Indonesia |
| bank_code | IDRBOC                     | Bank OF China |
| bank_code | BUKOPIN                    | Bank Bukopin |
| bank_code | BPR_KS                     | BPR KS |
| bank_code | JTRUST                     | Bank JTRUST |
| bank_code | MAYORA                     | Bank Mayora Indonesia |
| bank_code | MANDIRI_SYR                | Bank Syariah Mandiri (BSI) Bank Syariah Indonesia |


# 13. Pay-Out Payment Methods – Field `attach.pay_type`

| Field    | Value         | Description            |
|-----------|------------------|------------------------|
| pay_type | IDRPAY           | PAY public             |
| pay_type | IDROVO           | OVO wallet             |
| pay_type | IDRDANA          | DANA wallet            |
| pay_type | IDRLINKAJA       | LINKAJA wallet         |
| pay_type | IDRSHOPEEPAY     | SHOPEEPAY wallet       |
| pay_type | IDRGOPAY         | GOPAY wallet           |
| pay_type | VA_IDR        | VA_IDR           |

# 14. Pay-Out Bank Codes – Field `attach.bank_code`

| Field Name | Code                       | Bank Name |
|:---------|:---------------------------|:---------|
| bank_code | MEGA                       | Bank Mega |
| bank_code | BANK_SRI_PARTHA            | Bank Sri Partha |
| bank_code | PRIMA_MASTER               | Prima Master Bank |
| bank_code | OVO                        | OVO E-Wallet |
| bank_code | BTPN_SYARIAH               | Bank Purba Danarta |
| bank_code | CENTRATAMA                 | Centratama Nasional Bank |
| bank_code | PANIN                      | Bank Panin |
| bank_code | LINKAJA                    | LINKAJA E-Wallet |
| bank_code | BANK_AGRIS                 | Bank Agris |
| bank_code | BNC                        | Bank BNC |
| bank_code | NATIONALNOBU               | Bank Alfindo (Bank National Nobu) |
| bank_code | ARTHA                      | Bank Artha Graha Internasional |
| bank_code | JENIUS                     | JENIUS |
| bank_code | BNI                        | Bank BNI |
| bank_code | BANK_JATENG                | Bank Jateng |
| bank_code | BPD_KALTENG                | Bank Kalteng |
| bank_code | BANK_OF_INDIA              | Bank of India Indonesia |
| bank_code | BANK_AKITA                 | Bank Akita |
| bank_code | DANA                       | DANA E-Wallet |
| bank_code | BANK_ING                   | ING Indonesia Bank |
| bank_code | BNP_PARIBAS                | Bank BNP Paribas Indonesia |
| bank_code | BJB_SYR                    | Bank BJB Syariah |
| bank_code | OCBC                       | Bank Dipo International (Bank Sahabat Sampoerna) |
| bank_code | ARTOS                      | Bank Artos IND |
| bank_code | MITSUI                     | Bank Sumitomo Mitsui Indonesia |
| bank_code | DKI                        | Bank DKI |
| bank_code | DOMPETKU                   | Indosat Dompetku |
| bank_code | BAML                       | Bank of America, N.A |
| bank_code | IDRCAPITAL                 | Bank Capital Indonesia |
| bank_code | BANK_EKONOMI               | Bank Ekonomi |
| bank_code | BANK_SULTRA                | Bank Sultra |
| bank_code | CCB                        | Bank Windu Kentjana |
| bank_code | BCA_SYR                    | Bank BCA Syariah |
| bank_code | MULTI_ARTA_SENTOSA         | Bank Multi Arta Sentosa |
| bank_code | BANK_YUDHA                 | Bank Yudha Bhakti / BANK NEO COMMERCE |
| bank_code | BANK_INDOMONEX             | Bank Indomonex (Bank SBI Indonesia) |
| bank_code | BANK_KESEJAHTERAAN_EKONOMI | Bank Kesejahteraan Ekonomi (Bank Seabank Indonesia) |
| bank_code | BANK_DANAMON               | Korea Exchange Bank Danamon |
| bank_code | BANK_JATIM                 | Bank Jatim |
| bank_code | MALUKU                     | Bank Maluku Malut |
| bank_code | CHINATRUST                 | Bank CTBC (China Trust) Indonesia |
| bank_code | MANDIRI                    | Bank Mandiri |
| bank_code | BANK_LIPPO                 | Bank Lippo |
| bank_code | DAERAH_ISTIMEWA            | BPD DIY |
| bank_code | SINARMAS_UUS               | Bank Sinarmas |
| bank_code | MASPION                    | Bank Maspion Indonesia |
| bank_code | AGRONIAGA                  | Bank BRI Agro |
| bank_code | BANK_TOKYO                 | The Bank of Tokyo Mitsubishi UFJ LTD |
| bank_code | RESONA                     | Bank Resona Perdania |
| bank_code | MAYAPADA                   | Bank Mayapada |
| bank_code | BANK_HAGAKITA              | Bank Hagakita |
| bank_code | JAMBI                      | BPD Jambi |
| bank_code | FAMA                       | Bank Fama Internasional |
| bank_code | KALIMANTAN_BARAT           | Bank Kalimantan Barat |
| bank_code | MEGA_SYR                   | Bank Syariah Mega |
| bank_code | BANK_JABAR                 | Bank Jabar dan Banten (BJB) |
| bank_code | BRI                        | Bank BRI |
| bank_code | BANK_PERSY                 | Bank Persyarikatan Indonesia |
| bank_code | BANK_IFI                   | Bank IFI |
| bank_code | SUMSEL_DAN_BABEL_SULUT     | Bank Sulut Gorontalo |
| bank_code | BISNIS_INTERNASIONAL       | Bank Bisnis Internasional |
| bank_code | HARDA_INTERNASIONAL        | Bank Harda |
| bank_code | GOPAY                      | GOPAY E-Wallet |
| bank_code | DBS                        | Bank DBS Indonesia |
| bank_code | BANK_ANTAR                 | Bank Antardaerah |
| bank_code | BALI                       | BPD Bali |
| bank_code | EXIMBANK                   | Bank Ekspor Indonesia |
| bank_code | RIAU_DAN_KEPRI             | Bank Riau |
| bank_code | JASA_JAKARTA               | Bank Jasa Jakarta |
| bank_code | BANK_LIMAN                 | Liman International Bank |
| bank_code | CITIBANK                   | Citibank |
| bank_code | PERMATA                    | Permata Bank |
| bank_code | BANK_C_AGR                 | Bank Credit Agricole Indosuez |
| bank_code | NUSANTARA_PARAHYANGAN      | Bank Nusantara Parahyangan |
| bank_code | BANK_ANGLOMAS              | Anglomas Internasional Bank |
| bank_code | NISP                       | Bank OCBC NISP |
| bank_code | INDEX_SELINDO              | Bank Index Selindo |
| bank_code | MIZUHO                     | Bank Mizuho Indonesia |
| bank_code | LAMPUNG                    | Bank Lampung |
| bank_code | PAPUA                      | Bank Papua |
| bank_code | BANK_SWAGUNA               | Bank Swaguna |
| bank_code | ROYAL                      | Bank Royal Indonesia |
| bank_code | JPMORGAN                   | JP. Morgan Chase Bank, N.A |
| bank_code | BANK_ABN                   | Bank ABN Amro |
| bank_code | GANESHA                    | Bank Ganesha |
| bank_code | BANK_HIM                   | Bank Himpunan Saudara 1906 |
| bank_code | BANK_MERIN                 | Bank Merincorp |
| bank_code | DANAMON                    | Bank Danamon |
| bank_code | DEUTSCHE                   | Deutsche Bank AG. |
| bank_code | BUMI_ARTA                  | Bank Bumi Arta |
| bank_code | MESTIKA_DHARMA             | Bank Mestika Dharma |
| bank_code | QNB_INDONESIA              | Bank QNB Kesawan (Bank QNB Indonesia) |
| bank_code | BANK_HARFA                 | Bank Harfa |
| bank_code | BANK_BUANA                 | Bank UOB Indonesia |
| bank_code | STANDARD_CHARTERED         | Standard Chartered Bank |
| bank_code | BANK_KEPPEL                | Bank Keppel Tatlee Buana |
| bank_code | SULAWESI                   | Bank Sulawesi Tengah |
| bank_code | BANK_HARM                  | Bank Harmoni International |
| bank_code | BANK_COMP                  | The Bangkok Bank Comp. LTD |
| bank_code | BANK_HAGA                  | Bank Haga |
| bank_code | MANDIRI_TASPEN             | Bank Mandiri Taspen Pos |
| bank_code | BANK_WOOR                  | Bank Woori Indonesia |
| bank_code | BANK_NTT                   | Bank NTT |
| bank_code | BANK_BINTANG               | Bank Bintang Manunggal |
| bank_code | BANK_BUMIPUTERA            | Bank MNC / Bank Bumiputera |
| bank_code | BANK_INA                   | Bank Ina Perdana |
| bank_code | BANK_MAYBANK               | Bank Maybank Indocorp |
| bank_code | BTN                        | Bank Tabungan Negara (BTN) |
| bank_code | BCA                        | Bank BCA |
| bank_code | SHOPEEPAY                  | SHOPEEPAY E-Wallet |
| bank_code | BANK_ANZ                   | Bank ANZ Indonesia |
| bank_code | IDRBOC                     | Bank OF China |
| bank_code | BUKOPIN                    | Bank Bukopin |
| bank_code | BPR_KS                     | BPR KS |
| bank_code | JTRUST                     | Bank JTRUST |
| bank_code | MAYORA                     | Bank Mayora Indonesia |
| bank_code | MANDIRI_SYR                | Bank Syariah Mandiri (BSI) Bank Syariah Indonesia |

# 15. Error Codes

| Status Code | Description                                                                                                                 |
|------|-------------------|
| 200  | Success                                                                                                                     |
| 1000 | Internal error                                                                                                              |
| 1001 | IP is not in the merchant IP whitelist                                                                                      |
| 1002 | Parameter error                                                                                                             |
| 1003 | Signature error                                                                                                             |
| 1004 | This interface is not currently open for the merchant (Contact operations to verify: merchant or App does not exist, has been closed, or payment product is not configured) |
| 1005 | Merchant does not exist                                                                                                     |
| 1006 | The current user IP is on the blacklist                                                                                     |
| 1007 | The current user is on the blacklist                                                                                        |
| 1008 | Merchant App does not exist                                                                                                 |
| 1009 | Payment product does not exist                                                                                              |
| 1010 | Payment channel does not exist                                                                                              |
| 1011 | Payment channel is not yet fully developed and is temporarily unavailable                                                   |
| 1012 | Payment channel is abnormal. Please try again later.                                                                        |
| 1013 | Current order volume is too high. Please try again later.                                                                   |
| 1014 | Duplicate order number                                                                                                      |
| 1015 | Insufficient app balance                                                                                                    |
| 1016 | The same user is placing orders too frequently. Please try again later.                                                     |
| 1017 | Order record does not exist                                                                                                 |
| 1018 | The current amount is not supported                                                                                         |
| 1019 | Pay-In orders are not yet enabled for the country where the current app is located                                          |
| 1020 | Pay-Out orders are not yet enabled for the country where the current app is located                                         |
| 1021 | Failed                                                                                                                      |
| 1036 | Interface is not yet open                                                                                                   |
| 1037 | Currency not supported                                                                                                      |
| 1038 | Abnormal UTR returned for pay-in callback                                                                                   |
| 9999 | Other error                                                                                                                 |
| 3000 | System is under maintenance. Order placement is suspended. Please try again later.                                          |

# 16. Pay-In Cashier API

Address: https://{api_domain}/api/v1/cashApi/CashIn.html
Request Method: GET

### Parameters:

| Name    | Type  | Required | Description            |
|------------|--------|----------|------------------------------------|
| app_id   | string | true   | Merchant app_id.          |
| order_no  | string | true   | Merchant order number.       |
| amount   | string | true   | Merchant amount, unit: Indonesian Rupiah (`IDR`) |
| notice_url | string | false  | Asynchronous notification address. |
|pay_code|int|true|Product Code|

#### Example

```
https://{api_domain}/api/v1/cashApi/CashIn.html?app_id={{app_id}}&order_no={{MerchantOrderNumber}}&amount={{MerchantAmount}}&notice_url={{AsynchronousNotificationAddress}}&pay_code={{ProductCode}}
```

# 17. Document Update Time

```
2026-06-11 01:00:00
```
