---
lab:
  title: 演習 2 - API プラグイン定義をビルドする
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 演習 2 - API プラグイン定義をビルドする

次の手順では、プラグイン定義をプロジェクトに追加します。 プラグイン定義には、次の情報が含まれます。

- プラグインが実行できるアクション。
- 予期され、返されるデータの形状は何か。
- 宣言型エージェントが、基になる API をどのように呼び出す必要があるか。

### 演習の期間

- **推定所要時間**: 10 分

## タスク 1 - 基本的なプラグイン定義構造の追加

Visual Studio Code:

1. **appPackage** フォルダーに、**ai-plugin.json** という名前の新しいファイルを追加します。
1. 次の内容を貼り付けます。

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [ 
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

    このファイルには、人間とモデルの説明を含む API プラグインの基本的な構造が含まれています。 **description_for_model** には、プラグインの呼び出しを検討するタイミングをエージェントが理解するのに役立つ詳細な情報が含まれています。
1. 変更を保存します。

## タスク 2 - 関数の定義

API プラグインは、API 仕様で定義されている API 操作にマップされる 1 つ以上の関数を定義します。 各関数は、ユーザーにデータを表示する方法をエージェントに指示する名前、説明、応答の定義で構成されます。

### メニューを取得する関数を定義する

まず、今日のメニューに関する情報を取得する関数を定義します。

Visual Studio Code:

1. **appPackage/ai-plugin.json** ファイルを開きます。
1. **functions** 配列に、次のスニペットを追加します。

    ```json
    {
      "name": "getDishes",
      "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.dishes",
          "properties": {
            "title": "$.name",
            "subtitle": "$.description"
          }
        }
      }
    }
    ```

    まず、API 仕様から **getDishes** 操作を呼び出す関数を定義します。 次に、関数の説明を指定します。 Copilot は、これを使用してユーザーのプロンプトに対して呼び出す関数を決定するため、この説明は重要です。

    response_semantics プロパティでは、API から受け取ったデータを Copilot で表示する方法を指定します。 API は **dishes** プロパティのメニューの料理に関する情報を返すため、**data_path** プロパティを `$.dishes` JSONPath 式に設定します。

    次に、**properties** セクションで、API 応答からタイトル、説明、URL を表すプロパティをマップします。 この場合、料理には URL がないため、**タイトル**と**説明**のみをマップします。

1. 完成したコード スニペットは次のようになります。

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.dishes",
              "properties": {
                "title": "$.name",
                "subtitle": "$.description"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 変更を保存します。

### 注文する関数を定義する

次に、注文する関数を定義します。

Visual Studio Code:

1. **appPackage/ai-plugin.json** ファイルを開きます。
1. **functions** 配列の末尾に、次のスニペットを追加します。

    ```json
    {
      "name": "placeOrder",
      "description": "Places an order and returns the order details",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.order_id",
            "subtitle": "$.total_price"
          }
        }
      }
    }
    ```

    まず、ID **placeOrder** で API 操作を参照します。 次に、Copilot がこの関数をユーザーのプロンプトと照合するために使用する説明を指定します。 次に、データを返す方法を Copilot に指示します。 注文後に API から返されるデータを次に示します。

    ```json
    {
      "order_id": 6532,
      "status": "confirmed",
      "total_price": 21.97
    }
    ```

    表示するデータは応答オブジェクトのルートに直接配置されるため、JSON オブジェクトの最上位ノードを示す **$** に **data_path** を設定します。 注文の番号とサブタイトルの価格を表示するタイトルを定義します。

1. 完全なファイルは次のようになります。

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          "description": "Places an order and returns the order details",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.order_id",
                "subtitle": "$.total_price"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 変更を保存します。

## タスク 3 - ランタイムの定義

Copilot が呼び出す関数を定義した後、次の手順では、呼び出し方法を指示します。 これは、プラグイン定義の **runtimes** セクションで行います。

Visual Studio Code:

1. **appPackage/ai-plugin.json** ファイルを開きます。
1. ランタイム配列に、次のコードを追加します。

    ```json
    {
      "type": "OpenApi",
      "auth": {
        "type": "None"
      },
      "spec": {
        "url": "apiSpecificationFile/ristorante.yml"
      },
      "run_for_functions": [
        "getDishes",
        "placeOrder"
      ]
    }
    ```

    まず、呼び出す API (**type: OpenApi**) に関する OpenAPI 情報を提供し、匿名 (**auth.type: None**) であることを Copilot に指示します。 次に、**spec** セクションで、プロジェクトにある API 仕様への相対パスを指定します。 最後に、**run_for_functions** プロパティで、この API に属するすべての関数を一覧表示します。

1. 完全なファイルは次のようになります。

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          ...trimmed for brevity
        }
      ],
      "runtimes": [
        {
          "type": "OpenApi",
          "auth": {
            "type": "None"
          },
          "spec": {
            "url": "apiSpecificationFile/ristorante.yml"
          },
          "run_for_functions": [
            "getDishes",
            "placeOrder"
          ]
        }
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 変更を保存します。

