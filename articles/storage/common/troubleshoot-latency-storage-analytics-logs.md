---
title: 使用儲存體分析記錄來疑難排解延遲
description: 使用 Azure 儲存體分析記錄來識別和疑難排解延遲問題，並將用戶端應用程式優化。
author: v-miegge
ms.topic: troubleshooting
ms.author: kartup
manager: dcscontentpm
ms.date: 10/21/2019
ms.service: storage
ms.subservice: common
services: storage
tags: ''
ms.openlocfilehash: 2197a149235c0dca98a24a57549538b2a4cbb1c8
ms.sourcegitcommit: 8e31a82c6da2ee8dafa58ea58ca4a7dd3ceb6132
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/19/2019
ms.locfileid: "74196512"
---
# <a name="troubleshoot-latency-using-storage-analytics-logs"></a>使用儲存體分析記錄來疑難排解延遲

診斷和疑難排解是使用 Azure 儲存體建立和支援用戶端應用程式的重要技能。

由於 Azure 應用程式的分散式本質，診斷和疑難排解錯誤和效能問題可能會比在傳統環境中更複雜。

下列步驟示範如何使用 Azure 儲存體分析記錄來識別和疑難排解延遲問題，並將用戶端應用程式優化。

## <a name="recommended-steps"></a>建議的步驟

1. 下載[儲存體分析記錄](https://docs.microsoft.com/azure/storage/common/storage-analytics-logging#download-storage-logging-log-data)。

2. 使用下列 PowerShell 腳本，將原始格式記錄轉換成表格式格式：

   ```Powershell
   $Columns = 
        (   "version-number",
            "request-start-time",
            "operation-type",
            "request-status",
            "http-status-code",
            "end-to-end-latency-in-ms",
            "server-latency-in-ms",
            "authentication-type",
            "requester-account-name",
            "owner-account-name",
            "service-type",
            "request-url",
            "requested-object-key",
            "request-id-header",
            "operation-count",
            "requester-ip-address",
            "request-version-header",
            "request-header-size",
            "request-packet-size",
            "response-header-size",
            "response-packet-size",
            "request-content-length",
            "request-md5",
            "server-md5",
            "etag-identifier",
            "last-modified-time",
            "conditions-used",
            "user-agent-header",
            "referrer-header",
            "client-request-id"
        )

   $logs = Import-Csv “REPLACE THIS WITH FILE PATH” -Delimiter ";" -Header $Columns

   $logs | Out-GridView -Title "Storage Analytic Log Parser"
   ```

3. 腳本會啟動 GUI 視窗，您可以在其中依資料行篩選資訊，如下所示。

   ![儲存體分析記錄剖析器視窗](media/troubleshoot-latency-storage-analytics-logs/storage-analytic-log-parser-window.png)
 
4. 根據「操作類型」縮小記錄專案的範圍，並尋找在問題的時間範圍內建立的記錄專案。

   ![操作類型記錄專案](media/troubleshoot-latency-storage-analytics-logs/operation-type.png)

5. 發生問題時，下列值很重要：

   * Operation 類型 = GetBlob
   * 要求-狀態 = SASNetworkError
   * 端對端-延遲-毫秒 = 8453
   * 伺服器-延遲-毫秒 = 391

   端對端延遲是使用下列方程式來計算：

   * 端對端延遲 = 伺服器延遲 + 用戶端延遲

   使用記錄專案來計算用戶端延遲：

   * 用戶端延遲 = 端對端延遲–伺服器-延遲

          * Example: 8453 – 391 = 8062ms

   下表提供高延遲 OperationType 和 RequestStatus 結果的相關資訊：

   |   |RequestStatus =<br>成功|RequestStatus =<br>SASNetworkError|建議|
   |---|---|---|---|
   |GetBlob|yes|否|[**GetBlob 作業：** RequestStatus = 成功](#getblob-operation-requeststatus--success)|
   |GetBlob|否|yes|[**GetBlob 作業：** RequestStatus = （SAS） NetworkError](#getblob-operation-requeststatus--sasnetworkerror)|
   |PutBlob|yes|否|[**Put 作業：** RequestStatus = 成功](#put-operation-requeststatus--success)|
   |PutBlob|否|yes|[**Put 作業：** RequestStatus = （SAS） NetworkError](#put-operation-requeststatus--sasnetworkerror)|

## <a name="status-results"></a>狀態結果

### <a name="getblob-operation-requeststatus--success"></a>GetBlob 作業： RequestStatus = 成功

如「建議的步驟」一節的步驟5所述，檢查下列值：

* 端對端延遲
* 伺服器-延遲
* 用戶端-延遲

在**RequestStatus = Success**的**GetBlob**作業中，如果在**用戶端延遲**中花費**最大時間**，這表示 Azure 儲存體花大量時間將資料寫入至用戶端。 此延遲表示用戶端問題。

**建議：**

* 調查用戶端中的程式碼。
* 使用 Wireshark、Microsoft Message Analyzer 或 Tcping 調查用戶端的網路連線問題。 

### <a name="getblob-operation-requeststatus--sasnetworkerror"></a>GetBlob 作業： RequestStatus = （SAS） NetworkError

如「建議的步驟」一節的步驟5所述，檢查下列值：

* 端對端延遲
* 伺服器-延遲
* 用戶端-延遲

在**RequestStatus = （SAS） NetworkError**的**GetBlob**作業中，如果在**用戶端延遲**中花費**最大時間**，最常見的問題是用戶端在儲存體服務中的超時時間到期之前就中斷連線。

**建議：**

* 請調查用戶端裡的程式碼，了解用戶端何時及為何與儲存體服務中斷連線。
* 使用 Wireshark、Microsoft Message Analyzer 或 Tcping 調查用戶端的網路連線問題。 

### <a name="put-operation-requeststatus--success"></a>Put 作業： RequestStatus = Success

如「建議的步驟」一節的步驟5所述，檢查下列值：

* 端對端延遲
* 伺服器-延遲
* 用戶端-延遲

在**RequestStatus = Success**的**Put**作業中，如果在**用戶端延遲**中花費**最大時間**，這表示用戶端會花更多時間將資料傳送到 Azure 儲存體。 此延遲表示用戶端問題。

**建議：**

* 調查用戶端中的程式碼。
* 使用 Wireshark、Microsoft Message Analyzer 或 Tcping 調查用戶端的網路連線問題。 

### <a name="put-operation-requeststatus--sasnetworkerror"></a>Put Operation： RequestStatus = （SAS） NetworkError

如「建議的步驟」一節的步驟5所述，檢查下列值：

* 端對端延遲
* 伺服器-延遲
* 用戶端-延遲

在**RequestStatus = （SAS） NetworkError**的**PutBlob**作業中，如果在**用戶端延遲**中花費**最大時間**，最常見的問題是用戶端在儲存體服務中的超時時間到期之前就中斷連線。

**建議：**

* 請調查用戶端裡的程式碼，了解用戶端何時及為何與儲存體服務中斷連線。
* 使用 Wireshark、Microsoft Message Analyzer 或 Tcping 調查用戶端的網路連線問題。

