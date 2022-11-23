---
lab:
  title: Azure Cosmos DB SQL API コンテナーのインデックス作成ポリシーをクエリ用に最適化する
  module: Module 10 - Optimize query performance in Azure Cosmos DB SQL API
---

# <a name="optimize-an-azure-cosmos-db-sql-api-containers-indexing-policy-for-a-query"></a>Azure Cosmos DB SQL API コンテナーのインデックス作成ポリシーをクエリ用に最適化する

Azure Cosmos DB SQL API アカウントを計画するときは、最も一般的なクエリを把握することにより、クエリのパフォーマンスをできるだけ向上するように、インデックス作成ポリシーを調整することができます。

このラボでは、データ エクスプローラーを使用して、既定のインデックス作成ポリシーと複合インデックスを含むインデックス作成ポリシーを使用して SQL クエリをテストします。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API を選択します (たとえば、**Mongo API** または **SQL API**)。 Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB SQL API アカウントに接続する場合にそれらを使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (`portal.azure.com`) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、_Cosmos DB_ を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成し、残りのすべての設定を既定値のままにします。

   |               **設定** | **Value**                                                         |
   | ---------------------: | :---------------------------------------------------------------- |
   | **サブスクリプション** | ''_既存の Azure サブスクリプション_''                             |
   |  **リソース グループ** | ''_既存のリソース グループを選択するか、新しいものを作成します_'' |
   |       **アカウント名** | ''_グローバルに一意の名前を入力します_''                          |
   |               **場所** | ''_使用可能なリージョンを選びます_''                              |
   |         **容量モード** | _サーバーレス_                                                    |

   > &#128221; お使いのラボ環境では、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースにアクセスし、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** ペインで、 **[New Container]** を選択します。

1. **[New Container]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

   |                **設定** | **Value**                                |
   | ----------------------: | :--------------------------------------- |
   |     **データベース ID** | Create new : _cosmicworks_ |
   |       **コンテナー ID** | _products_                               |
   | **パーティション キー** | _/categoryId_                            |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開し、階層内の **products** コンテナー ノードを確認します。

1. リソース ブレードで、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

   1. **[URI]** フィールドの値を記録します。 この **エンドポイント** の値は、この演習で後ほど使用します。

   1. **[主キー]** フィールドの値を記録します。 この **キー** の値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="seed-your-azure-cosmos-db-sql-api-account-with-sample-data"></a>サンプル データを使用して Azure Cosmos DB SQL API アカウントにシードを設定する

**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。 ツールで一連の項目を作成して、ターミナル ウィンドウで実行されている変更フィード プロセッサを使用して確認します。

1. **Visual Studio Code** で、 **[ターミナル]** メニューを開き、 **[ターミナルの分割]** を選択して、新しいターミナルを既存のインスタンスと並べて開きます。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをインストールして、マシンにグローバルに使用できるようにします。

   ```
   dotnet tool install --global cosmicworks
   ```

   > &#128161; このコマンドが完了するまで数分かかる場合があります。 過去にこのツールの最新バージョンを既にインストールしている場合は、このコマンドによって警告メッセージ (\*ツール 'cosmicworks' は既にインストールされています) が出力されます。

1. cosmicworks を実行し、次のコマンドライン オプションを使用して Azure Cosmos DB アカウントをシードします。

   | **オプション** | **Value**                                          |
   | -------------: | :------------------------------------------------- |
   | **--endpoint** | ''_このラボで先ほどコピーしたエンドポイントの値_'' |
   |      **--key** | ''_このラボで先ほどコピーしたキーの値_''           |
   | **--datasets** | _product_                                          |

   ```
   cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
   ```

   > &#128221; たとえば、エンドポイントが **https&shy;://dp420.documents.azure.com:443/** で、キーが **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。`cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product`

1. **cosmicworks** コマンドによって、データベース、コンテナー、および項目がアカウントに設定されるまで待ちます。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

## <a name="execute-sql-queries-and-measure-their-request-unit-charge"></a>SQL のクエリを実行し、要求ユニットの料金を測定する

インデックス作成ポリシーを変更する前に、まず、いくつかのサンプル SQL クエリを実行して、ru に示されているベースライン要求ユニットの料金を取得します。インデックス作成ポリシーを変更する前に、まず、いくつかのサンプル SQL クエリを実行して、RU で表されるベースライン要求ユニットの料金を取得します。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (`portal.azure.com`) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択してから、 **[New SQL Query]** を選択します。

1. **[Execute Query]** を選択して、既定のクエリを実行します。

   ```
   SELECT * FROM c
   ```

1. クエリの結果を確認します。 **[Query Stats]** を選択すると、RU の要求ユニットの料金が表示されます。

1. エディター領域の内容を削除します。

1. **名前** が **HL Headset** と同じであるすべてのドキュメントを返す新しい SQL クエリを作成します。

   ```
   SELECT
       p.name,
       p.categoryName,
       p.price
   FROM
       products p
   ```

1. **[Execute Query]** を選択します。

1. クエリの結果と統計を確認します。 要求ユニットの料金は、最初のクエリとほぼ同じです。

1. エディター領域の内容を削除します。

1. **名前** が **HL Headset** と同じであるすべてのドキュメントを返す新しい SQL クエリを作成します。

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

1. **[Execute Query]** を選択します。

1. クエリの結果と統計を確認します。 **ORDER BY** 句が原因で、要求ユニットの料金が増加しています。

## <a name="create-a-composite-index-in-the-indexing-policy"></a>インデックス作成ポリシーで複合インデックスを作成する

ここで、複数のプロパティを使用して項目を並べ替える場合は、複合インデックスを作成する必要があります。 このタスクでは、項目を categoryName で並べ替えてから、実際の名前で並べ替える複合インデックスを作成します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択してから、 **[New SQL Query]** を選択します。

1. エディター領域の内容を削除します。

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

1. **[Execute Query]** を選択します。

1. クエリは、**並べ替えクエリには、サービスを提供できる対応する複合インデックスがありません** というエラーが発生し、失敗します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノード、**products** コンテナー ノードの順に展開してから、 **[Settings]** を選択します。

1. **[Settings]** タブで、 **[Indexing Policy]** セクションに移動します。

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

1. インデックス作成ポリシーを、次の変更した JSON オブジェクトに置き換え、変更内容を **Save** します。

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

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択してから、 **[New SQL Query]** を選択します。

1. エディター領域の内容を削除します。

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

1. **[Execute Query]** を選択します。

1. クエリの結果と統計を確認します。 複合インデックスが配置されたので、要求ユニットの料金は低くなります。

1. エディター領域の内容を削除します。

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

1. **[Execute Query]** を選択します。

1. エラーを確認します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノード、**products** コンテナー ノードの順に展開してから、 **[Settings]** を再度選択します。

1. **[Settings]** タブで、 **[Indexing Policy]** セクションに移動します。

1. インデックス作成ポリシーを、次の変更した JSON オブジェクトに置き換え、変更内容を **Save** します。

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

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択してから、 **[New SQL Query]** を選択します。

1. エディター領域の内容を削除します。

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

1. **[Execute Query]** を選択します。

1. クエリの結果と統計を確認します。 

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
