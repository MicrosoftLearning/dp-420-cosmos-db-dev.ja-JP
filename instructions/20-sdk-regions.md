---
lab:
  title: Azure Cosmos DB SQL API SDK を使用して異なるリージョンに接続する
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: 758a51237ee4c8b4e4eb173addb1e66fbcafbe9e
ms.sourcegitcommit: 694767b3c7933a8ee84beca79da880d5874486bc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2022
ms.locfileid: "139057400"
---
# <a name="connect-to-different-regions-with-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK を使用して異なるリージョンに接続する

Azure Cosmos DB SQL API アカウントの geo 冗長性を有効にすると、SDK を使用して、構成した任意の順序でリージョンからデータを読み取ることができます。 この手法は、読み取り要求を、使用可能なすべての読み取りリージョンにわたって配布する場合に役立ちます。

このラボでは、手動で構成するフォールバックの順序で読み取りリージョンに接続するように CosmosClient クラスを構成します。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業する環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、これらの手順に従って行います。 それ以外の場合は、以前にクローンされたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[作業の開始に関するドキュメント][code.visualstudio.com/docs/getstarted]をご覧ください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API を選択します (たとえば、**Mongo API** または **SQL API**)。 Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB SQL API アカウントに接続する場合にそれらを使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成し、残りのすべての設定を既定値のままにします。

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

1. 新しく作成した **Azure Cosmos DB** アカウント リソースにアクセスし、 **[データをグローバルにレプリケートする]** ペインに移動します。

1. **[データをグローバルにレプリケートする]** ペインで、アカウントにさらに 2 つの読み取りリージョンを追加し、変更内容を **保存** します。

1. このタスクを続行する前に、レプリケーション タスクが完了するのを待ちます。

    > &#128221; この操作には、5 から 10 分ほどかかる場合があります。

1. **書き込み** (プライマリ) リージョンと 2 つの **読み取り** リージョンの名前を記録します。 これらのリージョン名は、この演習で後ほど使用します。

    > &#128221; たとえば、プライマリ リージョンが **北ヨーロッパ** で、2 つの読み取りセカンダリ リージョンが **米国東部 2** と **南アフリカ北部** の場合は、この 3 つの名前をすべてそのまま記録します。

1. リソース ブレードで、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** ペインで、 **[新しいコンテナー]** を選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | 新しい &vert; *cosmicworks* *を作成する* |
    | **コンテナー間でスループットを共有する** | *選択しない* |
    | **コンテナー ID** | *製品* |
    | **パーティション キー** | */categoryId* |
    | **コンテナーのスループット** | *手動* &vert; *400* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開し、階層内の **products** コンテナー ノードを確認します。

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

    1. **[URI]** フィールドの値を記録します。 この **エンドポイント** の値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この **キー** の値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="connect-to-the-azure-cosmos-db-sql-api-account-from-the-sdk"></a>SDK から Azure Cosmos DB SQL API アカウントに接続する

新しく作成したアカウントの資格情報を使用して、SDK クラスに接続し、異なるリージョンのデータベースとコンテナー インスタンスにアクセスします。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**20-sdk-regions** フォルダーを参照します。

1. **20-sdk-regions** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、ターミナルが開き、開始ディレクトリが既に **20-sdk-regions** フォルダーに設定されています。

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドします。

    ```
    dotnet build
    ```

    > &#128221; コンパイラから、**endpoint** および **key** 変数は現在使用されていないという警告が表示される場合があります。 これらの変数はこのタスクで使用しますので、この警告は無視しても問題ありません。

1. 統合ターミナルを閉じます。

1. **20-sdk-regions** フォルダー内の **script.cs** コード ファイルを開きます。

    > &#128221; **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ライブラリは、既に NuGet から事前にインポートされています。

1. **endpoint** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの **エンドポイント** に設定します。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、ご自分のエンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの **キー** に設定します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、ご自分のキーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **script.cs** コード ファイルを **保存** します。

## <a name="configure-the-net-sdk-with-a-preferred-region-list"></a>優先リージョン リストを使用して .NET SDK を構成する

**CosmosClientOptions** クラスには、SDK を使用して接続するリージョンのリストを構成するためのプロパティが用意されています。 リストは、構成した順序で各リージョンへの接続を試行するように、フェールオーバー優先度順に並べます。

1. **List\<string\>** ジェネリック型の新しい変数を作成して、アカウントで構成したリージョンのリストを 3 番目のリージョンから開始し、1 番目の (プライマリ) リージョンで終了するように含めます。 たとえば、**米国西部** リージョンに Azure Cosmos DB SQL API アカウントを作成し、**南アフリカ北部** を追加し、最後に **東アジア** を追加した場合、リスト変数は次のようにします。

    ```
    List<string> regions = new()
    {
        "East Asia",
        "South Africa North",
        "West US"
    };
    ```

    > &#128161; 代わりに、[Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] 静的クラスを使用して、異なる Azure リージョンの組み込みの文字列プロパティを含めることもできます。

1. **options** という名前の **CosmosClientOptions** クラスの新しいインスタンスを作成し、[ApplicationPreferredRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions] プロパティを **region** 変数に設定します。

    ```
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
        , RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };
    ```

1. コンストラクター パラメーターとして **endpoint**、**key**、および **options** 変数を渡して、**client** という名前の **CosmosClient** クラスの新しいインスタンスを作成します。

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. **client** 変数の **GetContainer** メソッドを使用し、データベースの名前 (*cosmicworks*) とコンテナーの名前 (*products*) を指定して、既存のコンテナーを取得します。

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. **container** 変数の [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] メソッドを使用して、サーバーから特定の項目を取得し、Null 許容の [ItemResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse] 型の **response** という名前の変数に結果を格納します。

    ```
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    ```

1. **Console.WriteLine** 静的メソッドを呼び出して、現在の項目識別子と JSON 診断データを出力します。

    ```
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine($"Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    List<string> regions = new()
    {
        "<read-region-2>",
        "<read-region-1>",
        "<write-region>"
    };
    
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
        , RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };
    
    using CosmosClient client = new(endpoint, key, options);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine("Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**20-sdk-regions** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. ターミナルからの出力を確認します。 コンソール出力にコンテナーの名前と JSON 診断データが出力されているはずです。

1. JSON 診断データを確認します。 **HttpResponseStats** という名前のプロパティと **RequestUri** という名前の子プロパティを検索します。 このプロパティの値は、このラボで先ほど構成した名前とリージョンが含まれる URI のはずです。

    > &#128221; たとえば、アカウント名が **dp420** で、最初に構成したリージョンが **東アジア** の場合、JSON ロパティの値は、**dp420-eastasia.documents.azure.com/dbs/cosmicworks/colls/products** になります。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
