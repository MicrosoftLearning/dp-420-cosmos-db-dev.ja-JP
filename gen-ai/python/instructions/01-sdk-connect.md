---
title: 01 - SDK を使って Azure Cosmos DB for NoSQL に接続する
lab:
  title: 01 - SDK を使って Azure Cosmos DB for NoSQL に接続する
  module: Use the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 4
parent: Python SDK labs
---

# SDK を使って Azure Cosmos DB for NoSQL に接続する

Azure SDK for Python は、多くの Azure サービスと対話するための一貫した開発者インターフェイスを提供するクライアント ライブラリのスイートです。 クライアント ライブラリは、これらのリソースを使用して操作するために使用するパッケージです。

このラボでは、Azure SDK for Python を使用して Azure Cosmos DB for NoSQL アカウントに接続します。

## 開発環境を準備する

「**Azure Cosmos DB を使用して Coplilot を構築する**」用のラボ コード リポジトリをまだクローンしておらず、またローカル環境を設定していない場合は、「[ローカル ラボ環境のセットアップ](00-setup-lab-environment.md)」の手順を参照して実行します。

## Azure Cosmos DB for NoSQL アカウントを作成する

このサイトの「**Azure Cosmos DB を使用して Copilot を構築する**」ラボ用に Azure Cosmos DB for NoSQL アカウントを既に作成している場合は、そのアカウントをこのラボに使用して、[次のセクション](#install-the-azure-cosmos-library)に進むことができます。 それ以外の場合は、「[Azure Cosmos DB を設定する](../../common/instructions/00-setup-cosmos-db.md)」の手順を参照して、ラボ モジュール全体で使用する Azure Cosmos DB for NoSQL アカウントを作成し、そのアカウントを **Cosmos DB 組み込みデータ共同作成者**ロールに割り当てることで、アカウント内のデータを管理するためのアクセス権をユーザー ID に付与してください。

## azure-cosmos ライブラリをインストールする

**azure-cosmos** ライブラリは **PyPI** で使用でき、Python プロジェクトに簡単にインストールできるようになります。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/01-sdk-connect** フォルダーを参照します。

1. **python/01-sdk-connect** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **python/01-sdk-connect** フォルダーに既に設定されているターミナルが開きます。

1. 依存関係を管理する仮想環境を作成してアクティブにする

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. 次のコマンドを使用して、[azure-cosmos][pypi.org/project/azure-cosmos] パッケージをインストールします。

   ```bash
   pip install azure-cosmos
   ```

1. [azure-identity][pypi.org/project/azure-identity] ライブラリをインストールします。すると、次のコマンドを使用して Azure 認証により Azure Cosmos DB ワークスペースに接続できます。

   ```bash
   pip install azure-identity
   ```

1. 統合ターミナルを閉じます。

## azure-cosmos ライブラリを使用する

Azure SDK for Python から Azure Cosmos DB ライブラリがインポートされたら、すぐにそのクラスを使用して Azure Cosmos DB for NoSQL アカウントに接続できます。 **CosmosClient** クラスは、Azure Cosmos DB for NoSQL アカウントへの初期接続を確立するのに使用するコア クラスです。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**python/01-sdk-connect** フォルダーを参照します。

1. **script.py** という名前の空 Python スクリプトを開きます。

1. 次の `import` ステートメントを追加して、**CosmosClient** クラスと **DefaultAzureCredential** クラスをインポートします。

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential
   ```

1. **endpoint** および **credential** という名前の変数を追加し、**endpoint** 値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定します。 **credential** 変数は、**DefaultAzureCredential** クラスの新しいインスタンスに設定する必要があります。

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; たとえば、エンドポイントが **https://dp420.documents.azure.com:443/** の場合、ステートメントは **endpoint = "https://dp420.documents.azure.com:443/"** になります。

1. **client** という名前の新しい変数を追加して、それを **endpoint** 変数と **credential** 変数により **CosmosClient** クラスの新しいインスタンスとして初期化します。

   ```python
   client = CosmosClient(endpoint, credential=credential)
   ```

1. **main** という名前の関数を追加して、アカウントのプロパティの読み取りと出力を行います。

   ```python
   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
   ```

1. **script.py** ファイルは次のようになるはずです。

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   client = CosmosClient(endpoint, credential=credential)

   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
    ```

1. **script.py** ファイルを**保存**します。

## スクリプトをテストする

これで Azure Cosmos DB for NoSQL アカウントに接続するための Python コードが完成したので、スクリプトをテストできます。 このスクリプトは、既定の整合性レベルと、最初の書き込み可能なリージョンの名前を出力します。 アカウントを作成したときに場所を指定したので、このスクリプトの結果として同じ場所の値が出力されるはずです。

1. **Visual Studio Code** で、**python/01-sdk-connect** フォルダーのコンテキスト メニューを開き、**[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. スクリプトを実行する前に、`az login` コマンドを使用して Azure にログインする必要があります。 ターミナル ウィンドウで以下を実行します。

   ```bash
   az login
   ```

1. `python` コマンドを使用してスクリプトを実行します。

   ```bash
   python script.py
   ```

1. これで、スクリプトは既定の整合性レベルと最初の書き込み可能なリージョンを出力します。 たとえば、アカウントの既定の整合性レベルが **Session** で、最初の書き込み可能なリージョンが**米国東部**である場合、スクリプトでは次のように出力されます。

   ```text
   Consistency Policy:   {'defaultConsistencyLevel': 'Session'}
   Primary Region: East US
   ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
