---
title: Azure Cosmos DB 查詢語言中的 IS_ARRAY
description: 深入瞭解 Azure Cosmos DB 中的 SQL 系統函數 IS_ARRAY。
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 11ff29dcae7bf5c2e532eee658ee39d39b964e35
ms.sourcegitcommit: 7f6d986a60eff2c170172bd8bcb834302bb41f71
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2019
ms.locfileid: "71350971"
---
# <a name="is_array-azure-cosmos-db"></a>IS_ARRAY （Azure Cosmos DB）
 傳回布林值，表示指定之運算式的類型為陣列。  
  
## <a name="syntax"></a>語法
  
```sql
IS_ARRAY(<expr>)  
```  
  
## <a name="arguments"></a>引數
  
*expr*  
   這是任何運算式。  
  
## <a name="return-types"></a>傳回類型
  
  傳回布林運算式。  
  
## <a name="examples"></a>範例
  
  下列範例會使用 `IS_ARRAY` 函式，檢查 JSON 布林值的物件、數位、字串、null、物件、陣列以及未定義的類型。  
  
```sql
SELECT   
 IS_ARRAY(true) AS isArray1,   
 IS_ARRAY(1) AS isArray2,  
 IS_ARRAY("value") AS isArray3,  
 IS_ARRAY(null) AS isArray4,  
 IS_ARRAY({prop: "value"}) AS isArray5,   
 IS_ARRAY([1, 2, 3]) AS isArray6,  
 IS_ARRAY({prop: "value"}.prop2) AS isArray7  
```  
  
 以下為結果集。  
  
```json
[{"isArray1":false,"isArray2":false,"isArray3":false,"isArray4":false,"isArray5":false,"isArray6":true,"isArray7":false}]
```  

## <a name="next-steps"></a>後續步驟

- [型別檢查函數 Azure Cosmos DB](sql-query-type-checking-functions.md)
- [系統函數 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 簡介](introduction.md)
