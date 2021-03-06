---
title: "Azure Container Service クラスターのスケーリング | Microsoft Docs"
description: "Azure CLI または Azure Portal を使用して Azure Container Service クラスターをスケーリングする方法を説明します。"
services: container-service
documentationcenter: 
author: sauryadas
manager: madhana
editor: 
tags: acs, azure-container-service
keywords: "Docker、コンテナー、マイクロ サービス、Mesos、Azure"
ms.assetid: 4a567474-f9a2-4172-bf86-7522aa4d4d80
ms.service: container-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/10/2017
ms.author: saudas
translationtype: Human Translation
ms.sourcegitcommit: cb3fd28659eb09dfb74496d2aa526736d223631a
ms.openlocfilehash: d1571aa6191111c46c43b3a424cea415091adfc9


---
# <a name="scale-an-azure-container-service-cluster"></a>Azure Container Service クラスターのスケーリング
[Azure Container Service クラスターのデプロイ](container-service-deployment.md)を行った後、エージェントノード数の変更が必要になることがあります。 たとえば、もっと多くのコンテナー アプリケーションまたはインスタンスを実行できるように、より多くのエージェントが必要になることがあります。 

クラスター内のエージェントノードの数は、Azure Portal または Azure CLI 2.0 (プレビュー) を使用して変更できます。 Azure CLI 2.0 (プレビュー) は、Resource Manager デプロイ モデル用の[次世代の CLI](/cli/azure/old-and-new-clis) です。

> [!NOTE]
> 現時点では、コンテナー サービス Kubernetes クラスター内のエージェント ノードのスケーリングはサポートされません。


## <a name="scale-with-the-azure-portal"></a>Azure Portal を使用したスケーリング

1. [Azure Portal](https://portal.azure.com) で、**コンテナー サービス**を探し、変更するコンテナー サービスをクリックします。
2. **[コンテナー サービス]** ブレードで、**[エージェント]** をクリックします。
3. **[VM 数]** に、目的のエージェント ノードの数を入力します。

    ![ポータルでのプールのスケーリング](./media/container-service-scale/container-service-scale-portal.png)

4. 構成を保存するには、**[保存]** をクリックします。



## <a name="scale-with-the-azure-cli-20-preview"></a>Azure CLI 2.0 (プレビュー) を使用したスケーリング

Azure CLI 2.0 (プレビュー) の最新版が[インストール済み](/cli/azure/install-az-cli2)であることと、Azure アカウントにログインしていること (`az login`) を確認してください。


### <a name="see-the-current-agent-count"></a>現在のエージェント数の表示
クラスターの現在のエージェント数を表示するには、`az acs show` コマンドを実行します。 クラスター構成が表示されます。 たとえば、次のコマンドは、リソース グループ `myResourceGroup` 内の `containerservice-myACSName` という名前のコンテナー サービスの構成を表示します。

```azurecli
az acs show -g myResourceGroup -n containerservice-myACSName
```

このコマンドは、`AgentPoolProfiles` の `Count` 値に、エージェント数を返します。


### <a name="use-the-az-acs-scale-command"></a>az acs scale コマンドの使用
エージェントノードの数を変更するには、**リソース グループ**、**コンテナー サービス名**、および必要な**新しいエージェント数**を指定して、`az acs scale` コマンドを実行します。 数を増やしてスケールアップを、数を減らしてスケールダウンを実行できます。

たとえば、前のクラスターのエージェント数を 10 に変更するには、次のコマンドを入力します。

```azurecli
azure acs scale -g myResourceGroup -n containerservice-myACSName --new-agent-count 10
```

Azure CLI 2.0 (プレビュー) は、新しいエージェント数を含む、コンテナー サービスの新しい構成を表す JSON 文字列を返します。

その他のコマンド オプションについては、`az acs scale --help` を実行します。


## <a name="scaling-considerations"></a>スケーリングに関する考慮事項


* エージェント ノードの数は、1 ～ 100 の範囲にする必要があります。 

* コア クォータによって、クラスター内のエージェント ノードの数が制限される可能性があります。

* エージェント ノードのスケーリング操作は、エージェント プールを含む Azure 仮想マシン スケール セットに適用されます。 DC/OS クラスターでは、プライベート プール内のエージェント ノードのみが、この記事で説明している操作によってスケーリングされます。

* クラスターにデプロイしている Orchestrator によっては、クラスターで実行されるコンテナーのインスタンス数を個別にスケーリングできます。 たとえば、DC/OS クラスターでは、[Marathon UI](container-service-mesos-marathon-ui.md) を使用して、コンテナー アプリケーションのインスタンス数を変更します。

* 現時点では、コンテナー サービス クラスター内のエージェント ノードのスケーリングはサポートされていません。





## <a name="next-steps"></a>次のステップ
* Azure Container Service での Azure CLI 2.0 (プレビュー) のコマンドの[他の使用例](container-service-create-acs-cluster-cli.md)を参照します。
* Azure Container Service の [DC/OS エージェント プール](container-service-dcos-agents.md)の詳細を確認します。




<!--HONumber=Jan17_HO2-->


