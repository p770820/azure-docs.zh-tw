---
title: 教學課程︰以 Azure Active Directory 設定 Netsuite OneWorld 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Netsuite OneWorld 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.assetid: 8a6d3994-ee33-4a6f-b0a2-9d0389467f16
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4c9a823e6515c2bfe09e1ab7bcef471eb8169e75
ms.sourcegitcommit: db2d402883035150f4f89d94ef79219b1604c5ba
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/07/2020
ms.locfileid: "77063290"
---
# <a name="tutorial-configuring-netsuite-for-automatic-user-provisioning"></a>教學課程︰設定 Netsuite 來自動佈建使用者

本教學課程旨在說明您需要在 Netsuite OneWorld 和 Azure AD 中執行的步驟，以將使用者帳戶從 Azure AD 自動佈建和取消佈建至 Netsuite。

## <a name="prerequisites"></a>必要條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure Active Directory 租用戶。
*   Netsuite OneWorld 訂用帳戶。 請注意，自動使用者佈建目前只透過 NetSuite OneWorld 提供支援。
*   Netsuite 中具有系統管理員權限的使用者帳戶。

## <a name="assigning-users-to-netsuite-oneworld"></a>將使用者指派給 Netsuite OneWorld

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和群組會進行同步處理。

在設定並啟用佈建服務之前，您必須決定 Azure AD 中的哪些使用者及/或群組代表需要 Netsuite 應用程式存取權的使用者。 一旦決定後，您可以依照此處的指示，將這些使用者指派給 Netsuite 應用程式︰

[將使用者或群組指派給企業應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-netsuite-oneworld"></a>將使用者指派給 Netsuite OneWorld 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Netsuite，以測試佈建設定。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 Netsuite 時，您必須選取有效的使用者角色。 「預設存取」角色不適用於佈建。

## <a name="enable-user-provisioning"></a>啟用使用者佈建

本節會引導您將 Azure AD 連線至 Netsuite 的使用者帳戶佈建 API，以及根據 Azure AD 中的使用者和群組指派，設定佈建服務以在 Netsuite 中建立、更新和停用指派的使用者帳戶。

> [!TIP] 
> 您也可以選擇啟用 Netsuite 的 SAML 型單一登入，請遵循 [Azure 入口網站](https://portal.azure.com)中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。

### <a name="to-configure-user-account-provisioning"></a>若要設定使用者帳戶佈建：

本節的目的是要說明如何對 Netsuite 啟用 Active Directory 使用者帳戶的使用者佈建。

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [Azure Active Directory > 企業應用程式 > 所有應用程式] 區段。

1. 如果您已將 Netsuite 設定為單一登入，請使用 [搜尋] 欄位搜尋您的 Netsuite 執行個體。 否則，請選取 [新增]，並在應用程式庫中搜尋 [Netsuite]。 從搜尋結果中選取 Netsuite，並將它新增至您的應用程式清單。

1. 選取您的 Netsuite 執行個體，然後選取 [佈建] 索引標籤。

1. 將 [佈建模式] 設定為 [自動]。 

    ![佈建](./media/netsuite-provisioning-tutorial/provisioning.png)

1. 在 [管理員認證] 區段下，提供下列組態設定：
   
    a. 在 [管理員使用者名稱] 文字方塊中，輸入已在 Netsuite.com 指派**系統管理員**設定檔的 Netsuite 帳戶名稱。
   
    b. 在 [管理員密碼] 文字方塊中，輸入這個帳戶的密碼。
      
1. 在 Azure 入口網站中，按一下 [測試連線]，以確保 Azure AD 可以連線至您的 Netsuite 應用程式。

1. 在 [通知電子郵件] 欄位中，輸入應收到佈建錯誤通知的個人或群組之電子郵件地址，然後勾選 [發生失敗時傳送電子郵件通知] 核取方塊。

1. 按一下 [儲存]。

1. 在 [對應] 區段下，選取 [同步處理 Azure Active Directory 使用者至 Netsuite]。

1. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Netsuite 的使用者屬性。 請注意，選取為 [比對] 屬性的屬性會用來比對 Netsuite 中的使用者帳戶，以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。

1. 若要對 Netsuite 啟用 Azure AD 佈建服務，請在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]

1. 按一下 [儲存]。

這會對 [使用者和群組] 區段中指派給 Netsuite 的任何使用者和/或群組，啟動首次同步處理。 請注意，初始同步處理會比後續同步處理花費更多時間執行，只要服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並依循連結前往佈建活動記錄，此記錄會描述您 Netsuite 應用程式上佈建服務所執行的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../app-provisioning/check-status-user-account-provisioning.md)。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定單一登入](netsuite-tutorial.md)
