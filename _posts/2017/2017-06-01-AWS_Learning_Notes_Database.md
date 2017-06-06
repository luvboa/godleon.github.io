---
layout: post
title:  "AWS 學習筆記 - Database"
description: "This article is a memo recorded when learning AWS Database-related services"
date: 2017-06-01 05:30:00
published: true
comments: true
categories: [aws]
tags: [AWS, Database, RDS, DynamoDB, Redshift, Elasticachem Aurora]
---

Database 101
============

## What is a relational database?

Relational databases are what most of us are all used to. They have been around since the 70's. Think of a traditional spreadsheet:

- Databases

- Tables

- Row

- Fields (Columes)

### 目前 AWS 支援的 Relational Database Types

- SQL Server

- Oracle

- MySQL Server

- PostgreSQL

- Aurora

- MariaDB


## Non Relational Databases

### Database

- Collection = Table

- Document = Row

- Key/Value pairs = Fields

以下是個簡單的範例：

```json
{
    "id": "abc123456", 
    "firstname": "Leon", 
    "lastname": "Tseng", 
    "age": "30", 
    "address": [ { "street": "21 Back Street", "suburb": "Richmond", } ]
}
```


## What is Data Warehousing?

Used for business intelligence. Tools like Cognos, Jaspersoft, SQL Server Reporting Services, Oracle Hyperion, SAP NetWeaver.

Used to pull in very large and complex data sets. Usually used by management to do queries on data (such as current performance vs targets etc)

### OLTP vs OLAP

Online Transaction Processing (OLTP) differs from OLAP Online Analytics Processing (OLAP) in terms of the types of queries run.

### OLAP Example

Order number 123456

> Pulls up a row of data such as Name, Date, Address to Deliver to, Deliver Status etc.

### OLAP transaction Example

Net Profit for EMEA and Pacific for the Digital Radio Product.

- Pulls in large numbers of records

- Sum of Radios Sold in EMEA

- Sum of Radios Sold in Pacific

- Unit Cost of Radio in each region

- Sales price of each radio

- Sales price (unit cost)

Data Warehousing databases use different type of architecture both from a database perspective and infrastructure layer.


## What is Elasticache?

Elasticache is a web service that makes it easy to deploy, operate, and scale an in-memory cache in the cloud. The service improves the performance of web applications by allowing you to retrieve information from fast, managed, in-memory caches, instead of relying enrite on slower disk-based databases.

Elasticache supports two open-source in-memory caching engines:

- Memcached

- Redis


## What is DMS? (Database Migration Service)

Announced ar re:Invent 2015, DMS stands for Database Migration Service. Allow you to migrate your production database to AWS. Once the migration has started, AWS manages all the complexities of the migration process like data type transformation, compression, and parallel transfer (for faster data transfer) while ensuring that data changes to the source database that occur during the migration process are automatically replicated to the target.

AWS schema conversion tool automatically converts the source database schema and a majority of the custom code, including views, stored procedures, and functions, to a format compatible with the target database.


## AWS Database Types - Summary

- RDS: OLTP
  - SQL
  - MySQL
  - PostgreSQL
  - Oracle
  - Aurora
  - MariaDB

- DynamoDB: NoSQL

- RedShift: OLAP

- Elasticache: In-memory caching
  - Memcached
  - Redis

- DMS


Launching an RDS Instance
=========================

- 預設 RDS instance 只能被啟用該 instance 的 public IP 使用而已 (系統會自動偵測使用者所在的 public IP)

- security group 的部份可透過設定 Custom Source 的方式，根據需求讓特定的 AWS resource 可以存取 RDS instance

- 若要讓 EC2 instance 可以存取 RDS instance，則在 security group 的地方要允許 EC2 instance 所在的 security group 存取才可以

```bash
#!/bin/bash

yum install httpd php php-mysql -y
yum udpate -y
chkconfig httpd on
service httpd start
echo "<?php phpinfo(); ?>" > /var/www/html/index.php
```



Backups, Multi-AZ & Read Replicas
==================================

## Automated Backups

There are two different type of backups for AWS. **Automated Backups** and **Database Snapshots**.

Automated Backups allows you to recover your database to any point in time within a `retention period`. Retention period can be between 1 and 35 days. Automated Backups will take a full daily snapshot and will also store transaction logs throughout the day. When you do a recovery, AWS will first choose the most recent daily backup, and then apply transaction logs relevant to that day. This allow you to do a point in time recovery down to a second, within the retention period.

