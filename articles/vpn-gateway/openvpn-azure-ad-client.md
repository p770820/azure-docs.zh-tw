---
title: VPN 閘道： OpenVPN 通訊協定 P2S 連線的 VPN 用戶端： Azure AD 驗證
description: 您可以使用 P2S VPN，使用 Azure AD authentication 來連線到您的 VNet
services: vpn-gateway
author: anzaman
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 02/10/2020
ms.author: alzam
ms.openlocfilehash: 4b9678f72dd69db24b105d4b1d708928e29a09ba
ms.sourcegitcommit: f718b98dfe37fc6599d3a2de3d70c168e29d5156
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/11/2020
ms.locfileid: "77134505"
---
# <a name="configure-a-vpn-client-for-p2s-openvpn-protocol-connections-azure-ad-authentication"></a>設定 VPN 用戶端以進行 P2S OpenVPN 通訊協定連線： Azure AD 驗證

本文可協助您設定 VPN 用戶端，以使用點對站 VPN 和 Azure Active Directory 驗證來連線到虛擬網路。 您必須先設定 Azure AD 租使用者，才可以使用 Azure AD 進行連線和驗證。 如需詳細資訊，請參閱[設定 Azure AD 租](openvpn-azure-ad-tenant.md)使用者。

> [!NOTE]
> Azure AD 驗證僅支援 OpenVPN® 通訊協定連線。
>

## <a name="profile"></a>使用用戶端設定檔

若要連線，您必須下載 Azure VPN 用戶端，並在每一部要連線到 VNet 的電腦上設定 VPN 用戶端設定檔。 您可以在電腦上建立用戶端設定檔、將它匯出，然後將它匯入到其他電腦。

### <a name="to-download-the-azure-vpn-client"></a>下載 Azure VPN Client

