---
title: Reliable Services の概要 | Microsoft Docs
description: ステートレス サービスとステートフル サービスを使用して Microsoft Azure Service Fabric アプリケーションを作成する方法。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''

ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/06/2016
ms.author: vturecek

---
# Service Fabric の Reliable Services の概要
Azure Service Fabric アプリケーションには、コードを実行する 1 つ以上のサービスが含まれています。ここでは、[Reliable Services](service-fabric-reliable-services-introduction.md) を使用して、ステートレスとステートフルの両方の Service Fabric アプリケーションを作成する方法を説明します。

## ステートレス サービスの作成
ステートレス サービスは、クラウド アプリケーションで現在基準となっている種類のサービスです。ステートレスと見なされるのは、確実に格納する必要があるデータや高可用性を実現する必要があるデータが、サービス自体には含まれていないためです。ステートレス サービスのインスタンスが終了すると、すべての内部状態が失われます。この種類のサービスで、状態の高可用性と高い信頼性を実現するには、Azure テーブルや SQL データベースなどの外部ストアに状態を格納する必要があります。

Visual Studio 2015 を管理者として起動し、*HelloWorld* という名前の新しい Service Fabric アプリケーション プロジェクトを作成します。

![[新しいプロジェクト] ダイアログを使用して新しい Service Fabric アプリケーションを作成する](media/service-fabric-reliable-services-quick-start/hello-stateless-NewProject.png)

次に、*HelloWorldStateless* という名前のステートレス サービス プロジェクトを作成します。

![2 番目のダイアログ ボックスでステートレス サービス プロジェクトを作成する](media/service-fabric-reliable-services-quick-start/hello-stateless-NewProject2.png)

これで、ソリューションには、次の 2 つのプロジェクトが含まれています。

* *HelloWorld*。これは*サービス*を含む*アプリケーション* プロジェクトです。また、アプリケーションを説明するアプリケーション マニフェストと、アプリケーションをデプロイするのに役立つ多くの PowerShell スクリプトも含まれます。
* *HelloWorldStateless*。これはサービス プロジェクトです。ステートレス サービスの実装が含まれています。

## サービスの実装
サービス プロジェクト内にある **HelloWorldStateless.cs** ファイルを開きます。Service Fabric では、どのようなビジネス ロジックもサービスで実行できます。サービス API には、コードのエントリ ポイントが 2 つあります。

* *RunAsync* という変更可能なエントリ ポイント メソッドでは、実行時間の長いコンピューティング ワークロードなどの任意のワークロードの実行を開始できます。

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    ...
}
```

* 選択した通信スタック (ASP.NET Web API など) をプラグインできる通信エントリ ポイント。これは、ユーザーおよびその他のサービスからの要求の受信を開始できる場所です。

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    ...
}
```

このチュートリアルでは、`RunAsync()` エントリ ポイント メソッドを取り上げます。これは、コードの実行をすぐに開始できる場所です。プロジェクト テンプレートには、ローリング カウントをインクリメントする `RunAsync()` の実装例が含まれます。

> [!NOTE]
> 通信スタックを使用する方法の詳細については、「[OWIN 自己ホストによる Microsoft Azure Service Fabric Web API の概要](service-fabric-reliable-services-communication-webapi.md)」を参照してください。
> 
> 

### RunAsync
```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following sample code with your own logic
    //       or remove this RunAsync override if it's not needed in your service.

    long iterations = 0;

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        ServiceEventSource.Current.ServiceMessage(this, "Working-{0}", ++iterations);

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
}
```

プラットフォームは、サービスのインスタンスが配置され実行準備ができたときに、このメソッドを呼び出します。ステートレス サービスの場合、これは単にサービス インスタンスが開いたことを意味します。サービス インスタンスを終了する必要がある場合のために、キャンセル トークンが提供されています。Service Fabric では、サービス インスタンスの開始から終了のサイクルは、サービスのライフタイムで何度も発生する可能性があります。これは、さまざまな理由で発生する可能性があります。

* システムがリソース分散のためにサービス インスタンスを移動している。
* コードでエラーが発生している。
* アプリケーションまたはシステムがアップグレードされている。
* 基礎となるハードウェアで障害が発生している。

この調整は、サービスの可用性を高めて適切なバランスを取るために、システムによって管理されます。

`RunAsync()` は独自タスク内で実行されます。上記のコード スニペットでは、*while* ループがすぐに開始されることに注意してください。ワークロード用の別のタスクをスケジュール設定する必要はありません。ワークロードの取り消しは、提供されたキャンセル トークンを使用して協調的に調整されます。システムはタスクが完了 (正常に完了、キャンセル、または失敗) するまで待機してから、次に進みます。システムからキャンセルが要求された場合は、キャンセル トークンを利用して作業を完了し、できるだけ早く `RunAsync()` を終了することが重要です。

このステートレス サービスの例では、カウントはローカル変数に格納されます。これはステートレス サービスであるため、保存される値は、サービス インスタンスの現在のライフサイクルのみで保持されます。このサービスを移動または再起動すると、値は失われます。

## ステートフル サービスの作成
Service Fabric には、新しい種類のステートフルなサービスが導入されています。ステートフル サービスは、サービス内に状態を確実に維持でき、それを使用するコードと同じ場所に配置できます。Service Fabric によって状態の可用性が高まるため、外部ストアに状態を維持する必要がなくなります。

サービスが移動または再起動した場合でも、カウンター値をステートレスから高可用と永続性に変換するには、ステートフル サービスが必要です。

先ほどと同じ *HelloWorld* アプリケーションで、アプリケーション プロジェクトの [サービス] を右クリックし、**[追加] -> [新しい Service Fabric サービス]** を選択することで、新しいサービスを追加できます。

![Service Fabric アプリケーションにサービスを追加します。](media/service-fabric-reliable-services-quick-start/hello-stateful-NewService.png)

**[ステートフル サービス]** を選択し、*HelloWorldStateful* という名前を付けます。**[OK]** をクリックします。

![[新しいプロジェクト] ダイアログを使用して新しい Service Fabric のステートフル サービスを作成する](media/service-fabric-reliable-services-quick-start/hello-stateful-NewProject.png)

これで、アプリケーションには、ステートレス サービス *HelloWorldStateless* とステートフル サービス *HelloWorldStateful* の 2 つのサービスが含まれるようになります。

ステートフル サービスのエントリ ポイントは、ステートレス サービスと同じです。主な違いは、*状態プロバイダー*を使用して状態を確実に保存できることです。Service Fabric には、[Reliable Collection](service-fabric-reliable-services-reliable-collections.md) という状態プロバイダー実装が用意されています。Reliable Collection では、Reliable State Manager を使用してレプリケートされたデータ構造を作成できます。ステートフル Reliable Service では、この状態プロバイダーを既定で使用します。

