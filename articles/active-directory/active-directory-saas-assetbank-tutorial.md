---
title: 'チュートリアル: Azure Active Directory と Asset Bank の統合 | Microsoft Docs'
description: Azure Active Directory と Asset Bank の間でシングル サインオンを構成する方法について説明します。
services: active-directory
documentationcenter: ''
author: jeevansd
manager: femila
editor: ''

ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2016
ms.author: jeedes

---
# チュートリアル: Azure Active Directory と Asset Bank の統合
このチュートリアルの目的は、Asset Bank と Azure Active Directory (Azure AD) を統合する方法を説明することです。

Asset Bank と Azure AD の統合には、次の利点があります。

* Asset Bank にアクセスする Azure AD ユーザーを制御できます。
* ユーザーが自分の Azure AD アカウントで自動的に Asset Bank にサインオン (シングル サインオン) できるようにします。
* 1 つの中央サイト (Azure クラシック ポータル) でアカウントを管理できます。

SaaS アプリと Azure AD の統合の詳細については、「[Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](active-directory-appssoaccess-whatis.md)」を参照してください。

## 前提条件
Asset Bank と Azure AD の統合を構成するには、次のものが必要です。

* Azure AD サブスクリプション
* Asset Bank でのシングル サインオンが有効なサブスクリプション

> [!NOTE]
> このチュートリアルの手順をテストする場合、運用環境を使用しないことをお勧めします。
> 
> 

このチュートリアルの手順をテストするには、次の推奨事項に従ってください。

* 必要な場合を除き、運用環境は使用しないでください。
* Azure AD の評価環境がない場合は、[こちら](https://azure.microsoft.com/pricing/free-trial/)から 1 か月の評価版を入手できます。

## シナリオの説明
このチュートリアルの目的は、テスト環境で Azure AD のシングル サインオンをテストできるようにすることです。

このチュートリアルで説明するシナリオは、主に次の 2 つの要素で構成されています。

1. ギャラリーから Asset Bank を追加する
2. Azure AD シングル サインオンの構成とテスト

## ギャラリーから Asset Bank を追加する
Azure AD への Asset Bank の統合を構成するには、ギャラリーから管理対象 SaaS アプリの一覧に Asset Bank を追加する必要があります。

**ギャラリーから Asset Bank を追加するには、次の手順に従います。**

1. **Azure クラシック ポータル**の左側のナビゲーション ウィンドウで、**[Active Directory]** をクリックします。
   
    ![Active Directory][1]
2. **[ディレクトリ]** の一覧から、ディレクトリ統合を有効にするディレクトリを選択します。
3. アプリケーション ビューを開くには、ディレクトリ ビューでトップ メニューの **[アプリケーション]** をクリックします。
   
    ![アプリケーション][2]
4. ページの下部にある **[追加]** をクリックします。
   
    ![アプリケーション][3]
5. **[実行する内容]** ダイアログで、**[ギャラリーからアプリケーションを追加します]** をクリックします。
   
    ![アプリケーション][4]
6. [検索] ボックスに、「**Asset Bank**」と入力します。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_01.png)
7. 結果ウィンドウで **[Asset Bank]** を選択し、**[完了]** をクリックしてアプリケーションを追加します。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_02.png)

## Azure AD シングル サインオンの構成とテスト
このセクションの目的は、"Britta Simon" というテスト ユーザーに基づいて、Asset Bank で Azure AD のシングル サインオンを構成し、テストする方法について説明することです。

シングル サインオンを機能させるには、Azure AD ユーザーに対応する Asset Bank ユーザーが Azure AD で認識されている必要があります。言い換えると、Azure AD ユーザーと Asset Bank の関連ユーザーの間で、リンク関係が確立されている必要があります。

このリンク関係は、Azure AD の **[ユーザー名]** の値を、Asset Bank の **[Username]** の値として割り当てることで確立されます。

Asset Bank で Azure AD のシングル サインオンを構成してテストするには、次の構成要素を完了する必要があります。

