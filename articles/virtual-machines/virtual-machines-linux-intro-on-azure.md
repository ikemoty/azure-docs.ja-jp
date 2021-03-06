---
title: Azure での Linux 入門 | Microsoft Docs
description: Azure での Linux 仮想マシンの使用に関する詳細です。
services: virtual-machines-linux
documentationcenter: python
author: szarkos
manager: timlt
editor: ''
tags: azure-resource-manager,azure-service-management

ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/24/2016
ms.author: szark

---
# Azure での Linux 入門
このトピックでは、Azure クラウドで Linux 仮想マシンを使用するさまざまな局面について概説します。Linux 仮想マシンのデプロイは、ギャラリー内に存在するイメージを使用する際に簡単に使用できるプロセスです。

## 認証: ユーザー名、パスワード、SSH 鍵。
Azure クラシック ポータルを使用して Linux 仮想マシンを作成すると、ユーザー名、パスワードまたは SSH 公開キーの入力が求められます。Azure で Linux 仮想マシンをデプロイするユーザー名を選択する場合、root など、既に仮想マシン内に存在するシステム アカウント (UID <100) の名前は許可されない、という制約があります。

* 詳細については、「[Linux を実行する仮想マシンの作成](virtual-machines-linux-quick-create-cli.md)」を参照してください。
* 詳細については、「[Azure 上の Linux における SSH の使用方法](virtual-machines-linux-mac-create-ssh-keys.md)」を参照してください。

## `sudo` を使用したスーパーユーザー権限の取得
Azure での仮想マシン インスタンスをデプロイする際に指定したユーザー アカウントは、特権を持つアカウントです。このアカウントは、Azure Linux エージェントによって `sudo` ユーティリティを使用して権限を root (スーパーユーザー アカウント) に昇格できるように構成されます。このユーザー アカウントを使用してログインすると、コマンド構文を使用して、root としてコマンドを実行できるようになります。

    # sudo <COMMAND>

**sudo -s** を使用して root シェルを取得することもできます。

* 詳細については、「[Azure 上の Linux 仮想マシンでの root 権限の使用](virtual-machines-linux-use-root-privileges.md)」を参照してください。

## ファイアウォールの構成
Azure では、Azure クラシック ポータルで指定されたポートに接続を制限する受信パケット フィルターが用意されています。既定では、許可されている唯一のポートは SSH です。Azure クラシック ポータルでエンドポイントを構成することで、Linux 仮想マシンの追加ポートへのアクセスを設定できます。

* 詳細については、「[仮想マシンに対してエンドポイントを設定する方法](virtual-machines-windows-classic-setup-endpoints.md)」を参照してください。

既定では、Azure ギャラリー内の Linux のイメージを使用して*iptables* ファイアウォールを有効にすることはできません。必要があれば、このファイアウォールにフィルターを追加するように構成することはできます。

## ホスト名の変更
最初に Linux イメージのインスタンスをデプロイするときに、仮想マシンのホスト名を提供するように求められます。仮想マシンが実行されると、このホスト名はプラットフォーム DNS サーバーに公開されます。相互に接続された複数の仮想マシンがホスト名を使用して IP アドレス検索を実行できるようになります。

仮想マシンがデプロイされた後でホスト名を変更する場合は、次のコマンドを使用します。

    # sudo hostname <newname>

Azure Linux エージェントには、この名前の変更を自動的に検出する機能、この変更を保持するように仮想マシンを正しく構成する機能、この変更をプラットフォーム DNS サーバーに公開する機能があります。

* [Azure Linux エージェント ユーザー ガイド](virtual-machines-linux-agent-user-guide.md)

### Cloud-Init
**Ubuntu** イメージと **CoreOS** イメージでは、仮想マシンをブートストラップするための追加機能を提供する Azure の cloud-init を使用します。

* [カスタム データを挿入する方法](virtual-machines-windows-classic-inject-custom-data.md)
* [Microsoft Azure のカスタム データと Cloud-Init](https://azure.microsoft.com/blog/2014/04/21/custom-data-and-cloud-init-on-windows-azure/)
* [Cloud-Init を使用した Azure スワップ パーティションの作成](https://wiki.ubuntu.com/AzureSwapPartitions)
* [Azure で CoreOS を使用する方法 ](https://coreos.com/os/docs/latest/booting-on-azure.html)

## 仮想マシン イメージのキャプチャ
Azure には、既存の仮想マシンの状態をイメージにキャプチャする機能があります。このイメージは、後で追加の仮想マシンのインスタンスをデプロイするために使用できます。Azure Linux エージェントは、プロビジョニング中に実行したカスタマイズをロールバックするために使用することもできます。仮想マシンをイメージとしてキャプチャする手順は次のとおりです。

1. **waagent -deprovision** を実行してプロビジョニング中のカスタマイズを元に戻します。または、**waagent -deprovision+user** を実行して、プロビジョニング中に指定したユーザー アカウントとすべての関連データを削除します。
2. 仮想マシンをシャットダウン/電源オフします。
3. Azure クラシック ポータルで *[キャプチャ]* をクリックするか、Powershell ツールまたは CLI ツールを使用して仮想マシンをイメージとしてキャプチャします。
   
   * 詳細については、「[Linux を実行する仮想マシンのイメージをキャプチャする方法](virtual-machines-linux-classic-capture-image.md)」を参照してください。

## ディスクの接続
各仮想マシンには、一時的なローカル *リソース ディスク*が接続されています。リソース ディスク上のデータには、再起動を行った場合の持続性がないため、仮想マシン上で実行されているアプリケーションやプロセスは、多くの場合、このディスクを過渡的および**一時的な**ストレージとして使用します。また、オペレーティング システムがページ ファイルやスワップ ファイルを格納する場合にも使用されます。

Linux では通常、リソース ディスクは Azure Linux エージェントによって管理され、**/mnt/resource** (または Ubuntu イメージでは **/mnt**) に自動的にマウントされます。

> [!NOTE]
> リソース ディスクは **一時ディスク**であるため、仮想マシンが再起動されると削除され再フォーマットされることに注意してください。
> 
> 

Linux では、データ ディスクはカーネルによって `/dev/sdc` という名前が付けられる場合があり、ユーザーはこのリソースをパーティション分割し、フォーマットした上で、マウントする必要があります。ディスクの接続については、チュートリアル「[データ ディスクを Linux 仮想マシンに接続する方法](virtual-machines-linux-classic-attach-disk.md)」で詳しく説明しています。

* **関連項目:** 「[Linux でのソフトウェア RAID の構成](virtual-machines-linux-configure-raid.md)」と「[Azure で Linux VM の LVM を構成する](virtual-machines-linux-configure-lvm.md)」

<!---HONumber=AcomDC_0831_2016-->