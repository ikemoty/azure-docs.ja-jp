---
title: Azure Active Directory Identity Protection で検出されるリスク イベントの種類 | Microsoft Docs
description: このトピックでは、Azure Active Directory Identity Protection で利用可能な各種リスク イベントの詳細な概要を示します。
services: active-directory
keywords: Azure Active Directory Identity Protection, Cloud App Discovery, アプリケーションの管理, セキュリティ, リスク, リスク レベル, 脆弱性, セキュリティ ポリシー
documentationcenter: ''
author: markusvi
manager: femila
editor: ''

ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/02/2016
ms.author: markvi

---
# Azure Active Directory Identity Protection で検出されるリスク イベントの種類
Azure Active Directory Identity Protection のリスク イベントとは、次のようなイベントのことです。

* 疑いありのフラグが設定されたイベント
* ID が侵害された可能性を示すイベント

このトピックでは、利用可能な各種リスク イベントの詳細な概要を示します。

## 漏洩した資格情報
漏洩した資格情報が悪質な Web で公開されていることを、Microsoft のセキュリティ調査員が発見します。通常、このような資格情報はプレーン テキストで発見されます。発見された資格情報は、Azure AD の資格情報と照合されて、一致した場合は、Identity Protection で "漏洩した資格情報" として報告されます。

漏洩した資格情報のリスク イベントは、攻撃者がユーザー名とパスワードを利用できる明らかな兆候となるので、"高" 重大度のリスク イベントに分類されます。

## 特殊な場所へのあり得ない移動
このリスク イベントの種類では、地理的に離れた 2 つの場所で行われたサインインを識別します。少なくとも 1 つの場所は、ユーザーの過去の行動から考えて、ユーザーが普通いそうにない場所でもあります。さらに、2 つのサインインの間の時間が、最初の場所から 2 回目の場所にユーザーが移動するのに要する時間より短く、別のユーザーが同じ資格情報を使用していることを示唆します。

この機械学習アルゴリズムは、組織内の他のユーザーによって普通に使用される VPN や場所など、不可能な移動状況の原因になる明らかな "*誤検知*" を無視します。システムには 14 日間の初期学習期間があり、その間に新しいユーザーのサインイン行動が学習されます。

あり得ない移動は通常、ハッカーがサインインに成功したことのよいインジケーターとなります。ただし、ユーザーが新しい手段を使用して移動している場合、または組織内の他のユーザーが通常使用しない VPN を使用している場合、誤検知が発生する可能性があります。誤検知のもう 1 つの原因は、クライアント IP として誤ってサーバー IP を渡すアプリケーションです。その場合、そのアプリケーションのバックエンドがホストされているデータセンターからサインインが行われたように見えます (多くの場合、それは Microsoft のデータセンターであり、Microsoft 所有の IP アドレスからサインインが行われたように見えます)。このような誤検知のため、このリスク イベントのリスク レベルは "**中**" です。

## 感染しているデバイスからのサインイン
このリスク イベントの種類は、ボット サーバーと頻繁に通信していることがわかっている、マルウェアに感染したデバイスからのサインインを示します。これは、ボット サーバーと接触していた IP アドレスに対してユーザーのデバイスの IP アドレスを関連付けることによって決定されます。

このリスク イベントは、ユーザーのデバイスではなく IP アドレスを識別します。1 つの IP アドレスを複数のデバイスが使用していて、その一部だけがボット ネットワークによって制御されている場合、他のデバイスからサインインすると不要なイベントがトリガーされるため、このリスク イベントは "**低**" に分類されています。

ユーザーに連絡して、ユーザーのすべてのデバイスをスキャンして安全を確認することをお勧めします。また、ユーザーの個人デバイスが感染している可能性、または前に説明したようにユーザーと同じ IP アドレスから他のユーザーがウイルスに感染したデバイスを使用していた可能性もあります。感染したデバイスは、ウイルス対策ソフトウェアによってまだ識別されていないマルウェアに感染していることが多く、デバイスが感染する原因になるユーザーの悪い習慣を示していることもあります。

マルウェア感染に対処する方法の詳細については、[マルウェア対策センター](http://go.microsoft.com/fwlink/?linkid=335773&clcid=0x409)を参照してください。

## 匿名の IP アドレスからのサインイン
このリスク イベントの種類は、匿名プロキシ IP アドレスとして識別されている IP アドレスからのサインインにユーザーが成功したことを示します。このようなプロキシは、自分のデバイスの IP アドレスを隠したいユーザーによって使用され、悪意のある目的で使用される場合があります。

直ちにユーザーに連絡し、匿名 IP アドレスを使用していたかどうかを確認することをお勧めします。匿名 IP 自体はアカウント侵害を強く示しているわけではないので、このリスク イベントの種類のリスク レベルは "**中**" です。

## 不審なアクティビティのある IP アドレスからのサインイン
このリスク イベントの種類は、短期間に複数のユーザー アカウントで多数のサインイン試行失敗が検出された IP アドレスを示します。これは攻撃者によって使用された IP アドレスのトラフィック パターンと一致し、アカウントが既に侵害されたこと、または侵害されようとしていることを示す強いインジケーターです。これは、組織内の他のユーザーがよく使用する IP アドレスなど、明確な "*誤検知*" を無視する機械学習アルゴリズムです。システムには 14 日間の初期学習期間があり、その間に新しいユーザーおよび新しいテナントのサインイン行動が学習されます。

ユーザーに問い合わせて、疑わしい IP アドレスから実際にサインインしたかどうかを確認することをお勧めします。同じ IP アドレスを使用する複数のデバイスの一部だけが疑わしいアクティビティの原因になっている可能性があるため、このイベント種類のリスク レベルは "**中**" です。

## 未知の場所からのサインイン
このリスク イベントの種類はリアルタイム サインイン評価メカニズムであり、過去のサインインの場所 (IP、緯度/経度、ASN) を考慮して、新規/未知の場所を決定します。システムは、ユーザーが過去に使用した場所に関する情報を保持し、これらを "既知の" 場所と考えます。既知の場所のリストにまだ含まれない場所からサインインが行われると、リスク イベントがトリガーされます。システムには 14 日間の初期学習期間があり、この間はどの新しい場所にも未知の場所としてのフラグは設定されません。また、システムは、既知のデバイスからのサインイン、および既知の場所と地理的に近い場所からのサインインも無視します。<br>未知の場所は、攻撃者が盗まれた ID を使用しようとしていることを強く示している場合があります。ユーザーが移動しているとき、新しいデバイスを試した場合、新しい VPN を使用したときなど、誤検知が発生することがあります。このような誤検知のため、このリスク イベントの種類のリスク レベルは "**中**" です。

## 関連項目
* [Azure Active Directory Identity Protection](active-directory-identityprotection.md)

<!---HONumber=AcomDC_0803_2016-->