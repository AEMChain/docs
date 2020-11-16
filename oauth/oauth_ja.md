# AEM License Access Developer Documentation	

AEMウォレットのスキームは以下の通りです

## 認可されたログイン

### アプリの認証ログイン

#### 1、APPは、AEMのアプリケーションの承認のログインを呼び出す 

サードパーティアプリはSchemeURL経由でウォレットアプリを呼び出し、URLエンコードされたJSONを使用して以下のパラメータを渡します

| KEY   | 説明                          |
| ----- | ----------------------------- |
| type  | パスログイン     必須 |
| uuid  | アクセスサービスがアクティブ化されたときに取得されるアプリのuuidが必要です |
| callbackUrl | スキームURLの後のschemeurl形式のコールバックアドレスを空にすることはできません。 必須。 例：appscheme：// login |
| callbackType | 0の場合、openCodeに戻ると「？」または「＆」で接続されます。 1の場合、opencodeはcallbackUrlの最後に直接スプライスされます。デフォルトの0は必要ありません。 |
| openCallBackType | プラットフォームが指定されていない場合、またはプラットフォームがモバイルの場合に有効になります。 openCallBackTypeが0または空の場合、ウォレットはcallbackurlアプリケーションを開きます。 openCallBackTypeが1の場合、ウォレットはcallbackurlへのhttpリクエストを開始し、ウォレットを最小化します。 必要ありません。 |
| platform | プラットフォームはウォレット端末を呼び出します。 携帯電話から呼び出された場合は、モバイルに入力します<br /> WebページのQRコードまたはh5ページをスキャンする場合は、Webに入力します |

> IOSはスキーマ内の特殊なシンボルを許可していないので、JSONデータはURLエンコードする必要があります
>
> callbackurl 自体のパラメータに特別な文字が含まれている場合は、 url の内部パラメータよりも前にエンコードする必要があることに注意しましょう

以前の全体的なURLエンコーディング

```
aem://{"type":"login","uuid":"121212-121212-121212","callbackUrl":"appscheme://login/a/b?d=1&redirect=http%3A%2F%2Fwww.google.com%2Fs%3Fwd%3D12345"}
```

全体的なURLエンコーディング

```bash
aem://%7B%22type%22:%22login%22,%22uuid%22:%22121212-121212-121212%22,%22callbackUrl%22:%22appscheme://login/a/b?d=1&redirect=http%253A%252F%252Fwww.google.com%252Fs%253Fwd%253D12345%22%7D
```

#### 2、Dapatkan Alamat

​		ユーザがAEMで認証を許可すると、ウォレットは認証用のopenCodeを生成し、openCodeパラメータをキャリーしてcallbackUrlのアドレスにジャンプします

> openCodeの有効期限は1分間で、1回の使用で有効期限が切れます

openCodeを取得したら、次のリンクをリクエストしてアドレスと一意の識別子を取得します

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| パラメータ名 | 説明                                                      |
| ------------ | --------------------------------------------------------- |
| openCode     | 最初のステップで取得したopenCodeを入力します              |
| accessToken  | サードパーティのアクセスを開くときに取得されるAccessToken |
| appId        | サードパーティのアクセスを開くときに取得されるUuid        |

説明を返す

正しい場合に返されるJSONパケットは次のとおりです
   注：「idは一意のIDです」

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

エラーコードerrorCodeは次のように説明されています：**

| エラーコード | 説明              |
| ------------ | ----------------- |
| 900001       | accessTokenエラー |
| 010001       | 未知の間違い      |

### WEBスキャンコード認証ログイン

#### 1、QRコードを作成する

アクセスパーティは、AEMスキャンコードログインを提供するためにURLQRコードを提供する必要があります。 URLパラメータは次のとおりです

| パラメータ名 | 説明    |是否必填|
| -------- | --------- |---------|
| type     | ログインを入力します |是|
| platform | ウェブに記入 |是|
| callbackUrl | 承認後にopenCodeに取り込まれたコールバックアドレス |是|
| uuid | アクセスパーティのアプリケーションが渡された後のUuid |是|
| callbackType | 0の場合、openCodeを返し、webTokenは「？」または「＆」でスプライスされます。 1の場合、openCodeとwebTokenはcallbackUrlの最後で直接スプライスされます。デフォルトは0 |否|
| webToken | アクセスパーティは、許可されたログインユーザーアドレスを確認します |否|

