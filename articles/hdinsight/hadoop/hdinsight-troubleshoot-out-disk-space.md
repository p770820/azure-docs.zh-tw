---
title: 叢集節點在 Azure HDInsight 中的磁碟空間用盡
description: 針對 Azure HDInsight 中的 Apache Hadoop 叢集節點磁碟空間問題進行疑難排解。
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/05/2019
ms.openlocfilehash: fbfd82473b68f5032d19834ac809191d498a5a67
ms.sourcegitcommit: 8e9a6972196c5a752e9a0d021b715ca3b20a928f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2020
ms.locfileid: "75894119"
---
# <a name="scenario-cluster-node-runs-out-of-disk-space-in-azure-hdinsight"></a>案例：叢集節點在 Azure HDInsight 中的磁碟空間用盡

本文說明與 Azure HDInsight 叢集互動時，問題的疑難排解步驟和可能的解決方法。

## <a name="issue"></a>問題

作業可能會失敗，並出現類似以下的錯誤訊息： `/usr/hdp/2.6.3.2-14/hadoop/libexec/hadoop-config.sh: fork: No space left on device.`

或者，您可能會收到類似以下的 Apache Ambari 警示： `local-dirs usable space is below configured utilization percentage`。

## <a name="cause"></a>原因

Apache Yarn 應用程式快取可能已耗用所有可用的磁碟空間。 您的 Spark 應用程式可能執行效率不高。

## <a name="resolution"></a>解析度

1. 使用 Ambari UI 來判斷哪個節點的磁碟空間即將用盡。

1. 判斷麻煩節點中的哪個資料夾對大部分的磁碟空間有貢獻。 先以 SSH 連線到節點，然後執行 `df` 以列出所有掛接的磁片使用量。 通常它是 `/mnt`，這是 OS 所使用的暫存磁片。 您可以輸入至資料夾，然後輸入 `sudo du -hs` 以顯示資料夾下的摘要檔案大小。 如果您看到類似于 `/mnt/resource/hadoop/yarn/local/usercache/livy/appcache/application_1537280705629_0007`的資料夾，這表示應用程式仍在執行中。 這可能是因為 RDD 持續性或中繼隨機檔案所造成。

1. 若要減輕此問題，請終止應用程式，這將會釋放該應用程式所使用的磁碟空間。

1. 若要最終解決此問題，請將您的應用程式優化。

## <a name="next-steps"></a>後續步驟

如果您沒有看到您的問題，或無法解決您的問題，請瀏覽下列其中一個管道以取得更多支援：

* 透過[Azure 社區支援](https://azure.microsoft.com/support/community/)取得 azure 專家的解答。

* 連接[@AzureSupport](https://twitter.com/azuresupport) -官方 Microsoft Azure 帳戶，藉由將 Azure 社區連接至適當的資源，來改善客戶體驗：解答、支援和專家。

* 如果您需要更多協助，您可以從[Azure 入口網站](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/)提交支援要求。 從功能表列選取 [**支援**]，或開啟 [說明 **+ 支援**] 中樞。 如需詳細資訊，請參閱[如何建立 Azure 支援要求](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request)。 您的 Microsoft Azure 訂用帳戶包含訂用帳戶管理和帳單支援的存取權，而技術支援則透過其中一項[Azure 支援方案](https://azure.microsoft.com/support/plans/)提供。
