---
title: 為 Windows 虛擬桌面使用者自訂摘要-Azure
description: 如何使用 PowerShell Cmdlet 為 Windows 虛擬桌面使用者自訂摘要。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 08/29/2019
ms.author: helohr
ms.openlocfilehash: 32abddad600edf0f62efef48aea402b6d4f09c10
ms.sourcegitcommit: f97f086936f2c53f439e12ccace066fca53e8dc3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/15/2020
ms.locfileid: "77368869"
---
# <a name="customize-feed-for-windows-virtual-desktop-users"></a>適用於 Windows 虛擬桌面使用者的自訂摘要

您可以自訂摘要，讓 RemoteApp 和遠端桌面資源以可辨識的方式顯示給使用者。

首先，[下載並匯入 Windows 虛擬桌面的 PowerShell 模組](/powershell/windows-virtual-desktop/overview/)，以在您的 PowerShell 工作階段中使用 (如果您還沒這麼做的話)。 之後，請執行下列 Cmdlet 來登入您的帳戶：

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="customize-the-display-name-for-a-remoteapp"></a>自訂 RemoteApp 的顯示名稱

您可以藉由設定易記名稱來變更已發佈 RemoteApp 的顯示名稱。 根據預設，易記名稱與 RemoteApp 程式的名稱相同。

若要取得應用程式群組的已發佈 Remoteapp 清單，請執行下列 PowerShell Cmdlet：

```powershell
Get-RdsRemoteApp -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname>
```
![已反白顯示 [名稱] 和 [FriendlyName] 的 PowerShell Cmdlet RDSRemoteApp 螢幕擷取畫面。](media/get-rdsremoteapp.png)

若要為 RemoteApp 指派易記名稱，請執行下列 PowerShell Cmdlet：

```powershell
Set-RdsRemoteApp -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname> -Name <existingappname> -FriendlyName <newfriendlyname>
```
![已反白顯示 [名稱] 和 [新增 FriendlyName] 的 PowerShell Cmdlet 集 RDSRemoteApp 螢幕擷取畫面。](media/set-rdsremoteapp.png)

## <a name="customize-the-display-name-for-a-remote-desktop"></a>自訂遠端桌面的顯示名稱

您可以藉由設定易記名稱來變更已發佈之遠端桌面的顯示名稱。 如果您透過 PowerShell 手動建立主機集區和桌面應用程式群組，則預設的易記名稱為「會話桌面」。 如果您透過 GitHub Azure Resource Manager 範本或 Azure Marketplace 供應專案建立了主機集區和桌面應用程式群組，則預設的易記名稱會與主機集區名稱相同。

若要取出遠端桌面資源，請執行下列 PowerShell Cmdlet：

```powershell
Get-RdsRemoteDesktop -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname>
```
![已反白顯示 [名稱] 和 [FriendlyName] 的 PowerShell Cmdlet RDSRemoteApp 螢幕擷取畫面。](media/get-rdsremotedesktop.png)

若要為遠端桌面資源指派易記名稱，請執行下列 PowerShell Cmdlet：

```powershell
Set-RdsRemoteDesktop -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname> -FriendlyName <newfriendlyname>
```
![已反白顯示 [名稱] 和 [新增 FriendlyName] 的 PowerShell Cmdlet 集 RDSRemoteApp 螢幕擷取畫面。](media/set-rdsremotedesktop.png)

## <a name="next-steps"></a>後續步驟

既然您已自訂使用者的摘要，您可以登入 Windows 虛擬桌面的用戶端來進行測試。若要這麼做，請繼續進行連線至 Windows 虛擬桌面的作法：
    
 * [從 Windows 10 或 Windows 7 連線](connect-windows-7-and-10.md)
 * [從網頁瀏覽器連線](connect-web.md) 
