---
title: 'LNMP (Linux + Nginx + MySQL + PHP) 架設伺服器'
disqus: hackmd
---

LNMP (Linux + Nginx + MySQL + PHP) 
架設伺服器
===

## Table of Contents

[TOC]

## General Setup

If you are a total beginner to this, start here!

1. (Linux) Ubuntu 20.4.1
 `lsb_release -a`\
 ![](https://hackmd.io/_uploads/H1oc4oEjh.png)
 
2. Nginx 1.18.0
`nginx -v`\
 ![](https://hackmd.io/_uploads/H1F3EoNo3.png)

4. MySQL 8.0.30-0ubuntu0.22.04.2
 `mysql -v`\
 ![](https://hackmd.io/_uploads/ry2RNi4sn.png)

5. Php 7.4.3
 `php -v`\
 ![](https://hackmd.io/_uploads/SJPEBjNsh.png)



---
## Server DEMO

http://10.250.128.133/ \
 ![](https://hackmd.io/_uploads/rkrKSs4o3.png)
### 1. Login
 ![](https://hackmd.io/_uploads/ryP_LoEi3.png)
 
### 2. Show MySQL Database (contacts information)
    sudo apt-get install php-mysqlnd
> **(WEB)**\
 ![](https://hackmd.io/_uploads/Hy9s8iVo2.png)

>**(SSH)**\
 ![](https://hackmd.io/_uploads/ry7CLjNo3.png)

### 3. Contacts Information Page
 ![](https://hackmd.io/_uploads/S17k_sVj3.png)

### 4. Add/Edit Contacts
 

|                Add New Contact                | Edit |
|:---------------------------------------------:|:----:|
| ![](https://hackmd.io/_uploads/SyUadj4ih.png) |   ![](https://hackmd.io/_uploads/Byj6_oVo3.png)   |

### 5. Delete Contacts
 ![](https://hackmd.io/_uploads/HJWftsEj3.png)

### 6. Nginx Load Balance

```gherkin=
http {

  # At least 2 servers.
  upstream loadbalancer{
      server [server1 IP] weight=1;
      server [server2 IP] weight=1;
  
  }
   
```

```gherkin=
location / {

  proxy_pass http://loadbalancer;
  try_files $uri $uri/ =404;
  
  }
```

| Server1 | Server2 |
|:----:|:----:|
|![](https://hackmd.io/_uploads/SyWH2jEo3.png)|![](https://hackmd.io/_uploads/S1Ww3iEs2.png)


Vulnerability Scan & Fix
---
### 1. OWASP ZAP scanner
 > https://www.zaproxy.org/

●Result:\
 ![](https://hackmd.io/_uploads/S1KUpsVin.png) \
●Alerts:\
 ![](https://hackmd.io/_uploads/rkRtTjVin.png)

### 2. Fix vulnerabilities
#### (1) Content Security Policy (CSP) Header Not Set
  ![](https://hackmd.io/_uploads/SJtll0Vjh.png)

> **etc/nginx/sites-available/default
>    add_header Content-Security-Policy "default-src 'self';"** 

| ![](https://hackmd.io/_uploads/B1TIZANsn.png)  | ![](https://hackmd.io/_uploads/HJsP-CEjh.png)  | 



####  (2) Missig Anti-clickjacking Header
  ![](https://hackmd.io/_uploads/H1lFzCEs2.png)

> **/etc/nginx/conf.d
>     add_header X-Frame-Options sameorigin always;**

| ![](https://hackmd.io/_uploads/S1TqzANs2.png) | ![](https://hackmd.io/_uploads/BkBiM0Eo3.png)  | 
 
####  (3) Cookie No HttpOnly Flag
 ![](https://hackmd.io/_uploads/ryj_QRNon.png)

> **/etc/nginx/sites-available/default**
 ```gherkin=
location ~ \.php$ {
  ...
  add_header Set-Cookie "Path=/; HttpOnly; Secure";
  proxy_cookie_path / "/; HTTPOnly; Secure";
  
  }
```
 | ![](https://hackmd.io/_uploads/BJHKE0Vs2.png)| ![](https://hackmd.io/_uploads/SJl5N0Ei2.png)|

 
####  (4) Cookie without SameSite Attribute
 ![](https://hackmd.io/_uploads/H1c3VRVih.png)

> **/etc/nginx/sites-available/default
>     add_header Set-Cookie "Path=/; HttpOnly; Secure; SameSite=Lax"**

 |![](https://hackmd.io/_uploads/B1KrHAVs3.png)|![](https://hackmd.io/_uploads/HJHLSAVi3.png)|

####  (5) X-Content-Type-Options Header Missing
 ![](https://hackmd.io/_uploads/B1cQURVin.png)

> **/etc/nginx/sites-available/default
>     add_header X-Content-Type_Options nosniff;**

 |![](https://hackmd.io/_uploads/HyQ5URNih.png)|![](https://hackmd.io/_uploads/rJ0cI0Vsh.png)|
 

Database Audit (Enable MySQL Query Log)
---

> **/var/lib/mysql/
>     create mysql.log**

 ```gherkin=
mysql > Setting

    SET GLOBAL general_log = 'ON';
	SET GLOBAL general_log_file ='/var/lib/mysql/mysql.log';

```
Test:\
![](https://hackmd.io/_uploads/HJU3dAEjh.png)

###  The time difference between database audit is enabled or not.

#### ●Enable general_log -> Scaning Time :12.399 (s)
|![](https://hackmd.io/_uploads/BJd85R4in.png)|![](https://hackmd.io/_uploads/H1RI5AVsn.png)|

#### ●Disable general_log -> Scaning Time :9.788 (s)
|![](https://hackmd.io/_uploads/rJuY5A4i2.png)|![](https://hackmd.io/_uploads/BkptqAEsn.png)|

> [!NOTE]
>可以看出general_log開啟前後，在OWASP ZAP scanner掃描時間的差異。

###### tags: `LNMP` `Ubunutu` `Nginx` `MySQL` `PHP`
