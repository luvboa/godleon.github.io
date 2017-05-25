---
layout: post
title:  "AWS 學習筆記 - EC2"
description: "This article is a memo recorded when learning AWS EC2"
date: 2017-05-26 05:00:00
published: true
comments: true
categories: [aws]
tags: [AWS, EC2]
---


What is EC2?
============

Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is designed to make web-scale cloud computing easier for developers.

Amazon EC2’s simple web service interface allows you to obtain and configure capacity with minimal friction. It provides you with complete control of your computing resources and lets you run on Amazon’s proven computing environment. Amazon EC2 reduces the time required to obtain and boot new server instances to minutes, allowing you to quickly scale capacity, both up and down, as your computing requirements change. Amazon EC2 changes the economics of computing by allowing you to pay only for capacity that you actually use. Amazon EC2 provides developers the tools to build failure resilient applications and isolate them from common failure scenarios.


## Options

## 1. On Demand 
> allow you to pay a fixed rate by the hour with no commitment.

- Users that want the low cost and flexibility of Amazon EC2 without any up-front payment or long-term commitment.

- Applications with short term, spiky, or unpredictable workloads that cannot be interrupted.

- Applications being developed or tested on amazon EC2 for the first time.

## 2. Reserved
> provide you with a capacity reservation, and offer a significant discount on the hourly charge for an instance. 1 Year or 3 Year Terms.

- Applications with steady state or predictable usage.

- Applications that require reserved capacity

- Users able to make upfront payments to reduce their total computing costs even further. 


## 3. Spot
> enable you to bid whatever price you want for instance capacity, providing for even greater savings if your applications have flexible start and end times.

- Applications that have flexible start and end times.

- Applications that are only feasible at very low compute prices

- Users with urgent computing need s for large amounts of additional capacity



## 4. Dedicated Hosts
> physical EC2 server dedicated for your use. Dedicated Hosts can help you reduce costs by allowing you to use your existing server-bound software licenses.

- Useful for regulatory requirements that may not support multi-tenant virtualization.

- Great for licensing which does not support multi-tenancy or cloud deployments.

- Can be purchased On-Demand (hourly).

- Can be purchased as a Revervation for up to 70% off the On-Demand price.


EC2 Instance Types
==================

| Family | Speciality | Use Case |
|--------|------------|----------|
| D2 | Dense Storage | Fileservers / Data Warehousing / Hadoop |
| R4 | Memory Optimized | Memory Intensive Apps / DBs |
| M4 | General Purpose | Application Servers |
| C4 | Compute Optimized | CPU Intesive Apps/DB2 |
| G2 | Graphics Intensive | Video Encoding / 3D Application Streaming |
| I2 | High Speed Storage | NoSQL DBs, Data Warehousing etc |
| F1 | Field Programmable Gate Array (`FPGA`) | Hardware acceleration for your code |
| T2 | Low Cost, General Purpose | Web Servers / Small DBs |
| P2 | Graphics/General Purpose GPU | Machine Learning, Bit Coin Mining etc |
| X1 | Memory Optimized | SAP HANA / Apache Spark etc |

如何記憶：**DR Mc GIFT PX**

- `D` for **Density**

- `R` for **RAM**

- `M` - main choice for general purpose apps

- `C` for **Compute**

- `G` for **Graphics**

- `I` for **IOPS**

- `F` for **FPGA**

- `T` - cheap general purpose (think t2.micro)

- `P` for **Graphics** (think Pics)

- `X` for **Extreme Memory**


Amazon EBS (Elastic Block Store)?
=================================

## What is EBS?

> Persistent block storage for Amazon EC2

