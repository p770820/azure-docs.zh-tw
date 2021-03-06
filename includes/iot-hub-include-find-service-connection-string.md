---
title: 包含檔案
description: 包含檔案
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 07/16/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 55cdd864d73ce084d994c64233e79d5a58b17def
ms.sourcegitcommit: a6888fba33fc20cc6a850e436f8f1d300d03771f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2019
ms.locfileid: "69558488"
---
<!-- This tells how to get the connection string for the service shared access policy of your IoT hub -->

若要取得**服務**原則的 IoT 中樞連接字串, 請遵循下列步驟:

1. 在 [Azure 入口網站](https://portal.azure.com)中, 選取 [**資源群組**]。 選取您的中樞所在的資源群組, 然後從資源清單中選取您的中樞。

1. 在 IoT 中樞的左側窗格中, 選取 [**共用存取原則**]。

1. 從原則清單中, 選取 [**服務**原則]。

1. 在 [**共用存取金鑰**] 底下, 選取 [**連接字串-主要金鑰**] 的複製圖示, 然後儲存值。

    ![顯示如何擷取連接字串](./media/iot-hub-include-find-service-connection-string/iot-hub-get-connection-string.png)

如需 IoT 中樞共用存取原則和許可權的詳細資訊, 請參閱[存取控制和許可權](../articles/iot-hub/iot-hub-devguide-security.md#access-control-and-permissions)。
