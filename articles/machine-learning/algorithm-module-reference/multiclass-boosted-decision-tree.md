---
title: 多元促進式決策樹：模組參考
titleSuffix: Azure Machine Learning
description: 瞭解如何在 Azure Machine Learning 中使用多元促進式決策樹模組，以使用加上標籤的資料來建立分類器。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/19/2019
ms.openlocfilehash: 0bcca16bd89781428773eda168e6ee3c2f5784ef
ms.sourcegitcommit: 812bc3c318f513cefc5b767de8754a6da888befc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/12/2020
ms.locfileid: "77152171"
---
# <a name="multiclass-boosted-decision-tree"></a>多元促進式決策樹

本文說明 Azure Machine Learning 設計工具（預覽）中的模組。

使用此模組來建立以促進式決策樹演算法為基礎的機器學習模型。

促進式決策樹是一個集團學習方法，其中第二個樹狀結構會針對第一個樹狀結構的錯誤修正，第三個樹狀目錄會更正第一個和第二個樹系的錯誤，依此類推。 預測是以樹狀結構的集團為基礎。

## <a name="how-to-configure"></a>如何設定 

此模組會建立未定型的分類模型。 因為分類是受監督的學習方法，所以您需要一個加上標籤的*資料集*，其中包含所有資料列的值。

您可以使用[訓練模型](././train-model.md)來定型這種類型的模型。 

1.  將多元促進式**決策樹**模組新增至您的管線。

1.  藉由設定 [**建立定型模式]** 選項，指定您要如何訓練模型。

    + **單一參數**：如果您知道要如何設定模型，可以提供一組特定值做為引數。


    *  **每個樹狀結構的葉數上限**限制可以在任何樹狀結構中建立的終端機節點數目上限（葉子）。
    
        藉由增加此值，您可能會增加樹狀結構的大小，並達到更高的精確度，但有過度學習和較長定型時間的風險。
  
    * [**每個分葉節點的樣本數下限**] 表示在樹狀結構中建立任何終端節點（分葉）所需的案例數目。  

         藉由增加此值，您會增加建立新規則的臨界值。 例如，若預設值是 1，即使單一案例可能會造成新規則的建立。 如果您將此值增加為 5，則定型資料必須至少包含五個符合相同條件的案例。

    * 學習**速率**會定義學習時的步驟大小。 請輸入介於0和1之間的數位。

         學習速率會決定學習模組在最佳解決方案上的快速或緩慢。 如果步驟大小太大，您可能會下限最佳的解決方案。 如果步驟大小太小，訓練會花更長的時間在最佳解決方案上融合。

    * [**樹狀結構數**] 表示要在集團中建立的決策樹總數。 藉由建立多個決策樹，您或許能夠有較佳的涵蓋範圍，但是定型時間會拉長。

    *  **亂數字種子**可選擇性地設定非負整數，做為隨機種子值使用。 指定種子可確保跨回合的重現性具有相同的資料和參數。  

         隨機種子預設會設定為42。 使用不同隨機種子的後續執行可能會有不同的結果。

> [!Note]
> 如果您將 [**建立定型模式**] 設定為 [**單一參數**]，請連接已標記的資料集和 [[訓練模型](./train-model.md)] 模組。

## <a name="next-steps"></a>後續步驟

請參閱可用來 Azure Machine Learning 的[模組集合](module-reference.md)。 
