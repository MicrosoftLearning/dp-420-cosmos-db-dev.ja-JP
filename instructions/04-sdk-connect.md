---
lab:
  title: SDK を使って Azure Cosmos DB for NoSQL に接続する
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# SDK を使って Azure Cosmos DB for NoSQL に接続する

Azure SDK for .NET は、多くの Azure サービスと対話するための一貫した開発者インターフェイスを提供するライブラリのスイートです。 Azure SDK for .NET は .NET Standard 2.0 仕様に組み込まれており、.NET Framework (4.6.1 以上)、.NET Core (2.1 以上)、および .NET (5 以上) アプリケーションで使用できます。

このラボでは、Azure SDK for .NET を使用して Azure Cosmos DB for NoSQL アカウントに接続します。

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
    | **このアカウントでプロビジョニングできるスループットの総量を制限する** | *Unchecked* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. [**URI**] フィールドに注目します。 この**エンドポイント**の値は、この演習で後ほど使用します。

    1. [**主キー**] フィールドに注目してください。 この**キー**の値は、この演習で後ほど使用します。

1. 後で戻るので、ブラウザーのタブは開いたままにしておきます。

## NuGet で Microsoft.Azure.Cosmos ライブラリを表示する

NuGet Web サイトには、.NET アプリケーションにインポートできるパッケージの検索可能なインデックスが含まれています。 **Microsoft.Azure.Cosmos** などのプレリリース パッケージをインポートするには、NuGet Web サイトを使用して、パッケージをアプリケーションにインポートするための適切なバージョンとコマンドを取得できます。

1. 新しいブラウザー タブで、NuGet Web サイト (``nuget.org``) に移動します。

1. .NET のパッケージ マネージャーである NuGet とその機能の説明を確認します。

1. NuGet.org で **Microsoft.Azure.Cosmos** ライブラリを検索します。

1. **[.NET CLI]** タブを選択して、このライブラリの最新バージョンを .NET プロジェクトにインポートするために必要なコマンドを確認します。

    > &#128161; このコマンドを記録する必要はありません。 この演習の後半では、特定のバージョンのライブラリを使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## Microsoft.Azure.Cosmos ライブラリを .NET プロジェクトにインポートする

.NET CLI には、事前構成済みのパッケージ フィードからパッケージをインポートするための [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] コマンドが含まれています。 .NET インストールでは、既定のパッケージ フィードとして NuGet が使用されます。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**04-sdk-connect** フォルダーを参照します。

1. **04-sdk-connect** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **04-sdk-connect** フォルダーに既に設定されているターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos] パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. 統合ターミナルを閉じます。

## Microsoft.Azure.Cosmos ライブラリを使用する

Azure SDK for.NET の Azure Cosmos DB ライブラリがインポートされたら、すぐに [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 名前空間内のそのクラスを使用して、Azure Cosmos DB for NoSQL アカウントに接続できます。 [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] クラスは、Azure Cosmos DB for NoSQL アカウントへの初期接続を確立するために使用されるコア クラスです。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**04-sdk-connect** フォルダーを参照します。

1. 空の **script.cs** コード ファイルを開きます。

1. 組み込みの **System** および **System.Linq** 名前空間の using ブロックを追加します。

    ```
    using System;
    using System.Linq;
    ```

1. [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]名前空間の using ブロックを追加します。

    ```
    using Microsoft.Azure.Cosmos;
    ```

1. 以前に作成した Azure Cosmos DB アカウントの **endpoint** に値が設定された **endpoint** という名前の **string** 変数を追加します。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、ご自分のエンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を追加し、その値を先ほど作成した Azure Cosmos DB アカウントの **key** に設定します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、キーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. コンストラクターで **endpoint** 変数と **key** 変数を使用して、型 [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] の **client** という名前の新しい変数を追加します。
  
    ```
    CosmosClient client = new (endpoint, key);
    ```

1. **client** 変数の [ReadAccountAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync] メソッドを呼び出した非同期の結果を使用して、型 [AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] の **account** という名前の新しい変数を追加します。

    ```
    AccountProperties account = await client.ReadAccountAsync();
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、AccountProperties クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id] プロパティを **Account Name** というタイトルのヘッダーで出力します。

    ```
    Console.WriteLine($"Account Name:\t{account.Id}");
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して AccountProperties クラスの [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions] プロパティをクエリし、最初の結果の [Name][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name] プロパティを **Primary Region** というタイトルのヘッダーで出力します。

    ```
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. **script.cs** コード ファイルを**保存**します。

## スクリプトをテストする

これで Azure Cosmos DB for NoSQL アカウントに接続するための .NET コードが完成したので、スクリプトをテストできます。 このスクリプトは、アカウントの名前と、最初の書き込み可能なリージョンの名前を出力します。 アカウントを作成したときに場所を指定したので、このスクリプトの結果として同じ場所の値が出力されるはずです。

1. **Visual Studio Code** で、**04-sdk-connect** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. スクリプトによって、アカウントの名前と最初の書き込み可能なリージョンが出力されます。 たとえば、アカウントに **dp420** という名前を付け、最初の書き込み可能なリージョンが **[米国西部 2]** の場合、スクリプトでは次のように出力されます。

    ```
    Account Name:   dp420
    Primary Region: West US 2
    ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos
