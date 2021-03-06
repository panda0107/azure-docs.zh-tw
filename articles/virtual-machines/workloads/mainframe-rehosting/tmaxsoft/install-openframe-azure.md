---
title: 在 Azure 上安裝 TmaxSoft OpenFrame 虛擬機器
description: 使用 Azure 虛擬機器（Vm）上的 TmaxSoft OpenFrame 環境，重新裝載您的 IBM z/OS 大型主機工作負載。
services: virtual-machines-linux
documentationcenter: ''
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: virtual-machines-linux
ms.openlocfilehash: 1ad6e52c421d9cfec4640d3a330b5507d6ed3e9b
ms.sourcegitcommit: 77bfc067c8cdc856f0ee4bfde9f84437c73a6141
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2019
ms.locfileid: "72436058"
---
# <a name="install-tmaxsoft-openframe-on-azure"></a>在 Azure 上安裝 TmaxSoft OpenFrame

瞭解如何在 Azure 上設定適用于開發、示範、測試或生產工作負載的 OpenFrame 環境。 本教學課程將逐步引導您完成每個步驟。

OpenFrame 包含多個在 Azure 上建立大型主機模擬環境的元件。 例如，OpenFrame 線上服務將 IBM 客戶資訊控制系統（CICS）和 OpenFrame Batch 等大型主機中介軟體取代為其 TJES 元件，以取代 IBM 大型主機的作業進入子系統（JES）。

OpenFrame 適用于任何關係資料庫，包括 Oracle Database、Microsoft SQL Server、IBM Db2 和 MySQL。 這項 OpenFrame 安裝會使用 TmaxSoft Tibero 關係資料庫。 OpenFrame 和 Tibero 都是在 Linux 作業系統上執行。 本教學課程會安裝 CentOS 7.3，不過您可以使用其他支援的 Linux 發行版本。OpenFrame 應用程式伺服器和 Tibero 資料庫會安裝在一部虛擬機器（VM）上。

本教學課程會引導您完成安裝 OpenFrame suite 元件。 有些必須分開安裝。

主要 OpenFrame 元件：

- 必要的安裝套件。
- Tibero 資料庫。
- OpenFrame 中的應用程式會使用開放式資料庫連接（ODBC）來與 Tibero 資料庫通訊。
- OpenFrame 基底，這是管理整個系統的中介軟體。
- OpenFrame 批次，這是取代大型主機批次系統的解決方案。
- TACF，此服務模組可控制使用者對系統和資源的存取。
- ProSort，這是批次交易的排序工具。
- OFCOBOL，這是一種用來解讀大型主機 COBOL 程式的編譯器。
- OFASM，這是用來解讀大型主機組合程式的編譯器。
- OpenFrame 伺服器類型 C （.OSC），這是取代大型主機中介軟體和 IBM CICS 的解決方案。
- JAVA 企業使用者解決方案（JEUS），這是一種已通過 JAVA Enterprise Edition 6 認證的 web 應用程式伺服器。
- OFGW，這是提供3270接聽程式的 OpenFrame 閘道元件。
- OFManager，這是在 web 環境中提供 OpenFrame 操作和管理功能的解決方案。

其他必要的 OpenFrame 元件：

- OSI，這是取代大型主機中介軟體和 IMS DC 的解決方案。
- TJES，這是提供大型主機 JES 環境的解決方案。
- OFTSAM，這是可在開放系統中使用（V） SAM 檔案的解決方案。
- OFHiDB，這是取代大型主機 IMS DB 的解決方案。
- OFPLI，這是一種用來解讀大型主機 PL/I 程式的編譯器。
- PROTRIEVE，這是執行大型主機語言 CA 的解決方案-Easytrieve。
- OFMiner，此解決方案會分析大型主機資產，然後將其遷移至 Azure。

## <a name="architecture"></a>架構

下圖概述本教學課程中所安裝的 OpenFrame 7.0 架構元件：

![OpenFrame 元件](media/openframe-02.png)

## <a name="azure-system-requirements"></a>Azure 系統需求

下表列出在 Azure 上安裝的需求。
<!-- markdownlint-disable MD033 -->

<table>
<thead>
    <tr><th>需求</th><th>描述</th></tr>
</thead>
<tbody>
<tr><td>Azure 上支援的 Linux 發行版本
</td>
<td>
Linux x86 2.6 （32位，64位）<br/>
Red Hat 7。x<br/>
CentOS 7。x<br/>
</td>
</tr>
<tr><td>硬體
</td>
<td>核心：2（最少）<br/>
記憶體： 4 GB （最低需求）<br/>
交換空間： 1 GB （最低需求）<br/>
硬碟： 100 GB （最低需求）<br/>
</td>
</tr>
<tr><td>適用于 Windows 使用者的選用軟體
</td>
<td>PuTTY：本指南中用來設定 VM 功能<br/>
WinSCP：您可以使用的熱門 SFTP 用戶端和 FTP 用戶端<br/>
適用于 Windows 的 Eclipse： TmaxSoft 支援的開發平臺<br/>
（目前不支援 Microsoft Visual Studio）
</td>
</tr>
</tbody>
</table>

<!-- markdownlint-enable MD033 -->

## <a name="prerequisites"></a>先決條件

規劃花幾天的時間來組合所有必要的軟體，並完成所有的手動程式。

在開始使用之前，請執行下列動作：

