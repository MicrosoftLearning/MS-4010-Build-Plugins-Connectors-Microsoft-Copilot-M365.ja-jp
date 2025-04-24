---
lab:
  title: '演習 4: Microsoft 365 Copilot Chat で宣言型エージェントをテストする'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# 演習 34: Microsoft 365 Copilot で宣言型エージェントをテストする

この演習では、宣言型エージェントをテストして Microsoft 365 に展開し、Microsoft 365 Copilot Chat を使用してテストします。

### 演習の期間

- **推定所要時間**: 5 分

## タスク 1: Microsoft 365 Copilot の API プラグインを使用して宣言型エージェントをテストする

最後の手順では、Microsoft 365 Copilot の API プラグインを使用して宣言型エージェントをテストします。

Visual Studio Code:

1. Activity Bar で、**Teams Toolkit** 拡張機能を開きます。
1. **Teams Toolkit** 拡張機能パネルの **Accounts** セクションで、Microsoft 365 テナントにサインインしていることを確認します。

    ![Microsoft 365 への接続の状態を示す Teams Toolkit のスクリーンショット。](../media/LAB_05/3-teams-toolkit-account.png)

1. Activity Barで、**[Run and Debug]** ビューに切り替えます。
1. 構成の一覧から、**[Debug Copilot (Edge)]** を選択し、再生ボタンを押してデバッグを開始します。

    ![Visual Studio Code のデバッグ オプションのスクリーンショット。](../media/LAB_05/3-vs-code-debug.png)

    Visual Studio Code によって、Microsoft 365 Copilot で新しい Web ブラウザーが開きます。 メッセージが表示されたら、Microsoft 365 アカウントでサインインします。

Web ブラウザーで以下を行います。

1. サイド パネルで、**[da-repairs-oauthlocal]** エージェントを選択します。

    ![Microsoft 365 Copilot に表示されているカスタム エージェントのスクリーンショット。](../media/LAB_05/5-copilot-agent-sidebar.png)

1. プロンプト テキスト ボックスに `Show repair records assigned to Karin Blair` を入力し、プロンプトを送信します。

    > [!TIP]
    > プロンプトを入力する代わりに、会話スターターから選択できます。

    ![カスタム宣言型エージェントで開始された会話のスクリーンショット。](../media/LAB_05/5-conversation-starter.png)

1. **[Always allow]** ボタンを使用して、API プラグインにデータを送信することを確認します。

    ![API へのデータ送信を許可するプロンプトのスクリーンショット。](../media/LAB_05/5-allow-data.png)

1. メッセージが表示されたら、**[Sign in to da-repairs-oauthlocal]** を選択し、API にサインインして、microsoft 365 テナントへのサインインに使用するアカウントと同じアカウントを引き続き使用します。

    ![API をセキュリティで保護するアプリへのサインインを促すプロンプトのスクリーンショット。](../media/LAB_05/5-sign-in.png)

1. エージェントが応答するまで待ちます。

    ![ユーザーのプロンプトに対する宣言型エージェントの応答のスクリーンショット。](../media/LAB_05/5-agent-response.png)

API はローカル コンピューター上で実行されているため、匿名でアクセスできますが、Microsoft 365 Copilot は API 仕様で指定されているように認証された API を呼び出しています。要求にアクセス トークンが含まれていることを確認するには、**repairs** 関数にブレークポイントを設定し、宣言型エージェントで別のプロンプトを送信します。 コードがブレークポイントに到達したら、req.headers コレクションを展開し、JSON Web Token (JWT) を含む Authorization ヘッダーを探します。

![ブレークポイントと、着信要求の Authorization ヘッダーを示すデバッグ パネルを含む Visual Studio Code のスクリーンショット。](../media/LAB_05/5-vs-code-breakpoint-jwt.png)

テストが完了したら、Visual Studio Code でデバッグ セッションを停止します。
