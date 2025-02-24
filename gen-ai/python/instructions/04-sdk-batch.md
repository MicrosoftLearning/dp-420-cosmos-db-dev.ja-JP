---
title: 04 - Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をまとめてバッチ処理する
lab:
  title: 04 - Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をまとめてバッチ処理する
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 7
parent: Python SDK labs
---

# Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をまとめてバッチ処理する

`azure-cosmos` Python SDK には、1 つの論理ステップで複数のポイント操作を実行するための `execute_item_batch` メソッドが用意されています。 これにより、開発者は複数の操作を効率的にバンドルし、サーバー側で正常に完了したかどうかを判断できます。

このラボでは、Python SDK を使用して、成功したトランザクション バッチと誤ったトランザクション バッチの両方を示すデュアルアイテム バッチ操作を実行します。

## 開発環境を準備する

「**Azure Cosmos DB を使用して Coplilot を構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照して実行します。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用して Copilot を構築する**」ラボ用に Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#install-the-azure-cosmos-library)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## azure-cosmos ライブラリをインストールする

**azure-cosmos** ライブラリは **PyPI** で使用でき、Python プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/04-sdk-batch** フォルダーを参照します。

1. **python/04-sdk-batch** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **python/04-sdk-batch** フォルダーに既に設定されているターミナルが開きます。

1. 依存関係を管理する仮想環境を作成してアクティブにする

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. 次のコマンドを使用して、[azure-cosmos][pypi.org/project/azure-cosmos] パッケージをインストールします。

   ```bash
   pip install azure-cosmos
   ```

1. SDK の非同期バージョンを使用しているため、`asyncio` ライブラリもインストールする必要があります。

   ```bash
   pip install asyncio
   ```

1. SDK の非同期バージョンには、`aiohttp` ライブラリも必要です。 次のコマンドを使用してインストールします。

   ```bash
   pip install aiohttp
   ```

1. [azure-identity][pypi.org/project/azure-identity] ライブラリをインストールします。すると、次のコマンドを使用して Azure 認証により Azure Cosmos DB ワークスペースに接続できます。

   ```bash
   pip install azure-identity
   ```

## azure-cosmos ライブラリを使用する

新しく作成したアカウントの資格情報を使用して、SDK クラスに接続し、新しいデータベースとコンテナー インスタンスを作成します。 次に、データ エクスプローラーを使用して、Azure portal でインスタンスが存在することを検証します。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/03-sdk-crud** フォルダーを参照します。

1. **script.py** という名前の空 Python スクリプトを開きます。

1. 次の `import` ステートメントを追加して、**PartitionKey** クラスをインポートします。

   ```python
   from azure.cosmos import PartitionKey
   ```

1. 次の `import` ステートメントを追加して、非同期の **CosmosClient** クラス、**DefaultAzureCredential** クラス、および **asyncio** ライブラリをインポートします。

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio
   ```

1. **endpoint** および **credential** という名前の変数を追加し、**endpoint** 値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定します。 **credential** 変数は、**DefaultAzureCredential** クラスの新しいインスタンスに設定する必要があります。

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; たとえば、エンドポイントが **https://dp420.documents.azure.com:443/** の場合、ステートメントは **endpoint = "https://dp420.documents.azure.com:443/"** になります。

1. Cosmos DB の操作は、すべて `CosmosClient` のインスタンスから始まります。 非同期クライアントを使用するには、非同期メソッド内でのみ使用できる async/await キーワードを使用する必要があります。 **main** という名前の新しい非同期メソッドを作成し、次のコードを追加して、**endpoint** および **credential** 変数により非同期 **CosmosClient** クラスの新しいインスタンスを作成します。

   ```python
   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
   ```

    > &#128161; 非同期 **CosmosClient** クライアントを使用しているため、それを適切に使用するために、ウォームアップして閉じる必要もあります。 上記のコードで示されているように、`async with` キーワードを使用してクライアントを起動することをお勧めします。これらのキーワードにより、クライアントを自動的にウォームアップ、初期化、クリーンアップするコンテキスト マネージャーが作成されるため、自分で行う必要がなくなります。

1. データベースとコンテナーがまだない場合は、次のコードを追加して作成します。

   ```python
   # Create database
   database = await client.create_database_if_not_exists(id="cosmicworks")
    
   # Create container
   container = await database.create_container_if_not_exists(
       id="products",
       partition_key=PartitionKey(path="/categoryId"),
       offer_throughput=400
   )
   ```

1. `main` メソッドの下で、次のコードを追加することで、`asyncio` ライブラリを使用して `main` メソッドを実行します。

   ```python
   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **script.py** ファイルは次のようになるはずです。

   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **script.py** ファイルを**保存**します。

