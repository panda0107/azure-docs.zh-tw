---
title: 以 Azure API 管理進行進階要求節流
description: 了解如何使用 Azure API 管理來建立及套用彈性配額和速率限制原則。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: fc813a65-7793-4c17-8bb9-e387838193ae
ms.service: api-management
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/03/2018
ms.author: apimpm
ms.openlocfilehash: 467d9cee74567fc0d19031773415675ae7c51818
ms.sourcegitcommit: f209d0dd13f533aadab8e15ac66389de802c581b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2019
ms.locfileid: "71066759"
---
# <a name="advanced-request-throttling-with-azure-api-management"></a>以 Azure API 管理進行進階要求節流
能夠節流傳入要求是 Azure API 管理的重要角色。 藉由控制要求的速率或傳輸的要求/資料總量，API 管理讓 API 提供者能夠保護其 API 不被濫用，並建立不同 API 產品層級的價值。

## <a name="product-based-throttling"></a>產品型節流
到目前為止，速率節流功能都侷限於特定產品訂用帳戶的範圍內，定義於 Azure 入口網站。 這可協助 API 提供者將限制套用至註冊使用其 API 的開發人員，不過，舉例來說，它無法協助對 API 的個別使用者進行節流。 想讓開發人員的應用程式的單一使用者取用整個配額，並讓開發人員的其他客戶無法使用應用程式，是有可能的。 同樣的，數個產生大量要求的客戶可能會限制偶爾使用者的存取權。

## <a name="custom-key-based-throttling"></a>依自訂索引鍵節流

> [!NOTE]
> 在 Azure `quota-by-key` API 管理的取用層中，無法使用和原則。`rate-limit-by-key` 

新的 [rate-limit-by-key](/azure/api-management/api-management-access-restriction-policies#LimitCallRateByKey) 和 [quota-by-key](/azure/api-management/api-management-access-restriction-policies#SetUsageQuotaByKey) 原則能提供更有彈性的流量控制解決方案。 這些新原則可讓您定義運算式，以識別要用來追蹤流量使用量的索引鍵。 其運作的方式用範例來說明最簡單。 

## <a name="ip-address-throttling"></a>IP 位址節流
下列原則會限制單一用戶端 IP 位址每一分鐘只有 10 個呼叫，等於每個月總數為 1,000,000 個呼叫和 10,000 KB 頻寬。 

```xml
<rate-limit-by-key  calls="10"
          renewal-period="60"
          counter-key="@(context.Request.IpAddress)" />

<quota-by-key calls="1000000"
          bandwidth="10000"
          renewal-period="2629800"
          counter-key="@(context.Request.IpAddress)" />
```

如果網際網路上的所有用戶端皆使用唯一 IP 位址，這是可能是限制使用者使用量的有效方式。 不過，可能會有多個使用者共用單一公用 IP 位址，因為他們是透過 NAT 裝置存取網際網路。 儘管如此，對允許未驗證存取的 API 來說， `IpAddress` 可能是最佳選項。

## <a name="user-identity-throttling"></a>使用者身分識別節流
如果使用者經過驗證，則可根據該使用者的唯一身分識別產生節流索引鍵。

```xml
<rate-limit-by-key calls="10"
    renewal-period="60"
    counter-key="@(context.Request.Headers.GetValueOrDefault("Authorization","").AsJwt()?.Subject)" />
```

此範例會示範如何擷取授權標頭，將它轉換成 `JWT` 物件，然後使用權杖的主體來識別使用者，並使用它作為速率限制索引鍵。 如果使用者身分識別儲存於 `JWT` 以作為其中一個其他宣告，則可使用該值來代替它。

## <a name="combined-policies"></a>結合的原則
雖然新的節流原則比現有節流原則提供更多的控制，但仍會有結合兩種功能的值。 依產品訂用帳戶金鑰 ([依訂用帳戶限制呼叫率](/azure/api-management/api-management-access-restriction-policies#LimitCallRate)和[依訂用帳戶設定使用量配額](/azure/api-management/api-management-access-restriction-policies#SetUsageQuota)) 進行節流是一種根據使用量層級收費、讓 API 創造獲利的絕佳方式。 更精細的依使用者控制節流則和其互補，並防止一位使用者的行為降低另一位使用者的體驗。 

## <a name="client-driven-throttling"></a>用戶端導向節流
若使用 [原則運算式](/azure/api-management/api-management-policy-expressions)定義節流索引鍵，則是由 API 提供者選擇如何設定節流的範圍。 不過，開發人員可能想要控制他們對自己的客戶的速率限制。 API 提供者可以藉由導入自訂標頭來做到這一點，以允許開發人員的用戶端應用程式與 API 通訊索引鍵。

```xml
<rate-limit-by-key calls="100"
          renewal-period="60"
          counter-key="@(request.Headers.GetValueOrDefault("Rate-Key",""))"/>
```

這可讓開發人員的用戶端應用程式選擇要如何建立速率限制索引鍵。 用戶端開發人員可以透過將索引鍵組配置給使用者並輪流使用索引鍵，來建立自己的速率層。

## <a name="summary"></a>總結
Azure API 管理提供速率和配額節流，不但能保護您的 API 服務，並為您的 API 服務增加價值。 新的節流原則與自訂範圍規則，可讓您更精細的控制這些原則，進而讓您的客戶建置更好的應用程式。 本文中的範例示範如何使用這些新原則，分別使用用戶端 IP 位址、使用者身分識別及用戶端產生值來製造速率限制索引鍵。 不過，訊息中還有許多其他部份可以利用，例如使用者代理程式、URL 路徑片段、訊息大小。

## <a name="next-steps"></a>後續步驟
請將您的意見反應提供給我們，做為本主題的 GitHub 問題。 我們很想知道其他在您的案例中是合理選擇的可能索引鍵值。

