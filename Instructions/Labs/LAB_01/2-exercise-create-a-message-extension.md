---
lab:
  title: 演習 1 - ページ拡張機能を作成する
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 演習 1 - ページ拡張機能を作成する

この演習では、メッセージ拡張ソリューションを作成します。 Visual Studio で Teams Toolkit を使用して必要なリソースを作成し、デバッグ セッションを開始し、Microsoft Teams でテストします。

![Microsoft Teams の検索ベースのメッセージ拡張機能によって返される検索結果のスクリーンショット。](../media/1-search-results.png)

### 演習の期間

  - **推定所要時間**: 25 分

## タスク 1 - Teams Toolkit for Visual Studio を使用して新しいプロジェクトを作成する

まず、検索コマンドを含むメッセージ拡張機能で構成された新しい Microsoft Teams アプリ プロジェクトを作成します。 Teams Toolkit for Visual Studio プロジェクト テンプレートを使用してプロジェクトを作成することもできますが、このモジュールを完了するには、スキャフォールディングされたプロジェクトに変更を加える必要があります。 代わりに、NuGet パッケージとして使用できるカスタム プロジェクト テンプレートを使用します。 カスタム テンプレートを使用する利点は、必要なファイルと依存関係を使用してソリューションを作成し、時間を節約することです。

1. 管理者として新しい PowerShell セッションを開きます。

1. 次を実行して、適切な作業ディレクトリに変更します。

    ```Powershell
    cd ~\Documents
    ```

1. 次を実行して、NuGet からテンプレート パッケージをインストールすることから始めます。

    ```PowerShell
    dotnet new install M365Advocacy.Teams.Templates
    ```

