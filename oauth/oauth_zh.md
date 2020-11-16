# AEM授权接入开发者文档	

AEM钱包的Scheme为：aem。

## 授权登录

### app授权登录

#### 	1、APP调用 AEM 申请授权登录 

第三方APP通过SchemeURL调起钱包APP并使用URL编码的JSON传递如下参数

| KEY   | 描述                          |
| ----- | ----------------------------- |
| type  | 传login     必填  |
| uuid  | 开通接入服务时获得的app的uuid必填 |
| callbackUrl | schemeurl格式的回调地址，schemeurl后不能为空。必填。 例如：appscheme://login |
| callbackType | 为0时,返回openCode将通过"?"或者"&"进行拼接。为1时openCode直接拼接在callbackUrl的最后  默认0 非必填 |
| openCallBackType | platform未指定或platform为mobile时生效。openCallBackType为0或空时,  钱包打开callbackurl应用。openCallBackType为1时，   钱包向callbackurl发起http请求并最小化钱包。非必填 |
| platform | 调起钱包的终端。如果是手机内调用填写 mobile <br />如果是扫网页二维码或者h5页面填写web |

> 由于IOS系统不允许在scheme使用特殊符号，所以需要对JSON数据进行url编码
>
> 需要注意的是，callbackurl本身的参数中如果出现了特殊字符需要提前先对内部参数进行url编码

整体URL编码前

```
aem://{"type":"login","uuid":"121212-121212-121212","callbackUrl":"appscheme://login/a/b?d=1&redirect=http%3A%2F%2Fwww.google.com%2Fs%3Fwd%3D12345"}
```

整体URL编码后

```bash
aem://%7B%22type%22:%22login%22,%22uuid%22:%22121212-121212-121212%22,%22callbackUrl%22:%22appscheme://login/a/b?d=1&redirect=http%253A%252F%252Fwww.google.com%252Fs%253Fwd%253D12345%22%7D
```

#### 2、获取Address

​		用户在 AEM 中允许授权后，钱包生成授权openCode并携带openCode参数跳转回调callbackUrl的地址

> openCode有效期为1分钟，且使用一次后失效

获取openCode后，请求以下链接获取address和唯一身份标识

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| 参数名称    | 描述                              |
| ----------- | --------------------------------- |
| openCode    | 填写第一步获取的openCode          |
| accessToken | 开通第三方接入时获得的accessToken |
| appId       | 开通第三方接入时获得的uuid        |

返回说明

正确时返回的JSON数据包如下
  注：‘id为唯一身份标识’

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

**错误返回码errorCode说明如下：**

| 错误码 | 描述                      |
| ------ | ------------------------- |
| 900001 | accessToken错误           |
| 010001 | 未知错误                  |

### WEB扫码授权登录

#### 1、创建二维码

接入方需要提供URL二维码供应 AEM 扫码登录。URL参数如下

| 参数名称 | 描述      |是否必填|
| -------- | --------- |---------|
| type     | 填写login  |是|
| platform | 填写web    |是|
| callbackUrl | 授权后将openCode带入的回调地址 |是|
| uuid | 接入方申请通过后的uuid |是|
| callbackType | 为0时,返回openCode与webToken将通过"?"或者"&"进行拼接。为1时openCode与webToken直接拼接在callbackUrl的最后  默认0 |否|
| webToken | 接入方验证授权登录用户地址 |否|

例如：

```shell
aemlogin://{"type": "login","platform": "web","callbackUrl": "https://www.google.com/login","uuid": "123-123-123","webToken": "SSFJIEKALSM"}
```

用户确认授权后,APP会将授权openCode与二维码中的webToken带回callbackUrl参数地址中。

> openCode有效期为1分钟，且使用一次后失效

#### 2、获取Address和唯一身份标识

获取openCode后，请求以下链接获取address

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| 参数名称    | 描述                              |
| ----------- | --------------------------------- |
| openCode    | 填写第一步获取的openCode          |
| accessToken | 开通第三方接入时获得的accessToken |
| appId       | 开通第三方接入时获得的uuid        |

返回说明

正确时返回的JSON数据包如下

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

**错误返回码errorCode说明如下：**

| 错误码 | 描述                      |
| ------ | ------------------------- |
| 900001 | accessToken错误           |
| 010001 | 未知错误                  |



## 钱包支付

第三方APP可以调用 AEM 支付完成下单支付的流程。

### 1、申请用户支付

#### 1.1APP调用

APP钱包通过schemeurl调起 AEM ，并通过URL编码的JSON对象传递以下参数

| 参数名称         | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| type             | 类型。填写pay                                                |
| appUuid             | app申请时的uuid,必须。                                   |
| orderId          | 订单id，必须。                                 |
| toAddr           | 接收方地址，必须。                                             |
| assetType        | 付款资产类型，必须。                                         |
| amount           | 付款金额，必须。                                             |
| callbackUrl      | 付款成功 AEM 服务通知付款状态的后台回调，host需要在安全域名内，必须。 |
| callbackScheme   | 支付完成后跳转回的appschemeurl，必须。                       |
| description             | 描述,将展示给用户。                                          |
| signedContent    | 签名数据。使用接入时获得的rsa私钥对参数进行签名，必须。      |
| openCallBackType | 支付完成后APP回调方式。为0时手机打开callbackScheme。为1时不打开其他APP，仅将钱包APP最小化，此操作不影响支付状态回调。 |

