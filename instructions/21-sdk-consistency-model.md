---
lab:
  title: ポータルと Azure Cosmos DB SQL API SDK で整合性モデルを構成する
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: 280f43ff34be1d12ff9767531d6909743678d53e
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024964"
---
# <a name="configure-consistency-models-in-the-portal-and-the-azure-cosmos-db-sql-api-sdk"></a>ポータルと Azure Cosmos DB SQL API SDK で整合性モデルを構成する

新しい Azure Cosmos DB SQL API アカウントの既定の整合性レベルは、セッションの整合性です。 この既定の設定は、今後のすべての要求に対して変更できます。 個々の要求レベルでは、さらに一歩進んで、その特定の要求の整合性レベルを緩和できます。

このラボでは、Azure Cosmos DB SQL API アカウントのデフォルトの整合性レベルを構成してから、SDK を使用して個々の操作の整合性レベルを構成します。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業している環境に **DP-420** のラボ コードのリポジトリをまだ複製していない場合は、次の手順に従って複製します。 それ以外の場合は、以前に複製されたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[作業の開始に関するドキュメント][code.visualstudio.com/docs/getstarted]をご覧ください

1. コマンド パレットを開き、**Git: Clone** を実行して、選択したローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを複製します。

    > &#128161; **CTRL + SHIFT + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API を選択します (たとえば、**Mongo API** または **SQL API**)。 Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB SQL API アカウントに接続する場合にそれらを使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、 *[Cosmos DB]* を検索してから、次の設定で新しい **[Azure Cosmos DB SQL API]** アカウント リソースを作成し、残りのすべての設定を規定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | *既存の Azure サブスクリプション* |
    | **リソース グループ** | *既存のリソース グループを選択するか、新しいものを作成します* |
    | **アカウント名** | *グローバルに一意の名前を入力します* |
    | **Location** | *使用可能な任意のリージョンを選択します* |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **グローバル配布** &vert; **Geo 冗長性** | *有効にする* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースにアクセスし、 **[データをグローバルにレプリケートする]** ペインに移動します。

1. **[データをグローバルにレプリケートする]** ペインで、アカウントにさらに 2 つの読み取りリージョンを追加し、変更内容を **保存** します。

1. このタスクを続行する前に、レプリケーション タスクが完了するのを待ちます。

    > &#128221; この操作には約 5 〜 10 分かかる場合があり、 **[既定の整合性]** ウィンドウに移動します。

1. リソース ブレードで、 **[既定の整合性]** ウィンドウに移動します。

1. **[既定の整合性]** ウィンドウで、 **[強固]** オプションを選択し、変更を **[保存]** します。

1. このタスクを続行する前に、既定の整合性レベルの変更が保持されるまで待ちます。

1. リソース ブレードで、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** ペインで、 **[新しいコンテナー]** を選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | *新しい* &vert; *cosmicworks を作成する* |
    | **コンテナー間でスループットを共有する** | *選択しないでください* |
    | **コンテナー ID** | *製品* |
    | **パーティション キー** | */categoryId* |
    | **コンテナーのスループット** | *手動* &vert; *400* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認します。

1. **[データ エクスプローラー]** ペインで、**cosmicworks** データベース ノード、**products** コンテナー ノードの順に展開してから、 **[項目]** を選択します。

