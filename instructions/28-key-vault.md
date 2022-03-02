---
lab:
  title: Azure Cosmos DB SQL API のアカウント キーを Azure Key Vault に保存する
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: 929b8c5078ff27ae474f86393ad61f44dd3b66b3
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025077"
---
# <a name="store-azure-cosmos-db-sql-api-account-keys-in-azure-key-vault"></a>Azure Cosmos DB SQL API のアカウント キーを Azure Key Vault に保存する

Azure Cosmos DB アカウントの接続コードをアプリケーションに追加することは、アカウントの URI とキーを指定するのと同じように簡単です。 このセキュリティ情報は、アプリケーション コードにハード コーディングされている場合があります。 ただし、アプリケーションが Azure App Service に配置されている場合は、暗号化接続情報を Azure Key Vault に保存できます。

このラボでは、Azure Cosmos DB アカウント接続文字列を暗号化し、Azure Key Vault に保存します。 次に、これらの資格情報を Azure Key Vault から取得する Azure App Service Web アプリを作成します。 アプリケーションでは、これらの資格情報を使用し、Azure Cosmos DB アカウントに接続します。 その後、アプリケーションによって、Azure Cosmos DB アカウント コンテナーにいくつかのドキュメントが作成され、そのステータスが Web ページに戻されます。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業する環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、これらの手順に従って行います。 それ以外の場合は、以前にクローンされたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、Visual Studio Code の入門ガイドに関するページ (code.visualstudio.com/docs/getstarted) を参照してください

1. コマンド パレットを開き、**Git: Clone** を実行して、選択したローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、*Visual Studio Code を **閉じます***。 このファイルは、後で **28-key-vault** フォルダーを直接ポイントして開きます。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、アカウントでサポートする API を選択します (たとえば、**Mongo API** または **SQL API**)。 Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得できます。 エンドポイントとキーを使用して、Azure Cosmos DB SQL API アカウントにプログラムで接続します。 Azure SDK for .NET またはその他の SDK の接続文字列でエンドポイントとキーを使用します。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、 *[Cosmos DB]* を検索してから、次の設定で新しい **[Azure Cosmos DB SQL API]** アカウント リソースを作成し、残りのすべての設定を既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | *既存の Azure サブスクリプション* |
    | **リソース グループ** | *既存のリソース グループを選択するか、新しいものを作成します* |
    | **アカウント名** | *グローバルに一意の名前を入力します* |
    | **Location** | *使用可能なリージョンを選びます* |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **[プライマリ接続文字列]** フィールドの値を記録します。 この **接続文字列** の値は、この演習で後ほど使用します。

## <a name="create-an-azure-key-vault-and-store-the-azure-cosmos-db-account-credentials-as-a-secret"></a>Azure Key Vault を作成し、Azure Cosmos DB アカウントの資格情報をシークレットとして保存する

Web アプリを作成する前に、*Azure Key Vault* で暗号化された *シークレット* にコピーして、Azure Cosmos DB アカウントの接続文字列をセキュリティで保護します。 ここでは、その操作を行います。

1. Azure portal で、 **[キー コンテナー]** ページに移動します。

1. コンテナーを追加するには、*[ **+ 作成]** ボタンを選択します。次の設定を使用してコンテナーに入力し、残りのすべての設定を既定値のままにし*、それからコンテナーを作成するように選択します。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | *既存の Azure サブスクリプション* |
    | **リソース グループ** | *既存のリソース グループを選択するか、新しいものを作成します* |
    | **キー コンテナー名** | *グローバルに一意の名前を入力します* |
    | **リージョン** | *使用可能なリージョンを選びます* |

1. コンテナーが作成されたら、コンテナーに移動します。

1. *[設定]* セクションで、 **[シークレット]** を選択します。

1. **[+ 生成/インポート]** を選択して資格情報接続文字列を暗号化し、次の設定を使用して *シークレット* 値を入力します。*残りの設定はすべて既定値のままにし*、その後シークレットの作成を選択します。

    | **設定** | **Value** |
    | ---: | :--- |
    | **Upload options** | *[手動]* |
    | **名前** | *シークレットにラベル付けする名前* |
    | **Value** | *このフィールドは、最も重要な入力フィールドです。この値は、Azure Cosmos DB アカウントのキー セクションからコピーしたプライマリ接続文字列です。この値は、シークレットに変換されます。* |
    | **有効** | *あり* |
 
