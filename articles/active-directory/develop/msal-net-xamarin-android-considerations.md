---
title: Xamarin Android 考慮（MSAL.NET） |Azure
titleSuffix: Microsoft identity platform
description: 瞭解使用 Xamarin Android 搭配適用于 .NET 的 Microsoft 驗證程式庫（MSAL.NET）的考慮。
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/24/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 81b55253d757f641979c6f72001803d7d38d9af3
ms.sourcegitcommit: f718b98dfe37fc6599d3a2de3d70c168e29d5156
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/11/2020
ms.locfileid: "77132505"
---
# <a name="considerations-for-using-xamarin-android-with-msalnet"></a>使用 Xamarin Android 搭配 MSAL.NET 的考慮
本文討論當您使用 Xamarin Android 搭配適用于 .NET 的 Microsoft 驗證程式庫（MSAL.NET）時應考慮的事項。

## <a name="set-the-parent-activity"></a>設定父活動

在 Xamarin Android 上設定父活動，讓權杖在互動之後傳回。 以下是程式碼範例：

```csharp
var authResult = AcquireTokenInteractive(scopes)
 .WithParentActivityOrWindow(parentActivity)
 .ExecuteAsync();
```

在 MSAL 4.2 和更新版本中，您也可以在 `PublicClientApplication`層級設定這項功能。 若要這麼做，請使用回呼：

```csharp
// Requires MSAL.NET 4.2 or later
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => parentActivity)
  .Build();
```

