---
title: "Azure AD Connect: サポートされるトポロジ | Microsoft Docs"
description: "このトピックでは、Azure AD Connect のサポートされているトポロジとサポートされていないトポロジについて詳しく説明します。"
services: active-directory
documentationcenter: 
author: AndKjell
manager: femila
editor: 
ms.assetid: 1034c000-59f2-4fc8-8137-2416fa5e4bfe
ms.service: active-directory
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: identity
ms.topic: article
ms.date: 11/01/2016
ms.author: billmath
translationtype: Human Translation
ms.sourcegitcommit: 28b5da6098316f8fbe84966e0dac88f5b7d2cb1d
ms.openlocfilehash: e5983496cda061b28418dcf86c38fbe80c611503


---
# <a name="topologies-for-azure-ad-connect"></a>Azure AD Connect のトポロジ
このトピックでは、主要な統合ソリューションとして Azure AD Connect Sync を使用した、さまざまなオンプレミスと Azure AD のトポロジについて説明します。 サポートされている構成とサポートされていない構成の両方について説明します。

ドキュメント内の図の凡例:

| Description | アイコン |
| --- | --- |
| オンプレミスの Active Directory フォレスト |![AD](./media/active-directory-aadconnect-topologies/LegendAD1.png) |
| Active Directory とフィルター処理されたインポート |![AD](./media/active-directory-aadconnect-topologies/LegendAD2.png) |
| Azure AD Connect 同期サーバー |![同期](./media/active-directory-aadconnect-topologies/LegendSync1.png) |
| Azure AD Connect 同期サーバー "ステージング モード" |![同期](./media/active-directory-aadconnect-topologies/LegendSync2.png) |
| GALSync と FIM2010 または MIM2016 |![同期](./media/active-directory-aadconnect-topologies/LegendSync3.png) |
| Azure AD Connect 同期サーバー、詳細 |![同期](./media/active-directory-aadconnect-topologies/LegendSync4.png) |
| Azure AD ディレクトリ |![AAD](./media/active-directory-aadconnect-topologies/LegendAAD.png) |
| サポートされていないシナリオ |![サポートされていません](./media/active-directory-aadconnect-topologies/LegendUnsupported.png) |

## <a name="single-forest-single-azure-ad-tenant"></a>単一のフォレスト、単一の Azure AD テナント
![Single Forest Single Tenant](./media/active-directory-aadconnect-topologies/SingleForestSingleDirectory.png)

最も一般的なトポロジは、オンプレミスの 1 つのフォレスト (1 つまたは複数のドメイン) と、1 つの Azure AD テナントです。 Azure AD 認証では、パスワード同期が使用されます。 Azure AD Connect の高速インストールでは、このトポロジのみがサポートされます。

### <a name="single-forest-multiple-sync-servers-to-one-azure-ad-tenant"></a>単一のフォレスト、1 つの Azure AD テナントに対する複数の同期サーバー
![Single Forest Filtered Unsupported](./media/active-directory-aadconnect-topologies/SingleForestFilteredUnsupported.png)

