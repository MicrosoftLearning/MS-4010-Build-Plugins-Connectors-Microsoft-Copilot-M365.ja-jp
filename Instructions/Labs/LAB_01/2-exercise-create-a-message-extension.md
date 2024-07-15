---
lab:
  title: 演習 1 - ページ拡張機能を作成する
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 演習 1 - ページ拡張機能を作成する

この演習では、検索コマンドを使用してメッセージ拡張機能を作成します。 まず、Teams Toolkit プロジェクト テンプレートを使用してプロジェクトをスキャフォールディングし、それを更新して、ローカル開発に Azure AI Bot Service リソースを使用するように構成します。 ボット サービスとローカルで実行されている Web サービスの間の通信を有効にする開発トンネルを作成します。 次に、必要なリソースをプロビジョニングするようにアプリを準備します。 最後に、メッセージ拡張機能を実行してデバッグし、Microsoft Teams でテストします。

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Microsoft Teams の検索ベースのメッセージ拡張機能によって返される検索結果のスクリーンショット。" lightbox="../media/2-search-results-nuget.png":::

## タスク 1 - Teams Toolkit for Visual Studio を使用して新しいプロジェクトを作成する

まず、新しいプロジェクトを作成します。

1. **Visual Studio 2022** を開きます
1. **[ファイル]** メニューを開き、**[新規]** メニューを展開して、 **[新規プロジェクト]** を選択します。
1. [新しいプロジェクトの作成] 画面で、**[すべてのプラットフォーム]** ドロップダウンを展開し、**Microsoft Teams**を選択します。 **次へ**を選んで続行します。
1. [新しいプロジェクトの構成] を示すスクリーンショット。 次の値を指定します。
    1. **プロジェクト名**: MsgExtProductSupport
    1. **場所**: 任意の場所を選択します
    1. **ソリューションとプロジェクトを同じディレクトリに配置する**: オン
1. **[作成]** を選択してプロジェクトをスキャフォールディングする
1. [新しい Teams アプリケーションの作成] ダイアログで、**[All app types (すべてのアプリの種類)]** ドロップダウンを展開し、**Message 拡張機能**を選択します。
1. テンプレートの一覧で **[カスタム検索結果]** を選択します。
1. **[作成]** を選択してアプリをスキャフォールディングする

## タスク 2 - Azure AI Bot Service を構成する

Bot Service リソースは、リソースとして Azure に作成することも、dev.botframework.com 経由で作成することもできます。 デフォルトでは、カスタム検索結果テンプレートは、dev.botframework.com を使用してボットを登録します。 現時点では、ボットを dev.botframework.com に登録することは、Microsoft 365 用 Copilot と互換性がありません。

Copilot for Microsoft 365 をサポートするには、プロジェクトを更新して Azure で Azure AI Bot Service リソースをプロビジョニングし、ローカル開発に使用します。

まず、ファイル全体で再利用してリソースをプロビジョニングするときに使用できるアプリの内部名を一元化する環境変数を作成します。

Visual Studio:

1. **env** フォルダーで、**.env.local** を開きます
1.  ファイルに、次のコードを追加します。

    ```text
    APP_INTERNAL_NAME=msgext-product-support
    ```

1. 変更を保存します

`${{APP_INTERNAL_NAME}}` などのデータ バインディング式を使用すると、Teams Toolkit を使用してリソースをプロビジョニングするときに環境変数の値をファイルに挿入できます。

Azure AI Bot Service リソースをプロビジョニングするには、Microsoft Entra アプリの登録が必要です。 Teams Toolkit がアプリ登録のプロビジョニングに使用するアプリ登録マニフェスト ファイルを作成します。

Visual Studio での続行:

1. **infra** フォルダーに、**entra** という名前の新しいフォルダーを作成します
1. フォルダーに、**entra.bot.manifest.json** という名前のファイルを作成します
1.  ファイルに、次のコードを追加します。

    ```json
    {
      "id": "${{BOT_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{BOT_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}",
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
      "requiredResourceAccess": [],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": []
    }
    ```

1. 変更を保存。

Teams Toolkit では、Bicep ファイルを使用して Azure でリソースをプロビジョニングおよび構成します。 まず、パラメータ ファイルを作成します。 パラメータ ファイルは、環境変数を Bicep テンプレートに渡すために使用されます。

Visual Studio での続行:

