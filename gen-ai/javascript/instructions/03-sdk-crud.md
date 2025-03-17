---
lab:
  title: '03: Azure Cosmos DB for NoSQL SDK を使用してドキュメントを作成および更新する'
  module: Implement Azure Cosmos DB for NoSQL point operations
---

# Azure Cosmos DB for NoSQL SDK を使用してドキュメントを作成および更新する

`@azure/cosmos` ライブラリには、Azure Cosmos DB for NoSQL コンテナー内の (CRUD) 項目を作成、取得、更新、および削除するメソッドが含まれています。 これらのメソッドを組み合わせて、NoSQL API コンテナー内のさまざまな項目に対して、最も一般的な "CRUD" 操作の一部を実行します。

このラボでは、JavaScript SDK を使用して、Azure Cosmos DB for NoSQL コンテナー内の項目に対して日常的な CRUD 操作を実行します。

## 開発環境を準備する

「**Azure Cosmos DB を使用して Coplilot を構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照して実行します。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用して Copilot を構築する**」ラボ用に Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#import-the-azurecosmos-library)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## @azure/cosmos ライブラリをインポートする

**@azure/cosmos** ライブラリは **npm** で使用でき、JavaScript プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/03-sdk-crud** フォルダーを参照します。

1. **javascript/03-sdk-crud** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリがあらかじめ **javascript/03-sdk-crud** フォルダーに設定されてターミナルが開きます。

1. 新しい Node.js プロジェクトを初期化します。

    ```bash
    npm init -y
    ```

1. 次のコマンドを使用して [@azure/cosmos][npmjs.com/package/@azure/cosmos] パッケージをインストールします。

    ```bash
    npm install @azure/cosmos
    ```

1. [@azure/identity][npmjs.com/package/@azure/identity] ライブラリをインストールすると、次のコマンドを使用して Azure 認証により Azure Cosmos DB ワークスペースに接続できます。

    ```bash
    npm install @azure/identity
    ```

## @azure/cosmos ライブラリを使用する

Azure SDK for JavaScript から Azure Cosmos DB ライブラリがインポートされたら、すぐにそのクラスを使用して Azure Cosmos DB for NoSQL アカウントに接続できます。 **CosmosClient** クラスは、Azure Cosmos DB for NoSQL アカウントへの初期接続を確立するのに使用するコア クラスです。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/03-sdk-crud** フォルダーを参照します。

1. **script.js** という名前の空 JavaScript ファイルを開きます。

1. 次の `require` ステートメントを追加して、**@azure/cosmos** ライブラリと **@azure/identity** ライブラリをインポートします。

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    const { CosmosClient } = require("@azure/cosmos");
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

## SDK を使用して項目に対して作成および読み取りポイント操作を実行する

ここでは、**Container** クラスのメソッド セットを使用して、NoSQL API コンテナー内の項目に対する一般的な操作を実行します。

1. **Visual Studio Code** に戻ります。 開いたままになっていない場合は、**javascript/03-sdk-crud** フォルダー内の **script.js** コード ファイルを開きます。

1. 新しい製品項目を作成し、それを次のプロパティを使用して **saddle** という名前の変数に割り当てます。 必ず `main` 関数に次のコードを追加してください。

    | プロパティ | 値 |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **Price** | *45.99d* |
    | **tags** | *{ tan, new, crisp }* |

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };
    ```

1. コンテナーの **items** クラスの [`create`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-create) メソッドを呼び出し、メソッド パラメーターとして **saddle** 変数で渡します。

    ```javascript
    const { resource: item } = await container
        .items.create(saddle);
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const saddle = {
            id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
            categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
            name: "Road Saddle",
            price: 45.99,
            tags: ["tan", "new", "crisp"]
        };
    
        const { resource: item } = await container
                .items.create(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. スクリプトを**保存**してもう一度実行します。

    ```bash
    node script.js
    ```

1. **[データ エクスプローラー]** で新しい項目を確認します。

1. **Visual Studio Code** に戻ります。

1. **script.js** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };

    const { resource: item } = await container
            .items.create(saddle);
    ```

1. **item_id** という名前の文字列変数を **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009** という値で作成します。

    ```javascript
    const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. **partition_key** という名前の文字列変数を **9603ca6c-9e28-4a02-9194-51cdb7fea816** という値で作成します。

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. コンテナーの **item** クラスの [`read`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-read) メソッドを呼び出し、メソッド パラメーターとして **itemId** 変数と **partitionKey** 変数で渡します。

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();
    ```

    > &#128161; `read` メソッドを使用すると、コンテナー内の項目に対してポイント読み取り操作を実行できます。 このメソッドでは、読み取る項目を識別するために `itemId` パラメーターと `partitionKey` パラメーターが必要です。 Cosmos DB の SQL クエリ言語を使用して 1 つの項目を検索するクエリを実行するのとは対照的に、`read` メソッドは 1 つの項目を取得するのにより効率的でコスト効率の高い方法です。 ポイント読み取りではデータを直接読み取ることができ、クエリ エンジンが要求を処理する必要はありません。

1. 書式設定された出力文字列を使用して saddle オブジェクトを出力します。

    ```javascript
    print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    ```

1. 完了すると、コード ファイルが次のようになるはずです。

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();
    
        console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. スクリプトを**保存**してもう一度実行します。

    ```bash
    node script.js
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

1. **Visual Studio Code** に戻ります。 **script.js** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```javascript
    console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    ```

1. price プロパティの値を「**32.55**」に設定して、**saddle** 変数を変更します。

    ```javascript
    // Update the item
    saddle.price = 32.55;
    ```

1. **name** プロパティの値を「**Road LL Saddle**」に設定して、**saddle** 変数を再度変更します。

    ```javascript
    saddle.name = "Road LL Saddle";
    ```

1. コンテナーの **item** クラスの [`replace`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-replace) メソッドを呼び出し、メソッド パラメーターとして **saddle** 変数で渡します。

    ```javascript
    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. 完了すると、コード ファイルが次のようになるはずです。

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();

        // Update the item
        saddle.price = 32.55;
        saddle.name = "Road LL Saddle";
    
        await container.item(saddle.id, partitionKey).replace(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. スクリプトを**保存**してもう一度実行します。

    ```bash
    node script.js
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

1. **Visual Studio Code** に戻ります。 **script.js** コード ファイルのエディター タブに戻ります。

1. 次のコード行を削除します。

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();

    // Update the item
    saddle.price = 32.55;
    saddle.name = "Road LL Saddle";

    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. コンテナーの **item** クラスの [`delete`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-delete) メソッドを呼び出し、メソッド パラメーターとして **itemId** 変数と **partitionKey** 変数で渡します。

    ```javascript
    // Delete the item
    await container.item(itemId, partitionKey).delete();
    ```

1. スクリプトを保存してもう一度実行します。

    ```bash
    node script.js
    ```

1. 統合ターミナルを閉じます。

1. Web ブラウザーのウィンドウまたはタブに戻ります。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. [**データ エクスプローラー**] で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開します。

1. **Items** ノードを選択します。 項目リストが空になっていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
