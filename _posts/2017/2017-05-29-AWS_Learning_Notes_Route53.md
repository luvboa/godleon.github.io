---
layout: post
title:  "AWS 學習筆記 - Route 53"
description: "This article is a memo recorded when learning AWS Route 53"
date: 2017-05-29 05:30:00
published: true
comments: true
categories: [aws]
tags: [AWS, Route53]
---


DNS 101
=======

## SOA Records (查詢管理領域名稱的伺服器管理資訊)

SOA 主要是與領域有關，所以前面當然要寫 ksu.edu.tw 這個領域名。而 SOA 後面共會接七個參數，這七個參數的意義依序是：

1. **Master DNS 伺服器主機名稱**：這個領域主要是哪部 DNS 作為 master 的意思。在本例中， dns1.ksu.edu.tw 為 ksu.edu.tw 這個領域的主要 DNS 伺服器囉；

2. **管理員的 email**：那麼管理員的 email 為何？發生問題可以聯絡這個管理員。要注意的是， 由於 @ 在資料庫檔案中是有特別意義的，因此這裡就將 abuse@mail.ksu.edu.tw 改寫成 abuse.mail.ksu.edu.tw ，這樣看的懂了嗎？

3. **序號 (Serial)**：這個序號代表的是這個資料庫檔案的新舊，序號越大代表越新。 當 slave 要判斷是否主動下載新的資料庫時，就以序號是否比 slave 上的還要新來判斷，若是則下載，若不是則不下載。 所以當你修訂了資料庫內容時，記得要將這個數值放大才行！ 為了方便使用者記憶，通常序號都會使用日期格式『YYYYMMDDNU』來記憶，例如崑山科大的 2010080369 序號代表 2010/08/03 當天的第 69 次更新的感覺。不過，序號不可大於 2 的 32 次方，亦即必須小於 4294967296 才行喔。

4. **更新頻率 (Refresh)**：那麼啥時 slave 會去向 master 要求資料更新的判斷？ 就是這個數值定義的。崑山科大的 DNS 設定每 1800 秒進行一次 slave 向 master 要求資料更新。那每次 slave 去更新時， 如果發現序號沒有比較大，那就不會下載資料庫檔案。

5. **失敗重新嘗試時間 (Retry)**：如果因為某些因素，導致 slave 無法對 master 達成連線， 那麼在多久的時間內，slave 會嘗試重新連線到 master。在崑山科大的設定中，900 秒會重新嘗試一次。意思是說，每 1800 秒 slave 會主動向 master 連線，但如果該次連線沒有成功，那接下來嘗試連線的時間會變成 900 秒。若後來有成功，則又會恢復到 1800 秒才再一次連線。

6. **失效時間 (Expire)**：如果一直失敗嘗試時間，持續連線到達這個設定值時限， 那麼 slave 將不再繼續嘗試連線，並且嘗試刪除這份下載的 zone file 資訊。崑山科大設定為 604800 秒。意思是說，當連線一直失敗，每 900 秒嘗試到達 604800 秒後，崑山科大的 slave 將不再更新，只能等待系統管理員的處理。

7. **快取時間 (Minumum TTL)**：如果這個資料庫 zone file 中，每筆 RR 記錄都沒有寫到 TTL 快取時間的話，那麼就以這個 SOA 的設定值為主。

