---
title: 從 Salesforce Marketing Cloud 複製資料
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 Salesforce Marketing Cloud 複製到支援的接收資料存放區。
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/25/2019
ms.openlocfilehash: aa996ae14276ecf246104d8778fbb6b723448e04
ms.sourcegitcommit: a5ebf5026d9967c4c4f92432698cb1f8651c03bb
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/08/2019
ms.locfileid: "74931711"
---
# <a name="copy-data-from-salesforce-marketing-cloud-using-azure-data-factory"></a>使用 Azure Data Factory 從 Salesforce Marketing Cloud 複製資料

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Salesforce Marketing Cloud 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

下列活動支援此 Salesforce 行銷雲端連接器：

- [複製活動](copy-activity-overview.md)與[支援的來源/接收矩陣](copy-activity-overview.md)
- [查閱活動](control-flow-lookup-activity.md)

您可以將資料從 Salesforce Marketing Cloud 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

Salesforce Marketing 雲端連接器支援 OAuth 2 驗證。 它建置於[Salesforce Marketing Cloud REST API](https://developer.salesforce.com/docs/atlas.en-us.mc-apis.meta/mc-apis/index-api.htm)之上。

>[!NOTE]
>此連接器不支援擷取自訂物件或自訂資料延伸模組。

## <a name="getting-started"></a>使用者入門

您可以使用 .NET SDK、Python SDK、Azure PowerShell、REST API 或 Azure Resource Manager 範本來建立具有複製活動的管線。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](quickstart-create-data-factory-dot-net.md)。

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Salesforce Marketing Cloud 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Salesforce Marketing Cloud 已連結服務支援的屬性：

| 屬性 | 描述 | 必要項 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**SalesforceMarketingCloud** | 是 |
| clientId | 與 Salesforce Marketing Cloud 應用程式相關聯的用戶端識別碼。  | 是 |
| clientSecret | 與 Salesforce Marketing Cloud 應用程式相關聯的用戶端密碼。 您可以選擇將這個欄位標記為 SecureString 以將它安全地儲存在 ADF，或將密碼儲存在 Azure Key Vault，然後在執行複製資料時，讓 ADF 複製活動從該處提取 - 請參閱[將認證儲存在 Key Vault](store-credentials-in-key-vault.md) 以進一步了解。 | 是 |
| useEncryptedEndpoints | 指定是否使用 HTTPS 來加密資料來源端點。 預設值為 true。  | 否 |
| useHostVerification | 指定在透過 SSL 連線時，是否要求伺服器憑證中的主機名稱符合伺服器的主機名稱。 預設值為 true。  | 否 |
| usePeerVerification | 指定在透過 SSL 連線時，是否要確認伺服器的身分識別。 預設值為 true。  | 否 |

**範例：**

```json
{
    "name": "SalesforceMarketingCloudLinkedService",
    "properties": {
        "type": "SalesforceMarketingCloud",
        "typeProperties": {
            "clientId" : "<clientId>",
            "clientSecret": {
                 "type": "SecureString",
                 "value": "<clientSecret>"
            },
            "useEncryptedEndpoints" : true,
            "useHostVerification" : true,
            "usePeerVerification" : true
        }
    }
}

```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 Salesforce Marketing Cloud 資料集所支援的屬性清單。

若要從 Salesforce Marketing Cloud 複製資料，請將資料集的類型屬性設定為 **SalesforceMarketingCloudObject**。 以下是支援的屬性：

| 屬性 | 描述 | 必要項 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**SalesforceMarketingCloudObject** | 是 |
| tableName | 資料表的名稱。 | 否 (如果已指定活動來源中的"query") |

**範例**

```json
{
    "name": "SalesforceMarketingCloudDataset",
    "properties": {
        "type": "SalesforceMarketingCloudObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SalesforceMarketingCloud linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[Pipelines](concepts-pipelines-activities.md)一文。 本節提供 Salesforce Marketing Cloud 來源所支援的屬性清單。

### <a name="salesforce-marketing-cloud-as-source"></a>Salesforce Marketing Cloud 作為來源

若要從 Salesforce Marketing Cloud 複製資料，請將複製活動中的來源類型設定為 **SalesforceMarketingCloudSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 描述 | 必要項 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**SalesforceMarketingCloudSource** | 是 |
| query | 使用自訂 SQL 查詢來讀取資料。 例如： `"SELECT * FROM MyTable"` 。 | 否 (如果已指定資料集中的 "tableName") |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromSalesforceMarketingCloud",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SalesforceMarketingCloud input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SalesforceMarketingCloudSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>查閱活動屬性

若要瞭解屬性的詳細資料，請檢查[查閱活動](control-flow-lookup-activity.md)。

## <a name="next-steps"></a>接下來的步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
