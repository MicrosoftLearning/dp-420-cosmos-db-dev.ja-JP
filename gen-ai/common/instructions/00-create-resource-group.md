---
title: ラボ リソース グループを作成する
lab:
  title: ラボ リソース グループを作成する
  module: Setup
layout: default
nav_order: 1
parent: Common setup instructions
---

# ラボ用の Azure リソース グループを作成する

このラボを完了する前に、新しくデプロイされた Azure リソースを配置するための新しい[リソース グループ][docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]を作成する必要があります。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[ホーム]** ページで、**[リソース グループ]** を選択します。

    > &#128161; または、**&#8801;** メニューを展開し、**[すべてのサービス]** を選択し、**[すべて]** カテゴリで、**[リソース グループ]** を選択します。

1. **[+ 作成]** を選択します。

1. **[リソース グループの作成]** ポップアップで、次の設定を使用して新しいリソース グループを作成し、残りの設定はすべて既定値のままにします。

    | **設定** | **Value** |
    | ---: | :--- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | *リソース グループに一意の名前を付ける* |
    | **リージョン** | ''*使用可能なリージョンを選びます*'' |

1. デプロイ タスクが完了するまで待ってから、このタスクを続行してください。

[docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]: https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal
