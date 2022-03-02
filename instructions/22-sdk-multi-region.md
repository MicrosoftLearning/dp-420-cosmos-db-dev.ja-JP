---
lab:
  title: Azure Cosmos DB SQL API SDK を使用して複数リージョンの書き込みアカウントに接続する
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: bc9f23e136b5987fb55c386485916c701d016fd1
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025084"
---
# <a name="connect-to-a-multi-region-write-account-with-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK を使用して複数リージョンの書き込みアカウントに接続する

**CosmosClientBuilder** クラスは、コンテナーに接続して操作を実行する SDK クライアントをビルドするように設計されたフルーエント クラスです。 Azure Cosmos DB SQL API アカウントが複数リージョンの書き込み用に既に構成されている場合は、ビルダーを使用して、書き込み操作用に優先アプリケーション リージョンを構成できます。

このラボでは、リージョンが複数の Azure Cosmos DB SQL API アカウントを構成し、複数リージョンの書き込みを有効にします。 その後、SDK を使用して、特定のリージョンに対して操作を実行します。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業している環境に **DP-420** のラボ コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[作業の開始に関するドキュメント][code.visualstudio.com/docs/getstarted]をご覧ください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; コマンド パレットは、**Ctrl + Shift + P** キーボード ショートカットを使用して開くことができます。

1. リポジトリをクローンしたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API (たとえば、**Mongo API** や **SQL API**) を選択します。 Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB SQL API アカウントに接続する際に使用できます。

1. Web ブラウザーの新しいウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索して、新しい **Azure Cosmos DB SQL API** アカウント リソースを作成します。以下を設定して、残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | *既存の Azure サブスクリプション* |
    | **リソース グループ** | *既存のリソース グループを選択するか、新規作成する* |
    | **アカウント名** | *グローバルに一意の名前を入力する* |
    | **Location** | *使用可能な任意のリージョンを選択する* |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; お使いのラボ環境では、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. このタスクを続行する前に、デプロイ タスクが完了するのを待ちます。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースにアクセスし、 **[データをグローバルにレプリケートする]** ペインに移動します。

1. **[データをグローバルにレプリケートする]** ペインで、さらに少なくとも 1 つのリージョンをアカウントに追加します。

1. **[データをグローバルにレプリケートする]** ペインのまま、**複数リージョンの書き込み** を有効にしてから、変更内容を **保存** します。

1. このタスクを続行する前に、レプリケーション タスクが完了するのを待ちます。

    > &#128221; この操作には、5 から 10 分ほどかかる場合があります。

1. 作成した追加リージョンの少なくとも 1 つの値を記録します。 このリージョン値は、この演習で後ほど使用します。

1. リソース ブレードで、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** ペインで、 **[新しいコンテナー]** を選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | 新しい &vert; *cosmicworks* *を作成する* |
    | **コンテナー間でスループットを共有する** | *選択しない* |
    | **コンテナー ID** | *products* |
    | **パーティション キー** | */categoryId* |
    | **コンテナーのスループット** | *手動* &vert; *400* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開し、階層内の **products** コンテナー ノードを確認します。

1. リソース ブレードで、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が表示されます。 具体的な内容は次のとおりです。

    1. **[URI]** フィールドの値を記録します。 この **エンドポイント** 値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この **キー** 値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="connect-to-the-azure-cosmos-db-sql-api-account-from-the-sdk"></a>SDK から Azure Cosmos DB SQL API アカウントに接続する

新しく作成したアカウントの資格情報を使用して、SDK クラスに接続し、新しいデータベースとコンテナー インスタンスを作成します。 次に、データ エクスプローラーを使用して、Azure portal でインスタンスが存在することを検証します。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**22-sdk-multi-region** フォルダーを参照します。

1. **22-sdk-multi-region** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、ターミナルが開き、開始ディレクトリが既に **22-sdk-multi-region** フォルダーに設定されています。

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

    > &#128221; たとえば、ご自分のエンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの **キー** に設定します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、ご自分のキーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **script.cs** コード ファイルを **保存** します。

## <a name="configure-write-region-for-the-sdk"></a>SDK の書き込みリージョンを構成する

フルーエントな **WithApplicationRegion** メソッドを使用して、ビルダー クラスを使用して後続の操作の優先リージョンを構成します。

1. コンストラクター パラメーターとして **endpoint** および **key** 変数を渡して、**builder** という名前の [CosmosClientBuilder][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder] クラスの新しいインスタンスを作成します。

    ```
    CosmosClientBuilder builder = new (endpoint, key);
    ```

1. ラボで先ほど作成した追加リージョンの名前を使用して、**string** 型の **region** という名前の新しい変数を作成します。 たとえば、**米国東部** リージョンに Azure Cosmos DB SQL API アカウントを作成し、**ブラジル南部** を追加した場合は、文字列変数に以下を含めます。

    ```
    string region = "Brazil South"; 
    ```

    > &#128161; 代わりに、[Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] 静的クラスを使用して、異なる Azure リージョンの組み込みの文字列プロパティを含めることもできます。

1. [WithApplicationRegion][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion] メソッドを **region** のパラメーターを指定して呼び出し、**builder** 変数で [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build] メソッドをフルーエントに呼び出し、using ステートメント内にカプセル化された **CosmosClient** 型の **client** という名前の変数に結果を格納します。

    ```
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    ```

1. **client** 変数の **GetContainer** メソッドを使用し、データベースの名前 (*cosmicworks*) とコンテナーの名前 (*products*) を指定して、既存のコンテナーを取得します。

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. 新しい **Guid** 値を生成し、**id** と **categoryId** という名前の 2 つの **string** 変数を作成し、結果を文字列として格納します。

    ```
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    ```

1. コンストラクター パラメーターとして **id** 変数、**Polished Bike Frame** の文字列値、**categoryId** 変数を渡して、**Product** 型の **item** という名前の新しい変数を作成します。

    ```
    Product item = new (id, "Polished Bike Frame", categoryId);
    ```

1. パラメーターとして **item** 変数を渡して、**container** 変数の **CreateItemAsync\<\>** メソッドを非同期に呼び出し、結果を **response** という名前の変数に格納します。

    ```
    var response = await container.CreateItemAsync<Product>(item);
    ```

1. **Console.WriteLine** 静的メソッドを呼び出して、応答の HTTP 状態コードと要求料金 (要求ユニット単位) を出力します。

    ```
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. 完了すると、コード ファイルが次のようになるはずです。

    ```
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Fluent;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";    

    CosmosClientBuilder builder = new (endpoint, key);            
    
    string region = "West Europe";
    
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    Product item = new (id, "Polished Bike Frame", categoryId);
    
    var response = await container.CreateItemAsync<Product>(item);
    
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**22-sdk-multi-region** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. ターミナルからの出力を確認します。 HTTP 状態コードと要求料金 (RU 単位) がコンソールに出力されているはずです。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
