---
title: Compute、Network、Storage プロバイダー | Microsoft Docs
description: Azure Resource Manager デプロイ モデルにおける Linux アプリケーションの Compute、Network、Storage リソース プロバイダー (CRP、NRP、SRP) の概要
services: virtual-machines-linux
documentationcenter: ''
author: tfitzmac
manager: timlt
editor: tysonn
tags: azure-resource-manager,azure-service-management

ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/19/2015
ms.author: tomfitz

---
# Azure Resource Manager デプロイ モデルにおける Linux アプリケーション用の Azure Compute、Network、Storage プロバイダー
Azure Resource Manager デプロイ モデルにコンピューティング、ネットワーク、ストレージの機能が追加されたことによって、IaaS 上で動作する複雑なアプリケーションのデプロイと管理が大幅に単純化されます。多くのアプリケーションは、仮想ネットワーク、ストレージ アカウント、仮想マシン、ネットワーク インターフェイスなど、リソースの組み合わせを必要とします。Azure Resource Manager デプロイ モデルには、そうしたリソースをすべてひとまとめにして単一のアプリケーションとしてデプロイおよび管理する JSON テンプレートを作成する機能が用意されています。

[!INCLUDE [virtual-machines-common-compare-deployment-models](../../includes/virtual-machines-common-compare-deployment-models.md)]

<!---HONumber=AcomDC_0824_2016-->