---
title: 從 Azure 下載 Linux VHD
description: 使用 Azure CLI 和 Azure 入口網站來下載 Linux VHD。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 08/21/2019
ms.author: cynthn
ms.openlocfilehash: 257f3f723fc8a971b8253699f4beb002cf46ce52
ms.sourcegitcommit: 49cf9786d3134517727ff1e656c4d8531bbbd332
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/13/2019
ms.locfileid: "74036280"
---
# <a name="download-a-linux-vhd-from-azure"></a>從 Azure 下載 Linux VHD

本文說明如何使用 Azure CLI 和 Azure 入口網站，從 Azure 下載 Linux 虛擬硬碟 (VHD) 檔案。 

如果尚未安裝 [Azure CLI](https://docs.microsoft.com/cli/azure/install-az-cli2)，請先安裝。

## <a name="stop-the-vm"></a>停止 VM

如果 VHD 連接至執行中的 VM，便無法從 Azure 下載該 VHD。 您必須先停止 VM，才能下載 VHD。 如果您想要使用 VHD 作為[映像](tutorial-custom-images.md)，以新磁碟來建立其他 VM，則需要取消佈建並一般化包含在檔案中的作業系統，並停止 VM。 若要使用 VHD 作為現有 VM 或資料磁碟新執行個體的磁碟，您只需要停止並解除配置 VM。

若要使用 VHD 作為映像來建立其他 VM，請完成下列步驟：

1. 使用 SSH、帳戶名稱與 VM 的公用 IP 位址來連線至 VM，並取消佈建 VM。 您可以使用 [az network public-ip show](https://docs.microsoft.com/cli/azure/network/public-ip#az-network-public-ip-show) 來尋找公用 IP 位址。 +user 參數也會移除最後一個佈建的使用者帳戶。 如果要將帳戶認證備份至 VM 中，請省略此 +user 參數。 下列範例會移除最後一個佈建的使用者帳戶：

    ```bash
    ssh azureuser@<publicIpAddress>
    sudo waagent -deprovision+user -force
    exit 
    ```

2. 使用 [az login](https://docs.microsoft.com/cli/azure/reference-index) 來登入您的 Azure 帳戶。
3. 停止及解除配置 VM。

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

4. 一般化 VM。 

    ```azurecli
    az vm generalize --resource-group myResourceGroup --name myVM
    ``` 

若要使用 VHD 作為現有 VM 或資料磁碟新執行個體的磁碟，請完成下列步驟：

1.  登入 [Azure 入口網站](https://portal.azure.com/)。
2.  在左功能表上，選取 [虛擬機器]。
3.  從清單中選取 VM。
4.  在 VM 的頁面上，選取 [**停止**]。

    ![停止 VM](./media/download-vhd/export-stop.png)

## <a name="generate-sas-url"></a>產生 SAS URL

若要下載 VHD 檔案，您需要產生[共用存取簽章 (SAS)](../../storage/common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) URL。 產生 URL 時，會將到期時間指派給 URL。

1.  在 VM 頁面的功能表上，選取 [**磁片**]。
2.  選取 VM 的作業系統磁片，然後選取 [**磁片匯出**]。
3.  選取 [**產生 URL**]。

    ![產生 URL](./media/download-vhd/export-generate.png)

## <a name="download-vhd"></a>下載 VHD

1.  在產生的 URL 底下，選取 [**下載 VHD**檔案]。
**
    ![下載 VHD](./media/download-vhd/export-download.png)

2.  您可能需要在瀏覽器中選取 [**儲存**]，才能開始下載。 VHD 檔案的預設名稱是 *abcd*。

    ![在瀏覽器中選取 [儲存]](./media/download-vhd/export-save.png)

## <a name="next-steps"></a>後續步驟

- 了解如何[使用 Azure CLI 從自訂磁碟上傳並建立 Linux VM](upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 
- [使用 Azure CLI 管理 Azure 磁碟](tutorial-manage-disks.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

