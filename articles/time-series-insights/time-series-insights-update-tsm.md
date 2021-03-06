---
title: 時間序列模型-Azure 時間序列深入解析 |Microsoft Docs
description: 瞭解 Azure 時間序列深入解析 Preview 中的時間序列模型。
author: deepakpalled
ms.author: dpalled
manager: cshankar
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 02/14/2020
ms.custom: seodec18
ms.openlocfilehash: 884244b245be06f1477d27a4828cad18e36eca24
ms.sourcegitcommit: f97f086936f2c53f439e12ccace066fca53e8dc3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/15/2020
ms.locfileid: "77368629"
---
# <a name="time-series-model-in-azure-time-series-insights-preview"></a>Azure 時間序列深入解析預覽中的時間序列模型

本文描述時間序列模型、功能，以及如何開始在 Azure 時間序列深入解析預覽環境中建立及更新您自己的模型。

> [!TIP]
>  * 移至 [Contoso 風伺服器陣列示範](https://insights.timeseries.azure.com/preview/samples)環境，以取得即時時間序列模型範例。
> * 閱讀[Azure 時間序列深入解析 Preview explorer](time-series-insights-update-explorer.md)以瞭解如何流覽您的時間序列模型 UI。
> * 瞭解如何使用時間序列深入解析 web explorer[來處理時間序列模型](time-series-insights-update-how-to-tsm.md)。

## <a name="summary"></a>摘要

傳統上，從 IoT 裝置收集的資料缺發關聯的資訊，因此很難快速地尋找及分析感應器。 時間序列模型的主要動機是簡化尋找和分析 IoT 或時間序列資料的程式。 它可讓您鑒藏、維護和擴充時間序列資料，以協助準備取用者準備好用於分析的資料集，達到此目標。

## <a name="scenario-contosos-new-smart-oven"></a>案例： Contoso 的新智慧型 oven

**請考慮 Contoso 智慧型 oven 的虛構案例。** 在此案例中，假設每個 Contoso 智慧型 oven 有五個溫度感應器，分別用於四個熱門燒錄機，另一個用於 oven 本身。 直到最近，每個 Contoso 溫度感應器會個別傳送、儲存及視覺化其資料。 針對其廚房設備監視，Contoso 依賴基本圖表，每個個別感應器各一個。

雖然 Contoso 對其初始資料和視覺效果解決方案感到滿意，但有幾項限制變得很明顯：

* 客戶想要知道當大部分最熱門的燒錄機已開啟時，整體 oven 會得到的熱程度。 Contoso 在分析和呈現整體 oven 條件的整合解答時，比較困難。
* Contoso 工程師想要確認同時執行的熱門燒錄機不會導致電力繪製效率不佳。 交叉參照哪些溫度和電壓感應器彼此相關，以及如何在存放區中找到它們，是很困難的。
* Contoso 品質保證小組想要對兩個感應器版本之間的歷程記錄進行審核和比較。 判斷哪些資料屬於哪個感應器版本很困難。

若沒有結構、組織及定義整體智慧型 oven 時間序列模型的功能，每個溫度感應器都會維護 dislocated、隔離且更具資訊性的資料點。 將這些資料點轉換成可採取動作的深入解析變得更棘手，因為每個資料集的存留期都獨立。

這些限制揭示了 Contoso 新 oven 的智慧型資料匯總和視覺化檢視的重要性：

* 當您能夠建立關聯並將資料結合成方便的視圖時，資料視覺效果證明會很有用。 其中一個範例顯示電壓感應器和溫度感應器。
* 管理數個實體的多維度資料以及比較、縮放和時間範圍功能，可能會很容易完成。

**時間序列模型**會針對在此虛構範例中遇到的許多案例提供便利的解決方案：

[![時間序列模型智慧型 oven 圖表範例](media/v2-update-tsm/time-series-model-smart-oven.png)](media/v2-update-tsm/time-series-model-smart-oven.png#lightbox)

* 時間序列模型在查詢和導覽中扮演重要的角色，因為它可讓您跨時間範圍以及感應器與裝置種類之間繪製比較，藉此就資料。 （**A**） 
* 資料會進一步內容相關，因為時間序列模型中保存的資料會將時間序列查詢計算保留為變數，並在查詢時重複使用它們。
* 時間序列模型會組織並匯總資料，以改善視覺效果和管理功能。 （**B**） 

### <a name="key-capabilities"></a>主要功能

為了達到簡單輕鬆就能管理時間序列關聯化的目的，時間序列模型在時間序列深入解析中提供下列功能。 它可協助您：

* 撰寫和管理利用純量函數、匯總作業等等的計算或公式。
* 定義父子式關聯性，以啟用導覽、搜尋和參考。
* 定義與實例相關聯的屬性（定義為*實例欄位*），並使用它們來建立階層。

### <a name="components"></a>Components

時間序列模型有三個核心元件：

* [時間序列模型實例](#time-series-model-instances)
* [時間序列模型階層](#time-series-model-hierarchies)
* [時間序列模型類型](#time-series-model-types)

這些元件會結合以指定時間序列模型，並組織您的 Azure 時間序列深入解析資料。

[![時間序列模型總覽圖表](media/v2-update-tsm/time-series-model-overview.png)](media/v2-update-tsm/time-series-model-overview.png#lightbox)

您可以透過[時間序列深入解析 Preview](time-series-insights-update-how-to-tsm.md)介面來建立和管理時間序列模型。 時間序列模型設定可以透過[模型設定 API](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#model-settings-api)來管理。

## <a name="time-series-model-instances"></a>時間序列模型執行個體

時間序列模型*實例*是時間序列本身的虛擬標記法。

在大部分的情況下，實例是以**deviceId**或**assetId**唯一識別，並儲存為時間序列識別碼。

實例具有相關聯的描述性資訊，稱為*實例屬性*，例如時間序列識別碼、類型、名稱、描述、階層和實例欄位。 執行個體屬性至少會包含階層資訊。

*實例欄位*是描述性資訊的集合，其中可以包含階層層級的值，以及製造商、運算子等等。

為時間序列深入解析環境設定事件來源之後，系統會自動探索實例並在時間序列模型中建立。 您可以使用時間序列模型查詢，透過時間序列深入解析 explorer 來建立或更新實例。

[Contoso 風伺服器陣列示範](https://insights.timeseries.azure.com/preview/samples)提供數個即時實例範例。

[![時間序列模型實例範例](media/v2-update-tsm/time-series-model-instance.png)](media/v2-update-tsm/time-series-model-instance.png#lightbox)

### <a name="instance-properties"></a>實例屬性

實例是由**timeSeriesId**、 **typeId**、 **name**、 **description**、 **hierarchyIds**和**instancefields 定義**所定義。 每個實例只會對應至一個*類型*，以及一個或多個*階層。*

| 屬性 | 描述 |
| --- | ---|
| timeSeriesId | 與實例相關聯之時間序列的 UUID。 |
| typeId | 與實例相關聯之時間序列模型類型的 UUID。 根據預設，所有探索到的新實例都會與預設類型建立關聯。
| 名稱 | **Name**屬性是選擇性的，而且區分大小寫。 如果 [**名稱**] 無法使用，則預設為 [ **timeSeriesId**]。 如果[提供了名稱](time-series-insights-update-explorer.md#4-time-series-well)， **timeSeriesId**仍然可供使用。 |
| description | 實例的文字描述。 |
| hierarchyIds | 定義實例所屬的階層。 |
| Instancefields 定義 | 實例的屬性，以及定義實例的任何靜態資料。 它們定義階層或非階層的值，同時也支援建立索引以執行搜尋作業。 |

> [!NOTE]
> 階層是使用實例欄位所建立。 可以針對進一步的實例屬性定義加入額外的**instancefields 定義**。

實例具有下列 JSON 標記法：

```JSON
{
  "timeSeriesId": ["PU2"],
  "typeId": "545314a5-7166-4b90-abb9-fd93966fa39b",
  "hierarchyIds": ["95f0a8d1-a3ef-4549-b4b3-f138856b3a12"],
  "description": "Pump #2",
  "instanceFields": {
    "Location": "Redmond",
    "Fleet": "Fleet 5",
    "Unit": "Pump Unit 3",
    "Manufacturer": "Contoso",
    "ScalePres": "0.54",
    "scaleTemp": "0.54"
  }
}
```

> [!TIP]
> 如需時間序列深入解析實例 API 和建立、讀取、更新和刪除（CRUD）支援，請閱讀[資料查詢](time-series-insights-update-tsq.md#time-series-model-query-tsm-q-apis)文章和[實例 API REST 檔](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#instances-api)。

## <a name="time-series-model-hierarchies"></a>時間序列模型階層

時間*序列模型階層*會藉由指定屬性名稱及其關聯性來組織實例。

您可以在指定的時間序列深入解析環境中設定多個階層。 時間序列模型實例可以對應到單一階層或多個階層（多對多關聯性）。

[Contoso 風伺服器陣列示範](https://insights.timeseries.azure.com/preview/samples)用戶端介面會顯示標準實例和類型階層。

[![時間序列模型階層範例](media/v2-update-tsm/time-series-model-hierarchies.png)](media/v2-update-tsm/time-series-model-hierarchies.png#lightbox)

### <a name="hierarchy-definition"></a>階層定義

階層是由階層**識別碼**、**名稱**和**來源**所定義。

| 屬性 | 描述 |
| ---| ---|
| id | 階層的唯一識別碼，例如，當您定義實例時使用。 |
| 名稱 | 用來提供階層名稱的字串。 |
| 來源 | 指定組織階層或路徑，這是使用者想要建立之階層的由上而下的父子式順序。 父子式屬性會對應實例欄位。 |

階層會以 JSON 表示為：

```JSON
{
  "hierarchies": [
    {
      "id": "6e292e54-9a26-4be1-9034-607d71492707",
      "name": "Location",
      "source": {
        "instanceFieldNames": [
          "state",
          "city"
        ]
      }
    },
    {
      "id": "a28fd14c-6b98-4ab5-9301-3840f142d30e",
      "name": "ManufactureDate",
      "source": {
        "instanceFieldNames": [
          "year",
          "month"
        ]
      }
    }
  ]
}
```

在先前的 JSON 範例中：

* `Location` 會定義具有父系 `states` 和子 `cities`的階層。 每個 `location` 都可以有多個 `states`，而這可以有多個 `cities`。
* `ManufactureDate` 會定義具有父系 `year` 和子 `month`的階層。 每個 `ManufactureDate` 都可以有多個 `years`，而這可以有多個 `months`。

> [!TIP]
> 如需時間序列深入解析實例 API 和 CRUD 支援，請閱讀[資料查詢](time-series-insights-update-tsq.md#time-series-model-query-tsm-q-apis)文章和階層[API REST 檔](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#hierarchies-api)。

### <a name="hierarchy-example"></a>階層範例

假設有一個範例，其中的階層**H1**已 `building`、`floor`和 `room` 作為其**instanceFieldNames**定義的一部分：

```JSON
{
  "id": "aaaaaa-bbbbb-ccccc-ddddd-111111",
  "name": "H1",
  "source": {
    "instanceFieldNames": [
      "building",
      "floor",
      "room"
    ]
  }
}
```

假設先前定義中使用的實例欄位和數個時間序列中，階層屬性和值如下表所示：

| 時間序列識別碼 | 執行個體欄位 |
| --- | --- |
| ID1 | "building" = "1000", "floor" = "10", "room" = "55"  |
| ID2 | "building" = "1000", "room" = "55" |
| ID3 | "floor" = "10" |
| ID4 | "building" = "1000", "floor" = "10"  |
| ID5 | 未設定任何「建築物」、「樓層」或「房間」。 |

時間序列**ID1**和**ID4**會在[Azure 時間序列深入解析 Explorer](time-series-insights-update-explorer.md)中顯示為階層**H1**的一部分，因為它們具有完整定義且正確排序的*建築物*、*樓層*和*房間*參數。

其他專案會在*無上層實例*之下分類，因為它們不符合指定的資料層次。

## <a name="time-series-model-types"></a>時間序列模型類型

時間序列模型「類型」協助您定義執行計算的變數或公式。 類型與特定的時間序列深入解析實例相關聯。

一個類型可以有一或多個變數。 例如，時間序列模型實例可能屬於*溫度感應器*類型，這是由變數的*平均*溫度、*最小溫度*和*最大溫度*所組成。

[Contoso 風伺服器陣列示範](https://insights.timeseries.azure.com/preview/samples)會將數個與個別實例相關聯的時間序列模型類型視覺化。

[![時間序列模型類型範例](media/v2-update-tsm/time-series-model-types.png)](media/v2-update-tsm/time-series-model-types.png#lightbox)

> [!TIP]
> 如需時間序列深入解析實例 API 和 CRUD 支援，請閱讀[資料查詢](time-series-insights-update-tsq.md#time-series-model-query-tsm-q-apis)文章和[類型 API REST 檔](https://docs.microsoft.com/rest/api/time-series-insights/preview-model#types-api)。

### <a name="type-properties"></a>類型屬性

時間序列模型類型是由**識別碼**、**名稱**、**描述**和**變數**所定義。

| 屬性 | 描述 |
| ---| ---|
| id | 類型的 UUID。 |
| 名稱 | 用來提供型別名稱的字串。 |
| description | 類型的字串描述。 |
| 變數 | 指定與類型相關聯的變數。 |

類型符合下列 JSON 範例：

```JSON
{
  "types": [
    {
      "id": "1be09af9-f089-4d6b-9f0b-48018b5f7393",
      "name": "DefaultType",
      "description": "Default type",
      "variables": {
        "EventCount": {
          "kind": "aggregate",
          "value": null,
          "filter": null,
          "aggregation": {
            "tsx": "count()"
          }
        },
        "Interpolated Speed": {
          "kind": "numeric",
          "value": {
              "tsx": "$event.[speed].Double"
          },
          "filter": null,
          "interpolation": {
              "kind": "step",
              "boundary": {
                  "span": "P1D"
              }
          },
          "aggregation": {
              "tsx": "left($value)"
          }
        }
      }
    }
  ]
}
```

### <a name="variables"></a>變數

時間序列深入解析類型可能有許多變數，可指定事件的公式和計算規則。

每個變數都可以是下列三*種類型*之一：*數值*、*類別*和*匯總*。

* **數值**類型會使用連續的值。 
* **類別**類型會使用一組已定義的離散值。
* **匯總**值結合了單一種類的多個變數（全部為數值或所有類別）。

下表顯示每個變數種類的相關屬性。

[![時間序列模型變數資料表](media/v2-update-tsm/time-series-model-variable-table.png)](media/v2-update-tsm/time-series-model-variable-table.png#lightbox)

#### <a name="numeric-variables"></a>數值變數

| Variable 屬性 | 描述 |
| --- | ---|
| 變數篩選條件 | 篩選是選擇性的條件子句，可限制要考慮計算的資料列數目。 |
| 變數值 | 用於計算來自裝置或感應器，或使用時間序列運算式轉換的遙測值。 數數值型別變數必須是*Double*類型。|
| 變數插補 | 插補會指定如何使用現有的資料來重建信號。 [*步驟*] 和 [*線性*插補] 選項適用于數值變數。 |
| 變數彙總 | 支援透過*Avg*、 *min*、 *max*、 *sum*、 *Count*、 *First*、 *Last*和 time-加權（*Avg*、 *Min*、 *Max*、 *sum*、 *Left*）運算子進行計算。 |

變數符合下列 JSON 範例：

```JSON
"Interpolated Speed": {
  "kind": "numeric",
  "value": {
    "tsx": "$event.[speed].Double"
  },
  "filter": null,
  "interpolation": {
    "kind": "step",
    "boundary": {
      "span": "P1D"
    }
  },
  "aggregation": {
    "tsx": "left($value)"
  }
}
```

#### <a name="categorical-variables"></a>類別變數

| Variable 屬性 | 描述 |
| --- | ---|
| 變數篩選條件 | 篩選是選擇性的條件子句，可限制要考慮計算的資料列數目。 |
| 變數值 | 用於計算來自裝置或感應器的遙測值。 類別類型變數必須是*Long*或*String*。 |
| 變數插補 | 插補會指定如何使用現有的資料來重建信號。 [*步驟*內插補點] 選項適用于類別變數。 |
| 變數類別目錄 | 類別會在來自裝置或感應器的值與標籤之間建立對應。 |
| 變數預設分類 | 預設類別是針對未在 [分類] 屬性中對應的所有值。 |

變數符合下列 JSON 範例：

```JSON
"Status": {
  "kind": "categorical",
  "value": {
     "tsx": "toLong($event.[Status].Double)" 
},
  "interpolation": {
    "kind": "step",
    "boundary": {
      "span" : "PT1M"
    }
  },
  "categories": [
    {
      "values": [0, 1, 2, 3],
      "label": "Good"
    },
    {
      "values": [3],
      "label": "Bad"
    }
  ],
  "defaultCategory": {
    "label": "Not Applicable"
  }
}
```

#### <a name="aggregate-variables"></a>匯總變數

| Variable 屬性 | 描述 |
| --- | ---|
| 變數篩選條件 | 篩選是選擇性的條件子句，可限制要考慮計算的資料列數目。 |
| 變數彙總 | 支援透過*Avg*、 *Min*、 *Max*、 *Sum*、 *Count*、 *First*、 *Last*進行計算。 |

變數符合下列 JSON 範例：

```JSON
"Aggregate Speed": {
  "kind": "aggregate",
  "filter": null,
  "aggregation": {
    "tsx": "avg($event.Speed.Double)"
  }
}
```

變數會儲存在時間序列模型的類型定義中，而且可以透過[查詢 api](time-series-insights-update-tsq.md)以內嵌方式提供，以覆寫儲存的定義。

## <a name="next-steps"></a>後續步驟

- 閱讀[Azure 時間序列深入解析預覽儲存體和](./time-series-insights-update-storage-ingress.md)輸入。

- 瞭解[Azure 時間序列深入解析 Preview 中資料模型](./time-series-insights-update-how-to-tsm.md)化的一般時間序列模型作業

- 閱讀新的[時間序列模型](https://docs.microsoft.com/rest/api/time-series-insights/preview-model)參考檔。
