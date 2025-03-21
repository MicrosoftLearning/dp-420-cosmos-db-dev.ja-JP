---
lab:
  title: リソース プロバイダーを有効にする
  module: Setup
---

# Azure リソース プロバイダーを登録する

Azure サブスクリプションに登録する必要のあるリソース プロバイダーがいくつかあります。 次の手順に従って、登録されていることを確認してください。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[ホーム]** ページで、**[サブスクリプション]** を選択します。

    > &#128161; または、**&#8801;** メニューを展開し、**[すべてのサービス]** を選択し、**[すべて]** カテゴリで、**[サブスクリプション]** を選択します。

1. Azure サブスクリプションを選択します。

1. サブスクリプションのブレードにある **[設定]** セクションで、**[リソース プロバイダー]** を選択します。

1. リソース プロバイダーの一覧で、次のプロバイダーが登録されていることを確認します。
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.Insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; プロバイダーが登録されていない場合は、そのプロバイダーを選択して、**[登録]** を選択します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites
