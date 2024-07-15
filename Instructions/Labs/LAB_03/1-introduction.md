---
lab:
  title: 導入
  module: 'LAB 03: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# はじめに

このプロジェクトでは、Microsoft Copilot for Microsoft 365 のプラグインとして Teams メッセージ拡張機能を使用する方法について説明します。 このプロジェクトは、この同じ [GitHub リポジトリ](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/tree/main/samples/msgext-northwind-inventory-ts)に含まれる「Northwind Inventory」サンプルに基づいています。 古い [Northwind データベース](https://learn.microsoft.com/dotnet/framework/data/adonet/sql/linq/downloading-sample-databases)を使用すると、シミュレートされたエンタープライズ データを多数使用できます。

Northwind は、ワシントン州スポーカン市から特殊食品の e コマースビジネスを運営しています。 この課題では、製品在庫と財務情報へのアクセスを提供する Northwind Inventory アプリケーションを使用します。

この演習の所要時間は約 **60** 分です。

## 開始する前に

- まず、開発環境を設定してアプリケーションを実行する[**準備**](./2-prepare-development-environment.md)を行います。

- [**演習 1**](./3-exercise-1-run-message-extension.md) では、Microsoft Teamsと Outlook で[メッセージ拡張機能](https://learn.microsoft.com/microsoftteams/platform/messaging-extensions/what-are-messaging-extensions)として同じアプリケーションを実行します。

- [**演習 2**](./4-exercise-2-run-copilot-plugin.md) では、Microsoft 365 用 Copilot のプラグインとしてアプリケーションを実行します。 さまざまなプロンプトを試し、さまざまなパラメータを使用してプラグインがどのように呼び出されるかを確認します。 Copilot とチャットしながら、開発者コンソールを見て、作成されているクエリを確認できます。

- [**演習 3**](./5-exercise-3-add-new-command.md) では、プラグインの機能を拡張し、より多くのタスクを実行できるように、新しいコマンドをアプリケーションに追加する方法について説明します。

  ![製品を表示するアダプティブ カードのスクリーンショット。](../media/1-00-product-card-only.png)

- 最後に、[**演習 4 **](./6-exercise-4-explore-plugin-source-code.md)では、コードのツアーに進み、どのように動作するかを詳しく確認します。 Copilot をまだお持ちでない場合でも、それ以外はすべて Microsoft 365 のメッセージ拡張機能として機能します。

開始する準備ができたら、[次の演習に進んでください...](./2-prepare-development-environment.md)