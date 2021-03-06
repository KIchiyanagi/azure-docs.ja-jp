---
title: "Azure Portal で Windows VM の FQDN を作成する | Microsoft Docs"
description: "Azure Portal で仮想マシンに基づいて、Resource Manager の完全修飾ドメイン名 (FQDN) を作成する方法を説明します。"
services: virtual-machines-windows
documentationcenter: 
author: iainfoulds
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: a2ae5887-76df-485e-ae19-0efd96df8600
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 102/09/2017
ms.author: iainfou
translationtype: Human Translation
ms.sourcegitcommit: 746ed119239d5a0a71a0663e81d9c5b259354419
ms.openlocfilehash: e89e5769701f00c6f403bd54cb2b29bcce53bf15


---
# <a name="create-a-fully-qualified-domain-name-in-the-azure-portal"></a>Azure ポータルでの完全修飾ドメイン名の作成
Resource Manager デプロイメント モデルを使用して [Azure ポータル](https://portal.azure.com) で仮想マシン (VM) を作成すると、仮想マシン用のパブリック IP リソースが自動的に作成されます。 この IP アドレスを使用して、VM にリモートでアクセスします。 ポータルの既定では [完全修飾ドメイン名](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)(FQDN) が作成されませんが、VM の作成後に作成できます。 この記事では、DNS 名または FQDN を作成する手順を示します。

[!INCLUDE [virtual-machines-common-portal-create-fqdn](../../includes/virtual-machines-common-portal-create-fqdn.md)]

リモート デスクトップ プロトコル (RDP) 用など、この DNS 名を使用して、VM にリモートで接続できるようになります。

## <a name="next-steps"></a>次のステップ
これで、VM にパブリック IP と DNS 名が設定されたため、IIS、SQL、SharePoint などの一般的なアプリケーション フレームワークやサービスをデプロイできるようになりました。

Azure デプロイの構築に関するヒントについては、[Resource Manager の使用](../azure-resource-manager/resource-group-overview.md)に関する記事から確認することもできます。




<!--HONumber=Jan17_HO4-->