- 從 TmaxSoft 取得 OpenFrame 安裝媒體。 如果您是現有的 TmaxSoft 客戶，請洽詢您的 TmaxSoft 代表以取得授權複本。 否則，請從[TmaxSoft](https://www.tmaxsoft.com/contact/)要求試用版。

- 傳送電子郵件給 <support@tmaxsoft.com>，以要求 OpenFrame 檔。

- 取得 Azure 訂用帳戶（如果還沒有的話）。 您也可以在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

- 選用。 設定站對站 VPN 通道或 jumpbox，將 Azure VM 的存取限制為組織中允許的使用者。 這不是必要步驟，但這是最佳做法。

## <a name="set-up-a-vm-on-azure-for-openframe-and-tibero"></a>在 Azure 上為 OpenFrame 和 Tibero 設定 VM

您可以使用各種部署模式來設定 OpenFrame 環境，但下列程式會示範如何在一個 VM 上部署 OpenFrame 應用程式伺服器和 Tibero 資料庫。 在較大型的環境和相當大工作負載中，最佳作法是在自己的 VM 上個別部署資料庫，以獲得更好的效能。

**建立 VM**

1. 前往 <https://portal.azure.com> 的 Azure 入口網站，然後登入您的帳戶。

2. 按一下 [虛擬機器]。

    ![Azure 入口網站中的資源清單](media/vm-01.png)

3. 按一下 [新增]。

    ![在 Azure 入口網站中新增選項](media/vm-02.png)

4. 在 [**作業系統**] 右邊，按一下 [**更多**]。

     ![Azure 入口網站中的更多選項](media/vm-03.png)

5. 按一下 [以**CentOS 為基礎的 7.3** ] 可完全遵循此逐步解說，或者您也可以選擇其他支援的 Linux 發行版本。

     ![Azure 入口網站中的作業系統選項](media/vm-04.png)

6. 在 [**基本**] 設定中，輸入 [**名稱**]、[**使用者名稱**]、[**驗證類型**]、[**訂**用帳戶] （隨用隨付是 AWS 的付款樣式）和 [**資源群組**] （使用現有的帳戶或建立 TmaxSoft 群組）。

7. 完成時（包括**驗證類型**的公開/私密金鑰組），按一下 [**提交**]。

> [!NOTE]
> 如果使用**驗證類型**的 SSH 公開金鑰，請參閱下一節中的步驟來產生公開/私密金鑰組，然後繼續這裡的步驟。

### <a name="generate-a-publicprivate-key-pair"></a>產生公開/私密金鑰組

如果您使用的是 Windows 作業系統，則需要 PuTTYgen 來產生公開/私密金鑰組。

公開金鑰可以自由共用，但私密金鑰應該保持完全機密，而且永遠不會與另一方共用。 產生金鑰之後，您必須將**SSH 公開金鑰**貼入設定中，有效地將它上傳至 Linux VM。 它會儲存在使用者帳戶主目錄的 \~/.ssh 目錄中授權\_金鑰內。 一旦您在 SSH 用戶端（在我們的案例中為 PuTTY）提供相關聯的**ssh 私密金鑰**，Linux VM 就可以辨識及驗證連線。

提供新的個人存取 VM 時： 

- 每個新的人員都會使用 PuTTYgen 產生自己的公開/私密金鑰。
- 個人會分別儲存自己的私密金鑰，並將公開金鑰資訊傳送給 VM 的系統管理員。
- 系統管理員會將公開金鑰的內容貼入 \~/.ssh/authorized\_金鑰檔案。
- 新的人員會透過 PuTTY 連接。

**產生公開/私密金鑰組**

1.  從 <https://www.putty.org/> 下載 PuTTYgen，並使用所有預設設定來安裝。

2.  若要開啟 PuTTYgen，請在 C：\\Program Files\\PuTTY 中找出 PuTTY 安裝目錄。

    ![PuTTY 介面](media/puttygen-01.png)

3.  按一下 [產生]。

    ![[PuTTY 金鑰產生器] 對話方塊](media/puttygen-02.png)

4.  產生後，請儲存公開金鑰和私密金鑰。 在 [**建立虛擬機器 \> 基本概念**] 窗格的 [ **SSH 公開金鑰**] 區段中，貼上公開金鑰的內容（如上一節的步驟6和7所示）。

    ![[PuTTY 金鑰產生器] 對話方塊](media/puttygen-03.png)

### <a name="configure-vm-features"></a>設定 VM 功能

1. 在 Azure 入口網站的 [**選擇大小**] 分頁中，選擇您想要的 Linux 電腦硬體設定。 同時安裝 Tibero 和 OpenFrame 的*最低*需求是2個 cpu 和 4 GB 的 RAM，如下列範例安裝所示：

    ![建立虛擬機器-基本概念](media/create-vm-01.png)

2. 按一下 [ **3 個設定**]，並使用預設設定來設定選用功能。
3. 檢查您的付款詳細資料。

    ![建立虛擬機器-購買](media/create-vm-02.png)

4. 提交您的選取專案。 Azure 會開始部署 VM。 此程式通常需要幾分鐘的時間。

5. 部署 VM 時，會顯示其儀表板，其中顯示設定期間選取的所有設定。 記下 [**公用 IP 位址**]。

    ![Azure 儀表板上的 tmax](media/create-vm-03.png)

6. 開啟 PuTTY。

7. 在 [**主機名稱**] 中，輸入您的使用者名稱和您複製的公用 IP 位址。 例如， **username\@publicip**。

    ![[PuTTY 設定] 對話方塊](media/putty-01.png)

8. 在 [**類別**] 方塊中，按一下 [**連接] \> SSH \> Auth**。提供您的**私密金鑰**檔案的路徑。

    ![[PuTTY 設定] 對話方塊](media/putty-02.png)

9. 按一下 [**開啟**] 以啟動 [PuTTY] 視窗。 如果成功，則會連線到在 Azure 上執行的新 CentOS VM。

10. 若要以 root 使用者身分登入，請輸入**sudo bash**。

    ![命令視窗中的根使用者登入](media/putty-03.png)

## <a name="set-up-the-environment-and-packages"></a>設定環境和套件

既然 VM 已建立且您已登入，您必須執行幾個設定步驟，並安裝必要的預先安裝套件。

1. 使用 vi 來編輯 hosts 檔案（`vi /etc/hosts`），以將名稱**ofdemo**對應至本機 IP 位址。 假設我們的 IP 是 192.168.96.148 ofdemo，這會在變更之前：

    ```vi
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
    ::1              localhost localhost.localdomain localhost6 localhost6.localdomain 
    <IP Address>    <your hostname>
    ```

     這是在變更之後：

    ```vi
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
    ::1              localhost localhost.localdomain localhost6 localhost6.localdomain 
    192.168.96.148   ofdemo
    ```

2. 建立群組和使用者：

    ```vi
    [root@ofdemo ~]# adduser -d /home/oframe7 oframe7 
    [root@ofdemo ~]# passwd oframe7
    ```

3. 變更使用者 oframe7 的密碼：

    ```vi
    New password: 
    Retype new password: 
    passwd: all authentication tokens updated successfully.
    ```

4. 更新新增至/etc/sysctl.conf 中的核心參數：

    ```vi
    [root@ofdemo ~]# vi /etc/sysctl.conf
    kernel.shmall = 7294967296 
    kernel.sem = 10000 32000 10000 10000
    ```

5. 動態重新整理核心參數，而不需要重新開機：

    ```vi
    [root@ofdemo ~]# /sbin/sysctl -p
    ```

6. 取得所需的套件：確定伺服器已連線到網際網路、下載下列套件，然後加以安裝：

     - dos2unix
     - glibc
     - glibc. i686 glibc. x86\_64
     - libaio
     - ncurses

          > [!NOTE]
          > 安裝 ncurses 套件之後，請建立下列符號連結：
         ```
         ln -s /usr/lib64/libncurses.so.5.9 /usr/lib/libtermcap.so
         ln -s /usr/lib64/libncurses.so.5.9 /usr/lib/libtermcap.so.2
         ```

     - gcc
     - gcc-c + +
     - libaio-對內. x86\_64
     - strace
     - ltrace
     - gdb

7. 在安裝 JAVA RPM 時，請執行下列動作：

```
root@ofdemo ~]# rpm -ivh jdk-7u79-linux-x64.rpm
[root@ofdemo ~]# vi .bash_profile

# JAVA ENV
export JAVA_HOME=/usr/java/jdk1.7.0_79/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar

[root@ofdemo ~]# source /etc/profile
[root@ofdemo ~]# java –version

java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

[root@ofdemo ~]# echo $JAVA_HOME /usr/java/jdk1.7.0_79/
```

## <a name="install-the-tibero-database"></a>安裝 Tibero 資料庫

Tibero 在 Azure 上的 OpenFrame 環境中提供了數個主要功能：

- Tibero 是用來作為各種系統功能的 OpenFrame 內部資料存放區。
- VSAM 檔案（包括 KSDS、RRDS 和 ESDS）會在內部使用 Tibero 資料庫來儲存資料。
- TACF 資料存放庫會儲存在 Tibero 中。
- OpenFrame 目錄資訊儲存在 Tibero 中。
- Tibero 資料庫可以用來取代 IBM Db2 以儲存應用程式資料。

**若要安裝 Tibero**

1. 確認 Tibero 二進位安裝程式檔案存在，並檢查版本號碼。
2. 將 Tibero 軟體複製到 Tibero 使用者帳戶（oframe）。 例如︰

    ```
    [oframe7@ofdemo ~]$ tar -xzvf tibero6-bin-6_rel_FS04-linux64-121793-opt-tested.tar.gz 
    [oframe7@ofdemo ~]$ mv license.xml /opt/tmaxdb/tibero6/license/
    ```

3. 在 vi （`vi .bash_profile`）中開啟 bash\_設定檔，並在其中貼上下列內容：

    ```
    # Tibero6 ENV
    export TB_HOME=/opt/tmaxdb/tibero6 
    export TB_SID=TVSAM export TB_PROF_DIR=$TB_HOME/bin/prof 
    export LD_LIBRARY_PATH=$TB_HOME/lib:$TB_HOME/client/lib:$LD_LIBRARY_PATH 
    export PATH=$TB_HOME/bin:$TB_HOME/client/bin:$PATH
    ```

4. 若要執行 bash 設定檔，請在命令提示字元中輸入：

    ```
    source .bash_profile
    ```

5. 產生 tip 檔案（Tibero 的設定檔案），然後在 vi 中開啟它。 例如︰

    ```
    [oframe7@ofdemo ~]$ sh $TB_HOME/config/gen_tip.sh
    [oframe7@ofdemo ~]$ vi $TB_HOME/config/$TB_SID.tip
    ```

6. 修改 \$TB\_HOME/client/config/tbdsn. tbr 並將127.0.0.1 改為 oflocalhost，如下所示：

    ```
    TVSAM=( 
    (INSTANCE=(HOST=127.0.0.1)
        (PT=8629)
    (DB_NAME=TVSAM)
          )
     )
    ```

7. 建立資料庫。 下列輸出隨即出現：

    ```
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof.
    Listener port = 8629
    Tibero 6
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NOMOUNT mode).
     /--------------------- newmount sql ------------------------/
    create database character set MSWIN949 national character set UTF16;
    /-----------------------------------------------------------/
    Database created.
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof.
    Listener port = 8629
    Tibero 6
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NORMAL mode).
    /opt/tmaxdb/tibero6/bin/tbsvr
    ………………………..
    Creating agent table...
    Done.
    For details, check /opt/tmaxdb/tibero6/instance/TVSAM/log/system_init.log.
    ************************************************** 
    * Tibero Database TVSAM is created successfully on Fri Aug 12 19:10:43 UTC 2016.
    *     Tibero home directory ($TB_HOME) =
    *         /opt/tmaxdb/tibero6
    *     Tibero service ID ($TB_SID) = TVSAM
    *     Tibero binary path =
    *         /opt/tmaxdb/tibero6/bin:/opt/tmaxdb/tibero6/client/bin
    *     Initialization parameter file =
    *         /opt/tmaxdb/tibero6/config/TVSAM.tip
    * 
    * Make sure that you always set up environment variables $TB_HOME and
    * $TB_SID properly before you run Tibero.
     ******************************************************************************
    ```

8. 若要回收 Tibero，請先使用 `tbdown` 命令將它關閉。 例如︰

    ```
    [oframe7@ofdemo ~]$$ tbdown 
    Tibero instance terminated (NORMAL mode).
    ```

9. 現在使用 `tbboot`啟動 Tibero。 例如︰

    ```
    [oframe7@ofdemo ~]$ tbboot
    Change core dump dir to /opt/tmaxdb/tibero6/bin/prof. Listener port = 8629

    Tibero 6  
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Tibero instance started up (NORMAL mode).
    ```

10. 若要建立資料表空間，請使用 SYS user （sys/tmax）存取資料庫，然後為預設磁片區和 TACF 建立必要的資料表空間：

    ```
    [oframe7@ofdemo ~]$ tbsql tibero/tmax
    tbSQL 6  
    TmaxData Corporation Copyright (c) 2008-. All rights reserved.
    Connected to Tibero.
    ```

11. 現在，輸入下列 SQL 命令：

    ```
    SQL> create tablespace "DEFVOL" datafile 'DEFVOL.dbf' size 500M autoextend on; create tablespace "TACF00" datafile 'TACF00.dbf' size 500M autoextend on; create tablespace "OFM_REPOSITORY" datafile 'ofm_repository.dbf' size 300M autoextend on;
    SQL> Tablespace 'DEFVOL' created.
    SQL> Tablespace 'TACF00' created.
    SQL> Tablespace ' OFM_REPOSITORY ' created.
    SQL> SQL> Disconnected.
    ```

12. 開機 Tibero，並確認 Tibero 進程正在執行：

    ```
    [oframe7@ofdemo ~]$ tbboot 
    ps -ef | egrep tbsvr
    ```

輸出：

![Tibero 輸出](media/tibero-01.png)

## <a name="install-odbc"></a>安裝 ODBC

OpenFrame 中的應用程式會使用開放原始碼 unixODBC 專案所提供的 ODBC API，與 Tibero 資料庫通訊。

若要安裝 ODBC：

1. 確認 Unixodbc-2.3.1.tar.gz 2.3.4 gz 安裝程式檔案存在，或使用 `wget unixODBC-2.3.4.tar.gz` 命令。 例如︰

     ```
     [oframe7@ofdemo ~]$ wget ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.4.tar.gz
     ```

2. 將二進位檔解壓縮。 例如︰

     ```
     [oframe7@ofdemo ~]$ tar -zxvf unixODBC-2.3.4.tar.gz
     ```

3. 流覽至 unixODBC-2.3.4 目錄，並使用檢查電腦資訊來產生 Makefile。 例如︰

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ ./configure --prefix=/opt/tmaxapp/unixODBC/ --sysconfdir=/opt/tmaxapp/unixODBC/etc
     ```

     根據預設，unixODBC 會安裝在/usr/local 中，因此 `--prefix` 傳遞值來變更位置。 同樣地，設定檔預設會安裝在/etc 中，因此 `--sysconfdir` 會傳遞所需位置的值。

4. 執行 Makefile： `[oframe7@ofdemo unixODBC-2.3.4]$ make`

5. 編譯後，複製程式目錄中的可執行檔。 例如︰

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ make install
     ```

6. 使用 vi 編輯 bash 設定檔（`vi ~/.bash_profile`），並新增下列內容：

     ```
     # UNIX ODBC ENV 
     export ODBC_HOME=$HOME/unixODBC 
     export PATH=$ODBC_HOME/bin:$PATH 
     export LD_LIBRARY_PATH=$ODBC_HOME/lib:$LD_LIBRARY_PATH 
     export ODBCINI=$HOME/unixODBC/etc/odbc.ini 
     export ODBCSYSINI=$HOME
     ```

7. 套用 ODBC。 據以編輯下列檔案。 例如︰

     ```
     [oframe7@ofdemo unixODBC-2.3.4]$ source ~/.bash_profile

     [oframe7@ofdemo ~]$ cd

     [oframe7@ofdemo ~]$ odbcinst -j unixODBC 2.3.4
     DRIVERS............: /home/oframe7/odbcinst.ini
     SYSTEM DATA SOURCES: /home/oframe7/odbc.ini
     FILE DATA SOURCES..: /home/oframe7/ODBCDataSources
     USER DATA SOURCES..: /home/oframe7/unixODBC/etc/odbc.ini
     SQLULEN Size.......: 8
     SQLLEN Size........: 8
     SQLSETPOSIROW Size.: 8

     [oframe7@ofdemo ~]$ vi odbcinst.ini

     [Tibero]
     Description = Tibero ODBC driver for Tibero6
     Driver = /opt/tmaxdb/tibero6/client/lib/libtbodbc.so
     Setup = 
     FileUsage = 
     CPTimeout = 
     CPReuse = 
     Driver Logging = 7

     [ODBC]
     Trace = NO 
     TraceFile = /home/oframe7/odbc.log 
     ForceTrace = Yes 
     Pooling = No 
     DEBUG = 1

     [oframe7@ofdemo ~]$ vi odbc.ini

     [TVSAM]
     Description = Tibero ODBC driver for Tibero6 
     Driver = Tibero 
     DSN = TVSAM 
     SID = TVSAM 
     User = tibero 
     password = tmax
     ```

8. 建立符號連結並驗證 Tibero 資料庫連接：

     ```
     [oframe7@ofdemo ~]$ ln $ODBC_HOME/lib/libodbc.so $ODBC_HOME/lib/libodbc.so.1 [oframe7@ofdemo ~]$ ln $ODBC_HOME/lib/libodbcinst.so 
     $ODBC_HOME/lib/libodbcinst.so.1

     [oframe7@ofdemo lib]$ isql TVSAM tibero tmax
     ```

隨即顯示下列輸出：

![顯示已連接到 SQL 的 ODBC 輸出](media/odbc-01.png)

## <a name="install-openframe-base"></a>安裝 OpenFrame Base

基礎應用程式伺服器會在 OpenFrame 用來管理 Azure 系統的個別服務之前安裝，包括處理伺服器進程的交易。

**安裝 OpenFrame Base**

1. 請確定 Tibero 安裝成功，然後確認下列 OpenFrame\_Base7\_0\_Linux\_x86\_64. bin installer 檔案和 base. properties 設定檔案。

2. 使用下列 Tibero 特定資訊更新 bash 設定檔：

     ```bash
     alias ofhome='cd $OPENFRAME_HOME'
     alias ulog='cd $OPENFRAME_HOME/log/tmax/ulog'
     alias sysjcl='cd $OPENFRAME_HOME/volume_default/SYS1.JCLLIB'
     alias sysload='cd $OPENFRAME_HOME/volume_default/SYS1.LOADLIB'
     alias sysproc='cd $OPENFRAME_HOME/volume_default/SYS1.PROCLIB'
     alias oscsrc='cd $OPENFRAME_HOME/osc/oivp'
     alias osisrc='cd $OPENFRAME_HOME/osi/oivp'
     alias defvol='cd $OPENFRAME_HOME/volume_default'
     ```

3. 執行 bash 設定檔：`[oframe7@ofdemo ~]$ . .bash_profile`
4. 請確定 Tibero 進程正在執行。 例如︰

     ```linux
     [oframe7@ofdemo ~]$ ps -ef|grep tbsvr
     ```

    ![基本](media/base-01.png)

     > [!IMPORTANT]
     > 在安裝之前，請務必先啟動 Tibero。

5. 在[technet.tmaxsoft.com](https://technet.tmaxsoft.com/en/front/main/main.do)產生授權，並將 OpenFrame Base、BATCH、TACF、.OSC 授權放在適當的資料夾中：

     ```
     [oframe7@ofdemo ~]$ cp license.dat /opt/tmaxapp/OpenFrame/core/license/
     [oframe7@ofdemo ~]$ cp lictjes.dat lictacf.dat licosc.dat $OPENFRAME_HOME/license/
     ```

6. 下載 OpenFrame Base binary 和 base. properties 檔案：

     ```
     [oframe7@ofdemo ~]$ vi base.properties
     OPENFRAME_HOME= <appropriate location for installation> ex. /opt/tmaxapp/OpenFrame TP_HOST_NAME=<your IP Hostname> ex. ofdemo 
     TP_HOST_IP=<your IP Address> ex. 192.168.96.148 
     TP_SHMKEY=63481 
     TP_TPORTNO=6623 
     TP_UNBLOCK_PORT=6291 
     TP_NODE_NAME=NODE1 
     TP_NODE_LIST=NODE1 
     MASCAT_NAME=SYS1.MASTER.ICFCAT 
     MASCAT_CREATE=YES 
     DEFAULT_VOLSER=DEFVOL 
     VOLADD_DEFINE=YES TSAM_USERNAME=tibero 
     TSAM_PASSWORD=tmax 
     TSAM_DATABASE=oframe 
     DATASET_SHMKEY=63211 
     DSLOCK_DATA=SYS1.DSLOCK.DATA 
     DSLOCK_LOG=SYS1.DSLOCK.LOG 
     DSLOCK_SEQ=dslock_seq.dat 
     DSLOCK_CREATE=YES 
     OPENFRAME_LICENSE_PATH=/opt/tmaxapp/license/OPENFRAME TMAX_LICENSE_PATH=/opt/tmaxapp/license/TMAX
     ```

7. 使用基底. properties 檔案執行安裝程式。 例如︰

    ```
    [oframe7@ofdemo ~]$ chmod a+x OpenFrame_Base7_0_Linux_x86_64.bin 
    [oframe7@ofdemo ~]$ ./OpenFrame_Base7_0_Linux_x86_64.bin -f base.properties
    ```

    完成時，就會 diplayed 安裝完成訊息。

8. 使用 `ls -ltr` 命令來確認 OpenFrame 基底目錄結構。 例如︰

     ```
     [oframe7@ofdemo OpenFrame]$ ls -ltr
     total 44

     drwxrwxr-x. 4 oframe7 oframe7 61 Nov 30 16:57 UninstallerData 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 bin 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 cpm drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 data 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:57 include 
     drwxrwxr-x. 2 oframe7 oframe7 8192 Nov 30 16:57 lib 
     drwxrwxr-x. 6 oframe7 oframe7 48 Nov 30 16:57 log 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 profile 
     drwxrwxr-x. 7 oframe7 oframe7 62 Nov 30 16:57 sample 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 schema 
     drwxrwxr-x. 2 oframe7 oframe7 6 Nov 30 16:57 temp 
     drwxrwxr-x. 3 oframe7 oframe7 16 Nov 30 16:57 shared 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 license 
     drwxrwxr-x. 23 oframe7 oframe7 4096 Nov 30 16:58 core 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 config 
     drwxrwxr-x. 2 oframe7 oframe7 4096 Nov 30 16:58 scripts 
     drwxrwxr-x. 2 oframe7 oframe7 25 Nov 30 16:58 volume_default
     ```

9. 開始 OpenFrame 基底：

     ```
     [oframe7@ofdemo ~]$ cp /usr/lib/libtermcap.so.2 $TMAXDIR/lib
     Startup Tmax Server
     [oframe7@ofdemo ~]$ tmboot
     ```

     ![tmboot 命令輸出](media/base-02.png)

10. 使用 si 中的 tmadmin 命令，確認進程狀態已就緒。 RDY 會顯示在每個處理常式的 [**狀態**] 欄中：

     ![tmadmin 命令輸出](media/base-03.png)

11. 關閉 OpenFrame 基礎：

     ```
     [oframe7@ofdemo ~]$ tmdown 
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(vtammgr:43) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofruisvr:41) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: SERVER(ofrsmlog:42) downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: CLH downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: CLL downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TLM downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TMM downed: Wed Sep  7 15:37:21 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-openframe-batch"></a>安裝 OpenFrame 批次

OpenFrame 批次是由數個元件所組成，可模擬大型主機 Batch 環境，並用於在 Azure 上執行批次作業。

**若要安裝 Batch**

1. 請確定基底安裝成功，然後確認 OpenFrame\_Batch7\_0\_Fix2\_MVS\_Linux\_x86\_64. bin installer 檔案和 batch. properties 設定檔存在：

2. 在命令提示字元中，輸入 `vi batch.properties`，以使用 vi 來編輯 batch. properties 檔案。

3. 修改參數，如下所示：

     ```
     OPENFRAME_HOME = /opt/tmaxapp/OpenFrame
     DEFAULT_VOLSER=DEFVOL 
     TP_NODE_NAME=NODE1 
     TP_NODE_LIST=NODE1 
     RESOURCE_SHMKEY=66991 
     #JOBQ_DATASET_CREATE=YES 
     #OUTPUTQ_DATASET_CREATE=YES 
     DEFAULT_JCLLIB_CREATE=YES 
     DEFAULT_PROCLIB_CREATE=YES 
     DEFAULT_USERLIB_CREATE=YES 
     TJES_USERNAME=tibero 
     TJES_PASSWORD=tmax 
     TJES_DATABASE=oframe 
     BATCH_TABLE_CREATE=YES
     ```

4. 若要執行批次安裝程式，請在命令提示字元中輸入：

     ```
     ./OpenFrame_Batch7_0_Fix2_MVS_Linux_x86_64.bin -f batch.properties
     ```

5. 當安裝完成時，請在命令提示字元中輸入 `tmboot`，以啟動已安裝的 OpenFrame 套件。

    ![tmboot 輸出](media/tmboot-01.png)

6. 在命令提示字元中輸入 `tmadmin`，以檢查 OpenFrame 進程。

    ![Tmax 管理畫面](media/tmadmin-01.png)

7. 執行下列命令：

     ```
     $$2 NODE1 (tmadm): quit 
     ADM quit for node (NODE1)
     ```

8. 使用 `tmdown` 命令來啟動和關閉 Batch：

     ```
     [oframe7@ofdemo ~]$tmdown
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:44) downed: Wed Sep  7 16:01:46 2016
     TMDOWN: SERVER(vtammgr: 43) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:45) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:46) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:47) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjschd:54) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjinit:55) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:48) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjspbk:57) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:49) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:50) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:51) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:52) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjmsvr:53) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmjhist:56) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofruisvr:41) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(obmtsmgr:59) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrpmsvr:58) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: SERVER(ofrsmlog:42) downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: CLL downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TLM downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: CLH downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TMM downed: Wed Sep  7 16:01:46 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-tacf"></a>安裝 TACF

