---
title: 'Apache SSL 憑證更換'
disqus: hackmd
---
Apache SSL 憑證更換
===

>[!Note]
> SSL憑證具時效性，本篇只介紹如何更換
> 若是新服務申請憑證，可以參考 [Apache SSL 憑證申請安裝](https://github.com/Chw41/Server-conf./tree/main/Secure%20Sockets%20Layer)

# [Apache SSL 憑證申請安裝](https://hackmd.io/@CHW/Skyhc1v6T) 

# 向憑證頒發機構 (CA) 購買新的SSL憑證 
## 原本的私鑰 (server.key) 產生新的 CSR
如果更換憑證，通常需要重新生成 CSR ，並提供給 憑證頒發機構 (CA) 來取得新的憑證。\
私鑰位置 (`C:\xampp\apache\conf\ssl.key\{private.key}`)，

>[!Tip]
>若以上路徑沒有，也可以到 `C:\xampp\apache\conf\extra\httpd\ssl.conf`中看設定檔：\
>`SSLCertificateKeyFile "{Private key path}"`

```
openssl req -new -key {private.key} -out certrequest.csr
```
> 將 private key 生成 CSR，重新提供給憑證頒發機構 (CA)

## CSR 提交給憑證頒發機構 (CA)
以中華電信為例:\
![image](https://hackmd.io/_uploads/HJMj6ZcY1l.png)

# [Unexpected twist] CA 審核需要驗證 DNS TXT tag
將憑證頒發機構提供的 `DN_CHECK_FILE.htm`: JSON text data 放到以下路徑:
`C:\xampp\htdocs\.well-known\pki-validation`

```
PS C:\xampp\htdocs\.well-known\pki-validation> type .\DN_CHECK_FILE.htm
{"randomNumber":"{randomNumber}","expireTime":"2025-02-30T10:39:43"}
```
## 驗證 TXT tag 設定狀態

### 1. [digwebinterface](https://www.digwebinterface.com/?hostnames=XXXXX&type=TXT&useresolver=8.8.4.4&ns=auth&nameservers): 線上 DNS 查詢工具
- DNS 記錄查詢：支援 `A`、`AAAA`、`CNAME`、`MX`、`NS`、`TXT`、`SOA`、`PTR` 多種 record 類型。
- 跨伺服器查詢：可選擇不同的 DNS Server（如 Google DNS、Cloudflare DNS、OpenDNS）進行比對。
- 反向解析：查詢 IP address 對應的 domain（PTR record）。
- 故障診斷: DNS 解析問題

![image](https://hackmd.io/_uploads/Bk0ZpV79kg.png)

### 2. [dig cmd](https://en.wikipedia.org/wiki/Dig_(command))
dig 全名: Domain Information Groper
首先在 Debian/Ubuntu 系統上安裝 `bind9-dnsutils`，其中包含 dig、host、nslookup 等 DNS 查詢工具
```
 sudo apt install bind9-dnsutils
```
使用 `-h` 參數查看使用方法:
```
chw@CHW:$ dig -h
Usage:  dig [@global-server] [domain] [q-type] [q-class] {q-opt}
            {global-d-opt} host [@local-server] {local-d-opt}
            [ host [@local-server] {local-d-opt} [...]]
Where:  domain    is in the Domain Name System
        q-class  is one of (in,hs,ch,...) [default: in]
        q-type   is one of (a,any,mx,ns,soa,hinfo,axfr,txt,...) [default:a]
                 (Use ixfr=version for type ixfr)
        q-opt    is one of:
                 -4                  (use IPv4 query transport only)
                 -6                  (use IPv6 query transport only)
                 -b address[#port]   (bind to source address/port)
                 -c class            (specify query class)
                 -f filename         (batch mode)
                 -k keyfile          (specify tsig key file)
                 -m                  (enable memory usage debugging)
                 -p port             (specify port number)
                 -q name             (specify query name)
                 -r                  (do not read ~/.digrc)
                 -t type             (specify query type)
                 -u                  (display times in usec instead of msec)
                 -x dot-notation     (shortcut for reverse lookups)
                 -y [hmac:]name:key  (specify named base64 tsig key)
        d-opt    is of the form +keyword[=value], where keyword is:
                 +[no]aaflag         (Set AA flag in query (+[no]aaflag))
                 +[no]aaonly         (Set AA flag in query (+[no]aaflag))
                 +[no]additional     (Control display of additional section)
                 +[no]adflag         (Set AD flag in query (default on))
                 +[no]all            (Set or clear all display flags)
                 +[no]answer         (Control display of answer section)
                 +[no]authority      (Control display of authority section)
                 +[no]badcookie      (Retry BADCOOKIE responses)
                 +[no]besteffort     (Try to parse even illegal messages)
                 +bufsize[=###]      (Set EDNS0 Max UDP packet size)
                 +[no]cdflag         (Set checking disabled flag in query)
                 +[no]class          (Control display of class in records)
                 +[no]cmd            (Control display of command line -
                                      global option)
                 +[no]comments       (Control display of packet header
                                      and section name comments)
                 +[no]cookie         (Add a COOKIE option to the request)
                 +[no]crypto         (Control display of cryptographic
                                      fields in records)
                 +[no]defname        (Use search list (+[no]search))
                 +[no]dns64prefix    (Get the DNS64 prefixes from ipv4only.arpa)
                 +[no]dnssec         (Request DNSSEC records)
                 +domain=###         (Set default domainname)
                 +[no]edns[=###]     (Set EDNS version) [0]
                 +ednsflags=###      (Set EDNS flag bits)
                 +[no]ednsnegotiation (Set EDNS version negotiation)
                 +ednsopt=###[:value] (Send specified EDNS option)
                 +noednsopt          (Clear list of +ednsopt options)
                 +[no]expandaaaa     (Expand AAAA records)
                 +[no]expire         (Request time to expire)
                 +[no]fail           (Don't try next server on SERVFAIL)
                 +[no]header-only    (Send query without a question section)
                 +[no]https[=###]    (DNS-over-HTTPS mode) [/]
                 +[no]https-get      (Use GET instead of default POST method while using HTTPS)
                 +[no]http-plain[=###]    (DNS over plain HTTP mode) [/]
                 +[no]http-plain-get      (Use GET instead of default POST method while using plain HTTP)
                 +[no]identify       (ID responders in short answers)
                 +[no]idnin          (Parse IDN names [default=on on tty])
                 +[no]idnout         (Convert IDN response [default=on on tty])
                 +[no]ignore         (Don't revert to TCP for TC responses.)
                 +[no]keepalive      (Request EDNS TCP keepalive)
                 +[no]keepopen       (Keep the TCP socket open between queries)
                 +[no]multiline      (Print records in an expanded format)
                 +ndots=###          (Set search NDOTS value)
                 +[no]nsid           (Request Name Server ID)
                 +[no]nssearch       (Search all authoritative nameservers)
                 +[no]onesoa         (AXFR prints only one soa record)
                 +[no]opcode=###     (Set the opcode of the request)
                 +padding=###        (Set padding block size [0])
                 +qid=###            (Specify the query ID to use when sending queries)
                 +[no]qr             (Print question before sending)
                 +[no]question       (Control display of question section)
                 +[no]raflag         (Set RA flag in query (+[no]raflag))
                 +[no]rdflag         (Recursive mode (+[no]recurse))
                 +[no]recurse        (Recursive mode (+[no]rdflag))
                 +retry=###          (Set number of UDP retries) [2]
                 +[no]rrcomments     (Control display of per-record comments)
                 +[no]search         (Set whether to use searchlist)
                 +[no]short          (Display nothing except short
                                      form of answers - global option)
                 +[no]showbadcookie  (Show BADCOOKIE message)
                 +[no]showsearch     (Search with intermediate results)
                 +[no]split=##       (Split hex/base64 fields into chunks)
                 +[no]stats          (Control display of statistics)
                 +subnet=addr        (Set edns-client-subnet option)
                 +[no]tcflag         (Set TC flag in query (+[no]tcflag))
                 +[no]tcp            (TCP mode (+[no]vc))
                 +timeout=###        (Set query timeout) [5]
                 +[no]tls            (DNS-over-TLS mode)
                 +[no]tls-ca[=file]  (Enable remote server's TLS certificate validation)
                 +[no]tls-hostname=hostname (Explicitly set the expected TLS hostname)
                 +[no]tls-certfile=file (Load client TLS certificate chain from file)
                 +[no]tls-keyfile=file (Load client TLS private key from file)
                 +[no]trace          (Trace delegation down from root [+dnssec])
                 +tries=###          (Set number of UDP attempts) [3]
                 +[no]ttlid          (Control display of ttls in records)
                 +[no]ttlunits       (Display TTLs in human-readable units)
                 +[no]unknownformat  (Print RDATA in RFC 3597 "unknown" format)
                 +[no]vc             (TCP mode (+[no]tcp))
                 +[no]yaml           (Present the results as YAML)
                 +[no]zflag          (Set Z flag in query)
        global d-opts and servers (before host name) affect all queries.
        local d-opts and servers (after host name) affect only that lookup.
        -h                           (print help and exit)
        -v                           (print version and exit)
```
以 www.google.com 為例：
```
chw@CHW:$ dig www.google.com

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> www.google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2741
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.google.com.                        IN      A

;; ANSWER SECTION:
www.google.com.         265     IN      A       142.250.204.36

;; Query time: 18 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Wed Feb 19 19:21:43 CST 2025
;; MSG SIZE  rcvd: 59

chw@CHW:$ dig TXT www.google.com

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> TXT www.google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12308
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.google.com.                        IN      TXT

;; AUTHORITY SECTION:
google.com.             50      IN      SOA     ns1.google.com. dns-admin.google.com. 728129128 900 900 1800 60

;; Query time: 9 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Wed Feb 19 19:21:55 CST 2025
;; MSG SIZE  rcvd: 93

chw@CHW:$ dig TXT +noadditional +noquestion +nocomments +nocmd +nostats www.google.com
google.com.             56      IN      SOA     ns1.google.com. dns-admin.google.com. 728129128 900 900 1800 60
chw@CHW:$
```
> `TXT`：查詢 TXT 記錄（常用於 SPF、DKIM、DMARC、驗證碼等）\
`+noadditional`：不顯示額外資訊（如額外的 DNS 記錄）\
`+noquestion`：不顯示 QUESTION SECTION 部分\
`+nocomments`：不顯示註解文字\
`+nocmd`：不顯示查詢命令\
`+nostats`：不顯示查詢統計資訊（省略查詢時間、伺服器等資訊）。

# 取得 CA 核發 SSL 憑證
>[!Caution]
> 在取得 CA 認證的 Public CA 簽發憑證後

中華電信TLS憑證服務於 2024 年 11 月切換新根憑證CA: [憑證串鏈更新說明](https://chtca.hinet.net/newcatinfo.html)

>[!Note]
>CA 認證後，會取得四個憑證串鏈檔:
>1. 舊根憑證 (eCA自簽憑證，檔名: `ROOTeCA_64.crt`)
>2. 中繼轉接憑證(舊簽新根憑證，檔名: `eCA1-to-HRCA1.crt`)
>3. 中繼CA憑證（HiPKI OVCA自身憑證，檔名: `OVTLSCA1_b64.crt.crt`)
>4. 簽發給用戶的TLS伺服器軟體憑證 `{file}.cer`

# 根憑證/中繼憑證 安裝設定 
## 1. 取得 eCA自簽憑證 (根憑證 ROOTeCA)
中華電信公開金鑰基礎建設: [中華電信 eCA 憑證](https://eca.hinet.net/download/ROOTeCA_64.crt)\
![image](https://hackmd.io/_uploads/Bk4WIgIqJl.png)


## 2. 取得 eCA to HRCA 憑證鏈
中華電信公開金鑰基礎建設: [中華電信 eCA to HRCA 憑證](https://eca.hinet.net/download/eCA1-to-HRCA1.crt)\
![image](https://hackmd.io/_uploads/ByinUl85ye.png)

## 3. 取得 HiPKI OV TLS CA 憑證鏈
中華電信公開金鑰基礎建設: [中華電信 HiPKI OV TLS CA 憑證](https://eca.hinet.net/repository-h/download/OVTLSCA1_b64.crt)\
![image](https://hackmd.io/_uploads/Hk_Uvl8qJe.png)

## 4. 製作憑證鏈
(Windows WSL2)
```
$ cat OVTLSCA1_b64.crt eCA1-to-HRCA1.crt > server-ca.crt
```

# SSL伺服器憑證安裝
將 `server.key`、`server-ca.crt`、SSL 伺服器憑證(檔名: `{file}.cer`)移到特定資料夾
> :diamond_shape_with_a_dot_inside: `C:\xampp\apache\conf\ssl.*\` 依照自己 Apache 不同環境選擇不同資料夾

## 1. 將 SSL 伺服器憑證 .cer 憑證轉換成 ssl.crt
● Linux (.cer):
```command
cp {file}.cer server.crt
```
● Windows (.cer):
```command
copy {file}.cer server.crt
```

## 2. 編輯 ssl.conf
httpd-ssl 會根據不同環境存在不同的資料夾中 (大多都在 apache\conf 資料夾底下)
```
vi /xampp/apache/conf/extra/httpd-ssl.conf
```
分別編輯 `SSLCertificateFile`, `SSLCertificateKeyFile` & `SSLCertificateChainFile` (詳細步驟可參考 [Apache SSL 憑證申請安裝](https://hackmd.io/@CHW/Skyhc1v6T#3-%E7%B7%A8%E8%BC%AF-sslconf))

httpd-ssl.conf:
```
...
SSLCertificateFile "conf/ssl.crt/server.crt"
...
SSLCertificateKeyFile "conf/ssl.key/server.key"
...
SSLCertificateChainFile "/conf/ssl.crt/server-ca.crt"
...
```
>[!Caution]
若伺服器無法重啟遇到以下相關ERROR，無法重啟:\
AH02577: Init: SSLPassPhraseDialog builtin is not supported on Win32 (key file C:/xampp/apache/conf/ssl.key/{your Private key})\
AH02311: Fatal error initialising mod_ssl, exiting. See C:/xampp/apache/logs/error.log for more information\
AH02564: Failed to configure encrypted (?) private key www.example.com:443:0, check C:/xampp/apache/conf/ssl.key/{your Private key}\
SSL Library Error: error:04800068:PEM routines::bad password read -- You entered an incorrect pass phrase!?\
SSL Library Error: error:1E08010C:DECODER routines::unsupported (No supported data to decode.  Input type: DER, Input structure: type-specific)\
SSL Library Error: error:068000A8:asn1 encoding routines::wrong tag\
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error\
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error (Field=version, Type=RSAPrivateKey)\
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error (Field=version, Type=PKCS8_PRIV_KEY_INFO)\
SSL Library Error: error:1E08010C:DECODER routines::unsupported (No supported data to decode.  Input type: DER, Input structure: type-specific)\
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error (Type=RSAPrivateKey)\
SSL Library Error: error:0688010A:asn1 encoding routines::nested asn1 error (Type=PKCS8_PRIV_KEY_INFO)  


可能是 SSL 金鑰格式錯誤。先確保 SSL 金鑰檔案是正確的 PEM 格式，並且密碼 pass phrase 正確。
可使用以下 command，轉換 key 格式 (CER > PEM)
```
openssl x509 -in server.cer -inform DER -out server.pem
```

# 重啟 Apache (完成安裝)
```command
/apache/bin/apachectl stop
/apache/bin/apachectl start
```
:ballot_box_with_check: 確認伺服器&防火牆有開啟對應的 HTTPS PORT (443)

>[!Important]
重啟後瀏覽伺服器，確認完成SSL安裝

![image](https://hackmd.io/_uploads/ryPLhbL5kl.png)



###### tags: `Apache` `SSL` `TLS`
