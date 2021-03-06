---
title: 在 Azure 中於 SUSE Linux Enterprise Server 上設定 Pacemaker | Microsoft Docs
description: 在 Azure 中於 SUSE Linux Enterprise Server 上設定 Pacemaker
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: mssedusch
manager: gwallace
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/16/2018
ms.author: sedusch
ms.openlocfilehash: 74c8c7dfc2beda2d242bc21e12293dc6f3c1cffe
ms.sourcegitcommit: 64def2a06d4004343ec3396e7c600af6af5b12bb
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/19/2020
ms.locfileid: "77470831"
---
# <a name="setting-up-pacemaker-on-suse-linux-enterprise-server-in-azure"></a>在 Azure 中於 SUSE Linux Enterprise Server 上設定 Pacemaker

[planning-guide]:planning-guide.md
[deployment-guide]:deployment-guide.md
[dbms-guide]:dbms-guide.md
[sap-hana-ha]:sap-hana-high-availability.md
[virtual-machines-linux-maintenance]:../../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot
[virtual-machines-windows-maintenance]:../../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot
[sles-nfs-guide]:high-availability-guide-suse-nfs.md
[sles-guide]:high-availability-guide-suse.md

有兩個選項可以在 Azure 中設定 Pacemaker 叢集。 您可以使用隔離代理程式，這個代理程式會透過 Azure API 來重新啟動失敗的節點，或者您可以使用 SBD 裝置。

SBD 裝置至少需要一部額外的虛擬機器，作為 iSCSI 目標伺服器並且提供 SBD 裝置。 不過，這些 iSCSI 目標伺服器可以與其他 Pacemaker 叢集共用。 使用 SBD 裝置的優點是更快速的容錯移轉時間，如果您在內部部署使用 SBD 裝置，則不需要變更您操作 pacemaker 叢集的方式。 您可以對一個 Pacemaker 叢集使用最多三個 SBD 裝置，以容許在 iSCSI 目標伺服器的 OS 修補期間 (舉例來說)，有一個 SBD 裝置無法使用。 如果您想要使用一個以上的 SBD 裝置每 Pacemaker，請務必部署多部 iSCSI 目標伺服器，並從每部 iSCSI 目標伺服器連線一個 SBD。 我們建議使用一個或三個 SBD 裝置。 如果您只設定兩個 SBD 裝置，而且其中一個無法使用，Pacemaker 將無法自動隔離叢集節點。 如果您希望能夠在一部 iSCSI 目標伺服器關閉時隔離，您必須使用三個 SBD 裝置，因此會有三部 iSCSI 目標伺服器。

如果您不想要投資一部額外的虛擬機器，您也可以使用 Azure 隔離代理程式。 缺點是如果資源停止容錯或叢集節點無法再彼此進行通訊，則容錯移轉需要 10 到 15 分鐘。

![SLES 上的 Pacemaker 概觀](./media/high-availability-guide-suse-pacemaker/pacemaker.png)

