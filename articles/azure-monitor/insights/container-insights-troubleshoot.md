---
title: 如何對適用於容器的 Azure 監視器進行疑難排解 | Microsoft Docs
description: 本文說明如何對適用於容器的 Azure 監視器問題進行疑難排解並解決問題。
ms.topic: conceptual
ms.date: 10/15/2019
ms.openlocfilehash: 17a2817b320599b2aa2c331c354d316b9d864a32
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/25/2019
ms.locfileid: "75403371"
---
# <a name="troubleshooting-azure-monitor-for-containers"></a>對適用於容器的 Azure 監視器進行疑難排解

當您設定使用適用於容器的 Azure 監視器來監視 Azure Kubernetes Service (AKS) 叢集時，您可能會遇到阻止資料收集或報告狀態的問題。 本文將詳細說明一些常見問題與疑難排解步驟。

## <a name="authorization-error-during-onboarding-or-update-operation"></a>上線或更新作業期間發生授權錯誤
啟用容器的 Azure 監視器或更新叢集以支援收集計量時，您可能會收到類似下列的錯誤-*用戶端 < 使用者的身分識別 > '，其物件識別碼為 ' < 使用者的 objectId > ' 沒有在範圍內執行 ' Microsoft. authorization/roleAssignments/write ' 動作的授權*

在上架或更新程式期間，會嘗試對叢集資源授與**監視計量發行者**角色指派。 起始進程以啟用容器 Azure 監視器或更新以支援計量集合的使用者，必須能夠存取 AKS 叢集資源範圍上的**roleAssignments/write**許可權。 只有**擁有**者和**使用者存取系統管理員**內建角色的成員，才會被授與此許可權的存取權。 如果您的安全性原則需要指派細微層級的許可權，建議您查看[自訂角色](../../role-based-access-control/custom-roles.md)，並將它指派給需要它的使用者。 

您也可以執行下列步驟，從 Azure 入口網站手動授與此角色：

