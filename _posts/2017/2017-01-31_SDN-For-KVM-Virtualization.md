
SDN Overview
============

現今雲端產業莫不大力推動 SDN，原因是因為 SDN 比起傳統的網路架構，有以下幾個特點：

- 可以更快速的進行 provision

- 支援 multi tenancy

- 提升 network visibility

- 將 control plane 從 forwarding plane 分開，也因此網路在 SDN 的架構中變得可程式化控制 & 管理

而上面提到的 **control plane** & **forwarding plane**，則分別負責以下工作：

- **control plane**：負責決定網路封包 forwarding & routing 的路徑

- **forwarding plane**：實際上封包進行 forwarding 發生的地方

而在傳統的 switch & router 上，control plane & forwarding plane 都是在同一個 network device 內；也因為在 SDN 的概念中，control plane & forwarding plane 的分離，因此在 network 的管理上就取得了可程式化的彈性，藉此也大幅提升了開創新應用的可行性。

## 傳統 Linux bridge 的限制

- 不支援 tunneling protocol

- 不支援 OpenFlow protocol，因此在創新應用上跟擴充性上就會有所限制


--------------------------------------------

Open vSwitch
============

## Ovewview

Open vSwitch 支援以下幾種功能：

- 可透過 NetFlow & sFlow 觀察到 VM 內部的網路流量狀況

- 支援 802.1q

- 可針對 VM interface 層級進行 traffic 管理

- 支援 NIC bonding

- 支援 OpenFlow protocol

- 支援多種 tunneling protocol，例如：GRE、VxLAN、IPSec、GRE over IPSec ... 等等

