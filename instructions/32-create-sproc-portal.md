---
lab:
  title: Azure portal でストアド プロシージャを作成する
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB SQL API
ms.openlocfilehash: 0f390ab5e3bd796c3bfb17a060e2db4424e222e2
ms.sourcegitcommit: 58caf52fefd9f9cbeeef3629e98245544a299b44
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/02/2022
ms.locfileid: "146027671"
---
# <a name="create-a-stored-procedure-with-the-azure-portal"></a>Azure portal でストアド プロシージャを作成する

ストアド プロシージャは、Azure Cosmos DB でサーバー側のビジネス ロジックを実行するための方法の 1 つです。 ストアド プロシージャを使用すると、1 つのトランザクション スコープ内の複数のドキュメントに対して 1 つのコンテナーを使用して基本的な CRUD (作成、読み取り、更新、削除) 操作を実行できます。

このラボでは、コンテナー内にドキュメントを作成するためのストアド プロシージャを作成します。 次に、SQL クエリを使用して、ストアド プロシージャの結果を検証します。

## <a name="author-a-stored-procedure"></a>ストアド プロシージャを選択する

ストアド プロシージャは、統合言語 JavaScript で作成され、データベース エンジン内での基本的な CRUD 操作の実行をサポートします。 データベース エンジン内で JavaScript を実行することは、Azure Cosmos DB 用のサーバー側 JavaScript SDK および一連のヘルパー メソッドを使用することで可能になります。

1. 新しい Web ブラウザー ウィンドウまたはタブ内で、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成し、残りのすべての設定を既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **アカウント名** | ''*グローバルに一意の名前を入力します*'' |
    | **場所** | ''*使用可能なリージョンを選びます*'' |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースにアクセスし、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、 **[新しいコンテナー]** を選択してから、次の設定で新しいコンテナーを作成します。残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **データベース ID** | 新しい &vert; *cosmicworks* *を作成する* |
    | **コンテナー間でスループットを共有する** | ''*このオプションを選択します*'' |
    | **データベースのスループット** | *手動* &vert; *400* |
    | **コンテナー ID** | *products* |
    | **インデックス作成** | *自動* |
    | **パーティション キー** | */categoryId* |

1. 引き続き **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開してから、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを選択します。

1. **[新しいストアド プロシージャ]** を選択します。

1. **[ストアド プロシージャ ID]** フィールドに、値 **createDoc** を入力します。

1. エディター領域のコンテンツを削除します。

1. 次のように、入力パラメーターを指定せずに **createDoc** という名前の新しい JavaScript 関数を作成します。

    ```
    function createDoc() {
        
    }
    ```

1. **createDoc** 関数内で、組み込みの [getContext][azure.github.io/azure-cosmosdb-js-server/global.html] メソッドを呼び出し、その結果を **context** という名前の変数に格納します。

    ```
    var context = getContext();
    ```

1. コンテキスト オブジェクトの [getCollection][azure.github.io/azure-cosmosdb-js-server/context.html] メソッドを呼び出し、その結果を **container** という名前の変数に格納します。

    ```
    var container = context.getCollection();
    ```

1. 次の 2 つのプロパティを持つ **doc** という名前の新しいオブジェクトを作成します。

    | **プロパティ** | **Value** |
    | ---: | :--- |
    | **名前** | *first document* |
    | **カテゴリ ID** | *demo* |

    ```
    var doc = {
        name: 'first document',
        categoryId: 'demo'
    };
    ```

1. コンテナー オブジェクトの **getSelfLink** メソッドを呼び出した結果と、新しいドキュメントをパラメーターとして渡して、コンテナー オブジェクトの **createDocument** メソッドを呼び出します。

    ```
    container.createDocument(
      container.getSelfLink(),
      doc
    );
    ```

1. 完了すると、ストアド プロシージャ コードには、次の内容が含まれるようになります。

    ```
    function createDoc() {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: 'first document',
        categoryId: 'demo'
      };
      container.createDocument(
        container.getSelfLink(),
        doc
      );
    }
    ```

1. **[保存]** を選択して、ストアド プロシージャに加えた変更を保持します。

1. **[実行]** を選択してから、次の入力パラメーターを使用してストアド プロシージャを実行します。

    | **設定** | **キー** | **Value** |
    | ---: | :--- | :--- |
    | **パーティション キー値** | *String* | *demo* |

1. 空の結果を確認します。 ストアド プロシージャの実行が成功したにもかかわらず、JavaScript コードから、人が判読できる応答は返されませんでした。