1. シークレットの下に、新しいシークレットが表示されます。 ここでは、Web アプリのコードに追加する *シークレット識別子* を取得する必要があります。 作成した **シークレット** を選択します。

1. Azure Key Vault では、複数のバージョンのシークレットを作成できますが、このラボでは、1 つのバージョンのみが必要です。 **現在のバージョン** を選択します。

1. **[シークレット識別子]** フィールドの値を記録します。 この値をアプリケーションのコードで使用して、Key Vault からシークレットを取得します。  この値は URL であることに注意してください。 このシークレットを正常に動作させるために必要な手順はもう 1 つありますが、その手順は後でもう少し行います。

## <a name="create-an-azure-app-service-webapp"></a>Azure App Service Web アプリを作成する

Azure Cosmos DB アカウントに接続し、いくつかのコンテナーとドキュメントを作成する Web アプリを作成します。 このアプリでは Azure Cosmos DB の *資格情報* をハードコーディングすることはありませんが、代わりにキー コンテナーから **シークレット識別子** をハード コーディングしてください。 Azure レイヤーの Web アプリに適切な権限が割り当てられていなければ、この識別子が役に立たないということを確認します。 コーディングを始めましょう。



1. **Visual Studio Code** を開きます。  **28-key-vault** フォルダーを開きます。[ファイル]、[フォルダーを開く] の順に選択し、**28-key-vault** フォルダーに到達するまで参照します。

    > &#128221; **[エクスプローラー]** ツリーには、**28-key-vault** フォルダーとそのファイルとサブフォルダーのみが表示されるべきであることに注意してください。 前に複製した GitHub リポジトリ全体が表示される場合は、Web アプリが正常に動作しないため、 **[エクスプローラー]** ツリーでは、**28-key-vault** フォルダーとそのファイルとサブフォルダーのみが表示されていることを確認してください。

1. **28-key-vault** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **28-key-vault** フォルダーに既に設定されているターミナルが開きます。

1. MVC Web アプリケーションのシェルを作成しましょう。 後で生成されたファイルをいくつか置き換えます。 次のコマンドを実行して Web アプリを作成します。

    ```
    dotnet new mvc
    ```


1. そのコマンドによって Web アプリのシェルが作成されたので、複数のファイルとディレクトリが追加されました。 必要なすべてのコードを含むファイルが既に 2 つ存在します。 `.\KeyvaultFiles` ディレクトリ内のそれぞれのファイルの `.\Controllers\HomeController.cs` と `.\Views\Home\Index.cshtml` を置き換えます。

1. ファイルを置き換えた後、`.\KeyvaultFiles` ディレクトリを ***削除*** します。

## <a name="import-the-multiple-missing-libraries-into-the-net-script"></a>不足している複数のライブラリを .NET スクリプトにインポートする

.NET CLI には、パッケージの追加 (docs.microsoft.com/dotnet/core/tools/dotnet-add-package) コマンドが含まれています。事前構成済みのパッケージ フィードからパッケージをインポートします。 .NET インストールでは、既定のパッケージ フィードとして NuGet を使用します。

1. まだ実行していない場合は、**Visual Studio Code** の **[エクスプローラー]** ペインで、**28-key-vault** フォルダーを参照します。

1. まだ実行していない場合、**28-key-vault** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **28-key-vault** フォルダーに既に設定されているターミナルが開きます。

1. 次のコマンドを使用して、NuGet から Microsoft.Azure.Cosmos (nuget.org/packages/microsoft.azure.cosmos/3.22.1) パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 次のコマンドを使用して、NuGet から Newtonsoft.Json (nuget.org/packages/Newtonsoft.Json/13.0.1) パッケージを追加します。

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

1. 次のコマンドを使用して、NuGet から Microsoft.Azure.KeyVault (nuget.org/packages/Microsoft.Azure.KeyVault) パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.KeyVault
    ```

1. 次のコマンドを使用して、NuGet から Microsoft.Azure.Services.AppAuthentication (nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Services.AppAuthentication
    ```

## <a name="adding-the-secret-identifier-to-your-webapp"></a>Web アプリにシークレット識別子を追加する

1. Visual Studio で、`.\Controllers\HomeControler.cs` ファイルを開きます

1. **GetKeyVaultSecret** ユーザー定義関数では、Azure Cosmos DB アカウント シークレットを取得します。 関数は *98 行目* から始まるので、次のスクリプトのようになります。

