---
lab:
  title: Azure Cosmos DB SQL API SDK を使用してアプリケーションのトラブルシューティングを行う
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: e87e27c83e9aa41ed7494097ce3e0e64a2b46a2f
ms.sourcegitcommit: c3778722311b55568f083480ecc69c9b3e837a18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/19/2022
ms.locfileid: "138025142"
---
# <a name="troubleshoot-an-application-using-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK を使用してアプリケーションのトラブルシューティングを行う

Azure Cosmos DB には、さまざまな操作の型で発生する可能性のある問題のトラブルシューティングに役立つ、広範な応答コードのセットが用意されています。 キャッチは、Azure Cosmos DB 用のアプリを作成するときに適切なエラー処理をプログラムすることです。

このラボでは、2 つのドキュメントのいずれかを挿入または削除できるメニュー駆動型プログラムを作成します。 このラボの主な目的は、最も一般的な応答コードの一部を使用する方法と、アプリのエラー処理コードでそれらを使用する方法について説明することです。  複数の応答コードのエラー処理をコード化しますが、トリガーするのは 2 つの異なる型の条件のみです。  さらに、エラー処理では複雑なことは何もしません。応答コードに応じて、画面にメッセージを表示するか、10 秒待ってからもう一度操作を再試行します。 

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業する環境に **DP-420** のラボ コード リポジトリをまだ複製していない場合は、これらの手順に従って行います。 それ以外の場合は、以前に複製されたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスの詳細をまだ十分理解していない場合は、「[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]」を参照してください。

1. コマンド パレットを開き、**Git: Clone** を実行して、選択したローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを複製します。

    > &#128161; **CTRL + SHIFT + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、アカウントでサポートする API を選択します (たとえば、**Mongo API** または **SQL API**)。 Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得できます。 エンドポイントとキーを使用して、Azure Cosmos DB SQL API アカウントにプログラムで接続します。 Azure SDK for .NET またはその他の SDK の接続文字列でエンドポイントとキーを使用します。

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
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *`Do Not Apply`* |

    > &#128221; ラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **[URI]** フィールドの値を記録します。 この **エンドポイント** の値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この **キー** の値は、この演習で後ほど使用します。

1. 最小化しますが、ブラウザー ウィンドウは閉じません。 次の手順でバックグラウンド ワークロードを開始してから数分後に、Azure portal に戻ります。


## <a name="import-the-microsoftazurecosmos-library-into-a-net-script"></a>Microsoft.Azure.Cosmos ライブラリを .NET スクリプトにインポートする

.NET CLI には、事前構成済みのパッケージ フィードからパッケージをインポートするための [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] コマンドが含まれています。 .NET インストールでは、既定のパッケージ フィードとして NuGet が使用されます。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**26-sdk-troubleshoot** フォルダーを参照します。

1. **26-sdk-troubleshoot** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **26-sdk-troubleshoot** フォルダーに既に設定されているターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

## <a name="run-a-script-to-create-menu-driven-options-to-insert-and-delete-documents"></a>スクリプトを実行して、ドキュメントの挿入と削除を行うメニュー駆動型オプションを作成します。

アプリケーションを実行する前に、Azure Cosmos DB アカウントに接続する必要があります。 

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**26-sdk-troubleshoot** フォルダーを参照します。

1. **Program.cs** コード ファイルを開きます。