如果您使用[CurrentActivityPlugin](https://github.com/jamesmontemagno/CurrentActivityPlugin)，則 `PublicClientApplication` 產生器程式碼看起來如下列範例所示。

```csharp
// Requires MSAL.NET 4.2 or later
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => CrossCurrentActivity.Current)
  .Build();
```

## <a name="ensure-that-control-returns-to-msal"></a>確定控制項返回 MSAL 
當驗證流程的互動部分結束時，請確定該控制項會回到 MSAL。 在 Android 上，覆寫 `Activity`的 `OnActivityResult` 方法。 然後呼叫 `AuthenticationContinuationHelper` MSAL 類別的 `SetAuthenticationContinuationEventArgs` 方法。 

以下是範例：

```csharp
protected override void OnActivityResult(int requestCode, 
                                         Result resultCode, Intent data)
{
 base.OnActivityResult(requestCode, resultCode, data);
 AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(requestCode,
                                                                         resultCode,
                                                                         data);
}

```

這一行可確保控制項在驗證流程的互動部分結束時，會回到 MSAL。

## <a name="update-the-android-manifest"></a>更新 Android 資訊清單
*Androidmanifest.xml*應該包含下列值：

<!--Intent filter to capture System Browser or Authenticator calling back to our app after sign-in-->
```
  <activity
        android:name="com.microsoft.identity.client.BrowserTabActivity">
     <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="msauth"
                android:host="Enter_the_Package_Name"
                android:path="/Enter_the_Signature_Hash" />
     </intent-filter>
 </activity>
```

以您在 Azure 入口網站中註冊的封裝名稱取代 `android:host=` 值。 以您在 Azure 入口網站中註冊的金鑰雜湊取代 `android:path=` 值。 簽章雜湊*不*應以 URL 編碼。 確定前置斜線（`/`）出現在簽章雜湊的開頭。

或者，[在程式碼中建立活動](https://docs.microsoft.com/xamarin/android/platform/android-manifest#the-basics)，而不是手動編輯*androidmanifest.xml*。 若要在程式碼中建立活動，請先建立一個包含 `Activity` 屬性和 `IntentFilter` 屬性的類別。 

以下是代表 XML 檔案值的類別範例：

```csharp
  [Activity]
  [IntentFilter(new[] { Intent.ActionView },
        Categories = new[] { Intent.CategoryBrowsable, Intent.CategoryDefault },
        DataHost = "auth",
        DataScheme = "msal{client_id}")]
  public class MsalActivity : BrowserTabActivity
  {
  }
```

### <a name="xamarinforms-43x-manifest"></a>如表單 4.3. X 資訊清單

Androidmanifest.xml 會產生程式碼，將 `package` 屬性設定為中 `com.companyname.{appName}`。 如果您使用 `DataScheme` 做為 `msal{client_id}`，則您可能會想要變更值，使其符合 `MainActivity.cs` 命名空間的值。

## <a name="use-the-embedded-web-view-optional"></a>使用內嵌的 web view （選擇性）

根據預設，MSAL.NET 會使用系統網頁瀏覽器。 此瀏覽器可讓您使用 web 應用程式和其他應用程式來取得單一登入（SSO）。 在某些罕見的情況下，您可能會想要讓系統使用內嵌的 web view。 

這個程式碼範例示範如何設定內嵌的 web view：

```csharp
bool useEmbeddedWebView = !app.IsSystemWebViewAvailable;

var authResult = AcquireTokenInteractive(scopes)
 .WithParentActivityOrWindow(parentActivity)
 .WithEmbeddedWebView(useEmbeddedWebView)
 .ExecuteAsync();
```

如需詳細資訊，請參閱[使用適用于 MSAL.NET 的網頁瀏覽器](msal-net-web-browsers.md)和[Xamarin Android 系統瀏覽器考慮](msal-net-system-browser-android-considerations.md)。


## <a name="troubleshoot"></a>疑難排解
您可以建立新的 Xamarin 應用程式，並新增 MSAL.NET NuGet 套件的參考。
但是，如果您將現有的 Xamarin Forms 應用程式升級為 MSAL.NET preview 1.1.2 或更新版本，可能會發生組建問題。

若要針對組建問題進行疑難排解：

- 將現有的 MSAL.NET NuGet 套件更新為 MSAL.NET preview 1.1.2 或更新版本。
- 檢查 [Xamarin] 是否已自動更新為 [版本 2.5.0.122203]。 如有必要，請將 Xamarin 更新為此版本。
- 檢查是否已自動將支援更新為版本25.4.0.2。 如有必要，請更新至版本25.4.0.2。
- 確定所有的25.4.0.2 支援套件都以版本為目標。
- 清除或重建應用程式。
- 在 Visual Studio 中，嘗試將 [平行專案組建的最大數目] 設定為1。 若要這麼做，請選取 [**選項**] > [**專案和方案**] > **組建並執行** > [**平行專案的最大數目] 組建**。
- 如果您是從命令列建立，而您的命令使用 `/m`，請嘗試從命令中移除此元素。

### <a name="error-the-name-authenticationcontinuationhelper-doesnt-exist-in-the-current-context"></a>錯誤：名稱 AuthenticationContinuationHelper 不存在於目前的內容中

如果錯誤指出 `AuthenticationContinuationHelper` 不存在於目前的內容中，Visual Studio 可能不正確地更新 Android .csproj * 檔案。 有時候 *\<HintPath >* 檔案路徑不正確地包含*netstandard13* ，而不是*monoandroid90*。

這個範例包含正確的檔案路徑：

```xml
<Reference Include="Microsoft.Identity.Client, Version=3.0.4.0, Culture=neutral, PublicKeyToken=0a613f4dd989e8ae,
           processorArchitecture=MSIL">
  <HintPath>..\..\packages\Microsoft.Identity.Client.3.0.4-preview\lib\monoandroid90\Microsoft.Identity.Client.dll</HintPath>
</Reference>
```

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱[使用 Microsoft 身分識別平臺之 Xamarin 行動應用程式](https://github.com/azure-samples/active-directory-xamarin-native-v2#android-specific-considerations)的範例。 下表摘要說明讀我檔案中的相關資訊。

| 範例 | 平台 | 描述 |
| ------ | -------- | ----------- |
|[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin iOS、Android、UWP | 簡單的 Xamarin 應用程式，示範如何使用 MSAL 來驗證 Microsoft 個人帳戶，並透過 Azure AD 2.0 端點 Azure AD。 應用程式也會顯示如何存取 Microsoft Graph，並顯示產生的權杖。 <br>![拓撲](media/msal-net-xamarin-android-considerations/topology.png) |
