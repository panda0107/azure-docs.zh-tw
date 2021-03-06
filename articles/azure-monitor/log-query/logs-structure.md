---
title: Azure 監視器記錄的結構 |Microsoft Docs
description: 您需要記錄查詢來從 Azure 監視器擷取記錄資料。  本文描述新的記錄查詢在 Azure 監視器中的使用方式，並且提供在建立之前需要了解的概念。
ms.service: azure-monitor
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/22/2019
ms.openlocfilehash: 6ce8470da6b444cedb7bff1d14bcc6448b52fe94
ms.sourcegitcommit: 8bd85510aee664d40614655d0ff714f61e6cd328
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/06/2019
ms.locfileid: "74893632"
---
# <a name="structure-of-azure-monitor-logs"></a>Azure 監視器記錄的結構
使用[記錄查詢](log-query-overview.md)快速取得資料見解的能力，是 Azure 監視器的強大功能。 若要建立有效率且有用的查詢，您應該瞭解一些基本概念，例如您想要的資料位於何處，以及其結構化方式。 本文提供您開始使用所需的基本概念。

## <a name="overview"></a>概觀
Azure 監視器記錄檔中的資料會儲存在 Log Analytics 工作區或 Application Insights 應用程式中。 這兩種技術都是由 Azure 所提供技術支援[資料總管](/azure/data-explorer/)表示它們會運用其強大的資料引擎和查詢語言。

工作區和應用程式中的資料會組織成資料表，其中每一個都儲存不同種類的資料，而且有自己的唯一屬性集。 大部分[資料來源](../platform/data-sources.md)都會在 Log Analytics 工作區中寫入自己的資料表，而 Application Insights 會寫入 Application Insights 應用程式中預先定義的一組資料表。 記錄查詢具有極大的彈性，可讓您輕鬆地結合多個資料表中的資料，甚至可以使用跨資源查詢來結合多個工作區中資料表的資料，或撰寫結合工作區和應用程式資料的查詢。

下圖顯示寫入範例查詢中所使用之不同資料表的資料來源範例。

![資料表](media/logs-structure/queries-tables.png)

## <a name="log-analytics-workspace"></a>Log Analytics 工作區
除了 Application Insights 以外，Azure 監視器記錄收集的所有資料都會儲存在[Log Analytics 工作區](../platform/manage-access.md)中。 根據您的特定需求，您可以建立一或多個工作區。 來自 Azure 資源的活動記錄和資源記錄、虛擬機器上的代理程式，以及來自深入解析和監視解決方案的資料等[資料來源](../platform/data-sources.md)，都會將資料寫入您在上架時設定的一或多個工作區。 [Azure 資訊安全中心](/azure/security-center/)和[Azure Sentinel](/azure/sentinel/)等其他服務也會使用 log Analytics 工作區來儲存其資料，以便使用記錄查詢進行分析，以及監視來自其他來源的資料。

不同類型的資料會儲存在工作區中的不同資料表，而且每個資料表都有一組唯一的屬性。 在建立工作區時，會將一組標準的資料表新增到其中，並在上架時針對不同的資料來源、方案和服務新增資料表。 您也可以使用[資料收集器 API](../platform/data-collector-api.md)來建立自訂資料表。

您可以在工作區的 Log Analytics 中的 [**架構**] 索引標籤中，流覽工作區中的資料表及其架構。

![工作區架構](media/scope/workspace-schema.png)

使用下列查詢來列出工作區中的資料表，以及過去一天內收集到的記錄數目。 

```Kusto
union withsource = table * 
| where TimeGenerated > ago(1d)
| summarize count() by table
| sort by table asc
```
如需所建立資料表的詳細資訊，請參閱每個資料來源的檔。 範例包括[代理程式資料來源](../platform/agent-data-sources.md)、[資源記錄](../platform/diagnostic-logs-schema.md)和[監視解決方案](../insights/solutions-inventory.md)的文章。

### <a name="workspace-permissions"></a>工作區許可權
請參閱[設計 Azure 監視器記錄部署](../platform/design-logs-deployment.md)，以瞭解存取控制策略和建議，以提供工作區中資料的存取權。 除了授與工作區本身的存取權之外，您還可以使用[資料表層級 RBAC](../platform/manage-access.md#table-level-rbac)來限制個別資料表的存取權。

## <a name="application-insights-application"></a>Application Insights 應用程式
當您在 Application Insights 中建立應用程式時，會在 Azure 監視器記錄中自動建立對應的應用程式。 收集資料不需要任何設定，而且應用程式會自動寫入監視資料，例如頁面流覽、要求和例外狀況。

不同于 Log Analytics 工作區，Application Insights 應用程式有一組固定的資料表。 您無法將其他資料來源設定為寫入應用程式，因此不能建立其他資料表。 

| 表格 | 描述 | 
|:---|:---|
| availabilityResults | 可用性測試中的摘要資料。 |
| browserTimings      | 用戶端效能的相關資料，例如處理傳入資料所花費的時間。 |
| customEvents        | 您的應用程式所建立的自訂事件。 |
| customMetrics       | 您的應用程式所建立的自訂計量。 |
| 相依性        | 從應用程式呼叫外部元件。 |
| 例外狀況          | 應用程式執行時間擲回的例外狀況。 |
| pageViews           | 具有瀏覽器資訊的每個網站視圖的相關資料。 |
| performanceCounters | 支援應用程式之計算資源的效能測量。 |
| requests            | 每個應用程式要求的詳細資料。  |
| traces              | 分散式追蹤的結果。 |

針對應用程式，您可以在 Log Analytics 的 [**架構**] 索引標籤中，查看每個資料表的架構。

![應用程式架構](media/scope/application-schema.png)

## <a name="standard-properties"></a>標準屬性
雖然 Azure 監視器記錄中的每個資料表都有自己的架構，但有所有資料表共用的標準屬性。 如需每個內容的詳細資訊，請參閱[Azure 監視器記錄中的標準屬性](../platform/log-standard-properties.md)。

| Log Analytics 工作區 | Application Insights 應用程式 | 描述 |
|:---|:---|:---|
| TimeGenerated | timestamp  | 建立記錄的日期和時間。 |
| Type          | itemType   | 從中抓取記錄的資料表名稱。 |
| _ResourceId   |            | 與記錄相關聯之資源的唯一識別碼。 |
| _IsBillable   |            | 指定內嵌資料是否可計費。 |
| _BilledSize   |            | 指定將計費的資料大小（以位元組為單位）。 |

## <a name="next-steps"></a>後續步驟
- 瞭解如何使用[Log Analytics 來建立和編輯記錄搜尋](../log-query/portals.md)。
- 請參閱使用新的查詢語言[撰寫查詢的教學課程](../log-query/get-started-queries.md)。
