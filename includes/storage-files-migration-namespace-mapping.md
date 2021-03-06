---
title: フォルダー構造を Azure File Sync トポロジにマップする
description: Azure File Sync で使用するために既存のファイルおよびフォルダー構造を Azure ファイル共有にマップします。移行ドキュメント間で共有される一般的なテキスト ブロック。
author: fauhse
ms.service: storage
ms.topic: conceptual
ms.date: 2/20/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 948090d0ee956ca1798d7b0f46bb33276c4d6354
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/28/2020
ms.locfileid: "82143563"
---
この手順では、必要な Azure ファイル共有の数を評価します。 1 つの Windows Server インスタンス (またはクラスター) では、最大 30 個の Azure ファイル共有を同期できます。

現在、SMB 共有としてユーザーとアプリに対してローカルに共有しているボリュームに、さらに多くのフォルダーを持つことができます。 最も簡単な方法は、1:1 で Azure ファイル共有にマップするオンプレミスの共有を構想することです。 1 つの Windows Server インスタンスの個数が少数 (30 個未満) の場合は、1:1 のマッピングをお勧めします。

30 個を超える共有がある場合、通常、オンプレミスの共有を 1 対 1 で Azure ファイル共有にマッピングする必要はありません。 次のオプションを検討してください。

#### <a name="share-grouping"></a>共有のグループ化

(たとえば) 人事部に合計 15 個の共有がある場合、すべての人事部データを 1 つの Azure ファイル共有に格納することを検討してください。 1 つの Azure ファイル共有にオンプレミスの共有を複数格納しても、ローカルの Windows Server インスタンスに通常の 15 個の SMB 共有を作成できます。 これは、単にこれらの 15 個の共有のルート フォルダーを共通フォルダーの下のサブフォルダーとして整理することを意味します。 その後、この共通フォルダーを Azure ファイル共有に同期します。 そうすることで、このオンプレミスの共有グループに必要なのは、クラウド内の 1 つの Azure ファイル共有のみとなります。

#### <a name="volume-sync"></a>ボリュームの同期

Azure File Sync では、ボリュームのルートを Azure ファイル共有に同期することをサポートしています。 ルート フォルダーを同期した場合、すべてのサブフォルダーとファイルは同じ Azure ファイル共有に格納されます。

ボリュームのルートを同期することが常に最適な答えとは限りません。 複数の場所を同期することには利点があります。 たとえば、そうすることで、同期スコープあたりの項目数を少なく抑えることができます。 より少数の項目で Azure File Sync を設定することは、ファイルの同期にとって有益というだけではありません。項目の数が少ないと、次のようなシナリオでも利点があります。

* バックアップとして取得された Azure ファイル共有スナップショットから、クラウド側の復元を行うことができます。
* オンプレミスサーバーのディザスター リカバリーが大幅にスピードアップされます。
* Azure ファイル共有 (同期以外) で直接行われた変更を検出して、より迅速に同期できます。

#### <a name="a-structured-approach-to-a-deployment-map"></a>デプロイ マップへの構造化アプローチ

後の手順でクラウド ストレージをデプロイする前に、オンプレミス フォルダーと Azure ファイル共有の間にマップを作成することが重要です。 このマッピングを行うと、その後、プロビジョニングする Azure File Sync の "*同期グループ*" リソースの数と名称が通知されます。 同期グループは、Azure ファイル共有とサーバー上のフォルダーを連携させ、同期接続を確立します。

必要な Azure ファイル共有の数を決定するにあたっては、次の制限事項とベストプラクティスをレビューしてください。 そのことが、マップの最適化に役立ちます。

* Azure File Sync エージェントがインストールされているサーバーは、最大 30 個の Azure ファイル共有と同期できます。
* Azure ファイル共有は、ストレージ アカウント内にデプロイされます。 これにより、このストレージ アカウントが、IOPS やスループットなどのパフォーマンス数のスケールターゲットになります。 

  理論的には、2 つの標準 (プレミアムではない) Azure ファイル共有により、ストレージ アカウントが提供できる最大パフォーマンスが飽和状態になる可能性があります。 これらのファイル共有への Azure File Sync のみのアタッチを計画している場合には、複数の Azure ファイル共有を同じストレージ アカウントにグループ化しても問題は発生しません。 考慮すべき関連メトリックの詳細については、「Azure ファイル共有のパフォーマンス ターゲット」を参照してください。 

  Azure ファイル共有をネイティブで使用するアプリを Azure にリフトする場合は、Azure ファイル共有のパフォーマンスをさらに上げる必要があります。 現在または将来的にその可能がある場合には、Azure ファイル共有をそれ独自のストレージ アカウントにマッピングすることを推奨します。
