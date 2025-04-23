---
lab:
  title: '演習 3: OAuth で保護された API と API プラグインを統合する'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# 演習 3: OAuth で保護された API と API プラグインを統合する

Microsoft 365 Copilot 用の API プラグインを使用すると、OAuth で保護された API と統合できます。 API を保護するアプリのクライアント ID とシークレットは、Teams コンテナーに登録することで保護します。 実行時に、Microsoft 365 Copilot はプラグインを実行し、コンテナーから情報を取得し、それを使用してアクセス トークンを取得し、API を呼び出します。 このプロセスに従えば、クライアント ID とシークレットは保護され、クライアントに公開されることはありません。

### 演習の期間

- **推定所要時間**: 10 分

## タスク 1 - サンプル プロジェクトを開いて認証構成を調べる

まず、サンプル プロジェクトをダウンロードします。

1. Web ブラウザーで、[https://aka.ms/learn-da-api-ts-repairs](https://aka.ms/learn-da-api-ts-repairs) に移動します。 サンプル プロジェクトを含む ZIP ファイルをダウンロードするように求めるメッセージが表示されます。
1. ZIP ファイルを、お使いのコンピューターに保存します。
1. ZIP ファイルの内容を展開します。
1. Visual Studio Code で  フォルダーを開きます。

このサンプル プロジェクトは、宣言型エージェント、API プラグイン、Microsoft Entra ID で保護された API を含む Teams Toolkit プロジェクトです。 この API は Azure Functions 上で動作し、Azure Functions の組み込み認証および承認機能 (Easy Auth とも呼ばれます) を使用してセキュリティを実装します。

## タスク 2 - OAuth2 承認構成を調べる

続行する前に、サンプル プロジェクトの OAuth2 承認構成を調べます。

### API 定義を調べる

まず、プロジェクトに含まれる API 定義のセキュリティ構成を確認します。

Visual Studio Code:

1. **appPackage/apiSpecificationFile/repair.yml** ファイルを開きます。
1. **components.securitySchemes** セクションで、**oAuth2AuthCode** プロパティに注目してください。

  ```yml
  components:
    securitySchemes:
    oAuth2AuthCode:
      type: oauth2
      description: OAuth configuration for the repair service
      flows:
      authorizationCode:
        authorizationUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/authorize
        tokenUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/token
        scopes:
        api://${{AAD_APP_CLIENT_ID}}/repairs_read: Read repair records 
  ```

  このプロパティは OAuth2 セキュリティ スキームを定義しており、アクセス トークンを取得するために呼び出される URL と API で使用されるスコープに関する情報を含んでいます。

  > **重要** スコープがアプリケーション ID URI (**api://...**) で完全修飾されていることに注意してください。Microsoft Entra を使用する際は、カスタム スコープを完全修飾する必要があります。 Microsoft Entra では、修飾されていないスコープが検出されると、それは Microsoft Graph に属していると見なされるので、承認フロー エラーが発生します。

1. **paths./repairs.get.security** プロパティを見つけます。 これは **oAuth2AuthCode** セキュリティ スキームと、クライアントによる操作の実行に必要なスコープを参照していることに注意してください。

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - oAuth2AuthCode:
        - api://${{AAD_APP_CLIENT_ID}}/repairs_read
  [...]
  ```

  > **重要** API 仕様での必要なスコープの一覧は、単に情報提供を目的としたものです。 API を実装する際は、トークンを検証し、必要なスコープが含まれていることを確認する必要があります。

### API 実装を調べる

次に、API 実装について確認します。

Visual Studio Code:

1. **src/functions/dishes.ts** ファイルを開きます。
1. **repairs** ハンドラー関数で次の行を見つけます。この行では、必要なスコープを持つアクセス トークンが要求に含まれているかどうかを確認しています。

  ```typescript
  if (!hasRequiredScopes(req, 'repairs_read')) {
    return {
    status: 403,
    body: "Insufficient permissions",
    };
  }
  ```

1. **repairs.ts** ファイルで **hasRequiredScopes** 関数をさらに実装します。

  ```typescript
  function hasRequiredScopes(req: HttpRequest, requiredScopes: string[] | string): boolean {
    if (typeof requiredScopes === 'string') {
    requiredScopes = [requiredScopes];
    }
  
    const token = req.headers.get("Authorization")?.split(" ");
    if (!token || token[0] !== "Bearer") {
    return false;
    }
  
    try {
    const decodedToken = jwtDecode<JwtPayload & { scp?: string }>(token[1]);
    const scopes = decodedToken.scp?.split(" ") ?? [];
    return requiredScopes.every(scope => scopes.includes(scope));
    }
    catch (error) {
    return false;
    }
  }
  ```

  この関数は、まず、承認要求ヘッダーからベアラー トークンを抽出します。 次に、**jwt-decode** パッケージを使用してトークンをデコードし、**scp** 要求からスコープのリストを取得します。 最後に、必要なすべてのスコープが **scp** 要求に含まれているかどうかを確認します。

  この関数ではアクセス トークンを検証していないことに注意してください。 代わりに、必要なスコープがアクセス トークンに含まれているかどうかを確認しているだけです。 このテンプレートでは、API は Azure Functions 上で動作し、Easy Auth を使用してセキュリティを実装します。アクセス トークンの検証は Easy Auth が担当します。 有効なアクセス トークンが要求に含まれていない場合、Azure Functions ランタイムはコードに到達する前に要求を拒否します。 Easy Auth はトークンを検証しますが、必要なスコープの確認は行いません。それは自分で行う必要があります。

### コンテナー タスクの構成を調べる

このプロジェクトでは、Teams Toolkit を使用して OAuth 情報をコンテナーに追加します。 Teams Toolkit は、プロジェクトの構成で特別なタスクを使用して、OAuth 情報をコンテナーに登録します。

Visual Studio Code:

1. **./teampsapp.local.yml** ファイルを開きます。
1. **[provision]** セクションで、**oauth/register** タスクを見つけます。

  ```yml
  - uses: oauth/register
    with:
    name: oAuth2AuthCode
    flow: authorizationCode
    appId: ${{TEAMS_APP_ID}}
    clientId: ${{AAD_APP_CLIENT_ID}}
    clientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
    isPKCEEnabled: true
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    writeToEnvironmentFile:
    configurationId: OAUTH2AUTHCODE_CONFIGURATION_ID
  ```

  タスクは、**env/.env.local** ファイルと **env/.env.local.user** ファイルに格納されている **TEAMS_APP_ID**、**AAD_APP_CLIENT_ID**、**SECRET_AAD_APP_CLIENT_SECRET** プロジェクト変数の値を取得し、それらをコンテナーに登録します。 セキュリティ強化のために、Proof Key for Code Exchange (PKCE) も有効にします。 次に、コンテナーエントリ ID を受け取り、環境ファイル **env/.env.local** に書き込みます。 このタスクの結果は、**OAUTH2AUTHCODE_CONFIGURATION_ID** という名前の環境変数です。 Teams Toolkit は、プラグイン定義を含む **appPackages/ai-plugin.json** ファイルにこの変数の値を書き込みます。 実行時に、API プラグインを読み込む宣言型エージェントは、この ID を使用してコンテナーから OAuth 情報を取得し、認証フローを開始してアクセス トークンを取得します。

  > **重要** **oauth/register** タスクは、OAuth 情報がまだ存在しない場合にのみコンテナーに登録します。 情報が既に存在する場合、Teams Toolkit はこのタスクの実行をスキップします。

1. 次に、**oauth/update** タスクを見つけます。

  ```yml
  - uses: oauth/update
    with:
    name: oAuth2AuthCode
    appId: ${{TEAMS_APP_ID}}
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    configurationId: ${{OAUTH2AUTHCODE_CONFIGURATION_ID}}
    isPKCEEnabled: true
  ```

  このタスクにより、コンテナー内の OAuth 情報をプロジェクトと同期させます。 プロジェクトが正常に動作するためには、この操作が必要です。 重要なプロパティの 1 つは、API プラグインを使用可能な URL です。 プロジェクトを開始するたびに、Teams Toolkit は新しい URL で開発トンネルを開きます。 Copilot が API にアクセスできるようにするには、コンテナー内の OAuth 情報でこの URL を参照する必要があります。

### 認証と承認の構成を確認する

次に、Azure Functions の認証と認可の設定について説明します。 この演習の API では、Azure Functions の組み込みの認証および認可機能を使用します。 Teams Toolkit は、Azure Functions を Azure にプロビジョニングする際にこれらの機能を構成します。

Visual Studio Code:

1. **infra/azure.bicep** ファイルを開きます。
1. **authSettings** リソースを見つけます。

  ```bicep
  resource authSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    parent: functionApp
    name: 'authsettingsV2'
    properties: {
    globalValidation: {
      requireAuthentication: true
      unauthenticatedClientAction: 'Return401'
    }
    identityProviders: {
      azureActiveDirectory: {
      enabled: true
      registration: {
        openIdIssuer: oauthAuthority
        clientId: aadAppClientId
      }
      validation: {
        allowedAudiences: [
        aadAppClientId
        aadApplicationIdUri
        ]
      }
      }
    }
    }
  }
  ```

  このリソースにより、Azure Functions アプリの組み込みの認証および認可機能を有効にします。 まず、**globalValidation** セクションで、アプリが認証済みのリクエストのみを許可することを定義します。 アプリが認証されていないリクエストを受信した場合、401 HTTP エラーを返して拒否します。 次に、**identityProviders** セクションで、Microsoft Entra ID (以前の Azure Active Directory) を使用してリクエストを承認するように構成を定義します。 API をセキュリティで保護するために使用する Microsoft Entra アプリ登録と、API の呼び出しが許可される対象ユーザーを指定します。

### Microsoft Entraアプリケーション登録を確認する

最後に調べるのは、プロジェクトが API をセキュリティで保護するために使用する Microsoft Entra アプリケーションの登録です。 OAuth を使用する場合、アプリケーションを使用してリソースへのアクセスをセキュリティで保護します。 通常、アプリケーションでは、クライアント シークレットや証明書などのアクセス トークンを取得するために必要な資格情報が定義されています。 また、API を呼び出すときにクライアントが要求できる様々なアクセス許可 (別称スコープ) も指定されています。 Microsoft Entra アプリケーションの登録では、Microsoft クラウド内のアプリケーションを表し、OAuth 認可フローで使用するアプリケーションを定義します。

Visual Studio Code:

1. **./aad.manifest.json** ファイルを開きます。
1. **oauth2Permissions** プロパティを検索します。

  ```json
  "oauth2Permissions": [
    {
    "adminConsentDescription": "Allows Copilot to read repair records on your behalf.",
    "adminConsentDisplayName": "Read repairs",
    "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
    "isEnabled": true,
    "type": "User",
    "userConsentDescription": "Allows Copilot to read repair records.",
    "userConsentDisplayName": "Read repairs",
    "value": "repairs_read"
    }
  ],
  ```

  このプロパティは、修復 API から修復を読み取るためのアクセス許可をクライアントに付与する、**repairs_read** という名前のカスタム スコープを定義します。

1. **identifierUris** プロパティを検索します。

  ```json
  "identifierUris": [
    "api://${{AAD_APP_CLIENT_ID}}"
  ]
  ```

  **identifierUris** プロパティは、スコープを完全に修飾するために使用される識別子を定義します。
