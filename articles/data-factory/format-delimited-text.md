---
title: Azure Data Factory 中的分隔文字格式
description: 本主題描述如何處理 Azure Data Factory 中的分隔文字格式。
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/09/2020
ms.author: jingwang
ms.openlocfilehash: f2e70a7b900ad918cda05ce34204e2de1e6e67ef
ms.sourcegitcommit: 8b37091efe8c575467e56ece4d3f805ea2707a64
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/09/2020
ms.locfileid: "75830185"
---
# <a name="delimited-text-format-in-azure-data-factory"></a>Azure Data Factory 中的分隔文字格式

當您想要**剖析分隔符號文字檔，或將資料寫入分隔的文字格式**時，請遵循這篇文章。 

下列連接器支援分隔的文字格式： [Amazon S3](connector-amazon-simple-storage-service.md)、 [azure Blob](connector-azure-blob-storage.md)、 [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md)、 [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)、Azure 檔案[儲存體](connector-azure-file-storage.md)、[檔案系統](connector-file-system.md)、 [FTP](connector-ftp.md)、 [Google Cloud Storage](connector-google-cloud-storage.md)、 [HDFS](connector-hdfs.md)、 [HTTP](connector-http.md)和[SFTP](connector-sftp.md)。

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供分隔的文字資料集所支援的屬性清單。

