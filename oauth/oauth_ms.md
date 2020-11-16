# AEM membenarkan akses ke dokumentasi pembangun

Skema dompet AEM adalah: aem.

## Log masuk yang dibenarkan

### Log masuk kebenaran aplikasi

#### 1. APP memanggil AEM untuk memohon log masuk yang dibenarkan

Aplikasi pihak ketiga memanggil aplikasi dompet melalui SchemeURL dan menggunakan JSON yang dikodkan URL untuk melewati parameter berikut

| KEY   | penerangan                |
| ----- | ----------------------------- |
| type  | Log masuk diperlukan |
| uuid  | Uuid aplikasi yang diperoleh apabila perkhidmatan akses diaktifkan diperlukan |
| callbackUrl | Alamat panggil balik dalam format skemaurl, setelah skema tidak boleh kosong. Dikehendaki. Contohnya: appscheme: // log masuk |
| callbackType | Apabila 0, kembali ke openCode akan disambungkan dengan "?" Atau "&". Apabila ia adalah 1, openCode secara langsung disambungkan pada akhir callbackUrl. Lalai 0 tidak diperlukan |
| openCallBackType | pIa berlaku apabila platform tidak ditentukan atau platform mudah alih. Apabila openCallBackType 0 atau kosong, dompet membuka aplikasi callbackurl. Apabila openCallBackType adalah 1, dompet memulakan permintaan http untuk callbackurl dan meminimumkan dompet. Tidak dikehendaki |
| platform | Naikkan terminal dompet. Jika dipanggil dari telefon bimbit, isikan telefon bimbit <br /> Jika hendak mengimbas kod QR halaman web atau halaman h5, isi web |

> Oleh kerana sistem IOS tidak membenarkan penggunaan simbol khas dalam skema, perlu mengekodkan data JSON url
>
> Perlu diperhatikan bahawa jika watak khas muncul dalam parameter callbackurl itu sendiri, parameter dalaman perlu dikodkan url terlebih dahulu.

Sebelum pengekodan URL keseluruhan

```
aem://{"type":"login","uuid":"121212-121212-121212","callbackUrl":"appscheme://login/a/b?d=1&redirect=http%3A%2F%2Fwww.google.com%2Fs%3Fwd%3D12345"}
```

Selepas pengekodan URL keseluruhan

```bash
aem://%7B%22type%22:%22login%22,%22uuid%22:%22121212-121212-121212%22,%22callbackUrl%22:%22appscheme://login/a/b?d=1&redirect=http%253A%252F%252Fwww.google.com%252Fs%253Fwd%253D12345%22%7D
```

#### 2、Dapatkan Alamat

​		Setelah pengguna membenarkan keizinan dalam ARM, dompet menghasilkan openCode yang dibenarkan dan membawa parameter openCode untuk melompat ke alamat callbackUrl

> Tempoh sah openCode adalah 1 minit, dan menjadi tidak sah setelah satu penggunaan

Setelah memperoleh openCode, minta pautan berikut untuk mendapatkan alamat dan pengecam unik

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| Nama parameter | Penerangan |
| ----------- | --------------------------------- |
| openCode | Isi openCode yang diperoleh pada langkah pertama |
| accessToken | AccessToken yang diperoleh semasa membuka akses pihak ketiga |
| appId | uuid yang diperoleh semasa membuka akses pihak ketiga |

Keterangan kembali

Paket JSON dikembalikan apabila betul adalah seperti berikut
   Catatan: ‘id adalah pengenalan yang unik’

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

**Kesalahan kod errorCode dijelaskan seperti berikut：**

| kod salah | penerangan                     |
| --------- | ------------------------------ |
| 900001    | ralat accessToken              |
| 010001    | kesilapan yang tidak diketahui |
### Log masuk kebenaran kod imbasan WEB

#### 1、Buat kod QR

Pihak akses perlu memberikan kod QR URL untuk menyediakan log masuk kod imbasan AEM. Parameter URL adalah seperti berikut

| nama parameter | penerangan |Adakah diperlukan|
| -------- | --------- |---------|
| type     | Isi log masuk |Ya|
| platform | Isi laman web |Ya|
| callbackUrl | Alamat panggil balik dimasukkan ke dalam openCode selepas kebenaran |Ya|
| uuid | Uuid setelah permohonan pihak akses diluluskan |Ya|
| callbackType | Apabila angka 0, openCode dan webToken kembali akan disambungkan dengan "?" Atau "&". Apabila ia adalah 1, openCode dan webToken secara langsung disambungkan pada akhir panggilan balikUrl. Lalai 0 |tidak|
| webToken | Pihak akses mengesahkan alamat pengguna log masuk yang dibenarkan |tidak|



Cth:

