# AEM License Access Developer Documentation	

Scheme for AEM wallet is: aem.

## Authorized Login

### app authorization login

#### 1, APP call AEM application authorization login 

The third-party app calls up the wallet app via SchemeURL and passes the following parameters using URL-encoded JSON

| KEY   | description               |
| ----- | ----------------------------- |
| type  | Pass login     Required |
| uuid  | The uuid of the app obtained when the access service is activated is required |
| callbackUrl | The callback address in schemeurl format, after schemeurl cannot be empty. Required. For example: appscheme://login |
| callbackType | When it is 0, return to openCode will be spliced by "?" or "&". When it is 1, openCode is directly spliced at the end of callbackUrl. Default 0 is not required |
| openCallBackType | It takes effect when platform is not specified or platform is mobile. When openCallBackType is 0 or empty, the wallet opens the callbackurl application. When openCallBackType is 1, the wallet initiates an http request to callbackurl and minimizes the wallet. Not required |
| platform | Turn up the wallet terminal. If it is called from a mobile phone, fill in mobile <br />If it is to scan a webpage QR code or h5 page, fill in web |

> Since IOS does not allow special symbols in the schema, JSON data needs to be url-encoded.
>
> It should be noted that the callbackurl itself, if there are special characters in the parameters need to be encoded in advance of the internal parameters of the url

Before URL encoding

```
aem://{"type":"login","uuid":"121212-121212-121212","callbackUrl":"appscheme://login/a/b?d=1&redirect=http%3A%2F%2Fwww.google.com%2Fs%3Fwd%3D12345"}
```

After URL encoding

```bash
aem://%7B%22type%22:%22login%22,%22uuid%22:%22121212-121212-121212%22,%22callbackUrl%22:%22appscheme://login/a/b?d=1&redirect=http%253A%252F%252Fwww.google.com%252Fs%253Fwd%253D12345%22%7D
```

#### 2、Get Address

​		After the user allows authorization in AEM, the wallet generates the authorization openCode and carries the openCode parameter to jump to the address of the callbackUrl.

> The openCode is valid for 1 minute and expires after one use.

After getting the openCode, request the following link to get the address and unique identifier

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| parameter name | description                                          |
| -------------- | ---------------------------------------------------- |
| openCode       | Fill in the openCode obtained in the first step      |
| accessToken    | AccessToken obtained when opening third-party access |
| appId          | Uuid obtained when opening third-party access        |

Return description

The JSON packet returned when correct is as follows
   Note: ‘id is a unique identification’

```json
{
    "result":{
        "address":"address",
        "id":"*********"
    },
    "success":true,
    "errorCode":""
}
```

**The error code errorCode is described as follows：**

| error code | description       |
| ---------- | ----------------- |
| 900001     | accessToken error |
| 010001     | unknown mistake   |

### WEB scan code authorization login

#### 1、Create a QR code

The access party needs to provide the URL QR code to provide AEM scan code login. URL parameters are as follows

| 参数名称 | description |Is it required|
| -------- | --------- |---------|
| type     | Fill in login |Yes|
| platform | Fill in the web |Yes|
| callbackUrl | Callback address brought into openCode after authorization |Yes|
| uuid | Uuid after the access party's application is passed |Yes|
| callbackType | When it is 0, return openCode and webToken will be spliced by "?" or "&". When it is 1, openCode and webToken are directly spliced at the end of callbackUrl. Default 0 |No|
| webToken | The access party verifies the authorized login user address |No|

E.g:

```shell
aemlogin://{"type": "login","platform": "web","callbackUrl": "https://www.google.com/login","uuid": "123-123-123","webToken": "SSFJIEKALSM"}
```

After the user confirms the authorization, the APP will bring back the authorized openCode and the webToken in the QR code to the callbackUrl parameter address.

> The validity period of openCode is 1 minute, and it becomes invalid after one use

#### 2、Get Address and unique identity

After obtaining openCode, request the following link to obtain the address

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| parameter name | description                                          |
| -------------- | ---------------------------------------------------- |
| openCode       | Fill in the openCode obtained in the first step      |
| accessToken    | AccessToken obtained when opening third-party access |
| appId          | Uuid obtained when opening third-party access        |

Return description

The JSON packet returned when correct is as follows

```json
{
    "result":{
    	"address":"address",
    	"id":"**********"
    },
    "success":true,
    "errorCode":"",
    "errorMsg":""
 }
```

**The error code errorCode is described as follows：**

| error code | description       |
| ---------- | ----------------- |
| 900001     | accessToken error |
| 010001     | unknown mistake   |



## Wallet payment

The third-party APP can call AEM payment to complete the order payment process.

### 1、Apply for user payment

#### 1.1APP call

The APP wallet invokes AEM through schemeurl and passes the following parameters through the URL-encoded JSON object

