---
title: 使用 Azure PowerShell 部署和設定 Azure 防火牆
description: 在本文中，您將瞭解如何使用 Azure PowerShell 來部署和設定 Azure 防火牆。
services: firewall
author: vhorne
ms.service: firewall
ms.date: 4/10/2019
ms.author: victorh
ms.topic: conceptual
ms.openlocfilehash: 7f48012ca1f97c2e28380d95da37863c4bc17f63
ms.sourcegitcommit: 35715a7df8e476286e3fee954818ae1278cef1fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/08/2019
ms.locfileid: "73831847"
---
# <a name="deploy-and-configure-azure-firewall-using-azure-powershell"></a>使用 Azure PowerShell 部署和設定 Azure 防火牆

控制輸出網路存取是整體網路安全性計畫的重要部分。 例如，您可以限制對網站的存取。 或者，限制可存取的輸出 IP 位址和連接埠。

要控制從 Azure 子網路的輸出網路存取，使用 Azure 防火牆是可行的方式之一。 透過 Azure 防火牆，您可以設定：

* 應用程式規則，用以定義可從子網路存取的完整網域名稱 (FQDN)。
* 網路規則，用以定義來源位址、通訊協定、目的地連接埠和目的地位址。

當您將網路流量路由傳送到防火牆作為子網路預設閘道時，網路流量必須遵守設定的防火牆規則。

在本文中，您會建立具有三個子網的簡化單一 VNet，以便進行部署。 對於生產環境部署，建議您使用[中樞和支點模型](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)，其中防火牆會位於自己的 VNet 中。 工作負載伺服器位於相同區域中的對等互連 VNet，其中包含一個或多個子網路。

* **AzureFirewallSubnet** - 防火牆位於此子網路中。
* **Workload-SN** - 工作負載伺服器位於此子網路。 此子網路的網路流量會通過防火牆。
* **Jump-SN** - 「跳躍」伺服器位於此子網路。 跳躍伺服器具有公用 IP 位址，您可以使用遠端桌面與其連線。 接著，您可以從這裡連線到 (使用其他的遠端桌面) 工作負載伺服器。

![教學課程網路基礎結構](media/tutorial-firewall-rules-portal/Tutorial_network.png)

在本文中，您將了解：

> [!div class="checklist"]
> * 設定測試網路環境
> * 部署防火牆
> * 建立預設路由
> * 設定允許存取 www.google.com 的應用程式規則
> * 設定允許存取外部 DNS 伺服器的網路規則
> * 測試防火牆

如果您想要的話，可以使用[Azure 入口網站](tutorial-firewall-deploy-portal.md)完成此程式。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>必要條件

