---
lab:
  title: Azure CLI スクリプトを使用してプロビジョニングされたスループットを調整する
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# <a name="adjust-provisioned-throughput-using-an-azure-cli-script"></a>Azure CLI スクリプトを使用してプロビジョニングされたスループットを調整する

Azure CLI は、Azure 全体のさまざまなリソースを管理するために使用できるコマンドのセットです。 Azure Cosmos DB には、選択した API に関係なく、Azure Cosmos DB アカウントのさまざまなファセットを管理するために使用できる豊富なコマンド グループがあります。

このラボでは、Azure CLI を使用して Azure Cosmos DB アカウント、データベース、およびコンテナーを作成します。 次に、Azure CLI を使用してプロビジョニングされたスループットを調整します。

## <a name="log-in-to-the-azure-cli"></a>Azure CLI にログインする

Azure CLI を使用する前に、まず CLI のバージョンを確認し、Azure 資格情報を使用してログインする必要があります。

1. **Visual Studio Code** を起動します。

1. **[ターミナル]** メニューを開き、 **[新しいターミナル]** を選択して新しいターミナル インスタンスを開きます。

1. 次のコマンドを使用して Azure CLI のバージョンを表示します。

    ```
    az --version
    ```

1. 次のコマンドを使用して、最も一般的な Azure CLI コマンド グループを表示します。

    ```
    az --help
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

## <a name="create-azure-cosmos-db-account-using-the-azure-cli"></a>Azure CLI を使用して Azure Cosmos DB アカウントを作成する

**cosmosdb** コマンド グループには、CLI を使用して Azure Cosmos DB アカウントを作成および管理するための基本的なコマンドが含まれています。 Azure Cosmos DB アカウントにはアドレス指定可能な URI があるため、スクリプトを使用して作成する場合でも、新しいアカウントにグローバルに一意の名前を作成することが重要です。

1. **Visual Studio Code** 内ですでに開いているターミナル インスタンスに戻ります。

1. 次のコマンドを使用して、**Azure Cosmos DB** に関連するほとんどの Azure CLI コマンドを表示します。

    ```
    az cosmosdb --help
    ```

1. 次のコマンドを使用して、[Get-Random][docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random] PowerShell コマンドレットを使用して **suffix** という名前の新しい変数を作成します。

    ```
    $suffix=Get-Random -Maximum 1000000
    ```

    > &#128221; この Get-Random コマンドレットは、0 ～ 1,000,000 のランダムな整数を生成します。 サービスにグローバルに一意の名前が必要なので、これは便利です。

1. ハードコードされた文字列 **csms** と変数置換を使用して、別の新しい変数名 **accountName** を作成し、次のコマンドを使用して **$suffix** 変数の値を挿入します。

    ```
    $accountName="csms$suffix"
    ```

1. 次のコマンドを使用して、このラボで以前に作成または表示したリソース グループの名前を使用して、もう 1 つの新しい変数名 **resourceGroup** を作成します。

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; たとえば、リソース グループの名前が **dp420** の場合、コマンドは **$resourceGroup="dp420"** になります。

1. **echo** コマンドレットを使用して、次のコマンドを使用して **$accountName** 変数および **$resourceGroup** 変数の値をターミナル出力に書き込みます。

    ```
    echo $accountName
    echo $resourceGroup
    ```

1. 次のコマンドを使用して、**az cosmosdb create** のオプションを表示します。

    ```
    az cosmosdb create --help
    ```

1. 事前定義された変数と次のコマンドを使用して、新しい Azure Cosmos DB アカウントを作成します。

    ```
    az cosmosdb create --name $accountName --resource-group $resourceGroup
    ```

1. **create** コマンドの実行が完了して戻るのを待ってから、このラボに進んでください。

    > &#128161; **create** コマンドが完了するまで、平均して 2 〜 12 分かかる場合があります。

## <a name="create-azure-cosmos-db-for-nosql-resources-using-the-azure-cli"></a>Azure CLI を使用して Azure Cosmos DB for NoSQL リソースを作成する

**cosmosdb sql** コマンド グループには、Azure Cosmos DB の NoSQL API 固有のリソースを管理するためのコマンドが含まれています。 いつでも **--help** フラグを使用して、これらのコマンド グループのオプションを確認できます。

1. **Visual Studio Code** 内ですでに開いているターミナル インスタンスに戻ります。

1. 次のコマンドを使用して、**Azure Cosmos DB for NoSQL** に関連するほとんどのコマンド Azure CLI コマンド グループを表示します。

    ```
    az cosmosdb sql --help
    ```

1. 次のコマンドを使用して、**Azure Cosmos DB for NoSQL** データベースを管理するための Azure CLI コマンドを表示します。

    ```
    az cosmosdb sql database --help
    ```

1. 事前定義された変数、データベース名 **cosmicworks**、および次のコマンドを使用して、新しい Azure Cosmos DB データベースを作成します。

    ```
    az cosmosdb sql database create --name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. **create** コマンドの実行が完了して戻るのを待ってから、このラボに進んでください。

