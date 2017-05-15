---
layout: post
title:  "AWS 學習筆記 - CloudFront"
description: "This article is a memo recorded when learning AWS CloudFront CDN"
date: 2017-05-16 04:20:00
published: true
comments: true
categories: [aws]
tags: [AWS, CloudFront]
---

What is a CDN?
==============

A content delivery network(CDN) is a system of distributed servers(network) that deliver webpages and other web content to a user based on the geographic locations of the user, the origin of the webpage and a content delivery server.


CloudFront - Key Terminology
============================

### Edge Location

This is the location where content will be cached. This is separate to an AWS Region/AZ.

### Origin

This is the origin of all the files that the CDN will distribute. This can be either an S3 bucket, an EC2 instance, an Elastic Load Balancer or Route53.

### Distribution

This is the name given the CDN which consists of a collection of Edge Locations.

- **Web Distribution**: Typically used for websites.

- **RTMP**: Used for media streaming. (ex: Adobe Flash)


CloudFront
==========

Amazon CloudFront can be used to deliver your entire website, including dynamic, static, streaming, and interactive content using a global network of edge locations. Requests for your content are automatically routed to the nearest edge location, so content is delivered with the best possible performance.

> 連動態網頁、串流、互動內容...等資訊都可以放到 CloudFront (以前以為只有靜態資料可以放 CDN)

CloudFront is optimized to work with other Amazon Web Services, like Amazon Simple Storage Service ([Amazon S3](https://aws.amazon.com/s3/)), Amazon Elastic Compute Cloud ([Amazon EC2](https://aws.amazon.com/ec2/)), [Amazon Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/), and [Amazon Route 53](https://aws.amazon.com/route53/). Amazon CloudFront also works seamlessly with any non-AWS origin server, which stores the original, definitive versions of your files.


Exam Tips
=========

- terminology 的內容要搞清楚

- Edge Locations are **not just READ only**, you can write to them too. (ie. put an object on to them).

- Objects are cached for the life of the TTL (Time to Live)

- You can clear cached objects, but you will be charged.


CloudFont Lab
=============

## Create Distribution Page

- 一切的設定從 **Create Distribution** 開始

### 1. Origin Settings

#### Origin Path

可以指定 bucket 中特定的 folder 或是特定檔案

#### Restrict Bucket Access

若指定 **Yes**，則會設定在 Origin 的 bucket 無法被直接存取，僅能透過 CloudFront 來進行存取

#### Grant Read Permissions on Bucket

CloudFront 從 S3 拉資料必須要有 Read permission透過選擇 **Yes, Update Bucket Policy** 可以讓 AWS 自動協助將權限相關的部份自動設定完成

### 2. Default Cache Behavior Settings

#### Path Pattern

預設為全部 **Default(\*)**，若要調整(例如改為所有 JPG 檔)，可修改 **Origin Path**

#### Allowd HTTP Method

若選擇包含 **PUT,POST,PATCH,DELETE** 的選項，則使用者可以直接將修改的檔案內容上傳到 CloudFront，接著 CloudFront 會自動將變更同步回 Origin 中

#### TTL

- TTL 以秒為單位

- Default TTL 為 86400 (一天)

#### Restrict Viewer Access

透過這功能可以限制只有認証過的使用者(Use Signed URLs or Signed Cookies)才可以存取內容

#### AWS WAF Web ACL

透過 Web Application Firewall 來限制使用者的存取 (new feature)

#### Alternate Domain Names (CNAMEs)

讓 CDN 有個更容易記憶的名稱，可以與 Route53 或是自建的 DNS 進行整合 (必須額外提供 SSL certificate)

#### Default Root Object

若是使用 CloudFront 來架設靜態網站的話，這個選項可以用來設定 default page


## Distribution 設定頁面

- **Origin**: 可在同一個 distribution 中設定多個 origin

- **Behavior**: 可指定特定的內容(例如：只要 JPG)被拉到 CloudFront

- **Error Page**: 可自訂 error page

- **Restrictions**: 可用來設定 Geo Restriction，不要將內容發佈到特定地區的 CloudFront(透過 whitelist or blacklist)

- **Invalidations**：可指定特定的 object 在 TTL 結束之前就從 CloudFront 移除(**這功能要額外付費**)