```
        private static async Task<Tuple<bool,string>>  GetKeyVaultSecret()
        {
            AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=App;");

            try
            {
                var KVClient = new KeyVaultClient(
                    new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));

                var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
                    .ConfigureAwait(false);

                return new Tuple<bool,string>(true, KeyVaultSecret.Value.ToString());

            }
            catch (Exception exp)
            {
                return new Tuple<bool,string>(false, exp.Message);
            }

        }
```

3. この関数が行う重要な呼び出しを確認しましょう。

    - *100 行目* では、現在の Web アプリのトークンを定義します。 このトークンは、コンテナーにアクセスしようとしているアプリを識別するために、Azure Key Vault に提供されます。 
    - *104 から 105 行目* では、Azure Key Vault に接続する *Key Vault Client クライアント* を準備します。 Web アプリ トークンをパラメーターとして送信することに注目してください。 
    - *107 から 108 行目* では、Key Vault クライアントに **シークレット識別子** の URL アドレスを提供します。これは、そのキー コンテナーに格納されているシークレットを返します。 

1.  Web アプリをデプロイする前に、依然として **シークレット識別子** URL を送信する必要があります。  *107 行目で*、文字列 **`<Key Vault Secret Identifier>`** を *[シークレット]* セクションに記録した **シークレット識別子** URL に置き換え、ファイルを保存します。

```
        var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
```

## <a name="optional-install-the-azure-app-services-extension"></a>(省略可能) Azure App Services 拡張機能をインストールする

Visual Studio でコマンド パレット (**Ctrl + Shift + P**) を起動しても、Azure App リソース コマンドを検索しても何も返されない場合は、拡張機能をインストールする必要があります。

1. Visual Studio Code の左側メニューで、 **[拡張機能]** オプションを選択します。

1. 検索バーで、Azure App Service を検索して選択します。

1. [インストール] ボタンを選択してインストールします。

1. **[拡張機能]** タブを閉じて、コードに戻ります。

## <a name="deploy-your-application-to-azure-app-services"></a>アプリケーションを Azure App Service にデプロイする

コードの残りの部分は簡単です。接続文字列を取得し、Azure Cosmos DB に接続して、いくつかのドキュメントを追加します。 アプリケーションでは、問題に関するフィードバックも提供する必要があります。 アプリケーションをデプロイした後で、それ以上の変更を行う必要はないようにする必要があります。 それでは、始めましょう。 

> &#128221; 次の手順の大部分は、Visual Studio 画面の上部中央のコマンド パレットで実行されます。

1. Visual Studio Code で、コマンド パレットを開き、***Azure App Service: 新しい Web アプリの作成 (詳細)*** を検索します

1. ***[Azure へのサインイン]*** を選択します。このオプションを選択すると、Web ブラウザー ウィンドウが開きます。サインイン プロセスに従って、完了したらブラウザーを閉じます。

1. グローバルに一意の Web アプリ名を入力します。

1. 必要な場合、既存のリソース グループを選択するか、新しいリソース グループを作成します。

1. **[.NET 6 (LTS)]** を選択します。

1. **[Windows]** を選択します。

1. 使用可能な [場所] を選択します。

1. **[+ 新しい App Service プランの作成]** を選択します。

1. App Service プランの既定の名前を受け入れるか (Web アプリ名と同じ名前にしてください)、新しい名前を選択します。

1. **[Free (F1) 無料で Azure を試す]** を選択します。

1. Application Insights については、 **[今はスキップ]** を選択します。

1. デプロイが、右下隅のステータス バーで実行中になっているはずです。 

1. 確認を求められたら **[デプロイ]** を選択します。

1. **[参照]** を選択すると、**28-key-vault** フォルダー内になるはずです。そのフォルダーを選択します。

1. **"28-key-vault" にデプロイに必要な構成が見つかりません** というメッセージが表示されたポップアップが表示されたら、[構成の追加] ボタンを選択します。  このオプションを選択すると、不足している `.vscode` フォルダーが作成されます。

    > &#128221; 非常に重要なのは、このポップアップがアプリの最初のデプロイに表示されない場合、Azure App Services へのアップロードでファイルが不足しています。 デプロイは成功しますが、Web サイトでは常に、*このディレクトリまたはページを表示するアクセス許可が付与されていません* というメッセージが返されます。 最も可能性の高い原因は、Visual Studio Code が **28-key-vault** フォルダーのみではなく、GitHub Cloned リポジトリで開かれたということです。

1. 常にそのワークスペースにデプロイするように求めるプロンプトで、 **[はい]** を選択します。

