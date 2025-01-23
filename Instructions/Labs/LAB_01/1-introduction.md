---
lab:
  title: 導入
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# はじめに

宣言型エージェントを使用して Microsoft 365 Copilot を拡張します。 カスタム ナレッジを定義して、権限のあるコンテンツを使用して質問に回答できるエージェントを作成します。

## シナリオ例

たとえば、カスタマー サポート チームで働いているとします。 あなたとチームは、組織が顧客から作成した製品に関する問い合わせを処理します。 応答時間を改善し、より良いエクスペリエンスを提供する必要があります。 ドキュメントは、製品の仕様、よく寄せられる質問、および修理、返品、保証の取り扱いに関するポリシーを含む SharePoint Online サイトのドキュメント ライブラリに格納します。 自然言語を使用してこれらのドキュメント内の情報を照会し、顧客の問い合わせに対する回答をすばやく取得できるようにする必要があります。

## 学習内容

ここでは、Microsoft 365 のドキュメントに格納されている情報を使用して、製品サポートの質問に回答できる宣言型エージェントを作成します。

- **作成**: 宣言型エージェント プロジェクトを作成し、Visual Studio Code で Teams ツールキットを使用します。
- **カスタム指示**: カスタム指示を定義して応答を整形します。
- **カスタム グラウンディング**: グラウンディング データを構成して、エージェントにさらにコンテキストを追加します。
- **会話スターター**: 新しい会話を開始するためのプロンプトを定義します。
- **プロビジョニング**: 宣言型エージェントを Microsoft 365 Copilot にアップロードし、結果を検証します。

## 前提条件

- Microsoft 365 Copilot とは何か、およびそのしくみについての基本的な知識
- Microsoft 365 Copilot の宣言型エージェントとは何かについての基本的な知識
- Microsoft 365 Copilotを使用した Microsoft 365 テナント
- カスタム アプリを Microsoft Teams にアップロードする権限を持つアカウント
- Microsoft 365 Copilotを使用した Microsoft 365 テナントへのアクセス
- [Teams Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) 拡張機能がインストールされた [Visual Studio Code](https://code.visualstudio.com/)
- [Node.js v18](https://nodejs.org/en/download/package-manager)

## ラボの期間

- **推定所要時間**: 30 分

## 学習の目的

このモジュールを完了すると、宣言型エージェントを作成し、Microsoft 365 にアップロードしてから、Microsoft 365 Copilot で使用して結果を検証できるようになります。

開始する準備ができたら、[最初の演習に進んでください...](./2-exercise-create-declarative-agent.md)
