---
title: 快速入門：搜尋影像 - 適用於 Python 的 Bing 影像搜尋 SDK
titleSuffix: Azure Cognitive Services
description: Bing 影像搜尋 SDK 是 API 的包裝函式，而且功能相同，您可以透過此快速入門，完成初次使用此 SDK 進行的影像搜尋。 這個簡單的 Python 應用程式會傳送影像搜尋查詢、剖析 JSON 回應，以及顯示第一個傳回影像的 URL。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: quickstart
ms.date: 12/06/2019
ms.author: aahi
ms.custom: seodec2018
ms.openlocfilehash: 561162767a48a060763510310de77767f37d4eb4
ms.sourcegitcommit: aee08b05a4e72b192a6e62a8fb581a7b08b9c02a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/09/2020
ms.locfileid: "75770181"
---
# <a name="quickstart-search-for-images-with-the-bing-image-search-sdk-for-python"></a>快速入門：使用適用於 Python 的 Bing 影像搜尋 SDK 來搜尋影像

Bing 影像搜尋 SDK 是 API 的包裝函式，而且功能相同，您可以透過此快速入門，完成初次使用此 SDK 進行的影像搜尋。 這個簡單的 Python 應用程式會傳送影像搜尋查詢、剖析 JSON 回應，以及顯示第一個傳回影像的 URL。

此範例的原始程式碼可從 [GitHub](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/image-search-quickstart.py) 取得，其中含有其他錯誤處理和註釋。

## <a name="prerequisites"></a>Prerequisites

* [Python 2.7 或 3.4](https://www.python.org/) 和更新版本。

* 適用於 Python 的 [Azure 影像搜尋 SDK](https://pypi.org/project/azure-cognitiveservices-search-imagesearch/)
    * 使用 `pip install azure-cognitiveservices-search-imagesearch` 進行安裝

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>建立應用程式並將其初始化

1. 在您慣用的 IDE 或編輯器中建立新的 Python 專案以及下列匯入：

    ```python
    from azure.cognitiveservices.search.imagesearch import ImageSearchClient
    from msrest.authentication import CognitiveServicesCredentials
    ```

2. 針對您的訂用帳戶金鑰和搜尋字詞建立變數。

    ```python
    subscription_key = "Enter your key here"
    subscription_endpoint = "Enter your endpoint here"
    search_term = "canadian rockies"
    ```

## <a name="create-the-image-search-client"></a>建立影像搜尋用戶端

1. 建立 `CognitiveServicesCredentials` 的執行個體，並使用它來將用戶端具現化：

    ```python
    client = ImageSearchClient(endpoint=subscription_endpoint, credentials=CognitiveServicesCredentials(subscription_key))
    ```
1. 將搜尋查詢傳送到 Bing 影像搜尋 API：
    ```python
    image_results = client.images.search(query=search_term)
    ```
   ## <a name="process-and-view-the-results"></a>處理並檢視結果

剖析回應中所傳回的影像結果。


如果回應包含搜尋結果，請儲存第一個結果並列印出其詳細資料，例如縮圖 URL、原始 URL 及傳回影像的總數。  

```python
if image_results.value:
    first_image_result = image_results.value[0]
    print("Total number of images returned: {}".format(len(image_results.value)))
    print("First image thumbnail url: {}".format(
        first_image_result.thumbnail_url))
    print("First image content url: {}".format(first_image_result.content_url))
else:
    print("No image results returned!")
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 影像搜尋單頁應用程式教學課程](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/tutorial-bing-image-search-single-page-app) (英文)

## <a name="see-also"></a>另請參閱

* [什麼是 Bing 影像搜尋？](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/overview)  
* [試用線上互動式示範](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [取得免費認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-image-search-api)
* [Azure 認知服務 SDK 的 Python 範例](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)  
* [Azure 認知服務文件](https://docs.microsoft.com/azure/cognitive-services)
* [Bing 影像搜尋 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)