1. **infra** フォルダーに、**azure.parameters.local.json** という名前の新しいファイルを作成します
1.  ファイルに、次のコードを追加します。

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
        }
      }
    }
    ```

1. 変更を保存。

次に、パラメータ ファイルで使用される Bicep ファイルを作成します。

1. **infra** フォルダーに **azure.local.bicep** という名前の新しいファイルを作成します
1.  ファイルに、次のコードを追加します。

    ```bicep
    @maxLength(20)
    @minLength(4)
    @description('Used to generate names for all resources in this file')
    param resourceBaseName string
    
    @description('Required when create Azure Bot service')
    param botEntraAppClientId string
    @maxLength(42)
    param botDisplayName string
    param botAppDomain string
    
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
      }
    }
    ```

1. 変更を保存。

最後の手順では、Teams Toolkit プロジェクト ファイルを更新します。 Bot Framework アクションを使用する手順を置き換えて、マニフェスト ファイルを使用してボット Microsoft Entra アプリ登録をプロビジョニングし、Bicep ファイルを使用して Azure AI Bot Service リソースをプロビジョニングします。

Visual Studio での続行:

1. プロジェクトのルート フォルダーで、**teamsapp.local.yml** を開きます
1. ファイルで、**botAadApp/create** アクションを使用する手順を見つけて、次のように置き換えます。

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: BOT_ID
          clientSecret: SECRET_BOT_PASSWORD
          objectId: BOT_ENTRA_APP_OBJECT_ID
          tenantId: BOT_ENTRA_APP_TENANT_ID
          authority: BOT_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: BOT_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.bot.manifest.json"
          outputFilePath : "./build/entra.bot.manifest.${{TEAMSFX_ENV}}.json"
    
      - uses: arm/deploy
        with:
          subscriptionId: ${{AZURE_SUBSCRIPTION_ID}}
          resourceGroupName: ${{AZURE_RESOURCE_GROUP_NAME}}
          templates:
            - path: ./infra/azure.local.bicep
              parameters: ./infra/azure.parameters.local.json
              deploymentName: Create-resources-for-${{APP_INTERNAL_NAME}}-${{TEAMSFX_ENV}}
          bicepCliVersion: v0.9.1
    ```

1. ファイルで、**botFramework/create** アクションを使用する手順を削除します
1. 変更を保存。

アプリの登録は 2 つの手順でプロビジョニングされます。まず、**aadApp/create** アクションはクライアント シークレットを使用して新しいマルチテナント アプリ登録を作成し、その出力を環境変数として **.env.local** ファイルに書き込みます。 その後、**aadApp/update** アクションは、**entra.bot.manifest.json** ファイルを使用してアプリの登録を更新します。

最後の手順では、**arm/deploy** アクションを使用して、**azure.parameters.local.json** ファイルと **azure.local.bicep** ファイルを使用して Azure AI Bot Service リソースをリソース グループにプロビジョニングします。

## タスク 3 - 開発トンネルを作成する

ユーザーがメッセージ拡張機能と対話すると、Bot サービスは Web サービスに要求を送信します。 開発中、Web サービスはマシン上でローカルに実行されます。 Bot Service が Web サービスに到達できるようにするには、開発トンネルを使用してマシン外に公開する必要があります。

:::image type="content" source="../media/18-select-dev-tunnel.png" alt-text="Visual Studio で展開された [開発トンネル] メニューのスクリーンショット。" lightbox="../media/18-select-dev-tunnel.png":::

Visual Studio での続行:

1. ツール バーで、** Microsoft Teams (ブラウザー) ボタンの横にあるドロップダウン**を選択して、デバッグ プロファイル メニューを展開します
1. **[Dev Tunnels (no active tunnel) (開発トンネル (アクティブなトンネルなし))]** メニューを展開し、**[Create a Tunnel... (トンネルの作成...)]** を選択します
1. ダイアログで、次の値を指定します。
    1. **アカウント**: 任意のアカウントを選択します
    1. **名前**: MsgExtProductSupport
    1. **トンネルの種類**: 一時的
    1. **アクセス**: パブリック
1. **[OK]** を選択してトンネルを作成します。新しいトンネルが現在アクティブなトンネルであることを示すプロンプトが表示されます
1. **[OK]** を選択してプロンプトを閉じます

## タスク 4 - アプリ マニフェストを更新する

アプリ マニフェストは、アプリの機能について説明します。 アプリ マニフェストのプロパティを更新して、アプリの機能とその機能をより適切に説明します。

まず、アプリ アイコンをダウンロードし、プロジェクトに追加します。

:::image type="content" source="../media/app/color-local.png" alt-text="ローカル開発に使用される色アイコン。" lightbox="../media/app/color-local.png":::

:::image type="content" source="../media/app/color-dev.png" alt-text="リモート開発に使用される色アイコン。" lightbox="../media/app/color-dev.png":::

1. **color-local.png** と **color-dev.png** をダウンロードします
1. **appPackage** フォルダーに、**color-local.png** と **color-dev.png** を追加します
1. フォルダーで、**color.png** という名前のファイルを削除します

アプリ名はプロジェクト内のさまざまな場所にレプリケートされるため、この値を一元的に格納する新しい環境変数を作成します。

Visual Studio での続行:

