---
title: 從基本公用升級至標準公用 Azure Load Balancer
description: 本文說明如何將 Azure 公用 Load Balancer 從基本 SKU 升級至標準 SKU
services: load-balancer
author: irenehua
ms.service: load-balancer
ms.topic: article
ms.date: 01/23/2020
ms.author: irenehua
ms.openlocfilehash: 179d0ff8143b526e100b89cffbbac0bbc29ca3e1
ms.sourcegitcommit: 984c5b53851be35c7c3148dcd4dfd2a93cebe49f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2020
ms.locfileid: "76776660"
---
# <a name="upgrade-azure-public-load-balancer-from-basic-sku-to-standard-sku"></a>將 Azure 公用 Load Balancer 從基本 SKU 升級至標準 SKU
[Azure Standard Load Balancer](load-balancer-overview.md)透過區域冗余提供了一組豐富的功能和高可用性。 若要深入瞭解 Load Balancer SKU，請參閱[比較表](https://docs.microsoft.com/azure/load-balancer/concepts-limitations#skus)。

升級有兩個階段：

1. 遷移設定
2. 將 Vm 新增至 Standard Load Balancer 的後端集區

本文涵蓋設定遷移。 將 Vm 新增至後端集區可能會根據您的特定環境而有所不同。 不過，[系統會提供](#add-vms-to-backend-pools-of-standard-load-balancer)一些高階的一般建議。

## <a name="upgrade-overview"></a>升級概觀

有 Azure PowerShell 腳本可執行下列動作：

* 在資源群組和您指定的位置中建立標準公用 SKU Load Balancer。
* 將基本 SKU 公用 Load Balancer 的設定順暢地複製到新建立的標準公用 Load Balancer。

### <a name="caveatslimitations"></a>Caveats\Limitations

* 腳本僅支援公用 Load Balancer 升級。 針對內部基本 Load Balancer 升級，如果不想要輸出連線能力，請建立標準的內部 Load Balancer，並建立標準的內部 Load Balancer 和標準公用 Load Balancer （如果需要輸出連線能力）。
* Standard Load Balancer 具有新的公用位址。 不可能將與現有基本 Load Balancer 相關聯的 IP 位址順暢地移動到 Standard Load Balancer，因為它們有不同的 Sku。
* 如果在不同的區域中建立標準負載平衡器，您將無法將舊區域中現有的 Vm 與新建立的 Standard Load Balancer 建立關聯。 若要解決這項限制，請務必在新的區域中建立新的 VM。
* 如果您的 Load Balancer 沒有任何前端 IP 設定或後端集區，您可能會遇到執行腳本的錯誤。 請確認它們不是空的。

## <a name="download-the-script"></a>下載腳本

從[PowerShell 資源庫](https://www.powershellgallery.com/packages/AzurePublicLBUpgrade/1.0)下載遷移腳本。
## <a name="use-the-script"></a>使用腳本

有兩個選項可供您選擇，視您的本機 PowerShell 環境設定和偏好而定：

* 如果您未安裝 Azure Az 模組，或不想卸載 Azure Az 模組，最好的選擇是使用 [`Install-Script`] 選項來執行腳本。
* 如果您需要保留 Azure Az 模組，最好是下載並直接執行腳本。

若要判斷您是否已安裝 Azure Az 模組，請執行 `Get-InstalledModule -Name az`。 如果您看不到任何已安裝的 Az 模組，則可以使用 `Install-Script` 方法。

### <a name="install-using-the-install-script-method"></a>使用安裝腳本方法進行安裝

若要使用此選項，您的電腦上不得安裝 Azure Az 模組。 如果已安裝，下列命令就會顯示錯誤。 您可以卸載 Azure Az 模組，或使用另一個選項手動下載腳本並加以執行。
  
使用下列命令來執行指令碼：

`Install-Script -Name AzurePublicLBUpgrade`

此命令也會安裝必要的 Az 模組。  

### <a name="install-using-the-script-directly"></a>直接使用腳本安裝

如果您已安裝一些 Azure Az 模組，但無法將它們卸載（或不想要將它們卸載），您可以使用腳本下載連結中的 [**手動下載**] 索引標籤，手動下載腳本。 腳本會下載為原始的 nupkg 檔案。 若要從這個 nupkg 檔安裝腳本，請參閱[手動套件下載](/powershell/scripting/gallery/how-to/working-with-packages/manual-download)。

執行指令碼：

1. 使用 `Connect-AzAccount` 來連接到 Azure。

1. 使用 `Import-Module Az` 匯入 Az 模組。

1. 執行 `Get-Help AzureLBUpgrade.ps1` 以檢查必要的參數：

   ```
   AzurePublicLBUpgrade.ps1
    -oldRgName <name of the Resource Group where Basic Load Balancer exists>
    -oldLBName <name of existing Basic Load Balancer>
    -newrgName <Name of the Resource Group where the new Standard Load Balancer will be created>
    -newlocation <Name of the location where the new Standard Load Balancer will be created>
    -newLBName <Name of the Standard Load Balancer to be created>
   ```
   腳本的參數：
   * **oldRgName： [String]： Required** –這是您想要升級的現有基本 Load Balancer 的資源群組。 若要尋找此字串值，請流覽至 Azure 入口網站，選取您的基本 Load Balancer 來源，然後按一下負載平衡器的**總覽**。 資源群組位於該頁面。
   * **oldLBName： [String]： Required** –這是您想要升級的現有基本平衡器名稱。 
   * **newrgName： [String]： Required** –這是將在其中建立 Standard Load Balancer 的資源群組。 它可以是新的資源群組或現有的。 如果您選擇現有的資源群組，請注意，Load Balancer 的名稱在資源群組內必須是唯一的。 
   * **newlocation： [String]： Required** –這是將在其中建立 Standard Load Balancer 的位置。 建議將所選基本 Load Balancer 的相同位置繼承到 Standard Load Balancer，以便與其他現有資源更好的關聯。
   * **newLBName： [String]： Required** –這是要建立之 Standard Load Balancer 的名稱。
1. 使用適當的參數來執行腳本。 可能需要五到七分鐘的時間才能完成。

    **範例**

   ```azurepowershell
   ./AzurePublicLBUpgrade.ps1 -oldRgName "test_publicUpgrade_rg" -oldLBName "LBForPublic" -newrgName "test_userInput3_rg" -newlocation "centralus" -newLbName "LBForUpgrade"
   ```

### <a name="add-vms-to-backend-pools-of-standard-load-balancer"></a>將 Vm 新增至 Standard Load Balancer 的後端集區

首先，再次檢查腳本是否已成功建立新的標準公用 Load Balancer，並從您的基本公用 Load Balancer 遷移確切的設定。 您可以從 [Azure 入口網站] 進行驗證。

請務必透過 Standard Load Balancer 來傳送少量的流量以進行手動測試。
  
以下幾個案例會說明如何將 Vm 新增至新建立之標準公用 Load Balancer 的後端集區，以及我們對每個設定的建議：

* **將現有的 vm 從舊版基本公用 Load Balancer 的後端集區移到新建立之標準公用 Load Balancer 的後端**集區。
    1. 若要進行本快速入門中的工作，請登入 [Azure 入口網站](https://portal.azure.com)。
 
    1. 選取左側功能表上的 [**所有資源**]，然後從資源清單中選取**新建立的 Standard Load Balancer** 。
   
    1. 在 [設定] 底下選取 [後端集區]。
   
    1. 選取符合基本 Load Balancer 後端集區的後端集區，然後選取下列值： 
      - **虛擬機器**：下拉並從基本 Load Balancer 的相符後端集區中選取 vm。
    1. 選取 [儲存]。
    >[!NOTE]
    >針對具有公用 Ip 的 Vm，您必須先建立標準 IP 位址，而不保證會有相同的 IP 位址。 將 Vm 與基本 Ip 解除關聯，並將它們與新建立的標準 IP 位址建立關聯。 接著，您將能夠遵循指示，將 Vm 新增至 Standard Load Balancer 的後端集區。 

* **建立新的 vm，以新增至新建立之標準公用 Load Balancer 的後端**集區。
    * 如需有關如何建立 VM 並將它與 Standard Load Balancer 相關聯的詳細指示，請參閱[這裡](https://docs.microsoft.com/azure/load-balancer/quickstart-load-balancer-standard-public-portal#create-virtual-machines)。

## <a name="common-questions"></a>常見問題

### <a name="are-there-any-limitations-with-the-azure-powershell-script-to-migrate-the-configuration-from-v1-to-v2"></a>Azure PowerShell 腳本是否有任何限制，可將設定從 v1 遷移至 v2？

可以。 請參閱[警告/限制](#caveatslimitations)。

### <a name="does-the-azure-powershell-script-also-switch-over-the-traffic-from-my-basic-load-balancer-to-the-newly-created-standard-load-balancer"></a>Azure PowerShell 腳本是否也會將來自我的基本 Load Balancer 的流量切換到新建立的 Standard Load Balancer？

不會。 Azure PowerShell 腳本只會遷移設定。 實際的流量遷移是您在控制中的責任。

### <a name="i-ran-into-some-issues-with-using-this-script-how-can-i-get-help"></a>我在使用此腳本時遇到一些問題。 如何取得協助？
  
您可以傳送電子郵件給 slbupgradesupport@microsoft.com、向 Azure 支援服務開啟支援案例，或兩者都執行。

## <a name="next-steps"></a>後續步驟

[了解 Standard Load Balancer](load-balancer-overview.md)