1. **[Azure AD シングル サインオンの構成](#configuring-azure-ad-single-sign-on)** - ユーザーがこの機能を使用できるようにします。
2. **[Azure AD のテスト ユーザーの作成](#creating-an-azure-ad-test-user)** - Britta Simon で Azure AD のシングル サインオンをテストします。
3. **[Asset Bank のテスト ユーザーの作成](#creating-a-asset-bank-test-user)** - Asset Bank で Britta Simon に対応するユーザーを作成し、Azure AD の Britta Simon にリンクさせます。
4. **[Azure AD テスト ユーザーの割り当て](#assigning-the-azure-ad-test-user)** - Britta Simon が Azure AD のシングル サインオンを使用できるようにします。
5. **[シングル サインオンのテスト](#testing-single-sign-on)** - 構成が機能するかどうかを確認します。

### Azure AD シングル サインオンの構成
このセクションの目的は、Azure クラシック ポータルで Azure AD のシングル サインオンを有効にすることと、Asset Bank アプリケーションでシングル サインオンを構成することです。

**Asset Bank で Azure AD シングル サインオンを構成するには、次の手順に従います。**

1. Azure クラシック ポータルの **Asset Bank** アプリケーション統合ページで **[シングル サインオンの構成]** をクリックし、**[シングル サインオンの構成]** ダイアログを開きます。

![Configure Single Sign-On][6]

1. **[ユーザーの Asset Bank へのアクセスを設定してください]** ページで、**[Microsoft Azure AD シングル サインオン]** を選択し、**[次へ]** をクリックします。
   
    ![Configure Single Sign-On](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_03.png)
2. **[アプリケーション設定の構成]** ダイアログ ページで、次の手順に従います。
   
    ![Configure Single Sign-On](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_04.png)

    a.[サインオン URL] ボックスに、次のパターンを使用して、ユーザーが Asset Bank アプリケーションへのサインオンに使用する URL を入力します。**"https://<企業名>.assetbank-server.com"**

    b.**[次へ]** をクリックします。

1. **[Asset Bank でのシングル サインオンの構成]** ページで、次の手順を実行します。
   
    ![Configure Single Sign-On](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_05.png)
   
    a.**[メタデータのダウンロード]** をクリックし、コンピューターにファイルを保存します。
   
    b.**[次へ]** をクリックします。
2. お使いのアプリケーション用に構成された SSO を取得するには、メタデータ ファイルを電子メールに添付し、Asset Bank サポート チーム ([support@assetbank.co.uk](mailto:support@assetbank.co.uk)) にお問い合わせください。
3. Azure クラシック ポータルで、シングル サインオンの構成確認を選択し、**[次へ]** をクリックします。
   
    ![Azure AD Single Sign-On][10]
4. **[シングル サインオンの確認]** ページで **[完了]** をクリックします。
   
    ![Azure AD Single Sign-On][11]

### Azure AD のテスト ユーザーの作成
このセクションの目的は、Azure クラシック ポータルで Britta Simon というテスト ユーザーを作成することです。

ユーザーの一覧で **[Britta Simon]** を選択します。

![Azure AD ユーザーの作成][20]

**Azure AD でテスト ユーザーを作成するには、次の手順に従います。**

1. **Azure クラシック ポータル**の左側のナビゲーション ウィンドウで、**[Active Directory]** をクリックします。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/create_aaduser_09.png)
2. **[ディレクトリ]** の一覧から、ディレクトリ統合を有効にするディレクトリを選択します。
3. 上部のメニューで **[ユーザー]** をクリックして、ユーザーの一覧を表示します。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/create_aaduser_03.png)
4. 下部にあるツール バーで **[ユーザーの追加]** をクリックして、**[ユーザーの追加]** ダイアログ ボックスを開きます。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/create_aaduser_04.png)
5. **[このユーザーに関する情報の入力]** ダイアログ ページで、次の手順に従います。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/create_aaduser_05.png)
   
    a.[ユーザーの種類] として [組織内の新しいユーザー] を選択します。
   
    b.**[ユーザー名]** ボックスに「**BrittaSimon**」と入力します。
   
    c.**[次へ]** をクリックします。
6. **[ユーザー プロファイル]** ダイアログ ページで、次の手順に従います。
   
   ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/create_aaduser_06.png)
   
   a.**[名]** ボックスに「**Britta**」と入力します。
   
   b.**[姓]** ボックスに「**Simon**」と入力します。
   
   c.**[表示名]** ボックスに「**Britta Simon**」と入力します。
   
   d.**[ロール]** 一覧で **[ユーザー]** を選択します。
   
   e.**[次へ]** をクリックします。
7. **[一時パスワードの取得]** ダイアログ ページで、**[作成]** をクリックします。![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/create_aaduser_07.png)
8. **[一時パスワードの取得]** ダイアログ ページで、次の手順に従います。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-assetbank-tutorial/create_aaduser_08.png)
   
    a.**[新しいパスワード]** の値を書き留めます。
   
    b.**[完了]** をクリックします。

### Asset Bank テスト ユーザーの作成
このセクションの目的は、Asset Bank で Britta Simon というユーザーを作成することです。Asset Bank では、Just-In-Time プロビジョニングがサポートされています。この設定は、既定で有効になっています。

このセクションでは、ユーザー側で必要な操作はありません。存在しない Asset Bank ユーザーにアクセスしようとすると、新しいユーザーが自動的に作成されます。

> [!NOTE]
> ユーザーを手動で作成する必要がある場合は、Asset Bank のサポート チームにお問い合わせください。
> 
> 

### Azure AD テスト ユーザーの割り当て
このセクションの目的は、Britta Simon に Asset Bank へのアクセスを許可することで、このユーザーが Azure のシングル サインオンを使用できるようにすることです。

![ユーザーの割り当て][200]

**Asset Bank に Britta Simon を割り当てるには、次の手順に従います。**

1. Azure クラシック ポータルでアプリケーション ビューを開くために、ディレクトリ ビューでトップ メニューの **[アプリケーション]** をクリックします。
   
    ![ユーザーの割り当て][201]
2. アプリケーションの一覧で **[Asset Bank]** を選択します。
   
    ![Configure Single Sign-On](./media/active-directory-saas-assetbank-tutorial/tutorial_assetbank_50.png)
3. 上部のメニューで **[ユーザー]** をクリックします。
   
    ![ユーザーの割り当て][203]
4. ユーザーの一覧で **[Britta Simon]** を選択します。
5. 下部にあるツール バーで **[割り当て]** をクリックします。
   
    ![ユーザーの割り当て][205]

### シングル サインオンのテスト
このセクションの目的は、アクセス パネルを使用して Azure AD のシングル サインオン構成をテストすることです。

アクセス パネルで [Asset Bank] タイルをクリックすると、自動的に Asset Bank アプリケーションにサインオンします。

## その他のリソース
* [SaaS アプリと Azure Active Directory を統合する方法に関するチュートリアルの一覧](active-directory-saas-tutorial-list.md)
* [Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](active-directory-appssoaccess-whatis.md)

<!--Image references-->

[1]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-assetbank-tutorial/tutorial_general_205.png

<!---HONumber=AcomDC_0713_2016-->