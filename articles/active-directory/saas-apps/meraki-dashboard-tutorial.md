---
title: チュートリアル:Azure Active Directory シングル サインオン (SSO) と Meraki Dashboard の統合 | Microsoft Docs
description: Azure Active Directory と Meraki Dashboard の間でシングル サインオンを構成する方法について説明します。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 09766236-5de0-43fd-8950-5316390ce646
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6d5fd75a737a0a866b6e5c26c417458ee95845fb
ms.sourcegitcommit: 09a124d851fbbab7bc0b14efd6ef4e0275c7ee88
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/23/2020
ms.locfileid: "82084160"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-meraki-dashboard"></a>チュートリアル:Azure Active Directory シングル サインオン (SSO) と Meraki Dashboard の統合

このチュートリアルでは、Meraki Dashboard と Azure Active Directory (Azure AD) を統合する方法について説明します。 Azure AD と Meraki Dashboard を統合すると、次のことができます。

* Meraki Dashboard にアクセスできるユーザーを Azure AD で制御できます。
* ユーザーが自分の Azure AD アカウントを使用して Meraki Dashboard に自動的にサインインできるように設定できます。
* 1 つの中央サイト (Azure Portal) で自分のアカウントを管理します。

SaaS アプリと Azure AD の統合の詳細については、「[Azure Active Directory でのアプリケーションへのシングル サインオン](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)」を参照してください。

## <a name="prerequisites"></a>前提条件

開始するには、次が必要です。

