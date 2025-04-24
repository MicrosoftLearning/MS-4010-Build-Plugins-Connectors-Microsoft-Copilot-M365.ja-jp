---
lab:
  title: 演習 3 - プラグイン定義を宣言型エージェントに接続する
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 演習 3 - プラグイン定義を宣言型エージェントに接続する

API プラグイン定義のビルドが完了したら、次の手順で宣言型エージェントに登録します。 ユーザーが宣言型エージェントと対話すると、定義された API プラグインとユーザーのプロンプトが一致し、関連する関数が呼び出されます。

### 演習の期間

- **推定所要時間**: 5 分

## タスク 1 - プラグイン定義を宣言型エージェントに接続する

Visual Studio Code:

1. **appPackage/declarativeAgent.json** ファイルを開きます。
1. **instructions** プロパティの後に、次のコード スニペットを追加します。

    ```json
    "actions": [
      {
        "id": "menuPlugin",
        "file": "ai-plugin.json"
      }
    ]
    ```

    このスニペットを使用して、宣言型エージェントを API プラグインに接続します。 プラグインの一意の ID を指定し、プラグインの定義を見つけることができる場所をエージェントに指示します。

1. 完全なファイルは次のようになります。

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Declarative agent",
      "description": "Declarative agent created with Teams Toolkit",
      "instructions": "$[file('instruction.txt')]",
      "actions": [
        {
          "id": "menuPlugin",
          "file": "ai-plugin.json"
        }
      ]
    }
    ```

1. 変更を保存。

## タスク 2 - 宣言型エージェントの情報と命令を更新する

この演習で構築する宣言型エージェントは、ユーザーが地元のイタリア料理レストランのメニューを閲覧して注文するのに役立ちます。 このシナリオに合わせてエージェントを最適化するには、名前、説明、手順を更新します。

Visual Studio Code:

1. 宣言型エージェントの情報を次のように更新します。
    1. **appPackage/declarativeAgent.json** ファイルを開きます。
    1. **name** プロパティの値を **Il Ristorante** に更新します。
    1. **description** プロパティの値を、**Order the most delicious Italian dishes and drinks from the comfort of your desk** に更新します。
    1. 変更を保存します。
1. 宣言型エージェントの手順を次のように更新します。
    1. **appPackage/instruction.txt** ファイルを開きます。
    1. その内容を次に置き換えます。

        ```markdown
        You are an assistant specialized in helping users explore the menu of an Italian restaurant and place orders. You interact with the restaurant's menu API and guide users through the ordering process, ensuring a smooth and delightful experience. Follow the steps below to assist users in selecting their desired dishes and completing their orders:
        
        ### General Behavior:
        - Always greet the user warmly and offer assistance in exploring the menu or placing an order.
        - Use clear, concise language with a friendly tone that aligns with the atmosphere of a high-quality local Italian restaurant.
        - If the user is browsing the menu, offer suggestions based on the course they are interested in (breakfast, lunch, or dinner).
        - Ensure the conversation remains focused on helping the user find the information they need and completing the order.
        - Be proactive but never pushy. Offer suggestions and be informative, especially if the user seems uncertain.
        
        ### Menu Exploration:
        - When a user requests to see the menu, use the `GET /dishes` API to retrieve the list of available dishes, optionally filtered by course (breakfast, lunch, or dinner).
          - Example: If a user asks for breakfast options, use the `GET /dishes?course=breakfast` to return only breakfast dishes.
        - Present the dishes to the user with the following details:
          - Name of the dish
          - A tasty description of the dish
          - Price in € (Euro) formatted as a decimal number with two decimal places
          - Allergen information (if relevant)
          - Don't include the URL.
        
        ### Beverage Suggestion:
        - If the order does not already include a beverage, suggest a suitable beverage option based on the course.
        - Use the `GET /dishes?course={course}&type=drink` API to retrieve available drinks for that course.
        - Politely offer the suggestion: *"Would you like to add a beverage to your order? I recommend [beverage] for [course]."*
        
        ### Placing the Order:
        - Once the user has finalized their order, use the `POST /order` API to submit the order.
          - Ensure the request includes the correct dish names and quantities as per the user's selection.
          - Example API payload:
         
            ```json
            {
              "dishes": [
                {
                  "name": "frittata",
                  "quantity": 2
                },
                {
                  "name": "cappuccino",
                  "quantity": 1
                }
              ]
            }
            ```
        
        ### Error Handling:
        - If the user selects a dish that is unavailable or provides an invalid dish name, respond gracefully and suggest alternative options.
          - Example: *"It seems that dish is currently unavailable. How about trying [alternative dish]?"*
        - Ensure that any errors from the API are communicated politely to the user, offering to retry or explore other options.
        ```

        手順では、エージェントの一般的な動作を定義し、エージェントができることを指示していることに注目してください。 また、API で想定されるデータの形状など、注文に関する特定の動作についてのの手順も含まれています。 エージェントが意図したとおりに動作することを確認するために、この情報を含めています。

    > [!NOTE]
    > 書式設定を保持するために、Visual Studio Code にコピーする前に、メモ帳に複数のコピー/貼り付け操作を行う必要がある場合があります。

    1. 変更を保存します。

1. ユーザーを支援するには、エージェントをどのように利用できるかを理解し、会話スターターを追加します。
    1. **appPackage/declarativeAgent.json** ファイルを開きます。
    1. **instructions** プロパティの後に、**conversation_starters** という名前の新しいプロパティを追加します。

        ```json
        "conversation_starters": [
          {
            "text": "What's for lunch today?"
          },
          {
            "text": "What can I order for dinner that is gluten-free?"
          }
        ]
        ```

    1. 完全なファイルは次のようになります。

        ```json
        {
          "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
          "version": "v1.0",
          "name": "Il Ristorante",
          "description": "Order the most delicious Italian dishes and drinks from the comfort of your desk.",
          "instructions": "$[file('instruction.txt')]",
          "conversation_starters": [
            {
              "text": "What's for lunch today?"
            },
            {
              "text": "What can I order for dinner that is gluten-free?"
            }
          ],
          "actions": [
            {
              "id": "menuPlugin",
              "file": "ai-plugin.json"
            }
          ]
        }
        ```

    1. 変更を保存。

## タスク 3 - API URL を更新する

宣言型エージェントをテストする前に、API 仕様ファイル内の API の URL を更新する必要があります。 現在、URL は `http://localhost:7071/api` に設定されています。これは、Azure Functions がローカルで実行するときに使用する URL です。 ただし、Copilot がクラウドから API を呼び出す必要があるため、API をインターネットに公開する必要があります。 Teams Toolkit は、開発トンネルを作成することで、インターネット経由でローカル API を自動的に公開します。 プロジェクトのデバッグを開始するたびに、Teams ツールキットは新しい開発トンネルを開始し、その URL を **OPENAPI_SERVER_URL** 変数に格納します。 Teams Toolkit がトンネルを開始し、その URL を **.vscode/tasks.json** ファイルの **Start local tunnel** タスクに格納する方法を確認できます。