例：

URL编码前

```
aem://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","callbackScheme": "payApp",
	"description": "*****","signedContent": "*******","openCallBackType": "1"}
```

> 注意：amount为字符串类型

URL编码后

```
aem://%7B%22type%22:%20%22pay%22,%22appUuid%22:%20%225555-6666-8888-9999%22,%22orderId%22:%20%222020092487855441%22,%22toAddr%22:%20%22*********************%22,%22assetType%22:%20%22AEM%22,%22amount%22:%20%220.001%22,%22callbackUrl%22:%20%22https://localhost:8080/pay?orderId=2020092487855441%22,%22callbackScheme%22:%20%22payApp%22,%0A%09%22description%22:%20%22*****%22,%22signedContent%22:%20%22*******%22,%22openCallBackType%22:%20%221%22%7D
```

#### 1.2二维码调用

接入方需构建URL二维码由钱包APP扫码。参数使用json方式，内容如下

| 参数名称      | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| type          | 填写pay                                                      |
| signedContent | 签名数据。使用接入时获得的rsa私钥对参数进行签名，必须。      |
| appUuid       | 开通时获得的appuuid,必须。                                   |
| orderId       | 第三方业务系统订单号，必须。                                 |
| toAddr        | 收款地址，必须。                                             |
| assetType     | 付款资产类型，必须。                                         |
| amount        | 付款金额，必须。                                             |
| callbackUrl   | 付款成功 AEM 通知付款状态的后台回调，host需要在安全域名内，必须。 |
| description   | 描述,将展示给用户。                                          |

例如：

```
aempay://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","description": "*****","signedContent": "*******",}
```



#### 签名算法

使用申请时的rsaPrivate运用SHA1WithRSA算法对amount,appUuid,assetType,callbackUrl,orderId,toAddr参数进行签名。

假设需签名的参数如下：

```json
{"amount":"amountVal","appUuid":"uuidVal","assetType":"assetTypeVal","callbackUrl":"callbackUrlVal","orderId":"orderIdVal","toAddr":"toAddrVal"}
```

##### 1.按照amount,appUuid,assetType,callbackUrl,orderId,toAddr顺序将参数排成json结构。

##### 2.对参数使用SHA1withRSA进行签名，签名后的内容进行BASE64编码即为rsaContent。

### 2、支付完成

判断支付完成有以下两种方式：

#### 2.1 根据接收地址的交易记录判断

​	通过 AEM 授权支付的交易，在此笔交易中会携带metadata,内容为订单号，授权方可通过调用以下命令查看信息：

```bash
getrawtransaction "txid" 4 
```

用此笔交易中的订单号、收款地址、收款金额与授权方的数据进行比对，判断是否支付成功（判断时请注意确认次数）

#### 2.2 AEM系统发送GET表单请求至callbakcurl

若支付成功，且得到区块确认，那么 AEM系统将会发送GET表单请求至callbackUrl（若未调用成功，每隔15秒调用一次，直至调用成功或者调用失败次数超过6次）。

CallbackUrl中携带的参数如下：

| 参数名称    | 描述                                            |
| ----------- | ----------------------------------------------- |
| uuid        | 开通支付服务时申请的appUuid                     |
| orderId     | 第三方业务系统订单号                            |
| fromAddr    | 支付地址                                        |
| toAddr      | 收款地址                                        |
| assetType   | 付款资产类型                                    |
| amount      | 付款金额，字符串类型                            |
| txid        | 支付成功后的链上交易号                          |
| callbackUrl | 付款成功 AEM 服务通知付款状态的后台回调路径     |
| rsaContent  | 签名数据，使用接入时获得的rsa私钥对参数进行签名 |


### 签名算法

使用APP申请时的rsaPrivate运用SHA1WithRSA算法对参数进行签名。

##### 1.按照amount,assetType,callbackUrl,fromAddr,orderId,toAddr,txid,uuid顺序将参数排成json结构。

> amount为字符串类型,直接拼入JSON中,不要转浮点字符串(1.00000)

##### 2.对参数使用SHA1withRSA进行签名，签名后的内容进行BASE64编码即为rsaContent，第三方dapp可用公钥对参数和签名后信息进行验证。

第三方Dapp需要返回的信息如下：

| 返回字段  | 描述                                                 |
| --------- | ---------------------------------------------------- |
| success   | true or false,若为true,回调成功，若为false，回调失败 |
| errorCode | 错误码，若success为true，可不返回.                   |
| errorMsg  | 错误信息，若success为true，可不返回.                 |

建议对接授权支付方再对txid、收款地址、收款金额以及订单号进行校验。
