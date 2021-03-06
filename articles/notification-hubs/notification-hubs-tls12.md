---
title: 通知中樞 TLS 更新
description: 深入瞭解 Azure 通知中樞中的 TLS 支援。
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: ''
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 01/30/2020
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/28/2020
ms.openlocfilehash: 87309e20efd9d6f8bd1a659451e5a603e6b95bc8
ms.sourcegitcommit: 67e9f4cc16f2cc6d8de99239b56cb87f3e9bff41
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/31/2020
ms.locfileid: "76908523"
---
# <a name="transport-layer-security-tls"></a>傳輸層安全性 (TLS)

為確保較高層級的安全性，通知中樞將會在2020年4月30日停用 TLS 版本1.0 和1.1 的支援。 這些較舊的通訊協定提供弱式密碼編譯，而且容易遭受 BEAST 和貴賓攻擊。 部署到執行 Android 第5版或更新版本之裝置的應用程式（或 iOS 第5版或更新版本）不會受到這項變更的影響，因為這些作業系統支援 TLS 1.2，而用戶端和伺服器將會協調最高的相互支援版本連接時的通訊協定。

我們建議您查看所有使用 Azure 通知中樞的應用程式，以確保其使用最適用的程式庫和支援 TLS 1.2 的 TLS 堆疊。

## <a name="update-apps"></a>更新應用程式

您可以使用 Apple 的網路安全性功能（稱為應用程式傳輸安全性（ATS）），確保您的 iOS 應用程式使用 TLS 1.2。 ATS 無法用於 iOS 9.0 或 macOS 10.11 之前的 Sdk，您可以從[Apple 的檔](https://developer.apple.com/documentation/security/preventing_insecure_network_connections)進一步瞭解。

針對使用 SSLSocket 實例的 Android 應用程式，請在每個 SSLSocket 實例上設定啟用的通訊協定，如[setEnabledProtocols](https://developer.android.com/reference/javax/net/ssl/SSLSocket#setEnabledProtocols(java.lang.String%5B%5D))中所述。

[ [TLS 通訊協定相容性](https://support.globalsign.com/customer/portal/articles/2934392-tls-protocol-compatibility)支援] 頁面上的表格有助於對應具有相容 TLS 版本的作業系統。

如需詳細資訊，請參閱[Windows 上 TLS 通訊協定支援](https://docs.microsoft.com/archive/blogs/kaushal/support-for-ssltls-protocols-on-windows)的總覽。

## <a name="next-steps"></a>後續步驟

- [通知中樞概觀](notification-hubs-push-notification-overview.md)