---
title: "アプリケーションのシナリオと設計 | Microsoft Docs"
description: "Service Fabric のクラウド アプリケーションのカテゴリの概要 ステートフル サービスとステートレス サービスを使用したアプリケーションの設計について説明します。"
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: 
ms.assetid: 3a8ca6ea-b8e9-4bc3-9e20-262437d2528e
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/22/2016
ms.author: mfussell
translationtype: Human Translation
ms.sourcegitcommit: dfc2af63c7acb1fef47abb329a385cd7448d2186
ms.openlocfilehash: 6bbffcfd8acc531fd70c9db90a9f57d8db942002


---
# <a name="service-fabric-application-scenarios"></a>Service Fabric アプリケーションのシナリオ
Azure Service Fabric はさまざまな種類のビジネス アプリケーションやサービスを作成し実行できる信頼性の高い柔軟なプラットフォームを提供します。 これらのアプリケーションとマイクロサービスはステートレスまたはステートフルが考えられますが、最大限に効率化するために仮想マシン間でリソース分散されます。 Service Fabric 独自のアーキテクチャでは、リアルタイムに近いデータ分析、メモリ内のコンピューティング、並列トランザクション、アプリケーションでのイベント処理を実行できます。 リソース要件の変化に応じて、アプリケーションを簡単にスケール アップまたはスケール ダウン (実際にはスケール インまたはスケール アウト) できます。

Azure の Service Fabric プラットフォームは、次のカテゴリのアプリケーションやサービスに最適です。

* **高可用性サービス**: Service Fabric のサービスは、複数のセカンダリ サービス レプリカを作成することによって、高速のフェールオーバーを実現します。 ハードウェアまたはその他の障害によってノード、プロセス、または個々のサービスがダウンした場合、セカンダリ レプリカの&1; つが最小限のサービス中断でプライマリ レプリカに昇格されます。
* **スケーラブルなサービス**: クラスター間でスケールアウトの状態になれるよう個々のサービスをパーティション分割できます。 さらに、動作中に個々のサービスの作成および削除を行うことができます。 サービスはリソースのニーズに応じて、迅速かつ簡単に、2 ～&3; 個のノード上の&2; ～&3; 個のインスタンスから、多数のノード上の数千ものインスタンスにスケールアウトし、その後再びスケールインすることができます。 Service Fabric を使用することで、このようなサービスを構築し、その完全なライフサイクルを管理することができます。
* **非静的データのコンピューティング**: Service Fabric では、データ、入出力、多くのコンピューティング処理を要するステートフルなアプリケーションを構築できます。 Service Fabric では、アプリケーションでの処理 (計算) とデータのコロケーションを実現できます。 アプリケーションがデータへのアクセスを必要とする場合、外部のデータ キャッシュまたはストレージ階層に起因するネットワーク待機時間が発生するのが一般的です。 ステートフルな Service Fabric サービスを使用した場合、ネットワーク待機時間が解消されるので、パフォーマンスに優れた読み取りおよび書き込みが行えます。 たとえば、顧客に対してリアルタイムに近い推奨選択を実行するアプリケーションがあり、そのラウンド トリップ時間の要件が 100 ミリ秒未満であるとします。 リモート ストレージから必要なデータを取得する必要がある標準的な実装モデルの場合と比較して、Service Fabric サービスの待機時間およびパフォーマンス特性 (この場合、推奨選択の計算がデータおよびルールと共存されている) は、ユーザーに応答性の高いエクスペリエンスを提供します。  
* **セッション ベースの対話型アプリケーション**: Service Fabric は、オンライン ゲームやインスタント メッセージングなどのアプリケーションで低待機時間の読み取りと書き込みが必要な場合に便利です。 Service Fabric では、ステートレスなアプリに必要な専用ストアやキャッシュを作成することなく、対話型のステートフルなアプリケーションを構築できます。 (待機時間が増加し、一貫性の問題が発生する可能性があります。)
* **グラフの処理を分散**: ソーシャル ネットワークの拡大が大幅に増えると、大規模なグラフを分析する必要性も増します。 高速のスケーリングと並列するロード処理により、Service Fabric は大規模なグラフを処理する自然なプラットフォームになります。 ソーシャル ネットワー キング、ビジネス インテリジェンス、研究などのグループにとって非常にスケーラブルなサービスを構築できます。
* **データ分析とワークフロー**: Service Fabric の高速な読み取りや書き込みにより、イベントやデータのストリームを正しく処理する必要のあるアプリケーションを実現できます。 Service Fabric では、処理パイプラインを表すアプリケーションも可能です。処理パイプラインでは、信頼できる結果を、損失なく、次の処理ステージに渡す必要があります。 たとえば、取引システムや金融システムです。データの一貫性と確実な計算が重要となります。

