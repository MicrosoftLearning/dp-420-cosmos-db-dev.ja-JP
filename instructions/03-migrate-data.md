---
lab:
  title: Azure Data Factory を使用して既存のデータを移行する
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Azure Data Factory を使用して既存のデータを移行する

Azure Data Factory では、Azure Cosmos DB がデータ取り込みのソースとして、またデータ出力のターゲット (シンク) としてサポートされています。

このラボでは、便利なコマンドライン ユーティリティを使用して Azure Cosmos DB を設定し、Azure Data Factory を使用してデータのサブセットをコンテナー間で移動します。

## Azure Cosmos DB for NoSQL アカウントを作成してシードする

コマンドライン ユーティリティを使用して、**4,000** RU/s (秒あたりの要求ユニット数) の **cosmicworks** データベースと **products** コンテナーを作成します。 作成後、スループットを調整して 400 RU/s に下げます。

products コンテナーを加えるために、このラボの最後に ETL 変換と読み込み操作のターゲットとなる **flatproducts** コンテナーを手動で作成します。

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
    | **このアカウントでプロビジョニングできるスループットの総量を制限する** | *Unchecked* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. [**プライマリ接続文字列**] フィールドに注目します。 この**接続文字列**の値は、この演習で後ほど使用します。

1. 後で戻るので、ブラウザーのタブは開いたままにしておきます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]を参照してください

1. **Visual Studio Code** で、 **[ターミナル]** メニューを開き、 **[新しいターミナル]** を選択して新しいターミナル インスタンスを開きます。

1. コンピューターでグローバルに使用するために [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをインストールします。

    ```powershell
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

1. Web ブラウザーに戻り、新しいタブを開いて、Azure portal (``portal.azure.com``) に移動します。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks**データベース ノード、**products** コンテナー ノードの順に展開してから、 **[項目]** を選択します。

1. **products** コンテナー内のさまざまな JSON 項目を確認して選択します。 これらは、前の手順で使用したコマンドライン ツールによって作成された項目です。

1. **[スケール]** ノードを選択します。 **[スケール]** タブで、 **[手動]** を選択し、**[必要なスループット]** 設定を **4000 RU/秒** から **400 RU/秒** に更新し、変更内容を **[保存]** します。

1. **[データ エクスプローラー]** ペインで、 **[新しいコンテナー]** を選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | ''*既存の* &vert; *cosmicworks を使用します*'' |
    | **コンテナー ID** | *`flatproducts`* |
    | **パーティション キー** | *`/category`* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開し、階層内の **flatproducts** コンテナー ノードを確認します。

1. Azure portal の **[ホーム]** に戻ります。

## Azure Data Factory リソースを作成する

これで Azure Cosmos DB for NoSQL リソースが配置されたので、Azure Data Factory リソースを作成し、NoSQL API コンテナー間で 1 回限りのデータ移動を実行してデータの抽出、変換、および別の NoSQL API コンテナーへの読み込みを行うために必要なすべてのコンポーネントと接続を構成します。

1. [**+ リソースの作成**] を選択し、*Data Factory* を検索してから、次の設定で新しい **Data Factory** リソースを作成し、残りのすべての設定を既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **名前** | ''*グローバルに一意の名前を入力します*'' |
    | **リージョン** | ''*使用可能なリージョンを選びます*'' |
    | **Version** | *V2* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Data Factory** リソースに移動し、[**スタジオの起動**] を選択します。

    > &#128161; または、(``adf.azure.com/home``) に移動して、新しく作成された Data Factory リソースを選択し、ホーム アイコンを選択することができます。

1. ホーム画面から、 **[取り込み]** オプションを選択してクイック ウィザードを起動し、スケール操作で 1 回限りのデータ コピーを実行し、ウィザードの **[プロパティ]** ステップに進みます。

1. まず、ウィザードの **[プロパティ]** ステップで、 **[タスクの種類]** セクションの **[組み込みコピー タスク]** を選択します。

1. **[タスクの周期またはスケジュール]** セクションで、 **[今すぐ実行]** を選択し、 **[次へ]** を選択してウィザードの **[ソース]** ステップに進みます。

1. ウィザードの [**ソース**] ステップの [**ソースの種類**] 一覧で、[**Azure Cosmos DB for NoSQL**] を選択します。

1. **[接続]** セクションで、 **[+ 新しい接続]** を選択します。

1. [**新しい接続 (Azure Cosmos DB for NoSQL)**] ポップアップで、次の値を使用して新しい接続を構成し、[**作成**] を選択します。

    | **設定** | **Value** |
    | ---: | :--- |
    | **名前** | *`CosmosSqlConn`* |
    | **統合ランタイム経由で接続** | *AutoResolveIntegrationRuntime* |
    | **認証方法** | ''*アカウント キー* &vert; *接続文字列*'' |
    | **Account selection method (アカウントの選択方法)** | *Azure サブスクリプションから* |
    | **Azure サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **Azure Cosmos DB アカウント名** | ''*このラボで先ほど選択した既存の Azure Cosmos DB アカウント名*'' |
    | **データベース名** | *cosmicworks* |

1. **[ソース データ ストア]** セクションに戻り、 **[ソース テーブル]** セクション内で **[クエリの使用]** を選択します。

1. **[テーブル名]** 一覧で、 **[products]** を選択します。

1. **クエリ** エディターで、既存のコンテンツを削除し、次のクエリを入力します。

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. **[データのプレビュー]** を選択して、クエリの有効性をテストします。 **[次へ]** を選択して、ウィザードの **[ターゲット]** ステップに進みます。

1. ウィザードの [**宛先**] ステップの [**宛先の種類**] 一覧で、[**Azure Cosmos DB for NoSQL**] を選択します。

1. **[接続]** 一覧で、 **[CosmosSqlConn]** を選択します。

1. **[ターゲット]** 一覧で、 **[flatproducts]** を選択してから、 **[次へ]** を選択してウィザードの **[設定]** ステップに進みます。

1. ウィザードの **[設定]** ステップで、 **[タスク名]** フィールドに「 **`FlattenAndMoveData`** 」と入力します。

1. 残りのすべてのフィールドは既定の空白値のままにし、 **[次へ]** を選択してウィザードの最後のステップに進みます。

1. ウィザードで選択したステップの **[概要]** を確認し、 **[次へ]** を選択します。

1. デプロイのさまざまなステップを確認します。 デプロイが完了したら、 **[完了]** を選択します。

1. **Azure Cosmos DB アカウント**があるブラウザー タブに戻り、**データ エクスプローラー** ウィンドウに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**flatproducts** コンテナー ノードを選択してから、 **[新しい SQL クエリ]** を選択します。

1. エディター領域のコンテンツを削除します。

1. **名前**が **HL ヘッドセット**と同じであるすべてのドキュメントを返す新しい SQL クエリを作成します。

    ```
    SELECT 
        p.name, 
        p.category, 
        p.price 
    FROM
        flatproducts p
    WHERE
        p.name = 'HL Headset'
    ```

1. **[クエリの実行]** を選択します。

1. クエリの結果を確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
