---
lab:
  title: '演習 2: Microsoft 365 Copilot Chat で宣言型エージェントをテストする'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# 演習 2: Microsoft 365 Copilot Chat で宣言型エージェントをテストする

この演習では、宣言型エージェントをテストして Microsoft 365 に展開し、Microsoft 365 Copilot Chat を使用してテストします。

### 演習の期間

- **推定所要時間**: 10 分

## タスク 1: Microsoft 365 Copilot Chat の API プラグインを使用して宣言型エージェントをテストする

最後の手順では、Microsoft 365 Copilot の API プラグインを使用して宣言型エージェントをテストします。

Visual Studio Code:

1. Activity Bar で、**Teams Toolkit** 拡張機能を開きます。
1. **Teams Toolkit** 拡張機能パネルの **Accounts** セクションで、Microsoft 365 テナントにサインインしていることを確認します。

  ![Microsoft 365 への接続の状態を示す Teams Toolkit のスクリーンショット。](../media/LAB_05/3-teams-toolkit-account.png)

1. Activity Barで、[Run and Debug] ビューに切り替えます。
1. 構成の一覧から、**[Debug Copilot (Edge)]** を選択し、再生ボタンを押してデバッグを開始します。

  ![Visual Studio Code のデバッグ オプションのスクリーンショット。](../media/LAB_05/3-vs-code-debug.png)

  Visual Studio Code が、Microsoft 365 Copilot Chat を開いた状態の、新しい Web ブラウザーを開きます。 メッセージが表示されたら、Microsoft 365 アカウントでサインインします。

Web ブラウザーで以下を行います。

1. サイド パネルから、**da-repairs-keylocal** エージェントを選択します。

  ![Microsoft 365 Copilot に表示されているカスタム エージェントのスクリーンショット。](../media/LAB_05/3-copilot-agent-sidebar.png)

1. プロンプト テキスト ボックスに「`What repairs are assigned to Karin?`」と入力し、プロンプトを送信します。
1. **[Always allow]** ボタンを使用して、API プラグインにデータを送信することを確認します。

  ![API へのデータ送信を許可するプロンプトのスクリーンショット。](../media/LAB_05/3-allow-data.png)

1. エージェントが応答するまで待ちます。

  ![ユーザーのプロンプトに対するカスタム エージェントの応答のスクリーンショット。](../media/LAB_05/3-copilot-response.png)

テストが完了したら、Visual Studio Code でデバッグ セッションを停止します。