1. 次のコマンドを使用して、**Azure Cosmos DB for NoSQL** コンテナーを管理するための Azure CLI コマンドを表示します。

    ```
    az cosmosdb sql container --help
    ```

1. 事前定義された変数、データベース名 **cosmicworks**、コンテナー名 **products**、および次のコマンドを使用して、新しい Azure Cosmos DB コンテナーを作成します。

    ```
    az cosmosdb sql container create --name "products" --throughput 400 --partition-key-path "/categoryId" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. **create** コマンドの実行が完了して戻るのを待ってから、このラボに進んでください。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **csms** プレフィックスの付いた **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. **NoSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、 **[スケールと設定]** を選択します。

1. **[スケール]** タブ内の値を確認します。具体的には、 **[スループット]** セクションで **[手動]** オプションが選択されており、プロビジョニングされたスループットが **400** RU/秒に設定されていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="adjust-the-throughput-of-an-existing-container-using-the-azure-cli"></a>Azure CLI を使用して既存のコンテナーのスループットを調整する

Azure CLI を使用して、スループットの手動プロビジョニングと自動スケーリング プロビジョニングの間でコンテナーを移行できます。 コンテナーが自動スケーリング スループットを使用している場合、CLI を使用して最大許容スループット値を動的に調整できます。

1. **Visual Studio Code** 内ですでに開いているターミナル インスタンスに戻ります。

1. 次のコマンドを使用して、**Azure Cosmos DB for NoSQL** コンテナー スループットを管理するための Azure CLI コマンドを表示します。

    ```
    az cosmosdb sql container throughput --help
    ```

1. 次のコマンドを使用して、**products** コンテナーのスループットを手動プロビジョニングから自動スケーリングに移行します。

    ```
    az cosmosdb sql container throughput migrate --name "products" --throughput-type autoscale --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. **migrate** コマンドの実行が完了して戻るのを待ってから、このラボに進んでください。

1. 次のコマンドを使用して、**products** コンテナーに対してクエリを実行して、考えられる最小スループット値を確認します。

    ```
    az cosmosdb sql container throughput show --name "products" --query "resource.minimumThroughput" --output "tsv" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. 次のコマンドを使用して、**products** コンテナーの最大自動スケーリング スループットを既定値の **4,000** から新しい値の **5,000** に更新します。

    ```
    az cosmosdb sql container throughput update --name "products" --max-throughput 5000 --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. **update** コマンドの実行が完了して戻るのを待ってから、このラボに進んでください。

1. **Visual Studio Code** を閉じます。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **csms** プレフィックスの付いた **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、**NoSQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. **NoSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、 **[スケールと設定]** を選択します。

1. **[スケール]** タブ内の値を確認します。具体的には、 **[スループット]** セクションで **[自動スケーリング]** オプションが選択されており、プロビジョニングされたスループットが **5,000** RU/秒に設定されていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random]: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random
