---
title: 設定 Hyper-v 的 Azure Migrate 設備
description: 瞭解如何設定 Azure Migrate 應用裝置，以評估和遷移 Hyper-v Vm。
ms.topic: article
ms.date: 11/19/2019
ms.openlocfilehash: 0704adda314b94736b01fe114c3643ef8bd83753
ms.sourcegitcommit: dbcc4569fde1bebb9df0a3ab6d4d3ff7f806d486
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2020
ms.locfileid: "76029099"
---
# <a name="set-up-an-appliance-for-hyper-v-vms"></a>設定 Hyper-v Vm 的應用裝置

本文說明當您使用 Azure Migrate Server 評估工具來評估 Hyper-v Vm，或使用 Azure Migrate Server 遷移工具將 VMware Vm 遷移至 Azure 時，如何設定 Azure Migrate 設備。

Hyper-v VM 應用裝置是一種輕量設備，Azure Migrate Server 評估/遷移用來執行下列動作：

- 探索內部部署 Hyper-v Vm。
- 將探索到的 Vm 的中繼資料和效能資料傳送至 Azure Migrate 伺服器評量/遷移。

[深入瞭解](migrate-appliance.md)Azure Migrate 設備。


## <a name="appliance-deployment-steps"></a>設備部署步驟

若要設定設備，請：
- 從 Azure 入口網站下載壓縮的 Hyper-V VHD。
- 建立設備，並確認其可以連線至 Azure Migrate 伺服器評估。
- 第一次設定設備，並向 Azure Migrate 專案進行註冊。

## <a name="download-the-vhd"></a>下載 VHD

下載設備的已壓縮 VHD 範本。

1. 在[移轉目標] > [伺服器] > [Azure Migrate：伺服器評估] 中，按一下[探索]。
2. 在 [探索機器] > [機器是否已虛擬化?] 中，按一下 [是，使用 Hyper-V]。
3. 按一下 [下載] 以下載 VHD 檔案。

    ![下載 VM](./media/how-to-set-up-appliance-hyper-v/download-appliance-hyperv.png)


### <a name="verify-security"></a>確認安全性

請先確認 ZIP 檔案安全無虞再進行部署。

1. 在存放下載檔案的目標電腦上，開啟系統管理員命令視窗。
2. 執行下列命令以產生 VHD 的雜湊
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - 使用方式範例：```C:\>CertUtil -HashFile C:\AzureMigrate\AzureMigrate.vhd SHA256```
3.  針對 [設備版本 2.19.11.12]，產生的雜湊應符合這些設定。

  **演算法** | **雜湊值**
  --- | ---
  MD5 | 29a7531f32bcf69f32d964fa5ae950bc
  SHA256 | 37b3f27bc44f475872e355f04fcb8f38606c84534c117d1609f2d12444569b31



## <a name="create-the-appliance-vm"></a>建立設備 VM

匯入所下載的檔案，並建立 VM。

1. 在會裝載設備 VM 的 Hyper-V 主機上，將壓縮的 VHD 檔案解壓縮至其中的資料夾。 會解壓縮出三個資料夾。
2. 開啟 Hyper-V 管理員。 在 [動作] 中，按一下 [匯入虛擬機器]。

    ![部署 VHD](./media/how-to-set-up-appliance-hyper-v/deploy-vhd.png)

2. 在 [匯入虛擬機器精靈] > [開始之前]，按 [下一步]。
3. 在 [尋找資料夾] 中，指定已解壓縮的 VHD 所在的資料夾。 然後按一下 [下一步]。
1. 在 [選取虛擬機器] 中，按 [下一步]。
2. 在 [選擇匯入類型] 中，按一下 [複製虛擬機器 (建立新的唯一識別碼)]。 然後按一下 [下一步]。
3. 在 [選擇目的地] 中，保留預設設定。 按 [下一步]。
4. 在 [儲存資料夾] 中，保留預設設定。 按 [下一步]。
5. 在 [選擇網路] 中，指定 VM 會使用的虛擬交換器。 此交換器必須能夠連線到網際網路，以將資料傳送至 Azure。
6. 在 [摘要] 中檢閱設定。 然後按一下 [ **完成**]。
7. 在 [Hyper-V 管理員] > [虛擬機器] 中，啟動 VM。


### <a name="verify-appliance-access-to-azure"></a>確認設備是否能存取 Azure

