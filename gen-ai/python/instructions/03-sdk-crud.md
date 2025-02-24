---
title: '03: Azure Cosmos DB for NoSQL SDK を使用してドキュメントを作成および更新する'
lab:
  title: '03: Azure Cosmos DB for NoSQL SDK を使用してドキュメントを作成および更新する'
  module: Implement Azure Cosmos DB for NoSQL point operations
layout: default
nav_order: 6
parent: Python SDK labs
---

# Azure Cosmos DB for NoSQL SDK を使用してドキュメントを作成および更新する

`azure-cosmos` ライブラリには、Azure Cosmos DB for NoSQL コンテナー内の (CRUD) 項目を作成、取得、更新、および削除するメソッドが含まれています。 これらのメソッドを組み合わせて、NoSQL API コンテナー内のさまざまな項目に対して、最も一般的な "CRUD" 操作の一部を実行します。

このラボでは、Python SDK を使用して、Azure Cosmos DB for NoSQL コンテナー内の項目に対する日常的な CRUD 操作を実行します。

## 開発環境を準備する

「**Azure Cosmos DB を使用して Coplilot を構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照して実行します。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用して Copilot を構築する**」ラボ用に Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#install-the-azure-cosmos-library)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## azure-cosmos ライブラリをインストールする

**azure-cosmos** ライブラリは **PyPI** で使用でき、Python プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/03-sdk-crud** フォルダーを参照します。

1. **python/03-sdk-crud** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリがあらかじめ **python/03-sdk-crud** フォルダーに設定されてターミナルが開きます。

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
       asyncio.run(query_items_async())
   ```

1. **script.py** ファイルは次のようになるはずです。

   ```python
   from azure.cosmos import PartitionKey
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

## SDK を使用して項目に対して作成および読み取りポイント操作を実行する

ここでは、**ContainerProxy** クラスのメソッド セットを使用して、NoSQL API コンテナー内の項目に対する一般的な操作を実行します。

1. **Visual Studio Code** に戻ります。 開いたままになっていない場合は、**python/03-sdk-crud** フォルダー内の **script.py** コード ファイルを開きます。

1. 新しい製品品目を作成し、次のプロパティを使用して **saddle** という名前の変数に割り当てます。

    | プロパティ | 値 |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **Price** | *45.99d* |
    | **tags** | *{ tan, new, crisp }* |

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
   ```

1. メソッド パラメーターとして **saddle** 変数で渡す **container** 変数の [`create_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-create-item) メソッドを呼び出します。

   ```python
   await container.create_item(body=saddle)
   ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
   ```python
   from azure.cosmos import PartitionKey
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
        
           saddle = {
               "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
               "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
               "name": "Road Saddle",
               "price": 45.99,
               "tags": ["tan", "new", "crisp"]
           }
            
           await container.create_item(body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. スクリプトを**保存**してもう一度実行します。

   ```bash
   python script.py
   ```

1. **[データ エクスプローラー]** で新しい項目を確認します。

1. **Visual Studio Code** に戻ります。

1. **script.py** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
    
   await container.create_item(body=saddle)
   ```

1. **item_id** という名前の文字列変数を **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009** という値で作成します。

   ```python
   item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
   ```

1. **partition_key** という名前の文字列変数を **9603ca6c-9e28-4a02-9194-51cdb7fea816** という値で作成します。

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. メソッド パラメーターとして **item_id** 変数と **partition_key** 変数で渡す **container** 変数の [`read_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-read-item) メソッドを呼び出します。

   ```python
   # Read item    
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
   ```

    > &#128161; `read_item` メソッドを使用すると、コンテナー内の項目に対してポイント読み取り操作を実行できます。 このメソッドでは、読み取る項目を識別するために `item_id` パラメーターと `partition_key` パラメーターが必要です。 Cosmos DB の SQL クエリ言語を使用して 1 つの項目を検索するクエリを実行するのとは対照的に、`read_item` メソッドは 1 つの項目を取得するのにより効率的でコスト効率の高い方法です。 ポイント読み取りではデータを直接読み取ることができ、クエリ エンジンが要求を処理する必要はありません。

