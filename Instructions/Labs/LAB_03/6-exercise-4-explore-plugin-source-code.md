---
lab:
  title: 演習 4 - プラグインのソース コードを調べる
  module: 'LAB 03: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# 演習 4 - プラグインのソース コードを調べる

この演習では、アプリケーション コードを確認して、 **メッセージ拡張機能** のしくみを理解できるようにします。

## タスク 1 - マニフェストを確認する

Microsoft 365 アプリケーションの中核となるのは、そのアプリケーション マニフェストです。 ここでは、Microsoft 365 がアプリケーションにアクセスするために必要な情報を提供します。

**作業ディレクトリで****appPackackage/manifest.json** ファイルを開きます。 この JSON ファイルは、アプリケーション パッケージを作成するための 2 つのアイコン ファイルを含む zip アーカイブに配置されます。 **アイコン** プロパティには、これらのアイコンへのパスが含まれています。

```json
"icons": {
    "color": "Northwind-Logo3-192-${{TEAMSFX_ENV}}.png",
    "outline": "Northwind-Logo3-32.png"
},
```

アイコン名の 1 つにあるトークン`${{TEAMSFX_ENV}}`に 注意してください。 Teams Toolkit は、このトークンを環境名 ( **local** または **dev** (開発中の Azure デプロイの場合) に置き換えます。 したがって、アイコンの色は環境によって変わります。

### アプリケーションの説明

次に、**名前**と**説明**を見てみましょう。 **説明**がかなり長いことに注意してください。 これは、ユーザーとコパイロットの両方が、アプリケーションの機能と使用するタイミングを学習できるようにするために重要です。

```json
    "name": {
        "short": "Northwind Inventory",
        "full": "Northwind Inventory App"
    },
    "description": {
        "short": "App allows you to find and update product inventory information",
        "full": "Northwind Inventory is the ultimate tool for managing your product inventory. With its intuitive interface and powerful features, you'll be able to easily find your products by name, category, inventory status, and supplier city. You can also update inventory information with the app. \n\n **Why Choose Northwind Inventory:** \n\n Northwind Inventory is the perfect solution for businesses of all sizes that need to keep track of their inventory. Whether you're a small business owner or a large corporation, Northwind Inventory can help you stay on top of your inventory management needs. \n\n **Features and Benefits:** \n\n - Easy Product Search through Microsoft Copilot. Simply start by saying, 'Find northwind dairy products that are low on stock' \r - Real-Time Inventory Updates: Keep track of inventory levels in real-time and update them as needed \r  - User-Friendly Interface: Northwind Inventory's intuitive interface makes it easy to navigate and use \n\n **Availability:** \n\n To use Northwind Inventory, you'll need an active Microsoft 365 account . Ensure that your administrator enables the app for your Microsoft 365 account."
    },
```

### ボットの定義

少し下にスクロールして **composeExtensions**を表示します。 Compose 拡張機能は、メッセージ拡張機能の履歴用語です。これは、アプリのメッセージ拡張機能が定義されている場所です。 メッセージ拡張機能は、Azure Bot Framework を使用して通信します。これにより、Microsoft 365 とアプリケーション間の高速で安全な通信チャネルが提供されます。 最初にプロジェクトを実行したとき、Teams Toolkit によってボットが登録され、その **botID** がここに配置されます。

```json
    "composeExtensions": [
        {
            "botId": "${{BOT_ID}}",
            "commands": [
                {
                    ...
```

### コマンド定義

このメッセージ拡張には、 `commands`配列で定義されている 2 つのコマンドがあります。 前の演習を完了した場合は、会社名で検索する 3 番目のコマンドもあります。 最初のコマンドは最も複雑なものなので、とりあえずスキップしましょう。 次のコマンドを使用すると、コパイロット (またはユーザー) は Northwind カテゴリ内の割引製品を検索できます。 このコマンドは、単一のパラメーターである **categoryName** を受け入れます。

```json
{
    "id": "discountSearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search for discounted products by category",
    "title": "Discounts",
    "type": "query",
    "parameters": [
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category to find discounted products",
            "inputType": "text"
        }
    ]
},
```

次に、5 つのパラメーターを持つ最初のコマンドである **inventorySearch** に戻り、より高度なクエリを実行できるようにします。

```json
{
    "id": "inventorySearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search products by name, category, inventory status, supplier location, stock level",
    "title": "Product inventory",
    "type": "query",
    "parameters": [
        {
            "name": "productName",
            "title": "Product name",
            "description": "Enter a product name here",
            "inputType": "text"
        },
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category of the product",
            "inputType": "text"
        },
        {
            "name": "inventoryStatus",
            "title": "Inventory status",
            "description": "Enter what status of the product inventory. Possible values are 'in stock', 'low stock', 'on order', or 'out of stock'",
            "inputType": "text"
        },
        {
            "name": "supplierCity",
            "title": "Supplier city",
            "description": "Enter the supplier city of product",
            "inputType": "text"
        },
        {
            "name": "stockQuery",
            "title": "Stock level",
            "description": "Enter a range of integers such as 0-42 or 100- (for >100 items). Only use if you need an exact numeric range.",
            "inputType": "text"
        }
    ]
},
```

コパイロットは説明に基づいてこれらを入力することができ、メッセージ拡張機能は、すべての非空白パラメーターでフィルター処理された製品の一覧を返します。

## タスク 2 - ボット コードを確認する

次に、  **src/searchApp.ts** ファイルを開きます。このファイルには、 [Bot Builder SDK](https://learn.microsoft.com/azure/bot-service/index-bf-sdk) を使用して Azure Bot Framework と通信するボットのコードが含まれています。 ボットによって SDK クラスの **TeamsActivityHandler** が拡張されていることに注意してください。

```typescript
export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  ...
```

### メッセージ拡張機能クエリ

アプリケーションは、**TeamsActivityHandler** のメソッドをオーバーライドすることで、Microsoft 365 からのメッセージ (**activities** と呼ばれます) を処理できます。

その 1 つ目は、**メッセージ拡張機能クエリ** アクティビティです。 この関数は、ユーザーがメッセージ拡張機能に入力したとき、またはコパイロットによって呼び出されたときに呼び出されます。 ハンドラーは、 **commandID** に基づいてクエリをディスパッチしています。 これらは、アプリ マニフェストで使用されるのと同じ commandID です。

```typescript
  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }
  }
  ...
```

### アダプティブ カード アクション

アプリで処理する必要があるその他の種類のアクティビティは、ユーザーがアダプティブ カードで**在庫の更新**または**並べ替え**を選択した場合などのアダプティブ カードアクションです。 アダプティブ カード アクションには特定のメソッドがないため、コードは `onInvokeActivity()` をオーバーライドします。これは、メッセージ拡張機能クエリを含む、はるかに広範なアクティビティ クラスです。 そのため、コードはアクティビティ名を手動でチェックし、適切なハンドラーにディスパッチします。 アクティビティ名がアダプティブ カード アクション用でない場合、`else` 句は `onInvokeActivity()` の基本実装を実行します。特に、**呼び出し**アクティビティがクエリである場合、`handleTeamsMessagingExtensionQuery()` メソッドが呼び出されます。

```typescript
import {
  TeamsActivityHandler,
  TurnContext,
  MessagingExtensionQuery,
  MessagingExtensionResponse,
  InvokeResponse
} from "botbuilder";
import productSearchCommand from "./messageExtensions/productSearchCommand";
import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
import revenueSearchCommand from "./messageExtensions/revenueSearchCommand";
import actionHandler from "./adaptiveCards/cardHandler";

export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }

  }

  // Handle adaptive card actions
  public async onInvokeActivity(context: TurnContext): Promise<InvokeResponse> {
    let runEvents = true;
    // console.log (`🎬 Invoke activity received: ${context.activity.name}`);
    try {
      if(context.activity.name==='adaptiveCard/action'){
        switch (context.activity.value.action.verb) {
          case 'ok': {
            return actionHandler.handleTeamsCardActionUpdateStock(context);
          }
          case 'restock': {
            return actionHandler.handleTeamsCardActionRestock(context);
          }
          case 'cancel': {
            return actionHandler.handleTeamsCardActionCancelRestock(context);
          }
          default:
            runEvents = false;
            return super.onInvokeActivity(context);
        }
      } else {
          runEvents = false;
          return super.onInvokeActivity(context);
      }
    } 
```

## タスク 3 - メッセージ拡張機能コマンド コードを確認する

コードのモジュール性、読みやすさ、再利用性を高めるために、各メッセージ拡張機能コマンドは独自の TypeScript モジュールに配置されます。 例については、 **src/messageExtensions/discountSearchCommand.ts** を参照してください。

まず、モジュールが定数 `COMMAND_ID`をエクスポートすることを確認します。これには、アプリ マニフェストに含まれているのと同じ **commandID** が含まれており、 **searchApp.ts** の switch ステートメントが正常に動作することを確認します。

次に、**カテゴリ別の割引製品** の受信クエリを処理する関数 `handleTeamsMessagingExtensionQuery()` を提供します。

```typescript
async function handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
): Promise<MessagingExtensionResponse> {

    // Seek the parameter by name, don't assume it's in element 0 of the array
    let categoryName = cleanupParam(query.parameters.find((element) => element.name === "categoryName")?.value);
    console.log(`💰 Discount query #${++queryCount}: Discounted products with categoryName=${categoryName}`);

    const products = await getDiscountedProductsByCategory(categoryName);

    console.log(`Found ${products.length} products in the Northwind database`)
    const attachments = [];
    products.forEach((product) => {
        const preview = CardFactory.heroCard(product.ProductName,
            `Avg discount ${product.AverageDiscount}%<br />Supplied by ${product.SupplierName} of ${product.SupplierCity}`,
            [product.ImageUrl]);

        const resultCard = cardHandler.getEditCard(product);
        const attachment = { ...resultCard, preview };
        attachments.push(attachment);
    });
    return {
        composeExtension: {
            type: "result",
            attachmentLayout: "list",
            attachments: attachments,
        },
    };
}
```

`query.parameters` 配列内のインデックスが、マニフェスト内のパラメーターの位置に対応していない可能性があることに注意してください。 これは通常、マルチパラメータ コマンドの問題にすぎませんが、コードはインデックスをハード コーディングするのではなく、パラメータ名に基づいて値を取得します。

パラメータをクリーンアップした後 (パラメータをトリミングし、Copilot が「**\***」がすべてと一致するワイルドカードであると見なす場合があるという事実を処理) した後、コードは Northwind データ アクセス層を `getDiscountedProductsByCategory()` に呼び出します。

次に、製品を反復処理し、それぞれに 2 つのカードを作成します。

- **hero_ カードとして実装される_preview** カード これは、ユーザー インターフェイスの検索結果と、Copilot の一部の引用で表示されます。

- _result_ カード。すべての詳細を含む **adaptive** カードとして実装されます。

次のタスクでは、アダプティブ カード コードを確認し、アダプティブ カード デザイナーをチェックアウトします。

## タスク 4 - アダプティブ カードと関連コードを調べる

プロジェクトのアダプティブ カードは、 **src/adaptiveCards/** フォルダーにあります。 3 枚のカードがあり、それぞれが JSON ファイルとして実装されています。

- **editCard.json** - これは、メッセージ拡張機能または Copilot 参照によって表示される初期カードです。

- **successCard.json** - ユーザーがアクションを実行すると、成功を示すカードが表示されます。 編集カードとほとんど同じですが、ユーザーへのメッセージが含まれている点が異なります。

- **errorCard.json** - アクションが失敗した場合、このカードが表示されます。

**アダプティブ カード デザイナーで編集カードを見てみましょう**。 Web ブラウザーで [https://adaptivecards.io](https://adaptivecards.io) を開いて、上部にある **[デザイナー]** オプションを選択します。

![アダプティブ カード デザイナーのスクリーンショット。](../media/5-01-adaptive-card-designer-01.png)

`"text": "📦 ${productName}",` などのデータ バインディング式に注目してください。 これにより、データの `productName` プロパティがカード上のテキストにバインドされます。

次に、ホスト アプリケーションとして **Microsoft Teams** を選択します 1️⃣。 **editCard.json** の内容全体をカード ペイロード エディター 2️⃣ に貼り付け、**sampleData.json** の内容をサンプル データ エディター 3️⃣ に貼り付けます。 サンプル データは、コードで提供されている製品と同じです。 デザイナーがアダプティブ カード形式のいずれかを表示できないために発生する小さなエラーを除き、カードはレンダリング済みとして表示されます。

![json に基づいて Copilot でレンダリングされたアダプティブ カードのスクリーンショット。](../media/5-01-adaptive-card-designer-02.png)

ページの上部で、テーマまたはモバイル デバイスでカードがどのように表示されるかを確認するには、**テーマ**と**エミュレートされたデバイス**を変更してみてください。 これは、サンプル アプリケーションのアダプティブ カードを作成するために使用されたツールです。

Visual Studio Code に戻り、**cardHandler.ts** を開きます。 関数 `getEditCard()` は、各メッセージ拡張機能コマンドから呼び出され、**result** カードを取得します。 このコードは、アダプティブ カード JSON (テンプレートと見なされます) を読み取り、それを製品データにバインドします。 結果はより多くの JSON になります。テンプレートと同じカードで、データ バインディング式がすべて入力されます。 最後に、`CardFactory` モジュールを使用して、最終的な JSON をレンダリング用のアダプティブ カード オブジェクトに変換します。

```typescript
function getEditCard(product: ProductEx): any {

    var template = new ACData.Template(editCard);
    var card = template.expand({
        $root: {
            productName: product.ProductName,
            unitsInStock: product.UnitsInStock,
            productId: product.ProductID,
            categoryId: product.CategoryID,
            imageUrl: product.ImageUrl,
            supplierName: product.SupplierName,
            supplierCity: product.SupplierCity,
            categoryName: product.CategoryName,
            inventoryStatus: product.InventoryStatus,
            unitPrice: product.UnitPrice,
            quantityPerUnit: product.QuantityPerUnit,
            unitsOnOrder: product.UnitsOnOrder,
            reorderLevel: product.ReorderLevel,
            unitSales: product.UnitSales,
            inventoryValue: product.InventoryValue,
            revenue: product.Revenue,
            averageDiscount: product.AverageDiscount
        }
    });
    return CardFactory.adaptiveCard(card);
}
```

下にスクロールすると、カード上の各アクション ボタンのハンドラーが表示されます。 カードは、アクション ボタンがクリックされたときにデータを送信します。具体的には、カードの**数量**入力ボックスである `data.txtStock`、および、各カード アクションで送信されて更新する製品をコードに知らせる `data.productId` が使用されます。

```typescript
async function handleTeamsCardActionUpdateStock(context: TurnContext) {

    const request = context.activity.value;
    const data = request.action.data;
    console.log(`🎬 Handling update stock action, quantity=${data.txtStock}`);

    if (data.txtStock && data.productId) {

        const product = await getProductEx(data.productId);
        product.UnitsInStock = Number(data.txtStock);
        await updateProduct(product);

        var template = new ACData.Template(successCard);
        var card = template.expand({
            $root: {
                productName: product.ProductName,
                unitsInStock: product.UnitsInStock,
                productId: product.ProductID,
                categoryId: product.CategoryID,
                imageUrl: product.ImageUrl,
                ...
```

ご覧のように、コードはこれら 2 つの値を取得し、データベースを更新してから、メッセージと更新されたデータを含む新しいカードを送信します。

## 結論

演習 5 と  Copilot for Microsoft 365 メッセージ拡張機能プラグイン課題を完了しました。 これらの演習を実行していただきありがとうございました。

[課題の概要に進みます...](./7-summary.md)
