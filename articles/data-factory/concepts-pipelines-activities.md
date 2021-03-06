---
title: Azure Data Factory 中的管道及活動
description: 了解 Azure Data Factory 中的管道及活動。
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 11/19/2019
ms.openlocfilehash: 6e466675a9bd86693ce0ee048480712a55829ce6
ms.sourcegitcommit: 653e9f61b24940561061bd65b2486e232e41ead4
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/21/2019
ms.locfileid: "74280744"
---
# <a name="pipelines-and-activities-in-azure-data-factory"></a>Azure Data Factory 中的管道及活動

> [!div class="op_single_selector" title1="選取您目前使用的 Data Factory 服務版本："]
> * [第 1 版](v1/data-factory-create-pipelines.md)
> * [目前的版本](concepts-pipelines-activities.md)

本文協助您了解 Azure Data Factory 中的管線和活動，並使用這些項目來為您的資料移動和資料處理案例建構端對端的資料導向工作流程。

## <a name="overview"></a>Overview
資料處理站可以有一或多個管線。 管線是一起執行某個工作的活動所組成的邏輯群組。 例如，管線可能包含一組內嵌和清除記錄資料的活動，然後啟動對應資料流程來分析記錄資料。 管線可讓您以集合而不是個別的方式來管理活動。 您會部署和排程管線，而不是獨立的活動。

管線中的活動會定義要在資料上執行的動作。 例如，您可以使用複製活動將資料從內部部署 SQL Server 複製到「Azure Blob 儲存體」。 然後，使用「資料流程」活動或 Databricks 筆記本活動來處理資料，並將其從 blob 儲存體轉換成 Azure Synapse 分析集區，並在其中建立商業智慧報表解決方案。

Data Factory 有三個活動群組：[資料移動活動](copy-activity-overview.md)、[資料轉換活動](transform-data.md)，以及[控制活動](control-flow-web-activity.md)。 一個活動可以接受零個或多個輸入[資料集](concepts-datasets-linked-services.md)，並且會產生一個或多個輸出[資料集](concepts-datasets-linked-services.md)。 下圖顯示 Data Factory 中管線、活動及資料集之間的關聯性：

![資料集、活動及管道之間的關聯性](media/concepts-pipelines-activities/relationship-between-dataset-pipeline-activity.png)

輸入資料集表示管線中的活動輸入，而輸出資料集表示活動的輸出。 資料集可識別資料表、檔案、資料夾和文件等各種資料存放區中的資料。 建立資料集之後，您可以將其與管線中的活動搭配使用。 例如，資料集可以是複製活動或 HDInsightHive 活動的輸入/輸出資料集。 如需有關資料集的詳細資訊，請參閱 [Azure Data Factory 中的資料集](concepts-datasets-linked-services.md)一文。

## <a name="data-movement-activities"></a>資料移動活動

Data Factory 中的複製活動會將資料從來源資料存放區複製到接收資料存放區。 Data Factory 支援本節的表格中所列的資料存放區。 可將來自任何來源的資料寫入任何接收器。 按一下資料存放區，即可了解如何將資料複製到該存放區，以及從該存放區複製資料。

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores.md)]

如需詳細資訊，請參閱[複製活動 - 概觀](copy-activity-overview.md)一文。

## <a name="data-transformation-activities"></a>資料轉換活動
Azure Data Factory 支援下列可個別或與其他活動鏈結而新增至管線的轉換活動。

資料轉換活動 | 計算環境
---------------------------- | -------------------
[資料流程](control-flow-execute-data-flow-activity.md) | Azure Databricks 由 Azure Data Factory 管理
[Azure Function](control-flow-azure-function-activity.md) | Azure Functions
[Hive](transform-data-using-hadoop-hive.md) | HDInsight [Hadoop]
[Pig](transform-data-using-hadoop-pig.md) | HDInsight [Hadoop]
[MapReduce](transform-data-using-hadoop-map-reduce.md) | HDInsight [Hadoop]
[Hadoop 串流](transform-data-using-hadoop-streaming.md) | HDInsight [Hadoop]
[Spark](transform-data-using-spark.md) | HDInsight [Hadoop]
[Machine Learning 活動︰批次執行和更新資源](transform-data-using-machine-learning.md) | Azure VM
[預存程序](transform-data-using-stored-procedure.md) | Azure SQL、Azure SQL 資料倉儲或 SQL Server
[U-SQL](transform-data-using-data-lake-analytics.md) | Azure 資料湖分析
[自訂活動](transform-data-using-dotnet-custom-activity.md) | Azure Batch
[Databricks Notebook](transform-data-databricks-notebook.md) | Azure Databricks
[Databricks Jar 活動](transform-data-databricks-jar.md) | Azure Databricks
[Databricks Python 活動](transform-data-databricks-python.md) | Azure Databricks

如需詳細資訊，請參閱[資料轉換活動](transform-data.md)一文。

## <a name="control-flow-activities"></a>控制流程活動
支援下列的控制流程活動：

控制活動 | 描述
---------------- | -----------
[附加變數](control-flow-append-variable-activity.md) | 將值新增至現有的陣列變數。
[執行管線](control-flow-execute-pipeline-activity.md) | 「執行管道」活動可讓 Data Factory 管道叫用另一個管道。
[Filter](control-flow-filter-activity.md) | 將篩選運算式套用至輸入陣列
[針對每個](control-flow-for-each-activity.md) | ForEach 活動可定義管道中重複的控制流程。 這個活動用來反覆查詢集合，並在迴圈中執行指定的活動。 此活動的迴圈實作與程式設計語言中的 Foreach 迴圈結構相似。
[取得中繼資料](control-flow-get-metadata-activity.md) | GetMetadata 活動可用來擷取 Azure Data Factory 中任何資料的中繼資料。
[If 條件活動](control-flow-if-condition-activity.md) | 「If 條件」可用於根據評估為 true 或 false 的條件來分支。 If Condition 活動所提供的功能，與 If 陳述式在程式設計語言中提供的功能相同。 它能在條件評估為 `true` 時執行一系列的活動，並在條件評估為 `false` 時執行另一系列的活動。
[查閱活動](control-flow-lookup-activity.md) | 查閱活動可用來讀取或查閱任何外部來源的記錄/表格名稱/值。 此輸出可進一步供後續的活動參考。
[設定變數](control-flow-set-variable-activity.md) | 設定現有變數的值。
[Until 活動](control-flow-until-activity.md) | 實作 Do-Until 迴圈，類似於程式設計語言中的 Do-Until 迴圈結構。 它會以迴圈的方式執行一系列活動，直到與該活動相關聯的條件評估為 true 為止。 您可以在 Data Factory 中針對 until 活動指定逾時的值。
[驗證活動](control-flow-validation-activity.md) | 確保管線只有在參考資料集存在、符合指定的準則，或已達到超時時，才會繼續執行。
[Wait 活動](control-flow-wait-activity.md) | 在管線中使用 Wait (等待) 活動時，管線便會等待一段指定的時間，然後再繼續執行後續的活動。
[Web 活動](control-flow-web-activity.md) | 使用 Web 活動可以從 Data Factory 管線呼叫自訂的 REST 端點。 您可以傳遞資料集和連結服務，以供活動使用和存取。
[Webhook 活動](control-flow-webhook-activity.md) | 使用 webhook 活動，呼叫端點並傳遞回呼 URL。 管線執行會等候回呼被叫用，然後再繼續進行下一個活動。

## <a name="pipeline-json"></a>管線 JSON
以 JSON 格式定義管道的方式如下：

```json
{
    "name": "PipelineName",
    "properties":
    {
        "description": "pipeline description",
        "activities":
        [
        ],
        "parameters": {
        },
        "concurrency": <your max pipeline concurrency>,
        "annotations": [
        ]
    }
}
```

