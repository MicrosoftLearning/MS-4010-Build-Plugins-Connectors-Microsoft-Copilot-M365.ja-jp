---
lab:
  title: 演習 4 - Microsoft 365 用 Copilot で使用するメッセージ拡張機能を拡張して最適化する
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 演習 4 - Microsoft 365 用 Copilot で使用するメッセージ拡張機能を拡張して最適化する

この演習では、Microsoft 365 用 Copilot で使用するために、メッセージ拡張機能を拡張して最適化します。 対象ユーザーという名前の新しいパラメーターを追加し、複数のパラメーターを処理するようにメッセージ拡張ロジックを更新します。 最後に、メッセージ拡張機能を実行してデバッグし、Microsoft Teams 用 Copilot でテストします。

## タスク 1 - アプリ マニフェストを更新する

アプリ マニフェストに簡潔で正確な説明を指定することは、プラグインを呼び出すタイミングと方法を Copilot に確実に認識させるために不可欠です。 アプリ マニフェストでアプリ、コマンド、およびパラメータの説明を更新します。

Visual Studio を開きます。

1. **appPackage** フォルダーで、**manifest.json** という名前のファイルを開きます。
1. **description** オブジェクトを更新する

    ```json
    {
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
        },
    }
    ```

    コマンドに別のパラメータを追加するため、新しいパラメータを含むようにコマンドの説明も更新します。

1. **commands** 配列で、コマンドの **description** を更新します。

    ```json
    {
        "commands": [
            {
                "id": "Search",
                "type": "query",
                "title": "Products",
                "description": "Find products by name or by target audience",
                "initialRun": true,
                "fetchTask": false,
                "context": [...],
                "parameters": [...]
            }
        ]
    }
    ```

    次に、Copilot で使用できる新しいパラメータを追加します。 この新しいパラメータは、ユーザーが、個人や企業など、さまざまな対象ユーザーを対象とする Copilot を使用して製品を検索するのに役立ちます。

1. **parameters** 配列で、**ProductName** パラメータの後に **TargetAudience** パラメータを追加します。

    ```json
    {    
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
    }
    ```

1. 変更を保存します

**TargetAudience** パラメータの説明は、その内容を説明し、パラメータが許可される値 **Consumer** または **Enterprise** を受け入れる必要があることを説明します。

## タスク 2 - メッセージ拡張機能ロジックを更新する

新しいパラメータをサポートし、複雑なプロンプトをサポートするには、ボット アクティビティ ハンドラーの OnTeamsMessagingExtensionQueryAsync メソッドを更新して、複数のパラメータを処理します。

ユーザーが「Mark8 という名前の個人を対象とした Contoso 製品を検索する」というプロンプトを入力したとします。 パラメータの説明を指定すると、「個人向け」は **Consumer** に変換され、**TargetAudience** パラメータの値として渡されます。 「Mark8」は、**ProductName** パラメータの値として渡されます。

Visual Studio での続行:

1. **Search** フォルダーで、**SearchApp.cs** という名前のファイルを開きます
1. **OnTeamsMessagingExtensionQueryAsync** メソッドで、次のコード ブロックを見つけます。

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters); 
    ```

1. コード ブロックを更新して **TargetAudience** パラメーターの値を取得し、SharePoint Online リストに対してクエリを実行するときに使用するフィルター クエリを作成します。

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var retailCategory = GetQueryData(query.Parameters, "TargetAudience");
    
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var retailCategoryFilter = !string.IsNullOrEmpty(retailCategory) ? $"fields/RetailCategory eq '{retailCategory}'" : string.Empty;
    var filters = new List<string> { nameFilter };
    filters.RemoveAll(f => string.IsNullOrEmpty(f));
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. 変更を保存します

## タスク 3 - リソースをプロビジョニングする

リソースをプロビジョニングするには、Teams アプリの依存関係の準備プロセスを実行します。

Visual Studio での続行:

1. **Solution Explorer** で、**MsgExtProductSupport** プロジェクトを右クリックします
1. **Teams Toolkit** メニューを展開し、**[Prepare Teams App Dependencies (Teams アプリの依存関係を準備)]** を選択します
1. **Microsoft 365 アカウント**ダイアログで、**[続行]** を選択します
1. **[プロビジョニング]** ダイアログで、**[プロビジョニング]** を選択します。
1. **[Teams Toolkit warning (Teams Toolkit の警告)]** ダイアログで、**[プロビジョニング]** を選択します。
1. **[Teams Toolkit information (Teams Toolkit の情報]** ダイアログで、プロンプトを**閉じます**

## タスク 4 - 実行とデバッグ

次に、Web サービスを開始し、Microsoft 365 用 Copilot でメッセージ拡張機能をテストします。

1. **F5** キーを押してデバッグ セッションを開始し、新しいブラウザー ウィンドウを開くと Microsoft Teams Web クライアントに移動されます。
1. Microsoft 365 アカウントの資格情報を入力し、Microsoft Teams に進みます。
1. アプリのインストール ダイアログで、**[追加]** を選びます。
1. Microsoft Teams から **Copilot** アプリを開きます。
1. メッセージ作成領域で、**[プラグイン]** ポップアップを開きます。
1. プラグインの一覧で、**Contoso 製品**プラグインを切り替えて有効にします
1. メッセージとして**個人向けの Contoso 製品を探す**と入力し、送信します
1. Copilot 応答でサインイン ボタンが返されます。 **[サインイン]** ボタンを選択して認証します
1. 認証に成功したら、メッセージとして**個人を対象とした Contoso 製品を探す**と入力して送信します
1. Copilot 応答では、プラグイン応答で返されたデータが表示され、応答でプラグインが参照されます。
1. 結果に関連するアダプティブ カードを表示するには、Copilot 応答の参照をポイントします

ブラウザーを閉じてデバッグ セッションを終了します。

[課題の概要に進みます...](./6-summary.md)