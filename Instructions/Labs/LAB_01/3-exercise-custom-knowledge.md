---
lab:
  title: 演習 2 - カスタム ナレッジを構成する
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# 演習 2 - カスタム ナレッジを構成する

この演習では、エージェントのグラウンディング ソースとして Microsoft Learn を利用した学習を行います。 この演習を受講することで、エージェントは Microsoft 365 のエキスパートになるでしょう。

### 演習の期間

- **推定所要時間**: 10 分

## タスク 1 - グラウンディング データを構成する

宣言型エージェント マニフェストで、OneDrive フォルダーをグラウンディング データのソースとして構成します。

Visual Studio Code:

1. **appPackage** フォルダーで、**declarativeAgent.json** ファイルを開きます。
1. ファイル内の **[instructions]** 定義の後に以下のコード スニペットを追加します。**{URL}** は、Microsoft Learn の Microsoft 365 ランディング ページに直接リンクする URL に置き換えます。

    ```json
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "{URL}"
                }
            ]
        }
    ]
    ```

1. 変更を保存。

**declarativeAgent.json** ファイルは、次のようになります。

```json
{
    "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
    "version": "v1.0",
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]",
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "https://learn.microsoft.com/microsoft-365/"
                }
            ]
        }
    ]
}
```

## タスク 3 - カスタム指示を更新する

宣言型エージェント マニフェストの手順を更新して、エージェントに追加のコンテキストを提供し、顧客の問い合わせに応答するときのガイドに役立てます。

Visual Studio Code:

1. **appPackage/instruction.txt** ファイルを開き、次を使用して内容を更新します。

    ```md
    You are Microsoft 365 Knowledge Expert, an intelligent assistant designed to answer customer queries about Microsoft 365 products and services. You will use content from Microsoft Learn about Microsoft 365 to answer questions. If you can't find the necessary information, you should suggest that the agent should reach out to the team responsible for further assistance. Your responses should be concise and always include a cited source.
    ```

1. 変更を保存。

## タスク 4 - 宣言型エージェントを Microsoft 365 にアップロードする

変更内容を Microsoft 365 にアップロードします。

Visual Studio Code:

1. **Activity Bar** で、**Teams Toolkit** 拡張機能を開きます。
1. **[Lifecycle]** セクションで、**[Provision]** と **[Publish]** を選択します。
1. **[Confirm]** をクリックして、アプリカタログのアップデートの送信を確認します。
1. 公開タスクが完了するまで待ちます。

## タスク 5 - Microsoft 365 Copilot で宣言型エージェントをテストする

宣言型エージェントを Microsoft 365 でテストし、結果を検証します。 ブラウザー内で前の演習からそのまま続行し、ウィンドウを更新します (**F5**)

まず、手順をテストしてみましょう。

1. **Microsoft 365 Copilot** で、右上のアイコンを選択して、**Copilot サイド パネル**を展開します。
1. エージェントの一覧から **[Microsoft 365 Knowledge Expert]** を選択してイマーシブ エクスペリエンスを開き、エージェントと直接チャットします。
1. 製品サポート エージェントに対する質問として「**What can you do？**」と入力し、プロンプトを送信します。
1. 応答を待ちます。 応答が前の手順とどのように異なり、新しい手順が反映されているかに注目してください。

    ![Microsoft 365 Copilot を示す Microsoft Edge のスクリーンショット。 Microsoft 365 Knowledge Expert エージェントの機能を説明する応答が表示されます。](../media/LAB_01/test-m365-knowledge-expert.png)

次に、グラウンディング データをテストしてみましょう。

1. メッセージ ボックスに「**Tell me about Information Protection**」と入力し、メッセージを送信します。
1. 応答を待ちます。 情報保護に関する情報が応答に含まれていることに注意してください。 応答には、応答の生成に使用された特定の Web サイトへの引用や参照が含まれています。

    ![Microsoft 365 Copilot を示す Microsoft Edge のスクリーンショット。 Microsoft 365 Knowledge Expert エージェントからの応答が、Microsoft 365 の Information Protection に関する情報と共に表示されます。](../media/LAB_01/test-m365-knowledge-expert-1.png)

いくつかのプロンプトを試してみましょう。

1. メッセージ ボックスに、「**リアルタイム通信に適した製品を推奨する**」と入力します。
1. メッセージ ボックスに、「**Microsoft 365 のサポート オプションについて教えてください**」と入力します。

ブラウザーを閉じて、Visual Studio Code のデバッグ セッションを終了します。
