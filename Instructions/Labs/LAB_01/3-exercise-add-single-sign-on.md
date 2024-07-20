---
lab:
  title: 演習 2 - シングル サインオンを追加する
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 演習 2 - シングル サインオンを追加する

この演習では、ユーザーにサインインと認証を求めるメッセージが表示されるように、メッセージ拡張機能を更新します。 シングル サインオンを有効にするには、Bot Microsoft Entra アプリの登録とアプリ マニフェストを構成します。 Microsoft Graph で認証するように Microsoft Entra アプリの登録を構成し、Bot Framework トークン サービスを使用してアクセス トークンを取得するようにメッセージ拡張ロジックを更新します。 次に、メッセージ拡張機能を実行してデバッグし、Microsoft Teams でテストします。

## タスク 1 - シングル サインオンを構成する

まず、Microsoft Entra ID テナントでアプリ登録を構成します。

Visual Studio:

1. **infra\entra** フォルダーで、**entra.bot.manifest.json** という名前のファイルを開きます
1. ファイルで、**identifierUris** 配列を更新して、アプリケーション ID URI を設定します

    ```json
    "identifierUris": [
        "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    ],
    ```

1. ファイルで、**oauth2Permissions** 配列を更新して、Teams が Web API を管理者またはユーザーとして呼び出せるようにスコープを作成します。

    ```json
      "oauth2Permissions": [
        {
          "adminConsentDescription": "Allows Teams to call the app's web APIs as the current user.",
          "adminConsentDisplayName": "Teams can access app's web APIs",
          "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Enable Teams to call this app's web APIs with the same rights that you have",
          "userConsentDisplayName": "Teams can access app's web APIs and make requests on your behalf",
          "value": "access_as_user"
        }
      ]
    ```

1. ファイルで、**preAuthorizedApplications** 配列を更新して、Microsoft Teams、Microsoft Outlook、および Copilot for Microsoft 365 クライアントを承認されたクライアントの一覧に追加します。

    ```json
      "preAuthorizedApplications": [
        {
          "appId": "1fec8e78-bce4-4aaf-ab1b-5451cc387264",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "5e3ce6c0-2b1f-4285-8d4b-75ee78787346",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "4765445b-32c6-49b0-83e6-1d93765276ca",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "0ec893e0-5785-4de6-99da-4ed124e5296c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "d3590ed6-52b3-4102-aeff-aad2292ab01c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "bc59ab01-8403-45c6-8796-ac3ef710b3e3",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "27922004-5251-4030-b22d-91ecd9a37ea4",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        }
      ],
    ```

1. 変更を保存します

次に、アプリ マニフェスト ファイルを更新して、アプリでシングル サインオン フローを開始するときにクライアントが使用するリソースを定義します。

Visual Studio での続行:

1. **appPackage** フォルダーで、**manifest.json** という名前のファイルを開きます。
1. ファイルで、**description** の後に次のコードを追加します。

    ```json
    "webApplicationInfo": {
      "id": "${{BOT_ID}}",
      "resource": "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    },
    ```

1. 変更を保存します

## タスク 2 - Microsoft Graph 用の Microsoft Entra アプリ登録マニフェスト ファイルを作成する

Microsoft Graph で認証するには、新しいアプリ登録マニフェスト ファイルを作成します。

Visual Studio での続行:

1. **infra\entra** フォルダーに、**entra.graph.manifest.json** という名前のファイルを作成します
2.  ファイルに、次のコードを追加します。

    ```json
    {
      "id": "${{GRAPH_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{GRAPH_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}",
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
          "resourceAppId": "Microsoft Graph",
          "resourceAccess": [
            {
              "id": "Sites.ReadWrite.All",
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

1. 変更を保存します

**requiredResourceAccess** 配列は API アクセス許可スコープを定義し、**replyUrlsWithType** 配列はアプリ登録のリダイレクト URI を定義します。

次に、アプリの登録を作成するアクションでプロジェクト ファイルを更新します。

1. プロジェクトのルート フォルダーで、**teamsapp.local.yml** を開きます
1. ファイルで、**arm/deploy** アクションを使用する手順を見つけます
1. この手順の前に、次のコードを追加します。

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: GRAPH_ENTRA_APP_ID
          clientSecret: SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET
          objectId: GRAPH_ENTRA_APP_OBJECT_ID
          tenantId: GRAPH_ENTRA_APP_TENANT_ID
          authority: GRAPH_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: GRAPH_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.graph.manifest.json"
          outputFilePath : "./build/entra.graph.manifest.${{TEAMSFX_ENV}}.json"
    ```

1. 変更を保存します

## タスク 3 - OAuth 接続設定を作成する

Azure AI Bot Service の接続設定は、ボットとメッセージ拡張機能のユーザー認証を管理するために使用されます。

まず、接続設定を環境変数として作成するために使用される OAuth 接続設定の名前を一元化し、実行時に環境変数の値を使用するコードを追加します。

Visual Studio での続行:  

1. **env** フォルダーで、**.env.local** を開きます
1.  ファイルに、次のコードを追加します。

    ```text
    CONNECTION_NAME=MicrosoftGraph
    ```

