---
title: 建立具有叢集的進階 Azure Cache for Redis - Azure CLI
description: 此 Azure CLI 程式碼範例示範如何建立啟用叢集且具有兩分區的 6 GB 進階層 Azure Cache for Redis。
author: yegu-ms
ms.author: yegu
tags: azure-service-management
ms.service: cache
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/30/2017
ms.openlocfilehash: 68203fe2e054f32ce5764fe4f1b07013b0806104
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/25/2019
ms.locfileid: "75411154"
---
# <a name="create-a-premium-azure-cache-for-redis-with-clustering"></a>建立具有叢集的進階 Azure Cache for Redis

在此案例中，您將學習如何建立啟用叢集和兩分區的 6 GB 進階層 Azure Cache for Redis。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/redis-cache/create-premium-cache-cluster/create-premium-cache-cluster.sh "Azure Cache for Redis")]

[!INCLUDE [cli-script-clean-up](../../../includes/redis-cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組和啟用叢集的進階層 Azure Cache for Redis。 下表中的每個命令都會連結至命令特定的文件。

| Command | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group) | 建立用來存放所有資源的資源群組。 |
| [az redis create](https://docs.microsoft.com/cli/azure/redis) | 建立 Azure Cache for Redis 執行個體。 |


## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Cache for Redis 文件](../cli-samples.md)中找到其他 Azure Cache for Redis CLI 指令碼範例。