標記 | 描述 | 在系統提示您進行確認時，輸入 | 必要
--- | ----------- | ---- | --------
名稱 | 管線的名稱。 指定代表管線所執行之動作的名稱。 <br/><ul><li>字元數目上限︰140</li><li>開頭必須為字母、數字或底線 (\_)</li><li>不允許使用下列字元：“.”、“+”、“?”、“/”、“<”、”>”、”*”、”%”、”&”、”:”、”\”</li></ul> | 字串 | yes
Description | 指定說明管線用途的文字。 | 字串 | 否
活動 | [ **活動** ] 區段內可以有一或多個已定義的活動。 如需活動 JSON 元素的詳細資料，請參閱[活動 JSON](#activity-json) 一節。 | 陣列 | yes
參數 | **parameters** 區段可以在管道內定義一或多個參數，讓管道變得更有彈性而可重複使用。 | 列出 | 否
並行 | 管線可以擁有的並存執行數目上限。 根據預設，沒有最大值。 若達到並行限制，則其他管線執行會排入佇列，直到先前的工作完成為止 | 數字 | 否 
備註 | 與管線相關聯的標記清單 | 陣列 | 否

## <a name="activity-json"></a>活動 JSON
[ **活動** ] 區段內可以有一或多個已定義的活動。 主要有兩種類型的活動：執行和控制活動。

### <a name="execution-activities"></a>執行活動
執行活動包括[資料移動活動](#data-movement-activities)和[資料轉換活動](#data-transformation-activities)。 這些活動具有下列最上層結構：

```json
{
    "name": "Execution Activity Name",
    "description": "description",
    "type": "<ActivityType>",
    "typeProperties":
    {
    },
    "linkedServiceName": "MyLinkedService",
    "policy":
    {
    },
    "dependsOn":
    {
    }
}
```

下表說明活動 JSON 定義內的屬性：

標記 | 描述 | 必要
--- | ----------- | ---------
名稱 | 活動的名稱。 指定代表活動所執行之動作的名稱。 <br/><ul><li>字元數目上限︰55</li><li>開頭必須為字母、數字或底線 (\_)</li><li>不允許使用下列字元：“.”、“+”、“?”、“/”、“<”、”>”、”*”、”%”、”&”、”:”、”\” | yes</li></ul>
Description | 說明活動用途的文字 | yes
類型 | 活動的類型。 如需了解不同類型的活動，請參閱[資料移動活動](#data-movement-activities)、[資料轉換活動](#data-transformation-activities)和[控制活動](#control-flow-activities)各節。 | yes
linkedServiceName | 活動所使用的連結服務名稱。<br/><br/>活動可能會要求您指定可連結至所需計算環境的連結服務。 | 對於 HDInsight 活動、Azure Machine Learning 批次計分活動和預存程序活動而言為必要。 <br/><br/>否：所有其他
typeProperties | typeProperties 區段中的屬性視每一種活動而定。 若要查看活動的類型屬性，請按一下先前小節中的活動連結。 | 否
原則 | 會影響活動之執行階段行為的原則。 這個屬性包含逾時和重試行為。 如果未指定，則會使用預設值。 如需詳細資訊，請參閱[活動原則](#activity-policy)一節。 | 否
dependsOn | 這個屬性用來定義活動相依性，以及後續活動如何相依於先前活動。 如需詳細資訊，請參閱[活動相依性](#activity-dependency) | 否

### <a name="activity-policy"></a>活動原則
原則會影響活動的執行階段行為，也提供可設定性選項。 「活動原則」僅適用於執行活動。

### <a name="activity-policy-json-definition"></a>活動原則 JSON 定義

```json
{
    "name": "MyPipelineName",
    "properties": {
      "activities": [
        {
          "name": "MyCopyBlobtoSqlActivity",
          "type": "Copy",
          "typeProperties": {
            ...
          },
         "policy": {
            "timeout": "00:10:00",
            "retry": 1,
            "retryIntervalInSeconds": 60,
            "secureOutput": true
         }
        }
      ],
        "parameters": {
           ...
        }
    }
}
```

JSON 名稱 | 描述 | 允許的值 | 必要
--------- | ----------- | -------------- | --------
timeout | 指定活動執行的逾時。 | Timespan | 號 預設逾時為 7 天。
retry | 重試次數上限 | 整數， | 號 預設值為 0
retryIntervalInSeconds | 重試嘗試之間的延遲 (秒) | 整數， | 號 預設值為30秒
secureOutput | 設定為 true 時，活動的輸出會被視為安全，且不會記錄到監視。 | 布林值 | 號 預設值為 false。

### <a name="control-activity"></a>控制活動
控制活動具有下列最上層結構：

```json
{
    "name": "Control Activity Name",
    "description": "description",
    "type": "<ActivityType>",
    "typeProperties":
    {
    },
    "dependsOn":
    {
    }
}
```

標記 | 描述 | 必要
--- | ----------- | --------
名稱 | 活動的名稱。 指定代表活動所執行之動作的名稱。<br/><ul><li>字元數目上限︰55</li><li>開頭必須為字母、數字或底線 (\_)</li><li>不允許使用下列字元：“.”、“+”、“?”、“/”、“<”、”>”、”*”、”%”、”&”、”:”、”\” | yes</li><ul>
Description | 說明活動用途的文字 | yes
類型 | 活動的類型。 如需了解不同類型的活動，請參閱[資料移動活動](#data-movement-activities)、[資料轉換活動](#data-transformation-activities)和[控制活動](#control-flow-activities)各節。 | yes
typeProperties | typeProperties 區段中的屬性視每一種活動而定。 若要查看活動的類型屬性，請按一下先前小節中的活動連結。 | 否
dependsOn | 這個屬性用來定義活動相依性，以及後續活動如何相依於先前活動。 如需詳細資訊，請參閱[活動相依性](#activity-dependency)。 | 否

### <a name="activity-dependency"></a>活動相依性
「活動相依性」可定義後續活動如何相依於先前活動，根據條件以決定是否繼續執行下一項工作。 一個活動可以根據不同的相依性條件，而相依於一或多個先前活動。

各種相依性條件包括：成功、失敗、略過、完成。

例如，如果管道有活動 A -> 活動 B，則可能發生的各種情節如下：

- 活動 B 對於活動 A 的相依性條件為**成功**：只有當活動 A 的最終狀態為成功時，活動 B 才會執行
- 活動 B 對於活動 A 的相依性條件為**失敗**：只有當活動 A 的最終狀態為失敗時，活動 B 才會執行
- 活動 B 對於活動 A 的相依性條件為**完成**：如果活動 A 的最終狀態為成功或失敗，活動 B 會執行
- 活動 B 對於活動 A 的相依性條件為**略過**：如果活動 A 的最終狀態為略過，活動 B 會執行。 「略過」發生於活動 X -> 活動 Y -> 活動 Z 的情節中，每個活動只有在前一個活動成功時才會執行。 如果活動 X 失敗，則活動 Y 的狀態為「略過」，因為永遠不會執行。 同樣地，活動 Z 的狀態也是「略過」。

#### <a name="example-activity-2-depends-on-the-activity-1-succeeding"></a>範例：活動 2 相依於活動 1 成功

```json
{
    "name": "PipelineName",
    "properties":
    {
        "description": "pipeline description",
        "activities": [
         {
            "name": "MyFirstActivity",
            "type": "Copy",
            "typeProperties": {
            },
            "linkedServiceName": {
            }
        },
        {
            "name": "MySecondActivity",
            "type": "Copy",
            "typeProperties": {
            },
            "linkedServiceName": {
            },
            "dependsOn": [
            {
                "activity": "MyFirstActivity",
                "dependencyConditions": [
                    "Succeeded"
                ]
            }
          ]
        }
      ],
      "parameters": {
       }
    }
}

```

## <a name="sample-copy-pipeline"></a>範例複製管線
在以下的範例管線中， **Copy** in the **活動** 類型的活動。 在此範例中，[複製活動](copy-activity-overview.md)會將資料從 Azure Blob 儲存體複製到 Azure SQL 資料庫。

```json
{
  "name": "CopyPipeline",
  "properties": {
    "description": "Copy data from a blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "type": "Copy",
        "inputs": [
          {
            "name": "InputDataset"
          }
        ],
        "outputs": [
          {
            "name": "OutputDataset"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink",
            "writeBatchSize": 10000,
            "writeBatchTimeout": "60:00:00"
          }
        },
        "policy": {
          "retry": 2,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
請注意下列幾點：

- 在活動區段中，只會有一個 **type** 設為 **Copy** 的活動。
- 活動的輸入設定為 **InputDataset**，活動的輸出則設定為 **OutputDataset**。 若要了解如何以 JSON 定義資料集，請參閱[資料集](concepts-datasets-linked-services.md)一文。
- 在 **typeProperties** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。 在[資料移動活動](#data-movement-activities)區段中，按一下要作為來源或接收的資料存放區，以深入了解如何將資料移入/移出該資料存放區。

如需有關建立此管道的完整逐步解說，請參閱[快速入門：建立資料處理站](quickstart-create-data-factory-powershell.md)。

## <a name="sample-transformation-pipeline"></a>範例轉換管線
在以下的範例管線中， **CopyActivity** in the **活動** 類型的活動。 在此範例中， [HDInsight Hive 活動](transform-data-using-hadoop-hive.md) 會執行 Azure HDInsight Hadoop 叢集上的 Hive 指令碼檔案，來轉換 Azure Blob 儲存體的資料。

```json
{
    "name": "TransformPipeline",
    "properties": {
        "description": "My first Azure Data Factory pipeline",
        "activities": [
            {
                "type": "HDInsightHive",
                "typeProperties": {
                    "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                    "scriptLinkedService": "AzureStorageLinkedService",
                    "defines": {
                        "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                        "partitionedtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/partitioneddata"
                    }
                },
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "policy": {
                    "retry": 3
                },
                "name": "RunSampleHiveActivity",
                "linkedServiceName": "HDInsightOnDemandLinkedService"
            }
        ]
    }
}
```
請注意下列幾點：

- 在活動區段中，只會有一個 **type** 設為 **HDInsightHive** 的活動。
- Hive 指令碼檔案 **partitionweblogs.hql** 儲存於 Azure 儲存體帳戶 (由名為 AzureStorageLinkedService 的 scriptLinkedService 指定)，且位於 `adfgetstarted` 容器的 script 資料夾中。
- `defines` 區段可用來指定執行階段設定，這些設定會傳遞給 Hive 指令碼作為 Hive 設定值 (例如，$`{hiveconf:inputtable}`、`${hiveconf:partitionedtable}`)。

每個轉換活動的 **typeProperties** 區段都不同。 若要了解轉換活動支援的 type 屬性，請在[資料轉換活動](#data-transformation-activities)中按一下該轉換活動。

如需有關建立此管道的完整逐步解說，請參閱[教學課程：使用 Spark 轉換資料](tutorial-transform-data-spark-powershell.md)。

## <a name="multiple-activities-in-a-pipeline"></a>管線中的多個活動
前兩個範例管線都只包含一個活動。 您可以在一個管線中包含多個活動。 如果您在管道中有多個活動，而且後續活動不相依於先前活動，則活動可以平行執行。

您可以使用[活動相依性](#activity-dependency)來鏈結兩個活動，以定義後續活動如何相依於先前活動，根據條件以決定是否繼續執行下一項工作。 一個活動可以根據不同的相依性條件，而相依於一或多個先前活動。

## <a name="scheduling-pipelines"></a>排程管道
管道由觸發程序排程。 有各種不同類型的觸發程序 (排程器觸發程序：可依時鐘排程來觸發管道；手動觸發程序：依需求觸發管道)。 如需觸發程序的詳細資訊，請參閱[管道執行和觸發程序](concepts-pipeline-execution-triggers.md)一文。

若要讓觸發程序啟動管道執行，您必須在觸發程序定義中包含特定管道的管道參考。 管道和觸發程序具有 n-m 關聯性。 多個觸發程序可以啟動單一管道，而相同的觸發程序可以啟動多個管道。 定義觸發程序之後，您必須啟動觸發程序，才會開始觸發管道。 如需觸發程序的詳細資訊，請參閱[管道執行和觸發程序](concepts-pipeline-execution-triggers.md)一文。

例如，假設您有一個排程器觸發程式「觸發 A」，我想要啟動管線「MyCopyPipeline」。 您可以如下列範例所示來定義觸發程序：

### <a name="trigger-a-definition"></a>觸發程序 A 定義

```json
{
  "name": "TriggerA",
  "properties": {
    "type": "ScheduleTrigger",
    "typeProperties": {
      ...
      }
    },
    "pipeline": {
      "pipelineReference": {
        "type": "PipelineReference",
        "referenceName": "MyCopyPipeline"
      },
      "parameters": {
        "copySourceName": "FileSource"
      }
    }
  }
}
```



## <a name="next-steps"></a>後續步驟
請參閱下列教學課程中的逐步指示，以建立具有活動的管道：

- [建置具有複製活動的管道](quickstart-create-data-factory-powershell.md)
- [使用資料轉換活動來建置管線](tutorial-transform-data-spark-powershell.md)