使用此[連結](https://go.microsoft.com/fwlink/?linkid=2117554)來下載 Azure VPN 用戶端。

### <a name="cert"></a>建立以憑證為基礎的用戶端設定檔

使用以憑證為基礎的設定檔時，請確定已在用戶端電腦上安裝適當的憑證。 如需憑證的詳細資訊，請參閱[安裝用戶端憑證](point-to-site-how-to-vpn-client-install-azure-cert.md)。

  ![cert](./media/openvpn-azure-ad-client/create/create-cert1.jpg)

### <a name="radius"></a>建立 RADIUS 用戶端設定檔

  ![半徑](./media/openvpn-azure-ad-client/create/create-radius1.jpg)
  
> [!NOTE]
> 您可以在 P2S VPN 用戶端設定檔中匯出伺服器密碼。  如需如何匯出用戶端設定檔的指示，請參閱[這裡](about-vpn-profile-download.md)。
>

### <a name="export"></a>匯出和散發用戶端設定檔

一旦您有工作設定檔，而且需要將它散發給其他使用者，您可以使用下列步驟將它匯出：

1. 反白顯示您要匯出的 VPN 用戶端設定檔，選取 [ **...** ]，然後選取 [**匯出**]。

    ![匯出](./media/openvpn-azure-ad-client/export/export1.jpg)

2. 選取您要儲存此設定檔的位置，並將檔案名保持原狀，然後選取 [**儲存**] 以儲存 xml 檔案。

    ![匯出](./media/openvpn-azure-ad-client/export/export2.jpg)

### <a name="import"></a>匯入用戶端設定檔

1. 在頁面上，選取 [匯入]。

    ![入口](./media/openvpn-azure-ad-client/import/import1.jpg)

2. 瀏覽至設定檔 xml 檔案，並加以選取。 選取檔案後，請選取 [開啟]。

    ![入口](./media/openvpn-azure-ad-client/import/import2.jpg)

3. 指定設定檔的名稱，然後選取 [儲存]。

    ![入口](./media/openvpn-azure-ad-client/import/import3.jpg)

4. 選取 [連線] 以連線至 VPN。

    ![入口](./media/openvpn-azure-ad-client/import/import4.jpg)

5. 連線之後，圖示將會變成綠色，並顯示為 [已連線]。

    ![入口](./media/openvpn-azure-ad-client/import/import5.jpg)

### <a name="delete"></a>刪除用戶端設定檔

1. 選取您想要刪除的用戶端設定檔旁邊的省略號。 然後，選取 [移除]。

    ![delete](./media/openvpn-azure-ad-client/delete/delete1.jpg)

2. 選取 [移除] 以刪除。

    ![delete](./media/openvpn-azure-ad-client/delete/delete2.jpg)

## <a name="connection"></a>建立連接

1. 在頁面上，依序選取 [ **+** ] 和 [ **+ 新增**]。

    ![連接](./media/openvpn-azure-ad-client/create/create1.jpg)

2. 填寫連接資訊。 如果您不確定這些值，請洽詢您的系統管理員。 填寫值之後，選取 [**儲存**]。

    ![連接](./media/openvpn-azure-ad-client/create/create2.jpg)

3. 選取 [連線] 以連線至 VPN。

    ![連接](./media/openvpn-azure-ad-client/create/create3.jpg)

4. 選取適當的認證，然後選取 [**繼續**]。

    ![連接](./media/openvpn-azure-ad-client/create/create4.jpg)

5. 成功連線後，圖示將會變成綠色，並顯示 [**已連線**]。

    ![連接](./media/openvpn-azure-ad-client/create/create5.jpg)

### <a name="autoconnect"></a>自動連接

這些步驟可協助您將連線設定為使用 Always on 自動連接。

1. 在 VPN 用戶端的首頁上，選取 [ **VPN 設定**]。

    ![自動](./media/openvpn-azure-ad-client/auto/auto1.jpg)

2. 在 [切換應用程式] 對話方塊上選取 **[是]** 。

    ![自動](./media/openvpn-azure-ad-client/auto/auto2.jpg)

3. 請確定您要設定的連線尚未連接，然後將設定檔反白顯示，並勾選 [**自動連接]** 核取方塊。

    ![自動](./media/openvpn-azure-ad-client/auto/auto3.jpg)

4. 選取 **[連線]** 以起始 VPN 連線。

    ![自動](./media/openvpn-azure-ad-client/auto/auto4.jpg)

## <a name="diagnose"></a>診斷連線問題

1. 若要診斷連線問題，您可以使用**診斷**工具。 選取您要診斷之 VPN 連線旁的 [ **...** ]，以顯示功能表。 然後，選取 [診斷]。

    ![診斷](./media/openvpn-azure-ad-client/diagnose/diagnose1.jpg)

2. 在 [連線屬性] 頁面上，選取 [執行診斷]。

    ![診斷](./media/openvpn-azure-ad-client/diagnose/diagnose2.jpg)

3. 使用您的認證登入。

    ![診斷](./media/openvpn-azure-ad-client/diagnose/diagnose3.jpg)

4. 檢視診斷結果。

    ![診斷](./media/openvpn-azure-ad-client/diagnose/diagnose4.jpg)

## <a name="faq"></a>常見問題集

### <a name="how-do-i-add-dns-suffixes-to-the-vpn-client"></a>如何? 將 DNS 尾碼新增至 VPN 用戶端嗎？

您可以修改下載的設定檔 XML 檔案，並將 **\<dnssuffixes >\<dnssufix > \</dnssufix >\</dnssuffixes >** 標記

```
<azvpnprofile>
<clientconfig>

    <dnssuffixes>
          <dnssuffix>.mycorp.com</dnssuffix>
          <dnssuffix>.xyz.com</dnssuffix>
          <dnssuffix>.etc.net</dnssuffix>
    </dnssuffixes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-add-custom-routes-to-the-vpn-client"></a>如何? 將自訂路由新增至 VPN 用戶端？

您可以修改下載的設定檔 XML 檔案，並將 **\<route >\<includeroutes >\<目的地 >\<mask > \</route >\</includeroutes >\</destination >\</mask >** 標記

```
<azvpnprofile>
<clientconfig>

    <includeroutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </includeroutes>
    
</clientconfig>
</azvpnprofile>
```

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱[為使用 Azure AD authentication 的 P2S OPEN VPN 連線建立 Azure Active Directory 租](openvpn-azure-ad-tenant.md)使用者。
