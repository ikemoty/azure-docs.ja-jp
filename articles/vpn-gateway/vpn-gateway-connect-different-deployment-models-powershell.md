---
title: PowerShell を使用してクラシック仮想ネットワークを Resource Manager の仮想ネットワークに接続する方法 | Microsoft Docs
description: VPN Gateway と PowerShell を使用して、クラシック VNet と Resource Manager の VNet の間に VPN 接続を作成する方法について説明します。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: carmonm
editor: ''
tags: azure-service-management,azure-resource-manager

ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/23/2016
ms.author: cherylmc

---
# <a name="connect-virtual-networks-from-different-deployment-models-using-powershell"></a>異なるデプロイ モデルの仮想ネットワークを PowerShell を使用して接続する
> [!div class="op_single_selector"]
> * [ポータル](vpn-gateway-connect-different-deployment-models-portal.md)
> * [PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
> 
> 

Azure には現在、クラシックと Resource Manager (RM) の 2 つの管理モデルがあります。 これまで Azure を使用してきているユーザーであれば、おそらく Azure VM およびクラシック VNet で実行されているインスタンス ロールを利用されていることでしょう。 新しい VM とロール インスタンスが、Resource Manager で作成された VNet 上で実行されていることも考えられます。 この記事では、クラシック VNet を Resource Manager VNet に接続し、異なるデプロイ モデルにあるリソースをゲートウェイ接続経由で相互に通信できるようにする方法について説明します。

別のサブスクリプション、別のリージョンに存在する VNet 間で接続を作成することができます。 オンプレミスのネットワークに既に接続されている VNet を接続することもできます。ただし、Vnet が構成されているゲートウェイが動的またはルート ベースである場合に限ります。 VNet 間接続の詳細については、この記事の最後にある「[VNet 間接続に関してよく寄せられる質問](#faq)」を参照してください。

### <a name="deployment-models-and-methods-for-connecting-vnets-in-different-deployment-models"></a>さまざまなデプロイ モデルで Vnet を接続するためのデプロイのモデルと方法
[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

以下の表は、この構成について新しい記事、追加のツールが利用できるようになったら更新されるものです。 記事が利用できるようになったら、表から直接リンクできるようにします。<br><br>

[!INCLUDE [vpn-gateway-table-vnet-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

#### <a name="vnet-peering"></a>VNET ピアリング
[!INCLUDE [vpn-gateway-vnetpeeringlink](../../includes/vpn-gateway-vnetpeeringlink-include.md)]

## <a name="before-beginning"></a>作業を開始する前に
次の手順では、各 VNet 用に動的またはルート ベースのゲートウェイを構成してゲートウェイ間の VPN 接続を作成する際に必要な設定について説明します。 この構成は、静的またはポリシー ベースのゲートウェイをサポートしていません。

### <a name="prerequisites"></a>前提条件
* 両方の VNet が既に作成されている。
* これらの VNet のアドレス範囲が互いに重複していない。また、ゲートウェイの接続先になる可能性のある他の接続の範囲と重複していない。
* 最新の PowerShell コマンドレット (1.0.2 以降) がインストール済みである。 詳細については、「 [Azure PowerShell のインストールおよび構成方法](../powershell-install-configure.md) 」ご覧ください。 必ずサービス管理 (SM) と Resource Manager (RM) のコマンドレットの両方をインストールしてください。 

### <a name="<a-name="exampleref"></a>example-settings"></a><a name="exampleref"></a>設定例
設定の例は、リファレンスとして使用できます。

**クラシック VNet の設定**

VNet Name = ClassicVNet  <br>
Location = West US <br>
Virtual Network Address Spaces = 10.0.0.0/24 <br>
Subnet-1 = 10.0.0.0/27 <br>
GatewaySubnet = 10.0.0.32/29 <br>
Local Network Name = RMVNetLocal <br>
 GatewayType = DynamicRouting

**Resource Manager の VNet の設定**

VNet Name = RMVNet  <br>
Resource Group = RG1 <br>
Virtual Network IP Address Spaces = 192.168.0.0/16 <br>
Subnet-1 = 192.168.1.0/24 <br>
GatewaySubnet = 192.168.0.0/26 <br>
Location = East US <br>
Gateway public IP name = gwpip <br>
Local Network Gateway = ClassicVNetLocal <br>
Virtual Network Gateway name = RMGateway <br>
 Gateway IP addressing configuration = gwipconfig

## <a name="<a-name="createsmgw"></a>section-1---configure-the-classic-vnet"></a><a name="createsmgw"></a>セクション 1 - クラシック VNet を構成する
### <a name="part-1---download-your-network-configuration-file"></a>パート 1 - ネットワーク構成ファイルをダウンロードする
1. 管理特権を使って PowerShell コンソールで Azure アカウントにログインします。 次のコマンドレットは、Azure アカウントのログイン資格情報をユーザーに求めます。 ログイン後にアカウント設定がダウンロードされ、Azure PowerShell で使用できるようになります。 構成のこの部分を完了するには、SM PowerShell コマンドレットを使用します。
   
        Add-AzureAccount
2. 次のコマンドを実行して、Azure のネットワーク構成ファイルをエクスポートします。 必要に応じて、ファイルの場所を変更して別の場所にエクスポートすることもできます。 ファイルを編集し、Azure にインポートします。
   
        Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
3. ダウンロードした .xml ファイルを開き、編集します。 ネットワーク構成ファイルの例については、 [ネットワークの構成スキーマ](https://msdn.microsoft.com/library/jj157100.aspx)に関するページを参照してください。

### <a name="part-2--verify-the-gateway-subnet"></a>パート 2 - ゲートウェイ サブネットを確認する
**VirtualNetworkSites** 要素にゲートウェイ サブネットがまだ作成されていない場合は、VNet に追加します。 ネットワーク構成ファイルを使用する場合は、ゲートウェイ サブネットの名前が "GatewaySubnet" である必要があります。それ以外の場合は、Azure でこれを認識してゲートウェイ サブネットとして使用することができません。

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

**例:**

    <VirtualNetworkSites>
      <VirtualNetworkSite name="ClassicVNet" Location="West US">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/24</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="Subnet-1">
            <AddressPrefix>10.0.0.0/27</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.0.0.32/29</AddressPrefix>
          </Subnet>
        </Subnets>
      </VirtualNetworkSite>
    </VirtualNetworkSites>

### <a name="part-3---add-the-local-network-site"></a>パート 3 - ローカル ネットワーク サイトを追加する
追加するローカル ネットワーク サイトは、接続先の RM VNet を表します。 まだ存在していない場合は、 **LocalNetworkSites** 要素をファイルに追加することが必要になる場合があります。 構成のこの時点では、Resource Manager の VNet のゲートウェイをまだ作成していないため、VPNGatewayAddress は任意の有効なパブリック IP アドレスにすることができます。 ゲートウェイを作成したら、このプレースホルダー IP アドレスを、RM ゲートウェイに割り当てられている正しいパブリック IP アドレスに置き換えます。

    <LocalNetworkSites>
      <LocalNetworkSite name="RMVNetLocal">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>

### <a name="part-4---associate-the-vnet-with-the-local-network-site"></a>パート 4 - VNet をローカル ネットワーク サイトに関連付ける
このセクションでは、VNet の接続先のローカル ネットワーク サイトを指定します。 ここでは、先ほど参照した Resource Manager の VNet が該当します。 名前が一致することを確認します。 この手順では、ゲートウェイは作成しません。 ゲートウェイの接続先となるローカル ネットワークを指定します。

        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="RMVNetLocal">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>

### <a name="part-5---save-the-file-and-upload"></a>パート 5 - ファイルを保存してアップロードする
ファイルを保存してから、次のコマンドを実行して Azure にインポートします。 必要に応じて、ファイル パスを環境に合わせて変更してください。

        Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml

この結果と同様の情報が表示され、インポートが成功したことがわかります。

        OperationDescription        OperationId                      OperationStatus                                                
        --------------------        -----------                      ---------------                                                
        Set-AzureVNetConfig        e0ee6e66-9167-cfa7-a746-7casb9    Succeeded 

### <a name="part-6---create-the-gateway"></a>パート 6 - ゲートウェイを作成する
クラシック ポータルまたは PowerShell を使用して、仮想ネットワーク ゲートウェイを作成できます。

次の例を使用して、動的ルーティング ゲートウェイを作成します。

    New-AzureVNetGateway -VNetName ClassicVNet -GatewayType DynamicRouting

`Get-AzureVNetGateway` コマンドレットを使用するとゲートウェイの状態を確認できます。

## <a name="<a-name="creatermgw"></a>section-2:-configure-the-rm-vnet-gateway"></a><a name="creatermgw"></a>セクション 2: RM VNet ゲートウェイを構成する
RM VNet 用の VPN ゲートウェイを作成するには、次の手順に従います。 クラシック VNet のゲートウェイのパブリック IP アドレスを取得するまで、手順を開始しないでください。 

1. **Azure アカウントにログインします** 。 次のコマンドレットは、Azure アカウントのログイン資格情報をユーザーに求めます。 ログイン後にアカウント設定がダウンロードされ、Azure PowerShell で使用できるようになります。
   
        Login-AzureRmAccount 
   
    複数のサブスクリプションがある場合は、Azure サブスクリプションの一覧が表示されます。
   
        Get-AzureRmSubscription
   
    使用するサブスクリプションを指定します。 
   
        Select-AzureRmSubscription -SubscriptionName "Name of subscription"
2. **ローカル ネットワーク ゲートウェイの作成** 仮想ネットワークでは、ローカル ネットワーク ゲートウェイは通常、オンプレミスの場所を指します。 ここでは、ローカル ネットワーク ゲートウェイはクラシック VNet を意味します。 Azure で参照できる名前を付けて、アドレス空間のプレフィックスも指定します。 指定した IP アドレス プレフィックスは、Azure がオンプレミスの場所に送信するトラフィックを特定するときに使用されます。 ここの情報を後で調整する必要がある場合は、ゲートウェイを作成する前に、値を変更してサンプルをもう一度実行することができます。<br><br>
   
   * `-Name` は、ローカル ネットワーク ゲートウェイを参照するために割り当てる名前です。<br>
   * `-AddressPrefix` は、クラシック VNet のアドレス空間です。<br>
   * `-GatewayIpAddress` は、クラシック VNet のゲートウェイのパブリック IP アドレスです。 次の例を必ず変更して、正しい IP アドレスを反映させてください。
     
           New-AzureRmLocalNetworkGateway -Name ClassicVNetLocal `
           -Location "West US" -AddressPrefix "10.0.0.0/24" `
           -GatewayIpAddress "n.n.n.n" -ResourceGroupName RG1
3. Resource Manager VNet の仮想ネットワーク ゲートウェイに割り当てる**パブリック IP アドレスを要求します**。 使用する IP アドレスを指定することはできません。 IP アドレスは仮想ネットワーク ゲートウェイに動的に割り当てられます。 ただし、これは IP アドレスが変更されるという意味ではありません。 仮想ネットワーク ゲートウェイの IP アドレスが変更されるのは、ゲートウェイが削除され、再度作成されたときのみです。 ゲートウェイのサイズ変更、リセット、その他の内部メンテナンス/アップグレードの際には変更されません。<br>この手順では、後の手順で使用する変数も設定します。

        $ipaddress = New-AzureRmPublicIpAddress -Name gwpip `
        -ResourceGroupName RG1 -Location 'EastUS' `
        -AllocationMethod Dynamic

1. **仮想ネットワークにゲートウェイ サブネットがあることを確認します**。 ゲートウェイ サブネットが存在しない場合は追加します。 ゲートウェイ サブネットには、必ず *GatewaySubnet*という名前を付けてください。
2. **サブネットを取得します** 。 この手順では、次の手順で使用する変数も設定します。
   
   * `-Name` は、Resource Manager の VNet の名前です。
   * `-ResourceGroupName` は、VNet が関連付けられているリソース グループです。 ゲートウェイ サブネットが正常に動作するには、ゲートウェイ サブネットがこの VNet に対して既に存在し、 *GatewaySubnet* という名前が付けられている必要があります。

            $subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet `
            -VirtualNetwork (Get-AzureRmVirtualNetwork -Name RMVNet -ResourceGroupName RG1) 

1. **ゲートウェイ IP アドレス指定の構成を作成する** ゲートウェイの構成で、使用するサブネットとパブリック IP アドレスを定義します。 次のサンプルを使用して、ゲートウェイの構成を作成します。<br>この手順では、`-SubnetId` パラメーターと `-PublicIpAddressId` パラメーターはそれぞれ、サブネットの id プロパティと IP アドレス オブジェクトの id プロパティを渡さなければなりません。 単純な文字列を使用することはできません。 これらの変数は、パブリック IP を要求する手順とサブネットを取得する手順で設定されます。
   
        $gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig `
        -Name gwipconfig -SubnetId $subnet.id `
        -PublicIpAddressId $ipaddress.id
2. **Resource Manager の仮想ネットワーク ゲートウェイを作成します** 。 `-VpnType` は、 *RouteBased*である必要があります。 完了には 45 分以上かかることがあります。
   
        New-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1 `
        -Location "EastUS" -GatewaySKU Standard -GatewayType Vpn `
        -IpConfigurations $gwipconfig `
        -EnableBgp $false -VpnType RouteBased
3. **パブリック IP アドレスをコピーします** 。 このアドレスは、クラシック VNet のローカル ネットワーク設定を構成するときに使用します。 次のコマンドレットを使用して、パブリック IP アドレスを取得できます。 パブリック IP アドレスは、 *IpAddress*として戻り値に表示されます。
   
        Get-AzureRmPublicIpAddress -Name gwpip -ResourceGroupName RG1

## <a name="section-3:-modify-the-local-network-for-the-classic-vnet"></a>セクション 3: クラシック VNet 用のローカル ネットワークを変更する
ネットワーク構成ファイルのエクスポート、編集、保存を行い、このファイルを Azure にインポートして戻すことで、以下の手順を実行します。 また、クラシック ポータルでこの設定を変更することができます。 

### <a name="to-modify-in-the-portal"></a>ポータルで変更するには
1. [クラシック ポータル](https://manage.windowsazure.com)を開きます。
2. クラシック ポータルの画面左側のバーを下にスクロールし、 **[ネットワーク]**をクリックします。 **[ネットワーク]** ページの上部にある **[ローカル ネットワーク]** をクリックします。 
3. **[ローカル ネットワーク]** ページで、パート 1 で構成したローカル ネットワークをクリックして選択し、ページの下部に移動して **[編集]** をクリックします。
4. **[ローカル ネットワークの詳細を指定する]** ページで、プレースホルダー IP アドレスを、前のセクションで作成した Resource Manager のゲートウェイのパブリック IP アドレスに置き換えます。 矢印をクリックして次のセクションに進みます。 **[アドレス空間]** が正しいことを確認し、変更を確定するチェックマークをクリックします。

### <a name="to-modify-using-the-network-configuration-file"></a>ネットワーク構成ファイルを変更するには
1. ネットワーク構成ファイルをエクスポートします。
2. テキスト エディターで、VPN ゲートウェイの IP アドレスを変更します。
   
        <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
3. 変更内容を保存し、編集済みのファイルを Azure にインポートします。

## <a name="<a-name="connect"></a>section-4:-create-a-connection-between-the-gateways"></a><a name="connect"></a>セクション 4: ゲートウェイ間の接続を作成する
ゲートウェイ間の接続を作成するには PowerShell が必要です。 従来の PowerShell コマンドレットを使用して、Azure アカウントを追加する必要が生じる場合もあります。 これを行うには、次のコマンドレットを使用できます。 

    Add-AzureAccount

1. **共有キーを設定します** 。 このサンプルでは、`-VNetName` がクラシック VNet の名前で、`-LocalNetworkSiteName` はクラシック ポータルでローカル ネットワークを構成したときに指定した名前です。 `-SharedKey` は、生成して指定できる値です。 ここで指定する値は、次の手順で接続を作成するときに指定するものと同じ値である必要があります。 このサンプルの戻り値は、 **Status: Successful**を示します。
   
        Set-AzureVNetGatewayKey -VNetName ClassicVNet `
        -LocalNetworkSiteName RMVNetLocal -SharedKey abc123
2. 次のコマンドを実行して、VPN 接続を作成します。
   
    **変数を設定する**
   
        $vnet01gateway = Get-AzureRMLocalNetworkGateway -Name ClassicVNetLocal -ResourceGroupName RG1
        $vnet02gateway = Get-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1
   
    **接続を作成する**<br> `-ConnectionType` は "IPsec です"。"Vnet2Vnet" ではありません。
   
        New-AzureRmVirtualNetworkGatewayConnection -Name RM-Classic -ResourceGroupName RG1 `
        -Location "East US" -VirtualNetworkGateway1 `
        $vnet02gateway -LocalNetworkGateway2 `
        $vnet01gateway -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
3. ポータルまたは `Get-AzureRmVirtualNetworkGatewayConnection` コマンドレットを使用して、接続を表示することができます。
   
        Get-AzureRmVirtualNetworkGatewayConnection -Name RM-Classic -ResourceGroupName RG1

## <a name="<a-name="faq"></a>vnet-to-vnet-faq"></a><a name="faq"></a>VNet 間接続に関してよく寄せられる質問
VNet 間接続に関するその他の情報についてよく寄せられる質問の詳細を示します。

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-vnet-vnet-faq-include.md)]

<!--HONumber=Oct16_HO2-->


