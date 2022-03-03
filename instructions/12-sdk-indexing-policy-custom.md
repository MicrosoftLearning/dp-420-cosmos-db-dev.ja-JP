---
lab:
  title: ポータルで Azure Cosmos DB SQL API コンテナーのインデックス ポリシーを構成する
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: 0a4f1118d4a2f726df12dbeec1d3d96918450e12
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024955"
---
# <a name="configure-an-azure-cosmos-db-sql-api-containers-index-policy-using-the-sdk"></a>SDK を使用して Azure Cosmos DB SQL API コンテナーのインデックス ポリシーを構成する

インデックス作成ポリシーは、任意の Azure Cosmos DB SDK から管理できます。 特に、.NET SDK には、Azure Cosmos DB SQL API のコンテナーに新しいインデックス ポリシーを設計およびプッシュするために使用できる一連のクラスが含まれています。

このラボでは、.NET SDK を使用して、コンテナーのカスタム インデックス作成ポリシーを作成します。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業している環境に **DP-420** のラボ コードのリポジトリをまだ複製していない場合は、次の手順に従って複製します。 それ以外の場合は、以前に複製されたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスの詳細をまだ十分理解していない場合は、「[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]」を参照してください。

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
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **[URI]** フィールドの値を記録します。 この **endpoint** の値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この **キー** の値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="create-a-new-indexing-policy-using-the-net-sdk"></a>.NET SDK を使用して新しいインデックス作成ポリシーを作成する

.NET SDK には、コードで新しいインデックスポリシーを構築するために、親 [Microsoft.Azure.Cosmos.IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] クラスに関連する一連のクラスが含まれています。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**12-custom-index-policy** フォルダーを参照します。

1. **script.cs** コード ファイルを開きます。

1. **endpoint** という名前の既存の変数を、先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定された値で更新します。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、エンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の既存の変数を、先ほど作成した Azure Cosmos DB アカウントの **key** に設定された値で更新します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、キーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. 既定の空のコンストラクターを使用して、**policy** という名前の型 [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] の新しい変数を作成します。

    ```
    IndexingPolicy policy = new ();
    ```

1. **policy** 変数の [IndexingMode][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode] プロパティを [IndexingMode.Consistent][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields] の値に設定します。

    ```
    policy.IndexingMode = IndexingMode.Consistent;
    ```

1. [Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path] プロパティが **/** _ の値に設定された型 [ExcludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath] の新しいオブジェクトを、_ *policy** 変数の [ExcludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths] コレクション プロパティに追加します。

    ```
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    ```

1. [Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path] プロパティが **/name/?** の値に設定された型 [IncludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath] の新しいオブジェクトを、 **policy** 変数の [IncludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths] コレクション プロパティに追加します。

    ```
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );
    ```

1. **options** という名前の型 [ContainerProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties] の新しい変数を作成し、コンストラクター パラメーターとして値 ``products`` と ``/categoryId`` を渡します。

    ```
    ContainerProperties options = new ("products", "/categoryId");
    ```

1. **policy** 変数を **options** 変数の [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy] プロパティに割り当てます。

    ```
    options.IndexingPolicy = policy;
    ```

1. **database** 変数の [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] メソッドを非同期に呼び出し、**options** 変数をコンストラクター パラメーターとして渡し、結果を **container** という名前の [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 型の変数に格納します。

    ```
    Container container = await database.CreateContainerIfNotExistsAsync(options);
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、Container クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] プロパティを **Container Created** というタイトルのヘッダーと共に出力します。

    ```
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. 完了すると、コード ファイルに次のものが含まれるはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    IndexingPolicy policy = new ();
    policy.IndexingMode = IndexingMode.Consistent;
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );

    ContainerProperties options = new ("products", "/categoryId");
    options.IndexingPolicy = policy;

    Container container = await database.CreateContainerIfNotExistsAsync(options);
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. **script.cs** ファイルを **保存** します。

1. **Visual Studio Code** で、**12-custom-index-policy** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. スクリプトにより、新しく作成したコンテナーの名前が出力されます。

    ```
    Container Created [products]
    ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

## <a name="observe-an-indexing-policy-created-by-the-net-sdk-using-the-data-explorer"></a>データ エクスプローラーを使用して、.NET SDK によって作成されたインデックス作成ポリシーを確認する

他のインデックス作成ポリシーと同様に、データ エクスプローラーを使用して、.NET SDK を使用してプッシュしたポリシーを表示できます。 ここでは、ポータルを使用して、このラボで作成したポリシーをコードから確認します。

1. Web ブラウザーで、Azure portal (``portal.azure.com``) に移動します。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. **SQL API** ナビゲーション ツリーの **products** コンテナー ノード内で、 **[スケールと設定]** を選択します。

1. **[インデックス作成ポリシー]** セクションで、インデックス作成ポリシーを確認します。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

    > &#128221; これは、このラボで .NET SDK を使用して作成したインデックス作成ポリシーの JSON 表現です。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
