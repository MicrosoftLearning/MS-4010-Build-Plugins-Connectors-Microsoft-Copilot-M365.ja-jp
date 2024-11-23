---
lab:
  title: 演習 2 - シングル サインオンを追加する
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 演習 2 - シングル サインオンを追加する

この演習では、メッセージ拡張機能にシングル サインオンを追加して、ユーザー クエリを認証します。

![検索ベースのメッセージ拡張機能での認証チャレンジのスクリーンショット。 サインインへのリンクが表示されます。](../media/2-sign-in.png)

### 演習の期間

  - **推定所要時間**: 40 分

## タスク 1: バックエンド API アプリの登録を構成する

まず、バックエンド API の Microsoft Entra アプリ登録を作成します。 この演習では、新しいアプリを作成しますが、運用環境では既存のアプリ登録を使用します。

ブラウザー ウィンドウで次の手順を実行します。

1. [Azure Portal](https://portal.azure.com) に移動します。

1. ポータル メニューを開き、**[Microsoft Entra ID]** を選択します。

1. **[管理] - [アプリの登録]** を選択し、**[新しい登録]** を選択します。

1. [アプリケーションの登録] フォームで、次の値を指定します。

    1. **名前**: 製品 API

    1. **サポートされるアカウントの種類**: 任意の組織ディレクトリ (任意の Microsoft Entra ID テナント - マルチテナント) 内のアカウント

1. **[登録]** を選択して、アプリの登録を作成します。

1. アプリの登録の左側のメニューで、** [管理] - [API の公開]** を選択します。

1. **[アプリケーション ID URI]** の横にある **[追加]** を選択し、**[保存]** を選択して新しいアプリケーション ID URI を作成します。

1. [この API で定義されるスコープ] で、**[スコープの追加]** を選択します。

1. [スコープの追加] フォームで、次の値を指定します。

    1. **スコープ名**: Product.Read

    1. **同意できるのはだれですか?**: 管理者とユーザー

    1. **管理者の同意の表示名**: 製品読み取り

    1. **管理者の同意の説明**: アプリは製品データを読み取ることができます

    1. **ユーザーの同意の表示名**: 製品読み取り

    1. **ユーザーの同意の説明**: アプリは製品データを読み取ることができます

    1. **[状態]**: 有効

1. **[スコープの追加]** を選択して、スコープを作成します。

次に、アプリ登録 ID とスコープ ID をメモします。 バックエンド API のアクセス トークンを取得するために使用するアプリの登録を構成するには、これらの値が必要です。

1. アプリの登録の左側のメニューで、**[マニフェスト]** を選択します。

1. **appId** プロパティ値をコピーして、後で使用するために保存します。

1. **oauth2Permissions.id** プロパティ値をコピーして、後で使用するために保存します。

プロジェクトでこれらの値が必要になるため、環境ファイルに追加します。

Visual Studio と **TeamsApp** プロジェクトで、次の手順を実行します。

1. **env** フォルダーで、**.env.local** を開きます。

1. ファイルで、次の環境変数を追加して、前に保存した**アプリ登録 ID** と**スコープ ID** に値を設定します。

    ```text
    BACKEND_API_ENTRA_APP_ID=<app-registration-id>
    BACKEND_API_ENTRA_APP_SCOPE_ID=<scope-id>
    ```

1. 変更を保存。

## タスク 2: バックエンド API を使用して認証するためのアプリ登録マニフェスト ファイルを作成する

バックエンド API で認証するには、API を呼び出すアクセス トークンを取得するためのアプリの登録が必要です。

次に、アプリ登録マニフェスト ファイルを作成します。 マニフェストは、アプリの登録の API アクセス許可スコープとリダイレクト URI を定義します。

Visual Studio と **TeamsApp** プロジェクトで、次の手順を実行します。

1. **infra\entra** フォルダーに、**entra.products.api.manifest.json** という名前の新しいファイル (<kbd>Ctrl + Shift + A</kbd>) を作成します。

1.  ファイルに、次のコードを追加します。

    ```json
    {
      "id": "${{PRODUCTS_API_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{PRODUCTS_API_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-product-api-${{TEAMSFX_ENV}}",
      "accessTokenAcceptedVersion": 2,
      "signInAudience": "AzureADMultipleOrgs",
      "optionalClaims": {
        "idToken": [],
        "accessToken": [
          {
            "name": "idtyp",
            "source": null,
            "essential": false,
            "additionalProperties": []
          }
        ],
        "saml2Token": []
      },
      "requiredResourceAccess": [
        {
          "resourceAppId": "${{BACKEND_API_ENTRA_APP_ID}}",
          "resourceAccess": [
            {
              "id": "${{BACKEND_API_ENTRA_APP_SCOPE_ID}}",
              "type": "Scope"
            }
          ]
        }
      ],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": [
        {
          "url": "https://token.botframework.com/.auth/web/redirect",
          "type": "Web"
        }
      ]
    }
    ```

1. 変更を保存。

**requiredResourceAccess** プロパティは、バックエンド API のアプリ登録 ID とスコープ ID を指定します。

**replyUrlsWithType** プロパティは、ユーザーの認証後に、Bot Framework トークン サービスがトークン サービスにアクセス トークンを返すために使用するリダイレクト URI を指定します。

次に、自動化されたワークフローを更新して、アプリの登録を作成および更新します。

**TeamsApp** プロジェクトで次の手順を実行します。

1. **teamsapp.local.yml** を開きます。

1. ファイルで、**aadApp/update** アクションを使用する手順を見つけます。

1. アクションの後に、**aadApp/create** アクションと **aadApp/update** アクションを追加して、アプリの登録を作成および更新します (** 31 行目**以降)。

    ```yml
      - uses: aadApp/create
        with:
            name: ${{APP_INTERNAL_NAME}}-products-api-${{TEAMSFX_ENV}}
            generateClientSecret: true
            signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
            clientId: PRODUCTS_API_ENTRA_APP_ID
            clientSecret: SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET
            objectId: PRODUCTS_API_ENTRA_APP_OBJECT_ID
            tenantId: PRODUCTS_API_ENTRA_APP_TENANT_ID
            authority: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY
            authorityHost: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
            manifestPath: "./infra/entra/entra.products.api.manifest.json"
            outputFilePath : "./infra/entra/build/entra.products.api.${{TEAMSFX_ENV}}.json"
    ```

1. 変更を保存します

**aadApp/create** アクションは、指定した名前や対象ユーザーを使用して新しいアプリ登録を作成し、クライアント シークレットを生成します。 **writeToEnvironmentFile** プロパティは、アプリ登録 ID、クライアント シークレット、オブジェクト ID、テナント ID、機関、および機関ホストを環境ファイルに書き込みます。 クライアント シークレットは暗号化され、**env.local.user** ファイルに安全に格納されます。 クライアント シークレットの環境変数の名前の先頭には、**SECRET_** が付き、ログに値を書き込まないよう Teams Toolkit に指示します。

**aadApp/update** アクションは、指定したマニフェスト ファイルを使用してアプリ登録を更新します。

## タスク 3: 接続設定名を一元化する

次に、環境ファイルの接続設定名を一元化して、アプリ構成を更新し、実行時に環境変数の値にアクセスします。

Visual Studio と **TeamsApp** プロジェクトで続行します。

1. **env** フォルダーで、**.env.local** を開きます。

1.  ファイルに、次のコードを追加します。

    ```text
    CONNECTION_NAME=ProductsAPI
    ```

1. **teamsapp.local.yml** を開きます。

1. ファイルで、**./appsettings.Development.json** ファイルを対象とする **file/createOrUpdateJsonFile** アクションを使用する手順を見つけます。 **CONNECTION_NAME** 環境変数を含むようにコンテンツ配列を更新し、**appsettings Development.json** ファイルに値を書き込みます。

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../ProductsPlugin/appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. 変更を保存。

次に、アプリ構成を更新して、**CONNECTION_NAME** 環境変数にアクセスします。

**ProductsPlugin** プロジェクトで次の手順を実行します。

1. **Config.cs** を開きます。

1. **ConfigOptions** クラスで、**CONNECTION_NAME** という名前の新しい文字列プロパティを追加します。

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. 変更を保存。

1. **Program.cs** を開きます。

1. ファイルで、**CONNECTION_NAME** プロパティを含むように、アプリ構成を読み取るコードを更新します。

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["ConnectionName"] = config.CONNECTION_NAME;
    ```

1. 変更を保存。

次に、実行時に接続設定名を使用するように、ボット コードを更新します。

1. **Search** フォルダーで、**SearchApp.cs** を開きます。

1. **SearchApp** クラスの先頭 (14 行目の辺り) に、**IConfiguration** オブジェクトを受け取り、**CONNECTION_NAME** プロパティの値を **connectionName** という名前のプライベート フィールドに割り当てるコンストラクターを作成します。

    ```csharp
    private readonly string connectionName;
    public SearchApp(IConfiguration configuration)
    {
      connectionName = configuration["CONNECTION_NAME"];
    }  
    ```

1. 変更を保存。

## タスク 4: Products API 接続設定を構成する

バックエンド API で認証するには、Azure Bot リソースで接続設定を構成する必要があります。

Visual Studio と **TeamsApp** プロジェクトで続行します。

1. **infra** フォルダーで、**azure.parameters.local.json** を開きます。

1. ファイルに、**backendApiEntraAppClientId**、**productsApiEntraAppClientId**、**productsApiEntraAppClientSecret**、および **connectionName** パラメーターを追加します。

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "resourceBaseName": {
          "value": "bot-${{RESOURCE_SUFFIX}}-${{TEAMSFX_ENV}}"
        },
        "botEntraAppClientId": {
          "value": "${{BOT_ID}}"
        },
        "botDisplayName": {
          "value": "${{APP_DISPLAY_NAME}}"
        },
        "botAppDomain": {
          "value": "${{BOT_DOMAIN}}"
        },
        "backendApiEntraAppClientId": {
          "value": "${{BACKEND_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientId": {
          "value": "${{PRODUCTS_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientSecret": {
          "value": "${{SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. 変更を保存。

次に、新しいパラメーターを含むように Bicep ファイルを更新し、それらを Azure Bot リソースに渡します。

1. **infra** フォルダーで、**azure.local.bicep** という名前のファイルを開きます。

1. ファイルに、**botAppDomain** パラメーター宣言の後に、**backendApiEntraAppClientId**、**productsApiEntraAppClientId**、**productsApiEntraAppClientSecret**、および **connectionName** パラメーター宣言を追加します。

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. **azureBotRegistration** モジュール宣言で、新しいパラメーターを追加します。

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botEntraAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        backendApiEntraAppClientId: backendApiEntraAppClientId
        productsApiEntraAppClientId: productsApiEntraAppClientId
        productsApiEntraAppClientSecret: productsApiEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. 変更を保存。

最後に、新しい接続設定を含むように、ボット登録 Bicep ファイルを更新します。

1. **infra/botRegistration** フォルダーで、**azurebot.bicep** を開きます。

1. ファイルに、**botAppDomain** パラメーター宣言の後に、**backendApiEntraAppClientId**、**productsApiEntraAppClientId**、**productsApiEntraAppClientSecret**、および **connectionName** パラメーター宣言を追加します。

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. ファイルに、**botServicesProductsApiConnection** という名前の新しいリソースをファイルの末尾に追加します。

    ```bicep
    resource botServicesProductsApiConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: productsApiEntraAppClientId
        clientSecret: productsApiEntraAppClientSecret
        scopes: 'api://${backendApiEntraAppClientId}/Product.Read'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${botEntraAppClientId}'
          }
        ]
      }
    }
    ```

1. 変更を保存。

## タスク 5: メッセージ拡張機能で認証を構成する

メッセージ拡張機能でユーザー クエリを認証するには、Bot Framework SDK を使用して、Bot Framework トークン サービスからユーザーのアクセス トークンを取得します。 その後、アクセス トークンを使用して、外部サービスからデータにアクセスできます。

コードを簡略化するには、ユーザー認証を処理するヘルパー クラスを作成します。

Visual Studio と **ProductsPlugin** プロジェクトで続行します。

1. **Helpers** という名前の新しいフォルダーを作成します。

1. **Helpers** フォルダーに、**AuthHelpers.cs** という名前の新しいクラス ファイルを作成します。

1.  ファイルに、次のコードを追加します。

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    using Microsoft.Bot.Schema.Teams;
    internal static class AuthHelpers
    {
        internal static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
        {
            var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);
            return new MessagingExtensionResponse
            {
                ComposeExtension = new MessagingExtensionResult
                {
                    Type = "auth",
                    SuggestedActions = new MessagingExtensionSuggestedAction
                    {
                        Actions = [
                            new() {
                                Type = ActionTypes.OpenUrl,
                                Value = resource.SignInLink,
                                Title = "Sign In",
                            },
                        ],
                    },
                },
            };
        }
        internal static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
        {
            var magicCode = string.Empty;
            if (!string.IsNullOrEmpty(state))
            {
                if (int.TryParse(state, out var parsed))
                {
                    magicCode = parsed.ToString();
                }
            }
            return await userTokenClient.GetUserTokenAsync(userId, connectionName, channelId, magicCode, cancellationToken);
        }
        internal static bool HasToken(TokenResponse tokenResponse) => tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
    }
    ```

