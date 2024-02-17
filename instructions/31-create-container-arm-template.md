---
lab:
  title: Azure Resource Manager テンプレートを使用して Azure Cosmos DB for NoSQL コンテナーを作成する
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Azure Resource Manager テンプレートを使用して Azure Cosmos DB for NoSQL コンテナーを作成する

Azure Resource Manager テンプレートは、Azure にデプロイするインフラストラクチャを宣言的に定義する JSON ファイルです。 Azure Resource Manager テンプレートは、Azure にサービスをデプロイするための一般的なコードとしてのインフラストラクチャ ソリューションです。 Bicep は、JSON テンプレートの作成に使用できる、読みやすいドメイン固有言語を定義することで、概念をさらに詳しく説明します。

このラボでは、Azure Resource Manager テンプレートを使用して、新しい Azure Cosmos DB アカウント、データベース、およびコンテナーを作成します。 最初に未加工の JSON からテンプレートを作成してから、Bicep ドメイン固有言語を使用してテンプレートを作成します。

## 開発環境を準備する

このラボで作業している環境に **DP-420** のラボ コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]を参照してください

1. コマンド パレットを開き、**Git: Clone** を実行して、任意のローカル フォルダーに ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリをクローンします。

    > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## Azure Resource Manager テンプレートを使用して Azure Cosmos DB for NoSQL リソースを作成する

Azure Resource Manager の **Microsoft.DocumentDB** リソース プロバイダーを使用すると、JSON ファイルを使用してアカウント、データベース、およびコンテナーをデプロイできます。 ファイルは複雑な場合がありますが、予測可能な形式に従っており、Visual Studio Code 拡張機能を使用して書き込むことができます。

> &#128161; スタックしてしまい、テンプレートの構文エラーを理解できない場合は、この[ソリューションの Azure Resource Manager テンプレート][github.com/arm-template-guide]をガイドとして使用してください。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**31-create-container-arm-template** フォルダーを参照します。

1. **deploy.json** ファイルを開きます。

1. 空の Azure Resource Manager テンプレートを確認します。

    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        ]
    }
    ```

1. **resources** 配列内に、新しい JSON オブジェクトを追加して、新しい Azure Cosmos DB アカウントを作成します。

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
        "location": "[resourceGroup().location]",
        "properties": {
            "databaseAccountOfferType": "Standard",
            "locations": [
                {
                    "locationName": "westus"
                }
            ]
        }
    }
    ```

    オブジェクトは、次の設定で構成されます。

    | **設定** | **Value** |
    | ---: | :--- |
    | **リソースの種類** | *Microsoft.DocumentDB/databaseAccounts* |
    | **API バージョン** | *2021-05-15* |
    | **アカウント名** | *csmsarm* &amp; *アカウント名から生成された一意の文字列*  |
    | **場所** | *リソース グループの現在の場所* |
    | **アカウント オファーの種類** | *Standard* |
    | **場所** | *米国西部のみ* |

1. **deploy.json** ファイルを保存します。

1. **31-create-container-arm-template** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **31-create-container-arm-template** フォルダーに既に設定されているターミナルが開きます。

1. Azure にログインする前に、tls/ssl 証明書をインストールします。

    ```
    $CurrentDirectory=$pwd
    CD "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    CD $CurrentDirectory
    ```

1. 次のコマンドを使用して、Azure CLI の対話型ログイン プロシージャを開始します。

    ```
    az login
    ```

1. Azure CLI では、Web ブラウザーのウィンドウまたはタブが自動的に開き、ブラウザー インスタンス内で、サブスクリプションに関連付けられている Microsoft 資格情報を使用して Azure CLI にサインインします。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. 自分用のリソース グループをラボ プロバイダーが作成済みかどうかを確認します。それが済んでいる場合は、次のセクションで必要になるので、その名前をメモします。

    ```
    az group list --query "[].{ResourceGroupName:name}" -o table
    ```
    
    このコマンドを実行すると、複数のリソース グループ名を返すことができます。

1. (省略可能) ***"リソース グループが作成されていない場合" は***、リソース グループ名を選んで作成します。 *ラボ環境によってはロックされている場合があるので、自分用のリソース グループを管理者に作成してもらう必要があることに注意してください。*

    i. この一覧から、自分に最も近い場所の名前を取得します。

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. リソース グループを作成します。  *ラボ環境によってはロックされている場合があるので、自分用のリソース グループを管理者に作成してもらう必要があることに注意してください。*
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

