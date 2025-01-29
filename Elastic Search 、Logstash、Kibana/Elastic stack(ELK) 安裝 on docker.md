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
>[!Important]
>**資料來源 → Logstash → Elasticsearch → Kibana**\
Logstash 收集並處理資料。
將處理好的資料送入 Elasticsearch 進行儲存和索引。
Kibana 從 Elasticsearch 獲取資料，進行可視化和分析。

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

> [!TIP]
> :bulb: 完成後，在Kibana介面上建立index-pattern & 視覺化


# 初始設定
瀏覽 http://localhost:5601/ \
![image](https://hackmd.io/_uploads/S100-_wca.png)
> 預設帳號密碼:\
> elastic\
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

● 更新.env password\
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

# 設定區網固定 IP
>[!Note]
>Environment: 在 Mac 使用 Wireguard\
>將 ELK 架在 172.27.71.7 為例：
## 編輯 docker-compose.yml
```
services:

  elasticsearch:
    ...
    ports:
      - "172.27.71.7:9200:9200"
      - "172.27.71.7:9300:9300"
    ...
    networks:
      elk:
        ipv4_address: 172.27.71.7
    ...

  logstash:
    ...
    ports:
      - "172.27.71.7:5044:5044"
      - "172.27.71.7:50000:50000/tcp"
      - "172.27.71.7:50000:50000/udp"
      - "172.27.71.7:9600:9600"
    ...
    networks:
      elk:
        ipv4_address: 172.27.71.8
    ...

  kibana:
    ...
    ports:
      - "172.27.71.7:5601:5601"
    ...
    networks:
      elk:
        ipv4_address: 172.27.71.9
    ...

networks:
  elk:
    driver: bridge
    ipam:
      config:
        - subnet: "172.27.71.0/24"
          gateway: "172.27.71.1"
```
> 1. 將 ELK 服務指定到 172.27.71.7
> 2. 自訂 dcker networks
> > 命名： elk
> > 設定子網域
>
> 3. ELK 分別自定靜態 IP (`172.27.71.7`, `172.27.71.8`, `172.27.71.9`)，避免 Docker 分配動態 IP 導致 VPN 無法識別

- 重啟 container
```
docker-compose down && docker-compose up -d
```


## 設定 VPN
1. 安裝 wireguard-tools
```
$ brew install wireguard-tools
```
>[!Note]
> 若已安裝 Ｗireguard APP，仍需要安裝 wireguard-tools\
> 且 GUI 與 CLI 設定不會同步 :face_with_finger_covering_closed_lips: \
> **App Store 上的 WireGuard App 與你在 Terminal 安裝的 wireguard-tools 是 不同的工具** 
2. 建立 wireguard conf
```
[Interface]
Address = 172.27.71.7/24
PrivateKey = <你的私鑰>
DNS = 8.8.8.8, 1.1.1.1
ListenPort = 51820

[Peer]
PublicKey = <對方的公鑰>
AllowedIPs = 172.27.71.4/32
Endpoint = <對方的公網IP>:51820
```

3. 簡化設定檔、啟動 VPN
```
$ sudo mv {wireguard conf} /opt/homebrew/etc/wireguard/wg0.conf
$ sudo wg-quick up wg0
Password:
Warning: `/opt/homebrew/etc/wireguard/wg0.conf' is world accessible
[+] Interface for wg0 is utun10
wg-quick: `wg0' already exists as `{interface}'
```

4. 確認 VPN 狀態
```
$ sudo wg show
interface: {interface}
  public key: {public key}
  private key: (hidden)
  listening port: 51421

peer: y4IWe9FjQ1iJg5Ep6YekzmAqUCDXiSRW62X92qzK7HY=
  endpoint: {對方的公網IP}:48763
  allowed ips: 172.27.71.0/24
  latest handshake: 7 seconds ago
  transfer: 2.28 MiB received, 2.44 MiB sent
  persistent keepalive: every 25 seconds
```
互相 ping 連線測試
```
$ ping 172.27.71.7
PING 172.27.71.7 (172.27.71.7): 56 data bytes
64 bytes from 172.27.71.7: icmp_seq=0 ttl=63 time=344.646 ms
64 bytes from 172.27.71.7: icmp_seq=1 ttl=63 time=327.920 ms
64 bytes from 172.27.71.7: icmp_seq=2 ttl=63 time=331.835 ms
64 bytes from 172.27.71.7: icmp_seq=3 ttl=63 time=331.821 ms
```
>[!important]
> 若 ping 不到，檢查防火牆設定\
> `sudo pfctl -sr`
5. 重啟 VPN 更新設定
```
sudo wg-quick down wg0
sudo wg-quick up wg0
```
瀏覽 http://172.27.71.7:5601/\
![image](https://hackmd.io/_uploads/H1rwqPvOJl.png)


# Reference
https://github.com/deviantony/docker-elk

###### tags: `ELK` `Elasticsearch` `Logstash` `Kibana`