1. 次を実行して新しいプロジェクトを作成します。

    ```PowerShell
    dotnet new teams-msgext-search --name "ProductsPlugin" `
      --internal-name "msgext-products" `
      --display-name "Contoso products" `
      --short-description "Product look up tool." `
      --full-description "Get real-time product information and share them in a conversation." `
      --command-id "Search" `
      --command-description "Find products by name" `
      --command-title "Products" `
      --parameter-name "ProductName" `
      --parameter-title "Product name" `
      --parameter-description "The name of the product as a keyword" `
      --allow-scripts Yes
    ```

1. プロジェクトが作成されるまで待ちます。

1. `cd ProductsPlugin` を実行して、プロジェクト ディレクトリに変更します。

1. `.\ProductsPlugin.sln` を実行して、Visual Studio でソリューションを開きます。

1. アプリケーションの選択ウィンドウから **Visual Studio 2022** を選択し、**[Always]** を選択します。

## タスク 2: 開発トンネルを作成する

ユーザーがメッセージ拡張機能と対話すると、Bot サービスは Web サービスに要求を送信します。 開発中、Web サービスはマシン上でローカルに実行されます。 Bot Service が Web サービスに到達できるようにするには、開発トンネルを使用してマシン外に公開する必要があります。

![Visual Studio の [Dev tunnels] ウィンドウのスクリーンショット。](../media/14-select-dev-tunnel.png)

Visual Studio での続行:

1. ツール バーの **[Start]** ボタンの横にあるドロップダウンを選択し、**[Dev Tunnels (no active tunnel)]** メニューを展開し、**[Create a Tunnel]** を選択します。

1. ダイアログで、次の値を指定します。

    1. **アカウント**: 提供された Microsoft 365 アカウントを使用してサインインします。 **[職場または学校アカウント]** を選択します。

    1. **名前**: msgext-products

    1. **トンネルの種類**: 一時的

    1. **アクセス**: パブリック

1. **[OK]** を選択してトンネルを作成します。 新しいトンネルが現在アクティブなトンネルであることを示すプロンプトが表示されます。

1. **[OK]** を選択してプロンプトを閉じます。

## タスク 3: リソースを準備する

Teams Toolkit を使用して、すべてが整った状態で、**Teams アプリの依存関係の準備**プロセスを実行して、必要なリソースを作成します。

![Visual Studio で展開された Teams Toolkit メニューのスクリーンショット。](../media/15-prepare-teams-app-dependencies.png)

Teams アプリの依存関係の準備プロセスでは、アクティブな開発トンネル URL を使用して **TeamsApp\\env\\.env.local** ファイル内の **BOT_ENDPOINT** と **BOT_DOMAIN** 環境変数を更新し、**TeamsApp\\teamsapp.local.yml** ファイルで説明されているアクションを実行します。

少し時間を取って、**teamsapp.local.yml** ファイルの手順を調べてみましょう。

Visual Studio での続行:

1. **[Project]** メニューを開き (または、Solution Explorer で **TeamsApp** プロジェクトを右クリックすることもできます)、**[Teams Toolkit]** メニューを展開して、**[Prepare Teams App Dependencies]** を選択します。

1. **[Microsoft 365 account]** ダイアログで、サインインするか、既存のアカウントを選択して Microsoft 365 テナントにアクセスして、**[Continue]** を選択します。

1. **[Provision]** ダイアログで、サインインするか、Azure へのリソースのデプロイに使用する既存のアカウントを選択して、次の値を指定します。

      1. **サブスクリプション名**: ドロップダウンを使用して、サブスクリプションを選択します。

      1. **リソース グループ**: ドロップダウン リストから、あらかじめ用意されたリソース グループを選択します。

1. **[Provision]** を選択して、Azure でリソースを作成します。

1. Teams Toolkit 警告プロンプトで、**[Provision]** を選択します。

1. Teams Toolkit 情報プロンプトで、**[View provisioned resources (プロビジョニングされたリソースの表示)]** を選択して、新しいブラウザー ウィンドウを開きます。

少し時間を取って、Azure で作成されたリソースを調べ、**.env.local** ファイルで作成された環境変数も表示します。

> [!NOTE]
> Visual Studio を閉じて再度開くと、開発トンネルの URL が変更され、アクティブなトンネルとして選択されなくなります。 この場合は、トンネルをもう一度選択し、**Teams アプリの依存関係の準備**プロセスを実行して、更新された URL をアプリ マニフェストに反映する必要があります。

## タスク 4 - 実行とデバッグ

Teams Toolkit では、複数プロジェクト起動プロファイルが使用されます。 プロジェクトを実行するには、Visual Studio でプレビュー機能を有効にする必要があります。

Visual Studio:

1. **[Tools]** メニューを開き、**[Options...]** を選択します。

1. 検索ボックスに、「**multi-project**」と入力します。

1. **[Environment]** で、**[Preview Features]** を選択します。

1. **[Enable Multi-Project Launch Profiles]** の横にあるチェック ボックスをオンにし、**[OK]** を選択して変更を保存します。

デバッグ セッションを開始し、Microsoft Teams にアプリをインストールするには:

1. <kbd>F5</kbd> キーを押すか、ツール バーから **[Start]** を選択します。

1. アプリを初めて起動するときにポップアップ表示される SSL 証明書の警告を信頼または承認します。

1. ブラウザー ウィンドウが開き、Microsoft Teams Web クライアントにアプリのインストール ダイアログが表示されるまで待ちます。 メッセージが表示されたら、Microsoft 365 アカウント資格情報を入力します。

1. [アプリのインストール] ダイアログで、**[追加]** を選択します。

メッセージ拡張機能をテストするには:

1. 新しいチャットを開き (<kbd>Alt + N</kbd>)、**[宛先]** ボックスに「**Contoso**」と入力し、**Contoso 製品サポート**を選択します。

    > [!NOTE]
    > 自分のユーザー アカウントとチャットする場合は機能しません。 別のユーザーまたはグループである必要があります。

1. メッセージ作成領域で、「**/apps**」と入力してアプリ ピッカーを開きます。

1. アプリの一覧で、**[Contoso 製品]** を選択してメッセージ拡張機能を開きます。

1. テキスト ボックスに、「** hello**」と入力します。 検索を複数回入力する必要がある場合があります。

1. 検索結果が表示されるまで待ちます。

1. 結果の一覧で、**hello** を選択して作成メッセージ ボックスにカードを埋め込みます。

![Microsoft Teams の検索ベースのメッセージ拡張機能によって返される検索結果のスクリーンショット。](../media/1-search-results.png)

Visual Studio に戻り、ツール バーから **[Stop]** を選択するか、<kbd>Shift</kbd> + <kbd>F5</kbd> キーを押してデバッグ セッションを停止します。

[次の演習に進んでください...](./3-exercise-add-single-sign-on.md)