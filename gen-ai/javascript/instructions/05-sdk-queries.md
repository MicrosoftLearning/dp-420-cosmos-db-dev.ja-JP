---
lab:
  title: 05 - Azure Cosmos DB for NoSQL SDK を使用してクエリを実行する
  module: Query the Azure Cosmos DB for NoSQL
---

# Azure Cosmos DB for NoSQL SDK を使用してクエリを実行する

最新バージョンの JavaScript SDK for Azure Cosmos DB for NoSQL を使用すると、JavaScript の最新機能を使用したコンテナーのクエリと結果セットの反復処理が簡単になります。

`@azure/cosmos` ライブラリには、Azure Cosmos DB のクエリを効率的かつ直感的に実行できる機能が組み込まれています。

このラボでは、反復子を使用して、Azure Cosmos DB for NoSQL から返された大きな結果セットを処理します。 JavaScript SDK を使用して、結果のクエリと反復処理を行います。

## 開発環境を準備する

「**Azure Cosmos DB を使用して Coplilot を構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照して実行します。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用して Copilot を構築する**」ラボ用に Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#create-azure-cosmos-db-database-and-container-with-sample-data)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## サンプル データを使用して Azure Cosmos DB のデータベースとコンテナーを作成する

既に **cosmicworks-full** という名前の Azure Cosmos DB データベースを作成し、さらにサンプル データが事前に読み込まれている **products** という名前のコンテナーをそのデータベース内に作成している場合は、それらをこのラボに使用して、[次のセクション](#import-the-azurecosmos-library)に進むことができます。 それ以外の場合は、次の手順に従って新しいサンプル データベースとコンテナーを作成してください。

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

## @azure/cosmos ライブラリをインポートする

**@azure/cosmos** ライブラリは **npm** で使用でき、JavaScript プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/05-sdk-queries** フォルダーを参照します。

1. **javascript/05-sdk-queries** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリがあらかじめ **javascript/05-sdk-queries** フォルダーに設定されてターミナルが開きます。

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

## SDK を使用して SQL クエリの結果を反復処理する

新しく作成したアカウントの資格情報を使用して、SDK クラスに接続し、前の手順でプロビジョニングしたデータベースとコンテナーに接続し、SDK を使用して SQL クエリの結果を反復処理します。

ここでは、反復子を使用して、Azure Cosmos DB でのページ分割された結果に対して、わかりやすいループを作成します。 バックグラウンドでは、SDK でフィード反復子を管理し、後続の要求が正しく呼び出されるようにします。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/05-sdk-queries** フォルダーを参照します。

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

1. **queryContainer** という名前の新しいメソッドを作成し、スクリプトの実行時にそのメソッドを実行するコードを作成します。 このメソッド内でコンテナーにクエリを実行するコードを追加します。

    ```javascript
    async function queryContainer() {
        // Query the container
    }

    queryContainer().catch((error) => {
        console.error(error);
    });
    ```

1. **queryContainer** メソッド内に、次のコードを追加して、先ほど作成したデータベースとコンテナーに接続します。

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. `SELECT * FROM products p`の値を持つ `sql` という名前のクエリ文字列変数を作成します。

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. コンストラクターへのパラメーターとして `sql` 変数を使用して、[`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1) メソッドを呼び出します。 `enableCrossPartitionQuery` パラメーターを `true` に設定すると、複数の要求を送信して Azure Cosmos DB サービスでクエリを実行できます。 クエリのスコープが 1 つのパーティション キー値でない場合は、複数の要求が必要です。

    ```javascript
    const iterator = container.items.query(
        query,
        { enableCrossPartitionQuery: true }
    );
    ```

1. ページ分割された結果を反復処理し、各項目の `id`、`name`、および `price` を出力します。

    ```javascript
    while (iterator.hasMoreResults()) {
        const { resources } = await iterator.fetchNext();
        for (const item of resources) {
            console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
        }
    }
    ```

1. **script.js** ファイルは次のようになるはずです。

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function queryContainer() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products p";
    
        const iterator = container.items.query(
            query,
            { enableCrossPartitionQuery: true }
        );
        
        while (iterator.hasMoreResults()) {
            const { resources } = await iterator.fetchNext();
            for (const item of resources) {
                console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
            }
        }
    }
    
    queryContainer().catch((error) => {
        console.error(error);
    });
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

1. これで、スクリプトによって、コンテナー内のすべての製品が出力されます

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