* Azure AD サブスクリプション。 サブスクリプションがない場合は、[無料アカウント](https://azure.microsoft.com/free/)を取得できます。
* Meraki Dashboard でのシングル サインオン (SSO) が有効なサブスクリプション。

## <a name="scenario-description"></a>シナリオの説明

このチュートリアルでは、テスト環境で Azure AD の SSO を構成してテストします。

* Meraki Dashboard では、**IDP** Initiated SSO がサポートされます
* Meraki Dashboard を構成したら、組織の機密データを流出と侵入からリアルタイムで保護するセッション制御を適用することができます。 セッション制御は、条件付きアクセスを拡張したものです。 [Microsoft Cloud App Security でセッション制御を強制する方法](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app)をご覧ください。

## <a name="adding-meraki-dashboard-from-the-gallery"></a>ギャラリーからの Meraki Dashboard の追加

Azure AD への Meraki Dashboard の統合を構成するには、ギャラリーからマネージド SaaS アプリの一覧に Meraki Dashboard を追加する必要があります。

1. 職場または学校アカウントか、個人の Microsoft アカウントを使用して、[Azure portal](https://portal.azure.com) にサインインします。
1. 左のナビゲーション ウィンドウで **[Azure Active Directory]** サービスを選択します。
1. **[エンタープライズ アプリケーション]** に移動し、 **[すべてのアプリケーション]** を選択します。
1. 新しいアプリケーションを追加するには、 **[新しいアプリケーション]** を選択します。
1. **[ギャラリーから追加する]** セクションで、検索ボックスに「**Meraki Dashboard**」と入力します。
1. 結果のパネルから **[Meraki Dashboard]** を選択し、アプリを追加します。 お使いのテナントにアプリが追加されるのを数秒待機します。

## <a name="configure-and-test-azure-ad-single-sign-on-for-meraki-dashboard"></a>Meraki Dashboard の Azure AD シングル サインオンの構成とテスト

**B.Simon** というテスト ユーザーを使用して、Meraki Dashboard に対する Azure AD SSO を構成してテストします。 SSO が機能するためには、Azure AD ユーザーと Meraki Dashboard の関連ユーザーとの間にリンク関係を確立する必要があります。

Meraki Dashboard に対する Azure AD SSO を構成してテストするには、次の構成要素を完了します。

1. **[Azure AD SSO の構成](#configure-azure-ad-sso)** - ユーザーがこの機能を使用できるようにします。
    1. **[Azure AD のテスト ユーザーの作成](#create-an-azure-ad-test-user)** - B.Simon で Azure AD のシングル サインオンをテストします。
    1. **[Azure AD テスト ユーザーの割り当て](#assign-the-azure-ad-test-user)** - B.Simon が Azure AD シングル サインオンを使用できるようにします。
1. **[Meraki Dashboard SSO の構成](#configure-meraki-dashboard-sso)** - アプリケーション側でシングル サインオン設定を構成します。
    1. **[Meraki Dashboard のテスト ユーザーの作成](#create-meraki-dashboard-test-user)** - Meraki Dashboard で B.Simon に対応するユーザーを作成し、Azure AD の B.Simon にリンクさせます。
1. **[SSO のテスト](#test-sso)** - 構成が機能するかどうかを確認します。

## <a name="configure-azure-ad-sso"></a>Azure AD SSO の構成

これらの手順に従って、Azure portal で Azure AD SSO を有効にします。

1. [Azure portal](https://portal.azure.com/) の **Meraki Dashboard** アプリケーション統合ページで、 **[管理]** セクションを見つけて、 **[シングル サインオン]** を選択します。
1. **[シングル サインオン方式の選択]** ページで、 **[SAML]** を選択します。
1. **[SAML でシングル サインオンをセットアップします]** ページで、 **[基本的な SAML 構成]** の編集 (ペン) アイコンをクリックして設定を編集します。

   ![基本的な SAML 構成を編集する](common/edit-urls.png)

1. **[基本的な SAML 構成]** セクションで、次の手順を実行します。
     
    **[応答 URL]** ボックスに、`https://n27.meraki.com/saml/login/m9ZEgb/< UNIQUE ID >` のパターンを使用して URL を入力します。

    > [!NOTE]
    > 応答 URL 値は、実際の値ではありません。 この値を実際の応答 URL の値で更新します。これについては、このチュートリアルの後半で説明します。

1. **[保存]** ボタンをクリックします。

1. Meraki Dashboard アプリケーションは特定の形式の SAML アサーションを使用するため、カスタム属性マッピングを SAML トークン属性の構成に追加する必要があります。 次のスクリーンショットには、既定の属性一覧が示されています。

    ![image](common/default-attributes.png)

1. その他に、Meraki Dashboard アプリケーションでは、いくつかの属性が SAML 応答で返されることが想定されています。それらの属性を次に示します。 これらの属性も値が事前に設定されますが、要件に従ってそれらの値を確認することができます。
    
    | 名前 | ソース属性|
    | ---------------| --------- |
    | `https://dashboard.meraki.com/saml/attributes/username` | user.userprincipalname |
    | `https://dashboard.meraki.com/saml/attributes/role` | user.assignedroles |

    > [!NOTE]
    > Azure AD でロールを構成する方法については、[こちら](https://docs.microsoft.com/azure/active-directory/develop/active-directory-enterprise-app-role-management)を参照してください。

1. **[SAML 署名証明書]** セクションで **[編集]** ボタンをクリックして、 **[SAML 署名証明書]** ダイアログを開きます。

    ![SAML 署名証明書の編集](common/edit-certificate.png)

1. **[SAML 署名証明書]** セクションで **[Thumbprint Value]\(拇印の値\)** をコピーし、お使いのコンピューターに保存します。

    ![[Thumbprint]\(拇印\) の値をコピーする](common/copy-thumbprint.png)

1. **[Meraki Dashboard のセットアップ]** セクションで、[ログアウト URL] の値をコピーしてそれを自分のコンピューターに保存します。

    ![構成 URL のコピー](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Azure AD のテスト ユーザーの作成

このセクションでは、Azure portal 内で B.Simon というテスト ユーザーを作成します。

1. Azure portal の左側のウィンドウから、 **[Azure Active Directory]** 、 **[ユーザー]** 、 **[すべてのユーザー]** の順に選択します。
1. 画面の上部にある **[新しいユーザー]** を選択します。
1. **[ユーザー]** プロパティで、以下の手順を実行します。
   1. **[名前]** フィールドに「`B.Simon`」と入力します。  
   1. **[ユーザー名]** フィールドに「username@companydomain.extension」と入力します。 たとえば、「 `B.Simon@contoso.com` 」のように入力します。
   1. **[パスワードを表示]** チェック ボックスをオンにし、 **[パスワード]** ボックスに表示された値を書き留めます。
   1. **Create** をクリックしてください。

### <a name="assign-the-azure-ad-test-user"></a>Azure AD テスト ユーザーの割り当て

このセクションでは、B.Simon に Meraki Dashboard へのアクセスを許可することで、このユーザーが Azure シングル サインオンを使用できるようにします。

1. Azure portal で **[エンタープライズ アプリケーション]** を選択し、 **[すべてのアプリケーション]** を選択します。
1. アプリケーションの一覧で **[Meraki Dashboard]** を選択します。
1. アプリの概要ページで、 **[管理]** セクションを見つけて、 **[ユーザーとグループ]** を選択します。

   ![[ユーザーとグループ] リンク](common/users-groups-blade.png)

1. **[ユーザーの追加]** を選択し、 **[割り当ての追加]** ダイアログで **[ユーザーとグループ]** を選択します。

    ![[ユーザーの追加] リンク](common/add-assign-user.png)

1. **[ユーザーとグループ]** ダイアログの [ユーザー] の一覧から **[B.Simon]** を選択し、画面の下部にある **[選択]** ボタンをクリックします。
1. SAML アサーション内に任意のロール値が必要な場合、 **[ロールの選択]** ダイアログでユーザーに適したロールを一覧から選択し、画面の下部にある **[選択]** をクリックします。
1. **[割り当ての追加]** ダイアログで、 **[割り当て]** をクリックします。

## <a name="configure-meraki-dashboard-sso"></a>Meraki Dashboard SSO の構成

1. 別の Web ブラウザー ウィンドウで、管理者として Meraki Dashboard にサインインします。

1. **[Organization]\(組織\)**  ->  **[Settings]\(設定\)** に移動します。

    ![Meraki Dashboard の構成](./media/meraki-dashboard-tutorial/configure1.png)

1. [Authentication]\(認証\) で、 **[SAML SSO]** を **[SAML SSO enabled]\(SAML SSO が有効\)** に変更します。

    ![Meraki Dashboard の構成](./media/meraki-dashboard-tutorial/configure2.png)

1. **[Add a SAML IdP]\(SAML IdP の追加\)** をクリックします。

    ![Meraki Dashboard の構成](./media/meraki-dashboard-tutorial/configure3.png)

1. Azure portal からコピーした **[拇印]** の値を **[X.590 cert SHA1 fingerprint]\(X.590 証明書 SHA1 のフィンガープリント\)** ボックスに貼り付けます。 **[保存]** をクリックします。 保存すると、コンシューマー URL が表示されます。 [Consumer URL]\(コンシューマー URL\) の値をコピーして、Azure portal の **[基本的な SAML 構成] セクション**の **[応答 URL]** ボックスに貼り付けます。

    ![Meraki Dashboard の構成](./media/meraki-dashboard-tutorial/configure4.png)

### <a name="create-meraki-dashboard-test-user"></a>Meraki Dashboard のテスト ユーザーの作成

1. 別の Web ブラウザー ウィンドウで、管理者として Meraki Dashboard にサインインします。

1. **[Organization]\(組織\)**  ->  **[Administrators]\(管理者\)** に移動します。

    ![Meraki Dashboard の構成](./media/meraki-dashboard-tutorial/user1.png)

1. [SAML administrator roles]\(SAML 管理者ロール\) セクションで、 **[Add SAML role]\(SAML ロールの追加\)** をクリックします。

    ![Meraki Dashboard の構成](./media/meraki-dashboard-tutorial/user2.png)

1. ロール **meraki_full_admin** を入力し、 **[Organization access]\(組織アクセス\)** を **[Full]\(完全\)** としてマークします。次に、 **[Create role]\(ロールの作成\)** をクリックします。 **meraki_readonly_admin** に対してこの手順を繰り返します。今回は **[Organization access]\(組織アクセス\)** を **[Read-only]\(読み取り専用\)** ボックスとしてマークします。
 
    ![Meraki Dashboard の構成](./media/meraki-dashboard-tutorial/user3.png)

## <a name="test-sso"></a>SSO のテスト 

このセクションでは、アクセス パネルを使用して Azure AD のシングル サインオン構成をテストします。

アクセス パネル上で [Meraki Dashboar] タイルをクリックすると、SSO を設定した Meraki Dashboar に自動的にサインインします。 アクセス パネルの詳細については、[アクセス パネルの概要](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)に関する記事を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [SaaS アプリと Azure Active Directory を統合する方法に関するチュートリアルの一覧](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Azure Active Directory でのアプリケーション アクセスとシングル サインオンとは](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [Azure Active Directory の条件付きアクセスとは](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Azure AD で Meraki Dashboard を試す](https://aad.portal.azure.com/)

- [Microsoft Cloud App Security におけるセッション制御とは](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [高度な可視性と制御によって Meraki Dashboard を保護する方法](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