| parameter name | description                                              |
| ---------------- | ------------------------------------------------------------ |
| type             | Types. Fill in pay                              |
| appUuid             | uuid at the time of app application, which is required. |
| orderId          | Order id, which is required. |
| toAddr           | Recipient address, which is required.  |
| assetType        | Payment asset type, which is required. |
| amount           | Payment amount, which is required.    |
| callbackUrl      | The payment is successful AEM service notifies the background callback of the payment status, the host needs to be in the secure domain name, which is required. |
| callbackScheme   | The appschemeurl to be redirected back to after payment is complete，, which is required. |
| description             | Description, will be shown to users.      |
| signedContent    | Signature data. Sign the parameters with the rsa private key obtained during access, which is required. |
| openCallBackType | APP callback method after payment is completed. When it is 0, the phone opens the callbackScheme. When it is 1, no other apps will be opened, and only the wallet app will be minimized. This operation does not affect the payment status callback. |

E.g:

Before URL encoding

```
aem://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","callbackScheme": "payApp",
	"description": "*****","signedContent": "*******","openCallBackType": "1"}
```

> Note: amount is a string type

After URL encoding

```
aem://%7B%22type%22:%20%22pay%22,%22appUuid%22:%20%225555-6666-8888-9999%22,%22orderId%22:%20%222020092487855441%22,%22toAddr%22:%20%22*********************%22,%22assetType%22:%20%22AEM%22,%22amount%22:%20%220.001%22,%22callbackUrl%22:%20%22https://localhost:8080/pay?orderId=2020092487855441%22,%22callbackScheme%22:%20%22payApp%22,%0A%09%22description%22:%20%22*****%22,%22signedContent%22:%20%22*******%22,%22openCallBackType%22:%20%221%22%7D
```

#### 1.2QR code call

The access party needs to construct the URL QR code and scan the code by the wallet APP. The parameters use the json method, and the content is as follows

| parameter name | description                                                  |
| -------------- | ------------------------------------------------------------ |
| type           | Fill in pay                                                  |
| signedContent  | Signature data. Use the rsa private key obtained at the time of access to sign the parameters, which is required. |
| appUuid        | Appuuid obtained at the time of activation,must.             |
| orderId        | Third-party business system order number, which is required. |
| toAddr         | Receiving address, which is required.                        |
| assetType      | Payment asset type, which is required.                       |
| amount         | Payment amount, which is required.                           |
| callbackUrl    | The payment is successful. AEM will notify the background callback of the payment status. The host needs to be in the secure domain name, which is required. |
| description    | Description, will be shown to users。                        |

E.g:

```
aempay://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","description": "*****","signedContent": "*******",}
```



#### Signature algorithm

Use the SHA1WithRSA algorithm to sign the amount, appUuid, assetType, callbackUrl, orderId, and toAddr parameters using the rsaPrivate used in the application.

Suppose the parameters to be signed are as follows:

```json
{"amount":"amountVal","appUuid":"uuidVal","assetType":"assetTypeVal","callbackUrl":"callbackUrlVal","orderId":"orderIdVal","toAddr":"toAddrVal"}
```

##### 1.Arrange the parameters into a json structure in the order of amount, appUuid, assetType, callbackUrl, orderId, and toAddr.

##### 2.Use SHA1withRSA to sign the parameters, and the signed content is BASE64 encoded as rsaContent.

### 2、Payment completed

There are two ways to judge payment completion:

#### 2.1 Judging from the transaction record of the receiving address

​	For transactions authorized by AEM, metadata will be carried in this transaction, and the content is the order number. The authorized party can view the information by calling the following command:

```bash
getrawtransaction "txid" 4 
```

Use the order number, payment address, and payment amount in this transaction to compare with the data of the authorized party to determine whether the payment is successful (please pay attention to the number of confirmations when determining)

#### 2.2 AEM system sends GET form request to callbakcurl

If the payment is successful and the block is confirmed, the AEM system will send a GET form request to the callbackUrl (if the call is not successful, it will be called every 15 seconds until the call is successful or the number of call failures exceeds 6 times).

The parameters carried in CallbackUrl are as follows:

| parameter name | description                                                  |
| -------------- | ------------------------------------------------------------ |
| uuid           | The appUuid applied for when opening the payment service     |
| orderId        | Third-party business system order number                     |
| fromAddr       | Payment address                                              |
| toAddr         | Receiving address                                            |
| assetType      | Payment asset type                                           |
| amount         | Payment amount, string type                                  |
| txid           | Transaction number on the chain after successful payment     |
| callbackUrl    | Successful payment AEM service notification of the background callback path of the payment status |
| rsaContent     | Signature data, use the rsa private key obtained during access to sign the parameters |


### Signature algorithm

Use the SHA1WithRSA algorithm to sign the parameters using rsaPrivate when applying for APP.

##### 1. Arrange the parameters into a json structure in the order of amount, assetType, callbackUrl, fromAddr, orderId, toAddr, txid, uuid.

> Amount is a string type, directly spelled into JSON, do not convert a floating-point string (1.00000)

##### 2. Use SHA1withRSA to sign the parameters, and the signed content is BASE64 encoded as rsaContent. The third-party dapp can use the public key to verify the parameters and the signed information.

The information that the third-party Dapp needs to return is as follows:

| Return field | description                                                  |
| ------------ | ------------------------------------------------------------ |
| success      | true or false, if true, the callback is successful, if false, the callback fails |
| errorCode    | Error code, if success is true, it can not be returned.      |
| errorMsg     | Error message, if success is true, it may not be returned.   |

It is recommended that the docking authorized payer verify txid, payment address, payment amount and order number.

