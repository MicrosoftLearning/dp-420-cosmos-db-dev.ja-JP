---
title: 01 - SDK を使って Azure Cosmos DB for NoSQL に接続する
lab:
  title: 01 - SDK を使って Azure Cosmos DB for NoSQL に接続する
  module: Use the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 4
parent: JavaScript SDK labs
---

# SDK を使って Azure Cosmos DB for NoSQL に接続する

Azure SDK for JavaScript (Node.js & Browser) は、多くの Azure サービスと対話するための一貫した開発者インターフェイスを提供するクライアント ライブラリのスイートです。 クライアント ライブラリは、これらのリソースを使用して操作するために使用するパッケージです。

このラボでは、Azure SDK for JavaScript を使用して Azure Cosmos DB for NoSQL アカウントに接続します。

## 開発環境を準備する

「**Azure Cosmos DB を使用して Coplilot を構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照して実行します。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用してコパイロットを構築する**」ラボ用の Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#import-the-azurecosmos-library)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## @azure/cosmos ライブラリをインポートする

**@azure/cosmos** ライブラリは **npm** で使用でき、JavaScript プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/01-sdk-connect** フォルダーを参照します。

1. **javascript/01-sdk-connect** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **javascript/01-sdk-connect** フォルダーに既に設定されているターミナルが開きます。

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

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**javascript/01-sdk-connect** フォルダーを参照します。

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

1. **main** という名前の `async` 関数を追加して、アカウントのプロパティの読み取りと印刷を行います。

    ```javascript
    async function main() {
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }
    ```

1. **main** 関数を呼び出します。

    ```javascript
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
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }

    main().catch((error) => console.error(error));
    ```

1. **script.js** ファイルを**保存**します。

## スクリプトをテストする

これで Azure Cosmos DB for NoSQL アカウントに接続するための JavaScript コードが完成したので、スクリプトをテストできます。 このスクリプトは、既定の整合性レベルと、最初の書き込み可能なリージョンの名前を出力します。 アカウントを作成したときに場所を指定したので、このスクリプトの結果として同じ場所の値が出力されるはずです。

1. **Visual Studio Code** で、**javascript/01-sdk-connect** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. スクリプトを実行する前に、`az login` コマンドを使用して Azure にログインする必要があります。 ターミナル ウィンドウで以下を実行します。

    ```bash
    az login
    ```

1. `node` コマンドを使用してスクリプトを実行します。

    ```bash
    node script.js
    ```

1. スクリプトによって、アカウントの既定の整合性レベルと最初の書き込み可能なリージョンが出力されます。 たとえば、アカウントの既定の整合性レベルが **Session** で、最初の書き込み可能なリージョンが**米国東部**である場合、スクリプトでは次のように出力されます。

    ```text
    Consistency Policy: Session
    Primary Region: East US
    ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
