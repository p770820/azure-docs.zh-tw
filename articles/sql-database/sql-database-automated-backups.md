---
title: 自動、異地多餘備份
description: SQL Database 每隔幾鐘會自動建立一個本機資料庫備份，並使用 Azure 讀取權限異地備援儲存體來進行異地備援。
services: sql-database
ms.service: sql-database
ms.subservice: backup-restore
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, carlrab, danil
manager: craigg
ms.date: 12/13/2019
ms.openlocfilehash: f460bc3e4809b8a1cbabe1161c888255a7a484db
ms.sourcegitcommit: 76bc196464334a99510e33d836669d95d7f57643
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/12/2020
ms.locfileid: "77157494"
---
# <a name="automated-backups"></a>自動備份

SQL Database 會自動建立在設定的保留期間內保留的資料庫備份，並使用 Azure[讀取權限異地多餘儲存體（RA-GRS）](../storage/common/storage-redundancy.md)來確保即使資料中心無法使用，也會保留它們。 這些備份會自動建立。 資料庫備份可保護資料免於意外損毀或刪除，是商務持續性和災害復原策略中不可或缺的一部分。 如果您的安全性規則需要備份可供使用一段時間（最多10年），您可以在單一資料庫和彈性集區上設定[長期保留](sql-database-long-term-retention.md)。

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="what-is-a-sql-database-backup"></a>什麼是 SQL Database 備份