TACF Manager 是 OpenFrame 服務模組，透過 RACF 安全性控制使用者對系統和資源的存取。

**若要安裝 TACF**

1. 確認 OpenFrame\_Tacf7\_0\_Fix2\_Linux\_x86\_64. bin installer 檔案和 tacf. properties 設定檔案。
2. 請確定批次安裝成功，然後使用 vi 來開啟 tacf 檔案（`vi tacf.properties`）。
3. 修改 TACF 參數：

     ```
     OPENFRAME_HOME=/opt/tmaxapp/OpenFrame 
     USE_OS_AUTH=NO 
     TACF_USERNAME=tibero 
     TACF_PASSWORD=tmax 
     TACF_DATABASE=oframe 
     TACF_TABLESPACE=TACF00 
     TACF_TABLE_CREATE=YES
     ```

4. 完成 TACF 安裝程式之後，套用 TACF 環境變數。 在命令提示字元中，輸入：

     ```
     source \~/.bash\_profile
     ```

5. 執行 TACF 安裝程式。 在命令提示字元中，輸入：

     ```
     ./OpenFrame_Tacf7_0_Fix2_Linux_x86_64.bin -f tacf.properties
     ```

     輸出看起來會像這樣：

     ```
     Wed Dec 07 17:36:42 EDT 2016
     Free Memory: 18703 kB 
     Total Memory: 28800 kB

     4 Command Line Args: 
     0:  -f 1:  tacf.properties 
     2:  -m 
     3:  SILENT 
     java.class.path: 
     /tmp/install.dir.41422/InstallerData 
     /tmp/install.dir.41422/InstallerData/installer.zip 
     ZGUtil.CLASS_PATH: 
     /tmp/install.dir.41422/InstallerData 
     tmp/install.dir.41422/InstallerData/installer.zip 
     sun.boot.class.path: 
     /tmp/install.dir.41422/Linux/resource/jre/lib/resources.jar /tmp/install.dir.41422/Linux/resource/jre/lib/rt.jar /tmp/install.dir.41422/Linux/resource/jre/lib/sunrsasign.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jsse.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jce.jar /tmp/install.dir.41422/Linux/resource/jre/lib/charsets.jar /tmp/install.dir.41422/Linux/resource/jre/lib/jfr.jar /tmp/install.dir.41422/Linux/resource/jre/classes
     ```

