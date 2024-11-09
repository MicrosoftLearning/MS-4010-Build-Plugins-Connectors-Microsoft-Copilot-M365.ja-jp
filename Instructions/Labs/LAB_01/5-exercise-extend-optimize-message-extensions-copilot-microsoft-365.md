---
lab:
  title: 演習 4 - Microsoft 365 用 Copilot で使用するメッセージ拡張機能を拡張して最適化する
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 演習 4 - Microsoft 365 用 Copilot で使用するメッセージ拡張機能を拡張して最適化する

この演習では、Microsoft 365 用 Copilot で使用するために、メッセージ拡張機能を拡張して最適化します。 対象ユーザーという名前の新しいパラメーターを追加し、複数のパラメーターを処理するようにメッセージ拡張ロジックを更新します。 最後に、メッセージ拡張機能を実行してデバッグし、Microsoft Teams 用 Copilot でテストします。

![メッセージ拡張機能プラグインによって返される情報を含む Copilot for Microsoft 365 の回答のスクリーンショット。 製品情報を示すアダプティブ カードが表示されます。](../media/5-copilot-answer.png)

> [!NOTE]
> この演習で Microsoft 365 Copilot ライセンスを必要とする唯一のタスクは、タスク 5 です。 テナントに Copilot があるかどうかに関係なく、前のタスクを実行する必要があります。

### 演習の期間

  - **推定所要時間**: 40 分

## タスク 1: アプリの説明を更新する

アプリ マニフェストに簡潔で正確な説明を指定することは、プラグインを呼び出すタイミングと方法を Copilot に確実に認識させるために不可欠です。 アプリ マニフェストでアプリ、コマンド、およびパラメータの説明を更新します。

Visual Studio を開き、**TeamsApp** プロジェクトで次の手順を実行します。

1. **appPackage** フォルダーで、**manifest.json** を開きます。

1. **description** オブジェクトを更新する

    ```json
    "description": {
        "short": "Product look up tool.",
        "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
    },
    ```

## タスク 2: 新しいパラメーターを追加する

Copilot で使用できる新しいパラメーターを追加します。 この新しいパラメーターは、ユーザーが、個人や企業など、さまざまな対象ユーザーを対象とする Copilot を使用して製品を検索するのに役立ちます。

Visual Studio と **TeamsApp** プロジェクトで続行します。

1. **manifest.json** の **parameters** 配列で、**ProductName** パラメーターの後に **TargetAudience** パラメーターを追加します。

    ```json
    "parameters": [
        {
            "name": "ProductName",
            "title": "Product name",
            "description": "The name of the product as a keyword",
            "inputType": "text"
        },
        {
            "name": "TargetAudience",
            "title": "Target audience",
            "description": "Audience that the product is aimed at. Consumer products are sold to individuals. Enterprise products are sold to businesses",
            "inputType": "text"
        }
    ]
    ```

1. 変更を保存。

**TargetAudience** パラメータの説明は、その内容を説明し、パラメータが許可される値 **Consumer** または **Enterprise** を受け入れる必要があることを説明します。

次に、新しいパラメーターを含むようにコマンドの説明を更新してます。

1. **commands** 配列で、**36 行目**のコマンドの **description** を更新します。

    ```json
    "description": "Find products by name or by target audience.",
    ```

## タスク 2: メッセージ拡張機能ロジックを更新する

新しいパラメータをサポートし、複雑なプロンプトをサポートするには、ボット アクティビティ ハンドラーの **OnTeamsMessagingExtensionQueryAsync** メソッドを更新して、複数のパラメーターを処理します。

まず、**ProductService** クラスを更新して、名前パラメーターと対象ユーザー パラメーターに基づいて製品を取得します。

**ProductPlugin** プロジェクトの Visual Studio で続行します。

1. **Services** フォルダーで、**ProductsService.cs** を開きます。

1. ファイルで、**GetProductsByCategoryAsync** と **GetProductsByNameAndCategoryAsync** という名前の新しいメソッドを作成します。

    ```csharp
    internal async Task<Product[]> GetProductsByCategoryAsync(string category)
    {
        var response = await _httpClient.GetAsync($"{_baseUri}products?category={category}");
        response.EnsureSuccessStatusCode();
        var jsonString = await response.Content.ReadAsStringAsync();
        return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
    }
    internal async Task<Product[]> GetProductsByNameAndCategoryAsync(string name, string category)
    {
        var response = await _httpClient.GetAsync($"{_baseUri}?name={name}&category={category}");
        response.EnsureSuccessStatusCode();
        var jsonString = await response.Content.ReadAsStringAsync();
        return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
    }
    ```

1. 変更を保存。

次に、**MessageExtensionHelper** クラスに新しいメソッドを追加して、名前パラメーターと対象ユーザー パラメーターに基づいて製品を取得します。

1. **Helpers** フォルダーで、**MessageExtensionHelper.cs** を開きます。

1. ファイルで、名前パラメーターと対象ユーザー パラメーターに基づいて製品を取得する **RetrieveProducts** という新しいメソッドを作成します。

    ```csharp
    internal static async Task<IList<Product>> RetrieveProducts(string name, string audience, ProductsService productsService)
    {
        IList<Product> products;
        if (string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByCategoryAsync(audience);
        }
        else if (!string.IsNullOrEmpty(name) && string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByNameAsync(name);
        }
        else if (!string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByNameAndCategoryAsync(name, audience);
        }
        else
        {
            products = [];
        }
        return products;
    }
    ```

