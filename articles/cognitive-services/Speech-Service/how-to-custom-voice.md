---
title: 什麼是自訂語音？ -語音服務
titleSuffix: Azure Cognitive Services
description: 「自訂語音」是一組線上工具，可讓您為您的品牌建立可辨識的一種語音。 開始使用是一些音訊檔案和相關聯的轉譯。 請遵循下列連結，開始建立自訂的語音轉換文字體驗。
services: cognitive-services
author: IEvangelist
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/17/2020
ms.author: dapine
ms.openlocfilehash: 15d15ce2d4dfc55a51bf21ba005512606cc4997a
ms.sourcegitcommit: b8f2fee3b93436c44f021dff7abe28921da72a6d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77424961"
---
# <a name="get-started-with-custom-voice"></a>開始使用自訂語音

「[自訂語音](https://aka.ms/customvoice)」是一組線上工具，可讓您為您的品牌建立可辨識的一種語音。 開始使用是一些音訊檔案和相關聯的轉譯。 請遵循下列連結，開始建立自訂的文字轉換語音體驗。

## <a name="whats-in-custom-voice"></a>自訂語音有哪些功能？

開始使用自訂語音之前，您需要 Azure 帳戶和語音服務訂用帳戶。 建立帳戶之後，您就可以準備資料、訓練和測試模型、評估語音品質，最後部署您的自訂語音模型。

下圖強調使用[自訂語音入口網站](https://aka.ms/customvoice)來建立自訂語音模型的步驟。 若要深入瞭解，請使用連結。

![自訂語音架構圖表](media/custom-voice/custom-voice-diagram.png)

1. [訂閱並建立專案](#set-up-your-azure-account)-建立 Azure 帳戶，並建立語音服務訂用帳戶。 此整合訂閱可讓您存取語音轉換文字、文字轉換語音、語音翻譯和自訂語音入口網站。 然後，使用您的語音服務訂用帳戶，建立您的第一個自訂語音專案。

2. [上傳資料](how-to-custom-voice-create-voice.md#upload-your-datasets)-使用自訂語音入口網站或自訂語音 API 來上傳資料（音訊和文字）。 在入口網站中，您可以調查及評估發音分數和信噪比。 如需詳細資訊，請參閱[如何準備自訂語音的資料](how-to-custom-voice-prepare-data.md)。

3. [定型您的模型](how-to-custom-voice-create-voice.md#build-your-custom-voice-model)–使用您的資料來建立自訂文字轉換語音語音模型。 您可以使用不同的語言來定型模型。 定型之後，請測試您的模型，如果您對結果感到滿意，可以部署模型。

4. [部署您的模型](how-to-custom-voice-create-voice.md#create-and-use-a-custom-voice-endpoint)-為您的文字轉換語音語音模型建立自訂端點，並在您的產品、工具和應用程式中使用它來進行語音合成。

## <a name="custom-neural-voices"></a>自訂類神經語音

類神經語音自訂功能目前處於公開預覽狀態，僅限選取的客戶。 填寫此[應用程式表單](https://go.microsoft.com/fwlink/?linkid=2108737)以開始使用。

> [!NOTE]
> 在 Microsoft 致力於設計負責 AI 的過程中，我們的目的是要保護個人和社會的權利，並促進透明的人類電腦互動。 基於這個理由，自訂類神經語音不會正式提供給所有客戶。 只有在您的應用程式經過審查，而且您已致力於與我們的道德原則一致時，才可以取得技術的存取權。 深入瞭解我們的[應用程式管制流程](https://aka.ms/custom-neural-gating-overview)。

## <a name="set-up-your-azure-account"></a>設定您的 Azure 帳戶

必須要有語音服務訂用帳戶，才能使用自訂語音入口網站來建立自訂模型。 請遵循這些指示，在 Azure 中建立語音服務訂用帳戶。 如果您沒有 Azure 帳戶，您可以註冊一個新帳戶。  

建立 Azure 帳戶和語音服務訂用帳戶之後，您必須登入自訂語音入口網站並聯機到您的訂用帳戶。

1. 從 Azure 入口網站取得您的語音服務訂用帳戶金鑰。
2. 登入[自訂語音入口網站](https://aka.ms/custom-voice)。
3. 選取您的訂用帳戶，並建立語音專案。
4. 如果您想要切換至另一個語音訂用帳戶，請使用位於頂端導覽中的齒輪圖示。

> [!NOTE]
> 自訂語音服務不支援30天免費試用金鑰。 您必須先在 Azure 中建立 F0 或 S0 金鑰，才能使用此服務。

## <a name="how-to-create-a-project"></a>如何建立專案

像是資料、模型、測試和端點等內容，會組織成自訂語音入口網站中的**專案**。 每個專案都適用于國家/地區，以及您想要建立的語音性別。 例如，您可以為撥接中心的聊天機器人建立一個專案，以在美國（en-us）使用英文。

若要建立您的第一個專案，請選取 [**文字轉換語音/自訂語音**] 索引標籤，然後按一下 [**新增專案**]。 依照 wizard 提供的指示來建立您的專案。 建立專案之後，您會看到四個索引標籤：**資料**、**定型**、**測試**和**部署**。 使用[後續步驟](#next-steps)中提供的連結，以瞭解如何使用每個索引標籤。

> [!IMPORTANT]
> [自訂語音入口網站](https://aka.ms/custom-voice)最近已更新！ 如果您已在 CRIS.ai 入口網站中或透過 Api 建立先前的資料、模型、測試和已發佈的端點，您必須在新的入口網站中建立新的專案，才能連接到這些舊的實體。

## <a name="next-steps"></a>後續步驟

- [準備自訂語音資料](how-to-custom-voice-prepare-data.md)
- [建立自訂語音](how-to-custom-voice-create-voice.md)
- [指南：錄製您的語音範例](record-custom-voice-samples.md)
