---
title: 啟用或停用 Azure VM 中客體 OS 上的防火牆規則 | Microsoft Docs
description: ''
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
ms.date: 11/22/2018
ms.author: delhan
ms.openlocfilehash: 782240c51833fc841af9f4260860db4c03897c03
ms.sourcegitcommit: c79aa93d87d4db04ecc4e3eb68a75b349448cd17
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2019
ms.locfileid: "71086441"
---
# <a name="enable-or-disable-a-firewall-rule-on-an-azure-vm-guest-os"></a>啟用或停用 Azure VM 的客體作業系統上的防火牆規則

本文針對您懷疑客體作業系統防火牆篩選虛擬機器 (VM) 之部分流量的情況，提供進行疑難排解的參考資訊。 這可能很有用，原因如下：

*   如果對造成 RDP 連線失敗的防火牆刻意進行變更，則使用自訂指令碼擴充功能可以解決此問題。

*   與設定 RDP 專用的防火牆規則相比，停用所有防火牆設定檔是更加安全的方法。

## <a name="solution"></a>方案

設定防火牆規則的方式取決於虛擬機器所需的存取權層級而定。 下列範例使用 RDP 規則。 不過，藉由指向正確的登錄機碼，可以將相同的方法套用至任何其他類型的流量。

### <a name="online-troubleshooting"></a>線上疑難排解 

#### <a name="mitigation-1-custom-script-extension"></a>降低風險 1：自訂指令碼延伸模組

1.  下列範例建立指令碼。

    *   若要啟用規則：
        ```cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=yes
        ```

    *   若要停用規則：
        ```cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=no
        ```

2.  在 Azure 入口網站中使用[自訂指令碼擴充功能](../extensions/custom-script-windows.md)，上傳此指令碼。 

#### <a name="mitigation-2-remote-powershell"></a>降低風險 2：遠端 Powershell

如果虛擬機器為連線狀態，而且可以在相同的虛擬網路上存取另一個虛擬機器，您可以使用其他虛擬機器來進行後續的緩解措施。

1.  在疑難排解的虛擬機器上，開啟 PowerShell 主控台視窗。

2.  視需要執行下列命令。

    *   若要啟用規則：
        ```powershell
        Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
        Enable-NetFirewallRule -DisplayName  "RemoteDesktop-UserMode-In-TCP"
        exit
        ```

    *   若要停用規則：
        ```powershell
        Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
        Disable-NetFirewallRule -DisplayName  "RemoteDesktop-UserMode-In-TCP"
        exit
        ```

#### <a name="mitigation-3-pstools-commands"></a>降低風險 3：PSTools 命令

如果虛擬機器為連線狀態，而且可以在相同的虛擬網路上存取另一個虛擬機器，您可以使用其他虛擬機器來進行後續的緩解措施。

