---
title: 在 Azure API 管理中管理通訊協定和加密 | Microsoft Docs
description: 了解如何在 Azure API 管理中管理通訊協定 (TLS、SSL) 和加密 (DES)。
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 05/29/2019
ms.author: apimpm
ms.openlocfilehash: f7c7fdd06480ce3da70c86d38ab0685b9b3aaaf2
ms.sourcegitcommit: 82499878a3d2a33a02a751d6e6e3800adbfa8c13
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2019
ms.locfileid: "70072394"
---
# <a name="manage-protocols-and-ciphers-in-azure-api-management"></a>在 Azure API 管理中管理通訊協定和加密

Azure API 管理支援適用於用戶端和後端的多個 TLS 通訊協定版本以及 3DES 加密。

本指南示範如何管理適用於 Azure API 管理執行個體的通訊協定和加密設定。

![在 APIM 中管理通訊協定和加密](./media/api-management-howto-manage-protocols-ciphers/api-management-protocols-ciphers.png)

## <a name="prerequisites"></a>必要條件

若要依照本文中的步驟進行，您必須有：

* API 管理執行個體

## <a name="how-to-manage-tls-protocols-and-3des-cipher"></a>如何管理 TLS 通訊協定和 3DES 加密

1. 在 Azure 入口網站中瀏覽至您的 **API 管理執行個體**。
2. 從功能表中選取 [**通訊協定設定**]。  
3. 啟用或停用所需的通訊協定或加密。
4. 按一下 [儲存]。 變更將在一小時內套用。  

## <a name="next-steps"></a>後續步驟

* 深入了解 [TLS (傳輸層安全性)](https://docs.microsoft.com/dotnet/framework/network-programming/tls)。
* 查看更多有關 API 管理的 [視訊](https://azure.microsoft.com/documentation/videos/index/?services=api-management) 。