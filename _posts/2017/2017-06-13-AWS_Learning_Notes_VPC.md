---
layout: post
title:  "AWS 學習筆記 - VPC"
description: "This article is a memo recorded when learning AWS VPC"
date: 2017-06-13 05:30:00
published: true
comments: true
categories: [aws]
tags: [AWS, VPC]
---

VPC Introduction & Overview
===========================

Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the Amazon Web Services (AWS) cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.  You can use both IPv4 and IPv6 in your VPC for secure and easy access to resources and applications.
 

You can easily customize the network configuration for your Amazon Virtual Private Cloud. For example, you can create a public-facing subnet for your webservers that has access to the Internet, and place your backend systems such as databases or application servers in a private-facing subnet with no Internet access. You can leverage multiple layers of security, including security groups and network access control lists, to help control access to Amazon EC2 instances in each subnet.

Additionally, you can create a Hardware Virtual Private Network (VPN) connection between your corporate datacenter and your VPC and leverage the AWS cloud as an extension of your corporate datacenter.

## VPC Diagram

![Amazon VPC](http://i.imgur.com/IlqSoD1.png)

- AWS 允許使用者建立最大 **x.y.0.0/16** 的網路，因此一般都會設定為最普遍的 `10.0.0.0/16`
> 若是 VPC 一開始規劃太小，後面想要增大 VPC 的網路範圍是沒辦法的，所以建議一開始就設定到最大

- **Internet Gateway**: 連外使用 (**每一個 VPC 只能有一個 Internet Gateway**)

- **Virtual Private Gateway**: 用來建立 site-to-site VPN (VPN <--> Local Data Center)

- **Route Table**: 決定網路怎麼走

- **Network ACL**: 負責網路存取的安全

- **Public Subnet**: 可被 Internet 存取的網段

- **Private Subnet**: 只有內部的 resource 可以存取的往段

- 每個 subnet 只會存在一個 AZ 內，無法 cross-AZ


## What you can do with a VPC?

- Launch instances into a subnet of your choosing

- Assign custom IP address ranges in each subnet

- Configure route tables between subnets

- Create internet gateway and attach it to your VPC

- Much better security control over your AWS resources

- Instance security groups (**stateful**)

- Subnet network access control lists (ACLs) (**Stateless**)


## Default VPC vs Custom VPC

- Default VPC is user friendly, allowing you to immediately deploy instances

- All subnets in default VPC have a route out to the internet

- Each EC2 instance has both a public and private IP address

- If you delete the default VPC, the only to get it back it to contact AWS


## VPC Peering

- Allow you to connect one VPC with another via a direct network route using private IP addresses.
> 在一個 region 內可以設定多個 VPC，並透過 VPC Peering 的功能互連

- Instances behave as if they were on the same private network

- You can peer VPC's with other AWS accouns as well as with other VPCs in the same account.

- Peering is in a start configuration, ie 1 central VPC peers with 4 others. **NO Transitive Peering!!!**

![VPC Peering](http://docs.aws.amazon.com/AmazonVPC/latest/PeeringGuide/images/transitive-peering-diagram.png)
> 如上圖所示，B & C 無法透過 A 溝通，必須額外建立一個 peer 在 B & C 之間才行


## Exam Tips

- Think of a VPC as a logical datacenter in AWS

- Consist of IGW's(or Virtual Private Gateways), Route Tables, Network Access Control Lists, Subnets, Security Groups

- 1 Subnet = 1 Availability Zone

- Security Groups are Stateful, Network Access Control Lists are Stateless

- No Transitive Peering



Build your own custom VPC
=========================

## 讓 VPC subnet 可以連外的設定流程

1. 建立 Internet Gateway，並與 VPC 關聯 (每個 VPC 只能有一個 IGW)

2. 建立對外的 routing rule
  - 建立專屬對外的 Route Table (例如：myPublicRoute)，並與 VPC 關聯
  - 在 Route Table 中建立新的 routing rules (**Routes**)
> 若要讓 VM instance 可以連外，可以設定 **Destination 0.0.0.0/0**，**Target Internet Gateway**
  - 設定與 Route Table 所關聯的 subnet (預設只會跟 main Route Table 關聯)

3. 在 Subnets 的設定中，針對可以已經關聯連外 Route Table 的 subnet，啟用 **auto-assign Public IP**

![AWS VPC diagram - for public subnet](http://i.imgur.com/lK7zLfs.png)

> 此外，針對新的 VPC，必須要重新建立新的 Security Group 讓服務可以正確的 expose


## 建立一個只有內部可以連的 EC2 Instance

1. 建立一個 internal EC2 instance，使用上個步驟設定好的 VPC，並**指定僅有內部可連的 Subnet**，並且**不要勾選 Auto-assign Public IP**

2. 建立一個新的 security group (以 tcp 3306 為例, 這邊的設定是 **stateful**)
  - 保留 SSH，設定 Source(Custom, 指定可連線的 VPC Subnet)
  - 設定 Type(MYSQL/Aurora), Protocol(TCP), Port Range(3306), Source(Custom, 指定可連線的 VPC Subnet)
  - Allow ICMP traffic, 設定 Type(All ICMP), Protocol(ICMP), Port Range(0-65535), Source(Custom, 指定可連線的 VPC Subnet)

3. 要把 local SSH access key 先放到可連外的 EC2 instance，才可以由該 instance 連到 internal EC2 insance



NAT Instances & NAT Gateways
============================

要讓 internal instance 連外的方式有兩種：

## NAT Instance

1. 選取 Community AMI(**amzn-ami-vpn-nat-hvm-xxxxx**) 作為 Image

2. **選擇可以連外的 VPC Subnet**

3. 防火牆的部分 AWS 已經協助處理好了，但必須確定要透過 NAT 連外的 internal instance 要與 NAT instance 在同一個 security group

4. 當 NAT instance 開機完畢，進行以下設定
  - 到 `Actions -> Networking -> Change Source/Dest. Check` 設定中，按下 `Yes, Disable`
  - 設定 secutiry group，確認此 NAT instance 所在的 security 有 SSH, HTTP, HTTPS 的權限

5. 到 VPC 設定中，設定可讓 internal instance 透過 NAT 連外的 routing rules:
  - 到 Route Tables 的設定中，修改 main rule，並增加 Destination(`0.0.0.0`), **Target 指定為 NAT instance**

完成以上設定後，基本上 internal instance 應該就可以連外了。

![AWS VPC NAT instance](http://i.imgur.com/j0oXvl5.png)

設定 NAT instance 需要注意的地方有幾個：

1. 需要 disable Source/Dest. Check
2. NAT instance 會變成連外的 single point of failure
> 此問題可以透過把 NAT instance 放到 auto-scaling group 並設定 minimum instance = 1 來解決，但整體設定上會比較複雜


## NAT Gateways

1. 建立 NAT Gateways(`VPC -> NAT Gateways`)，**指定 Subnet 為 public subnet**，並建立新的 Elastic IP

2. 到 Route Tables 的設定中，修改 main rule，並增加 Destination(`0.0.0.0`), **Target 指定為 NAT Gateway**

> 無法設定 security gateway，因為 NAT Gateway 並不會受到 Security Group 控制，單純只是將 internal instance 的流量導向 Internet 而已，**若要控制只能透過 ACL**

![AWS VPC NAT Gateway](http://i.imgur.com/UMfjdo1.png)



Netwokr Access Control Lists vs Security Group
==============================================

- Network ACL 可以設定 Allow & Deny 的規則，但 Security Group 只能設定開放的規則

- 新增 VPC 時，預設的 Network ACL 的全部開放的；但 custom network ACL 則預設全部關閉

- 雖然 Network ACL 可以關聯到 multi-AZs 的 subnet；但一個 subnet 只能關聯一個 Network ACL

- Security Group & Network ACL 的 [比較表](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-comparison.html) 要注意看

- 由於 Network ACL 是 stateless，因此在設定規則時要同時考慮 inbound rule & outbound rule，以設定開放連線到 EC2 instance HTTP 服務為例：
  - inbound: Type(`HTTP(80)`), Protocol(`TCP`), Port Range(`80`), Source(`0.0.0.0/0`), Allow
  - outbound: Type(`Custom TCP Rule`), Protocol(`TCP`), Port Range(`1024-65535`), Destination(`0.0.0.0/24`), Allow
> 其中 outbound rule 的設定稱為 [ephemeral ports](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html#VPC_ACLs_Ephemeral_Ports)；其實這個就是 TCP/IP 原理中，client 會從 port 1024-65535 中隨機選一個沒有使用的 port 跟 server 通訊


Custom VPC's and ELBs
======================

若要將 ELB 配置為 HA，就必須要包含來自於 multi-AZs 的多個 subnet 來完成 (至少要兩個 public subnet)


NAT vs Bastion
==============

Bastion 其實就是個跳板的概念，讓外面的使用者可以透過 Internet 存取到位於 NAT gateway 後方的 resource
> 先連到 Bastion，再透過 private connection 連到內部的


VPC Clean Up
============





Exam Tips
=========

## NAT instance

- When creating a NAT instance, Disable Source/Destination Check on the instance

- NAT instance must be in a public subnet
> 1 subnet = 1 AZ, 若要達成 HA，就要有多個 subnet 來實現 multi-AZ

- There must be a route out of the private subnet to the NAT instance, in order for this to work

- 可承載的網路流量與 instance size 有關，流量太大就必須要調大 instance size 來因應

- You can create high availability using Autoscaling Groups, multiple subnets in different AZ's and a script to automate failover

- Behind a Security Group


## NAT Gateway

- very new, may not be in the exams yet.

- prefered by the enterprise

- Scale automatically up to 10Gbps

- No need to patch

- No associated with security groups

- Automatically assigned a public ip address

- Remember to update your route tables

- No need to disable Source/Destination Checks


## Network ACL

- VPC 建立時會自帶一個 default network ACL，並允許所有 inbound & outbound 的流量

- 自訂的 network ACL，預設會拒絕所有 inbound & outbound 的流量

- 所有在 VPC 的 subnet 都會關連到一個 network ACL，若沒有特別指定就會自動關聯 default network ACL

- 當 subnet 與自訂的 network ACL 關聯時，原本的 network ACL 關聯就會移除 (**每個 subnet 只能關聯一個 network ACL**)

- network ACL 的規則判定順序是以設定中的 **Rule #** 欄位中的數字為主，數字越小越早使用

- network ACL 可設定 inbound & outbound rules，且可設定 allow or deny (Security Group 只能設定 allow rule)

- network ACL 是 stateless，因此設定規則時要同時考慮 inbound & outbound rule (也就是要考慮 [ephemeral ports](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html#VPC_ACLs_Ephemeral_Ports) 的設定)

- Block IP addresses using network ACL's not Security Groups


## NAT vs Bastion

- A NAT is used to provide internet traffic to EC2 instances in private subnets

- A Bastion is used to securely administer EC2 instances(using SSH or RDP) in private subnet.



VPC Summary & Exam Tips
=======================

- created a VPC
  - defined our IP address range
  - by default this created a network ACL & route table

- created a custom route table

- created 3 subnets

- created an Internet gateway

- attached to our custom route

- adjusted our public subnet to use the newly defined route

- provisioned an EC2 instance with an Elastic IP address

## NAT instance

- created a security group

- allowed inbound connections to 10.0.1.0/24 and 10.0.2.0/24 on HTTP and HTTPS

- Allowed outbound connections on HTTP and HTTPS for all traffic

- Provisioned our NAT instance inside our public subnet

- disabled source/destination check

- set up a route on our private subnets to route through the NAT instance

## Network ACL

- ACLs can be across multiple subnets

- ACLs encompass all security groups under the subnets associated with them

- rule numbers, lowest is incremented first


Quiz
====

1. How many VPC's am I allowed in each AWS Region by default?
> 5

2. You have a VPC with both public and private subnets. You have 3 EC2 instances that have been deployed in to the public subnet and each has internet access. You deploy a 4th instance using the same AMI and this instance does not have internet access. What could be the cause of this?
> The instance need either an Elastic IP address/Public IP address assigned to it

> The instances needs a private route out to the internet (x)

> The instance's security group is preventing it from connecting to the internet. (x)

> The ACL of the subnet is preventing access to the internet



References
==========

- [AWS | Amazon Virtual Private Cloud (VPC) | FAQs](https://aws.amazon.com/vpc/faqs/)

- [VPCs and Subnets (VPC and Subnet Sizing) - Amazon Virtual Private Cloud](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html#VPC_Sizing)

- [Comparison of NAT Instances and NAT Gateways - Amazon Virtual Private Cloud](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-comparison.html)

- [Security - Amazon Virtual Private Cloud](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Security.html)