1. 登入 [Azure 入口網站](https://portal.azure.com)。 
2. 在 Azure 入口網站中，按一下左上角的 [所有服務]。 在資源清單中，輸入**Kubernetes**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [ **Azure Kubernetes**]。
3. 在 Kubernetes 叢集清單中，從清單中選取一個。
2. 從左側功能表中，按一下 [**存取控制（IAM）** ]。
3. 選取 [ **+ 新增**] 以新增角色指派，並選取 [**監視計量] [發行者]** 角色，然後在 [**選取**] 方塊中輸入**AKS** ，只在訂用帳戶中定義的叢集服務主體上篩選結果。 從該叢集專屬的清單中選取一個。
4. 選取 [儲存] 以完成角色指派。 

## <a name="azure-monitor-for-containers-is-enabled-but-not-reporting-any-information"></a>適用於容器的 Azure 監視器已啟用但未報告任何資訊
如果成功啟用和設定容器的 Azure 監視器，但是您無法查看狀態資訊，或記錄查詢未傳回任何結果，請遵循下列步驟來診斷問題： 

1. 執行下列命令來檢查代理程式的狀態： 

    `kubectl get ds omsagent --namespace=kube-system`

    輸出應該像下面這樣，這表示它已正確部署：

    ```
    User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
    NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
    omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
    ```  
2. 使用下列命令，利用代理程式版本 *06072018* 或更新版本來檢查部署狀態：

    `kubectl get deployment omsagent-rs -n=kube-system`

    輸出應該會像下列範例，這表示它已正確部署：

    ```
    User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system 
    NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
    omsagent   1         1         1            1            3h
    ```

3. 執行下列命令來檢查 Pod 的狀態，以確認其正在執行：`kubectl get pods --namespace=kube-system`

    輸出應該會像下列範例，且 omsagent 的狀態為「執行中」：

    ```
    User@aksuser:~$ kubectl get pods --namespace=kube-system 
    NAME                                READY     STATUS    RESTARTS   AGE 
    aks-ssh-139866255-5n7k5             1/1       Running   0          8d 
    azure-vote-back-4149398501-7skz0    1/1       Running   0          22d 
    azure-vote-front-3826909965-30n62   1/1       Running   0          22d 
    omsagent-484hw                      1/1       Running   0          1d 
    omsagent-fkq7g                      1/1       Running   0          1d 
    ```

4. 檢查代理程式記錄。 容器化的代理程式完成部署時，它會執行 OMI 命令並顯示代理程式與提供者版本，以執行快速檢查。 

5. 若要確認代理程式是否已成功部署，請執行下列命令： `kubectl logs omsagent-484hw --namespace=kube-system`

    狀態應該會像下列範例：

    ```
    User@aksuser:~$ kubectl logs omsagent-484hw --namespace=kube-system
    :
    :
    instance of Container_HostInventory
    {
        [Key] InstanceID=3a4407a5-d840-4c59-b2f0-8d42e07298c2
        Computer=aks-nodepool1-39773055-0
        DockerVersion=1.13.1
        OperatingSystem=Ubuntu 16.04.3 LTS
        Volume=local
        Network=bridge host macvlan null overlay
        NodeRole=Not Orchestrated
        OrchestratorType=Kubernetes
    }
    Primary Workspace: b438b4f6-912a-46d5-9cb1-b44069212abc    Status: Onboarded(OMSAgent Running)
    omi 1.4.2.2
    omsagent 1.6.0.23
    docker-cimprov 1.0.0.31
    ```

## <a name="error-messages"></a>錯誤訊息

下表簡要說明使用適用於容器的 Azure 監視器時，可能遇到的已知錯誤。

| 錯誤訊息  | 行動 |  
| ---- | --- |  
| 錯誤訊息 `No data for selected filters`  | 為新建立的叢集打造監視資料流程可能需要點時間。 允許至少10到15分鐘的時間，讓您的叢集顯示資料。 |   
| 錯誤訊息 `Error retrieving data` | 雖然 Azure Kubernetes Service 叢集是針對健全狀況和效能監視進行設定，但叢集與 Azure Log Analytics 工作區之間會建立連線。 Log Analytics 工作區是用來儲存叢集的所有監視資料。 當您的 Log Analytics 工作區已刪除時，可能會發生此錯誤。 檢查工作區是否已刪除，如果是，您將需要使用容器的 Azure 監視器來重新啟用叢集的監視，並指定現有的或建立新的工作區。 若要重新啟用，您將需要[停](container-insights-optout.md)用叢集的監視，並再次[啟用](container-insights-enable-new-cluster.md)容器的 Azure 監視器。 |  
| 透過 az aks cli 新增適用於容器的 Azure 監視器後，會出現 `Error retrieving data` | 使用 `az aks cli`啟用監視時，可能無法正確部署容器的 Azure 監視器。 檢查是否已部署解決方案。 若要檢查，請前往 Log Analytics 工作區，選取左側窗格的 [Solutions (解決方案)]，查看解決方案是否可使用。 若要解決此問題，您必須按照[如何部署適用於容器的 Azure 監視器](container-insights-onboard.md)的指示，重新部署這個解決方案 |  

為協助診斷問題，[在此](https://github.com/Microsoft/OMS-docker/tree/ci_feature_prod/Troubleshoot#troubleshooting-script)提供疑難排解指令碼。

## <a name="azure-monitor-for-containers-agent-replicaset-pods-are-not-scheduled-on-non-azure-kubernetes-cluster"></a>未在非 Azure Kubernetes 叢集上排程容器代理程式 ReplicaSet pod 的 Azure 監視器

容器代理程式 ReplicaSet pod 的 Azure 監視器相依于排程的背景工作角色（或代理程式）節點上的下列節點選取器：

```
nodeSelector:
  beta.kubernetes.io/os: Linux
  kubernetes.io/role: agent
```

如果您的背景工作節點未附加節點標籤，則不會排程代理程式 ReplicaSet pod。 如需如何附加標籤的指示，請參閱[Kubernetes 指派標籤選取器](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)。

## <a name="performance-charts-dont-show-cpu-or-memory-of-nodes-and-containers-on-a-non-azure-cluster"></a>效能圖表不會顯示非 Azure 叢集上節點和容器的 CPU 或記憶體

容器的 Azure 監視器代理程式 pod 會使用節點代理程式上的 cAdvisor 端點來收集效能計量。 確認節點上的容器化代理程式已設定為允許在叢集中的所有節點上開啟 `cAdvisor port: 10255`，以收集效能計量。

## <a name="non-azure-kubernetes-cluster-are-not-showing-in-azure-monitor-for-containers"></a>非 Azure Kubernetes 叢集不會顯示在容器的 Azure 監視器中

若要在適用于容器的 Azure 監視器中，查看非 Azure Kubernetes 叢集，支援此深入解析的 Log Analytics 工作區和 Container Insights 解決方案資源**ContainerInsights （*工作區*）** 上都需要讀取權限。

## <a name="next-steps"></a>後續步驟

藉由啟用監視來擷取 AKS 叢集節點和 Pod 的健康情況計量，這些健康情況計量都可在 Azure 入口網站中取得。 若要了解如何使用適用於容器的 Azure 監視器，請參閱[檢視 Azure Kubernetes Service 健康情況](container-insights-analyze.md)。
