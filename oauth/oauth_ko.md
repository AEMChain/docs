# 개발자 문서에 대한 AEM 승인 액세스

AEM 지갑의 구성표는 aem입니다.

## 승인 된 로그인

### 앱 인증 로그인

#### 1. APP가 AEM을 호출하여 인증 된 로그인을 신청합니다.

타사 앱은 SchemeURL을 통해 지갑 앱을 호출하고 URL 인코딩 JSON을 사용하여 다음 매개 변수를 전달합니다.

| KEY   | 기술                        |
| ----- | ----------------------------- |
| type  | 로그인 패스    필수 |
| uuid  | 액세스 서비스 활성화시 획득 한 앱의 uuid가 필요합니다. |
| callbackUrl | schemeurl 형식의 콜백 주소는 schemeurl 이후에 비워 둘 수 없습니다. 필수입니다. 예 : appscheme : // login |
| callbackType | 0 인 경우 openCode로 복귀하면 "?"또는 "&"가 이어집니다. 1 인 경우 callbackUrl 끝에 openCode가 직접 연결됩니다. 기본값 0은 필요하지 않습니다. |
| openCallBackType | 플랫폼이 지정되지 않았거나 플랫폼이 모바일 일 때 적용됩니다. openCallBackType이 0이거나 비어있는 경우 지갑은 callbackurl 애플리케이션을 엽니 다. openCallBackType이 1이면 지갑은 callbackurl에 대한 http 요청을 시작하고 지갑을 최소화합니다. 필요하지 않음 |
| platform | 지갑 단말기를 켜십시오. 휴대 전화에서 전화를 걸 경우 모바일 입력 <br /> 웹 페이지 QR 코드 스캔 또는 h5 페이지 스캔 일 경우 web 입력 |

> IOS 시스템은 체계에서 특수 기호 사용을 허용하지 않으므로 JSON 데이터를 URL 인코딩해야합니다.
>
> callbackurl 자체의 매개 변수에 특수 문자가 표시되는 경우 내부 매개 변수를 미리 URL 인코딩해야합니다.

전체 URL 인코딩 전

```
aem://{"type":"login","uuid":"121212-121212-121212","callbackUrl":"appscheme://login/a/b?d=1&redirect=http%3A%2F%2Fwww.google.com%2Fs%3Fwd%3D12345"}
```

전체 URL 인코딩 후

```bash
aem://%7B%22type%22:%22login%22,%22uuid%22:%22121212-121212-121212%22,%22callbackUrl%22:%22appscheme://login/a/b?d=1&redirect=http%253A%252F%252Fwww.google.com%252Fs%253Fwd%253D12345%22%7D
```

#### 2、주소 받기

​		사용자가 AEM에서 인증을 허용하면 지갑이 인증 된 openCode를 생성하고 openCode 매개 변수를 전달하여 callbackUrl의 주소로 이동합니다.

> 오픈 코드의 유효 기간은 1 분이며 1 회 사용 후 무효가됩니다

오픈 코드 획득 후 다음 링크를 요청하여 주소 및 고유 식별자 획득

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| 매개 변수 이름 | 기술                                           |
| -------------- | ---------------------------------------------- |
| openCode       | 첫 번째 단계에서 얻은 openCode를 입력하십시오. |
| accessToken    | 타사 액세스를 열 때 얻은 AccessToken           |
| appId          | 타사 액세스를 열 때 얻은 Uuid                  |

반품 설명

올바른 경우 반환되는 JSON 패킷은 다음과 같습니다.
   참고 :‘ID는 고유 한 ID입니다.’

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

**오류 코드 errorCode는 다음과 같이 설명됩니다：**

| 에러 코드 | 기술             |
| --------- | ---------------- |
| 900001    | accessToken 오류 |
| 010001    | 알 수없는 실수   |

### 웹 스캔 코드 인증 로그인

#### 1、QR 코드 생성

액세스 당사자는 AEM 스캔 코드 로그인을 제공하기 위해 URL QR 코드를 제공해야합니다. URL 매개 변수는 다음과 같습니다

| 매개 변수 이름 | 기술    |필요합니까|
| -------- | --------- |---------|
| type     | 로그인 입력 |예|
| platform | 웹 채우기 |예|
| callbackUrl | 인증 후 openCode로 가져온 콜백 주소 |예|
| uuid | 액세스 당사자의 애플리케이션이 통과 된 후의 Uuid |예|
| callbackType | 0이면 openCode를 반환하고 webToken은 "?"또는 "&"로 연결됩니다. 1 일 경우 callbackUrl 끝에 openCode와 webToken이 직접 연결됩니다. 기본값 0 |아니|
| webToken | 액세스 당사자는 인증 된 로그인 사용자 주소를 확인합니다 |아니|

예 :

```shell
aemlogin://{"type": "login","platform": "web","callbackUrl": "https://www.google.com/login","uuid": "123-123-123","webToken": "SSFJIEKALSM"}
```

사용자가 인증을 확인하면 APP는 인증 된 openCode와 QR 코드의 webToken을 callbackUrl 매개 변수 주소로 다시 가져옵니다.

> 오픈 코드의 유효 기간은 1 분이며 1 회 사용 후 무효가됩니다.

#### 2、주소 및 고유 한 신원 확인

오픈 코드 획득 후 다음 링크를 요청하여 주소 획득

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| 반품 설명   | 기술                                           |
| ----------- | ---------------------------------------------- |
| openCode    | 첫 번째 단계에서 얻은 openCode를 입력하십시오. |
| accessToken | 타사 액세스를 열 때 얻은 AccessToken           |
| appId       | 타사 액세스를 열 때 얻은 Uuid                  |

반품 설명

올바른 경우 반환되는 JSON 패킷은 다음과 같습니다.

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

**오류 코드 errorCode는 다음과 같이 설명됩니다.：**

| 에러 코드 | 기술             |
| --------- | ---------------- |
| 900001    | accessToken 오류 |
| 010001    | 알 수없는 실수   |



## 지갑 결제

타사 앱은 AEM Payment를 호출하여 주문 결제 프로세스를 완료 할 수 있습니다.

### 1、사용자 결제 신청

#### 1.1APP 호출

APP 지갑은 schemeurl을 통해 AEM을 호출하고 URL 인코딩 된 JSON 객체를 통해 다음 매개 변수를 전달합니다.

| 매개 변수 이름 | 기술                                                       |
| ---------------- | ------------------------------------------------------------ |
| type             | 유형. 급여 입력                                       |
| appUuid             | 앱 신청시 uuid가 필요합니다.                 |
| orderId          | 주문 ID, 필수입니다.                    |
| toAddr           | 수신자 주소, 필수입니다.                               |
| assetType        | 결제 자산 유형, 필수입니다.                         |
| amount           | 결제 금액, 필수입니다.                                |
| callbackUrl      | 결제가 성공했습니다. AEM 서비스가 결제 상태를 백그라운드 콜백에 알립니다. 호스트는 보안 도메인 이름에 있어야합니다. |
| callbackScheme   | 결제가 완료된 후 리디렉션 될 appschemeurl은 필수입니다. |
| description             | 기술,사용자에게 표시됩니다.。                            |
| signedContent    | 서명 데이터.액세스 중에 얻은 rsa 개인 키로 매개 변수에 서명합니다.해야합니다. |
| openCallBackType | 결제 완료 후 APP 콜백 방법. 0이면 전화기가 callbackScheme을 엽니 다. 1 일 경우 다른 앱이 열리지 않고 지갑 앱만 최소화되며 결제 상태 콜백에는 영향을주지 않습니다. |

예 :

URL 인코딩 전

```
aem://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","callbackScheme": "payApp",
	"description": "*****","signedContent": "*******","openCallBackType": "1"}
```

> 참고 : 금액은 문자열 유형입니다.

URL 인코딩 후

```
aem://%7B%22type%22:%20%22pay%22,%22appUuid%22:%20%225555-6666-8888-9999%22,%22orderId%22:%20%222020092487855441%22,%22toAddr%22:%20%22*********************%22,%22assetType%22:%20%22AEM%22,%22amount%22:%20%220.001%22,%22callbackUrl%22:%20%22https://localhost:8080/pay?orderId=2020092487855441%22,%22callbackScheme%22:%20%22payApp%22,%0A%09%22description%22:%20%22*****%22,%22signedContent%22:%20%22*******%22,%22openCallBackType%22:%20%221%22%7D
```

#### 1.2QR 코드 호출

액세스 당사자는 URL QR 코드를 구성하고 지갑 앱으로 코드를 스캔해야합니다. 매개 변수는 json 모드를 사용하며 내용은 다음과 같습니다.

| 매개 변수 이름 | 기술                                                         |
| -------------- | ------------------------------------------------------------ |
| type           | 급여 입력                                                    |
| signedContent  | 서명 데이터.使액세스 중에 얻은 rsa 개인 키로 매개 변수에 서명합니다. 해야. |
| appUuid        | 활성화시 획득 한 Appuuid,해야합니다.                         |
| orderId        | 타사 비즈니스 시스템 주문 번호,해야합니다.                   |
| toAddr         | 수신 주소,해야합니다.                                        |
| assetType      | 결제 자산 유형,해야합니다.                                   |
| amount         | 결제 금액,해야합니다.                                        |
| callbackUrl    | 결제가 완료되었습니다. AEM에서 결제 상태에 대한 백그라운드 콜백을 알립니다. 호스트는 보안 도메인 이름에 있어야합니다, 해야합니다. |
| description    | 사용자에게 설명이 표시됩니다.                                |