1. **endpoint** という名前の既存の変数を、先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定された値で更新します。
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、エンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の既存の変数を、先ほど作成した Azure Cosmos DB アカウントの **key** に設定された値で更新します。

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、キーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **private static readonly string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```
    > &#128221; これはとてもシンプルなプログラムです。  次に示すように、5 つのオプションを含むメニューが表示されます。 事前定義されたドキュメントを挿入する 2 つのオプション、事前定義されたドキュメントを削除する 2 つのオプション、およびプログラムを終了するオプション。

    >```
    >1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >5) Exit
    >Select an option:
    >```

## <a name="time-to-insert-and-delete-documents"></a>ドキュメントを挿入および削除する時間。

1. **1** を選択し、**Enter キー** を押して最初のドキュメントを挿入します。 プログラムによって最初のドキュメントが挿入され、次のメッセージが返されます。

    ```
    Insert Successful.
    Document for customer with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828' Inserted.
    Press [ENTER] to continue
    ```

1. ここでも、**1** を選択し、**Enter キー** を押して最初のドキュメントを挿入します。 今回は、プログラムが例外でクラッシュします。 エラー スタックを調べると、プログラムが失敗した理由を見つけることができます。 エラー スタックから抽出されたメッセージからわかるように、ハンドルされない例外 "Conflict (409)" が発生します。

    ```
    Unhandled exception. Microsoft.Azure.Cosmos.CosmosException : Response status code does not indicate success: Conflict (409);
    ```

1. ドキュメントを挿入しているため、ドキュメントの作成時に返される一般的な[ドキュメント作成ステータス コード][/rest/api/cosmos-db/create-a-document#status-codes]の一覧を確認する必要があります。 このコードの説明は次のとおりです。*新しいドキュメント用に指定した ID が、既存のドキュメントによって取得されています*。 少し前にメニュー オプションを実行して同じドキュメントを作成したので、これは明らかです。

1. スタックをさらに掘り下げると、この例外が 99 行目から呼び出され、次に 52 行目から呼び出されたことがわかります。

    ```
    at Program.CreateDocument1(Container Customer) in C:\WWL\Git\DP-420\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 101   at Program.Main(String[] args) in C:\WWL\Git\DP-420\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 47
    ```

1. 101 行目を確認すると、予期した通り、エラーは CreateItemAsync 操作が原因で発生しました。 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
    ```

1. さらに、101 行目から 104 行目を確認すると、このコードにエラー処理が存在しないのは明らかです。 これを修正する必要があります。 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
        Console.WriteLine("Insert Successful.");
        Console.WriteLine("Document for customer with id = '" + customerID + "' Inserted.");
    ```

1. エラー処理コードで何を実行する必要があるかを決定する必要があります。 [ドキュメント状態コードの作成][/rest/api/cosmos-db/create-a-document#status-codes]を確認すると、この操作で発生する可能性のあるすべての状態コードに対してエラー処理コードを作成することを選択できます。  このラボでは、この一覧の状態コード 403 a 409 のみを考慮します。  返される他のすべての状態コードでは、システム エラー メッセージだけが表示されます。

    > &#128221; 403 例外が発生した場合に実行するタスクをコード化しますが、このラボでは、例外を含むその種類のエラーは生成しません。

1. `CompeteTaskOnCosmosDB` という名前の上位関数のエラー処理を追加しましょう。 45 行目について関数 `Main` で `while` サイクルを見つけ、`CompeteTaskOnCosmosDB` の呼び出しをエラー処理でラップします。 この新しいコードで最初に気付くのは、**catch** で型 `CosmosException` クラスの例外をキャプチャしているということです。  このクラスには、Azure Cosmos DB サービスから要求完了状態コードを取得するプロパティ `StatusCode` が含まれていました。 `StatusCode` プロパティの型は `System.Net.HttpStatusCode` であるため、実際にはその値を .NET [HTTP 状態コード][dotnet/api/system.net.httpstatuscode]のフィールド名と比較しています。  

    ```C#
        try
        {
            await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
        }
        catch (CosmosException e)
        {
                    switch (e.StatusCode.ToString())
                    {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                    }

        }

    ```

1. ファイルを保存すると、クラッシュしたため、メニュー プログラムを再度実行する必要があります。このため、次のコマンドを実行します。

    ```
    dotnet run
    ```
 
1. ここでも、**1** を選択し、**Enter キー** を押して最初のドキュメントを挿入します。 今回はクラッシュしませんが、何が起こったのかについてユーザー フレンドリなメッセージを受け取ります。

    ```
    Insert Failed. 
    Response Code (409).
    Can not insert a duplicate partition key, customer with the same ID already exists.
    Press [ENTER] to continue
    ```

1. ドキュメント作成における 2 つの特定の例外、403 と 409 を処理しているのは良いことですが、他の 3 つの通信タイプの例外は、どのタイプの操作でも発生する可能性があります。  これらの例外は、それぞれ 429、503、および 408 であるか、要求が多すぎる、サービスが利用できない、および要求がタイムアウトになることです。 これらの例外のいずれかが見つかった場合は、次のコードを追加し、10 分待ってから、もう一度ドキュメントを挿入してみます。  コードを超えて追加しましょう。

    ```C#
                    default:
                        Console.WriteLine(e.Message);
                        break;
    ```


    例外処理用のコード:


    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
    ```

    > &#128221; 429、503、または 408 の例外が発生した場合のタスクをコード化しますが、このラボでは、そのタイプの例外でエラーを生成しないことに注意してください。

1. 現在、`Main` 関数は次のようになります。

    ```C#
        public static async Task Main(string[] args)
        {

            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

1. `CreateDocument2` 関数は、上記の変更によっても修正されることに注意してください。

1. 最後に、関数 `DeleteDocument1` および `DeleteDocument2` も、`CreateDocument1` 関数と同様の適切なエラー処理コードに置換するために、次のコードを置換する必要があります。 **CreateItemAsync** の代わりに **DeleteItemAsync** を使用する以外のこれらの関数との唯一の違いは、[削除状態コード][/rest/api/cosmos-db/delete-a-document]が挿入状態コードとは異なるということです。 削除では、**404** 状態コードのみが考慮されます。 追加のケースで `CompeteTaskOnCosmosDB` 関数のエラー処理を更新しましょう。


    ```C#
                    case ("NotFound"):
                        Console.WriteLine("Delete Failed. Response Code (404).");
                        Console.WriteLine("Can not delete customer, customer not found.");
                        break;         
    ```

1.  先に進み、エラー処理を行うコードで削除関数を処理するための次のコードを追加します。 また、状態コード **429**、**503**、および **508** の再試行ロジックも含まれます。 `default` ケースの上に次のコードを追加する必要があります。

    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;       
    ```

1. すべての関数の修正が完了したら、すべてのメニュー オプションを数回テストして、例外が発生したときにアプリがメッセージを返し、クラッシュしないことを確認します。  アプリがクラッシュした場合は、エラーを修正し、コマンドを再実行します。

    ```
    dotnet run
    ```


1. 見てはいけませんが、完了すると、`Main` コードは次のようになります。

    ```C#
        public static async Task Main(string[] args)
        {
            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            case ("NotFound"):
                                Console.WriteLine("Delete Failed. Response Code (404).");
                                Console.WriteLine("Can not delete customer, customer not found.");
                                break; 
                            case ("TooManyRequests"):
                            case ("ServiceUnavailable"):
                            case ("RequestTimeout"):
                                // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                                await Task.Delay(10000); // Wait 10 seconds
                                try
                                {
                                    Console.WriteLine("Try one more time...");
                                    await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                                }
                                catch (CosmosException e2)
                                {
                                    Console.WriteLine("Insert Failed. " + e2.Message);
                                    Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                    break;
                                }
                                break;    
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

## <a name="conclusion"></a>まとめ

最も駆け出しの開発者でさえ、適切なエラー処理をすべてのコードに追加する必要があることを知っています。 このコードでのエラー処理は単純ですが、コードで堅牢なエラー処理ソリューションを作成できるようにする Azure Cosmos DB 例外コンポーネントの基本を理解しているはずです。


[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[/rest/api/cosmos-db/create-a-document#status-codes]:https://docs.microsoft.com/rest/api/cosmos-db/create-a-document#status-codes
[dotnet/api/system.net.httpstatuscode]:https://docs.microsoft.com/dotnet/api/system.net.httpstatuscode?view=net-6.0
[/rest/api/cosmos-db/delete-a-document]:https://docs.microsoft.com/rest/api/cosmos-db/delete-a-document#status-codes

