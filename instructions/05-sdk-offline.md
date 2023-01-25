---
lab:
  title: Azure Cosmos DB for NoSQL SDK をオフライン開発用に構成する
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# <a name="configure-the-azure-cosmos-db-for-nosql-sdk-for-offline-development"></a>Azure Cosmos DB for NoSQL SDK をオフライン開発用に構成する

Azure Cosmos DB Emulator は、開発とテストのために Azure Cosmos DB サービスをエミュレートするローカル ツールです。 エミュレーターでは for NoSQL がサポートされており、Azure SDK for .NET を使用してコードを開発するときにクラウド サービスの代わりに使用できます。

このラボでは、Azure SDK for .NET から Azure Cosmos DB Emulator に接続します。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業する環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、これらの手順に従って行います。 それ以外の場合は、以前にクローンされたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[作業の開始に関するドキュメント][code.visualstudio.com/docs/getstarted]をご覧ください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリがクローンされたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="start-the-azure-cosmos-db-emulator"></a>Azure Cosmos DB Emulator を起動する

ご利用の環境に既にエミュレーターがプレインストールされている必要があります。 そうでない場合は、[インストール手順][docs.microsoft.com/azure/cosmos-db/local-emulator]を参照して Azure Cosmos DB Emulator をインストールしてください。 エミュレーターが起動したら、接続文字列を取得し、それを使用して、Azure SDK for .NET または任意の他の SDK を使用してエミュレーターに接続できます。

1. **Azure Cosmos DB Emulator** を起動します。

    > &#128221; エミュレーターを起動するための管理者アクセス権を付与するように求められる場合があります。 ラボ環境では、**管理者**アカウントのパスワードは**学生**アカウントと同じになります。

    > &#128161; Azure Cosmos DB Emulator は、Windows タスクバーと [スタート] メニューの両方に固定されています。 ***Emulator がピン留めされたアイコンから始まらない場合は、*** **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** ***ファイル***をダブルクリックして開いてみてください。 エミュレーターの起動には、20 から 30 秒かかることに注意してください。

1. エミュレーターで自動的に既定のブラウザーが開き、**localhost:8081/_explorer/index.html** のランディング ページに移動するのを待ちます。

1. **Azure Cosmos DB Emulator** のランディング ページで、 **[クイック スタート]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **[プライマリ接続文字列]** フィールドの値を記録します。 この**接続文字列**の値は、この演習で後ほど使用します。

1. **[エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**NoSQL API** ナビゲーション ツリー内にノードがないことを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="connect-to-the-emulator-from-the-sdk"></a>SDK からエミュレーターに接続する

この演習で使用する .NET スクリプトには、**Microsoft.Azure.Cosmos** ライブラリが既にプレインストールされています。 さらに、時間を節約するために、いくつかの定型コードが既に記述されています。 定型接続文字列値を更新し、スクリプトを完了するためにいくつかのコード行を記述する必要があります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**05-sdk-offline** フォルダーを参照します。

1. **05-sdk-offline** フォルダー内の **script.cs** コード ファイルを開きます。

1. Azure Cosmos DB Emulator の**接続文字列**に値が設定された **connectionString** という名前の既存の変数を更新します。
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    > &#128221; エミュレーターの URI は通常、既定のポートが **8081** に設定された SSL を使用する ***localhost:[port]*** です。

    > &#128221; *C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==* は、エミュレーターのすべてのインストールに使用される既定のキーです。 このキーは、コマンド ライン オプションを使用して変更できます。

1. エミュレーター内に作成する新しいデータベースの名前 (**cosmicworks**) を渡し、[Database][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database] 型の変数に結果を格納する **client** 変数の [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync] メソッドを非同期的に呼び出します。

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、Database クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] プロパティを **New Databaseー** というタイトルのヘッダーと共に出力します。

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. 完了すると、コード ファイルに次の情報が表示されます。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. **script.cs** コード ファイルを**保存**します。

1. **Visual Studio Code** で、**05-sdk-offline** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **05-sdk-offline** フォルダーに既に設定されているターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じます。

## <a name="view-the-changes-in-the-emulator"></a>エミュレーターで変更を表示する

Azure Cosmos DB エミュレーターに新しいデータベースを作成したので、オンライン **データ エクスプローラー**を使用して、エミュレーター内の新しい NoSQL API データベースを確認します。

1. Windows システム トレイのエミュレーター アイコンに移動し、コンテキスト メニューを開き、 **[データ エクスプローラーを開く]** を選択して、既定のブラウザーを使用して **localhost: 8081/_explorer/** ランディング ページに移動します。

1. **Azure Cosmos DB Emulator** のランディング ページで、 **[エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**NoSQL API** ナビゲーション ツリー内の新しい **cosmicworks** データベース ノードを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="create-and-view-a-new-container"></a>新しいコンテナーを作成して表示する

新しいコンテナーの作成は、新しいデータベースの作成に使用されるパターンに似ています。 クラウドまたはエミュレーターにリソースを作成するかどうかに関係なく、ここで学習するコードが関連し、接続文字列を変更するだけで済みます。 スクリプト ファイルをさらに拡張して、データベースと共に新しいコンテナーを作成します。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**05-sdk-offline** フォルダーを参照します。

1. 再度 **05-sdk-offline** フォルダー内の **script.cs** コード ファイルを開きます。

1. **cosmicworks** データベース内に作成する新しいコンテナーの名前 (**products**)、パーティション キー パス ( **/categoryId**)、およびスループット (**400**) を渡し、[Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 型の変数に結果を格納する **database** 変数の [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] メソッドを非同期的に呼び出します。

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、Container クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] プロパティを **New Container** というタイトルのヘッダーと共に出力します。

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. 完了すると、コード ファイルに次の情報が表示されます。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. **script.cs** コード ファイルを**保存**します。

1. **Visual Studio Code** で、**05-sdk-offline** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

1. Windows システム トレイのエミュレーター アイコンに移動し、コンテキスト メニューを開き、 **[データ エクスプローラーを開く]** を選択して、既定のブラウザーを使用して **localhost: 8081/_explorer/** ランディング ページに移動します。

1. **Azure Cosmos DB Emulator** のランディング ページで、 **[エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="stop-the-azure-cosmos-db-emulator"></a>Azure Cosmos DB Emulator を停止する

作業が終了したら、エミュレーターを必ず停止してください。お使いの環境のシステム リソースを消費する可能性があります。 システム トレイ アイコンを使用して、エミュレーターと実行中のすべてのインスタンスを停止します。

1. Windows システム トレイのエミュレーター アイコンに移動し、コンテキスト メニューを開き、 **[終了]** を選択してエミュレーターをシャットダウンします。

    > &#128221; エミュレーターのすべてのインスタンスが終了するまでしばらくかかる場合があります。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
