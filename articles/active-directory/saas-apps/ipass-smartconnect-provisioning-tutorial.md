---
title: チュートリアル:iPass SmartConnect を構成し、Azure Active Directory を使用した自動ユーザー プロビジョニングに対応させる | Microsoft Docs
description: Azure Active Directory を構成して、ユーザー アカウントを iPass SmartConnect に自動的にプロビジョニング/プロビジョニング解除する方法を説明します。
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: 43d6de4a-b3a2-439b-95f2-8883fffded52
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/26/2019
ms.author: zhchia
ms.openlocfilehash: fb3a5d03f390b88f9856f03bdc97a35b845874ff
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2020
ms.locfileid: "77057529"
---
# <a name="tutorial-configure-ipass-smartconnect-for-automatic-user-provisioning"></a>チュートリアル:iPass SmartConnect を構成し、自動ユーザー プロビジョニングに対応させる

このチュートリアルの目的は、Azure AD が自動的にユーザーまたはグループを iPass SmartConnect にプロビジョニングまたはプロビジョニング解除するように構成するために、iPass SmartConnect と Azure Active Directory (Azure AD) で実行される手順を示すことです。

> [!NOTE]
> このチュートリアルでは、Azure AD ユーザー プロビジョニング サービスの上にビルドされるコネクタについて説明します。 このサービスが実行する内容、しくみ、よく寄せられる質問の重要な詳細については、「[Azure Active Directory による SaaS アプリへのユーザー プロビジョニングとプロビジョニング解除の自動化](../app-provisioning/user-provisioning.md)」を参照してください。
>
> 現在、このコネクタはパブリック プレビュー段階にあります。 プレビュー機能を使用するための一般的な Microsoft Azure 使用条件の詳細については、「[Microsoft Azure プレビューの追加使用条件](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)」を参照してください。

## <a name="prerequisites"></a>前提条件

このチュートリアルで説明するシナリオでは、次の前提条件目があることを前提としています。

