---
title: Azure Marketplace SEO 指引
description: 提供有關如何最大化搜尋引擎最佳化 (SEO) 的指導方針。
services: Azure, Marketplace, Cloud Partner Portal,
author: v-miclar
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 04/09/2019
ms.author: pabutler
ms.openlocfilehash: 7115798faadc3209413d22a384433417ec0ddff0
ms.sourcegitcommit: ac56ef07d86328c40fed5b5792a6a02698926c2d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/08/2019
ms.locfileid: "73819574"
---
# <a name="azure-marketplace-seo-guidance"></a>Azure Marketplace SEO 指引

本文說明如何透過[Azure Marketplace](https://azuremarketplace.microsoft.com)和[AppSource](https://appsource.microsoft.com)中的搜尋功能，將供應專案的探索能力最大化。 


## <a name="general-explanation-of-algorithm"></a>演算法的一般說明

Microsoft marketplace 利用 Azure 認知搜尋來加強網站的搜尋功能。 演算法是以詞彙頻率–反向文件頻率 ([TF-IDF](https://en.wikipedia.org/wiki/Tf–idf)) 為基礎。 將會使用標準 [Lucene 分析器](https://lucene.apache.org/core/)。

一般而言，所有文字欄位、類別與產業都會納入相關性的權重中。 應用程式不常使用但您的應用程式經常使用的特殊化詞彙將會產生較高的搜尋符合分數。 因此，納入 "VM" 之類的詞彙將提供一些助益，而納入 "Azure search" 則更為特殊化。
下面是要考慮的最相關欄位。

 
|  欄位                   | 重要性 | 指引                                                                                            |
|  --------------------    | ----------                   | ---------------                                                                   |
| 供應項目名稱               |  高      | 精確相符或近乎相符的搜尋查詢將會獲得高排名。                       |
| 發行者名稱           |  高      | 精確相符或近乎相符的搜尋查詢將會獲得高排名。                       |
| 簡短描述        |  中型    | 應用程式命名與發行者名稱幾乎能保證高排名，但可能不是最相關。 在此案例中，簡短描述非常重要。 讓文字簡潔並中肯。 應該納入關鍵字語預期搜尋詞彙，以獲得最佳結果。  例如，「這是完全以 Dynamics 365 為基礎而建置的零售 POS」將不如「Dynamics 365 的零售 POS (銷售點)」有效。  | 
| 完整描述         |  低       | 描述提供讓人更深入探索的動機。 最有效的描述必須有合理的長度，而且必須使用關鍵字。  使用關鍵字的中肯描述比冗長的文字好。 確定描述中有關鍵詞彙 (例如 "IoT")。  |
| 產品類別       | 中型     |  產品類別是由發行者選擇與 Microsoft 的結合所判斷。 適當地選取這些類別，以便使用者可以在正確的類別中尋找應用程式。 |
|  |  |  |


## <a name="other-tips"></a>其他祕訣

-   搜尋建議可取得大量使用者活動。 它會將應用程式名稱/發行者的符合程度優先順序提高最高。 當搜尋字詞不完全與發行者/應用程式名稱相符時，簡短描述便成為關鍵欄位。
-   供下載的文件未包含在搜尋權重中。
-   實際取得及使用您的應用程式也會影響搜尋排名。 例如，有兩個類似的應用程式，擁有較多使用者的應用程式將會取得較高的排名。