1. 変更を保存。

**RetrieveProduct** メソッドは、名前パラメーターと対象ユーザー パラメーターに基づいて製品を取得します。 名前パラメーターが空で、対象ユーザー パラメーターが空でない場合、メソッドは対象ユーザー パラメーターに基づいて製品を取得します。 名前パラメーターが空でなく、対象ユーザー パラメーターが空の場合、メソッドは名前パラメーターに基づいて製品を取得します。 名前パラメーターと対象ユーザー パラメーターの両方が空でない場合、メソッドは両方のパラメーターに基づいて製品を取得します。 両方のパラメーターが空の場合、メソッドは空のリストを返します。

次に、新しいパラメーターを処理するように **SearchApp** クラスを更新します。

1. **Search** フォルダーで、**SearchApp.cs** を開きます。

1. **OnTeamsMessagingExtensionQueryAsync** メソッドで、**30 行目**以降を次のコードに置き換えます。

    ```csharp
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var productService = new ProductsService(tokenResponse.Token);
    var products = await productService.GetProductsByNameAsync(name);
    ```

    置換後のコード:

    ```csharp
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var audience = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "TargetAudience");
    var productService = new ProductsService(tokenResponse.Token);
    var products = await MessageExtensionHelpers.RetrieveProducts(name, audience, productService);
    ```

1. 変更を保存。

**OnTeamsMessagingExtensionQueryAsync** メソッドは、クエリ パラメーターから名前パラメーターと対象ユーザー パラメーターを取得するようになります。 次に、**RetrieveProducts** メソッドを使用して、名前パラメーターと対象ユーザー パラメーターに基づいて製品を取得します。

## タスク 4: リソースを作成して更新する

すべての準備が整ったので、**Teams アプリの依存関係の準備**プロセスを実行して新しいリソースを作成し、既存のリソースを更新します。

Visual Studio での続行:

1. **Solution Explorer** で、**TeamsApp** プロジェクトを右クリックします。

1. **[Teams Toolkit]** メニューを展開し、**[Prepare Teams App Dependencies]** を選択します。

1. **[Microsoft 365 account]** ダイアログで、**[Continue]** を選択します。

1. **[Provision]** ダイアログで、**[Provision]** を選択します。

1. **[Teams Toolkit warning]** ダイアログで、**[Provision]** を選択します。

1. **[Teams Toolkit information]** ダイアログで、十字アイコンを選択してダイアログを閉じます。

## タスク 5: 実行とデバッグ

リソースがプロビジョニングされたら、デバッグ セッションを開始してメッセージ拡張機能をテストします。

まず、**Dev Proxy** を起動してカスタム API をシミュレートします。

1. **[コマンド プロンプト] ウィンドウ**をまだ開いている場合は、次のコマンドを実行して Dev Proxy を起動します。

1. 次のコマンドを実行して、Dev Proxy を起動します。

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. メッセージが表示されたら、証明書の警告を受け入れます。

> [!NOTE]
> Dev Proxy が実行されている場合は、システム全体のプロキシとして機能します。

次に、Visual Studio でデバッグ セッションを開始します。

1. 新しいデバッグ セッションを開始するには、<kbd>F5</kbd> キーを押すか、ツール バーから **[Start]** を選択します。

1. ブラウザー ウィンドウが開き、Microsoft Teams Web クライアントにアプリのインストール ダイアログが表示されるまで待ちます。 メッセージが表示されたら、Microsoft 365 アカウント資格情報を入力します。

1. [アプリのインストール] ダイアログで、**[追加]** を選択します。

1. Microsoft Teams で **Copilot** アプリを開きます。

1. メッセージ作成領域で、**[Plugins]** ポップアップを開きます。

1. プラグインの一覧で、**Contoso 製品**プラグインを切り替えて有効にします。

    ![Contoso 製品プラグインが有効になっている Microsoft Teams の Copilot for Microsoft 365 のスクリーンショット。](../media/20-copilot-plugin-enabled.png)

1. メッセージとして「**個人向けの Contoso 製品を探す**」と入力し、送信します。

1. Copilot が応答するまで待ちます。

    ![ユーザーの要求の処理中に表示されるコパイロット メッセージを示す、Microsoft Teams の Copilot for Microsoft 365 のスクリーンショット。](../media/21-copilot-thinking.png)

1. Copilot の応答では、プラグイン応答で返されたデータが表示され、応答でプラグインが参照されます。

    ![メッセージ拡張機能プラグインによって返される情報を含む Copilot for Microsoft 365 の回答のスクリーンショット。 製品情報を示すアダプティブ カードが表示されます。](../media/5-copilot-answer.png)

1. 結果に関連するアダプティブ カードを表示するには、Copilot 応答の参照をポイントします。

    ![製品情報が表示されたアダプティブ カードを示す、Microsoft Teams の Copilot for Microsoft 365 のスクリーンショット。 このカードは、ユーザーが Copilot 応答の参照の上にマウス ポインターを置くと表示されます。](../media/22-copilot-reference.png)

Visual Studio に戻り、ツール バーから **[Stop]** を選択するか、<kbd>Shift</kbd>  +  <kbd>F5</kbd> キーを押してデバッグ セッションを停止します。 また、<kbd>Ctrl</kbd>  +  <kbd>C</kbd> を使用して Dev Proxy をシャットダウンします。

[課題の概要に進みます...](./6-summary.md)