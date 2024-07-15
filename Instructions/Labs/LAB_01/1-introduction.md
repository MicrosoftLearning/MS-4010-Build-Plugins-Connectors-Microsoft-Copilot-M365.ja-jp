---
lab:
  title: 導入
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# はじめに

メッセージ拡張機能を使用すると、ユーザーは Microsoft Teams および Microsoft Outlook の外部システムを操作できます。 ユーザーは、メッセージ拡張機能を使用してデータを検索および変更し、これらのシステムの情報をメッセージやメールでリッチフォーマットカードとして共有できます。

組織に関連する最新の製品情報を含む SharePoint Online リストがあるとします。 この情報を Microsoft 365 で検索して共有したいと考えています。 また、Copilot for Microsoft 365 でこの情報を回答で使用することもできます。

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="製品サポート SharePoint Online チーム サイトのホームページのスクリーンショット。最近リリースされた製品の一覧が表示されます。" lightbox="../media/1-sharepoint-online-product-support-site.png":::

このモジュールでは、メッセージ拡張機能を作成します。 メッセージ拡張機能では、ボットを使用して、Microsoft Teams、Microsoft Outlook、および Copilot for Microsoft 365 と通信します。

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Microsoft Teams の検索ベースのメッセージ拡張機能によって返される検索結果のスクリーンショット。" lightbox="../media/2-search-results-nuget.png":::

Microsoft Entra を使用してユーザーを認証し、ユーザーに代わって Microsoft Graph API を使用して SharePoint Online からデータを返すことができます。

:::image type="content" source="../media/3-sign-in.png" alt-text="検索ベースのメッセージ拡張機能での認証チャレンジのスクリーンショット。サインインへのリンクが表示されます。" lightbox="../media/3-sign-in.png":::

ユーザーが認証されると、メッセージ拡張機能は Microsoft Graph API を使用して SharePoint Online から製品情報を取得します。 メッセージやメールにリッチフォーマットされたカードとして埋め込んで共有できる検索結果が返されます。

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Microsoft Teams の検索ベースのメッセージ拡張機能によって返される検索結果のスクリーンショット。検索結果は SharePoint Online から返されます。各検索結果には、製品名、カテゴリ、製品イメージが表示されます。" lightbox="../media/4-search-results-sharepoint-online.png":::

:::image type="content" source="../media/5-adaptive-card.png" alt-text="Microsoft Teams のメッセージに埋め込まれている検索結果のスクリーンショット。検索結果は、製品名、カテゴリ、通話量、リリース日を含むアダプティブ カードとしてレンダリングされます。タイトル ビューが表示されたアクション ボタンが表示され、ユーザーは SharePoint Online の製品リスト項目に移動できます。" lightbox="../media/5-adaptive-card.png":::

これはプラグインとして Copilot for Microsoft 365 と連携し、ユーザーに代わって SharePoint Online リストに対してクエリを実行し、返されたデータをその回答で使用できるようにします。

:::image type="content" source="../media/6-copilot-answer.png" alt-text="メッセージ拡張機能プラグインによって返される情報を含む Copilot for Microsoft 365 の回答のスクリーンショット。製品情報を示すアダプティブ カードが表示されます。" lightbox="../media/6-copilot-answer.png":::

このモジュールを終了するまでには、C# (.NET で実行) で記述されたメッセージ拡張機能を作成できます。 これは、Microsoft Teams、Microsoft Outlook、Copilot for Microsoft 365 で使用できます。 保護された API の背後にあるデータに対してクエリを実行し、結果をリッチ フォーマット カードとして返すことができます。

## 前提条件

- C# の基本的な知識
- Bicep の基本的な知識
- 認証に関する基本的な知識
- Microsoft 365 テナントへの管理者アクセス権
- Azure サブスクリプションへのアクセス
- Microsoft 365 用 Copilot へのアクセスは省略可能であり、1 つの演習を完了するためにのみ必要です
- Visual Studio 2022 17.9 と [Teams Toolkit](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) (Microsoft Teams 開発ツール コンポーネント) がインストールされている
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

構成を開始する準備ができたら、[次の演習に進んでください...](./2-exercise-create-a-message-extension.md)