此程式會要求您在本機執行 PowerShell。 您必須已安裝 Azure PowerShell 模組。 執行 `Get-Module -ListAvailable Az` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](https://docs.microsoft.com/powershell/azure/install-Az-ps)。 驗證 PowerShell 版本之後，請執行 `Connect-AzAccount` 以建立與 Azure 的連線。

## <a name="set-up-the-network"></a>設定網路

首先，請建立資源群組，以包含部署防火牆所需的資源。 接著建立 VNet、子網路，並測試伺服器。

### <a name="create-a-resource-group"></a>建立資源群組

資源群組包含部署的所有資源。

```azurepowershell
New-AzResourceGroup -Name Test-FW-RG -Location "East US"
```

### <a name="create-a-vnet"></a>建立 VNet

此虛擬網路有三個子網：

> [!NOTE]
> AzureFirewallSubnet 子網路的大小是 /26。 如需有關子網路大小的詳細資訊，請參閱 [Azure 防火牆的常見問題集](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size)。

```azurepowershell
$FWsub = New-AzVirtualNetworkSubnetConfig -Name AzureFirewallSubnet -AddressPrefix 10.0.1.0/26
$Worksub = New-AzVirtualNetworkSubnetConfig -Name Workload-SN -AddressPrefix 10.0.2.0/24
$Jumpsub = New-AzVirtualNetworkSubnetConfig -Name Jump-SN -AddressPrefix 10.0.3.0/24
```
現在，建立虛擬網路：

```azurepowershell
$testVnet = New-AzVirtualNetwork -Name Test-FW-VN -ResourceGroupName Test-FW-RG `
-Location "East US" -AddressPrefix 10.0.0.0/16 -Subnet $FWsub, $Worksub, $Jumpsub
```

### <a name="create-virtual-machines"></a>建立虛擬機器

現在建立跳躍和工作負載虛擬機器，並將它們放在適當的子網路中。
出現提示時，請輸入虛擬機器的使用者名稱和密碼。

建立 Srv 跳躍虛擬機器。

```azurepowershell
New-AzVm `
    -ResourceGroupName Test-FW-RG `
    -Name "Srv-Jump" `
    -Location "East US" `
    -VirtualNetworkName Test-FW-VN `
    -SubnetName Jump-SN `
    -OpenPorts 3389 `
    -Size "Standard_DS2"
```

建立沒有公用 IP 位址的工作負載虛擬機器。
出現提示時，請輸入虛擬機器的使用者名稱和密碼。

```azurepowershell
#Create the NIC
$NIC = New-AzNetworkInterface -Name Srv-work -ResourceGroupName Test-FW-RG `
 -Location "East US" -Subnetid $testVnet.Subnets[1].Id 

#Define the virtual machine
$VirtualMachine = New-AzVMConfig -VMName Srv-Work -VMSize "Standard_DS2"
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName Srv-Work -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2016-Datacenter' -Version latest

#Create the virtual machine
New-AzVM -ResourceGroupName Test-FW-RG -Location "East US" -VM $VirtualMachine -Verbose
```

## <a name="deploy-the-firewall"></a>部署防火牆

現在將防火牆部署到虛擬網路中。

```azurepowershell
# Get a Public IP for the firewall
$FWpip = New-AzPublicIpAddress -Name "fw-pip" -ResourceGroupName Test-FW-RG `
  -Location "East US" -AllocationMethod Static -Sku Standard
# Create the firewall
$Azfw = New-AzFirewall -Name Test-FW01 -ResourceGroupName Test-FW-RG -Location "East US" -VirtualNetworkName Test-FW-VN -PublicIpName fw-pip

#Save the firewall private IP address for future use

$AzfwPrivateIP = $Azfw.IpConfigurations.privateipaddress
$AzfwPrivateIP
```

請記下私人 IP 位址。 稍後當您建立預設路由時將使用到它。

## <a name="create-a-default-route"></a>建立預設路由

建立已停用 BGP 路由傳播的資料表

```azurepowershell
$routeTableDG = New-AzRouteTable `
  -Name Firewall-rt-table `
  -ResourceGroupName Test-FW-RG `
  -location "East US" `
  -DisableBgpRoutePropagation

#Create a route
 Add-AzRouteConfig `
  -Name "DG-Route" `
  -RouteTable $routeTableDG `
  -AddressPrefix 0.0.0.0/0 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $testVnet `
  -Name Workload-SN `
  -AddressPrefix 10.0.2.0/24 `
  -RouteTable $routeTableDG | Set-AzVirtualNetwork
```

## <a name="configure-an-application-rule"></a>設定應用程式規則

應用程式規則允許對 www.google.com 進行輸出存取。

```azurepowershell
$AppRule1 = New-AzFirewallApplicationRule -Name Allow-Google -SourceAddress 10.0.2.0/24 `
  -Protocol http, https -TargetFqdn www.google.com

$AppRuleCollection = New-AzFirewallApplicationRuleCollection -Name App-Coll01 `
  -Priority 200 -ActionType Allow -Rule $AppRule1

$Azfw.ApplicationRuleCollections = $AppRuleCollection

Set-AzFirewall -AzureFirewall $Azfw
```

Azure 防火牆包含內建的規則集合，適用於依預設允許的基礎結構 FQDN。 這些 FQDN 特定於平台，且無法用於其他用途。 如需詳細資訊，請參閱[基礎結構 FQDN](infrastructure-fqdns.md)。

## <a name="configure-a-network-rule"></a>設定網路規則

網路規則允許在埠53（DNS）對兩個 IP 位址進行輸出存取。

```azurepowershell
$NetRule1 = New-AzFirewallNetworkRule -Name "Allow-DNS" -Protocol UDP -SourceAddress 10.0.2.0/24 `
   -DestinationAddress 209.244.0.3,209.244.0.4 -DestinationPort 53

$NetRuleCollection = New-AzFirewallNetworkRuleCollection -Name RCNet01 -Priority 200 `
   -Rule $NetRule1 -ActionType "Allow"

$Azfw.NetworkRuleCollections = $NetRuleCollection

Set-AzFirewall -AzureFirewall $Azfw
```

### <a name="change-the-primary-and-secondary-dns-address-for-the-srv-work-network-interface"></a>變更 **Srv-Work** 網路介面的主要和次要 DNS 位址

基於此程式中的測試目的，請設定伺服器的主要和次要 DNS 位址。 這不是一般的 Azure 防火牆需求。

```azurepowershell
$NIC.DnsSettings.DnsServers.Add("209.244.0.3")
$NIC.DnsSettings.DnsServers.Add("209.244.0.4")
$NIC | Set-AzNetworkInterface
```

## <a name="test-the-firewall"></a>測試防火牆

現在請測試防火牆，以確認其運作符合預期。

1. 記下**Srv 工作**虛擬機器的私人 IP 位址：

   ```
   $NIC.IpConfigurations.PrivateIpAddress
   ```

1. 將遠端桌面連線到 **Srv-Jump** 虛擬機器，然後登入。 從該處開啟與**Srv 工作**私人 IP 位址的遠端桌面連線，然後登入。

3. 在 [ **SRV-工作**] 上，開啟 PowerShell 視窗並執行下列命令：

   ```
   nslookup www.google.com
   nslookup www.microsoft.com
   ```

   這兩個命令都應該會傳回答案，顯示您的 DNS 查詢正在通過防火牆。

1. 執行以下命令：

   ```
   Invoke-WebRequest -Uri https://www.google.com
   Invoke-WebRequest -Uri https://www.google.com

   Invoke-WebRequest -Uri https://www.microsoft.com
   Invoke-WebRequest -Uri https://www.microsoft.com
   ```

   `www.google.com` 要求應會成功，而且 `www.microsoft.com` 要求應該會失敗。 這會示範您的防火牆規則如預期般運作。

因此，現在您已確認防火牆規則正在運作：

* 您可以使用設定的外部 DNS 伺服器來解析 DNS 名稱。
* 您可以瀏覽至允許 FQDN 的防火牆規則，但不可瀏覽至任何其他的防火牆規則。

## <a name="clean-up-resources"></a>清除資源

您可以保留防火牆資源供下一個教學課程，如果不再需要，請刪除**測試-FW-RG**資源群組來刪除所有防火牆相關資源：

```azurepowershell
Remove-AzResourceGroup -Name Test-FW-RG
```

## <a name="next-steps"></a>後續步驟

* [教學課程：監視 Azure 防火牆記錄](./tutorial-diagnostics.md)
