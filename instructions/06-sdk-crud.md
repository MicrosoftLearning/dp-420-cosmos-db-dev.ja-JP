---
lab:
  title: Azure Cosmos DB SQL API SDK を使用してドキュメントを作成および更新する
  module: Module 4 - Implement Azure Cosmos DB SQL API point operations
ms.openlocfilehash: 5aa8e7ec314243dce08e3d14a561e45d2ac83049
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024985"
---
# <a name="create-and-update-documents-with-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK を使用してドキュメントを作成および更新する

[Microsoft.Azure.Cosmos.Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] クラスには、Azure Cosmos DB SQL API コンテナー内で項目を作成、取得、更新、削除するための一連のメンバー メソッドが用意されています。 これらのメソッドを組み合わせて、SQL API コンテナー内のさまざまな項目に対して、最も一般的な "CRUD" 操作の一部を実行します。

このラボでは、この SDK を使用して、Azure Cosmos DB SQL API コンテナー内の項目に対して日常的な CRUD 操作を実行します。

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

1. 新しく作成した **Azure Cosmos DB** アカウント リソースにアクセスし、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が表示されます。 具体的な内容は次のとおりです。

    1. **[URI]** フィールドの値を記録します。 この **エンドポイント** 値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この **キー** 値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="connect-to-the-azure-cosmos-db-sql-api-account-from-the-sdk"></a>SDK から Azure Cosmos DB SQL API アカウントに接続する

新しく作成したアカウントの資格情報を使用して、SDK クラスに接続し、新しいデータベースとコンテナー インスタンスを作成します。 次に、データ エクスプローラーを使用して、Azure portal でインスタンスが存在することを検証します。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**06-sdk-crud** フォルダーを参照します。

1. **06-sdk-crud** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、ターミナルが開き、開始ディレクトリが既に **06-sdk-crud** フォルダーに設定されています。

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドします。

    ```
    dotnet build
    ```

1. 統合ターミナルを閉じます。

1. **06-sdk-crud** フォルダー内の **script.cs** コード ファイルを開きます。

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

1. エミュレーター内に作成する新しいデータベースの名前 (**cosmicworks**) を渡して **client** 変数の CreateDatabaseIfNotExistsAsync メソッドを非同期に呼び出し、**Database** 型の変数に結果を格納します。

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. **cosmicworks** データベース内に作成する新しいコンテナーの名前 (**products**)、パーティション キー パス ( **/categoryId**)、およびスループット (**400**) を渡して **database** 変数の **CreateContainerIfNotExistsAsync** メソッドを非同期に呼び出し、**Container** 型の変数に結果を格納します。
  
    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);    
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);  
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**06-sdk-crud** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じます。

1. Web ブラウザーの新しいウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="perform-create-and-read-point-operations-on-items-with-the-sdk"></a>SDK を使用して項目に対して作成および読み取りポイント操作を実行する

ここでは、Microsoft.Azure.Cosmos.Container クラスの非同期メソッド セットを使用して、SQL API コンテナー内の項目に対して一般的な操作を実行します。 これらの操作はすべて、C# のタスクの非同期プログラミング モデルを使用して行います。

1. **Visual Studio Code** に戻ります。 **06-sdk-crud** フォルダー内の **product.cs** コード ファイルを開きます。

    > &#128221; **script.cs** ファイルのエディターは閉じないでください。

1. このコード ファイル内の **Product** クラスを確認します。 このクラスは、このコンテナー内に格納および操作する product 項目を表します。

1. **script.cs** コード ファイルのエディター タブに戻ります。

1. **Product** 型の新しいオブジェクトを **saddle** という名前で作成し、次のプロパティを設定します。

    | プロパティ | 値 |
    | ---: | :--- |
    | **id** | ``706cd7c6-db8b-41f9-aea2-0e0c7e8eb009`` |
    | **categoryId** | ``9603ca6c-9e28-4a02-9194-51cdb7fea816`` |
    | **name** | ``Road Saddle`` |
    | **price** | ``45.99d`` |
    | **tags** | ``{ tan, new, crisp }`` |

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };
    ```

1. メソッド パラメーターとして **saddle** 変数を渡し、ジェネリック型として **Product** を使用して、**container** 変数の [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync] 汎用メソッドを非同期に呼び出します。

    ```
    await container.CreateItemAsync<Product>(saddle);
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);  
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**06-sdk-crud** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じます。