*HelloWorldStateful* で、次の RunAsync メソッドを含む **HelloWorldStateful.cs** を開きます。

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following sample code with your own logic
    //       or remove this RunAsync override if it's not needed in your service.

    var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        using (var tx = this.StateManager.CreateTransaction())
        {
            var result = await myDictionary.TryGetValueAsync(tx, "Counter");

            ServiceEventSource.Current.ServiceMessage(this, "Current Counter Value: {0}",
                result.HasValue ? result.Value.ToString() : "Value does not exist.");

            await myDictionary.AddOrUpdateAsync(tx, "Counter", 0, (key, value) => ++value);

            // If an exception is thrown before calling CommitAsync, the transaction aborts, all changes are
            // discarded, and nothing is saved to the secondary replicas.
            await tx.CommitAsync();
        }

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
```

### RunAsync
`RunAsync()` は、ステートフル サービスとステートレス サービスで同様に動作します。ただし、ステートフル サービスでは、プラットフォームは、`RunAsync()` を実行する前に、ユーザーに代わって追加の作業を行います。この作業には、Reliable State Manager と Reliable Collection が使用できる状態にあることの確認が含まれる場合があります。

### Reliable Collection と Reliable State Manager
```csharp
var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");
```

[IReliableDictionary](https://msdn.microsoft.com/library/dn971511.aspx) は、サービスに状態を確実に格納するために使用できるディクショナリ実装です。Service Fabric と Reliable Collection を使用すると、データをサービスに直接格納できるため、外部の永続ストアが必要ありません。Reliable Collection により、データの可用性が向上します。Service Fabric では、サービスの複数の*レプリカ*を作成して管理することでこれを実現します。また、これらのレプリカとその状態遷移の管理の複雑さを取り除く API も提供します。

Reliable Collection にはカスタム型を含むすべての .NET 型を格納できます。ただし次の点にご注意ください。

* Service Fabric がノード全体で状態を*レプリケート*して状態の可用性を高め、Reliable Collection が各レプリカでデータをローカル ディスクに保存します。これは、Reliable Collection で保存されるすべてのデータは*シリアル化可能である*必要があることを意味します。既定では、Reliable Collection は [DataContract](https://msdn.microsoft.com/library/system.runtime.serialization.datacontractattribute%28v=vs.110%29.aspx) を使用してシリアル化します。そのため、既定のシリアライザーを使用する場合は、使用する型が[データ コントラクト シリアライザーでサポートされている](https://msdn.microsoft.com/library/ms731923%28v=vs.110%29.aspx)ことを確認することが重要です。
* Reliable Collection でトランザクションをコミットすると、可用性が高めるためにオブジェクトがレプリケートされます。Reliable Collection に格納されるオブジェクトは、サービスのローカル メモリに保持されます。これは、オブジェクトへのローカルな参照があることを意味します。
  
   トランザクションの Reliable Collection を更新せずに、これらのオブジェクトのローカル インスタンスを変更しないようにしてください。オブジェクトのローカル インスタンスの変更は自動的にレプリケートされないためです。オブジェクトをディクショナリに再挿入するか、ディクショナリで *update* メソッドのいずれかを使用する必要があります。

Reliable Collection の管理は Reliable State Manager が行います。サービス内のどの場所でも、Reliable Collection の名前を指定することで、Reliable State Manager に Reliable Collection をいつでも要求できます。Reliable State Manager により、参照を確実に取得できます。Reliable Collection インスタンスへの参照をクラス メンバー変数やプロパティに保存することはお勧めしません。サービスのライフサイクル中、参照が常にインスタンスに設定されていることを保証するために特に注意を払う必要があります。この作業は Reliable State Manager によって処理され、繰り返されるアクセスのために最適化されます。

### トランザクション処理と非同期処理
```C#
using (ITransaction tx = this.StateManager.CreateTransaction())
{
    var result = await myDictionary.TryGetValueAsync(tx, "Counter-1");

    await myDictionary.AddOrUpdateAsync(tx, "Counter-1", 0, (k, v) => ++v);

    await tx.CommitAsync();
}
```

Reliable Collection には、LINQ を除き、対応する `System.Collections.Generic` と `System.Collections.Concurrent` が実行する操作と同じ操作が多数あります。Reliable Collection での操作は非同期です。Reliable Collection での書き込み操作では、データをレプリケートしてディスクに保持するために I/O 操作が実行されるためです。

Reliable Collection の操作は*トランザクション*であるため、複数の Reliable Collection と操作で状態の整合性を維持できます。たとえば、1 つのトランザクション内で Reliable Queue から作業項目をデキューし、その項目の操作を実行してから、結果を Reliable Dictionary に保存するとします。これはアトミック操作として扱われので、操作全体が成功するか、操作全体がロールバックされることが保証されます。項目をデキューしたが、結果を保存する前にエラーが発生した場合は、トランザクション全体がロールバックされ、項目は処理のためにキューに残ります。

## アプリケーションの実行
*HelloWorld* アプリケーションに戻ります。ここからサービスを構築してデプロイできます。**F5** キーを押すと、アプリケーションがビルドされ、ローカル クラスターにデプロイされます。

サービスが開始されたら、生成された Event Tracing for Windows (ETW) イベントを **[診断イベント]** ウィンドウで確認できます。アプリケーションのステートレス サービスとステートフル サービスの両方のイベントが表示されるのでご注意ください。**[一時停止]** ボタンをクリックして、ストリームを一時停止できます。その後、メッセージを展開して、メッセージの詳細を調べることができます。

> [!NOTE]
> アプリケーションを実行する前に、ローカル開発クラスターが実行されていることを確認します。ローカル環境の設定については、[ファースト ステップ ガイド](service-fabric-get-started.md)をご覧ください。
> 
> 

![Visual Studio で診断イベントを表示する](media/service-fabric-reliable-services-quick-start/hello-stateful-Output.png)

## 次のステップ
[Visual Studio での Service Fabric アプリケーションのデバッグ](service-fabric-debugging-your-application.md)

[はじめに: OWIN 自己ホストによる Service Fabric Web API サービス](service-fabric-reliable-services-communication-webapi.md)

[Reliable Collection の詳細](service-fabric-reliable-services-reliable-collections.md)

[アプリケーションをデプロイする](service-fabric-deploy-remove-applications.md)

[アプリケーションのアップグレード](service-fabric-application-upgrade.md)

[Reliable Services の開発者向けリファレンス](https://msdn.microsoft.com/library/azure/dn706529.aspx)

<!---HONumber=AcomDC_0713_2016-->