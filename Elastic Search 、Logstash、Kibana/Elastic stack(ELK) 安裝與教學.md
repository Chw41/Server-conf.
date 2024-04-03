---
title: 'Elastic stack(ELK) 安裝與教學'
disqus: hackmd
---
Elastic stack(ELK) 安裝與教學
===

## Table of Contents

[TOC]

> [!NOTE]
>:bulb: ELK = Elasticsearch + Logstash + Kibana


# ELK 介紹


● **Elasticsearch**：
Elasticsearch為分散式、RESTful 的搜尋及分析的搜尋引擎，能解決與日俱增的使用案例 ，做為 ELK 的核心，Elasticsearch 能集中儲存資料以便於做快速搜尋，並做關聯性的微調，及能輕易擴展的分析功能。如此強大的性能使 Elasticsearch 在用戶中獲得很高的評價跟人氣，公司也因此聲名大噪 。\
![image](https://hackmd.io/_uploads/HJegyXU5p.png)

● **Logstash**：
Logstash 是一種輕量型且開源的伺服器端資料處理管道，能夠從多個資料源擷取、轉換、再將其傳送至您的「存放區」。即使資料複雜多元，Logstash 依然能靈活運用動態搜索以找到正確資料，並即時轉換、傳送到指定的儲存裝置，且不受資料源格式和架構影響。\
● **Kibana**：
是一套免費開源的前端應用程式，將 ELK作為其基礎，為 Elasticsearch 裡的數據資料提供搜尋和可視化的功能， Kibana 不僅是 ELK 的繪圖工具，也能作為儀表板，用於監控、管理和維護 ELK 集群，甚至能作為 ELK 開發及解決方案的彙整中心。

> Ref: 
> https://www.omniwaresoft.com.tw/product-news/elastic-news/elk-what-is-elk-stack/
> https://www.youtube.com/watch?v=u6hD_2gLTa4&ab_channel=TPIsoftware

# Elasticsearch
## 下載安裝 
Download Elasticsearch: 
https://www.elastic.co/downloads/elasticsearch
>**將下載的Elasticsearch解壓縮到的指定的目錄中(ELK三個套件需在同一路徑下)**
> C:\Users\User\AIS3 EOF_ELK\
## 啟動
個人使用: Windows環境 PowerShell

```command
cd elasticsearch-8.12.0\bin
.\elasticsearch
```
![image](https://hackmd.io/_uploads/HJshh-H5a.png)

瀏覽 http://localhost:9200/
```
{
  "name" : "DESKTOP-CHW",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "~~~~",
  "version" : {
    "number" : "8.12.0",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "~hash",
    "build_date" : "2024-01-11T10:05:27.953830042Z",
    "build_snapshot" : false,
    "lucene_version" : "9.9.1",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

> 若顯示 http client did not trust this server's certificate
![image](https://hackmd.io/_uploads/B1TOgzHqT.png)
瀏覽器不信任伺服器(沒有SSL憑證)
在/config/ elasticsearch.yml中，可以設定伺服器在禁用 SSL 的情況下運作
> ```
>$ sudo vi /config/elasticsearch.yml
># 將xpack.security.http.ssl預設改成false
>
>xpack.security.http.ssl:
>enabledL fasle
> ```
>![image](https://hackmd.io/_uploads/HyXx4GB96.png)

> 若顯示帳號/密碼 輸入
> ![image](https://hackmd.io/_uploads/S1uaNfH5p.png)
> 可在/config/elasticsearch.yml 關閉密碼驗證模式
> ```
>$ sudo vi /config/elasticsearch.yml
># 將xpack.security.enabled預設改成false
>
>xpack.security.http.ssl:
>enabledL fasle
> ```
> ![image](https://hackmd.io/_uploads/r1VCHGH5p.png)

# Logstash
## 下載安裝
Download Logstash: 
https://www.elastic.co/downloads/logstash
>**將下載的Logstash解壓縮到的指定的目錄中(ELK三個套件需在同一路徑下)**
> C:\Users\User\AIS3 EOF_ELK\

## 更改設定檔
設定 logstash 如何解析 log 檔案
```command
cd logstash-8.12.0\bin
mkdir conf
cd conf
vi logstash-indexer.conf
```
logstash-indexer.conf
```conf=
# 輸入層：指定log讀取位置(可以多個文件同時讀取)，可依照你 log 檔的位置撰寫
input {
 file {
   path => "/Users/User/Desktop/CyberSec/2024 AIS3 EOF/決賽/eof_final.log"
   start_position => "beginning"
  }
}
# 解析層：指定解析欄位方式
filter {
}
# 輸出層：指定elasticsearch存放的主機位置與index名稱
output {
  elasticsearch {
        hosts => ["localhost:9200"]
        index => "mylocal_logstash"
  }
  stdout {codec => rubydebug}
}
```
## 啟動
```command
cd logstash-8.12.0\bin
.\logstash -f conf/logstash-indexer.conf
```

# Kibana
## 下載安裝
Download Kibana: 
https://www.elastic.co/downloads/kibana
>**將下載的Kibana解壓縮到的指定的目錄中(ELK三個套件需在同一路徑下)**
> C:\Users\User\AIS3 EOF_ELK\

## 更改設定檔
刪除註解elasticsearch 的主機位置
```command
cd kibana-8.12.0\config
vi kibana.yml
```
![image](https://hackmd.io/_uploads/S1K6ab896.png)

## 啟動
```command
cd kibana-8.12.0\bin
.\kibana.bat
```
連線 http://localhost:5601/ 可以成功看到Kibana介面
![image](https://hackmd.io/_uploads/r1fA44LqT.png)

> [!IMPORTANT]
:bulb: 完成後，在Kibana介面上建立index-pattern & 視覺化


###### tags: `ELK` `Elasticsearch` `Logstash` `Kibana`
