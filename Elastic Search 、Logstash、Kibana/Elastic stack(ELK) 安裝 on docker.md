---
title: 'Elastic stack(ELK) 安裝 on docker'
disqus: hackmd
---
Elastic stack(ELK) 安裝 on docker [一鍵安裝]
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

# 安裝
```command
git clone https://github.com/deviantony/docker-elk.git
```
![image](https://hackmd.io/_uploads/BJ9wt4Lca.png)

```command
// 初始化 docker-elk 所需的 Elasticsearch user和group
cd docker-elk
docker-compose up setup
```
![image](https://hackmd.io/_uploads/rywlq4L5a.png)

```command
// 自動創建&啟動docker
docker-compose up
```
在Docker Desktop可以看到Elasticsearch、Logstash、Kibana都裝好了
![image](https://hackmd.io/_uploads/BJ4mbuDca.png)

:::success
:bulb: 完成後，在Kibana介面上建立index-pattern & 視覺化
:::

# 初始設定
瀏覽 http://localhost:5601/
![image](https://hackmd.io/_uploads/S100-_wca.png)
> 預設帳號密碼:
> elastic
> changeme

![image](https://hackmd.io/_uploads/H1fQGdv56.png)
● [Security settings in Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html): 身分驗證設定

## 更改預設密碼
● 重置 elastic password
```command
docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user elastic
```
![image](https://hackmd.io/_uploads/HJAvXuv9a.png)
> New value: 78phu4bMjCbhJtyrNJ7C

● 重置 logstash_internal password
```command
docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user logstash_internal
```
![image](https://hackmd.io/_uploads/ryjxVdDqp.png)
> New value: adj3hGgN-tup75T9Yv-2

● 重置 kibana_system password
```command
docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user kibana_system
```
![image](https://hackmd.io/_uploads/BJXVV_vqp.png)
> New value: YapaMJlVP+fMYvKnoR62 

● 更新.env password
雖然這個密碼在核心套件中沒有被使用，但是一些擴展（可能是指套件或其他應用程式）需要它來連接到Elasticsearch。
![image](https://hackmd.io/_uploads/H1neYdv9a.png)

● 重啟 Logstash 和 Kibana 更新設定 
```command
docker-compose up -d logstash kibana
```
![image](https://hackmd.io/_uploads/rkMFFdv56.png)

> [!TIP]
>Elasticsearch 配置在 elasticsearch/config/elasticsearch.yml\
>Logstash 配置在 logstash/config/logstash.yml\
>Kibana 配置在 kibana/config/kibana.yml

# Reference:
https://github.com/deviantony/docker-elk

###### tags: `ELK` `Elasticsearch` `Logstash` `Kibana`
