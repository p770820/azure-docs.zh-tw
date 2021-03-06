---
title: 包含檔案
description: 包含檔案
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 08/07/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 3386cb51a8a728576f6615002d6154d89ca662c1
ms.sourcegitcommit: aa042d4341054f437f3190da7c8a718729eb675e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68883821"
---
<!-- This tells how to get the connection string for the registryReadWrite shared access policy of your IoT hub -->

若要取得**registryReadWrite**原則的 IoT 中樞連接字串, 請遵循下列步驟:

1. 在 [Azure 入口網站](https://portal.azure.com)中, 選取 [**資源群組**]。 選取您的中樞所在的資源群組, 然後從資源清單中選取您的中樞。

2. 在中樞的左側窗格中, 選取 [**共用存取原則**]。

3. 從原則清單中, 選取 [ **registryReadWrite** ] 原則。

4. 在 [**共用存取金鑰**] 底下, 選取 [**連接字串-主要金鑰**] 的複製圖示, 然後儲存值。

    ![顯示如何擷取連接字串](./media/iot-hub-include-find-registryrw-connection-string/iot-hub-get-connection-vs2019.png)

如需 IoT 中樞共用存取原則和許可權的詳細資訊, 請參閱[存取控制和許可權](../articles/iot-hub/iot-hub-devguide-security.md#access-control-and-permissions)。