同じ Azure AD テナントへの複数の Azure AD Connect 同期サーバーの接続はサポートされません ([ステージング サーバー](#staging-server)は除きます)。 これは、同期サーバーが相互に排他的な一連のオブジェクトを同期するように構成されている場合でもサポートされません。 1 台のサーバーからフォレスト内のすべてのドメインに到達できない場合や複数のサーバー間で負荷を分散する場合は、この点について考慮する必要があります。

## <a name="multiple-forests-single-azure-ad-tenant"></a>複数のフォレスト、単一の Azure AD テナント
![Multi Forest Single Tenant](./media/active-directory-aadconnect-topologies/MultiForestSingleDirectory.png)

多くの組織の環境には、オンプレミス Active Directory フォレストが複数存在します。 複数のオンプレミス Active Directory フォレストが使用される理由はさまざまです。 一般的な例として、アカウント リソース フォレストを持つ設計や、合併や買収の結果としての状況があります。

複数のフォレストがある場合は、1 つの Azure AD Connect 同期サーバーがすべてのフォレストにアクセスできる必要があります。 サーバーをドメインに参加させる必要はありません。 すべてのフォレストにアクセスできるようにするために、必要に応じてサーバーをネットワーク DMZ に配置できます。

Azure AD Connect のインストール ウィザードには、複数のフォレストで表されるユーザーを統合するためのいくつかのオプションが用意されています。 その目的は、ユーザーが Azure AD 内で 1 回だけ表されるようにすることです。 インストール ウィザードのカスタム インストール パスで構成できる一般的なトポロジはいくつかあります。 **[ユーザーを一意に識別]**ページで、使用しているトポロジに対応するオプションを選択します。 統合は、ユーザーに対してのみ構成されます。 重複しているグループは既定の構成に統合されません。

一般的なトポロジである[分離トポロジ](#multiple-forests-separate-topologies)、[フル メッシュ](#multiple-forests-full-mesh-with-optional-galsync)、および[アカウント リソース](#multiple-forests-account-resource-forest)については、次のセクションで説明します。

Azure AD Connect 同期の既定の構成では、次のことを前提としています。

1. ユーザーが持つ有効なアカウントは 1 つのみで、このアカウントが配置されているフォレストがユーザーの認証に使用されます。 この前提は、パスワード同期とフェデレーションの両方に該当します。 UserPrincipalName と sourceAnchor/immutableID は、このフォレストから取得されます。
2. ユーザーのメールボックスは 1 つのみです。
3. ユーザーのメールボックスをホストするフォレストは、Exchange のグローバル アドレス一覧 (GAL) で確認できる属性に対して最適なデータ品質を備えています。 ユーザーにメールボックスがない場合、どのフォレストを使用してもこれらの属性値を提供できます。
4. リンクされたメールボックスがある場合は、サインインに使用される別のアカウントも別のフォレストにあります。

環境がこれらの前提と一致しない場合は、次のようになります。

* アクティブなアカウントまたはメールボックスが複数ある場合、同期エンジンは 1 つを選び、他は無視します。
* 他のアクティブなアカウントを持たないリンクされたメールボックスは、Azure AD にはエクスポートされません。 ユーザー アカウントは、どのグループのメンバーとしても表されません。 DirSync のリンクされたメールボックスは、常に通常のメールボックスとして表されます。 この変更は、マルチフォレスト シナリオをよりよくサポートするための意図的に異なる動作です。

詳細については、[既定の構成](active-directory-aadconnectsync-understanding-default-configuration.md)に関するページを参照してください。

### <a name="multiple-forests-multiple-sync-servers-to-one-azure-ad-tenant"></a>複数のフォレスト、1 つの Azure AD テナントに対する複数の同期サーバー
![Multi Forest Multi Sync Unsupported](./media/active-directory-aadconnect-topologies/MultiForestMultiSyncUnsupported.png)

1 つの Azure AD テナントへの複数の Azure AD Connect 同期サーバーの接続はサポートされません。 例外として、 [ステージング サーバー](#staging-server)の使用があります。

### <a name="multiple-forests--separate-topologies"></a>複数のフォレスト – 分離トポロジ
**[ユーザーは、すべてのディレクトリで 1 度だけ示されます。]**

![Multi Forest Users Once](./media/active-directory-aadconnect-topologies/MultiForestUsersOnce.png)

![Multi Forest Seperate Topologies](./media/active-directory-aadconnect-topologies/MultiForestSeperateTopologies.png)

この環境では、すべてのオンプレミス フォレストは個別のエンティティと見なされ、その他のフォレストにユーザーは存在しません。
各フォレストには独自の Exchange 組織があり、フォレスト間に GALSync はありません。 このトポロジは、合併や買収の後の組織や各部署が互いに独立して運用されている組織に見受けられます。 Azure AD ではこれらのフォレストは同じ組織内にあり、統合された GAL で表示されます。
この図では、すべてのフォレスト内の各オブジェクトが、一度メタバースに表され、ターゲットの Azure AD テナントに集約されます。

### <a name="multiple-forests--match-users"></a>複数のフォレスト - ユーザーの一致
**[複数のディレクトリにユーザー ID が存在します。]**

これらのすべてのシナリオで共通することとして、配布グループとセキュリティ グループでは、ユーザー、連絡先、および FSP (外部セキュリティ プリンシパル) を組み合わせて含めることができます。

FSP は、セキュリティ グループ内の他のフォレストのメンバーを表すために、ADDS で使用されます。 すべての FSP は、Azure AD 内の実際のオブジェクトに解決されます。

### <a name="multiple-forests--full-mesh-with-optional-galsync"></a>複数のフォレスト - オプションの GALSync を使用したフル メッシュ
**[複数のディレクトリにユーザー ID が存在します。次を使用した一致: メール属性]**

![Multi Forest Users Mail](./media/active-directory-aadconnect-topologies/MultiForestUsersMail.png)

![Multi Forest Full Mesh](./media/active-directory-aadconnect-topologies/MultiForestFullMesh.png)

フル メッシュ トポロジでは、ユーザーとリソースを任意のフォレストに置くことができ、一般的にフォレスト間には双方向の信頼があります。

複数のフォレストに Exchange が存在する場合、必要に応じてオンプレミス GALSync ソリューションが存在する可能性があります。 その場合、すべてのユーザーが他のすべてのフォレストにおける連絡先として表されます。 GALSync は、通常、Forefront Identity Manager 2010 または Microsoft Identity Manager 2016 を使用して実装されます。 オンプレミス GALSync では、Azure AD Connect は使用できません。

このシナリオでは、ID オブジェクトはメール属性を使用して結合されます。 あるフォレストでメールボックスを使用するユーザーは、他のフォレストの連絡先と結合されます。

### <a name="multiple-forests--account-resource-forest"></a>複数のフォレスト – アカウント リソース フォレスト
**[複数のディレクトリにユーザー ID が存在します。次を使用した一致: ObjectSID および msExchMasterAccountSID 属性]**

![Multi Forest Users ObjectSID](./media/active-directory-aadconnect-topologies/MultiForestUsersObjectSID.png)

![Multi Forest AccountResource](./media/active-directory-aadconnect-topologies/MultiForestAccountResource.png)

アカウント リソース フォレスト トポロジでは、アクティブなユーザー アカウントを使用する 1 つ以上のアカウント フォレストが存在します。 また、アカウントが無効になった 1 つ以上のリソース フォレストが存在します。

このシナリオでは、1 つ (以上) の**リソース フォレスト**がすべての**アカウント フォレスト**を信頼します。 このリソース フォレストには、通常、Exchange および Lync を使用する拡張 AD スキーマがあります。 すべての Exchange および Lync サービスと、他の共有サービスは、このフォレストに配置されます。 ユーザーのユーザー アカウントはこのフォレストで無効になり、メールボックスはアカウント フォレストにリンクされます。

## <a name="office-365-and-topology-considerations"></a>Office 365 とトポロジの考慮事項
Office 365 の一部のワークロードでは、サポートされるトポロジに一定の制限が生じます。 これらのいずれかを使用する場合は、対象のワークロードのサポートされているトポロジに関するトピックを参照してください。

| ワークロード |
| --- | --- |
| Exchange Online |
| Skype for Business |

## <a name="staging-server"></a>ステージング サーバー
![ステージング サーバー](./media/active-directory-aadconnect-topologies/MultiForestStaging.png)

Azure AD Connect では、**ステージング モード**でのセカンド サーバーのインストールがサポートされています。 このモードのサーバーは、接続されたすべてのディレクトリからデータを読み取りますが、接続されたディレクトリへの書き込みは行いません。 このサーバーは、通常の同期サイクルを使用しているため、ID データの更新されたコピーを保持します。 プライマリ サーバーで障害が発生した場合は、ステージング サーバーにフェールオーバーできます。 この操作は、Azure AD Connect ウィザードで実行します。 このセカンド サーバーは、別のデータ センターに配置することをお勧めします。そうすれば、インフラストラクチャをプライマリ サーバーと共有することがありません。 プライマリ サーバーで行われたすべての構成の変更をセカンド サーバーに手動でコピーする必要があります。

ステージング サーバーは、新しいカスタム構成と、それがデータに与える影響をテストする場合にも使用できます。 変化をプレビューし、構成を調整できます。 新しい構成に問題がなければ、ステージング サーバーをアクティブ サーバーにし、元のアクティブ サーバーをステージング モードに設定できます。

この方法は、アクティブな同期サーバーを交換する場合にも使用できます。 新しいサーバーを準備し、ステージング モードに設定してください。 サーバーが良好な状態であることを確認し、ステージング モードを無効 (アクティブ) にしたら、現在アクティブなサーバーをシャットダウンします。

異なるデータ センターに複数のバックアップを用意する場合は、複数のステージング サーバーを持つことができます。

## <a name="multiple-azure-ad-tenants"></a>複数の Azure AD テナント
組織の Azure AD には 1 つのテナントを置くことをお勧めします。
複数の Azure AD テナントの使用を計画する前に、これらのトピックでは 1 つのテナントを使用できる一般的なシナリオについて説明しています。

| トピック |
| --- | --- |
| 管理単位を使用した委任 |

![Multi Forest Multi tenant](./media/active-directory-aadconnect-topologies/MultiForestMultiDirectory.png)

Azure AD Connect 同期サーバーと Azure AD テナントには、1:1 のリレーションシップがあります。 各 Azure AD テナントに、1 つの Azure AD Connect 同期サーバーをインストールする必要があります。 各 Azure AD テナント インスタンスは分離されるデザインになっており、1 つのテナントのユーザーは他のテナント内のユーザーを見ることができません。 この分離が意図しているものであれば、これはサポートされている構成ですが、そうでない場合は単一 Azure AD テナント モデルを使用する必要があります。

### <a name="each-object-only-once-in-an-azure-ad-tenant"></a>Azure AD テナントでの各オブジェクトの 1 回のみの使用
![Single Forest Filtered](./media/active-directory-aadconnect-topologies/SingleForestFiltered.png)

このトポロジでは、1 つの Azure AD Connect 同期サーバーが各 Azure AD テナントに接続されます。 各 Azure AD Connect 同期サーバーは、操作対象のオブジェクトのセットが相互排他的になるようなフィルター処理を構成する必要があります。 たとえば、各サーバーのスコープを特定のドメインまたは OU に設定できます。 DNS ドメインは 1 つの Azure AD テナントにのみ登録できます。 オンプレミス AD のユーザーの UPN でも、別の名前空間を使用する必要があります。 たとえば、上の図では、3 つの個別の UPN サフィックスがオンプレミス AD の contoso.com、fabrikam.com、および wingtiptoys.com に登録されています。 各オンプレミス AD ドメインのユーザーは、別の名前空間を使用します。

Azure AD テナント インスタンス間に GALsync はありません。 Exchange Online と Skype for Business のアドレス帳には、同じテナント内のユーザーのみが表示されます。

このトポロジには次の制約があります。その制約を除けば、各種シナリオがサポートされます。

* 1 つの Azure AD テナントのみが、オンプレミスの Active Directory を持つ Exchange ハイブリッドを有効にできます。
* Windows 10 デバイスを関連付けることができる Azure AD テナントは 1 つのみです。

オブジェクトの相互排他的なセットの要件は、書き戻しにも適用されます。 書き戻し機能の中には単一構成のオンプレミスを前提としているものがあります。そのような機能では、このトポロジではサポートされません。

* 既定の構成によるグループの書き戻し
* デバイスの書き戻し

### <a name="each-object-multiple-times-in-an-azure-ad-tenant"></a>Azure AD テナントでの各オブジェクトの複数回の使用
![Single Forest Multi Tenant Unsupported](./media/active-directory-aadconnect-topologies/SingleForestMultiDirectoryUnsupported.png) ![Single Forest Multi Connectors Unsupported](./media/active-directory-aadconnect-topologies/SingleForestMultiConnectorsUnsupported.png)

* 複数の Azure AD テナントへの同じユーザーの同期はサポートされていません。
* 1 つの Azure AD のユーザーを他の Azure AD テナントで連絡先として表示するような構成の変更はサポートされていません。
* 複数の Azure AD テナントに接続するような Azure AD Connect 同期の変更はサポートされていません。

### <a name="galsync-by-using-writeback"></a>書き戻しの使用による GALsync
![MultiForestMultiDirectoryGALSync1Unsupported](./media/active-directory-aadconnect-topologies/MultiForestMultiDirectoryGALSync1Unsupported.png) ![MultiForestMultiDirectoryGALSync2Unsupported](./media/active-directory-aadconnect-topologies/MultiForestMultiDirectoryGALSync2Unsupported.png)

Azure AD のテナントは、分離するように設計されています。

* 他の Azure AD テナントからデータを読み取るような Azure AD Connect 同期の構成の変更はサポートされていません。
* Azure AD Connect 同期を使用している別のオンプレミス AD の連絡先としてユーザーをエクスポートすることはサポートされていません。

### <a name="galsync-with-on-premises-sync-server"></a>GALsync とオンプレミスの同期サーバー
![MultiForestMultiDirectoryGALSync](./media/active-directory-aadconnect-topologies/MultiForestMultiDirectoryGALSync.png)

2 つの Exchange 組織間でユーザーを GALsync 処理するための、オンプレミスでの FIM2010/MIM2016 の使用はサポートされています。 1 つの組織内のユーザーは、他の組織では外部ユーザーおよび連絡先として表示されます。 これらの異なるオンプレミス AD は、独自の Azure AD テナントと同期できます。

## <a name="next-steps"></a>次のステップ
これらのシナリオのために Azure AD Connect をインストールする方法については、「 [Azure AD Connect のカスタム インストール](active-directory-aadconnect-get-started-custom.md)」を参照してください。

[Azure AD Connect Sync](active-directory-aadconnectsync-whatis.md) の構成に関するページをご覧ください。

「 [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)」をご覧ください。




<!--HONumber=Dec16_HO3-->


