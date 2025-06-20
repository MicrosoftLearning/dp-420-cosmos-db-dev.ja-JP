---
lab:
  title: Azure Cosmos DB for NoSQL SDK を使用して変更フィード イベントを処理する
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Azure Cosmos DB for NoSQL SDK を使用して変更フィード イベントを処理する

Azure Cosmos DB for NoSQL の変更フィードは、プラットフォームからのイベントによって駆動される補足アプリケーションを作成するためのカギとなります。 Azure Cosmos DB for NoSQL 用の .NET SDK には、変更フィードと統合し、コンテナー内の操作に関する通知をリッスンするアプリケーションを構築するためのクラス一式が用意されています。

このラボでは、.NET SDK の変更フィード プロセッサ機能を使用して、指定したコンテナー内の項目で作成または更新操作が実行されたという通知を受け取るアプリケーションを作成します。

## 開発環境を準備する

このラボで作業している環境に **DP-420** のラボ コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]を参照してください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API (たとえば、**Mongo API** や **NoSQL API**) を選択します。 Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続する際に使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索して、新しい **Azure Cosmos DB for NoSQL** アカウント リソースを作成します。以下を設定して、残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **ワークロードの種類** | **学習** |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **アカウント名** | ''*グローバルに一意の名前を入力します*'' |
    | **場所** | ''*使用可能なリージョンを選びます*'' |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. [**URI**] フィールドに注目します。 この**エンドポイント**の値は、この演習で後ほど使用します。

    1. [**主キー**] フィールドに注目してください。 この**キー**の値は、この演習で後ほど使用します。

    1. [**プライマリ接続文字列**] フィールドに注目します。 この**接続文字列**の値は、この演習で後ほど使用します。

1. リソース メニューで **[データ エクスプローラー]** を選択します。

1. **[データ エクスプローラー]** ペインで、**[新しいコンテナー]** を展開してから、**[新しいデータベース]** を選択します。

1. **[新しいデータベース]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | *``cosmicworks``* |
    | **スループットのプロビジョニング** | 有効 |
    | **データベースのスループット** | **[手動]** |
    | **データベースに必要な RU/秒** | ``1000`` |

1. **[データ エクスプローラー]** ペインに戻り、階層内の **cosmicworks** データベース ノードを確認します。

1. **[データ エクスプローラー]** ペインで、 **[新しいコンテナー]** を選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | ''*既存の* &vert; *cosmicworks を使用します*'' |
    | **コンテナー ID** | *``products``* |
    | **パーティション キー** | *``/category/name``* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認します。

1. **[データ エクスプローラー]** ペインで、**[新しいコンテナー]** を再度選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | ''*既存の* &vert; *cosmicworks を使用します*'' |
    | **コンテナー ID** | *``productslease``* |
    | **パーティション キー** | *``/partitionKey``* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開し、階層内の **productslease** コンテナー ノードを確認します。

1. **Visual Studio Code** に戻ります。

## .NET SDK で変更フィード プロセッサを実装する

**Microsoft.Azure.Cosmos.Container** クラスには、変更フィード プロセッサをフルーエントにビルドするための一連のメソッドが用意されています。 開始するには、監視対象のコンテナーへの参照、リース コンテナー、C\# のデリゲート (変更の各バッチを処理するため) が必要です。

1. [**エクスプローラー**] ウィンドウで、**13-change-feed** フォルダーを参照します。

1. **product.cs** コード ファイルを開きます。

1. **Product** クラスとその対応するプロパティを確認します。 具体的には、このラボでは **id** および **name** プロパティを使用します。

1. **Visual Studio Code** の **[エクスプローラー]** ペインに戻り、**script.cs** コード ファイルを開きます。

