---
title: 快速入門：將語音翻譯成多種目標語言 - 語音服務
titleSuffix: Azure Cognitive Services
description: TBD
services: cognitive-services
author: yulin-li
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 12/09/2019
ms.author: yulili
ms.openlocfilehash: 8c8cbc4e4f531d7a06ae3a33c33df9264c2cc6f2
ms.sourcegitcommit: 5ab4f7a81d04a58f235071240718dfae3f1b370b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/10/2019
ms.locfileid: "74981428"
---
在本快速入門中，您將會使用[語音 SDK](~/articles/cognitive-services/speech-service/speech-sdk.md) 來以互動方式將某一種語言的語音翻譯成另一種語言的語音。 在滿足幾個必要條件之後，以多種語言翻譯語音轉換文字只需要採取六個步驟：
> [!div class="checklist"]
> * 從您的訂用帳戶金鑰和區域建立 ````SpeechTranslationConfig```` 物件。
> * 更新 ````SpeechTranslationConfig```` 物件以指定語音辨識來源語言。
> * 更新 ````SpeechTranslationConfig```` 物件以指定多個翻譯目標語言。
> * 使用上面的 ````SpeechTranslationConfig```` 物件來建立 ````TranslationRecognizer```` 物件。
> * 使用 ````TranslationRecognizer```` 物件，開始針對單一表達進行辨識程序。
> * 檢查所傳回的 ````TranslationRecognitionResult````。
