---
title: 使用 OAuth 2.0 搭配 AAD 和 API 管理來保護 API
titleSuffix: Azure API Management
description: 了解如何使用 Azure Active Directory 與 API 管理保護 Web API 後端。
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 05/21/2019
ms.author: apimpm
ms.openlocfilehash: 8b396b782c1254b3229aeeb8e51b61cc744d6318
ms.sourcegitcommit: b07964632879a077b10f988aa33fa3907cbaaf0e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/13/2020
ms.locfileid: "77190371"
---
# <a name="protect-an-api-by-using-oauth-20-with-azure-active-directory-and-api-management"></a>使用 OAuth 2.0 搭配 Azure Active Directory 與 API 管理來保護 API

本指南說明如何使用 OAuth 2.0 通訊協定與 Azure Active Directory (Azure AD)，設定 Azure API 管理執行個體來保護 API。 

> [!NOTE]
> 這項功能適用于 API 管理的**開發人員**、**標準** **和高階**層次。

## <a name="prerequisites"></a>必要條件
若要依照本文中的步驟進行，您必須有：
* API 管理執行個體
* 正在使用 API 管理執行個體發佈的 API
* Azure AD 租用戶

## <a name="overview"></a>概觀

以下為步驟概述：

1. 在 Azure AD 中註冊應用程式 (後端應用程式) 來代表 API。
2. 在 Azure AD 中註冊另一個應用程式 (用戶端應用程式) 來代表需要呼叫 API 的用戶端應用程式。
3. 在 Azure AD 中授與權限，允許用戶端應用程式呼叫後端應用程式。
4. 將開發人員主控台設定為使用 OAuth 2.0 使用者授權呼叫 API。
5. 新增**驗證 JWT** 原則，以驗證每個連入要求的 OAuth 權杖。

## <a name="register-an-application-in-azure-ad-to-represent-the-api"></a>在 Azure AD 中註冊應用程式來代表 API

若要使用 Azure AD 保護 API，第一個步驟是在 Azure AD 中註冊一個應用程式來代表 API。 

