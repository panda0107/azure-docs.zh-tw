---
title: 使用 Azure 檔案共用搭配 Azure 儲存體 |Microsoft Docs
description: 了解如何搭配 Windows 和 Windows Server 使用 Azure 檔案共用。
author: todmccoy
manager: dcscontentpm
ms.service: storage
ms.topic: conceptual
ms.date: 11/19/2019
ms.author: rogarana
ms.subservice: files
services: storage
tags: ''
ms.openlocfilehash: e1e581134b4e3821659c43bb7f55003239594bd9
ms.sourcegitcommit: d6b68b907e5158b451239e4c09bb55eccb5fef89
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/20/2019
ms.locfileid: "74233788"
---
# <a name="how-to-recover-a-deleted-storage-account"></a>如何復原已刪除的儲存體帳戶

Azure 儲存體透過自動化複本提供資料恢復功能，但不會防止使用者或應用程式程式碼損毀資料（不論是不小心或惡意）。 在應用程式或使用者錯誤的實例中維持資料精確度需要更先進的技術，例如將資料複製到具有 audit 記錄檔的次要儲存體位置。

下表提供儲存體帳戶復原範圍的總覽，視複寫策略而定。

| |LRS|ZRS|GRS|RA-GRS|
|---|---|---|---|---|
|儲存體帳戶 Azure Resource Manager|yes|yes|yes|yes|
|傳統儲存體帳戶|yes|yes|yes|yes|

收集下列資訊，並使用 Microsoft 支援服務提出支援要求：

* 儲存體帳戶名稱
* 刪除日期
* 儲存體帳戶區域
* 如何刪除儲存體帳戶？
* 哪些方法會刪除儲存體帳戶？ （入口網站、PowerShell 等）

重要重點

* 最多可能需要15天的時間來刪除儲存體服務，才能執行垃圾收集，因此無法使用 SLA 來復原儲存體帳戶。
* Microsoft 支援服務將會嘗試以最佳方式復原容器/帳戶，而且無法保證修復。

> [!NOTE]
> 如果帳戶已重新建立，則復原可能無法成功。 如果您已經重新建立帳戶，您必須先將它刪除，然後才能嘗試復原。