1. **script.cs** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **id** という名前の文字列変数を **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009** の値を指定して作成します。

    ```
    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. **categoryId** という名前の文字列変数を **9603ca6c-9e28-4a02-9194-51cdb7fea816** の値を指定して作成します。

    ```
    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. コンストラクター パラメーターとして **categoryId** 変数を渡して、**partitionKey** という名前の [PartitionKey][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey] 型の変数を作成します。

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. メソッド パラメーターとして **id** および **partitionkey** 変数を渡し、ジェネリック型として **Product** を使用して、**container** 変数の [ReadItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] 汎用メソッドを非同期に呼び出し、結果を **Product** 型の **saddle** という名前の変数に格納します。

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. **Console.WriteLine** 静的メソッドを呼び出して、書式付きの出力文字列を使用して、saddle オブジェクトを出力します。

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);  
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**06-sdk-crud** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. ターミナルからの出力を確認します。 具体的には、項目の ID、名前、および価格を含む、書式付きの出力テキストを確認します。

1. 統合ターミナルを閉じます。

## <a name="perform-update-and-delete-point-operations-with-the-sdk"></a>SDK を使用してポイントの更新および削除操作を実行する

SDK を学習しながら、オンラインの Azure Cosmos DB SDK アカウントまたはエミュレーターを使用して項目を更新し、操作を実行して変更が適用されたかどうかを確認する際に、データ エクスプローラーとお使いの IDE の間を行き来することは珍しくありません。 ここでは、SDK を使用して項目の更新と削除を行います。

1. Web ブラウザーの新しいウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開します。

1. **Items** ノードを選択します。 コンテナー内の唯一の項目を選択し、項目の **name** および **price** プロパティの値を確認します。

    | **プロパティ** | **Value** |
    | ---: | :--- |
    | **名前** | Road Saddle |
    | **価格** | $45.99 |

    > &#128221; この時点では、これらの値は、項目を作成したときから変更されていないはずです。 この演習で、これらの値を変更します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. **Visual Studio Code** に戻ります。 **script.cs** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. price プロパティの値を「**32.55**」に設定して、**saddle** 変数を変更します。

    ```
    saddle.price = 32.55d;
    ```

1. **name** プロパティの値を「**Road LL Saddle**」に設定して、**saddle** 変数を再度変更します。

    ```
    saddle.name = "Road LL Saddle";
    ```

1. メソッド パラメーターとして **saddle** 変数を渡し、ジェネリック型として **Product** を使用して、**container** 変数の [UpsertItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync] 汎用メソッドを非同期に呼び出します。

    ```
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);  
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**06-sdk-crud** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じます。

1. Web ブラウザーの新しいウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開します。

1. **Items** ノードを選択します。 コンテナー内の唯一の項目を選択し、項目の **name** および **price** プロパティの値を確認します。

    | **プロパティ** | **Value** |
    | ---: | :--- |
    | **名前** | Road LL Saddle |
    | **価格** | $32.55 |

    > &#128221; この時点で、これらの値は、項目を確認したときから変更されているはずです。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. **Visual Studio Code** に戻ります。 **script.cs** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. メソッド パラメーターとして **id** および **partitionkey** 変数を渡し、ジェネリック型として **Produc** を使用して、**container** 変数の [DeleteItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync] 汎用メソッドを非同期に呼び出します。

    ```
    await container.DeleteItemAsync<Product>(id, partitionKey);
    ```

1. **script.cs** コード ファイルを **保存** します。

1. **Visual Studio Code** で、**06-sdk-crud** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じます。

1. Web ブラウザーの新しいウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開します。

1. **Items** ノードを選択します。 項目リストが空になっていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
