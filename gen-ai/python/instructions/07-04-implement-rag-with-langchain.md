---
title: '07.4: LangChain と Azure Cosmos DB for NoSQL ベクトル検索を使用して RAG を実装する'
lab:
  title: '07.4: LangChain と Azure Cosmos DB for NoSQL ベクトル検索を使用して RAG を実装する'
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 13
parent: Python SDK labs
---

# LangChain と Azure Cosmos DB for NoSQL ベクトル検索を使用して RAG を実装する

LangChain のオーケストレーション機能は、Azure OpenAI クライアントを直接使用してコパイロットの LLM 統合を実装するよりも多くの利点をもたらします。 LangChain を使用すると、Azure Cosmos DB を含むさまざまなデータ ソースとのよりシームレスな統合が可能になり、取得プロセスが強化された効率的なベクトル検索が実現します。 LangChainは、ワークフローを管理および最適化するための信頼性の高いツールを提供し、再利用可能なモジュール式コンポーネントを使用して複雑なアプリケーションが簡単にビルドできるようになります。 この柔軟性により、開発が簡素化されるだけでなく、スケーラビリティと保守容易性も確保されます。

このラボでは、API の `/chat` エンドポイントを Azure OpenAI クライアントの使用から LangChain の強力なオーケストレーション機能の活用に移行することで、コパイロットを強化します。 このシフトにより、ベクトル検索機能を Azure Cosmos DB for NoSQL と統合することで、より効率的なデータ取得とパフォーマンスの向上が可能になります。 アプリの情報取得プロセスを最適化する場合でも、RAG の可能性を単に調べる場合でも、このモジュールではシームレスな変換について説明し、LangChain がどのようにアプリの機能を効率化して向上させるのかを示します。 LangChain と Azure Cosmos DB を使用して新しい効率とインサイトを引き出すこの取り組みを開始しましょう。

> &#128721; このモジュールにおける前の演習は、このラボの前提条件です。 これらの演習のいずれかを完了する必要がある場合は、このラボに必要なインフラストラクチャとスタート コードが提供されるため、次に進む前に完了してください。

## LangChain ライブラリをインストールする

1. Visual Studio Code を使用して、「**Azure Cosmos DB を使用してコパイロットを構築する**」学習モジュールのためにクローンしたラボ コード リポジトリのクローン先フォルダー開きます。

2. Visual Studio Code の **[エクスプローラー]** ペインで、**python/07-build-copilot** フォルダーを参照し、その中にある `requirements.txt` ファイルを開きます。

3. `requirements.txt` ファイルを更新して、必要な LangChain ライブラリが含まれるようにします。

   ```ini
   langchain==0.3.9
   langchain-openai==0.2.11
   ```

4. Visual Studio Code で新しい統合ターミナル ウィンドウを起動し、ディレクトリを `python/07-build-copilot` に変更します。

5. 次の表にある OS とシェルに対する適切なコマンドを使用して、統合ターミナル ウィンドウをアクティブ化することで、統合ターミナル ウィンドウが Python 仮想環境内で実行されるようにします。

    | プラットフォーム | シェル | 仮想環境をアクティブ化するコマンド |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

6. 統合ターミナル プロンプトで次のコマンドを実行して、LangChain ライブラリにより仮想環境を更新します。

   ```bash
   pip install -r requirements.txt
   ```

7. 統合ターミナルを閉じます。

## バックエンド API を更新する

前のラボでは、Azure OpenAI クライアントと Azure Cosmos DB のデータを使用して RAG パターンを実行しました。 ここでは、同じアクションを実行するツールで LangChain エージェントを使用するようにバックエンド API を更新します。

LangChain を使用して、Azure OpenAI サービスにデプロイされた言語モデルと対話することは、コードの観点からはやや単純です。

1. `main.py` ファイルの先頭にある `from openai import AzureOpenAI` import ステートメントを削除します。 Azure OpenAI とのすべての対話は LangChain で提供されるクラスを経由することになるので、そのクライアント ライブラリは不要になります。

2. `main.py` ファイルの先頭にある次の import ステートメントは不要になるため、削除します。

   ```python
   from openai import AsyncAzureOpenAI
   import json
   ```

### 埋め込みエンドポイントを更新する

1. `main.py` ファイルの先頭に次の import ステートメントを追加して、`langchain_openai` ライブラリから `AzureOpenAIEmbeddings` クラスをインポートします。

   ```python
   from langchain_openai import AzureOpenAIEmbeddings
   ```

