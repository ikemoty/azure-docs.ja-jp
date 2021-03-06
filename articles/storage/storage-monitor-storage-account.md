---
title: ストレージ アカウントの監視方法 | Microsoft Docs
description: Azure ポータルを使用して Azure でストレージ アカウントを監視する方法について説明します。
services: storage
documentationcenter: ''
author: robinsh
manager: carmonm
editor: tysonn

ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/03/2016
ms.author: robinsh

---
# <a name="monitor-a-storage-account-in-the-azure-portal"></a>Azure ポータルでのストレージ アカウントの監視
## <a name="overview"></a>Overview
ストレージ アカウントは、 [Azure ポータル](https://portal.azure.com)で監視できます。 ポータルを通じて監視するためのストレージ アカウントを構成する場合、Azure Storage は [Storage Analytics](http://msdn.microsoft.com/library/azure/hh343270.aspx) を使用してアカウントのメトリックを追跡し、要求データを記録します。

> [!NOTE]
> で監視データを調査すると、追加のコストがかかります [Azure ポータル](https://portal.azure.com)。 詳細については、 <a href="http://msdn.microsoft.com/library/azure/hh360997.aspx">ストレージの分析と課金に関するページ</a>を参照してください。 <br />
> 
> 現在、Azure File ストレージは、Storage Analytics のメトリックをサポートしますが、ログ記録はまだサポートされていません。 [Azure ポータル](https://portal.azure.com)から Azure File ストレージのメトリックを有効にすることができます。
> 
> 現時点では、レプリケーションの種類がゾーン冗長ストレージ (ZRS) のストレージ アカウントでは、メトリックまたはログ機能が有効になっていません。 
> 
> Storage Analytics や他のツールを使用した Azure Storage 関連の問題の特定、診断、トラブルシューティングに関する詳しいガイドについては、「 [Microsoft Azure ストレージの監視、診断、およびトラブルシューティング](storage-monitoring-diagnosing-troubleshooting.md)」をご覧ください。
> 
> 

## <a name="how-to-configure-monitoring-for-a-storage-account"></a>方法: ストレージ アカウントの監視の設定
1. [Azure ポータル](https://portal.azure.com)で、 **[ストレージ]**をクリックし、目的のストレージ アカウント名をクリックしてダッシュボードを開きます。
2. **[構成]** をクリックして、BLOB、テーブル、キューの各サービスの **[監視]** 設定まで下にスクロールします。
   
    ![MonitoringOptions](./media/storage-monitor-storage-account/Storage_MonitoringOptions.png)
3. **[監視]**で、各サービスの監視レベルおよびデータ保有ポリシーを設定します。
   
   * 監視レベルを設定するには、以下のいずれかを選択します。
     
     **[最小]** - 受信/送信、空き時間情報、遅延時間、成功のパーセンテージなどのメトリックを収集して、BLOB、テーブル、キューのサービスごとに集計します。
     
     **[詳細]** - 最小レベルのメトリックに加えて、Azure Storage サービス API のストレージ操作ごとに同じメトリックを収集します。 詳細メトリックにより、アプリケーションの操作中に発生する問題を詳しく分析できます。
     
     **[オフ]** - 監視しません。 既存の監視データは、保有期間が経過するまで残ります。

* データ保有ポリシーを設定するには、 **[保有期間 (日)]**ボックスに、データを保有する日数を 1 ～ 365 日の範囲で入力します。 保有ポリシーを設定しない場合は、「0」(ゼロ) を入力します。 保有ポリシーがない場合、監視データを削除する責任はユーザーが負います。 古くて使用しない分析データがコストをかけずに自動的に削除されるように、アカウントのストレージ分析データをどの程度の期間保持するかに基づいて、データ保有ポリシーを設定することをお勧めします。

1. 監視の構成が完了したら、 **[保存]**をクリックします。

約 1 時間後に、ダッシュボードおよび **[監視]** ページに監視データが表示されるようになります。

ストレージ アカウントの監視を構成するまで、監視データは収集されず、ダッシュボードおよび **[監視]** ページのメトリック チャートは空です。

監視レベルと保有ポリシーを設定した後、 [Azure ポータル](https://portal.azure.com)で監視する利用可能なメトリック、およびメトリック チャートにプロットするメトリックを選択できます。 監視レベルごとに既定のメトリック セットが表示されます。 **[メトリックの追加]** を使用してメトリック一覧のメトリックを追加または削除できます。

メトリックは、ストレージ アカウントの $MetricsTransactionsBlob、$MetricsTransactionsTable、$MetricsTransactionsQueue、$MetricsCapacityBlob という名前の 4 つのテーブルに保存されます。 詳細については、「 [Storage Analytics Metrics について](http://msdn.microsoft.com/library/azure/hh343258.aspx)」を参照してください。

## <a name="how-to-customize-the-dashboard-for-monitoring"></a>方法: ダッシュボードの監視用のカスタマイズ
ダッシュボードで、利用できる 9 個のメトリックから最大 6 個のメトリックを選択してメトリック チャートにプロットできます。 サービスごとに (BLOB、テーブル、およびキュー)、空き時間情報、成功のパーセンテージ、および要求数合計のメトリックが利用できます。 ダッシュボードで利用できるメトリックは、最少監視でも詳細監視でも同じです。

1. [Azure ポータル](https://portal.azure.com)で、 **[Storage]**をクリックし、目的のストレージ アカウント名をクリックしてダッシュボードを開きます。
2. チャートにプロットされるメトリックを変更するには、以下のいずれかを実行します。
   
   * 新しいメトリックをチャートに追加するには、チャートの下にあるテーブルでメトリック ヘッダーの横にある色つきのチェック ボックスをオンにします。
   * チャートにプロットされているメトリックを非表示にするには、メトリック ヘッダーの横にある色つきのチェック ボックスをオフにします。
     
       ![Monitoring_nmore](./media/storage-monitor-storage-account/storage_Monitoring_nmore.png)
3. 既定では、チャートには傾向が表示され、各メトリックの現在の値だけが表示されます (チャート上部にある **[相対]** オプション)。 Y 軸を表示して絶対値を確認するには、 **[絶対]**をクリックします。
4. メトリック チャートに表示する期間を変更するには、チャートの上部で 6 時間、24 時間、または 7 日を選択します。

## <a name="how-to-customize-the-monitor-page"></a>方法: [監視] ページのカスタマイズ
**[監視]** ページでは、ストレージ アカウントのメトリックをすべて表示できます。

* ストレージ アカウントが最小監視に構成されている場合は、受信/送信、空き時間情報、遅延時間、成功のパーセンテージなどのメトリックが BLOB、テーブル、キューの各サービスから集計されれます。
* ストレージ アカウントが詳細監視に構成されている場合は、サービス単位の集計に加えて、もっときめ細かい個々のストレージ操作のメトリックも利用できます。

以下の手順を使用して、 **[監視]** ページに表示されるメトリック チャートとテーブルに表示するストレージ メトリックを選択します。 この設定は、ストレージ アカウント監視データの収集、集計、および保存には影響しません。

## <a name="how-to-add-metrics-to-the-metrics-table"></a>How to: メトリック テーブルへのメトリックの追加
1. [Azure ポータル](https://portal.azure.com)で、 **[Storage]**をクリックし、目的のストレージ アカウント名をクリックしてダッシュボードを開きます。
2. **[監視]**をクリックします。
   
    **[監視]** ページが開きます。 既定で、メトリック テーブルには監視に使用できるメトリックのサブセットが表示されます。 図は、3 つのサービスすべてに詳細監視を構成したストレージ アカウントで表示される既定の [監視] ページを示しています。 **[メトリックの追加]** を使用して、使用できるすべてのメトリックから監視するメトリックを選択します。
   
    ![Monitoring_VerboseDisplay](./media/storage-monitor-storage-account/Storage_Monitoring_VerboseDisplay.png)
   
   > [!NOTE]
   > メトリックを選択するときはコストを考慮してください。 監視の表示を更新すると、トランザクションと送信のコストがかかります。 詳細については、 [ストレージの分析と課金に関するページ](http://msdn.microsoft.com/library/azure/hh360997.aspx)を参照してください。
   > 
   > 
3. **[メトリックの追加]**をクリックします。
   
    最少監視で使用できる集計メトリックは一覧の上部に表示されています。 チェック ボックスをオンにすると、そのメトリックがメトリック一覧に表示されます。
   
    ![AddMetricsInitialDisplay](./media/storage-monitor-storage-account/Storage_AddMetrics_InitialDisplay.png)
4. ダイアログ ボックスの右側をポイントするとスクロール バーが表示され、ドラッグして追加のメトリックを表示できます。
   
    ![AddMetricsScrollbar](./media/storage-monitor-storage-account/Storage_AddMetrics_Scrollbar.png)
5. メトリックの横にある下向き矢印をクリックして、メトリックの対象となる操作の一覧を展開します。 [Azure ポータル](https://portal.azure.com)のメトリック テーブルに表示する操作をそれぞれ選択します。
   
    下図では、承認エラーのパーセンテージ メトリックが展開されています。
   
    ![ExpandCollapse](./media/storage-monitor-storage-account/Storage_AddMetrics_ExpandCollapse.png)
6. すべてのサービスについてメトリックを選択したら、[OK](チェックマーク.md) をクリックして監視の構成を更新します。 選択したメトリックがメトリック テーブルに追加されます。
7. テーブルからメトリックを削除するには、メトリックをクリックして選択し、 **[メトリックの削除]**をクリックします。
   
    ![DeleteMetric](./media/storage-monitor-storage-account/Storage_DeleteMetric.png)

## <a name="how-to-customize-the-metrics-chart-on-the-monitor-page"></a>方法: [監視] ページのメトリック チャートのカスタマイズ
1. ストレージ アカウントの **[監視]** ページで、メトリック チャートにプロットするメトリックをメトリック テーブルで最大 6 個選択します。 メトリックを選択するには、メトリックの左側にあるチェック ボックスをオンにします。 チャートからメトリックを削除するには、チェック ボックスをオフにします。
2. チャートで相対値 (最終値だけ表示) と絶対値 (Y 軸を表示) を切り替えるには、チャートの上部で **[相対]** または **[絶対]** を選択します。
3. メトリック チャートに表示する期間を変更するには、チャートの上部で **[6 時間]**、**[24 時間]**、または **[7 日]** をクリックします。

## <a name="how-to-configure-logging"></a>方法: ログの構成
ストレージ アカウントで利用できるストレージ サービス (BLOB、テーブル、およびキュー) ごとに、読み取り要求、書き込み要求、および削除要求の診断ログを保存でき、サービスごとにデータ保有ポリシーを設定できます。

1. [Azure ポータル](https://portal.azure.com)で、 **[Storage]**をクリックし、目的のストレージ アカウント名をクリックしてダッシュボードを開きます。
2. **[構成]** をクリックし、キーボードの下方向キーを使って、**[ログ]** まで下にスクロールします。
   
    ![Storagelogging](./media/storage-monitor-storage-account/Storage_LoggingOptions.png)
3. サービス (BLOB、テーブル、およびキュー) ごとに、次のように構成します。
   
   * ログを記録する要求の種類: 読み取り要求、書き込み要求、および削除要求。
   * ログ データを保持する日数。 保有ポリシーを設定しない場合は、「0」(ゼロ) を入力します。 保有ポリシーを設定しない場合、ログを削除する責任はユーザーが負います。
4. **[保存]**をクリックします。

診断ログは、ストレージ アカウントの $logs という名前の BLOB コンテナーに保存されます。 $logs コンテナーへのアクセスの詳細については、「 [Storage Analytics Logging について](http://msdn.microsoft.com/library/azure/hh343262.aspx)」を参照してください。

<!---HONumber=Oct16_HO2-->


