---
lab:
  title: 演習 3 - 新しいコマンドを追加する
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# 演習 3 - 新しいコマンドを追加する

この演習では、新しいコマンドを追加して、Teams メッセージ拡張機能と Copilot プラグインを強化します。 現在のメッセージ拡張機能は、Northwind Inventory データベース内の製品に関する情報を効果的に提供しますが、Northwind の顧客に関連する情報は提供しません。 ユーザーが指定した顧客名で並べ替えられた製品を取得する API 呼び出しに関連付けられた新しいコマンドを導入します。 この演習では、少なくとも演習 1、2、3 が完了していることを前提としています。 Microsoft 365 Copilot ライセンスがない場合は、演習 4 をスキップしても構いません。

これを行うには、次のタスクを実行します。

1. Teams アプリ マニフェストを変更して**メッセージ拡張機能/プラグインのユーザー インターフェイスを拡張します**。 これには、**companySearch** という新しいコマンドの導入が含まれます。 メッセージ拡張機能の UI がアダプティブ カードであることを確認します。Copilot の場合は、Copilot チャットでのテキストの入力と出力です。

1. ** コマンドのハンドラーを作成します**。 これにより、メッセージ ルーティング コードから渡されたクエリ文字列が解析され、入力が検証され、会社の API によって製品検索が呼び出されます。 このタスクでは、返された製品リストがアダプティブ カードに入力され、メッセージまたは Copilot チャット UI に表示されます。

1. コマンド **ルーティング** コードを更新して、前のタスクで作成したハンドラーに新しいコマンドをルーティングします。 これを行うには、ユーザーが Northwind データベース (**handleTeamsMessagingExtensionQuery**) に対してクエリを実行するときに Bot Framework によって呼び出されるメソッドを拡張します。 

1. その会社が注文した製品の一覧を返す**会社別の製品検索を実装**します。

1. **アプリを実行**して、指定した会社によって購入された製品を検索します。

## タスク 1 - メッセージ拡張機能/プラグインのユーザー インターフェイスを拡張する 

1. Visual Studio Code の**作業フォルダー**から **manifest.json** を開き、`discountSearch` コマンド (**98 行目**) の直後に次の json を追加します。 この追加情報を使用して、プラグインでサポートされているコマンドの一覧を定義する `commands` 配列に追加します。

   ```json
   {
       "id": "companySearch",
       "context": [
           "compose",
           "commandBox"
       ],
       "description": "Given a company name, search for products ordered by that company",
       "title": "Customer",
       "type": "query",
       "parameters": [
           {
               "name": "companyName",
               "title": "Company name",
               "description": "The company name to find products ordered by that company",
               "inputType": "text"
           }
       ]
   }
   ```

> [!NOTE] 
> **id** は、UI とコードの間のリンクです。 この値は、**discount/product/SearchCommand.ts** ファイルで **COMMAND_ID** として定義されます。 これらの各ファイルに、**id** の値に対応する一意の **COMMAND_ID** がどのように含まれているかを確認します。

## タスク 2 -「companySearc」コマンドのハンドラーを作成する

この演習では、既存のコードの一部をコピーして、コマンドの新しいハンドラーを作成します。 

1. Visual Studio Code の**作業ディレクトリ**の下で **.\src\messageExtensions** に移動し、'**productSearchCommand.ts**' をコピーし、同じフォルダーに貼り付けてコピーを作成します。 このファイルを **customerSearchCommand.ts** に名前変更します。

1. 7 行目を次のように変更します。

    ```typescript
    import { searchProductsByCustomer } from "../northwindDB/products";
    ```

1. 10 行目を次のように変更します:

   ```javascript
   const COMMAND_ID = "companySearch";
   ```



