---
lab:
  title: ポータルで Azure Cosmos DB for NoSQL コンテナーの既定のインデックス ポリシーを確認する
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# ポータルで Azure Cosmos DB for NoSQL コンテナーの既定のインデックス ポリシーを確認する

Azure Cosmos DB のすべてのコンテナーには、コンテナー内の項目のインデックスを作成する方法をサービスに指示するインデックス作成ポリシーがあります。 既定では、このインデックス作成ポリシーは、すべての項目のすべてのプロパティにインデックスを付けます。 既定のインデックス作成ポリシーを使用すると、プロジェクトの開始時にインデックス作成、パフォーマンス、および管理について考える必要がないため、Azure Cosmos DB をすばやく簡単に開始できます。

このラボでは、データ エクスプローラーを使用して、いくつかのコンテナーのデフォルトのインデックス ポリシーを監視および操作します。

## Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API (たとえば、**Mongo API** や **NoSQL API**) を選択します。 Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続する際に使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索して、新しい **Azure Cosmos DB for NoSQL** アカウント リソースを作成します。以下を設定して、残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **ワークロードの種類** | **学習** |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **アカウント名** | ''*グローバルに一意の名前を入力します*'' |
    | **場所** | ''*使用可能なリージョンを選びます*'' |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. [**プライマリ接続文字列**] フィールドに注目します。 この**接続文字列**の値は、この演習で後ほど使用します。

## Azure Cosmos DB for NoSQL アカウントにデータをシードする

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを使用して、Azure Cosmos DB for NoSQL アカウントにサンプル データをデプロイします。 このツールはオープンソースで、NuGet から入手できます。 このツールを Azure Cloud Shell にインストールして、データベースのシードに使用します。

1. **Visual Studio Code** を起動します。

1. **Visual Studio Code** で、 **[ターミナル]** メニューを開き、 **[新しいターミナル]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]を参照してください

1. コンピューターでグローバルに使用するために [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをインストールします。

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```
  
    > &#128161; このコマンドが完了するまで数分かかる場合があります。 過去にこのツールの最新バージョンを既にインストールしている場合は、このコマンドによって警告メッセージ (*ツール 'cosmicworks' は既にインストールされています) が出力されます。

1. cosmicworks を実行し、次のコマンドライン オプションを使用して Azure Cosmos DB アカウントをシードします。

    | **オプション** | **Value** |
    | ---: | :--- |
    | **-c** | *このラボで先ほど確認した接続文字列の値* |
    | **--number-of-employees** | *特に指定がない限り、cosmicworks コマンドは、従業員 1000 人と製品コンテナー 200 項目をデータベースに入力します* |

    ```powershell
    cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```

    > &#128221; たとえば、エンドポイントが **https&shy;://dp420.documents.azure.com:443/** で、キーが **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。``cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys``

1. **cosmicworks** コマンドによって、データベース、コンテナー、および項目がアカウントに設定されるまで待ちます。

1. 統合ターミナルを閉じます。

## 既定のインデックス作成ポリシーを表示および操作する

コンテナーがコード、ポータル、またはツールによって作成される場合、特に指定しなければ、インデックス作成ポリシーはインテリジェントな既定値に設定されます。 既定のインデックス作成ポリシーを確認し、ポリシーを変更します。

1. Web ブラウザーに戻ります。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. [**データ エクスプローラー**] で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. **NoSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、[**新しい SQL クエリ**] を選択します。

1. エディター領域のコンテンツを削除します。

1. **名前**が **HL ヘッドセット**と同じであるすべてのドキュメントを返す新しい SQL クエリを作成します。

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果を確認します。

1. **[クエリ]** タブで、**[クエリ統計]** を選択します。

1. [**クエリ統計**] セクション内の [**要求使用量**] フィールドの値を確認します。

    > &#128221; 現在、すべてのパスにインデックスが付けられているため、このクエリは比較的効率的です。

1. **NoSQL API** ナビゲーション ツリーの **products** コンテナー ノード内で、**[設定]** を選択します。

1. **[インデックス作成ポリシー]** セクションで、既定のインデックス作成ポリシーを確認します。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

    > &#128221; この既定のポリシーでは、**_etag** を除くすべての可能なパスにインデックスを付けます。

1. エディター内で、インデックス作成ポリシーの内容を置換して、**/price** パスのみにインデックスを付けます。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/price/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        }
      ]
    }
    ```

1. **[保存]** を選択してご自身の変更を保存します。

1. **[新しい SQL クエリ]** を選択します。

1. エディター領域のコンテンツを削除します。

1. **名前**が **HL ヘッドセット**と同じであるすべてのドキュメントを返す新しい SQL クエリを作成します。

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果を確認します。

1. **[クエリ]** タブで、**[クエリ統計]** を選択します。

1. [**クエリ統計**] セクション内の [**要求使用量**] フィールドの値を確認します。

    > &#128221; **name** プロパティにインデックスが付けられなくなったため、要求料金が高くなりました。

1. エディター領域のコンテンツを削除します。

1. **price** が **$3,000** より大きいすべてのドキュメントを返す新しい SQL クエリを作成します。

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果を確認します。

1. **[クエリ]** タブで、**[クエリ統計]** を選択します。

1. [**クエリ統計**] セクション内の [**要求使用量**] フィールドの値を確認します。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