* Azure AD テナント。
* [iPass SmartConnect テナント](https://www.ipass.com/buy-ipass/)。
* Admin アクセス許可がある iPass SmartConnect のユーザー アカウント。

## <a name="assigning-users-to-ipass-smartconnect"></a>ユーザーを iPass SmartConnect に割り当てる

Azure Active Directory では、選択されたアプリへのアクセスが付与されるユーザーを決定する際に "*割り当て*" という概念が使用されます。 自動ユーザー プロビジョニングのコンテキストでは、Azure AD 内のアプリケーションに割り当て済みのユーザーとグループのみが同期されます。

自動ユーザー プロビジョニングを構成して有効にする前に、iPass SmartConnect へのアクセスが必要な Azure AD のユーザーやグループを決定しておく必要があります。 決定し終えたら、次の手順に従って、これらのユーザーやグループを iPass SmartConnect に割り当てることができます。
* [エンタープライズ アプリケーションにユーザーまたはグループを割り当てる](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-ipass-smartconnect"></a>ユーザーを iPass SmartConnect に割り当てる際の重要なヒント

* 単一の Azure AD ユーザーを iPass SmartConnect に割り当てて、自動ユーザー プロビジョニングの構成をテストすることをお勧めします。 後でユーザーやグループを追加で割り当てられます。

* iPass SmartConnect にユーザーを割り当てるときは、有効なアプリケーション固有ロール (使用可能な場合) を割り当てダイアログで選択する必要があります。 **既定のアクセス** ロールのユーザーは、プロビジョニングから除外されます。

## <a name="setup-ipass-smartconnect-for-provisioning"></a>iPass SmartConnect をプロビジョニングに対応するように構成する

Azure AD での自動ユーザー プロビジョニング用に iPass SmartConnect を構成する前に、iPass SmartConnect 管理コンソールから構成情報を取得する必要があります。

1. iPass SmartConnect の SCIM エンドポイントに対して認証を行うために必要なベアラー トークンを取得するには、iPass SmartConnect を初めて設定するときに参照してください。この値はその時にしか提供されないためです。 
2. ベアラー トークンがない場合は、[iPass SmartConnect のサポートチーム](mailto:help@ipass.com)に問い合わせて新しいトークンを取得します。

## <a name="add-ipass-smartconnect-from-the-gallery"></a>ギャラリーから iPass SmartConnect を追加する

Azure AD で自動ユーザー プロビジョニング用に iPass SmartConnect を構成するには、Azure AD アプリケーション ギャラリーから iPass SmartConnect をマネージド SaaS アプリケーションの一覧に追加する必要があります。

**Azure AD アプリケーション ギャラリーから iPass SmartConnect を追加するには、次の手順を行います。**

1. **[Azure portal](https://portal.azure.com)** の左側のナビゲーション パネルで、 **[Azure Active Directory]** を選択します。

    ![Azure Active Directory のボタン](common/select-azuread.png)

2. **[エンタープライズ アプリケーション]** に移動し、 **[すべてのアプリケーション]** を選択します。

    ![[エンタープライズ アプリケーション] ブレード](common/enterprise-applications.png)

3. 新しいアプリケーションを追加するには、ウィンドウの上部にある **[新しいアプリケーション]** ボタンを選びます。

    ![[新しいアプリケーション] ボタン](common/add-new-app.png)

4. 検索ボックスに「**iPass SmartConnect**」と入力し、結果パネルで **[iPass SmartConnect]** を選択し、 **[追加]** ボタンをクリックして、アプリケーションを追加します。

    ![結果一覧の iPass SmartConnect](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-ipass-smartconnect"></a>iPass SmartConnect への自動ユーザー プロビジョニングを構成する 

このセクションでは、Azure AD プロビジョニング サービスを構成し、Azure AD でのユーザーやグループの割り当てに基づいて iPass SmartConnect のユーザーやグループを作成、更新、無効化する手順について説明します。

> [!TIP]
>  iPass SmartConnect では SAML ベースのシングル サインオンを有効にすることもできます。これを行うには、[BitaBIZ シングル サインオンのチュートリアル](ipasssmartconnect-tutorial.md)に関するページで説明されている手順に従ってください。 シングル サインオンは自動ユーザー プロビジョニングとは別に構成できますが、これらの 2 つの機能は相補的な関係にあります。

### <a name="to-configure-automatic-user-provisioning-for-ipass-smartconnect-in-azure-ad"></a>Azure AD で iPass SmartConnect の自動ユーザー プロビジョニングを構成するには:

1. [Azure portal](https://portal.azure.com) にサインインします。 **[エンタープライズ アプリケーション]** を選択し、 **[すべてのアプリケーション]** を選択します。

    ![[エンタープライズ アプリケーション] ブレード](common/enterprise-applications.png)

2. アプリケーションの一覧で **[iPass SmartConnect]** を選択します。

    ![[アプリケーション] 一覧の iPass SmartConnect リンク](common/all-applications.png)

3. **[プロビジョニング]** タブを選択します。

    ![[プロビジョニング] タブ](common/provisioning.png)

4. **[プロビジョニング モード]** を **[自動]** に設定します。

    ![[プロビジョニング] タブ](common/provisioning-automatic.png)

5. **[管理者資格情報]** セクションの **[テナントの URL]** に「`https://openmobile.ipass.com/moservices/scim/v1`」と入力します。 以前に取得したベアラー トークンを **[シークレット トークン]** に入力します。 **[テスト接続]** をクリックして、Azure AD から iPass SmartConnect に接続できることを確認します。 接続できない場合は、使用中の iPass SmartConnect アカウントに管理者アクセス許可があることを確認してから、もう一度試します。

    ![テナント URL + トークン](common/provisioning-testconnection-tenanturltoken.png)

6. **[通知用メール]** フィールドに、プロビジョニングのエラー通知を受け取るユーザーまたはグループの電子メール アドレスを入力して、 **[エラーが発生したときにメール通知を送信します]** チェック ボックスをオンにします。

    ![通知用メール](common/provisioning-notification-email.png)

7. **[保存]** をクリックします。

8. **[マッピング]** セクションの **[Synchronize Azure Active Directory Users to iPass SmartConnect]\(Azure Active Directory ユーザーを iPass SmartConnect に同期する\)** を選択します。

    ![iPass SmartConnect ユーザーのマッピング](media/ipass-smartconnect-provisioning-tutorial/usermapping.png)

9. **[属性マッピング]** セクションで、Azure AD から iPass SmartConnect に同期されるユーザー属性を確認します。 **[Matching]\(照合\)** プロパティとして選択されている属性は、更新処理で iPass SmartConnect のユーザー アカウントとの照合に使用されます。 **[保存]** ボタンをクリックして変更をコミットします。

    ![iPass SmartConnect ユーザーのマッピング](media/ipass-smartconnect-provisioning-tutorial/userattribute.png)


10. スコープ フィルターを構成するには、[スコープ フィルターのチュートリアル](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)の次の手順を参照してください。

11. iPass SmartConnect に対して Azure AD プロビジョニング サービスを有効にするには、 **[設定]** セクションで **[プロビジョニングの状態]** を **[オン]** に変更します。

    ![プロビジョニングの状態を [オン] に切り替える](common/provisioning-toggle-on.png)

12. **[設定]** セクションの **[スコープ]** で目的の値を選択して、iPass SmartConnect にプロビジョニングするユーザーやグループを定義します。

    ![プロビジョニングのスコープ](common/provisioning-scope.png)

13. プロビジョニングの準備ができたら、 **[保存]** をクリックします。

    ![プロビジョニング構成の保存](common/provisioning-configuration-save.png)

これにより、 **[設定]** セクションの **[スコープ]** で 定義したユーザーやグループの初期同期が開始されます。 初期同期は後続の同期よりも実行に時間がかかります。後続の同期は、Azure AD のプロビジョニング サービスが実行されている限り約 40 分ごとに実行されます。 **[同期の詳細]** セクションを使用すると、進行状況を監視できるほか、リンクをクリックしてプロビジョニング アクティビティ レポートを取得できます。このレポートには、Azure AD プロビジョニング サービスによって iPass SmartConnect に対して実行されたすべてのアクションが記載されています。

Azure AD プロビジョニング ログの読み取りの詳細については、「[自動ユーザー アカウント プロビジョニングについてのレポート](../app-provisioning/check-status-user-account-provisioning.md)」をご覧ください。

## <a name="connector-limitations"></a>コネクタの制限事項

* iPass SmartConnect では、ドメインが iPass SmartConnect 管理コンソールに登録されているユーザー名のみ受け入れられます。  

## <a name="additional-resources"></a>その他のリソース

* [エンタープライズ アプリのユーザー アカウント プロビジョニングの管理](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>次のステップ

* [プロビジョニング アクティビティのログの確認方法およびレポートの取得方法](../app-provisioning/check-status-user-account-provisioning.md)
