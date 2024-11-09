---
lab:
  title: 導入
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# はじめに

メッセージ拡張機能を使用すると、ユーザーは Microsoft Teams および Microsoft Outlook の外部システムを操作できます。 ユーザーは、メッセージ拡張機能を使用してデータを検索および変更し、これらのシステムのデータをメッセージやメールでリッチ フォーマット カードとして共有できます。

組織に関連する最新の製品情報にアクセスするために使用するカスタム API があるとします。 この情報を Microsoft 365 で検索して共有したいと考えています。 また、Copilot for Microsoft 365 でこの情報を回答で使用することもできます。

このモジュールでは、メッセージ拡張機能を作成します。 メッセージ拡張機能では、ボットを使用して、Microsoft Teams、Microsoft Outlook、および Copilot for Microsoft 365 と通信します。

![Microsoft Teams の検索ベースのメッセージ拡張機能によって返される検索結果のスクリーンショット。](../media/1-search-results.png)

Microsoft Entra を使用してユーザーを認証し、ユーザーに代わって API を使用してデータを返すことができます。

ユーザーが認証されると、メッセージ拡張機能は API からデータを取得し、メッセージやメールにリッチ フォーマット カードとして埋め込める検索結果を返します。

![Microsoft Teams の外部 API からのデータを使用する検索結果のスクリーンショット。](../media/3-search-results-api.png)

![Microsoft Teams のメッセージに埋め込まれている検索結果のスクリーンショット。](../media/4-adaptive-card.png)

これはプラグインとして Copilot for Microsoft 365 と連携し、ユーザーに代わって製品データに対してクエリを実行し、返されたデータをその回答で使用できるようにします。

![メッセージ拡張機能プラグインによって返される情報を含む Copilot for Microsoft 365 の回答のスクリーンショット。 製品情報を示すアダプティブ カードが表示されます。](../media/5-copilot-answer.png)

このモジュールを終了するまでには、C# (.NET で実行) で記述されたメッセージ拡張機能を作成できます。 これは、Microsoft Teams、Microsoft Outlook、Copilot for Microsoft 365 で使用できます。 保護された API の背後にあるデータに対してクエリを実行し、結果をリッチ フォーマット カードとして返すことができます。

## 前提条件

- C# の基本的な知識
- Bicep の基本的な知識
- 認証に関する基本的な知識
- Microsoft 365 テナントへの管理者アクセス権
- Azure サブスクリプションへのアクセス
- Copilot for Microsoft 365 へのアクセスは省略可能であり、**演習 4: タスク 5** を完了するためにのみ必要です。
- Visual Studio 2022 17.10 以降と [Teams Toolkit](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) (Microsoft Teams 開発ツール コンポーネント) がインストールされている
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Dev Proxy 0.19.1 以降](https://aka.ms/devproxy)

> [!NOTE]
> Microsoft 365 Copilot ライセンスを必要とするこのラボの唯一の演習は、**演習 4: タスク 5** です。 テナントに Copilot があるかどうかに関係なく、その時点までのすべてを行う必要があります。

## ラボの期間

  - **推定所要時間**: 150 分

## 学習の目的

このモジュールを完了すると、次のことができるようになります。

- メッセージ拡張機能とは何か、およびそれらを構築する方法について理解します。
- メッセージ拡張機能を作成します。
- シングル サインオンを使用してユーザーを認証し、Microsoft Entra 認証で保護されたカスタム API を呼び出す方法について理解します。
- Copilot for Microsoft 365 で使用するメッセージ拡張機能を拡張して最適化する方法について理解します。

開始する準備ができたら、[最初の演習に進んでください...](./2-exercise-create-a-message-extension.md)
