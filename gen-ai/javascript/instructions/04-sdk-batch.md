---
title: 04 - Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をまとめてバッチ処理する
lab:
  title: 04 - Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をまとめてバッチ処理する
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 7
parent: JavaScript SDK labs
---

# Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をまとめてバッチ処理する

JavaScript SDK for Azure Cosmos DB の `TransactionalBatch` クラスには、同じ論理パーティション キー内でバッチ操作を作成して実行する機能が用意されています。 この機能を使用すると、1 つのトランザクションで複数の操作を実行し、すべての操作を完了する、または操作を何も完了しないようにすることができます。

このラボでは、JavaScript SDK を使用して、2 つの項目を 1 つの論理ユニットとして作成しようとするデュアル項目のバッチ操作を実行します。

## 開発環境を準備する

「**Azure Cosmos DB を使用して Coplilot を構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照して実行します。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用してコパイロットを構築する**」ラボ用の Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#import-the-azurecosmos-library)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## @azure/cosmos ライブラリをインポートする

**@azure/cosmos** ライブラリは **npm** で使用でき、JavaScript プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/04-sdk-batch** フォルダーを参照します。

1. **javascript/04-sdk-batch** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリがあらかじめ **javascript/04-sdk-batch** フォルダーに設定されてターミナルが開きます。

1. 新しい Node.js プロジェクトを初期化します。

    ```bash
    npm init -y
    ```

1. 次のコマンドを使用して [@azure/cosmos][npmjs.com/package/@azure/cosmos] パッケージをインストールします。

    ```bash
    npm install @azure/cosmos
    ```

1. [@azure/identity][npmjs.com/package/@azure/identity] ライブラリをインストールします。すると、次のコマンドを使用して Azure 認証により Azure Cosmos DB ワークスペースに接続できます。

    ```bash
    npm install @azure/identity
    ```

## @azure/cosmos ライブラリを使用する

Azure SDK for JavaScript から Azure Cosmos DB ライブラリがインポートされたら、すぐにそのクラスを使用して Azure Cosmos DB for NoSQL アカウントに接続できます。 **CosmosClient** クラスは、Azure Cosmos DB for NoSQL アカウントへの初期接続を確立するのに使用するコア クラスです。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/04-sdk-batch** フォルダーを参照します。

1. **script.js** という名前の空 JavaScript ファイルを開きます。

1. 次の `require` ステートメントを追加して、**@azure/cosmos** ライブラリと **@azure/identity** ライブラリをインポートします。

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    ```

1. **endpoint** および **credential** という名前の変数を追加し、**endpoint** 値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定します。 **credential** 変数は、**DefaultAzureCredential** クラスの新しいインスタンスに設定する必要があります。

    ```javascript
    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();
    ```

    > &#128221; たとえば、エンドポイントが **https://dp420.documents.azure.com:443/** の場合、ステートメントは **const endpoint = "https://dp420.documents.azure.com:443/";** になります。

1. **client** という名前の新しい変数を追加して、それを **endpoint** 変数と **credential** 変数により **CosmosClient** クラスの新しいインスタンスとして初期化します。

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. データベースとコンテナーがまだない場合は、次のコードを追加して作成します。

    ```javascript
    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. **script.js** ファイルは次のようになるはずです。

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. **script.js** ファイルを**保存**します。

1. スクリプトを実行する前に、`az login` コマンドを使用して Azure にログインする必要があります。 ターミナル ウィンドウで以下を実行します。

    ```bash
    az login
    ```

1. スクリプトを実行してデータベースとコンテナーを作成します。

    ```bash
    node script.js
    ```

1. Web ブラウザー ウィンドウに戻ります。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. [**データ エクスプローラー**] で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

## トランザクション バッチの作成

まず、2 つの架空の製品をコンテナーに追加する単純なトランザクション バッチを作成しましょう。 このバッチでは、同じ "used accessories" カテゴリ識別子を持つコンテナーに、worn saddle と rusty handlebar を挿入します。 どちらの項目にも同じ論理パーティション キーがあります。これにより、確実にバッチ操作が成功します。

1. **Visual Studio Code** に戻ります。 まだ開かない場合は、**javascript/04-sdk-batch** フォルダー内の **script.js** コード ファイルを開きます。

1. トランザクション バッチに挿入する 2 つの製品項目を定義します。

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    ```

1. 同じ論理パーティション キーのトランザクション バッチを作成し、項目を追加します。

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = container.items.batch(partitionKey)
        .create(saddle)
        .create(handlebar);
    ```

1. バッチを実行し、操作の状態を出力します。

    ```javascript
    const response = await batch.execute();
    console.log(`Status: ${response.statusCode}`);
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: saddle },
            { operationType: BulkOperationType.Create, resourceBody: handlebar },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. スクリプトを**保存**してもう一度実行します。

    ```bash
    node script.js
    ```

1. 出力は、各操作の成功の状態コードを示す必要があります。

## 誤ったトランザクション バッチの作成

次は、意図的にエラーを発生させるトランザクション バッチを作成しましょう。 このバッチでは、異なる論理パーティション キーを持つ 2 つの項目の挿入が試行されます。 "used accessories" カテゴリには flickering strobe light を作成し、"pristine accessories" カテゴリには new helmet を作成します。 定義上、これは誤った要求であり、このトランザクションを実行するとエラーが返されるはずです。

1. **script.js** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };

    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: saddle },
        { operationType: BulkOperationType.Create, resourceBody: handlebar },
    ];
    ```

1. 異なるパーティション キー値を持つ新しい **flckering strobe light** と **new helmet** を作成するようにスクリプトを変更します。

    ```javascript
    const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    ```

1. **partition_key** という名前の文字列変数を **9603ca6c-9e28-4a02-9194-51cdb7fea816** という値で作成します。

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. **light** 項目と **helmet** 項目を使用して新しいバッチを作成します。

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: light },
        { operationType: BulkOperationType.Create, resourceBody: helmet },
    ];
    ```

1. 完了すると、コード ファイルが次のようになるはずです。

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: light },
            { operationType: BulkOperationType.Create, resourceBody: helmet },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. スクリプトを**保存**してもう一度実行します。

    ```bash
    node script.js
    ```

1. ターミナルからの出力を確認します。 項目の状態コードは、**Failed Dependency** の場合は **424**、**Bad Request** の場合は **400** である必要があります。 これは、トランザクション内のすべての項目で、トランザクション バッチと同じパーティション キー値が共有されなかったため発生しました。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
