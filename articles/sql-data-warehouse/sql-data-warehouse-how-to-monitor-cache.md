---
title: 優化您的 Gen2 快取
description: 了解如何使用 Azure 入口網站監視 Gen2 快取。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.subservice: performance
ms.topic: conceptual
ms.date: 09/06/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 924705b7ce1d324583077797714491bdf3fc5cc9
ms.sourcegitcommit: f52ce6052c795035763dbba6de0b50ec17d7cd1d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/24/2020
ms.locfileid: "76721212"
---
# <a name="how-to-monitor-the-gen2-cache"></a>如何監視 Gen2 快取
在針對 Gen2 資料倉儲所設計的 NVMe 型 SSD 上，Gen2 儲存體架構會自動在位於其中的快取內將最常查詢的資料行存放區區段分層。 當查詢擷取位於快取中的區段時，將可獲得更好的效能。 本文說明如何藉由判斷工作負載是否有充分利用 Gen2 快取，來監視緩慢的查詢效能並加以疑難排解。  
## <a name="troubleshoot-using-the-azure-portal"></a>使用 Azure 入口網站進行疑難排解
您可以使用 Azure 監視器來檢視 Gen2 快取計量，以針對查詢效能進行疑難排解。 請先移至 Azure 入口網站，然後按一下監視器：

![Azure 監視器](./media/sql-data-warehouse-cache-portal/cache_0.png)

選取 [計量] 按鈕，並填入資料倉儲的**訂**用帳戶、**資源** **群組**、**資源類型**和**資源名稱**。

用於疑難排解 Gen2 快取的關鍵計量是 [快取命中百分比] 和 [已用快取百分比]。 請設定 Azure 計量圖表，以顯示這兩個計量。

![快取計量](./media/sql-data-warehouse-cache-portal/cache_1.png)


## <a name="cache-hit-and-used-percentage"></a>快取命中的百分比和已用快取的百分比
以下矩陣描述以快取計量值為基礎的案例：

|                                | **高快取命中百分比** | **低快取命中百分比** |
| :----------------------------: | :---------------------------: | :--------------------------: |
| **高已用快取百分比** |          實例 1           |          案例 2          |
| **低已用快取百分比**  |          案例 3           |          案例 4          |

**案例 1：** 您充分使用快取。 針對可能會減緩查詢速度的其他區域[進行疑難排解](sql-data-warehouse-manage-monitor.md)。

**案例 2：** 您目前的工作資料集不適合該快取，導致由於實體讀取次數的關係而產生低快取命中百分比。 請考慮相應增加效能層級，然後重新執行工作負載來填入快取。

**案例 3：** 查詢可能是因為與快取無關的理由而導致執行速度緩慢。 針對可能會減緩查詢速度的其他區域[進行疑難排解](sql-data-warehouse-manage-monitor.md)。 您也可以考慮[相應減少執行個體](sql-data-warehouse-manage-monitor.md)來降低快取大小，以節省成本。 

**案例 4：** 您擁有冷快取，這可能是查詢速度緩慢的理由。 工作資料集現在應該已完成快取，所以請考慮重新執行查詢。 

> [!IMPORTANT]
> 如果重新執行您的工作負載之後，快取命中百分比或快取使用百分比未更新，則您的工作集可能已經位於記憶體中。 只會快取叢集資料行存放區資料表。

## <a name="next-steps"></a>後續步驟
如需一般查詢效能微調的詳細資訊，請參閱[監視查詢的執行](../sql-data-warehouse/sql-data-warehouse-manage-monitor.md#monitor-query-execution)。