1.  在疑難排解虛擬機器上，下載 [PSTools](https://docs.microsoft.com/sysinternals/downloads/pstools)。

2.  開啟 CMD 執行個體，並透過其內部 IP (DIP) 存取虛擬機器。 

    * 若要啟用規則：
        ```cmd
        psexec \\<DIP> -u <username> cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=yes
        ```

    *   若要停用規則：
        ```cmd
        psexec \\<DIP> -u <username> cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=no
        ```

#### <a name="mitigation-4-remote-registry"></a>降低風險 4：遠端登錄

如果虛擬機器為連線狀態，而且可以在相同的虛擬網路上存取另一個虛擬機器，您可以在其他虛擬機器上使用[遠端登錄](https://support.microsoft.com/help/314837/how-to-manage-remote-access-to-the-registry)。

1.  在疑難排解虛擬機器上，啟動登錄編輯程式 (regedit.exe)，然後選取 [檔案] > [連線網路登錄]。

2.  開啟*目的機器*\SYSTEM 分支，然後指定下列值：

    * 若要啟用規則，請開啟下列登錄值：
    
        *TARGET MACHINE*\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP
    
        然後，在字串中，將 **Active=FALSE** 變更為 **Active=TRUE**：

        **v 2.22 |動作 = 允許 |Active = TRUE |Dir = In |通訊協定 = 6 |設定檔 = 網域 |設定檔 = 私用 |設定檔 = 公用 |LPort = 3389 |應用程式 =%SystemRoot%\system32\svchost.exe |Svc = termservice |名稱 =\@FirewallAPI，-28775 |Desc =\@FirewallAPI，-28756 |EmbedCtxt =\@FirewallAPI，-28752 |**
    
    * 若要停用規則，請開啟下列登錄值：
    
        *TARGET MACHINE*\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP

        然後，將 **Active =TRUE** 變更為 **Active=FALSE**：
        
        **v 2.22 |動作 = 允許 |Active = FALSE |Dir = In |通訊協定 = 6 |設定檔 = 網域 |設定檔 = 私用 |設定檔 = 公用 |LPort = 3389 |應用程式 =%SystemRoot%\system32\svchost.exe |Svc = termservice |名稱 =\@FirewallAPI，-28775 |Desc =\@FirewallAPI，-28756 |EmbedCtxt =\@FirewallAPI，-28752 |**

3.  重新啟動虛擬機器以套用變更。

### <a name="offline-troubleshooting"></a>離線疑難排解 

如果任何方法皆無法存取虛擬機器，使用自訂指令碼擴充將會失敗，而且您必須直接透過系統磁碟，在離線模式下工作。

在按照下列步驟進行之前，請先擷取受影響虛擬機器系統磁碟的快照集作為備份。 如需詳細資訊，請參閱[擷取磁碟快照集](../windows/snapshot-copy-managed-disk.md)。

1.  [將系統磁碟連結至復原 VM](troubleshoot-recovery-disks-portal-windows.md)。

2.  啟動復原 VM 的遠端桌面連線。

3.  確定該磁碟在磁碟管理主控台中標示為 [線上]。 記下指派給已連結系統磁碟的磁碟機代號。

4.  在進行任何變更之前，請先建立 \windows\system32\config 資料夾的複本，以便在需要回復變更時使用。

5.  在疑難排解 VM 上，啟動登錄編輯程式（regedit.exe）。

6.  反白顯示**HKEY_LOCAL_MACHINE**機碼，然後從功能表**中選取** > [檔案] [**載入 Hive** ]。

    ![Regedit](./media/enable-or-disable-firewall-rule-guest-os/load-registry-hive.png)

7.  找出，然後開啟 \windows\system32\config\SYSTEM 檔案。 

    > [!Note]
    > 系統會提示您輸入名稱。 輸入**BROKENSYSTEM**，然後展開**HKEY_LOCAL_MACHINE**。 您現在會看到名為**BROKENSYSTEM**的其他索引鍵。 在此疑難排解中，我們會將這些問題 hive 裝載為**BROKENSYSTEM**。

8.  在 BROKENSYSTEM 分支上進行下列變更：

    1.  檢查虛擬機器從哪個 **ControlSet** 登錄機碼開始。 您會在 HKLM\BROKENSYSTEM\Select\Current. 中看到其金鑰編號

    2.  若要啟用規則，請開啟下列登錄值：
    
        HKLM\BROKENSYSTEM\ControlSet00X\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP
        
        然後，將 **Active=FALSE** 變更為 **Active=True**。
        
        **v 2.22 |動作 = 允許 |Active = TRUE |Dir = In |通訊協定 = 6 |設定檔 = 網域 |設定檔 = 私用 |設定檔 = 公用 |LPort = 3389 |應用程式 =%SystemRoot%\system32\svchost.exe |Svc = termservice |名稱 =\@FirewallAPI，-28775 |Desc =\@FirewallAPI，-28756 |EmbedCtxt =\@FirewallAPI，-28752 |**

    3.  若要停用規則，請開啟下列登錄機碼：

        HKLM\BROKENSYSTEM\ControlSet00X\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP

        然後，將 **Active=True** 變更為 **Active=FALSE**。
        
        **v 2.22 |動作 = 允許 |Active = FALSE |Dir = In |通訊協定 = 6 |設定檔 = 網域 |設定檔 = 私用 |設定檔 = 公用 |LPort = 3389 |應用程式 =%SystemRoot%\system32\svchost.exe |Svc = termservice |名稱 =\@FirewallAPI，-28775 |Desc =\@FirewallAPI，-28756 |EmbedCtxt =\@FirewallAPI，-28752 |**

9.  反白顯示**BROKENSYSTEM**，然後從功能表**中選取** > [檔案] **[卸載 Hive** ]。

10. [中斷連結系統磁碟並重新建立 VM](troubleshoot-recovery-disks-portal-windows.md)。

11. 檢查問題是否已解決。
