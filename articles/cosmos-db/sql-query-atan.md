---
title: Azure Cosmos DB 查詢語言中的 ATAN
description: 瞭解 Azure Cosmos DB 中的反正切（ATAN） SQL 系統函式如何傳回以弧度為單位的角度，其正切函數是指定的數值運算式
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 8fd398e343a3e617ceb9f49ca8f31ee776a7f073
ms.sourcegitcommit: 9405aad7e39efbd8fef6d0a3c8988c6bf8de94eb
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/05/2019
ms.locfileid: "74873449"
---
# <a name="atan-azure-cosmos-db"></a>ATAN （Azure Cosmos DB）
 傳回角度，以弧度為單位，其正切函數是指定的數值運算式。 這也稱為反正切值。  
  
## <a name="syntax"></a>語法
  
```sql
ATAN(<numeric_expr>)  
```  
  
## <a name="arguments"></a>引數
  
*numeric_expr*  
   為數值運算式。  
  
## <a name="return-types"></a>傳回類型
  
  傳回數值運算式。  
  
## <a name="examples"></a>範例
  
  下列範例會傳回指定值的 `ATAN`。  
  
```sql
SELECT ATAN(-45.01) AS atan  
```  
  
 以下為結果集。  
  
```json
[{"atan": -1.5485826962062663}]  
```  
  

## <a name="next-steps"></a>後續步驟

- [數學函數 Azure Cosmos DB](sql-query-mathematical-functions.md)
- [系統函數 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 簡介](introduction.md)
