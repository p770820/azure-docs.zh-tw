---
title: Azure HDInsight 上的 Apache Spark 最佳作法
description: 瞭解在 Azure HDInsight 中使用 Apache Spark 的最佳做法。
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 09/18/2019
ms.openlocfilehash: 20033e52e862f086d1491c06d38cdf4f2c57ba8d
ms.sourcegitcommit: 1c9858eef5557a864a769c0a386d3c36ffc93ce4
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2019
ms.locfileid: "71106121"
---
# <a name="apache-spark-best-practices"></a>Apache Spark 最佳做法

本文提供在 Azure HDInsight 上使用 Apache Spark 的各種最佳作法。

## <a name="how-do-i-run-or-submit-spark-jobs"></a>如何? 執行或提交 Spark 作業？

| 選項 | 文件 |
|---|---|
| VSCode | [使用 Spark & Hive 工具進行 Visual Studio Code](../hdinsight-for-vscode.md) |
| Jupyter Notebook | [教學課程：在 Azure HDInsight 的 Apache Spark 叢集上載入資料和執行查詢](./apache-spark-load-data-run-query.md) |
| IntelliJ | [教學課程：使用 Azure Toolkit for IntelliJ 建立適用于 HDInsight 叢集的 Apache Spark 應用程式](./apache-spark-intellij-tool-plugin.md) |
| IntelliJ | [教學課程：使用 IntelliJ 為 HDInsight 中的 Apache Spark 建立 Scala Maven 應用程式](./apache-spark-create-standalone-application.md) |
| Zeppelin 筆記本 | [在 Azure HDInsight 上搭配使用 Apache Zeppelin Notebook 和 Apache Spark 叢集](./apache-spark-zeppelin-notebook.md) |
| 使用 Livy 提交遠端作業 | [使用 Apache Spark REST API 將遠端作業提交至 HDInsight Spark 叢集](./apache-spark-livy-rest-interface.md) |

## <a name="how-do-i-monitor-and-debug-spark-jobs"></a>如何? 監視和調試 Spark 作業？

| 選項 | 文件 |
|---|---|
| Azure Toolkit for IntelliJ | [使用 Azure Toolkit for IntelliJ （預覽）進行 spark 作業調試失敗](apache-spark-intellij-tool-failure-debug.md) |
| 透過 SSH Azure Toolkit for IntelliJ | [使用 Azure Toolkit for IntelliJ 透過 SSH 對 HDInsight 叢集上的 Apache Spark 應用程式進行本機或遠端偵錯](apache-spark-intellij-tool-debug-remotely-through-ssh.md) |
| 透過 VPN Azure Toolkit for IntelliJ | [使用 Azure Toolkit for IntelliJ 透過 VPN 在 HDInsight 中從遠端 Apache Spark 應用程式進行調試](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md) |
| Apache Spark 歷程記錄伺服器上的作業圖形 | [使用擴充的 Apache Spark 記錄伺服器對 Apache Spark 應用程式進行偵錯和診斷](./apache-azure-spark-history-server.md) |

## <a name="how-do-i-make-my-spark-jobs-run-more-efficiently"></a>如何? 讓我的 Spark 作業執行得更有效率嗎？

| 選項 | 文件 |
|---|---|
| IO 快取 | [使用 Azure HDInsight IO 快取來改善 Apache Spark 工作負載的效能（預覽）](./apache-spark-improve-performance-iocache.md) |
| 設定選項 | [將 Apache Spark 作業優化](./apache-spark-perf.md) |

## <a name="how-do-i-connect-to-other-azure-services"></a>如何? 連接到其他 Azure 服務嗎？

| 選項 | 文件 |
|---|---|
| HDInsight 上的 Apache Hive | [整合 Apache Spark 和 Apache Hive 與 Hive Warehouse Connector](../interactive-query/apache-hive-warehouse-connector.md) |
| HDInsight 上的 Apache HBase | [使用 Apache Spark 來讀取和寫入 Apache HBase 資料](../hdinsight-using-spark-query-hbase.md) |
| HDInsight 上的 Apache Kafka | [教學課程：搭配使用 Apache Spark 結構化串流與 HDInsight 上的 Apache Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md) |
| Azure Cosmos DB | [Azure Cosmos DB：在 Azure 平臺上執行 lambda 架構](../../cosmos-db/lambda-architecture.md) |

## <a name="what-are-my-storage-options"></a>我的儲存體選項有哪些？

| 選項 | 文件 |
|---|---|
| Data Lake Storage Gen2 | [搭配 Azure HDInsight 叢集使用 Data Lake Storage Gen2](../hdinsight-hadoop-use-data-lake-storage-gen2.md) |
| Data Lake Storage Gen1 | [搭配 Azure HDInsight 叢集使用 Data Lake Storage Gen1](../hdinsight-hadoop-use-data-lake-store.md) |
| Azure Blob 儲存體 | [搭配使用 Azure 儲存體與 Azure HDInsight 叢集](../hdinsight-hadoop-use-blob-storage.md) |

## <a name="next-steps"></a>後續步驟

* [設定 Apache Spark 設定](apache-spark-settings.md)
* [將 HDInsight 中的 Apache Spark 作業優化](apache-spark-perf.md)
