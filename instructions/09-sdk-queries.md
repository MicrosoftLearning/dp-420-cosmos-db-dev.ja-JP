---
lab:
  title: Azure Cosmos DB for NoSQL SDK を使用してクエリを実行する
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Azure Cosmos DB for NoSQL SDK を使用してクエリを実行する

Azure Cosmos DB for NoSQL 用の .NET SDK の最新バージョンでは、C# の最新のベスト プラクティスと言語機能を使用して、コンテナーのクエリと結果セットの非同期反復処理がこれまで以上に簡単になります。

このライブラリには、[https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet] を使用して Azure Cosmos DB にクエリを実行しやすくするための特別な機能があります。

このラボでは、非同期ストリームを使用して、Azure Cosmos DB for NoSQL から返された大きな結果セットを反復処理します。 .NET SDK を使用して、結果のクエリと反復を行います。

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

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索して、新しい **Azure Cosmos DB forNoSQL** アカウント リソースを作成します。以下を設定して、残りの設定はすべて既定値のままにします。

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

    1. [**プライマリ接続文字列**] フィールドに注目します。 この**接続文字列**の値は、この演習で後ほど使用します。

1. **Visual Studio Code** に戻ります。

## Azure Cosmos DB for NoSQL アカウントにデータをシードする

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを使用して、Azure Cosmos DB for NoSQL アカウントにサンプル データをデプロイします。 このツールはオープンソースで、NuGet から入手できます。 このツールを Azure Cloud Shell にインストールして、データベースのシードに使用します。

1. **Visual Studio Code** で、 **[ターミナル]** メニューを開き、 **[新しいターミナル]** を選択して新しいターミナル インスタンスを開きます。

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

1. 統合ターミナルを閉じます。

## SDK を使用して SQL クエリの結果を反復処理する

ここでは、非同期ストリームを使用して、Azure Cosmos DB でのページ分割された結果に対して、わかりやすい foreach ループを作成します。 バックグラウンドでは、SDK でフィード反復子を管理し、後続の要求が正しく呼び出されるようにします。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**09-execute-query-sdk** フォルダーを参照します。

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

1. **script.cs** ファイルの末尾に新しいコードを追加して、*string* 型で値が **SELECT * FROM products p** である **sql** という新しい変数を作成しましょう。

    ```
    string sql = "SELECT * FROM products p";
    ```

1. コンストラクターへのパラメーターとして [sql][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition] 変数を渡す **QueryDefinition** 型の新しい変数を作成します。

    ```
    QueryDefinition query = new (sql);
    ```

1. [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer] クラスの汎用 [GetItemQueryIterator][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] メソッドを呼び出して、**query** 変数をパラメーターとして渡すことにより、新しい **while** ループを作成し、結果を反復処理します。

    ```
    using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
        queryDefinition: query
    );

    while (feed.HasMoreResults)
    {
    }
    ```

1. **while** ループ内で、次の結果を非同期で読み取り、組み込みの **Console.WriteLine** 静的メソッドを使用して、**product** 変数の **id**、**name**、**price** プロパティを書式設定して出力します

    ```
    FeedResponse<Product> response = await feed.ReadNextAsync();
    foreach (Product product in response)
    {
        Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    }
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    string sql = "SELECT * FROM products p";
    QueryDefinition query = new (sql);

    using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
        queryDefinition: query
    );

    while (feed.HasMoreResults)
    {
        FeedResponse<Product> response = await feed.ReadNextAsync();
        foreach (Product product in response)
        {
            Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
        }
    }
    ```

1. **script.cs** ファイルを**保存**します。

1. **Visual Studio Code** で、**09-execute-query-sdk** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0] パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. これで、スクリプトによって、コンテナー内のすべての製品が出力されます

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator
[learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet]: https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