1. **handleTeamsMessagingExtensionQuery** の内容を次のように置き換えます。

   ```javascript
    {
       let companyName;
   
       // Validate the incoming query, making sure it's the 'companySearch' command
       // The value of the 'companyName' parameter is the company name to search for
       if (query.parameters.length === 1 && query.parameters[0]?.name === "companyName") {
           [companyName] = (query.parameters[0]?.value.split(','));
       } else { 
           companyName = cleanupParam(query.parameters.find((element) => element.name === "companyName")?.value);
       }
       console.log(`🍽️ Query #${++queryCount}:\ncompanyName=${companyName}`);    
   
       const products = await searchProductsByCustomer(companyName);
   
       console.log(`Found ${products.length} products in the Northwind database`)
       const attachments = [];
       products.forEach((product) => {
           const preview = CardFactory.heroCard(product.ProductName,
               `Customer: ${companyName}`, [product.ImageUrl]);
   
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

> [!NOTE]
> `searchProductsByCustomer` はタスク 4 で実装します。

## タスク 3 - コマンド ルーティングを更新する

このタスクでは、前のタスクで実装したハンドラーに `companySearch` コマンドをルーティングします。

1. **searchApp.ts** を開き、**10 行目**でこれを探します。

   ```javascript
   import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
   ```

1. これを **11 行目**に追加します。

   ```javascript
   import customerSearchCommand from "./messageExtensions/customerSearchCommand";
   ```

1. このステートメントの下に:

   ```javascript
         case discountedSearchCommand.COMMAND_ID: {
           return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
         }
   ```

1. このステートメントを追加します:

   ```javascript
         case customerSearchCommand.COMMAND_ID: {
           return customerSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
         }
   ```

> [!NOTE]
> プラグインの UI ベースの操作が明示的に呼び出されます。 ただし、Microsoft 365 Copilot によって呼び出されると、コマンドは Copilot オーケストレーターによってトリガーされます。

## タスク 4 - 会社別の製品検索を実装する

このタスクでは、**会社名による**製品検索を実装して会社の注文済み製品の一覧を返します。 クエリからのテーブル出力は次のようになります。

| テーブル         | ［検索］        | 次によって参照    |
| ------------- | ----------- | ------------- |
| カスタマー      | 顧客 ID | 顧客名 |
| 注文        | 注文 ID    | Customer ID   |
| OrderDetail   | Product     | 注文 ID      |

クエリのしくみ 

1. **Customer** テーブルを使用して、**顧客名** で **顧客 ID** を検索します。 

1. **顧客 ID** を使用して **Orders** テーブルに対してクエリを実行し、関連付けられている **注文 ID** を取得します。 

1. **注文 ID** ごとに、**OrderDetail** テーブルで関連する製品を見つけます。 

1. 最後に、指定した会社名で並べ替えられた製品の一覧を返します。

次に、**products.ts** ファイルを変更して、新しい検索クエリを追加します。

1. **.\src\northwindDB\products.ts** を開きます

1. 1 行目の `import` ステートメントを更新して、OrderDetail、Order、Customer を含めます。 次のようになっているはずです。

   ```javascript
   import {
       TABLE_NAME, Product, ProductEx, Supplier, Category, OrderDetail,
       Order, Customer
   } from './model';
   ```

1. **8 行目**のこの下:

   ```javascript
   import { getInventoryStatus } from '../adaptiveCards/utils';
   ```

       Add the new function `searchProductsByCustomer()`:

   ```javascript
   export async function searchProductsByCustomer(companyName: string): Promise<ProductEx[]> {

       let result = await getAllProductsEx();
   
       let customers = await loadReferenceData<Customer>(TABLE_NAME.CUSTOMER);
       let customerId="";
       for (const c in customers) {
           if (customers[c].CompanyName.toLowerCase().includes(companyName.toLowerCase())) {
               customerId = customers[c].CustomerID;
               break;
           }
       }
       
       if (customerId === "") 
           return [];

       let orders = await loadReferenceData<Order>(TABLE_NAME.ORDER);
       let orderdetails = await loadReferenceData<OrderDetail>(TABLE_NAME.ORDER_DETAIL);
       // build an array orders by customer id
       let customerOrders = [];
       for (const o in orders) {
           if (customerId === orders[o].CustomerID) {
               customerOrders.push(orders[o]);
           }
       }
       
       let customerOrdersDetails = [];
       // build an array order details customerOrders array
       for (const od in orderdetails) {
           for (const co in customerOrders) {
               if (customerOrders[co].OrderID === orderdetails[od].OrderID) {
                   customerOrdersDetails.push(orderdetails[od]);
               }
           }
       }

       // Filter products by the ProductID in the customerOrdersDetails array
       result = result.filter(product => 
           customerOrdersDetails.some(order => order.ProductID === product.ProductID)
       );

       return result;
   }
   ```

## タスク 5 - アプリを実行する 会社名で製品を検索します

これで、Microsoft 365 Copilot のプラグインとしてサンプルをテストする準備ができました。

1. Teams で **Northwest Inventory** アプリを削除します。 このタスクはマニフェストを更新するために必要です。 マニフェストの更新では、アプリを再インストールする必要があります。 これを行う最もクリーンな方法は、最初に Teams から削除することです。

    1. Teams サイドバーで、3 つのドット (...) 1️⃣ を選択します。 アプリケーションの一覧に **Northwind Inventory** 2️⃣ が表示されます。

    1. **Northwest Inventory** アイコンを右クリックし、[アンインストール] 3️⃣ を選択します。

        ![Northwind Inventory をアンインストールする方法のスクリーンショット。](../media/3-01-uninstall-app.png)

1. **F5** キーを押して、**Debug in Teams (Edge)** プロファイルを使用して Visual Studio Code でアプリを起動します。

1. Teams で、**[チャット]** を選択し、次に ** Copilot** を選択します。 Copilot は、最上位のオプションである必要があります。

1. **Plugin アイコン**を選択し、**Northwind Inventory** を選択してプラグインを有効にします。

1. プロンプトを入力します。 

   ```console
   What are the products ordered by 'Consolidated Holdings' in Northwind Inventory?
   ```

   ターミナルの出力は、Copilot がクエリを理解し、`companySearch` コマンドを実行し、Copilot によって抽出された会社名を渡したことを示しています。

   ![クエリを理解し、「companySearch」コマンドを実行した Copilot のスクリーンショット。](../media/3-08-terminal-query-output.png)

   Copilot の出力を次に示します。

   ![コマンドから結果を生成する Copilot のスクリーンショット。](../media/3-07-response-customer-search.png)

試してみる他のプロンプトを次に示します。

```console
What are the products ordered by 'Consolidated Holdings' in Northwind Inventory? Please list the product name, price and supplier in a table.
```

もちろん、前の演習で行ったように、サンプルをメッセージ拡張機能として使用して、この新しいコマンドをテストすることもできます。 

1. Teams サイドバーで、**チャット**セクションに移動し、任意のチャットを選択するか、同僚と新しいチャットを開始します。

1. **+** 記号を選択して、[アプリ] メニューにアクセスします。

1. **Northwind Inventory** アプリを選択します。

1. **Customer** という名前の新しいタブがどのように表示されるかに注目してください。

1. **Consolidated Holdings** を検索し、この会社が注文した製品を参照してください。 これらは、前のタスクで Copilot から返されたものと一致します。

    ![メッセージ拡張機能として使用される新しいコマンドのスクリーンショット。](../media/3-08-customer-message-extension.png)

## 作業を確認

演習が完了したら、Northwind Inventory アプリで会社別の注文を検索する新しいコマンドが必要です。 また、Copilot でプラグインを正常に使用し、他のアプリでメッセージ拡張機能として使用することもできます。 

次の演習では、プラグインのソース コードとアダプティブ カードを調べて、アプリの構築方法と、さらにカスタマイズする方法について学習します。

[次の演習に進んでください...](./6-exercise-4-explore-plugin-source-code.md)