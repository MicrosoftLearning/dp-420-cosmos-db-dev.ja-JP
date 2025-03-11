---
lab:
  title: Azure Functions を使用して Azure Cosmos DB for NoSQL のデータを処理する
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Azure Functions を使用して Azure Cosmos DB for NoSQL のデータを処理する

Azure Functions の Azure Cosmos DB トリガーは、変更フィード プロセッサを使用して実装されます。 この知識を活用して、Azure Cosmos DB for NoSQL コンテナーでの作成および更新操作に応答する関数を作成できます。 変更フィード プロセッサを手動で実装した場合、Azure Functions のセットアップも同様です。

このラボでは、データベースを監視し、その中で検出された各操作のログ情報を出力する関数アプリとそのすべての必要なリソースを作成します。

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

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. [**URI**] フィールドに注目します。 この**エンドポイント**の値は、この演習で後ほど使用します。

    1. [**主キー**] フィールドに注目してください。 この**キー**の値は、この演習で後ほど使用します。

1. リソース メニューで **[データ エクスプローラー]** を選択します。

1. **[データ エクスプローラー]** ペインで、**[新しいコンテナー]** を展開してから、**[新しいデータベース]** を選択します。

1. **[新しいデータベース]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | *``cosmicworks``* |

1. **[データ エクスプローラー]** ペインに戻り、階層内の **cosmicworks** データベース ノードを確認します。

1. **[データ エクスプローラー]** ペインで、 **[新しいコンテナー]** を選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | ''*既存の* &vert; *cosmicworks を使用します*'' |
    | **コンテナー ID** | *``products``* |
    | **パーティション キー** | *``/categoryId``* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認します。

1. **[データ エクスプローラー]** ペインで、**[新しいコンテナー]** を再度選択します。

1. **[新しいコンテナー]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **Value** |
    | --: | :-- |
    | **データベース ID** | ''*既存の* &vert; *cosmicworks を使用します*'' |
    | **コンテナー ID** | *``productslease``* |
    | **パーティション キー** | *``/id``* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開し、階層内の **productslease** コンテナー ノードを確認します。

1. Azure portal の **[ホーム]** に戻ります。

## Application Insight を作成する

"Azure Functionion アプリケーション" を作成する前に、アプリケーションの機能を監視できるように、*Azure Application Insight* を有効にする必要があります。** Application Insight には、まず "Log Analytics ワークスペース" が必要です。**

1. 検索ボックスで「**Log Analytics ワークスペース**」を検索します。

1. 新しい *Log Analytics* ワークスペースの **[+ 作成]** を選びます。

1. **[Log Analytics ワークスペース]** ダイアログで各設定に次の値を入力し、**[確認および作成]** を選んでから、**[作成]** を選びます。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **名前** | *``lab14laworkspace``* |
    | **場所** | ''*使用可能なリージョンを選びます*'' |

1. "Log Analytics ワークスペース" が作成されたら、検索ボックスで **Application Insights** を検索します。**

1. 新しい *Application Insight* の **[+ 作成]** を選びます。

1. **[Application Insights]** ダイアログで各設定に次の値を入力し、**[確認および作成]** を選んでから、**[作成]** を選びます。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション (両方のエントリ)** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **名前** | *``lab14appinsight``* |
    | **場所** | ''*使用可能なリージョンを選びます*'' |
    | **Log Analytics ワークスペース** | *lab14laworkspace* |

これで、アプリケーションの機能を監視できるようになります。

## Azure 関数アプリと Azure Cosmos DB によってトリガーされる関数を作成する

コードの記述を開始する前に、作成ウィザードを使用して Azure Functions リソースとそれに依存するリソース (Application Insights、Storage) を作成する必要があります。

1. **[+ リソースの作成]** を選択し、*Functions* を検索して、新しい**関数アプリ** アカウント リソースを作成します。 ホスティング オプションとして **[従量課金]** を選択し、次の設定 (残りの設定はすべて既定値のまま) でアプリをセットアップします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **名前** | ''*グローバルに一意の名前を入力します*'' |
    | **発行** | *コード* |
    | **ランタイム スタック** | *.NET* |
    | **Version** | *8 (LTS) インプロセス* |
    | **リージョン** | ''*使用可能なリージョンを選びます*'' |
    | **ストレージ アカウント** | *Create a new storage account \(新しいストレージ アカウントの作成\)* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成した **Azure Functions** アカウント リソースに移動し、概要ページで **[関数の作成]** を選択します。

