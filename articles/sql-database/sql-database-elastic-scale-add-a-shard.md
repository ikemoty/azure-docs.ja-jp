---
title: Elastic Database ツールを使用してシャードを追加する | Microsoft Docs
description: Elastic Scale API を使用して新しいシャードをシャード セットに追加する方法。
services: sql-database
documentationcenter: ''
manager: jhubbard
author: ddove
editor: ''

ms.service: sql-database
ms.workload: sql-database
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/27/2016
ms.author: ddove

---
# Elastic Database ツールを使用してシャードを追加する
## 新しい範囲またはキー用のシャードを追加するには
多くの場合、アプリケーションは、既に存在しているシャード マップに対し、新しいキーまたはキー範囲から期待されるデータを処理するために新しいシャードを単に追加する必要があります。たとえば、テナント ID によってシャード化されるアプリケーションの場合、新しいテナントに対して新しいシャードをプロビジョニングすることが必要になる場合があります。また、毎月シャード化されるデータの場合、新しい月が始まる前に新しいシャードをプロビジョニングすることが必要になる場合があります。

キー値の新しい範囲が既に既存のマッピングの一部になっていない場合は、新しいシャードの追加と、そのシャードへの新しいキーまたは範囲の関連付けはとてもシンプルな作業です。

### 例: シャードとその範囲を既存のシャード マップに追加する
このサンプルでは、[TryGetShard](https://msdn.microsoft.com/library/azure/dn823929.aspx)、[CreateShard](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.createshard.aspx)、[CreateRangeMapping](https://msdn.microsoft.com/library/azure/dn807221.aspx#M:Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.RangeShardMap`1.CreateRangeMapping\(Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.RangeMappingCreationInfo{`0}\)) の各メソッドを使用して、[ShardLocation](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardlocation.shardlocation.aspx#M:Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.ShardLocation.) クラスのインスタンスを作成します。次の例では、範囲 [300, 400) を保持する、**sample\_shard\_2** という名前のデータベースと、その中の必要なすべてのスキーマ オブジェクトが作成されています。

    // sm is a RangeShardMap object.
    // Add a new shard to hold the range being added. 
    Shard shard2 = null; 

    if (!sm.TryGetShard(new ShardLocation(shardServer, "sample_shard_2"),out shard2)) 
    { 
        shard2 = sm.CreateShard(new ShardLocation(shardServer, "sample_shard_2"));  
    } 

    // Create the mapping and associate it with the new shard 
    sm.CreateRangeMapping(new RangeMappingCreationInfo<long> 
                            (new Range<long>(300, 400), shard2, MappingStatus.Online)); 


別の方法として、Powershell を使用して新しいシャード マップ マネージャーを作成できます。[こちら](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)の例を利用できます。

## 既存の範囲の空き部分用のシャードを追加するには
状況によっては、シャードに対する範囲のマッピングを既に実行し、部分的にデータが入力されている場合がありますが、以降のデータを別のシャードに宛てることができます。たとえば、日単位で範囲のシャーディングを実行し、既に 50 日がシャードに割り当てられているが、24 日目に今後のデータを別のシャードに保存できます。エラスティック データベースの[分割/マージ](sql-database-elastic-scale-overview-split-and-merge.md) ツールでこの操作を実行できますが、データ移動が不要な場合 (たとえば [25、50) 日の範囲のデータはまだ存在していません。これは 25 日の分を含み 50 日の分を除くデータです) は、シャード マップ管理 API を直接使用することで操作全体を実行できます。

### 例: 範囲の分割と新しく追加したシャードへの空き部分の割り当て
"sample\_shard\_2" という名前のデータベースと、その中のすべての必要なスキーマ オブジェクトが作成されています。

    // sm is a RangeShardMap object.
    // Add a new shard to hold the range we will move 
    Shard shard2 = null; 

    if (!sm.TryGetShard(new ShardLocation(shardServer, "sample_shard_2"),out shard2)) 
    { 

        shard2 = sm.CreateShard(new ShardLocation(shardServer, "sample_shard_2"));  
    } 

    // Split the Range holding Key 25 

    sm.SplitMapping(sm.GetMappingForKey(25), 25); 

    // Map new range holding [25-50) to different shard: 
    // first take existing mapping offline 
    sm.MarkMappingOffline(sm.GetMappingForKey(25)); 
    // now map while offline to a different shard and take online 
    RangeMappingUpdate upd = new RangeMappingUpdate(); 
    upd.Shard = shard2; 
    sm.MarkMappingOnline(sm.UpdateMapping(sm.GetMappingForKey(25), upd)); 

**重要**: この手法は、更新されるマッピング用の範囲が空であることが確実である場合のみ使用します。上記の方法では、移動される範囲のデータはチェックされないため、コード内にチェックを含めることが最善です。移動される範囲内に行が存在する場合、実際のデータ分布は更新されたシャード マップと一致しなくなります。このような場合は、[分割/マージ ツール](sql-database-elastic-scale-overview-split-and-merge.md)を代わりに使用して操作を行ってください。

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!---HONumber=AcomDC_0601_2016-->