請確定設備 VM 可以連線至 [Azure URL](migrate-appliance.md#url-access)。

## <a name="configure-the-appliance"></a>設定設備

第一次設定設備。

1. 在 [Hyper-V 管理員] > [虛擬機器] 中，以滑鼠右鍵按一下 [VM] > [連線]。
2. 提供設備的語言、時區和密碼。
3. 在任何可連線至 VM 的電腦上開啟瀏覽器，並開啟設備 web 應用程式的 URL： **HTTPs://*設備名稱或 IP 位址*： 44368**。

   或者，您也可以按一下應用程式捷徑，從設備桌面開啟應用程式。
1. 在 [Web 應用程式] > [設定必要條件] 中，執行下列動作：
    - **授權**：接受授權條款，並閱讀協力廠商資訊。
    - 連線**能力**：應用程式會檢查 VM 是否可存取網際網路。 如果 VM 使用 Proxy：
        - 按一下 [Proxy 設定]，然後以 http://ProxyIPAddress 或 http://ProxyFQDN 格式指定 Proxy 位址和接聽連接埠。
        - 如果 Proxy 需要驗證，請指定認證。
        - 僅支援 HTTP Proxy。
    - **時間同步**：已驗證時間。 設備上的時間應該與網際網路時間同步，VM 探索才能正常運作。
    - **安裝更新**： Azure Migrate Server 評估會檢查設備是否已安裝最新的更新。

### <a name="register-the-appliance-with-azure-migrate"></a>向 Azure Migrate 註冊設備

1. 按一下 [登入]。 如果未出現，請確定您已在瀏覽器中停用快顯封鎖程式。
2. 在新的索引標籤上，使用您的 Azure 認證登入。
    - 以您的使用者名稱和密碼登入。
    - 不支援使用 PIN 登入。
3. 成功登入後，返回 Web 應用程式。
4. 選取 Azure Migrate 專案建立所在的訂用帳戶。 然後選取專案。
5. 指定設備的名稱。 名稱應該是英數位元，且長度不超過 14 個字元。
6. 按一下 [註冊]。


### <a name="delegate-credentials-for-smb-vhds"></a>委派 SMB VHD 的認證

如果您要在 SMB 上執行 VHD，就必須將認證從設備委派到 Hyper-V 主機。 若要從設備執行此動作：

1. 在設備 VM 上，執行此命令。 HyperVHost1/HyperVHost2 是主機名稱範例。

    ```
    Enable-WSManCredSSP -Role Client -DelegateComputer HyperVHost1.contoso.com HyperVHost2.contoso.com -Force
    ```

2. 或者，也可以在設備上的本機群組原則編輯器中執行此動作：
    - 在 [本機電腦原則] > [電腦設定] 中，按一下 [系統管理範本] > [系統] > [認證委派]。
    - 按兩下 [允許委派全新認證]，然後選取 [已啟用]。
    - 在 [選項] 中，按一下 [顯示]，然後以 **wsman/** 作為前置詞將您想要探索的每一部 Hyper-V 主機新增至清單中。
    - 在 [認證委派] 中，按兩下 [允許委派僅具有 NTLM 伺服器驗證的全新認證]。 再次以 **wsman/** 作為前置詞將您想要探索的每一部 Hyper-V 主機新增至清單中。

## <a name="start-continuous-discovery"></a>開始連續探索

從設備連線至 Hyper-V 主機或叢集，然後開始探索 VM。

1. 在 [使用者名稱] 和 [密碼] 中，指定設備會用來探索 VM 的帳戶認證。 為認證指定自訂名稱，然後按一下 [儲存詳細資料]。
2. 按一下 [新增主機]，然後指定 Hyper-V 主機/叢集詳細資料。
3. 按一下 **[驗證]** 。 通過驗證後，系統就會顯示每個主機/叢集上可探索的 VM 數目。
    - 如果主機驗證失敗，請將滑鼠停留在 [狀態] 資料行中的圖示上，以檢閱錯誤。 修正問題，然後再次驗證。
    - 若要移除主機或叢集，請選取 [刪除]。
    - 您無法從叢集中移除特定主機。 您只能移除整個叢集。
    - 即使叢集中的特定主機有問題，您還是可以新增叢集。
4. 驗證之後，按一下 [儲存並開始探索] 來開始探索程序。

這會開始探索。 所探索到 VM 的中繼資料會在大約 15 分鐘後出現在 Azure 入口網站中。

## <a name="verify-vms-in-the-portal"></a>在入口網站中確認 VM

探索完成後，便可以確認 VM 是否出現在入口網站中。

1. 開啟 Azure Migrate 儀表板。
2. 在 [Azure Migrate - 伺服器] > [Azure Migrate：伺服器評估] 頁面中，按一下圖示以顯示 [探索到的伺服器] 計數。


## <a name="next-steps"></a>後續步驟

試用 Azure Migrate Server 評定的[hyper-v 評估](tutorial-assess-hyper-v.md)。