(以上資料截自[鳥哥的 Linux 私房菜 -- DNS Server](http://linux.vbird.org/linux_server/0350dns.php#DNS_master_rr))


## NS ：查詢管理領域名稱 (zone) 的伺服器主機名

NS stands for Name Server records and are used by Top Level Domain servers to direct traffic to the Content DNS server which contains the authoritative DNS recrods.


## A, AAAA ：查詢 IP 的記錄

domain name 與實際 ip address 轉換的對應紀錄


## CNAME ：設定某主機名稱的別名 (alias)

Canonical Name, 若不想要針對某個主機名稱設定 A 的標誌，而是想透過另外一部主機名稱的 A 來規範這個新主機名稱時，可以使用別名 (CNAME) 的設定


## TTL

The length that a DNS record is cached on either the Resolving Server or the users own local PC is equal to the value of the "Time To Live" (TTL) in seconds. The lower the time to live, the faster changes to DNS records take to propagate throughout the Internet.


## Alias Records

Alias records are used to map resource record sets in your hosted zone to Elastic Load Balancer, CloudFront distributions, or S3 buckets that are configured as websites.

Alias records work like a CNAME record in that you can map one DNS name(www.example.com) to another "target" DNS name(elb1234.elb.amazonaws.com).

Alias resource records sets can save your time because Amazon Route53 automatically recognizes changes in the record sets that the alias resource record set refer to.

For example, suppose an alias resource record set for example.com points to an ELB load balancer ar lb1-1234.us-east-1.elb.amazonaws.com. If the IP address of the load balancer changes, Amazon Route53 will automatically reflect those changes in DNS answers for example.com without any changes ro the hosted zone that contains resource record sets for example.com.

### Key differece

A CNAME can't be used to naked domain names (zone apex). You can't have a CNAME for http://www.google.com, it must be either an A record or an Alias.


## Exam Tips

- ELB's do not have pre-defined IPv4 addressed, you resolve to them using a DNS name.

- Understand the difference between an Alias Record and a CNAME.

- Given the choice, always choose an Alias Record over a CNAME.



Route53 Routing Policies
========================

目前 AWS 提供了五種 routing policy：

1. Simple
> This is the default routing policy when you create a new record set. This is most commonly used when you have a single resource that performs a given function for your domain, for example, one web server that serves content for the http://www.google.com website.

2. Weighted
> Weighted Routing Policies let you split your traffic based on different weights assigned.
> For example you can set 10% of your traffic to go to US-EAST-1 and 90% to go to EU-WEST-1

3. Latency
> Latency basrd routing allows you to route your traffic based on the lowest network latency for your end user (ie. which region will give them the fastest response time).
> To use latency-based routing you create a latency resource record set for the Amazon EC2(or ELB) resource in each region that hosts your website. When Amazon Route53 receives a query for your site, it selects the latency resource record set for the region that gives the user the lowest latency. Route53 then responds with the value associated with that resource record set.

4. Failover
> Failover routing policies are used when you want to create an active/passsive set up. For example you may want your primary site to be in EU-WEST-2 and your secondary DR site in AP-SOUTHEAST-2.
> Route53 will monitor the health of your primary site using a health check.
> A health check monitors the health of your end points.

5. Geolocation
> Geolocation routing lets you choose where your traffic will be sent based on the geographic location of your users (ie. the location from which DNS queries originate). For example, you might want all queries from Europe to be routed to a fleet of EC2 instances that are specifically configured for your European customers. These servers may have the local language of your European customers and all prices are displayed in Euros.


## Simple Routing Policy

CNAME & Alias 的差異在於，可以幫 zone apex 設定 Alias，但不能設定 CNAME。

那什麼是 zone apex?
> 假設申請了 mydomain.net 這個網域，zone apax = `mydomain.net`


## Weighted Routing Policy

![Route53 Weighted Routing Policy](https://amansardana.files.wordpress.com/2017/02/weighted-routing-policy.png)

用來做 A/B Test 很適合，可以把一定流量的使用者導到新版的網站上

## Latency Based Routing

![Route53 Latency Based Routing](https://s3.amazonaws.com/classconnection/532/flashcards/12156532/jpg/latency-based-routing-policy-15AAA9754B05E19FC24.jpg)


## Failover Routing Policy

![Route53 Failover Routing Policy](http://www.cloudmonix.com/wp-content/uploads/2017/05/route53_failover-diagram.png)

所以其實 active/passive site 不一定都要在 AWS 上....

## Geolocation Routing Policy

![Route53 Geolocation Routing Policy](https://amansardana.files.wordpress.com/2017/02/geolocation-routing1.png)


## DNS Example Tips

- ELB's don't have pre-defined IPv4 addresses, you resolve to them using a DNS name.

- Understand the difference between an Alias Record and a CNAME.

- Given the choice, always choose an Alias Record over a Name

- Remember the different routing policies and the use cases:
    - Simple
    - Weighted
    - Latency
    - Failover
    - Geolocation

- There is a limit to the number of domain names that you can manage using Route 53
> There is 50 domain names available by default, however it is a soft limit and can be raised by contacting AWS support. 