1. 移至 [ [Azure 入口網站](https://portal.azure.com)] 以註冊您的應用程式。 搜尋並選取 [**應用程式註冊**]。

1. 選取 [新增註冊]。 

1. 當 [註冊應用程式] 頁面出現時，輸入您應用程式的註冊資訊： 
    - 在 [**名稱**] 區段中，輸入將對應用程式使用者顯示且有意義的應用程式名稱，例如 [*後端應用*程式]。 
    - 在 [**支援的帳戶類型**] 區段中，選取適合您案例的選項。 

1. 將 [重新**導向 URI** ] 區段保留空白。

1. 選取 [註冊] 以建立應用程式。 

1. 在應用程式 [概觀] 頁面上，尋找 [應用程式 (用戶端) 識別碼] 值並將它記下供稍後使用。

1. 選取 [**公開 API** ]，並以預設值設定 [**應用程式識別碼 URI** ]。 記錄此值以供稍後查看。

1. 選取 [**新增範圍**] 按鈕以顯示 [**新增範圍**] 頁面。 然後建立 API 支援的新範圍（例如 `Files.Read`）。 最後，選取 [**新增領域**] 按鈕以建立範圍。 重複此步驟，以新增 API 支援的所有範圍。

1. 建立範圍時，請記下它們以用於後續步驟。 

## <a name="register-another-application-in-azure-ad-to-represent-a-client-application"></a>在 Azure AD 中註冊另一個應用程式來代表用戶端應用程式

每個呼叫 API 的用戶端應用程式也必須在 Azure AD 中註冊為應用程式。 在此範例中，用戶端應用程式是 API 管理開發人員入口網站中的開發人員主控台。 以下說明如何在 Azure AD 中註冊另一個應用程式，來代表開發人員主控台。

1. 移至 [ [Azure 入口網站](https://portal.azure.com)] 以註冊您的應用程式。 搜尋並選取 [**應用程式註冊**]。

1. 選取 [新增註冊]。

1. 當 [註冊應用程式] 頁面出現時，輸入您應用程式的註冊資訊： 
    - 在 [**名稱**] 區段中，輸入將對應用程式使用者顯示且有意義的應用程式名稱，例如*用戶端應用*程式。 
    - 在 [**支援的帳戶類型**] 區段中，選取 [**任何組織目錄中的帳戶（任何 Azure AD 目錄-** 多租使用者）]。 

1. 在 [重新**導向 URI** ] 區段中，選取 [`Web`]，然後輸入 URL `https://contoso5.portal.azure-api.net/signin`。

1. 選取 [註冊] 以建立應用程式。 

1. 在應用程式 [概觀] 頁面上，尋找 [應用程式 (用戶端) 識別碼] 值並將它記下供稍後使用。

現在，請建立此應用程式的用戶端密碼，以便在後續步驟中使用。

1. 從用戶端應用程式的頁面清單中，選取 [**憑證 & 密碼**]，然後選取 [**新增用戶端密碼**]。

1. 在 [**新增用戶端密碼**] 底下，提供**描述**。 選擇金鑰應到期的時間，然後選取 [**新增**]。

建立密碼時，請記下要在後續步驟中使用的金鑰值。 

## <a name="grant-permissions-in-azure-ad"></a>在 Azure AD 中授與權限

現在您已註冊兩個應用程式來代表 API 和開發人員主控台，您需要授與權限，允許用戶端應用程式呼叫後端應用程式。  

1. 請移至[Azure 入口網站](https://portal.azure.com)，將許可權授與您的用戶端應用程式。 搜尋並選取 [**應用程式註冊**]。

1. 選擇您的用戶端應用程式。 然後在應用程式的頁面清單中，選取 [ **API 許可權**]。

1. 選取 [**新增許可權**]。

1. 在 [**選取 API**] 底下，選取 [**我的 api**]，然後尋找並選取您的後端應用程式。

1. 在 [**委派的許可權**] 底下，選取您後端應用程式的適當許可權，然後選取 [**新增許可權**]。

1. （選擇性）在 [ **API 許可權**] 頁面上，選取 [授與系統**管理員同意以 \<您的租使用者名稱] >** ，代表此目錄中的所有使用者授與同意。 

## <a name="enable-oauth-20-user-authorization-in-the-developer-console"></a>在開發人員主控台中啟用 OAuth 2.0 使用者授權

此時，您已在 Azure AD 中建立我們的應用程式，並已授與適當的權限，讓用戶端應用程式可以呼叫後端應用程式。 

在此範例中，開發人員主控台是用戶端應用程式。 下列步驟說明如何在開發人員主控台中啟用 OAuth 2.0 使用者授權。 

1. 在 Azure 入口網站中，流覽至您的 API 管理實例。

1. 選取 [OAuth 2.0] > [新增]。

1. 提供[顯示名稱] 和 [描述]。

1. 針對 [用戶端註冊頁面 URL]，輸入預留位置值，例如 `http://localhost`。 [**用戶端註冊頁面 URL** ] 指向使用者可用來建立及設定其專屬帳戶的頁面，供支援此的 OAuth 2.0 提供者使用。 在此範例中，使用者沒有建立和設定自己的帳戶，因此您可改使預留位置。

1. 針對 [授權授與類型]，選取 [授權碼]。

1. 指定 [授權端點 URL] 和 [權杖端點 URL]。 從 Azure AD 租用戶中的 [端點] 頁面擷取這些值。 瀏覽至 [應用程式註冊] 頁面，然後選取 [端點]。


1. 複製 [OAuth 2.0 授權端點] 並貼到 [授權端點 URL] 文字方塊中。 在 [授權要求方法] 底下，選取 [ **POST** ]。

1. 複製 [OAuth 2.0 權杖端點] 並貼到 [權杖端點 URL] 文字方塊中。 

    >[!IMPORTANT]
    > 您可以使用**v1**或**v2**端點。 不過，根據您選擇的版本而定，下列步驟將會不同。 我們建議使用 v2 端點。 

1. 如果您使用**v1**端點，請新增名為**resource**的主體參數。 針對此參數的值，請使用後端應用**程式的應用程式識別碼**。 

1. 如果您使用**v2**端點，請使用您在 [**預設範圍**] 欄位中為後端應用程式建立的範圍。

1. 接著，指定用戶端認證。 這些是用戶端應用程式的認證。

1. 針對 [**用戶端識別碼**]，使用用戶端應用程式的**應用程式識別碼**。

1. [用戶端密碼] 請使用您稍早為用戶端應用程式建立的金鑰。 

1. 緊接在用戶端密碼後面的是授權碼授與類型的 [redirect_url]。 記下此 URL。

1. 選取 [建立]。

1. 返回您的用戶端應用程式，然後選取 [**驗證**]。

1. 在 [重新**導向 uri**] 底下，選取 [ **Web**] 類型，在 [重新**導向 URI**] 底下貼上**redirect_url** ，然後按一下 [儲存]。

現在您已經設定 OAuth 2.0 授權伺服器，開發人員主控台可以從 Azure AD 取得存取權杖。 

下一個步驟是要讓您的 API 能夠使用 OAuth 2.0 使用者授權。 這可讓開發人員主控台知道它需要先代表使用者取得存取權杖，然後才能呼叫我們的 API。

1. 瀏覽至您的 API 管理執行個體，然後移至 [API]。

2. 選取您要保護的 API。 例如，您可以使用 `Echo API`。

3. 移至 [設定]。

4. 在 [安全性] 之下，選擇 [OAuth 2.0]，然後選取您先前設定的 OAuth 2.0 伺服器。 

5. 選取 [儲存]。

## <a name="successfully-call-the-api-from-the-developer-portal"></a>從開發人員入口網站成功呼叫 API

> [!NOTE]
> 本節不適用於不支援開發人員入口網站的**使用**層。

現在，您的 API 已啟用 OAuth 2.0 使用者授權，開發人員主控台會代表使用者取得存取權杖，然後才呼叫 API。

1. 在開發人員入口網站中，流覽至 API 底下的任何作業，然後選取 [**試試看**]。 這將帶您前往 [開發人員主控台]。

2. 請注意，在 [授權] 區段中有一個新項目對應到您剛才新增的授權伺服器。

3. 從授權下拉式清單中選取 [授權碼]，系統會提示您登入 Azure AD 租用戶。 如果您已經使用帳戶登入，就不會提示您。

4. 成功登入之後，要求中就會新增 `Authorization` 標頭，以及來自 Azure AD 的存取權杖。 以下是範例權杖 (Base64 編碼)：

   ```
   Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IlNTUWRoSTFjS3ZoUUVEU0p4RTJnR1lzNDBRMCIsImtpZCI6IlNTUWRoSTFjS3ZoUUVEU0p4RTJnR1lzNDBRMCJ9.eyJhdWQiOiIxYzg2ZWVmNC1jMjZkLTRiNGUtODEzNy0wYjBiZTEyM2NhMGMiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC80NDc4ODkyMC05Yjk3LTRmOGItODIwYS0yMTFiMTMzZDk1MzgvIiwiaWF0IjoxNTIxMTUyNjMzLCJuYmYiOjE1MjExNTI2MzMsImV4cCI6MTUyMTE1NjUzMywiYWNyIjoiMSIsImFpbyI6IkFWUUFxLzhHQUFBQUptVzkzTFd6dVArcGF4ZzJPeGE1cGp2V1NXV1ZSVnd1ZXZ5QU5yMlNkc0tkQmFWNnNjcHZsbUpmT1dDOThscUJJMDhXdlB6cDdlenpJdzJLai9MdWdXWWdydHhkM1lmaDlYSGpXeFVaWk9JPSIsImFtciI6WyJyc2EiXSwiYXBwaWQiOiJhYTY5ODM1OC0yMWEzLTRhYTQtYjI3OC1mMzI2NTMzMDUzZTkiLCJhcHBpZGFjciI6IjEiLCJlbWFpbCI6Im1pamlhbmdAbWljcm9zb2Z0LmNvbSIsImZhbWlseV9uYW1lIjoiSmlhbmciLCJnaXZlbl9uYW1lIjoiTWlhbyIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJpcGFkZHIiOiIxMzEuMTA3LjE3NC4xNDAiLCJuYW1lIjoiTWlhbyBKaWFuZyIsIm9pZCI6IjhiMTU4ZDEwLWVmZGItNDUxMS1iOTQzLTczOWZkYjMxNzAyZSIsInNjcCI6InVzZXJfaW1wZXJzb25hdGlvbiIsInN1YiI6IkFGaWtvWFk1TEV1LTNkbk1pa3Z3MUJzQUx4SGIybV9IaVJjaHVfSEM1aGciLCJ0aWQiOiI0NDc4ODkyMC05Yjk3LTRmOGItODIwYS0yMTFiMTMzZDk1MzgiLCJ1bmlxdWVfbmFtZSI6Im1pamlhbmdAbWljcm9zb2Z0LmNvbSIsInV0aSI6ImFQaTJxOVZ6ODBXdHNsYjRBMzBCQUEiLCJ2ZXIiOiIxLjAifQ.agGfaegYRnGj6DM_-N_eYulnQdXHhrsus45QDuApirETDR2P2aMRxRioOCR2YVwn8pmpQ1LoAhddcYMWisrw_qhaQr0AYsDPWRtJ6x0hDk5teUgbix3gazb7F-TVcC1gXpc9y7j77Ujxcq9z0r5lF65Y9bpNSefn9Te6GZYG7BgKEixqC4W6LqjtcjuOuW-ouy6LSSox71Fj4Ni3zkGfxX1T_jiOvQTd6BBltSrShDm0bTMefoyX8oqfMEA2ziKjwvBFrOjO0uK4rJLgLYH4qvkR0bdF9etdstqKMo5gecarWHNzWi_tghQu9aE3Z3EZdYNI_ZGM-Bbe3pkCfvEOyA
   ```

5. 選取 [傳送]，而您可以成功呼叫 API。


## <a name="configure-a-jwt-validation-policy-to-pre-authorize-requests"></a>設定 JWT 驗證原則來預先授權要求

此時，當使用者嘗試從開發人員主控台進行呼叫時，系統會提示使用者登入。 開發人員主控台會代表使用者取得存取權杖，並在對 API 提出的要求中包含權杖。

不過，如果有人呼叫您的 API，但沒有權杖或權杖無效？ 例如，嘗試在沒有 `Authorization` 標頭的情況下呼叫 API，呼叫仍然會繼續進行。 這是因為 API 管理此時不會驗證存取權杖。 它只會將 `Authorization` 標頭傳遞至後端 API。

您可以使用[驗證 JWT](api-management-access-restriction-policies.md#ValidateJWT) 原則，藉由驗證每個傳入要求的存取權杖來預先授權 API 管理中的要求。 如果要求沒有有效的權杖，則 API 管理會封鎖該要求。 例如，將下列原則新增至 `Echo API`的 `<inbound>` 原則區段。 它會檢查存取權杖中的 audience 宣告，並在權杖無效時傳回錯誤訊息。 如需如何設定原則的資訊，請參閱[設定或編輯原則](set-edit-policies.md)。

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/{aad-tenant}/.well-known/openid-configuration" />
    <required-claims>
        <claim name="aud">
            <value>{Application ID of backend-app}</value>
        </claim>
    </required-claims>
</validate-jwt>
```
> [!NOTE]
> 此 `openid-config` URL 會對應至 v1 端點。 針對 v2 `openid-config`端點，請使用 `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`。

## <a name="build-an-application-to-call-the-api"></a>建置應用程式來呼叫 API

在本指南中，您使用了 API 管理中的開發人員主控台作為範例用戶端應用程式，以呼叫 OAuth 2.0 所保護的 `Echo API`。 若要深入了解如何建置應用程式及實作 OAuth 2.0，請參閱 [Azure Active Directory 程式碼範例](../active-directory/develop/sample-v2-code.md)。

## <a name="next-steps"></a>後續步驟
* 深入了解 [Azure Active Directory 和 OAuth2.0](../active-directory/develop/authentication-scenarios.md)。
* 查看更多有關 API 管理的 [視訊](https://azure.microsoft.com/documentation/videos/index/?services=api-management) 。
* 如需其他保護後端服務的方式，請參閱[相互憑證驗證](api-management-howto-mutual-certificates.md)。

* [建立 API 管理服務執行個體](get-started-create-service-instance.md)。

* [管理第一個 API](import-and-publish.md)。