1. 次のコマンドを使用して、このラボで以前に作成または表示したリソース グループの名前を使用して、新しい変数名 **resourceGroup** を作成します。

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; たとえば、リソース グループの名前が **dp420** の場合、コマンドは **$resourceGroup="dp420"** になります。

1. **echo** コマンドレットを使用して、次のコマンドを使用して **$resourceGroup** 変数の値をターミナル出力に書き込みます。

    ```
    echo $resourceGroup
    ```

1. [az deployment group create][docs.microsoft.com/cli/azure/deployment/group] コマンドを使用して Azure Resource Manager テンプレートをデプロイします。

    ```
    az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 統合ターミナルを開いたままにして、**deploy.json** ファイルのエディターに戻ります。

1. **resources** 配列内に、別の新しい JSON オブジェクトを追加して、新しい Azure Cosmos DB for NoSQL データベースを作成します。

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
        ],
        "properties": {
            "resource": {
                "id": "cosmicworks"
            }
        }
    }
    ```

    オブジェクトは、次の設定で構成されます。

    | **設定** | **Value** |
    | ---: | :--- |
    | **リソースの種類** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API バージョン** | *2021-05-15* |
    | **アカウント名** | *csmsarm* &amp; *アカウント名から生成された一意の文字列* &amp; */cosmicworks*  |
    | **リソース ID** | *cosmicworks* |
    | **依存関係** | *テンプレートで前に作成した databaseAccount* |

1. **deploy.json** ファイルを保存します。

1. 統合ターミナルに戻ります。

1. **az deployment group create** コマンドを使用して Azure Resource Manager テンプレートをデプロイします。

    ```
    az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 統合ターミナルを開いたままにして、**deploy.json** ファイルのエディターに戻ります。

1. **resources** 配列内に、別の新しい JSON オブジェクトを追加して、新しい Azure Cosmos DB for NoSQL コンテナーを作成します。

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
            "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
        ],
        "properties": {
            "options": {
                "throughput": 400
            },
            "resource": {
                "id": "products",
                "partitionKey": {
                    "paths": [
                        "/categoryId"
                    ]
                }
            }
        }
    }
    ```

    オブジェクトは、次の設定で構成されます。

    | **設定** | **Value** |
    | ---: | :--- |
    | **リソースの種類** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers* |
    | **API バージョン** | *2021-05-15* |
    | **アカウント名** | *csmsarm* &amp; *アカウント名から生成された一意の文字列* &amp; */cosmicworks/products*  |
    | **リソース ID** | *products* |
    | **スループット** | *400* |
    | **パーティション キー** | */categoryId* |
    | **依存関係** | *テンプレートで先に作成したアカウントとデータベース* |

1. **deploy.json** ファイルを保存します。

1. 統合ターミナルに戻ります。

1. **az deployment group create** コマンドを使用して最後の Azure Resource Manager テンプレートをデプロイします。

    ```
    az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 統合ターミナルを閉じます。

## デプロイされた Azure Cosmos DB リソースを確認する

Azure Cosmos DB for NoSQL リソースがデプロイされたら、Azure portal でそのリソースに移動できます。 データ エクスプローラーを使用して、アカウント、データベース、およびコンテナーがすべて正しくデプロイおよび構成されていることを検証します。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **csmsarm** プレフィックスの付いた **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. **NoSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、**[スケールと設定]** を選択します。

1. **[スケール]** セクション内の値を確認します。 具体的には、**[スループット]** セクションで **[手動]** オプションが選択されており、プロビジョニングされたスループットが **400** RU/秒に設定されていることを確認します。

1. **[設定]** セクション内の値を確認します。 具体的には、**パーティション キー**の値が **/categoryId** に設定されていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## Bicep テンプレートを使用して Azure Cosmos DB for NoSQL リソースを作成する

Bicep は、効率的なドメイン固有言語であり、Azure Resource Manager テンプレートよりも単純かつ簡単に Azure リソースをデプロイできます。 違いを説明するために、Bicep と別の名前を使用してまったく同じリソースをデプロイします。\[\]

> &#128161; スタックしてしまい、テンプレートの構文エラーを理解できない場合は、この[ソリューションの Bicep テンプレート][github.com/bicep-template-guide]をガイドとして使用してください。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**31-create-container-arm-template** フォルダーを参照します。

1. 空の **deploy.bicep** ファイルを開きます。

1. ファイル内に、新しいオブジェクトを追加して、新しい Azure Cosmos DB アカウントを作成します。

    ```
    param location string = resourceGroup().location
    
    resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
      name: 'csmsbicep${uniqueString(resourceGroup().id)}'
      location: location
      properties: {
        databaseAccountOfferType: 'Standard'
        locations: [
          { 
            locationName: 'westus' 
          }
        ]
      }
    }
    ```

    オブジェクトは、次の設定で構成されます。

    | **設定** | **Value** |
    | ---: | :--- |
    | **エイリアス** | *取引先企業* |
    | **名前** | *csmsarm* &amp; *アカウント名から生成された一意の文字列* |
    | **リソースの種類** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API バージョン** | *2021-05-15* |
    | **場所** | *リソース グループの現在の場所* |
    | **アカウント オファーの種類** | *Standard* |
    | **場所** | *米国西部のみ* |

1. **deploy.bicep** ファイルを保存します。

1. **31-create-container-arm-template** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. 次のコマンドを使用して、このラボで以前に作成または表示したリソース グループの名前を使用して、新しい変数名 **resourceGroup** を作成します。

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; たとえば、リソース グループの名前が **dp420** の場合、コマンドは **$resourceGroup="dp420"** になります。

1. **az deployment group create** コマンドを使用して Bicep テンプレートをデプロイします。

    ```
    az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 統合ターミナルを開いたままにして、**deploy.bicep** ファイルのエディターに戻ります。

