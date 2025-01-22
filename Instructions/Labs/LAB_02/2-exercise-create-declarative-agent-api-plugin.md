---
lab:
  title: 演習 1 - プロジェクトをダウンロードしてファイルを調べる
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 演習 1 - プロジェクトをダウンロードしてファイルを調べる

宣言型エージェントをアクションで拡張すると、外部システムに格納されているデータをリアルタイムで取得および更新できます。 API プラグインを使用すると、API を介して外部システムに接続して情報を取得および更新できます。

### 演習の期間

- **推定所要時間**: 10 分

## タスク 1 - スターター プロジェクトをダウンロードする

まず、サンプル プロジェクトをダウンロードします。 Web ブラウザーで以下を行います。

1. [https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript) に移動します。
    1. 手順に従って、コンピューターに[リポジトリのソース コードをダウンロード](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view)します。
    1. ダウンロードした ZIP ファイルの内容をコンピューター上の **Documents フォルダー**に展開します。
    1. Visual Studio Code で  フォルダーを開きます。

サンプル プロジェクトは、宣言型エージェントと Azure Functions で実行されている匿名 API を含む Teams Toolkit プロジェクトです。 宣言型エージェントは、Teams Toolkit を使用して新しく作成された宣言型エージェントと同じです。 API は架空のイタリア料理レストランに属しており、今日のメニューを参照して注文することができます。

## タスク 2 - API 定義を調べる

まず、イタリア料理レストランの API における API 定義を確認します。

Visual Studio Code:

1. **Explorer** ビューで、**appPackage/apiSpecificationFile/ristorante.yml** ファイルを開きます。 このファイルは、イタリア料理レストランの API を記述する OpenAPI 仕様です。
1. **servers.url** プロパティの検索

    ```yaml
    servers:
      - url: http://localhost:7071/api
        description: Il Ristorante API server
    ```

    Azure Functions をローカルで実行するときに、標準 URL と一致するローカル URL を指していることに注意してください。

1. **paths** プロパティを見つけます。このプロパティには、今日のメニューを取得するための **/dishes** と、注文を行う **/orders** の 2 つの操作が含まれています。

    > [!IMPORTANT]
    > 各操作には、API 仕様で操作を一意に識別する **operationId** プロパティが含まれていることに注意してください。 Copilot では、特定のユーザー プロンプトに対して呼び出す API を認識できるように、各操作に一意の ID が必要です。

## タスク 3 - API の実装を調べる

次に、この演習で使用するサンプル API を確認します。

Visual Studio Code:

1. **Explorer** ビューで、**src/data.json** ファイルを開きます。 このファイルには、イタリア料理レストランの架空のメニュー項目が含まれています。 各料理の内容は次のとおりです。

    - 名称、
    - 説明、
    - 画像へのリンク、
    - 価格、
    - どのコースで提供されるか、
    - 種類 (料理または飲み物)、
    - 必要に応じて、アレルギー物質のリスト

    この演習では、API はこのファイルをデータ ソースとして使用します。
1. 次に、**src/functions** フォルダーを展開します。 **dishes.ts** と **placeOrder.ts** という名前の 2 つのファイルに注目してください。 これらのファイルには、API 仕様で定義されている 2 つの操作の実装が含まれています。
1. **src/functions/dishes.ts** ファイルを開きます。 少し時間を取って、API の動作を確認してください。 まず、**src/functions/data.json** ファイルからサンプル データを読み込みます。

    ```typescript
    import data from "../data.json";
    ```

    次に、API を呼び出すクライアントが渡す可能性のあるフィルターについて、さまざまなクエリ文字列パラメーターを調べます。

    ```typescript
    const course = req.query.get('course');
    const allergensString = req.query.get('allergens');
    const allergens: string[] = allergensString ? allergensString.split(",") : [];
    const type = req.query.get('type');
    const name = req.query.get('name');
    ```

    要求で指定されたフィルターに基づいて、API はデータ セットをフィルター処理し、応答を返します。

1. 次に、**src/functions/placeOrder.ts** ファイルで定義された注文を配置するための API を調べます。 API は、サンプル データの参照から始まります。 次に、クライアントが要求本文で送信する注文の形状を定義します。

    ```typescript
    interface OrderedDish {
      name?: string;
      quantity?: number;
    }
    interface Order {
      dishes: OrderedDish[];
    }
    ```

    API は、要求を処理するときに、最初に要求に本文が含まれているかどうか、および形状が正しいるかどうかを確認します。 そうでない場合は、400 Bad Request エラーで要求が拒否されます。

    ```typescript
    let order: Order | undefined;
    try {
      order = await req.json() as Order | undefined;
    }
    catch (error) {
      return {
        status: 400,
        jsonBody: { message: "Invalid JSON format" },
      } as HttpResponseInit;
    }
    if (!order.dishes || !Array.isArray(order.dishes)) {
      return {
        status: 400,
        jsonBody: { message: "Invalid order format" }
      } as HttpResponseInit;
    }
    ```

    次に、API により要求がメニューの料理に解決され、合計価格が計算されます。

    ```typescript
    let totalPrice = 0;
    const orderDetails = order.dishes.map(orderedDish => {
      const dish = data.find(d => d.name.toLowerCase().includes(orderedDish.name.toLowerCase()));
      if (dish) {
        totalPrice += dish.price * orderedDish.quantity;
        return {
          name: dish.name,
          quantity: orderedDish.quantity,
          price: dish.price,
        };
      }
      else {
        context.error(`Invalid dish: ${orderedDish.name}`);
        return null;
      }
    });
    ```

    > [!IMPORTANT]
    > API では、クライアントが ID ではなく名前の一部によって料理を指定することが想定されている点に注目してください。 これは、大規模言語モデルが数値よりも単語でより適切に機能するため、意図的に行われます。 さらに、API を呼び出して注文する前に、Copilot には、ユーザーのプロンプトの一部としてすぐに使用できる料理の名前があります。 Copilot がその ID で料理を参照する必要がある場合は、まずどれが追加の API 要求を必要とし、どの Copilot が今実行できないかを取得する必要があります。

    API の準備ができたら、合計価格、架空の注文 ID、状態を含む応答が返されます。

    ```typescript
    const orderId = Math.floor(Math.random() * 10000);
    return {
      status: 201,
      jsonBody: {
        order_id: orderId,
        status: "confirmed",
        total_price: totalPrice,
      }
    } as HttpResponseInit;
    ```