例えば：

```shell
aemlogin://{"type": "login","platform": "web","callbackUrl": "https://www.google.com/login","uuid": "123-123-123","webToken": "SSFJIEKALSM"}
```

ユーザーが承認を確認した後、APPは承認されたopenCodeとQRコード内のwebTokenをcallbackUrlパラメーターアドレスに戻します。

> openCodeの有効期間は1分で、1回使用すると無効になります

#### 2、アドレスと一意のIDを取得する

openCodeを取得したら、次のリンクをリクエストしてアドレスを取得してください

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

|ポーナ名|説明|
| ------------ | --------------------------------- |
| openCode |最初のステップで取得したopenCodeを入力します|
| accessToken |サードパーティアクセスを開くときに取得されるaccessToken |
| appId |サードパーティアクセスを開くときに取得されるuuid |

説明を返す

正しい場合に返されるJSONパケットは次のとおりです

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

**エラーコードerrorCodeの説明は次のとおりです：**

| 错误码 | 説明                      |
| ------ | ------------------------- |
| 900001 | accessToken错误           |
| 010001 | 未知错误                  |



## ウォレットの支払い

サードパーティのAPPは、AEM Paymentを呼び出して、注文の支払いプロセスを完了することができます。

### 1、ユーザー支払いを申請する

#### 1.1APPコール

APPウォレットはschemeurlを介してAEMを呼び出し、URLでエンコードされたJSONオブジェクトを介して次のパラメーターを渡します

| パラメータ名   | 説明                                                        |
| ---------------- | -------------------------------- ---------------------------- |
|タイプ|タイプ。支払いを記入する|
| appUuid |アプリを申請するときのUuid、必須。 |
| orderId |注文ID、必須。 |
| toAddr |レシーバーアドレス、必須。 |
| AssetType |支払い資産タイプ、必須。 |
|金額|支払い金額、必須。 |
| callbackUrl |支払いが成功しましたAEMサービスは、バックグラウンドコールバックに支払いステータスを通知します。ホストはセキュアドメイン名である必要があります。 |
| callbackScheme |支払いが完了した後にリダイレクトされるappschemeurl。必須。 |
|説明|説明は、ユーザーに表示されます。 |
| signedContent |署名されたデータ。アクセス中に取得したrsa秘密鍵を使用して、必要なパラメーターに署名します。 |
| openCallBackType |支払い完了後のAPPコールバックメソッド。 0の場合、電話はcallbackSchemeを開きます。 1の場合、他のアプリは開かれず、ウォレットアプリのみが最小化されます。この操作は、支払いステータスのコールバックには影響しません。 |

例：

URLエンコード前

```
aem://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","callbackScheme": "payApp",
	"description": "*****","signedContent": "*******","openCallBackType": "1"}
```

> 注意：amount为字符串类型

URLエンコード後

```
aem://%7B%22type%22:%20%22pay%22,%22appUuid%22:%20%225555-6666-8888-9999%22,%22orderId%22:%20%222020092487855441%22,%22toAddr%22:%20%22*********************%22,%22assetType%22:%20%22AEM%22,%22amount%22:%20%220.001%22,%22callbackUrl%22:%20%22https://localhost:8080/pay?orderId=2020092487855441%22,%22callbackScheme%22:%20%22payApp%22,%0A%09%22description%22:%20%22*****%22,%22signedContent%22:%20%22*******%22,%22openCallBackType%22:%20%221%22%7D
```

#### 1.2QRコードコール

アクセスパーティは、URL QRコードを作成し、ウォレットAPPでコードをスキャンする必要があります。 パラメータはjsonメソッドを使用し、内容は次のとおりです。

