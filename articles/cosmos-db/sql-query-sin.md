---
title: Azure Cosmos DB 查詢語言中的 SIN
description: 瞭解 Azure Cosmos DB 中的 SQL 系統函數 SIN。
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: b4450ea9dd50f8093fa9569d7f82fe124022c3a1
ms.sourcegitcommit: 7f6d986a60eff2c170172bd8bcb834302bb41f71
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2019
ms.locfileid: "71349475"
---
# <a name="sin-azure-cosmos-db"></a>SIN （Azure Cosmos DB）
 在指定運算式中傳回指定角度的三角正弦函數 (以弧度為單位)。  
  
## <a name="syntax"></a>語法
  
```sql
SIN(<numeric_expr>)  
```  
  
## <a name="arguments"></a>引數
  
*numeric_expr*  
   為數值運算式。  
  
## <a name="return-types"></a>傳回類型
  
  傳回數值運算式。  
  
## <a name="examples"></a>範例
  
  下列範例會計算指定角度的 `SIN`。  
  
```sql
SELECT SIN(45.175643) AS sin  
```  
  
 以下為結果集。  
  
```json
[{"sin": 0.929607286611012}]  
```  

## <a name="next-steps"></a>後續步驟

- [數學函數 Azure Cosmos DB](sql-query-mathematical-functions.md)
- [系統函數 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 簡介](introduction.md)
