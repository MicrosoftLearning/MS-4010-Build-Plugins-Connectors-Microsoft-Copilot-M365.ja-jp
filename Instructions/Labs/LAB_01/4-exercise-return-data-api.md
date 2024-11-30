---
lab:
  title: '演習 3: Microsoft Entra で保護された API から製品データを返す'
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 演習 3: Microsoft Entra で保護された API から製品データを返す

この演習では、カスタム API からデータを取得するようにメッセージ拡張機能を更新します。 ユーザー クエリに基づいてカスタム API からデータを取得し、検索結果のデータをユーザーに返します。

![Microsoft Teams の検索ベースのメッセージ拡張機能によって返される検索結果のスクリーンショット。](../media/3-search-results-api.png)

### 演習の期間

  - **推定所要時間:** 50 分

## タスク 1: 開発プロキシをインストールして構成する

この演習では、API をシミュレートできるコマンド ライン ツールである開発プロキシを使用します。 これは、実際の API を作成せずにアプリをテストする場合に便利です。

この演習を完了するには、[最新バージョンの Dev Proxy](/microsoft-cloud/dev/dev-proxy/get-started) をインストールして、このモジュールの Dev Proxy プリセットをダウンロードする必要があります。

このプリセットは、Microsoft Entra によって保護されているメモリ内データ ストアを使用して、CRUD (作成、読み取り、更新、削除) API をシミュレートします。 つまり、認証が必要な実際の API を呼び出しているかのようにアプリをテストできます。

1. Dev Proxy をインストールするには、管理者として新しい **[コマンド プロンプト] ウィンドウ**を開きます。

    ```bash
    winget install Microsoft.DevProxy --silent
    ```

1. プリセットをダウンロードするには、次のコマンドを実行します。

    ```bash
    devproxy preset get learn-copilot-me-plugin
    ```

1. [コマンド プロンプト] ウィンドウを開いたままにして、後で使用できるようにします。

## タスク 4: ユーザー クエリ値を取得する

パラメーターの名前でユーザー クエリ値を取得するメソッドを作成します。

Visual Studio と **ProductsPlugin** プロジェクトで、次の手順を実行します。

1. **Helpers** フォルダーに、**MessageExtensionHelpers.cs** という名前の新しいファイルを作成します。

1. ファイルで、コードを次のように置き換えます。

   ```csharp
   using Microsoft.Bot.Schema.Teams;
   internal class MessageExtensionHelpers
   {
       internal static string GetQueryParameterValueByName(IList<MessagingExtensionParameter> parameters, string name) => parameters.FirstOrDefault(p => p.Name == name)?.Value as string ?? string.Empty;
   }
   ```

1. 変更を保存。

次に、SearchApp クラスの **OnTeamsMessagingExtensionQueryAsync** メソッドを更新して、新しいヘルパー メソッドを使用します。

1. **Search** フォルダーで、**SearchApp.cs** を開きます。

1. **OnTeamsMessagingExtensionQueryAsync** メソッドで、次のコードに置き換えます。

   ```csharp
   var text = query?.Parameters?[0]?.Value as string ?? string.Empty;
   ```

   with

   ```csharp
   var text = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
   ```

1. カーソルを **text** 変数に移動し、`Ctrl + R` と `Ctrl + R` を使用して、変数の名前を **name** に変更します。

1. **Enter** キーを押して、3 つのファイルで変数の名前を変更します。

1. 変更を保存。

**OnTeamsMessagingExtensionQueryAsync** メソッドは、次の図のようになります。

```csharp
protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
{
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
    var template = new AdaptiveCardTemplate(card);
    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "result",
            AttachmentLayout = "list",
            Attachments = [
                new MessagingExtensionAttachment
                    {
                        ContentType = AdaptiveCard.ContentType,
                        Content = JsonConvert.DeserializeObject(template.Expand(new { title = name })),
                        Preview = new ThumbnailCard { Title = name }.ToAttachment()
                    }
            ]
        }
    };
}
```

## タスク 3: カスタム API からデータを取得する

カスタム API からデータを取得するには、要求の Authorization ヘッダーでアクセス トークンを送信し、応答を製品データを表すモデルに逆シリアル化する必要があります。

まず、カスタム API から返される製品データを表すモデルを作成します。

Visual Studio と **ProductsPlugin** プロジェクトで、次の手順を実行します。

1. **Modules** という名前のフォルダーを作成します。

1. **Models** フォルダーに **Product.cs** という名前の新しいファイルを作成します。