1. **endpoint** という名前の既存の変数を、先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定された値で更新します。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、ご自分のエンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の既存の変数を、先ほど作成した Azure Cosmos DB アカウントの **key** に設定された値で更新します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、キーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **client** 変数の **GetContainer** メソッドを使用して、データベースの名前 (*cosmicworks*) とコンテナーの名前 (*products*) を使用して既存のコンテナーを取得し、結果を **Container** 型の **sourceContainer** という名前の変数に格納します。

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. **client** 変数の **GetContainer** メソッドを使用して、データベースの名前 (*cosmicworks*) とコンテナーの名前 (*productslease*) を使用して既存のコンテナーを取得し、結果を **Container** 型の **leaseContainer** という名前の変数に格納します。

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. 2 つの入力パラメーターを持つ空の非同期匿名関数を使用して、[ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1] 型の **handleChanges** という名前の新しいデリゲート変数を作成します。

    1. **IReadOnlyCollection\<Product\>** 型の **changes** という名前のパラメーター。
    
    1. **CancellationToken** 型の **cancellationToken** という名前のパラメーター。

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. 匿名関数内で、組み込みの **Console.WriteLine** 静的メソッドを使用して、生文字列 **START\tHandling batch of changes...** を出力します。

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. 引き続き、匿名関数内で、変数 **product** を使用して **changes** 変数を反復処理する foreach ループを作成して、**Product** 型のインスタンスを表します。

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. 匿名関数の foreach ループ内で、組み込みの非同期 **Console.WriteLineAsync** 静的メソッドを使用して、**product** 変数の **id** および **name** プロパティを出力します。

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. foreach ループおよび匿名関数の外部で、次のパラメーターを使用して **sourceContainer** 変数で [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] を呼び出した結果を格納する **builder** という名前の新しい変数を作成します。

    | **パラメーター** | **Value** |
    | ---: | :--- |
    | **processorName** | *productsProcessor* |
    | **onChangesDelegate** | *handleChanges* |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. [WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename] メソッドを **consoleApp** のパラメーターを指定して呼び出し、[WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer] メソッドを **leaseContainer** のパラメーターを指定して呼び出し、[Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] メソッドを **builder** 変数でフルーエントに呼び出し、結果を [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor] 型の **processor** という名前の変数に格納します。

    ```
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    ```

1. **processor** 変数の [StartAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync] を非同期に呼び出します。

    ```
    await processor.StartAsync();
    ```

1. 組み込みの **Console.WriteLine** および **Console.ReadKey** 静的メソッドを使用して、コンソールに出力し、アプリケーションにキーの押下を待機させます。

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();  
    ```

1. **processor** 変数の [StopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync] を非同期に呼び出します。

    ```
    await processor.StopAsync();
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using Microsoft.Azure.Cosmos;
    using static Microsoft.Azure.Cosmos.Container;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
        Console.WriteLine($"START\tHandling batch of changes...");
        foreach(Product product in changes)
        {
            await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
        }
    };
    
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
            processorName: "productsProcessor",
            onChangesDelegate: handleChanges
        );
    
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    await processor.StartAsync();
    
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    
    await processor.StopAsync();
    ```

1. **script.cs** ファイルを**保存**します。

1. **Visual Studio Code** で、**13-change-feed** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. **Visual Studio Code** とターミナルの両方を開いたままにします。

    > &#128221; 別のツールを使用して、Azure Cosmos DB for NoSQL コンテナーに項目を生成します。 項目を生成したら、このターミナルに戻って出力を確認します。 ターミナルを早々に閉じないようにします。

## Azure Cosmos DB for NoSQL アカウントにサンプル データをシードする

**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。 ツールで一連の項目を作成して、ターミナル ウィンドウで実行されている変更フィード プロセッサを使用して確認します。

1. **Visual Studio Code**で、**[ターミナル]** メニューを開き、**[ターミナルの分割]** を選択して、新しいターミナルを既存のインスタンスと並べて開きます。

1. コンピューターでグローバルに使用するために [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをインストールします。

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```

    > &#128161; このコマンドが完了するまで数分かかる場合があります。 過去にこのツールの最新バージョンを既にインストールしている場合は、このコマンドによって警告メッセージ (*ツール 'cosmicworks' は既にインストールされています) が出力されます。

1. cosmicworks を実行し、次のコマンドライン オプションを使用して Azure Cosmos DB アカウントをシードします。

    | **オプション** | **Value** |
    | ---: | :--- |
    | **-c** | *このラボで先ほど確認した接続文字列の値* |
    | **--number-of-employees** | *特に指定がない限り、cosmicworks コマンドは、従業員 1000 人と製品コンテナー 200 項目をデータベースに入力します* |

    ```powershell
    cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```

    > &#128221; たとえば、エンドポイントが **https&shy;://dp420.documents.azure.com:443/** で、キーが **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。``cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys``

1. **cosmicworks** コマンドによって、データベース、コンテナー、および項目がアカウントに設定されるまで待ちます。

1. .NET アプリケーションのターミナル出力を確認します。 ターミナルには、変更フィードを使用して送信された変更ごとに、**[検出された操作]** というメッセージが出力されます。

1. 両方の統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
