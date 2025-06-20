---
lab:
  title: SDK を使ってユーザー定義関数を実装してから使用する
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# SDK を使ってユーザー定義関数を実装してから使用する

Azure Cosmos DB for NoSQL 用の .NET SDK 使用して、コンテナーから直接サーバー側のプログラミング コンストラクトを管理し、呼び出すことができます。 新しいコンテナーを準備する際、データ エクスプローラーを使用して手動でタスクを実行する代わりに、.NET SDK を使用して UDF をコンテナーに直接発行する方が理にかなっている場合があります。

このラボでは、.NET SDK を使用して新しい UDF を作成し、データ エクスプローラーを使用して UDF が正常に機能していることを検証します。

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

1. ブラウザー ウィンドウを閉じずに、**Visual Studio Code** を開きます。

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

## .NET SDK を使用してユーザー定義関数 (UDF) を作成する

.NET SDK の [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] クラスには、SDK から直接ストアド プロシージャ、UDF、およびトリガーに対して CRUD 操作を実行するために使用する [Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] プロパティが用意されています。 このプロパティを使用して、新しい UDF を作成し、その UDF を Azure Cosmos DB for NoSQL コンテナーにプッシュします。 UDF を SDK を使用して作成すると、製品の税込み価格が計算され、税込み価格を使用して製品で SQL クエリを実行できるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**33-create-use-udf-sdk** フォルダーを参照します。

1. **script.cs** コード ファイルを開きます。

1. [Microsoft.Azure.Cosmos.Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts] 名前空間の using ブロックを追加します。

    ```
    using Microsoft.Azure.Cosmos.Scripts;
    ```

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

1. 既定の空のコンストラクターを使用して、props という名前の [UserDefinedFunctionProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties] 型の新しい変数を作成します。

    ```
    UserDefinedFunctionProperties props = new ();
    ```

1. **props** 変数の [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id] プロパティを **tax** の値に設定します。

    ```
    props.Id = "tax";
    ```

1. **props** 変数の [Body][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body] プロパティを **props.Body = "function tax(i) { return i * 1.25; }";** に設定します。

    ```
    props.Body = "function tax(i) { return i * 1.25; }";
    ```

1. パラメーターとして **props** 変数を渡して、**container** 変数の [Scripts.CreateUserDefinedFunctionAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] メソッドを非同期に呼び出し、結果を [UserDefinedFunctionResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse] 型の **udf** という名前の変数に保存します。

    ```
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、**Created UDF** というタイトルのヘッダーを持つ UserDefinedFunctionResponse クラスの [Resource.Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource] プロパティを出力します。

    ```
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Scripts;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    UserDefinedFunctionProperties props = new ();
    props.Id = "tax";
    props.Body = "function tax(i) { return i * 1.25; }";
    
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. **script.cs** ファイルを**保存**します。

1. **Visual Studio Code** で、**33-create-use-udf-sdk** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. スクリプトにより、新しく作成した UDF の名前が出力されます。

    ```
    Created UDF [tax]
    ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

## データ エクスプローラーを使用して UDF をテストする

Azure Cosmos DB コンテナーに新しい UDF を作成したので、データ エクスプローラーを使用して UDF が想定どおりに動作していることを検証します。

1. Web ブラウザーに戻ります。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. [**データ エクスプローラー**] で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. **NoSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、[**新しい SQL クエリ**] を選択します。

1. [クエリ] タブで、**[クエリの実行]** を選択して、フィルターなしですべての項目を選択する標準クエリを表示します。

1. エディター領域のコンテンツを削除します。

1. 2 つの価格値が射影されたすべてのドキュメントを返す新しい SQL クエリを作成します。 最初の値はコンテナーからの生の価格値で、2 番目の値は UDF によって計算された価格値です。

    ```
    SELECT p.id, p.price, udf.tax(p.price) AS priceWithTax FROM products p
    ```

1. **[クエリの実行]** を選択します。

1. ドキュメントを確認し、それらの **price** と **priceWithTax** のフィールドを比較します。

    > &#128221; **priceWithTax** フィールドには、**price** フィールドよりも 25% 大きい値が含まれているはずです。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