6. 在命令提示字元中，輸入 `tmboot` 以重新開機 OpenFrame。 輸出看起來會像這樣：

     ```
     TMBOOT for node(NODE1) is starting: 
     Welcome to Tmax demo system: it will expire 2016/11/4 
     Today: 2016/9/7 
     TMBOOT: TMM is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: CLL is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: CLH is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: TLM(tlm) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrsasvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrlhsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrdmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrdsedt) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrcmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofruisvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrsmlog) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(vtammgr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjschd) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjinit) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjhist) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmjspbk) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(ofrpmsvr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(obmtsmgr) is starting: Wed Sep  7 17:48:53 2016 
     TMBOOT: SVR(tmsvr) is starting: Wed Sep  7 17:48:53 2016
     ```

7. 使用 `si` 命令中的 `tmadmin`，確認進程狀態已就緒。 例如︰

     ```
     [oframe7\@ofdemo \~]\$ tmadmin
     ```

     在 [**狀態**] 欄中，RDY 隨即出現：

    ![狀態資料行中的 RDY](media/tmboot-02.png)

8. 執行下列命令：

     ```
     $$2 NODE1 (tmadm): quit 
     DM quit for node (NODE1)

     [oframe7@ofdemo ~]$ tacfmgr 
     Input USERNAME  : ROOT 
     Input PASSWORD  : SYS1

     TACFMGR: TACF MANAGER START!!!
     QUIT TACFMGR: TACF MANAGER END!!!

     [oframe7@ofdemo ~]$ tmdow
     ```

