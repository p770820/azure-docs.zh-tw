---
title: 教學課程：Azure Active Directory 與 Secret Server (On-Premises) 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Secret Server (On-Premises) 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: be4ba84a-275d-4f71-afce-cb064edc713f
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/07/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4926fc1833cc14b2ad81a01e230a5c3c37ba6ab3
ms.sourcegitcommit: aa042d4341054f437f3190da7c8a718729eb675e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68880147"
---
# <a name="tutorial-integrate-secret-server-on-premises-with-azure-active-directory"></a>教學課程：整合 Secret Server (On-premises) 與 Azure Active Directory

在本教學課程中，您將了解如何整合 Secret Server (On-Premises) 與 Azure Active Directory (Azure AD)。 當您整合 Secret Server (On-premises) 與 Azure AD 時，您可以：

* 在 Azure AD 中控制可存取 Secret Server (On-Premises) 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 Secret Server (On-Premises)。
* 在 Azure 入口網站集中管理您的帳戶。

若要深入了解 SaaS 應用程式與 Azure AD 整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 Secret Server (On-Premises) 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* Secret Server (on-premises) 支援 **SP 和 IDP** 起始的 SSO

## <a name="adding-secret-server-on-premises-from-the-gallery"></a>從資源庫新增 Secret Server (On-Premises)

若要設定 Secret Server (On-Premises) 與 Azure AD 整合，您需要從資源庫將 Secret Server (On-Premises) 新增到受控 SaaS 應用程式清單中。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 [Azure 入口網站](https://portal.azure.com)。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory]  服務。
1. 巡覽至 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增]  區段的搜尋方塊中輸入 **Secret Server (On-Premises)** 。
1. 從結果面板中選取 [Secret Server (On-Premises)]  ，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。


## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

以名為 **B.Simon** 的測試使用者，設定及測試與 Secret Server (On-Premises) 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Secret Server (On-Premises) 中相關使用者之間的連結關聯性。

若要使用 Secret Server (On-Premises) 來設定並測試 Azure AD SSO，請完成下列建構元素：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
2. **[設定 Secret Server (On-Premises) SSO](#configure-secret-server-on-premises-sso)** - 在應用程式端設定單一登入設定。
3. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
5. **[建立 Secret Server (On-Premises) 測試使用者](#create-secret-server-on-premises-test-user)** - 在 Secret Server (On-Premises) 中建立 B.Simon 的對應項目，且該項目與 Azure AD 中代表 Britta Simon 的項目連結。
6. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 [Azure 入口網站](https://portal.azure.com/)的 [Secret Server (On-Premises)]  應用程式整合頁面上，尋找 [管理]  區段並選取 [單一登入]  。
1. 在 [選取單一登入方法]  頁面上，選取 [SAML]  。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 設定]  區段上，如果您想要以 **IDP** 起始模式設定應用程式，請輸入下列欄位的值：

    a. 在 [識別碼]  文字方塊中，輸入使用者選擇的值作為範例：`https://secretserveronpremises.azure`

    b. 在 [回覆 URL]  文字方塊中，使用下列模式來輸入 URL：`https://<SecretServerURL>/SAML/AssertionConsumerService.aspx`

    > [!NOTE]
    > 上面顯示的實體識別碼只是範例，您可以自由選擇任何唯一值以在 Azure AD 中識別您的 Secret Server 執行個體。 您需要將此實體識別碼傳送到 [Secret Server (On-Premises) 用戶端支援小組](https://thycotic.force.com/support/s/)，以在他們那端設定此識別碼。 如需詳細資料，請參閱[這篇文章](https://thycotic.force.com/support/s/article/Configuring-SAML-in-Secret-Server)。

1. 如果您想要以 **SP** 起始模式設定應用程式，請按一下 [設定其他 URL]  ，然後執行下列步驟：

    在 [登入 URL]  文字方塊中，以下列模式輸入 URL︰`https://<SecretServerURL>/login.aspx`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的回覆 URL 與登入 URL 更新這些值。 請連絡 [Secret Server (On-Premises) 用戶端支援小組](https://thycotic.force.com/support/s/)以取得這些值。 您也可以參考 Azure 入口網站中**基本 SAML 組態**區段所示的模式。

1. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，尋找 [憑證 (Base64)]  並選取 [下載]  ，以下載憑證並將其儲存在電腦上。

    ![憑證下載連結](common/certificatebase64.png)

1. 在 [以 SAML 設定單一登入]  頁面上按一下 [編輯]  圖示，以開啟 [SAML 簽署憑證]  對話方塊。

    ![簽署選項](./media/secretserver-on-premises-tutorial/edit-saml-signon.png)

1. 針對 [簽署選項]  選取 [簽署 SAML 回應及判斷提示]  。

    ![簽署選項](./media/secretserver-on-premises-tutorial/signing-option.png)

1. 在 [設定 Secret Server (On-Premises)]  區段上，依據您的需求複製適當的 URL。

    ![複製組態 URL](common/copy-configuration-urls.png)

### <a name="configure-secret-server-on-premises-sso"></a>設定 Secret Server (On-Premises) SSO

若要設定 **Secret Server (On-Premises)** 端的單一登入，您必須將從 Azure 入口網站下載的 [憑證 (Base64)]  和複製的適當 URL 傳送給 [Secret Server (On-Premises) 支援小組](https://thycotic.force.com/support/s/)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `B.Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下頁面底部的 [新增]  。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Secret Server (On-Premises) 的存取權授與 B.Simon，讓她能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [Secret Server (On-Premises)]  。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。

   ![[使用者和群組] 連結](common/users-groups-blade.png)

1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。

    ![[新增使用者] 連結](common/add-assign-user.png)

1. 在 [使用者和群組]  對話方塊的 [使用者] 清單中選取 [B.Simon]  ，然後按一下畫面底部的 [選取]  按鈕。
1. 如果您在 SAML 判斷提示中需要任何角色值，請在 [選取角色]  對話方塊的清單中為使用者選取適當的角色，然後按一下畫面底部的 [選取]  按鈕。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

### <a name="create-secret-server-on-premises-test-user"></a>建立 Secret Server (On-Premises) 測試使用者

在本節中，您要在 Secret Server (On-Premises) 中建立名為 Britta Simon 的使用者。 請與  [Secret Server (On-Premises) 支援小組](https://thycotic.force.com/support/s/)合作，在 Secret Server (On-Premises) 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。

### <a name="test-sso"></a>測試 SSO

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 Secret Server (On-Premises) 圖格時，應該會自動登入您設定 SSO 的 Secret Server (On-Premises)。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。

## <a name="additional-resources"></a>其他資源

- [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什麼是 Azure Active Directory 中的條件式存取？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)