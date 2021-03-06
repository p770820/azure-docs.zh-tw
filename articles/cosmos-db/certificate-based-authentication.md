---
title: 以憑證為基礎的驗證與 Azure Cosmos DB 和 Active Directory
description: 瞭解如何設定 Azure AD 身分識別以進行憑證型驗證，以從 Azure Cosmos DB 存取金鑰。
author: voellm
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 06/11/2019
ms.author: tvoellm
ms.reviewer: sngun
ms.openlocfilehash: acdf268874b1dc1c24116ba36e2b4233a2702a5f
ms.sourcegitcommit: db2d402883035150f4f89d94ef79219b1604c5ba
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/07/2020
ms.locfileid: "77064490"
---
# <a name="certificate-based-authentication-for-an-azure-ad-identity-to-access-keys-from-an-azure-cosmos-db-account"></a>以憑證為基礎的驗證，用於從 Azure Cosmos DB 帳戶存取金鑰的 Azure AD 身分識別

憑證式驗證可讓您使用 Azure Active Directory (Azure AD) 搭配用戶端憑證，驗證您的用戶端應用程式。 您可以在需要身分識別的電腦 (例如 Azure 中的內部部署機器或虛擬機器) 上執行憑證式驗證。 您的應用程式可以讀取 Azure Cosmos DB 金鑰，而不需要直接在應用程式中擁有金鑰。 本文說明如何建立範例 Azure AD 應用程式、將其設定為以憑證為基礎的驗證、使用新的應用程式身分識別登入 Azure，然後從您的 Azure Cosmos 帳戶中抓取金鑰。 本文使用 Azure PowerShell 來設定身分識別，並提供C#範例應用程式來驗證和存取 Azure Cosmos 帳戶中的金鑰。  

## <a name="prerequisites"></a>必要條件

* 安裝[最新版](/powershell/azure/install-az-ps)的 Azure PowerShell。

* 如果您沒有 [Azure 訂用帳戶](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing)，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

## <a name="register-an-app-in-azure-ad"></a>在 Azure AD 中註冊應用程式

在此步驟中，您將在 Azure AD 帳戶中註冊範例 web 應用程式。 稍後會使用此應用程式從您的 Azure Cosmos DB 帳戶讀取金鑰。 使用下列步驟來註冊應用程式： 

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 [Azure **Active Directory** ] 窗格，移至**應用程式註冊**窗格，然後選取 [**新增註冊**]。 

   ![Active Directory 中的新應用程式註冊](./media/certificate-based-authentication/new-app-registration.png)

1. 在 [**註冊應用程式**] 表單中填入下列詳細資料：  

   * **名稱**–提供應用程式的名稱，它可以是任何名稱，例如 "sampleApp"。
   * **支援的帳戶類型**–選擇 [**僅限此組織目錄中的帳戶（預設目錄）** ]，以允許您目前的目錄中的資源存取此應用程式。 
   * 重新**導向 URL** –選擇**Web**類型的應用程式，並提供裝載您應用程式的 url，它可以是任何 url。 在此範例中，您可以提供測試 URL （例如 `https://sampleApp.com`，即使應用程式不存在也一樣）。

   ![註冊範例 web 應用程式](./media/certificate-based-authentication/register-sample-web-app.png)

1. 填寫表單之後，請選取 [**註冊**]。

1. 註冊應用程式之後，記下 [**應用程式（用戶端）識別碼**] 和 [**物件識別碼**]，您將在後續步驟中使用這些詳細資料。 

   ![取得應用程式和物件識別碼](./media/certificate-based-authentication/get-app-object-ids.png)

## <a name="install-the-azuread-module"></a>安裝 AzureAD 模組

在此步驟中，您將安裝 Azure AD PowerShell 模組。 需要此模組才能取得您在上一個步驟中註冊的應用程式識別碼，並將自我簽署憑證與該應用程式產生關聯。 

1. 以系統管理員許可權開啟 Windows PowerShell ISE。 如果您尚未這麼做，請安裝 AZ PowerShell 模組，並連接到您的訂用帳戶。 如果您有多個訂閱，您可以設定目前訂用帳戶的內容，如下列命令所示：

   ```powershell
   Install-Module -Name Az -AllowClobber
   Connect-AzAccount

   Get-AzSubscription
   $context = Get-AzSubscription -SubscriptionId <Your_Subscription_ID>
   Set-AzContext $context 
   ```

