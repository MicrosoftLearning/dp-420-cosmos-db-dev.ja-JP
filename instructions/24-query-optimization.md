---
lab:
  title: Azure Cosmos DB for NoSQL コンテナーのインデックス ポリシーを特定のクエリ用に最適化する
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Azure Cosmos DB for NoSQL コンテナーのインデックス作成ポリシーをクエリ用に最適化する

Azure Cosmos DB for NoSQL アカウントを計画するときは、最も一般的なクエリを把握することにより、クエリのパフォーマンスができるだけ向上するように、インデックス作成ポリシーを調整することができます。

このラボでは、データ エクスプローラーを使用して、既定のインデックス作成ポリシーと複合インデックスを含むインデックス作成ポリシーを使用して SQL クエリをテストします。

## Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API (たとえば、**Mongo API** や **NoSQL API**) を選択します。 Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続する際に使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索して、新しい **Azure Cosmos DB for NoSQL** アカウント リソースを作成します。以下を設定して、残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **アカウント名** | ''*グローバルに一意の名前を入力します*'' |
    | **場所** | ''*使用可能なリージョンを選びます*'' |
    | **容量モード** | *サーバーレス* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースにアクセスし、**[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** ペインで、 **[新しいコンテナー]** を選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | *新規作成* &vert; *``cosmicworks``* |
    | **コンテナー ID** | *``products``* |
    | **パーティション キー** | *``/categoryId``* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認します。

1. リソース ブレードで、**[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. [**URI**] フィールドに注目します。 この**エンドポイント**の値は、この演習で後ほど使用します。

    1. [**主キー**] フィールドに注目します。 この**キー**の値は、この演習で後ほど使用します。

1. **Visual Studio Code** を開きます。

## Azure Cosmos DB for NoSQL アカウントにサンプル データをシードする

**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。 ツールで一連の項目を作成して、ターミナル ウィンドウで実行されている変更フィード プロセッサを使用して確認します。

1. **Visual Studio Code**で、**[ターミナル]** メニューを開き、**[ターミナルの分割]** を選択して、新しいターミナルを既存のインスタンスと並べて開きます。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをインストールして、マシンにグローバルに使用できるようにします。

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; このコマンドが完了するまで数分かかる場合があります。 過去にこのツールの最新バージョンを既にインストールしている場合は、このコマンドによって警告メッセージ (*ツール 'cosmicworks' は既にインストールされています) が出力されます。

1. cosmicworks を実行し、次のコマンドライン オプションを使用して Azure Cosmos DB アカウントをシードします。

    | **オプション** | **Value** |
    | ---: | :--- |
    | **--endpoint** | ''*このラボで先ほどコピーしたエンドポイントの値*'' |
    | **--key** | ''*このラボで先ほどコピーしたキーの値*'' |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; たとえば、エンドポイントが **https&shy;://dp420.documents.azure.com:443/** で、キーが **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドによって、データベース、コンテナー、および項目がアカウントに設定されるまで待ちます。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じて、ブラウザーに戻ります。

## SQL のクエリを実行し、要求ユニットの料金を測定する

インデックス作成ポリシーを変更する前に、まず、いくつかのサンプル SQL クエリを実行して、ru に示されているベースライン要求ユニットの料金を取得します。インデックス作成ポリシーを変更する前に、まず、いくつかのサンプル SQL クエリを実行して、RU で表されるベースライン要求ユニットの料金を取得します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択してから、**[新しい SQL クエリ]** を選択します。

1. **[クエリの実行]** を選択して、既定のクエリを実行します。

    ```
    SELECT * FROM c
    ```

1. クエリの結果を確認します。 **[クエリ統計]** を選択すると、RU の要求ユニットの料金が表示されます。

1. エディター領域のコンテンツを削除します。

1. すべてのドキュメントから 3 つの値をすべて返す新しい SQL クエリを作成します。

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p    
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果と統計を確認します。 要求ユニットの料金は、最初のクエリとほぼ同じです。

1. エディター領域のコンテンツを削除します。

1. すべてのドキュメントから 3 つの値を **categoryName** 順に並べ替えて返す新しい SQL クエリを作成します。

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果と統計を確認します。 **ORDER BY** 句が原因で、要求ユニットの料金が増加しています。

## インデックス作成ポリシーで複合インデックスを作成する

ここで、複数のプロパティを使用して項目を並べ替える場合は、複合インデックスを作成する必要があります。 このタスクでは、項目を categoryName で並べ替えてから、実際の名前で並べ替える複合インデックスを作成します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択してから、**[新しい SQL クエリ]** を選択します。

1. エディター領域のコンテンツを削除します。

1. 結果を最初に **categoryName** の降順で並べ替え、次に **price** の昇順で並べ替える新しい SQL クエリを作成します。

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. **[クエリの実行]** を選択します。

1. クエリは、**並べ替えクエリには、サービスを提供できる対応する複合インデックスがありません**というエラーが発生し、失敗します。

1. **[データ エクスプローラー]** で、**cosmicworks**データベース ノード、**products** コンテナー ノードの順に展開してから、**[設定]** を選択します。

1. **[設定]** タブで、**[インデックス作成ポリシー]** セクションに移動します。

1. 既定のインデックス作成ポリシーを確認します。

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

1. インデックス作成ポリシーを、次の変更した JSON オブジェクトに置き換え、変更内容を**保存**します。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択してから、**[新しい SQL クエリ]** を選択します。

1. エディター領域のコンテンツを削除します。

1. 結果を最初に **categoryName** の降順で並べ替え、次に **price** の昇順で並べ替える新しい SQL クエリを作成します。

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果と統計を確認します。 複合インデックスが配置されたので、要求ユニットの料金は低くなります。

1. エディター領域のコンテンツを削除します。

1. 結果を最初に **categoryName** の降順で、次に **name** の昇順で、最後に **price** の昇順で並べ替える新しい SQL クエリを作成します。

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. **[クエリの実行]** を選択します。

1. クエリは、**並べ替えクエリには、サービスを提供できる対応する複合インデックスがありません**というエラーが発生し、失敗します。

1. **[データ エクスプローラー]** で、**cosmicworks**データベース ノード、**products** コンテナー ノードの順に展開してから、**[設定]** を再度選択します。

1. **[設定]** タブで、**[インデックス作成ポリシー]** セクションに移動します。

1. インデックス作成ポリシーを、次の変更した JSON オブジェクトに置き換え、変更内容を**保存**します。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/name",
            "order": "ascending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択してから、**[新しい SQL クエリ]** を選択します。

1. エディター領域のコンテンツを削除します。

1. 結果を最初に **categoryName** の降順で、次に **name** の昇順で、最後に **price** の昇順で並べ替える新しい SQL クエリを作成します。

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果と統計を確認します。 複合インデックスが配置されたので、要求ユニットの料金は低くなります。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
