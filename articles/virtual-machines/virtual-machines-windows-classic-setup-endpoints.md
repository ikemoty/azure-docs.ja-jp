---
title: クラシック Windows VM でのエンドポイントのセットアップ | Microsoft Docs
description: Azure 上で Windows 仮想マシンとの通信を許可するように Azure クラシック ポータルで Windows VM のエンドポイントをセットアップする方法について説明します。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: timlt
editor: ''
tags: azure-service-management

ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 09/27/2016
ms.author: cynthn

---
# <a name="how-to-set-up-endpoints-on-a-classic-windows-virtual-machine-in-azure"></a>Azure 上でクラシック Windows 仮想マシンにエンドポイントをセットアップする方法
クラシック デプロイメント モデルを使用して Azure で作成したすべての Windows 仮想マシンが、プライベート ネットワーク チャネルを介して、同じクラウド サービスまたは仮想ネットワーク内の他の仮想マシンと自動的に通信できます。 ただし、インターネットまたは他の仮想ネットワークにあるコンピューターと通信するには、仮想マシンへの着信ネットワーク トラフィックを転送するエンドポイントが必要になります。 この記事は、 [Linux 仮想マシン](virtual-machines-linux-classic-setup-endpoints.md)にも利用できます。

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]

**Resource Manager** デプロイメント モデルでは、エンドポイントは **ネットワーク セキュリティ グループ (NSG)** を使用して構成します。 詳細については、「Azure Portal を使用して VM への外部アクセスを許可する」 (virtual-machines-windows-nsg-quickstart-portal.md) を参照してください。

Azure クラシック ポータルで Windows 仮想マシンを作成すると、共通エンドポイント (リモート デスクトップや Windows PowerShell リモート処理用のエンドポイントなど) が通常は自動的に作成されます。 必要に応じて、仮想マシンの作成中や作成後に、追加のエンドポイントを構成できます。

[!INCLUDE [virtual-machines-common-classic-setup-endpoints](../../includes/virtual-machines-common-classic-setup-endpoints.md)]

## <a name="next-steps"></a>次のステップ
* Azure PowerShell コマンドレットを使用して VM エンドポイントをセットアップするには、「 [Add-AzureEndpoint](https://msdn.microsoft.com/library/azure/dn495300.aspx)」をご覧ください。
* Azure PowerShell コマンドレットを使用してエンドポイントの ACL を管理するには、「 [PowerShell を使用したエンドポイントのアクセス制御リスト (ACL) の管理](../virtual-network/virtual-networks-acl-powershell.md)」をご覧ください。
* Resource Manager デプロイメント モデルで仮想マシンを作成した場合は、Azure PowerShell を使用して、VM へのトラフィックを制御する [ネットワーク セキュリティ グループを作成](../virtual-network/virtual-networks-create-nsg-arm-ps.md) できます。

<!--HONumber=Oct16_HO2-->


