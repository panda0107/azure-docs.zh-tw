---
title: 使用串流分析整合來串流資料（預覽）
description: 使用 Azure 串流分析將資料串流至 Azure SQL 資料庫。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: ajetasin
ms.author: ajetasi
ms.reviewer: sstein
ms.date: 11/04/2019
ms.openlocfilehash: d233d3c98cc495e4b9e84142781f5eb9faa7eec8
ms.sourcegitcommit: ac56ef07d86328c40fed5b5792a6a02698926c2d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/08/2019
ms.locfileid: "73820834"
---
# <a name="stream-data-by-using-azure-sql-database-stream-analytics-integration-preview"></a>使用 Azure SQL Database 串流分析整合來串流資料（預覽）

使用者現在可以使用[Azure 串流分析](../stream-analytics/stream-analytics-introduction.md)，直接從 Azure 入口網站中的 SQL database 內嵌、處理、查看和分析即時串流資料到資料表中。 這項體驗可提供各種不同的案例，例如連線的汽車、遠端監視、詐騙偵測等等。 在 Azure 入口網站中，您可以選取事件來源（事件中樞/IoT 中樞）、查看傳入的即時事件，然後選取要儲存事件的資料表。 您也可以在入口網站中撰寫串流分析查詢語言查詢，以轉換傳入的事件，並將其儲存在選取的資料表中。 除了串流分析中已存在的建立和設定體驗之外，這個新的進入點。 這項體驗會從資料庫的內容開始，讓您快速設定串流分析作業，並在 Azure SQL Database 和串流分析體驗之間順暢地流覽。

![串流分析流程](media/sql-database-stream-analytics/stream-analytics-flow.png)

## <a name="key-benefits"></a>主要權益

- 最小內容切換：您可以從入口網站中的 SQL Database 開始，並開始將即時資料內嵌到資料表中，而不需要切換至任何其他服務。 
- 減少步驟數：您的資料庫和資料表的內容會用來預先設定串流分析作業。
- 使用預覽資料的額外易用性：在選取的資料表內容中預覽來自事件來源的傳入資料（事件中樞/IoT 中樞） 


## <a name="prerequisites"></a>必要條件

若要完成本文中的步驟，您需要下列資源：

- Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請[建立免費帳戶](https://azure.microsoft.com/free/)。 
- SQL 資料庫。 如需詳細資訊，請參閱[在 Azure SQL Database 中建立單一資料庫](sql-database-single-database-get-started.md)。
- 防火牆規則，可讓您的電腦連線到 Azure SQL server。 如需詳細資訊，請參閱[建立伺服器層級防火牆規則](sql-database-server-level-firewall-rule.md)。


## <a name="configure-stream-analytics-integration"></a>設定串流分析整合

1. 登入 Azure 入口網站。 
2. 流覽至您想要內嵌串流資料的 SQL database。 選取 **[串流分析（預覽）** ]。 

    ![串流分析](media/sql-database-stream-analytics/stream-analytics.png)

3. 若要開始將串流資料內嵌到此 SQL 資料庫，請選取 [**建立**] 並為您的串流作業指定名稱，然後選取 **[下一步：輸入]** 。 

    ![建立串流分析作業](media/sql-database-stream-analytics/create-job.png)

4. 輸入您的事件來源詳細資料，然後選取 **[下一步：輸出]** 。

   - **輸入類型**：事件中樞/IoT 中樞
   - **輸入別名**：輸入用來識別事件來源的名稱 
   - **訂**用帳戶：與 SQL Database 訂用帳戶相同 
   - **事件中樞命名空間**：命名空間的名稱 
   - **事件中樞名稱**：所選取命名空間內的事件中樞名稱 
   - **事件中樞原則名稱**（預設為建立新的）：提供原則名稱 
   - **事件中樞取用者群組**（預設為建立新的）：提供取用者組名  
     - 我們建議您為每個您從這裡建立的新 Azure 串流分析作業，建立取用者群組和原則。 取用者群組只允許五個並行讀取器，因此為每項作業提供專用的取用者群組，將可避免因超出該限制而引發的任何錯誤。 專用原則可讓您旋轉金鑰或撤銷許可權，而不會影響其他資源。

    ![建立串流分析作業](media/sql-database-stream-analytics/create-job-output.png)

5. 選取您想要內嵌串流資料的目標資料表。 完成後，選取 [**建立**]。
   - 使用者**名稱**、**密碼**：輸入您的認證以進行 SQL server 驗證。 選取 [驗證]。
   - **資料表**：選取 [**新建**] 或 [**使用現有**的]。 在此流程中，讓我們選取 [**建立**]。 當您啟動串流分析作業時，這會建立新的資料表。

    ![建立串流分析作業](media/sql-database-stream-analytics/create.png)

6. [查詢] 頁面隨即開啟，其中包含下列詳細資料：

   - 您將從中內嵌資料的**輸入**（輸入事件來源）  
   - 將儲存已轉換資料的**輸出**（輸出資料表） 
   - 使用 SELECT 語句的範例[SAQL 查詢](../stream-analytics/stream-analytics-stream-analytics-query-patterns.md)。 
   - **輸入預覽**：顯示輸入事件來源中最新傳入資料的快照集。  
     - 系統會自動偵測您資料中的序列化類型（JSON/CSV）。 您也可以手動將它變更為 JSON/CSV/AVRO。 
     - 您可以預覽資料表格式或原始格式的傳入資料。 
     - 如果顯示的資料不是最新的，**請選取 [** 重新整理] 以查看最新的事件。 
     - 選取 [**選取時間範圍**]，以根據傳入事件的特定時間範圍來測試查詢。 
     - 選取 **[上傳範例輸入**]，藉由上傳範例 JSON/CSV 檔案來測試您的查詢。 如需有關測試 SAQL 查詢的詳細資訊，請參閱[使用範例資料測試 Azure 串流分析作業](../stream-analytics/stream-analytics-test-query.md)。 

    ![測試查詢](media/sql-database-stream-analytics/test-query.png)


   - **測試結果**：選取 [**測試查詢**]，您可以看到串流查詢的結果 

    ![測試結果](media/sql-database-stream-analytics/test-results.png)

   - **測試結果架構**：顯示測試之後，串流查詢結果的架構。 請確定測試結果架構符合您的輸出架構。 

    ![測試結果架構](media/sql-database-stream-analytics/test-results-schema.png)


   - **輸出架構**：這包含您在步驟5中選取之資料表的架構（新的或現有的）。
     - 建立新的：如果您在步驟5中選取此選項，在您開始串流作業之前，您將不會看到架構。 建立新的資料表時，請選取適當的資料表索引。 如需資料表索引編制的詳細資訊，請參閱叢集[與非叢集索引說明](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described/)。
    - 使用現有的：如果您在步驟5中選取此選項，您會看到所選資料表的架構。 
 
7. 完成撰寫 & 測試查詢之後，請選取 [**儲存查詢**]。 選取 [**啟動串流分析作業**]，開始將已轉換的資料內嵌至 SQL 資料表。 完成下欄欄位之後，請**啟動**作業。 
   - **輸出開始時間**：這會定義工作第一個輸出的時間。  
     - 現在：作業將會立即啟動，並處理新的傳入資料。
     - 自訂：作業將會立即啟動，但會處理來自特定時間點（可能是過去或未來）的資料。 如需詳細資訊，請參閱[如何啟動 Azure 串流分析作業](../stream-analytics/start-job.md)。
   - **串流處理單位**： Azure 串流分析的定價是將資料處理到服務所需的串流處理單位數。 如需詳細資訊，請參閱[Azure 串流分析定價](https://azure.microsoft.com/pricing/details/stream-analytics/)。 
   - **輸出資料錯誤處理**：  
     - 重試：發生錯誤時，Azure 串流分析會無限期地重試寫入事件，直到寫入成功為止。 重試不會逾時。 最終所有後續事件都會遭到正在重試的事件封鎖，而無法處理。 此選項是預設輸出錯誤處理原則。 
     - Drop： Azure 串流分析將會卸載任何導致資料轉換錯誤的輸出事件。 無法復原卸除的事件以供稍後重新處理。 無論輸出錯誤處理原則組態為何，所有暫時性錯誤 (例如網路錯誤) 都會重試。 
   - **SQL Database 輸出設定**：繼承上一個查詢步驟之資料分割配置的選項，可讓多個寫入器具有資料表的完全平行拓撲。 如需詳細資訊，請參閱 [Azure 串流分析輸出至 Azure SQL Database](../stream-analytics/stream-analytics-sql-output-perf.md)。
   - **批次計數上限**：與每個大量插入交易一起傳送之記錄數目的建議上限。  
    如需輸出錯誤處理的詳細資訊，請參閱[Azure 串流分析中的輸出錯誤原則](../stream-analytics/stream-analytics-output-error-policy.md)。  

    ![啟動作業](media/sql-database-stream-analytics/start-job.png)

8. 當您啟動作業時，您會在清單中看到執行中的作業，而且您可以採取下列動作： 
   - **啟動/停止作業**：如果作業正在執行，您可以停止作業。 如果作業已停止，您可以啟動作業。 
   - **編輯作業**：您可以編輯查詢。 如果您想要對作業進行更多變更（例如，新增更多輸入/輸出），請在串流分析中開啟作業。 當作業正在執行時，會停用 [編輯] 選項。 
   - **預覽輸出資料表**：您可以在 [SQL 查詢編輯器] 中預覽資料表。 
   - **在串流分析中開啟**：在串流分析服務中開啟作業，以查看監視、調試作業的詳細資料。 


    ![串流分析作業](media/sql-database-stream-analytics/jobs.png)






## <a name="next-steps"></a>後續步驟

- [Azure 串流分析文件](https://docs.microsoft.com/azure/stream-analytics/)
- [Azure 串流分析解決方案模式](../stream-analytics/stream-analytics-solution-patterns.md)
