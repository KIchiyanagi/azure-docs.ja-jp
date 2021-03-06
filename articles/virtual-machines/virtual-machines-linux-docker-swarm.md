---
title: "Azure で Docker Swarm の使用を開始する"
description: "Docker VM 拡張機能を使って VM のグループを作成する方法と、Swarm を使って Docker クラスターを作成する方法について説明します。"
services: virtual-machines-linux
documentationcenter: virtual-machines
author: squillace
manager: timlt
editor: tysonn
tags: azure-service-management
ms.assetid: d529b1f5-864e-4163-9b34-b52d48ceedb1
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 01/04/2016
ms.author: rasquill
translationtype: Human Translation
ms.sourcegitcommit: f6537e4ebac76b9f3328223ee30647885ee15d3e
ms.openlocfilehash: 834bb8a60c3dcb2d0b50809ffb0e4780579b2cda


---
# <a name="how-to-use-docker-with-swarm"></a>Docker Swarm を使用する方法
> [!IMPORTANT] 
> Azure には、リソースの作成と操作に関して、 [Resource Manager とクラシック](../azure-resource-manager/resource-manager-deployment-model.md)の 2 種類のデプロイメント モデルがあります。 この記事では、クラシック デプロイ モデルの使用方法について説明します。 最新のデプロイでは、リソース マネージャー モデルを使用することをお勧めします。 Docker Swarm をデプロイするための Resource Manager テンプレートについては、[こちら](https://azure.microsoft.com/documentation/templates/docker-swarm-cluster/)を参照してください。

このトピックでは、[docker](https://www.docker.com/) と [swarm](https://github.com/docker/swarm) を使用して Azure で swarm 管理クラスターを作成する非常に簡単な方法を示します。 ここでは Azure で 4 つの仮想マシンが作成されます。1 つは Swarm マネージャーとして動作し、その他の 3 つは Docker ホストのクラスターの一部として機能します。 終了すれば、Swarm を使ってクラスターを確認し、クラスター上で Docker を使用できるようになります。 さらに、このトピックで使用される Azure CLI 呼び出しは、サービス管理 (asm) モードを使用します。 

> [!NOTE]
> このトピックでは、異なるツールがそれぞれ独立性を保ちながら連携するしくみを示すため、 *docker-machine* を **使用せずに** 、docker を swarm および Azure CLI と共に使用します。 **docker-machine** には、**docker-machine** を使用してswarm にノードを直接追加できる **--swarm** スイッチがあります。 具体例については、 [docker-machine](https://github.com/docker/machine) のドキュメントを参照してください。 Azure VM に対して実行している **docker-machine** がない場合は、「[Azure で docker-machine を使用する方法](virtual-machines-linux-docker-machine.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)」を参照してください。
> 
> 

## <a name="create-docker-hosts-with-azure-virtual-machines"></a>Azure 仮想マシンでの Docker ホストの作成
ここでは 4 つの VM を作成しますが、作成する VM の数は任意に決められます。 *&lt;password&gt;* をご自分のものに置き換え、次のように呼び出します。

    azure vm docker create swarm-master -l "East US" -e 22 $imagename ops <password>
    azure vm docker create swarm-node-1 -l "East US" -e 22 $imagename ops <password>
    azure vm docker create swarm-node-2 -l "East US" -e 22 $imagename ops <password>
    azure vm docker create swarm-node-3 -l "East US" -e 22 $imagename ops <password>

呼び出しが終了すると、 **azure vm list** を使って Azure VM を確認できるようになります。

    $ azure vm list | grep "swarm-[mn]"
    data:    swarm-master     ReadyRole           East US       swarm-master.cloudapp.net                               100.78.186.65
    data:    swarm-node-1     ReadyRole           East US       swarm-node-1.cloudapp.net                               100.66.72.126
    data:    swarm-node-2     ReadyRole           East US       swarm-node-2.cloudapp.net                               100.72.18.47  
    data:    swarm-node-3     ReadyRole           East US       swarm-node-3.cloudapp.net                               100.78.24.68  

## <a name="installing-swarm-on-the-swarm-master-vm"></a>Swarm マスター VM での Swarm のインストール
このトピックでは、 [Docker Swarm ドキュメントのインストールするコンテナーのモデル](https://github.com/docker/swarm#1---docker-image) を使用しますが、 **swarm-master**に対する SSH を使うこともできます。 このモデルでは、 **Swarm** は Swarm を実行する Docker コンテナーとしてダウンロードされます。 下記では、*Docker を使ってノート パソコンからリモートで*この手順を実行し、**swarm-master** VM に接続してクラスターの ID 生成コマンドである **swarm create** を使うように指定しています。 クラスター ID を使うことで、 **Swarm** で Swarm グループのメンバーを見つけられるようになります(リポジトリを複製して自身で構築することも可能です。 そうすると完全に制御でき、デバッグもできます)。

    $ docker --tls -H tcp://swarm-master.cloudapp.net:2376 run --rm swarm create
    Unable to find image 'swarm:latest' locally
    511136ea3c5a: Pull complete
    a8bbe4db330c: Pull complete
    9dfb95669acc: Pull complete
    0b3950daf974: Pull complete
    633f3d9a9685: Pull complete
    bba5f98a0414: Pull complete
    defbc1ab4462: Pull complete
    92d78d321ff2: Pull complete
    swarm:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
    Status: Downloaded newer image for swarm:latest
    36731c17189fd8f450c395db8437befd

最後の行がクラスター ID です。ノード VM を Swarm マスターに結合して「Swarm」を作成する際に再度使いますので、どこかに保存しておいてください）。 この例では、クラスター ID は **36731c17189fd8f450c395db8437befd** です。

> [!NOTE]
> ここでは、ローカルにインストールされている Docker を使用して Azure の **swarm-master** VM に接続しています。また、**swarm-master** で **create** コマンドをダウンロード、インストール、実行する手順を説明します。このコマンドによって、後で検索に使用するクラスター ID が返されます。
> <!-- -->
> 確認するには、`docker -H tcp://`*&lt;hostname&gt;* ` images` を実行して **swarm-master** マシンや、比較のための別のノードで処理されるコンテナーを一覧表示します (前に Swarm コマンドを **--rm** スイッチで実行したため、終了後にコンテナーが削除されます。**docker ps -a** を使用すると、なにも返されなくなります)。
> 
> 

        $ docker --tls -H tcp://swarm-master.cloudapp.net:2376 images
        REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
        swarm               latest              92d78d321ff2        11 days ago         7.19 MB
        $ docker --tls -H tcp://swarm-node-1.cloudapp.net:2376 images
        REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
        $
<P />

> **Docker** の使用経験がある場合は、イメージがダウンロードされておらず実行されていないため、その他のノードにエントリがないことはご存知でしょう。
> 
> 

## <a name="join-the-node-vms-to-our-docker-cluster"></a>ノード VM の Docker クラスターへの結合
各ノードで Azure CLI を使ってエンドポイント情報を一覧表示します。 ここではノードの Docker ポートを取得するため、 **swarm-node-1** Docker ホストに対してこれを行っています。

    $ azure vm endpoint list swarm-node-1
    info:    Executing command vm endpoint list
    + Getting virtual machines
    data:    Name    Protocol  Public Port  Private Port  Virtual IP      EnableDirectServerReturn  Load Balanced
    data:    ------  --------  -----------  ------------  --------------  ------------------------  -------------
    data:    docker  tcp       2376         2376          138.91.112.194  false                     No
    data:    ssh     tcp       22           22            138.91.112.194  false                     No
    info:    vm endpoint list command OK


**Docker** と `-H` オプションを使用してノード VM における Docker クライアントを指定し、クラスター ID とノードの Docker ポート (**--addr** を使用) を送って作成している Swarm にそのノードを結合します。

    $ docker --tls -H tcp://swarm-node-1.cloudapp.net:2376 run -d swarm join --addr=138.91.112.194:2376 token://36731c17189fd8f450c395db8437befd
    Unable to find image 'swarm:latest' locally
    511136ea3c5a: Pull complete
    a8bbe4db330c: Pull complete
    9dfb95669acc: Pull complete
    0b3950daf974: Pull complete
    633f3d9a9685: Pull complete
    bba5f98a0414: Pull complete
    defbc1ab4462: Pull complete
    92d78d321ff2: Pull complete
    swarm:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
    Status: Downloaded newer image for swarm:latest
    bbf88f61300bf876c6202d4cf886874b363cd7e2899345ac34dc8ab10c7ae924

問題ないようです。 **swarm** が **swarm-node-1** で実行されていることを確認するには、次のように入力します。

    $ docker --tls -H tcp://swarm-node-1.cloudapp.net:2376 ps -a
        CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
        bbf88f61300b        swarm:latest        "/swarm join --addr=   13 seconds ago      Up 12 seconds       2375/tcp            angry_mclean

クラスターのすべてのノードに対して繰り返します。 ここでは、**swarm-node-2** と **swarm-node-3** に対して行っています。

## <a name="begin-managing-the-swarm-cluster"></a>Swarm クラスター管理の開始
    $ docker --tls -H tcp://swarm-master.cloudapp.net:2376 run -d -p 2375:2375 swarm manage token://36731c17189fd8f450c395db8437befd
    d7e87c2c147ade438cb4b663bda0ee20981d4818770958f5d317d6aebdcaedd5

これで次のようにクラスターのノードを一覧表示できます。

    ralph@local:~$ docker --tls -H tcp://swarm-master.cloudapp.net:2376 run --rm swarm list token://73f8bc512e94195210fad6e9cd58986f
    54.149.104.203:2375
    54.187.164.89:2375
    92.222.76.190:2375

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>次のステップ
Swarm でお試しください。 詳細については、[https://github.com/docker/swarm/](https://github.com/docker/swarm/) や、[ビデオ](https://www.youtube.com/watch?v=EC25ARhZ5bI)もご覧ください。

<!-- links -->

[docker-machine-azure]: virtual-machines-linux-docker-machine.md




<!--HONumber=Dec16_HO1-->


