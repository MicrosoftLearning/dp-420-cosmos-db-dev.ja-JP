---
lab:
  title: ポータルで Azure Cosmos DB SQL API コンテナーの既定のインデックス ポリシーを確認する
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: a5918d41746f82da08d66c486aa53f782d9e8e17
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025088"
---
# <a name="review-the-default-index-policy-for-an-azure-cosmos-db-sql-api-container-with-the-portal"></a>ポータルで Azure Cosmos DB SQL API コンテナーの既定のインデックス ポリシーを確認する

Azure Cosmos DB のすべてのコンテナーには、コンテナー内の項目のインデックスを作成する方法をサービスに指示するインデックス作成ポリシーがあります。 既定では、このインデックス作成ポリシーは、すべての項目のすべてのプロパティにインデックスを付けます。 既定のインデックス作成ポリシーを使用すると、プロジェクトの開始時にインデックス作成、パフォーマンス、および管理について考える必要がないため、Azure Cosmos DB をすばやく簡単に開始できます。

このラボでは、データ エクスプローラーを使用して、いくつかのコンテナーのデフォルトのインデックス ポリシーを監視および操作します。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API を選択します (たとえば、**Mongo API** または **SQL API**)。 Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB SQL API アカウントに接続する場合にそれらを使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、 *[Cosmos DB]* を検索してから、次の設定で新しい **[Azure Cosmos DB SQL API]** アカウント リソースを作成し、残りのすべての設定を規定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | *既存の Azure サブスクリプション* |
    | **リソース グループ** | *既存のリソース グループを選択するか、新しいものを作成します* |
    | **アカウント名** | *グローバルに一意の名前を入力します* |
    | **Location** | *使用可能な任意のリージョンを選択します* |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **[URI]** フィールドの値を記録します。 この **endpoint** の値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この **キー** の値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="seed-the-azure-cosmos-db-sql-api-account-with-data"></a>Azure Cosmos DB SQL API アカウントにデータをシードする

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを使用して、Azure Cosmos DB SQL API アカウントにサンプル データをデプロイします。 このツールはオープンソースで、NuGet から入手できます。 このツールを Azure Cloud Shell にインストールして、データベースのシードに使用します。

1. **Visual Studio Code** を起動します。

1. **Visual Studio Code** で、 **[ターミナル]** メニューを開き、 **[新しいターミナル]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; Visual Studio Code インターフェイスの詳細をまだ十分理解していない場合は、「[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]」を参照してください。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをご自分のコンピューターでグローバルに使用できるようにインストールします。

    ```
    dotnet tool install --global cosmicworks
    ```
  
    > &#128161; このコマンドが完了するまで数分かかる場合があります。 過去にこのツールの最新バージョンを既にインストールしている場合は、このコマンドで、警告メッセージ (*ツール 'cosmicworks' は既にインストールされています) が出力されます。

1. cosmicworks を実行して、次のコマンドライン オプションを使用して Azure Cosmos DB アカウントをシードします。

    | **オプション** | **Value** |
    | ---: | :--- |
    | **--endpoint** | *このラボで先ほどコピーしたエンドポイント値* |
    | **--key** | *このラボで先ほどコピーしたキー値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; たとえば、ご自分のエンドポイントが **https&shy;://dp420.documents.azure.com:443/** で、キーが **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドによって、データベース、コンテナー、および項目がアカウントに設定されるまで待ちます。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

## <a name="view-and-manipulate-the-default-indexing-policy"></a>既定のインデックス作成ポリシーを表示および操作する

コンテナーがコード、ポータル、またはツールによって作成される場合、特に指定しなければ、インデックス作成ポリシーはインテリジェントな既定値に設定されます。 既定のインデックス作成ポリシーを確認し、ポリシーを変更します。

1. Web ブラウザーで、Azure portal (``portal.azure.com``) に移動します。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. **SQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選んでから、 **[新しい SQL クエリ]** を選択します。

1. エディター領域の内容を削除します。

1. **名前** が **HL ヘッドセット** と同じであるすべてのドキュメントを返す新しい SQL クエリを作成します。

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果を確認します。

1. **[クエリ]** タブで、 **[クエリ統計]** を選択します。

1. 引き続き、 **[クエリ]** タブで、 **[クエリ統計]** セクション内の **[要求料金]** フィールドの値を確認します。

    > &#128221; 現在、すべてのパスにインデックスが付けられているため、このクエリは比較的効率的です。

1. **SQL API** ナビゲーション ツリーの **products** コンテナー ノード内で、 **[スケールと設定]** を選択します。

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

    > &#128221; この既定のポリシーでは、 **_etag** を除くすべての可能なパスにインデックスを付けます。

1. エディター内で、インデックス作成ポリシーの内容を置換して、 **/price** パスのみにインデックスを付けます。

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

1. エディター領域の内容を削除します。

1. **名前** が **HL ヘッドセット** と同じであるすべてのドキュメントを返す新しい SQL クエリを作成します。

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果を確認します。

1. **[クエリ]** タブで、 **[クエリ統計]** を選択します。

1. 引き続き、 **[クエリ]** タブで、 **[クエリ統計]** セクション内の **[要求料金]** フィールドの値を確認します。

    > &#128221; **name** プロパティにインデックスが付けられなくなったため、要求料金が高くなりました。

1. エディター領域の内容を削除します。

1. **料金が** **$3,000** を超えるすべてのドキュメントを返す新しい SQL クエリを作成します。

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果を確認します。

1. **[クエリ]** タブで、 **[クエリ統計]** を選択します。

1. 引き続き、 **[クエリ]** タブで、 **[クエリ統計]** セクション内の **[要求料金]** フィールドの値を確認します。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