>[!IMPORTANT]
> 規劃和部署 Linux Pacemaker 叢集節點與 SBD 裝置時，對於完整叢集設定的整體可靠性而言，最重要的是相關 VM 與裝載 SBD 裝置的 VM 之間，其路由傳送不會通過 [NVA](https://azure.microsoft.com/solutions/network-appliances/) 等任何其他裝置。 否則，與 NVA 相關的問題與維護事件，會對整體叢集設定的穩定性與可靠性造成負面影響。 為了避免這類障礙，請不要定義 Nva 或[使用者定義的路由規則](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)的路由規則，這些規則會在規劃和部署 Linux Pacemaker 叢集節點和 SBD 裝置時，透過 nva 和類似的裝置來路由傳送叢集節點與 SBD 裝置之間的流量。 
>

## <a name="sbd-fencing"></a>SBD 隔離

如果您想要使用 SBD 裝置進行隔離，請遵循以下步驟。

### <a name="set-up-iscsi-target-servers"></a>設定 iSCSI 目標伺服器

您必須先建立 iSCSI 目標虛擬機器。 iSCSI 目標伺服器可以與多個 Pacemaker 叢集共用。

1. 部署新的 SLES 12 SP1 或更新版本的虛擬機器，並透過 SSH 連線到這些機器。 電腦不需要太大。 諸如 Standard_E2s_v3 或 Standard_D2s_v3 的虛擬機器大小就已足夠。 請務必使用 OS 磁碟的進階儲存體。

在所有 **iSCSI 目標虛擬機器**上執行下列命令。

1. 更新 SLES

   <pre><code>sudo zypper update
   </code></pre>

   > [!NOTE]
   > 升級或更新 OS 之後，您可能需要重新開機作業系統。 

1. 移除套件

   若要避免 targetcli 和 SLES 12 SP3 的已知的問題，請解除安裝下列套件。 您可以忽略與找不到套件有關的錯誤

   <pre><code>sudo zypper remove lio-utils python-rtslib python-configshell targetcli
   </code></pre>

1. 安裝 iSCSI 目標套件

   <pre><code>sudo zypper install targetcli-fb dbus-1-python
   </code></pre>

1. 啟用 iSCSI 目標服務

   <pre><code>sudo systemctl enable targetcli
   sudo systemctl start targetcli
   </code></pre>

### <a name="create-iscsi-device-on-iscsi-target-server"></a>在 iSCSI 目標伺服器上建立 iSCSI 裝置

在所有 **iSCSI 目標虛擬機器**上執行下列命令，為 SAP 系統所用的叢集建立 iSCSI 磁碟。 在下列範例中，會建立多個叢集的 SBD 裝置。 它會顯示如何對多個叢集使用一部 iSCSI 目標伺服器。 SBD 裝置會置於 OS 磁碟上。 確定您有足夠的空間。

**`nfs`** 是用來識別 NFS 叢集， **ascsnw1**是用來識別**NW1**的 ASCS 叢集， **dbnw1**是用來識別**NW1**的資料庫叢集、 **nfs-0**和**NFS-1**是 nfs 叢集節點的主機名稱、 **NW1-xscs-0**和**NW1-xscs-1**是**NW1** ASCS 叢集節點的主機名稱，而**NW1-db-0**和**NW1-db-1**是資料庫叢集節點的主機名稱。 使用您的叢集節點主機名稱和 SAP 系統 SID 取代它們。

<pre><code># Create the root folder for all SBD devices
sudo mkdir /sbd

# Create the SBD device for the NFS server
sudo targetcli backstores/fileio create sbdnfs /sbd/sbdnfs 50M write_back=false
sudo targetcli iscsi/ create iqn.2006-04.nfs.local:nfs
sudo targetcli iscsi/iqn.2006-04.nfs.local:nfs/tpg1/luns/ create /backstores/fileio/sbdnfs
sudo targetcli iscsi/iqn.2006-04.nfs.local:nfs/tpg1/acls/ create iqn.2006-04.<b>nfs-0.local:nfs-0</b>
sudo targetcli iscsi/iqn.2006-04.nfs.local:nfs/tpg1/acls/ create iqn.2006-04.<b>nfs-1.local:nfs-1</b>

# Create the SBD device for the ASCS server of SAP System NW1
sudo targetcli backstores/fileio create sbdascs<b>nw1</b> /sbd/sbdascs<b>nw1</b> 50M write_back=false
sudo targetcli iscsi/ create iqn.2006-04.ascs<b>nw1</b>.local:ascs<b>nw1</b>
sudo targetcli iscsi/iqn.2006-04.ascs<b>nw1</b>.local:ascs<b>nw1</b>/tpg1/luns/ create /backstores/fileio/sbdascs<b>nw1</b>
sudo targetcli iscsi/iqn.2006-04.ascs<b>nw1</b>.local:ascs<b>nw1</b>/tpg1/acls/ create iqn.2006-04.<b>nw1-xscs-0.local:nw1-xscs-0</b>
sudo targetcli iscsi/iqn.2006-04.ascs<b>nw1</b>.local:ascs<b>nw1</b>/tpg1/acls/ create iqn.2006-04.<b>nw1-xscs-1.local:nw1-xscs-1</b>

# Create the SBD device for the database cluster of SAP System NW1
sudo targetcli backstores/fileio create sbddb<b>nw1</b> /sbd/sbddb<b>nw1</b> 50M write_back=false
sudo targetcli iscsi/ create iqn.2006-04.db<b>nw1</b>.local:db<b>nw1</b>
sudo targetcli iscsi/iqn.2006-04.db<b>nw1</b>.local:db<b>nw1</b>/tpg1/luns/ create /backstores/fileio/sbddb<b>nw1</b>
sudo targetcli iscsi/iqn.2006-04.db<b>nw1</b>.local:db<b>nw1</b>/tpg1/acls/ create iqn.2006-04.<b>nw1-db-0.local:nw1-db-0</b>
sudo targetcli iscsi/iqn.2006-04.db<b>nw1</b>.local:db<b>nw1</b>/tpg1/acls/ create iqn.2006-04.<b>nw1-db-1.local:nw1-db-1</b>

# save the targetcli changes
sudo targetcli saveconfig
</code></pre>

您可以檢查一切是否都已正確設定：

<pre><code>sudo targetcli ls

o- / .......................................................................................................... [...]
  o- backstores ............................................................................................... [...]
  | o- block ................................................................................... [Storage Objects: 0]
  | o- fileio .................................................................................. [Storage Objects: 3]
  | | o- <b>sbdascsnw1</b> ................................................ [/sbd/sbdascsnw1 (50.0MiB) write-thru activated]
  | | | o- alua .................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ........................................................ [ALUA state: Active/optimized]
  | | o- <b>sbddbnw1</b> .................................................... [/sbd/sbddbnw1 (50.0MiB) write-thru activated]
  | | | o- alua .................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ........................................................ [ALUA state: Active/optimized]
  | | o- <b>sbdnfs</b> ........................................................ [/sbd/sbdnfs (50.0MiB) write-thru activated]
  | |   o- alua .................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ........................................................ [ALUA state: Active/optimized]
  | o- pscsi ................................................................................... [Storage Objects: 0]
  | o- ramdisk ................................................................................. [Storage Objects: 0]
  o- iscsi ............................................................................................. [Targets: 3]
  | o- <b>iqn.2006-04.ascsnw1.local:ascsnw1</b> .................................................................. [TPGs: 1]
  | | o- tpg1 ................................................................................ [no-gen-acls, no-auth]
  | |   o- acls ........................................................................................... [ACLs: 2]
  | |   | o- <b>iqn.2006-04.nw1-xscs-0.local:nw1-xscs-0</b> ............................................... [Mapped LUNs: 1]
  | |   | | o- mapped_lun0 ............................................................ [lun0 fileio/<b>sbdascsnw1</b> (rw)]
  | |   | o- <b>iqn.2006-04.nw1-xscs-1.local:nw1-xscs-1</b> ............................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 ............................................................ [lun0 fileio/<b>sbdascsnw1</b> (rw)]
  | |   o- luns ........................................................................................... [LUNs: 1]
  | |   | o- lun0 .......................................... [fileio/sbdascsnw1 (/sbd/sbdascsnw1) (default_tg_pt_gp)]
  | |   o- portals ..................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ...................................................................................... [OK]
  | o- <b>iqn.2006-04.dbnw1.local:dbnw1</b> ...................................................................... [TPGs: 1]
  | | o- tpg1 ................................................................................ [no-gen-acls, no-auth]
  | |   o- acls ........................................................................................... [ACLs: 2]
  | |   | o- <b>iqn.2006-04.nw1-db-0.local:nw1-db-0</b> ................................................... [Mapped LUNs: 1]
  | |   | | o- mapped_lun0 .............................................................. [lun0 fileio/<b>sbddbnw1</b> (rw)]
  | |   | o- <b>iqn.2006-04.nw1-db-1.local:nw1-db-1</b> ................................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .............................................................. [lun0 fileio/<b>sbddbnw1</b> (rw)]
  | |   o- luns ........................................................................................... [LUNs: 1]
  | |   | o- lun0 .............................................. [fileio/sbddbnw1 (/sbd/sbddbnw1) (default_tg_pt_gp)]
  | |   o- portals ..................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ...................................................................................... [OK]
  | o- <b>iqn.2006-04.nfs.local:nfs</b> .......................................................................... [TPGs: 1]
  |   o- tpg1 ................................................................................ [no-gen-acls, no-auth]
  |     o- acls ........................................................................................... [ACLs: 2]
  |     | o- <b>iqn.2006-04.nfs-0.local:nfs-0</b> ......................................................... [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ................................................................ [lun0 fileio/<b>sbdnfs</b> (rw)]
  |     | o- <b>iqn.2006-04.nfs-1.local:nfs-1</b> ......................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 ................................................................ [lun0 fileio/<b>sbdnfs</b> (rw)]
  |     o- luns ........................................................................................... [LUNs: 1]
  |     | o- lun0 .................................................. [fileio/sbdnfs (/sbd/sbdnfs) (default_tg_pt_gp)]
  |     o- portals ..................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ...................................................................................... [OK]
  o- loopback .......................................................................................... [Targets: 0]
  o- vhost ............................................................................................. [Targets: 0]
  o- xen-pvscsi ........................................................................................ [Targets: 0]
</code></pre>

### <a name="set-up-sbd-device"></a>設定 SBD 裝置

從叢集連線到上一個步驟中建立的 iSCSI 裝置。
在您想要建立之新叢集的節點上，執行下列命令。
下列項目會加上下列其中一個前置詞： **[A]** - 適用於所有節點、 **[1]** - 僅適用於節點 1 或 **[2]** - 僅適用於節點 2。

1. **[A]** 連線到 iSCSI 裝置

   首先，啟用 iSCSI 和 SBD 服務。

   <pre><code>sudo systemctl enable iscsid
   sudo systemctl enable iscsi
   sudo systemctl enable sbd
   </code></pre>

1. **[1]** 變更第一個節點上的啟動器名稱

   <pre><code>sudo vi /etc/iscsi/initiatorname.iscsi
   </code></pre>

   變更檔案內容，以符合您在 iSCSI 目標伺服器上建立 iSCSI 裝置時使用的 ACL，例如 NFS 伺服器。

   <pre><code>InitiatorName=<b>iqn.2006-04.nfs-0.local:nfs-0</b>
   </code></pre>

1. **[2]** 變更第二個節點上的啟動器名稱

   <pre><code>sudo vi /etc/iscsi/initiatorname.iscsi
   </code></pre>

   變更檔案內容，以符合您在 iSCSI 目標伺服器上建立 iSCSI 裝置時使用的 ACL

   <pre><code>InitiatorName=<b>iqn.2006-04.nfs-1.local:nfs-1</b>
   </code></pre>

1. **[A]** 重新啟動 iSCSI 服務

   立即重新啟動 iSCSI 服務以套用變更

   <pre><code>sudo systemctl restart iscsid
   sudo systemctl restart iscsi
   </code></pre>

   連線到 iSCSI 裝置。 在下列範例中，iSCSI 目標伺服器的 IP 位址是 10.0.0.17，預設連接埠是 3260。 <b>iqn.2006-04.nfs.local:nfs</b> 是當您執行以下第一個命令 (iscsiadm -m discovery) 時列出的其中一個目標名稱。

   <pre><code>sudo iscsiadm -m discovery --type=st --portal=<b>10.0.0.17:3260</b>   
   sudo iscsiadm -m node -T <b>iqn.2006-04.nfs.local:nfs</b> --login --portal=<b>10.0.0.17:3260</b>
   sudo iscsiadm -m node -p <b>10.0.0.17:3260</b> --op=update --name=node.startup --value=automatic
   
   # If you want to use multiple SBD devices, also connect to the second iSCSI target server
   sudo iscsiadm -m discovery --type=st --portal=<b>10.0.0.18:3260</b>   
   sudo iscsiadm -m node -T <b>iqn.2006-04.nfs.local:nfs</b> --login --portal=<b>10.0.0.18:3260</b>
   sudo iscsiadm -m node -p <b>10.0.0.18:3260</b> --op=update --name=node.startup --value=automatic
   
   # If you want to use multiple SBD devices, also connect to the third iSCSI target server
   sudo iscsiadm -m discovery --type=st --portal=<b>10.0.0.19:3260</b>   
   sudo iscsiadm -m node -T <b>iqn.2006-04.nfs.local:nfs</b> --login --portal=<b>10.0.0.19:3260</b>
   sudo iscsiadm -m node -p <b>10.0.0.19:3260</b> --op=update --name=node.startup --value=automatic
   </code></pre>

   請確定 iSCSI 裝置可以使用，並且記下裝置名稱 (在下列範例中為 /dev/sde)

   <pre><code>lsscsi
   
   # [2:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sda
   # [3:0:1:0]    disk    Msft     Virtual Disk     1.0   /dev/sdb
   # [5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc
   # [5:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdd
   # <b>[6:0:0:0]    disk    LIO-ORG  sbdnfs           4.0   /dev/sdd</b>
   # <b>[7:0:0:0]    disk    LIO-ORG  sbdnfs           4.0   /dev/sde</b>
   # <b>[8:0:0:0]    disk    LIO-ORG  sbdnfs           4.0   /dev/sdf</b>
   </code></pre>

   現在，擷取 iSCSI 裝置的識別碼。

   <pre><code>ls -l /dev/disk/by-id/scsi-* | grep <b>sdd</b>
   
   # lrwxrwxrwx 1 root root  9 Aug  9 13:20 /dev/disk/by-id/scsi-1LIO-ORG_sbdnfs:afb0ba8d-3a3c-413b-8cc2-cca03e63ef42 -> ../../sdd
   # <b>lrwxrwxrwx 1 root root  9 Aug  9 13:20 /dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03 -> ../../sdd</b>
   # lrwxrwxrwx 1 root root  9 Aug  9 13:20 /dev/disk/by-id/scsi-SLIO-ORG_sbdnfs_afb0ba8d-3a3c-413b-8cc2-cca03e63ef42 -> ../../sdd
   
   ls -l /dev/disk/by-id/scsi-* | grep <b>sde</b>
   
   # lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-1LIO-ORG_cl1:3fe4da37-1a5a-4bb6-9a41-9a4df57770e4 -> ../../sde
   # <b>lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df -> ../../sde</b>
   # lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-SLIO-ORG_cl1_3fe4da37-1a5a-4bb6-9a41-9a4df57770e4 -> ../../sde
   
   ls -l /dev/disk/by-id/scsi-* | grep <b>sdf</b>
   
   # lrwxrwxrwx 1 root root  9 Aug  9 13:32 /dev/disk/by-id/scsi-1LIO-ORG_sbdnfs:f88f30e7-c968-4678-bc87-fe7bfcbdb625 -> ../../sdf
   # <b>lrwxrwxrwx 1 root root  9 Aug  9 13:32 /dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf -> ../../sdf</b>
   # lrwxrwxrwx 1 root root  9 Aug  9 13:32 /dev/disk/by-id/scsi-SLIO-ORG_sbdnfs_f88f30e7-c968-4678-bc87-fe7bfcbdb625 -> ../../sdf
   </code></pre>

   此命令會為每個 SBD 裝置列出三個裝置識別碼。 我們建議使用以 scsi-3 開頭的識別碼，在上述範例中即為

   * **/dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03**
   * **/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df**
   * **/dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf**

1. **[1]** 建立 SBD 裝置

   使用 iSCSI 裝置的裝置識別碼，在第一個叢集節點上建立新的 SBD 裝置。

   <pre><code>sudo sbd -d <b>/dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03</b> -1 60 -4 120 create

   # Also create the second and third SBD devices if you want to use more than one.
   sudo sbd -d <b>/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df</b> -1 60 -4 120 create
   sudo sbd -d <b>/dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf</b> -1 60 -4 120 create
   </code></pre>

1. **[A]** 調整 SBD 組態

   開啟 SBD 組態檔

   <pre><code>sudo vi /etc/sysconfig/sbd
   </code></pre>

   變更 SBD 裝置的屬性，啟用 Pacemaker 整合，並且變更 SBD 的啟動模式。

   <pre><code>[...]
   <b>SBD_DEVICE="/dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03;/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df;/dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf"</b>
   [...]
   <b>SBD_PACEMAKER="yes"</b>
   [...]
   <b>SBD_STARTMODE="always"</b>
   [...]
   <b>SBD_WATCHDOG="yes"</b>
   </code></pre>

   建立 `softdog` 設定檔

   <pre><code>echo softdog | sudo tee /etc/modules-load.d/softdog.conf
   </code></pre>

   立即載入模組

   <pre><code>sudo modprobe -v softdog
   </code></pre>

## <a name="cluster-installation"></a>叢集安裝

下列項目會加上下列其中一個前置詞： **[A]** - 適用於所有節點、 **[1]** - 僅適用於節點 1 或 **[2]** - 僅適用於節點 2。

1. **[A]** 更新 SLES

   <pre><code>sudo zypper update
   </code></pre>

1. **[A]** 安裝叢集資源所需的元件

   <pre><code>sudo zypper in socat
   </code></pre>

1. **[A]** 設定作業系統

   在某些情況下，Pacemaker 會建立許多程序，因此用盡允許的程序數目。 在此情況下，叢集節點之間的活動訊號可能會失敗，而導致您的資源容錯移轉。 我們建議設定下列參數，以增加允許的程序上限。

   <pre><code># Edit the configuration file
   sudo vi /etc/systemd/system.conf
   
   # Change the DefaultTasksMax
   #DefaultTasksMax=512
   DefaultTasksMax=4096
   
   #and to activate this setting
   sudo systemctl daemon-reload
   
   # test if the change was successful
   sudo systemctl --no-pager show | grep DefaultTasksMax
   </code></pre>

   縮減已變更的快取大小。 如需詳細資訊，請參閱[使用大量 RAM 的 SLES 11/12 伺服器出現低寫入效能](https://www.suse.com/support/kb/doc/?id=7010287) (英文)。

   <pre><code>sudo vi /etc/sysctl.conf

   # Change/set the following settings
   vm.dirty_bytes = 629145600
   vm.dirty_background_bytes = 314572800
   </code></pre>

1. **[A]** 設定適用于 HA 叢集的雲端 netconfig-azure

   變更網路介面的設定檔，如下所示，以防止雲端網路外掛程式移除虛擬 IP 位址（Pacemaker 必須控制 VIP 指派）。 如需詳細資訊，請參閱[SUSE KB 7023633](https://www.suse.com/support/kb/doc/?id=7023633)。 

   <pre><code># Edit the configuration file
   sudo vi /etc/sysconfig/network/ifcfg-eth0 
   
   # Change CLOUD_NETCONFIG_MANAGE
   # CLOUD_NETCONFIG_MANAGE="yes"
   CLOUD_NETCONFIG_MANAGE="no"
   </code></pre>

1. **[1]** 啟用 SSH 存取

   <pre><code>sudo ssh-keygen
   
   # Enter file in which to save the key (/root/.ssh/id_rsa): -> Press ENTER
   # Enter passphrase (empty for no passphrase): -> Press ENTER
   # Enter same passphrase again: -> Press ENTER
   
   # copy the public key
   sudo cat /root/.ssh/id_rsa.pub
   </code></pre>

1. **[2]** 啟用 SSH 存取

   <pre><code>
   sudo ssh-keygen
   
   # Enter file in which to save the key (/root/.ssh/id_rsa): -> Press ENTER
   # Enter passphrase (empty for no passphrase): -> Press ENTER
   # Enter same passphrase again: -> Press ENTER
   
   # insert the public key you copied in the last step into the authorized keys file on the second server
   sudo vi /root/.ssh/authorized_keys   
   
   # copy the public key
   sudo cat /root/.ssh/id_rsa.pub
   </code></pre>

1. **[1]** 啟用 SSH 存取

   <pre><code># insert the public key you copied in the last step into the authorized keys file on the first server
   sudo vi /root/.ssh/authorized_keys
   </code></pre>

1. **[A]** 安裝柵欄代理程式
   
   <pre><code>sudo zypper install fence-agents
   </code></pre>

   >[!IMPORTANT]
   > 如果使用 Suse Linux Enterprise Server for SAP 15，請注意您必須啟用其他模組並安裝其他元件，這是使用 Azure 隔離代理程式的必要條件。 若要深入瞭解 SUSE 模組和擴充功能，請參閱[說明的模組和擴充](https://www.suse.com/documentation/sles-15/singlehtml/art_modules/art_modules.html)功能。 遵循鈴的指示來安裝 Azure Python SDK。 

   下列有關如何安裝 Azure Python SDK 的指示僅適用于 SAP **15**的 Suse Enterprise Server。  

    - 如果您使用「自備訂用帳戶」，請遵循這些指示  

    <pre><code>
    #Activate module PackageHub/15/x86_64
    sudo SUSEConnect -p PackageHub/15/x86_64
    #Install Azure Python SDK
    sudo zypper in python3-azure-sdk
    </code></pre>

     - 如果您使用隨用隨付訂用帳戶，請遵循這些指示  

    <pre><code>#Activate module PackageHub/15/x86_64
    zypper ar https://download.opensuse.org/repositories/openSUSE:/Backports:/SLE-15/standard/ SLE15-PackageHub
    #Install Azure Python SDK
    sudo zypper in python3-azure-sdk
    </code></pre>

1. **[A]** 設定主機名稱解析

   您可以使用 DNS 伺服器，或修改所有節點上的 /etc/hosts。 這個範例示範如何使用 /etc/hosts 檔案。
   請取代下列命令中的 IP 位址和主機名稱。 使用 /etc/hosts 的好處在於，您的叢集會變成不受 DNS 影響，而 DNS 也可能是單一失敗點。

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   將下列幾行插入至 /etc/hosts。 變更 IP 位址和主機名稱以符合您的環境   

   <pre><code># IP address of the first cluster node
   <b>10.0.0.6 prod-cl1-0</b>
   # IP address of the second cluster node
   <b>10.0.0.7 prod-cl1-1</b>
   </code></pre>

1. **[1]** 安裝叢集

   <pre><code>sudo ha-cluster-init -u
   
   # ! NTP is not configured to start at system boot.
   # Do you want to continue anyway (y/n)? <b>y</b>
   # /root/.ssh/id_rsa already exists - overwrite (y/n)? <b>n</b>
   # Address for ring0 [10.0.0.6] <b>Press ENTER</b>
   # Port for ring0 [5405] <b>Press ENTER</b>
   # SBD is already configured to use /dev/disk/by-id/scsi-36001405639245768818458b930abdf69;/dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03;/dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf - overwrite (y/n)? <b>n</b>
   # Do you wish to configure an administration IP (y/n)? <b>n</b>
   </code></pre>

1. **[2]** 將節點新增至叢集

   <pre><code>sudo ha-cluster-join
   
   # ! NTP is not configured to start at system boot.
   # Do you want to continue anyway (y/n)? <b>y</b>
   # IP address or hostname of existing node (e.g.: 192.168.1.1) []<b>10.0.0.6</b>
   # /root/.ssh/id_rsa already exists - overwrite (y/n)? <b>n</b>
   </code></pre>

1. **[A]** 將 hacluster 密碼變更為同一個密碼

   <pre><code>sudo passwd hacluster
   </code></pre>

1. **[A]** 調整 corosync 設定。  

   <pre><code>sudo vi /etc/corosync/corosync.conf
   </code></pre>

   將下列粗體內容新增至檔案 (如果檔案中沒有這些值或不同)。 請務必將權杖變更為 30000，以允許記憶體保留維護。 如需詳細資訊, 請參閱[適用于 Linux][virtual-machines-linux-maintenance] 或 [Windows][virtual-machines-windows-maintenance] 的這篇文章。

   <pre><code>[...]
     <b>token:          30000
     token_retransmits_before_loss_const: 10
     join:           60
     consensus:      36000
     max_messages:   20</b>
     
     interface { 
        [...] 
     }
     transport:      udpu
   } 
   nodelist {
     node {
      ring0_addr:10.0.0.6
     }
     node {
      ring0_addr:10.0.0.7
     } 
   }
   logging {
     [...]
   }
   quorum {
        # Enable and configure quorum subsystem (default: off)
        # see also corosync.conf.5 and votequorum.5
        provider: corosync_votequorum
        <b>expected_votes: 2</b>
        <b>two_node: 1</b>
   }
   </code></pre>

   然後重新啟動 corosync 服務

   <pre><code>sudo service corosync restart
   </code></pre>

## <a name="create-azure-fence-agent-stonith-device"></a>建立 Azure Fence 代理程式 STONITH 裝置

STONITH 裝置會使用服務主體來對 Microsoft Azure 授權。 請遵循下列步驟來建立服務主體。

1. 移至 <https://portal.azure.com>。
1. 開啟 [Azure Active Directory] 刀鋒視窗  
   移至 [屬性]，並記下目錄識別碼。 這是「租用戶識別碼」。
1. 按一下 [應用程式註冊]
1. 按一下 [新增註冊]
1. 輸入名稱，選取 [僅此組織目錄中的帳戶] 
2. 選取應用程式類型 [Web]，輸入登入 URL （例如 HTTP：\//localhost），然後按一下 [新增]  
   登入 URL 並未使用，而且可以是任何有效的 URL
1. 選取 [憑證和密碼]，然後按一下 [新增用戶端密碼]
1. 輸入新金鑰的描述，選取 [永不過期]，然後按一下 [新增]
1. 記下值。 此值會用來做為服務主體的**密碼**
1. 選取 [總覽]。 記下應用程式識別碼。 此識別碼會用來做為服務主體的使用者名稱 (以下步驟中的「登入識別碼」)

### <a name="1-create-a-custom-role-for-the-fence-agent"></a>**[1]** 為柵欄代理程式建立自訂角色

根據預設，服務主體不具有存取 Azure 資源的許可權。 您需要為服務主體提供權限來啟動和停止 (解除配置) 叢集的所有虛擬機器。 如果您尚未建立自訂角色，您可以使用 [PowerShell](https://docs.microsoft.com/azure/role-based-access-control/custom-roles-powershell#create-a-custom-role) 或 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/custom-roles-cli) 來建立

針對輸入檔使用下列內容。 您必須調整訂用帳戶的內容，也就是使用您訂用帳戶的識別碼取代 c276fc76-9cd4-44c9-99a7-4fd71546436e 和 e91d47c4-76f3-4271-a796-21b4ecfe3624。 如果您只有一個訂用帳戶，請在 AssignableScopes 中移除第二個項目。

```json
{
  "Name": "Linux Fence Agent Role",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows to deallocate and start virtual machines",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/start/action", 
    "Microsoft.Compute/virtualMachines/powerOff/action" 
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e",
    "/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624"
  ]
}
```

### <a name="a-assign-the-custom-role-to-the-service-principal"></a>**[A]** 將自訂角色指派給服務主體

將在上一章中建立的自訂角色「Linux 柵欄代理程式角色」指派給服務主體。 不再使用擁有者角色！

1. 移至 [https://portal.azure.com](https://portal.azure.com)
1. 開啟 [所有資源] 刀鋒視窗
1. 選取第一個叢集節點的虛擬機器
1. 選取 [存取控制 (IAM)]
1. 按一下 [新增角色指派]
1. 選取「Linux 柵欄代理程式角色」角色
1. 輸入您先前建立的應用程式名稱
1. 按一下 [Save] \(儲存)。

針對第二個叢集節點重複上述步驟。

### <a name="1-create-the-stonith-devices"></a>**[1]** 建立 STONITH 裝置

當您編輯虛擬機器的權限之後，就可以在叢集中設定 STONITH 裝置。

<pre><code># replace the bold string with your subscription ID, resource group, tenant ID, service principal ID and password
sudo crm configure primitive rsc_st_azure stonith:fence_azure_arm \
   params subscriptionId="<b>subscription ID</b>" resourceGroup="<b>resource group</b>" tenantId="<b>tenant ID</b>" login="<b>login ID</b>" passwd="<b>password</b>"

sudo crm configure property stonith-timeout=900
sudo crm configure property stonith-enabled=true
</code></pre>

## <a name="default-pacemaker-configuration-for-sbd"></a>SBD 的預設 Pacemaker 組態

1. **[1]** 能夠啟用 STONITH 裝置及設定柵欄延遲

<pre><code>sudo crm configure property stonith-timeout=144
sudo crm configure property stonith-enabled=true

# List the resources to find the name of the SBD device
sudo crm resource list
sudo crm resource stop stonith-sbd
sudo crm configure delete <b>stonith-sbd</b>
sudo crm configure primitive <b>stonith-sbd</b> stonith:external/sbd \
   params pcmk_delay_max="15" \
   op monitor interval="15" timeout="15"
</code></pre>

## <a name="pacemaker-configuration-for-azure-scheduled-events"></a>Azure 已排程事件的 Pacemaker 設定

Azure 提供[排程的事件](https://docs.microsoft.com/azure/virtual-machines/linux/scheduled-events)。 排定的事件是透過中繼資料服務所提供，並可讓應用程式針對 VM 關機、VM 重新部署等事件進行準備。資源代理程式 **[azure-](https://github.com/ClusterLabs/resource-agents/pull/1161)** 已排程 azure 事件的事件監視。 如果偵測到事件，代理程式將會嘗試停止受影響 VM 上的所有資源，並將其移至叢集中的另一個節點。 若要達到此目的，必須設定額外的 Pacemaker 資源。 

1. **[A]** 安裝**azure 事件**代理程式。 

<pre><code>sudo zypper install resource-agents
</code></pre>

2. **[1]** 在 Pacemaker 中設定資源。 

<pre><code>
#Place the cluster in maintenance mode
sudo crm configure property maintenance-mode=true

#Create Pacemaker resources for the Azure agent
sudo crm configure primitive rsc_azure-events ocf:heartbeat:azure-events op monitor interval=10s
sudo crm configure clone cln_azure-events rsc_azure-events

#Take the cluster out of maintenance mode
sudo crm configure property maintenance-mode=false
</code></pre>

   > [!NOTE]
   > 設定 azure 事件代理程式的 Pacemaker 資源之後，當您將叢集放入或移出維護模式時，您可能會收到類似下列的警告訊息：  
     警告： cib-啟動程式-選項：不明的屬性 ' hostName_<strong>主機名稱</strong>'  
     警告： cib-啟動程式-選項：不明的屬性 ' azure-events_globalPullState '  
     警告： cib-啟動程式-選項：不明的屬性 ' hostName_<strong>主機名稱</strong>'  
   > 您可以忽略這些警告訊息。

## <a name="next-steps"></a>後續步驟

* [適用于 SAP 的 Azure 虛擬機器規劃和執行][planning-guide]
* [適用于 SAP 的 Azure 虛擬機器部署][deployment-guide]
* [適用于 SAP 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* [SUSE Linux Enterprise Server 上 Azure Vm 上 NFS 的高可用性][sles-nfs-guide]
* [SAP NetWeaver 在適用於 SAP 應用程式之 SUSE Linux Enterprise Server 上的 Azure VM 高可用性][sles-guide]
* 若要瞭解如何建立高可用性並規劃 Azure Vm 上 SAP Hana 的嚴重損壞修復，請參閱[azure 虛擬機器（vm）上 SAP Hana 的高可用性][sap-hana-ha]
