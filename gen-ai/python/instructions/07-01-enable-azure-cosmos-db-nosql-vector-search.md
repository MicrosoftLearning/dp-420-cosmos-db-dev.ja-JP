---
title: 07.1 - Azure Cosmos DB for NoSQL のベクトル検索を有効にする
lab:
  title: 07.1 - Azure Cosmos DB for NoSQL のベクトル検索を有効にする
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 10
parent: Python SDK labs
---

# Azure Cosmos DB for NoSQL のベクトル検索を有効にする

Azure Cosmos DB for NoSQL は、あらゆる規模で高次元ベクトルを効率的かつ正確に格納およびクエリを実行するように設計された効率的なベクトル インデックス作成と検索機能を提供します。 この機能を利用するには、アカウントで *NoSQL API のベクトル検索*機能を使用できるようにする必要があります。

このラボでは、ベクトル ストアとして使用するデータベースを準備するために、Azure Cosmos DB for NoSQL アカウントを作成し、その上でベクトル検索機能を有効にします。

## 開発環境を準備する

「**Azure Cosmos DB を使用して Coplilot を構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照して実行します。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用して Copilot を構築する**」ラボ用に Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#enable-vector-search-for-nosql-api)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## NoSQL API のベクトル検索を有効にする

このタスクでは、Azure CLI を使用して、Azure Cosmos DB アカウントで *NoSQL API のベクトル検索*機能を有効にします。

1. [Azure portal](https://portal.azure.com) のツール バーから Cloud Shell を開きます。

    ![Azure portal のツール バーでは Cloud Shell アイコンが強調表示されています。](media/07-azure-portal-toolbar-cloud-shell.png)

2. Cloud Shell プロンプトで、`az account set -s <SUBSCRIPTION_ID>` を実行し、`<SUBSCRIPTION_ID>` プレースホルダー トークンをこの演習で使用しているサブスクリプションの ID に置き換えて、演習サブスクリプションが後続のコマンドに使用されていることを確認します。

3. *NoSQL API のベクトル検索*機能を有効にするには、Azure Cloud Shell から次のコマンドを実行し、`<RESOURCE_GROUP_NAME>` トークンと `<COSMOS_DB_ACCOUNT_NAME>` トークンをそれぞれリソース グループの名前と Azure Cosmos DB のアカウント名に置き換えます。

     ```bash
     az cosmosdb update \
       --resource-group <RESOURCE_GROUP_NAME> \
       --name <COSMOS_DB_ACCOUNT_NAME> \
       --capabilities EnableNoSQLVectorSearch
     ```

4. コマンドが正常に実行されるまで待ってから、Cloud Shell を終了します。

5. Cloud Shell を閉じます。

## ベクトルをホストするためのデータベースとコンテナーを作成する

1. [Azure portal](https://portal.azure.com) の Azure Cosmos DB アカウントの左側のメニューから **[データ エクスプローラー]** を選択し、**[新しいコンテナー]** を選択します。

2. **[新しいコンテナー]** ダイアログで次の手順を実行します。
   1. **[データベース ID]** で **[新規作成]** を選択し、データベース ID フィールドに "CosmicWorks" と入力します。
   2. **[コンテナー ID]** ボックスに、"Products" という名前を入力します。
   3. **パーティション キー**として "/category_id" と入力します。

      ![ダイアログに入力された、上記で指定した [新しいコンテナー] 設定のスクリーンショット。](media/07-azure-cosmos-db-new-container.png)

   4. **[新しいコンテナー]** ダイアログの一番下までスクロールし、**Container Vector Policy** を展開して、**[埋め込みベクトルの追加]** を選択します。

   5. **Container Vector Policy** 設定で、次の設定を行います。

      | 設定 | Value |
      | ------- | ----- |
      | **Path** | */embedding* を入力します。 |
      | **データの種類** | *float32* を選択します。 |
      | **distance 関数** | *cosine* を選択します。 |
      | **Dimensions** | OpenAI の `text-embedding-3-small` モデルによって生成されたディメンションの数と一致するように、*1536* と入力します。 |
      | **[インデックスの種類]** | *diskANN* を選択します。 |
      | **量子化 byte サイズ** | 空白のままにします。 |
      | **検索リスト サイズのインデックス作成** | 既定値 *100* を受け入れます。 |

      ![[新しいコンテナー] ダイアログに入力された、上記で指定した Container Vector Policy のスクリーンショット。](media/07-azure-cosmos-db-container-vector-policy.png)

   6. **[OK]** を選択してコンテナーとデータベースを作成します。

   7. コンテナーが作成されるのを待ってから続行します。 コンテナーの準備が整うまで、数分かかる場合があります。
