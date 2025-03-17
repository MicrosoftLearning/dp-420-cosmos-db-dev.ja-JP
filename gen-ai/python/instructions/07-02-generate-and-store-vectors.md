---
lab:
  title: '07.2: Azure OpenAI でベクトル埋め込みを生成し、Azure Cosmos DB for NoSQL に格納する'
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# Azure OpenAI でベクトル埋め込みを生成し、Azure Cosmos DB for NoSQL に格納する

Azure OpenAI では、`text-embedding-ada-002`、`text-embedding-3-small`、`text-embedding-3-large` モデルなど、OpenAI の高度な言語モデルにアクセスできます。 これらのモデルのいずれかを利用することで、テキスト データのベクトル表現を生成でき、それは Azure Cosmos DB for NoSQL などのベクトル ストアに格納できます。 この結果、効率的で正確な類似性検索が容易になり、関連する情報を取得し、コンテキストが豊富な対話を提供するコパイロットの能力が大幅に向上します。

このラボでは、Azure OpenAI サービスを作成し、埋め込みモデルをデプロイします。 そして、Python コードにより、それぞれの Python SDK を使用して Azure OpenAI クライアントと Cosmos DB クライアントを作成することで、製品の説明のベクトル表現を生成し、データベースに書き込みます。

> &#128721; このモジュールの前の演習は、このラボの前提条件です。 その演習を完了する必要がある場合は、このラボに必要なインフラストラクチャが提供されるため、次に進む前に完了してください。

## Azure OpenAI サービスを作成する

Azure OpenAI は、OpenAI の強力な言語モデルへの REST API アクセスを提供します。 これらのモデルは、特定のタスクに合わせて簡単に調整できます。たとえば、コンテンツの生成、要約、画像の解釈、セマンティック検索、自然言語からコードへの翻訳などです。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

2. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

