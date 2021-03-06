---
title: 關於語音裝置 SDK-語音服務
titleSuffix: Azure Cognitive Services
description: 開始使用語音裝置 SDK。 語音服務可與各種裝置和音訊來源搭配運作。 語音裝置 SDK 是已預先微調的程式庫，可搭配有目的建置的麥克風陣列開發套件。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: erhopf
ms.openlocfilehash: 854e7beb2afd8ae838455f77ff448f13d8b3fbea
ms.sourcegitcommit: b07964632879a077b10f988aa33fa3907cbaaf0e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/13/2020
ms.locfileid: "77188968"
---
# <a name="about-the-speech-devices-sdk"></a>關於語音裝置 SDK

[語音服務](overview.md)可與各種裝置和音訊來源搭配運作。 您現在可以利用相符的硬體和軟體，將語音應用程式提升至下一個層級。 語音裝置 SDK 是已預先微調的程式庫，可搭配有目的建置的麥克風陣列開發套件。

語音裝置 SDK 可協助您：

- 快速測試新的語音案例。
- 更輕鬆地將雲端式語音服務整合到您的裝置。
- 為客戶建立卓越的使用者體驗。

語音裝置 SDK 會取用[語音 SDK](speech-sdk.md)， 使用先進的音訊處理演算法搭配裝置的麥克風陣列，將音訊傳送至[語音服務](overview.md)。 它透過雜訊抑制、回應取消、波束成形和 dereverberation，提供精確的現場[語音辨識](speech-to-text.md)。

您也可以使用語音裝置 SDK 來建立具有您自己[自訂關鍵字](speech-devices-sdk-create-kws.md)的環境裝置。 自訂關鍵字提供的提示會啟動使用者互動，這對您的品牌而言是唯一的。

語音裝置 SDK 可提供各種語音功能的案例，例如[語音助理](https://aka.ms/bots/speech/va)、磁片磁碟機排序系統、[交談](conversation-transcription-service.md)轉譯和智慧型喇叭。 您可以利用文字來回應使用者、以預設或[自訂語音](how-to-customize-voice-font.md)回答他們、提供搜尋結果、[翻譯](speech-translation.md)成其他語言等等。 我們期待看到您所打造的不凡成果！

## <a name="get-the-speech-devices-sdk"></a>取得語音裝置 SDK

### <a name="android"></a>Android

若是 Android，裝置會下載最新版的[Android 語音裝置 SDK](https://aka.ms/sdsdk-download-android)。

### <a name="windows"></a>Windows

針對 Windows，範例應用程式會以跨平臺 JAVA 應用程式的形式提供。 下載最新版的[JRE 語音裝置 SDK](https://aka.ms/sdsdk-download-JRE)。
此應用程式是使用語音 SDK 套件，以及 64 位元 Windows 上的 Eclipse Java IDE (v4.8) 來建置。 它會在 64 位元 Java 8 Runtime Environment (JRE) 上執行。

### <a name="linux"></a>Linux

針對 Linux，範例應用程式會以跨平臺 JAVA 應用程式的形式提供。 下載最新版的[JRE 語音裝置 SDK](https://aka.ms/sdsdk-download-JRE)。
此應用程式是使用語音 SDK 套件，以及 64 位元 Linux (Ubuntu 16.04、Ubuntu 18.04、Debian 9) 上的 Eclipse Java IDE (v4.8) 來建置。 它會在 64 位元 Java 8 Runtime Environment (JRE) 上執行。

另外還提供其他二進位檔，以支援即將推出的裝置、 [Roobo V2 DDK](https://aka.ms/sdsdk-download-roobov2)和[Urbetter DDK](https://aka.ms/sdsdk-download-urbetter)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [選擇您的語音裝置](get-speech-devices-sdk.md)
> [!div class="nextstepaction"]
> [免費取得語音服務的訂用帳戶金鑰](get-started.md)
