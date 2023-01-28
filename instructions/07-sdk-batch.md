---
lab:
  title: Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をまとめてバッチ処理する
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# <a name="batch-multiple-point-operations-together-with-the-azure-cosmos-db-for-nosql-sdk"></a>Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をまとめてバッチ処理する

[TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] と [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] のクラスを一緒に使うことは、構成操作と分解操作を 1 つの論理ステップにまとめることができるため重要です。 これらのクラスを使用すると、複数の操作を実行するコードを記述し、サーバー側で正常に完了したかどうかを判断できます。

このラボでは、SDK を使用して、2 つの項目を 1 つの論理ユニットとして作成しようとする 2 つのデュアル項目操作を実行します。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業する環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、これらの手順に従って行います。 それ以外の場合は、以前にクローンされたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[作業の開始に関するドキュメント][code.visualstudio.com/docs/getstarted]をご覧ください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリがクローンされたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="create-an-azure-cosmos-db-for-nosql-account-and-configure-the-sdk-project"></a>Azure Cosmos DB for NoSQL アカウントを作成し、SDK プロジェクトを構成する

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索して、新しい **Azure Cosmos DB for NoSQL** アカウント リソースを作成します。以下を設定して、残りの設定はすべて既定値のままにします。

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

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**07-sdk-batch** フォルダーを参照します。

1. **07-sdk-batch** フォルダー内の **script.cs** コード ファイルを開きます。

    > &#128221; **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ライブラリは、既に NuGet から事前にインポートされています。

1. **endpoint** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの**エンドポイント**に設定します。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、ご自分のエンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの**キー**に設定します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、キーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **script.cs** コード ファイルを**保存**します。

1. **07-sdk-batch** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **07-sdk-batch** フォルダーに既に設定されているターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドします。

    ```
    dotnet build
    ```

1. 統合ターミナルを閉じます。

## <a name="creating-a-transactional-batch"></a>トランザクション バッチの作成

まず、2 つの架空の製品を作成する単純なトランザクション バッチを作成しましょう。 このバッチでは、同じ "used accessories" カテゴリ識別子を持つコンテナーに、worn saddle と rusty handlebar を挿入します。 どちらの項目にも同じ論理パーティション キーがあります。これにより、確実にバッチ操作が成功します。

1. **script.cs** コード ファイルのエディター タブに戻ります。

1. 一意識別子が **0120**、名前が **Worn Saddle**、およびカテゴリ識別子が **9603ca6c-9e28-4a02-9194-51cdb7fea816** の **saddle** という名前の **Product** 変数を作成します。

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 一意識別子が **012A**、名前が **Rusty Handlebar**、およびカテゴリ識別子が **9603ca6c-9e28-4a02-9194-51cdb7fea816** の **handlebar** という名前の **Product** 変数を作成します。

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. コンストラクター パラメーターとして **9603ca6c-9e28-4a02-9194-51cdb7fea816** を渡す **partitionKey** という名前の **PartitionKey** 型の変数を作成します。

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. メソッド パラメーターとして **partitionkey** 変数を渡す **container** 変数の [CreateTransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch] メソッドを呼び出し、fluent 構文を使用して、個々の操作で作成する項目として **saddle** および **handlebar** 変数を渡す [CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem] ジェネリック メソッドを呼び出し、結果を **TransactionalBatch** 型の **batch** という名前の変数に格納します。

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. using ステートメント内で、**batch** 変数の **ExecuteAsync** メソッドを非同期的に呼び出し、結果を **response** という名前の **TransactionalBatchResponse** 型の変数に格納します。

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出して、**response** 変数の **StatusCode** プロパティの値を出力します。

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 完了すると、コード ファイルに次の情報が表示されます。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **script.cs** コード ファイルを**保存**します。

1. **Visual Studio Code** で、**07-sdk-batch** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. ターミナルからの出力を確認します。 状態コードは HTTP 200 **OK** になるはずです。

1. 統合ターミナルを閉じます。

## <a name="creating-an-errant-transactional-batch"></a>誤ったトランザクション バッチの作成

次は、意図的にエラーを発生させるトランザクション バッチを作成しましょう。 このバッチでは、異なる論理パーティション キーを持つ 2 つの項目の挿入が試行されます。 "used accessories" カテゴリには flickering strobe light を、"pristine accessories" カテゴリには new helmet を作成します。 定義上、これは誤った要求であり、このトランザクションを実行するとエラーが返されるはずです。

1. **script.cs** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 一意識別子が **012B**、名前が **Flickering Strobe Light**、およびカテゴリ識別子が **9603ca6c-9e28-4a02-9194-51cdb7fea816** の **light** という名前の **Product** 変数を作成します。

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 一意識別子が **012C**、名前が **New Helmet**、およびカテゴリ識別子が **0feee2e4-687a-4d69-b64e-be36afc33e74** の **helmet** という名前の **Product** 変数を作成します。

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. コンストラクター パラメーターとして **9603ca6c-9e28-4a02-9194-51cdb7fea816** を渡す **partitionKey** という名前の **PartitionKey** 型の変数を作成します。

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. メソッド パラメーターとして **partitionkey** 変数を渡す **container** 変数の **CreateTransactionalBatch** メソッドを呼び出し、fluent 構文を使用して、個々の操作で作成する項目として **light** および **helmet** 変数を渡す **CreateItem<>** ジェネリック メソッドを呼び出し、結果を **TransactionalBatch** 型の **batch** という名前の変数に格納します。

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    ```

1. using ステートメント内で、**batch** 変数の **ExecuteAsync** メソッドを非同期的に呼び出し、結果を **response** という名前の **TransactionalBatchResponse** 型の変数に格納します。

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出して、**response** 変数の **StatusCode** プロパティの値を出力します。

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 完了すると、コード ファイルに次の情報が表示されます。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **script.cs** コード ファイルを**保存**します。

1. **Visual Studio Code** で、**07-sdk-batch** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. ターミナルからの出力を確認します。 状態コードは、HTTP 400 **正しくない要求**または 409 **競合**のいずれかになるはずです。 これは、トランザクション内のすべての項目で、トランザクション バッチと同じパーティション キー値が共有されなかったため発生しました。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
