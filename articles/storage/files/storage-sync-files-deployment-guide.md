---
title: 部署 Azure 檔案同步 | Microsoft Docs
description: 了解部署 Azure 檔案同步的完整程序。
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 07/19/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 4f9a2842f99c7f8b0bb9f820584fb2cd4e41a2b2
ms.sourcegitcommit: a5ebf5026d9967c4c4f92432698cb1f8651c03bb
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/08/2019
ms.locfileid: "74927885"
---
# <a name="deploy-azure-file-sync"></a>部署 Azure 檔案同步
使用 Azure 檔案同步，將組織的檔案共用集中在 Azure 檔案服務中，同時保有內部部署檔案伺服器的靈活度、效能及相容性。 Azure 檔案同步會將 Windows Server 轉換成 Azure 檔案共用的快速快取。 您可以使用 Windows Server 上可用的任何通訊協定以從本機存取資料，包括 SMB、NFS 和 FTPS。 您可以視需要存取多個散佈於世界各地的快取。

強烈建議您先閱讀[規劃 Azure 檔案服務部署](storage-files-planning.md)和[規劃 Azure 檔案同步部署](storage-sync-files-planning.md)，再完成本文章中描述的步驟。

## <a name="prerequisites"></a>必要條件
* 您想要部署 Azure 檔案同步的相同區域中的 Azure 檔案共用。如需詳細資訊，請參閱：
    - Azure 檔案同步的[區域可用性](storage-sync-files-planning.md#region-availability)。
    - [建立檔案共用](storage-how-to-create-file-share.md)以取得如何建立檔案共用的逐步說明。
* 至少一個支援的 Windows Server 或 Windows Server cluster 實例與 Azure 檔案同步同步。如需有關支援的 Windows Server 版本的詳細資訊，請參閱[與 Windows server 的互通性](storage-sync-files-planning.md#azure-file-sync-system-requirements-and-interoperability)。
* Az PowerShell 模組可用於 PowerShell 5.1 或 PowerShell 6 +。 您可以在任何支援的系統（包括非 Windows 系統）上使用 Az PowerShell 模組來進行 Azure 檔案同步，不過，伺服器註冊 Cmdlet 一律必須在您要註冊的 Windows Server 實例上執行（可以直接或透過 PowerShell 來執行）遠端處理）。 在 Windows Server 2012 R2 上，您可以確認您至少執行 PowerShell 5.1。藉由查看 **$PSVersionTable**物件的**PSVersion**屬性值來\*：

    ```powershell
    $PSVersionTable.PSVersion
    ```

    如果您的 PSVersion 值小於 5.1.\* (大部分的 Windows Server 2012 R2 全新安裝都會發生此類情況)，您可以下載並安裝 [Windows Management Framework (WMF) 5.1](https://www.microsoft.com/download/details.aspx?id=54616) \(英文\) 來輕鬆地升級。 適用於 Windows Server 2012 R2 下載和安裝的適當套件為 **Win8.1AndW2K12R2-KB\*\*\*\*\*\*\*-x64.msu**。 

    PowerShell 6 + 可以搭配任何支援的系統使用，並可透過其[GitHub 頁面](https://github.com/PowerShell/PowerShell#get-powershell)下載。 

    > [!Important]  
    > 如果您打算使用伺服器註冊 UI，而不是直接從 PowerShell 註冊，則必須使用 PowerShell 5.1。

* 如果您選擇使用 PowerShell 5.1，請確定至少已安裝 .NET 4.7.2。 深入瞭解您系統上的[.NET Framework 版本和](https://docs.microsoft.com/dotnet/framework/migration-guide/versions-and-dependencies)相依性。

    > [!Important]  
    > 如果您要在 Windows Server Core 上安裝 .NET 4.7.2 +，您必須使用 `quiet` 和 `norestart` 旗標來安裝，否則安裝將會失敗。 例如，如果安裝 .NET 4.8，命令看起來會像下面這樣：
    > ```PowerShell
    > Start-Process -FilePath "ndp48-x86-x64-allos-enu.exe" -ArgumentList "/q /norestart" -Wait
    > ```

* 您可以遵循這裡的指示來安裝 Az PowerShell 模組：[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps)。
     
    > [!Note]  
    > 當您安裝 Az PowerShell 模組時，Microsoft.storagesync 模組現在會自動安裝。

## <a name="prepare-windows-server-to-use-with-azure-file-sync"></a>準備 Windows Server 以搭配 Azure 檔案同步使用
針對要與 Azure 檔案同步搭配使用的每個伺服器 (包括容錯移轉叢集中的每個伺服器節點)，停用 [Internet Explorer 增強式安全性設定]。 此動作只需要在初始伺服器註冊時執行。 您可以在註冊伺服器後重新啟用它。

# <a name="portaltabazure-portal"></a>[入口網站](#tab/azure-portal)
> [!Note]  
> 如果您要在 Windows Server Core 上部署 Azure 檔案同步，可以略過此步驟。

1. 開啟 [伺服器管理員]。
2. 按一下 [本機伺服器]：  
    ![位於伺服器管理員 UI 左側的 [本機伺服器]](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-1.PNG)
3. 選取 [屬性] 子窗格上的 [IE 增強式安全性設定] 連結。  
    ![位於伺服器管理員 UI 中的 [IE 增強式安全性設定] 窗格](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-2.PNG)
4. 在 [Internet Explorer 增強式安全性設定] 對話方塊中，針對 [系統管理員] 和 [使用者] 選取 [關閉]：  
    ![已選取 [關閉] 的 [Internet Explorer 增強式安全性設定] 快顯視窗](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-3.png)

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
若要停用 [Internet Explorer 增強式安全性設定]，請從提升權限的 PowerShell 工作階段中執行下列命令：

```powershell
$installType = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\").InstallationType

# This step is not required for Server Core
if ($installType -ne "Server Core") {
    # Disable Internet Explorer Enhanced Security Configuration 
    # for Administrators
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force
    
    # Disable Internet Explorer Enhanced Security Configuration 
    # for Users
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A8-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force
    
    # Force Internet Explorer closed, if open. This is required to fully apply the setting.
    # Save any work you have open in the IE browser. This will not affect other browsers,
    # including Microsoft Edge.
    Stop-Process -Name iexplore -ErrorAction SilentlyContinue
}
``` 

---

## <a name="deploy-the-storage-sync-service"></a>部署儲存體同步服務 
部署 Azure 檔案同步的第一步，先將**儲存體同步服務**資源放到所選訂用帳戶的資源群組中。 建議您佈建需要的少數資源群組即可。 您將在伺服器與此資源之間建立信任關係，而一個伺服器只能註冊到一個儲存體同步服務。 因此，建議您根據需要部署多個儲存體同步服務，以分隔伺服器群組。 請記住，不同儲存體同步服務中的伺服器不可以彼此同步。

> [!Note]
> 儲存體同步服務會從訂用帳戶及其已部署的資源群組繼承存取權限。 我們建議您仔細檢查誰可以存取此服務。 具有寫入存取權的實體可以開始將新的一組檔案從已註冊伺服器同步到此儲存體同步服務，並讓資料流向實體可存取的 Azure 儲存體。

# <a name="portaltabazure-portal"></a>[入口網站](#tab/azure-portal)
若要部署儲存體同步服務，請移至[Azure 入口網站](https://portal.azure.com/)，按一下 [*建立資源*]，然後搜尋 Azure 檔案同步。在搜尋結果中，選取 [ **Azure 檔案同步**]，然後選取 [**建立**] 以開啟 [**部署儲存體同步**] 索引標籤。

在開啟的窗格中，輸入下列資訊：

- **名稱**：儲存體同步服務的唯一名稱 (每一訂用帳戶)。
- **訂用帳戶**：您要在其中建立儲存體同步服務的訂用帳戶。 視您組織的設定策略而定，您可能具有一或多個訂用帳戶的存取權限。 Azure 訂用帳戶是針對每個雲端服務 (例如 Azure 檔案服務) 計費的最基本容器。
- **資源群組**：資源群組是 Azure 資源的邏輯群組，例如儲存體帳戶或儲存體同步服務。 您可以建立新的資源群組，或使用現有的資源群組來進行 Azure 檔案同步。（建議使用資源群組作為容器，以邏輯方式為組織隔離資源，例如將 HR 資源或特定專案的資源分組）。
- **位置**：您要在其中部署 Azure 檔案同步的區域。只有支援的區域可在此清單中取得。

完成時，請選取 [建立] 來部署儲存體同步服務。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
以您自己的值取代 **< Az_Region >** 、 **< RG_Name >** 和 **<** my_storage_sync_service >，然後使用下列的 []，建立並部署儲存體同步服務：

```powershell
$hostType = (Get-Host).Name

if ($installType -eq "Server Core" -or $hostType -eq "ServerRemoteHost") {
    Connect-AzAccount -UseDeviceAuthentication
}
else {
    Connect-AzAccount
}

# this variable holds the Azure region you want to deploy 
# Azure File Sync into
$region = '<Az_Region>'

# Check to ensure Azure File Sync is available in the selected Azure
# region.
$regions = @()
Get-AzLocation | ForEach-Object { 
    if ($_.Providers -contains "Microsoft.StorageSync") { 
        $regions += $_.Location 
    } 
}

if ($regions -notcontains $region) {
    throw [System.Exception]::new("Azure File Sync is either not available in the selected Azure Region or the region is mistyped.")
}

# the resource group to deploy the Storage Sync Service into
$resourceGroup = '<RG_Name>'

# Check to ensure resource group exists and create it if doesn't
$resourceGroups = @()
Get-AzResourceGroup | ForEach-Object { 
    $resourceGroups += $_.ResourceGroupName 
}

if ($resourceGroups -notcontains $resourceGroup) {
    New-AzResourceGroup -Name $resourceGroup -Location $region
}

$storageSyncName = "<my_storage_sync_service>"
$storageSync = New-AzStorageSyncService -ResourceGroupName $resourceGroup -Name $storageSyncName -Location $region
```

---

## <a name="install-the-azure-file-sync-agent"></a>安裝 Azure 檔案同步代理程式
Azure 檔案同步代理程式是可下載的套件，可讓 Windows Server 能夠和 Azure 檔案共用進行同步處理。 

# <a name="portaltabazure-portal"></a>[入口網站](#tab/azure-portal)
您可以從 [Microsoft 下載中心](https://go.microsoft.com/fwlink/?linkid=858257) \(英文\) 下載該代理程式。 下載之後，請按兩下 MSI 套件以啟動 Azure 檔案同步代理程式安裝。

> [!Important]  
> 如果您打算將 Azure 檔案同步搭配容錯移轉叢集使用，則必須在叢集中的每個節點上安裝 Azure 檔案同步代理程式。 叢集中的每個節點都必須註冊，才能與 Azure 檔案同步搭配使用。

建議您採取下列動作︰
- 保留預設的安裝路徑 (C:\Program Files\Azure\StorageSyncAgent)，以簡化疑難排解和伺服器維護。
- 啟用 Microsoft Update 以將 Azure 檔案同步保持在最新狀態。 Azure 檔案同步代理程式的所有更新 (包括功能更新和 Hotfix) 都會從 Microsoft Update 進行。 建議您將最新的更新安裝到 Azure 檔案同步。如需詳細資訊，請參閱[Azure 檔案同步更新原則](storage-sync-files-planning.md#azure-file-sync-agent-update-policy)。

當 Azure 檔案同步代理程式安裝結束時，伺服器註冊 UI 會自動開啟。 您必須先有儲存體同步服務才能註冊；請參閱下一節以了解如何建立儲存體同步服務。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
執行下列 PowerShell 程式碼，下載適合您作業系統的 Azure 檔案同步代理程式版本，並將其安裝在您的系統上。

> [!Important]  
> 如果您打算將 Azure 檔案同步搭配容錯移轉叢集使用，則必須在叢集中的每個節點上安裝 Azure 檔案同步代理程式。 叢集中的每個節點都必須經過註冊，才能使用 Azure 檔案同步。

```powershell
# Gather the OS version
$osver = [System.Environment]::OSVersion.Version

# Download the appropriate version of the Azure File Sync agent for your OS.
if ($osver.Equals([System.Version]::new(10, 0, 17763, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2019 `
        -OutFile "StorageSyncAgent.msi" 
} elseif ($osver.Equals([System.Version]::new(10, 0, 14393, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2016 `
        -OutFile "StorageSyncAgent.msi" 
} elseif ($osver.Equals([System.Version]::new(6, 3, 9600, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2012R2 `
        -OutFile "StorageSyncAgent.msi" 
} else {
    throw [System.PlatformNotSupportedException]::new("Azure File Sync is only supported on Windows Server 2012 R2, Windows Server 2016, and Windows Server 2019")
}

# Install the MSI. Start-Process is used to PowerShell blocks until the operation is complete.
# Note that the installer currently forces all PowerShell sessions closed - this is a known issue.
Start-Process -FilePath "StorageSyncAgent.msi" -ArgumentList "/quiet" -Wait

# Note that this cmdlet will need to be run in a new session based on the above comment.
# You may remove the temp folder containing the MSI and the EXE installer
Remove-Item -Path ".\StorageSyncAgent.msi" -Recurse -Force
```

---

## <a name="register-windows-server-with-storage-sync-service"></a>向儲存體同步服務註冊 Windows Server
向儲存體同步服務註冊 Windows Server 會在您的伺服器 (或叢集) 與儲存體同步服務之間建立信任關係。 一部伺服器只能向一個儲存體同步服務註冊，但可以與其他關聯至相同儲存體同步服務的伺服器和 Azure 檔案共用同步。

> [!Note]
> 伺服器註冊會使用您的 Azure 認證來建立儲存體同步服務和 Windows Server 之間的信任關係；但是，之後伺服器會建立並使用自有的身分識別，只要伺服器維持註冊狀態，且目前的共用存取簽章權杖 (儲存體 SAS) 有效，此身分識別就會是有效的。 如果伺服器未註冊，新的 SAS 權杖就無法發行至伺服器，並會因此移除伺服器存取您 Azure 檔案共用的能力，進而停止任何同步。

# <a name="portaltabazure-portal"></a>[入口網站](#tab/azure-portal)
安裝 Azure 檔案同步代理程式之後，伺服器註冊 UI 應該會自動開啟。 如果沒有，您可以從其檔案位置手動開啟它：C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe。 伺服器註冊 UI 開啟時，請選取 [登入] 以開始。

登入之後，系統會提示您輸入下列資訊：

![伺服器註冊 UI 的螢幕擷取畫面](media/storage-sync-files-deployment-guide/register-server-scubed-1.png)

- **Azure 訂用帳戶**：包含儲存體同步服務的訂用帳戶 (請參閱[部署儲存體同步服務](#deploy-the-storage-sync-service))。 
- **資源群組**：包含儲存體同步服務的資源群組。
- **儲存體同步服務**：您要註冊的目標儲存體同步服務名稱。

選取適當的資訊之後，請選取 [註冊] 以完成伺服器註冊。 在註冊過程中，系統會提示您額外再登入一次。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
$registeredServer = Register-AzStorageSyncServer -ParentObject $storageSync
```

---

## <a name="create-a-sync-group-and-a-cloud-endpoint"></a>建立同步群組和雲端端點
同步群組定義一組檔案的同步拓撲。 同步群組內的端點會與彼此保持同步。 同步群組必須包含一個雲端端點，代表 Azure 檔案共用和一個或多個伺服器端點。 伺服器端點代表已註冊伺服器上的路徑。 伺服器可以在多個同步群組中有多個伺服器端點。 您可以根據需要建立多個同步群組，以適當地描述您想要的同步拓撲。

雲端端點是指向 Azure 檔案共用的指標。 所有伺服器端點都會與雲端端點同步，使雲端端點成為中樞。 Azure 檔案共用的儲存體帳戶必須位於儲存體同步服務的所在區域。 Azure 檔案共用的全部內容都會同步，但有一個例外狀況：系統會佈建一個特殊資料夾，其相當於 NTFS 磁碟區上隱藏的「系統磁碟區資訊」資料夾。 此目錄稱為 ".SystemShareInformation"。 它包含重要的同步中繼資料，而這不會同步到其他端點。 請勿使用或刪除此資料夾！

> [!Important]  
> 您可以對同步群組中的任何雲端端點或伺服器端點進行變更，您的檔案將會與同步群組中的其他端點同步。 如果直接對雲端端點 (Azure 檔案共用) 進行變更，則必須先由 Azure 檔案同步變更偵測作業探索到該變更。 針對雲端端點的變更偵測作業，每隔 24 小時才會起始一次。 如需詳細資訊，請參閱 [Azure 檔案服務常見問題集](storage-files-faq.md#afs-change-detection)。

# <a name="portaltabazure-portal"></a>[入口網站](#tab/azure-portal)
若要建立同步群組，請在 [Azure 入口網站](https://portal.azure.com/)中移至您的儲存體同步服務，然後選取 [+ 同步群組]：

![在 Azure 入口網站中建立新的同步群組](media/storage-sync-files-deployment-guide/create-sync-group-1.png)

在開啟的窗格中，輸入下列資訊以建立具有雲端端點的同步群組：

- **同步群組名稱**：要建立的同步群組名稱。 此名稱在儲存體同步服務中必須是唯一的，但可以是任何對您而言合理的名稱。
- **訂用帳戶**：您在[部署儲存體同步服務](#deploy-the-storage-sync-service)中部署儲存體同步服務的訂用帳戶。
- **儲存體帳戶**：如果您選取 [選取儲存體帳戶]，則會顯示另一個窗格，您可在其中選取具有您想要同步之 Azure 檔案共用的儲存體帳戶。
- **Azure 檔案共用**：您要同步處理的 Azure 檔案共用名稱。

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
若要建立同步群組，請執行下列 PowerShell。 請記得以所需的同步群組名稱取代 `<my-sync-group>`。

```powershell
$syncGroupName = "<my-sync-group>"
$syncGroup = New-AzStorageSyncGroup -ParentObject $storageSync -Name $syncGroupName
```

一旦成功建立同步群組之後，您就可以建立雲端端點。 請務必將 `<my-storage-account>` 和 `<my-file-share>` 取代為預期的值。

```powershell
# Get or create a storage account with desired name
$storageAccountName = "<my-storage-account>"
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroup | Where-Object {
    $_.StorageAccountName -eq $storageAccountName
}

if ($storageAccount -eq $null) {
    $storageAccount = New-AzStorageAccount `
        -Name $storageAccountName `
        -ResourceGroupName $resourceGroup `
        -Location $region `
        -SkuName Standard_LRS `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly:$true
}

# Get or create an Azure file share within the desired storage account
$fileShareName = "<my-file-share>"
$fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object {
    $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    $fileShare = New-AzStorageShare -Context $storageAccount.Context -Name $fileShareName
}

# Create the cloud endpoint
New-AzStorageSyncCloudEndpoint `
    -Name $fileShare.Name `
    -ParentObject $syncGroup `
    -StorageAccountResourceId $storageAccount.Id `
    -AzureFileShareName $fileShare.Name
```

---

## <a name="create-a-server-endpoint"></a>建立伺服器端點
伺服器端點代表已註冊伺服器上的特定位置，例如伺服器磁碟區上的資料夾。 伺服器端點必須是已註冊伺服器 (不是裝載共用) 上的路徑，而且若要使用雲端階層，此路徑必須位在非系統磁碟區上。 不支援網路連接儲存裝置 (NAS)。

# <a name="portaltabazure-portal"></a>[入口網站](#tab/azure-portal)
若要新增伺服器端點，請移至新建立的同步群組，然後選取 [新增伺服器端點]。

![在 [同步群組] 窗格中新增新的伺服器端點](media/storage-sync-files-deployment-guide/create-sync-group-2.png)

在 [新增伺服器端點] 窗格中，輸入下列資訊以建立伺服器端點：

- **已註冊的伺服器**：您要建立伺服器端點的伺服器或叢集名稱。
- **路徑**：要隨著同步群組同步處理的 Windows Server 路徑。
- **雲端階層處理**：啟用或停用雲端階層處理的開關。 透過雲端階層處理，可將不常使用或存取的檔案分層處理至 Azure 檔案服務。
- **磁碟區可用空間**：在伺服器端點所在磁碟區上要保留的可用空間數量。 例如，如果具有單一伺服器端點之磁碟區上的磁碟區可用空間是設定為 50%，則大約有一半的資料量會分層處理至 Azure 檔案服務。 無論雲端階層處理是否啟用，您的 Azure 檔案共用在同步群組中一律會有完整的資料複本。

若要新增伺服器端點，請選取 [建立]。 您的檔案現在會在 Azure 檔案共用和 Windows Server 之間保持同步。 

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)
執行下列 PowerShell 命令以建立伺服器端點，並務必以所需的值取代 `<your-server-endpoint-path>` 和 `<your-volume-free-space>`。

```powershell
$serverEndpointPath = "<your-server-endpoint-path>"
$cloudTieringDesired = $true
$volumeFreeSpacePercentage = <your-volume-free-space>

if ($cloudTieringDesired) {
    # Ensure endpoint path is not the system volume
    $directoryRoot = [System.IO.Directory]::GetDirectoryRoot($serverEndpointPath)
    $osVolume = "$($env:SystemDrive)\"
    if ($directoryRoot -eq $osVolume) {
        throw [System.Exception]::new("Cloud tiering cannot be enabled on the system volume")
    }

    # Create server endpoint
    New-AzStorageSyncServerEndpoint `
        -Name $registeredServer.FriendlyName `
        -SyncGroup $syncGroup `
        -ServerResourceId $registeredServer.ResourceId `
        -ServerLocalPath $serverEndpointPath `
        -CloudTiering `
        -VolumeFreeSpacePercent $volumeFreeSpacePercentage
} else {
    # Create server endpoint
    New-AzStorageSyncServerEndpoint `
        -Name $registeredServer.FriendlyName `
        -SyncGroup $syncGroup `
        -ServerResourceId $registeredServer.ResourceId `
        -ServerLocalPath $serverEndpointPath 
}
```

---

## <a name="configure-firewall-and-virtual-network-settings"></a>設定防火牆和虛擬網路設定

### <a name="portal"></a>入口網站
如果您想要將 Azure 檔案同步設定為使用防火牆和虛擬網路設定，請執行下列動作：

1. 從 [Azure 入口網站] 中，流覽至您要保護的儲存體帳戶。
1. 選取左側功能表上的 [**防火牆和虛擬網路**] 按鈕。
1. 選取 [**允許存取來源**] 底下的 [**選取的網路**]。
1. 請確定您的伺服器 IP 或虛擬網路已列在適當的區段底下。
1. 請確定已核取 [**允許信任的 Microsoft 服務存取此儲存體帳戶**]。
1. 選取 [儲存] 以儲存您的設定。

![將防火牆和虛擬網路設定為使用 Azure 檔案同步](media/storage-sync-files-deployment-guide/firewall-and-vnet.png)

## <a name="onboarding-with-azure-file-sync"></a>透過 Azure 檔案同步上架
第一次於 Azure 檔案同步上架時，如果不想要停機，同時還要保留完整檔案忠實性和存取控制清單 (ACL)，其建議步驟如下所示：
 
1. 部署儲存體同步服務。
2. 建立同步群組。
3. 在具有完整資料集的伺服器上安裝 Azure 檔案同步代理程式。
4. 註冊該伺服器，並在共用上建立伺服器端點。 
5. 讓同步服務將所有資料完整上傳至 Azure 檔案共用 (雲端端點)。  
6. 完成初始上傳之後，在其餘的各個伺服器上安裝 Azure 檔案同步代理程式。
7. 在其餘的各個伺服器上建立新的檔案共用。
8. 如有需要，請使用雲端階層處理原則，在新的檔案共用上建立伺服器端點。 (此步驟需有額外的儲存體以進行初始設定。)
9. 讓 Azure 檔案同步 agent 快速還原完整的命名空間，而不需要實際的資料傳輸。 在完整命名空間同步之後，同步引擎將會根據伺服器端點的雲端階層處理原則，填滿本機磁碟空間。 
10. 確定同步完成，並視需要測試拓撲。 
11. 將使用者和應用程式重新導向到這個新共用。
12. 您可以選擇性地刪除伺服器上任何重複的共用。
 
如果您沒有額外儲存體可供初始上架，而想要連結至現有的共用，您可以在 Azure 檔案共用中預先植入資料。 唯有您可接受停機時間，且絕對保證在初始上架程序期間，伺服器共用上的資料不會變更，才建議使用此方法。 
 
1. 確定在上架程式期間，任何伺服器上的資料都無法變更。
2. 透過 SMB 使用任何資料傳輸工具來預先植入具有伺服器資料的 Azure 檔案共用，例如 Robocopy、direct SMB 複製。 因為 AzCopy 不會透過 SMB 上傳資料，所以無法用於預先植入。
3. 使用指向現有共用的所需伺服器端點，建立 Azure 檔案同步拓撲。
4. 讓同步服務在所有端點上完成調整程序。 
5. 完成調整後，您可以開啟共用進行變更。
 
目前，預先植入的方法有幾個限制： 
- 檔案不會保持完整不失真。 例如，檔案會失去 ACL 和時間戳記。
- 在同步拓撲完全啟動並執行之前，伺服器上的資料變更會導致伺服器端點發生衝突。  
- 建立雲端端點之後，Azure 檔案同步在啟動初始同步處理之前，先執行一個程式來偵測雲端中的檔案。完成此程式所需的時間會視各種因素而異，例如網路速度、可用的頻寬，以及檔案和資料夾的數目。 根據預覽版本中的粗略估計，偵測程序會以大約每秒 10 個檔案的速度執行。因此，即使預先植入的執行速度快，但若在雲端預先植入資料，則系統完整執行的整體時間可能會更長。

## <a name="self-service-restore-through-previous-versions-and-vss-volume-shadow-copy-service"></a>透過舊版和 VSS 進行自助式還原（磁碟區陰影複製服務）

> [!IMPORTANT]
> 下列資訊只能與儲存體同步代理程式的第9版（或更新版本）搭配使用。 低於9的版本將不會有 StorageSyncSelfService Cmdlet。

先前的版本是一項 Windows 功能，可讓您利用磁片區的伺服器端 VSS 快照集，將檔案的可還原版本呈現給 SMB 用戶端。
這可為資訊工作者直接提供強大的案例，通常稱為自助式還原，而不是根據 IT 系統管理員的還原。

VSS 快照和舊版可獨立于 Azure 檔案同步工作。不過，雲端階層處理必須設定為相容模式。 許多 Azure 檔案同步伺服器端點可以存在於相同的磁片區上。 您必須針對每個磁片區進行下列 PowerShell 呼叫，而您計畫或使用雲端階層處理的伺服器端點也是一個。

```powershell
Import-Module ‘<SyncAgentInstallPath>\StorageSync.Management.ServerCmdlets.dll’
Enable-StorageSyncSelfServiceRestore [-DriveLetter] <string> [[-Force]] 
```

VSS 快照集是由整個磁片區所組成。 根據預設，指定的磁片區可能會有最多64個快照集，但已授與足夠的空間來儲存快照集。 VSS 會自動處理這種情況。 預設快照集排程每天從星期一到星期五取得兩個快照集。 該排程可透過 Windows 排程工作來設定。 上述 PowerShell Cmdlet 會執行兩項動作：
1. 它會在指定的磁片區上設定 Azure 檔案同步雲端階層處理，使其與先前的版本相容，並保證可以從舊版還原檔案，即使該檔案已分層到伺服器上的雲端也一樣。 
2. 它會啟用預設的 VSS 排程。 接著，您可以決定稍後加以修改。 

> [!Note]  
> 有兩個重點值得注意：
>- 如果您使用-Force 參數，而且目前已啟用 VSS，則它會覆寫目前的 VSS 快照集排程，並將它取代為預設排程。 執行 Cmdlet 之前，請務必先儲存自訂設定。
> - 如果您在叢集節點上使用此 Cmdlet，您也必須在叢集中的所有其他節點上執行它！ 

為了查看是否已啟用自助式還原相容性，您可以執行下列 Cmdlet。

```powershell
    Get-StorageSyncSelfServiceRestore [[-Driveletter] <string>]
```

它會列出伺服器上的所有磁片區，以及每個磁片區的雲端階層處理相容日數。 這個數位會根據每個磁片區的可能快照集和預設快照集排程自動計算。 因此，根據預設，呈現給資訊工作者的所有舊版都可以用來從還原。 如果您變更預設排程來取得更多快照集，也是如此。
不過，如果您將排程變更為會導致磁片區上的可用快照集超過相容的天數值，則使用者將無法使用這種較舊的快照集（舊版）來還原。

> [!Note]
> 啟用自助式還原可能會影響您的 Azure 儲存體耗用量和帳單。 這種影響僅限於伺服器上目前階層式檔案。 啟用此功能可確保雲端中有可用的檔案版本，可透過舊版（VSS 快照集）專案來參考。
>
> 如果您停用此功能，Azure 儲存體耗用量將會緩慢地拒絕，直到 [相容的天數] 視窗通過為止。 沒有辦法加快此速度。 

預設的 [每個磁片區的 VSS 快照集數目上限（64）] 和 [預設排程] 會產生最多45天的舊版資訊工作者可以從中還原的版本，視您可以在磁片區上儲存多少個 VSS 快照集而定。

如果上限。64每個磁片區的 VSS 快照集不是正確的設定，您可以透過登錄機[碼來變更](https://docs.microsoft.com/windows/win32/backup/registry-keys-for-backup-and-restore#maxshadowcopies)該值。
若要讓新的限制生效，您必須重新執行 Cmdlet，以便在先前啟用的每個磁片區上啟用先前的版本相容性，並使用-Force 旗標，將每個磁片區的最大 VSS 快照集數目納入考慮。 這會產生新計算的相容日數。 請注意，這項變更只會在新階層式檔案上生效，並會覆寫您所做的 VSS 排程上的任何自訂。

## <a name="migrate-a-dfs-replication-dfs-r-deployment-to-azure-file-sync"></a>將 DFS 複寫 (DFS-R) 部署移轉至 Azure 檔案同步
若要將 DFS-R 部署移轉至 Azure 檔案同步處理：

1. 建立同步群組來代表要取代的 DFS-R 拓樸。
2. 在要移轉 DFS R 拓撲中，於有一組完整資料的伺服器上啟動。 在該伺服器上安裝 Azure 檔案同步。
3. 註冊該伺服器，並為要移轉的第一個伺服器建立伺服器端點。 請勿啟用雲端階層。
4. 將所有資料同步至 Azure 檔案共用 (雲端端點)。
5. 在其餘的各個 DFS-R 伺服器上安裝並註冊 Azure 檔案同步代理程式。
6. 停用 DFS-R。 
7. 在每一個 DFS-R 伺服器上建立伺服器端點。 請勿啟用雲端階層。
8. 確定同步完成，並視需要測試拓撲。
9. 淘汰 DFS-R。
10. 現在可在所需的任何伺服器端點上啟用雲端階層。

如需詳細資訊，請參閱[Azure 檔案同步與分散式檔案系統 (DFS) 互通](storage-sync-files-planning.md#distributed-file-system-dfs)。

## <a name="next-steps"></a>後續步驟
- [新增或移除 Azure 檔案同步伺服器端點](storage-sync-files-server-endpoint.md)
- [向 Azure 檔案同步註冊或取消註冊伺服器](storage-sync-files-server-registration.md)
- [監視 Azure 檔案同步](storage-sync-files-monitoring.md)