* 1 つの Azure リージョンにつき、1サブスクリプションあたりのストレージ アカウント数は 250 に制限されています。

> [!TIP]
> この情報を考慮して、ボリューム上の複数のトップレベルフォルダーを共通の新しいルート ディレクトリにグループ化することがしばしば必要になります。 次に、この新しいルート ディレクトリと、その中にグループ化したすべてのフォルダーを、1 つの Azure ファイル共有に同期します。 この手法を使用すると、サーバーあたり 30 個の Azure ファイル共有同期の制限内に抑えることができます。
>
> 共通のルートの下でのこのグループ化は、データへのアクセスにはいっさい影響しません。 ACL はそのままです。 今ここで共通のルートに変更したサーバー フォルダー上に必要共有パス (SMB 共有や NFS 共有など) がある場合に限り、その調整が必要となります。 それ以外の変更はありません。

Azure File Sync と、そのバランスの取れたパフォーマンスおよびエクスペリエンスを維持する上でもう 1 つ重要なことは、Azure File Sync パフォーマンスのスケール ファクターを理解することです。 当然ながら、ファイルがインターネット経由で同期されている場合、ファイルが大きくなるほど、同期に要する時間と帯域幅も大きくなります。

> [!IMPORTANT]
> Azure File Sync の最も重要なスケール ベクターは、同期が必要な項目 (ファイルとフォルダー) の数です。

Azure File Sync は、1つの Azure ファイル共有に対して最大 100,000 項目の同期をサポートしています。 ただし、Azure File Sync チームが定期的に行うテスト内容の表示に限っては、この制限を超えることができます。

ここでのベストプラクティスは、同期スコープあたりの項目数を少なくしておくことです。 これは、フォルダーを Azure ファイル共有にマッピングする際に考慮する必要がある重要な要素です。

状況によっては、一連のフォルダーが同じ Azure ファイル共有に論理的に同期される可能性があります (前述の新しい共通のルート フォルダーのアプローチを使用します)。 ただし、1 つの Azure ファイル共有ではなく 2 つに同期されるように、フォルダーを再グループ化することをお勧めします。 このアプローチを使用すると、ファイル共有あたりのファイルとフォルダーの数をサーバー間に分散させることができます。

#### <a name="create-a-mapping-table"></a>マッピング テーブルを作成する

:::row:::
    :::column:::
        [![](media/storage-files-migration-namespace-mapping/namespace-mapping.png "An example of a mapping table. Download the file below to experience and use the content of this image.")](media/storage-files-migration-namespace-mapping/namespace-mapping-expanded.png#lightbox)
    :::column-end:::
    :::column:::
        上記のコンセプトを組み合わせて使用すると、必要とする Azure ファイル共有の数を決定し、既存のデータのどの部分がどの Azure ファイル共有に格納されるかを判断する際の役に立ちます。
        
        次のステップでの参照用に、あなたの考えを記録したテーブルを作成します。 一度に多数の Azure リソースをプロビジョニングする際には、マッピング計画の詳細がおろそかになりがちです。ですから、情報が整理された状態を維持することが重要です。 Microsoft Excel ファイルをテンプレートとしてダウンロードし、完全なマッピング作成のための支援ツールに使うこともできます。

[//]: # (HTML 表示は、入れ子になった2列のテーブルを追加するための唯一の方法です。ここでは、作業イメージの解析とテキスト/ハイパーリンクが同じ行にあります。)

<br>
<table>
    <tr>
        <td>
            <img src="media/storage-files-migration-namespace-mapping/excel.png" alt="Microsoft Excel file icon that helps to set the context for the type of file download for the link next to it.">
        </td>
        <td>
            <a href="https://download.microsoft.com/download/1/8/D/18DC8184-E7E2-45EF-823F-F8A36B9FF240/Azure File Sync - Namespace Mapping.xlsx">名前空間マッピング テンプレートをダウンロードします。</a>
        </td>
    </tr>
</table>
    :::column-end:::
:::row-end:::
