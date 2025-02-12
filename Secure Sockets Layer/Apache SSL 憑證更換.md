---
title: 'Apache SSL 憑證更換'
disqus: hackmd
---
Apache SSL 憑證更換
===

>[!Note]
> SSL憑證具時效性，本篇只介紹如何更換
> 若是新服務申請憑證，可以參考 [Apache SSL 憑證申請安裝](https://hackmd.io/@CHW/Skyhc1v6T)

# [Apache SSL 憑證申請安裝](https://hackmd.io/@CHW/Skyhc1v6T) 

# 原本的私鑰 (server.key) 產生新的 CSR
如果更換憑證，通常需要重新生成 CSR ，並提供給 憑證頒發機構 (CA) 來取得新的憑證。\
私鑰位置 (`C:\xampp\apache\conf\ssl.key\{private.key}`)，

>[!Tip]
>若以上路徑沒有，也可以到 `C:\xampp\apache\conf\extra\httpd->ssl.conf`中看設定檔：\
>`SSLCertificateKeyFile "{Private key path}"`

```
openssl req -new -key {private.key} -out certrequest.csr
```
> 將 private key 生成 CSR，重新提供給憑證頒發機構 (CA)

# CSR 提交給憑證頒發機構 (CA)
以中華電信為例:\
![image](https://hackmd.io/_uploads/HJMj6ZcY1l.png)


# 等待中...

###### tags: `Apache` `SSL` `TLS`