1. **[関数の作成]** ポップアップで、次の設定を使用して新しい関数を作成し、残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **Select a template** | *Azure Cosmos DB のトリガー* |
    | **関数名** | *``ItemsListener``* |
    | **Cosmos DB アカウント接続** | *[新規選択]* &vert; *[Azure Cosmos DB アカウントを選択]* &vert; *[先ほど作成した Azure Cosmos DB アカウントを選択]* |
    | **データベース名** | *``cosmicworks``* |
    | **[コレクション名]** | *``products``* |
    | **リースのコレクション名** | *``productslease``* |
    | **リースのコレクションが存在しない場合、作成します** | *いいえ* |

## .NET で関数コードを実装する

先ほど作成した関数は、ポータルで編集された C# スクリプトです。 次に、ポータルを使用して、コンテナーに挿入または更新された項目の一意の識別子を出力する短い関数を記述します。

1. **[ItemsListener]**&vert;**[コードとテスト]** ペインで、**run.csx** スクリプトのエディターに移動し、その内容を削除します。

1. エディター領域で、**Microsoft.Azure.DocumentDB.Core** ライブラリを参照します。

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. **System**、**System.Collections.Generic**、および [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents] 名前空間の using ブロックを追加します。

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. 次の 2 つのパラメーターを持つ **Run** という名前の新しい静的メソッドを作成します。

    1. 型 **IReadOnlyList\<\>** の **input** という名前のパラメーターで、ジェネリック型は [Document][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document] です。

    1. 型 **ILogger** の **log** という名前のパラメーター。

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. **Run** メソッド内で、**log** 変数の **LogInformation** メソッドを呼び出して、現在のバッチの項目数を計算する文字列を渡します。

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}"); 
    ```

1. 引き続き **Run** メソッド内で、変数 **item** を使用して **input** 変数を反復処理し、型 **Document** のインスタンスを表す foreach ループを作成します。

    ```
    foreach(Document item in input)
    {
    }
    ```

1. **Run** メソッドの foreach ループ内で、**log** 変数の **LogInformation** メソッドを呼び出して、**item** 変数の [Id][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] プロパティを出力する文字列を渡します。

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. 完了すると、コード ファイルが次のようになるはずです。
  
    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
        log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");
    
        foreach(Document item in input)
        {
            log.LogInformation($"Detected Operation:\t{item.Id}");
        }
    }
    ```

1. ページの下部にある **[ログ]** セクションを展開し、**[App Insights ログ]** を展開し、**[ファイルシステム ログ]** を選択して現在の関数のストリーミング ログに接続します。

    > &#128161; ストリーミング ログ サービスに接続するには数秒かかる場合があります。 接続すると、ログ出力にメッセージが表示されます。

1. 現在の関数コードを**保存**します。

1. C# コード コンパイルの結果を確認します。 ログ出力の最後に **[コンパイルに成功しました]** というメッセージが表示されるはずです。

    > &#128221; ログ出力に警告メッセージが表示される場合があります。 これらの警告は、このラボに影響しません。

1. ログ セクションを**最大化**して、出力ウィンドウを拡張し、使用可能な最大スペースを埋めます。

    > &#128221; 別のツールを使用して、Azure Cosmos DB for NoSQL コンテナーに項目を生成します。 項目を生成したら、このターミナルに戻って出力を確認します。 ブラウザー ウィンドウを途中で閉じないでください。

## Azure Cosmos DB for NoSQL アカウントにサンプル データをシードする

**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。 ツールで一連の項目を作成して、ターミナル ウィンドウで実行されている変更フィード プロセッサを使用して確認します。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]を参照してください

1. **Visual Studio Code** で、 **[ターミナル]** メニューを開き、 **[新しいターミナル]** を選択して新しいターミナル インスタンスを開きます。

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

1. **Visual Studio Code** を閉じます。

1. Azure Functions ログ セクションが展開された状態で、現在開いているブラウザー ウィンドウまたはタブに戻ります。

1. 関数からのログ出力を確認します。 ターミナルには、変更フィードを使用して送信された変更ごとに、**[検出された操作]** というメッセージが出力されます。 操作は、最大 100 の操作グループにバッチ処理されます。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.documents]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.document
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id