2. ファイル内の `generate_embeddings` メソッドを見つけて、次のように上書きします。このメソッドは、`AzureOpenAIEmbeddings` クラスを使用して Azure OpenAI との対話を処理します。

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Use LangChain's Azure OpenAI Embeddings class
       azure_openai_embeddings = AzureOpenAIEmbeddings(
           azure_deployment = EMBEDDING_DEPLOYMENT_NAME,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
       return await azure_openai_embeddings.aembed_query(text)
   ```

    `AzureOpenAIEmbeddings` クラスは、Azure OpenAI Embeddings API と対話するためのインターフェイスを提供し、生成されたベクトルのみを含む簡略化された応答オブジェクトを返します。

### チャット エンドポイントを更新する

1. `lanchain_openai` import ステートメントを更新して、`AzureChatOpenAI` クラスを追加します。

   ```python
   from langchain_openai import AzureOpenAIEmbeddings, AzureChatOpenAI
   ```

1. 変更後の `/chat` エンドポイントをビルドするときに使用される次の追加 LangChain オブジェクトをインポートします。

   ```python
   from langchain.agents import AgentExecutor, create_openai_functions_agent
   from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
   from langchain_core.tools import StructuredTool
   ```

1. チャット履歴は、LangChain エージェントを使用して異なる方法でコパイロットの会話に挿入されるため、`system_prompt` 定義の直後にあるコード行を削除します。 削除する必要がある行は次のとおりです。

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{ "role": "system", "content": system_prompt }]

   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)

   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

1. 今削除したコードの代わりに、LangChain の `ChatPromptTemplate` クラスを使用して `prompt` オブジェクトを定義します。

   ```python
   prompt = ChatPromptTemplate.from_messages(
       [
           ("system", system_prompt),
           MessagesPlaceholder("chat_history", optional=True),
           ("user", "{input}"),
           MessagesPlaceholder("agent_scratchpad")
       ]
   )
   ```

    `ChatPromptTemplate` は、特定の順序で複数のコンポーネントを使用して作成されています。 これらの要素がどのような関係になるかを示します。

    - **システム メッセージ**: `system_prompt` を使用してペルソナを Copilot に提供し、アシスタントの動作とユーザーとの対話方法に関する手順を提供します。
    - **チャット履歴**: 会話内の過去のメッセージのリストを含む `chat_history` を、LLM が動作しているコンテキストに組み込むことができるようにします。
    - **ユーザー入力**: 現在のユーザー メッセージ。
    - **エージェント スクラッチパッド**: エージェントによって実行される中間メモまたは手順を許可します。

    結果のプロンプトは、対話的 AI エージェントの構造化された入力を提供し、特定のコンテキストに基づいて応答を生成するのに役立ちます。

1. 次に、`tools` 配列定義を次のように置き換えます。ここでは、LangChain の `StructuredTool` クラスを使用して関数定義を適切な形式に抽出します。

   ```python
   tools = [
       StructuredTool.from_function(coroutine=apply_discount),
       StructuredTool.from_function(coroutine=get_category_names),
       StructuredTool.from_function(coroutine=get_similar_products)
   ]
   ```

    LangChain の `StructuredTool.from_function` メソッドは、入力パラメーターと関数の docstring の説明を使用して、特定の関数からツールを作成します。 非同期メソッドで使用するには、`coroutine` 入力パラメーターに関数名を渡します。

    Python では、docstring (ドキュメント文字列の省略形) は、関数、メソッド、クラス、またはモジュールをドキュメント化するために使用される特殊な型の文字列です。 これは、ドキュメントを Python コードに関連付ける便利な方法を提供し、通常は三重引用符 (""" または "'') で囲まれます。 docstring は、ドキュメント化する関数 (またはメソッド、クラス、またはモジュール) の定義の直後に配置されます。

    この関数を使用すると、Azure OpenAI クライアントを使用して手動で作成する必要があった JSON 関数定義の作成が自動化され、関数呼び出しのプロセスが簡略化されます。

1. 上記で完了した `tools` 配列定義と関数の末尾にある `return` ステートメントの間のすべてのコードを削除します。 Azure OpenAI クライアントを使用して、言語モデルの呼び出しを 2 回実行する必要がありました。 1 回目は、どの関数呼び出し (ある場合) かを判断できるようにするためにプロンプトを拡張するために必要で、2 回目では RAG 完了を要求する必要があります。 その間に、コードを使用して最初の呼び出しからの応答を調べて関数呼び出しが必要かどうかを判断し、それらの関数の呼び出しを "処理" するコードを記述する必要がありました。 その後、LLM に送信されるメッセージにこれらの関数呼び出しの出力を挿入する必要がありました。そのため、完了応答を作成する際の理由に応じて、エンリッチされたプロンプトが表示される可能性がありました。 LangChain は、次に示すように、RAG パターンを使用して LLM を呼び出すプロセスを大幅に簡略化します。 削除する必要があるコードは次のとおりです。

   ```python
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

   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )

   return final_response.choices[0].message.content
   ```

1. `tools` 配列定義のすぐ下から作業し、LangChain の `AzureChatOpenAI` クラスを使用して Azure OpenAI API への参照を作成します。

   ```python
   # Connect to Azure OpenAI API
   azure_openai = AzureChatOpenAI(
       azure_deployment=COMPLETION_DEPLOYMENT_NAME,
       azure_endpoint=AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
       api_version=AZURE_OPENAI_API_VERSION
   )
   ```

1. LangChain エージェントが、定義した関数で操作できるようにするには、`create_openai_functions_agent` メソッドを使用してエージェントを作成します。これには、`AzureChatOpenAI` objedt、`tools` 配列、および `ChatPromptTemplate` オブジェクトを指定します。

   ```python
   agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
   ```

    LangChain の `create_openai_functions_agent` 関数は、外部関数を呼び出して、指定された言語モデルとツールを使用してタスクを実行できるエージェントを作成します。 これにより、さまざまなサービスと機能をエージェントのワークフローに統合することが可能になり、柔軟性と強化された機能が提供されます。

1. LangChain では、`AgentExecutor` クラスを使用して、`create_openai_functions_agent` メソッドで作成したエージェントなど、エージェントの実行フローを管理します。 入力の処理、ツールまたはモデルの呼び出し、出力の操作を処理します。 次のコードを使用して、エージェントのエージェント Executor を作成します。

   ```python
   agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
   ```

    `AgentExecutor` により、応答の生成に必要なすべての手順が正しい順序で実行されます。 エージェントの実行の複雑さを抽象化し、機能と構造の追加レイヤーを提供し、高度なエージェントの構築、管理、スケーリングを容易にします。

1. エージェント Executor の `invoke` メソッドを使用して、着信ユーザー メッセージを LLM に送信します。 また、チャット履歴も含めます。 次のコードを `agent_executor` 定義の下に挿入します。

   ```python
   completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
   ```

   `input` トークンと `chat_history` トークンは、`ChatPromptTemplate` を使用して作成されたプロンプト オブジェクトで定義されました。 `invoke` メソッドを使用すると、これらはプロンプトに挿入され、LLM が応答の作成時にその情報を使用できるようになります。

1. 最後に、return ステートメントを更新して、エージェントの完了オブジェクトの `output` を使用します。

   ```python
   return completion["output"]
   ```

1. `main.py` ファイルを保存します。 更新した `/chat` エンドポイント関数は次のようになります。

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
       When asked to provide a list of products, you should:
           - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
       """
       prompt = ChatPromptTemplate.from_messages(
           [
               ("system", system_prompt),
               MessagesPlaceholder("chat_history", optional=True),
               ("user", "{input}"),
               MessagesPlaceholder("agent_scratchpad")
           ]
       )
    
       # Define function calling tools
       tools = [
           StructuredTool.from_function(apply_discount),
           StructuredTool.from_function(get_category_names),
           StructuredTool.from_function(get_similar_products)
       ]
    
       # Connect to Azure OpenAI API
       azure_openai = AzureChatOpenAI(
           azure_deployment=COMPLETION_DEPLOYMENT_NAME,
           azure_endpoint=AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
           api_version=AZURE_OPENAI_API_VERSION
       )
    
       agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
       agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
        
       completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
            
       return completion["output"]
   ```