1. 変更を保存。

**AuthHelpers** クラスの 3 つのヘルパー メソッドは、メッセージ拡張機能のユーザー認証を処理します。

- **CreateAuthResponse** メソッドは、ユーザー インターフェイスにサインイン リンクをレンダリングする応答を構築します。 **GetSignInResourceAsync** メソッドを使用して、トークン サービスからサインイン リンクを取得します。

- **GetToken** メソッドは、トークン サービス クライアントを使用して、現在のユーザーのアクセス トークンを取得します。 メソッドは、マジック コードを使用して、要求の信頼性を検証します。

- **HasToken** メソッドは、トークン サービスからの応答にアクセス トークンが含まれているかどうかを確認します。 トークンが null または空でない場合、メソッドは true を返します。

次に、ヘルパー メソッドを使用してユーザー クエリを認証するように、メッセージ拡張コードを更新します。

1. **Search** フォルダーで、**SearchApp.cs** を開きます。

1. ファイルの先頭に次の using ステートメントを追加します。

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. **OnTeamsMessagingExtensionQueryAsync** メソッドの先頭に次のコードを追加します。

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

1. 変更を保存。

次に、トークン サービス ドメインをアプリ マニフェスト ファイルに追加して、クライアントがシングル サインオン フローを開始するときにドメインを信頼できるようにします。

