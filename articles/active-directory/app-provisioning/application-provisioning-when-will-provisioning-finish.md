---
title: Azure AD 資源庫應用程式的使用者布建花費數小時或更多時間
description: 如何查明佈建至應用程式為什麼比您預期的更久
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5c5cc2c47eafba7d6afdf506eab03a17aa8987be
ms.sourcegitcommit: db2d402883035150f4f89d94ef79219b1604c5ba
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/07/2020
ms.locfileid: "77067041"
---
# <a name="user-provisioning-to-an-azure-ad-gallery-application-is-taking-hours-or-more"></a>將使用者佈建至 Azure AD 資源庫應用程式花費數小時以上

當第一次啟用應用程式的自動布建時，初始週期可能需要20分鐘到數小時的時間，視 Azure AD 目錄大小和布建範圍中的使用者數目而定。 

在初始週期之後的後續同步處理會更快，因為布建服務會在初始週期後儲存代表兩個系統狀態的浮水印，以改善後續同步處理的效能。

## <a name="how-to-improve-provisioning-performance"></a>如何改善佈建的效能

如果初始週期花費的時間超過幾個小時，您可以執行下列動作來改善效能：

-   **使用者範圍設定篩選條件。** 範圍設定篩選條件可讓您根據特定的屬性值篩選出使用者，以微調佈建服務從 Azure AD 擷取的資料。 如需範圍設定篩選條件的詳細資訊，請參閱[含範圍篩選器的屬性型應用程式佈建](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)。

## <a name="next-steps"></a>後續步驟
[自動化使用 Azure Active Directory 對於 SaaS 應用程式的使用者佈建和解除佈建](user-provisioning.md)