1. **[データ エクスプローラー]** ペインのまま、コマンド バーから **[新しい項目]** を選択します。 エディターで、プレースホルダーの JSON 項目を次の内容に置き換えます。

    ```
    {
      "id": "7d9273d9-5d91-404c-bb2d-126abb6e4833",
      "categoryId": "78d204a2-7d64-4f4a-ac29-9bfc437ae959",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. コマンド バーから **[保存]** を選択して、JSON 項目を追加します。

1. **[項目]** タブで、 **[項目]** ペインの新しい項目を確認します。

1. リソース ブレードで、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **[URI]** フィールドの値を記録します。 この **endpoint** の値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この **キー** の値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="connect-to-the-azure-cosmos-db-sql-api-account-from-the-sdk"></a>SDK から Azure Cosmos DB SQL API アカウントに接続する

新しく作成したアカウントの資格情報を使用して、SDK クラスに接続し、新しいデータベースとコンテナー インスタンスを作成します。 次に、データ エクスプローラーを使用して、Azure portal でインスタンスが存在することを検証します。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**21-sdk-consistency-model** フォルダーを参照します。

1. **21-sdk-consistency-model** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **21-sdk-consistency-model** フォルダーに既に設定されているターミナルが開きます。

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドします。

    ```
    dotnet build
    ```

    > &#128221; コンパイラから、**endpoint** および **key** 変数は現在使用されていないという警告が表示される場合があります。 これらの変数はこのタスクで使用しますので、この警告は無視しても問題ありません。

1. 統合ターミナルを閉じます。

1. **product.cs** コード ファイルを開きます。

1. **Product** レコードとその対応するプロパティを確認します。 具体的には、このラボでは **id**、**name**、および **categoryId** プロパティを使用します。

1. **Visual Studio Code** の **[エクスプローラー]** ペインに戻り、**script.cs** コード ファイルを開きます。

    > &#128221; **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ライブラリは、既に NuGet から事前にインポートされています。

1. **endpoint** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの **エンドポイント** に設定します。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、エンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの **キー** に設定します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、キーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **script.cs** コード ファイルを **保存** します。

## <a name="configure-consistency-level-for-a-point-operation"></a>ポイント操作の整合性レベルを構成する

**ItemRequestOptions** クラスには、要求ごとの構成プロパティが含まれています。 このクラスを使用すると、整合性レベルを現在の既定である強固な整合性から最終的な整合性に緩和できます。

1. **id** という名前の文字列変数を、値 **7d9273d9-5d91-404c-bb2d-126abb6e4833** で作成します。

    ```
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    ```

1. **categoryId** という名前の文字列変数を、値 **78d204a2-7d64-4f4a-ac29-9bfc437ae959** で作成します。

    ```
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    ```

1. コンストラクター パラメーターとして **categoryId** 変数を渡して、**partitionKey** という名前の **PartitionKey** 型の変数を作成します。

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. メソッド パラメーターとして **id** および **partitionkey** 変数を渡し、ジェネリック型として **Product** を使用して、**container** 変数の **ReadItemAsync\<\>** 汎用メソッドを非同期に呼び出し、結果を **ItemResponse\<Product\>** 型の **response** という名前の変数に格納します。

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. **Console.WriteLine** 静的メソッドを呼び出して、書式付きの出力文字列を使用して、要求料金を出力します。

    ```
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 完了すると、コード ファイルに次のものが含まれるはずです。

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    using CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**21-sdk-consistency-model** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. ターミナルからの出力を確認します。 要求料金 (RU 単位) をコンソールに出力する必要があります。

    > &#128221; 現在の要求料金は **2 RUs** になっているはずです。 これは、強力な一貫性では、最新の書き込みが行われるように、少なくとも 2 つのレプリカからの読み取りが必要であるためです。

1. 統合ターミナルを閉じます。

1. **script.cs** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Itemrequestoptions** 型の [options][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions] という名前の新しい変数を作成し、[ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel] プロパティを [ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel] 列挙値に設定します。

    ```
    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    ```

1. メソッド パラメーターとして **id**、**partitionkey**、および **options** 変数を渡し、ジェネリック型として **Product** を使用して、**container** 変数の **ReadItemAsync\<\>** 汎用メソッドを非同期に呼び出し、結果を **ItemResponse\<Product\>** 型の **response** という名前の変数に格納します。

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    ```

1. **Console.WriteLine** 静的メソッドを呼び出して、書式付きの出力文字列を使用して、要求料金を出力します。

    ```
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 完了すると、コード ファイルに次のものが含まれるはずです。

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    using CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**21-sdk-consistency-model** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. ターミナルからの出力を確認します。 要求料金 (RU 単位) をコンソールに出力する必要があります。

    > &#128221; 現在の要求料金は **1 RUs** になっているはずです。 これは、最終的な整合性によって、1 つのレプリカからの読み取りのみが必要になるためです。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