> 隨著新版本的推出，未來預計會支援更多的功能，詳細的功能列表可參考 [Open vSwitch 官方網站說明](http://openvswitch.org/features/)

跟上面提到的 Linux bridge 相比，以下有個簡單的比較表：

| **Open vSwitch** | **Linux bridge** |
|------------------|------------------|
| Designed for virtual and cloud networking | Originally not designed for virtual networking |
| Full L2-L4 matching capability | Just L2 device |
| Decision in UserSpace | Scaling is limited |
| ACLs, QoS, Bonding | Simple forwarding |
| Mobility of state | Not suitable for cloud environment |
| OpenFlow controller | No tunneling support |
| Distributed vSwitches |  |
| netflow and sflow support |  |


## Architecture

Open vSwitch 大致上可以分為兩大部份，分別是 **Open vSwitch kernel module(Data Plane)** & **user space tools(Control Plane)**；=而為了讓網路封包可以以最快的速度被處理，**data plane** 的部份就被放進了 kernel space。

下圖是 Open vSwitch 的架構：

![Open vSwitch architecture](http://www.yet.org/images/posts/ovs-archi.png)

從以上的架構圖來看各個 component 所負責的工作為：

1. OVS kernel module 使用 netlink socket 與 vswitchd daemon 溝通，用來實作 & 管理在機器上的多台 OVS switch

2. SDN controller 透過 OpenFlow protocol 與 vswitchd daemon 溝通

3. ovsdb-server 負責維護 & 儲存 switch table 資料庫，並支援讓外部 client 透過 JSON RPC 直接溝通

> 目前 ovsdb 中約有 13 個 table 用來儲存運作相關資訊，資料在重開機之後依然會持續存在；此外，為了提昇 OVS 的效能，data plane 被移到 kernel space 的地方進行處理

OVS 可以在兩種模式以下運作，使用者可以根據自己的需求進行選擇：

### (1) Normal Mode

在 Normal Mode 中，OVS bridge 會運作的像是一台 L2 switch，並自行負責處理所有的 swtching & forwarding 的功能；此模式適合用在需要在環境中設定多個 overlay network 且沒有管理 switch flow 的情況下。

### (2) Flow Mode

在 Flow Mode 中，OVS bridge flow table 被用來決定封包是如何被遞送的，而這些 flow table 會統一交給 SDN controller 來進行管理；管理 flow table 的方式有以下幾種：

- 透過 SDN controller

- 透過 SDN controller 提供的 REST API

- 使用 `ctl` 指令

這個模式適合用在大型的環境且網路環境變動頻繁的情況下。


--------------------------------------------


安裝 & 啟動 Open vSwitch
======================

## 安裝 OVS

```bash
$ yum install -y https://rdoproject.org/repos/rdo-release.rpm

# 預計安裝的版本為 2.5.0
$ yum info openvswitch
.....
Name        : openvswitch
Arch        : x86_64
Version     : 2.5.0
Release     : 2.el7
Size        : 9.7 M
Repo        : installed
From repo   : openstack-mitaka
.....

# 查詢相關設定檔
$ rpm -qc openvswitch
/etc/logrotate.d/openvswitch
/etc/openvswitch/conf.db
/etc/openvswitch/system-id.conf
/etc/sysconfig/openvswitch

# 由於 OVS 目前與 Network Manager 並不相容，因此要先停止 NetworkMananger.service，並啟用傳統的 network.service
$ systemctl stop NetworkManager.service
$ systemctl disable NetworkManager.service 
$ chkconfig network on

# 啟動 OVS service
$ systemctl enable openvswitch.service 
$ systemctl start openvswitch.service

# 檢視目前 OVS kernel module 資訊
$ modinfo openvswitch
filename:       /lib/modules/3.10.0-327.28.3.el7.x86_64/kernel/net/openvswitch/openvswitch.ko
license:        GPL
description:    Open vSwitch switching datapath
rhelversion:    7.2
srcversion:     F4BC74559C24C3F264B0F4B
depends:        libcrc32c
intree:         Y
vermagic:       3.10.0-327.28.3.el7.x86_64 SMP mod_unload modversions 
signer:         CentOS Linux kernel signing key
sig_key:        15:64:6F:1E:11:B7:3F:8C:2A:ED:8A:E2:91:65:5D:52:58:05:6E:E9
sig_hashalgo:   sha256

```

關於設定檔的部份，其中 `/etc/openvswitch/conf.db` 其實只是 OVS DB，實際的設定檔是 `/etc/sysconfig/openvswitch`，儲存 OVS 運作時使用的環境變數資訊。


--------------------------------------------

設定 Open vSwitch bridging
==========================

首先說明實體網卡上層的網路設定：

- Native VLAN: `1190` (**10.20.190.0/24**)

- Trunk VLAN：`1187~1189`(**10.20.[187-189].0/24**), `1221-1240`(**10.20.[221-240].0/24**)

接著要設定 Open vSwitch 與實體網卡進行連結，以下透過 network configuration 檔案來設定：

```bash
# OVS 設定
$  cat /etc/sysconfig/network-scripts/ifcfg-ovsbr0
DEVICE="ovsbr0"
BOOTPROTO="none"
DNS1=8.8.8.8
IPADDR=10.20.190.2
PREFIX=24
GATEWAY=10.20.190.1
DEFROUTE="yes"
IPV4_FAILURE_FATAL="yes"
IPV6INIT=no
ONBOOT="yes"
TYPE="OVSBridge"
DEVICETYPE="ovs"

# 網卡設定
$ cat /etc/sysconfig/network-scripts/ifcfg-ens1f0
DEVICE="ens1f0"
ONBOOT="yes"
HWADDR="2c:60:0c:b1:63:d5"
TYPE="OVSPort"
DEVICETYPE="ovs"
OVS_BRIDGE="ovsbr0"

# 重啟網路
$ /etc/init.d/network restart

# 顯示 ovs 目前狀態
$ ovs-vsctl show
6f1e1e8f-0321-4759-b966-1296db6921c3
    Bridge "ovsbr0"
        Port "ens1f0"
            Interface "ens1f0"
        Port "ovsbr0"
            Interface "ovsbr0"
                type: internal
    ovs_version: "2.5.0"

# 顯示目前 IP 狀態 (IP 已經掛在 OVS bridge 上，而非實體網卡了)
$ ip addr show
.....
3: ens1f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP qlen 1000
    link/ether 2c:60:0c:b1:63:d5 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::2e60:cff:feb1:63d5/64 scope link
       valid_lft forever preferred_lft forever
.....
5: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN
    link/ether 06:87:40:70:9a:e4 brd ff:ff:ff:ff:ff:ff
.....
55: ovsbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN
    link/ether 2c:60:0c:b1:63:d5 brd ff:ff:ff:ff:ff:ff
    inet 10.20.190.2/24 brd 10.20.190.255 scope global ovsbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::9802:3cff:feb9:a440/64 scope link
       valid_lft forever preferred_lft forever
```

--------------------------------------------

整合 OVS & KVM VM
=================


```bash
$ virsh dumpxml rhel7.3 | grep -i 'interface type' -A 5
    <interface type='network'>
      <mac address='52:54:00:db:3c:57'/>
      <source network='default'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>

$ virsh domiflist rhel7.3
Interface  Type       Source     Model       MAC
-------------------------------------------------------
-          network    default    virtio      52:54:00:db:3c:57


```


--------------------------------------------


References
==========