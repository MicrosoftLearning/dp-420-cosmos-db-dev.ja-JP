---
title: Azure Cosmos DB を設定する
lab:
  title: Azure Cosmos DB を設定する
  module: Setup
layout: default
nav_order: 3
parent: Common setup instructions
---

# Azure Cosmos DB を設定する

この演習では、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与します。 これにより、Azure 認証を使用してラボ コードからデータベースにアクセスできるようになり、キーを格納および管理する必要がなくなります。

## Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API を選択します。 Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for Python または任意の他の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続する際に使用できます。

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
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

1. 新しく作成された **Azure Cosmos DB** アカウント リソースに移動し、 **[キー]** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。 具体的な内容は次のとおりです。

    1. **URI** フィールドをコピーし、後で使用できるようにテキスト エディターに保存します。 この**エンドポイント**の値は、この演習で後ほど使用します。

1. 次の手順のために、ブラウザー タブは開いたままにします。

## Cosmos DB 組み込みデータ共同作成者 RBAC ロールをユーザー ID に提供する

この演習の最後のタスクとして、**Cosmos DB 組み込みデータ共同作成者** RBAC ロールに割り当てることで、Azure Cosmos DB for NoSQL アカウント内のデータを管理するためのアクセス権を Microsoft Entra ID ユーザー ID に付与します。 これにより、Azure 認証を使用してコードからデータベースにアクセスできるようになり、キーを格納および管理する必要がなくなります。

> &#128221; Microsoft Entra ID のロールベースのアクセス制御 (RBAC) を使用して Azure Cosmos DB などの Azure サービスに対して認証を行う場合、キーベースの方法よりも主な利点がいくつかあります。 Entra ID RBAC は、ユーザー ロールに合わせて調整された正確なアクセス制御によってセキュリティを強化し、不正アクセスのリスクを効果的に軽減します。 また、ユーザー管理が効率化され、管理者は、暗号化キーの配布と保守に手間をかけることなく、アクセス許可を動的に割り当てて変更できます。 さらに、このアプローチでは、組織のポリシーに合わせ、包括的なアクセスの監視とレビューを容易にすることで、コンプライアンスと監視能力を高めます。 セキュリティで保護されたアクセス管理を合理化することで、Entra ID RBAC は Azure サービスを利用するためのより効率的でスケーラブルなソリューションを実現します。

1. [Azure portal](https://portal.azure.com) のツール バーから Cloud Shell を開きます。

    ![Azure portal のツール バーでは Cloud Shell アイコンが強調表示されています。](media/azure-portal-toolbar-cloud-shell.png)

1. Cloud Shell プロンプトで、`az account set -s <SUBSCRIPTION_ID>` を実行し、`<SUBSCRIPTION_ID>` プレースホルダー トークンをこの演習で使用しているサブスクリプションの ID に置き換えて、演習サブスクリプションが後続のコマンドに使用されていることを確認します。

1. 上記のコマンドの出力をコピーして、以下の `az cosmosdb sql role assignment create` コマンドの `<PRINCIPAL_OBJECT_ID>` トークンとして使用します。

1. 次に、**Cosmos DB 組み込みデータ共同作成者**ロールの定義 ID を取得します。 次のコマンドを実行して、必ず `<RESOURCE_GROUP_NAME>` トークンと `<COSMOS_DB_ACCOUNT_NAME>` トークンを置き換えます。

    ```bash
    az cosmosdb sql role definition list --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>"
    ```

    出力を確認し、**Cosmos DB 組み込みデータ共同作成者**というロールの定義を見つけます。 出力には、`name` プロパティ内のロールの定義の一意識別子が含まれます。 後で次のステップの割り当て手順で使用する必要があるため、この値を記録してください。

1. これで、**Cosmos DB 組み込みデータ共同作成者**ロールの定義に自分自身を割り当てる準備ができました。 必ず `<RESOURCE_GROUP_NAME>` トークンと `<COSMOS_DB_ACCOUNT_NAME>` トークンを置き換えて、プロンプトで次のコマンドを入力します。

    > &#128221; 次のコマンドでは、`role-definition-id` が `00000000-0000-0000-0000-000000000002` に設定されています。これは、**Cosmos DB 組み込みデータ共同作成者**ロールの定義の既定値です。 `az cosmosdb sql role definition list` コマンドから取得した値が異なる場合は、実行前に以下のコマンドの値を置き換えてください。 `az ad signed-in-user show` コマンドは、サインインしている Entra ID ユーザーのオブジェクト ID を取得します。

    ```bash
    az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "00000000-0000-0000-0000-000000000002" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
    ```

1. コマンドの実行が完了すると、コードをローカルで実行して、Cosmos DB NoSQL データベースに格納されているデータの操作を挿入できます。

1. Cloud Shell を閉じます。
