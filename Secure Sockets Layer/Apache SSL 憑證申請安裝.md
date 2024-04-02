---
title: 'Apache SSL 憑證申請安裝'
disqus: hackmd
---
Apache SSL 憑證申請安裝
===

## Table of Contents

[TOC]

> [!NOTE]
> 建立私鑰RSA 2048 、 產生憑證請求檔(CSR)、


# 建立私鑰 Server.key

## OpenSSL 工具
### Download
● Linux:
```command
sudo apt-get install openssl
```
● Windows (Chocolatey):
```command
choco install openssl
```
● Windows (Scoop):
```command
scoop install openssl
```
參考文獻: [Linux](https://ioflood.com/blog/install-openssl-command-linux/#:~:text=In%20most%20Linux%20distributions%2C%20OpenSSL,with%20the%20command%2C%20openssl%20version%20.&text=This%20command%20will%20install%20OpenSSL,the%20tip%20of%20the%20iceberg.) 、 [Windows](https://monovm.com/blog/install-openssl-on-windows/)
### Location & Version
預設下載位置在 /usr/local/ssl/bin
```command
find /-name openssl -print #找出工具安裝位置
```
● 查詢openssl 版本
```command
openssl version
```
> 建議使用ver.1.0.2beta之後版本，1.0.1~1.0.2受到Heartbleed Bug 影響

## 建立Server.key 私鑰檔案
### 1. 產生RSA 私鑰
```command
openssl genrsa -des3 -out server.key 2048
```
> `genrsa`: 表示產生RSA 金鑰的操作。
> `-des3`: 表示使用3DES 加密演算法對產生的私密金鑰進行加密，並要求使用者輸入密碼來保護私密金鑰。
> `-out server.key`: 指定產生的私鑰的輸出檔名為 server.key
> `2048`: 指定產生的RSA 金鑰的長度為2048 位元。 #國際密碼學規範需使用2048bits以上金鑰

OpenSSL 會產生一個有密碼保護的RSA 私鑰檔案`server.key`，並使用3DES 演算法對私鑰進行加密。產生金鑰後會要求輸入密碼(PEM pass phase)
```command
Enter PEM pass phrase:
Verifying password - Enter PEM pass phrase:
```
以後使用私鑰時都會需要輸入。\
![image](https://hackmd.io/_uploads/H1PWsewpa.png)


### 2. 產生憑證請求檔(CSR)
憑證請求檔(CSR)通常會發送給證書頒發機構(CA) 以獲取相應的數位證書，以便在伺服器上使用HTTPS 等安全協定。
```command
openssl req -new -key server.key -out certrequest.txt
```
> `openssl`: OpenSSL 工具的命令列工具。
`req`: 表示產生憑證簽發要求的操作。
`-new`: 表示建立一個新的憑證要求。
`-key server.key`: 指定私鑰檔案server.key用於產生憑證請求。
`-out certrequest.txt`: 指定產生的憑證要求的輸出檔案名稱certrequest.txt。

產出過程中需要進行身分驗證:
```commad
Country Name (2 letter code) [AU]: 國家名稱 (ex.TW)
State or Province Name (full name) [Some-State]: 州或省的名稱 (ex.Taiwan)
Locality Name (eg, city) []: 城市名稱 (ex.Taipei)
Organization Name (eg, company) [Internet Widgits Pty Ltd]: 組織名稱 (ex.Google)
Organizational Unit Name (eg, section) []: 部門名稱 (ex.Information)
Common Name (e.g. server FQDN or YOUR name) []: 網域名稱 (ex. www.google.com)
Email Address []: 電子信箱 (ex. yahoo@gmail.com)

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []: 若不需要按Enter
An optional company name []: 若不需要按Enter
```

3. 查看RSA 內容
```command
openssl req -noout -text -in certrequest.txt
```
會列出
```
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: C = TW, ST = Taiwan, L = Taipei, O = Google, OU = Information, CN = www.google.com, emailAddress = yahoo@gmail.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:ce:5d:40:84:dc:82:bd:25:43:bb:cf:57:26:3b:
                    d0:67:af:71:0a:cd:54:53:ca:f9:a7:e7:f0:82:78:
                    ~~
等等
```

# 將certrequest.txt 送至證書頒發機構(CA)
依照證書頒發機構填寫 個人/公司/法人 資料
如中華電信:
![image](https://hackmd.io/_uploads/BkHRjew6p.png)\
Reference: https://publicca.hinet.net/documents.htm 

# 根憑證/中繼憑證 安裝設定 
> [!TIP]
> 在取得CA認證的Public CA 簽發憑證後

## 取得eCA自簽憑證 (根憑證 ROOTeCA)

中華電信公開金鑰基礎建設:\ [中華電信自簽憑證](http://epki.com.tw/download/ROOTeCA_64.crt)

![image](https://hackmd.io/_uploads/ry6ECO6pa.png)

## 取得Public CA 憑證鏈 (中繼CA憑證)
中華電信公開金鑰基礎建設: [中華電信自身通用憑證](http://epki.com.tw/download/PublicCA2_64.crt)\
![image](https://hackmd.io/_uploads/H1b4ktpaa.png)

## 匯出 ROOTeCA(根憑證) & Public CA(中繼CA憑證)
### 1. PublicCA2_64 > 詳細資料 > 簽發者

![image](https://hackmd.io/_uploads/SkIb-taTT.png)
> 複製到檔案
### 2. 憑證匯出精靈
![image](https://hackmd.io/_uploads/HkAQZtpTa.png)
> 下一步(N)

### 3. 匯出格式設定
![image](https://hackmd.io/_uploads/Bk5a-YTTp.png)
> 選擇 :red_circle:  密碼編譯訊息語法標準- PKCS #7 憑證 (.P7B)
> 勾選 :ballot_box_with_check: 如果可能的話，包含憑證路徑中的所有憑證

### 4. 另存 .P7B 檔
![image](https://hackmd.io/_uploads/BJSSXta66.png)
> 存檔

![image](https://hackmd.io/_uploads/SyvImFpTp.png)

### 5. 完成憑證匯出
![image](https://hackmd.io/_uploads/S1JcQKapT.png)
> 已成功匯出並串聯 ROOTeCA(根憑證) & Public CA(中繼CA憑證)
![image](https://hackmd.io/_uploads/ryEpEFTpT.png)

## Apache 憑證匯入 & 設定

### 1. 將已匯出的.P7B 匯入Apache
將完成匯出的.P7B 複製到 Apache Server 
> :spiral_note_pad: 使用FTP需要以Binary格式傳輸

### 2. Apache console
#### 2.1 格式轉換
原憑證(.P7B檔)為DER encode，需要轉換成PEM encode (Base64)
```command
openssl pkcs7 -in eCA_PublicCA.p7b -inform DER -print_certs -out eCA_PublicCA.pem
```
> `openssl`: OpenSSL 工具的命令列工具。
`pkcs7`: 表示PKCS#7 相關操作。
`-in eCA_PublicCA.p7b`: 指定輸入的PKCS#7 檔案名為eCA_PublicCA.p7A。
`-inform DER`: 指定輸入檔的格式為DER 格式。PKCS#7 檔案可以以DER 或PEM 格式儲存。
`-print_certs`: 指示輸出列印PKCS#7 檔案中所包含的憑證。
`-out eCA_PublicCA.pem`: 指定輸出檔案的檔案名為eCA_PublicCA.pem，並將憑證以PEM 格式儲存。

![image](https://hackmd.io/_uploads/SJcadKp6T.png)

#### 2.2 將產出的PEM encode 憑證移置/apache/conf/ssl.crt
● Linux:
```command
cp eCA_PublicCA.pem C:\xampp\apache\conf\ssl.crt\ca.crt
```
● Windows:
```command
copy eCA_PublicCA.pem C:\xampp\apache\conf\ssl.crt\ca.crt
```

> :diamond_shape_with_a_dot_inside: C:\xampp\apache\conf\ssl.crt 依照自己Apache不同環境選擇不同資料夾

### 3 編輯 ssl.conf 中的 SSLCertificateChainFile
編輯 httpd-ssl.conf 設定檔
```command
vi ~apache/conf/extra/httpd-ssl.conf
```
httpd-ssl 會根據不同環境存在不同的資料夾中
(大多都在apache\conf 資料夾底下)
```conf=158
#   Server Certificate Chain:
#   Point SSLCertificateChainFile at a file containing the
#   concatenation of PEM encoded CA certificates which form the
#   certificate chain for the server certificate. Alternatively
#   the referenced file can be the same as SSLCertificateFile "conf/ssl.crt/server.crt"
#   certificate for convenience.
SSLCertificateChainFile "/conf/ssl.crt/yougood-ca.crt"
#上步驟申請的ca.crt
```
> 將SSLCertificateChainFile取消註解，加入上步驟產出的CA.CRT路徑

# SSL伺服器憑證安裝
## 1. server.cer格式轉換
:::warning
**若收到的CA簽發的cer檔為PEM encode，則可忽略此步驟1。**
驗證方式: 使用文字編輯器開啟cer檔
1.顯示亂碼: DER encode
2.顯示BEGIN CERTIFICATE開頭: PEM encode
:::
原憑證(.cer)為DER encode，需要轉換成PEM encode (Base64)
```command
openssl x509 -in server.cer -inform DER -out server.pem
```
> `openssl`: OpenSSL 工具的命令列工具。
`x509`: 表示X.509 相關操作。
`-in server.cer`: 指定輸入的X.509 憑證檔案名稱server.cer。
`-inform DER`: 指定輸入檔的格式為DER 編碼。X.509 憑證可以以DER 或PEM 格式儲存。
`-out server.pem`: 指定輸出檔案的檔案名為server.pem，並將憑證以PEM 格式儲存。

## 2. 將產出的.cer/.PEM 憑證移置/apache/conf/ssl.crt
● Linux (.cer):
```command
cp server.cer C:\xampp\apache\conf\ssl.crt\server.crt
```
● Windows (.cer):
```command
copy server.cer C:\xampp\apache\conf\ssl.crt\server.crt
```
● Linux (.pem):
```command
cp server.pem C:\xampp\apache\conf\ssl.crt\server.crt
```
● Windows (.pem):
```command
copy server.pem C:\xampp\apache\conf\ssl.crt\server.crt
```

> :diamond_shape_with_a_dot_inside: C:\xampp\apache\conf\ssl.crt 依照自己Apache不同環境選擇不同資料夾

![image](https://hackmd.io/_uploads/rkeXa5TTT.png)

## 3. 編輯 ssl.conf
### 3.1 編輯 ssl.conf 中的 SSLCertificateFile
編輯 httpd-ssl.conf 設定檔
```command
vi ~apache/conf/extra/httpd-ssl.conf
```
httpd-ssl 會根據不同環境存在不同的資料夾中
(大多都在apache\conf 資料夾底下)
```conf=134
#   Server Certificate:
#   Point SSLCertificateFile "conf/ssl.crt/server.crt"
#   the certificate is encrypted, then you will be prompted for a
#   pass phrase.  Note that a kill -HUP will prompt again.  Keep
#   in mind that if you have both an RSA and a DSA certificate you
#   can configure both in parallel (to also allow the use of DSA
#   ciphers, etc.)
#   Some ECC cipher suites (http://www.ietf.org/rfc/rfc4492.txt)
#   require an ECC certificate which can also be configured in
#   parallel.
SSLCertificateFile "conf/ssl.crt/yougood-server.crt" 
#上步驟申請的server.crt
#SSLCertificateFile "conf/ssl.crt/server.crt"
```
> 將SSLCertificateFile編輯成上步驟產出的SERVER.CRT路徑

### 3.1 編輯 ssl.conf 中的 SSLCertificateKeyFile
:::info
將步驟 [**1. 產生RSA 私鑰**](https://hackmd.io/@CHW/Skyhc1v6T#1-%E7%94%A2%E7%94%9FRSA-%E7%A7%81%E9%91%B0) 產生的server.key複製到/apache/conf/ssl.key下
:::
編輯 httpd-ssl.conf 設定檔
```command
vi ~apache/conf/extra/httpd-ssl.conf
```
httpd-ssl 會根據不同環境存在不同的資料夾中
(大多都在apache\conf 資料夾底下)
```conf=148
#   Server Private Key:
#   If the key is not combined with the certificate, use this
#   directive to point at the key file.  Keep in mind that if
#   you've both a RSA and a DSA private key you can configure
#   both in parallel (to also allow the use of DSA ciphers, etc.)
#   ECC keys, when in use, can also be configured in parallel
SSLCertificateKeyFile "conf/ssl.key/yougood-server.key"
#SSLCertificateKeyFile "conf/ssl.key/server.key"
#SSLCertificateKeyFile "conf/ssl.key/server.key"
```
> 將SSLCertificateKeyFile 編輯成匯入的SERVER.KEY路徑

:::danger
若伺服器無法重啟遇到以下相關ERROR，無法重啟:
AH02577: Init: SSLPassPhraseDialog builtin is not supported on Win32 (key file C:/xampp/apache/conf/ssl.key/yougood-server.key)
AH02311: Fatal error initialising mod_ssl, exiting. See C:/xampp/apache/logs/error.log for more information
AH02564: Failed to configure encrypted (?) private key www.example.com:443:0, check C:/xampp/apache/conf/ssl.key/yougood-server.key
SSL Library Error: error:04800068:PEM routines::bad password read -- You entered an incorrect pass phrase!?
SSL Library Error: error:1E08010C:DECODER routines::unsupported (No supported data to decode.  Input type: DER, Input structure: type-specific)
SSL Library Error: error:068000A8:asn1 encoding routines::wrong tag
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error (Field=version, Type=RSAPrivateKey)
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error (Field=version, Type=PKCS8_PRIV_KEY_INFO)
SSL Library Error: error:1E08010C:DECODER routines::unsupported (No supported data to decode.  Input type: DER, Input structure: type-specific)
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error (Type=RSAPrivateKey)
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error (Type=PKCS8_PRIV_KEY_INFO) 
:::
可能是 SSL 金鑰格式錯誤。先確保SSL金鑰檔案是正確的PEM 格式，並且密碼pass phrase正確。
可使用以下command，轉換key格式 (DER > PEM)
```
openssl rsa -inform DER -outform PEM -in C:/xampp/apache/conf/ssl.key/server.key -out C:/xampp/apache/conf/ssl.key/server.pem
```

# 重啟 Apache (完成安裝)
```command
/apache/bin/apachectl stop
/apache/bin/apachectl start
```
:ballot_box_with_check: 確認伺服器&防火牆有開啟對應的 HTTPS PORT (443)
:::success
重啟後瀏覽伺服器，確認完成SSL安裝
:::
![image](https://hackmd.io/_uploads/HJSct6pTa.png)

###### tags: `Apache` `SSL` `TLS`