|ポーナ名|説明|
| ------------- | ----------------------------------- ------------------------- |
|タイプ|支払いを記入|
| signedContent |署名されたデータ。アクセス時に取得したrsa秘密鍵を使用して、必要なパラメーターに署名します。 |
| appUuid |アクティベーション時に取得したAppuuidが必要です。 |
| orderId |サードパーティのビジネスシステムの注文番号、必須。 |
| toAddr |支払い先住所、必須。 |
| AssetType |支払い資産タイプ、必須。 |
|金額|支払い金額、必須。 |
| callbackUrl |支払い成功AEMは、バックグラウンドコールバックに支払いステータスを通知します。ホストはセキュアドメイン名である必要があります。 |
|説明|説明は、ユーザーに表示されます。 |

例如：

```
aempay://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","description": "*****","signedContent": "*******",}
```



#### 署名アルゴリズム

SHA1WithRSAアルゴリズムを使用して、アプリケーションで使用されるrsaPrivateを使用して、amount、appUuid、assetType、callbackUrl、orderId、およびtoAddrパラメーターに署名します。

署名するパラメータが次のとおりであるとします。

```json
{"amount":"amountVal","appUuid":"uuidVal","assetType":"assetTypeVal","callbackUrl":"callbackUrlVal","orderId":"orderIdVal","toAddr":"toAddrVal"}
```

1.パラメータを、amount、appUuid、assetType、callbackUrl、orderId、toAddrの順にjson構造に配置します。

##### 2. SHA1withRSAを使用してパラメーターに署名すると、署名されたコンテンツはRSAContentとしてBASE64エンコードされます。

### 2。支払いが完了しました

支払いの完了を判断する方法は2つあります。

#### 2.1受信アドレスの取引記録に基づく判断

AEMによって承認されたトランザクションの場合、メタデータはこのトランザクションで運ばれ、内容は注文番号です。承認された当事者は、次のコマンドを呼び出すことで情報を表示できます。

```bash
getrawtransaction "txid" 4 
```

このトランザクションの注文番号、支払い先住所、支払い金額を使用して、承認された当事者のデータと比較し、支払いが成功したかどうかを判断します（判断を行う際は確認の数に注意してください）。

#### 2.2AEMシステムはGETフォームリクエストをcallbakcurlに送信します

支払いが成功し、ブロックが確認されると、AEMシステムはGETフォーム要求をcallbackUrlに送信します（呼び出しが成功しなかった場合、呼び出しが成功するか、呼び出しの失敗回数が6回を超えるまで、15秒ごとに呼び出されます）。

|ポーナ名|説明|
| ----------- | ------------------------------------- ---------- |
| uuid |支払いサービスを開くときに申請されたappUuid |
| orderId |サードパーティのビジネスシステムの注文番号|
| fromAddr |支払いアドレス|
| toAddr |受信アドレス|
| AssetType |支払い資産タイプ|
|金額|支払い金額、文字列タイプ|
| txid |支払い成功後のオンチェーントランザクション番号|
| callbackUrl |支払い成功AEMサービス通知支払いステータスバックグラウンドコールバックパス|
| rsaContent |署名データ、アクセス中に取得したrsa秘密鍵を使用してパラメーターに署名します|


### 署名アルゴリズム

APPを申請するときに、SHA1WithRSAアルゴリズムを使用して、rsaPrivateを使用してパラメーターに署名します。

##### 1.パラメータを、amount、assetType、callbackUrl、fromAddr、orderId、toAddr、txid、uuidの順にjson構造に配置します。

> amountは文字列タイプであり、JSONに直接スペルされ、浮動小数点文字列を変換しません（1.00000）

##### 2. SHA1withRSAを使用してパラメーターに署名すると、署名されたコンテンツはrsaContentとしてBASE64でエンコードされます。サードパーティのdappは、公開キーを使用してパラメーターと署名された情報を確認できます。

サードパーティのDappが返す必要のある情報は次のとおりです。

|フィールドに戻る|説明|
| --------- | --------------------------------------- ------------- |
|成功| trueまたはfalse、trueの場合、コールバックは成功し、falseの場合、コールバックは失敗します|
| errorCode |エラーコード。成功がtrueの場合、返すことはできません。|
| errorMsg |エラーメッセージ。成功がtrueの場合、返されません。|

ドッキングを許可された支払人は、txid、支払いアドレス、支払い金額、および注文番号を確認することをお勧めします。
