---
title: 使用新增 Azure 資料總管的資料庫主體C#
description: 在本文中，您將瞭解如何使用C#來新增 Azure 資料總管的資料庫主體。
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 02/03/2020
ms.openlocfilehash: 797d1253d44739f2026563e3df72bc85a8ef382e
ms.sourcegitcommit: 42517355cc32890b1686de996c7913c98634e348
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/02/2020
ms.locfileid: "76965030"
---
# <a name="add-database-principals-for-azure-data-explorer-by-using-c"></a>使用新增 Azure 資料總管的資料庫主體C#

> [!div class="op_single_selector"]
> * [C#](database-principal-csharp.md)
> * [Python](database-principal-python.md)
> * [Azure Resource Manager 範本](database-principal-resource-manager.md)

Azure 資料總管是一項快速又可高度調整的資料探索服務，可用於處理記錄和遙測資料。 在本文中，您會使用C#來新增 Azure 資料總管的資料庫主體。

## <a name="prerequisites"></a>必要條件

* 如果您尚未安裝 Visual Studio 2019，您可以下載並使用**免費**的[Visual Studio 2019 的社區版本](https://www.visualstudio.com/downloads/)。 務必在 Visual Studio 設定期間啟用 **Azure 開發**。
* 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費 Azure 帳戶](https://azure.microsoft.com/free/)。
* [建立叢集和資料庫](create-cluster-database-csharp.md)。

## <a name="install-c-nuget"></a>安裝C# NuGet

* 請安裝[kusto](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/)。
* 安裝[ClientRuntime](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication)以進行驗證。

[!INCLUDE [data-explorer-authentication](../../includes/data-explorer-authentication.md)]

## <a name="add-a-database-principal"></a>新增資料庫主體

下列範例會示範如何以程式設計方式加入資料庫主體。

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";

var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, clientSecret);
var kustoManagementClient = new KustoManagementClient(serviceCreds)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster and database that are created as part of the Prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
string principalAssignmentName = "databasePrincipalAssignment1";
string principalId = "xxxxxxxx";//User email, application ID, or security group name
string role = "Admin";//Admin, Ingestor, Monitor, User, UnrestrictedViewers, Viewer
string tenantIdForPrincipal = tenantId;
string principalType = "App";//User, App, or Group

var databasePrincipalAssignment = new DatabasePrincipalAssignment(principalId, role, principalType, tenantId: tenantIdForPrincipal);
await kustoManagementClient.DatabasePrincipalAssignments.CreateOrUpdateAsync(resourceGroupName, clusterName, databaseName, principalAssignmentName, databasePrincipalAssignment);
```

|**設定** | **建議的值** | **欄位描述**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-xxxxx-xxxx-xxxxxxxxx* | 您的租用戶識別碼。 也稱為目錄識別碼。|
| subscriptionId | *xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-xxxxx-xxxx-xxxxxxxxx* | 您用來建立資源的訂用帳戶識別碼。|
| clientId | *xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-xxxxx-xxxx-xxxxxxxxx* | 應用程式的用戶端識別碼，可存取您租使用者中的資源。|
| clientSecret | *xxxxxxxxxxxxxx* | 應用程式的用戶端密碼，可以存取您租使用者中的資源。 |
| resourceGroupName | *testrg* | 包含您叢集的資源組名。|
| clusterName | *mykustocluster* | 叢集的名稱。|
| databaseName | *mykustodatabase* | 您的資料庫名稱。|
| principalAssignmentName | *databasePrincipalAssignment1* | 資料庫主體資源的名稱。|
| principalId | *xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-xxxxx-xxxx-xxxxxxxxx* | 主體識別碼，可以是使用者電子郵件、應用程式識別碼或安全性群組名稱。|
| 角色 (role) | *管理員* | 資料庫主體的角色，可以是 ' Admin '、' 擷取器 '、' Monitor '、' User '、' UnrestrictedViewers '、' Viewer '。|
| tenantIdForPrincipal | *xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-xxxxx-xxxx-xxxxxxxxx* | 主體的租使用者識別碼。|
| PrincipalType | *應用程式* | 主體的類型，可以是「使用者」、「應用程式」或「群組」|

## <a name="next-steps"></a>後續步驟

* [使用 Azure 資料總管 Node 程式庫內嵌資料](node-ingest-data.md)
