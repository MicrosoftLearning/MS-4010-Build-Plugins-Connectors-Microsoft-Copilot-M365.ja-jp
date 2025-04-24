---
lab:
  title: 演習 3 - 宣言型エージェントに会話スターターを追加する
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# 演習 3 - 宣言型エージェントに会話スターターを追加する

この演習では、ユーザーが質問できる質問の種類を理解するのに役立つサンプル プロンプトをユーザーに提供する会話スターターを含むように宣言型エージェントを更新します。

### 演習の期間

- **推定所要時間**: 5 分

## タスク 1 - 会話スターターを追加する

Visual Studio Code:

1. **appPackage** フォルダーで、**declarativeAgent.json** ファイルを開きます。
1. 次のコード スニペットをファイルに追加します。

   ```json
   "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
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
    ],
  "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
  ]
}
```

## タスク 2 - Microsoft 365 Copilot Chat で宣言型エージェントをテストする

次に、変更をアップロードし、デバッグ セッションを開始します。

Visual Studio Code:

1. **Activity Bar** で、**Teams Toolkit** 拡張機能を開きます。
1. **[Lifecycle]** セクションで、**[Provision]** と **[Publish]** を選択します。
1. **[Confirm]** をクリックして、アプリカタログのアップデートの送信を確認します。
1. アップロードが完了するまで待ちます。

Web ブラウザーでの続行:

1. **Microsoft 365 Copilot** で、右上のアイコンを選択して、**Copilot サイド パネル**を展開します。
1. エージェントの一覧から **Product support** を見つけ、それを選択して、イマーシブ エクスペリエンスを入力し、エージェントと直接チャットします。 マニフェストで定義した会話スターターがユーザー インターフェイスに表示されることに注目してください。

![Microsoft 365 Knowledge Expert の宣言型エージェントを、カスタム会話スターターを使用したイマーシブ エクスペリエンスでの製品サポート宣言型エージェントを示す Microsoft Edge のスクリーンショット。](../media/LAB_01/test-conversation-starters.png)

ブラウザーを閉じて、Visual Studio Code のデバッグ セッションを終了します。