1. メッセージが表示されたら、[Web サイトの参照] を選択します。  または、ブラウザーを開き、 **`https://<yourwebappname>.azurewebsites.net`** に移動します。 どちらの場合も、問題があります。 Web ページにユーザー定義のメッセージが表示されている必要があります。 拡張エラー メッセージと **Key Vault にアクセスできませんでした** というメッセージが表示されます。 これを修正しましょう。

## <a name="allow-our-app-to-use-a-managed-identity"></a>アプリでマネージド ID の使用を許可する

修正する必要がある最初の問題は、アプリでマネージド ID を使用できるようにすることです。 マネージド ID を使用すると、Azure Key Vault などの Azure サービスをアプリで使用できます。

1. ブラウザーを開き、Azure portal にログインします。

1. **[App Services]** ページを開きます。 Web アプリ名が一覧表示されるので、それを選択します。

1. *[設定]* セクションで、 **[ID]** を選択します。

1. [状態] で、 **[オン]** と **[保存]** を選択します。  **割り当てられたマネージド ID** を有効にするプロンプトで、 *[はい]* を選択します。

1. もう一度 Web アプリを試しましょう。  ブラウザーで **`https://<yourwebappname>.azurewebsites.net`** に移動します。

1. まだ 1 つの問題があります。 最初のメッセージはプログラムから送信されているユーザー定義メッセージですが、2 番目のメッセージはシステムによって生成されたメッセージです。 2 番目のメッセージは、Key Vault への接続へのアクセスが許可されているが、コンテナー内のシークレットを表示するためのアクセス権が付与されていないという意味です。  この問題を解決するための最後に 1 つ設定しましょう。

## <a name="granting-our-web-application-an-access-policy-to-the-key-vault-secrets"></a>Web アプリケーションに Key Vault シークレットへのアクセス ポリシーを付与する

このラボの当初の目標は、Azure Cosmos DB アカウントがアプリケーションでハードコーディングされるのを防ぐことでした。 しかし、誰でも確認できる **シークレット識別子** URL をハード コーディングしました。 では、資格情報をセキュリティで保護するにはどうすれば良いでしょうか。 幸い、シークレット識別子そのものだけでは役に立ちません。 **シークレット識別子** では Azure Key Vault のドアまでアクセスできず、コンテナーで誰が入れるか、誰がドアにとどまるのかを決定します。 これは、アプリケーションからそのコンテナー内のシークレットを確認できるよう、Key Vault アクセス ポリシーを作成する必要があるということを意味します。 その解決策を見てみしましょう。

1. (省略可能) ポリシーを作成する前に、Azure Cosmos DB データベースの現在の内容を確認しましょう。  Azure portal で、Azure Cosmos DB アカウントに移動します。**GlobalCustomers** データベースはありますか。 ない場合は、Web アプリの正常な実行によって作成されます。 ある場合は、データベース内の項目の数を確認します。Web アプリが正常に実行されると、項目が追加されます。

1. Azure portal で、前に作成したキー コンテナーに移動します。

1. *[設定]* セクションで、 **[アクセス ポリシー]** を選択します。

1. **[+ アクセス ポリシーの追加]** を選択します。

1. *[アクセス ポリシー]* の値に次の設定を入力し、*残りのすべての設定を既定値のままにして*、選択してポリシーを追加します。

    | **設定** | **Value** |
    | ---: | :--- |
    | **キーのアクセス許可** | *Get* |
    | **シークレットのアクセス許可** | *Get* |
    | **プリンシパルの選択** | *アプリケーション名を検索して選択する* |

    > &#128221; 承認されているアプリケーションを選択しないでください。

1. 新しいポリシーを **保存** します。

1. もう一度 Web アプリを試しましょう。  ブラウザーで **`https://<yourwebappname>.azurewebsites.net`** に移動します。

1. 成功です。 Web ページには、customer コンテナーに新しい項目を挿入したことが示されています。 また、実際に表示されているシークレットを確認することもできます。

    > &#128221; 運用環境では **決して** シークレットが表示しないでください。これは説明のために行いました。


1. Azure Cosmos DB アカウントにアクセスし、データが含まれた新しい **GlobalCustomers** データベースがあるか、あるいはデータベースが既に存在する場合は、データベース内に項目が追加で存在するかどうかを確認します。

これで、Azure Key Vault を適切に使用して、Azure Cosmos DB アカウントのキーを保護できました。