1. ファイル内に、別の新しいオブジェクトを追加して、新しい Azure Cosmos DB データベースを作成します。

    ```
    resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
      parent: Account
      name: 'cosmicworks'
      properties: {
        resource: {
            id: 'cosmicworks'
        }
      }
    }
    ```

    オブジェクトは、次の設定で構成されます。

    | **設定** | **Value** |
    | ---: | :--- |
    | **Parent** | *テンプレートで前に作成したアカウント* |
    | **エイリアス** | *データベース* |
    | **名前** | *cosmicworks*  |
    | **リソースの種類** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API バージョン** | *2021-05-15* |
    | **リソース ID** | *cosmicworks* |

1. **deploy.bicep** ファイルを保存します。

1. 統合ターミナルに戻ります。

1. **az deployment group create** コマンドを使用して Bicep テンプレートをデプロイします。

    ```
    az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 統合ターミナルを開いたままにして、**deploy.bicep** ファイルのエディターに戻ります。

1. ファイル内に、別の新しいオブジェクトを追加して、新しい Azure Cosmos DB コンテナーを作成します。

    ```
    resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
      parent: Database
      name: 'products'
      properties: {
        options: {
          throughput: 400
        }
        resource: {
          id: 'products'
          partitionKey: {
            paths: [
              '/categoryId'
            ]
          }
        }
      }
    }
    ```

    オブジェクトは、次の設定で構成されます。

    | **設定** | **Value** |
    | ---: | :--- |
    | **Parent** | *テンプレートで前に作成したデータベース* |
    | **エイリアス** | *コンテナー* |
    | **名前** | *products*  |
    | **リソース ID** | *products* |
    | **スループット** | *400* |
    | **パーティション キーのパス** | */categoryId* |

1. **deploy.bicep** ファイルを保存します。

1. 統合ターミナルに戻ります。

1. **az deployment group create** コマンドを使用して最後の Bicep テンプレートをデプロイします。

    ```
    az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

## Bicep テンプレートのデプロイ結果を確認する

Bicep デプロイは、Azure Resource Manager デプロイと同じ手法の多くを使用して検証できます。 アカウント、データベース、コンテナーが正常にデプロイされたことを検証するだけでなく、6 つのデプロイすべてにわたってデプロイ履歴も表示します。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選択します。

1. リソース グループ内で、**[デプロイ]** ペインに移動します。

1. Azure Resource Manager テンプレートと Bicep ファイルからの 6 つのデプロイを確認します。

1. 引き続きリソース グループ内で、**[概要]** ペインに移動します。

1. 引き続きリソース グループ内で、このラボで作成した **Azure Cosmos DB アカウント** リソースに **csmsbicep** プレフィックスを付けて選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. **NoSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、**[スケールと設定]** を選択します。

1. **[スケール]** セクション内の値を確認します。 具体的には、**[スループット]** セクションで **[手動]** オプションが選択されており、プロビジョニングされたスループットが **400** RU/秒に設定されていることを確認します。

1. **[設定]** セクション内の値を確認します。 具体的には、**パーティション キー**の値が **/categoryId** に設定されていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/cli/azure/deployment/group]: https://docs.microsoft.com/cli/azure/deployment/group
[github.com/arm-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.json
[github.com/bicep-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.bicep
