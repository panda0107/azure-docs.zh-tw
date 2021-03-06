---
title: Azure 安全性控制-記錄和監視
description: 安全性控制記錄和監視
author: msmbaldwin
manager: rkarlin
ms.service: security
ms.topic: conceptual
ms.date: 12/30/2019
ms.author: mbaldwin
ms.custom: security-recommendations
ms.openlocfilehash: ae9c678d9dfca895ec74ed92bcb1b541db6b134e
ms.sourcegitcommit: 87781a4207c25c4831421c7309c03fce5fb5793f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2020
ms.locfileid: "76545495"
---
# <a name="security-control-logging-and-monitoring"></a>安全性控制：記錄和監視

安全性記錄和監視著重于與啟用、取得及儲存 Azure 服務的審核記錄相關的活動。

## <a name="21-use-approved-time-synchronization-sources"></a>2.1：使用已核准的時間同步處理來源

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.1 | 6.1 | Microsoft |

不過，Microsoft 會維護 Azure 資源的時間來源，您可以選擇管理計算資源的時間同步處理設定。

如何設定 Azure 計算資源的時間同步處理：

https://docs.microsoft.com/azure/virtual-machines/windows/time-sync

## <a name="22-configure-central-security-log-management"></a>2.2：設定中央安全性記錄管理

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.2 | 6.5、6。6 | 客戶 |

透過 Azure 監視器內嵌記錄，以匯總端點裝置、網路資源和其他安全性系統所產生的安全性資料。 在 Azure 監視器中，使用 Log Analytics 工作區來查詢和執行分析，並將 Azure 儲存體帳戶用於長期/封存儲存體。

或者，您可以啟用和內部資料，以 Azure Sentinel 或協力廠商 SIEM。 如何上架 Azure Sentinel：

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

如何使用 Azure 監視器收集平臺記錄和計量：

https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings

如何使用 Azure 監視器收集 Azure 虛擬機器內部主機記錄檔：

https://docs.microsoft.com/azure/azure-monitor/learn/quick-collect-azurevm

如何開始使用 Azure 監視器和協力廠商 SIEM 整合：

https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/

## <a name="23-enable-audit-logging-for-azure-resources"></a>2.3：啟用 Azure 資源的審核記錄

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.3 | 6.2、6。3 | 客戶 |

啟用 Azure 資源的診斷設定，以存取 audit、security 和診斷記錄。 活動記錄（可自動取得）包含事件來源、日期、使用者、時間戳記、來源位址、目的地位址，以及其他實用的元素。

如何使用 Azure 監視器收集平臺記錄和計量：

https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings

瞭解 Azure 中的記錄和不同的記錄類型：

https://docs.microsoft.com/azure/azure-monitor/platform/platform-logs-overview

## <a name="24-collect-security-logs-from-operating-systems"></a>2.4：從作業系統收集安全性記錄

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.4 | 6.2、6。3 | 客戶 |

如果計算資源是由 Microsoft 所擁有，則 Microsoft 會負責監視它。 如果計算資源由您的組織所擁有，則您必須負責監視它。 您可以使用 Azure 資訊安全中心來監視 OS。 資訊安全中心從作業系統收集的資料包括作業系統類型和版本、作業系統記錄（Windows 事件記錄）、執行中的進程、電腦名稱稱、IP 位址，以及已登入的使用者。 Log Analytics 代理程式也會收集損毀傾印檔案。

如何使用 Azure 監視器收集 Azure 虛擬機器內部主機記錄檔：

https://docs.microsoft.com/azure/azure-monitor/learn/quick-collect-azurevm

瞭解 Azure 資訊安全中心資料收集：

https://docs.microsoft.com/azure/security-center/security-center-enable-data-collection

## <a name="25-configure-security-log-storage-retention"></a>2.5：設定安全性記錄儲存體保留

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.5 | 6.4 | 客戶 |

在 Azure 監視器中，根據貴組織的合規性法規來設定您的 Log Analytics 工作區保留期間。 針對長期/封存儲存體使用 Azure 儲存體帳戶。

如何設定 Log Analytics 工作區的記錄檔保留參數：

https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period

## <a name="26-monitor-and-review-logs"></a>2.6：監視和審核記錄

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.6 | 6.7 | 客戶 |

分析和監視記錄中的異常行為，並定期查看結果。 使用 Azure 監視器的 Log Analytics 工作區來審查記錄，並對記錄資料執行查詢。

或者，您可以啟用和內部資料，以 Azure Sentinel 或協力廠商 SIEM。 

如何上架 Azure Sentinel：

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

瞭解 Log Analytics 工作區：

https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

如何在 Azure 監視器中執行自訂查詢：

https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries

## <a name="27-enable-alerts-for-anomalous-activity"></a>2.7：啟用異常活動的警示

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.7 | 6.8 | 客戶 |

使用 Azure 資訊安全中心搭配 Log Analytics 工作區，以在安全性記錄和事件中發現的異常活動上進行監視和警示。

或者，您可以啟用和麵板上的資料來 Azure Sentinel。

如何上架 Azure Sentinel：

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

如何管理 Azure 資訊安全中心中的警示：

https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts

如何對 log analytics 記錄資料發出警示：

https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response

## <a name="28-centralize-anti-malware-logging"></a>2.8：集中反惡意程式碼記錄

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.8 | 8.6 | 客戶 |

針對 Azure 虛擬機器和雲端服務啟用反惡意程式碼事件收集。

如何設定虛擬機器的 Microsoft Antimalware：

https://docs.microsoft.com/powershell/module/servicemanagement/azure/set-azurevmmicrosoftantimalwareextension?view=azuresmps-4.0.0

如何設定雲端服務的 Microsoft Antimalware：

https://docs.microsoft.com/powershell/module/servicemanagement/azure/set-azureserviceantimalwareextension?view=azuresmps-4.0.0

瞭解 Microsoft Antimalware：

https://docs.microsoft.com/azure/security/fundamentals/antimalware

## <a name="29-enable-dns-query-logging"></a>2.9：啟用 DNS 查詢記錄

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.9 | 8.7 | 客戶 |

為 DNS 記錄執行協力廠商解決方案。

## <a name="210-enable-command-line-audit-logging"></a>2.10：啟用命令列審核記錄

| Azure 識別碼 | CIS 識別碼 | 責任 |
|--|--|--|
| 2.1 | 8.8 | 客戶 |

手動設定每個節點的主控台記錄和 PowerShell 轉譯。

## <a name="next-steps"></a>後續步驟

請參閱下一個安全性控制：身分[識別和存取控制](security-control-identity-access-control.md)
