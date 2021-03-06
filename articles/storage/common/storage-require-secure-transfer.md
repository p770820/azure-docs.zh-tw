---
title: 需要安全傳輸以確保連接安全
titleSuffix: Azure Storage
description: 瞭解如何要求 Azure 儲存體的要求的安全傳輸。 當您需要儲存體帳戶的安全傳輸時，來自不安全連線的任何要求都會遭到拒絕。
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/12/2019
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.openlocfilehash: 3b2d78bd929e23d49a57f337022f6678114bb5fe
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/25/2019
ms.locfileid: "75457438"
---
# <a name="require-secure-transfer-to-ensure-secure-connections"></a>需要安全傳輸以確保連接安全

您可以設定儲存體帳戶，以接受來自安全連線的要求，僅透過設定儲存體帳戶的 [**需要安全傳輸**] 屬性。 當您需要安全傳輸時，來自不安全連線的任何要求都會遭到拒絕。 Microsoft 建議您一律針對所有儲存體帳戶要求安全傳輸。

需要進行安全傳輸時，必須透過 HTTPS 呼叫 Azure 儲存體 REST API 作業。 任何透過 HTTP 提出的要求都會遭到拒絕。

當儲存體帳戶需要安全傳輸時，透過 SMB 連線至 Azure 檔案共用而不加密會失敗。 不安全連線的範例包括透過 SMB 2.1、沒有加密的 SMB 3.0，或某些 Linux SMB 用戶端版本所建立的連接。

根據預設，當您在 Azure 入口網站中建立儲存體帳戶時，就會啟用 [**需要安全傳輸**] 屬性。 不過，當您使用 SDK 建立儲存體帳戶時，它會停用。

> [!NOTE]
> 因為 Azure 儲存體針對自訂網域名稱並不支援 HTTPS，當您使用自訂網域名稱時，將不會套用此選項。 不支援傳統儲存體帳戶。

## <a name="require-secure-transfer-in-the-azure-portal"></a>Azure 入口網站中需要安全傳輸

當您在[Azure 入口網站](https://portal.azure.com)中建立儲存體帳戶時，可以開啟 [**需要安全傳輸**] 屬性。 您也可以為現有的儲存體帳戶啟用它。

### <a name="require-secure-transfer-for-a-new-storage-account"></a>針對新的儲存體帳戶要求使用安全傳輸

1. 在 Azure 入口網站中開啟 [建立儲存體帳戶] 窗格。
1. 在 [需要安全傳輸] 下，選取 [已啟用]。

   ![[建立儲存體帳戶] 刀鋒視窗](./media/storage-require-secure-transfer/secure_transfer_field_in_portal_en_1.png)

### <a name="require-secure-transfer-for-an-existing-storage-account"></a>針對現有儲存體帳戶要求使用安全傳輸

1. 在 Azure 入口網站中選取現有儲存體帳戶。
1. 在 [儲存體帳戶] 功能表窗格中，選取 [設定] 底下的 [組態]。
1. 在 [需要安全傳輸] 下，選取 [已啟用]。

   ![[儲存體帳戶] 功能表窗格](./media/storage-require-secure-transfer/secure_transfer_field_in_portal_en_2.png)

## <a name="require-secure-transfer-from-code"></a>需要從程式碼進行安全傳輸

若要以程式設計方式要求安全傳輸，請在儲存體帳戶上設定_supportsHttpsTrafficOnly_屬性。 您可以使用儲存體資源提供者 REST API、用戶端程式庫或工具來設定此屬性：

* [REST API](/rest/api/storagerp/storageaccounts)
* [PowerShell](/powershell/module/az.storage/set-azstorageaccount)
* [CLI](/cli/azure/storage/account)
* [NodeJS](https://www.npmjs.com/package/azure-arm-storage/)
* [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Management.Storage)
* [Python SDK](https://pypi.org/project/azure-mgmt-storage)
* [Ruby SDK](https://rubygems.org/gems/azure_mgmt_storage)

## <a name="require-secure-transfer-with-powershell"></a>需要使用 PowerShell 進行安全傳輸

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

此範例需要 Azure PowerShell 模組 Az 0.7 版或更新版本。 執行 `Get-Module -ListAvailable Az` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-Az-ps)。

執行 `Connect-AzAccount` 來建立與 Azure 的連線。

 使用下列命令列檢查設定：

```powershell
Get-AzStorageAccount -Name "{StorageAccountName}" -ResourceGroupName "{ResourceGroupName}"
StorageAccountName     : {StorageAccountName}
Kind                   : Storage
EnableHttpsTrafficOnly : False
...

```

使用下列命令列啟用設定：

```powershell
Set-AzStorageAccount -Name "{StorageAccountName}" -ResourceGroupName "{ResourceGroupName}" -EnableHttpsTrafficOnly $True
StorageAccountName     : {StorageAccountName}
Kind                   : Storage
EnableHttpsTrafficOnly : True
...

```

## <a name="require-secure-transfer-with-azure-cli"></a>需要 Azure CLI 的安全傳輸

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 使用下列命令來檢查設定：

```azurecli-interactive
az storage account show -g {ResourceGroupName} -n {StorageAccountName}
{
  "name": "{StorageAccountName}",
  "enableHttpsTrafficOnly": false,
  "type": "Microsoft.Storage/storageAccounts"
  ...
}

```

使用下列命令來啟用設定：

```azurecli-interactive
az storage account update -g {ResourceGroupName} -n {StorageAccountName} --https-only true
{
  "name": "{StorageAccountName}",
  "enableHttpsTrafficOnly": true,
  "type": "Microsoft.Storage/storageAccounts"
  ...
}

```

## <a name="next-steps"></a>後續步驟

[Blob 儲存體的安全性建議](../blobs/security-recommendations.md)
