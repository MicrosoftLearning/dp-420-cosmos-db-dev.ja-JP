---
Lab:
  title: 復旧ポイントからデータベースまたはコンテナーを復旧する
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: 1e934b994735c51ad3d699474186e2255ba1f057
ms.sourcegitcommit: c3778722311b55568f083480ecc69c9b3e837a18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/19/2022
ms.locfileid: "138025141"
---
# <a name="recover-a-database-or-container-from-a-recovery-point"></a>復旧ポイントからデータベースまたはコンテナーを復旧する 

Azure では、暗号化されたデータのバックアップを自動的に実行します。 これらのバックアップは、**定期的** バックアップ モードおよび **継続的** バックアップ モードの 2 つのモードで実行されます。

このラボでは、継続的バックアップ モードを使用して `backup` および `restores` を実行します。 まず、Azure Cosmos DB アカウントを作成します。 次に、2 つのコンテナーを作成し、そこにいくつかのドキュメントを追加します。 次に、それらのコンテナー内のいくつかのドキュメントを更新します。 最後に、削除する前の時点までのアカウントの復元を作成します。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、アカウントでサポートする API を選択します (たとえば、**Mongo API** または **SQL API**)。 Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得できます。 エンドポイントとキーを使用して、Azure Cosmos DB SQL API アカウントにプログラムで接続します。 Azure SDK for .NET またはその他の SDK の接続文字列でエンドポイントとキーを使用します。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[+ リソースの作成]** を選択し、 *[Cosmos DB]* を検索してから、次の設定で新しい **[Azure Cosmos DB SQL API]** アカウント リソースを作成し、残りのすべての設定を規定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | *既存の Azure サブスクリプション* |
    | **リソース グループ** | *既存のリソース グループを選択するか、新しいものを作成します* |
    | **アカウント名** | *グローバルに一意の名前を入力します* |
    | **Location** | *使用可能な任意のリージョンを選択します* |
    | **容量モード** | *プロビジョニング済みスループット* |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

    > &#128221; **[バックアップ ポリシー]** タブで選択することにより、Azure Cosmos DB アカウントの作成中に **[継続的]** モードを有効にできることに注意してください。このラボでは、アカウントの作成中、または以下のオプションのセクションでアカウントの作成後に、この機能を有効にすることを選択できます。 **アカウントが作成された *後に* 機能を有効にするには、*5 分以上かかる場合があります*。**

    > &#128221; ラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

## <a name="add-a-database-and-two-containers-to-the-account"></a>アカウントにデータベースと 2 つのコンテナーを追加する

データベースと 2 つのコンテナーを作成しましょう。

1. Azure portal で、Azure Cosmos DB アカウント ページに移動します。

1. **[データ エクスプローラー]** で、次の設定で新しいデータベースを追加します

    | **設定** | **Value** |
    | ---: | :--- |
    | **データベース ID** | *`Sales`* |
    | **コンテナー間でスループットを共有する** | *選択しないでください* |

1. **[データ エクスプローラー]** で、次の設定で新しいコンテナーを追加します

    | **設定** | **Value** |
    | ---: | :--- |
    | **データベース ID** | *既存の* 名前を使用する: *Sales* |
    | **コンテナー ID** | *`customer`* |
    | **パーティション キー** | *`/id`* |
    | **コンテナーのスループット (400 - 無制限 RU/秒)** | *手動* スループット: *400*|

1. **[データ エクスプローラー]** で、次の設定で新しいコンテナーを追加します

    | **設定** | **Value** |
    | ---: | :--- |
    | **データベース ID** | *既存の* 名前を使用する: *Sales* |
    | **コンテナー ID** | *`salesOrder`* |
    | **パーティション キー** | *`/id`* |
    | **コンテナーのスループット (400 - 無制限 RU/秒)** | *手動* スループット: *400*|

## <a name="add-items-to-the-containers"></a>コンテナーに項目を追加する

これらのコンテナーにドキュメントをいくつか追加してみましょう。

1. Azure portal で、Azure Cosmos DB アカウント ページに移動します。

1. **[データ エクスプローラー]** で、次の 2 つのドキュメントを **[customer]** コンテナーに追加します。

```
  {
    "id": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "title": "",
    "firstName": "Franklin",
    "lastName": "Ye",
    "emailAddress": "franklin9@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0139",
    "creationDate": "2014-02-05T00:00:00",
    "addresses": [
      {
        "addressLine1": "1796 Westbury Dr.",
        "addressLine2": "",
        "city": "Melton",
        "state": "VIC",
        "country": "AU",
        "zipCode": "3337"
      }
    ],
    "password": {
      "hash": "GQF7qjEgMl3LUppoPfDDnPtHp1tXmhQBw0GboOjB8bk=",
      "salt": "12C0F5A5"
    }
  }
```

```
  {
    "id": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "title": "",
    "firstName": "Victor",
    "lastName": "Moreno",
    "emailAddress": "victor8@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0134",
    "creationDate": "2011-10-09T00:00:00",
    "addresses": [
      {
        "addressLine1": "Parkstr 42",
        "addressLine2": "",
        "city": "Hamburg",
        "state": "HH ",
        "country": "DE",
        "zipCode": "20354"
      }
    ],
    "password": {
      "hash": "n8l+wY/klP/hwTC3wSr8BLMA9tm3tGTyDsCgG/Q9EYI=",
      "salt": "AC22BC8C"
    }
  }
```
1. **[データ エクスプローラー]** で、次の 3 つのドキュメントを **[salesOrder]** コンテナーに追加します。

```
  {
    "id": "000C23D8-B8BC-432E-9213-6473DFDA2BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2014-02-16T00:00:00",
    "shipDate": "2014-02-23T00:00:00",
    "details": [
      {
        "sku": "BK-R64Y-42",
        "name": "Road-550-W Yellow, 42",
        "price": 1120.49,
        "quantity": 1
      },
      {
        "sku": "HL-U509-B",
        "name": "Sport-100 Helmet, Blue",
        "price": 34.99,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "001676F7-0B70-400B-9B7D-24BA37B97F70",
    "customerId": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "orderDate": "2013-06-02T00:00:00",
    "shipDate": "2013-06-09T00:00:00",
    "details": [
      {
        "sku": "HL-U509-R",
        "name": "Sport-100 Helmet, Red",
        "price": 34.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "0019092E-BD25-48F5-8050-7051B2655BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2013-09-14T00:00:00",
    "shipDate": "2013-09-21T00:00:00",
    "details": [
      {
        "sku": "TI-T723",
        "name": "Touring Tire",
        "price": 28.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      },
      {
        "sku": "TT-T092",
        "name": "Touring Tire Tube",
        "price": 4.99,
        "quantity": 1
      }
    ]
  }
```

## <a name="change-the-default-backup-mode-to-continuous-optional-if-feature-not-enabled-during-the-account-creation"></a>既定のバックアップ モードを [継続的] に変更します (アカウントの作成中に機能が無効になっている場合は省略可能)

*Azure Cosmos DB アカウントの作成中にこの機能を有効にしなかった場合は、ここで実行する必要があります。*  バックアップ モードの変更はシンプルです。必要なのは、1 つの設定を **[オン]** に変更することだけです。 では、変更してみましょう。

1. Azure portal で、Azure Cosmos DB アカウント ページに移動します。

1. **[設定]** セクションで、 **[機能]** を選択します。

1. **[継続的バックアップ]** オプションを選択して、この機能を有効にします。 このオプションを選択すると、ウィンドウが表示され、 **[有効]** ボタンを選択します。  この機能を有効にするには、5 分以上かかることがあります。

## <a name="delete-one-of-the-salesorder-documents"></a>SalesOrder ドキュメントの 1 つを削除する

1. **[データ エクスプローラー]** で、次のクエリを実行して現在の日付と時刻を取得します。 そのタイムスタンプをメモ帳にコピーします。 このタイムスタンプは UTC である必要があります。

```
SELECT GetCurrentDateTime ()
```

