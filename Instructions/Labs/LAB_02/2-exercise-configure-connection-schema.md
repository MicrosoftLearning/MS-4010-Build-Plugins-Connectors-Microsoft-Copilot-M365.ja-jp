---
lab:
  title: 演習 1 - 外部接続を構成し、スキーマをデプロイする
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 演習 - 外部接続を構成してスキーマをデプロイする

この演習では、カスタムの Microsoft Graph コネクタをコンソール アプリケーションとして構築します。 新しい Microsoft Entra アプリを登録し、外部接続を作成してそのスキーマをデプロイするコードを追加します。

## 開始する前に

このステップは、完了までに約 **XX 分**かかります。

## タスク 1 - 新しい Microsoft Entra アプリを登録する

まず、カスタム Graph コネクタが Microsoft 365 での認証に使用する新しい Entra アプリを登録します。

Web ブラウザーで以下を行います。

1. **Azure portal** ([https://portal.azure.com](https://portal.azure.com)) に移動します。
1. ナビゲーションから **Microsoft Entra ID** を選択します。
1. サイド ナビゲーションから、**[アプリの登録]** を選択します。
1. トップ ナビゲーションで、**[新規登録]** を選択します。
1. 次の値を指定します。
   1. **名前:** MSGraph docs Graph connector
   1. **サポートするアカウント タイプ:** この組織ディレクトリが含むアカウントのみ (1 件のテナント)
1. **[登録]** を選択してエントリを確認します
1. 概要画面で、**"Application ID"** プロパティと **"Directory (tenant) ID"** プロパティの値をコピーします。 これらは後で必要になります。

## タスク 2 - 資格情報を作成する

このカスタム Graph コネクタはユーザー操作なしで実行されるため、自動的に認証されるように構成する必要があります。 わかりやすくするために、シークレットを作成します。

Web ブラウザーでの続行:

1. サイド ナビゲーションで、**[証明書とシークレット]** を選択します。
1. **[クライアント シークレット]** タブをアクティブ化して、**[新しいクライアント シークレット]** ボタンを選択します。
1. **[追加]** を選択してシークレットを作成します。
1. 新しいシークレットの **[値]** をコピーします。 この情報は後で必要になります。

## タスク 3 - API アクセス許可を付与する

Entra アプリの登録を構成する最後の手順は、外部接続とスキーマを作成できるように API アクセス許可を付与することです。

Web ブラウザーでの続行:

1. サイド ナビゲーションで、**[API のアクセス許可]** を選択します。
1. **アクセス許可の追加** を選択します。
1. 利用可能な API の一覧から **[Microsoft Graph]** を選択します。
1. 次に、**[アプリケーションのアクセス許可]** を選択します。
1. フィルター テキスト ボックスに、「**external**」と入力します。
1. **ExternalConnection** セクションを展開し、**ExternalConnection.ReadWrite.OwnedBy** アクセス許可を選択します。
1. **ExternalItem** セクションを展開し、**ExternalItem.ReadWrite.OwnedBy** アクセス許可を選択します。
1. 選択を確定するには、**[アクセス許可の追加]** ボタンを選択します。
1. 構成を完了するには、**[Grant admin consent for (tenant) ((テナント) に対する管理者の同意の付与)]** ボタンを選択して、管理者の同意を付与します。
1. [確認] ダイアログ ボックスで、**[はい]** を選択します。

## タスク 4 - 新しいコンソール アプリを作成して依存関係をインストールする

Entra アプリの登録を構成した後の次の手順は、Graph コネクタのコードを実装するコンソール アプリを作成することです。

ターミナルで次の手順を実行します。

1. 新しいフォルダーを作成し、作業ディレクトリを新しいフォルダーに変更します。
1. `dotnet new console` を実行して新しいコンソール アプリケーションを作成します。
1. 依存関係を追加します。これは、コネクタをビルドするために必要です。
   1. Microsoft 365 で認証するために必要なライブラリを追加するには、`dotnet add package Azure.Identity` を実行します。
   1. Graph API と通信するクライアント ライブラリを追加するには、`dotnet add package Microsoft.Graph` を実行します。
   1. 次の手順で構成するユーザー シークレットを操作するために必要なライブラリを追加するには、`dotnet add package Microsoft.Extensions.Configuration.UserSecrets` を実行します。
   1. Graph コネクタは、2 つのコマンドを使用してコンソール アプリとして実装します。1 つは外部接続を作成してスキーマをデプロイし、もう 1 つはコンテンツをインポートするコマンドです。 アプリでのコマンドの定義をサポートするには、`dotnet add package System.CommandLine --prerelease` を実行します。

## タスク 5 - Entra アプリの登録情報を安全に格納する

Entra アプリの登録を作成した後、アプリケーションとテナント ID、シークレットなどの情報を確認しました。 この情報は、コネクタで Microsoft 365 で認証するために使用します。 コードで使用できるようにするには、プロジェクトをユーザー シークレットとして安全に格納します。

ターミナルで次の手順を実行します。

1. 作業ディレクトリが新しく作成されたコンソール アプリに設定されていることを確認します。
1. ユーザー シークレットを開始するには、`dotnet user-secrets init` を実行します。
1. アプリの登録に関する情報を安全に格納するには、トークンを前にコピーした実際の値に置き換えて次を実行します: 

   ```dotnetcli
   dotnet user-secrets set "EntraId:ClientId" "[application id]"
   dotnet user-secrets set "EntraId:ClientSecret" "[secret value]"
   dotnet user-secrets set "EntraId:TenantId" "[directory (tenant) id]"
   ```

## タスク 6 - Microsoft Graph クライアントを作成する

カスタム Graph コネクタでは、Microsoft Graph API を使用して外部接続と -items を管理します。 まず、プロジェクトにインストールした **Microsoft.Graph** NuGet パッケージから `GraphServiceClient` クラスのインスタンスを作成します。

1. 目的のコード エディタでプロジェクトを開きます。
1. プロジェクトに、**GraphService.cs** という名前の新しいコード ファイルを追加します。
1. ファイルで、使用する名前空間への参照を追加することから始めます。次を追加します。

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   ```

1. 次に、GraphService という名前の新しいクラスを定義します。

   ```csharp
   class GraphService
   {
   }
   ```

1. `GraphService` クラスで、Microsoft Graph API と通信するための `GraphServiceClient` のインスタンスを格納するためのシングルトンを定義します。

   ```csharp
   class GraphService
   {
     static GraphServiceClient? _client;

     public static GraphServiceClient Client
     {
       get
       {
         // TODO: implement
       }
     }
   }
   ```

1. `Client` singleton でゲッターを実装し、まだ存在しない場合は `GraphServiceClient` の新しいインスタンスを作成します。

   ```csharp
   public static GraphServiceClient Client
   {
     get
     {
       if (_client is null)
       {
         // TODO: implement
       }
       return _client;
     }
   }
   ```

1. 資格情報として、前に保存した Entra アプリの登録に関する情報を使用して、`GraphServiceClient` の新しいインスタンスを作成します。

   ```csharp
   var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
   var config = builder.Build();

   var clientId = config["EntraId:ClientId"];
   var clientSecret = config["EntraId:ClientSecret"];
   var tenantId = config["EntraId:TenantId"];

   var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
   _client = new GraphServiceClient(credential);
   ```

   まず、ユーザー シークレットに格納されている Entra アプリ登録に関する情報にアクセスするコンフィギュレーション ビルダーを作成します。 次に、ビルダーを使用してアプリ登録情報を取得します。 次に、テナント ID とクライアント ID、およびクライアント シークレットを渡す新しいクライアント シークレット資格情報を作成します。 最後に、新しく作成した資格情報を渡す `GraphServiceClient` のインスタンスを作成します。

1. 完全なコードは以下のようになります。

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   
   class GraphService
   {
     static GraphServiceClient? _client;
   
     public static GraphServiceClient Client
     {
       get
       {
         if (_client is null)
         {
           var builder = new ConfigurationBuilder().   AddUserSecrets<GraphService>();
           var config = builder.Build();
     
           var clientId = config["EntraId:ClientId"];
           var clientSecret = config["EntraId:ClientSecret"];
           var tenantId = config["EntraId:TenantId"];
           
           var credential = new ClientSecretCredential(tenantId, clientId,    clientSecret);
           _client = new GraphServiceClient(credential);
         }
   
         return _client;
       }
     }
   }
   ```

1. 変更を保存します

## タスク 7 - 外部接続とスキーマ構成を定義する

次の手順では、Graph コネクタで使用する外部接続とスキーマを定義します。 コネクタのコードは複数の場所で外部接続の ID にアクセスする必要があるため、コード内の中央の場所に格納します。

コード エディタで次を行います。

1. **ConnectionConfiguration.cs** という名前の新しいファイルを作成します。
1. Microsoft Graph モデルを使用して名前空間への参照を追加します。

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. 次に、同じファイルで、`ConnectionConfiguration` という名前の新しい静的クラスを定義します。

   ```csharp
   static class ConnectionConfiguration
   {
   
   }
   ```

1.  `ConnectionConfiguration` クラスで、`ExternalConnection` という名前の新しいプロパティを追加します。 これを実装して、`ExternalConnection` Microsoft Graph モデルのインスタンスを返します。

   ```csharp
   public static ExternalConnection ExternalConnection
   {
     get
     {
       return new ExternalConnection
       {
         Id = "msgraphdocs",
         Name = "Microsoft Graph documentation",
         Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
       };
     }
   }
   ```

1. 次に、`Schema` という名前の別のプロパティを追加します。 これを実装して、`Schema` Graph モデルのインスタンスを返します。

   ```csharp
   public static Schema Schema
   {
     get
     {
       return new Schema
       {
         BaseType = "microsoft.graph.externalItem",
         Properties = new()
         {
           // TODO: implement
         }
       };
     }
   }
   ```

1. スキーマで、コネクタを使用して取り込むすべての外部項目について追跡するプロパティを定義します。

   ```csharp
   new Property
   {
     Name = "title",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true,
     Labels = new() { Label.Title }
   },
   new Property
   {
     Name = "description",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true
   },
   new Property
   {
     Name = "iconUrl",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.IconUrl }
   },
   new Property
   {
     Name = "url",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.Url }
   }
   ```

   まず、**title** プロパティを定義します。このプロパティには、Microsoft 365 にインポートされた外部項目のタイトルが格納されます。 項目のタイトルは、フルテキスト インデックス (`IsSearchable = true`) の一部です。 ユーザーは、キーワード クエリ (`IsQueryable = true`) でその内容を明示的に照会することもできます。 タイトルを取得して検索結果に表示することもできます (`IsRetrievable = true`)。 **title** プロパティは、`Title` セマンティック ラベルを使用して指定する項目のタイトルを表します。

   次に、外部項目の内容の概要を格納する **description** プロパティを定義します。 その定義はタイトルに似ています。 ただし、説明にはセマンティック ラベルがないため、定義しません。

   次に、各項目のアイコンの URL を格納するプロパティを定義します。 Microsoft 365 用 Copilot では、このプロパティが必要であり、`IconUrl` セマンティック ラベルを使用してマップする必要があります。

   最後に、外部項目の元の URL を格納する **url** プロパティを定義します。 ユーザーはこの URL を使用して、検索結果から外部項目に移動するか、Microsoft 365 の Copilot に移動します。 URL は、Copilot for Microsoft 365 に必要なプロパティの 1 つです。これが、`Url` セマンティック ラベルを使用してマップする理由です。

1. 完全なコードは以下のようになります。

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionConfiguration
   {
     public static ExternalConnection ExternalConnection
     {
       get
       {
         return new ExternalConnection
         {
           Id = "msgraphdocs",
           Name = "Microsoft Graph documentation",
           Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
         };
       }
     }
     public static Schema Schema
     {
       get
       {
         return new Schema
         {
           BaseType = "microsoft.graph.externalItem",
           Properties = new()
           {
             new Property
             {
               Name = "title",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true,
               Labels = new() { Label.Title }
             },
             new Property
             {
               Name = "description",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true
             },
             new Property
             {
               Name = "iconUrl",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.IconUrl }
             },
             new Property
             {
               Name = "url",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.Url }
             }
           }
         };
       }
     }
   }
   ```

1. 変更を保存します

## タスク 8 - 外部接続を作成する

前のセクションで定義した外部接続に関する情報を使用して、Microsoft 365 で外部接続を作成するコードの追加に進みます。

コード エディタで次を行います。

1. **ConnectionService.cs** という名前の新しいファイルを作成します。
1. ファイルで、まず名前空間への参照を追加します。

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. 次に、同じファイルで、`ConnectionService`という名前の新しい静的クラスを定義します。

   ```csharp
   static class ConnectionService
   {
   
   }
   ```

1. `ConnectionService` クラスで、`CreateConnection` という名前のメソッドを追加します。

   ```csharp
   async static Task CreateConnection()
   {
   }
   ```

1. `CreateConnection` メソッドで、Microsoft Graph クライアント インスタンスを使用して Microsoft Graph API を呼び出し、前に定義した接続情報を使用して外部接続を作成します。

   ```csharp
   Console.Write("Creating connection...");
   
   await GraphService.Client.External.Connections
     .PostAsync(ConnectionConfiguration.ExternalConnection);
   
   Console.WriteLine("DONE");
   ```

1. 完全なコードは以下のようになります。

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   }
   ```