```shell
aemlogin://{"type": "login","platform": "web","callbackUrl": "https://www.google.com/login","uuid": "123-123-123","webToken": "SSFJIEKALSM"}
```

Setelah pengguna mengesahkan kebenaran, APP akan mengembalikan openCode dan webToken yang dibenarkan dalam kod QR ke alamat parameter callbackUrl.

> > openCode berlaku selama 1 minit, dan menjadi tidak sah setelah satu penggunaan

#### 2、Dapatkan Alamat dan identiti unik

获取openCode后，请求以下链接获取address

https://mobile.aemchain.com:8443/aem/api/oauth/login/idInfo?openCode={openCode}&accessToken={accessToken}&appId={appId}

| Nama parameter | Penerangan |
| ----------- | --------------------------------- |
| openCode | Isi openCode yang diperoleh pada langkah pertama |
| accessToken | AccessToken yang diperoleh semasa membuka akses pihak ketiga |
| appId | uuid yang diperoleh semasa membuka akses pihak ketiga |

Keterangan kembali

Paket JSON dikembalikan apabila betul adalah seperti berikut

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

**Huraian kod kesalahan errorCode adalah seperti berikut：**

| Kod ralat | Penerangan |
| ------ | ------------------------- |
| 900001 | ralat capaian akses |
| 010001 | Ralat tidak diketahui |



## Bayaran dompet

APP pihak ketiga boleh memanggil Pembayaran AEM untuk menyelesaikan proses pembayaran pesanan.

### 1. Mohon pembayaran pengguna

#### 1.1 Panggilan APP

Dompet APP memanggil AEM melalui skemaurl dan melewati parameter berikut melalui objek JSON yang dikodkan URL

| nama parameter | penerangan                                               |
| ---------------- | ------------------------------------------------------------ |
| type             | enis. Isi gaji                                  |
| appUuid             | Uuid pada masa aplikasi diperlukan. |
| orderId          | Id pesanan, diperlukan.          |
| toAddr           | Alamat penerima, diperlukan.                 |
| assetType        | Jenis aset pembayaran, diperlukan.       |
| amount           | Amaun pembayaran diperlukan.                 |
| callbackUrl      | Pembayaran berjaya. Perkhidmatan AEM memberitahu panggilan balik latar belakang mengenai status pembayaran. Host mesti berada dalam nama domain selamat. |
| callbackScheme   | Appchemeurl yang akan dialihkan kembali setelah pembayaran selesai, diperlukan. |
| description             | Penerangan akan ditunjukkan kepada pengguna. |
| signedContent    | Data tandatangan. Gunakan kunci peribadi rsa yang diperoleh semasa akses untuk menandatangani parameter, yang diperlukan. |
| openCallBackType | Kaedah panggilan balik APP setelah pembayaran selesai. Apabila 0, telefon akan membuka Skema panggil balik Apabila sudah 1, aplikasi lain tidak akan dibuka dan hanya aplikasi dompet yang akan diminimumkan. Operasi ini tidak mempengaruhi panggilan balik status pembayaran. |

contoh:

Sebelum pengekodan URL

```
aem://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","callbackScheme": "payApp",
	"description": "*****","signedContent": "*******","openCallBackType": "1"}
```

> Nota: jumlah adalah jenis rentetan

Selepas pengekodan URL

```
aem://%7B%22type%22:%20%22pay%22,%22appUuid%22:%20%225555-6666-8888-9999%22,%22orderId%22:%20%222020092487855441%22,%22toAddr%22:%20%22*********************%22,%22assetType%22:%20%22AEM%22,%22amount%22:%20%220.001%22,%22callbackUrl%22:%20%22https://localhost:8080/pay?orderId=2020092487855441%22,%22callbackScheme%22:%20%22payApp%22,%0A%09%22description%22:%20%22*****%22,%22signedContent%22:%20%22*******%22,%22openCallBackType%22:%20%221%22%7D
```

#### 1.2 Panggilan kod QR

Pihak akses perlu membina kod QR URL dan mengimbas kod dengan aplikasi dompet. Parameter menggunakan kaedah json, dan isinya adalah seperti berikut

| nama parameter | penerangan                                                   |
| -------------- | ------------------------------------------------------------ |
| type           | Isi gaji                                                     |
| signedContent  | Data tandatangan. Gunakan kunci peribadi rsa yang diperoleh semasa akses untuk menandatangani parameter, yang diperlukan. |
| appUuid        | Appuuid yang diperoleh pada masa pengaktifan diperlukan.     |
| orderId        | Nombor pesanan sistem perniagaan pihak ketiga, diperlukan.   |
| toAddr         | Menerima alamat, diperlukan.                                 |
| assetType      | Jenis aset pembayaran, diperlukan.                           |
| amount         | Amaun pembayaran diperlukan.                                 |
| callbackUrl    | Pembayaran berjaya. AEM memberitahu panggilan balik latar belakang mengenai status pembayaran. Host mesti ada dalam nama domain selamat. |
| description    | Penerangan akan ditunjukkan kepada pengguna.                 |