9. 使用 `tmdown` 命令關閉伺服器。 輸出看起來會像這樣：

     ```
     [oframe7@ofdemo ~]$ tmdown 
     Do you really want to down whole Tmax? (y : n): y

     TMDOWN for node(NODE1) is starting: 
     TMDOWN: SERVER(ofrlhsvr:37) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrdsedt:39) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjschd:54) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:47) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:48) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrdmsvr:38) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjmsvr:50) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjhist:56) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrsasvr:36) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrcmsvr:40) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmjspbk:57) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(tmsvr:60) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(ofrpmsvr:58) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: SERVER(obmtsmgr:59) downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: CLL downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: CLH downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TLM downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TMM downed: Wed Sep  7 17:50:50 2016 
     TMDOWN: TMAX is down
     ```

## <a name="install-prosort"></a>安裝 ProSort

ProSort 是用於排序資料的批次交易中使用的公用程式。

**若要安裝 ProSort**

1. 請確定批次安裝成功，然後確認**prosort-bin-prosort\_2sp3-linux64-2123-opt。 gz** installer 檔案是否存在。

2. 使用屬性檔案執行安裝程式。 在命令提示字元中，輸入：

     ```
     tar -zxvf prosort-bin-prosort\_2sp3-linux64-2123-opt.tar.gz
     ```