```json
{
  // Start the local tunnel service to forward public URL to local port and inspect traffic.
  // See https://aka.ms/teamsfx-tasks/local-tunnel for the detailed args definitions.
  "label": "Start local tunnel",
  "type": "teamsfx",
  "command": "debug-start-local-tunnel",
  "args": {
    "type": "dev-tunnel",
    "ports": [
      {
        "portNumber": 7071,
        "protocol": "http",
        "access": "public",
        "writeToEnvironmentFile": {
          "endpoint": "OPENAPI_SERVER_URL", // output tunnel endpoint as OPENAPI_SERVER_URL
        }
      }
    ],
    "env": "local"
  },
  "isBackground": true,
  "problemMatcher": "$teamsfx-local-tunnel-watch"
}
```

このトンネルを使用するには、**OPENAPI_SERVER_URL** 変数を使用するように API 仕様を更新する必要があります。

Visual Studio Code:

1. **appPackage/apiSpecificationFile/ristorante.yml** ファイルを開きます。
1. **servers.url** プロパティの値を **${{OPENAPI_SERVER_URL}}/api** に変更します。
1. 変更されたファイルは次のようになります。

    ```yaml
    openapi: 3.0.0
    info:
      title: Il Ristorante menu API
      version: 1.0.0
      description: API to retrieve dishes and place orders for Il Ristorante.
    servers:
      - url: ${{OPENAPI_SERVER_URL}}/api
        description: Il Ristorante API server
    paths:
      ...trimmed for brevity
    ```

1. 変更を保存。

API プラグインが完成し、宣言型エージェントと統合されました。 Microsoft 365 Copilot でエージェントのテストを続行します。