## <a name="application-design-case-studies"></a>アプリケーション設計のケース スタディ
アプリケーション設計における Service Fabric の使用方法を紹介するケース スタディが、[マイクロサービス ソリューション サイト](https://azure.microsoft.com/solutions/microservice-applications/)で多数公開されています。

## <a name="design-applications-composed-of-stateless-and-stateful-microservices"></a>ステートレスとステートフルなマイクロサービスから成るアプリケーションを設計する
Azure Cloud Service worker ロールでのアプリケーションの構築は、ステートレス サービスの一例です。 対照的に、ステートフルなマイクロサービスは、要求とその応答を超える権限のある状態を維持します。 正しい処理をレプリケーションでバックアップするシンプルな API 経由で状態の高可用性と一貫性が実現します。 Service Fabric のステートフルなサービスでは、高可用性を平等に実現可能にすることで、データベースなどのデータ ストアだけでなく、すべての種類のアプリケーションに高可用性をもたらします。 これは自然な進展です。 アプリケーションは既に、高可用性のための純粋なリレーショナル データベースから NoSQL データベースに移っています。 アプリケーション自体を「ホット」な状態にして、データを中で管理し、信頼性、一貫性、可用性を犠牲にすることなく、パフォーマンスを上げることができるようになりました。

マイクロサービスで構成されるアプリケーションを構築するとき、通常、ステートレスに呼び出すステートレスな Web アプリ (ASP.NET、Node.js など) と Service Fabric デプロイ コマンドを使ってすべてが同じ Service Fabric クラスターにデプロイされたステートフルなビジネス中間層サービスの組み合わせがあります。 これらのサービスは、スケール、信頼性、およびソース使用率について、それぞれ独立性を有しているため、開発環境とライフサイクル管理で機敏性が大幅に向上します。

純粋にステートレスなアプリケーションの可用性と待機時間の要件に対応するために従来必要だった追加のキューとキャッシュは、ステートフル マイクロサービスでは不要になるため、アプリケーション設計が単純化されます。 ステートフル サービスは高可用性と低待機時間を特徴とするため、アプリケーション全体では管理すべき変動要素が少なくなります。 次の図では、ステートレスとステートフルなアプリケーション設計の違いを示しています。 [Reliable Services](service-fabric-reliable-services-introduction.md) と [Reliable Actors](service-fabric-reliable-actors-introduction.md) プログラミング モデルを利用することで、高スループットと低待機時間を実現しながら、ステートフル サービスによりアプリケーションの複雑さが軽減されます。

## <a name="an-application-built-using-stateless-services"></a>ステートレス サービスを使用して構築されたアプリケーション
![ステートレス サービスを使用したアプリケーション][Image1]

## <a name="an-application-built-using-stateful-services"></a>ステートフル サービスを使用して構築されたアプリケーション
![ステートレス サービスを使用したアプリケーション][Image2]

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>次のステップ
* Service Fabric の [Reliable Services](service-fabric-reliable-services-quick-start.md) プログラミング モデルと [Reliable Actors](service-fabric-reliable-actors-get-started.md) プログラミング モデルで、ステートレス サービスとステートフル サービスの構築を始めます。
* パターンとシナリオの詳細については、[こちら](service-fabric-patterns-and-scenarios.md)をご覧ください。
* [お客様導入事例](https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=qDJnf86yC_5206218965
)について学習してください。
* 次のトピックもご覧ください。
  * [マイクロサービスについて](service-fabric-overview-microservices.md)
  * [サービスの状態を定義し、管理する](service-fabric-concepts-state.md)
  * [Service Fabric サービスの可用性](service-fabric-availability-services.md)
  * [Service Fabric サービスのスケーリング](service-fabric-concepts-scalability.md)
  * [Service Fabric サービスのパーティション分割](service-fabric-concepts-partitioning.md)

[Image1]: media/service-fabric-application-scenarios/AppwithStatelessServices.jpg
[Image2]: media/service-fabric-application-scenarios/AppwithStatefulServices.jpg



<!--HONumber=Feb17_HO3-->