3. 將 prosort 目錄移至首頁位置。 在命令提示字元中，輸入：

     ```
     mv prosort /opt/tmaxapp/prosort
     ```

4. 建立授權子目錄，並在該處複製授權檔案。 例如︰

     ```
     cd /opt/tmaxapp/prosort 
     mkdir license 
     cp /opt/tmaxsw/oflicense/prosort/license.xml /opt/tmaxapp/prosort/license
     ```

5. 在 vi （`vi .bash_profile`）中開啟 bash. profile，並如下所示更新：

     ```bash
     #       PROSORT

     PROSORT_HOME=/opt/tmaxapp/prosort 
     PROSORT_SID=gbg 
     PATH=$PATH:$PROSORT_HOME/bin LD_LIBRARY_PATH=$PROSORT_HOME/lib:$LD_LIBRARY_PATH LIBPATH$PROSORT_HOME/lib:$LIBPATH 
     export PROSORT_HOME PROSORT_SID 
     PATH LD_LIBRARY_PATH LIBPATH 
     PATH=$PATH:$OPENFRAME_HOME/shbin 
     export PATH
     ```

6. 若要執行 bash 設定檔，請在命令提示字元中輸入： `. .bash_profile`

7. 建立設定檔。 例如︰

     ```
     oframe@oframe7: cd /opt/tmaxapp/prosort/config 
     oframe@oframe7: ./gen_tip.sh 
     Using PROSORT_SID "gbg"
      /home/oframe7/prosort/config/gbg.tip generated
     ```

8. 建立符號連結。 例如︰

     ```
     oframe@oframe7: cd /opt/tmaxapp/OpenFrame/util/ 
     oframe@oframe7home/oframe7/OpenFrame/util :  ln -s DFSORT SORT
     ```

9. 執行 `prosort -h` 命令來確認 ProSort 安裝。 例如︰

     ```
     oframe@oframe7: prosort -h

     Usage: prosort [options] [sort script files]
     options ------
     -h             Display this information 
     -v             Display version information 
     -s             Display state information 
     -j             Display profile information 
     -x             Use SyncSort compatible mode
     ```

## <a name="install-ofcobol"></a>安裝 OFCOBOL

OFCOBOL 是用來解讀大型主機 COBOL 程式的 OpenFrame 編譯器。 

**若要安裝 OFCOBOL**

1. 請確定批次/線上安裝成功，然後確認 OpenFrame\_COBOL3\_0\_40\_Linux\_x86\_64. bin installer 檔案存在。

2. 若要執行 OFCOBOL 安裝程式，請在命令提示字元中輸入：

     ```
      ./OpenFrame\_COBOL3\_0\_40\_Linux\_x86\_64.bin
     ```

3. 閱讀授權合約，然後按 Enter 繼續。

4. 接受授權合約。 安裝完成時，會出現下列情況：

     ```
     Choose Install Folder 
     --------------------
     Where would you like to install?
     Default Install Folder: /home/oframe7/OFCOBOL

     ENTER AN ABSOLUTE PATH, OR PRESS <ENTER> TO ACCEPT THE DEFAULT : /opt/tmaxapp/OFCOBOL

     INSTALL FOLDER IS: /opt/tmaxapp/OFCOBOL 
     IS THIS CORRECT? (Y/N): Y[oframe7@ofdemo ~]$ vi .bash_profile

     ============================================================================ Installing... 
     ------------
     [==================|==================|==================|==================] 
     [------------------|------------------|------------------|------------------]

     =============================================================================== Installation Complete 
     --------------------
     Congratulations. OpenFrame_COBOL has been successfully installed
     PRESS <ENTER> TO EXIT THE INSTALLER
     ```

5. 在 vi （`vi .bash_profile`）中開啟 bash 設定檔，並確認已使用 OFCOBOL 變數進行更新。
6. 執行 bash 設定檔。 在命令提示字元中，輸入：

     ```
      source ~/.bash_profile
     ```

7. 將 OFCOBOL 授權複製到已安裝的資料夾。 例如︰
     ```
     mv licofcob.dat $OFCOB_HOME/license
     ```
8. 移至 OpenFrame tjclrun. 會議設定檔，並在 vi 中開啟它。 例如︰
     ```
     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/config 
     [oframe7@ofdemo ~]$ vi tjclrun.conf
     ```

   以下是變更前的 SYSLIB 區段：
     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bin LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${COBDIR}/lib:/ usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib
     ```
   以下是變更之後的 SYSLIB 區段：
     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bin LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${COBDIR}/lib:/ usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib :${ODBC_HOME}/lib 
     :${OFCOB_HOME}/lib
     ```
9. 查看 OpenFrame\_COBOL\_中的 .Installlog 記錄檔，並確認沒有任何錯誤。 例如︰
     ```
     [oframe7@ofdemo ~]$ vi $OFCOB_HOME/UninstallerData/log/OpenFrame_COBOL_InstallLog.log 
     …….. 
     Summary 
     ------
     Installation: Successful. 
     131 Successes 
     0 Warnings 
     0 NonFatalErrors 
     0 FatalError
     ```
10. 使用 `ofcob --version` 命令並檢查版本號碼，以確認安裝。 例如︰

     ```
     [oframe7@ofdemo ~]$ ofcob --version 
     OpenFrame COBOL Compiler 3.0.54 
     CommitTag:: 645f3f6bf7fbe1c366a6557c55b96c48454f4bf
     ```

11. 使用 `tmdown/tmboot` 命令重新開機 OpenFrame。

## <a name="install-ofasm"></a>安裝 OFASM

OFASM 是用來解讀大型主機組合程式的 OpenFrame 編譯器。

**若要安裝 OFASM**

1. 請確定批次/線上安裝成功，然後確認 OpenFrame\_ASM3\_0\_Linux\_x86\_安裝程式檔案是否存在。

2. 執行安裝程式。 例如︰

     ```
     [oframe7@ofdemo ~]$ ./OpenFrame_ASM3_0_Linux_x86_64.bin
     ```

3. 閱讀授權合約，然後按 Enter 繼續。
4. 接受授權合約。
5. 確認 bash 設定檔已更新為 OFASM 變數。 例如︰

     ```
     [oframe7@ofdemo ~]$ source .bash_profile
     [oframe7@ofdemo ~]$ ofasm --version 
     # TmaxSoft OpenFrameAssembler v3 r328 
     (3ff35168d34f6e2046b96415bbe374160fcb3a34)

     [oframe7@ofdemo OFASM]$ vi .bash_profile

     # OFASM ENV 
     export OFASM_HOME=/opt/tmaxapp/OFASM 
     export OFASM_MACLIB=$OFASM_HOME/maclib/free_macro 
     export PATH="${PATH}:$OFASM_HOME/bin:" 
     export LD_LIBRARY_PATH="./:$OFASM_HOME/lib:$LD_LIBRARY_PATH"
     ```

