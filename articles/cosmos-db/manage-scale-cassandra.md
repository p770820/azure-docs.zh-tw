---
title: 在 Azure Cosmos DB 中使用 Cassandra API 進行彈性調整
description: 瞭解可用來調整 Azure Cosmos DB Cassandra API 帳戶的選項及其優點/缺點
author: TheovanKraay
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: thvankra
ms.openlocfilehash: 668e9ddadf151a86be0d8c09fc91b4c70db12f3a
ms.sourcegitcommit: 2823677304c10763c21bcb047df90f86339e476a
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/14/2020
ms.locfileid: "77210784"
---
# <a name="elastically-scale-an-azure-cosmos-db-cassandra-api-account"></a>彈性調整 Azure Cosmos DB Cassandra API 帳戶

有各種不同的選項可探索 Azure Cosmos DB API 的彈性本質，以進行 Cassandra。 若要瞭解如何在 Azure Cosmos DB 中有效率地進行調整，請務必瞭解如何布建適當數量的要求單位（RU/s），以考慮系統中的效能需求。 若要深入瞭解要求單位，請參閱[要求單位](request-units.md)一文。 

![資料庫作業會取用要求單位](./media/request-units/request-units.png)

## <a name="handling-rate-limiting-429-errors"></a>處理速率限制（429錯誤）

如果用戶端耗用的資源數超過您所布建的數量（RU/秒），Azure Cosmos DB 將會傳回速率限制（429）的錯誤。 Azure Cosmos DB 中的 Cassandra API 會將這些例外狀況轉譯成 Cassandra 原生通訊協定上的多載錯誤。 

如果您的系統不會受到延遲的影響，使用重試來處理輸送量速率限制可能就已足夠。 請參閱[java 程式碼範例](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample)，以瞭解如何在 java 中使用[Cassandra 重試原則](https://docs.datastax.com/drivers/java/2.0/com/datastax/driver/core/policies/RetryPolicy.html)的[Azure Cosmos DB 延伸](https://github.com/Azure/azure-cosmos-cassandra-extensions)模組，以透明的方式處理速率限制。 您也可以使用[Spark 延伸](https://mvnrepository.com/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper)模組來處理速率限制。

## <a name="manage-scaling"></a>管理調整

如果您需要將延遲降到最低，有一系列選項可用於管理 Cassandra API 中的調整和布建輸送量（ru）：

* [使用 Azure 入口網站手動](#use-azure-portal)
* [以程式設計方式使用控制平面功能](#use-control-plane)
* [以程式設計方式使用 CQL 命令搭配特定的 SDK](#use-cql-queries)
* [使用 Autopilot 動態地](#use-autopilot)

下列各節說明每種方法的優缺點。 接著，您可以決定最佳策略來平衡系統的規模調整需求、整體成本，以及解決方案的效率需求。

## <a id="use-azure-portal"></a>使用 Azure 入口網站

您可以使用 Azure 入口網站來調整 Azure Cosmos DB Cassandra API 帳戶中的資源。 若要深入瞭解，請參閱在[容器和資料庫](set-throughput.md)上布建輸送量一文。 本文說明在 Azure 入口網站中的[資料庫](set-throughput.md#set-throughput-on-a-database)或[容器](set-throughput.md#set-throughput-on-a-container)層級設定輸送量的相對優勢。 這些文章中提到的「資料庫」和「容器」詞彙分別對應至 Cassandra API 的「keyspace」和「資料表」。

這種方法的優點是，它是在資料庫上管理輸送量容量的直接通行方式。 不過，缺點是，在許多情況下，您進行調整的方法可能需要特定層級的自動化，以符合成本效益且高效能。 下一節將說明相關的案例和方法。

## <a id="use-control-plane"></a>使用控制平面

Azure Cosmos DB 的 Cassandra API 可讓您使用各種控制平面功能，以程式設計方式調整輸送量。 如需指引和範例，請參閱[Azure Resource Manager](manage-cassandra-with-resource-manager.md)、 [Powershell](powershell-samples-cassandra.md)和[Azure CLI](cli-samples-cassandra.md)文章。

這個方法的優點是，您可以根據計時器來自動相應增加或減少資源，以考慮尖峰活動或低活動期間。 請參閱[這裡](https://github.com/Azure-Samples/azure-cosmos-throughput-scheduler)的範例，瞭解如何使用 Azure Functions 和 Powershell 來完成此操作。

這種方法的缺點是，您無法即時回應無法預期的變更規模需求。 相反地，您可能需要在系統中、用戶端/SDK 層級，或使用[Autopilot](provision-throughput-autopilot.md)來運用應用程式內容。

## <a id="use-cql-queries"></a>使用特定 SDK 的 CQL 查詢

您可以針對指定的資料庫或容器執行[CQL ALTER 命令](cassandra-support.md#keyspace-and-table-options)，以動態方式調整系統的程式碼。

這種方法的優點是，它可讓您以符合您應用程式的自訂方式，動態回應規模需求。 使用這種方法，您仍然可以利用標準 RU/秒的費用和費率。 如果您的系統規模需求大多是可預測的（大約70% 或以上），使用 SDK 搭配 CQL 可能會比使用 Autopilot 更符合成本效益的方式來進行自動調整。 這種方法的缺點是，在速率限制的情況下執行重試，可能會增加延遲。

## <a id="use-autopilot"></a>使用 Autopilot

除了手動或以程式設計方式布建輸送量，您也可以在 Autopilot 模式中設定 Azure cosmos 容器。 Autopilot 模式會自動且立即調整為指定 RU 範圍內的耗用量需求，而不會影響 Sla。 若要深入瞭解，請參閱在[autopilot 模式中建立 Azure Cosmos 容器和資料庫](provision-throughput-autopilot.md)一文。

這種方法的優點是，它是在您的系統中管理調整需求的最簡單方式。 它保證不會**在設定的 RU 範圍內**套用速率限制。 缺點是，如果您的系統中的調整需求是可預測的，則 Autopilot 可能會比使用上述的定制控制平面或 SDK 層級方法更符合成本效益的方式處理您的調整需求。

## <a name="next-steps"></a>後續步驟

- 開始使用 Java 應用程式來[建立 Cassandra API 帳戶、資料庫及資料表](create-cassandra-api-account-java.md)。