1. 書式設定された出力文字列を使用して saddle オブジェクトを出力します。

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. 完了すると、コード ファイルが次のようになるはずです。

   ```python
   from azure.cosmos import PartitionKey
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
       
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. スクリプトを**保存**してもう一度実行します。

   ```bash
   python script.py
   ```

1. ターミナルからの出力を確認します。 具体的には、項目の ID、名前、および価格を含む、書式付きの出力テキストを確認します。

## SDK を使用してポイントの更新および削除操作を実行する

SDK の学習においては、オンラインの Azure Cosmos DB アカウントまたはエミュレーターを使用して項目を更新し、操作を実行して変更が適用されたかどうかを確認する際に、データ エクスプローラーと使用している IDE の間を行き来することは珍しくありません。 ここでは、SDK を使用して項目の更新と削除を行います。

1. Web ブラウザーのウィンドウまたはタブに戻ります。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. [**データ エクスプローラー**] で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開します。

1. **Items** ノードを選択します。 コンテナー内の唯一の項目を選択し、項目の **name** および **price** プロパティの値を確認します。

    | **プロパティ** | **値** |
    | ---: | :--- |
    | **名前** | *Road Saddle* |
    | **価格** | *$45.99* |

    > &#128221; この時点では、これらの値は、項目を作成したときから変更されていないはずです。 この演習で、これらの値を変更します。

1. **Visual Studio Code** に戻ります。 **script.py** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. price プロパティの値を「**32.55**」に設定して、**saddle** 変数を変更します。

   ```python
   saddle["price"] = 32.55
   ```

1. **name** プロパティの値を「**Road LL Saddle**」に設定して、**saddle** 変数を再度変更します。

   ```python
   saddle["name"] = "Road LL Saddle"
   ```

1. メソッド パラメーターとして **item_id** 変数と **saddle** 変数で渡す **container** 変数の [`replace_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-replace-item) メソッドを呼び出します。

   ```python
   await container.replace_item(item=item_id, body=saddle)
   ```

1. 完了すると、コード ファイルが次のようになるはずです。

   ```python
   from azure.cosmos import PartitionKey
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
        
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           saddle["price"] = 32.55
           saddle["name"] = "Road LL Saddle"
    
           await container.replace_item(item=item_id, body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. スクリプトを**保存**してもう一度実行します。

   ```bash
   python script.py
   ```

1. Web ブラウザーのウィンドウまたはタブに戻ります。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. [**データ エクスプローラー**] で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開します。

1. **Items** ノードを選択します。 コンテナー内の唯一の項目を選択し、項目の **name** および **price** プロパティの値を確認します。

    | **プロパティ** | **値** |
    | ---: | :--- |
    | **名前** | *Road LL Saddle* |
    | **価格** | *$32.55* |

    > &#128221; この時点で、これらの値は、項目を確認したときから変更されているはずです。

1. **Visual Studio Code** に戻ります。 **script.py** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

   ```python
   # Read item
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
    
   saddle["price"] = 32.55
   saddle["name"] = "Road LL Saddle"
    
   await container.replace_item(item=item_id, body=saddle)
   ```

1. メソッド パラメーターとして **item_id** 変数と **partition_key** 変数で渡す **container** 変数の [`delete_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-delete-item) メソッドを呼び出します。

   ```python
   # Delete the item
   await container.delete_item(item=item_id, partition_key=partition_key)
   ```

1. スクリプトを保存してもう一度実行します。

   ```bash
   python script.py
   ```

1. 統合ターミナルを閉じます。

1. Web ブラウザーのウィンドウまたはタブに戻ります。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. [**データ エクスプローラー**] で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開します。

1. **Items** ノードを選択します。 項目リストが空になっていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
