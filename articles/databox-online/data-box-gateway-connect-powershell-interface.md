---
title: 透過 Windows PowerShell 介面連接到 Microsoft Azure 資料箱閘道裝置並加以管理 |Microsoft Docs
description: 說明如何透過 Windows PowerShell 介面連接到資料箱閘道, 然後再進行管理。
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 08/02/2019
ms.author: alkohli
ms.openlocfilehash: 650777d849e172686e8b46502a84db8c519174e7
ms.sourcegitcommit: 4b5dcdcd80860764e291f18de081a41753946ec9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2019
ms.locfileid: "68775187"
---
# <a name="manage-an-azure-data-box-gateway-device-via-windows-powershell"></a>透過 Windows PowerShell 管理 Azure 資料箱閘道裝置

Azure 資料箱的閘道解決方案可讓您透過網路將資料傳送至 Azure。 本文說明資料箱閘道裝置的一些設定和管理工作。 您可以使用 Azure 入口網站、本機 web UI 或 Windows PowerShell 介面來管理您的裝置。

本文著重于您使用 PowerShell 介面進行的工作。

本文包含下列程式:

- 連接到 PowerShell 介面
- 建立支援封裝
- 上傳憑證
- 在非 DHCP 環境中開機
- 查看裝置資訊

## <a name="connect-to-the-powershell-interface"></a>連接到 PowerShell 介面

[!INCLUDE [Connect to admin runspace](../../includes/data-box-edge-gateway-connect-minishell.md)]

## <a name="create-a-support-package"></a>建立支援封裝

[!INCLUDE [Create a support package](../../includes/data-box-edge-gateway-create-support-package.md)]

## <a name="upload-certificate"></a>上傳憑證

[!INCLUDE [Upload certificate](../../includes/data-box-edge-gateway-upload-certificate.md)]

## <a name="boot-up-in-non-dhcp-environment"></a>在非 DHCP 環境中開機

[!INCLUDE [Boot up in non-DHCP environment](../../includes/data-box-edge-gateway-boot-non-dhcp.md)]

## <a name="view-device-information"></a>查看裝置資訊

[!INCLUDE [View device information](../../includes/data-box-edge-gateway-view-device-info.md)]

## <a name="next-steps"></a>後續步驟

- 在 Azure 入口網站中部署 [Azure 資料箱閘道](data-box-gateway-deploy-prep.md)。