6. 在 vi 中開啟 OpenFrame tjclrun 設定檔，然後編輯它，如下所示：

     ```
     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/config 
     [oframe7@ofdemo ~]$ vi tjclrun.conf
     ```

     以下是變更*前*的 [SYSLIB] 區段：

     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bi n:${OPENFRAME_HOME}/volume_default/SYS1.LOADLIB LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${CO BDIR}/lib:/usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib:${OFCOB_HOM E}/lib:${ODBC_HOME}/lib:${OFPLI_HOME}/lib
     ```

     以下是變更*之後*的 [SYSLIB] 區段：

     ```
     [SYSLIB] BIN_PATH=${OPENFRAME_HOME}/bin:${OPENFRAME_HOME}/util:${COBDIR}/bin:/usr/local/bin:/bi n:${OPENFRAME_HOME}/volume_default/SYS1.LOADLIB LIB_PATH=${OPENFRAME_HOME}/lib:${OPENFRAME_HOME}/core/lib:${TB_HOME}/client/lib:${CO BDIR}/lib:/usr/lib:/lib:/lib/i686:/usr/local/lib:${PROSORT_HOME}/lib:/opt/FSUNbsort/lib:${OFCOB_HOM E}/lib:${ODBC_HOME}/lib:${OFPLI_HOME}/lib:${OFASM_HOME}/lib
     ```

7. 在 vi 中開啟 OpenFrame\_ASM\_.Installlog，並確認沒有任何錯誤。 例如︰

     ```
     [oframe7@ofdemo ~]$ vi 
     $OFASM_HOME/UninstallerData/log/OpenFrame_ASM_InstallLog.log 
     …….. 
     Summary 
     ------

     Installation: Successful.

     55 Successes 
     0 Warnings 
     0 NonFatalErrors 
     0 FatalErrors
     ```

8. 發出下列其中一個命令來重新開機 OpenFrame：

     ```
     tmdown / tmboot
     ```

     或

     ```
     oscdown / oscboot
     ```

## <a name="install-osc"></a>安裝 .OSC

.OSC 是類似于 IBM CICS 的 OpenFrame 環境，其支援高速 OLTP 交易和其他管理功能。

**安裝 .OSC**

1. 請確定基底安裝成功，然後確認 OpenFrame\_OSC7\_0\_Fix2\_Linux\_x86\_64. bin installer 檔案和 .osc. properties 設定檔案。
2. 編輯 .osc. properties 檔案中的下列參數：
     ```
     OPENFRAME_HOME=/opt/tmaxapp/OpenFrame OSC_SYS_OSC_NCS_PATH=/opt/tmaxapp/OpenFrame/temp/OSC_NCS OSC_APP_OSC_TC_PATH=/opt/tmaxapp/OpenFrame/temp/OSC_TC
     ```

3. 使用屬性檔案執行安裝程式，如下所示：

     ```
     [oframe7@ofdemo ~]$ chmod a+x OpenFrame_OSC7_0_Fix2_Linux_x86_64.bin [oframe7@ofdemo ~]$ ./OpenFrame_OSC7_0_Fix2_Linux_x86_64.bin -f osc.properties
     ```

     完成時，會顯示 [安裝完成] 訊息。

4. 確認 bash 設定檔已更新為使用的 .OSC 變數。
5. 請參閱 OpenFrame\_OSC7\_0\_Fix2\_.Installlog .log 檔案。 您應該會看到類似下面的畫面：

     ```
     Summary 
     ------ 
     Installation: Successful.

     233 Successes
     0 Warnings
     0 NonFatalErrors
     0 FatalError
     ```

6. 使用 vi 開啟 ofsys 的設定檔。 例如︰

     ```
     vi $OPENFRAME_HOME/config/ofsys.seq
     ```

7. 在 [\#基底] 和 [\#批次] 區段中，編輯參數，如下所示。

     ```
     Before changes
     #BASE
     ofrsasvr
     ofrlhsvr
     ofrdmsvr
     ofrdsedt
     ofrcmsvr
     ofruisvr
     ofrsmlog
     vtammgr
     TPFMAGENT

     #BATCH 
     #BATCH#obmtsmgr
     #BATCH#ofrpmsvr
     #BATCH#obmjmsvr
     #BATCH#obmjschd
     #BATCH#obmjinit
     #BATCH#obmjhist
     #BATCH#obmjspbk
     #TACF #TACF#tmsvr

     After changes  #BATCH
     #BASE          obmtsmgr 
     ofrsasvr       ofrpmsvr
     ofrlhsvr       obmjmsvr
     ofrdmsvr       obmjschd
     ofrdsedt       obmjinit
     ofrcmsvr       obmjhist
     ofruisvr       obmjspbk
     ofrsmlog
     vtammgr        #TACF
     TPFMAGENT      tmsvr
    ```

8. 複製授權檔案。 例如︰

     ```
     [oframe7@ofdemo ~]$ cp /home/oframe7/oflicense/ofonline/licosc.dat 

     $OPENFRAME_HOME/license

     [oframe7@ofdemo ~]$ cd $OPENFRAME_HOME/license 
     oframe@oframe7/OpenFrame/license / ls -l 
     -rwxr-xr-x. 1 oframe mqm 80 Sep 12 01:37 licosc.dat 
     -rwxr-xr-x. 1 oframe mqm 80 Sep  8 09:40 lictacf.dat 
     -rwxrwxr-x. 1 oframe mqm 80 Sep  3 11:54 lictjes.da
     ```

9. 若要啟動和關閉 .OSC，請在命令提示字元中輸入 `osctdlinit OSCOIVP1`，以初始化 CICS 區域共用記憶體。

10. 執行 `oscboot` 來啟動 .OSC。 輸出看起來會像這樣：

     ```
     OSCBOOT : pre-processing       [ OK ]

     TMBOOT for node(NODE1) is starting: 
     Welcome to Tmax demo system: it will expire 2016/11/4 
     Today: 2016/9/12 
          TMBOOT: TMM is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: CLL is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: CLH is starting: Mon Sep 12 01:40:25 2016 
          TMBOOT: TLM(tlm) is starting: Mon Sep 12 01:40:25 2016 
     ```

11. 若要確認進程狀態為 [就緒]，請使用 si 中的 `tmadmin` 命令。 所有的進程應該會在 [**狀態**] 資料行中顯示 RDY。

    ![顯示 RDY 的進程](media/tmadmin-02.png)

12. 使用 `oscdown` 命令來關閉 [.OSC]。

## <a name="install-jeus"></a>安裝 JEUS

JEUS （JAVA 企業使用者解決方案）提供 OpenFrame web 應用程式伺服器的展示層。

安裝 JEUS 之前，請先安裝 Apache Ant 套件，其中提供安裝 JEUS 所需的程式庫和命令列工具。

**若要安裝 Apache Ant**

1. 使用 `wget` 命令下載 Ant 二進位檔。 例如︰

     ```
     wget http://apache.mirror.cdnetworks.com/ant/binaries/apacheant-1.9.7-bin.tar.gz
     ```

2. 使用 `tar` 公用程式來解壓縮二進位檔案，並將它移到適當的位置。 例如︰

     ```
     tar -xvzf apache-ant-1.9.7-bin.tar.gz
     ```

3. 為求效率，請建立符號連結：

     ```
     ln -s apache-ant-1.9.7 ant
     ```

4. 在 vi （`vi .bash_profile`）中開啟 bash 設定檔，並使用下列變數加以更新：

     ```
     # Ant ENV
     export ANT_HOME=$HOME/ant 
     export PATH=$HOME/ant/bin:$PATH
     ```

5.  套用修改過的環境變數。 例如︰

     ```
     [oframe7\@ofdemo \~]\$ source \~/.bash\_profile
     ```

**若要安裝 JEUS**

1. 使用 `tar` 公用程式來展開安裝程式。 例如︰

     ```
     [oframe7@ofdemo ~]$ tar -zxvf jeus704.tar.gz
     ```

2. 建立**jeus**資料夾（`mkdir jeus7`），並將二進位檔解壓縮。
3. 變更為**安裝**目錄（或針對您自己的環境使用 JEUS 參數）。 例如︰

     ```
     [oframe7@ofdemo ~]$ cd jeus7/setup/
     ```

4. 執行組建之前，請先執行 `ant clean-all`。 輸出看起來會像這樣：

     ```
     Buildfile: /home/oframe7jeus7/setup/build.xml

     clean-bin:
     delete-domain:
     [echo] Deleting a domain configuration: domain = jeus_domain
     delete-nodesxml:
     clean-config:
     clean-all:
     BUILD SUCCESSFUL
     Total time: 0 seconds
     ```

5.  建立網域-config-範本. properties 檔案的備份。 例如︰

     ```
     [oframe7@ofdemo ~]$ cp domain-config-template.properties domain-configtemplate.properties.bkp
     ```

6. 在 vi 中開啟網域-config-範本. properties 檔案：

     ```
     [oframe7\@ofdemo setup]\$ vi domain-config-template.properties
     ```

7. 將 `jeus.password=jeusadmin nodename=Tmaxsoft` 變更為 `jeus.password=tmax1234 nodename=ofdemo`

8. 執行 `ant install` 命令來建立 JEUS。
9.  使用 JEUS 變數來更新 bash\_設定檔檔案，如下所示：

     ```
     # JEUS ENV 
     export JEUS_HOME=/opt/tmaxui/jeus7 PATH="/opt/tmaxui/jeus7/bin:/opt/tmaxui/jeus7/lib/system:/opt/tmaxui/jeus7/webserver/bin:$ {PATH}" 
     export PATH
     ```

10. 執行 bash 設定檔。 例如︰

     ```
     [oframe7@ofdemo setup]$ . .bash_profile
     ```

11. *選擇性*。 建立別名以輕鬆關閉和啟動 JEUS 元件：

     ```     
     # JEUS alias

     alias dsboot='startDomainAdminServer -domain jeus_domain -u administrator -p jeusadmin'
     alias msboot='startManagedServer -domain jeus_domain -server server1 -u administrator -p jeusadmin' 
     alias msdown=‘jeusadmin -u administrator -p tmax1234 "stop-server server1“’ 
     alias dsdown=‘jeusadmin -domain jeus_domain -u administrator -p tmax1234 "local-shutdown“’
     ```

12. 若要確認安裝，請啟動網域系統管理員伺服器，如下所示：

     ```
     [oframe7@ofdemo ~]$ startDomainAdminServer -domain jeus_domain -u administrator -p jeusadmin
     ```

13. 使用下列語法來驗證 web 登入：

     ```
     http://<IP>:<port>/webadmin/login
     ```

     例如，<http://192.168.92.133:9736/webadmin/login.> 登入畫面會隨即出現：
    
     ![JEUS WebAdmin 登入畫面](media/jeus-01.png)

     > [!NOTE]
     > 如果您遇到埠安全性的任何問題，請開啟埠9736或停用防火牆（`systemctl stop firewall`）。

14. 若要變更 server1 的主機名稱，請按一下 [**鎖定] & [編輯**]，然後按一下 [ **server1**]。 在 [伺服器] 視窗中，變更主機名稱，如下所示：

    1.  將**Nodename**變更為**ofdemo**。
    2.  按一下視窗右側的 **[確定]** 。
    3.  按一下視窗左下方的 [套用**變更**]，並針對 [描述] 輸入*主機名稱變更*。

    ![JEUS WebAdmin 畫面](media/jeus-02.png)

15. 確認確認畫面中的設定是否成功。

    ![jeus_domain 伺服器畫面](media/jeus-03.png)

16. 使用下列命令啟動受管理伺服器進程 "server1"：

     ```
     [oframe7@ofdemo ~]$ startManagedServer -domain jeus_domain -server server1 -u administrator -p jeusadmin
     ```

## <a name="install-ofgw"></a>安裝 OFGW

OFGW 是 OpenFrame 閘道，支援3270終端機模擬器和 OSI 基底之間的通訊，並管理終端機模擬器和 OSI 之間的會話。

**若要安裝 OFGW**

1. 請確定已成功安裝 JEUS，然後確認 OFGW7\_0\_1\_一般的 bin 安裝程式檔案是否存在。
2. 執行安裝程式。 例如︰

     ```
     [oframe7@ofdemo ~]$ ./OFGW7_0_1_Generic.bin
     ````

