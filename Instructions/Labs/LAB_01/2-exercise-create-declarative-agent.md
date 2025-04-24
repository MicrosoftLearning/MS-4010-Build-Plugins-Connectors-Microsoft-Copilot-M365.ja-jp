---
lab:
  title: 演習 1 - Visual Studio Code で宣言型エージェントを作成する
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# 演習 1 - 宣言型エージェントを作成する

この演習では、テンプレートから宣言型エージェント プロジェクトを作成し、マニフェストを更新し、エージェントを Microsoft 365 にアップロードして、Microsoft 365 Copilot でエージェントをテストします。 

宣言型エージェントは、Microsoft 365 アプリに実装されます。 次を含むアプリ パッケージを作成します。

- app.manifest.json: アプリ マニフェスト ファイルには、アプリの構成方法 (機能など) が記述されています。
- declarative-agent.json: 宣言型エージェント マニフェストには、宣言型エージェントの構成方法が記述されています。
- color.png と outline.png: Microsoft 365 Copilot ユーザー インターフェイスで宣言型エージェントを表すために使用される色とアウトラインのアイコン。

### 演習の期間

- **推定所要時間**: 15 分

## タスク 1 - Teams 管理センターでカスタム アプリのアップロードを有効にする

Teams Toolkit を使用して宣言型エージェントを Microsoft 365 にアップロードするには、Teams 管理センターで **[カスタム アプリのアップロード]** を有効にする必要があります。

