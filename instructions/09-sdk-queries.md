---
lab:
  title: Azure Cosmos DB for NoSQL SDK を使用してクエリを実行する
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# <a name="execute-a-query-with-the-azure-cosmos-db-for-nosql-sdk"></a>Azure Cosmos DB for NoSQL SDK を使用してクエリを実行する

Azure Cosmos DB for NoSQL 用の .NET SDK の最新バージョンでは、コンテナーのクエリがこれまで以上に簡単になり、C# の最新のベスト プラクティスと言語機能を使用して結果セットを非同期的に反復できます。

> &#128161; このラボでは、NuGet の [Azure.Cosmos][nuget.org/packages/azure.cosmos/4.0.0-preview3] ライブラリの *4.0.0-preview3* リリースを使用します。 このライブラリには、[非同期ストリーム][docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams]を使用して Azure Cosmos DB にクエリを実行しやすくするための特別な機能があります。

このラボでは、非同期ストリームを使用して、Azure Cosmos DB for NoSQL から返された大きな結果セットを反復処理します。 .NET SDK を使用して、結果のクエリと反復を行います。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業する環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、これらの手順に従って行います。 それ以外の場合は、以前にクローンされたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]を参照してください。

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリがクローンされたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="create-an-azure-cosmos-db-for-nosql-account"></a>Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API (たとえば、**Mongo API** や **NoSQL API**) を選択します。 Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続する際に使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索して、新しい **Azure Cosmos DB forNoSQL** アカウント リソースを作成します。以下を設定して、残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | *既存のリソース グループを選択するか、新しいものを作成します* |
    | **アカウント名** | ''*グローバルに一意の名前を入力します*'' |
    | **場所** | *使用可能なリージョンを選びます* |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **[URI]** フィールドの値を記録します。 この**エンドポイント**の値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この**キー**の値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="seed-the-azure-cosmos-db-for-nosql-account-with-data"></a>Azure Cosmos DB for NoSQL アカウントにデータをシードする

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを使用して、Azure Cosmos DB for NoSQL アカウントにサンプル データをデプロイします。 このツールはオープンソースで、NuGet から入手できます。 このツールを Azure Cloud Shell にインストールして、データベースのシードに使用します。

1. **Visual Studio Code** で、 **[ターミナル]** メニューを開き、 **[新しいターミナル]** を選択して新しいターミナル インスタンスを開きます。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをご自分のコンピューターでグローバルに使用できるようにインストールします。

    ```
    dotnet tool install --global cosmicworks
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

## <a name="iterate-over-the-results-of-a-sql-query-using-the-sdk"></a>SDK を使用して SQL クエリの結果を反復処理する

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

1. *string* 型の **sql** という名前の新しい変数を **SELECT * FROM products p** の値で作成します。

    ```
    string sql = "SELECT * FROM products p";
    ```

1. コンストラクターへのパラメーターとして [sql][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition] 変数を渡す **QueryDefinition** 型の新しい変数を作成します。

    ```
    QueryDefinition query = new (sql);
    ```

1. [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer] クラスの汎用 [GetItemQueryIterator][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] メソッドを呼び出して、**query** 変数をパラメーターとして渡すことにより、新しい **await foreach** ループを作成します。次に、変数 **product** を使用して結果を非同期的に反復し、**Product** 型のインスタンスを表します。

    ```
    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
    }
    ```

1. **await foreach** ループ内で、組み込みの **Console.WriteLine** 静的メソッドを使用して、**product** 変数の **id**、**name**、および **price** プロパティを書式設定して出力します。

    ```
    Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    ```

1. 完了すると、コード ファイルに次の情報が表示されます。
  
    ```
    using System;
    using Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    CosmosDatabase database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    CosmosContainer container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT * FROM products p";
    QueryDefinition query = new (sql);

    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
        Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    }
    ```

1. **script.cs** ファイルを**保存**します。

1. **Visual Studio Code** で、**09-execute-query-sdk** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

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
[docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams]: https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/azure.cosmos/4.0.0-preview3]: https://www.nuget.org/packages/azure.cosmos/4.0.0-preview3
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
