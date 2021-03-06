---
title: SaaS 履行 Api-常見問題 |Azure Marketplace
description: Azure Marketplace 中 SaaS 供應專案客戶的探索和購買體驗。
author: MaggiePucciEvans
manager: evansma
ms.author: evansma
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 07/11/2019
ms.openlocfilehash: d5b7a4e880cbd5dca73b3b091402e2c7dd3b7901
ms.sourcegitcommit: 014e916305e0225512f040543366711e466a9495
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2020
ms.locfileid: "75934197"
---
# <a name="saas-fulfillment-apis---faq"></a>SaaS 履行 API - 常見問題集

會列出 Azure Marketplace 的整合需求，可讓 Azure 客戶訂閱 SaaS 供應專案。

## <a name="discovery-experience"></a>探索體驗

供應專案發佈之後，Azure 使用者就可以在 Azure Marketplace 中探索 SaaS 供應專案。 您的客戶將能夠根據產品類型（SaaS）篩選供應專案，並探索他們感興趣的 SaaS 服務。

## <a name="purchase-experience"></a>購買經驗

一旦使用者對特定的 SaaS 服務感興趣，使用者就可以從 Azure Marketplace 訂閱它。

### <a name="what-does-it-mean-for-an-azure-user-to-subscribe-to-a-saas-offer-in-azure-marketplace"></a>Azure 使用者在 Azure Marketplace 中訂閱 SaaS 供應專案的意義為何？

這表示使用者可以查看與 SaaS 服務相關聯的使用規定和隱私權聲明，並同意根據您在 Microsoft 發票上的 SaaS 供應專案發行者所設定的計費條款來支付費用。 使用者可以在 Azure 中使用其現有的付款設定檔來支付 SaaS 服務耗用量。

這很有用，原因很多。 客戶現在可以使用 Microsoft Cloud 平臺做為信任的來源，在單一位置進行探索及訂閱，而不需要審查其打算使用的每個 ISV 軟體。 此外，客戶也可以使用其現有的付款設定檔，而不需要個別明確支付每個 ISV 軟體。

### <a name="is-the-user-charged-automatically-when-the-offer-is-subscribed"></a>當供應專案訂閱時，使用者會自動收費嗎？

訂閱 SaaS 供應專案時，使用者已同意透過 Microsoft 平臺使用 SaaS 服務。 不過，只有在取用供應專案時，才會開始收費。 使用者必須移至您的 SaaS 供應專案，並確認帳戶建立以開始使用供應專案。 接著，您會通知 Microsoft 開始針對此客戶 SaaS 訂用帳戶計費。

### <a name="how-are-you-notified-when-a-user-subscribes-to-your-saas-offer"></a>當使用者訂閱您的 SaaS 供應專案時，您會如何收到通知？

訂閱供應專案之後，Azure 使用者可以在 Azure 中探索及管理其所有供應專案。 根據預設，新訂閱的 SaaS 供應專案狀態為「布建 **，履行擱置中**」。 在此狀態下，Azure 使用者會收到「**設定帳戶**」動作的提示，以便流覽至 Azure 入口網站中的 SaaS 訂閱管理體驗。

當使用者按一下 [**設定帳戶**] 時，系統會將他們重新導向至 SaaS 服務網站。 在發佈供應專案時，發行者會提供流覽的 URL。 此頁面稱為「發行者」登陸頁面。 Azure 使用者應該能夠根據其在 Azure 中的現有 AAD 認證，登入 SaaS 登陸頁面。

將 Azure 使用者重新導向至登陸頁面時，會將權杖加入至查詢 URL。 此權杖的存留期很短，且在24小時的期間內有效。 然後，您可以偵測此權杖是否存在，並呼叫 Microsoft 的 API 以取得與該權杖相關聯的更多內容。

![客戶訂用帳戶流程](media/saas-metering-service-integration-flow-a.png)

如需 API 合約的詳細資訊，以在 SaaS 供應專案的生命週期中處理交易案例，請參閱[saas 履行 API](https://docs.microsoft.com/azure/marketplace/partner-center-portal/pc-saas-fulfillment-api-v2)檔。

### <a name="how-do-you-know-the-saas-offer-to-which-the-user-subscribes-in-azure"></a>您如何知道使用者在 Azure 中訂閱的 SaaS 供應專案？

`Resolve` API 的回應包含與 SaaS 訂用帳戶相關聯的供應專案和方案資訊。

### <a name="how-can-the-azure-user-change-the-plan-associated-with-this-azure-subscription"></a>Azure 使用者可以如何變更與此 Azure 訂用帳戶相關聯的方案？

* Azure 使用者可以直接在 SaaS 體驗中，或透過 Microsoft 平臺，變更與 SaaS 訂用帳戶相關聯的方案。

* 您可以隨時在計費週期中進行轉換。 您必須認可任何轉換，這會在認可之後生效。

* 預付型方案（**每月**或**每年**）費率已按比例計算。 任何在轉換時間所發出的超額部分，將會在下一張發票中收取費用。 新的超額部分將會根據新的計畫發出。

>[!Note]
>如果您不想要支援特定的轉換路徑，可以封鎖降級。

下列順序會在 Azure 客戶變更 SaaS 體驗中的計畫時，捕捉流程：

![客戶方案變更流程](media/saas-metering-service-integration-flow-b.png)

下列順序會在 Azure 客戶變更 Microsoft 店面中的方案時，捕捉流程

![客戶店面方案變更流程](media/saas-metering-service-integration-flow-c.png)

### <a name="how-can-the-azure-user-unsubscribe-from-the-plan-associated-with-azure-subscription"></a>Azure 使用者如何取消訂閱與 Azure 訂用帳戶相關聯的方案？

Azure 使用者可以直接在 SaaS 體驗中或透過 Microsoft 平臺取消訂閱購買的 SaaS 供應專案。 使用者取消訂閱之後，將不再向下一個計費週期收取費用。

下列順序會在 Azure 客戶取消訂閱 SaaS 體驗中的 SaaS 供應專案時，捕捉流程：

![SaaS 體驗中的客戶取消訂閱](media/saas-metering-service-integration-flow-d.png)

下列順序會在 Azure 使用者于 Microsoft 店面中取消訂閱時，捕捉流程：

![Microsoft 店面中的客戶取消訂閱](media/saas-metering-service-integration-flow-e.png)

## <a name="next-steps"></a>後續步驟

- 如需詳細資訊，請參閱[Marketplace 計量服務 api](./marketplace-metering-service-apis.md) 。