1. Teams 管理センターで [Teams アプリ]、[アプリの設定ポリシー] の順に移動するか、[[アプリの設定ポリシー]](https://admin.teams.microsoft.com/policies/app-setup) に直接移動します。
1. ポリシーのリストから **[グローバル (組織全体の既定値)]** を選びます。
1. **[カスタム アプリのアップロード]** を有効にします。
1. **[保存]** を選択したあと、その選択を**確認**します。

## タスク 2 - スターター プロジェクトをダウンロードする

まず、Web ブラウザーで GitHub からサンプル プロジェクトをダウンロードします。

1. [https://github.com/microsoft/learn-declarative-agent-vscode](https://github.com/microsoft/learn-declarative-agent-vscode) テンプレート リポジトリに移動します。
    1. 手順に従って、コンピューターに[リポジトリのソース コードをダウンロード](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view)します。
    1. ダウンロードした ZIP ファイルの内容をコンピューター上の **Documents フォルダー**に展開します。

スターター プロジェクトには、宣言型エージェントを含む Teams Toolkit プロジェクトが含まれています。

1. Visual Studio Code で  プロジェクト フォルダーを開きます。
1. プロジェクトのルート フォルダーで、**README.md** ファイルを開きます。 プロジェクト構造の詳細については、内容を確認します。

![エクスプローラー ビューのスターター プロジェクトの Readme ファイルとフォルダー構造を示す Visual Studio Code のスクリーンショット。](../media/LAB_01/create-complete.png)

## タスク 3 - 宣言型エージェント マニフェストを調べる

宣言型エージェント マニフェスト ファイルを調べてみましょう。

- **appPackage/declarativeAgent.json** ファイルを開き、内容を確認します。

    ```json
    {
        "$schema": "https://aka.ms/json-schemas/agent/declarative-agent/v1.0/schema.json",
        "version": "v1.0",
        "name": "da-product-support",
        "description": "Declarative agent created with Teams Toolkit",
        "instructions": "$[file('instruction.txt')]"
    }
    ```

**instructions** プロパティの値には、**instruction.txt** という名前のファイルへの参照が含まれています。 **$[file(path)]** 関数は Teams Toolkit によって提供されます。 **instruction.txt** の内容は、Microsoft 365 にプロビジョニングされるときに宣言型エージェント マニフェスト ファイルに含まれます。

- **appPackage** フォルダーで、**instruction.txt** ファイルを開き、内容を確認します。

    ```md
    You are a declarative agent and were created with Team Toolkit. You should start every response and answer to the user with "Thanks for using Teams Toolkit to create your declarative agent!\n\n" and then answer the questions and help the user.
    ```

## タスク 4 - 宣言型エージェント マニフェストを更新する

**name** プロパティと **description** プロパティを、このシナリオに関連するように更新しましょう。

1. **appPackage** フォルダーで、**declarativeAgent.json** ファイルを開きます。
1. **name** プロパティ値を **Microsoft 365 Knowledge Expert** に更新します。
1. **description** プロパティ値を **Microsoft 365 に関するどんな質問にも回答できる Microsoft 365 Knowledge Expert** に更新します。
1. 変更を保存します

更新されたファイルの内容は次のとおりです。

```json
{
    "$schema": "https://aka.ms/json-schemas/agent/declarative-agent/v1.0/schema.json",
    "version": "v1.0",
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]"
}
```

## タスク 5 - 宣言型エージェントを Microsoft 365 にアップロードする

次に、宣言型エージェントを Microsoft 365 テナントにアップロードします。

Visual Studio Code:

1. **Activity Bar** で、**Teams Toolkit** 拡張機能を開きます。

    ![Visual Studio Code のスクリーンショット。 Teams Toolkit アイコンがアクティビティ バーで強調表示されています。](../media/LAB_01/teams-toolkit-open.png)

1. **[Lifecycle]** セクションで、**[Provision]** を選択します。

    ![Teams Toolkit ビューを示す Visual Studio Code の のスクリーンショット。 [Lifecycle] セクションで 'Provision' 関数が強調表示されています。](../media/LAB_01/provision.png)

1. プロンプトで、**[Sign in]** を選択し、プロンプトに従って Teams Toolkit を使用して Microsoft 365 テナントにサインインします。 プロビジョニング プロセスは、サインイン後に自動的に開始されます。

    ![ユーザーに Microsoft 365 へのサインインを求める Visual Studio Code からのプロンプトのスクリーンショット。 [Sign in] ボタンが強調表示されています。](../media/LAB_01/provision-sign-in.png)

    ![プロビジョニング プロセスが進行中であることを示す Visual Studio Code のスクリーンショット。 プロビジョニング中のメッセージが強調表示されています。](../media/LAB_01/provision-in-progress.png)

1. アップロードが完了するまで待ってから次に進みます。

    ![プロビジョニング プロセスが完了したことを確認するトースト通知を示す Visual Studio Code のスクリーンショット。 トースト通知が強調表示されます。](../media/LAB_01/provision-complete.png)

次に、プロビジョニング プロセスの出力を確認します。

- **appPackage/build** フォルダーで、**declarativeAgent.dev.json** ファイルを開きます。

**instructions** プロパティの値に、**instruction.txt** ファイルの内容が含まれていることに注目してください。 **declarativeAgent.dev.json** ファイルは、**manifest.dev.json**、**color.png**、**outline.png** の各ファイルと共に、**appPackage.dev.zip** ファイルに含まれています。 **appPackage.dev.zip** ファイルは Microsoft 365 にアップロードされます。

> [!IMPORTANT]
> Microsoft 365 アカウントにログインすると、Visual Studio Code に次の警告またはエラー メッセージが表示されることがあります。 Microsoft Teams でカスタム アプリのアップロードを有効にしたばかりの場合は、設定が有効になるまでに時間がかかる場合があります。  数分待ってからもう一度試すか、ログアウトして Microsoft 365 アカウントでログインし直してください。 テナントには完全な Copilot ライセンスがないので、Microsoft 365 Copilot アクセスに関する 2 番目のメッセージが予想されます。
> 
> ![Visual Studio Code の警告のスクリーンショット。](../media/LAB_01/ttk-login-errors.png)

## タスク 6 - Microsoft 365 Copilot Chat で宣言型エージェントをテストする

次に、Microsoft 365 Copilot Chat で宣言型エージェントを実行し、その機能を検証してみましょう。

1. **Activity Bar** で、**Teams Toolkit** 拡張機能を開きます。

    ![Visual Studio Code のスクリーンショット。 Teams Toolkit アイコンがアクティビティ バーで強調表示されています。](../media/LAB_01/teams-toolkit-open.png)

1. **[ライフサイクル]** セクションで、**[発行]** を選択します。 アクションが完了するまで待ちます。

1. Microsoft Edge を開き、Microsoft 365 Copilot Chat ([https://www.microsoft365.com/chat](https://www.microsoft365.com/chat)) に移動します。

1. **Microsoft 365 Copilot Chat** で、右上にあるアイコンを選択して、Copilot サイド パネルを展開します。 パネルに最近のチャットと利用可能なエージェントが表示されることに注目してください。

1. サイド パネルで、**[Microsoft 365 Knowledge Expert]** を選択してイマーシブ エクスペリエンスに入り、エージェントと直接チャットします。

1. エージェントに「**What can you do?**」と質問し、プロンプトを送信します。

    ![Microsoft 365 Copilot を示す Microsoft Edge のスクリーンショット。 サイド パネルを開くアイコンと、パネル内の製品サポート エージェントが強調表示されています。](../media/LAB_01/test-immersive-side-panel.png)

次の演習に進んでください。