---
title: Windows 7 虛擬機器 Windows 虛擬桌面-Azure
description: 如何解決 Windows 虛擬桌面環境中 Windows 7 虛擬機器（Vm）的問題。
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: troubleshooting
ms.date: 09/23/2019
ms.author: helohr
ms.openlocfilehash: ee641a694371ffd3fbc691a05edac707e37e959d
ms.sourcegitcommit: c62a68ed80289d0daada860b837c31625b0fa0f0
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/05/2019
ms.locfileid: "73607331"
---
# <a name="troubleshoot-windows-7-virtual-machines-in-windows-virtual-desktop"></a>Windows 虛擬桌面中的 Windows 7 虛擬機器進行疑難排解

使用本文來針對設定 Windows 虛擬桌面工作階段主機虛擬機器（Vm）時所發生的問題進行疑難排解。

## <a name="known-issues"></a>已知問題

Windows 虛擬桌面電腦上的 windows 7 不支援下列功能：

- 虛擬化應用程式（Remoteapp）
- 時區重新導向
- 自動 DPI 縮放比例

Windows 虛擬桌面只能虛擬化 Windows 7 的完整桌上型電腦。

雖然不支援自動 DPI 縮放比例，但您可以在遠端桌面用戶端的圖示上按一下滑鼠右鍵，然後選取 [**解決**]，以手動方式變更虛擬機器上的解析度。

## <a name="error-cant-access-the-remote-desktop-user-group"></a>錯誤：無法存取遠端桌面使用者群組

如果 Windows 虛擬桌面在遠端桌面使用者群組中找不到您或您的使用者認證，您可能會看到下列其中一個錯誤訊息：

- 「此使用者不是遠端桌面使用者群組的成員」
- 「您必須透過遠端桌面服務授與您登入的許可權」

若要修正此錯誤，請將使用者新增至遠端桌面使用者群組：

1. 開啟 Azure 入口網站。
2. 選取您看到的錯誤訊息所在的虛擬機器。
3. 選取 [**執行命令**]。
4. 以您要新增的使用者名稱取代 `<username>`，執行下列命令：
   
   ```cmd
   net localgroup "Remote Desktop Users" <username> /add
   ```