## API アプリと UI アプリを起動する

1. API を起動するには、Visual Studio Code で新しい統合ターミナル ウィンドウを開きます。

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

1. 新しい統合ターミナル ウィンドウを開き、ディレクトリを `python/07-build-copilot` に変更して Python 環境をアクティブ化した後、ディレクトリを `ui` フォルダーに変更し、次を実行して UI アプリを起動します。

   ```bash
   python -m streamlit run index.py
   ```

1. UI がブラウザー ウィンドウで自動的に開かない場合は、新しいブラウザー タブまたはウィンドウを起動し、<http://localhost:8501> にアクセスして UI を開きます。

## コパイロットをテストする

1. UI にメッセージを送信する前に、Visual Studio Code に戻り、API アプリと結び付いている統合ターミナル ウィンドウを選択します。 このウィンドウ内には、LangChain エージェント エグゼキューターによって生成された "詳細な" 出力が表示されます。ここでは、送信した要求を LangChain がどのように処理しているかについての分析情報が提供されます。 以下の要求で送信し、各呼び出しの後に再度チェックインするときに、このウィンドウの出力に注意してください。

1. UI のチャット プロンプトで、「割引を適用してください」と入力し、メッセージを送信します。

    適用する割引率と製品カテゴリを尋ねる返信を受け取るはずです。

1. 「手袋」と返信します。

    "手袋" カテゴリに適用する割引率を尋ねる応答が表示されます。

1. 「25%」というメッセージを送信します。

    "25% の割引が "手袋" カテゴリのすべての製品に正常に適用されました" という応答が表示されるはずです。

1. コパイロットに "すべての手袋を見せる" ように依頼します。

    返信で、データベース内のすべての手袋の一覧が表示されるはずです。これには 25% の割引価格が含まれます。

1. 最後に、「寒い天候で自転車に乗るのに一番おすすめの手袋は何ですか?」と尋ねて、 ベクトル検索を実行します。 これには、`get_similar_items` メソッドの関数呼び出しが含まれ、更新した両方の `generate_embeddings` メソッドを呼び出して LangChain 実装と `vector_search` 関数を使用します。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。