1. 変更を保存。

このコードをテストする前に、スキーマを作成するコードを追加します。 そうすることで、外部接続の作成と構成の完全なフローをテストできます。

## タスク 9 - 外部接続スキーマを作成する

外部接続を作成する最後の部分は、そのスキーマを作成することです。

コード エディタで次を行います。

1. **ConnectionService.cs** ファイルを開きます。
1. ConnectionService クラスに、`CreateSchema`という名前の新しいメソッドを追加します。

   ```csharp
   async static Task CreateSchema()
   {
   }
   ```

1. `CreateSchema` メソッドで、Microsoft Graph クライアント インスタンスを使用して Microsoft Graph API を呼び出してスキーマを作成します。 次に、その作成を待ちます。

   ```csharp
   Console.WriteLine("Creating schema...");
   
   await GraphService.Client.External
     .Connections[ConnectionConfiguration.ExternalConnection.Id]
     .Schema
     .PatchAsync(ConnectionConfiguration.Schema);
   
   do
   {
     var externalConnection = await GraphService.Client.External
       .Connections[ConnectionConfiguration.ExternalConnection.Id]
       .GetAsync();
   
     Console.Write($"State: {externalConnection?.State.ToString()}");
   
     if (externalConnection?.State != ConnectionState.Draft)
     {
       Console.WriteLine();
       break;
     }
   
     Console.WriteLine($". Waiting 60s...");
   
     await Task.Delay(60_000);
   }
   while (true);
   
   Console.WriteLine("DONE");
   ```

1. 同じファイルに、`ProvisionConnection` という名前の新しいメソッドを追加します。 そのコードで、前に定義した `CreateConnection` メソッドと `CreateSchema` メソッドを呼び出します。

   ```csharp
   public static async Task ProvisionConnection()
   {
     try
     {
       await CreateConnection();
       await CreateSchema();
     }
     catch (Exception ex)
     {
       Console.WriteLine(ex.Message);
     }
   }
   ```

1. 完全なコードは以下のようになります。

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   
     async static Task CreateSchema()
     {
       Console.WriteLine("Creating schema...");
   
       await GraphService.Client.External
         .Connections[ConnectionConfiguration.ExternalConnection.Id]
         .Schema
         .PatchAsync(ConnectionConfiguration.Schema);
   
       do
       {
         var externalConnection = await GraphService.Client.External
           .Connections[ConnectionConfiguration.ExternalConnection.Id]
           .GetAsync();
   
         Console.Write($"State: {externalConnection?.State.ToString()}");
   
         if (externalConnection?.State != ConnectionState.Draft)
         {
           Console.WriteLine();
           break;
         }
   
         Console.WriteLine($". Waiting 60s...");
   
         await Task.Delay(60_000);
       }
       while (true);
   
       Console.WriteLine("DONE");
     }
   
     public static async Task ProvisionConnection()
     {
       try
       {
         await CreateConnection();
         await CreateSchema();
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

1. 変更を保存。

## タスク 10 - コードをテストする

最後の手順では、アプリケーションにエントリ ポイントを作成し、接続とそのスキーマを作成します。 これを行うには、コマンド ラインからアプリケーションを起動して呼び出すコマンドを作成します。

コード エディタで次を行います。

1. **Program.cs** ファイルを開きます。
1. その内容を次のコードに置き換えます

   ```csharp
   using System.CommandLine;
   
   var provisionConnectionCommand = new Command("provision-connection",    "Provisions external connection");
   provisionConnectionCommand.SetHandler(ConnectionService.   ProvisionConnection);
   
   var rootCommand = new RootCommand();
   rootCommand.AddCommand(provisionConnectionCommand);
   Environment.Exit(await rootCommand.InvokeAsync(args));
   ```

   まず、`provision-connection.` という名前のコマンドを定義します。このコマンドは、前に定義した `ConnectionService.ProvisionConnection` メソッドを呼び出します。 最後に、コマンド ライン プロセッサにコマンドを登録し、コマンド ラインで渡されたアプリケーション監視引数を開始します。

1. 変更を保存します

アプリケーションをテストするには:

1. ターミナルを開きます。
1. 作業ディレクトリをプロジェクト フォルダーに変更します。
1. `dotnet build` を実行してプロジェクトをビルドします。
1. `dotnet run -- provision-connection` を実行してアプリを起動します。
1. 接続とスキーマが作成されるまで数分待ちます。

[次の演習に進んでください...](./3-exercise-import-external-content.md)