예 :

```
aempay://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","description": "*****","signedContent": "*******",}
```



#### 서명 알고리즘

SHA1WithRSA 알고리즘을 사용하여 애플리케이션에서 사용되는 rsaPrivate를 사용하여 amount, appUuid, assetType, callbackUrl, orderId 및 toAddr 매개 변수에 서명합니다.

서명 할 매개 변수가 다음과 같다고 가정합니다.

```json
{"amount":"amountVal","appUuid":"uuidVal","assetType":"assetTypeVal","callbackUrl":"callbackUrlVal","orderId":"orderIdVal","toAddr":"toAddrVal"}
```

##### 1.amount, appUuid, assetType, callbackUrl, orderId 및 toAddr 순서로 매개 변수를 json 구조로 정렬하십시오.

##### 2.SHA1withRSA를 사용하여 매개 변수에 서명하면 서명 된 콘텐츠는 rsaContent로 BASE64로 인코딩됩니다.

### 2、결제 완료

결제 완료를 판단하는 방법에는 두 가지가 있습니다.

#### 2.1 수신 주소의 거래 기록에 따라

​	AEM을 통해 지불하도록 승인 된 거래는이 거래에서 메타 데이터를 전달하며 콘텐츠는 주문 번호입니다. 승인 된 당사자는 다음 명령을 호출하여 정보를 볼 수 있습니다.

```bash
getrawtransaction "txid" 4 
```

이 거래의 주문 번호, 지불 주소, 지불 금액을 사용하여 승인 된 당사자의 데이터와 비교하여 지불이 성공적 이었는지 확인하십시오 (판단 할 때 확인 횟수에주의하십시오).

#### 2.2 AEM 시스템은 callbakcurl에 GET 양식 요청을 보냅니다.

결제가 성공하고 차단이 확인되면 AEM 시스템은 callbackUrl에 GET 양식 요청을 보냅니다 (호출이 성공하지 못하면 호출이 성공하거나 호출 실패 횟수가 6 회를 초과 할 때까지 15 초마다 호출됩니다). AEM 시스템은 callbakcurl에 GET 양식 요청을 보냅니다.

CallbackUrl에서 전달되는 매개 변수는 다음과 같습니다.

| 매개 변수 이름 | 기술                                                         |
| -------------- | ------------------------------------------------------------ |
| uuid           | 결제 서비스 오픈시 신청 한 appUuid                           |
| orderId        | 타사 비즈니스 시스템 주문 번호                               |
| fromAddr       | 지불 주소                                                    |
| toAddr         | 수신 주소                                                    |
| assetType      | 결제 자산 유형                                               |
| amount         | 결제 금액, 문자열 유형                                       |
| txid           | 결제 성공 후 체인상의 거래 번호                              |
| callbackUrl    | 결제 상태의 백그라운드 콜백 경로에 대한 결제 성공 AEM 서비스 알림 |
| rsaContent     | 서명 데이터, 액세스 중에 얻은 rsa 개인 키를 사용하여 매개 변수에 서명 |


### 서명 알고리즘

APP를 신청할 때 rsaPrivate를 사용하여 매개 변수에 서명하려면 SHA1WithRSA 알고리즘을 사용하십시오.

##### 1. amount, assetType, callbackUrl, fromAddr, orderId, toAddr, txid, uuid의 순서로 매개 변수를 json 구조로 배열하십시오.

> Amount는 JSON으로 직접 입력 된 문자열 유형이며 부동 소수점 문자열 (1.00000)을 변환하지 않습니다.

##### 2.SHA1withRSA를 사용하여 매개 변수에 서명하고 서명 된 콘텐츠는 BASE64로 rsaContent로 인코딩됩니다. 타사 dapp은 공개 키를 사용하여 매개 변수와 서명 된 정보를 확인할 수 있습니다.

타사 Dapp이 반환해야하는 정보는 다음과 같습니다.

| 반환 필드 | 기술                                                     |
| --------- | -------------------------------------------------------- |
| success   | true 또는 false, true이면 콜백 성공, false이면 콜백 실패 |
| errorCode | 성공이 참이면 오류 코드를 반환 할 수 없습니다.           |
| errorMsg  | 성공이 참이면 오류 메시지가 반환되지 않을 수 있습니다.   |

도킹 승인 된 지불 인이 txid, 지불 주소, 지불 금액 및 주문 번호를 확인하는 것이 좋습니다.
