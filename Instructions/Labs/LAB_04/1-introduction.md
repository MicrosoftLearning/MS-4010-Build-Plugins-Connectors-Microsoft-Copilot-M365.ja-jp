---
lab:
  title: 導入
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# はじめに

Microsoft 365 Copilot エージェントを使用すると、特定のシナリオに最適化された AI を利用したアシスタントを作成できます。 手順を使用して、コパイロットのコンテキストを定義し、話し方や応答方法などの設定を指定します。 エージェントのナレッジを構成し、大規模言語モデル (LLM) の一部ではない外部データへのアクセス権をエージェントに付与して、より正確に応答できるようにします。 

## シナリオ例

大規模な組織の IT 部門で働いているとします。 組織は、特殊なシステムに格納されているさまざまなポリシーを通じて IT を標準化します。 IT 部門のあなたと同僚は、ポリシーで説明されている質問を定期的に受け取ります。 ポリシー管理システムで回答を調べることには時間がかかります。 ポリシーの権限のある情報を使用して同僚の質問に回答できる AI アシスタントを組織に提供したいと考えています。

## 学習の目的

このモジュールを終了すると、Microsoft 365 Copilot 用の宣言型エージェントを構築できるようになります。 特定のシナリオに合わせて指示を最適化するように構成する方法について説明します。 また、Microsoft Graph コネクタと統合して外部データにアクセスできるようにする方法についても説明します。これは Microsoft 365 Copilot の LLM の一部ではありません。

## 前提条件

- Microsoft 365 Copilot の概要と動作についての初心者レベルの知識
- Microsoft 365 Copilot の宣言型エージェントを構築する方法についての知識
- Graph コネクタ を構築する方法についての知識
- Microsoft 365 Copilot の Microsoft 365 テナントとテナント管理者の特権
- [Teams Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) 拡張機能がインストールされた [Visual Studio Code](https://code.visualstudio.com/)
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
- [Node.js v18](https://nodejs.org/)
