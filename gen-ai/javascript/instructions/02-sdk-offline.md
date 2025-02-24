---
title: '02: Azure Cosmos DB JavaScript SDK をオフライン開発用に構成する'
lab:
  title: '02: Azure Cosmos DB JavaScript SDK をオフライン開発用に構成する'
  module: Configure the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 5
parent: JavaScript SDK labs
---

# Azure Cosmos DB JavaScript SDK をオフライン開発用に構成する

Azure Cosmos DB Emulator は、開発とテストのために Azure Cosmos DB サービスをエミュレートするローカル ツールです。 このエミュレーターでは NoSQL API がサポートされており、Azure SDK for JavaScript を使用してコードを開発するときにクラウド サービスの代わりに使用できます。

このラボでは、Azure SDK for JavaScript から Azure Cosmos DB Emulator に接続します。

## 開発環境を準備する

「**Azure Cosmos DB を使用してコパイロットを構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照してそれらを行ってください。

## Azure Cosmos DB Emulator を起動する

ホストされているラボ環境を使用している場合は、エミュレーターが既にインストールされている必要があります。 そうでない場合は、[インストール手順](https://docs.microsoft.com/azure/cosmos-db/local-emulator)を参照して Azure Cosmos DB Emulator をインストールしてください。 エミュレーターが起動したら、接続文字列を取得し、それを使用して Azure SDK for JavaScript によりエミュレーターに接続できます。

> &#128161; 必要に応じて、Docker コンテナーとして使用できる、[新しい Linux ベースの Azure Cosmos DB Emulator (プレビュー段階)](https://learn.microsoft.com/azure/cosmos-db/emulator-linux) をインストールできます。 さまざまなプロセッサとオペレーティング システムでの実行をサポートしています。

1. **Azure Cosmos DB Emulator** を起動します。

    > 💡 Windows を使用している場合、Azure Cosmos DB Emulator は Windows タスク バーとスタート メニューの両方にぴん留めされています。 ピン留めされたアイコンから起動しない場合は、**C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** ファイルをダブルクリックで開いてみてください。

1. エミュレーターで既定のブラウザーが開き、ランディング ページ **https://localhost:8081/_explorer/index.html** に移動するのを待ちます。

1. **[クイック スタート]** ペインで、**プライマリ接続文字列**をメモします。 この接続文字列は後で使用します。

> &#128221; エミュレーターが実行されている場合でも、ランディング ページが正常に読み込まれない場合があります。 このような場合は、既知の接続文字列を使用してエミュレーターに接続できます。 既知の接続文字列は次のとおりです: `AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## @azure/cosmos ライブラリをインポートする

**@azure/cosmos** ライブラリは **npm** で使用でき、JavaScript プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/02-sdk-offline** フォルダーを参照します。

1. **javascript/02-sdk-offline** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > 💡 このコマンドを実行すると、開始ディレクトリがあらかじめ **javascript/02-sdk-offline** フォルダーに設定されてターミナルが開きます。

1. 新しい Node.js プロジェクトを初期化します。

    ```bash
    npm init -y
    ```

1. 次のコマンドを使用して [@azure/cosmos][npmjs.com/package/@azure/cosmos] パッケージをインストールします。

    ```bash
    npm install @azure/cosmos
    ```

## JavaScript SDK からエミュレーターに接続する

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/02-sdk-offline** フォルダーを参照します。

1. **script.js**という名前の空 JavaScript ファイルを開きます。

1. 次のコードを追加して、エミュレーターに接続し、データベースを作成し、その ID を出力します。

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    
    // Connection string for the Azure Cosmos DB Emulator
    const endpoint = "https://127.0.0.1:8081/";
    const key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    // Initialize the Cosmos client
    const client = new CosmosClient({ endpoint, key });
    
    async function main() {
        // Create a database
        const databaseName = "cosmicworks";
        const { database } = await client.databases.createIfNotExists({ id: databaseName });
    
        // Print the database ID
        console.log(`New Database: Id: ${database.id}`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **script.js** ファイルを**保存**します。

## スクリプトを実行する

1. このラボのライブラリのインストールに使用したのと同じ、**Visual Studio Code** のターミナル ウィンドウを使用します。 そのウィンドウが閉じられた場合は、**javascript/02-sdk-offline** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. `node` コマンドを使用してスクリプトを実行します。

    ```bash
    node script.js
    ```

1. このスクリプトにより、エミュレーターに `cosmicworks` という名前のデータベースが作成されます。 次のような出力が表示されます。

    ```text
    New Database: Id: cosmicworks
    ```

## 新しいコンテナーを作成して表示する

スクリプトを拡張することで、データベース内にコンテナーを作成できます。

### 更新されたコード

1.  `script.js` ファイルを変更してファイルの下部にある次の行 (`main().catch((error) => console.error(error));`) を**置き換え**、コンテナーを作成します。

```javascript
async function createContainer() {
    const containerName = "products";
    const partitionKeyPath = "/categoryId";
    const throughput = 400;

    const { container } = await client.database("cosmicworks").containers.createIfNotExists({
        id: containerName,
        partitionKey: { paths: [partitionKeyPath] },
        throughput: throughput
    });

    // Print the container ID
    console.log(`New Container: Id: ${container.id}`);
}

main()
    .then(createContainer)
    .catch((error) => console.error(error));
```

これで、`script.js` ファイルは次のようになります。

```javascript
const { CosmosClient } = require("@azure/cosmos");
process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

// Connection string for the Azure Cosmos DB Emulator
const endpoint = "https://127.0.0.1:8081/";
const key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";

// Initialize the Cosmos client
const client = new CosmosClient({ endpoint, key });

async function main() {
    // Create a database
    const databaseName = "cosmicworks";
    const { database } = await client.databases.createIfNotExists({ id: databaseName });

    // Print the database ID
    console.log(`New Database: Id: ${database.id}`);
}

async function createContainer() {
    const containerName = "products";
    const partitionKeyPath = "/categoryId";
    const throughput = 400;

    const { container } = await client.database("cosmicworks").containers.createIfNotExists({
        id: containerName,
        partitionKey: { paths: [partitionKeyPath] },
        throughput: throughput
    });

    // Print the container ID
    console.log(`New Container: Id: ${container.id}`);
}

main()
    .then(createContainer)
    .catch((error) => console.error(error));
```

### 更新されたスクリプトを実行する

1. 次のコマンドを使用して更新されたスクリプトを実行します。

    ```bash
    node script.js
    ```

1. このスクリプトにより、エミュレーターに `products` という名前のコンテナーが作成されます。 次のような出力が表示されます。

    ```text
    New Database: Id: cosmicworks
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
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
