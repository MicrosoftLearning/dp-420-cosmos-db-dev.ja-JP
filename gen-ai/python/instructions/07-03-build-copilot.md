---
title: '07.3: Python と Azure Cosmos DB for NoSQL を使用してコパイロットをビルドする'
lab:
  title: '07.3: Python と Azure Cosmos DB for NoSQL を使用してコパイロットをビルドする'
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 12
parent: Python SDK labs
---

# Python と Azure Cosmos DB for NoSQL を使用してコパイロットをビルドする

Python の汎用性の高いプログラミング機能と Azure Cosmos DB のスケーラブルな NoSQL データベースおよびベクトル検索機能を利用することで、強力で効率的な AI コパイロットを作成し、複雑なワークフローを合理化できます。

このラボでは、Python と Azure Cosmos DB for NoSQL を使用してコパイロットをビルドし、Azure サービス (Azure OpenAI や Azure Cosmos DB) とのやりとりに必要なエンドポイントを提供するバックエンド API と、ユーザーによるコパイロットとの対話を容易にするフロントエンド UI を作成します。 コパイロットは、Cosmic Works ユーザーが自転車関連の製品を管理したり探したりするのを支援するアシスタントとして機能します。 具体的には、コパイロットにより、ユーザーは、製品のカテゴリに対する割引の適用および解除、利用可能な製品の種類をユーザーに知らせるのに役立つ製品カテゴリの検索、およびベクトル検索を使用した製品の類似性検索の実行ができるようになります。

![Streamlit を使用して Python で開発された UI、Python で記述されたバックエンド API、および Azure Cosmos DB や Azure OpenAI との対話を示す、コパイロット アーキテクチャ概要図。](media/07-copilot-high-level-architecture-diagram.png)

Python でコパイロットを作成するときにアプリの機能を専用の UI とバックエンド API に分離すると、いくつかの利点が得られます。 まず、モジュール性と保守性が向上し、UI またはバックエンドを他の要素を中断することなく個別に更新できるようになります。 Streamlit は、ユーザーの操作が簡素化された直感的な対話型インターフェイスを提供し、FastAPI では、高パフォーマンスの非同期要求処理とデータ処理が実現されます。 この分離により、さまざまなコンポーネントを複数のサーバーにデプロイでき、リソースの使用を最適化できるため、スケーラビリティも向上します。 さらに、バックエンド API では機密データと認証を個別に処理できるため、UI レイヤーの脆弱性が露出するリスクが軽減されるため、セキュリティ プラクティスが向上します。 このアプローチにより、より堅牢かつ効率的で使いやすいアプリケーションが実現します。

> &#128721; このモジュールにおける前の演習は、このラボの前提条件です。 これらの演習のいずれかを完了する必要がある場合は、このラボに必要なインフラストラクチャとスタート コードが提供されるため、次に進む前に完了してください。

## バックエンド API を構築する

コパイロットのバックエンド API は、複雑なデータを処理し、リアルタイムの分析情報を提供し、多様なサービスとシームレスに接続する機能を強化し、対話がより動的かつ有益になります。 コパイロットの API をビルドするには、FastAPI Python ライブラリを使用します。 FastAPI は、標準の Python 型ヒントに基づいて Python により API をビルドできるように設計された、最新の高パフォーマンス Web フレームワークです。 このアプローチを使用してバックエンドからコパイロットを切り離すことで、柔軟性、保守容易性、スケーラビリティが向上し、バックエンドの変更なしで独立してコパイロットを発展させることができます。

> &#128721; バックエンド API は、前の演習で `python/07-build-copilot/api/app` フォルダー内の `main.py` ファイルに追加したコードに基づきます。 前の演習をまだ完了していない場合は、次に進む前に完了してください。