1. スクリプトを実行する前に、`az login` コマンドを使用して Azure にログインする必要があります。 ターミナル ウィンドウで以下を実行します。

   ```bash
   az login
   ```

1. スクリプトを実行してデータベースとコンテナーを作成します。

   ```bash
   python script.py
   ```

1. Web ブラウザー ウィンドウに戻ります。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. [**データ エクスプローラー**] で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

## トランザクション バッチの作成

まず、2 つの架空の製品を作成する単純なトランザクション バッチを作成しましょう。 このバッチでは、同じ "used accessories" カテゴリ識別子を持つコンテナーに、worn saddle と rusty handlebar を挿入します。 どちらの項目にも同じ論理パーティション キーがあります。これにより、確実にバッチ操作が成功します。

1. **Visual Studio Code** に戻ります。 開いたままになっていない場合は、**python/04-sdk-batch** フォルダー内の **script.py** コード ファイルを開きます。

1. **worn saddle** と **rusty handlebar** という製品を表す 2 つの辞書を作成します。 どちらの品目も、**"9603ca6c-9e28-4a02-9194-51cdb7fea816"** という同じパーティション キー値を共有します。

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   ```

1. パーティション キー値を定義します。

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. 上記の 2 つの品目を含むバッチを作成します。

   ```python
   batch = [saddle, handlebar]
   ```

1. `container` オブジェクトの `execute_item_batch` メソッドを使用してバッチを実行し、バッチ内の各品目に対する応答を出力します。

```python
try:
        # Execute the batch
        batch_response = await container.execute_item_batch(batch, partition_key=partition_key)

        # Print results for each operation in the batch
        for idx, result in enumerate(batch_response):
            status_code = result.get("statusCode")
            resource = result.get("resourceBody")
            print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
    except exceptions.CosmosBatchOperationError as e:
        error_operation_index = e.error_index
        error_operation_response = e.operation_responses[error_operation_index]
        error_operation = batch[error_operation_index]
        print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
    except Exception as ex:
        print(f"An error occurred: {ex}")
```

1. 完了すると、コード ファイルが次のようになるはずです。
  
   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

           saddle = ("create", (
               {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           handlebar = ("create", (
               {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [saddle, handlebar]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. スクリプトを**保存**してもう一度実行します。

   ```bash
   python script.py
   ```

1. 出力は、各操作の成功の状態コードを示す必要があります。

## 誤ったトランザクション バッチの作成

次は、意図的にエラーを発生させるトランザクション バッチを作成しましょう。 このバッチでは、異なる論理パーティション キーを持つ 2 つの項目の挿入が試行されます。 "used accessories" カテゴリには flickering strobe light を作成し、"pristine accessories" カテゴリには new helmet を作成します。 定義上、これは誤った要求であり、このトランザクションを実行するとエラーが返されるはずです。

1. **script.py** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"

   batch = [saddle, handlebar]
   ```

1. 異なるパーティション キー値を持つ新しい **flckering strobe light** と **new helmet** を作成するようにスクリプトを変更します。

   ```python
   light = ("create", (
       {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   helmet = ("create", (
       {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
   ))
   ```

1. バッチのパーティション キー値を定義します。

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. 上記の 2 つの品目を含む新しいバッチを作成します。

   ```python
   batch = [light, helmet]
   ```

1. 完了すると、コード ファイルが次のようになるはずです。

   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

           light = ("create", (
               {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           helmet = ("create", (
               {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [light, helmet]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. スクリプトを**保存**してもう一度実行します。

   ```bash
   python script.py
   ```

1. ターミナルからの出力を確認します。 2 つ目の品目 ("New Helmet") の状態コードは、**無効な要求**に対する **400** となるはずです。 これは、トランザクション内のすべての項目で、トランザクション バッチと同じパーティション キー値が共有されなかったため発生しました。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