1. 安裝和匯入[AzureAD](/powershell/module/azuread/?view=azureadps-2.0)模組

   ```powershell
   Install-Module AzureAD
   Import-Module AzureAD 
   ```

## <a name="sign-into-your-azure-ad"></a>登入您的 Azure AD

登入您已註冊應用程式的 Azure AD。 使用 AzureAD 命令登入您的帳戶，並在快顯視窗中輸入您的 Azure 帳號憑證。 

```powershell
Connect-AzureAD 
```

## <a name="create-a-self-signed-certificate"></a>建立自我簽署憑證

開啟 Windows PowerShell ISE 的另一個實例，然後執行下列命令來建立自我簽署憑證，並讀取與憑證相關聯的金鑰：

```powershell
$cert = New-SelfSignedCertificate -CertStoreLocation "Cert:\CurrentUser\My" -Subject "CN=sampleAppCert" -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData()) 
```

## <a name="create-the-certificate-based-credential"></a>建立以憑證為基礎的認證 

接下來，執行下列命令以取得應用程式的物件識別碼，並建立以憑證為基礎的認證。 在此範例中，我們會將憑證設定為一年後到期，您可以將它設定為任何所需的結束日期。

```powershell
$application = Get-AzureADApplication -ObjectId <Object_ID_of_Your_Application>

New-AzureADApplicationKeyCredential -ObjectId $application.ObjectId -CustomKeyIdentifier "Key1" -Type AsymmetricX509Cert -Usage Verify -Value $keyValue -EndDate "2020-01-01"
```

上述命令會產生類似下列螢幕擷取畫面的輸出：

![以憑證為基礎的認證建立輸出](./media/certificate-based-authentication/certificate-based-credential-output.png)

## <a name="configure-your-azure-cosmos-account-to-use-the-new-identity"></a>設定您的 Azure Cosmos 帳戶以使用新的身分識別

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 流覽至您的 Azure Cosmos 帳戶，開啟 [**存取控制（IAM）** ] 分頁。

1. 選取 [**新增**並**新增角色指派**]。 新增您在上一個步驟中使用**參與者**角色建立的 sampleApp，如下列螢幕擷取畫面所示：

   ![設定 Azure Cosmos 帳戶以使用新的身分識別](./media/certificate-based-authentication/configure-cosmos-account-with-identify.png)

1. 填寫表單之後，選取 [**儲存**]

## <a name="register-your-certificate-with-azure-ad"></a>使用 Azure AD 註冊您的憑證

您可以在 Azure 入口網站的 Azure AD 中，將憑證型認證與用戶端應用程式建立關聯。 若要建立認證的關聯，您必須使用下列步驟來上傳憑證檔案：

在用戶端應用程式的 Azure 應用程式註冊中：

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 [Azure **Active Directory** ] 窗格，移至 [**應用程式註冊**] 窗格，然後開啟您在上一個步驟中建立的範例應用程式。 

1. 選取 [**憑證 & 密碼**]，然後**上傳 [憑證**]。 流覽您在上一個步驟中建立的憑證檔案，以進行上傳。

1. 選取 [新增]。 憑證上傳之後，即會顯示 [指紋]、[開始日期] 和 [到期] 值。

## <a name="access-the-keys-from-powershell"></a>從 PowerShell 存取金鑰

在此步驟中，您將使用應用程式和您建立的憑證來登入 Azure，並存取您的 Azure Cosmos 帳戶金鑰。 

1. 一開始會清除您用來登入帳戶的 Azure 帳號憑證。 您可以使用下列命令來清除認證：

   ```powershell
   Disconnect-AzAccount -Username <Your_Azure_account_email_id> 
   ```

1. 接下來，請驗證您可以使用應用程式的認證登入 Azure 入口網站，並存取 Azure Cosmos DB 金鑰：

   ```powershell
   Login-AzAccount -ApplicationId <Your_Application_ID> -CertificateThumbprint $cert.Thumbprint -ServicePrincipal -Tenant <Tenant_ID_of_your_application>

   Invoke-AzResourceAction -Action listKeys -ResourceType "Microsoft.DocumentDB/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName <Resource_Group_Name_of_your_Azure_Cosmos_account> -ResourceName <Your_Azure_Cosmos_Account_Name> 
   ```