1. Visual Studio Code を使用して、「**Azure Cosmos DB を使用してコパイロットを構築する**」学習モジュールのためにクローンしたラボ コード リポジトリのクローン先フォルダー開きます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**python/07-build-copilot/api/app** フォルダーを参照し、その中にある `main.py` ファイルを開きます。

1. `main.py` ファイルの先頭にある既存の `import` ステートメントの下に次のコード行を追加して、FastAPI で非同期アクションを実行するのに使用するライブラリを取り込みます。

   ```python
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
   ```

1. 作成する `/chat` エンドポイントが要求本文でデータを受信できるようにするために、プロジェクト *models* モジュールで定義されている `CompletionRequest` オブジェクトを介してコンテンツを渡します。 ファイルの先頭にある `from models import Product` import ステートメントを、`models` モジュールの `CompletionRequest` クラスをインクルードするように更新します。 import ステートメントは次のようになるはずです。

   ```python
   from models import Product, CompletionRequest
   ```

1. Azure OpenAI Service で作成したチャット入力候補モデルのデプロイ名が必要になります。 Azure OpenAI 構成変数ブロックの下部に変数を作成して、それを提供します。

   ```python
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
   ```

    入力候補デプロイ名が異なる場合は、適宜変数に割り当てられた値を更新します。

1. Azure Cosmos DB と ID SDK には、これらのサービスを操作するための非同期メソッドが用意されています。 これらの各クラスは API 内の複数の関数で使用されるため、各クラスのグローバル インスタンスを作成し、メソッド間で同じクライアントを共有できるようにします。 Cosmos DB 構成変数ブロックの下に、次のグローバル変数宣言を挿入します。

   ```python
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   ```

1. 付与された機能は次の手順で定義する `lifespan` 関数に移動するため、ファイルから次のコード行を削除します。

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

1. `CosmosClient` クラスと `DefaultAzureCredentail` クラスのシングルトン インスタンスを作成するために、FastAPI の `lifespan` オブジェクトを利用します。このメソッドは、API アプリのライフサイクルを通じてこれらのクラスを管理します。 次のコードを挿入して、`lifespan` を定義します。

   ```python
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
   ```

   FastAPI では、ライフスパン イベントは、アプリケーションのライフ サイクルの開始時と終了時に実行される特殊な操作です。 この操作は、アプリが要求の処理を開始する前とその処理を停止した後に実行されるため、アプリケーション全体で使用されるリソースや、要求間で共有されるリソースの、初期化とクリーンアップに最適です。 このアプローチにより、要求が処理される前に必要なセットアップが完了し、シャットダウン時にリソースが適切に管理されるようになります。

1. 次のコードを使用して、FastAPI クラスのインスタンスを作成します。 これは、`lifespan` 関数の下に挿入する必要があります。

   ```python
   app = FastAPI(lifespan=lifespan)
   ```

   `FastAPI()` を呼び出すことで、FastAPI アプリケーションの新しいインスタンスを初期化します。 このインスタンスは `app` といい、Web アプリケーションのメイン エントリ ポイントとして機能します。 `lifespan` に渡すと、ライフスパン イベント ハンドラーがアプリにアタッチされます。

