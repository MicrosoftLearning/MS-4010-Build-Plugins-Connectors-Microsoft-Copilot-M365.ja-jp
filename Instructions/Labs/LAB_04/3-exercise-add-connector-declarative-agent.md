---
lab:
  title: '演習 2: 宣言型エージェントを作成し、Graph コネクタを統合する'
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# 演習 2: 宣言型エージェントを作成し、Graph コネクタを統合する

この演習では、新しい宣言型エージェントをゼロから作成し、外部接続をグラウンディング ソースとして使用するように構成します。

### 演習の期間

- **推定所要時間**: 10 分

## タスク 1: 新しい宣言型エージェントを作成する

宣言型エージェントを作成する 1 つの方法は、Teams Toolkit を使用することです。 Teams ツールキットには、宣言型エージェントを作成するためのテンプレート プロジェクトが用意されており、エージェントの設定や、追加機能の追加からはじめるのに最適な場所です。

宣言型エージェントを作成するために、Visual Studio Code を開きます。

Visual Studio Code:

1. **Activity Bar** (サイド バー) で、**[Teams Toolkit]** 拡張機能を開きます。
1. Teams Toolkit ペインで、**[新しいアプリの作成]** ボタンを選択します。
1. **[新しいプロジェクト]** ダイアログ ボックスで、**[Agent]** オプションを選択します。
1. 次のダイアログで、**[宣言型エージェント]** オプションを選択します。
1. **[プラグインがありません]** オプションを選択して、プラグインを追加しないでください。
1. コンピューターにプロジェクトを保存するフォルダーを選択します。
1. プロジェクトに `da-it-policies` という名前を付けます。

## タスク 2: 宣言型エージェントの手順を構成する

Teams Toolkit は、新しい宣言型エージェント プロジェクトを作成します。 シナリオにスコープを設定するには、エージェントの説明と手順を更新します。

Visual Studio Code:

1. **appPackage/declarativeAgent.json** ファイルを開きます。
1. **name** プロパティの値を `Contoso IT policies` に更新します。
1. **description** プロパティの値を `Assistant specialized in Contoso IT policies` に更新します。
1. 変更を保存。
1. 更新されたファイルの内容は次のようになります。

    ```json
    {
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
    }
    ```

1. **appPackage/instruction.txt** ファイルを開きます。
1. 内容を次の内容に更新します。

    ```text
    You are a helpful assistant that can answer questions from users in a friendly manner. You should take your time to respond. Your responses should be accurate and helpful. If you don't have the information, you should say so in your response. When answering follow-up questions, you should review the information you gathered from external sources, if you don't already have the information to give an accurate answer, you should search for more information. Only answer when you have the information to give an accurate response.
    ```

1. 変更を保存。

## タスク 3: Microsoft Graph コネクタを宣言型エージェントと統合する

宣言型エージェントを作成した後の次の手順は、外部データにアクセスできるように Microsoft Graph コネクタと統合することです。

Visual Studio Code:

1. **appPackage/declarativeAgent.json** ファイルを開きます。
1. **instructions** プロパティの後に、次のコードで **capabilities** という名前の新しいプロパティを追加します。

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": ""
            }
          ]
        }
      ]
    } 
    ```

1. **connection_id** プロパティで、外部接続の ID として `policieslocal` を指定します。 `policieslocal` は、Graph コネクタが前の手順で作成した外部接続の ID です。
1. 更新されたファイルの内容は次のようになります。

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": "policieslocal"
            }
          ]
        }
      ]
    } 
    ```

1. 変更を保存。
