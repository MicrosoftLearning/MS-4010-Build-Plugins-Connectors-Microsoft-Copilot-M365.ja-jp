---
lab:
  title: 導入
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# はじめに

ナレッジ ベース記事を保存する外部システムがあるとします。 これらの記事には、組織内のさまざまなプロセスに関する情報が含まれています。 Microsoft 365 から関連情報を簡単に見つけて検出できるようにする必要があります。 また、Copilot for Microsoft 365 に、これらのサポート情報記事の情報を応答に含めたいと思っています。

Microsoft 365 内でこの外部情報を公開するには、カスタム Microsoft Graph コネクタを構築します。 Microsoft Graph コネクタは、外部システムに接続 (1) してコンテンツを取得し、Microsoft Entra ID から情報を使用して Microsoft 365 で認証 (2) し、Microsoft Graph API を使用して Microsoft 365 にコンテンツをインポート (3) します。

:::image type="content" source="../media/1-graph-connector-concept.png" alt-text="Microsoft Graph コネクタの概念的な動作を示す図。":::

このモジュールでは、Microsoft Graph コネクタの概要と、組織内での使用を検討する必要がある理由について説明します。 ローカル マークダウン ファイルを Microsoft 365 にインポートする Microsoft Graph コネクタを構築します。 また、インポートする外部コンテンツが、適切に割り当てられたアクセス許可を持つ個人のみがアクセスできるようにする方法についても説明します。 最後に、Microsoft Graph コネクタを最適化して、Microsoft 365 用 Copilot で使用します。

## 前提条件

- C# の基本的な知識
- 認証に関する基本的な知識
- [Microsoft 365 開発者テナント](https://developer.microsoft.com/microsoft-365/dev-program?ocid=MSlearn)へのアクセス
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

開始する準備ができたら、[次の演習に進んでください...](./2-exercise-configure-connection-schema.md)