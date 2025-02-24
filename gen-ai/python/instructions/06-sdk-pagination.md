---
title: '06: Azure Cosmos DB for NoSQL SDK を使用して外積クエリの結果をページ分割する'
lab:
  title: '06: Azure Cosmos DB for NoSQL SDK を使用して外積クエリの結果をページ分割する'
  module: Author complex queries with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 9
parent: Python SDK labs
---

# Azure Cosmos DB for NoSQL SDK を使用して外積クエリの結果をページ分割する

Azure Cosmos DB クエリには、通常、複数の結果ページがあります。 改ページは、Azure Cosmos DB から 1 回の実行ですべてのクエリ結果を返せない場合に、自動的にサーバー側で行われます。 多くのアプリケーションでは、SDK を使用して、パフォーマンスの高い方法でクエリ結果を一括で処理するコードを記述する必要があります。

このラボでは、結果セット全体を反復処理するためにループで使用できるフィード反復子を作成します。

## 開発環境を準備する

「**Azure Cosmos DB を使用してコパイロットを構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照してそれらを行ってください。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用してコパイロットを構築する**」ラボ用の Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#create-azure-cosmos-db-database-and-container-with-sample-data)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## サンプル データを使用して Azure Cosmos DB のデータベースとコンテナーを作成する

既に **cosmicworks-full** という名前の Azure Cosmos DB データベースを作成し、さらにサンプル データが事前に読み込まれている **products** という名前のコンテナーをそのデータベース内に作成している場合は、それらをこのラボに使用して、[次のセクション](#install-the-azure-cosmos-library)に進むことができます。 それ以外の場合は、次の手順に従って新しいサンプル データベースとコンテナーを作成してください。

<details markdown=1>
<summary markdown="span"><strong>サンプル データを含むデータベースとコンテナーを作成するには、クリックしてステップの展開または折りたたみを行います</strong></summary>

1. 新しく作成された **Azure Cosmos DB** アカウント リソース内で、**[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、ホームページの **[Launch quick start]** を選択します。

1. **[New Container]** フォーム内で、以下の値を入力します。

    - **データベース ID**: `cosmicworks-full`
    - **コンテナー ID**: `products`
    - **パーティション キー**: `/categoryId`
    - **分析ストア**: `Off`

1. **[OK]** を選択して新しいコンテナーを作成します。 このプロセスは、リソースを作成し、サンプル製品データを含むコンテナーの事前読み込みを行うのに 1、2 分かかります。

1. 後で戻るので、ブラウザーのタブは開いたままにしておきます。

1. **Visual Studio Code** に戻ります。

</details>

## azure-cosmos ライブラリをインストールする

**azure-cosmos** ライブラリは **PyPI** で使用でき、Python プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/06-sdk-pagination** フォルダーを参照します。

1. **python/06-sdk-pagination** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリがあらかじめ **python/06-sdk-pagination** フォルダーに設定されてターミナルが開きます。

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

## SDK を使用して SQL クエリの小さな結果セットで改ページする

クエリ結果を処理するときは、コードが結果のすべてのページを通過し、後続の要求を行う前にこれ以上ページが残っていないかどうかを確認する必要があります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/06-sdk-pagination** フォルダーを参照します。

1. **script.py** という名前の空 Python スクリプトを開きます。

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

1. 次のコードを追加して、先ほど作成したデータベースとコンテナーに接続します。

   ```python
   database = client.get_database_client("cosmicworks-full")
   container = database.get_container_client("products")
   ```

1. *string* 型の **sql** という名前の新しい変数を、「**SELECT * FROM products WHERE products.price > 500**」という値で作成します。

   ```python
   sql = "SELECT * FROM products WHERE products.price > 500"
   ```

1. コンストラクターへのパラメーターとして `sql` 変数を使用して、[`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items) メソッドを呼び出します。 `max_item_count` を `50` に設定して、各ページで返される項目数を制限します。

   ```python
   iterator = container.query_items(
       query=sql,
       max_item_count=50  # Set maximum items per page
   )
   ```

1. 反復子オブジェクトで [`by_page`](https://learn.microsoft.com/python/api/azure-core/azure.core.paging.itempaged?view=azure-python#azure-core-paging-itempaged-by-page) メソッドを非同期で呼び出す非同期 **for** ループを作成します。 このメソッドは、呼び出されるたびに結果のページを返します。

   ```python
   async for page in iterator.by_page():
   ```

1. 非同期 **for** ループ内で、ページ分割された結果を非同期で反復処理し、各項目の `id`、`name`、および `price` を出力します。

   ```python
   async for product in page:
       print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")
   ```

1. `main` メソッドの下で、次のコードを追加することで、`asyncio` ライブラリを使用して `main` メソッドを実行します。

   ```python
   if __name__ == "__main__":
       asyncio.run(query_items_async())
   ```

1. **script.py** ファイルは次のようになるはずです。

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Get database and container clients
           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products WHERE products.price > 500"
        
           iterator = container.query_items(
               query=sql,
               max_item_count=50  # Set maximum items per page
           )
        
           async for page in iterator.by_page():
               async for product in page:
                   print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")

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

1. このスクリプトは、一度に 50 項目を表示するページを出力するようになります。

    > &#128161; このクエリは、products コンテナー内の数百の項目と一致します。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