例如：

```
aempay://{"type": "pay","appUuid": "5555-6666-8888-9999","orderId": "2020092487855441","toAddr": "*********************","assetType": "AEM","amount": "0.001","callbackUrl": "https://localhost:8080/pay?orderId=2020092487855441","description": "*****","signedContent": "*******",}
```



#### Algoritma Tandatangan

Gunakan algoritma SHA1WithRSA untuk menandatangani parameter, appUuid, assetType, callbackUrl, orderId, dan toAddr menggunakan rsaPrivate yang digunakan dalam aplikasi.

Katakan parameter yang akan ditandatangani adalah seperti berikut:

```json
{"amount":"amountVal","appUuid":"uuidVal","assetType":"assetTypeVal","callbackUrl":"callbackUrlVal","orderId":"orderIdVal","toAddr":"toAddrVal"}
```

##### 1. Susun parameter ke dalam struktur json dalam urutan jumlah, appUuid, assetType, callbackUrl, orderId, toAddr.

##### 2. Gunakan SHA1withRSA untuk menandatangani parameter, dan kandungan yang ditandatangani adalah BASE64 dikodkan sebagai rsaContent.

### 2. Pembayaran selesai

Terdapat dua cara untuk menilai penyelesaian pembayaran:

#### 2.1 Menilai berdasarkan catatan transaksi alamat penerima

Untuk transaksi yang disahkan oleh AEM, metadata akan dilakukan dalam transaksi ini, dan isinya adalah nombor pesanan. Pihak yang berwenang dapat melihat maklumat dengan memanggil perintah berikut:

```bash
getrawtransaction "txid" 4 
```

Gunakan nomor pesanan, alamat pembayaran, jumlah pembayaran dalam transaksi ini untuk membandingkan dengan data pihak yang berwenang untuk menentukan apakah pembayaran berhasil (harap perhatikan jumlah pengesahan ketika membuat penilaian)

#### 2.2 Sistem AEM menghantar permintaan borang GET ke callbakcurl

Sekiranya pembayaran berjaya dan blok disahkan, sistem AEM akan menghantar permintaan borang GET ke callbackUrl (jika panggilan tidak berjaya, ia akan dipanggil setiap 15 saat sehingga panggilan berjaya atau jumlah kegagalan panggilan melebihi 6 kali).

Parameter yang dibawa dalam CallbackUrl adalah seperti berikut:

| Nama parameter | Penerangan |
| ----------- | ------------------------------------- ---------- |
| uuid | AplikasiUuid yang dilamar semasa membuka perkhidmatan pembayaran |
| orderId | Nombor pesanan sistem perniagaan pihak ketiga |
| fromAddr | Alamat pembayaran |
| toAddr | Alamat penerima |
| Jenis aset | Jenis aset pembayaran |
| jumlah | Jumlah pembayaran, jenis tali |
| txid | Nombor transaksi dalam rangkaian setelah pembayaran berjaya |
| callbackUrl | Pembayaran berjaya AEM perkhidmatan pemberitahuan status pembayaran latar belakang jalan balik |
| rsaContent | Data tandatangan, gunakan kunci peribadi rsa yang diperoleh semasa akses untuk menandatangani parameter |

### Algoritma Tandatangan

Gunakan algoritma SHA1WithRSA untuk menandatangani parameter menggunakan rsaPrivate semasa memohon APP.

##### 1. Susun parameter ke dalam struktur json dalam urutan jumlah, jenis aset, callbackUrl, dariAddr, orderId, toAddr, txid, uuid.

> jumlah adalah jenis rentetan, dieja secara langsung menjadi JSON, jangan ubah rentetan titik terapung (1.00000)

##### 2. Gunakan SHA1withRSA untuk menandatangani parameter, dan kandungan yang ditandatangani adalah BASE64 dikodkan sebagai rsaContent. Dapp pihak ketiga dapat menggunakan kunci awam untuk mengesahkan parameter dan maklumat yang ditandatangani.

Maklumat yang perlu dikembalikan oleh Dapp pihak ketiga adalah seperti berikut:

| Kembali ke Medan | Penerangan |
| --------- | --------------------------------------- ------------- |
| kejayaan | benar atau salah, jika benar, panggilan balik berjaya, jika salah, panggilan balik gagal |
| errorCode | Kod ralat, jika kejayaan itu benar, ia tidak dapat dikembalikan. |
| errorMsg | Mesej ralat, jika kejayaan itu benar, tidak kembali. |

Adalah disyorkan bahawa pembayar yang diberi kuasa mengesahkan txid, alamat pembayaran, jumlah pembayaran dan nombor pesanan.
