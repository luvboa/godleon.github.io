---
layout: post
title:  "AWS 學習筆記 - Storage Gateway"
description: "This article is a memo recorded when learning AWS Storage Gateway"
date: 2017-05-16 04:30:00
published: true
comments: true
categories: [aws]
tags: [AWS, Storage Gateway]
---

Storage Gateway
===============

AWS Storage Gateway is a service that connects an on-premises software appliance with cloud-based storage to provide seamless and secure integration between an organization's on-premises IT environment and AWS's storage infrastructure. The service enables you to securely store data to the AWS cloud for scalable and cost-effective storage.

![Storage Gateway (1)](https://image.slidesharecdn.com/3-170425003051/95/deep-dive-on-the-aws-storage-gateway-april-2017-aws-online-tech-talks-4-638.jpg?cb=1493080374)

AWS Storage Gateway's software appliance is available for download as a virtual machine (VM) image that you install on a host in your datacenter. Storage Gateway supports either VMware ESXi or Microsoft Hyper-V. Once you've installed your gateway and associated it with your AWS account through the activation process, you can use the AWS Management Console to create the storage gateway option that is right for you.


## AWS 標準定義

AWS Storage Gateway is a hybrid storage service that enables your on-premises applications to seamlessly use [storage in the AWS Cloud](https://aws.amazon.com/what-is-cloud-storage/). You can use the service for backup and archiving, disaster recovery, cloud bursting, storage tiering, and migration. Your applications connect to the service through a gateway appliance using standard storage protocols, such as NFS and iSCSI. The gateway connects to AWS storage services, such as Amazon S3, Amazon Glacier, and Amazon EBS, providing storage for [files](https://aws.amazon.com/storagegateway/#fileserver), [volumes](https://aws.amazon.com/storagegateway/#volume), and [virtual tapes](https://aws.amazon.com/storagegateway/#VTL) in AWS. The service includes a highly-optimized data transfer mechanism, with bandwidth management, automated network resilience, and efficient data transfer, along with a local cache for low-latency on-premises access to your most active data.


Four Types of Storage Gateways
==============================

- File Gateway (NFS)

- Volumes Gateway (iSCSI)
    - Stored Volumes
    - Cacahed Volumes

- Tape Gateway (VTL)

## File Gateway (比較不常考)

Files are stored as objects in your S3 buckets, accessed through a Network File System (NFS) mount point. Ownership, permissions, and timestamps are durably stored in S3 in the user-metadata of the object associated with the file. Once objects are transferred to S3, they can be managed as native S3 objects, and buckets policies such as versioning, lifecycle management, and cross-region replication apply directly to objects stored in your bucket.

![Storage Gateway (2)](http://docs.aws.amazon.com/storagegateway/latest/userguide/images/file-gateway-concepts-diagram.png)

> file gateway 一般會以 VM 會放在 on-premises 的環境上，但也可以是在 VPC 內的一個 EC2 instance


## Volume Gateway (virtual hard disks)

The volume gateway interface presents your applications with disk volumes using the iSCSI block protocol.

Data written to these volumes can be asynchronously backed up as point-in-time snapshops of your volumes, and stored in the cloud as Amazon EBS snapshots.

Snapshots are incremental backups that captures only changed blocks. All snapshot storage is also compressed to minimized your storage charges.

> 資料以非同步的方式存到 接著進行 snapshot 之後再存放到 S3 上 (incrementally)

### (1) Stored Volumes

Stored volumes let you store your primary data locally, while asychronnously backing up that data to AWS. Stored volumes provide your on-premises applications with low-latency access to their entire datasets, while providing durable, off-site backups. You can create storage volumes and mount them as iSCSI devices from your on-premises application servers. Data written to your stored volumes is stored on your on-premises storage hardware. This data is asynchronously backed up to Amazon Simple Storage Service (Amazon S3) in the form of Amazon Elastic Block Store (Amazon EBS) snapshots. 1GB - 16TB in size for Stored Volumes.

![Volume Gateway - Stored Volumes](http://docs.aws.amazon.com/storagegateway/latest/userguide/images/aws-storage-gateway-stored-diagram.png)

> 本地端保留完整資料(complete copy)

### Cached Volumes

Cached volumes let you use Amazon Simple Storage Service (Amazon S3) as your primary data storage while retaining frequently accessed data locally in your storage gateway. Cacahed volumes minimize the need to scale your on-premises storage infrastructure, while still providing your applications with low-latency access to their frequently accessed data.

You can create storage volumes up to 32 TB in size and attach to them as iSCSI devices from your on-premises application servers. Your gateway stores data that you write to these volumes in Amazon S3 and retains recently read data in your on-premises storage gateway's cache and upload buffer storage. 1GB - 32TB in size for Cached Volumes.

![Volume Gateway - Cached Volumes](http://docs.aws.amazon.com/storagegateway/latest/userguide/images/aws-storage-gateway-cached-diagram.png)

> 所有的資料都會存到 AWS，在本地端只保留常用的部份資料(**Read Data**)，因此本地端實際上並不需要很大的儲存空間

> 從上面的架構圖來說，寫入的資料會被移到 AWS S3 存放，但讀取資料則是從本地端的 Gateway VM 的 Cache Storage 中取得


## Volume Gateway (Tap Gateway)

Tap Gateway offers a durable, cost-effective solution to archive your data in AWS cloud. The VTL interface it provides lets you leverage your existing tape-based backup application infrastructure to store data on virtual tape cartridges that you create on your tape gateway. Each tape gateway is preconfigured with a media changer and tape drivers, which are available to your existing client backup applications as iSCSI devices. You add tape cartridges as you need to archive your data. Supported by NetBackup, Backup Exec, Veam etc.

![Volume Gateway - Tap Gateway](http://docs.aws.amazon.com/storagegateway/latest/userguide/images/Gateway-VTL-iSCSI-vtl-diagram.png)


Exam Tips
=========

- **File Gateway**: For flat files, stored directly on S3

- **Volume Gateway**
    - **Stored Volumes** - Entire Dataset is stored on site and is asynchronously backed up to S3.
    - **Cached Volumes** - Entire Dataset is stored on S3 and the most frequently accessed data is cached on site.

- **Gateway Virtual Tape Library (VTL)**: Used for backup and uses popular backup applications like NetBackup, Backup Exec, Veam etc




References
==========

- [What Is AWS Storage Gateway? - AWS Storage Gateway](http://docs.aws.amazon.com/storagegateway/latest/userguide/WhatIsStorageGateway.html)