**TeamsApp** プロジェクトで次の手順を実行します。

1. **appPackage** フォルダーで、**manifest.json** を開きます。

1. ファイルで、**validDomains** 配列を更新し、トークン サービスのドメインを追加します。

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]
    ```

1. 変更を保存。

## タスク 6: リソースを作成して更新する

すべての準備が整ったので、**Teams アプリの依存関係の準備**プロセスを実行して新しいリソースを作成し、既存のリソースを更新します。

> [!NOTE]
> プロビジョニングで依存関係の準備に失敗する場合は、**BACKEND_API_ENTRA_APP_ID** と **BACKEND_API_ENTRA_APP_SCOPE_ID** の適切な値が **env.local** にあることを確認してください。

Visual Studio での続行:

1. **Solution Explorer** で、**TeamsApp** プロジェクトを右クリックします。

1. **[Teams Toolkit]** メニューを展開し、**[Prepare Teams App Dependencies]** を選択します。

1. **[Microsoft 365 account]** ダイアログで、**[Continue]** を選択します。

1. **[Provision]** ダイアログで、**[Provision]** を選択します。

1. **[Teams Toolkit warning]** ダイアログで、**[Provision]** を選択します。

1. **[Teams Toolkit information]** ダイアログで、十字アイコンを選択してダイアログを閉じます。

## タスク 7 - 実行とデバッグ

リソースがプロビジョニングされたら、デバッグ セッションを開始してメッセージ拡張機能をテストします。

1. 新しいデバッグ セッションを開始するには、<kbd>F5</kbd> キーを押すか、ツール バーから **[Start]** を選択します。

1. ブラウザー ウィンドウが開き、Microsoft Teams Web クライアントにアプリのインストール ダイアログが表示されるまで待ちます。 メッセージが表示されたら、Microsoft 365 アカウント資格情報を入力します。

1. [アプリのインストール] ダイアログで、**[追加]** を選択します。

1. 新規または既存の Microsoft Teams チャットを開きます。

1. メッセージ作成領域で、「**/apps**」と入力を開始してアプリ ピッカーを開きます。

1. アプリの一覧で、**[Contoso 製品]** を選択してメッセージ拡張機能を開きます。

1. テキスト ボックスに、「** hello**」と入力します。 検索を複数回入力する必要がある場合があります。

1. 「**このアプリを使用するにはサインインする必要があります**」というメッセージが表示されます。

    ![検索ベースのメッセージ拡張機能での認証チャレンジのスクリーンショット。 サインインへのリンクが表示されます。](../media/2-sign-in.png)

1. **サインイン** リンクに従って認証フローを開始します。

1. 要求されたアクセス許可に同意し、Microsoft Teams に戻ります。

    ![[Microsoft Entra API のアクセス許可の同意] ダイアログのスクリーンショット。](../media/18-api-permission-consent.png)

1. 検索が完了し、結果が表示されるまで待ちます。

1. 結果の一覧で、**hello** を選択して作成メッセージ ボックスにカードを埋め込みます。

Visual Studio に戻り、ツール バーから **[Stop]** を選択するか、<kbd>Shift</kbd> + <kbd>F5</kbd> キーを押してデバッグ セッションを停止します。

[次の演習に進んでください...](./4-exercise-return-data-api.md)