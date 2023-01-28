---
lab:
  title: 一般的な操作に対して Azure Cosmos DB for NoSQL コンテナーのインデックス作成ポリシーを最適化する
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# <a name="optimize-an-azure-cosmos-db-for-nosql-containers-indexing-policy-for-common-operations"></a>一般的な操作に対して Azure Cosmos DB for NoSQL コンテナーのインデックス作成ポリシーを最適化する

書き込みの多いワークロードや大きな JSON オブジェクトを扱うワークロードでは、インデックス作成ポリシーをクエリで使用したいことがわかっているプロパティのインデックスのみを作成するように最適化すると都合が良い場合があります。

このラボでは、テスト用の .NET アプリケーションを使って Azure Cosmos DB for NoSQL コンテナーに大きな JSON 項目を挿入します。その際、既定のインデックス作成ポリシーを使用し、次に、少し調整したインデックス作成ポリシーを使用します。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業する環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、これらの手順に従って行います。 それ以外の場合は、以前にクローンされたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[作業の開始に関するドキュメント][code.visualstudio.com/docs/getstarted]をご覧ください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリがクローンされたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="create-an-azure-cosmos-db-for-nosql-account"></a>Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API (たとえば、**Mongo API** や **NoSQL API**) を選択します。 Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の他の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続する際に使用できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、*Cosmos DB* を検索して、新しい **Azure Cosmos DB for NoSQL** アカウント リソースを作成します。以下を設定して、残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | *既存のリソース グループを選択するか、新しいものを作成します* |
    | **アカウント名** | ''*グローバルに一意の名前を入力します*'' |
    | **場所** | *使用可能なリージョンを選びます* |
    | **容量モード** | *サーバーレス* |

    > &#128221; ラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースにアクセスし、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** ペインで、 **[新しいコンテナー]** を選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | *新しい* &vert; *cosmicworks を作成する* |
    | **コンテナー ID** | *製品* |
    | **パーティション キー** | */categoryId* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認します。

1. リソース ブレードで、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **[URI]** フィールドの値を記録します。 この**エンドポイント**の値は、この演習で後ほど使用します。

    1. **[主キー]** フィールドの値を記録します。 この**キー**の値は、この演習で後ほど使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="run-the-test-net-application-using-the-default-indexing-policy"></a>既定のインデックス作成ポリシーを使用してテスト用の .NET アプリケーションを実行する

このラボでは、大きな JSON オブジェクトを受け取り、Azure Cosmos DB for NoSQL コンテナーに新しい項目を作成するテスト用の .NET アプリケーションが事前構築されています。 1 回の書き込み操作が完了すると、このアプリケーションから、項目の一意識別子と RU 料金がコンソール ウィンドウに出力されます。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**23-index-optimization** フォルダーを参照します。

1. **23-index-optimization** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、ターミナルが開き、開始ディレクトリが既に **23-index-optimization** フォルダーに設定されています。

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドします。

    ```
    dotnet build
    ```

    > &#128221; コンパイラから、**endpoint** および **key** 変数は現在使用されていないという警告が表示される場合があります。 これらの変数はこのタスクで使用しますので、この警告は無視しても問題ありません。

1. 統合ターミナルを閉じます。

1. **script.cs** コード ファイルを開きます。

1. **endpoint** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの**エンドポイント**に設定します。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、ご自分のエンドポイントが **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけます。 その値を、先ほど作成した Azure Cosmos DB アカウントの**キー**に設定します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、ご自分のキーが **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **script.cs** コード ファイルを**保存**します。

1. **Visual Studio Code**,で **23-index-optimization** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、プロジェクトをビルドして実行します。

    ```
    dotnet run
    ```

1. ターミナルからの出力を確認します。 項目の一意識別子と操作の要求料金 (RU 単位) がコンソールに出力されているはずです。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、さらに少なくとも 2 回プロジェクトをビルドして実行します。 コンソール出力で RU 料金を確認します。

    ```
    dotnet run
    ```

1. 統合ターミナルは開いたままにします。

    > &#128221; このターミナルは、この演習で後ほど、再度使用します。 元の RU 料金と更新された RU 料金を比較できるよう、ターミナルを開いたままにしておくことが重要です。

## <a name="update-the-indexing-policy-and-rerun-the-net-application"></a>インデックス作成ポリシーを更新し、.NET アプリケーションを再実行する

このラボ シナリオでは、今後のクエリで name および categoryName プロパティに焦点を当てることを想定します。 大きな JSON 項目に最適化するには、すべてのパスを除外して開始するインデックス作成ポリシーを作成して、インデックスから他のすべてのフィールドを除外します。 次に、特定のパスを選択してポリシーに含めます。

1. Web ブラウザーの新しいウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks**データベース ノード、**products** コンテナー ノードの順に展開してから、 **[設定]** を選択します。

1. **[設定]** タブで、 **[インデックス作成ポリシー]** セクションに移動します。

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
          "path": "/name/?"
        },
        {
          "path": "/categoryName/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. **Visual Studio Code** に戻ります。 開いているターミナルに戻ります。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、さらに少なくとも 2 回プロジェクトをビルドして実行します。 コンソール出力で新しい RU 料金を確認します。元の料金よりかなり低くなっているはずです。

    ```
    dotnet run
    ```

    > &#128221;更新された RU 料金が表示されない場合は、数分待つ必要があります。

1. Web ブラウザーの新しいウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks**データベース ノード、**products** コンテナー ノードの順に展開してから、 **[設定]** を選択します。

1. **[設定]** タブで、 **[インデックス作成ポリシー]** セクションに移動します。

1. インデックス作成ポリシーを、次の変更した JSON オブジェクトに置き換え、変更内容を**保存**します。

    ```
    {
      "indexingMode": "none"
    }
    ```

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. **Visual Studio Code** に戻ります。 開いているターミナルに戻ります。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、さらに少なくとも 2 回プロジェクトをビルドして実行します。 コンソール出力で新しい RU 料金を確認します。元の料金よりわずかに低くなっているはずです。

    ```
    dotnet run
    ```

    > &#128221;更新された RU 料金が表示されない場合は、数分待つ必要があります。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
