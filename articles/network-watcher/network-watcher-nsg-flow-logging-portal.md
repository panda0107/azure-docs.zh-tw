---
title: 記錄往返 VM 的流量 - 教學課程 - Azure 入口網站 | Microsoft Docs
description: 了解如何使用網路監看員的 NSG 流量記錄功能來記錄往返 VM 的流量。
services: network-watcher
documentationcenter: na
author: damendo
tags: azure-resource-manager
Customer intent: I need to log the network traffic to and from a VM so I can analyze it for anomalies.
ms.assetid: 01606cbf-d70b-40ad-bc1d-f03bb642e0af
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/30/2018
ms.author: damendo
ms.custom: mvc
ms.openlocfilehash: f3448765eecf4a586e13155903f1c093607781dc
ms.sourcegitcommit: 67e9f4cc16f2cc6d8de99239b56cb87f3e9bff41
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/31/2020
ms.locfileid: "76896442"
---
# <a name="tutorial-log-network-traffic-to-and-from-a-virtual-machine-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站記錄往返於虛擬機器的網路流量

> [!div class="op_single_selector"]
> - [Azure 入口網站](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [Azure CLI](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)
> - [Azure Resource Manager](network-watcher-nsg-flow-logging-azure-resource-manager.md)

網路安全性群組 (NSG) 可讓您篩選虛擬機器 (VM) 的輸入流量和輸出流量。 您可以使用網路監看員的 NSG 流量記錄功能，記錄流經 NSG 的網路流量。 在本教學課程中，您會了解如何：

> [!div class="checklist"]
> * 建立具有網路安全性群組的 VM
> * 啟用網路監看員，並註冊 Microsoft.Insights 提供者
> * 啟用 NSG 的流量記錄 (使用網路監看員的 NSG 流程記錄功能)
> * 下載記錄的資料
> * 檢視記錄的資料

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="create-a-vm"></a>建立 VM

1. 選取 Azure 入口網站左上角的 [+ 建立資源]  。
2. 選取 [計算]  ，然後選取 [Windows Server 2016 Datacenter]  或某個版本的 [Ubuntu Server]  。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [確定]  ：

    |設定|值|
    |---|---|
    |名稱|myVm|
    |[使用者名稱]| 輸入您選擇的使用者名稱。|
    |密碼| 輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |訂用帳戶| 選取您的訂用帳戶。|
    |資源群組| 選取 [新建]  ，然後輸入 **myResourceGroup**。|
    |Location| 選取 [美國東部] |

4. 選取 VM 的大小，然後選取 [選取]  。
5. 在 [設定]  底下，接受所有預設值，然後選取 [確定]  。
6. 在 [摘要]  的 [建立]  底下，選取 [建立]  來開始部署 VM。 部署 VM 需要幾分鐘的時間。 等候虛擬機器完成部署，再繼續進行其餘步驟。

建立 VM 需要幾分鐘的時間。 完成 VM 建立之前，請不要繼續其餘步驟。 入口網站建立 VM 的同時，也會建立名為 **myVm-nsg** 的網路安全性群組，並讓它與 VM 的網路介面相關聯。

## <a name="enable-network-watcher"></a>啟用網路監看員

如果您已經在美國東部區域中啟用網路監看員，請跳至[註冊 Insights 提供者](#register-insights-provider)。

1. 在入口網站中，選取 [所有服務]  。 在 [篩選條件]  方塊中，輸入*網路監看員*。 當結果中出現**網路監看員**時，請加以選取。
2. 選取 [地區]  、展開它，然後選取 [美國東部]  右邊的 [...]  ，如下圖所示：

    ![啟用網路監看員](./media/network-watcher-nsg-flow-logging-portal/enable-network-watcher.png)

3. 選取 [啟用網路監看員]  。

## <a name="register-insights-provider"></a>註冊 Insights 提供者

NSG 流量記錄需要 **Microsoft.Insights** 提供者。 若要註冊提供者，請完成下列步驟：

1. 在入口網站的左上角，選取 [所有服務]  。 在 [篩選] 方塊中輸入 [訂用帳戶]  。 當搜尋結果中出現**訂用帳戶**時加以選取。
2. 從訂用帳戶的清單中，選取您要為其啟用提供者的訂用帳戶。
3. 在 [設定]  下，選取 [資源提供者]  。
4. 確認 **microsoft.insights** 提供者的**狀態**是 [已註冊]  ，如下圖所示。 如果狀態為 [未註冊]  ，則選取提供者右邊的 [註冊]  。

    ![註冊提供者](./media/network-watcher-nsg-flow-logging-portal/register-provider.png)

## <a name="enable-nsg-flow-log"></a>啟用 NSG 流量記錄

1. NSG 流量記錄資料會寫入至 Azure 儲存體帳戶。 若要建立 Azure 儲存體帳戶，請選取入口網站左上角的 [+ 建立資源]  。
2. 選取 [儲存體]  ，然後選取 [儲存體帳戶 - Blob、檔案、資料表、佇列]  。
3. 輸入或選取下列資訊、接受其餘預設值，然後選取 [建立]  。

    | 設定        | 值                                                        |
    | ---            | ---   |
    | 名稱           | 3-24 個字元長，只能包含小寫英文字母和數字，並且必須是所有 Azure 儲存體帳戶中唯一的名稱。                                                               |
    | Location       | 選取 [美國東部]                                            |
    | 資源群組 | 選取 [使用現有的]  ，然後選取 [myResourceGroup]  |

    建立儲存體帳戶可能需要一分鐘的時間。 建好儲存體帳戶之前，請不要繼續其餘步驟。 在所有情況下，儲存體帳戶必須與 NSG 位在同一個區域中。
4. 在入口網站的左上角，選取 [所有服務]  。 在 [篩選]  方塊中，輸入*網路監看員*。 當搜尋結果中出現**網路監看員**時，請加以選取。
5. 在 [記錄]  下，選取 [NSG 流量記錄]  ，如下列圖所示：

    ![NSG](./media/network-watcher-nsg-flow-logging-portal/nsgs.png)

6. 從 NSG 清單中，選取名為 **myVm-nsg** 的 NSG。
7. 在 [流量記錄設定]  下，選取 [開啟]  。
8. 選取流程記錄版本。 第 2 版包含流量工作階段統計資料 (位元組和封包)

   ![選取流量記錄版本](./media/network-watcher-nsg-flow-logging-portal/select-flow-log-version.png)

9. 選取您在步驟 3 建立的儲存體帳戶。
   > [!NOTE]
   > NSG 流量記錄無法與已啟用[階層式命名空間](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-namespace)的儲存體帳戶搭配使用。
1. 在入口網站的左上角，選取 [所有服務]  。 在 [篩選]  方塊中，輸入*網路監看員*。 當搜尋結果中出現**網路監看員**時，請加以選取。
10. 將 [保留 (天數)]  設定為 5，然後選取 [儲存]  。

## <a name="download-flow-log"></a>下載流量記錄

1. 從入口網站的網路監看員中，選取 [記錄]  下方的 [NSG 流量記錄]  。
2. 選取 [您可從設定的儲存體帳戶下載流量記錄]  ，如下圖所示：

   ![下載流量記錄](./media/network-watcher-nsg-flow-logging-portal/download-flow-logs.png)

3. 選取您在步驟 2 ([啟用 NSG 流量記錄](#enable-nsg-flow-log)) 設定的儲存體帳戶。
4. 在 [Blob 服務]  下選取 [Blob]  ，然後選取 **insights-logs-networksecuritygroupflowevent** 容器。
5. 在容器中，瀏覽資料夾階層，直到看到 PT1H.json 檔案為止，如下圖所示。 記錄檔會寫入至遵循下列命名慣例的資料夾階層： https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json

   ![流量記錄](./media/network-watcher-nsg-flow-logging-portal/log-file.png)

6. 選取 PT1H.json 檔案右邊的 [...]  ，然後選取 [下載]  。

## <a name="view-flow-log"></a>檢視流量記錄

對於每個已記錄其資料的流量，下列 json 範例就是您會在 PT1H.json 檔案中看到的內容：

### <a name="version-1-flow-log-event"></a>第 1 版的流程記錄事件
```json
{
    "time": "2018-05-01T15:00:02.1713710Z",
    "systemId": "<Id>",
    "category": "NetworkSecurityGroupFlowEvent",
    "resourceId": "/SUBSCRIPTIONS/<Id>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/MYVM-NSG",
    "operationName": "NetworkSecurityGroupFlowEvents",
    "properties": {
        "Version": 1,
        "flows": [
            {
                "rule": "UserRule_default-allow-rdp",
                "flows": [
                    {
                        "mac": "000D3A170C69",
                        "flowTuples": [
                            "1525186745,192.168.1.4,10.0.0.4,55960,3389,T,I,A"
                        ]
                    }
                ]
            }
        ]
    }
}
```
### <a name="version-2-flow-log-event"></a>第 2 版的流程記錄事件
```json
{
    "time": "2018-11-13T12:00:35.3899262Z",
    "systemId": "a0fca5ce-022c-47b1-9735-89943b42f2fa",
    "category": "NetworkSecurityGroupFlowEvent",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
    "operationName": "NetworkSecurityGroupFlowEvents",
    "properties": {
        "Version": 2,
        "flows": [
            {
                "rule": "DefaultRule_DenyAllInBound",
                "flows": [
                    {
                        "mac": "000D3AF87856",
                        "flowTuples": [
                            "1542110402,94.102.49.190,10.5.16.4,28746,443,U,I,D,B,,,,",
                            "1542110424,176.119.4.10,10.5.16.4,56509,59336,T,I,D,B,,,,",
                            "1542110432,167.99.86.8,10.5.16.4,48495,8088,T,I,D,B,,,,"
                        ]
                    }
                ]
            },
            {
                "rule": "DefaultRule_AllowInternetOutBound",
                "flows": [
                    {
                        "mac": "000D3AF87856",
                        "flowTuples": [
                            "1542110377,10.5.16.4,13.67.143.118,59831,443,T,O,A,B,,,,",
                            "1542110379,10.5.16.4,13.67.143.117,59932,443,T,O,A,E,1,66,1,66",
                            "1542110379,10.5.16.4,13.67.143.115,44931,443,T,O,A,C,30,16978,24,14008",
                            "1542110406,10.5.16.4,40.71.12.225,59929,443,T,O,A,E,15,8489,12,7054"
                        ]
                    }
                ]
            }
        ]
    }
}
```

上述輸出中的 **mac** 值是網路介面的 MAC 位址，該網路介面是在建立 VM 時建立的。 以逗號分隔的 **flowTuples** 資訊代表下列內容：

| 範例資料 | 資料代表的意義   | 說明                                                                              |
| ---          | ---                    | ---                                                                                      |
| 1542110377   | 時間戳記             | 流量發生時的時間戳記，格式為 UNIX EPOCH。 在上述範例中，日期會轉換為 2018 年 5 月 1 日下午 2:59:05 GMT。                                                                                    |
| 10.0.0.4  | 來源 IP 位址      | 流量源頭的來源 IP 位址。 對於您在[建立 VM](#create-a-vm) 中建立的 VM，10.0.0.4 是此 VM 的私人 IP 位址。
| 13.67.143.118     | 目的地 IP 位址 | 流量流向的目的地 IP 位址。                                                                                  |
| 44931        | 來源連接埠            | 流量源頭的來源連接埠。                                           |
| 443         | 目的地連接埠       | 流量流向的目的地連接埠。 因為流量已流向連接埠 443，記錄檔中名為 **UserRule_default-allow-rdp** 的規則會處理此流量。                                                |
| T            | 通訊協定               | 流量的通訊協定是 TCP (T) 或 UDP (U)。                                  |
| O            | 方向              | 流量是輸入 (I) 或輸出 (O)。                                     |
| A            | 動作                 | 允許 (A) 或拒絕 (D) 流量。  
| C            | 流程狀態 **僅限第 2 版** | 擷取流程的狀態。 可能的狀態為 **B**：開始，當流量建立時。 不提供統計資料。 **C**：繼續進行中的流量。 提供 5 分鐘間隔的統計資料。 **E**：結束，當流量終止時。 提供統計資料。 |
| 30 | 已傳送的封包數 - 來源到目的地 **僅限第 2 版** | 上次更新之後從來源傳送到目的地的 TCP 或 UDP 封包總數。 |
| 16978 | 已傳送的位元組數 - 來源到目的地 **僅限第 2 版** | 上次更新之後從來源傳送到目的地的 TCP 或 UDP 封包位元組總數。 封包位元組包括封包標頭與承載。 |
| 24 | 已傳送的封包數 - 目的地到來源 **僅限第 2 版** | 上次更新之後從目的地傳送到來源的 TCP 或 UDP 封包總數。 |
| 14008| 已傳送的位元組數 - 目的地到來源 **僅限第 2 版** | 上次更新之後從目的地傳送到來源的 TCP 與 UDP 封包位元組總數。 封包位元組包括封包標頭與承載。|

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何啟用 NSG 的 NSG 流量記錄。 您也了解如何下載和檢視記錄在檔案中的資料。 Json 檔案中的原始資料可能難以解譯。 若要將流程記錄資料視覺化，您可以使用 [Azure 流量分析](traffic-analytics.md)、[Microsoft PowerBI](network-watcher-visualize-nsg-flow-logs-power-bi.md) 和其他工具。 您可以嘗試其他方法來啟用 NSG 流量記錄，例如 [PowerShell](network-watcher-nsg-flow-logging-powershell.md)、[Azure CLI](network-watcher-nsg-flow-logging-cli.md)、[REST API](network-watcher-nsg-flow-logging-rest.md) 和 [ARM 範本](network-watcher-nsg-flow-logging-azure-resource-manager.md)。