1. **[データ エクスプローラー]** で、**id** が `0019092E-BD25-48F5-8050-7051B2655BC5` である **salesOrder** ドキュメントを見つけます。 ドキュメントを削除し、ドキュメントがなくなったことを確認します。

## <a name="restore-the-database-to-the-point-before-you-deleted-the-salesorder-document"></a>SalesOrder ドキュメントを削除する前の時点にデータベースを復元する

1. Azure portal で、Azure Cosmos DB アカウント ページに移動します。

1. *[設定]* セクションで、 **[特定の時点に復元]** を選択します。 次の設定を使用します。

    | **設定** | **Value** |
    | ---: | :--- |
    | **復元ポイント (UTC)** | 日付と時刻を正しく変換します。 時刻は AM/PM 形式である必要があります|
    | **Location** | *利用可能な場所を選択しました* |
    | **復元するリソースを選択してください** | *選択したデータベース/コンテナー* |
    | **リソースの復元** | *salesOrder* |
    | **ターゲット アカウントの復元** |  ***新しい** _ _Azure Cosmos DB アカウント名を *選択** |

    > &#128221; Azure Cosmos DB の復元の場合、既存の* アカウントの上に ***決して** 復元せず、常に新しい Azure Cosmos DB アカウントを作成する必要があります。

    > &#128221; データベース全体またはアカウント全体を復元することを選択することもできますが、実際の運用環境では、データベースが巨大になる可能性があります。 多くのシナリオでは、必要なコンテナーまたはデータベースだけを復元する方が高速である場合があります。

1. この復元には 15 分以上かかることがあります。次のセクションに移動し、この復元をバックグラウンドで実行したままにしておきます。

## <a name="delete-the-customer-container"></a>customer コンテナーを削除する

1. **[データ エクスプローラー]** で、次のクエリを実行して現在の日付と時刻を取得します。 そのタイムスタンプをメモ帳にコピーします。

```
SELECT GetCurrentDateTime ()
```

1. **customer** コンテナーを削除します。

## <a name="restore-the-database-to-the-point-before-you-deleted-the-salesorder-document"></a>SalesOrder ドキュメントを削除する前の時点にデータベースを復元する

1. Azure portal で、Azure Cosmos DB アカウント ページに移動します。

1. *[設定]* セクションで、 **[特定の時点に復元]** を選択します。 次の設定を使用します。

    | **設定** | **Value** |
    | ---: | :--- |
    | **Location** | *利用可能な場所を選択しました* |
    | **復元ポイント (UTC)** | 日付と時刻を正しく変換します。 時刻は AM/PM 形式である必要があります|
    | **復元するリソースを選択してください** | *選択したデータベース/コンテナー* |
    | **リソースの復元** | *`customer`* |
    | **ターゲット アカウントの復元** |  ***新しい** _ _Azure Cosmos DB アカウント名を *選択** |

    > &#128221; Azure Cosmos DB の復元の場合、既存の* アカウントの上に ***決して** 復元せず、常に新しい Azure Cosmos DB アカウントを作成する必要があります。

    > &#128221; データベース全体またはアカウント全体を復元することを選択することもできますが、実際の運用環境では、データベースが巨大になる可能性があります。 多くのシナリオでは、必要なコンテナーまたはデータベースだけを復元する方が高速である場合があります。

1. この復元には 15 分以上かかることがあります。次のセクションに移動し、この復元をバックグラウンドで実行したままにしておきます。

## <a name="review-the-data-restored"></a>復元されたデータを確認する

データベースのサイズやその他の要因によっては、復元に時間がかかることがあります。 Azure Cosmos DB アカウントの復元が完了したら、次の操作を行います。

1. 最初の復元では、3 番目のドキュメントが復旧されていることを確認します。

1. 2 番目の復元では、customer テーブルを復元する必要があります。

## <a name="cleanup"></a>クリーンアップ

1. アカウントの復元によって作成された 2 つの新しい Azure Cosmos DB アカウントを削除します。

1. Sales データベースを削除し、必要に応じて、元の Azure Cosmos DB アカウントを削除します。


