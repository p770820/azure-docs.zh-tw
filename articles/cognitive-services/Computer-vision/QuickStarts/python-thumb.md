---
title: 快速入門：產生縮圖 - REST、Python
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將搭配使用電腦視覺 API 與 Python 來產生影像的縮圖。
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: quickstart
ms.date: 12/05/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 9a76d4f083c791bb1e3c29b4068c2f82f3cd0bc5
ms.sourcegitcommit: 5ab4f7a81d04a58f235071240718dfae3f1b370b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/10/2019
ms.locfileid: "74973691"
---
# <a name="quickstart-generate-a-thumbnail-using-the-computer-vision-rest-api-and-python"></a>快速入門：使用電腦視覺 REST API 和 Python 產生縮圖

在此快速入門中，您會使用電腦視覺 REST API 從影像產生縮圖。 使用[取得縮圖](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb)方法時，您可以指定所需的高度和寬度，然後電腦視覺就會使用智慧型裁剪，以智慧方式識別關注區域，並根據該區域產生裁剪座標。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/try/cognitive-services/)。

## <a name="prerequisites"></a>必要條件

- 您必須有電腦視覺的訂用帳戶金鑰。 您可以從[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=computer-vision)取得免費試用的金鑰。 或者，依照[建立認知服務帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)中的指示訂閱電腦視覺並取得金鑰。 然後，分別為名為 `COMPUTER_VISION_SUBSCRIPTION_KEY` 和 `COMPUTER_VISION_ENDPOINT` 的金鑰和服務端點字串[建立環境變數](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication)。
- 程式碼編輯器，例如 [Visual Studio Code](https://code.visualstudio.com/download)。

## <a name="create-and-run-the-sample"></a>建立並執行範例

若要建立及執行範例，請將下列程式碼複製到程式碼編輯器中。 

```python
import requests
# If you are using a Jupyter notebook, uncomment the following line.
# %matplotlib inline
import matplotlib.pyplot as plt
from PIL import Image
from io import BytesIO

# Add your Computer Vision subscription key and endpoint to your environment variables.
if 'COMPUTER_VISION_SUBSCRIPTION_KEY' in os.environ:
    subscription_key = os.environ['COMPUTER_VISION_SUBSCRIPTION_KEY']
else:
    print("\nSet the COMPUTER_VISION_SUBSCRIPTION_KEY environment variable.\n**Restart your shell or IDE for changes to take effect.**")
    sys.exit()

if 'COMPUTER_VISION_ENDPOINT' in os.environ:
    endpoint = os.environ['COMPUTER_VISION_ENDPOINT']

thumbnail_url = endpoint + "vision/v2.1/generateThumbnail"

# Set image_url to the URL of an image that you want to analyze.
image_url = "https://upload.wikimedia.org/wikipedia/commons/9/94/Bloodhound_Puppy.jpg"

headers = {'Ocp-Apim-Subscription-Key': subscription_key}
params = {'width': '50', 'height': '50', 'smartCropping': 'true'}
data = {'url': image_url}
response = requests.post(thumbnail_url, headers=headers,
                         params=params, json=data)
response.raise_for_status()

thumbnail = Image.open(BytesIO(response.content))

# Display the thumbnail.
plt.imshow(thumbnail)
plt.axis("off")

# Verify the thumbnail size.
print("Thumbnail is {0}-by-{1}".format(*thumbnail.size))
```

接下來，執行下列作業：
1. (選擇性) 將 `image_url` 的值取代為要產生縮圖之不同影像的 URL。
1. 將程式碼儲存為副檔名為 `.py` 的檔案。 例如： `get-thumbnail.py` 。
1. 開啟命令提示字元視窗。
1. 出現提示時，使用 `python` 命令執行範例。 例如： `python get-thumbnail.py` 。

## <a name="examine-the-response"></a>檢查回應

成功的回應會以二進位資料的形式傳回，代表縮圖的影像資料。 此範例應該會顯示此影像。 如果要求失敗，則回應會顯示於命令提示字元視窗中，且應該會包含錯誤碼。

## <a name="run-in-jupyter-optional"></a>在 Jupyter 中執行 (選擇性)

您可以選擇性地使用 [MyBinder](https://mybinder.org) (英文) 上的 Jupyter Notebook，以逐步方式執行本快速入門。 若要啟動 Binder，請選取下列按鈕：

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=VisionAPI.ipynb)

## <a name="next-steps"></a>後續步驟

接下來，請深入了解縮圖產生功能的詳細資訊。

> [!div class="nextstepaction"]
> [產生縮圖](../concept-generating-thumbnails.md)