SQL Database 使用 SQL Server 的技術，每週建立[完整備份](https://docs.microsoft.com/sql/relational-databases/backup-restore/full-database-backups-sql-server)、每12小時進行[差異備份](https://docs.microsoft.com/sql/relational-databases/backup-restore/differential-backups-sql-server)，以及每5-10 分鐘建立一次[交易記錄備份](https://docs.microsoft.com/sql/relational-databases/backup-restore/transaction-log-backups-sql-server)。 這些備份會儲存在[GRS 儲存體 blob](../storage/common/storage-redundancy.md)中，並複寫到配對的[資料中心](../best-practices-availability-paired-regions.md)，以防止資料中心中斷。 在您還原資料庫時，服務會判斷需要還原的完整、差異及交易記錄備份。

您可以使用這些備份︰

- 使用 [Azure 入口網站]、[Azure PowerShell]、[Azure CLI] 或 [REST API]，將**現有的資料庫還原到**保留期限內的過去時間點。 在單一資料庫和彈性集區中，這項作業會在與原始資料庫相同的伺服器中建立新的資料庫。 在受控執行個體中，這項作業可以在相同的訂用帳戶下建立資料庫複本或相同或不同的受控執行個體。
- 將**已刪除的資料庫還原至其刪除**或保留期間內的任何時間。 已刪除的資料庫只能在原始資料庫建立所在的相同邏輯伺服器或受控執行個體中進行還原。
- **將資料庫還原到另一個地理區域**。 異地還原可讓您在無法存取您的伺服器和資料庫時，從地理災害中復原。 在世界各地任何現有的伺服器中建立新的資料庫。
- 如果資料庫已設定長期保留原則（LTR），請在單一資料庫或彈性集區上**從特定長期備份還原資料庫**。 LTR 可讓您使用[Azure 入口網站](sql-database-long-term-backup-retention-configure.md#using-azure-portal)或[Azure PowerShell](sql-database-long-term-backup-retention-configure.md#using-powershell)來還原舊版本的資料庫，以滿足合規性要求或執行舊版的應用程式。 如需詳細資訊，請參閱[長期保存](sql-database-long-term-retention.md)。
- 若要執行還原，請參閱[從備份還原資料庫](sql-database-recovery-using-backups.md)。

> [!NOTE]
> 在 Azure 儲存體中，「複寫」一詞指的是將檔案從某個位置複製到另一個位置。 SQL 的「資料庫複寫」指的是保持多個次要資料庫與主要資料庫同步。

您可以使用下列範例來嘗試其中一些作業：

| | Azure 入口網站 | Azure PowerShell |
|---|---|---|
| 變更備份保留期 | [單一資料庫](sql-database-automated-backups.md?tabs=managed-instance#change-pitr-backup-retention-period-using-azure-portal) <br/> [受控執行個體](sql-database-automated-backups.md?tabs=managed-instance#change-pitr-backup-retention-period-using-azure-portal) | [單一資料庫](sql-database-automated-backups.md#change-pitr-backup-retention-period-using-powershell) <br/>[受控執行個體](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlinstancedatabasebackupshorttermretentionpolicy) |
| 變更長期備份保留期 | [單一資料庫](sql-database-long-term-backup-retention-configure.md#configure-long-term-retention-policies)<br/>受控執行個體-N/A  | [單一資料庫](sql-database-long-term-backup-retention-configure.md)<br/>受控執行個體-N/A  |
| 從時間點還原資料庫 | [單一資料庫](sql-database-recovery-using-backups.md#point-in-time-restore) | [單一資料庫](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase) <br/> [受控執行個體](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqlinstancedatabase) |
| 還原已刪除的資料庫 | [單一資料庫](sql-database-recovery-using-backups.md) | [單一資料庫](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldeleteddatabasebackup) <br/> [受控執行個體](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldeletedinstancedatabasebackup)|
| 從 Azure Blob 儲存體還原資料庫 | 單一資料庫-N/A <br/>受控執行個體-N/A  | 單一資料庫-N/A <br/>[受控執行個體](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-get-started-restore) |


## <a name="backup-frequency"></a>備份頻率

### <a name="point-in-time-restore"></a>時間點還原

SQL Database 透過自動建立完整備份、差異備份和交易記錄備份，以支援自助式時間點還原 (PITR)。 根據計算大小和資料庫活動量的頻率，完整資料庫備份會每週建立，差異資料庫備份通常每隔 12 小時建立，而交易記錄備份通常每隔 5-10 分鐘建立。 建立資料庫之後，會立即排程第一次完整備份。 通常會在 30 分鐘內完成，但如果資料庫很大，則時間可能更久。 比方說，在還原的資料庫或資料庫複本上，初始備份可能需要較長的時間。 第一次完整備份之後，將會自動排程進一步的備份，並在背景中以無訊息方式管理。 資料庫備份的確切時間，依 SQL Database 服務整體系統工作負載維持平衡而決定。 您無法變更或停用備份作業。

PITR 備份會受到異地多餘儲存體的保護。 如需詳細資訊，請參閱[Azure 儲存體冗余](../storage/common/storage-redundancy.md)。

如需詳細資訊，請參閱[還原時間點](sql-database-recovery-using-backups.md#point-in-time-restore)

### <a name="long-term-retention"></a>長期保留

單一和集區資料庫提供選項讓您在 Azure Blob 儲存體中設定完整備份的長期保留 (LTR)，最長可達 10 年。 如果啟用 LTR 原則，則會將每週完整備份自動複製到不同的 RA-GRS 儲存體容器。 為了符合不同的合規性需求，您可以針對每週、每月和/或每年備份選取不同的保留期限。 儲存體耗用量取決於選取的備份頻率和保留期間。 您可以使用 [LTR 定價計算機](https://azure.microsoft.com/pricing/calculator/?service=sql-database)來估算 LTR 儲存體的成本。

就像 PITR，LTR 備份會以異地多餘儲存體來保護。 如需詳細資訊，請參閱[Azure 儲存體冗余](../storage/common/storage-redundancy.md)。

如需詳細資訊，請參閱[長期備份保留](sql-database-long-term-retention.md)。

## <a name="backup-storage-consumption"></a>備份儲存體耗用量 

針對單一資料庫，備份儲存體使用量總計的計算方式如下：   
`Total backup storage size = (size of full backups + size of differential backups + size of log backups) – database size`第 1 課：建立 Windows Azure 儲存體物件{2}。

針對彈性集區，備份儲存體大小總計會匯總在集區層級，計算方式如下：   
`Total backup storage size = (total size of all full backups + total size of all differential backups + total size of all log backups) - allocated pool data storage`第 1 課：建立 Windows Azure 儲存體物件{2}。 

比保留週期舊的備份會根據其時間戳記自動清除。 由於差異備份和記錄備份需要較舊的完整備份，因此它們會在每週的區塊中一起清除。 

Azure SQL Database 會將您的保留備份儲存體總計計算為累計值。 此值每小時會回報給 Azure 計費管線，負責匯總此每小時使用量，以計算每個月結束時的耗用量。 卸載資料庫之後，耗用量會隨著備份存留期而降低。 一旦備份會比保留期間還舊，就會停止計費。 


### <a name="monitoring-consumption"></a>監視耗用量

每種類型的備份（完整、差異和記錄）都會在 [資料庫監視] 分頁上報告為個別的計量。 下圖顯示如何監視備份儲存體耗用量。  

![在 Azure 入口網站的 [資料庫監視] 分頁上監視資料庫備份耗用量](media/sql-database-automated-backup/backup-metrics.png)

### <a name="fine-tune-the-backup-storage-consumption"></a>微調備份儲存體耗用量

多餘的備份儲存體耗用量將取決於個別資料庫的工作負載和大小。 您可以考慮執行下列一些微調技術，以進一步減少備份儲存體的耗用量：

* 將[備份保留期限](#change-pitr-backup-retention-period-using-azure-portal)縮減為您需求的最小值。
* 避免執行大型寫入作業的頻率高於所需，例如索引重建。
* 對於大型資料載入作業，請考慮使用叢集資料行存放區[索引](https://docs.microsoft.com/sql/database-engine/using-clustered-columnstore-indexes)、減少非叢集索引的數目，同時考慮資料列計數大約1000000的大量載入作業。
* 在一般用途服務層級中，已布建的資料儲存體比超額備份儲存體的價格便宜，因為有哪些客戶會持續大量的備份儲存體成本，可能會考慮增加資料儲存空間，以便儲存在備份儲存體。
* 使用 ETL 邏輯中的 TempDB 來儲存暫存結果，而不是永久資料表（僅適用于受控實例）。
* 請考慮針對不包含機密資料的資料庫（例如開發或測試資料庫）關閉 TDE 加密。 非加密資料庫的備份通常會以較高的壓縮比率進行壓縮。

> [!IMPORTANT]
> 針對分析、資料超市 \ 資料倉儲工作負載，強烈建議使用叢集資料行存放區[索引](https://docs.microsoft.com/sql/database-engine/using-clustered-columnstore-indexes)、減少非叢集索引的數目，同時考慮以1000000的資料列計數進行大量載入作業，以減少多餘的備份儲存體耗用量。


## <a name="storage-costs"></a>儲存成本


### <a name="dtu-model"></a>DTU 模型

使用 DTU 模型，資料庫和彈性集區的備份儲存體不需要額外付費。 

### <a name="vcore-model"></a>虛擬核心模型

針對單一資料庫，會免費提供等於100% 資料庫大小的備份儲存體數量下限。 對於彈性集區和受控實例而言，會分別以最小的備份儲存體數量（等於每個配置資料儲存區或實例大小的100%）提供給您，而不需要額外付費。 備份儲存體的額外使用量會按每月每 GB 來收費。 這項額外的耗用量將取決於個別資料庫的工作負載和大小。

Azure SQL DB 會以累計值計算您的保留備份儲存體總計。 此值每小時會回報給 Azure 計費管線，負責匯總此每小時使用量，以在每個月結束時取得您的耗用量。 在卸載資料庫之後，我們會將耗用量減少為備份存留期。 一旦超過保留期限，就會停止計費。 因為所有記錄備份和差異備份都會保留一段完整的保留期間，所以經常修改的資料庫會有較高的備份費用。 

假設資料庫已累積 744 GB 的備份儲存體，而此數量在整個月都保持不變。 若要將此累計儲存體耗用量轉換成每小時使用量，我們將它除以744.0 （每月31天 * 每天24小時）。 因此，SQL DB 會每小時報告資料庫耗用 1 GB 的 PITR 備份。 Azure 計費會匯總這項資訊，並根據您區域中的 $/GB/mo 費率，顯示整個月份的 744 GB 使用量和成本。 

現在是更複雜的範例。 假設資料庫的保留期已增加至該月的14天，而此（假設情況下）會導致備份儲存體總計加倍到 1488 GB。 SQL DB 會報告 1 GB 的使用量（小時）1-372，然後將使用量報告為 2 GB （小時）373-744。 這會匯總為每月 1116 GB 的最終帳單。 

您可以使用 Azure 訂用帳戶成本分析來判斷您目前的備份儲存體費用。

![備份儲存體成本分析](./media/sql-database-automated-backup/check-backup-storage-cost-sql-mi.png)

例如，若要瞭解受控實例的備份儲存體成本，請前往 Azure 入口網站中的訂用帳戶，然後開啟 [成本分析] 分頁。 選取計量子類別**mi [pitr 備份儲存體**]，以查看您目前的備份成本和費用預測。 您也可以包含其他計量子類別，例如**受控實例一般目的儲存體**或**受控實例一般目的-計算第5代**，以比較備份儲存體成本與其他成本類別。

## <a name="backup-retention"></a>備份保留期

所有 Azure SQL 資料庫（單一、集區和受控實例資料庫）的預設備份保留期限為**7**天。 您可以[將備份保留期限變更為最多35天](#change-pitr-backup-retention-period)。

如果您刪除資料庫，則 SQL Database 會以保存線上資料庫備份的相同方式保存備份。 例如，如果您刪除保留期間為七天的基本資料庫，則為期四天的備份還會再儲存三天。

如果您需要保留備份的時間超過保留週期上限，則可以修改備份的屬性，以將一或多個長期保留週期新增至資料庫。 如需詳細資訊，請參閱[長期保存](sql-database-long-term-retention.md)。

> [!IMPORTANT]
> 如果您刪除裝載 SQL 資料庫的 Azure SQL Server，則也會一併刪除所有屬於該伺服器的彈性集區和資料庫，且無法復原。 您無法還原已刪除的伺服器。 但是，如果您已設定長期保留，則不會刪除具有 LTR 之資料庫的備份，而且可以還原這些資料庫。

## <a name="encrypted-backups"></a>加密的備份

如果您的資料庫使用 TDE 加密，則備份會在靜止時自動加密 (包括 LTR 備份)。 Azure SQL 資料庫啟用 TDE 時，也會加密備份。 所有新的 Azure SQL 資料庫預設都會設定為啟用 TDE。 如需 TDE 的詳細資訊，請參閱 [Azure SQL Database 的透明資料加密](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)。

## <a name="backup-integrity"></a>備份完整性

Azure SQL Database 工程小組會持續自動測試在邏輯伺服器和彈性集區中的資料庫之自動資料庫備份的還原（這在受控執行個體中無法使用）。 在還原時間點時，資料庫也會使用 DBCC CHECKDB 來接收完整性檢查。

受控執行個體會在完成遷移之後，使用原生 `RESTORE` 命令或資料移轉服務還原的 `CHECKSUM` 資料庫，自動進行初始備份。

在完整性檢查期間找到的任何問題都會對工程小組發出警示。 如需有關 Azure SQL Database 中資料完整性的詳細資訊，請參閱 [Azure SQL Database 中的資料完整性](https://azure.microsoft.com/blog/data-integrity-in-azure-sql-database/)。

## <a name="compliance"></a>法規遵循

當您將資料庫從以 DTU 為基礎的服務層級遷移至以 vCore 為基礎的服務層級時，會保留 PITR 保留，以確保應用程式的資料恢復原則不會受到危害。 如果預設保留週期不符合合規性需求，您可以使用 PowerShell 或 REST API 變更 PITR 保留週期。 如需詳細資訊，請參閱[變更備份保留週期](#change-pitr-backup-retention-period)。

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="change-pitr-backup-retention-period"></a>變更 PITR 備份保留期限

您可以使用 Azure 入口網站、PowerShell 或 REST API 來變更預設的 PITR 備份保留期限。 下列範例說明如何將 PITR 保留變更為 28 天。

> [!WARNING]
> 如果您降低目前的保留期間，則無法再使用新保留期間之前的所有現有備份。 如果您延長目前的保留期間，則 SQL Database 將保留現有備份，直到達到較長的保留期間為止。

> [!NOTE]
> 這些 API 只會影響 PITR 保留期間。 如果您已將資料庫設定為 LTR，則它不受影響。 如需如何變更 LTR 保留期間的詳細資訊，請參閱[長期保留](sql-database-long-term-retention.md)。

### <a name="change-pitr-backup-retention-period-using-azure-portal"></a>使用 Azure 入口網站變更 PITR 備份保留期限

若要使用 Azure 入口網站來變更 PITR 備份保留期限，請流覽至您想要在入口網站中變更其保留期限的伺服器物件，然後根據您要修改的伺服器物件，選取適當的選項。

#### <a name="single-database--elastic-poolstabsingle-database"></a>[單一資料庫與彈性集區](#tab/single-database)

單一 Azure SQL 資料庫的 PITR 備份保留變更會在伺服器層級執行。 在伺服器層級進行的變更會套用至該伺服器上的資料庫。 若要從 Azure 入口網站變更 Azure SQL Database server 的 PITR，請流覽至 [伺服器總覽] 分頁，按一下導覽功能表上的 [管理備份]，然後按一下巡覽列上的 [設定保留]。

![變更 PITR Azure 入口網站](./media/sql-database-automated-backup/configure-backup-retention-sqldb.png)

#### <a name="managed-instancetabmanaged-instance"></a>[受控執行個體](#tab/managed-instance)

SQL Database 受控實例的 PITR 備份保留變更會在個別資料庫層級執行。 若要從 Azure 入口網站變更實例資料庫的 PITR 備份保留，請流覽至 [個別資料庫總覽] 分頁，然後按一下巡覽列上的 [設定備份保留]。

![變更 PITR Azure 入口網站](./media/sql-database-automated-backup/configure-backup-retention-sqlmi.png)

---

### <a name="change-pitr-backup-retention-period-using-powershell"></a>使用 PowerShell 變更 PITR 備份保留期間

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Azure SQL Database 仍然支援 PowerShell Azure Resource Manager 模組，但所有未來的開發都是針對 Az .Sql 模組。 如需這些 Cmdlet，請參閱[AzureRM](https://docs.microsoft.com/powershell/module/AzureRM.Sql/)。 Az 模組和 AzureRm 模組中命令的引數本質上完全相同。

```powershell
Set-AzSqlDatabaseBackupShortTermRetentionPolicy -ResourceGroupName resourceGroup -ServerName testserver -DatabaseName testDatabase -RetentionDays 28
```

### <a name="change-pitr-retention-period-using-rest-api"></a>使用 REST API 變更 PITR 保留期間

#### <a name="sample-request"></a>範例要求

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/resourceGroup/providers/Microsoft.Sql/servers/testserver/databases/testDatabase/backupShortTermRetentionPolicies/default?api-version=2017-10-01-preview
```

#### <a name="request-body"></a>要求本文

```json
{
  "properties":{
    "retentionDays":28
  }
}
```

#### <a name="sample-response"></a>範例回應

狀態碼：200

```json
{
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/providers/Microsoft.Sql/resourceGroups/resourceGroup/servers/testserver/databases/testDatabase/backupShortTermRetentionPolicies/default",
  "name": "default",
  "type": "Microsoft.Sql/resourceGroups/servers/databases/backupShortTermRetentionPolicies",
  "properties": {
    "retentionDays": 28
  }
}
```

如需詳細資訊，請參閱[備份保留 REST API](https://docs.microsoft.com/rest/api/sql/backupshorttermretentionpolicies)。

## <a name="next-steps"></a>後續步驟

- 資料庫備份可保護資料免於意外損毀或刪除，是商務持續性和災害復原策略中不可或缺的一部分。 若要深入了解其他 Azure SQL Database 業務持續性解決方案，請參閱[業務持續性概觀](sql-database-business-continuity.md)。
- 若要使用 Azure 入口網站還原至某個時間點，請參閱[使用 Azure 入口網站將資料庫還原至時間點](sql-database-recovery-using-backups.md)。
- 若要使用 PowerShell 還原至某個時間點，請參閱[使用 PowerShell 將資料庫還原至時間點](scripts/sql-database-restore-database-powershell.md)。
- 若要使用 Azure 入口網站在 Azure Blob 儲存體中設定、管理自動備份的長期保留及從該保留還原，請參閱[使用 Azure 入口網站來管理長期備份保留 (英文)](sql-database-long-term-backup-retention-configure.md)。
- 若要使用 PowerShell 在 Azure Blob 儲存體中設定、管理自動備份的長期保留，並從中還原，請參閱[使用 Powershell 管理長期備份保留](sql-database-long-term-backup-retention-configure.md)。
