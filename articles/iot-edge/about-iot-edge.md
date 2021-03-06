---
title: 什麼是 Azure IoT Edge | Microsoft Docs
description: Azure IoT Edge 服務的概觀
author: kgremban
manager: philmea
ms.reviewer: chipalost
ms.service: iot-edge
services: iot-edge
ms.topic: overview
ms.date: 10/28/2019
ms.author: kgremban
ms.custom: mvc
ms.openlocfilehash: 1ba133acda414d9779e2fb10150bbdd57285e9a5
ms.sourcegitcommit: 38b11501526a7997cfe1c7980d57e772b1f3169b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2020
ms.locfileid: "76511207"
---
# <a name="what-is-azure-iot-edge"></a>什麼是 Azure IoT Edge

Azure IoT Edge 將雲端分析和自訂商務邏輯移至裝置，讓您的組織可以專注於商業洞見而非資料管理。 將您的商務邏輯封裝到標準容器中，以擴充您的 IoT 解決方案，然後將這些容器部署到任何裝置，並從雲端進行全面監視。

進行分析可促進 IoT 解決方案的商務價值，但並非所有分析都必須在雲端進行。 如果您希望盡快回應緊急事件，可以在邊緣執行異常偵測工作負載。 如果您想要降低頻寬成本，並避免傳送數 TB 的原始資料，您可以在本機清除和彙總資料，然後只將見解傳送至雲端進行分析。

Azure IoT Edge 是由三個元件組成：

* **IoT Edge 模組**是執行 Azure 服務、第三方服務或自有程式碼的容器。 這類模組會部署到 IoT Edge 裝置，並在這些裝置本機上執行。
* **IoT Edge 執行階段**會在每個 IoT Edge 裝置上執行，並管理部署到每個裝置的模組。
* **雲端式介面**可讓您在遠端監視及管理 IoT Edge 裝置。

>[!NOTE]
>Azure IoT Edge 適用於 IoT 中樞的免費和標準層。 免費層僅供測試和評估之用。 如需有關基本和標準層的詳細資訊，請參閱[如何選擇適合的 IoT 中樞層](../iot-hub/iot-hub-scaling.md)。

## <a name="iot-edge-modules"></a>IoT Edge 模組

IoT Edge 模組是執行的單位，實作為 Docker 相容容器，可在終端執行您的商務邏輯。 您可以將多個模組設定為彼此通訊，並建立資料處理的管道。 您可以開發自訂模組，或將特定 Azure 服務封裝為可離線和在終端提供深入解析的模組。

### <a name="artificial-intelligence-at-the-edge"></a>邊緣人工智慧

Azure IoT Edge 可讓您部署複雜的事件處理、機器學習、影像辨識和其他高價值的 AI，而不需要在公司內部另外撰寫。 Azure 服務 (例如 Azure Functions、Azure Stream Analytics 和 Azure Machine Learning) 全都可以透過 Azure IoT Edge 在內部部署環境執行。 然而，您不限於使用 Azure 服務。 任何人都能夠建立 AI 模組，並透過 Azure Marketplace 供社群使用。

### <a name="bring-your-own-code"></a>自備程式碼

當您要將自己的程式碼部署到裝置時，Azure IoT Edge 也支援該功能。 Azure IoT Edge 採用與其他 Azure IoT 服務相同的程式設計模型。 您可以在裝置上或雲端中執行相同的程式碼。 Azure IoT Edge 支援 Linux 和 Windows，因此您可以在您所選的平台撰寫程式碼。 它可支援 Java、.NET Core 2.0、Node.js、C 及 Python，所以您的開發人員可以用他們已知的語言撰寫程式碼，並使用現有的商務邏輯。

## <a name="iot-edge-runtime"></a>IoT Edge 執行階段

Azure IoT Edge 執行階段可在 IoT Edge 裝置上啟用自訂和雲端邏輯。 此執行階段位於 IoT Edge 裝置上，可執行管理和通訊作業。 此執行階段會執行數個功能：

* 在裝置上安裝和更新工作負載。
* 在裝置上維護 Azure IoT Edge 安全性標準。
* 確保 IoT Edge 模組一律執行中。
* 將模組健康情況報告至雲端，以便進行遠端監控。
* 管理下游分葉裝置與 IoT Edge 裝置之間、IoT Edge 裝置上的模組之間，以及 IoT Edge 裝置與雲端之間的通訊。

![IoT Edge 執行階段會將深入解析和報告傳送至 IoT 中樞](./media/about-iot-edge/runtime.png)

您可以自行決定如何使用 Azure IoT Edge 裝置。 此執行階段通常用於將 AI 部署至閘道裝置，以便彙總及處理其他內部部署裝置上的資料，但是此部署模型只是選項之一。

Azure IoT Edge 執行階段可在能夠以各種不同方式使用執行階段的大量 IoT 裝置上執行。 它可支援 Linux 和 Windows 作業系統，並擷取硬體詳細資料。 如果您不需要處理很多資料，也不需要使用工業伺服器以執行資源密集的工作負載，那麼您也可以使用比 Raspberry Pi 3 更小的裝置。

## <a name="iot-edge-cloud-interface"></a>IoT Edge 雲端介面

數百萬個 IoT 裝置通常屬於不同的製造商和型號或分散各地，因此很難管理其軟體生命週期。 系統會針對特定類型的裝置建立和設定工作負載、將其部署到您所有的裝置，並且進行監視以攔截任何異常裝置。 這些活動無法逐一為裝置進行，而必須大規模進行。

Azure IoT Edge 能夠與 Azure IoT 解決方案加速器緊密整合，以針對您的解決方案需求提供一個控制平面。 雲端服務可讓您：

* 建立並設定要在特定類型的裝置上執行的工作負載。
* 將工作負載傳送到一組裝置。
* 監視在現場裝置上執行的工作負載。

![裝置遙測及動作會透過雲端協調](./media/about-iot-edge/cloud-interface.png)

## <a name="next-steps"></a>後續步驟

[在模擬裝置上部署 IoT Edge](quickstart.md)，試著了解這些概念。