1. **env** フォルダーで、**.env.local** という名前のファイルを開きます
1.  ファイルに、次のコードを追加します。

    ```text
    APP_DISPLAY_NAME=Contoso products
    ```

1. 変更を保存します

最後に、アプリ マニフェスト ファイル内のアイコン、名前、および説明オブジェクトを更新します。

1. **appPackage** フォルダーで、**manifest.json** という名前のファイルを開きます。
1. ファイルで、**icon**、**name、および **description** オブジェクトを次の値で更新します。

    ```json
    {
        "icons": {
            "color": "color-${{TEAMSFX_ENV}}.png",
            "outline": "outline.png"
        },
        "name": {
            "short": "${{APP_DISPLAY_NAME}}",
            "full": "${{APP_DISPLAY_NAME}}"
        },
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation."
        }
    }
    ```

1. 変更を保存します

## タスク 6 - リソースをプロビジョニングする

Teams ツールキットを使用して、すべてが整った状態で、Teams アプリの依存関係の準備プロセスを実行して、必要なリソースをプロビジョニングします。

:::image type="content" source="../media/19-prepare-teams-app-dependencies.png" alt-text="Visual Studio で展開された Teams Toolkit メニューのスクリーンショット。" lightbox="../media/19-prepare-teams-app-dependencies.png":::

Teams アプリの依存関係の準備プロセスでは、アクティブな開発トンネル URL を使用して .env.local ファイル内の **BOT_ENDPOINT** と **BOT_DOMAIN** 環境変数を更新し、 **teamsapp.local.yml** ファイルで説明されているアクションを実行します。

Visual Studio での続行:

1. Solution Explorer で、**MsgExtProductSupport** プロジェクトを右クリックします
1. **Teams Toolkit** メニューを展開し、**[Prepare Teams App Dependencies (Teams アプリの依存関係を準備)]** を選択します
1. **Microsoft 365 アカウント**ダイアログで、開発者テナントのアカウントを選択し、**[続行] ** を選択します。
1. **[プロビジョニング]** ダイアログで、Azure へのリソースのデプロイに使用するアカウントを選択し、次の値を指定します。
    1. **サブスクリプション名**: 使用するサブスクリプションをドロップダウンから選択します
    1. **リソース グループ**: [新規...] を選択してダイアログを開き、「**rg-msgext-product-support-local」と入力し、**[OK]** を選択します
    1. **リージョン**: ドロップダウンで、最も近いリージョンを選択します
1. **[プロビジョニング]** を選択して Azure でリソースをプロビジョニングします
1. Teams Toolkit の警告プロンプトで、**[プロビジョニング]** を選択します
1. Teams Toolkit 情報プロンプトで、**[View provisioned resources (プロビジョニングされたリソースの表示)]** を選択して、新しいブラウザー ウィンドウを開きます。

少し時間を取って、Azure で作成されたリソースを調べてみましょう。

## タスク 7 - 実行とデバッグ  

次に、Web サービスを開始し、メッセージ拡張機能をテストします。 Teams Toolkit を使用してアプリ マニフェストをアップロードし、Microsoft Teamsでメッセージ拡張機能をテストします。

Visual Studio での続行:

1. F5 キーを押してデバッグ セッションを開始し、新しいブラウザー ウィンドウを開くと Microsoft Teams Web クライアントに移動されます。
1. メッセージが表示されたら、Microsoft 365  アカウント資格情報を入力します。

  > [!IMPORTANT]
  > Microsoft Teams に「This app cannot be found (このアプリが見つかりません)」というメッセージが含まれるダイアログ ボックスが表示された場合は、次の手順に従ってアプリ パッケージを手動でアップロードします
  >
  >  1. ダイアログを閉じます
  >  2. サイド バーで **[アプリ]** に移動します
  >  3. 左側のメニューで、** [Manage your apps (アプリの管理)]** を選択します
  >  4. コマンド バーで、**[ファイルのアップロード]** を選択します。
  >  5. ダイアログ ボックスで、**[Upload a customized app (カスタマイズしたアプリをアップロードする)]** を選択します
  >  6. エクスプローラーでソリューション フォルダーに移動し、**appPackage\build** フォルダーを開き、**appPackage.local.zip** を選択してから、**追加**します

続行してアプリをインストールします。

1. アプリのインストール ダイアログで、**[追加]** を選びます。
1. 新規または既存の Microsoft Teams チャットを開きます
1. メッセージ作成領域で、**[...] **を選択します アプリのポップアップが開きます
1. アプリの一覧で **C[ontoso 製品]** を選択してメッセージ拡張機能を開きます
1. テキスト ボックスに「 **Bot Builder** 」と入力して検索を開始します。
1. 結果の一覧で、作成メッセージ ボックスにカードを埋め込む結果を選択します

ブラウザーを閉じてデバッグ セッションを終了します。

[次の演習に進んでください...](./3-exercise-add-single-sign-on.md)