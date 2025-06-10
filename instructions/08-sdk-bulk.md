---
lab:
  title: Azure Cosmos DB for NoSQL SDK を使用して複数のドキュメントを一括移動する
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Azure Cosmos DB for NoSQL SDK を使用して複数のドキュメントを一括移動する

一括操作を実行する方法を学習する最も簡単な方法は、クラウド内の Azure Cosmos DB for NoSQL アカウントに多くのドキュメントをプッシュしてみることです。 SDK の一括機能を使用すると、[System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks] 名前空間のわずかな助けを借りてこれを行うことができます。

このラボでは、NuGet の [Bogus][nuget.org/packages/bogus/33.1.1] ライブラリを使用して架空のデータを生成し、それを Azure Cosmos DB アカウントに配置します。

## 開発環境を準備する

このラボで作業している環境に **DP-420** のラボ コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[作業の開始に関するドキュメント][code.visualstudio.com/docs/getstarted]をご覧ください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## Azure Cosmos DB for NoSQL アカウントを作成し、SDK プロジェクトを構成する

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

1. 引き続き、新しく作成された **Azure Cosmos DB** アカウント リソース内で、**[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**[新しいコンテナー]** を選択してから、次の設定で新しいコンテナーを作成します。残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **データベース ID** | *新規作成* &vert; *`cosmicworks`* |
    | **コンテナー間でスループットを共有する** | *選択しない* |
    | **コンテナー ID** | *`products`* |
    | **パーティション キー** | *`/categoryId`* |
    | **コンテナーのスループット** | *自動スケーリング* &vert; *`4000`* |

1. **Visual Studio Code** に戻ります。

1. [**エクスプローラー**] ウィンドウで、**08-sdk-bulk** フォルダーを参照します。

1. **08-sdk-bulk** フォルダー内の **script.cs** コード ファイルを開きます。

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

1. **08-sdk-bulk** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **08-sdk-bulk** フォルダーに既に設定されているターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0] パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドします。

    ```
    dotnet build
    ```

1. 統合ターミナルを閉じます。

## 25,000 ドキュメントの一括挿入

"がんばって" 多くのドキュメントを挿入してみて、このしくみを確認しましょう。 内部テストでは、ラボ仮想マシンと Azure Cosmos DB for NoSQL アカウントが互いに地理的に比較的近い場合、これには約 1 分から 2 分かかる場合があります。

1. **script.cs** コード ファイルのエディター タブに戻ります。

1. **AllowBulkExecution** プロパティが **true** の値に設定された **options** クラスという名前の [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions] の新しいインスタンスを作成します。

    ```
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    ```

1. コンストラクター パラメーターとして **endpoint**、**key**、および **options** 変数を渡して、**client** という名前の **CosmosClient** クラスの新しいインスタンスを作成します。

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. **client** 変数の [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer] メソッドを使用し、データベースの名前 (*cosmicworks*) とコンテナーの名前 (*products*) を使って既存のコンテナーを取得します。

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. この特殊なサンプル コードを使用して、NuGet からインポートされた Bogus ライブラリの **Faker** クラスを使用して **25,000** の架空の製品を生成します。

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    > &#128161; [Bogus][nuget.org/packages/bogus/33.1.1] ライブラリは、ユーザー インターフェイス アプリケーションをテストするために架空のデータを設計するために使用されるオープンソース ライブラリであり、一括インポート/エクスポート アプリケーションの開発方法を学習するために最適です。

1. **concurrentTasks** という名前の **Task** 型の新しいジェネリック **List<>** を作成します。

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. このアプリケーションで以前に生成された製品の一覧を反復処理する foreach ループを作成します。

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. foreach ループ内に、Azure Cosmos DB for NoSQL に製品を非同期的に挿入する **Task** を作成します。必ず、パーティション キーを明示的に指定し、**concurrentTasks** という名前のタスクの一覧にタスクを追加してください。

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );   
    ```

1. foreach ループの後、**concurrentTasks** 変数の **Task.WhenAll** の結果を非同期的に待ちます。

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、**Bulk tasks complete** の静的メッセージをコンソールに出力します。

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Bogus;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    
    CosmosClient client = new (endpoint, key, options);  
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
        
    List<Task> concurrentTasks = new List<Task>();
    
    foreach(Product product in productsToInsert)
    {    
        concurrentTasks.Add(
            container.CreateItemAsync(product, new PartitionKey(product.categoryId))
        );
    }
    
    await Task.WhenAll(concurrentTasks);   

    Console.WriteLine("Bulk tasks complete");
    ```

1. **script.cs** コード ファイルを**保存**します。

1. **Visual Studio Code** で、**08-sdk-bulk** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. アプリケーションは自動的に実行されるはずです。実行されて自動的に完了するまで、約 1 分から 2 分かかります。

1. 統合ターミナルを閉じます。

## 結果を確認する

これで、Azure Cosmos DB に 25,000 項目を送信しました。次はデータ エクスプローラーを見てみましょう。

1.Web ブラウザーに戻り、[**データ エクスプローラー**] ウィンドウに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開して、**NOSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを確認します。

1. **products** ノードを展開してから、**[項目]** ノードを選択します。 コンテナー内の項目の一覧を確認します。

1. **NoSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選んでから、**[新しい SQL クエリ]** を選択します。

1. エディター領域のコンテンツを削除します。

1. 一括操作を使用して作成されたすべてのドキュメントの数を返す新しい SQL クエリを作成します。

    ```
    SELECT COUNT(1) FROM items
    ```

1. **[クエリの実行]** を選択します。

1. コンテナー内の項目の数を確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions
[docs.microsoft.com/dotnet/api/system.threading.tasks]: https://docs.microsoft.com/dotnet/api/system.threading.tasks
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/bogus/33.1.1]: https://www.nuget.org/packages/bogus/33.1.1
