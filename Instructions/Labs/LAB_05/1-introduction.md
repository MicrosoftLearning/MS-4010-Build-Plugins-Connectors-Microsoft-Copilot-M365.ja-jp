---
lab:
  title: 導入
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# はじめに

Microsoft 365 Copilot エージェントを使用すると、特定のシナリオに最適化された AI を利用したアシスタントを作成できます。 手順を使用して、エージェントのコンテキストを定義し、話し方や応答方法などを設定します。 エージェントを API と統合すれば、エージェントが外部アプリケーションに接続してデータを取得し、アクションを実行できるようになります。

## シナリオ例

あなたは自動車修理工場で働いているとします。 組織は、特殊なシステムを使用してさまざまな修復要求を追跡します。 あなたと同僚は、修理に関するさまざまな情報を定期的に検索します。 現在のシステムには、一致するキーワードのみを修復する基本的な検索機能が用意されています。 自然言語を使用して尋ねられた修復に関する質問に回答できる、AI を利用したアシスタントを用意したいと考えています。

## 学習の目的

このモジュールを終了すると、宣言型エージェントとセキュリティで保護された API を統合できるようになります。 API キーや OAuth で保護された API に、宣言型エージェントを安全に接続する方法について説明します。

## 前提条件

- Microsoft 365 Copilot の概要と動作についての初心者レベルの知識
- Microsoft 365 Copilot の宣言型エージェントを構築する方法についての知識
- Graph コネクタ を構築する方法についての知識
- Microsoft 365 Copilot の Microsoft 365 テナントとテナント管理者の特権
- [Teams Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) 拡張機能がインストールされた [Visual Studio Code](https://code.visualstudio.com/)
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
- [Node.js v18](https://nodejs.org/)
