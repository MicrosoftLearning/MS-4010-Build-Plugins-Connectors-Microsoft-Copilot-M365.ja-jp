---
lab:
  title: 演習 2 - 外部コンテンツをインポートする
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 演習 - 外部コンテンツをインポートする

この演習では、ローカル マークダウン ファイルを Microsoft 365 にインポートするコードを使用して、カスタム Microsoft Graph コネクタを拡張します。

## 開始する前に

このステップは、完了までに約 **XX 分**かかります。

## タスク 1 - 外部コンテンツをダウンロードする

この演習に従うには、この演習で使用するサンプル コンテンツ ファイルを [GitHub](https://pnp.github.io/download-partial/?url=https://github.com/pnp/graph-connectors-samples/tree/main/samples/dotnet-csharp-graphdocs/content) からコピーし、プロジェクトの **content** という名前のフォルダーに保存します。

:::image type="content" source="../media/6-content-files.png" alt-text="この演習で使用されるコンテンツ ファイルを示すコード エディタのスクリーンショット。":::

コードが正しく機能するためには、**content** フォルダーとその内容をビルド出力フォルダーにコピーする必要があります。

コード エディタで次を行います。

1. **.csproj** ファイルを開き、`</Project>` タグの前に次のコードを追加します。

   ```xml
   <ItemGroup>
     <ContentFiles Include="content\**"    CopyToOutputDirectory="PreserveNewest" />
   </ItemGroup>
   
   <Target Name="CopyContentFolder" AfterTargets="Build">
     <Copy SourceFiles="@(ContentFiles)" DestinationFiles="@   (ContentFiles->'$(OutputPath)\content\%(RecursiveDir)%(Filename)%   (Extension)')" />
   </Target>
   ```

1. 変更を保存。

## タスク 2 - Markdown と YAML を解析するライブラリを追加する

ビルドしている Microsoft Graph コネクタは、ローカル Markdown ファイルを Microsoft 365 にインポートします。 これらの各ファイルには、YAML 形式のメタデータ (frontmatter とも呼ばれる) を含むヘッダーが含まれています。 さらに、各ファイルの内容は Markdown で書き込まれます。 メタデータを抽出し、本文を HTML に変換するには、カスタム ライブラリを使用します。

1. ターミナルを開いて、作業ディレクトリをプロジェクトに変更します。
1. Markdown 処理ライブラリを追加するには、次のコマンドを実行します: `dotnet add package Markdig`。
1. 次のコマンドを実行して、YAML 処理ライブラリを追加します: `dotnet add package YamlDotNet`。

## タスク 3 - インポートされたファイルを表すクラスを定義する

インポートした Markdown ファイルとその内容の操作を簡略化するために、必要なプロパティを持つクラスを定義しましょう。

コード エディタで次を行います。

1. **ContentService.cs** という名前の新しいファイルを作成します。
1. 次のコードを追加します。

   ```csharp
   using YamlDotNet.Serialization;
   
   public interface IMarkdown
   {
     string? Markdown { get; set; }
   }
   
   class DocsArticle : IMarkdown
   {
     [YamlMember(Alias = "title")]
     public string? Title { get; set; }
     [YamlMember(Alias = "description")]
     public string? Description { get; set; }
     public string? Markdown { get; set; }
     public string? Content { get; set; }
     public string? RelativePath { get; set; }
   }
   ```

   `IMarkdown` インターフェイスは、ローカル Markdown ファイルの内容を表します。 ファイルの内容の逆シリアル化をサポートするには、個別に定義する必要があります。 `DocsArticle` クラスは、解析された YAML プロパティと HTML コンテンツを含む最終的なドキュメントを表します。 `YamlMember` 属性は、各ドキュメントのヘッダー内のメタデータにプロパティをマップします。

1. 変更を保存。

## タスク 4 - `ContentService` クラスを定義する

次に、ローカル Markdown ファイルを読み込み、それらを外部項目に変換して Microsoft 365 に読み込むためのコードを含むクラスをビルドします。

コード エディタで次を行います。

1. **ContentService.cs** ファイルを編集していることを確認します。
1. ファイルの先頭に次のステートメントを追加します。

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. ファイルの末尾に次のコードを追加します。

   ```csharp
   static class ContentService
   {
     static IEnumerable<DocsArticle> Extract()
     {}
   
     static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
     {}
   
     static async Task Load(IEnumerable<ExternalItem> items)
     {}
   
     public static async Task LoadContent()
     {
       var content = Extract();
       var transformed = Transform(content);
       await Load(transformed);
     }
   }
   ```

   `ContentService` クラスは、コンテンツ処理プロセスを表す 3 つのメソッドを定義します。

   1. `Extract`: ローカル Markdown ファイルを読み込み、`DocsArticle` クラスのインスタンスに解析して処理を容易にします。
   1. `Transform`: `DocsArticle` オブジェクトを `ExternalItems` クラスのインスタンスに変換します。これは Microsoft Graph .NET SDK の一部であり、Microsoft 365 に読み込まれる外部項目を表します。
   1. `Load`: Microsoft Graph API を使用して外部項目を Microsoft 365 に読み込みます。

   これらのメソッドは、`LoadContent` メソッドからこの特定の順序で呼び出されます。

1. 変更を保存。

## タスク 5 - Markdown 処理を構成する

まず、ローカル Markdown ファイルからコンテンツを抽出してみましょう。

まず、`Markdig` ライブラリと `YamlDotNet` ライブラリを簡単に使用するためのヘルパー メソッドを追加します。

コード エディタで次を行います。

1. **MarkdownExtensions.cs** という名前の新しいファイルを作成します。
1.  ファイルに、次のコードを追加します。

   ```csharp
   // from: https://khalidabuhakmeh.com/parse-markdown-front-matter-with-csharp
   using Markdig;
   using Markdig.Extensions.Yaml;
   using Markdig.Syntax;
   using YamlDotNet.Serialization;
   
   public static class MarkdownExtensions
   {
     private static readonly IDeserializer YamlDeserializer =
         new DeserializerBuilder()
         .IgnoreUnmatchedProperties()
         .Build();
         
     private static readonly MarkdownPipeline Pipeline
         = new MarkdownPipelineBuilder()
         .UseYamlFrontMatter()
         .Build();
   }
   ```

   `YamlDeserializer` プロパティは、抽出する各 Markdown ファイルの YAML ブロックの新しい逆シリアライザーを定義します。 逆シリアライザーは、ファイルが逆シリアル化されるクラスの一部ではないすべてのプロパティを無視するように構成します。

   `Pipeline` プロパティは、Markdown パーサーの処理パイプラインを定義します。 YAML ヘッダーを解析するように構成します。 この構成がないと、ヘッダーからの情報は破棄されます。

1. 次に、次のコードを使って、`MarkdownExtensions` クラスを拡張します。

   ```csharp
   public static T GetContents<T>(this string markdown) where T :    IMarkdown, new()
   {
     var document = Markdown.Parse(markdown, Pipeline);
     var block = document
         .Descendants<YamlFrontMatterBlock>()
         .FirstOrDefault();
   
     if (block == null)
       return new T { Markdown = markdown };
   
     var yaml =
         block
         // this is not a mistake
         // we have to call .Lines 2x
         .Lines // StringLineGroup[]
         .Lines // StringLine[]
         .OrderByDescending(x => x.Line)
         .Select(x => $"{x}\n")
         .ToList()
         .Select(x => x.Replace("---", string.Empty))
         .Where(x => !string.IsNullOrWhiteSpace(x))
         .Aggregate((s, agg) => agg + s);
   
     var t = YamlDeserializer.Deserialize<T>(yaml);
     t.Markdown = markdown.Substring(block.Span.End + 1);
     return t;
   }
   ```

   `GetContents` メソッドは、ヘッダー内の YAML メタデータを含む Markdown 文字列を、`IMarkdown` インターフェイスを実装する指定された型に変換します。 Markdown 文字列から YAML ヘッダーを抽出し、指定した型に逆シリアル化します。 次に、記事の本文を抽出し、さらに処理するために `Markdown` プロパティに設定します。

1. 変更を保存。

## タスク 6 - Markdown と YAML コンテンツを抽出する

ヘルパー メソッドを配置して、Extract メソッドを実装してローカル Markdown ファイルを読み込み、そこから情報を抽出します。

コード エディタで次を行います。

1. **ContentService.cs** ファイルを開きます。
1. ファイルの先頭に次の using ステートメントを追加します。

   ```csharp
   using Markdig;
   ```

1. 次に、`ContentService` クラスに次のコードを追加して、`Extract` メソッドを実装します。

   ```csharp
   static IEnumerable<DocsArticle> Extract()
   {
     var docs = new List<DocsArticle>();
   
     var contentFolder = "content";
     var contentFolderPath = Path.Combine(Directory.GetCurrentDirectory(),    contentFolder);
     var files = Directory.GetFiles(contentFolder, "*.md", SearchOption.   AllDirectories);
   
     foreach (var file in files)
     {
       try
       {
         var contents = File.ReadAllText(file);
         var doc = contents.GetContents<DocsArticle>();
         doc.Content = Markdown.ToHtml(doc.Markdown ?? "");
         doc.RelativePath = Path.GetRelativePath(contentFolderPath, file);
         docs.Add(doc);
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   
     return docs;
   }
   ```

   このメソッドは、**content** フォルダーから Markdown ファイルを読み込んで開始します。 ファイルごとに、その内容が文字列として読み込まれます。 `MarkdownExtensions` クラスで前に定義した `GetContents` 拡張メソッドを使用して、メタデータとコンテンツを個別のプロパティに格納したオブジェクトに文字列を変換します。 次に、Markdown 文字列を HTML に変換します。 最後に、ファイルへの相対パスを格納し、さらに処理するためにオブジェクトをコレクションに追加します。

1. 変更を保存。

## タスク 7 - コンテンツを外部項目に変換する

外部コンテンツを読み取った後、次の手順では外部項目に変換します。これは Microsoft 365 に読み込まれます。

まず、相対ファイル パスに基づいて外部項目ごとに一意の ID を生成するヘルパー メソッドを追加します。

コード エディタで次を行います。

1. **ContentService.cs** ファイルを編集していることを確認します。
1. `ContentService` クラスで、次のメソッドを追加します。

   ```csharp
   static string GetDocId(string relativePath)
   {
     var id = relativePath.Replace(Path.DirectorySeparatorChar.ToString(),    "__").Replace(".md", "");
     return id;
   }
   ```

   `GetDocId` メソッドは、相対ファイル パスを受け取り、すべてのディレクトリ区切り記号を二重アンダースコアに置き換えます。 これは、外部項目 ID でパス区切り文字を使用できないために必要です。

1. 変更を保存。

次に、ローカル Markdown ファイルを表すオブジェクトを Microsoft Graph の外部項目に変換する `Transform` メソッドを実装します。

コード エディタで次を行います。

1. **ContentService.cs** ファイル内にいることを確認します。
1. 次のコードを使って `Transform` メソッドを実装します。

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var docId = GetDocId(a.RelativePath ?? '');
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
               { "title", a.Title ?? "" },
               { "description", a.Description ?? "" },
               { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md",    "")).ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
             new()
             {
               Type = AclType.Everyone,
               Value = "everyone",
               AccessType = AccessType.Grant
             }
         }
       };
     });
   }
   ```

   最初に、ベース URL を定義します。 この URL を使用して各項目の完全な URL を作成し、項目がユーザーに表示されるときに元の項目に移動できるようにします。 次に、各項目を `DocsArticle` から `ExternalItem`に変換します。 まず、相対ファイル パスに基づいて各項目の一意の ID を取得します。 次に、`ExternalItem` の新しいインスタンスを作成し、そのプロパティに `DocsArticle`からの情報を入力します。 次に、項目のコンテンツをローカル ファイルから抽出された HTML コンテンツに設定し、項目のコンテンツ タイプを HTML に設定します。 最後に、組織内のすべてのユーザーに表示されるように、項目のアクセス許可を構成します。

1. 変更を保存。

## タスク 8 - 外部項目を Microsoft 365 に読み込む

コンテンツを処理する最後の手順では、変換された外部項目を Microsoft 365 に読み込みます。

コード エディタで次を行います。

1. **ContentService.cs** ファイルを編集していることを確認します。
1. `ContentService` クラスに次のコードを追加して、`Load` メソッドを実装します。

   ```csharp
   static async Task Load(IEnumerable<ExternalItem> items)
   {
     foreach (var item in items)
     {
       Console.Write(string.Format("Loading item {0}...", item.Id));
   
       try
       {
         await GraphService.Client.External
           .Connections[Uri.EscapeDataString(ConnectionConfiguration.   ExternalConnection.Id!)]
           .Items[item.Id]
           .PutAsync(item);
   
         Console.WriteLine("DONE");
       }
       catch (Exception ex)
       {
         Console.WriteLine("ERROR");
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

   外部項目ごとに、Microsoft Graph .NET SDK を使用して Microsoft Graph API を呼び出し、項目をアップロードします。 要求では、以前に作成した外部接続の ID、アップロードする項目の ID、および完全な項目の内容を指定します。

1. 変更を保存。

## タスク 9 - コンテンツ読み込みコマンドを追加する

コードをテストする前に、コンテンツ読み込みロジックを呼び出すコマンドを使用してコンソール アプリケーションを拡張する必要があります。

コード エディタで次を行います。

1. **Program.cs** ファイルを開きます。
1. 次のコマンドを使用して、コンテンツを読み込む新しいコマンドを追加します。

    ```csharp
    var loadContentCommand = new Command("load-content", "Loads content   into the external connection");
    loadContentCommand.SetHandler(ContentService.LoadContent);
    ```

1. 次のコードを使用して、新しく定義したコマンドをルート コマンドに登録して、呼び出すことができるようにします。

     ```csharp
     rootCommand.AddCommand(loadContentCommand);
     ```

1. 変更を保存。

## タスク 10 - コードをテストする

最後に、Microsoft Graph コネクタが外部コンテンツを正しくインポートすることをテストします。

1. ターミナルを開きます。
1. 作業ディレクトリをプロジェクトに変更します。
1. `dotnet build` コマンドを実行してプロジェクトをビルドします。
1. `dotnet run -- load-content` コマンドを実行して、コンテンツの読み込みを開始します。
1. コマンドが完了するのを待ち、コンテンツを読み込みます。

[次の演習に進んでください...](./4-exercise-ensure-secure-access.md)