1. ファイルで、既存のコードを次のように置き換えます。

   ```csharp
   using System.Text.Json.Serialization;
   internal class Product
   {
       [JsonPropertyName("productId")]
       public int Id { get; set; }
       [JsonPropertyName("imageUrl")]
       public string ImageUrl { get; set; }
       [JsonPropertyName("name")]
       public string Name { get; set; }
       [JsonPropertyName("category")]
       public string Category { get; set; }
       [JsonPropertyName("callVolume")]
       public int CallVolume { get; set; }
       [JsonPropertyName("releaseDate")]
       public string ReleaseDate { get; set; }
   }
   ```

1. 変更を保存。

次に、カスタム API から製品データを取得するサービス クラスを作成します。

Visual Studio と **ProductsPlugin** プロジェクトで、次の手順を実行します。

1. **Services** という名前のフォルダーを作成します。

1. **Search** フォルダーに、**TokenProvider.cs** という名前の新しいファイルを作成します。

1. ファイルで、既存のコードを次のように置き換えます。

    ```csharp
    using System.Net.Http.Headers;
    internal class ProductsService
    {
        private readonly HttpClient _httpClient;
        private readonly string _baseUri = "https://api.contoso.com/v1/";
        internal ProductsService(string token)
        {
            _httpClient = new HttpClient();
            _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
        }
        internal async Task<Product[]> GetProductsByNameAsync(string name)
        {
            var response = await _httpClient.GetAsync($"{_baseUri}products?name={name}");
            response.EnsureSuccessStatusCode();
            var jsonString = await response.Content.ReadAsStringAsync();
            return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
        }
    }
    ```

1. 変更を保存。

**ProductsService** クラスには、カスタム API から製品データを取得するメソッドが含まれています。 クラス コンストラクターは、アクセス トークンをパラメーターとして受け取り、Authorization ヘッダーのアクセス トークンを使用して **HttpClient** インスタンスを設定します。

次に、**OnTeamsMessagingExtensionQueryAsync** メソッドを更新して、**ProductsService** クラスを使用してカスタム API から製品データを取得します。

1. **Search** フォルダーで、**SearchApp.cs** を開きます。

1. **OnTeamsMessagingExtensionQueryAsync** メソッドで、**name** 変数宣言の後に次のコードを追加して、カスタム API から製品データを取得します。

   ```csharp
   var productService = new ProductsService(tokenResponse.Token);
   var products = await productService.GetProductsByNameAsync(name);
   ```

1. 変更を保存。

## タスク 4: 検索結果を作成する

これで製品データが作成されたので、ユーザーに返される検索結果に含めることができます。

まず、既存のアダプティブ カード テンプレートを更新して、製品情報を表示してみましょう。

Visual Studio と **ProductsPlugin** プロジェクトで続行します。

1. **Resources** フォルダーで、名前を **card.json** から **Product.json** に変更します。

1. **Resources** フォルダーに、**Product.json** という名前の新しいファイルを作成します。 このファイルには、Visual Studio がアダプティブ カード テンプレートのプレビューを生成するために使用するサンプル データが含まれています。

1. ファイルに、次の JSON を追加します。

    ```json
    {
      "callVolume": 36,
      "category": "Enterprise",
      "imageUrl": "https://raw.githubusercontent.com/SharePoint/sp-dev-provisioning-templates/master/tenant/productsupport/source/Product%20Imagery/Contoso4.png",
      "name": "Contoso Quad",
      "productId": 1,
      "releaseDate": "2019-02-09"
    }
    ```

1. 変更を保存。

1. **Resources** フォルダーで、**Product.json** を開きます。