1. プロジェクトのルート フォルダーで、**teamsapp.local.yml** という名前のファイルを開きます
1. ファイルで、**./appsettings.Development.json** ファイルを対象とする **file/createOrUpdateJsonFile** アクションを使用する手順を見つけます
1. コンテンツ配列を更新し、**CONNECTION_NAME** 変数を追加します

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. 変更を保存します
1. プロジェクトのルート フォルダーで、**Config.cs** を開きます
1. **ConfigOptions** クラスで、**CONNECTION_NAME** という名前の新しい文字列プロパティを追加します

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. 変更を保存します
1. プロジェクトのルート フォルダーで、**Program.cs** という名前のファイルを開きます
1. ファイルで、**CONNECTION_NAME** 環境変数をアプリ構成設定として追加する新しい行を追加します。

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    ```

次に、ボット アクティビティ ハンドラーを更新して、アプリの構成にアクセスします。

1. **Search** フォルダーで、**SearchApp.cs** という名前のファイルを開きます
1. **SearchApp** クラスで、**connectionName** という名前の読み取り専用文字列プロパティを作成します。

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    }
    ```

1. コンストラクターを作成し、App Configuration を挿入し、connectionName プロパティの値を設定します。

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    
      public SearchApp(IConfiguration configuration)
      {
        connectionName = configuration["CONNECTION_NAME"];
      }  
    }
    ```

1. 変更を保存します

次に、OAuth 接続設定をプロビジョニングするように Bicep ファイルを更新します。

まず、パラメータ ファイルを更新して、Microsoft Graph Microsoft Entra アプリの登録の資格情報と接続設定の名前を渡します。

1.  **infra** フォルダーで、 **azure.parameters.local.json** という名前のファイルを開きます
1. パラメータ **object** に、 **graphEntraAppClientId**、 **graphEntraAppClientSecret**、**connectionName** パラメータを追加します

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
        "graphEntraAppClientId": {
          "value": "${{GRAPH_ENTRA_APP_ID}}"
        },
        "graphEntraAppClientSecret": {
          "value": "${{SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. 変更を保存します

次に、Bicep ファイルを更新します。

1. **infra** フォルダーで、**azure.local.bicep** という名前のファイルを開きます
1. ファイルで、**botAppDomain** パラメータ宣言の後に、**graphEntraAppClientId**、 **graphEntraAppClientSecret**、**connectionName** パラメータ宣言を追加します。

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. **azureBotRegistration** モジュールで、**graphEntraAppClientId**、**graphEntraAppClientSecret**、**connectionName** パラメータを追加します。

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        graphEntraAppClientId: graphEntraAppClientId
        graphEntraAppClientSecret: graphEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. 変更を保存。

最後に、ボット登録 Bicep ファイルを更新します。

1. **infra/botRegistration** フォルダーで、**azurebot.bicep** という名前のファイルを開きます
1. ファイルで、**botAppDomain** パラメータ宣言の後に、**graphEntraAppClientId**、**graphEntraAppClientSecret**、**connectionName** パラメータ宣言を追加します

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. **botServiceM365ExtensionsChannel** リソースの後に、Microsoft Graph 接続用の新しいリソースを追加します

    ```bicep
    resource botServicesMicrosoftGraphConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: graphEntraAppClientId
        clientSecret: graphEntraAppClientSecret
        scopes: 'email offline_access openid profile Sites.ReadWrite.All'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${ botAadAppClientId}'
          }
        ]
      }
    }
    ```

1. 変更を保存します

## タスク 4 - ユーザー クエリを認証する

次に、メッセージ拡張機能を使用して検索を開始するときにユーザーを認証するコードを追加します。

Visual Studio での続行:

1. **Search** フォルダーで、**SearchApp.cs** という名前のファイルを開きます
1. ファイルで、まず Bot Framework SDK から **Authentication** 名前空間を追加します。

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. **OnTeamsMessagingExtensionQueryAsync** メソッドの先頭に次のコードを追加します。

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    
    if (!HasToken(tokenResponse))
    {
        return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

上記のコード ブロックでは、ユーザー認証を処理する 3 つの方法が使用されています。

- **GetToken** はトークン サービス クライアントを使用して、現在のユーザーのアクセス トークンを取得します
- **HasToken** は、トークン サービスからの応答にアクセス トークンが含まれているかどうかを確認します
- トークンが返されない場合は **CreateAuthResponse** が呼び出され、応答が返されて、ユーザー インターフェイスにサインイン リンクが表示されます。

次に、**SearchApp** クラスにメソッドを作成します。

- 次のコードを使用して **GetToken** という名前のメソッドを作成します。

```csharp
private static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
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
```

まず、状態パラメータが null または空でないかどうかを確認します。コードは int.TryParse メソッドを使用して整数として解析しようとします。 解析が成功した場合、解析された値は文字列として magicCode 変数に割り当てられます。 その後、magicCode は、他のパラメータと共に GetUserTokenAsync メソッドに引数として渡されます。 GetUserTokenAsync メソッドは、magicCode を使用して要求の信頼性を検証します。 これにより、認証プロセスを開始したのと同じエンティティによってユーザー トークンが要求されます。

- 次のコードを使用して HasToken メソッドを作成します。

```csharp
private static bool HasToken(TokenResponse tokenResponse)
{
    return tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
}
```

トークン サービスから有効なトークンが取得されたかどうかを確認するには、応答のトークン応答と "Token" プロパティが空でも null でもないことを確認します。

- 次のコードを使用して、**CreateAuthResponse** メソッドを作成します。

```csharp
private static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
{
    var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);

    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "auth",
            SuggestedActions = new MessagingExtensionSuggestedAction
            {
                Actions = new List<CardAction>
                {
                    new() {
                        Type = ActionTypes.OpenUrl,
                        Value = resource.SignInLink,
                        Title = "Sign In",
                    },
                },
            },
        },
    };
}
```

まず、**GetSignInResourceAsync** メソッドを使用して、トークン サービスからサインイン リンクを取得します。 サインイン リンクは、**MessagingExtensionResponse** オブジェクトを構築するために使用されます。 新しいオブジェクトを作成し、応答の **ComposeExtension** プロパティを新しい **MessagingExtensionResult** オブジェクトに設定します。 結果の type プロパティは「auth」に設定され、結果が認証応答であることを示します。 結果の **SuggestedActions** プロパティは、新しい **MessagingExtensionSuggestedAction** オブジェクトに設定されます。 推奨されるアクションの Actions プロパティは、1 つの **CardAction** オブジェクトを含むリストに設定されます。 この **CardAction** オブジェクトは、ユーザーが実行できるアクションを表します。 **CardAction** の Type プロパティは **ActionTypes.OpenUrl** に設定され、URL を開くアクションであることを示します。 Value プロパティは、リソースから取得したサインイン リンクに設定されます。 Title プロパティは、アクションのタイトルを指定する「Sign In」に設定されます。 最後に、構築された応答がメソッドから返されます。

ユーザーがサインイン リンクに従うと、外部ドメインでホストされているリソースに移動します。 ドメインはアプリ マニフェスト ファイルに含まれている必要があります。 Bot Framework トークン サービスのドメインをアプリ マニフェストに追加します。

Visual Studio での続行:

1. **appPackage** フォルダーで、**manifest.json** を開きます
1. ファイルで、**validDomains** 配列を更新し、トークン サービスのドメインを追加します。

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]    
    ```