Automated Backups are enabled by default. The backup data is stored in S3 and you get free storage space equal to the size of your database. so if you have an RDS instance of 10Gb you will get 10Gb worth of storage.

Backups are taken within a defiend window. During the backup window, storage I/O maybe suspended while your data is being backed up and you may experience elevated latency.


## Snapshots

DB Snapshots are done manually (ie. they are user initiated). They are stored even after you delete the original RDS instance, unlike automated backups.


## Restoring Backups

Whenever you restore wither an Automatic Backups or a manual Snapshot, the restored version of the database will be a new RDS instance with a new endpoint.


## Encryption

- 目前支援使用 AWS Key Management Service (KMS) 加密 MySQL, Oracle, SQL Server, PostgreSQL 與 MariaDB ...　等類型的資料庫

- 一旦資料庫加密後，包含後續的 Automated Backups, Read Replicas & Snapshots 都會一併被加密

- 目前加密功能僅能用在新建立的 DB instance 上，無法使用在已經存在的 DB instanced 上 (因此若是要加密原本的 DB instance，只能新增後再把資料從舊的 instance 轉移到新的上)


## What is a Multi-AZ?

![AWS Multi-AZ](http://www.trinimbus.com/wp-content/uploads/2014/06/triimage.png)

Multi-AZ allows you to have an exact copy of your production database in another Availability Zone. AWS handles the replication for you, so when your production database is written to, this write will automatically be synchronized to the standby database.

In the event of planned database maintenance, DB instanfe failire, or an Availability Zone failure, Amazon RDS will automatically failover to the standby so that database operations can resume quickly without administrative intervention.

> Multi-AZ is for **Dosaster Recovery** only. It is not primarily used for improving perforamce. For performance improvement you need **Read Replicas**.

Multi-AZ 支援以下 Database Type:

- SQL Server

- Oracle

- MySQL Server

- PostgreSQL

- MariaDB


## What is a Read Replica?

![RDS Read Replica](http://d1nqddva888cns.cloudfront.net/rds_read_replicas.png)

Read Replica's allow you to have a read only copy of your production database. This is achieved by using Asynchronous replication from the primary RDB instance to the read replica. You use read replica's primarily for very read-heavy database workloads.

僅支援以下三種 Database Type:

- MySQL Server

- PostgreSQL

- MariaDB


### Read Replica Databases

- Used for **Scaling**!!! Not for **DR**!

- Must have automatic backups turned on in order to deplo a read replica.

- You can have up to 5 read replica copies of any database.

- You can have read replicas of read replicas (but watch out for latency)

- each read replica will have its own DNS endpoint.

- You cannot have Read Replicas that have Multi-AZ.

- You can create Read Replica's of Multi-AZ source database however.

- Read Replicas can be promoted to be their own databases. This breaks the replicaion.

- Read Replica in a second region for MySQL and MariaDB. Not for PostgreSQL.


## DynamoDB vs RDS

DynamoDB offers **push button** scaling, meaning that you can scale your database on the fly, without any down time.

RDS is not so easy and you usually have to use a bigger instance size or to add a read replica.



DynamoDB
========

Amazon DynamoDB is a fast and flexible NoSQL database service for all applications that need consistent, single-digit millisecond latency at any scale. It is a fully managed database and supports both document and key-value data models. Its flexible data model and reliable performance make it a great git for mobile, web, gaming, ad-tech, IoT, and many other applications.

- stored on SSD storage

- spread across 3 geographically distinct data centers

- eventually Consistent Reads (Default)
> Consistency across all copies of data is usually reaches within a second. Repeating a read after a short time should return the updated data. (Best Read Performance)

- strongly Consistent Reads
> A strongly consistent read returns a result that reflects all writes that received a successfuly response prior to the read.


## DynamoDB Pricing

- Provisioned Throughput Capacity
  - write throughput $0.0065 per hour for every 10 units
  - read throughput $0.0065 per hour for every 50 units

- Stprage costs of $0.25/Gb per month

### Pricing Example

Let's assume that your application nedds to perform 1 million writes and 1 million reads per day, while storing 3GB of data.

First, you need to calculate how many writes and reads per second you need.

1 million evenly spread writes per day is equivalent to 1,000,000(writes) / 24(hours) / 60(minutes) / 60(seconds) = 11.6 writes per second.

Then........

A DynamoDB Write Capacity Unit can handle 1 write per second, so you need 12 Write Capacity Units.

Similarly, to handle 1 million stronly consistent reads per day, you need 12 Read Capacity Units.

With Read Capacity Units, you are billed in blocks of 50, with Write Capacity Units you are billed in blocks of 10.

- Read Capacity Units = (0.0065/50) x 12 x 24 = 0.0374

- Write Capacity Units = (0.0065/10) x 12 x 24 = 0.1872



Redshift
========

## What is Redshift?

Amazon Redshift is a fast and powerful, fully managedm petabyte-scale dat warehouse service in the cloud. Customers can start small for just $0.25 per hour with no commitments or upfront costs and scale to a petabyte or more for $1,000 per terabyte per year, less than 1/10 of most other data warehousing solutions.


## Redshift Configuration

- Single Node (160Gb)

- Multi-Node
  - Leader Node (manages client connections and receives queries).
  - Compute Node (store data and perform queries amd computations). Up to 128 compute nodes.


## Redshift (10 times faster)

### (1) Columnar Data Storage

Instead of storing data as a series of rows, Amazon Redshift organizes the data by comumn. Unlike row-based systems, which are ideal for transaction processing, column-based systems are ideal for data warehousing and analytics, where queries often involve aggregates performed over large data sets. Since only the columns involved in the queries are processed and columnar data is stored sequentially on the storage media, column-based systems require far fewer I/Os, greatly improving query performance.

### (2) Advanced Compression

- 欄位式的資料可以比傳統式 row-based 的資料得到更大的壓縮比，因為相似的資料都是**循序地**存放在磁碟上。

- Amazon Redshift 使用了多種壓縮科技，可以達到相較於傳統關聯式資料庫更小的壓縮結果。

- 相較於傳統關聯式資料庫，Amazon Redshift 不需要 index or materialized view，並耗費較少的空間儲存資料

- 當資料載入到空白的 table 中，Amazon Redshift 就會自動地產生樣本並挑選最合適的資料壓縮方式

### (3) Massiveky Parallel Processing (MPP)

- Amazon Redshift 會自動將資料分散儲存，也會把 query 分散到所有不同的 node 上

- Amazon Redshift 讓增加 node 到 warehous 這件事情變得很容易，因此讓使用者可以很容易根據需求來提升 query 的效能


## Redshift Pricing

- Compute Node Hours (只有 compute node 需要計費，leader node 是免費的)

- Backup

- Data Transfer (only within a VPC, not outside it)


## Redshift Security

- Encrypted in transit using SSL

- Encrypted at rest using AES-256 encryption

- By default Redshift takes care of key management.
  - Manage your own keys through HSM
  - AWS Key Management Service


## Redshift Availability

- Currently only available in 1 AZ
> 因為 Redshift 目的是內部分析管理用，並非 24 小時對使用者提供服務，因此目前並不支援 multi-AZ

- Can restore snapshots to new AZ's in the event of an outage.


ElasticCache
============

ElasticCache is a web service that makes it easy to deploy, operate, and scale an in-memory cache in the cloud. The service improves the performance of web applications by allowing you to retrieve information from fast, managed, in-memory caches, instead of relying entirely on slower disk-based databases.


## What is ElasticCache?

Amazon ElasticCache can be used to significant;y improve latency and throughput for many read-heavy application workloads(such as social networking, gaming, media sharing and Q&A portals) or compute-intensive workloads(such as a recommendation engine).

Caching improves applicaion performance by storing critical pieces of data in memory for low latency access. Cached information may include the result of I/O-intensive database queries or the results of computationally-intensive calculations.


## Type of ElasticCache

- Memcached
> A widely adopted memory object caching system. ElasticCache is protocol compliant with Memcached, so popular tools that you use today with existing Memcached environments will work seamlessly with the service.

- Redis
> A polular open-source in-memory key-value store that supports data structures such as sorted sets and lists. ElasticCache supports Master/Slave replication and Multi-AZ which can be used to achieve cross AZ redundancy.


## ElasticCache Example Tips

Q: 會被問到若是某個 database 目前負載很大，可用哪種 service 來協助改善?

- **ElasticCache**: 用來應付資料庫處於大量的讀取(read)負載下，且資料並非時常變更(write)

- **Redshit**: 適合處理資料庫因為大量的 OLAP 交易工作所導致負載過重的情況



Aurora
======

## What is Aurora?

Amazon Aurora is a MySQL-compatible [relational database](https://aws.amazon.com/relational-database/) engine that combines the speed and availability of high-end commercial databases with the simplicity and cost-effectiveness of open source databases. Amazon Aurora provides up to five times better performance than MySQL with the security, availability, and reliability of a commercial database at one tenth the cost.


## Aurora Scaling

- Start with 10Gb, scale in 10Gb increments to 64Tb (**Storage Autoscaling**)
> 用滿了 10Gb 會自動再長出 10Gb 給你....

- Compute resources can scale up to 32 vCPUs and 244 Gb of memory

- 2 copies of your data is contained in each availability zone, with minimum of 3 availability zones. 6 copies of your data.

- Aurora 被設計成可以容忍 2 份 copy 遺失並保持 write availability & 3 份 copy 遺失並保持 read availability

- Aurora storage is also self-healing. Data blocks and disks are continuously scanned for errors and repaired automatically.


## Aurora Replicas

目前有兩種 replica type: 

- Aurora Replicas (currently 15)

- MySQL Read Replicas (currently 5)


## 其他

- 建立 Aurora instance 要提供一個 **DB Cluster Identifier** 的欄位資料，用來作為 DNS 之用(**Cluster Endpoint**)

- 建立 Aurora instance 時，可在列表中發現有個 `Replication Role` 的欄位: 
  - **writer**: 使用者必須把資料寫到此 instance，才會被同步到其他的 replica instance(reader) 上，可接受 read/write traffic
  - **reader**: 會從 writer instance 同步資料，可接受 read traffic

- 當 Aurora replica 建立完成後，在 overview 的頁面中，就可以看到 Multi-AZ 的欄位的資料從 **No** -> **2 Zones**

- 每個 Aurora replica 都會有自己的 Instance Endpoint，當服務 Cluster Endpoint 的 instance 因為某些原因掛掉時，Cluster Enpoint 會自動 failover 到其他可正常提供服務的 instance 上
> 因此要確保服務可以正確存取，要存取的就必須是 **Cluster Endpoint**



Summary
========

## AWS Database Types

- RDS: OLTP
  - SQL
  - MySQL
  - PostgreSQL
  - Oracle
  - Aurora
  - MariaDB

- DynamoDB: NoSQL

- RedShift: OLAP

- Elasticache: In-memory caching
  - Memcached
  - Redis

- DMS


## What is Read Replicas?

![AWS RDS Read Replicas](http://d1nqddva888cns.cloudfront.net/rds_read_replicas.png)


## Aurora Scaling

- Start with 10Gb, scale in 10Gb increments to 64Tb (**Storage Autoscaling**)
> 用滿了 10Gb 會自動再長出 10Gb 給你....

- Compute resources can scale up to 32 vCPUs and 244 Gb of memory

- 2 copies of your data is contained in each availability zone, with minimum of 3 availability zones. 6 copies of your data.

- Aurora 被設計成可以容忍 2 份 copy 遺失並保持 write availability & 3 份 copy 遺失並保持 read availability

- Aurora storage is also self-healing. Data blocks and disks are continuously scanned for errors and repaired automatically.


## Aurora Replicas

目前有兩種 replica type: 

- Aurora Replicas (currently 15)

- MySQL Read Replicas (currently 5)


## DynamoDB vs RDS

DynamoDB offers **push button** scaling, meaning that you can scale your database on the fly, without any down time.

RDS is not so easy and you usually have to use a bigger instance size or to add a read replica.


## DynamoDB

- stored on SSD storage

- spread across 3 geographically distinct data centers

- eventually Consistent Reads (Default)

- strongly Consistent Reads
> 需要一秒內就可以讀取到修改的資料


## Redshift Configuration

- Single Node (160Gb)

- Multi-Node
  - Leader Node (manages client connections and receives queries).
  - Compute Node (store data and perform queries amd computations). Up to 128 compute nodes.


## What is Elasticache?

ElasticCache is a web service that makes it easy to deploy, operate, and scale an in-memory cache in the cloud. The service improves the performance of web applications by allowing you to retrieve information from fast, managed, in-memory caches, instead of relying entirely on slower disk-based databases.

Elasticache supports two open-source in-memory caching engines: 

- Memcached

- Redis


Quiz
====

- Q: When replicating data from your primary RDS instance to your secondary RDS instance, what is the charge?
>A: No Charge, It's free.

- Q: If you are using Amazon RDS Provisioned IOPS storage with MySQL and Oracle database engines what is the maximum size RDS volume you can have by default?
> A: 6TB

- Q: In RDS when using multiple availability zones, can you use the secondary database as an independent read node?
> A: No

- Q: By default, the maximum provisioned IOPS capacity on an Oracle and MySQL RDS instance (using provisioned IOPS) is 30,000 IOPS.
> A: True


References
==========

- [Amazon RDS FAQs – Amazon Web Services (AWS)](https://aws.amazon.com/rds/faqs/)