3. 針對對應的提示，請使用下列位置：
     -   JEUS 主目錄
     -   JEUS 功能變數名稱
     -   JEUS 伺服器名稱
     -   Tibero 驅動程式
     -   Tmax 節點識別碼 ofdemo

4. 接受其餘的預設值，然後按 Enter 結束安裝程式。

5. 確認 OFGW 的 URL 如預期般運作：

     ```
     Type URL 
     http://192.168.92.133:8088/webterminal/ and press enter
      < IP >               :8088/webterminal/
     ```

     下列畫面隨即出現：

    ![OpenFrame WebTerminal](media/ofgw-01.png)

## <a name="install-ofmanager"></a>安裝 OFManager

OFManager 提供在 web 環境中 OpenFrame 的作業和管理功能。

**若要安裝 OFManager**

1. 確認 OFManager7\_一般的 bin 安裝程式檔案存在。
2. 執行安裝程式。 例如︰

     ```
     OFManager7_Generic.bin
     ```

3.  按 Enter 鍵以繼續，然後接受授權合約。
4.  選擇 [安裝] 資料夾。
5.  接受預設值。
6.  選擇 [Tibero] 作為資料庫。
7.  按 Enter 鍵以結束安裝程式。
8.  確認 OFManager 的 URL 如預期般運作：

     ```
     Type URL http://192.168.92.133:8088/ofmanager and press enter <  IP >  : < PORT >  ofmanager Enter ID:   ROOT 
     Password: SYS1
     ```

[開始] 畫面隨即出現：

![Tmax OpenFrame 管理員登入畫面](media/ofmanager-01.png)

這會完成 OpenFrame 元件的安裝。

## <a name="next-steps"></a>後續步驟

如果您考慮進行大型主機遷移，我們可以使用擴充的合作夥伴生態系統來協助您。 如需選擇合作夥伴解決方案的詳細指引，請參閱 [Platform Modernization Alliance](https://datamigration.microsoft.com/)。

-   [開始使用 Azure](https://docs.microsoft.com/azure/)
-   [Host Integration Server (HIS) 文件](https://docs.microsoft.com/host-integration-server/)
-   [Azure 虛擬資料中心隨即轉移指南](https://blogs.msdn.microsoft.com/azurecat/2018/03/12/new-whitepaper-azure-virtual-datacenter-lift-and-shift-guide/)