| 屬性         | 說明                                                  | 必要項 |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | 資料集的類型屬性必須設定為**DelimitedText**。 | 是      |
| location         | 檔案的位置設定。 每個以檔案為基礎的連接器在 `location`之下都有自己的位置類型和支援的屬性。  | 是      |
| columnDelimiter  | 用來分隔檔案中資料行的字元。 目前，只有對應資料流程才支援多字元分隔符號，而非複製活動。 <br>預設值為**逗號 `,`** ，當資料行分隔符號定義為空字串（表示沒有分隔符號）時，就會將整條程式碼當做單一資料行。 | 否       |
| rowDelimiter     | 單一字元或 "\r\n"，用來分隔檔案中的資料列。<br>預設值是讀取時的下列任何值 **： ["\r\n"，"\r"，"\n"]** ，而 " **\n" 或 "\r\n" 在寫入**時分別對應資料流程和複製活動。 <br>當 `rowDelimiter` 設定為 [沒有分隔符號（空字串）] 時，`columnDelimiter` 也必須設定為 [沒有分隔符號（空字串）]，這表示要將整個內容視為單一值。 | 否       |
| quoteChar        | 如果包含資料行分隔符號，則為要加上引號的單一字元。 <br>預設值為**雙引號**`"`。 <br>針對對應資料流程，`quoteChar` 不可以是空字串。 <br>若為複製活動，當 `quoteChar` 定義為空字串時，表示沒有引號字元，而且資料行值未加上引號，而 `escapeChar` 用來將資料行分隔符號和其本身轉義。 | 否       |
| escapeChar       | 用來將引號括在引號值內的單一字元。<br>預設值為 [**反斜線] `\`** 。 <br>針對對應資料流程，`escapeChar` 不可以是空字串。 <br/>針對複製活動，當 `escapeChar` 定義為空字串時，`quoteChar` 也必須設為空字串，在此情況下，請確定所有資料行值都不包含分隔符號。 | 否       |
| firstRowAsHeader | 指定是否要將第一個資料列視為具有資料行名稱的標頭行。<br>允許的值為**true**和**false** （預設值）。 | 否       |
| nullValue        | 指定 null 值的字串表示。 <br>預設值為**空字串**。 | 否       |
| encodingName     | 用來讀取/寫入測試檔案的編碼類型。 <br>允許的值如下： "UTF-8"、"UTF-16"、"UTF-UTF-16BE"、"UTF-32"、"32BE"、"US-ASCII"，"UTF-7"，"BIG5"，"EUC-JP"，"EUC-KR"，"GB2312"，"GB18030"，"JOHAB"，"SHIFT-JIS"，"CP875"，"CP866"，"IBM00858"，"IBM037"，"IBM273"，"IBM437"，"IBM500"，"IBM737", "IBM775", "IBM850", "IBM852", "IBM855", "IBM857", "IBM860", "IBM861", "IBM863", "IBM864", "IBM865", "IBM869", "IBM870", "IBM01140", "IBM01141", "IBM01142", "IBM01143", "IBM01144", "IBM01145", "IBM01146", "IBM01147", "IBM01148", "IBM01149"，"ISO-2022-JP"，"ISO-2022-KR"，"ISO-8859-1"，"ISO-8859-2"，"ISO-8859-3"，"ISO-8859-4"，"ISO-8859-5"，"ISO-8859-6"，"ISO-8859-7"，"ISO-8859-8"，"ISO-8859-9"，"ISO-8859-13"，"ISO-8859-15"，"WINDOWS-874"，"WINDOWS-1250"，"WINDOWS-1251"，"WINDOWS-1252 "，" WINDOWS-1253 "，" WINDOWS-1254 "，" WINDOWS-1255 "，" WINDOWS-1256 "，" WINDOWS-1257 "，" WINDOWS-1258 "。<br>注意：對應的資料流程不支援 UTF-7 編碼。 | 否       |
| compressionCodec | 用來讀取/寫入文字檔的壓縮編解碼器。 <br>允許的值為**bzip2**、 **gzip**、 **deflate**、 **ZipDeflate**、 **snappy**或**lz4**。 表示在儲存檔案時使用。 <br>注意：目前的複製活動不支援 "snappy" & "lz4"，而且對應資料流程不支援 "ZipDeflate"。 <br>請注意，使用複製活動來解壓縮 ZipDeflate 檔案並寫入檔案型接收資料存放區時，檔案將會解壓縮到資料夾： `<path specified in dataset>/<folder named as source zip file>/`。 | 否       |
| compressionLevel | 壓縮比例。 <br>允許的值為**最佳**或**最快**。<br>- **最快：** 即使未以最佳方式壓縮所產生的檔案，壓縮作業還是應該儘快完成。<br>- **最佳**：即使作業需要較長的時間才能完成，壓縮作業也應以最佳方式壓縮。 如需詳細資訊，請參閱 [壓縮層級](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx) 主題。 | 否       |

以下是 Azure Blob 儲存體上的分隔文字資料集範例：

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供分隔的文字來源和接收所支援的屬性清單。

### <a name="delimited-text-as-source"></a>分隔的文字做為來源 

以下支援的屬性將複製活動中 ***\*來源\**** 一節。

| 屬性       | 說明                                                  | 必要項 |
| -------------- | ------------------------------------------------------------ | -------- |
| type           | 複製活動來源的類型屬性必須設定為**DelimitedTextSource**。 | 是      |
| formatSettings | 屬性的群組。 請參閱下方的**分隔文字讀取設定**表格。 | 否       |
| storeSettings  | 一組屬性，說明如何從資料存放區讀取資料。 每個以檔案為基礎的連接器在 `storeSettings`之下都有自己支援的讀取設定。 | 否       |

`formatSettings`下支援的**分隔文字讀取設定**：

| 屬性      | 說明                                                  | 必要項 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | FormatSettings 的類型必須設定為**DelimitedTextReadSettings**。 | 是      |
| skipLineCount | 指出從輸入檔讀取資料時，要略過的**非空白**資料列數。 <br>如果同時指定 skipLineCount 和 firstRowAsHeader，則會先略過行，然後從輸入檔讀取標頭資訊。 | 否       |

### <a name="delimited-text-as-sink"></a>分隔的文字做為接收器

以下支援的屬性將複製活動中 ***\*接收器\**** 一節。

| 屬性       | 說明                                                  | 必要項 |
| -------------- | ------------------------------------------------------------ | -------- |
| type           | 複製活動來源的類型屬性必須設定為**DelimitedTextSink**。 | 是      |
| formatSettings | 屬性的群組。 請參閱下方的**分隔文字寫入設定**表格。 |          |
| storeSettings  | 一組屬性，說明如何將資料寫入資料存放區。 每個以檔案為基礎的連接器在 `storeSettings`之下都有它自己的支援寫入設定。  | 否       |

`formatSettings`下支援的**分隔文字寫入設定**：

| 屬性      | 說明                                                  | 必要項                                              |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| type          | FormatSettings 的類型必須設定為**DelimitedTextWriteSettings**。 | 是                                                   |
| fileExtension | 用來命名輸出檔的副檔名，例如 `.csv`、`.txt`。 當輸出 DelimitedText 資料集中未指定 `fileName` 時，就必須指定它。 在輸出資料集中設定檔案名時，將會使用該名稱做為接收檔案名，而且將會忽略副檔名設定。  | 當輸出資料集中未指定檔案名時為是 |

## <a name="mapping-data-flow-properties"></a>對應資料流程屬性

深入瞭解對應資料流程中[來源轉換](data-flow-source.md)和[接收轉換](data-flow-sink.md)的詳細資料。

## <a name="next-steps"></a>後續步驟

- [複製活動概觀](copy-activity-overview.md)
- [對應資料流程](concepts-data-flow-overview.md)
- [查閱活動](control-flow-lookup-activity.md)
- [GetMetadata 活動](control-flow-get-metadata-activity.md)
