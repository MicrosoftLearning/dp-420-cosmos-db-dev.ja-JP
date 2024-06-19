---
lab:
  title: Azure Cosmos DB for NoSQL SDK を使用して外積クエリの結果をページ分割する
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Azure Cosmos DB for NoSQL SDK を使用して外積クエリの結果をページ分割する

Azure Cosmos DB クエリには、通常、複数の結果ページがあります。 改ページは、Azure Cosmos DB から 1 回の実行ですべてのクエリ結果を返せない場合に、自動的にサーバー側で行われます。 多くのアプリケーションでは、SDK を使用して、パフォーマンスの高い方法でクエリ結果を一括で処理するコードを記述する必要があります。

このラボでは、結果セット全体を反復処理するためにループで使用できるフィード反復子を作成します。

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

1. **Visual Studio Code** に戻ります。

## Azure Cosmos DB for NoSQL アカウントにデータをシードする

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを使用して、Azure Cosmos DB for NoSQL アカウントにサンプル データをデプロイします。 このツールはオープンソースで、NuGet から入手できます。 このツールを Azure Cloud Shell にインストールして、データベースのシードに使用します。

1. **Visual Studio Code** で、 **[ターミナル]** メニューを開き、 **[新しいターミナル]** を選択して新しいターミナル インスタンスを開きます。

1. コンピューターでグローバルに使用するために [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをインストールします。

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; このコマンドが完了するまで数分かかる場合があります。 過去にこのツールの最新バージョンを既にインストールしている場合は、このコマンドによって警告メッセージ (*ツール 'cosmicworks' は既にインストールされています) が出力されます。

1. cosmicworks を実行し、次のコマンドライン オプションを使用して Azure Cosmos DB アカウントをシードします。

    | **オプション** | **Value** |
    | ---: | :--- |
    | **--endpoint** | ''*このラボで先ほどコピーしたエンドポイントの値*'' |
    | **--key** | ''*このラボで先ほどコピーしたキーの値*'' |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; たとえば、エンドポイントが **https&shy;://dp420.documents.azure.com:443/** で、キーが **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドによって、データベース、コンテナー、および項目がアカウントに設定されるまで待ちます。

1. 統合ターミナルを閉じます。

## SDK を使用して SQL クエリの小さな結果セットで改ページする

クエリ結果を処理するときは、コードが結果のすべてのページを通過し、後続の要求を行う前にこれ以上ページが残っていないかどうかを確認する必要があります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**10-paginate-results-sdk** フォルダーを参照します。

1. **product.cs** コード ファイルを開きます。

1. **Product** クラスとその対応するプロパティを確認します。 具体的には、このラボでは **id**、**name**、および **price** プロパティが使用されます。

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

1. *string* 型の **sql** という名前の新しい変数を **SELECT p.id, p.name, p.price FROM products p** の値で作成します。

    ```
    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    ```

1. コンストラクターへのパラメーターとして [sql][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition] 変数を渡す **QueryDefinition** 型の新しい変数を作成します。

    ```
    QueryDefinition query = new (sql);
    ```

1. 既定の空のコンストラクターを使用して、**options** という名前の [QueryRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions] 型の新しい変数を作成します。

    ```
    QueryRequestOptions options = new ();
    ```

1. **options** 変数の [MaxItemCount][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount] プロパティを、**50** の値に設定します。

    ```
    options.MaxItemCount = 50;
    ```

1. **query** および **options** 変数をパラメーターとして渡す [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] クラスのジェネリック [GetItemQueryIterator][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator] メソッドを呼び出して、[FeedIterator<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1] 型の **iterator** という名前の新しい変数を作成します。

    ```
    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);
    ```

1. **iterator** 変数の [HasMoreResults][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults] プロパティを確認する **while** ループを作成します。

    ```
    while (iterator.HasMoreResults)
    {
        
    }
    ```

1. **while** ループ内で、**Product** クラスを使用するジェネリック型 [FeedResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1] の **products** という名前の変数に結果を格納する **iterator** 変数の [ReadNextAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync] メソッドを非同期的に呼び出します。

    ```
    FeedResponse<Product> products = await iterator.ReadNextAsync();
    ```

1. 引き続き **while** ループ内で、**Product** 型のインスタンスを表すために変数 **product** を使用する **products** 変数を反復処理し、新しい **foreach** ループを作成します。

    ```
    foreach (Product product in products)
    {

    }
    ```

1. **foreach** ループ内で、組み込みの **Console.WriteLine** 静的メソッドを使用して、**product** 変数の **id**、**name**、および **price** プロパティを書式設定して出力します。

    ```
    Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
    ```

1. **while** ループ内に戻り、組み込みの **Console.WriteLine** 静的メソッドを使用して、*Press any key to get more results* というメッセージを出力します。

    ```
    Console.WriteLine("Press any key to get more results");
    ```

1. 引き続き **while** ループ内で、組み込みの **Console.ReadKey** 静的メソッドを使用して、次の keypress 入力をリッスンします。

    ```
    Console.ReadKey();
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    QueryDefinition query = new (sql);

    QueryRequestOptions options = new ();
    options.MaxItemCount = 50;

    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);

    while (iterator.HasMoreResults)
    {
        FeedResponse<Product> products = await iterator.ReadNextAsync();
        foreach (Product product in products)
        {
            Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
        }

        Console.WriteLine("Press any key for next page of results");
        Console.ReadKey();        
    }
    ```

1. **script.cs** ファイルを**保存**します。

1. **Visual Studio Code** で、**10-paginate-results-sdk** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. これでスクリプトによって、クエリに一致する 50 項目の最初のセットが出力されるようになります。 任意のキーを押して、一致するすべての項目に対してクエリが反復処理されるまで、50 項目の次のセットを取得します。

    > &#128161; このクエリは、products コンテナー内の数百の項目と一致します。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
