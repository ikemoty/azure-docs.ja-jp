---
title: .NET アプリケーションから Azure Search を使用する方法 | Microsoft Docs
description: .NET アプリケーションから Azure Search を使用する方法
services: search
documentationcenter: ''
author: brjohnstmsft
manager: pablocas
editor: ''

ms.service: search
ms.devlang: dotnet
ms.workload: search
ms.topic: article
ms.tgt_pltfrm: na
ms.date: 10/06/2016
ms.author: brjohnst

---
# <a name="how-to-use-azure-search-from-a-.net-application"></a>.NET アプリケーションから Azure Search を使用する方法
この記事では、 [Azure Search .NET SDK](https://msdn.microsoft.com/library/azure/dn951165.aspx)を使用する手順について説明します。 .NET SDK を使用すると、Azure Search を使用してアプリケーションにリッチな検索エクスペリエンスを実装できます。

## <a name="what's-in-the-azure-search-sdk"></a>Azure Search SDK の内容
SDK は、クライアント ライブラリ `Microsoft.Azure.Search`で構成されます。 SDK を使用すると、インデックス、データ ソース、インデクサーの管理、ドキュメントのアップロードと管理、クエリの実行を行うことができ、HTTP や JSON の細部を処理する必要はありません。

クライアント ライブラリでは、`Index`、`Field`、`Document` などのクラス、および `SearchServiceClient` や `SearchIndexClient` クラス上の `Indexes.Create` や `Documents.Search` などの操作が定義されています。 これらのクラスは、次の名前空間にまとめられています。

* [Microsoft.Azure.Search](https://msdn.microsoft.com/library/azure/microsoft.azure.search.aspx)
* [Microsoft.Azure.Search.Models](https://msdn.microsoft.com/library/azure/microsoft.azure.search.models.aspx)

Azure Search .NET SDK の最新バージョンが一般公開されました。 次のバージョンに組み込むためにフィードバックを提供する場合は、 [フィードバック ページ](https://feedback.azure.com/forums/263029-azure-search/)を使用してください。

.NET SDK はバージョン `2015-02-28` の Azure Search REST API をサポートします。ドキュメントは [MSDN](https://msdn.microsoft.com/library/azure/dn798935.aspx) をご覧ください。 このバージョンでは、Lucene クエリ構文と Microsoft 言語アナライザーがサポートされるようになりました。 `moreLikeThis` 検索パラメーターのサポートなど、このバージョンの一部*ではない*新機能は[プレビュー](search-api-2015-02-28-preview.md)段階であり、SDK ではまだ使用できません。 これらの機能の最新の状態については、「[Azure Search サービスのバージョン](https://msdn.microsoft.com/library/azure/dn864560.aspx)」で確認できます。

この SDK でサポートされない他の機能は次のとおりです。

* [管理操作](https://msdn.microsoft.com/library/azure/dn832684.aspx)。 管理操作には、Azure Search サービスのプロビジョニングや API キーの管理が含まれます。 これらは、将来別の Azure Search .NET Management SDK でサポートされます。

## <a name="upgrading-to-the-latest-version-of-the-sdk"></a>最新バージョンの SDK へのアップグレード
古いバージョンの Azure Search .NET SDK を既に使用しており、一般公開された新しいバージョンにアップグレードする場合、方法については [この記事](search-dotnet-sdk-migration.md) をご覧ください。

## <a name="requirements-for-the-sdk"></a>SDK の要件
1. Visual Studio 2013 または Visual Studio 2015。
2. 自分が所有する Azure Search サービス。 SDK を使用するには、サービスの名前および 1 つまたは複数の API キーが必要です。 [ポータルでの Azure Search サービスの作成](search-create-service-portal.md) 」は、これらの手順の参考になります。
3. Visual Studio の [NuGet パッケージの管理] を使用して、Azure Search .NET SDK の [NuGet パッケージ](http://www.nuget.org/packages/Microsoft.Azure.Search) をダウンロードします。 NuGet.org でパッケージの名前 `Microsoft.Azure.Search` を検索してください。

Azure Search .NET SDK は、.NET Framework 4.5 を対象とするアプリケーション、および Windows 8.1 と Windows Phone 8.1 を対象とする Windows ストア アプリケーションをサポートします。 Silverlight はサポートされません。

## <a name="core-scenarios"></a>主要なシナリオ
検索アプリケーションではいくつかの処理を実行する必要があります。 このチュートリアルではこれらの主要なシナリオについて説明します。

* インデックスの作成
* インデックスへのドキュメントの設定
* フルテキスト検索およびフィルターを使用したドキュメントの検索

後のサンプル コードではこれらについて示します。 これらのコード スニペットを独自のアプリケーションに自由に使用してください。

### <a name="overview"></a>Overview
これから説明するサンプル アプリケーションは、"hotels" という名前のインデックスを新しく作成し、いくつかのドキュメントをそこに格納してから、検索クエリを実行します。 全体的な流れがわかるメイン プログラムを次に示します。

    // This sample shows how to delete, create, upload documents and query an index
    static void Main(string[] args)
    {
        // Put your search service name here. This is the hostname portion of your service URL.
        // For example, if your service URL is https://myservice.search.windows.net, then your
        // service name is myservice.
        string searchServiceName = "myservice";

        string apiKey = "Put your API admin key here.";

        SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(apiKey));

        Console.WriteLine("{0}", "Deleting index...\n");
        DeleteHotelsIndexIfExists(serviceClient);

        Console.WriteLine("{0}", "Creating index...\n");
        CreateHotelsIndex(serviceClient);

        ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");

        Console.WriteLine("{0}", "Uploading documents...\n");
        UploadDocuments(indexClient);

        Console.WriteLine("{0}", "Searching documents 'fancy wifi'...\n");
        SearchDocuments(indexClient, searchText: "fancy wifi");

        Console.WriteLine("\n{0}", "Filter documents with category 'Luxury'...\n");
        SearchDocuments(indexClient, searchText: "*", filter: "category eq 'Luxury'");

        Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
        Console.ReadKey();
    }

このプログラムの手順を詳しく見ていきましょう。 最初に、新しい `SearchServiceClient`を作成する必要があります。 このオブジェクトを使用してインデックスを管理できます。 このオブジェクトを作成するには、Azure Search サービス名および管理 API キーを提供する必要があります。

        // Put your search service name here. This is the hostname portion of your service URL.
        // For example, if your service URL is https://myservice.search.windows.net, then your
        // service name is myservice.
        string searchServiceName = "myservice";

        string apiKey = "Put your API admin key here.";

        SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(apiKey));

> [!NOTE]
> 正しくないキーを提供すると (たとえば、管理者キーが必要なときにクエリ キーを渡すなど)、`Indexes.Create` などの操作メソッドを初めて呼び出したときに、`SearchServiceClient` は `CloudException` をスローして "アクセス不可" メッセージを表示します。 このような場合は、API キーを再確認してください。
> 
> 

次の数行では、メソッドを呼び出して "hotels" という名前のインデックスを作成します。インデックスが既にある場合は最初に削除します。 これらのメソッドについては後で説明します。

        Console.WriteLine("{0}", "Deleting index...\n");
        DeleteHotelsIndexIfExists(serviceClient);

        Console.WriteLine("{0}", "Creating index...\n");
        CreateHotelsIndex(serviceClient);

次に、インデックスを設定する必要があります。 そのためには、 `SearchIndexClient`が必要です。 これを取得するには、作成する方法と、`SearchServiceClient` で `Indexes.GetClient` を呼び出す方法があります。 ここでは簡単な後者を使用します。

        ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");

> [!NOTE]
> 一般的な検索アプリケーションでは、インデックスの管理とインデックスの設定は、検索クエリとは別のコンポーネントによって処理されます。 `Indexes.GetClient` は、別の `SearchCredentials` を指定する手間を省くため、インデックスを作成するのに便利です。 そのためには、`SearchServiceClient` を作成するときに使用した管理者キーを新しい `SearchIndexClient` に渡します。 ただし、アプリケーションのクエリを実行する部分では、管理者キーの代わりにクエリ キーを渡すことができるように、 `SearchIndexClient` を直接作成する方が適しています。 これは、最小権限の原則にも適合しており、アプリケーションのセキュリティ強化に役立ちます。 管理者キーとクエリ キーの詳細については、 [こちら](https://msdn.microsoft.com/library/azure/dn798935.aspx)を参照してください。
> 
> 

`SearchIndexClient`を作成したので、インデックスを設定できます。 これは、後で説明する別のメソッドで行います。

        Console.WriteLine("{0}", "Uploading documents...\n");
        UploadDocuments(indexClient);

最後に、再び `SearchIndexClient`を使用して、いくつか検索クエリを実行し、結果を表示します。

        Console.WriteLine("{0}", "Searching documents 'fancy wifi'...\n");
        SearchDocuments(indexClient, searchText: "fancy wifi");

        Console.WriteLine("\n{0}", "Filter documents with category 'Luxury'...\n");
        SearchDocuments(indexClient, searchText: "*", filter: "category eq 'Luxury'");

        Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
        Console.ReadKey();

有効なサービス名と API キーを使用してこのアプリケーションを実行した場合、出力は次のようになります。

    Deleting index...

    Creating index...

    Uploading documents...

    Searching documents 'fancy wifi'...

    ID: 1058-441    Name: Fancy Stay        Category: Luxury        Tags: [pool, view, concierge]
    ID: 956-532     Name: Express Rooms     Category: Budget        Tags: [wifi, budget]

    Filter documents with category 'Luxury'...

    ID: 1058-441    Name: Fancy Stay        Category: Luxury        Tags: [pool, view, concierge]
    ID: 566-518     Name: Surprisingly Expensive Suites     Category: Luxury    Tags: []
    Complete.  Press any key to end application...

アプリケーションの完全なソース コードは、この記事の最後で提供します。

次に、 `Main`によって呼び出される各メソッドを詳しく見ていきます。

### <a name="creating-an-index"></a>インデックスの作成
`SearchServiceClient` を作成した後、`Main` は次に、"hotels" インデックスが既に存在する場合はそれを削除します。 この処理は次のメソッドで行います。

    private static void DeleteHotelsIndexIfExists(SearchServiceClient serviceClient)
    {
        if (serviceClient.Indexes.Exists("hotels"))
        {
            serviceClient.Indexes.Delete("hotels");
        }
    }

このメソッドは、指定された `SearchServiceClient` を使用してインデックスが存在するかどうかを確認し、存在する場合は、それを削除します。

> [!NOTE]
> この記事のコード例では、わかりやすくするため、Azure Search .NET SDK の同期メソッドを使用します。 実際のアプリケーションでは、高い拡張性と応答性を維持するため、非同期メソッドを使用することをお勧めします。 たとえば、上記のメソッドでは、`Exists` と `Delete` の代わりに、`ExistsAsync` および `DeleteAsync` を使用できます。
> 
> 

次に、 `Main` は次のメソッドを呼び出すことによって、新しい "hotels" インデックスを作成します。

    private static void CreateHotelsIndex(SearchServiceClient serviceClient)
    {
        var definition = new Index()
        {
            Name = "hotels",
            Fields = new[]
            {
                new Field("hotelId", DataType.String)                       { IsKey = true },
                new Field("hotelName", DataType.String)                     { IsSearchable = true, IsFilterable = true },
                new Field("baseRate", DataType.Double)                      { IsFilterable = true, IsSortable = true },
                new Field("category", DataType.String)                      { IsSearchable = true, IsFilterable = true, IsSortable = true, IsFacetable = true },
                new Field("tags", DataType.Collection(DataType.String))     { IsSearchable = true, IsFilterable = true, IsFacetable = true },
                new Field("parkingIncluded", DataType.Boolean)              { IsFilterable = true, IsFacetable = true },
                new Field("lastRenovationDate", DataType.DateTimeOffset)    { IsFilterable = true, IsSortable = true, IsFacetable = true },
                new Field("rating", DataType.Int32)                         { IsFilterable = true, IsSortable = true, IsFacetable = true },
                new Field("location", DataType.GeographyPoint)              { IsFilterable = true, IsSortable = true }
            }
        };

        serviceClient.Indexes.Create(definition);
    }

このメソッドは、新しいインデックスのスキーマを定義する `Field` オブジェクトのリストで新しい `Index` オブジェクトを作成します。 各フィールドには、名前、データ型、および検索動作を定義するいくつかの属性があります。 フィールドに加えて、スコアリング プロファイル、サジェスター、または CORS オプションを Index に追加することもできます (簡潔さを優先し、サンプルではこれらは省略されています)。 Index オブジェクトとその構成要素の詳細については、[MSDN](https://msdn.microsoft.com/library/azure/microsoft.azure.search.models.index_members.aspx) の SDK リファレンスおよび [Azure Search REST API リファレンス](https://msdn.microsoft.com/library/azure/dn798935.aspx)を参照してください。

### <a name="populating-the-index"></a>インデックスの設定
`Main` の次の手順では、新しく作成したインデックスを設定します。 この処理は次のメソッドで行います。

    private static void UploadDocuments(ISearchIndexClient indexClient)
    {
        var documents =
            new Hotel[]
            {
                new Hotel()
                {
                    HotelId = "1058-441",
                    HotelName = "Fancy Stay",
                    BaseRate = 199.0,
                    Category = "Luxury",
                    Tags = new[] { "pool", "view", "concierge" },
                    ParkingIncluded = false,
                    LastRenovationDate = new DateTimeOffset(2010, 6, 27, 0, 0, 0, TimeSpan.Zero),
                    Rating = 5,
                    Location = GeographyPoint.Create(47.678581, -122.131577)
                },
                new Hotel()
                {
                    HotelId = "666-437",
                    HotelName = "Roach Motel",
                    BaseRate = 79.99,
                    Category = "Budget",
                    Tags = new[] { "motel", "budget" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(1982, 4, 28, 0, 0, 0, TimeSpan.Zero),
                    Rating = 1,
                    Location = GeographyPoint.Create(49.678581, -122.131577)
                },
                new Hotel()
                {
                    HotelId = "970-501",
                    HotelName = "Econo-Stay",
                    BaseRate = 129.99,
                    Category = "Budget",
                    Tags = new[] { "pool", "budget" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(1995, 7, 1, 0, 0, 0, TimeSpan.Zero),
                    Rating = 4,
                    Location = GeographyPoint.Create(46.678581, -122.131577)
                },
                new Hotel()
                {
                    HotelId = "956-532",
                    HotelName = "Express Rooms",
                    BaseRate = 129.99,
                    Category = "Budget",
                    Tags = new[] { "wifi", "budget" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(1995, 7, 1, 0, 0, 0, TimeSpan.Zero),
                    Rating = 4,
                    Location = GeographyPoint.Create(48.678581, -122.131577)
                },
                new Hotel()
                {
                    HotelId = "566-518",
                    HotelName = "Surprisingly Expensive Suites",
                    BaseRate = 279.99,
                    Category = "Luxury",
                    ParkingIncluded = false
                }
            };

        try
        {
            var batch = IndexBatch.Upload(documents);
            indexClient.Documents.Index(batch);
        }
        catch (IndexBatchException e)
        {
            // Sometimes when your Search service is under load, indexing will fail for some of the documents in
            // the batch. Depending on your application, you can take compensating actions like delaying and
            // retrying. For this simple demo, we just log the failed document keys and continue.
            Console.WriteLine(
                "Failed to index some of the documents: {0}",
                String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
        }

        // Wait a while for indexing to complete.
        Thread.Sleep(2000);
    }

このメソッドには 4 つの部分があります。 最初の部分では、インデックスにアップロードする入力データとして使用される `Hotel` オブジェクトの配列を作成します。 このデータは、わかりやすくするためハードコーディングされています。 実際のアプリケーションでは、通常、データは SQL Database などの外部データ ソースから取得されます。

2 番目の部分では、ドキュメントを含む `IndexBatch` を作成します。 この場合は `IndexBatch.Upload`を呼び出すことによって、作成時にバッチに適用する操作を指定します。 その後、バッチは `Documents.Index` メソッドによって Azure Search インデックスにアップロードされます。

> [!NOTE]
> この例では、単にドキュメントをアップロードします。 既存のドキュメントに変更をマージする、またはドキュメントを削除する場合は、代わりに `IndexBatch.Merge`、`IndexBatch.MergeOrUpload`、または `IndexBatch.Delete` を呼び出すことによってバッチを作成できます。 `IndexBatch.New` を呼び出すことによって 1 つのバッチでさまざまな操作を組み合わせることもできます。これによって、`IndexAction` オブジェクトのコレクションを受け取り、各オブジェクトがドキュメントで特定の操作を実行するよう Azure Search に指示します。 `IndexAction.Merge` や `IndexAction.Upload` などの対応するメソッドを呼び出すことによって、独自の操作を行う `IndexAction` を作成できます。
> 
> 

このメソッドの 3 番目の部分は、インデックス作成の重要なエラー ケースを処理する catch ブロックです。 Azure Search がバッチ内の一部のドキュメントのインデックス作成に失敗した場合、`Documents.Index` は `IndexBatchException` をスローします。 サービスの負荷が高いときにドキュメントのインデックスを作成すると、これが発生する場合があります。 **コードでこのケースを明示的に処理することを強くお勧めします。** しばらく待ってから失敗したドキュメントのインデックス作成を再試行したり、サンプルと同じようにログに記録してから続けることができます。または、アプリケーションのデータ整合性要件に応じて他の処理を行うこともできます。

最後に、メソッドは 2 秒間遅延します。 インデックスの作成は Azure Search サービスで非同期的に行われるので、サンプル アプリケーションは短い時間待機して、確実にドキュメントを検索に使用できるようにする必要があります。 通常、このような遅延は、デモ、テスト、およびサンプル アプリケーションでのみ必要です。

#### <a name="how-the-.net-sdk-handles-documents"></a>.NET SDK がドキュメントを処理する方法
Azure Search .NET SDK が `Hotel` のようなユーザー定義クラスのインスタンスをどのようにしてインデックスにアップロードできるのか不思議に思われるかもしれません。 その質問に答えるため、 `Hotel` クラスを見ていくことにします。

    [SerializePropertyNamesAsCamelCase]
    public class Hotel
    {
        public string HotelId { get; set; }

        public string HotelName { get; set; }

        public double? BaseRate { get; set; }

        public string Category { get; set; }

        public string[] Tags { get; set; }

        public bool? ParkingIncluded { get; set; }

        public DateTimeOffset? LastRenovationDate { get; set; }

        public int? Rating { get; set; }

        public GeographyPoint Location { get; set; }

        public override string ToString()
        {
            return String.Format(
                "ID: {0}\tName: {1}\tCategory: {2}\tTags: [{3}]",
                HotelId,
                HotelName,
                Category,
                (Tags != null) ? String.Join(", ", Tags) : String.Empty);
        }
    }

最初に気付くのは、`Hotel` の各パブリック プロパティがインデックス定義のフィールドに対応していることですが、1 つ重要な違いがあります。各フィールドの名前が小文字で始まっているのに対し ("camel case")、`Hotel` の各パブリック プロパティの名前は大文字で始まっています ("Pascal case")。 これは、ターゲット スキーマをアプリケーション開発者が制御できない場合にデータ バインドを実行する .NET アプリケーションでの一般的なシナリオです。 プロパティ名を camel-case にして .NET の命名ガイドラインに違反するのではなく、プロパティ名を自動的に camel-case にマップするように `[SerializePropertyNamesAsCamelCase]` 属性で SDK に指示できます。

> [!NOTE]
> Azure Search .NET SDK は、 [NewtonSoft JSON.NET](http://www.newtonsoft.com/json/help/html/Introduction.htm) ライブラリを使用して、カスタムのモデル オブジェクトから JSON 形式へのシリアル化や JSON 形式からの逆シリアル化を行います。 必要に応じてこのシリアル化をカスタマイズできます。 詳細については、「[JSON.NET 使用したシリアル化のカスタマイズ](#JsonDotNet)」をご覧ください。
> 
> 

`Hotel` クラスに関する 2 番目の重要な点は、パブリック プロパティのデータ型です。 これらのプロパティの .NET 型は、インデックス定義でそれらと同等のフィールド型にマップします。 たとえば、`Category` 文字列プロパティは、`Edm.String` 型の `category` フィールドにマップします。 `bool?` と `Edm.Boolean`、`DateTimeOffset?` と `Edm.DateTimeOffset` などの間にも、同じような型のマッピングがあります。型のマッピングの具体的なルールについては、[MSDN](https://msdn.microsoft.com/library/azure/dn931291.aspx) で `Documents.Get` メソッドを参照してください。

ドキュメントとして独自のクラスを使用するこの機能は、両方向で動作します。また、次のセクションで見るように、検索結果を取得し、SDK で自動的に任意の型に逆シリアル化することもできます。

> [!NOTE]
> Azure Search .NET SDK は、`Document` クラスを使用して動的に型指定されたドキュメントもサポートします。これは、フィールドの値に対するフィールド名のキー/値マッピングです。 この機能は、設計時にインデックス スキーマがわからない場合、または特定のモデル クラスにバインドすると不都合な場合に便利です。 ドキュメントを処理する SDK のすべてのメソッドには、`Document` クラスを使用するオーバーロード、およびジェネリック型パラメーターを使用する厳密な型指定のオーバーロードがあります。 このチュートリアルのサンプル コードでは、後者のみを使用しています。 `Document` クラスの詳細については、[こちら](https://msdn.microsoft.com/library/azure/microsoft.azure.search.models.document.aspx)をご覧ください。
> 
> 

**データ型に関する重要な注意事項**

Azure Search インデックスにマップする独自のモデル クラスを設計するときは、`bool` や `int` などの値型のプロパティを null 許容型として宣言することをお勧めします (たとえば、`bool` ではなく `bool?` を使用する)。 null 非許容プロパティを使用する場合、対応するフィールドに null 値が含まれるドキュメントがインデックス内に存在しないことを、開発者が **保証する** 必要があります。 SDK または Azure Search サービスで、これを強制することはできません。

これは単なる仮定上の問題ではありません。`Edm.Int32` 型の既存のインデックスに新しいフィールドを追加する場合を考えてみてください。 インデックスの定義を更新した後、(Azure Search ではすべての型が null を許容するので) すべてのドキュメントでその新しいフィールドの値が null になります。 その後、そのフィールドが null 非許容型の `int` プロパティであるモデル クラスを使用した場合、ドキュメントを取得しようとすると、次のような `JsonSerializationException` が発生します。

    Error converting value {null} to type 'System.Int32'. Path 'IntValue'.

このため、ベスト プラクティスとして、モデル クラスでは null 許容型を使用することをお勧めします。

<a name="JsonDotNet"></a>

#### <a name="custom-serialization-with-json.net"></a>JSON.NET 使用したシリアル化のカスタマイズ
SDK では、ドキュメントのシリアル化と逆シリアル化に JSON.NET を使用します。 独自の `JsonConverter` または `IContractResolver` を定義して、必要に応じてシリアル化と逆シリアル化をカスタマイズできます (詳細については、「[JSON.NET のドキュメント](http://www.newtonsoft.com/json/help/html/Introduction.htm)」をご覧ください)。 この機能は、アプリケーションの既存のモデル クラスを Azure Search 用に適合させる場合、およびその他の高度なシナリオに役立ちます。 たとえば、カスタム シリアル化を使用すると次のことが可能です。

* ドキュメント フィールドとして格納されるものに、モデル クラスの特定のプロパティを含める、または除外する。
* コードのプロパティ名とインデックスのフィールド名をマップする。
* ドキュメント フィールドへのプロパティのマッピングおよび対応するインデックス定義の作成の両方に使用できるカスタム属性を作成する。

Azure Search .NET SDK のユニット テストにカスタム シリアル化を実装する例については、GitHub を参照してください。 手始めとしては、[このフォルダー](https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/Search/Search.Tests/Tests/Models)が適しています。 カスタム シリアル化のテストに使用されるクラスが含まれます。

### <a name="searching-for-documents-in-the-index"></a>インデックス内のドキュメントの検索
サンプル アプリケーションでは最後に、インデックス内のいくつかのドキュメントを検索します。 次のメソッドがこれを行います。

    private static void SearchDocuments(ISearchIndexClient indexClient, string searchText, string filter = null)
    {
        // Execute search based on search text and optional filter
        var sp = new SearchParameters();

        if (!String.IsNullOrEmpty(filter))
        {
            sp.Filter = filter;
        }

        DocumentSearchResult<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
        foreach (SearchResult<Hotel> result in response.Results)
        {
            Console.WriteLine(result.Document);
        }
    }

最初に、このメソッドは新しい `SearchParameters` オブジェクトを作成します。 このオブジェクトは、並べ替え、フィルター処理、ページング、ファセットなどの追加オプションをクエリに対して指定するために使用されます。 この例では、 `Filter` プロパティのみを設定しています。

次の手順では、検索クエリを実際に実行します。 これは `Documents.Search` メソッドを使用して行われます。 この例では、使用する検索テキストを文字列として、また前に作成した検索パラメーターを渡します。 また、`Documents.Search` に対する型パラメーターとして `Hotel` も指定します。これは、検索結果のドキュメントを `Hotel` 型のオブジェクトに逆シリアル化するように SDK に指示します。

最後に、このメソッドは検索結果のすべての一致を反復処理し、各ドキュメントをコンソールに出力します。

このメソッドの呼び出し方法を詳しく見ていきます。

    SearchDocuments(indexClient, searchText: "fancy wifi");

    SearchDocuments(indexClient, searchText: "*", filter: "category eq 'Luxury'");

最初の呼び出しでは、クエリ語句 "fancy" または "wifi" を含むすべてのドキュメントを検索します。 2 番目の呼び出しでは、検索テキストが "*" に設定されています。これは、「すべてを検索する」ことを意味します。 検索クエリ式の構文の詳細については、[こちら](https://msdn.microsoft.com/library/azure/dn798920.aspx)をご覧ください。

3 番目の呼び出しでは、OData の `$filter` 式 `category eq 'Luxury'` を使用しています。 これは、 `category` フィールドが文字列 "Luxury" と厳密に一致するドキュメントだけを返すように検索を制限します。 Azure Search がサポートする OData 構文の詳細については、 [こちら](https://msdn.microsoft.com/library/azure/dn798921.aspx)を参照してください。

これら 2 つの呼び出しで行われている処理がわかったので、出力が次のようになる理由を理解しやすいでしょう。

    Searching documents 'fancy wifi'...

    ID: 1058-441    Name: Fancy Stay        Category: Luxury        Tags: [pool, view, concierge]
    ID: 956-532     Name: Express Rooms     Category: Budget        Tags: [wifi, budget]

    Filter documents with category 'Luxury'...

    ID: 1058-441    Name: Fancy Stay        Category: Luxury        Tags: [pool, view, concierge]
    ID: 566-518     Name: Surprisingly Expensive Suites     Category: Luxury    Tags: []

最初の検索では 2 つのドキュメントが返されます。 1 番目のドキュメントは名前に "Fancy" が含まれ、2 番目のドキュメントは `tags` フィールドに "wifi" が含まれます。 2 番目の検索では 2 つのドキュメントが返されます。インデックスの `category` フィールドが "Luxury" に設定されているドキュメントはこの 2 つだけです。

チュートリアルはここまでですが、ここで止めないでください。 **次のステップ** では、Azure Search をさらに学習するための他のリソースを提供します。

## <a name="next-steps"></a>次のステップ
* MSDN で、[.NET SDK](https://msdn.microsoft.com/library/azure/dn951165.aspx) と [REST API](https://msdn.microsoft.com/library/azure/dn798935.aspx) のリファレンスを参照してください。
* [ビデオおよび他のサンプルとチュートリアル](search-video-demo-tutorial-list.md)によって、知識を深めてください。
* このバージョンの Azure Search SDK の機能については、「 [Azure Search の概要](https://msdn.microsoft.com/library/azure/dn798933.aspx)
* [名前付け規則](https://msdn.microsoft.com/library/azure/dn857353.aspx) で、さまざまなオブジェクトに名前を付けるときの規則を学習してください。
* Azure Search で [サポートされるデータ型](https://msdn.microsoft.com/library/azure/dn798938.aspx) を確認してください。

## <a name="sample-application-source-code"></a>サンプル アプリケーションのソース コード
このチュートリアルで使用したサンプル アプリケーションの完全なソース コードを次に示します。 サンプルをビルドして実行する場合、Program.cs のサービス名と API キーのプレースホルダーを実際の値に置き換える必要があることに注意してください。

Program.cs:

```csharp
using System;
using System.Configuration;
using System.Linq;
using System.Threading;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Microsoft.Spatial;

namespace AzureSearch.SDKHowTo
{
    class Program
    {
        // This sample shows how to delete, create, upload documents and query an index
        static void Main(string[] args)
        {
            // Put your search service name here. This is the hostname portion of your service URL.
            // For example, if your service URL is https://myservice.search.windows.net, then your
            // service name is myservice.
            string searchServiceName = "myservice";

            string apiKey = "Put your API admin key here.";

            SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(apiKey));

            Console.WriteLine("{0}", "Deleting index...\n");
            DeleteHotelsIndexIfExists(serviceClient);

            Console.WriteLine("{0}", "Creating index...\n");
            CreateHotelsIndex(serviceClient);

            ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");

            Console.WriteLine("{0}", "Uploading documents...\n");
            UploadDocuments(indexClient);

            Console.WriteLine("{0}", "Searching documents 'fancy wifi'...\n");
            SearchDocuments(indexClient, searchText: "fancy wifi");

            Console.WriteLine("\n{0}", "Filter documents with category 'Luxury'...\n");
            SearchDocuments(indexClient, searchText: "*", filter: "category eq 'Luxury'");

            Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
            Console.ReadKey();
        }

        private static void DeleteHotelsIndexIfExists(SearchServiceClient serviceClient)
        {
            if (serviceClient.Indexes.Exists("hotels"))
            {
                serviceClient.Indexes.Delete("hotels");
            }
        }

        private static void CreateHotelsIndex(SearchServiceClient serviceClient)
        {
            var definition = new Index()
            {
                Name = "hotels",
                Fields = new[]
                {
                    new Field("hotelId", DataType.String)                       { IsKey = true },
                    new Field("hotelName", DataType.String)                     { IsSearchable = true, IsFilterable = true },
                    new Field("baseRate", DataType.Double)                      { IsFilterable = true, IsSortable = true },
                    new Field("category", DataType.String)                      { IsSearchable = true, IsFilterable = true, IsSortable = true, IsFacetable = true },
                    new Field("tags", DataType.Collection(DataType.String))     { IsSearchable = true, IsFilterable = true, IsFacetable = true },
                    new Field("parkingIncluded", DataType.Boolean)              { IsFilterable = true, IsFacetable = true },
                    new Field("lastRenovationDate", DataType.DateTimeOffset)    { IsFilterable = true, IsSortable = true, IsFacetable = true },
                    new Field("rating", DataType.Int32)                         { IsFilterable = true, IsSortable = true, IsFacetable = true },
                    new Field("location", DataType.GeographyPoint)              { IsFilterable = true, IsSortable = true }
                }
            };

            serviceClient.Indexes.Create(definition);
        }

        private static void UploadDocuments(ISearchIndexClient indexClient)
        {
            var documents =
                new Hotel[]
                {
                    new Hotel()
                    {
                        HotelId = "1058-441",
                        HotelName = "Fancy Stay",
                        BaseRate = 199.0,
                        Category = "Luxury",
                        Tags = new[] { "pool", "view", "concierge" },
                        ParkingIncluded = false,
                        LastRenovationDate = new DateTimeOffset(2010, 6, 27, 0, 0, 0, TimeSpan.Zero),
                        Rating = 5,
                        Location = GeographyPoint.Create(47.678581, -122.131577)
                    },
                    new Hotel()
                    {
                        HotelId = "666-437",
                        HotelName = "Roach Motel",
                        BaseRate = 79.99,
                        Category = "Budget",
                        Tags = new[] { "motel", "budget" },
                        ParkingIncluded = true,
                        LastRenovationDate = new DateTimeOffset(1982, 4, 28, 0, 0, 0, TimeSpan.Zero),
                        Rating = 1,
                        Location = GeographyPoint.Create(49.678581, -122.131577)
                    },
                    new Hotel()
                    {
                        HotelId = "970-501",
                        HotelName = "Econo-Stay",
                        BaseRate = 129.99,
                        Category = "Budget",
                        Tags = new[] { "pool", "budget" },
                        ParkingIncluded = true,
                        LastRenovationDate = new DateTimeOffset(1995, 7, 1, 0, 0, 0, TimeSpan.Zero),
                        Rating = 4,
                        Location = GeographyPoint.Create(46.678581, -122.131577)
                    },
                    new Hotel()
                    {
                        HotelId = "956-532",
                        HotelName = "Express Rooms",
                        BaseRate = 129.99,
                        Category = "Budget",
                        Tags = new[] { "wifi", "budget" },
                        ParkingIncluded = true,
                        LastRenovationDate = new DateTimeOffset(1995, 7, 1, 0, 0, 0, TimeSpan.Zero),
                        Rating = 4,
                        Location = GeographyPoint.Create(48.678581, -122.131577)
                    },
                    new Hotel()
                    {
                        HotelId = "566-518",
                        HotelName = "Surprisingly Expensive Suites",
                        BaseRate = 279.99,
                        Category = "Luxury",
                        ParkingIncluded = false
                    }
                };

            try
            {
                var batch = IndexBatch.Upload(documents);
                indexClient.Documents.Index(batch);
            }
            catch (IndexBatchException e)
            {
                // Sometimes when your Search service is under load, indexing will fail for some of the documents in
                // the batch. Depending on your application, you can take compensating actions like delaying and
                // retrying. For this simple demo, we just log the failed document keys and continue.
                Console.WriteLine(
                    "Failed to index some of the documents: {0}",
                    String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
            }

            // Wait a while for indexing to complete.
            Thread.Sleep(2000);
        }

        private static void SearchDocuments(ISearchIndexClient indexClient, string searchText, string filter = null)
        {
            // Execute search based on search text and optional filter
            var sp = new SearchParameters();

            if (!String.IsNullOrEmpty(filter))
            {
                sp.Filter = filter;
            }

            DocumentSearchResult<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
            foreach (SearchResult<Hotel> result in response.Results)
            {
                Console.WriteLine(result.Document);
            }
        }
    }
}
```

Hotel.cs:

```csharp
using System;
using Microsoft.Azure.Search.Models;
using Microsoft.Spatial;

namespace AzureSearch.SDKHowTo
{
    [SerializePropertyNamesAsCamelCase]
    public class Hotel
    {
        public string HotelId { get; set; }

        public string HotelName { get; set; }

        public double? BaseRate { get; set; }

        public string Category { get; set; }

        public string[] Tags { get; set; }

        public bool? ParkingIncluded { get; set; }

        public DateTimeOffset? LastRenovationDate { get; set; }

        public int? Rating { get; set; }

        public GeographyPoint Location { get; set; }

        public override string ToString()
        {
            return String.Format(
                "ID: {0}\tName: {1}\tCategory: {2}\tTags: [{3}]",
                HotelId,
                HotelName,
                Category,
                (Tags != null) ? String.Join(", ", Tags) : String.Empty);
        }
    }
}
```



<!--HONumber=Oct16_HO2-->


