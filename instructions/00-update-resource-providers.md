---
lab:
  title: リソース プロバイダーを有効にする
  module: Setup
ms.openlocfilehash: 4bb2ea5cdd9d123d1b235b28da67f295b9e26969
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025089"
---
# <a name="enable-azure-resource-providers"></a>Azure リソース プロバイダーを登録する

Azure サブスクリプションに登録する必要のあるリソース プロバイダーがいくつかあります。 次の手順に従って、登録されていることを確認してください。

1. 新しい Web ブラウザー ウィンドウまたはタブ内で、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[ホーム]** ページで、 **[サブスクリプション]** を選択します。

    > &#128161; または、 **&#8801;** メニューを展開し、 **[すべてのサービス]** を選択し、 **[すべて]** カテゴリで、 **[サブスクリプション]** を選択します。

1. Azure サブスクリプションを選択します。

    > &#128221; 複数のサブスクリプションがある場合は、Azure Pass を利用して作成したものを選択します。

1. サブスクリプションのブレードにある **[設定]** セクションで、 **[リソース プロバイダー]** を選択します。

1. リソース プロバイダーの一覧で、次のプロバイダーが登録されていることを確認します。
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.Insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; プロバイダーが登録されていない場合は、そのプロバイダーを選択して、 **[登録]** を選択します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites
