---
title: 根據時間分割的檔案名，以累加方式複製新檔案
description: 建立 Azure 資料處理站，然後使用資料複製工具，只根據時間分割的檔案名，以累加方式載入新的檔案。
services: data-factory
documentationcenter: ''
author: dearandyxu
ms.author: yexu
ms.reviewer: ''
manager: ''
ms.service: data-factory
ms.workload: data-services
ms.devlang: na
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 1/24/2019
ms.openlocfilehash: 38e0028efd7fbda8aa5bee497836ce1161109e03
ms.sourcegitcommit: 3dc1a23a7570552f0d1cc2ffdfb915ea871e257c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2020
ms.locfileid: "75982617"
---
# <a name="incrementally-copy-new-files-based-on-time-partitioned-file-name-by-using-the-copy-data-tool"></a>使用資料複製工具根據時間分割的檔案名，以累加方式複製新檔案

在這個教學課程中，您會使用 Azure 入口網站來建立資料處理站。 然後，您可以使用資料複製工具來建立管線，以根據從 Azure Blob 儲存體將時間分割的檔案名，以累加方式將新檔案複製到 Azure Blob 儲存體。

> [!NOTE]
> 如果您不熟悉 Azure Data Factory，請參閱 [Azure Data Factory 簡介](introduction.md)。

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。
> * 使用複製資料工具建立管線。
> * 監視管線和活動執行。

## <a name="prerequisites"></a>必要條件

* **Azure 訂用帳戶**：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
* **Azure 儲存體帳戶**：使用 Blob 儲存體作為_來源_和_接收_資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-account-create.md)中的指示。

### <a name="create-two-containers-in-blob-storage"></a>在 Blob 儲存體中建立兩個容器

執行下列步驟，為教學課程準備您的 Blob 儲存體。

1. 建立名為**source**的容器。  在您的容器中建立資料夾路徑為**2019/02/26/14** 。 建立空的文字檔，並將其命名為**file1 .txt**。 將 file1 上傳至儲存體帳戶中的資料夾路徑**來源/2019/02/26/14** 。  您可以使用各種工具來執行這些工作，例如 [Azure 儲存體總管](https://storageexplorer.com/)。

    ![上傳檔案](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/upload-file.png)

    > [!NOTE]
    > 請以您的 UTC 時間調整資料夾名稱。  例如，如果目前的 UTC 時間在2019年2月26日下午2:03，您可以將**來源/{Year}/{Month}/{Day}/{Hour**}/的規則建立為**source/2019/02/26/14/** 的資料夾路徑。

2. 建立名為**destination**的容器。 您可以使用各種工具來執行這些工作，例如 [Azure 儲存體總管](https://storageexplorer.com/)。

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 在左側功能表上，選取 [建立資源] > [資料 + 分析] > [資料處理站]：

   ![在 [新增] 窗格中選取資料處理站](./media/doc-common-process/new-azure-data-factory-menu.png)

2. 在 [新增資料處理站] 頁面的 [名稱] 下，輸入 **ADFTutorialDataFactory**。

    資料處理站的名稱必須是「全域唯一」的名稱。 您可能會收到下列錯誤訊息：

   ![新增資料處理站錯誤訊息](./media/doc-common-process/name-not-available-error.png)

   如果您收到有關名稱值的錯誤訊息，請輸入不同的資料處理站名稱。 例如，使用**您的名稱** **ADFTutorialDataFactory**。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory 命名規則](naming-rules.md)。
3. 選取要在其中建立新資料處理站的 Azure **訂用帳戶**。
4. 針對 [資源群組]，採取下列其中一個步驟︰

    a. 選取 [使用現有的]，然後從下拉式清單選取現有的資源群組。

    b. 選取 [建立新的]，然後輸入資源群組的名稱。 
         
    若要了解資源群組，請參閱[使用資源群組管理您的 Azure 資源](../azure-resource-manager/management/overview.md)。

5. 在 [版本] 下，選取 [V2] 作為版本。
6. 在 [位置] 下，選取資料處理站的位置。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區 (例如 Azure 儲存體和 SQL Database) 和計算 (例如 Azure HDInsight) 可位於其他地區和區域。
7. 選取 [釘選到儀表板]。
8. 選取 [建立]。
9. 儀表板上的 [部署資料處理站] 圖格會顯示程序狀態。

    ![部署資料處理站圖格](media/tutorial-copy-data-tool/deploying-data-factory.png)
10. 建立完成後，隨即會顯示 **Data Factory** 首頁。

    ![Data Factory 首頁](./media/doc-common-process/data-factory-home-page.png)
11. 選取 [編寫與監視] 圖格，可在另一個索引標籤中啟動 Azure Data Factory 使用者介面 (UI)。

## <a name="use-the-copy-data-tool-to-create-a-pipeline"></a>使用複製資料工具建立管線

1. 在 [**現在就開始**吧] 頁面上，選取**資料複製**的標題來啟動資料複製工具。

   ![複製資料工具圖格](./media/doc-common-process/get-started-page.png)

2. 在 [**屬性**] 頁面上，採取下列步驟：

    a. 在 [工作**名稱**] 下，輸入**DeltaCopyFromBlobPipeline**。

    b. 在 [工作**步調] 或**[工作排程] 底下，選取 [依**排程定期執行**]。

    c. 在 [**觸發程式類型**] 底下，選取 [**輪轉視窗]** 。

    d. 在 **[週期**] 底下，輸入**1 小時**。

    e. 選取 [下一步]。

    Data Factory 使用者介面會使用指定的工作名稱建立管線。

    ![屬性頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/copy-data-tool-properties-page.png)
3. 在 [來源資料存放區] 頁面上，完成下列步驟：

    a. 按一下 [ **+ 建立新**連線] 以新增連接。

    ![來源資料存放區頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/source-data-store-page.png)

    b. 從資源庫中選取 [ **Azure Blob 儲存體**]，然後按一下 [**繼續**]。

    ![來源資料存放區頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/source-data-store-page-select-blob.png)

    c. 在 [**新增連結服務**] 頁面上，從 [**儲存體帳戶名稱**] 清單中選取您的儲存體帳戶，然後按一下 **[完成]** 。

    ![來源資料存放區頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/source-data-store-page-linkedservice.png)

    d. 選取新建立的連結服務，然後按 **[下一步]** 。

   ![來源資料存放區頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/source-data-store-page-select-linkedservice.png)
4. 在 [選擇輸入檔案或資料夾] 頁面上，執行以下步驟︰

    a. 流覽並選取**來源**容器，然後選取 **[選擇**]。

    ![選擇輸入檔案或資料夾](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/choose-input-file-folder.png)

    b. 在 [檔案**載入行為**] 底下，選取 [累加**式載入：時間分割的資料夾/檔案名**]。

    ![選擇輸入檔案或資料夾](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/choose-loading-behavior.png)

    c. 將動態資料夾路徑當做**來源/{year}/{month}/{day}/{hour}/** 寫入，並如下所示變更格式：

    ![選擇輸入檔案或資料夾](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/input-file-name.png)

    d. 檢查**二進位複製**，然後按 **[下一步]** 。

    ![選擇輸入檔案或資料夾](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/check-binary-copy.png)     
5. 在 [**目的地資料存放區**] 頁面上，選取**AzureBlobStorage**，這是與資料來源存放區相同的儲存體帳戶，然後按 **[下一步]** 。

    ![目的地資料存放區頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/destination-data-store-page-select-linkedservice.png)
6. 在 [**選擇輸出檔案或資料夾**] 頁面上，執行下列步驟：

    a. 流覽並選取 [**目的地**] 資料夾，然後按一下 **[選擇**]。

    ![選擇輸出檔案或資料夾](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/choose-output-file-folder.png)

    b. 將動態資料夾路徑當做**來源/{year}/{month}/{day}/{hour}/** 寫入，並如下所示變更格式：

    ![選擇輸出檔案或資料夾](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/input-file-name2.png)

    c. 按 [下一步]。

    ![選擇輸出檔案或資料夾](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/click-next-after-output-folder.png)
7. 在 [設定] 頁面上，選取 [下一步]。

    ![設定頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/settings-page.png)
8. 在 [摘要] 頁面上檢閱設定，然後選取 [下一步]。

    ![摘要頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/summary-page.png)

9. 在**部署頁面**上選取 [監視] 來監視管線 (工作)。
    ![部署 頁面](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/deployment-page.png)

10. 請注意，系統會自動選取左側的 [監視] 索引標籤。  您需要等候管線執行自動觸發（大約一小時後）。  執行時，[**動作**] 資料行會包含可查看活動執行詳細資料，以及重新執行管線的連結。 選取 [重新整理] 以重新整理清單，**然後選取 [** **動作**] 資料行中的 [ **View Activity 執行**] 連結。

    ![監視管線回合](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/monitor-pipeline-runs1.png)
11. 管線中只有一個活動 (複製活動)，所以您只會看到一個項目。 您可以看到原始程式檔（file1）已從**來源/2019/02/26/14/** 複製到**目的地/2019/02/26/14/** （具有相同的檔案名）。  

    ![監視管線回合](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/monitor-pipeline-runs2.png)

    您也可以使用 Azure 儲存體總管（ https://storageexplorer.com/) 掃描檔案來驗證相同的。

    ![監視管線回合](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/monitor-pipeline-runs3.png)
12. 建立另一個空白文字檔，新名稱為**file2 .txt**。 將 file2 檔案上傳至儲存體帳戶中的資料夾路徑**來源/2019/02/26/15** 。   您可以使用各種工具來執行這些工作，例如 [Azure 儲存體總管](https://storageexplorer.com/)。

    ![監視管線回合](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/monitor-pipeline-runs4.png)

    > [!NOTE]
    > 您可能會注意到必須建立新的資料夾路徑。 請以您的 UTC 時間調整資料夾名稱。  例如，如果目前的 UTC 時間為下午3:20，2019年2月26日，您可以將資料夾路徑建立為**來源/2019/02/26/15/** （依據 **{Year}/{Month}/{Day}/{Hour}/** 的規則）。

13. 若要回到 [**管線執行**] 視圖，請選取 [**所有管線執行**]，然後等待另一個小時再次觸發相同的管線。  

    ![監視管線回合](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/monitor-pipeline-runs5.png)

14. 選取第二個管線執行時的 [ **View Activity run** ]，並執行相同的步驟來查看詳細資料。  

    ![監視管線回合](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/monitor-pipeline-runs6.png)

    您可以看到原始程式檔（file2 .txt）已從**來源/2019/02/26/15/** 複製到**目的地/2019/02/26/15/** （具有相同的檔案名）。

    ![監視管線回合](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/monitor-pipeline-runs7.png)

    您也可以使用 Azure 儲存體總管（ https://storageexplorer.com/) 掃描**目的地**容器中的檔案來驗證相同的

    ![監視管線回合](./media/tutorial-incremental-copy-partitioned-file-name-copy-data-tool/monitor-pipeline-runs8.png)


## <a name="next-steps"></a>後續步驟
進入下列教學課程，以了解如何在 Azure 上使用 Spark 叢集來轉換資料：

> [!div class="nextstepaction"]
>[在雲端中使用 Spark 叢集轉換資料](tutorial-transform-data-spark-portal.md)
