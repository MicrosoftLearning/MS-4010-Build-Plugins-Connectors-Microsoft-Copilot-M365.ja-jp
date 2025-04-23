---
lab:
  title: '演習 1: キーで保護された API と API プラグインを統合する'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# 演習 1: キーで保護された API と API プラグインを統合する

Microsoft 365 Copilot 用の API プラグインを使用すると、キーで保護された API と統合できます。 API キーは、Teams コンテナーに登録すると、セキュリティで保護されます。 実行時に、Microsoft 365 Copilot はプラグインを実行し、コンテナーから API キーを取得し、それを使用して API を呼び出します。 このプロセスに従えば、API キーはセキュリティで保護され、クライアントに公開されることはありません。

この演習では、生成した API キーで認証を行う API プラグインを使用して、新しい宣言型エージェントを作成します。

### 演習の期間

- **推定所要時間**: 10 分

## タスク 1: 新しいプロジェクトを作成する

まず、Microsoft 365 Copilot 用の新しい API プラグインを作成します。 Visual Studio Code を開きます。

Visual Studio Code:

1. **Activity Bar** (サイド バー) で、Teams Toolkit 拡張機能を開きます。
1. **Teams Toolkit** 拡張機能パネルで、**[新しいアプリの作成]** を選択します。
1. プロジェクト テンプレートの一覧から、**[Copilot エージェント]** を選択します。
1. アプリ機能の一覧から、**[宣言型エージェント]** を選択します。
1. **[プラグインの追加]** オプションを選択します。
1. **[新しい API で開始]** オプションを選択します。
1. 認証の種類の一覧から、**[API キー (ベアラー トークン認証)]** を選択します。
1. プログラミング言語として **[TypeScript]** を選択します。
1. プロジェクトを格納するフォルダーを選択します。
1. プロジェクトに **da-repairs-key** という名前を付けます。

Teams Toolkit は、宣言型エージェント、API プラグイン、キーで保護された API を含む新しいプロジェクトを作成します。

## タスク 2: API キーの認証構成を調べる

続行する前に、生成されたプロジェクトの API キー認証構成を確認します。

### API 定義を調べる

まず、API 定義で API キー認証がどのように定義されているかを確認します。

Visual Studio Code:

1. **appPackage/apiSpecificationFile/repair.yml** ファイルを開きます。 このファイルには、API の OpenAPI 定義が含まれています。
1. **components.securitySchemes** セクションで、**apiKey** プロパティに注目してください。

  ```yml
  components:
    securitySchemes:
    apiKey:
      type: http
      scheme: bearer
  ```

  このプロパティは、承認要求ヘッダーのベアラー トークンとして API キーを使用するセキュリティ スキームを定義します。

1. **paths./repairs.get.security** プロパティを見つけます。 **apiKey** セキュリティ スキームを参照していることに注意してください。

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - apiKey: []
  [...] 
  ```

### API 実装を調べる

次に、API が各要求で API キーを検証する方法を確認します。

Visual Studio Code:

1. **src/functions/dishes.ts** ファイルを開きます。
1. **repairs** ハンドラー関数で、承認されていないすべての要求を拒否する次の行を見つけます。

  ```typescript
  if (!isApiKeyValid(req)) {
    // Return 401 Unauthorized response.
    return {
    status: 401,
    };
  } 
  ```

1. **isApiKeyValid** 関数は、repairs.ts ファイルにさらに実装されます。

  ```typescript
  function isApiKeyValid(req: HttpRequest): boolean {
    const apiKey = req.headers.get("Authorization")?.replace("Bearer ", "").trim();
    return apiKey === process.env.API_KEY;
  }
  ```

  この関数は、承認ヘッダーにベアラー トークンが含まれているかどうかを確認し、**API_KEY** 環境変数で定義されている API キーと比較します。

このコードは、API キー セキュリティの単純な実装を示していますが、実際の API キー セキュリティのしくみを示しています。

### コンテナー タスクの構成を調べる

このプロジェクトでは、Teams Toolkit を使用して API キーをコンテナーに追加します。 Teams Toolkit は、プロジェクトの構成で特別なタスクを使用して、コンテナーに API キーを登録します。

Visual Studio Code:

1. **./teampsapp.local.yml** ファイルを開きます。
1. **provision** セクションで、**apiKey/register** タスクを見つけます。

  ```yml
  # Register API KEY
  - uses: apiKey/register
    with:
    # Name of the API Key
    name: apiKey
    # Value of the API Key
    primaryClientSecret: ${{SECRET_API_KEY}}
    # Teams app ID
    appId: ${{TEAMS_APP_ID}}
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    # Write the registration information of API Key into environment file for
    # the specified environment variable(s).
    writeToEnvironmentFile:
    registrationId: APIKEY_REGISTRATION_ID
  ```

  タスクは、**SECRET_API_KEY** プロジェクト変数の値を受け取り、**env/.env.local.user** ファイルに格納し、コンテナーに登録します。 次に、コンテナーエントリ ID を受け取り、環境ファイル **env/.env.local** に書き込みます。 このタスクの結果は、**APIKEY_REGISTRATION_ID** という名前の環境変数です。 Teams Toolkit は、プラグイン定義を含む **appPackages/ai-plugin.json** ファイルにこの変数の値を書き込みます。 実行時、宣言型エージェントは、API プラグインを読み込み、この ID を使用してコンテナーから API キーを取得し、API を安全に呼び出します。

## タスク 3: ローカル開発用に API キーを構成する

プロジェクトをテストする前に、API の API キーを定義する必要があります。 次に、API キーをコンテナーに格納し、API プラグインにコンテナー エントリ ID を記録します。 ローカル開発の場合は、プロジェクトに API キーを格納し、Teams ツールキットを使用してコンテナーに登録します。

Visual Studio Code:

1. **Terminal** ペインを開きます (Ctrl + ')。
1. コマンドラインで次の操作を行います。
  1. プロジェクトの依存関係を復元するには、`npm install` を実行します。
  1. `npm run keygen`を実行して、新しい API キーを生成します。
  1. 生成されたキーをクリップボードにコピーします。
1. **env/.env.local.user** ファイルを開きます。
1. **SECRET_API_KEY** プロパティを新しく生成された API キーに更新します。 更新されたプロパティは次のようになります。

  ```text
  SECRET_API_KEY='your_key'
  ```

1. 変更を保存。

プロジェクトをビルドするたびに、Teams ツールキットによってコンテナー内の API キーが自動的に更新され、コンテナー エントリ ID でプロジェクトが更新されます。