上一個命令會顯示 Azure Cosmos 帳戶的主要和次要主要金鑰。 您可以查看 Azure Cosmos 帳戶的活動記錄，以驗證 get 金鑰要求成功，且事件是由 "sampleApp" 應用程式所起始。 
 
![驗證 Azure AD 中的 get 金鑰呼叫](./media/certificate-based-authentication/activity-log-validate-results.png)


## <a name="access-the-keys-from-a-c-application"></a>從C#應用程式存取金鑰 

您也可以從C#應用程式存取金鑰來驗證此案例。 下列C#主控台應用程式，可使用 Active Directory 中註冊的應用程式來存取 Azure Cosmos DB 金鑰。 執行程式碼之前，請務必先更新 tenantId、clientID、certName、資源組名、訂用帳戶識別碼、Azure Cosmos 帳戶名稱詳細資料。 

```csharp
using System;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Linq;
using System.Net.Http;
using System.Security.Cryptography.X509Certificates;
using System.Threading;
using System.Threading.Tasks;
 
namespace TodoListDaemonWithCert
{
    class Program
    {
        private static string aadInstance = "https://login.windows.net/";
        private static string tenantId = "<Your_Tenant_ID>";
        private static string clientId = "<Your_Client_ID>";
        private static string certName = "<Your_Certificate_Name>";
 
        private static int errorCode = 0;
        static int Main(string[] args)
        {
            MainAync().Wait();
            Console.ReadKey();
 
            return 0;
        } 
 
        static async Task MainAync()
        {
            string authContextURL = aadInstance + tenantId;
            AuthenticationContext authContext = new AuthenticationContext(authContextURL);
            X509Certificate2 cert = ReadCertificateFromStore(certName);
 
            ClientAssertionCertificate credential = new ClientAssertionCertificate(clientId, cert);
            AuthenticationResult result = await authContext.AcquireTokenAsync("https://management.azure.com/", credential);
            if (result == null)
            {
                throw new InvalidOperationException("Failed to obtain the JWT token");
            }
 
            string token = result.AccessToken;
            string subscriptionId = "<Your_Subscription_ID>";
            string rgName = "<ResourceGroup_of_your_Cosmos_account>";
            string accountName = "<Your_Cosmos_account_name>";
            string cosmosDBRestCall = $"https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.DocumentDB/databaseAccounts/{accountName}/listKeys?api-version=2015-04-08";
 
            Uri restCall = new Uri(cosmosDBRestCall);
            HttpClient httpClient = new HttpClient();
            httpClient.DefaultRequestHeaders.Remove("Authorization");
            httpClient.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);
            HttpResponseMessage response = await httpClient.PostAsync(restCall, null);
 
            Console.WriteLine("Got result {0} and keys {1}", response.StatusCode.ToString(), response.Content.ReadAsStringAsync().Result);
        }
 
        /// <summary>
        /// Reads the certificate
        /// </summary>
        private static X509Certificate2 ReadCertificateFromStore(string certName)
        {
            X509Certificate2 cert = null;
            X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
            store.Open(OpenFlags.ReadOnly);
            X509Certificate2Collection certCollection = store.Certificates;
 
            // Find unexpired certificates.
            X509Certificate2Collection currentCerts = certCollection.Find(X509FindType.FindByTimeValid, DateTime.Now, false);
 
            // From the collection of unexpired certificates, find the ones with the correct name.
            X509Certificate2Collection signingCert = currentCerts.Find(X509FindType.FindBySubjectName, certName, false);
 
            // Return the first certificate in the collection, has the right name and is current.
            cert = signingCert.OfType<X509Certificate2>().OrderByDescending(c => c.NotBefore).FirstOrDefault();
            store.Close();
            return cert;
        }
    }
}
```

此腳本會輸出主要和次要主要金鑰，如下列螢幕擷取畫面所示：

![csharp 應用程式輸出](./media/certificate-based-authentication/csharp-application-output.png)

類似上一節，您可以查看 Azure Cosmos 帳戶的活動記錄，以驗證「取得金鑰要求」事件是由「sampleApp」應用程式所起始。 


## <a name="next-steps"></a>後續步驟

* [使用 Azure Key Vault 保護 Azure Cosmos 金鑰](access-secrets-from-keyvault.md)

* [Azure Cosmos DB 的安全性控制項](cosmos-db-security-controls.md)