1. 次に、API のエンドポイントをスタブ アウトします。 `api_status` メソッドは API のルート URL にアタッチされ、API が正常に稼働していることを示すステータス メッセージとして機能します。 この演習の後半で、`/chat` エンドポイントをビルドします。 Cosmos DB クライアント、データベース、コンテナーを作成するコードの下に、次のコードを挿入します。

   ```python
   @app.get("/")
   async def api_status():
       """Display a status message for the API"""
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

1. ファイルがコマンド ラインから実行されたときに、ファイルの下部にあるメイン ガード ブロックを上書きして、`uvicorn` ASGI (非同期サーバー ゲートウェイ インターフェイス) Web サーバーを起動します。

   ```python
   if __name__ == "__main__":
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. `main.py` ファイルを保存します。 これで、前の演習で追加した `generate_embeddings` メソッドや `upsert_product` メソッドなどは、次のようになります。

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product, CompletionRequest
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
    
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
    
   app = FastAPI(lifespan=lifespan)
    
   @app.get("/")
   async def api_status():
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """ Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
    
   async def generate_embeddings(text: str):
       # Create Azure OpenAI client
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
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. API をすばやくテストするには、Visual Studio Code で新しい統合ターミナル ウィンドウを開きます。

1. `az login` コマンドを使用して Azure にログインしていることを確認します。 ターミナル プロンプトで次を実行します。

   ```bash
   az login
   ```

1. ブラウザーでログイン プロセスを完了します。

1. ターミナル プロンプトでディレクトリを `python/07-build-copilot` に変更します。

1. 次の表のコマンドを使用してアクティブ化し、OS とシェルに適したコマンドを選択して、統合ターミナル ウィンドウが Python 仮想環境内で実行されていることを確認します。

    | プラットフォーム | シェル | 仮想環境をアクティブ化するコマンド |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

1. ターミナル プロンプトでディレクトリを `api/app` に変更し、次のコマンドを実行して FastAPI Web アプリを実行します。

   ```bash
   uvicorn main:app
   ```

1. 自動的に開かない場合は、新しい Web ブラウザーのウィンドウまたはタブを起動し、<http://127.0.0.1:8000> に移動します。

    ブラウザー ウィンドウの `{"status":"ready"}` のメッセージは、API が実行されていることを示しています。

1. URL: <http://127.0.0.1:8000/docs> の末尾に `/docs` を追加して、API の Swagger UI に移動します。

    > &#128221; Swagger UI は、OpenAPI 仕様から生成された API エンドポイントを探索およびテストするための対話型の Web ベースのインターフェイスです。 これにより、開発者とユーザーはリアルタイムの API 呼び出しを視覚化、操作、デバッグできるようになり、使いやすさとドキュメント化が強化されます。

1. Visual Studio Code に戻り、関連付けられている統合ターミナル ウィンドウで **CTRL + C** キーを押して API アプリを停止します。

## Azure Cosmos DB の製品データを組み込む

Azure Cosmos DB のデータを活用することで、Copilot は複雑なワークフローを合理化し、ユーザーがタスクを効率的に完了できるように支援します。 Copilot は、レコードを更新し、リアルタイムで lookup 値を取得し、正確でタイムリーな情報を確保できます。 この機能により、Copilot は高度な対話の提供が可能で、ユーザーのタスクをすばやく正確にナビゲートして完了する機能が強化されています。

関数を使用すると、製品管理のコパイロットがカテゴリ内の製品に割引を適用できるようになります。 この関数は、コパイロットが Azure Cosmos DB から Cosmos Works 製品データを取得して操作するメカニズムになります。

1. コパイロットは、`apply_discount` という名前の非同期関数を使用して、指定されたカテゴリ内の製品に対し割引と販売価格を追加および削除します。 `main.py` ファイルの下部付近にある `upsert_product` 関数の下に、次の関数コードを挿入します。

   ```python
   async def apply_discount(discount: float, product_category: str) -> str:
       """Apply a discount to products in the specified category."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT * FROM Products p WHERE CONTAINS(LOWER(p.category_name), LOWER(@product_category))
           """,
           parameters = [
               {"name": "@product_category", "value": product_category}
           ]
       )
    
       # Apply the discount to the products
       async for item in query_results:
           item['discount'] = discount
           item['sale_price'] = item['price'] * (1 - discount) if discount > 0 else item['price']
           await container.upsert_item(item)
    
       return f"A {discount}% discount was successfully applied to {product_category}." if discount > 0 else f"Discounts on {product_category} removed successfully."
   ```

    この関数は、Azure Cosmos DB で検索を実行して、カテゴリ内のすべての製品をプルし、要求された割引をその製品に適用します。 また、指定された割引を使用して品目の販売価格を計算し、データベースに挿入します。

2. 次に、`get_category_names` という名前の 2 つ目の関数を追加します。この関数は、製品の割引を適用または削除するときに利用可能な製品カテゴリを把握するのに役立てるために、コパイロットが呼び出します。 ファイル内の `apply_discount` 関数の下に次のメソッドを追加します。

   ```python
   async def get_category_names() -> list:
       """Retrieve the names of all product categories."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
       # Get distinct product categories
       query_results = container.query_items(
           query = "SELECT DISTINCT VALUE p.category_name FROM Products p"
       )
       categories = []
       async for category in query_results:
           categories.append(category)
       return list(categories)
   ```

    `get_category_names` 関数は、`Products` コンテナーに対してクエリを実行し、データベースから個別のカテゴリ名の一覧を取得します。

3. `main.py` ファイルを保存します。

## チャット エンドポイントを実装する

バックエンド API の `/chat` エンドポイントは、フロントエンド UI が Azure OpenAI モデルおよび内部の Cosmos Works 製品データと対話するインターフェイスとして機能します。 このエンドポイントは通信ブリッジとして機能し、UI 入力を Azure OpenAI サービスに送信できるようになり、この入力は高度な言語モデルを使用して処理されます。 その後、結果がフロントエンドに返され、リアルタイムのインテリジェントな会話が実現します。 開発者はこの設定を活用することで、バックエンドで自然言語の処理と適切な応答の生成という複雑なタスクを処理しながら、シームレスで応答性の高いユーザー エクスペリエンスを実現できます。 このアプローチでは、基になる AI インフラストラクチャからフロントエンドを切り離すことで、スケーラビリティと保守性もサポートされます。

1. `main.py` ファイル内で以前に追加した `/chat` エンドポイント スタブを見つけます。

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

    この関数は、`CompletionRequest` をパラメーターとして受け取ります。 入力パラメーターにクラスを使用すると、要求本文で API エンドポイントに複数のプロパティを渡すことができます。 `CompletionRequest` クラスは、*models* モジュール内で定義され、ユーザー メッセージ、チャット履歴、および最大履歴プロパティが入っています。 チャット履歴を使用すると、コパイロットがユーザーとの会話の過去の側面を参照できるようになるため、ディスカッション全体のコンテキストに関する知識が維持されます。 `max_history` プロパティを使用すると、LLM のコンテキストに渡す履歴メッセージの数を定義できます。 この結果、プロンプトのトークンの使用を制御し、要求に対する TPM の制限を回避できるようになります。

2. 開始するには、エンドポイントの実装プロセスを開始するときに、関数から `raise NotImplementedError("The chat endpoint is not implemented yet.")` の行を削除します。

3. チャット エンドポイント メソッド内で最初に行うことは、システム プロンプトの提供です。 このプロンプトでは、コパイロットの "ペルソナ" を定義し、コパイロットがユーザーと対話する方法、質問に応答する方法、および使用可能な関数を利用してアクションを実行する方法を指示します。

   ```python
   # Define the system prompt that contains the assistant's persona.
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   """
   ```

4. 次に、LLM に送信するメッセージの配列を作成し、システム プロンプト、チャット履歴内のメッセージ、および受信ユーザー メッセージを追加します。 このコードは、関数のシステム プロンプト宣言のすぐ下に配置する必要があります。

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{"role": "system", "content": system_prompt }]
    
   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)
    
   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

    `messages` プロパティは、進行中の会話の履歴をカプセル化します。 これにはユーザー入力のシーケンス全体と AI の応答が含まれており、モデルがコンテキストを維持するのに役立ちます。 この履歴を参照することで、AI はコンテキストに関連した一貫性のある応答を生成でき、対話がスムーズかつ動的であり続けます。 このプロパティは、進行するにつれて会話の流れとニュアンスを AI が理解できるようにする上で重要です。

5. 上記で定義した関数によりコパイロットが Azure Cosmos DB からのデータを操作できるようにするには、"ツール" のコレクションを定義する必要があります。 LLM は、実行の一環としてこのツールを呼び出します。 Azure OpenAI では、関数定義を使用して、AI とさまざまなツールまたは API との間の構造化された相互作用が可能になります。 関数が定義されると、実行できる操作、必要なパラメーター、および求められる入力が記述されます。 `tools`の配列を作成するには、以前に定義した `apply_discount` メソッドと `get_category_names` メソッドの関数定義を含む次のコードを指定します。

   ```python
   # Define function calling tools
   tools = [
       {
           "type": "function",
           "function": {
               "name": "apply_discount",
               "description": "Apply a discount to products in the specified category",
               "parameters": {
                   "type": "object",
                   "properties": {
                       "discount": {"type": "number", "description": "The percent discount to apply."},
                       "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                   },
                   "required": ["discount", "product_category"]
               }
           }
       },
       {
           "type": "function",
           "function": {
               "name": "get_category_names",
               "description": "Retrieves the names of all product categories"
           }
       }
   ]
   ```

    Azure OpenAI では、関数定義を使用することで、AI と外部システム間の相互作用が、適切に整理され、セキュリティで保護され、効率的になります。 この構造化されたアプローチにより、AI は複雑なタスクをシームレスかつ高い信頼性で実行でき、全体的な機能とユーザー エクスペリエンスが向上します。

6. チャット入力候補モデルに要求を行う非同期 Azure OpenAI クライアントを作成します。

   ```python
   # Create Azure OpenAI client
   aoai_client = AsyncAzureOpenAI(
       api_version = AZURE_OPENAI_API_VERSION,
       azure_endpoint = AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
   )
   ```

7. チャット エンドポイントは、関数呼び出しを利用するために Azure OpenAI を 2 回呼び出します。 1 回目では、ツールへの Azure OpenAI クライアント アクセスが提供されます。

   ```python
   # First API call, providing the model to the defined functions
   response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages,
       tools = tools,
       tool_choice = "auto"
   )
    
   # Process the model's response and add it to the conversation history
   response_message = response.choices[0].message
   messages.append(response_message)
   ```

8. この 1 回目の呼び出しからの応答には、要求に応答するために必要と判断したツールまたは関数に関する LLM からの情報が含まれます。 関数呼び出しの出力を処理するコードを入れ、その出力を会話履歴に挿入することで、LLM がそれを使用して、その出力に含まれるデータに対する応答を作成できるようにする必要があります。

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

    Azure OpenAI での関数呼び出しにより、外部 API またはツールをモデルの出力に直接シームレスに統合できます。 モデルは、関連する要求を検出すると、必要なパラメーターを持つ JSON オブジェクトを構築し、その後ユーザーがそれを実行します。 結果がモデルに返され、外部データで強化された包括的な最終応答を提供できるようになります。

9. Azure Cosmos DB からエンリッチされたデータを使用して要求を完了するには、入力候補を生成するための 2 回目の要求を Azure OpenAI に送信する必要があります。

   ```python
   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )
   ```

10. 最後に、UI に入力候補の応答を返します。

   ```python
   return final_response.choices[0].message.content
   ```

11. `main.py` ファイルを保存します。 `/chat` エンドポイントの `generate_chat_completion` メソッドは次のようになるはずです。

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       # Define the system prompt that contains the assistant's persona.
       system_prompt = """
       You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
       You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
       If asked to apply a discount:
           - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
           - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
       If asked to remove discounts from a category:
           - Remove any discounts applied to products in the specified category by setting the discount value to 0.
       """
       # Provide the copilot with a persona using the system prompt.
       messages = [{ "role": "system", "content": system_prompt }]
    
       # Add the chat history to the messages list
       for message in request.chat_history[-request.max_history:]:
           messages.append(message)
    
       # Add the current user message to the messages list
       messages.append({"role": "user", "content": request.message})
    
       # Define function calling tools
       tools = [
           {
               "type": "function",
               "function": {
                   "name": "apply_discount",
                   "description": "Apply a discount to products in the specified category",
                   "parameters": {
                       "type": "object",
                       "properties": {
                           "discount": {"type": "number", "description": "The percent discount to apply."},
                           "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                       },
                       "required": ["discount", "product_category"]
                   }
               }
           },
           {
               "type": "function",
               "function": {
                   "name": "get_category_names",
                   "description": "Retrieves the names of all product categories"
               }
           }
       ]
       # Create Azure OpenAI client
       aoai_client = AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
    
       # First API call, providing the model to the defined functions
       response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages,
           tools = tools,
           tool_choice = "auto"
       )
    
       # Process the model's response
       response_message = response.choices[0].message
       messages.append(response_message)
    
       # Handle function call outputs
       if response_message.tool_calls:
           for call in response_message.tool_calls:
               if call.function.name == "apply_discount":
                   func_response = await apply_discount(**json.loads(call.function.arguments))
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": func_response
                       }
                   )
               elif call.function.name == "get_category_names":
                   func_response = await get_category_names()
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": json.dumps(func_response)
                       }
                   )
       else:
           print("No function calls were made by the model.")
    
       # Second API call, asking the model to generate a response
       final_response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages
       )
    
       return final_response.choices[0].message.content
   ```

## シンプルなチャット UI をビルドする

Streamlit UI には、ユーザーがコパイロットと対話するためのインターフェイスが用意されています。

1. この UI は、`python/07-build-copilot/ui` フォルダーにある `index.py` ファイルを使用して定義されます。

2. `index.py` ファイルを開き、ファイルの先頭に次の import ステートメントを追加して開始します。

   ```python
   import streamlit as st
   import requests
   ```

3. `import` ステートメントの下に次の行を追加して、`index.py` ファイル内で定義された Streamlit ページを構成します。

   ```python
   st.set_page_config(page_title="Cosmic Works Copilot", layout="wide")
   ```

4. UI は、`requests` ライブラリを使用してバックエンド API と対話し、API で定義した `/chat` エンドポイントを呼び出します。 現在のユーザー メッセージとチャット履歴からのメッセージの一覧を要求するメソッドで API 呼び出しをカプセル化できます。

   ```python
   async def send_message_to_copilot(message: str, chat_history: list = []) -> str:
       """Send a message to the Copilot chat endpoint."""
       try:
           api_endpoint = "http://localhost:8000"
           request = {"message": message, "chat_history": chat_history}
           response = requests.post(f"{api_endpoint}/chat", json=request, timeout=60)
           return response.json()
       except Exception as e:
           st.error(f"An error occurred: {e}")
           return""
   ```

5. アプリケーションへの呼び出しのエントリ ポイントである `main` 関数を定義します。

   ```python
   async def main():
       """Main function for the Cosmic Works Product Management Copilot UI."""
    
       st.write(
           """
           # Cosmic Works Product Management Copilot
        
           Welcome to Cosmic Works Product Management Copilot, a tool for managing and finding bicycle-related products in the Cosmic Works system.
        
           **Ask the copilot to apply or remove a discount on a category of products or to find products.**
           """
       )
    
       # Add a messages collection to the session state to maintain the chat history.
       if "messages" not in st.session_state:
           st.session_state.messages = []
    
       # Display message from the history on app rerun.
       for message in st.session_state.messages:
           with st.chat_message(message["role"]):
               st.markdown(message["content"])
    
       # React to user input
       if prompt := st.chat_input("What can I help you with today?"):
           with st. spinner("Awaiting the copilot's response to your message..."):
               # Display user message in chat message container
               with st.chat_message("user"):
                   st.markdown(prompt)
                
               # Send the user message to the copilot API
               response = await send_message_to_copilot(prompt, st.session_state.messages)
    
               # Display assistant response in chat message container
               with st.chat_message("assistant"):
                   st.markdown(response)
                
               # Add the current user message and assistant response messages to the chat history
               st.session_state.messages.append({"role": "user", "content": prompt})
               st.session_state.messages.append({"role": "assistant", "content": response})
   ```

6. 最後に、ファイルの末尾に **main guard** ブロックを追加します。

   ```python
   if __name__ == "__main__":
       import asyncio
       asyncio.run(main())
   ```

7. `index.py` ファイルを保存します。

## UI 経由でコパイロットをテストする

1. API プロジェクトのために Visual Studio Code で開いた統合ターミナル ウィンドウに戻り、次のように入力して API アプリを起動します。

   ```bash
   uvicorn main:app
   ```

2. 新しい統合ターミナル ウィンドウを開き、ディレクトリを `python/07-build-copilot` に変更して Python 環境をアクティブ化した後、ディレクトリを `ui` フォルダーに変更し、次を実行して UI アプリを起動します。

   ```bash
   python -m streamlit run index.py
   ```

3. UI がブラウザー ウィンドウで自動的に開かない場合は、新しいブラウザー タブまたはウィンドウを起動し、<http://localhost:8501> にアクセスして UI を開きます。

4. UI のチャット プロンプトで、「割引を適用してください」と入力し、メッセージを送信します。

    アクションの詳細をコパイロットに提供する必要があったため、応答は、適用する割引率や割引を適用する製品カテゴリの指定など、追加情報を要求するものとなるはずです。

5. 利用可能なカテゴリを理解するために、製品カテゴリの一覧を提供するようにコパイロットに依頼します。

    コパイロットは、`get_category_names` 関数を使用して関数呼び出しを行い、そのカテゴリで会話メッセージをエンリッチして、その内容に応じて応答できるようになります。

6. また、「衣服関連のカテゴリの一覧を提供してください」などのように、より具体的なカテゴリのセットを要求することもできます。

7. 次に、すべての衣料品に 15% の割引を適用するようにコパイロットに依頼します。

8. Azure portal で Azure Cosmos DB アカウントを開き、**[データ エクスプローラー]** を選択し、`Products` コンテナーに対しクエリを実行して、次のような "clothing" カテゴリのすべての製品を表示することで、価格の割引が適用されたことを確認できます。

   ```sql
   SELECT c.category_name, c.name, c.description, c.price, c.discount, c.sale_price FROM c
   WHERE CONTAINS(LOWER(c.category_name), "clothing")
   ```

    クエリ結果の各品目の `discount` 値が `0.15` であり、`sale_price` が元の `price` より 15% 小さくなっていることを確認します。

9. Visual Studio Code に戻り、API アプリが動作しているターミナル ウィンドウで **Ctrl + C** キーを押してアプリを停止します。 UI は実行したままにしておくことができます。

## ベクトル検索を統合する

ここまでで、製品に割引を適用するアクションを実行する機能をコパイロットに与えましたが、コパイロットにはデータベース内に格納されている製品に関する知識はまだありません。 このタスクでは、特定の品質の製品について尋ね、データベース内にある類似の製品を見つけることができるようになる、ベクトル検索機能を追加します。

1. `api/app` フォルダー内の `main.py` ファイルに戻り、Azure Cosmos DB アカウントの `Products` コンテナーに対してベクトル検索を実行するメソッドを追加します。 このメソッドは、ファイルの下部付近にある既存の関数の下に挿入できます。

   ```python
   async def vector_search(query_embedding: list, num_results: int = 3, similarity_score: float = 0.25):
       """Search for similar product vectors in Azure Cosmos DB"""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT TOP @num_results p.name, p.description, p.sku, p.price, p.discount, p.sale_price, VectorDistance(p.embedding, @query_embedding) AS similarity_score
           FROM Products p
           WHERE VectorDistance(p.embedding, @query_embedding) > @similarity_score
           ORDER BY VectorDistance(p.embedding, @query_embedding)
           """,
           parameters = [
               {"name": "@query_embedding", "value": query_embedding},
               {"name": "@num_results", "value": num_results},
               {"name": "@similarity_score", "value": similarity_score}
           ]
       )
       similar_products = []
       async for result in query_results:
           similar_products.append(result)
       formatted_results = [{'similarity_score': product.pop('similarity_score'), 'product': product} for product in similar_products]
       return formatted_results
   ```

2. 次に、LLM がデータベースに対してベクトル検索を実行するのに使用する関数として機能する `get_similar_products` という名前のメソッドを作成します。

   ```python
   async def get_similar_products(message: str, num_results: int):
       """Retrieve similar products based on a user message."""
       # Vectorize the message
       embedding = await generate_embeddings(message)
       # Perform vector search against products in Cosmos DB
       similar_products = await vector_search(embedding, num_results=num_results)
       return similar_products
   ```

    `get_similar_products` 関数は、前の演習で作成した `generate_embeddings` 関数と同様に、上記で定義した `vector_search` 関数を非同期で呼び出します。 埋め込みは、Cosmos DB の組み込みの `VectorDistance` 関数を使用してデータベースに格納されているベクトルと比較できるように、受信ユーザー メッセージに生成されます。

3. LLM が新しい関数を使用できるようにするには、先ほど作成した `tools` 配列を更新し、`get_similar_products` メソッドの関数定義を追加する必要があります。

   ```json
   {
       "type": "function",
       "function": {
           "name": "get_similar_products",
           "description": "Retrieve similar products based on a user message.",
           "parameters": {
               "type": "object",
               "properties": {
                   "message": {"type": "string", "description": "The user's message looking for similar products"},
                   "num_results": {"type": "integer", "description": "The number of similar products to return"}
               },
               "required": ["message"]
           }
       }
   }
   ```

4. また、新しい関数の出力を処理するコードも追加する必要があります。 関数呼び出しの出力を処理するコード ブロックに、次の `elif` 条件を追加します。

   ```python
   elif call.function.name == "get_similar_products":
       func_response = await get_similar_products(**json.loads(call.function.arguments))
       messages.append(
           {
               "role": "tool",
               "tool_call_id": call.id,
               "name": call.function.name,
               "content": json.dumps(func_response)
           }
       )
   ```

    完成したブロックは次のようになります。

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
           elif call.function.name == "get_similar_products":
               func_response = await get_similar_products(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

5. 最後に、システム プロンプトの定義を更新して、どのようにベクトル検索を実行するかの指示を与える必要があります。 `system_prompt` の下部に次を挿入します。

   ```plaintext
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   ```

    更新されたシステム プロンプトは次のようになります。

   ```python
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   """
   ```

6. `main.py` ファイルを保存します。

## ベクトル検索機能をテストする

1. Visual Studio Code において、そのアプリに対して開いている統合ターミナル ウィンドウで次のコマンドを実行して、API アプリを再起動します。

   ```bash
   uvicorn main:app
   ```

2. UI は引き続き実行されているはずですが、停止している場合は、統合ターミナル ウィンドウに戻って実行します。

   ```bash
   python -m streamlit run index.py
   ```

3. UI を実行しているブラウザー ウィンドウに戻り、チャット プロンプトで次のように入力します。

   ```bash
   Tell me about the mountain bikes in stock
   ```

    この質問では、検索に一致する製品が数個返されます。

4. 「耐久性のあるペダルを表示してください」、「スタイリッシュなジャージ 5 点のリストを出してください」、「暖かい天候に適した手袋すべての詳細を出してください」など、他の検索をいくつか試してみてください。

    最後の 2 つのクエリでは、以前に適用した 15% の割引と販売価格が製品に含まれていることを確認します。
