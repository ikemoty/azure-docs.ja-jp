---
title: "Microsoft Azure IoT Suite の概要 | Microsoft Docs"
description: "Azure IoT Suite は、事前構成済みの IoT (モノのインターネット) ソリューションによって、データの収集、分析、保存、可視化、他のシステムとの統合を実現します。"
services: 
suite: iot-suite
documentationcenter: 
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 2d38d08a-4133-4e5c-8b28-f93cadb5df05
ms.service: iot-suite
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/09/2016
ms.author: dobett
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 7b0df1fc79e82a99ea65296a4e09bdb48fde9451


---
# <a name="what-is-azure-iot-suite"></a>Azure IoT Suite とは
Azure IoT (モノのインターネット) サービスには、さまざまな機能が用意されています。 このエンタープライズ クラスのサービスを使用すると、次の操作を実行できます。

* デバイスからデータを収集する
* インモーション データ ストリームを分析する
* 大規模なデータ セットを格納して照会する
* リアルタイム データと履歴データを表示する
* バックオフィス システムと統合する

これらの機能を提供するため、Azure IoT Suite では、複数の Azure サービスとカスタム拡張機能が "*構成済みソリューション*" としてパッケージ化されています。 構成済みソリューションは、一般的な IoT ソリューション パターンの基本実装です。このソリューションを使用すると、IoT ソリューションの提供に必要な時間を短縮できます。 [IoT ソフトウェア開発キット][lnk-sdks]を使用し、独自の要件を満たすようにこのソリューションをカスタマイズして拡張することができます。 新しい IoT ソリューションを開発する場合は、例やテンプレートとしてこのソリューションを使用することもできます。

次のビデオで、Azure IoT Suite の概要について説明します。

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON309/player]
> 
> 

## <a name="azure-iot-services-in-azure-iot-suite"></a>Azure IoT Suite の Azure IoT サービス
通常、構成済みソリューションでは次のサービスが使用されます。

* Azure IoT Suite の中核となるものは [Azure IoT Hub][lnk-iot-hub] サービスです。 このサービスは、デバイスからクラウドへ、およびクライアントからデバイスへのメッセージング機能を提供し、クラウドや他の主要な IoT Suite サービスに対するゲートウェイとして機能します。 このサービスを使用すると、デバイスから大規模にメッセージを受信し、デバイスにコマンドを送信することができます。
* [Azure Stream Analytics][lnk-asa] には、インモーション データ分析機能があります。 IoT Suite は、このサービスを利用して、受信テレメトリを処理し、集計を実行してイベントを検出します。 また、事前構成されたソリューションは、ストリーム分析を使用して、メタデータやデバイスからのコマンド応答などのデータを含む情報メッセージも処理します。 ソリューションでは、Stream Analytics を使用し、デバイスからのメッセージを処理して他のサービスに配信します。
* [Azure Storage][lnk-azure-storage] と [Azure DocumentDB][lnk-document-db] には、データ ストレージ機能があります。 構成済みソリューションでは、BLOB ストレージを使用してテレメトリを保存し、分析に使用することができます。 ソリューションでは、DocumentDB を使用してデバイスのメタデータを格納し、ソリューションのデバイス管理機能を有効にします。
* [Azure Web Apps][lnk-web-apps] と [Microsoft Power BI][lnk-power-bi] には、データ視覚化機能があります。 Power BI は柔軟なので、IoT Suite データを使用する独自の対話型ダッシュボードをすばやく構築できます。

一般的な IoT ソリューションのアーキテクチャの概要については、[Microsoft Azure とモノのインターネット (IoT)][iot-suite-what-is-azure-iot] に関するページを参照してください。

## <a name="preconfigured-solutions"></a>構成済みのソリューション
IoT Suite に用意されている構成済みソリューションを使用すると、Azure IoT Suite で実現できる一般的な IoT シナリオ ("*リモート監視*"、"*予測的なメンテナンス*" など) をすぐに試してみることができます。 これらのソリューションを Azure サブスクリプションにデプロイし、完全なエンドツーエンドの IoT シナリオを実行することができます。

## <a name="next-steps"></a>次のステップ
これで、IoT Suite の機能についての概要と、主要なコンポーネントを把握できました。次は、[Azure IoT の構成済みソリューションの概要][lnk-what-are-preconfig]に関するページを参照して、IoT Suite の構成済みソリューションについて詳しく学習しましょう。

[lnk-sdks]: https://azure.microsoft.com/documentation/articles/iot-hub-sdks-summary/
[lnk-iot-hub]: https://azure.microsoft.com/documentation/services/iot-hub/
[lnk-asa]: https://azure.microsoft.com/documentation/services/stream-analytics/
[lnk-azure-storage]: https://azure.microsoft.com/documentation/services/storage/
[lnk-document-db]: https://azure.microsoft.com/documentation/services/documentdb/
[lnk-power-bi]: https://powerbi.microsoft.com/
[lnk-web-apps]: https://azure.microsoft.com/documentation/services/app-service/web/
[iot-suite-what-is-azure-iot]: iot-suite-what-is-azure-iot.md
[lnk-what-are-preconfig]: iot-suite-what-are-preconfigured-solutions.md



<!--HONumber=Nov16_HO2-->


