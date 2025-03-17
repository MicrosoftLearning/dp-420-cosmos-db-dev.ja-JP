---
lab:
  title: '02: Azure Cosmos DB JavaScript SDK をオフライン開発用に構成する'
  module: Configure the Azure Cosmos DB for NoSQL SDK
---

# Azure Cosmos DB Python SDK をオフライン開発用に構成する

Azure Cosmos DB Emulator は、開発とテストのために Azure Cosmos DB サービスをエミュレートするローカル ツールです。 このエミュレーターでは NoSQL API がサポートされており、Azure SDK for Python を使用してコードを開発するときにクラウド サービスの代わりに使用できます。

このラボでは、Azure SDK for Python から Azure Cosmos DB Emulator に接続します。

## 開発環境を準備する

「**Azure Cosmos DB を使用してコパイロットを構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照してそれらを行ってください。

## Azure Cosmos DB Emulator を起動する

ホストされているラボ環境を使用している場合は、エミュレーターが既にインストールされている必要があります。 そうでない場合は、[インストール手順](https://docs.microsoft.com/azure/cosmos-db/local-emulator)を参照して Azure Cosmos DB Emulator をインストールしてください。 エミュレーターが起動したら、接続文字列を取得し、それを使用して Azure SDK for Python によりエミュレーターに接続できます。

> &#128161; 必要に応じて、Docker コンテナーとして使用できる、[新しい Linux ベースの Azure Cosmos DB Emulator (プレビュー段階)](https://learn.microsoft.com/azure/cosmos-db/emulator-linux) をインストールできます。 さまざまなプロセッサとオペレーティング システムでの実行をサポートしています。

1. **Azure Cosmos DB Emulator** を起動します。

    > 💡 Windows を使用している場合、Azure Cosmos DB Emulator は Windows タスク バーとスタート メニューの両方にぴん留めされています。 ピン留めされたアイコンから起動しない場合は、**C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** ファイルをダブルクリックで開いてみてください。

1. エミュレーターで既定のブラウザーが開き、ランディング ページ **https://localhost:8081/_explorer/index.html** に移動するのを待ちます。

1. **[クイック スタート]** ペインで、**プライマリ接続文字列**をメモします。 この接続文字列は後で使用します。

> &#128221; エミュレーターが実行されている場合でも、ランディング ページが正常に読み込まれない場合があります。 このような場合は、既知の接続文字列を使用してエミュレーターに接続できます。 既知の接続文字列は次のとおりです: `AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## azure-cosmos ライブラリをインストールする

**azure-cosmos** ライブラリは **PyPI** で使用でき、Python プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/02-sdk-offline** フォルダーを参照します。

1. **python/02-sdk-offline** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリがあらかじめ **python/02-sdk-offline** フォルダーに設定されてターミナルが開きます。

1. 依存関係を管理する仮想環境を作成してアクティブにする

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. 次のコマンドを使用して、[azure-cosmos][pypi.org/project/azure-cosmos] パッケージをインストールします。

   ```bash
   pip install azure-cosmos
   ```

## Python SDK からエミュレーターに接続する

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/02-sdk-offline** フォルダーを参照します。

1. **script.py** という名前の空 Python スクリプトを開きます。

1. 次のコードを追加して、エミュレーターに接続し、データベースを作成し、その ID を出力します。

   ```python
   from azure.cosmos import CosmosClient, PartitionKey
   
   # Connection string for the Azure Cosmos DB Emulator
   connection_string = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="
    
   # Initialize the Cosmos client
   client = CosmosClient.from_connection_string(connection_string)
    
   # Create a database
   database_name = "cosmicworks"
   database = client.create_database_if_not_exists(id=database_name)
    
   # Print the database ID
   print(f"New Database: Id: {database.id}")
   ```

1. **script.py** ファイルを**保存**します。

## スクリプトを実行する

1. このラボの Python 環境を設定したのと同じ、**Visual Studio Code** のターミナル ウィンドウを使用します。 そのウィンドウが閉じられた場合は、**python/02-sdk-offline** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. `python` コマンドを使用してスクリプトを実行します。

   ```bash
   python script.py
   ```

1. このスクリプトにより、エミュレーターに `cosmicworks` という名前のデータベースが作成されます。 次のような出力が表示されます。

   ```text
   New Database: Id: cosmicworks
   ```

## 新しいコンテナーを作成して表示する

スクリプトを拡張することで、データベース内にコンテナーを作成できます。

### 更新されたコード

1. コンテナーを作成するために、`script.py` ファイルを変更して次のコードをファイルの下部に追加します。

   ```python
   # Create a container
   container_name = "products"
   partition_key_path = "/categoryId"
   throughput = 400
    
   container = database.create_container_if_not_exists(
       id=container_name,
       partition_key=PartitionKey(path=partition_key_path),
       offer_throughput=throughput
   )
    
   # Print the container ID
   print(f"New Container: Id: {container.id}")
   ```

### 更新されたスクリプトを実行する

1. 次のコマンドを使用して更新されたスクリプトを実行します。

   ```bash
   python script.py
   ```

1. このスクリプトにより、エミュレーターに `products` という名前のコンテナーが作成されます。 次のような出力が表示されます。

   ```text
   New Container: Id: products
   ```

### 結果を確認する

1. エミュレーターのデータ エクスプローラーが開いているブラウザーに切り替えます。

1. **NoSQL API** を更新して、新しい **cosmicworks** データベースと **products** コンテナーを確認します。

## Azure Cosmos DB Emulator を停止する

使用が終わったらエミュレーターを停止して、システム リソースを解放することが重要です。 オペレーティング システムに基づき、次の手順を実行します。

### macOS または Linux の場合:

ターミナル ウィンドウでエミュレーターを起動した場合は、次の手順に従います。

1. エミュレーターが実行されているターミナル ウィンドウを特定します。

1. `Ctrl + C` キーを押してエミュレーター プロセスを終了します。

または、エミュレーター プロセスを手動で停止する必要がある場合は、次のようにします。

1. 新しいターミナル ウィンドウを開きます。

1. 次のコマンドを使用して、エミュレーター プロセスを検索します。

   ```bash
   ps aux | grep CosmosDB.Emulator
   ```

出力でエミュレーター プロセスの **PID** (プロセス ID) を特定します。 kill コマンドを使用してエミュレーター プロセスを終了します。

```bash
kill <PID>
```

### Windows の場合:

1. Windows システム トレイ (タスク バー上の時刻付近) にある Azure Cosmos DB Emulator アイコンを見つけます。

1. エミュレーター アイコンを右クリックして、コンテキスト メニューを開きます。

1. **[終了]** を選択してエミュレーターをシャットダウンします。

> 💡 エミュレーターのすべてのインスタンスが終了するまで 1 分かかる場合があります。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