3. **[+ リソースの作成]** を選択し、*Azure OpenAI* を見つけ、残りのすべての設定を既定値のままににして、次の設定で新しい **Azure OpenAI** リソースを作成します。

    | 設定 | 値 |
    | ------- | ----- |
    | **サブスクリプション** | ''*既存の Azure サブスクリプション*'' |
    | **リソース グループ** | ''*既存のリソース グループを選択するか、新しいものを作成します*'' |
    | **リージョン** | [サポート リージョンのリスト](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-embeddings#tabpanel_3_standard-embeddings)から、*`text-embedding-3-small` モデルをサポートしている利用可能なリージョンを選択します*。 |
    | **名前** | ''*グローバルに一意の名前を入力します*'' |
    | **価格レベル** | *Standard 0 を選択します* |

    > &#128221; ご利用のラボ環境には、新しいリソース グループを作成できない制限が存在する場合があります。 その場合は、事前に作成されている既存のリソース グループを使用します。

4. デプロイ タスクが完了するまで待ってから、次のタスクに進んでください。

## 埋め込みモデルをデプロイする

Azure OpenAI を使用して埋め込みを生成するには、まず目的の埋め込みモデルのインスタンスをサービス内にデプロイする必要があります。

1. Azure portal (``portal.azure.com``) で新しく作成した Azure OpenAI サービスに移動します。

2. Azure OpenAI サービスの **[概要]** ページで、ツール バーの **[Go to Azure AI Foundry portal]** リンクを選択して、**Azure AI Foundry** を起動します。

3. Azure AI Foundry で、左側のメニューから **[デプロイ]** を選択します。

4. **[モデル デプロイ]** ページで、**[モデルのデプロイ]** を選択し、ドロップダウンから **[基本モデルをデプロイする]** を選択します。

5. モデルの一覧から `text-embedding-3-small` を選択します。

    > &#128161; 推論タスク フィルターを使用して、*埋め込み*モデルのみを表示するように一覧をフィルター処理できます。

    > &#128221; `text-embedding-3-small` モデルが表示されない場合は、現在そのモデルをサポートしていない Azure リージョンを選択している可能性があります。 この場合は、このラボに `text-embedding-ada-002` モデルを使用できます。 どちらのモデルも 1536 次元のベクトルを生成するため、Azure Cosmos DB の `Products` コンテナーで定義したコンテナー ベクトル ポリシーの変更が必要ありません。

6. **[確認]** を選択して、モデルをデプロイします。

7. この演習の後半で必要になるため、Azure AI Foundry の **[モデル デプロイ]** ページにある、`text-embedding-3-small` モデル デプロイの**名前**を書き留めます。

## チャット入力候補モデルをデプロイする

埋め込みモデルに加えて、コパイロット用のチャット入力候補モデルが必要になります。 OpenAI の `gpt-4o` 大規模言語モデルを使用して、コパイロットからの応答を生成します。

1. Azure AI Foundry の **[モデル デプロイ]** ページのままで、**[モデルのデプロイ]** ボタンをもう一度選択し、ドロップダウンから **[基本モデルをデプロイする]** を選択します。

2. 一覧から **gpt-4o** チャット入力候補モデルを選択します。

3. **[確認]** を選択して、モデルをデプロイします。

4. この演習の後半で必要になるため、Azure AI Foundry の **[モデル デプロイ]** ページにある、`gpt-4o` モデル デプロイの**名前**を書き留めます。

## Cognitive Services OpenAI ユーザーに RBAC の役割を割り当てる

自分のユーザー ID で Azure OpenAI サービスと対話できるようにするには、アカウントに **Cognitive Services OpenAI ユーザー** ロールを割り当てることができます。 Azure OpenAI Service では、Azure リソースへの個々のアクセスを管理するための認可システムである、Azure のロールベースのアクセス制御 (Azure RBAC) がサポートされています。 Azure RBAC を使用すると、特定のプロジェクトに対するニーズに応じて、チーム メンバーごとに異なるレベルのアクセス許可を割り当てることができます。

> &#128221; Azure OpenAI などの Azure サービスに対して認証を行うための Microsoft Entra ID のロールベースのアクセス制御 (RBAC) は、ユーザー ロールに合わせた正確なアクセス制御によってセキュリティを強化し、不正なアクセス リスクを効果的に軽減します。 Entra ID RBAC を使用してセキュリティで保護されたアクセス管理を合理化すると、Azure サービスを利用するためのより効率的でスケーラブルなソリューションになります。

1. Azure portal (``portal.azure.com``) で、Azure OpenAI リソースに移動します。

2. 左側のナビゲーション ウィンドウで **[アクセス制御 (IAM)]** を選択します。

3. **[追加]** を選択し、 **[ロールの割り当ての追加]** を選択します。

4. **[ロール]** タブで、**[Cognitive Services OpenAI User]** ロールを選択し、**[次へ]** を選択します。

5. **[メンバー]** タブの [アクセスの割り当て先] で [ユーザー、グループ、またはサービス プリンシパル] を選択し、**[メンバーを選択する]** を選択します。

6. **[メンバーを選択する]** ダイアログで、自分の名前またはメール アドレスを検索し、自分のアカウントを選択します。

7. **[確認と 割り当て]** タブで、 **[確認と割り当て]** を選択して ロールを割り当てます。

## Python 仮想環境を作成する

Python の仮想環境は、クリーンで整理された開発スペースを維持するために不可欠であり、個々のプロジェクトが他のプロジェクトから分離された独自の依存関係セットを持つことができます。 この結果、異なるプロジェクト間の競合が回避され、開発ワークフローの一貫性が確保されます。 仮想環境を使用すると、パッケージのバージョンを簡単に管理し、依存関係の競合を回避し、プロジェクトをスムーズに実行し続けることができます。 これは、コーディング環境を安定した信頼できる状態に保ち、開発プロセスの効率を高め、問題が発生しにくくするベスト プラクティスです。

1. Visual Studio Code を使用して、「**Azure Cosmos DB を使用してコパイロットを構築する**」学習モジュールのためにクローンしたラボ コード リポジトリのクローン先フォルダー開きます。

2. Visual Studio Code で新しいターミナル ウィンドウを開き、ディレクトリを `python/07-build-copilot` フォルダーに変更します。

3. ターミナル プロンプトで次のコマンドを実行して、`.venv` という名前の仮想環境を作成します。

    ```bash
    python -m venv .venv 
    ```

    上記のコマンドを実行すると、`07-build-copilot` フォルダーの下に `.venv` フォルダーが作成されます。これにより、このラボの演習専用の Python 環境が提供されます。

4. 次の表から OS とシェルの適切なコマンドを選択し、ターミナル プロンプトで実行して、仮想環境をアクティブ化します。

    | プラットフォーム | シェル | 仮想環境をアクティブ化するコマンド |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

5. `requirements.txt`で定義されているライブラリをインストールします。

    ```bash
    pip install -r requirements.txt
    ```

    `requirements.txt` ファイルには、このラボを通して使用する Python ライブラリのセットが含まれています。

    | ライブラリ | Version | 説明 |
    | ------- | ------- | ----------- |
    | `azure-cosmos` | 4.9.0 | Python 用 Azure Cosmos DB SDK - クライアント ライブラリ |
    | `azure-identity` | 1.19.0 | Python 用 Azure Identity SDK |
    | `fastapi` | 0.115.5 | Python を使用して API をビルドするための Web フレームワーク |
    | `openai` | 1.55.2 | Python アプリから Azure OpenAI REST API にアクセスできます。 |
    | `pydantic` | 2.10.2 | Python の型ヒントを使用したデータ検証。 |
    | `requests` | 2.32.3 | HTTP 要求を送信する。 |
    | `streamlit` | 1.40.2 | Python スクリプトを対話型 Web アプリに変換する。 |
    | `uvicorn` | 0.32.1 | Python 用の ASGI Web サーバーの実装。 |
    | `httpx` | 0.27.2 | Python 用の次世代 HTTP クライアント。 |

## テキストをベクター化する Python 関数を追加する

Python SDK for Azure OpenAI では、テキスト データの埋め込みを作成するために使用できる同期クラスと非同期クラスの両方のアクセスを提供しています。 この機能は、Python コードの関数にカプセル化できます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、`python/07-build-copilot/api/app` フォルダーに移動し、その中にある `main.py` ファイルを開きます。

    > &#128221; このファイルは、次の演習でビルドするバックエンド Python API へのエントリ ポイントとして機能します。 この演習では、API によって利用される Azure Cosmos DB に埋め込みデータをインポートするために使用できる一部の非同期関数を提供します。

2. Asychronous Azure OpenAI SDK for Python を使用するには、`main.py` ファイルの先頭に次のコードを追加してライブラリをインポートします。

   ```python
   from openai import AsyncAzureOpenAI
   ```

3. Azure 認証と、以前にユーザー ID に割り当てた Entra ID RBAC ロールを使用して、Azure OpenAI と Cosmos DB に非同期的にアクセスします。 `azure-identity` ライブラリから必要なクラスをインポートするには、ファイルの先頭にある `openai` import ステートメントの下に次の行を追加します。

   ```python
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   ```

    > &#128221; API から Azure サービスを安全に操作できるようにするには、Azure Identity SDK for Python を使用します。 この方法を使用すると、前の演習で Azure Cosmos DB と Azure OpenAI にアクセスするためにアカウントに割り当てた RBAC ロールを利用するのではなく、コードからキーを格納または操作する必要がなくなります。

4. Azure OpenAI API のバージョンとエンドポイントを格納する変数を作成し、`<AZURE_OPENAI_ENDPOINT>` トークンを Azure OpenAI サービスのエンドポイント値に置き換えます。 また、埋め込みモデル デプロイの名前の変数を作成します。 ファイル内の `import` ステートメントの下に次のコードを挿入します。

   ```python
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   ```

    埋め込みデプロイ名が異なる場合は、それに応じて変数に割り当てられた値を更新します。

    > &#128161; `2024-10-21` の API バージョンは、この執筆時点の最新の GA リリース バージョンでした。 使用可能な場合は、そのバージョンまたは新しいバージョンを使用できます。 API 仕様のドキュメントには、[最新の API バージョンを含むテーブル](https://learn.microsoft.com/azure/ai-services/openai/reference#api-specs)が含まれています。

    > &#128221; `EMBEDDING_DEPLOYMENT_NAME` は、`text-embedding-3-small` モデルを Azure AI Foundry にデプロイした後にメモした **Name** 値です。 もう一度参照する必要がある場合は、Azure AI Foundry を起動し、**[デプロイメント]** ページに移動し、**[モデル名]** が `text-embedding-3-small` になっているデプロイを見つけます。 次に、その項目の **Name** フィールドの値をコピーします。 `text-embedding-ada-002` モデルをデプロイした場合は、そのデプロイの名前を使用します。

5. Azure Identity SDK for Python の `DefaultAzureCredential` クラスを使用して、変数宣言の下に次のコードを挿入して、Microsoft Entra ID RBAC 認証を使用して Azure OpenAI と Azure Cosmos DB にアクセスするための非同期資格情報を作成します。

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

6. 埋め込みの作成を処理するには、次を挿入します。これは、Azure OpenAI クライアントを使用して埋め込みを生成する関数を追加します。

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

    Azure OpenAI クライアントの作成では、Azure Identity SDK の `get_bearer_token_provider` クラスを使用してベアラー トークンを取得するため、`api_key` 値は必要ありません。

7. `main.py` ファイルは次のようになります。

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

8. `main.py` ファイルを保存します。

## 埋め込み関数をテストする

`main.py` ファイルの `generate_embeddings` 関数が正しく動作していることを確認するには、ファイルの下部に数行のコードを追加して、直接実行できるようにします。 これらの行を使用すると、埋め込むテキストを渡して、コマンド ラインから `generate_embeddings` 関数を実行できます。

1. `main.py` ファイルの下部にある `generate_embeddings` に呼び出しを含む **main guard** ブロックを追加します。

   ```python
   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

    > &#128221; `if __name__ == "__main__":` ブロックは、Python では通常、**main guard** または **entry point** と呼ばれます。 これにより、特定のコードは、スクリプトが直接実行されたときにのみ実行され、別のスクリプトのモジュールとしてインポートされる場合は実行されません。 このプラクティスは、コードの整理に役立ち、再利用可能性とモジュール性を高めます。

2. `main.py` ファイルを保存すると、次のようになります。

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

3. Visual Studio Code で、新しい統合ターミナル ウィンドウを開きます。

4. Azure OpenAI に要求を送信する API を実行する前に、`az login` コマンドを使用して Azure にログインする必要があります。 ターミナル ウィンドウで以下を実行します。

   ```bash
   az login
   ```

5. ブラウザーでログイン プロセスを完了します。

6. ターミナル プロンプトで、ディレクトリを `python/07-build-copilot` に変更します。

7. 次の表のコマンドを使用して仮想環境をアクティブ化し、OS とシェルに適したコマンドを選択して、統合ターミナル ウィンドウが Python の仮想環境内で実行されていることを確認します。

    | プラットフォーム | シェル | 仮想環境をアクティブ化するコマンド |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

8. ターミナル プロンプトで、ディレクトリを `api/app` に変更して、次のコマンドを実行します。

   ```python
   python main.py "Hello, world!"
   ```

9. ターミナル ウィンドウで出力を確認します。 浮動小数点数の配列が表示されます。これは、"Hello, world!" のベクトル表現です。 string。 次の省略された出力のようになっているはずです。

   ```bash
   [-0.019184619188308716, -0.025279032066464424, -0.0017195191467180848, 0.01884828321635723...]
   ```

## Azure Cosmos DB にデータを書き込むための関数を作成する

Azure Cosmos DB SDK for Python を使用すると、ドキュメントをデータベースにアップサートできる関数を作成できます。 アップサート操作では、一致が見つかった場合はレコードが更新され、一致がない場合は新しいレコードが挿入されます。

1. Visual Studio Code で開いている `main.py` ファイルに戻り、このファイルに既に含まれている `import` ステートメントのすぐ下に次の行を挿入して、Azure Cosmos DB SDK for Python から非同期 `CosmosClient` クラスをインポートします。

   ```python
   from azure.cosmos.aio import CosmosClient
   ```

2. `api/app` フォルダーの *models* モジュールから `Product` クラスを参照する別の import ステートメントを追加します。 `Product` クラスは、Cosmic Works データセット内の製品の形状を定義します。

   ```python
   from models import Product
   ```

3. Azure Cosmos DB に関連付けられている構成値を含む変数の新しいグループを作成し、前に挿入した Azure OpenAI 変数の下の `main.py` ファイルに追加します。 `<AZURE_COSMOSDB_ENDPOINT>` トークンを Azure Cosmos DB アカウントのエンドポイントに必ず置き換えてください。

   ```python
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
   ```

4. ドキュメントを Cosmos DB にアップサート (更新または挿入) するための `upsert_product` という名前の関数を追加し、`main.py` ファイルの `generate_embeddings` 関数の下に次のコードを挿入します。

   ```python
   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
   ```

5. `main.py` ファイルを保存すると、次のようになります。

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"

   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
    
   if __name__ == "__main__":
       import sys
       print(generate_embeddings(sys.argv[1]))
   ```

## サンプル データのベクター化

これで、`generate_embeddings` 関数と `upsert_document` 関数の両方を一緒にテストする準備ができました。 これを行うには、GitHub から Cosmic Works の製品情報を含むサンプル データ ファイルをダウンロードして、各製品の `description` フィールドをベクター化し、Azure Cosmos DB データベースの `Products` コンテナー内にドキュメントをアップサートするコードで `if __name__ == "__main__"` main guard ブロックを上書きします。

> &#128221; このアプローチは、Azure OpenAI を使用して生成し、Azure Cosmos DB に埋め込みを格納する手法を示すために使用されています。 ただし、実際のシナリオでは、Azure Cosmos DB 変更フィードによってトリガーされる Azure 関数を使用するなど、より堅牢なアプローチが、既存のドキュメントと新しいドキュメントへの埋め込みの追加を処理する場合に適しています。

1. `main.py` ファイルで、`if __name__ == "__main__":` コード ブロックを次のように上書きします。

   ```python
   if __name__ == "__main__":
       import asyncio
       from models import Product
       import requests
    
       async def main():
           product_raw_data = "https://raw.githubusercontent.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs/refs/heads/main/data/07/products.json?v=1"
           products = [Product(**data) for data in requests.get(product_raw_data).json()]
    
           # Call the generate_embeddings function, passing in an argument from the command line.    
           for product in products:
               print(f"Generating embeddings for product: {product.name}", end="\r")
               product.embedding = await generate_embeddings(product.description)
               await upsert_product(product.model_dump())
    
           print("All products with vectorized descriptions have been upserted to the Cosmos DB container.")
           # Close open credential sessions
           await credential.close()
    
       asyncio.run(main())
   ```

2. Visual Studio Code で開いている統合ターミナル プロンプトで、コマンドを使用して `main.py` ファイルをもう一度実行します。

   ```python
   python main.py
   ```

3. コードの実行が完了するまで待ちます。完了すると、ベクター化された説明を含むすべての製品が Cosmos DB コンテナーにアップサートされたことを示すメッセージが表示されます。 products データセット内の 295 のレコードのベクター化とデータ アップサート プロセスが完了するまでに、最大で 10 ～ 15 分かかる場合があります。 すべての製品が挿入されていない場合は、上記のコマンドを使用して `main.py` を再度実行し、残りの製品を追加します。

## Cosmos DB でアップサートされたサンプル データを確認する

1. Azure Portal (``portal.azure.com``) に戻り、Azure Cosmos DB アカウントに移動します。

2. 左側のナビゲーション メニューで、**Data Explorer** を選択します。

3. **CosmicWorks** データベースと **Products** コンテナーを展開し、コンテナーの下にある **Items** を選択します。

4. コンテナー内のいくつかのランダムなドキュメントを選択し、`embedding` フィールドにベクター配列が設定されていることを確認します。