## <a name="implement-best-practices-for-a-stored-procedure"></a>ストアド プロシージャのベスト プラクティスを実装する

このラボで先に作成したストアド プロシージャには、基本的な機能が含まれているだけであり、すべてのストアド プロシージャに実装する必要がある一般的なエラー処理手法は欠落しています。 まず、ストアド プロシージャでは、操作を完了するための時間が常に確保されることを前提としています。十分な時間を確保できるように、**createDocument** メソッドの戻り値はチェックしません。 次に、ストアド プロシージャでは、すべてのドキュメントの挿入が成功していることを前提としていて、潜在的なエラー メッセージの確認またはスローは行われません。 最後に、最初にストアド プロシージャを呼び出した要求に対する HTTP 応答として、新しく作成されたドキュメントが、ストアド プロシージャから返されることはありません。 ストアド プロシージャに対して次の 3 つの変更を加えて、一般的なベスト プラクティスを実装します。

1. **createDoc** ストアド プロシージャに対するエディターに戻ります。

1. **createDoc** 関数を定義する、コード内の 1 行目を探します。

    ```
    function createDoc() {
    ```

    そのコード行を次のように更新して、**title** という名前のパラメーターを含めます。

    ```
    function createDoc(title) {
    ```

1. **doc** オブジェクトの **name** プロパティを設定する、コード内の 5 行目を探します。

    ```
    name: 'first document',
    ```

    そのコード行を次のように更新して、**title** パラメーターの値を使用するようにします。

    ```
    name: title,
    ```

1. **createDocument** メソッドを呼び出す、コード内の 8 行目を見つけます。

    ```
    container.createDocument(
    ```

    次のようにコード行を更新して、メソッド呼び出しの結果を **accepted** という名前の変数に格納するようにします。

    ```
    var accepted = container.createDocument(
    ```

1. **createDocument** メソッドの呼び出しの後に、次のような新しいコード行を追加することで、**accepted** 変数の値を確認し、true でない場合にそのメソッドを返すようにします。

    ```
    if (!accepted) return;
    ```

1. 最後に、**createDocument** メソッドの呼び出しに 3 番目のパラメーターを追加します。これは、**error** および **newDoc** という名前の 2 つのパラメーターを受け取り、エラーが null かどうかを確認してから、ストアド プロシージャの応答本文に newDoc を設定するための関数です。

    ```
    (error, newDoc) => {
      if (error) throw new Error(error.message);
      context.getResponse().setBody(newDoc);
    }
    ```

1. 完了すると、ストアド プロシージャ コードには、次の内容が含まれるようになります。

    ```
    function createDoc(title) {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: title,
        categoryId: 'demo'
      }
      var accepted = container.createDocument(
        container.getSelfLink(),
        doc,
        (error, newDoc) => {
          if (error) throw new Error(error.message);
          context.getResponse().setBody(newDoc);
        }
      );
      if (!accepted) return;
    }
    ```

1. **[更新]** を選択して、ストアド プロシージャに加えた変更を保持します。

1. **[実行]** を選択してから、次の入力パラメーターを使用してストアド プロシージャを実行します。

    | **設定** | **キー** | **Value** |
    | ---: | :--- | :--- |
    | **パーティション キー値** | *String* | *demo* |
    | **入力パラメーター。** | *String* | *2 番目のドキュメント* |

1. JSON の結果を確認します。 ストアド プロシージャの実行が成功すると、最初の HTTP 要求に対する応答として、新しく作成されたドキュメントが返されました。

## <a name="query-documents"></a>ドキュメントにクエリを実行する

最後に、データ エクスプローラー を使用して、このラボで作成した 2 つのドキュメントを返す SQL クエリを発行します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開して、**SQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択します。

1. **[新しい SQL クエリ]** を選択します。

1. エディター領域のコンテンツを削除します。

1. すべてのドキュメントを返す、次のような新しい SQL クエリを作成します。ここで、**categoryId** は **demo** と同じです。

    ```
    SELECT * FROM docs WHERE docs.categoryId = 'demo'
    ```

1. **[クエリの実行]** を選択します。

1. このクエリを実行した結果が、このラボで作成した 2 つのドキュメントであることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[azure.github.io/azure-cosmosdb-js-server/context.html]: https://azure.github.io/azure-cosmosdb-js-server/Context.html
[azure.github.io/azure-cosmosdb-js-server/global.html]: https://azure.github.io/azure-cosmosdb-js-server/global.html