1. 変更を保存します

## タスク 5 - リソースをプロビジョニングする

すべてが整ったので、Teams アプリの依存関係の準備プロセスを実行して、必要なリソースをプロビジョニングします。

Visual Studio での続行:

1. **ソリューション エクスプローラー**で、**TeamsApp** プロジェクトを右クリックします
1. **Teams Toolkit** メニューを展開し、**[Prepare Teams App Dependencies (Teams アプリの依存関係を準備)]** を選択します
1. **Microsoft 365 アカウント**ダイアログで、**[続行]** を選択します
1. **[プロビジョニング]** ダイアログで、**[プロビジョニング]** を選択します。
1. **[Teams Toolkit warning (Teams Toolkit の警告)]** ダイアログで、**[プロビジョニング]** を選択します。
1. **[Teams Toolkit information (Teams Toolkit の情報)]** ダイアログで、**[View provisioned resources (プロビジョニングされたリソースの表示)]** を選択して、新しいブラウザー ウィンドウを開きます。

少し時間を取って、Azure で作成および更新されたリソースを調べてみましょう。

## タスク 6 - 実行とデバッグ

次に、Web サービスを開始し、Microsoft Teams でメッセージ拡張機能をテストします。

Visual Studio での続行:

1. **F5** キーを押してデバッグ セッションを開始し、新しいブラウザー ウィンドウを開くと Microsoft Teams Web クライアントに移動されます。
1. ブラウザーで、必要に応じて Microsoft 365 アカウントの資格情報を入力し、Microsoft Teams に進みます。
1. アプリのインストール ダイアログで、**[追加]** を選びます。
1. 新規または既存の Microsoft Teams チャットを開きます
1. メッセージ作成領域で、**[...] **を選択します アプリのポップアップが開きます
1. アプリの一覧で **C[ontoso 製品]** を選択してメッセージ拡張機能を開きます
1. テキスト ボックスに「 **Bot Builder** 」と入力して検索を開始します。
1. 「 **このアプリを使用するにはサインインする必要があります**」というメッセージが表示されます
1.  **サインイン リンク**を選択して新しいタブを開き、認証フローを開始します
1. アクセス許可の同意ページで、要求されているアクセス許可を確認します。
1. **[同意]** を選択してタブを閉じ、Microsoft Teams に戻ります
1. 結果の一覧で、**結果を選択**して作成メッセージ ボックスにカードを埋め込み、送信します。

ブラウザーを閉じてデバッグ セッションを終了します。

[次の演習に進んでください...](./4-exercise-retrieve-product-information-from-sharepoint-online.md)