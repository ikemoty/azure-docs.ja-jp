---
title: Azure Active Directory と Visual Studio 接続済みサービスの概要 (WebApi プロジェクト) | Microsoft Docs
description: Visual Studio 接続済みサービスを使用して Azure Active Directory を接続または作成した後に、Web API プロジェクトで Azure AD の使用を開始する方法について説明します。
services: active-directory
documentationcenter: ''
author: TomArcher
manager: douge
editor: ''

ms.service: active-directory
ms.workload: web
ms.tgt_pltfrm: vs-getting-started
ms.devlang: na
ms.topic: article
ms.date: 08/15/2016
ms.author: tarcher

---
# Azure Active Directory と Visual Studio 接続済みサービスの概要 (Web API プロジェクト)
> [!div class="op_single_selector"]
> * [作業の開始](vs-active-directory-webapi-getting-started.md)
> * [変更内容](vs-active-directory-webapi-what-happened.md)
> 
> 

## コントローラーへのアクセスに対して認証を要求する
プロジェクトに含まれるすべてのコントローラーには、**Authorize** 属性が設定されています。この属性により、ユーザーがこれらのコントローラーによって定義された API にアクセスする際に認証が求められます。これらのコントローラーに匿名でアクセスできるようにするには、コントローラーからこの属性を削除します。より細かなレベルでアクセス許可を設定するには、コントローラー クラスではなく、認証を必要とするそれぞれのメソッドに対してこの属性を割り当てます。

[Azure Active Directory の詳細を確認する](https://azure.microsoft.com/services/active-directory/)

<!---HONumber=AcomDC_0817_2016-->