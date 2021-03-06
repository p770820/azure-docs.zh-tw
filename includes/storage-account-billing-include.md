---
title: 包含檔案
description: 包含檔案
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/19/2018
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: cdcbe993bd1100b2060a1f8d38eb82ac97121c0d
ms.sourcegitcommit: c38a1f55bed721aea4355a6d9289897a4ac769d2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/05/2019
ms.locfileid: "74851618"
---
系統會根據您的儲存體帳戶使用量向您收取 Azure 儲存體費用。 儲存體帳戶中的所有物件會做為群組共同計費。 

儲存體成本是根據下列因素來計算： 

* **「區域」** 係指您帳戶所在的地理區域。
* **帳戶類型**指的是您所使用的儲存體帳戶類型。 
* 「**存取層**」指的是您為一般用途 V2 或 Blob 儲存體帳戶指定的資料使用模式。
* 儲存體**容量**指的是您用來儲存資料的儲存體帳戶配額大小。
* 「複寫」會決定一次維護多少個資料複本，以及在哪些位置維護。
* 「交易」係指對「Azure 儲存體」進行的所有讀取和寫入作業。
* 「資料輸出」係指從 Azure 區域傳出的任何資料。 當儲存體帳戶中的資料是由不是在相同區域中執行的應用程式存取時，您需支付資料輸出的費用。 如需了解如何使用資源群組來將您的資料和服務聚集在相同區域以限制輸出費用，請參閱[什麼是 Azure 資源群組?](https://docs.microsoft.com/azure/cloud-adoption-framework/govern/resource-consistency/resource-access-management#what-is-an-azure-resource-group) 

[Azure 儲存體價格](https://azure.microsoft.com/pricing/details/storage/) 頁面提供了以帳戶類型、儲存體容量、複寫和交易為基礎的詳細價格資訊。 [資料傳輸定價詳細資料](https://azure.microsoft.com/pricing/details/data-transfers/) 則提供了出口流量的詳細定價資訊。 您可以使用 [Azure 儲存體定價計算機](https://azure.microsoft.com/pricing/calculator/?scenario=data-management) ，以協助消除成本。