Amazon Elastic Block Store (Amazon EBS) provides persistent block storage volumes for use with [Amazon EC2](https://aws.amazon.com/ec2-sla/) instances in the AWS Cloud. Each Amazon EBS volume is automatically replicated within its Availability Zone to protect you from component failure, offering high availability and durability. Amazon EBS volumes offer the consistent and low-latency performance needed to run your workloads. With Amazon EBS, you can scale your usage up or down within minutes – all while paying a low price for only what you provision.

Amazon Elastic Block Store (Amazon EBS) 可在 AWS 雲端提供用於 Amazon EC2 執行個體的持久性區塊儲存磁碟區。每個 Amazon EBS 磁碟區會在其可用區域內自動複寫，以保護您免於元件故障的威脅，同時提供高可用性和耐久性。Amazon EBS 磁碟區為您提供執行工作負載所需的一致性和低延遲效能。使用 Amazon EBS，您可在幾分鐘內調整用量大小 – 您只需為佈建的資源量支付低廉的價格。

Amazon EBS is designed for application workloads that benefit from fine tuning for performance, cost and capacity. Typical use cases include Big Data analytics engines (like the Hadoop/HDFS ecosystem and [Amazon EMR](https://aws.amazon.com/emr/) clusters), relational and NoSQL databases (like Microsoft SQL Server and MySQL or Cassandra and MongoDB), stream and log processing applications (like Kafka and Splunk), and data warehousing applications (like Vertica and Teradata).

Amazon EBS 是專為應用程式工作負載所設計，而這些工作負載可從效能、成本和容量的微調中獲益。典型的使用案例包含大數據分析引擎 (像是 Hadoop/HDFS 生態系統和 Amazon EMR 叢集)、關聯式和 NoSQL 資料庫 (像是 Microsoft SQL Server 和 MySQL 或 Cassandra 和 MongoDB)、串流和日誌處理應用程式 (像是 Kafka 和 Splunk)，以及資料倉儲應用程式 (像是 Vertica 和 Teradata)。


## EBS Volume Types

### 1. General Purpose SSD (GP2)

- General purpose, balances both price and performance.

- Ratio of 3 IOPS per GB with up to 10,000 IOPS and the ability to burst up to 3000 IOPS for extended periods of time for olumes under 1Gib.

### 2. Provisioned IOPS SSD (IO1)

- Designed for I/O intensive applications such as large relational or NoSQL databases.

- Use if you need more than 10,000 IOPS.

- Can provision up to 20,000 IOPS per volume.

### 3. Throughput Optimized HDD (ST1)

- Big data

- Data warehouses

- Log processing

- Cannot be a boot volume

### 4. Code HDD (SC1)

- Lowest Cost Storage for infrequently accessed workloads

- File Server

- Cannot be a boot volume

### Magnetic (Standard)

Lowest cost per GB of all EBS volume types that is bootable. Magnetic volumes are ideal for workloads whete data is accessed infrequently, and pplications where the lowest storage cost is important.


## Lab Summary

- Termination Protection 預設是關閉的，有需要的話必須手動打開

- 若 EC2 instance 被移除了，預設該 instance 的 root EBS volume 也會被一併移除

- EBS Root Volumes of your DEFAULT AMI's cannot be envrypted. You can also use a third party tool (such as bit locker etc) to encrypt the root volume, or this can be done when creating AMI's in the AWS console or using the API.

- 其他額外增加的 EBS volume 可以被加密

- 每個 subnet 僅會屬於一個 AZ, 不會同時屬於多個 AZ

- Status Check
  - **System Status Checks**: 檢測底層 hypervisor, network 是否有正常
  - **Instance Status Checks**: 檢測 instance OS 層級是否有正常

## Snapshots

- Volumes exist on EBS (virtual hard disk)

- Snapshots exist on S3

- Snapshots are point in time copies of Volumes.

- Snapshots are incrementalm this means that only the blocks that have changed since your last snapshot are moved to S3.

- If this is your first snapshot, it may take some time to create.


Example Tip
===========

## Spot Prirces

如果 Spot instance 是被 Amazon EC2 強制關掉的，那就不會被收取該 instance 的使用費用；但如果是自己關閉的，那就會根據使用時間被收費

## EC2

- Know the differences between:
  - On Demand
  - Spot
  - Reserved
  - Dedicated Hosts

- Remember with spot instances:
  - If you terminate the instance, you pay for the hour
  - If AWS terminate the spot instance, you get the hour it was terminated in for free



Security Group
==============

- All inbound traffic is **Blocked by default**

- All outbound traffic is **Allowed**

- Change to security groups take effect immediately

- You can have any number of EC2 instances within a security group

- 可以將多個 security group 套用到同一個 EC2 instance 中

- Security Group is **STATEFUL**
  - If you create an inbound rule allowing traffic in, that traffic is automatically allowed back out again.

- Network Access Control Lists is **STATELESS**

- You cannot block specific IP addresses using Security Groups, instead use Network Access Control Lists.

- You can specify allow rules, but not deby rules.


RAID, Volumes & Snapshots
=========================

## RAID (Redundant Array of Independent Disks)

- **RAID 0**: Striped, No Redundancy, Good Performance

- **RAID 1**: Mirrored, Redundancy

- **RAID 5**: Good for reads, bad for writes, AWS **does not recommand** ever putting RAID 5's on EBS

- **RAID 10**: Striped & Mirrored, Good Redundancy, Good Performance


## How can I take a snapshot of a RAID Array?

### Problem

Take a snapshot, the snapshot excludes data held in the cache by applications and OS. This tends not to matter on a single volume, however using multiple volumes in a RAID array, this can be a problem due to interdependencies of the array.

### Solutions

Take a application consistent snapshot: 
- Stop the application from writing to disk.
- Flush all caches to the disk.

How can we do this?
- Freeze the file system
- Unmount the RAID array
- Shutting down the associated EC2 instance


Encrypted Root Device Volumes & Snapshots
=========================================

## Snapshots of Root Device Volumes

To create a snapshot for Amazon EBS volumes that serve as root devices, you should stop the instance before taking the snapshot.


## Volumes vs Snapshots - Security

- Snapshots of encrypted volumes are encrypted automatically.

- Volumes restored from encrypted snapshots are encrypted automatically.

- You can share snapshots, but only if they are unencrypted. (加密用的金鑰僅能用在單一帳號上)
  - These snapshots can be shared with other AWS accounts or made public.


AMI Types (EBS vs Instance Store)
=================================

You can select your AMI based on:

- Region (see Regions & Availability Zones)

- Operating System

- Architecture (32-bits or 64-bits)

- Launch Permissions

- Storage for the Root Device (Root Device Volume)
  - Instance Store (Ephemeral Storage)
  - EBS Backed Volumes


## EBS vs Instance Store

- All AMIs are categorized as either backed by Amazon EBS or backed by instance store.

- For **EBS volumes**: The root device for an instance launched from the AMI is an Amazon EBS volume created from an Amazon EBS snapshot.

- For **Instance Store Volumes**: The root device for an instance launched from the AMI is an instance store volume created from a template stored in Amazon S3.


## Exam Tips

- Instance Store Volumes are sometimes called **Ephemeral Storage**.

- Instance Store Volumes cannot be stopped. If the underlying host fails, you will lose your data.

- EBS backed instances can be stopped. You will not lose the data on this instance if it is stopped.

- You can reboot both, you will not lose your data.

- By default, both ROOT volumes will be deleted on termination, however with EBS volumes, you can tell AWS to keep the root device volume.



Elastic Load Balancers
======================

- Instance monitored by ELB are reported as `InService` or `OutOfService`

- Health Checks check the instance health by talking to it.
> 透過指定 port & 用來檢查健康狀態的頁面 (不一定是 index.html，可以設計一個很簡單的頁面供檢查)

- Have their own DNS name. You are never given an IP address.

- Read the ELB FAQ for Classsic Load Balancers
> 2016 AWS 推出了 Application Load Balancer，運作於 layer 7 上，因為很新還不見得會考

- Application Load Balancer 必須透過設定 **Target Group**，指定 **protocol** & **port** 來監控不同的 layer 7 service



CloudWatch
==========

## Exam Tips

- Standard Monitoring = 5 minutes

- Details Monitoring = 1 minute


## What can I do with Cloudwatch?

- **Dashboard**: creates awesome dashboards to see what is happening with your AWS environment.

- **Alarm**: allows you to set Alarms that notify you when particular thresholds are hit.

- **Events**: CloudWatch Events helps you to responsd to state changes in your AWS resources.

- **Logs**: CloudWatch Logs helps you to aggergate, monitoring, and store logs.


Identity Access Management Roles
================================

## Role

- Roles are more secure than storing your access key and secret access key on individual EC2 instances.

- Roles are easier to manage.

- Roles can be assigned to an EC2 instance after it is created, but currenctly only using the commad line.

- Roles are universal, you can use them in any region.


Metadata
========

`http://169.254.169.254/latest/meta-data/`


Auto Scaling
============


Placement Group
===============

## What is a Placement Group?

A placement group is a logical grouping of instances within a single Availability Zone. Using placement groups enables applications to participate in a low-latency, 10 Gbps network. Placement groups are recommended for applications that benefit from low network latency, high network throughput, or both.

- A placement group can't span multiple Avalability Zones.

- The name you specify for a placement group must be unique within your AWS account.

- Only certain types of instances can be launched in a placement group (Compute Optimized, GPU, Memoery Optimized, Storage Optimized)

- AWS recommend homogenous instances within placement groups.

- You can't merge placement groups.

- You can't move an existing instance into a placement group. You can create an AMI from your existing instance, and then launch a new instance from the AMI ino a placement group.


## References

- [Placement Groups - Amazon Elastic Compute Cloud](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)


Elastic File System (EFS)
=========================

Amazon Elastic File System (Amazon EFS) provides simple, scalable file storage for use with Amazon EC2 instances in the AWS Cloud. Amazon EFS is easy to use and offers a simple interface that allows you to create and configure file systems quickly and easily. With Amazon EFS, storage capacity is elastic, growing and shrinking automatically as you add and remove files, so your applications have the storage they need, when they need it.

## Features

- Supports the Network File System version 4(NFSv4) protocol.

- You only pay for the storage you use (no pre-provisioning required)

- Can scale up to the petabytes

- Can support thousands of concurrent NFS connections

- Data is stored across multiple AZ's within a region

- Read After Write Consistency

- 屬於 **block-based** storage


Lambda
======

## What is Lambda?

AWS Lambda is a compute service where you can upload your code and create a Lambda function. AWS Lambda takes care of provisioning and managing the servers that you use to run the code. You don't have to worry about operating systems, patching, scaling, etc. You can use Lambda in the following ways:

- As an event-driven compute service where AWS Lambda runs your code in response to events. These events could be changes to data in an Amazon S3 bucket or an Amazon DynamoDB table.

- As a compute service to run your code in response to HTTP requests using Amazon API Gateway or API calls made using AWS SDKs.

- 支援的程式語言：Node.js, Java, Python. C#


基本上，Lambda 把以下東西給包裝起來了，使用者可以不用管：

- Data Center

- Hardware

- Assembly Code/Protocols

- High Level Language

- Operating Systems

- Application Layer/AWS APIs
 

## How is Lambda Priced?

### Number of requests

- First 1 million requests are free. $0.20 per 1 million requests thereafter.

### Duration

- Duration is calculated from the time your code begins executing until it returns or otherwise terminated, rounded up to the nearest 100ms. The price depends on the amount of memory you allocate to your function. You are charged $0.00001667 for every GB-second used.


## Why is Lambda Cool?

- NO SERVERS!

- Continuous Scaling

- Super super super cheap!



Summary & Exam Tips
===================

## 考前必讀

- [Amazon EC2 FAQs - Amazon Web Services](https://aws.amazon.com/ec2/faqs/)

## Exam Tips EC2

- Know the differences between:
  - On Demand
  - Spot
  - Reserved
  - Dedicated Hosts

- Remember with spot instances;
  - If you terminate the instance, you pay for the hour
  - If AWS terminates the spot instance, you get the hour it was terminated in for free.


## Exam Tips EBS

- EBS consists of; 
  - SSD, General Purpose - GP2 (up to 10,000 IOPS)
  - SSD, Provisioned IOPS - IO1 (more than 10,000 IOPS)
  - HDD, Throughput Optimized - ST1 (frequently accessed workloads) (例如：data warehousing, transaction logging)
  - HDD, Cold - SC1 (less frequently accessed data)
  - HDD, Magnetic - Standard (cheap, infrequently accessed storage) (其實跟 HDD Cold 相同，只是這個可以當開機碟)

- You cannot mount 1 EBS volume to multiple EC2 instances, instead use EFS.


## EC2 Lab Exam Tips

- **Termination Protection** is turned off by default, you must turn it on.

- On an EBS-backed instance, the default action is for the root EBS volume to be deleted when the instance is terminated.

- Root Volumes cannot encrypted by default, you need a third party tool (such as bit locker etc) to encrypt the roo volume.


## Volumes vs Snapshots

- Volumes exist on EBS (virtual hard disk)

- Snapshots exist on S3

- You can take a snapshot of a volume, this will store that volume

- Snapshots are point in time copies of volumes.

- Snapshots are incremental, this means that only the blocks that have changed since your last snapshot are moved to S3.
> 第一次的 snapshot 需要花比較多的時間完成

### Security

- Snapshots of encrypted volumes are encrypted automatically.

- Volumes restored from encrypted snapshots are encrypted automatically

- You can share snapshots, but only if they are unencrypted
> These snapshots can be shared with other AWS accounts or made public


## Snapshots of Root Device Volumes

To create a snapshot for Amazon EBS volumes that serve as root devices, you should stop the instance before taking the snapshot.


## EBS vs Instance Store

- Instance store volumes are sometimes called **Ephemeral Storage**.

- Instance store volumes cannot be stopped. If the underlying host fails, you will lose your data.

- EBS backed instances can be stopped. You will not lose the data on this instance if it is stopped.

- You can reboot both, you will not lose your data.

- By default, both ROOT volumes will be deleted on termination, however with EBS volumes, you can tell AWS to keep the root device volume.


## How can I take a Snapshot of a RAID Array?

- **Problem**: Take a snapshot, the snapshot excludes data held in the cache by applications and the OS. This tends not to matter on a single volume, however using multiple volumes in a RAID array, this can be a problem due to interdependencies of the array.

- **Solution**: Take an application consistent snapshot.

- Stop the application from writing to disk.

- Flush all caches to the disk.

- How can we do this?
  - Freeze the file syste,
  - Unmount the RAID array
  - Shutting down the associated EC2 instance.


## Amazon Machine Images

AMI's are regional. You can only launch an AMI from the region in which it is stored. However you can copy AMI's to other regions using the console, command line or the Amazon EC2 API.


## Clodwatch

- Standard Monitoring = 5 mins
- Detailed Monitoring = 1 min

- CloudWatch is for performance monitoring
- CloudTrail is for auditing

### What can I do with CloudWatch?

- **Dashboards**: create awesome dashboards to see what is happening with your AWS environment.

- **Alarms**: Allows you to set alarms that notify you when particular thresholds are hit.

- **Event**: CloudWatch events helps you to response to state changes in your AWS resources.

- **Logs**: CloudWatch logs helps you to aggregate, monitoring, and store logs.


## Roles

- Roles are more secure than storing your access key and secret access key on individual EC2 instances.

- Roles are easier to manage

- Roles can only be assigned when that EC2 instance is being provisioned.

- Roles are universal, you can use them in any region.


## Instance Metadata

- Used to get information about an instance (such as public ip)

- curl http://169.254.169.254/latest/meta-data

- No such thing as user-data for an instance


## EFS

- Support the Network File System version 4 (NFSv4) protocol

- You only pay for the storage you use (no pre-provisioning required)

- Can scale up to the petabytes

- Can support thousands of concurrent NFS connections

- Data is stored across multiple AZ's within a region

- Read After Write Consistency


## Quiz

2. Do Amazon EBS volumes persist independently from the life of an Amazon EC2 instance, for example, if I terminated an EC2 instance, would that EBS volume remain?
> By default EBS volumes are set to 'Delete on Termination', meaning they persist only if instructed.

3. Can I delete a snapshot of an EBS Volume that is used as the root device of a registered AMI?
> No, you must deregister the AMI before being able to delete the root device.
> you must deregister the AMI before being able to delete the root device. http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-deleting-snapshot.html