1. ファイルで、内容を次の JSON に置き換えます。

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "text": "${name}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${category}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${imageUrl}",
              "altText": "${name}"
            }
          ],
          "minHeight": "350px",
          "verticalContentAlignment": "Center",
          "horizontalAlignment": "Center"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Call Volume",
              "value": "${formatNumber(callVolume,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(releaseDate,'dd/MM/yyyy')}"
            }
          ]
        }
      ]
    }
    ```

1. 変更を保存。

アダプティブ カード テンプレートは、バインディング式を使用して製品情報を表示します。 **\$\{name\}**、**\$\{category\}**、**\$\{imageUrl\}**、**\$\{callVolume\}**、および **\$\{releaseDate\}** 式は、製品データの対応する値に置き換えられます。 **formatNumber** テンプレート関数と **formatDateTime** テンプレート関数は、**callVolume** と **releaseDate** の値をそれぞれ数値と日付に書式設定するために使用されます。

少し時間を取って、Visual Studio のアダプティブ カード プレビューを確認してください。 プレビューには、製品データがテンプレートにバインドされている場合のアダプティブ カード テンプレートの外観が示されます。 **Product.data.json** ファイルのサンプル データを使用してプレビューを生成します。

次に、アプリ マニフェストの **validDomains** プロパティを更新して、**raw.githubusercontent.com** ドメインを含め、アダプティブ カード テンプレート内のイメージを Microsoft Teams に表示できるようにします。

**TeamsApp** プロジェクトで次の手順を実行します。

1. **appPackage** フォルダーで、**manifest.json** を開きます。

1. ファイルで、**validDomains** プロパティに GitHub ドメインを追加します。

    ```json
      "validDomains": [
        "token.botframework.com",
        "raw.githubusercontent.com",
        "${{BOT_DOMAIN}}"
      ],
    ```

1. 変更を保存。

次に、**OnTeamsMessagingExtensionQueryAsync** メソッドを更新して、製品情報を含む添付ファイルの一覧を作成します。

**ProductsPlugin** プロジェクトで次の手順を実行します。

1. **Search** フォルダーで、**SearchApp.cs** を開きます。

1. ファイル名の変更を反映するように、**card.json** を **Product.json**に更新します。 34 行目を次のコードに置き換えます。

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
   ```

   with

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "Product.json"), cancellationToken);
   ```

1. **template** 変数宣言の後に次のコードを追加して、添付ファイルの一覧を作成します。

   ```csharp
    var attachments = products.Select(product =>
    {
        var content = template.Expand(product);
        return new MessagingExtensionAttachment
        {
            ContentType = AdaptiveCard.ContentType,
            Content = JsonConvert.DeserializeObject(content),
            Preview = new ThumbnailCard
            {
                Title = product.Name,
                Subtitle = product.Category,
                Images = [new() { Url = product.ImageUrl }]
            }.ToAttachment()
        };
    }).ToList();
   ```

1. **attachments** 変数を含むように **return** ステートメントを更新します。

   ```csharp
   return new MessagingExtensionResponse
   {
       ComposeExtension = new MessagingExtensionResult
       {
           Type = "result",
           AttachmentLayout = "list",
           Attachments = attachments
       }
   };
   ```

1. 変更の保存

## タスク 5: リソースを作成して更新する

すべての準備が整ったので、**Teams アプリの依存関係の準備**プロセスを実行して新しいリソースを作成し、既存のリソースを更新します。

Visual Studio での続行:

1. **Solution Explorer** で、**TeamsApp** プロジェクトを右クリックします。

1. **[Teams Toolkit]** メニューを展開し、**[Prepare Teams App Dependencies]** を選択します。

1. **[Microsoft 365 account]** ダイアログで、**[Continue]** を選択します。

1. **[Provision]** ダイアログで、**[Provision]** を選択します。

1. **[Teams Toolkit warning]** ダイアログで、**[Provision]** を選択します。

1. **[Teams Toolkit information]** ダイアログで、十字アイコンを選択してダイアログを閉じます。

## タスク 6 - 実行とデバッグ

リソースがプロビジョニングされたら、デバッグ セッションを開始してメッセージ拡張機能をテストします。

まず、Dev Proxy を起動してカスタム API をシミュレートします。

1. **[コマンド プロンプト] ウィンドウ**をまだ開いている場合は、次のコマンドを実行して Dev Proxy を起動します。

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

1. 新規または既存の Microsoft Teams チャットを開きます。

1. メッセージ作成領域で、「**/apps**」と入力してアプリ ピッカーを開きます。

1. アプリの一覧で、**[Contoso 製品]** を選択してメッセージ拡張機能を開きます。

1. テキスト ボックスに「**mark8**」と入力します。 検索クエリを複数回入力する必要がある場合があります。

1. 検索が完了し、結果が表示されるまで待ちます。

    ![Microsoft Teams の検索ベースのメッセージ拡張機能によって返される検索結果のスクリーンショット。](../media/3-search-results-api.png)

1. 結果の一覧で、検索結果を選択して作成メッセージ ボックスにカードを埋め込みます。

Visual Studio に戻り、ツール バーから **[Stop]** を選択するか、<kbd>Shift</kbd>  +  <kbd>F5</kbd> キーを押してデバッグ セッションを停止します。 また、<kbd>Ctrl</kbd>  +  <kbd>C</kbd> を使用して Dev Proxy をシャットダウンします。

[次の演習に進んでください...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)